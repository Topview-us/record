## MyBatis-note

## 2.1 添加jar包

（1）导入jar
（2）若是maven项目直接添加依赖

- mysql的jar也要添加

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.1</version>
</dependency>
```

## 2.2 建库+表

```mysql
create database mybatis;
use mybatis;
CREATE TABLE users(id INT PRIMARY KEY AUTO_INCREMENT, NAME VARCHAR(20), age INT);
```

## 2.3 添加MyBatis的配置文件conf.xml到src目录下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--约束-->
<configuration>
	<environments default="development">
      <!--
		工作模式:work
		开发模式:development
-->
		<environment id="development">
			<transactionManager type="JDBC" />
			<!--
			事务管理类型：
				JDBC
				MANAGED
-->
			<dataSource type="POOLED">
              <!--
			数据源类型：
				POOLED：使用连接池
				UNPOOLED：不使用连接池
-->
			<!--
			数据源的属性：
			driver
			url
			username
			password
-->
				<property name="driver" value="com.mysql.jdbc.Driver" />
				<property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
				<property name="username" value="root" />
				<property name="password" value="root" />
			</dataSource>
		</environment>
	</environments>
</configuration>
```

## 2.4 定义表所对应的实体类

## `重要`2.5 定义操作users表的sql映射文件userMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
<mapper namespace="com.atguigu.mybatis_test.test1.userMapper"> 
	<select id="getUser" parameterType="int" 
		resultType="com.atguigu.mybatis_test.test1.User">
      <!--select标签
		resultType是由反射实现的因此应该写该类的全类名
-->
		select * from users where id=#{id}<!--占位符-->
	</select>
</mapper>
```

namespace是id，要确保他是唯一的，故其值应该是包名加这个文件名不加后缀（com,zttc.itat.mybatis.UserMapper）
映射文件放在与User同级目录下

## 2.6 在conf.xml文件中注册userMapper.xml

```xml
 <mappers>
		<mapper resource="zttc/itat/user/mybatis/userMapper.xml"/>
		<!-- 是路径结构而不是包的结构 ，应该是这个mapper相对于conf.xml的相对路径-->
	</mappers> 
```

`重要`**conf.xml和userMapper.xml此类资源文件都会放在maven项目的src/main/resources项目下，而这里mapper的resources的值是一个相对路径**
最好不要放在其他文件下

## 小结（理解）
- -准备：conf.xml注册------->Mapper.xml的定义（namespace和mapper）
  - 1.conf.xml里面----->注册userMapper.xml资源文件
  - 2.userMapper.xml里面----->定义这个文件：
    - 2.1 最基本的两个部分：
      - 2.1.1.（mapper的属性值）namespace:命名空间，要指定映射的接口类类名（全类名）
      - 2.1.2（mapper标签里面的的一些设置）执行SQL语句是怎么定义的
    - 2.2 一个标签可以映射接口的一个方法，一般其id可以与该方法同名比较方便
      - 2.3select标签里面写入语句
    - 2.4ps:语句格式：（注意语句格式#{id}用占位符来作为该方法的参数
- 开始：Resource------->Factory------->session------->mapper
  - 1.由Resource对象获取资源文件conf.xml的一个输入流，将输入流传给建造SqlSessionFactory
     一个数据会话工厂类的一个构造器（builder）以获得一个工厂类 
  - 2.由这个工厂类得到一个session
  - 3.将UserMapper对象的类对象（里面有UserMapper对象的信息）传入     -       session的getMapper方法
     （返回一个泛型，可以用接口引用）得到一个mapper
  - 4.这个mapper对象的方法可以根据userMapper.xml的定义得到预期的结果
##优化
 ### 配置文件单独放在一个properties
 1.在conf.xml同级目录写下文件jdbc.properties
 2.在conf.xml设置，然后用EL表达式使用
	<properties resource="jdbc.properties"></properties>
	<propertiy name="driver" value="${driver}"/>
 ### 为实体类定义别名
	<typeAliases>
		<!-- <typeAlias type="zttc.itat.user.mybatis.User" alias="_User"/> -->
		<package name="zttc.itat.user.mybatis"/>
	</typeAliases>
##解决字段名与实体类属性名不同的冲突
###解决方式1
直接在sql语句上设置别名
`	select order_id id, order_no orderNo,order_price price from orders where order_id=#{id}`
###解决方式2
在mapper.xml里面的resultMap标签设置键值对，然后使想要执行的语句所在标签加上属性（resultMap的id值）
```xml
<select id="selectOrderResultMap" parameterType="int" resultMap="orderResultMap">
	select * from orders where order_id=#{id}
</select>
<resultMap type="_Order" id="orderResultMap">
	<id property="id" column="order_id"/>
	<result property="orderNo" column="order_no"/>
	<result property="price" column="order_price"/>
</resultMap>
```
