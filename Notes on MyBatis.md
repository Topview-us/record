# Notes on MyBatis

标签（空格分隔）： 未分类

---

##第一章 MyBatis简介
MyBatis 是支持普通 SQL查询，存储过程和高级映射的优秀持久层框架。MyBatis 消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Ordinary Java Objects，普通的 Java对象）映射成数据库中的记录。
每个MyBatis应用程序主要都是使用SqlSessionFactory实例的，一个SqlSessionFactory实例可以通过SqlSessionFactoryBuilder获得。SqlSessionFactoryBuilder可以从一个xml配置文件或者一个预定义的配置类的实例获得。
用xml文件构建SqlSessionFactory实例是非常简单的事情。推荐在这个配置中使用类路径资源（classpath resource)，但你可以使用任何Reader实例，包括用文件路径或file://开头的url创建的实例。MyBatis有一个实用类----Resources，它有很多方法，可以方便地从类路径及其它位置加载资源。

##第二章  MyBatis配置文件

###第一节 走近 mybatis-config.xml

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	
	<typeAliases>
		<package name="com.mybatis.model"/>
	</typeAliases>
	
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driverClassName}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
		
		<environment id="test">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driverClassName}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
		</environments>

	<mappers>
		<package name="com.mybatis.mappers"/>
	</mappers>
</configuration>

```

###第二节 配置文件的介绍
####一. mybatis-config.xml

1，environments:Mybatis支持多个环境

如
```XML
    <environments default="development">
		<environment id="development">
		....
		</environment>
		
		<environment id="test">
		....
		</environment>
	</environments>
```
2.transactionManager 事务管理器 
MyBatis 支持两种类型的事务管理器：JDBC 和MANAGED(托管)；
每个environment都有一个事务管理器，一般为JDBC类型；
JDBC：应用程序负责管理数据库连接的生命周期；
MANAGED ： 由应用服务器负责管理数据库连接的生命周期； ( 一般商业服务器才有此功能， 如JBOSS,WebLogic)
```XML
	<transactionManager type="JDBC" />
```
3.dataSource
用来配置数据源；类型有：UNPOOLED，POOLED，JNDI；
UNPOOLED，没有连接池，每次数据库操作，MyBatis 都会创建一个新的连接，用完后，关闭；适合小并发
项目；
POOLED，用上了连接池；
JNDI，使用应用服务器配置JNDI 数据源获取数据库连接；
```XML
<dataSource type="POOLED">
	 <property name="driver" value="${jdbc.driverClassName}" />
     <property name="url" value="${jdbc.url}" />
     <property name="username" value="${jdbc.username}" />
     <property name="password" value="${jdbc.password}" />
</dataSource>
```		
4.properties
配置属性
```XML
	<properties>
		<property name="jdbc.driverClassName" value="com.mysql.jdbc.Driver"/>
		<property name="jdbc.url" value="jdbc:mysql://localhost:3306/db_mybatis"/>
		<property name="jdbc.username" value="root"/>
		<property name="jdbc.password" value="123456"/>
	</properties>
```
5.typeAliases
给类取别名
原始做法：
```XML
<typeAliases>
  <typeAlias alias="Student" type="com.mybatis.model.Student"/>
</typeAliases>
```
缺点:类过多时费时费力

改进为
```XML
<typeAliases>
<package name="com.mybatis.po">
</typeAliases>
```
执行机制是对整个包下的每一个类都自动的取别名，且对应的别名就是它们的类名

6.mappers
引入映射文件
原始做法：
1.
```XML
<mappers>
<mapper resource="com/mybatis/mappers/StudentMapper.xml"/>
</mappers>
```
2.
```XML
<mappers>
<mapper class="com/mybatis/mappers/StudentMapper"/>
</mappers>
```
改进做法：
```XML
<mappers>
<package name="com.mybatis.mappers">
</mappers>
```
即扫描该包下的所有配置信息

####二. Log4j 配置日志信息
1.导入Log4j的jar包
2.新建log4j.properties
```
log4j.rootLogger=info,appender1,appender2

log4j.appender.appender1=org.apache.log4j.ConsoleAppender //输出到控制台

log4j.appender.appender2=org.apache.log4j.FileAppender 
log4j.appender.appender2.File=C:/logFile.txt   //输出到指定文本
 
log4j.appender.appender1.layout=org.apache.log4j.TTCCLayout
log4j.appender.appender2.layout=org.apache.log4j.TTCCLayout 
```
3.在要输出日志的类中
```
	private static Logger logger=Logger.getLogger(StudentTest.class);
