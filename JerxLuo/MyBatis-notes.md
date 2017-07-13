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
####一.mybatis-config.xml

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

####二.Log4j 配置日志信息
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

##第二节.在mapper中写sql
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

####4.MyBatis 关系映射
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
第一节.if条件
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

第二节.choose，when 和otherwise 条件
第三节.where 条件
第四节.trim 条件
第五节.foreach 循环
第六节.set 条件
1，自动加上where；
2，如果where 子句以and 或者or 开头，则自动删除第一个and 或者or；
功能和where 元素类似，提供了前缀，后缀功能，更加灵活；
1，自动加上set；
2，自动剔除最后一个逗号“，”；
