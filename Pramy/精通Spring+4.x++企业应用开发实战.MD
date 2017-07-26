# 精通Spring+4.x++企业应用开发实战

## 第四章

4.2.2 类加载器ClassLoader

- 类装载机制
  - 装载，连接（校验，给静态变量分配空间，解析），初始化（对类的静态变量，静态代码块初始化）
  - ClassLoader：根加载器
  - ExtClassLoader：扩展类装载器
  - AppClassLoader：应用类装载器

其中跟装载器不是ClassLoader的子类，底层实现是c++，在运行时负责加载jre核心库（rt.jar,charsets.jar）

ExtClassLoader和AppClassLoader都是ClassLoader的子类，AppClassLoader负责加载ClassPath路径下的jar包，而ExtClassLoader负责加载jre目录下的ext中jar包，根加载器在jvm中属于底层，在程序中不可见，但是三个加载器之间有着明显的父子层级。

ExtClassLoader是AppClassLoader的父装载器，一般默认用AppClassLoader加载类，jvm加载使用“全盘责任委托机制”：先委托父加载器去寻找目标类，在找不到的时候自己才会从自己的方式中加载

## 第五章

### 5.3.3工厂注入

```java
public class CarFactory {
   public Car createHongQiCar(){
	   Car car = new Car();
	   car.setBrand("红旗CA72");
	   return car;
   }
   
   public static Car createCar(){
	   Car car = new Car();
	   return car;
   }
}

```

- 通过工厂类注入

  ```xml
  	<!--首先定义工厂类-->
  	<bean id="carFactory" class="com.smart.ditype.CarFactory" />
  	<!--然后配置Bean，说明使用工厂carFactory的createHongQiCar来创建-->
  	<bean id="car5" factory-bean="carFactory" factory-method="createHongQiCar"/>
  ```

- 通过静态工厂注入

  ```xml
  <!--指定工厂类，然后调用它的静态构造方法-->
  <bean id="car6" class="com.smart.ditype.CarFactory" factory-method="createCar"/>
  ```

### 5.4.1字面值

XML特殊标签<![CDATA[ ]]>，这标签的作用就是让XML解析器将标签中的字符串当做普通的文本对待，防止对特殊字符对XML解析造成影响

一般XML会忽略元素标签内部字符串的前后空格，但是spring不会

### 5.4.2引用其他Bean

```xml
<ref bean=""/>或者<ref local=""/>或者<ref praent=""/>
```

bean：可以用引用同一容器或者父类容器的bean

local：引用本地

praent：引用父类的

### 5.4.4null

如果用`<value></value>`的方式给bean赋值，只会得到空字符串，不会的到null，必须用null元素标签`<value><null/></value>`

### 5.4.5级联属性

```xml
<bean id="boss" class="com.smart.attr.Boss">
	<property name="car.brand" value="111"/>
</bean>
```

我们为boss类里面的car对象的brand元素赋值，但是前提是boss对象里面存在car实例

### 5.6.1继承

```xml
    <!-- 父子<bean> -->
	<bean id="abstractCar" class="com.smart.tagdepend.Car"
	      p:brand="红旗CA72" p:price="2000.00" p:color="黑色"
	      abstract="true"/>
	<bean id="car3" parent="abstractCar">
		<property name="color" value="红色"/>
	</bean>
	<bean id="car4" parent="abstractCar" >
		<property name="color" value="白色"/>
	</bean>	
```

子类继承父类的信息，并且重新给属性color赋值，父类的主要功能是简化子类的配置，所以声明**abstract=true **，这表示spring Ioc容器不会实例化这个bean

### 5.6.2依赖

一般情况下我们用ref来表示对其他bean的依赖，但是这bean是死的，所有属性都定死。

这时候就可以用依赖关系来实现，用**depends-on ** 显示来指定依赖关系

```xml
<!-- <bean>的信赖 -->
	<bean id="cacheManager" class="com.smart.tagdepend.CacheManager" depends-on="sysInit" />
	<bean id="sysInit" class="com.smart.tagdepend.SysInit" />
```

在实例化cacheManager之前会先实例化sysinit，你就可以在sysinit里写各种各样的操作

比如A-->B  但是可以显式 指定A ：depends-on C     ，C可以在对B的属性的修改，达到A动态变化的效果

### 5.6.3引用

`ref`和`idref` 的区别