```
然后用info方法输出信息
```
logger.info("添加成功！");
```

此时在控制台和文件logFile.txt中均可看到日志信息


##第三章 获取SqlSession
```
package com.mybatis.util;

import java.io.InputStream;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public class SqlSessionFactoryUtil {

	private static SqlSessionFactory sqlSessionFactory;
	
	public static SqlSessionFactory getSqlSessionFactory(){
		if(sqlSessionFactory==null){
			InputStream inputStream=null;
			try{
				inputStream=Resources.getResourceAsStream("mybatis-config.xml");
				sqlSessionFactory=new SqlSessionFactoryBuilder().build(inputStream);
			}catch(Exception e){
				e.printStackTrace();
			}
		}
		return sqlSessionFactory;
	}
	
	public static SqlSession openSession(){
		return getSqlSessionFactory().openSession();
	}
}

```

SqlSession的作用：在Test Case中用来获取Mapper,如：
```
package com.mybatis.test;

import org.apache.ibatis.session.SqlSession;
import com.mybatis.mappers.StudentMapper;
import com.mybatis.po.Student;
import com.mybatis.utils.SqlSessionFactoryUtil;

public class StudentTest {
    private static Logger logger=Logger.getLogger(StudentTest.class);

public static void main(String[] args) {
	SqlSession sqlSession=SqlSessionFactoryUtil.openSession();
	StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
	Student Jack=new Student(11,"Jack");
	int result=studentMapper.add(Jack);
	sqlSession.commit();   //此处不提交无法将数据搞进数据库
	if(result==1){
	logger.info("添加成功！");
	}
}
}
	}

```


##第四章.mappers

###第一节.走近mappers
1.namespace
用namespace绑定接口后，可以不用写接口实现类，mybatis会通过该绑定自动
找到对应要执行的SQL语句，如下：
假设定义了AddressMapper接口
```
package com.mybatis.mappers;

import com.mybatis.model.Address;

public interface AddressMapper {

	public Address findById(Integer id);

}
```
 
对于映射文件AddressMapper.xml如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mappers.AddressMapper">
	
	<select id="findById" parameterType="Integer" resultType="Address">
		select * from t_address where id=#{id}
	</select>
</mapper> 
```
**注意接口中的方法与映射文件中的SQL语句的ID一一对应 。**
则在代码中可以直接使用AddressMapper面向接口编程而不需要再编写实现类。

2.parameterType
MyBatis的传入参数parameterType类型分两种
   1. 1. 基本数据类型：int,string,long,Date;
   1. 2. 复杂数据类型：类和Map
   注：不同版本的MyBatis对基本类型传递过来的参数名称不能识别，要使用_parameter来代替。
```
<select id="getWinLogByEventId" parameterType="java.lang.Long" resultMap="BaseResultMap">  
    select <include refid="Base_Column_List"/> from win_log where eventId = #{_parameter,jdbcType=BIGINT}  
</select>  
```

###第二节.在mapper中写sql
先看接口
```
package com.mybatis.mappers;

import com.mybatis.po.Student;

public int add(Student student);
	
	public int update(Student student);
	
	public int delete(Integer id);
	
	public Student findById(Integer id);
	
	public List<Student> find();
	
	public Student findStudentWithAddress(Integer id);
	
	public Student findByGradeId(Integer gradeId);
}
```
1.新增数据
```
<insert id="add" parameterType="Student">
insert into t_student values(null,#{name},#{age});
</insert>
```
2.删除数据
```
<delete id="delete" parameterType="Integer">
delete from t_student where id=#{id};
</delete>
```
3.修改数据
```
<update id="modify" parameterType="Student" >
update t_student set name=#{name},age=#{age} where id=#{id}
</update>
```
3.查找数据
(1)查找单个学生
```
<select id="findById" parameterType="Integer" resultType="Student">
select * from t_student where id=#{id}
</select>
```

(2)查找学生以集合形式返回
第一步：建立映射关系
```
<resultMap type="Student" id="StudentResult">
<id property="id" column="id"/>  //对应数据库中表的主键
<result property="name" column="name"/>
<result property="age" column="age"/>
</resultMap>
```
第二步：
```
<select id="find" resultMap="StudentResult">
select * from t_student
```

###第三节.MyBatis 关系映射
(1)一对一

POJO Student的主要属性
```
	private Integer id;
	private int age;
	private String name;
	private Address address;
	]
```
新建一个POJO Address,主要属性有
```
private int id;
private String province;
private String city;
private String district;
private Address address;
```

