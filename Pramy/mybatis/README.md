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

- 第一种class注册是采用全类名 beanMapper.xml的namespace要与接口一一对应
- 第二种resource是采用相对路径指定一个xml
- 第三种url用绝对路径指定xml
- 第四种是指定package路径，扫描改路径下的所有xml，beanMapper.xml的namespace要与接口的路径一一对应

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

1.**namespace**是指定该需要绑定的类的位置

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

# 四 总结SqlSession和mybatisConfig.xml和beanMapper.xml关系。

- 有两种方法实现mybatis的sql工作	

  - 第一种：

    ```java
    sqlSession.selectOne("com.pramy.UserMapper.selectByPrimaryKey",1)
    ```

    ​通过namespace加id找到xml中的方法来实现。这种办法namespace不一定要指向接口类

  - 第二种：

    ```java
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    userMapper.selectByPrimaryKey(1);
    ```

    通过反射来实现。

- mybatisConfig.xml中的**Mapper**是先通过找到xml的位置,通过namespace来查找对应路径下的接口


# 五  加载方式浅谈



## mybatis会从xml解析一系列参数到configure(org.apache.ibatis.session.configure)对象中.

其中主要来研究一下Mapper的加载方式

### 首先创建工厂的时候会回用到SqlSessionFactory的build方法

```java
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      //这里的parser已经读取到mybatis的配置文件，下面开始讲怎么读
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
   
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
```



### XMLConfigureBuile

里面有一个加载xml的方法

```java
private void parseConfiguration(XNode root) {
  try {
    Properties settings = settingsAsPropertiess(root.evalNode("settings"));
    //issue #117 read properties first
    propertiesElement(root.evalNode("properties"));
    loadCustomVfs(settings);
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    mapperElement(root.evalNode("mappers")); //这里开始注册beanMapper.xml
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```

构建configure要用到XMLConfigureBuiler对象里面用到一个方法

```java
mapperElement(root.evalNode("mappers"));
```

向下找这个方法

```java
private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        //先用检验是否用package方式来注册
        if ("package".equals(child.getName())) {
          //用（XNode）的方式去找package标签的value
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          //用（XNode）的方式去找resource标签的value
          String resource = child.getStringAttribute("resource");
          //用（XNode）的方式去找url标签的value
          String url = child.getStringAttribute("url");
          //用（XNode）的方式去找class标签的value
          String mapperClass = child.getStringAttribute("class");
          //然后再用resource加载，这里用到的是 断位与：&& （&&与&的区别）
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse(); //这个方法开始读取beanMapp.xml
            
            //然后到url
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse(); //这个方法开始读取beanMapp.xml
            
          } else if (resource == null && url == null && mapperClass != null) {
            //然后到class
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

**加载优先级package>resource>url>class**

可以看出除了package和class是以```configuration.addMapper(mapperInterface)``` 方式来加载Mapper

### XNode

构建XNode的方式——它对应的内容是xml里面的内容

```java
  //成员变量
  private Node node;
  private String name;
  private String body;
  private Properties attributes;
  private Properties variables;   
  private XPathParser xpathParser;

//构建方法
public XNode(XPathParser xpathParser, Node node, Properties variables) {
    this.xpathParser = xpathParser;
    this.node = node;
    this.name = node.getNodeName(); //获得的就是根节点 
    this.variables = variables;
    this.attributes = parseAttributes(node);
    this.body = parseBody(node);
  }

//根节点的类型和值记住 
  private Properties parseAttributes(Node n) {
    Properties attributes = new Properties();
    NamedNodeMap attributeNodes = n.getAttributes();
    if (attributeNodes != null) {
      for (int i = 0; i < attributeNodes.getLength(); i++) {
        Node attribute = attributeNodes.item(i);
        String value = PropertyParser.parse(attribute.getNodeValue(), variables);
        attributes.put(attribute.getNodeName(), value);
      }
    }
    return attributes;
  }

  private String parseBody(Node node) {
    String data = getBodyData(node);
    if (data == null) {
      NodeList children = node.getChildNodes();
      for (int i = 0; i < children.getLength(); i++) {
        Node child = children.item(i);
        data = getBodyData(child);
        if (data != null) {
          break;
        }
      }
    }
    return data;
  }
