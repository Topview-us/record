

# MyBatis

## 第一讲：MyBatis简介及配置

### 1、简介：

​	MyBatis 是支持 普通 SQL查询，存储过程 和 高级映射 的优秀持久层框架。MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及对结果集的检索封装。MyBatis 可以使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的 POJO（Plain Old Java Objects，普通的 Java 对象）映射成数据库中的记录namespace （确保是唯一的）：包+文件名（不含后缀）（例：userMapper）

### 2、配置文件conf.xml

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
 <!-- 
 	development: 开发模式
 	work： 工作模式
  -->
<configuration> 
	<environments default="development">
		<environment id="development"> 
			<transactionManager type="JDBC" /> 
			<dataSource type="POOLED">
              <property name="driver" value="com.mysql.jdbc.Driver" />
              <property name="url" value="jdbc:mysql://localhost:3306/xxxxx" /> 
              <property name="username" value="root" /> 
              <property name="password" value="123456" /> 
			</dataSource> 
		</environment> 
	</environments> 
  	<mappers>
        <mapper resource="com/atguigu/mybatis_test/test1/userMapper.xml"/> 
    </mappers>
</configuration>
```



### 3、在 conf.xml 文件中注册 userMapper.xml 文件

**（注意相对conf.xml的路径与直接路径，maven项目直接放在资源文件夹中，只需部署相对路径`resource="userMapper.xml"`，以下为直接路径）**

```xml
<mappers>
	<mapper resource="com/atguigu/mybatis_test/test1/userMapper.xml"/> 
</mappers>
```



### 4、定义操作users表的sql 映射文件 userMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace=" com.atguigu.mybatis_test.test1.userMapper">
	<select id="getUser" parameterType="int" resultType="com.atguigu.mybatis_test.test1.User"> 
		select * from users where id=#{id} 
	</select> 
</mapper>
```

resultType：查询返回的结果类型，全类名（反射）（类的copy qualified name）



## 第二讲：定义增删查改操作

**`@test`为test 测试的注解**

### 1、在xml中定义

```xml
<mapper namespace="MyBatis_maven.MyBatis_maven_test.userMapper">
	<insert id="addUser" parameterType="MyBatis_maven.MyBatis_maven_test.User">
		insert into users(name, age) values(#{name}, #{age}); 
	</insert>
  
	<delete id="deleteUser" parameterType="int">
		delete from users where id = #{id};
	</delete>
	
	<update id="updateUser">
		update users set name = #{name} , age = #{age} where id = #{id};
	</update>

	<select id="getAllUser" parameterType="int" resultType="MyBatis_maven.MyBatis_maven_test.User"> 
		select * from users 
	</select> 
</mapper>
```

如图，parameterType是变量的类型，resultType是返回变量的类型，**谨记要在conf.xml中注册映射的xml文件**

如此定义操作应在java文件中编码：

```java
@Test
public void testDelete(){
  String resource = "conf.xml"; 
  InputStream is = Test.class.getClassLoader().getResourceAsStream(resource);//加载 mybatis 的配置文件（它也加载关联的映射文件）
  SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is); //构建 sqlSession 的工厂 
  SqlSession session = sessionFactory.openSession(); //创建能执行映射文件中 sql 的 sqlSession,默认手动提交
  String statement = "MyBatis_maven.MyBatis_maven_test.userMapper.deleteUser";//命名空间（namespace） + 操作id
  int delete = session.delete(statement, 7);
  session.commit();//提交
  System.out.println(delete);
}
```

**其中sessionFactory.openSession()有多个重载方法，其中布尔类型的参数指定是否能将操作自动提交到数据库的表中，自动提交的参数为true，而平时默认为手动提交，需要在后面加上`session.commit `手动提交**



### 2、用注解定义



创建一个interface：

```java
public interface UserMap {
	@Insert("insert into users(name, age) values(#{name}, #{age})")
	public int addUser(User user);
}
```

然后要在conf.xml注册，为class：

```xml
<mappers>
  <mapper resource="userMapper.xml"/>
  <mapper resource="userMapper2.xml"/>  
  <mapper class="MyBatis_maven.MyBatis_maven_test.UserMap"/>
</mappers>
```

如此定义操作应在java文件中编码：

```java
@Test
public void testNote(){
  String resource = "conf.xml"; 
  InputStream is = Test.class.getClassLoader().getResourceAsStream(resource);//加载 mybatis 的配置文件（它也加载关联的映射文件）
  SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is); //构建 sqlSession 的工厂 
  SqlSession session = sessionFactory.openSession(true); //创建能执行映射文件中 sql 的 sqlSession
  UserMap usermap = session.getMapper(UserMap.class);//接口实体类
  int insert = usermap.addUser(new User("lili", 15));
  System.out.println(insert);
}
```