**关联**
方法1：
```
<resultMap type="Student" id="StudentResult">
	<id property="id" column="id"/>
	<result property="name" column="name"/>
	<result property="age" column="age"/>
	
	<result property="address.id" column="address_id"/>
	<result property="address.province" column="province"/>
	<result property="address.city" column="city"/>
	<result property="address.district" column="district"/>
	</resultMap>
```

方法2：
```
<resultMap type="Address" id="AddressResult">
	<result property="id" column="add_id"/>
	<result property="province" column="province"/>
	<result property="city" column="city"/>
	<result property="district" column="district"/>
	</resultMap>
	
	<resultMap type="Student" id="StudentResult">
	<id property="id" column="id"/>
	<result property="name" column="name"/>
	<result property="age" column="age"/>
	<association property="address" resultMap="AddressResult"> </association>
	</resultMap>
```
方法3：
```
<resultMap type="Student" id="StudentResult">
	<id property="id" column="id"/>
	<result property="name" column="name"/>
	<result property="age" column="age"/>
	
	<association property="address" javaType="Address"> 
	<result property="id" column="add_id"/>
	<result property="province" column="province"/>
	<result property="city" column="city"/>
	<result property="district" column="district"/>
	</association>
	</resultMap>
```

**合并查询**
```
<select resultMap="StudentResult" id="findStudentWithAddress" parameterType="Integer">
	select * from t_student t1,t_address t2 where t1.address_id=t2.add_id and t1.id=#{id}
</select>
```
**测试代码**：
```
@Test
	public void testFindStudentWithAddress() {
		Student student=studentMapper.findStudentWithAddress(3); //传入的是student的id
		sqlSession.commit();  
		System.out.println(student);
	}
```
测试结果
```
Student [id=3, age=11, name=王五, address=Address [id=1, province=广东省, city=广州市, district=番禺区]]
```

**为追求模块化开发，以上三种方法均不用，请看方法4(吐血)：**

新建Address对应文件
1.AddressMapper.java
```
package com.mybatis.mappers;
import com.mybatis.po.Address;

public interface AddressMapper {
public Address findById(Integer id);
}
```
2.AddressMapper.xml  
(注意要在mybatis.xml中配置该mapper，不然会抛org.apache.ibatis.exceptions.PersistenceException: )
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mappers.AddressMapper">

	<resultMap type="Address" id="AddressResult">
	<result property="id" column="add_id"/>
	<result property="province" column="province"/>
	<result property="city" column="city"/>
	<result property="district" column="district"/>
	</resultMap>
	
	<select id="findById" parameterType="Integer" resultType="Address">
	select * from t_address where add_id=#{id}
	</select>
</mapper> 
```

StudentMapper.xml
```
<resultMap type="Student" id="StudentResult">
	<id property="id" column="id"/>
	<result property="name" column="name"/>
	<result property="age" column="age"/>
	<association property="address" column="address_id" select="com.mybatis.mappers.AddressMapper.findById" ></association>
	</resultMap>
```
合并与测试略过

(2)一对多
新建一个POJO Grade，其主要属性有：
```
private Integer id;
private String gradeName;
private List<Student> students;
```
GradeMapper.java
```
package com.mybatis.mappers;

import com.mybatis.po.Grade;

public interface GradeMapper {

public Grade findById(Integer gradeId);

}
```
GradeMapper.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mappers.GradeMapper">

	<resultMap type="Grade" id="GradeResult">
	<result property="id" column="g_id"/>
	<result property="gradeName" column="grade_name"/>
	<collection property="students" column="g_id" select="com.mybatis.mappers.StudentMapper.findStudentByGradeId"> </collection>
	</resultMap>
	
	<select id="findById" parameterType="Integer" resultMap="GradeResult">
	select * from t_grade where g_id=#{gradeId}
	</select>	
</mapper> 
```
**注意** collection中的column填入的是t_grade的主键

StudentMapper。java
```
package com.mybatis.mappers;

import java.util.List;

import com.mybatis.po.Student;

public interface StudentMapper {
public Student findStudentByGradeId(Integer gradeId);
}
```

StudentMapper。xml
```
	<resultMap type="Student" id="StudentResult">
	<id property="id" column="id"/>
	<result property="name" column="name"/>
	<result property="age" column="age"/>
	<association property="address" column="address_id" select="com.mybatis.mappers.AddressMapper.findById" ></association>
	</resultMap>
	
	<select resultMap="StudentResult" id="findStudentWithAddress" parameterType="Integer">
	select * from t_student t1,t_address t2 where t1.address_id=t2.add_id and t1.id=#{id}
	</select>
	
	<select id="findStudentByGradeId" parameterType="Integer" resultMap="StudentResult">
	select * from t_student where grade_id=#{gradeId}
	</select>
```