```

在构造方法中调用了本身的几个方法去获得name，attributes，body

这个document是通过parse模块来把xml转成document，从而将信息存在在document中，回到XMLMapperBuilder

所有信息存储：configure-->XMLConfigureBuiler-->XPathParser-->document-->xml

然后一XNode的方法去一个个读取每一个根和它的值

XNode是由node构建的，而node是以Xpath方式去构建的

```java
Node node =xpath.evaluate(root, document, returnType);
```



### XMLMapperBuilder



#### configure里面的 mapperElement方法构建完XMLMapperBuilder后，执行XMLMapperBuilder的parse()方法，这个方法又调用parser（XPathParser）的evalNode方法

```java
//这个方法是根据：resourcse去加载beanMapper.xml，通过namespace绑定接口


public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
      //parser.evalNode("/mapper")返回来的是一个XNode类,由Node类构建，而Node又以Xpath方式去以根节点（/mapper）+document去构建。
      configurationElement(parser.evalNode("/mapper"));
      configuration.addLoadedResource(resource);
      //绑定的的最开始入口
      bindMapperForNamespace();
    }
	
    parsePendingResultMaps();
    parsePendingChacheRefs();
    parsePendingStatements();
  }
//绑定的方法
  private void bindMapperForNamespace() {
    String namespace = builderAssistant.getCurrentNamespace();
    if (namespace != null) {
      Class<?> boundType = null;
      try {
        boundType = Resources.classForName(namespace);
      } catch (ClassNotFoundException e) {
        //ignore, bound type is not required
      }
      if (boundType != null) {
        if (!configuration.hasMapper(boundType)) {
          // Spring may not know the real resource name so we set a flag
          // to prevent loading again this resource from the mapper interface
          // look at MapperAnnotationBuilder#loadXmlResource
          configuration.addLoadedResource("namespace:" + namespace);
          configuration.addMapper(boundType);
        }
      }
    }
  }
```



### 总结：通过把一个xml转成document，然后存在对象中，让后以一个个XNode对象去读取每一个元素，





### 在mybatis的源码中有一个MapperRegistry(**映射器注册器**)

  ```java

package org.apache.ibatis.binding;

import org.apache.ibatis.builder.annotation.MapperAnnotationBuilder;
import org.apache.ibatis.io.ResolverUtil;
import org.apache.ibatis.session.Configuration;
import org.apache.ibatis.session.SqlSession;

import java.util.Collection;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * @author Clinton Begin
 * @author Eduardo Macarron
 * @author Lasse Voss
 */
public class MapperRegistry {

  private final Configuration config;
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<Class<?>, MapperProxyFactory<?>>();

  public MapperRegistry(Configuration config) {
    this.config = config;
  }

  @SuppressWarnings("unchecked")
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }
  
  public <T> boolean hasMapper(Class<T> type) {
    return knownMappers.containsKey(type);
  }

  public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
      if (hasMapper(type)) {
        throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
      }
      boolean loadCompleted = false;
      try {
        knownMappers.put(type, new MapperProxyFactory<T>(type));
        // It's important that the type is added before the parser is run
        // otherwise the binding may automatically be attempted by the
        // mapper parser. If the type is already known, it won't try.
        MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
        parser.parse();
        loadCompleted = true;
      } finally {
        if (!loadCompleted) {
          knownMappers.remove(type);
        }
      }
    }
  }

  /**
   * @since 3.2.2
   */
  public Collection<Class<?>> getMappers() {
    return Collections.unmodifiableCollection(knownMappers.keySet());
  }

  /**
   * @since 3.2.2
   */
  public void addMappers(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
    for (Class<?> mapperClass : mapperSet) {
      addMapper(mapperClass);
    }
  }

  /**
   * @since 3.2.2
   */
  public void addMappers(String packageName) {
    addMappers(packageName, Object.class);
  }
  
}
  ```



  源码中有一段

  ```java
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<Class<?>, MapperProxyFactory<?>>();
  ```

  **knownMappers**中类型和动态工场一一对应，MapperProxyFactory是映射器代理工厂，通过这个工厂类可以获取到对应的映射器代理类MapperProxy，这里只需要保存一个映射器的代理工厂，根据工厂就可以获取到对应的映射器。

### 添加添加映射器有三个方法

**1.addMapper:添加**

```java
public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
      if (hasMapper(type)) {
        throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
      }
      boolean loadCompleted = false;
      try {
        //创建工厂
        knownMappers.put(type, new MapperProxyFactory<T>(type));
		//这里parser的解析
        //找到对用的xml后进行解析mapper节点里面的节点
        MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
        parser.parse();
        loadCompleted = true;
      } finally {
        if (!loadCompleted) {
          knownMappers.remove(type);
        }
      }
    }
  }