### 3、创建接口并在beanmapper.xml内定义

创建接口

```java
public interface UserMap {
	public int selectUser(int id);
}
```

beanmapper.xml定义（然后在conf.xml中注册）

```xml
<select id="getAllUser" parameterType="int" resultType="MyBatis_maven.MyBatis_maven_test.User"> 
  select * from users 
</select>
```

 

## 第三讲、conf.xml的一些其他属性

（摘自钊雄...)

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

（1）例子：

可先创建一个properties文件，如db.properties

```properties
Driver = com.mysql.jdbc.Driver
url = jdbc:mysql://localhost:3306/mybatis-test
name = root
password = 123456
```

然后在conf.xml中配置

```xml
<properties resource="db.properties"/>
```

则可将数据库连接的四个参数改写为

```xml
<property name="driver" value="${Driver}" />
<property name="url" value="${url}" /> 
<property name="username" value="${name}" /> 
<property name="password" value="${password}" /> 
```

（2）log4j.properties

```properties
log4j.properties， 
log4j.rootLogger=DEBUG, Console 
#Console 
log4j.appender.Console=org.apache.log4j.ConsoleAppender 
log4j.appender.Console.layout=org.apache.log4j.PatternLayout 
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n 
log4j.logger.java.sql.ResultSet=INFO 
log4j.logger.org.apache=INFO 
log4j.logger.java.sql.Connection=DEBUG 
log4j.logger.java.sql.Statement=DEBUG 
log4j.logger.java.sql.PreparedStatement=DEBUG
```

然后在conf.xml配置

```xml
<properties resource="log4j.properties"/>
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

该属性可以给包中的类注册别名，注册后可以直接使用类名，而不用使用全限定的类名（就是不用包含包名)



另：log4j用xml配置（用这个）

```xml
<?xml version="1.0" encoding="UTF-8" ?> 
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd"> 
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/"> 
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender"> 
      <layout class="org.apache.log4j.PatternLayout"> 
        <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m  (%F:%L) \n" />
       </layout> 
    </appender> 
  <logger name="java.sql"> 
    <level value="debug" />
  </logger>
  <logger name="org.apache.ibatis"> 
    <level value="debug" /> 
  </logger> 
  <root> 
    <level value="debug" /> 
    <appender-ref ref="STDOUT" /> 
  </root> 
</log4j:configuration>
```





## 第四讲：解决数据库表字段名与实体类属性名不相同的冲突

**select返回po类型会有冲突这种情况，返回基本类型则不会产生**



假设有表如下：

```sql
CREATE TABLE orders( 
  order_id INT PRIMARY KEY AUTO_INCREMENT, 
  order_no VARCHAR(20), 
  order_price FLOAT
); 
  INSERT INTO orders(order_no, order_price) VALUES('aaaa', 23); 
  INSERT INTO orders(order_no, order_price) VALUES('bbbb', 33);
  INSERT INTO orders(order_no, order_price) VALUES('cccc', 22);
```

实体类如下：

```java
public class Order { 
  private int id;
  private String orderNo;
  private float price;
}
```

则要执行查询操作需要有：

```xml
<!-- 方式一: 通过在 sql 语句中定义别 -->
<select id="selectOrder" parameterType="int" resultType="_Order"> 
  select order_id id, order_no orderNo,order_price price from orders where order_id=#{id} 
</select>

<!-- 方式二: 通过<resultMap> -->
<select id="selectOrderResultMap" parameterType="int" resultMap="orderResultMap">
  select * from orders where order_id=#{id} 
</select>

<!-- 
	reslutMap : 封装一些映射关系
	id ： 专门针对主键
	result ： 针对其他字段
-->
<resultMap type="_Order" id="orderResultMap"> 
  <id property="id" column="order_id"/>
  <result property="orderNo" column="order_no"/> 
  <result property="price" column="order_price"/>
</resultMap>
```



## 第五讲、一对一查询

association 用于一对一的关联查询

property 对象属性的名称

javaType 对象属性的类型

column 所对应的外键字段名称

select 使用另一个查询封装的结果



```xml
<!-- 
 方式一：嵌套结果：
 使用嵌套结果映射来处理重复的联合结果的子集      
  封装联表查询的数据(去除重复的数据) 
 select * from class c, teacher t where c.teacher_id=t.t_id and  c.c_id=1 