测试类
```
package com.mybatis.test;

import org.apache.ibatis.session.SqlSession;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import com.mybatis.mappers.GradeMapper;
import com.mybatis.po.Grade;
import com.mybatis.utils.SqlSessionFactoryUtil;

public class GradeTest {
	SqlSession sqlSession=null;
    GradeMapper gradeMapper=null;
	
	@Before
	public void setUp() throws Exception {
		sqlSession=SqlSessionFactoryUtil.openSession();
		gradeMapper=sqlSession.getMapper(GradeMapper.class);
	}

	@After
	public void tearDown() throws Exception {
		sqlSession.close();
	}

	@Test
	public void testFindStudentByGradeId() {
		Grade grade=gradeMapper.findById(1);
		sqlSession.commit();  
		System.out.println(grade);
	}
}
```

测试结果
```
Grade [id=1, gradeName=大一, students=[Student [id=3, age=11, name=王五, address=Address [id=1, province=广东省, city=广州市, district=番禺区]], Student [id=4, age=11, name=王五五, address=Address [id=2, province=广东省, city=深圳市, district=宝安区]]]]
```

分析
findById(1)-->GradeMapper.xml-->select * from t_grade where g_id=#{gradeId}-->resultMap(gradeResult)

g_id-->Student的collection  (一对多)

collection中的每个学生的address_id-->address （一对一）


##第五章.动态SQL

t_student:
| id       | name  |  age | grade_id |
| --------   | ----: | :----:  |:----: |
| 3   | 王五|   11   |1 |
| 4       |   王五五  |   11   |1 |
| 5      |    里大大  |  12 |2 |
|6    | 陈时  |  32 | 2 |

###第一节.if条件
StudentMapper.java
```
package com.mybatis.mappers;

import java.util.List;
import java.util.Map;

import com.mybatis.po.Student;

public interface StudentMapper {
public List<Student> searchStudents(Map<String,Object> map);
}
```
StudentMapper.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.mybatis.mappers.StudentMapper">
  
	<resultMap type="Student" id="StudentResult">
	<id property="id" column="id"/>
	<result property="name" column="name"/>
	<result property="age" column="age"/>
	</resultMap>
	
	<select resultMap="StudentResult" id="searchStudents" parameterType="Map">
	select * from t_student where grade_id=#{gradeId}
	<if test="name!=null">
	and name like #{name}
	</if>
	<if test="age!=null">
	and age=#{age}
	</if>
	</select>	
</mapper> 
```
测试：
```
@Test
	public void testFind() {
		logger.info("查询学生");
		Map<String,Object> map=new HashMap<String,Object>();
		map.put("gradeId", 1);
		//map.put("name", "%王%");
		map.put("age", 11);
		List<Student> studentList=studentMapper.searchStudents(map);
		sqlSession.commit();  //此处不提交无法将数据搞进数据库
		for(Student s:studentList){
			System.out.println(s);
		}
	}
```

测试结果
```：
[main] INFO com.mybatis.test.StudentTest2 - 查询学生
Student [id=3, age=11, name=王五]
Student [id=4, age=11, name=王五五]
```

###第二节 choose，when 和otherwise 条件
StudentMapper里新增一个方法
```
public List<Student> searchStudents2(Map<String,Object> map);
```
StudentMapper.xml
```
<select resultMap="StudentResult" id="searchStudents2" parameterType="Map">
	select * from t_student 
	<choose>
	<when test="searchBy=='gradeId'">
	where grade_id=#{gradeId}
	</when>
	
	<when test="searchBy=='name'">
	where name like #{name}
	</when>

	<otherwise>
	where age=#{age}
	</otherwise>
	</choose>
</select>
```
测试：
```
	@Test
	public void testSearchStudents2() {
		logger.info("查询学生2");
		Map<String,Object> map=new HashMap<String,Object>();
		map.put("searchBy", "gradeId");
	    map.put("gradeId", 1);
//		map.put("name", "%王%");
		map.put("age", 11);
		List<Student> studentList=studentMapper.searchStudents2(map);
		sqlSession.commit(); 
		for(Student s:studentList){
			System.out.println(s);
		}
	}