ref是引用bean实例，idref是引用该bean的id 或者是name，相当于value元素，当时value是没有具备检验功能的，但是idref是有的，会检查该容器中是否有这个bean存在，如果没有回抛出**BeanDefinitionStoreException **

### 5.7 整合多个配置文件

```xml
<import resource="classpath:com/samrt/impt/beans1.xml"/>
<!--可以通过引入另外一个配置文件，达到拥有完全的属性-->
```

a.xml引入b.xml，spring在启动的时候就会只加载a.xml就可以了，不引入的情况就是会同时加载a.xml和b.xml然后再内容中合并

### 5.8单例模式singleton

在内存中只有一个实例，在spring默认的方式就是单例模式，但是这有一个点，在spring启动的时候会去实例化一个bean但是如果这个bean被实例化了但是很久都没有用到，或者是用户的业务逻辑没有使用到这个bean，就造成了浪费，所以采用懒加载的模式，在bean后面加上一个`lazy-init=true` 的元素,就是在用户使用的时候才会被实例化

### 5.8.4作用域依赖问题

```xml
<bean name="car" class="com.smart.scope.Car" scope="request">
	<!--需要用到aop命名空间，创建代理-->	
	<aop :scoped-proxy/>	
</bean>
<bean id="boss" class="com.smart.scope.Boss">
  	<!--引用相关作用域的car Bean-->
	<propertry name="car" ref="car"/>  
</bean>
```

![1](image/1.png)

由于boss是单实例的每一次request都应当实例相对作用域的car，所以引入aop代理的概念，代理给boss和car做了桥接，当boss Bean在Web环境下调用car Bean时，Spring AOP将启动动态代理只能判断boss位于哪一个HTTP请求中，并且让boos引用到相应的HTTP请求的car Bean中

但是启动代理之后直接被boss引用的不是car Bean 而是AOP代理

### 5.10.2扫描注解定义的bean

 @Value("#{beanName.propName}")

# 第七章 AOP

### 7.12AOP术语：

![2](image/2.png)

![3](image/3.png)

![4](image/4.png)

![5](image/5.png)

![6](image/6.png)

![7](image/7.png)

![8](image/8.png)

![9](image/9.png)

### 7.2.1

JDK的动态代理，CGLib的动态代理

区别在于JDK只支持接口代理，不支持类的代理

**JDK代理 **

```java
public class PerformaceHandler implements InvocationHandler {
    private Object target;
	public PerformaceHandler(Object target){
		this.target = target;
	}
	public Object invoke(Object proxy, Method method, Object[] args)
			throws Throwable {
		PerformanceMonitor.begin(target.getClass().getName()+"."+ method.getName());
		Object obj = method.invoke(target, args);
		PerformanceMonitor.end();
		return obj;
	}
}
```

使用：

```java
		ForumService target = new ForumServiceImpl();
		PerformaceHandler handler = new PerformaceHandler(target);
		ForumService proxy = (ForumService) Proxy.newProxyInstance(target
				.getClass().getClassLoader(),
				target.getClass().getInterfaces(), handler);
```

CglibProxy代理

```java
public class CglibProxy implements MethodInterceptor {
	private Enhancer enhancer = new Enhancer();

	public Object getProxy(Class clazz) {
		enhancer.setSuperclass(clazz);
		enhancer.setCallback(this);
		return enhancer.create();
	}

	public Object intercept(Object obj, Method method, Object[] args,
			MethodProxy proxy) throws Throwable {
		PerformanceMonitor.begin(obj.getClass().getName()+"."+method.getName());
		Object result=proxy.invokeSuper(obj, args);
		PerformanceMonitor.end();
		return result;
	}
}
```

使用

```java
CglibProxy cglibProxy = new CglibProxy();
		ForumService forumService = (ForumService)cglibProxy.getProxy(ForumServiceImpl.class);
```

CGLib是采用拦截父类方法，通过创建子类的方式生成代理对象，所以不能对目标类中final或者private方法进行代理

### 7.3.2前置增强

[**MethodBeforeAdvice**]()

ProxyFactory：里面承载了GCLib的代理方法和JDK动态代理的方法