-->
<select id="getClass" parameterType="int" resultMap="ClassResultMap">
  select * from class c, teacher t where c.teacher_id=t.t_id and  c.c_id=1
</select>
<resultMap type="Classes" id="ClassResultMap">
  <id property="id" column="c_id"/>
  <result property="name" column="c_name"/>
  <association property="teacher" column="teacher_id" javaType="Teacher">
    <id property="id" column="t_id"/>
    <result property="name" column="t_name"/>
  </association>
</resultMap>
```



```xml
<!-- 
 方式二：嵌套查询：通过执行另外一个 SQL 映射语句来返回预期的复杂类
 SELECT * FROM class WHERE c_id=1;
 SELECT * FROM teacher WHERE t_id=1   //1 是上一个查询得到的 teacher_id 的值
-->

<select id="getClass2" parameterType="int" resultMap="ClassResultMap2">
  select * from class where c_id = #{id}
</select>

<select id="getTeacher" parameterType="int" resultType="Teacher">
  select t_id id , t_name name from teacher where t_id = #{id}
</select>

<resultMap type="Classes" id="ClassResultMap2">
  <id property="id" column="c_id"/>
  <result property="name" column="c_name"/>
  <association property="teacher" column="teacher_id" javaType="Teacher" select="getTeacher"></association>
  <!-- 从teacher_id获得值给第二条select语句使用 -->
</resultMap>
```



## 第六讲、一对多查询

```xml
<!-- 
  方式一: 
  嵌套结果: 使用嵌套结果映射来处理重复的联合结果的子集 
  SELECT * FROM class c, teacher t,student s WHERE c.teacher_id=t.t_id AND c.C_id=s.class_id AND  c.c_id=#{id}
-->
<select id="getClassAndStudent" parameterType="int" resultMap="getClassResultMap">
  SELECT * FROM class c , teacher t , student s WHERE c.teacher_id = t.t_id and c.c_id = s.class_id AND c.c_id = #{id}	
</select>

<resultMap type="Classes" id="getClassResultMap">
  <id property="id" column="c_id"/>
  <result property="name" column="c_name"/>
  
  <association property="teacher" column="teacher_id"  javaType="Teacher">
    <id property="id" column="t_id"/>
    <result property="name" column="t_name"/>
  </association>
  
  <!-- ofType 指定 students 集合中的对象类型 --> 
  <collection property="list" ofType="Student">
    <id property="id" column="s_id"/>
    <result property="name" column="s_name"/>
  </collection>
  
</resultMap>
```



```xml
<!-- 
  方式二：嵌套查询：通过执行另外一个 SQL 映射语句来返回预期的复杂类型 （不建议使用）
  SELECT * FROM class WHERE c_id=1;
  SELECT * FROM teacher WHERE t_id=1  //1 是上一个查询得到的 teacher_id 的值 
  SELECT * FROM student WHERE class_id=1  //1 是第一个查询得到的 c_id 字段的值
  -->

<select id="getClass" parameterType="int" resultMap="getClassResult">
  select * from class where c_id = #{id};
</select>

<select id="getTeacher" parameterType="int" resultType="Teacher">
  select t_id id , t_name name from teacher where t_id = #{id};
</select>

<select id="getStudent" parameterType="int" resultType="Student">
  select s_id id , s_name name from student where class_id = #{id};
</select>

<resultMap type="Classes" id="getClassResult">
  <id property="id" column="c_id"/>
  <result property="name" column="c_name"/>
  <association property="teacher" column="teacher_id" javaType="Teacher" select="getTeacher">
  </association>
  <collection property="list" column="c_id" ofType="Student" select="getStudent">
  </collection>
</resultMap>