```

测试结果：
```
[main] INFO com.mybatis.test.StudentTest2 - 查询学生2
Student [id=3, age=11, name=王五]
Student [id=4, age=11, name=王五五]
```

###第三节 where 条件
当where与and/or冲突时，会自动将最前面的and/or去掉
StudentMapper.xml
```
	<select resultMap="StudentResult" id="searchStudents3" parameterType="Map">
	select * from t_student 
	<where>
	<if test="gradeId!=null">
	grade_id=#{gradeId}
	</if>
	
	<if test="name!=null">
	and name like #{name}
	</if>
	
	<if test="age!=null">
	and age=#{age}
	</if>	
	</where>
	</select>
```
测试类似，不贴代码了

###第四节 trim 条件
prefix表示没有时会自动加上的前缀，prefixOverrides是会”智能的“覆盖掉的词，当然还有与后缀相关的用法，不过一般少遇见

```
<select id="searchStudents4" parameterType="Map" resultMap="StudentResult">
		select * from t_student 
		 <trim prefix="where" prefixOverrides="and|or">
			 <if test="gradeId!=null">
			 	grade_id=#{gradeId}
			 </if>
			 <if test="name!=null">
			 	and name like #{name}
			 </if>
			 <if test="age!=null">
			 	and age=#{age}
			 </if>
		 </trim>
	</select>
```


###第五节.foreach 循环
```
<select id="searchStudents5" parameterType="Map" resultMap="StudentResult">
		select * from t_student 
		 <if test="gradeIds!=null">
		 	<where>
		 		grade_id in 
		 		<foreach item="gradeId" collection="gradeIds" open="(" separator="," close=")">
		 		 #{gradeId}
		 		</foreach>
		 	</where>
		 </if>
	</select>
```

测试：
```
	@Test
	public void testSearchStudents5() {
		logger.info("查询学生5");
		List<Integer> gradeIds=new ArrayList<Integer>();
//		gradeIds.add(1);
		gradeIds.add(2);		
		Map<String,Object> map=new HashMap<String,Object>();
		map.put("gradeIds", gradeIds);
		List<Student> studentList=studentMapper.searchStudents5(map);
		sqlSession.commit(); 
		for(Student s:studentList){
			System.out.println(s);
		}
	}
```

测试结果：
```
[main] INFO com.mybatis.test.StudentTest2 - 查询学生5
Student [id=3, age=11, name=王五]
Student [id=4, age=11, name=王五五]

```

###第六节.set 条件
1，自动加上where；
2，如果where 子句以and 或者or 开头，则自动删除第一个and 或者or；
功能和where 元素类似，提供了前缀，后缀功能，更加灵活；
1，自动加上set；
2，自动剔除最后一个逗号“，”；
```
public int updateStudent(Student student);
```
```
<update id="updateStudent" parameterType="Student">
		update t_student
		<set>
		 <if test="name!=null">
		 	name=#{name},
		 </if>
		 <if test="age!=null">
		 	age=#{age},
		 </if>
		</set>
		where id=#{id}
	</update>
```

测试：
```
	@Test
	public void testUpdateStudent(){
		logger.info("更新学生(带条件)");
		Student student=new Student();
		student.setId(3);
		student.setName("张三3");
		student.setAge(13);
		studentMapper.updateStudent(student);
		sqlSession.commit();
	}
```



##第六章 Mybatis 杂项

###第一节.处理CLOB、BLOB 类型数据
1.插入数据
POJO Student的属性(对应数据库的字段)
```
	private Integer id;
	private int age;
	private String name;
	private String remark;
	private byte[] pic;