```java
        Waiter target = new NaiveWaiter();
        BeforeAdvice  advice = new GreetingBeforeAdvice();
        ProxyFactory pf = new ProxyFactory();
//      pf.setInterfaces(target.getClass().getInterfaces());
        pf.setTargetClass(target.getClass());
        pf.setOptimize(true);
        pf.setTarget(target);
        pf.addAdvice(advice);

        Waiter proxy = (Waiter)pf.getProxy(); 
```

如果使用setInterfaces()的方法就使用JDK的动态代理

如果使用的setTargetClass()的方法就是使用GCLib动态代理

GCLib代理xml配置

```xml
<bean id="waiter" class="org.springframework.aop.framework.ProxyFactoryBean"
         p:target-ref="target" 					//代理目标
         p:interceptorNames="greetingAround"	//这里配置增强器，多个用逗号隔开
         p:proxyTargetClass="true" 				//这里实现Class代理，使用CGLib代理
         p:optimize="true"						//优化
   />
```

JDK代理的配置

```xml
<bean id="waiter" class="org.springframework.aop.framework.ProxyFactoryBean"
         p:target-ref="target"
         p:interceptorNames="greetingAround"
         p:interfaces="com.smart.advice.Waiter"
   />		
```

optimize：当设置为true是，强制使用CGlib代理，对于其他作用域的代理最好用JDK代理，因为是GCLib创建时速度慢，但是创建出来的代理对象运行效率高，因为之创建子类对象，调用子类方法，而Jdk每一次都需要调用反射机制

# 第十一章

## 11.1.1 数据库事务

ACID

原子性（Atomic）：表示组成一个事物的多个数据库操作是一个不可分割的原子单元，只有所有的操作执行成功，整个事务才提交。事务中的任何一个数据库操作失败已经执行的任何操作都必须撤销，让数据库返回到初始状态

一致性（Consistency）：事务操作成功后，数据库所处的状态和他的业务规则是一致的，即数据不会被破坏

隔离性（Isolation）：在并发数据库操作时，不同的事物拥有各自的数据空间，他们的操作不会对对方产生干扰。

持久性（Durability）：一旦事务提交成功后，事务中所有的数据操作都必须被持久化到数据库中

- 脏读（dirty read）:由于多线程操作对于同一目标进行操作，事务提交有出入

  ![11](image/11.png)

- 不可重复读（unrepeatable read）

- 幻想读（phantom read）

  ![12](image/12.png)

- ​

## 11.2.1  ThreadLocal

**ThreadLoacal** 它是一个保存线程本地化对象的容器。当运行于多线程环境的谋个对象使用ThreadLocal维护变量是，ThreadLocal为每一个使用该变量的线程分配一个独立的变量副本，每一个线程都可以独立得该店自己的副本，而不会影响其他线程对副本。因为每一个线程创建的时候自己都会有一个副本。而线程和线程之间的副本都是相对其他线程的副本独立存在的。

**InheritableThreadLocal** 继承于ThreadLocal，他自动为子线程赋值一份从父线程那里继承而来的本定变量：在创建子线程时，子线程会接收所有可继承的线程本定变量的初始值

get，set ，remove，initialValue

initialValue：返回该线程局部变量的初始值，可以重写，在线程第一次使用get或者set时才执行，只执行一次

## 11.3.2 Spring的事物管理器实现类

![10](image/10.png)

## 11.5事务管理的配置

XML配置：

```xml
<!-- 事务配置 start -->
<!--声明事务管理器-->
<bean id="transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource">
    </property>
</bean>
<!--声明自己的事务-->
<tx:advice id="txadvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="save*" propagation="REQUIRED"/>
        <tx:method name="delete*" propagation="REQUIRED"/>
        <tx:method name="update*" propagation="REQUIRED"/>
        <tx:method name="transfer*" propagation="REQUIRED"/>
        <tx:method name="insert*" propagation="REQUIRED"/>
        <tx:method name="change*" propagation="REQUIRED"/>
        <tx:method name="do*" propagation="REQUIRED"/>
        <tx:method name="*" propagation="REQUIRED" read-only="true"/>
    </tx:attributes>
</tx:advice>
<aop:config>
    <aop:pointcut
            expression="execution(* cn.kiiwii.framework.spring.TestSpring.springwithmybatis.service.impl.*.*(..))"
            id="txpointcut"/>
    <aop:advisor advice-ref="txadvice" pointcut-ref="txpointcut"/>
</aop:config>

<!-- 事务配置 end -->
```

![13](image/13.png)