```



源码中会捕获从xml扫描过来的注册器，规定只有接口类型的class才会被添加，如果这个注册器重复了，会报错。

然后

```java
this.knownMappers.put(type, new MapperProxyFactory(type));
```

将会以type的类型创建一个动态代理工厂，与注册器一起放到 knownMappers中



**2.addMappers(String packageName, Class<?> superType) :查找包下所有是superType的类**

```java
  public void addMappers(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
    for (Class<?> mapperClass : mapperSet) {
      addMapper(mapperClass);
    }
  }
```



**3 .addMappers(String packageName):查找包下的所有类**

```java
  public void addMappers(String packageName) {
    addMappers(packageName, Object.class);
  }
```



可以看出3调用2，2调用1,所以方法1才是最核心的东西

然而方法3中带了一个参数object.class 调用2，得到的结果就是该包下的所有类都可以添加，因为所有的类都是object的子类，所以在方法2中针对每一个类继续调用方法1。确认是接口类并且没有重发之后就会添加到注册器中；一开始给定一个标志**loadCompleted**一开始没有加载完成所以默认为false，如果添加完成后就把它置为true；如果失败了，就会在finally里面从**knownMappers**中删除

### MapperAnnotationBuilder

我们可以看到每一次添加注册器的时候都会添加先new一个MapperAnnotationBuilder解析

```java
public void parse() {
  //先拿到接口的全路径
  String resource = type.toString();
  //判断一下是否已经被加载过了，因为每一次加载完成后都会添加到configure对象的set中
  if (!configuration.isResourceLoaded(resource)) {
    //重点是这里，这里开始装载接口对应的xml，方法在下面
    loadXmlResource();
    //解析完成后如果找得到namespace就是xml的namespace，如果找不到，就是null，然后把路径添加到set里面
    configuration.addLoadedResource(resource);
    //然后把把全接口名设置为namespace，
    assistant.setCurrentNamespace(type.getName());
    parseCache();
    parseCacheRef();
    Method[] methods = type.getMethods();
    for (Method method : methods) {
      try {
        // issue #237
        if (!method.isBridge()) {
          parseStatement(method);
        }
      } catch (IncompleteElementException e) {
        configuration.addIncompleteMethod(new MethodResolver(this, method));
      }
    }
  }
  parsePendingMethods();
}

  private void loadXmlResource() {
    // Spring may not know the real resource name so we check a flag
    // to prevent loading again a resource twice
    // this flag is set at XMLMapperBuilder#bindMapperForNamespace
    // 这里判断configure对象的set里面是否有namespace
    if (!configuration.isResourceLoaded("namespace:" + type.getName())) {
      //这里是重点看下面
      String xmlResource = type.getName().replace('.', '/') + ".xml";
      InputStream inputStream = null;
      try {
        inputStream = Resources.getResourceAsStream(type.getClassLoader(), xmlResource);
      } catch (IOException e) {
        // ignore, resource is not required
      }
      if (inputStream != null) {
        //这里开始对xml解析，
        XMLMapperBuilder xmlParser = new XMLMapperBuilder(inputStream, assistant.getConfiguration(), xmlResource, configuration.getSqlFragments(), type.getName());
        xmlParser.parse();
        //这个type.getName就是接口全名
      }
    }
  }