```
StudentMapper接口方法
```
public int insertStudent(Student student);
```
StudentMapper.xml
二进制图片用byte数组获取，直接#{pic}就行，强
```
<insert id="insertStudent" parameterType="Student">
insert into t_student values(null,#{name},#{age},#{pic},#{remark});
</insert>
```

测试方法：
```
	@Test
	public void testInsertStudent(){
		logger.info("添加学生");
		Student student=new Student(4,"刘六","很长很长的文本.........");
		byte[]pic=null;
		try{
			File file=new File("c://kobe.jpg");
		    InputStream inputStream=new FileInputStream(file);
			pic=new byte[inputStream.available()];
			inputStream.read(pic);
			inputStream.close();
		}catch(Exception e){
			e.printStackTrace();
		}
		student.setPic(pic);
		studentMapper.insertStudent(student);
		sqlSession.commit();
	}
```

2.获取数据

接口方法
```
public Student getStudent(Integer id);
```
StudentMapper.xml

```
<select id="getStudent" parameterType="Integer" resultType="Student">
	select * from t_student where id=#{id}
	</select>
```

测试
```
	@Test
	public void testGetStudent(){
		logger.info("通过id查找学生");
		Student student=studentMapper.getStudent(2);
		System.out.println(student);
		byte[]pic=student.getPic();
		try{
			File file=new File("D://kobe1.jpg");
		    OutputStream outputStream=new FileOutputStream(file);
		    outputStream.write(pic);
		    outputStream.close();
		}catch(Exception e){
			e.printStackTrace();
		}
	}
```
测试结果：pic要找到 D://kobe1.jpg
```
Student [id=2, age=4, name=刘六, remark=很长很长的文本.........]
```

###第二节.传入多个输入参数
主要用Map或HashMap,还有一种#{param}方法，了解即可

接口方法
```
public List<Student> searchStudents6(String name,int age);
```
StudentMapper.xml
```
<select id="searchStudents6"  resultMap="StudentResult">
	select * from t_student where name like #{param1} and age=#{param2}
	</select>
```
测试
```
	@Test
	public void testSearchStudent6(){
		logger.info("通过两个参数查找学生");
		List<Student> studentList=studentMapper.searchStudents6("%刘%",4);
		for(Student s:studentList){
		System.out.println(s);
 	}
}
```

###第三节.Mybatis 分页

t_student
| id       | name  |  age | 
| --------   | ----: | :----:  |
| 1  | 李一|   11   |
| 2     |   陈二 |   22  |
| 3     |   张三 | 33|
|4  | 李四  |  44|
|5  | 王五  |  55|
|6  | 刘六  |  66|

1，逻辑分页；先把所有数据放到内存里再取相应的数据

接口方法：
```
public List<Student> findStudents1(RowBounds rowBounds);
```
StudentMapper.xml
```
<select id="findStudents1"  resultMap="StudentResult">
	select * from t_student
</select>
```
测试：
```
	@Test
	public void testFindStudents1(){
		logger.info("逻辑分页查询学生");
		int offset=2,limit=3;
		RowBounds rowBounds=new RowBounds(offset,limit);
		List<Student> studentList=studentMapper.findStudents1(rowBounds);
		for(Student s:studentList){
		System.out.println(s);
	}	
```

测试结果:
```
[main] INFO com.mybatis.test.StudentTest2 - 分页查询学生
Student [id=3, age=33, name=张三]
Student [id=4, age=44, name=李四]
Student [id=5, age=55, name=王五]
```

2，物理分页(常用)

接口方法：
```
public List<Student> findStudents2(Map<String,Object> map);
```
StudentMapper.xml
```
	<select id="findStudents2" parameterType="Map" resultMap="StudentResult">
	select * from t_student
	<if test="start!=null and size!=null"> 
	limit #{start},#{size}
	</if>	
	</select>
```

测试：
```
		@Test
		public void testFindStudents2(){
			logger.info("物理分页查询学生");
			Map<String,Object> map=new HashMap<String,Object>();
	        map.put("start", 0);
		    map.put("size", 3);
			List<Student> studentList=studentMapper.findStudents2(map);
			for(Student s:studentList){
			System.out.println(s);
		}	
}
```

测试结果:
```
[main] INFO com.mybatis.test.StudentTest2 - 物理分页查询学生
Student [id=1, age=11, name=李一]
Student [id=2, age=22, name=陈二]
Student [id=3, age=33, name=张三]
```

###第四节 Mybatis 缓存

Mybatis 默认情况下，MyBatis 启用一级缓存，即同一个SqlSession 接口对象调用了相同的select语句，则直接会从缓存中返回结果，而不是再查询一次数据库；开发者可以自己配置二级缓存，二级缓存是全局的；
默认情况下，select 使用缓存的，insert update delete 是不使用缓存的；

一般小项目不用特殊配置缓存，按照默认的即可,当然也可在XxxMapper.xml中如配置
```
<!--
    	1，size:表示缓存cache中能容纳的最大元素数。默认是1024；
    	2，flushInterval:定义缓存刷新周期，以毫秒计；
     	3，eviction:定义缓存的移除机制；默认是LRU(least recently userd，最近最少使用),还有FIFO(first in first out，先进先出)
     	4，readOnly:默认值是false，假如是true的话，缓存只能读。
     -->
	<cache size="1024" flushInterval="60000" eviction="LRU" readOnly="false"/>
```
查询语句是默认使用缓存且不清除缓存的
```	
	<select id="findStudents" resultMap="StudentResult" flushCache="false" useCache="true">
		select * from t_student
	</select>
```
insert update delete 语句默认清除缓存
```
<insert id="insertStudent" parameterType="Student" flushCache="true">
		insert into t_student values(null,#{name},#{age},#{pic},#{remark});
	</insert>
```

##第七章 使用注解配置SQL 映射器 (少用)

###第一节：基本映射语句
1，@Insert
StudentMapper.java
```
@Insert("insert into t_student values(null,#{name},#{age})")
	public int insertStudent(Student student);
```
测试
```
	@Test
	public void testInsert(){
		logger.info("添加学生");
		Student student=new Student(77,"朱七");
		int result=studentMapper.insertStudent(student);
		sqlSession.commit();  //变更操作要提交事务
```
2，@Update
```
    @Update("update t_student set name=#{name},age=#{age} where id=#{id}")
	public int updateStudent(Student student);
```

测试：
```	
	@Test
	public void testUpdate(){
		logger.info("更新学生");
		Student student=new Student(12,7777,"朱七");
		int result=studentMapper.updateStudent(student);
		sqlSession.commit();
	}
```

3，@Delete
```
@Delete("delete from t_student where id=#{id}")
	public int deleteStudent(Integer id);
```

测试：
```
	@Test
	public void testDelete(){
		logger.info("删除学生");		
		int result=studentMapper.deleteStudent(13);
		sqlSession.commit();
}
```
4，@Select
(1)查找单个学生
```
	@Select("select * from t_student where id=#{id}")
	public Student getStudentById(int id);
```	
测试
```
		@Test
	public void testGetStudentByid(){
		logger.info("查找学生");		
		Student student=studentMapper.getStudentById(12);
		System.out.println(student);
}
```
]
(2)查找所有学生
```
	@Select("select * from t_student")
	@Results(
			{
				@Result(id=true,column="id",property="id"),
				@Result(column="name",property="name"),
				@Result(column="age",property="age")
			}
			)
	public List<Student> getStdudents();
```
测试
```
	@Test
	public void testGetStudents(){
		logger.info("查找所有学生");		
		List<Student> studentList=studentMapper.getStdudents();
	for(Student s:studentList){
		System.out.println(s);
	}
}
```

###第二节：结果集映射语句


###第三节：关系映射
1，一对一映射
```
@Select("select * from t_student where id=#{id}")
	@Results(
			{
				@Result(id=true,column="id",property="id"),
				@Result(column="name",property="name"),
				@Result(column="age",property="age"),
				@Result(column="address_id",property="address",one=@One(select="com.mybatis.mappers.AddressMapper.getAddressById"))
			}
			)
	public Student findStudentWithAddress(Integer id);
```

测试
```
	@Test
	public void testfindStudentWithAddress(){
		logger.info("带地址查询学生");
		Student student=studentMapper.findStudentWithAddress(3);
		System.out.println(student);
}
```
2，一对多映射；
AddressMapper
```
package com.mybatis.mappers;

import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;

import com.mybatis.po.Address;

public interface AddressMapper {
	@Results(
			{
				@Result(id=true,column="add_id",property="id"),
				@Result(column="proince",property="proince"),
				@Result(column="city",property="city"),
				@Result(column="district",property="district"),
			}
)	
	@Select("select * from t_address where add_id=#{id}")
	public Address getAddressById(Integer id);
}
```
GradeMapper
```
package com.mybatis.mappers;

import org.apache.ibatis.annotations.Many;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;

import com.mybatis.po.Grade;

public interface GradeMapper {
		
	@Select("select * from t_grade where g_id=#{id}")
	@Results(
			{
				@Result(id=true,column="g_id",property="id"),
				@Result(column="grade_name",property="gradeName"),
				@Result(column="g_id",property="students",many=@Many(select="com.mybatis.mappers.StudentMapper.findStudentsByGradeId"))
			}
)
	
	public Grade getById(Integer id);

}
```
StudentMapper
```
package com.mybatis.mappers;

import java.util.List;

import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Many;
import org.apache.ibatis.annotations.One;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;

import com.mybatis.po.Student;

public interface StudentMapper {
	
	@Select("select * from t_student where grade_id=#{gradeId}")
	@Results(
			{
				@Result(id=true,column="id",property="id"),
				@Result(column="name",property="name"),
				@Result(column="age",property="age"),
				@Result(column="grade_id",property="gradeId"),
				@Result(column="address_id",property="address",one=@One(select="com.mybatis.mappers.AddressMapper.getAddressById"))
			}
			)	
	public Student findStudentsByGradeId(int gradeId);
}
```
测试：
```
	@Test
	public void testGetById(){
		logger.info("查询年级带学生");
		Grade grade=gradeMapper.getById(2);
		System.out.println(grade);
		List<Student> list=grade.getStudents();
		for(Student s: list){
			System.out.println(s);
		}
}
```
测试结果:
```
Student [id=3, age=22, name=陈二, address=Address [id=1, province=广东省, city=广州市, district=番禺区], gradeId=2]
Student [id=4, age=33, name=陈三, address=Address [id=2, province=广东省, city=深圳市, district=宝安区], gradeId=2]
Student [id=5, age=44, name=陈四, address=Address [id=2, province=广东省, city=深圳市, district=宝安区], gradeId=2]
Student [id=6, age=55, name=陈五, address=Address [id=2, province=广东省, city=深圳市, district=宝安区], gradeId=2]
```

###第四节：动态SQL
一. @InsertProvider

StudentDynaSqlProvider
```
package com.mybatis.mappers;

import org.apache.ibatis.jdbc.SQL;

import com.mybatis.po.Student;

public class StudentDynaSqlProvider {

	public String insertStudent(final Student student){
		return new SQL(){
			{
				INSERT_INTO("t_student");
				if(student.getName()!=null){
					VALUES("name", "#{name}");
				}
				if(student.getAge()!=null){
					VALUES("age", "#{age}");
				}
			}
		}.toString();
	}
}
```
StudentMapper
```
@InsertProvider(type=StudentDynaSqlProvider.class,method="insertStudent")
	public int insertStudent(Student student);
```
测试略过

二 .@UpdateProvider
StudentDynaSqlProvider
```
public String updateStudent(final Student student){
		return new SQL(){
			{
				UPDATE("t_student");
				if(student.getName()!=null){
					SET("name=#{name}");
				}
				if(student.getAge()!=null){
					SET("age=#{age}");
				}
				WHERE("id=#{id}");
			}
		}.toString();
	}
```
StudentMapper
```
@UpdateProvider(type=StudentDynaSqlProvider.class,method="updateStudent")
	public int updateStudent(Student student);
	
```
三.@DeleteProvider
StudentDynaSqlProvider
```
	public String deleteStudent(){
		return new SQL(){
			{
				DELETE_FROM("t_student");
				WHERE("id=#{id}");
			}
		}.toString();
	}
```
StudentMapper
```
	@DeleteProvider(type=StudentDynaSqlProvider.class,method="deleteStudent")
	public int deleteStudent(Integer id);
```
测试：
```
int result=studentMapper.deleteStudent(2);
sqlSession.commit();
```

四. @SelectProvider
(1)按id查询学生
StudentMapper
```
@SelectProvider(type=StudentDynaSqlProvider.class,method="findStudentById")
	public Student findStudentById(Integer id);
```
StudentDynaSqlProvider
```
	public String findStudentById(){
		return new SQL(){
			{
				SELECT("*");
				FROM("t_student");
				WHERE("id=#{id}");
			}
		}.toString();
	}
```
测试
```
	@Test
	public void testSelect(){
		logger.info("查询学生");
	    Student student=studentMapper.findStudentById(3);
		System.out.println(student);
}
```

(2)按多条件查询学生
StudentMapper
```
@SelectProvider(type=StudentDynaSqlProvider.class,method="findStudents")
	public List<Student> findStudents(Map<String,Object> map);
```

StudentDynaSqlProvider
```
public String findStudents(final Map<String,Object> map){
		return new SQL(){
			{
				SELECT("*");
				FROM("t_student");
				StringBuffer sb=new StringBuffer();
				if(map.get("name")!=null){
					sb.append("and name like '"+map.get("name")+"'");
				}
				if(map.get("age")!=null){
					sb.append("and age="+map.get("age"));
				}
			    if(!sb.toString().equals("")){
				WHERE(sb.toString().replaceFirst("and", ""));
			}
			}
		}.toString();
	}
```
测试
```
	@Test
	public void testSelects(){
		logger.info("查询学生");
		Map<String,Object> map=new HashMap<>();
		map.put("name", "%三%");
	    List<Student> studentList=studentMapper.findStudents(map);
		for(Student s: studentList){
			System.out.println(s);
		}
}
```
测试结果
```
[main] INFO com.mybatis.test.StudentTest - 查询学生
Student [id=4, age=33, name=陈三]
```

##第八章Mybatis 与Spring，SpringMvc 整合

###第一节：Spring 与SpringMvc 整合
###第二节：Spring 与Mybatis 整合