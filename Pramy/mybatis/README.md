# 一   mybatisConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  
  <properties resource="db.properties"/>
  
  <typeAliases>  
      <package name="${moduleLocation}"/>     
      <typeAlias alias="User" type="com.pramy.module.User"/>  
      <!--还有一种方法用注解@ -->
  </typeAliases> 
  
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driverClass}"/>
                <property name="url" value="${jdbc.jdbcUrl}"/>
                <property name="username" value="${jdbc.user}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
  
  <mappers>
    <mapper class="com.pramy.mapper.UserMapper"/>
    <mapper resource="com/pramy/mapper/UserMapper.xml"/>
    <mapper url="file:///E:/UserMapper.xml"/> 
    <package name="${mapperLocation}"/> 
  </mappers>
</configuration>
```

##    同目录下的db.properties

```xml
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.jdbcUrl=jdbc:mysql://localhost:3306/test
jdbc.user=root
jdbc.password=root


moduleLocation=com.pramy.module
mapperLocation=com.pramy.mapper
```

**用db.properties的好处就是可以统一路径，修改方便，那么配置文件就可以直接用EL表达式来获取值**



1.```typeAliases```是指定bean别名，如果再这里配置好bean的别名，然后在后面的**BeanMapper.xml**中**ResultType**可以直接填别名。

方法一：可以通过```typeAlias```指明一个bean

方法二：也可以直接指定```package```的名字， mybatis会自动扫描你指定包下面的javabean,  并且默认设置一个别名，默认的名字为： javabean 的首字母小写的非限定类名来作为它的别名

方法三：也可在javabean 加上**注解@Alias** 来自定义别名， 例如： @Alias(user)

2.```environments```一般设置为development，```transactionManager```一般为JDBC

3.```dataSource```的Type设置为POOLED意思是采用连接池，UNPOOLED是不采用

4.```mapper```设置：

- 第一种class注册是采用package路径+Mapper名字
- 第二种resource是采用相对路径指定一个xml
- 第三种url用绝对路径指定xml
- 第四种是指定package路径，扫描改路径下的所有xml

# 二   beanMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
   
<mapper namespace="com.mybatis.entity.UserMapper">  
    <insert id="insertUser" parameterType="User" useGeneratedKeys="true" keyColumn="id">  
       insert into t_user(name, age) values(#{name}, #{age})  
    </insert>  
     
    <update id="updateUser" parameterType="User">  
       update t_user set name=#{name}, age=#{age} where id=#{id}  
    </update>  
     
    <select id="findById" parameterType="int" resultType="User">  
       select * from t_user where id=#{id}  
    </select>  
     
    <delete id="deleteUser" parameterType="int">  
       delete from t_user where id=#{id}  
    </delete>  
</mapper> 
```

1.**namespace**是指定该beanMapper.xml的位置

2.```parameterType```指定传入参数的类型，```resultType```是返回值的类型，如果有在conf.xml中指定typeAliases的话，这里就可以用别名，如果没有，就得用**全类名**

3.#{name}是一个占位符

# 三 创建SqlSession

使用了工厂设计模式，最好用静态代码块去加载SqlSessionFactory，用到的Resource是**org.apache.ibatis.io.Resources**

```java
Reader reader = Resources.getResourceAsReader("mybatisConfig.xml");
SessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
SqlSession sqlSession = sessionFactory.openSession();
//这默认是手动提交 后面要sqlSession.commit才提交数据，设置为true就会自动提交
```



获得SqlSeesion**(以后要细看源码实现方式)**

然后通过反射来加载和绑定beanMapper

```java
 UserMapper userMapper = session.getMapper(UserMapper.class);
        userMapper.insert(user);
```

其中UserMapper是一个interface，绑定了userMapper.xml**（问题：实现原理。实现过程）**

 