```

### 保存Mapper.xml信息的重要对象MapperBuilderAssistant

他是它里面设置命名空间的一个set方法

```java
 public void setCurrentNamespace(String currentNamespace) {
    if (currentNamespace == null) {
      throw new BuilderException("The mapper element requires a namespace attribute to be specified.");
    }

    if (this.currentNamespace != null && !this.currentNamespace.equals(currentNamespace)) {
      throw new BuilderException("Wrong namespace. Expected '"
          + this.currentNamespace + "' but found '" + currentNamespace + "'.");
    }

    this.currentNamespace = currentNamespace;
  }
```

XMLMapperBuilder类里面有个MapperBuilderAssistant对象，XMLMapperBuilder的构造方法里面会把MapperBuilderAssistant初始化，并且用接口的全名作为参数（currentNamespace）调用MapperBuilderAssistant的**setCurrentNamespace** 方法

```java
  public XMLMapperBuilder(InputStream inputStream, Configuration configuration, String resource, Map<String, XNode> sqlFragments, String namespace) {
    this(inputStream, configuration, resource, sqlFragments);
    this.builderAssistant.setCurrentNamespace(namespace);//设置namespace
  }
```



然后设置了currentNamespace；当XMLMapperBuilder开始解析xml文件的时候，会获取到namespace的值，然后继续调用setCurrentNamespace方法，这时候如果namespace的值不是接口的全名，然后就会抛异常

```java
private void configurationElement(XNode context) {
  try {
    String namespace = context.getStringAttribute("namespace");
    if (namespace == null || namespace.equals("")) {
      throw new BuilderException("Mapper's namespace cannot be empty");
    }
    builderAssistant.setCurrentNamespace(namespace);
    cacheRefElement(context.evalNode("cache-ref"));
    cacheElement(context.evalNode("cache"));
    parameterMapElement(context.evalNodes("/mapper/parameterMap"));
    resultMapElements(context.evalNodes("/mapper/resultMap"));
    sqlElement(context.evalNodes("/mapper/sql"));
    buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing Mapper XML. Cause: " + e, e);
  }
}
```





如果是以resource和url注册的话他是采先用**XMLMapperBuilder ** 解析而不是用 **MapperAnnotationBuilder ** ，并且构造XMLMapperBuilder 的时候并没有传类型进来，调用一下这个方法

```java
  public XMLMapperBuilder(InputStream inputStream, Configuration configuration, String resource, Map<String, XNode> sqlFragments) {
    this(new XPathParser(inputStream, true, configuration.getVariables(), new XMLMapperEntityResolver()),
        configuration, resource, sqlFragments);
  }
```

这个方法没有设置namespace，所以在后面解析xml的时候namespace是null

XMLMapperBuilder里面有一个**bindMapperForNamespace ** 方法，里面的顺序

```java
          configuration.addLoadedResource("namespace:" + namespace);
          configuration.addMapper(boundType);
```

先把namespace插入configure的set集合里面，然后再addMapper，虽然addMapper里面还重复使用`MapperAnnotationBuilder` 去解析，但是`loadXmlResource()` 会判断configure的set集合里面有没有namespace，由于之前已经add进去了，所以`loadXmlResource()` 方法就不会执行了，但是package和class是直接用`MapperAnnotationBuilder`所以没有用到 所以没有用到`bindMapperForNamespace` 方法，所以configure对象中的set的集合是没有namespace的，所以执行该方法，我们来看一下怎么去找到xml文件的，就是下面这一句

```java
 String xmlResource = type.getName().replace('.', '/') + ".xml";