```



| property      | 映射数据库列的字段或属性。如果JavaBean 的属性与给定的名称匹配，就会使用匹配的名字。否则，MyBatis 将搜索给定名称的字段。两种情况下您都可以使用逗点的属性形式。比如，您可以映射到“username”，也可以映射到“address.street.number”。 |
| ------------- | :--------------------------------------- |
| **column**    | **数据库的列名或者列标签别名。与传递给resultSet.getString(columnName)的参数名称相同。** |
| **javaType**  | **完整[Java](http://lib.csdn.net/base/java)类名或别名(参考上面的内置别名列表)。如果映射到一个JavaBean，那MyBatis 通常会自行检测到。然而，如果映射到一个HashMap，那您应该明确指定javaType 来确保所需行为。** |
| **select**    | **通过这个属性，通过ID引用另一个加载复杂类型的映射语句。从指定列属性中返回的值，将作为参数设置给目标select 语句。表格下方将有一个例子。注意：在处理组合键时，您可以使用column=”{prop1=col1,prop2=col2}”这样的语法，设置多个列名传入到嵌套语句。这就会把prop1和prop2设置到目标嵌套语句的参数对象中。** |
| **resultMap** | **一个可以映射联合嵌套结果集到一个适合的对象视图上的ResultMap 。这是一个替代的方式去调用另一个select 语句。它允许您去联合多个表到一个结果集里。这样的结果集可能包括冗余的、重复的需要分解和正确映射到一个嵌套对象视图的数据组。简言之，MyBatis 让您把结果映射‘链接’到一起，用来处理嵌套结果。** |

详细查询：[这里](http://blog.csdn.net/wxwzy738/article/details/24742495)



## 第七讲、动态sql与模糊查询

例：（关键是映射文件的sql语句有区别）

需求：实现多条件查询用户(姓名模糊匹配, 年龄在指定的最小值到最大值之间）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.day03_mybatis.test6.userMapper">
 <select id="getUser" parameterType="com.atguigu.day03_mybatis.test6.ConditionUser"     
   resultType="com.atguigu.day03_mybatis.test6.User"> 
   select * from d_user where age between #{minAge} and #{maxAge}
   <--!
        判断name是否为空
   -->
   <if test='name!="%null%"'>
     and name like #{name}
   </if> 
 </select> 
</mapper>
```

其余的还有choose, when, otherwise等标签，可在文档内查询使用方法



## 第八讲、调用存储过程

**mapper.xml内：**

```xml

<select id="getSexCount" parameterMap="getSexMap" statementType="CALLABLE">
	CALL ges_user_count(?,?);
</select>

<parameterMap type="java.util.Map" id="getSexMap">
  <parameter property="sexid" mode="IN" jdbcType="INTEGER"/>
  <parameter property="usercount" mode="OUT" jdbcType="INTEGER"/>
</parameterMap>
```

支持的jdbc类型可在文档中查找

mode：标识in 和 out 类型

**java内：**

```java
@Test
	public void testSelectSexCount(){
		
      String resource = "conf.xml"; 
      InputStream is = Test.class.getClassLoader().getResourceAsStream(resource);//加载 mybatis 的配置文件（它也加载关联的映射文件）
      SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is); //构建 sqlSession 的工厂 
      SqlSession session = sessionFactory.openSession(true); //创建能执行映射文件中 sql 的 sqlSession,默认手动提交
      String statement = "MyBatis_maven.MyBatis_maven_test.userMapper4.getSexCount";
      Map<String, Integer> parameterMap = new HashMap<String, Integer>();
      parameterMap.put("sexid", 1);
      parameterMap.put("usercount", -1);
      session.selectOne(statement, parameterMap);
      System.out.println(parameterMap.get("usercount"));
      session.close();
    }
```



## 第九讲、一级缓存和二级缓存

1. 一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空
2. 二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于 其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache
3. 对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存 Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。



### 1、一级缓存

例子：

（1）清除缓存：

java中测试

```java
/** 查询一个cuser**/

@Test
public void testCacheOne(){

  String resource = "conf.xml";
  InputStream is = Test.class.getClassLoader().getResourceAsStream(resource);
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  SqlSession session = factory.openSession(true);
  String statement = "MyBatis_maven.MyBatis_maven_test.userMapper5.getUser";
  Cuser cuser = session.selectOne(statement, 1);
  System.out.println(cuser);
  session.clearCache();//清除缓存
  cuser = session.selectOne(statement, 1);
  System.out.println(cuser);
  session.close();
}
```

测试结果：

```
DEBUG 07-14 23:16:07,506 ==>  Preparing: select * from c_user where id=?   (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:16:07,547 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:16:07,570 <==      Total: 1  (BaseJdbcLogger.java:145) 
Cuser [id=1, name=Tom, age=12]
DEBUG 07-14 23:16:07,572 ==>  Preparing: select * from c_user where id=?   (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:16:07,573 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:16:07,574 <==      Total: 1  (BaseJdbcLogger.java:145) 
Cuser [id=1, name=Tom, age=12]
DEBUG 07-14 23:16:07,575 Closing JDBC Connection [com.mysql.jdbc.Connection@1134fada]  (JdbcTransaction.java:90) 
DEBUG 07-14 23:16:07,575 Returned connection 288684762 to pool.  (PooledDataSource.java:344) 
```

（2）未清除缓存