```

### 由此可得，package和url是以 直接寻找当前目录下而且是名字一样的xml

### 所以xml和接口的路径和名字都不一样的话，就没办法找到xml，值得注意的是XMLMapperBuilder解析的时候会调用bindMapperForNamespace方法。但是由于接口已经注册所以不会再执行一次绑定操作



SqlSessionFactoryBean实际上对应的是SqlSessionFactory类，它会扫描sql xml文件，并对接口创建动态代理，将接口类的Class和动态代理关系保存在SqlSessionFactory中，这仅仅是完成了动态代理的生成，而动态代理在哪里被使用到，怎么使用，这些都是由MapperScannerConfigurer完成，接下来看看MapperScannerConfigurer都做了些什么？

### 在MapperRegistry映射器注册器中有一个getMapper()方法

![6](image/6.png)

从集合中获取指定接口类型的映射器代理工厂，然后使用这个代理工厂创建映射器代理实例并返回，那么我们就可以获取到映射器的代理实例



###   MapperProxyFactory 映射器代理工厂

![3](image/3.png)

//用JDK自带的动态代理生成映射器,用类加载器去new一个实体类

```java
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
```

在这个代理工厂中定义了一个缓存集合，其实为了调用MapperProxy的构造器而设，这个缓存集合用于保存当前映射器中的映射方法的。

### MapperProxy 映射器代理

```java
package org.apache.ibatis.binding;

import java.io.Serializable;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.util.Map;

import org.apache.ibatis.reflection.ExceptionUtil;
import org.apache.ibatis.session.SqlSession;

/**
 * @author Clinton Begin
 * @author Eduardo Macarron
 */
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private static final long serialVersionUID = -6424540398559729838L;
  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache;

  public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
    this.sqlSession = sqlSession;
    this.mapperInterface = mapperInterface;
    this.methodCache = methodCache;
  }

  //代理以后实体类调用什么方法是都是执行invoke方法
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (Object.class.equals(method.getDeclaringClass())) {
      try {
        return method.invoke(this, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
  }

  private MapperMethod cachedMapperMethod(Method method) {
    //这是是从methodCache(一个Map，用来做缓存区)找这个方法。如果没有才new出来，提高了效率
    MapperMethod mapperMethod = methodCache.get(method);
    if (mapperMethod == null) {
      mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
      methodCache.put(method, mapperMethod);
    }
    return mapperMethod;
  }

}

```

### MapperMethod  方法映射器

（待定）







### typeAliases标签--->TypeAliasRegistry（注册器）

```java
private final Map<String, Class<?>> TYPE_ALIASES = new HashMap<String, Class<?>>();
```

也是通过注册器来实现，有5个方法，前3个方法方法和MapperRegistry大同小异，

有两个方法进行注册

**package：1-->2-->3-->4**

**alis:5-->4**



```java
//1
  public void registerAliases(String packageName){
    registerAliases(packageName, Object.class);
  }
//2
  public void registerAliases(String packageName, Class<?> superType){
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
    for(Class<?> type : typeSet){
      // Ignore inner classes and interfaces (including package-info.java)
      // Skip also inner classes. See issue #6
      if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) {
        registerAlias(type);
      }
    }
  }
//3
  public void registerAlias(Class<?> type) {
    String alias = type.getSimpleName();
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    if (aliasAnnotation != null) {
      alias = aliasAnnotation.value();
    } 
    registerAlias(alias, type);
  }
//4最核心的方法
  public void registerAlias(String alias, Class<?> value) {
    if (alias == null) {
      throw new TypeException("The parameter alias cannot be null");
    }
    // 这里看到，你在xml所写的别名都会转为小写
    String key = alias.toLowerCase(Locale.ENGLISH);
    if (TYPE_ALIASES.containsKey(key) && TYPE_ALIASES.get(key) != null && !TYPE_ALIASES.get(key).equals(value)) {
      throw new TypeException("The alias '" + alias + "' is already mapped to the value '" + TYPE_ALIASES.get(key).getName() + "'.");
    }
    TYPE_ALIASES.put(key, value);  //此处进行类型注册
  }
//5
  public void registerAlias(String alias, String value) {
    try {
      registerAlias(alias, Resources.classForName(value));
    } catch (ClassNotFoundException e) {
      throw new TypeException("Error registering type alias "+alias+" for "+value+". Cause: " + e, e);
    }
  }