```
DEBUG 07-14 23:18:37,974 ==>  Preparing: select * from c_user where id=?   (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:18:38,015 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:18:38,059 <==      Total: 1  (BaseJdbcLogger.java:145) 
Cuser [id=1, name=Tom, age=12]
Cuser [id=1, name=Tom, age=12]
```



### 2、二级缓存

**如果开启了二级缓存，那么在关闭sqlsession后，会把该sqlsession一级缓存中的数据添加到namespace的二级缓存中**

（1）在mapper.xml中添加<cache>来开启二级缓存（默认是关闭的）

```xml
<mapper namespace="com.atguigu.mybatis.test8.userMapper"> 
  <cache/>
```

（2）java中测试

```java
@Test
public void testCacheTwo(){
  
  String resource = "conf.xml";
  InputStream is = Test.class.getClassLoader().getResourceAsStream(resource);
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  SqlSession session1 = factory.openSession();
  SqlSession session2 = factory.openSession();
  String statement = "MyBatis_maven.MyBatis_maven_test.userMapper5.getUser";
  Cuser user = session1.selectOne(statement, 1);
  session1.commit();
  System.out.println(user);
  user = session2.selectOne(statement, 1);
  session2.commit();
  System.out.println(user);
  
}
```

测试结果：

```
DEBUG 07-14 23:22:03,529 ==>  Preparing: select * from c_user where id=?   (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:22:03,562 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:145) 
DEBUG 07-14 23:22:03,598 <==      Total: 1  (BaseJdbcLogger.java:145) 
Cuser [id=1, name=Tom, age=12]
DEBUG 07-14 23:22:03,606 Cache Hit Ratio [MyBatis_maven.MyBatis_maven_test.userMapper5]: 0.5  (LoggingCache.java:62) 
Cuser [id=1, name=Tom, age=12]
```

**更具体的内容查文档**







## 另：关于sqlsession与xml的关系

### 1、创建sqlsession

一切以java代码为主，Sqlsession对应着一次数据库会话。由于数据库回话不是永久的，因此Sqlsession的生命周期也不应该是永久的，相反，在你每次访问数据库时都需要创建它（当然并不是说在Sqlsession里只能执行一次sql，你可以执行多次，当一旦关闭了Sqlsession就需要重新创建它）。创建Sqlsession的地方只有一个，那就是SqlsessionFactory的openSession方法。所以，当需要生成sqlsession时，我们要实现opensession的方法，而opensession的方法中有以下过程：

1)       从配置中获取Environment；

2)       从Environment中取得DataSource；

3)       从Environment中取得TransactionFactory；

4)       从DataSource里获取数据库连接对象Connection；

5)       在取得的数据库连接上创建事务对象Transaction；

6)       创建Executor对象（该对象非常重要，事实上sqlsession的所有操作都是通过它完成的）；

7)       创建sqlsession对象。

也即opensession方法的实现需要有Environment、DataSource等信息资源，因而配置了conf.xml来满足这个需求，通过以下三行代码来找到conf.xml文件并成功创建sqlsession

```java
String resource = "conf.xml"; 
InputStream is = Test.class.getClassLoader().getResourceAsStream(resource);//加载 mybatis 的配置文件（它也加载关联的映射文件）,并将配置文件转换为流给工厂读取
SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is); //构建 sqlSession 的工厂 
SqlSession session = sessionFactory.openSession(); //创建能执行映射文件中 sql 的 sqlSession
```



### 2、执行语句

因为我们需要用代码执行增删改查的操作，因此创建了另一个资源文件，即beanMapper.xml文件。namespace，即命名空间，是这个xml资源空间的名字，在这个空间即xml文件中存放着各种操作的id、内容等。当在conf.xml文件中映射这些mapper.xml文件，即如下：

```xml
<mappers>
  <mapper resource="userMapper.xml"/>
  <mapper resource="userMapper2.xml"/>
  <mapper class="MyBatis_maven.MyBatis_maven_test.UserMap"/>
  <mapper resource="userMapper3.xml"/>
</mappers>
```

映射文件也会随之被加载，在java代码的statement中声明执行哪个命名空间的哪个操作来完成操作，返回所需的对象信息，如下代码：

```java
String statement = "MyBatis_maven.MyBatis_maven_test.userMapper.deleteUser";
int delete = session.delete(statement, 7);
session.commit();//提交
```

不确定是不是这样，具体源码尚未查看...

**注：同一个命名空间中不能有相同的操作id**

​	**对于mybatis的select，实体类中要有空的构造方法，而java内默认有空的构造方法**