```

这类的构造方法里面调用方法1给我们注册了许多默认的类型，例如int string long。

---



里面还有一个找到class的方法

```java
  public <T> Class<T> resolveAlias(String string) {
    try {
      if (string == null) {
        return null;
      }
      // 全小写
      String key = string.toLowerCase(Locale.ENGLISH);
      Class<T> value;
      //如果注册器（map）中有这个类
      if (TYPE_ALIASES.containsKey(key)) {
        //从注册器中找
        value = (Class<T>) TYPE_ALIASES.get(key);
      } else {
        //如果没有就从 org.apache.ibatis.io.Resources这个类中建造一个
        value = (Class<T>) Resources.classForName(string);
      }
      return value;
    } catch (ClassNotFoundException e) {
      throw new TypeException("Could not resolve type alias '" + string + "'.  Cause: " + e, e);
    }
  }
```

**org.apache.ibatis.io.Resources;**里面的classForName方法

```java
  public static Class<?> classForName(String className) throws ClassNotFoundException {
    //用jdk的加载器去加载这个类
    return classLoaderWrapper.classForName(className);
  }
```



### TypeHandlerReister(类型注册器)

里面有三个成员变量

```java
private final Map<JdbcType, TypeHandler<?>> JDBC_TYPE_HANDLER_MAP = new EnumMap<JdbcType, TypeHandler<?>>(JdbcType.class);

private final Map<Type, Map<JdbcType, TypeHandler<?>>> TYPE_HANDLER_MAP = new HashMap<Type, Map<JdbcType, TypeHandler<?>>>();

private final Map<Class<?>, TypeHandler<?>> ALL_TYPE_HANDLERS_MAP = new HashMap<Class<?>, TypeHandler<?>>();
```

- 第一种**JDBC_TYPE_HANDLER_MAP**：是一种枚举Map，这种Map先天存在key，而key就是数据库类型，用来写入数据库
- 第二种**TYPE_HANDLER_MAP**，是一个嵌套Map，里面的Ma就是第一种，外面的Map的key就是java类型，这就相当于javaType-jdbcType一一对应。
- 第三种**ALL_TYPE_HANDLERS_MAP**，虽然我不是很明白什么意思，不过看名字应该是保存所有的注册器

### TypeReference（用于获取原生类）

```java

package org.apache.ibatis.type;

import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

/**
 * References a generic type.
 *
 * @param <T> the referenced type
 * @since 3.1.0
 * @author Simone Tripodi
 */
public abstract class TypeReference<T> {

  private final Type rawType;

  protected TypeReference() {
    //构造方法中调用，自身的方法,把自己的class传进去
    rawType = getSuperclassTypeParameter(getClass());
  }

  
 //核心方法
  Type getSuperclassTypeParameter(Class<?> clazz) {
    //用自身的class调用getGenericSuperclass()来获取直接带参数的类，即泛型类T
    Type genericSuperclass = clazz.getGenericSuperclass();
    
    /*由于每一个类都是Class类的子类，而Class本身也是一个类，但是它又是所有类的父类,查阅资料的到Class的实例表示的是在一个运行的应用中的所有类和接口，所以Class类的实类就是借口与类。jdk里面规定了，只有泛型不属于Class类（具体我也不清楚）*/
 =========================   
    if (genericSuperclass instanceof Class) {
      
      //如果父类又不是TypeReference就向下递归找到参数类
      if (TypeReference.class != genericSuperclass) {
        return getSuperclassTypeParameter(clazz.getSuperclass());
      }

      //如果是TypeReference，这里TypeException有一个内部的构造方法，把TypeReference类传进去，在里面如果不是TypeReference类型，就没事，继续递归，如果不是就抛异常，说明程序出错了
      throw new TypeException("'" + getClass() + "' extends TypeReference but misses the type parameter. "
        + "Remove the extension or add a type parameter to it.");
    }
===========================
    
    //如果找到泛型类了。就钱转它的子类ParameterizedType，既参数化，然后获取实际的参数，获取泛型的参数类型，可能参数不止一个，所以获取第一个
    
    Type rawType = ((ParameterizedType) genericSuperclass).getActualTypeArguments()[0];
    // TODO remove this when Reflector is fixed to return Types
    if (rawType instanceof ParameterizedType) {
      rawType = ((ParameterizedType) rawType).getRawType();
    }

    return rawType;
  }

  public final Type getRawType() {
    return rawType;
  }

  @Override
  public String toString() {
    return rawType.toString();
  }

}

```

在type下面还有很多类型处理器，不想看了



## 小问题讨论与发现：

在不用到动态代理的时候：namespace可以随便定义。应为直接在sqlSession上使用Statement，没有涉及到代理。向下直接调用excute的方法与数据库交互。只要把module注册了就可以。

```java
sqlSession.selectOne(statement,parameter);
```

注册方式有两种。

- ```java
      <typeAliases>
          <package name="${moduleLocation}"/>
      </typeAliases>
  ```

  在typeAliases注册

- 当你使用接interface的时候可以通过namespace去注册



junit的实现原理

# 四  CURD

### 单表查询

 当接口返回类型是一个object的时候，mybatis是在查询的时候如果返回多个数据，会抛异常，因为mybatis是默认使用selectOne()的方法

```xml
    <select id="selectAll" resultType="User">
      SELECT * FROM user
    </select>
```

如果要查询多个同一类的object的话，就必须在**接口类** 声明返回值是list



当bean和数据库别名不一样的时候用定义 < resultMap>标签来自定义返回结果集，当别名有冲突的时候

```xml
    <resultMap id="selectMap" type="Classes">
        <id property="cId" column="c_id"/>
        <result property="cName" column="c_name"/>
        <result property="teacherId" column="teacher_id"/>
    </resultMap>

    <select id="select"  resultMap="selectMap">
        SELECT * FROM  classes
    </select>
```

标签resultMap的id一定要跟下面select标签的 resultMap的value相同

里面的标签id 对应着主键，property对应po类的字段 column对应数据库字段



### 一对一查询（班级——老师）

##### 连表查询

```xml
    <resultMap id="selectMap" type="Classes">
        <id property="cId" column="c_id"/>
        <result property="cName" column="c_name"/>
        <association property="teacher" javaType="Teacher">
            <id property="tId" column="t_id"></id>
            <result property="tName" column="t_name"></result>
        </association>
    </resultMap>

    <select id="select"  parameterType="int" resultMap="selectMap">
        SELECT * FROM  classes c,teacher t WHERE c.teacher_id=t.t_id AND c.c_id=#{id}
    </select>
```

Class对象成员标量的定义

```java
    private Integer cId;
    private String cName;
    private Teacher teacher;
```

由于我们里面有一个对象所以我们要在resultMap里用association来定义Teacher对象的打包方式，property依然是 成员变量，JavaType的指定打包类型

##### 执行两次Sql

```xml
    <resultMap id="select2Map" type="Classes">
        <id property="cId" column="c_id"/>
        <result property="cName" column="c_name"/>
        <association property="teacher"  column="teacher_id" select="selectTeacher">
        </association>
    </resultMap>

    <select id="selectClasses" parameterType="int" resultMap="select2Map">
        SELECT * FROM classes WHERE c_id = #{c_id}
    </select>

    <select id="selectTeacher" parameterType="int" resultType="com.pramy.module.Teacher">
        SELECT t_id tId,t_name tName FROM teacher WHERE t_id=#{id}
    </select>
```

分析：在执行selectClasses的过程中会执行select2Map,在select2Map中指定成员变量teacher 的类型 和column() ，然后根据查询的到的sql字段中的teacher_id，把它作为参数再执行selectTeacher（注意名称），然后返回一个Teacher对象储存在Classes对象中

### 一对多查询(班级——老师——学生)

```xml
    <select id="selectClasses3" parameterType="int" resultMap="selectClasses3">
        SELECT * FROM classes c,teacher t,student s WHERE t.t_id=c.teacher_id AND c.c_id=s.class_id AND c.c_id=#{id}
    </select>

    <resultMap id="selectClasses3" type="com.pramy.module.Classes">
        <id property="cId" column="c_id"/>
        <result property="cName" column="c_name"/>
        <association property="teacher" javaType="com.pramy.module.Teacher" >
            <id property="tId" column="t_id"/>
            <result property="tName" column="t_name"/>
        </association>
        <collection property="list"  ofType="com.pramy.module.Student">
            <id property="id" column="s_id"/>
            <result property="name" column="s_name"/>
        </collection>
    </resultMap>
```

就是多了一个collection来封装集合，ofType：指定容器装的类型



## 动态Sql查询

### 1.if标签

```xml
<if test=""> </if>    条件判断
```

在做模糊查询的时候可以写成这样

```xml
        <if test='cName!="null"' >
            WHERE c_name LIKE concat(concat('%',#{cName}),'%') //sql本身的字符串拼接
        </if>
```

或者这样

```xml
        <if test='cName!="null"' >
            WHERE c_name LIKE '%${cName}%'
        </if>
```



#### 2.where,trim,set标签

```xml
           select * from classes 
           <where>
               <if test="cId != null">
                   c_id=#{cId}
               </if>
               and cName='b';
           </where>
```

- where标签可以处理前缀的AND 和 OR 
- set可以处理后缀的逗号
- trim自定义标签





### foreach标签

```xml
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
```

collection：如果传过来的是list就写list 是array就写array，是map就写map



### choose 标签

```xml
<choose>
	<when>
 	······	 
  	</when>
  	<when>
  	·······
  	</when>
  	<otherwise>
	·····
     </otherwise>
</choose>
```



**如果有一个成立，则choose结束。**当choose中所有when的条件都不满则时，则执行otherwise中的sql。

### 转义字符

将语句中的位运算（与）”&“符使用“&amp;”替换

mybatis配置文件写SQL语句的某些字符需要转义：

```xml
 　 &lt;       < 
    &gt;       >  
    &lt;&gt;   <>
    &amp;      & 
    &apos;      '
    &quot;      "
```



## 缓存机制

默认开启一级缓存，生命周期是一个session，每一次查询到的内容会加入到缓存中，如果下次还有同样的操作就从缓存中取出，为了提高性能和节省资源

有3个方法清空缓存

> 1.如果执行了session.clearCache()的方法
>
> 2.执行了update，delete，insert
>
> 3.session.close()

# 实践问题：

- 用接口的时候用package的时候要统一路径

- sqlSession用完记得关闭

- 传两个参数的办法

  ```java

  List<Section> selectList(@Param("section") Section section, @Param("page")PageUtil page);
  ```

  ```xml
  //对应的xml配置
  <select id="selectList"  resultMap="BaseResultMap">
      SELECT
      <include refid="Base_Column_List"/>
      FROM section
      <where>
        <if test="section.sectionName!=null">
          <trim prefix="(" suffix=")" prefixOverrides="or">
            or section_name like concat(concat('%',#{section.sectionName}),'%')
            or creat_time like concat(concat('%',#{section.sectionName}),'%')
            or creater_name like concat(concat('%',#{section.sectionName}),'%')
          </trim>
        </if>
      </where>
      ORDER BY creater_name desc
      <if test="page.pageSize!=0">
        limit #{page.index} , #{page.pageSize}
      </if>
    </select>
    <resultMap id="BaseResultMap" type="com.pramy.module.Section">
      <constructor>
        <idArg column="id" javaType="java.lang.Integer" jdbcType="INTEGER" />
        <arg column="section_name" javaType="java.lang.String" jdbcType="VARCHAR" />
        <arg column="creat_time" javaType="java.util.Date" jdbcType="TIMESTAMP" />
        <arg column="level" javaType="java.lang.Integer" jdbcType="INTEGER" />
        <arg column="creater_name" javaType="java.lang.String" jdbcType="VARCHAR" />
      </constructor>
    </resultMap>
  ```

  千万千万要注意sql，写错了会很麻烦

  当错误提示显示**not getter for **

  ### 这里千万要注意

  - 如果是一个参数对象，绝对不能用**参数名+成员变量 **来取值
  - 如果是两个参数最好的办法就是家注解，但是要获取某一个参数对象的成员变量的值，一定要要用参数名（注解写的名字）+成员变量

  ```java
  public static Date getDateBefore(Date d, int day) {  
          Calendar now = Calendar.getInstance();  
          now.setTime(d);  
          now.set(Calendar.DATE, now.get(Calendar.DATE) - day);  //获得前几天的日期
          return now.getTime();  
      } 
  ```

  ​