

# Notes on Spring

标签（空格分隔）： 框架

---
##第一章 认识 Spring
一.简介
Spring 作者：Rod Johnson；
官方网站：http://spring.io/
最新开发包及文档下载地址：http://repo.springsource.org/libs-release-local/org/springframework/spring/
核心思想：IOC 控制反转；AOP 面向切面；
百度百科


二.Spring--HelloWorld

beans.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="helloWorld" class="com.spring.po.HelloWorld"></bean>
</beans>
```
HelloWorld.java
```java
package com.spring.po;

public class HelloWorld {
public void sayHello(){
	System.out.println("hello");
}
}
```

测试：
```java
package com.spring.test;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.spring.po.HelloWorld;

public class Test {
	public static void main(String[] args) {
		ApplicationContext ac= new ClassPathXmlApplicationContext("beans.xml");
		HelloWorld helloWorld=(HelloWorld) ac.getBean("helloWorld"); //获取Spring代理的HelloWorld对象
		helloWorld.sayHello();
	}	
}
```

##第二章Spring 之IOC 详解

### 第一节：spring ioc 简介

假如现在有一项测试任务需要人做

```java
package com.spring.po;

public class JavaWork {
public void doTest(){
	ZhangSan zs=new ZhangSan(); //一开始指定张三做
	zs.test();
}
}
```

``` java
package com.spring.po;

public class ZhangSan {
public void test(){
	System.out.println("张三做测试");
}
}
```

主体代码：

```java
package com.spring.test;

import com.spring.po.JavaWork;

	public class Test1 {
	public static void main(String[] args) {
		JavaWork jw=new JavaWork();
		jw.doTest();
	}
	}
```

但这样的话张三与测试这一工作的耦合性太强了

改进：

```java
package com.spring.test;;

public interface Tester {

	public void test();
}
```

```java
package com.spring.po;

public class ZhangSan implements Tester{
public void test(){
	System.out.println("张三做测试");
}
}
```

```java
package com.spring.po;

public class Lisi implements Tester {
	public void test(){
		System.out.println("李四做测试");
	}
}
```

```java
package com.spring.po;

public class JavaWork {
private Tester tester;

public Tester getTester() {
	return tester;
}

public void setTester(Tester tester) {
	this.tester = tester;
} 
	
public void doTest(){
	tester.test();
}
}
```

主体类

```java
package com.spring.test;

import com.spring.po.JavaWork;
import com.spring.po.ZhangSan;

	public class Test1 {
	public static void main(String[] args) {
		JavaWork jw=new JavaWork();
		jw.setTester(new ZhangSan());
		jw.doTest();
	}
	}
```



用Spring方法改进

在bean.xml中设置tester为zhangsan，配置文件会调用JavaWorK中的setTester()

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="helloWorld" class="com.spring.po.HelloWorld"></bean>
	
	<bean id="zhangsan" class="com.spring.po.ZhangSan"> </bean>
	
	<bean id="javaWork" class="com.spring.po.JavaWork">
	<property name="tester" ref="zhangsan"></property>
	</bean>
</beans>
```

```java
package com.spring.test;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.spring.po.JavaWork;

public class Test2 {
	public static void main(String[] args) {
		ApplicationContext ac= new ClassPathXmlApplicationContext("beans.xml");
		JavaWork iavaWork=(JavaWork) ac.getBean("javaWork"); 
		iavaWork.doTest();
		((AbstractApplicationContext) ac).close();
	}	
}
```

```java
package com.spring.po;

public class JavaWork {
private Tester tester;


public void setTester(Tester tester) {
	this.tester = tester;
} 
	
public void doTest(){
	tester.test();
}
}

```





1.耦合性：

耦合性(Coupling)，也叫耦合度，是对模块间关联程度的度量。耦合的强弱取决于模块间接口的复杂性、调用模块的方式以及通过界面传送数据的多少。模块间的耦合度是指模块之间的依赖关系，包括控制关系、调用关系、数据传递关系。模块间联系越多，其耦合性越强，同时表明其独立性越差。软件设计中通常用耦合度和内聚度作为衡量模块独立程度的标准。划分模块的一个准则就是*`高内聚低耦合`*。

形象的说，就是要将代码写的和电脑一样，主类就是电脑的主机箱，当程序需要实现什么功能的时候只需要加其他的类引入接口，就像电脑上的usb接口。



2. Spring 能有效地组织J2EE应用各层的对象。不管**是**控制层的Action对象，还是业务层的Service对象，还是持久层的DAO对象，都可在Spring的 管理下有机地协调、运行。Spring将各层的对象以松耦合的方式组织在一起，Action对象无须关心Service对象的具体实现，Service对 象无须关心持久层对象的具体实现，各层对象的调用完全面向接口。当系统需要重构时，代码的改写量将大大减少。

上面所说的一切都得宜于Spring的核心机制，依赖注入**。**依赖注入让bean与bean之间以配置文件组织在一起，而不是以硬编码的方式耦合在一起。

依赖注入(Dependency Injection)和控制反转(Inversion of Control)是同一个概念。具体含义是:当某个角色(可能是一个Java实例，调用者)需要另一个角色(另一个Java实例，被调用者)的协助时，在 传统的程序设计过程中，通常由调用者来创建被调用者的实例。但在Spring里，创建被调用者的工作不再由调用者来完成，因此称为控制反转，创建被调用者 实例的工作通常由Spring容器来完成，然后注入调用者，因此也称为依赖注入。

第二节：spring ioc 实例讲解

第三节：装配一个bean

```java
package com.spring.po;

public class People {
private int id;
private String name;
private String password;
public People() {
	super();
}
public People(int id,String name, String password) {
	super();
	this.id=id;
	this.name = name;
	this.password = password;
}
public int getId() {
	return id;
}
public void setId(int id) {
	this.id = id;
}
public String getName() {
	return name;
}
public void setName(String name) {
	this.name = name;
}
public String getPassword() {
	return password;
}
public void setPassword(String password) {
	this.password = password;
}
@Override
public String toString() {
	return "People [id=" + id + ", name=" + name + ", password=" + password + "]";
}
}

```



第四节：依赖注入
1，属性注入；

2，构造函数注入；(通过类型；通过索引；联合使用)

```java
package com.spring.test;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.spring.po.People;


public class Test {
	public static void main(String[] args) {
		ApplicationContext ac= new ClassPathXmlApplicationContext("beans.xml");
		People people=(People) ac.getBean("people");
		System.out.println(people);
		//属性注入
		People people1=(People) ac.getBean("people1");
		System.out.println(people1);
		//构造方法-类型注入
		People people2=(People) ac.getBean("people2");
		System.out.println(people2);
		//构造方法-索引注入
		People people3=(People) ac.getBean("people3");
		System.out.println(people3);
		
		//构造方法-类型和索引混合注入
		People people4=(People) ac.getBean("people4");
		System.out.println(people4);
	}	
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="people" class="com.spring.po.People"></bean>
	
	<bean id="people1" class="com.spring.po.People">
	<property name="id" value="1"> </property>
	<property name="name" value="张三"> </property>
	<property name="password"  value="|123"> </property>
	</bean>
	
	<bean id="people2" class="com.spring.po.People">
	<constructor-arg type="int" value="2"></constructor-arg>
	<constructor-arg type="String" value="李四"></constructor-arg>
	<constructor-arg type="String" value="444"></constructor-arg>
	</bean>
	
	<bean id="people3" class="com.spring.po.People">
	<constructor-arg index="0" value="3"></constructor-arg>
	<constructor-arg index="1" value="王五"></constructor-arg>
	<constructor-arg index="2" value="55"></constructor-arg>
	</bean>


<bean id="people4" class="com.spring.po.People">
	<constructor-arg index="0" type="int"  value="6"></constructor-arg>
	<constructor-arg index="1" type="String"  value="刘六"></constructor-arg>
	<constructor-arg index="2" type="String" value="666"></constructor-arg>
	</bean>

</beans>
```

测试结果

```java
People [id=0, name=null, password=null]
People [id=1, name=张三, password=|123]
People [id=2, name=李四, password=444]
People [id=3, name=王五, password=55]
People [id=6, name=刘六, password=666]
```



3，工厂方法注入；(非静态工厂，静态工厂)

（1）新建一个造人非静态工厂（害怕）

```java
package com.spring.factory;

import com.spring.po.People;

public class PeopleFactory {
public People createPeople(){
	People p=new People(7,"朱七","77");	
	return p;
}
}
```

bean.xml

```xml
<bean id="peopleFactory" class="com.spring.factory.PeopleFactory">
	</bean>
	<bean id="people5" factory-bean="peopleFactory" factory-method="createPeople"></bean>	
```

测试

```java
//工厂方法
People people5=(People) ac.getBean("people5");
System.out.println(people5);
```

（2）静态工厂

将createPeople改为静态方法

```java
public static People createPeople(){
	People p=new People(7,"朱七","77");	
	return p;
}
```

bean.xml

```xml
<bean id="people6" class="com.spring.factory.PeopleFactory" factory-method="createPeople"></bean>
```

4，泛型依赖注入；(Spring4 整合Hibernate4 的时候顺带讲)



###第五节：注入参数

1，基本类型值；

测试

```java
	@Test
	public void test() {
		People people1=(People) ac.getBean("people1");
		System.out.println(people7);
	}
```



2，注入bean；

新建一个Dog类并在People中新增Dog的对象属性

```java
private int id;
private String name;
private String password;
private Dog dog;
```

bean.xml

```xml
<bean id="Dog1" class="com.spring.po.Dog">
<property name="age" value="1"></property>
<property name="name" value="Jack"></property>
</bean>

<bean id="people7" class="com.spring.po.People">
<property name="id" value="99" ></property>
<property name="name" value="小九" ></property>
<property name="password" value="999" ></property>
<property name="dog" ref="Dog1" ></property>
</bean>
```

测试

```java
	@Test
	public void test() {
		People people7=(People) ac.getBean("people7");
		System.out.println(people7);
	}
```

测试结果:

```java
People [id=99, name=小九, password=999, dog=Dog [age=1, name=Jack]]
```

3，内部bean；

xml

```xml
<bean id="people8" class="com.spring.po.People">
<property name="id" value="1010" ></property>
<property name="name" value="十" ></property>
<property name="password" value="1010" ></property>
<property name="dog">
<bean class="com.spring.po.Dog">
<property name="age" value="2"></property>
<property name="name" value="Tom"></property>
</bean>
</property>
</bean>
```

4，null 值；注意用Dog对象时会抛空指针异常

```xml
<bean id="people8" class="com.spring.po.People">
<property name="id" value="1010" ></property>
<property name="name" value="十" ></property>
<property name="password" value="1010" ></property>
<property name="dog">
   <null></null>
</property>
</bean>
```



5，级联属性；

通过People中的dog找到Dog中的属性

```xml
<bean id="people9" class="com.spring.po.People">
<property name="id" value="1111" ></property>
<property name="name" value="十1" ></property>
<property name="password" value="111" ></property>
<property name="dog.name" value="级联属性"></property>
<property name="dog.age" value="11"></property>
</bean>
```

但要注意一点是，因为dog没有关联Dog的bean，所以要在People中new一个Dog实例，不然会报异常

````java
private int id;
private String name;
private String password;
private Dog dog=new Dog();
````

【注意】确保xml的第一句<?xml version="1.0" encoding="UTF-8"?>之前没有空格

，不然会报org.xml.sax.SAXParseException; lineNumber: 1; columnNumber: 8; 不允许有匹配 "[xX][mM][lL]" 的处理指令目标

6，集合类型属性；

xml

```xml
<bean id="people10" class="com.spring.po.People">
<property name="id" value="1212" ></property>
<property name="name" value="十1" ></property>
<property name="password" value="111" ></property>
<property name="dog" ref="Dog1"></property>
<property name="hobbies" > 
<list>
<value>唱歌</value>
<value>跳舞</value>
</list>
</property>

<property name="loves">
<set>
<value>唱歌2</value>
<value>跳舞2</value>
</set>
</property>

<property name="works">
<map>
<entry>
<key><value>上午</value></key>
<value>打代码</value>
</entry>
<entry>
<key><value>下午</value></key>
<value>打篮球</value>
</entry>
</map>
</property>
  
<property name="address">
<props>
<prop key="province">广东省</prop>
<prop key="city">广州市</prop>
</props>
</property>
</bean>
```

People..java

```java
private int id;
private String name;
private String password;
private Dog dog=new Dog();
private List<String> hobbies=new ArrayList<String>();
private Set<String> loves=new HashSet<String>();
private Map<String,Object> works=new HashMap<String,Object>();
private Properties address=new Properties();
```

测试：

```java
@Test
	public void test() {
		People people10=(People) ac.getBean("people10");
		System.out.println(people10);
	}
```

测试结果

```java
People [id=1212, name=十1, password=111, dog=Dog [age=1, name=Jack], hobbies=[唱歌, 跳舞], loves=[唱歌2, 跳舞2], works={上午=打代码, 下午=打篮球}, address={province=广东省, city=广州市}]
```



###第六节：Spring 自动装配

通过配置default-autowire 属性，Spring IOC 容器可以自动为程序注入bean；默认是no，不启用自动装配；
default-autowire 的类型有byName,byType,constructor；

1.byName：通过名称进行自动匹配；:在bean.xml的收个bean标签中加上参数条件 default-autowire="byName"，即：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="byName">
        
    <bean id="dog1" class="com.spring.po.Dog">
	<property name="name" value="Jack"></property>
	</bean>
	
	 <bean id="dog" class="com.spring.po.Dog">
	<property name="name" value="Tom"></property>
	</bean>
       
	<bean id="people1" class="com.spring.po.People">
	<property name="id" value="1"> </property>
	<property name="name" value="张三"> </property>
	<property name="password"  value="123"> </property>
	</bean>

</beans>
```

因为Peopel POJO中Dog对象名为dog，所以xml会自动的找到id为dog的bean

把里面的Tom设置进dog里面去,而不是Jack（注意Dog类里面必须有setName()）

People.java

```java
	private int id;
	private String name;
	private int age;
	private Dog dog;
```

测试结果：

```java
People [id=1, name=张三, password=123, dog=Dog [name=Tom]]
```


byType：根据类型进行自动匹配；，根据类型匹配，当有多个Dog的bean的时候会报异常

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="byType">
        
    <bean id="dog1" class="com.spring.po.Dog">
	<property name="name" value="Jack"></property>
	</bean>

	<bean id="people1" class="com.spring.po.People">
	<property name="id" value="1"> </property>
	<property name="name" value="张三"> </property>
	<property name="password"  value="123"> </property>
	</bean>
</beans>
```

测试结果：

```java
People [id=1, name=张三, password=123, dog=Dog [name=Jack]] 
```

constructor：和byType 类似，只不过它是根据构造方法注入而言的，根据类型，自动注入；
建议：自动装配机制慎用，它屏蔽了装配细节，容易产生潜在的错误；

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="constructor">
        
    <bean id="dog" class="com.spring.po.Dog">
	<property name="name" value="Jack"></property>
	</bean>
	
	<bean id="people1" class="com.spring.po.People">
	<property name="id" value="1"> </property>
	<property name="name" value="张三"> </property>
	<property name="password"  value="123"> </property>
	</bean>

</beans>
```

注意People中要有以下的构造方法

```java
public People(Dog dog) {
	this.dog = dog;
}
```

运行结果:

```
People [id=1, name=张三, password=123, dog=Dog [name=Jack]]
```



比较以下情况：

(1)xml

```xml
   <bean id="dog1" class="com.spring.po.Dog">
	<property name="name" value="Jack"></property>
	</bean>
	
	<bean id="dog2" class="com.spring.po.Dog">
	<property name="name" value="Tom"></property>
	</bean>
```

结果：

```java
People [id=1, name=张三, password=123, dog=null]
```

(2)

```xml
   <bean id="dog" class="com.spring.po.Dog">
	<property name="name" value="Jack"></property>
	</bean>
	
	<bean id="dog2" class="com.spring.po.Dog">
	<property name="name" value="Tom"></property>
	</bean>
```

结果：

```java
People [id=1, name=张三, password=123, dog=Dog [name=Jack]]
```

(3)

```xml
      
	<bean id="dog2" class="com.spring.po.Dog">
	<property name="name" value="Tom"></property>
	</bean>
```

结果

```java
People [id=1, name=张三, password=123, dog=Dog [name=Tom]]
```



### 第七节：方法注入

Spring bean 作用域默认是单例singleton； 可以通过配置prototype ，实现多例；
方法注入lookup-method

测试1：

```java
System.out.println(ac.getBean("dog")==ac.getBean("dog"));
```

结果：

```java
true
```

说明Springz两次引用的是同一个对象，即单例singleton，怎么获取两个不同的对象？

在bean.xml中的odg的bean属性中加上 scope="prototype" 

即：

```xml
<bean id="dog" class="com.spring.po.Dog" scope="prototype" >
	<property name="name" value="Tom"></property>
	</bean>
```



测试2:现将dog注入到两个不同的People对象中，再比较两条狗的的引用

```xml
<bean id="people" class="com.spring.po.People">
	<property name="id" value="1"> </property>
	<property name="name" value="张三"> </property>
	<property name="password"  value="123"> </property>
		<property name="dog"  ref="dog" > </property>
	</bean>
```

测试：

```java
@Test
	public void test() {
		People people1=(People) ac.getBean("people");
		People people2=(People) ac.getBean("people");
		System.out.println(people1.getDog()==people2.getDog());
	}

```



测试结果：(说明还是同一个对象)

```
true
```

现在一如方法注入:

（1）将People改为抽象类，将getDog()改为抽象方法

（2）在bean.xml中加上look-method 标签

````xml
<bean id="people" class="com.spring.po.People">
	<property name="id" value="1"> </property>
	<property name="name" value="张三"> </property>
	<property name="password"  value="123"> </property>
	<lookup-method name="getDog" bean="dog"/>
	</bean>

````

### 第八节：方法替换

贤假设People有一条狗叫Jack，People2有条狗叫Tom，怎么通过people对象获取people2的狗Tom呢

People.java

```java
package com.spring.po;

public class People {
private int id;
private String name;
private String password;
private Dog dog;
public People() {
	super();
}
public People(Dog dog) {
	this.dog = dog;
}
public People(int id, String name, String password, Dog dog) {
	super();
	this.id = id;
	this.name = name;
	this.password = password;
	this.dog = dog;
}
public int getId() {
	return id;
}
public void setId(int id) {
	this.id = id;
}
public String getName() {
	return name;
}
public void setName(String name) {
	this.name = name;
}
public String getPassword() {
	return password;
}
public void setPassword(String password) {
	this.password = password;
}
public Dog getDog(){
	Dog dog=new Dog();
	dog.setName("Jack");
	return dog;
}

public void setDog(Dog dog) {
	this.dog = dog;
}
@Override
public String toString() {
	return "People [id=" + id + ", name=" + name + ", password=" + password + ", dog=" + dog + "]";
}
}

```

People2.java

````java
package com.spring.po;

import java.lang.reflect.Method;

import org.springframework.beans.factory.support.MethodReplacer;

public class People2 implements MethodReplacer{
@Override
public Object reimplement(Object arg0, Method arg1, Object[] arg2) throws Throwable {
	Dog dog=new Dog();
	dog.setName("Tom");
	return dog;
}
}
````

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
       >
        
	<bean id="dog" class="com.spring.po.Dog" scope="prototype" >
	<property name="name" value="Tom"></property>
	</bean>

	<bean id="people" class="com.spring.po.People">
	<property name="id" value="1"> </property>
	<property name="name" value="张三"> </property>
	<property name="password"  value="123"> </property>
	<replaced-method name="getDog" replacer="people2"></replaced-method>
	</bean>
	
	<bean id="people2" class="com.spring.po.People2">
	</bean>
</beans>
```

测试：

```java
@Test
	public void test() {
		People people=(People) ac.getBean("people");
		System.out.println(people.getDog().getName());
	}
```

结果：

```
Tom
```



### 第九节：bean 之间的关系

1，继承；

People主要属性

```java
private int id;
private String name;
private int age;
private String className;
```



bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >	
	
    <bean id="zhangsan" parent="obstractPeople">
	<property name="id" value="3"></property>
	<property name="name" value="张三"></property>
	</bean>
	
	<bean id="lisi" parent="obstractPeople">
	<property name="id" value="4"></property>
	<property name="name" value="李四"></property>
	<property name="age" value="19"></property>
	</bean>

    <bean id="obstractPeople" class="com.spring.po.People">
	<property name="className" value="高三5班"> </property>
	<property name="age"  value="18"> </property>
	</bean>	
</beans>
```

测试

```java
	@Test
	public void test() {
		People zhangsan=(People) ac.getBean("zhangsan");
     	System.out.println(zhangsan);
     	
    	People lisi=(People) ac.getBean("lisi");
     	System.out.println(lisi);
	}
```



测试结果：

```java
People [id=3, name=张三, age=18, className=高三5班]
People [id=4, name=李四, age=19, className=高三5班]
```



2，依赖；

新建一个Bean POJO

```java
package com.spring.po;

public class Authority {
public Authority(){
	System.out.println("获取权限");
}
}
```

People类无参构造

```java
public People() {
System.out.println("初始化People");
}
```

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
 
	<bean id="zhangsan" parent="obstractPeople">
	<property name="id" value="3"></property>
	<property name="name" value="张三"></property>
	</bean>
	
	<bean id="lisi" parent="obstractPeople">
	<property name="id" value="4"></property>
	<property name="name" value="李四"></property>
	<property name="age" value="19"></property>
	</bean>

    <bean id="obstractPeople" class="com.spring.po.People">
	<property name="className" value="高三5班"> </property>
	<property name="age"  value="18"> </property>
	</bean>
	
	<bean id="authority" class="com.spring.po.Authority"></bean>
	
</beans>
```

测试

```java
@Test
	public void test() {
		People zhangsan=(People) ac.getBean("zhangsan");
     	System.out.println(zhangsan);
     	
    	People lisi=(People) ac.getBean("lisi");
     	System.out.println(lisi);
	}
```

结果：（说明bean是按顺序加载的）

```java
初始化People
初始化People
初始化People
获取权限
People [id=3, name=张三, age=18, className=高三5班]
People [id=4, name=李四, age=19, className=高三5班]
```

假如想先获取权限再初始化People

(1)在第一个bean中加上depends-on属性，加载的时候就会先加载authority

````xml
	<bean id="zhangsan" parent="obstractPeople" depends-on="authority">
	<property name="id" value="3"></property>
	<property name="name" value="张三"></property>
	</bean>
````

运行结果:

````java
获取权限
初始化People
初始化People
初始化People
People [id=3, name=张三, age=18, className=高三5班]
People [id=4, name=李四, age=19, className=高三5班]
````

3，引用

Dog.java的主要属性：

```java
private int age;
private String name;
```

bean.xml   （张三引用了一条狗）

```xml
<bean id="dog" class="com.spring.po.Dog">
   <property name="name" value="Jack"></property>
      <property name="age" value="1"></property>
   </bean>
	
	<bean id="zhangsan" parent="obstractPeople" depends-on="authority">
	<property name="id" value="3"></property>
	<property name="name" value="张三"></property>
	<property name="dog" ref="dog"></property>
	</bean>
```

测试：

```java
	@Test
	public void test() {
		People zhangsan=(People) ac.getBean("zhangsan");
     	System.out.println(zhangsan);
     	
    	People lisi=(People) ac.getBean("lisi");
     	System.out.println(lisi);
	}
```

结果

```java
People [id=3, name=张三, age=18, className=高三5班, dog=Dog [age=1, name=Jack]]
People [id=4, name=李四, age=19, className=高三5班, dog=null]
```



### 第十节：bean 作用范围

1，singleton Spring ioc 容器中仅有一个Bean 实例，Bean 以单例的方式存在；

bean.xml

```xml
  <bean id="dog" class="com.spring.po.Dog">
   <property name="name" value="Jack"></property>
      <property name="age" value="1"></property>
   </bean>
```

测试：

````java
	@Test
	public void test() {
		Dog dog1=(Dog) ac.getBean("dog");
		Dog dog2=(Dog) ac.getBean("dog");
     	System.out.println(dog1==dog2);
	}
````

测试结果：说明返回的是同一条狗

```java
true
```

2，prototype 每次从容器中调用Bean 时，都返回一个新的实例；

bean.xml

````xml
  <bean id="dog" class="com.spring.po.Dog" scope="prototype">
   <property name="name" value="Jack"></property>
      <property name="age" value="1"></property>
   </bean>
````

测试结果：

```java
false
```

3，request 每次HTTP 请求都会创建一个新的Bean；
4，session 同一个HTTP Session 共享一个Bean；
5，global session 同一个全局Session 共享一个Bean，一般用于Portlet 应用环境；
6，application 同一个Application 共享一个Bean；



## 第三章Spring 之AOP 详解

### 第一节：AOP 简介

AOP 简介：百度百科；
面向切面编程(也叫面向方面编程)：Aspect Oriented Programming(AOP)，是软件开发中的一个热点，也是Spring
框架中的一个重要内容。利用AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度
降低，提高程序的可重用性，同时提高了开发的效率。
主要的功能是：日志记录，性能统计，安全控制，事务处理，异常处理等等

实例：

UserService接口

```java
package com.spring.service;

public interface UserService {
public void addUser(String name);
}
```

具体实现

```java
package com.spring.service.Impl;

import com.spring.service.UserService;

public class UserServiceImpl implements UserService{

	@Override
	public void addUser(String name) {
	System.out.println("添加学生:"+name);	
	}
}

```

bean.xml

````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
  <bean id="userServiceImpl"    class="com.spring.service.Impl.UserServiceImpl">
  </bean>
	
</beans>
````

测试`

```java
@Test
	public void test() {
		System.out.println("记录日志");
		UserService userService= (UserService) ac.getBean("userServiceImpl");
		userService.addUser("张三");
		System.out.println("业务结束");
	}
```

测试结果  (从测试代码中可以看出业务代码被侵入了，耦合性增强)

````java
记录日志
添加学生:张三
业务结束
````

解决方法是在处理业务的前后切入日志信息（面向切面编程）

(1)导入相关的jar

```java
aspectjweaver-1.8.10.jar
aopalliance-1.0.jar
spring-aspects-4.3.9.RELEASE.jar
```

(2)新建一个切面通知类

```java
package com.spring.advice;

import org.aspectj.lang.JoinPoint;

public class UserServiceAspect {
public void doBefore(JoinPoint jp){
	System.out.println("类名:"+jp.getTarget().getClass().getName());  //通过JoinPoint获取类名
	System.out.println("方法名:"+jp.getSignature().getName());
	System.out.println("获取参数:"+jp.getArgs()[0]);
	System.out.println("开始添加学生");
	}
  
public void doAfter(JoinPoint jp){
    System.out.println("类名:"+jp.getTarget().getClass().getName());  //通过JoinPoint获取类名
    System.out.println("方法名:"+jp.getSignature().getName());
    System.out.println("获取参数:"+jp.getArgs()[0]);
    System.out.println("学生添加完成");	
  }

 public Object doAround(ProceedingJoinPoint pjp) throws Throwable{
    System.out.println("添加学生前");
    Object reObject=pjp.proceed();
    System.out.println("添加学生后");
    return reObject;
  }
  
 public void doAfterReturning(JoinPoint pjp) throws Throwable{
	System.out.println("返回通知");
	}
}
}
```

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">
       
  <bean id="userServiceImpl" class="com.spring.service.Impl.UserServiceImpl"></bean>
	
	<aop:config>
	<aop:aspect id="uerServiceAspect" ref="uerServiceAspect">
	<aop:pointcut expression="execution(* com.spring.service.*.*(..))"  id="businessService"/>
	<aop:before method="doBefore" pointcut-ref="businessService"/>
    <aop:after method="doAfter" pointcut-ref="businessService"/>
    <aop:around method="doAround" pointcut-ref="businessService"/>
    <aop:after-returning method="doAfterReturning" pointcut-ref="businessService"/>
	</aop:aspect>
	</aop:config>
	<bean id="uerServiceAspect" class="com.spring.advice.UserServiceAspect"></bean>
</beans>
```

对于 

```xml
	<aop:pointcut expression="execution(* com.spring.service.*.*(..))"  id=""/>
```

*代表任意的，括号代表方法名，(..)表示方法参数任意

com.spring.service.* 则com.spring.service.Impl也会被扫描到



测试：

```java
@Test
	public void test() {
		UserService userService= (UserService) ac.getBean("userServiceImpl");
		userService.addUser("张三");
}
```

测试结果：

````java
类名:com.spring.service.Impl.UserServiceImpl
方法名:addUser
获取参数:张三
开始添加学生
添加学生前
添加学生:张三
返回通知
添加学生后
类名:com.spring.service.Impl.UserServiceImpl
方法名:addUser
获取参数:张三
学生添加完成
````





### 第二节：Spring AOP 实例。

1，前置通知:

```xml
<aop:before method="doBefore" pointcut-ref="businessService"/>
```

```java
public void doBefore(JoinPoint jp){
	System.out.println("类名:"+jp.getTarget().getClass().getName());  //通过JoinPoint获取类名
	System.out.println("方法名:"+jp.getSignature().getName());
	System.out.println("获取参数:"+jp.getArgs()[0]);
	System.out.println("开始添加学生");
	}
```

结果：

```java
类名:com.spring.service.Impl.UserServiceImpl
方法名:addUser
获取参数:张三
开始添加学生
```



2，后置通知；

```xml
 <aop:after method="doAfter" pointcut-ref="businessService"/>
```

````java
public void doAfter(JoinPoint jp){
    System.out.println("类名:"+jp.getTarget().getClass().getName());  //通过JoinPoint获取类名
    System.out.println("方法名:"+jp.getSignature().getName());
    System.out.println("获取参数:"+jp.getArgs()[0]);
    System.out.println("学生添加完成");	
  }
````

测试结果:

```java
类名:com.spring.service.Impl.UserServiceImpl
方法名:addUser
获取参数:张三
学生添加完成
```

3，环绕通知；

````xml
  <aop:around method="doAround" pointcut-ref="businessService"/>
````

````java
 public Object doAround(ProceedingJoinPoint pjp) throws Throwable{
    System.out.println("添加学生前");
    Object reObject=pjp.proceed();
    System.out.println("添加学生后");
    return reObject;
  }
````

测试结果:

```java
添加学生前
添加学生:张三
添加学生后
```

4，返回通知；

````xml
<aop:after-returning method="doAfterReturning" pointcut-ref="businessService"/>
````

````java
public void doAfterReturning(JoinPoint pjp) throws Throwable{
	System.out.println("返回通知");
	}
````

测试结果：

````java
返回通知
````

5，异常通知

````xml
<aop:after-throwing method="doAfterThrowing" pointcut-ref="businessService" throwing="ex"/>
````

```java
public void doAfterThrowing(JoinPoint jp,Throwable ex) throws Throwable{
	System.out.println("异常通知");
	System.out.println("异常信息"+ex.getMessage());
	}
```

将业务代码改为

```java
package com.spring.service.Impl;

import com.spring.service.UserService;

public class UserServiceImpl implements UserService{

	@Override
	public void addUser(String name) {
	System.out.println("添加学生:"+name);	
	System.out.println(1/0);
	}
}
```

测试结果

```java
类名:com.spring.service.Impl.UserServiceImpl
方法名:addUser
获取参数:张三
开始添加学生
添加学生前
添加学生:张三
异常通知
异常信息/ by zero
类名:com.spring.service.Impl.UserServiceImpl
方法名:addUser
获取参数:张三
学生添加完成

```





## 第四章Spring 对DAO 的支持；

### 第一节：Spring 对JDBC 的支持

1，配置数据源dbcp；

2，使用JdbcTemplate；

导入jars

```java
commons-dbcp-1.4.jar
commons-pool-1.6.jar

mysql-connector-java-5.0.8-bin.jar

spring-tx-4.3.9.RELEASE.jar
spring-jdbc-4.3.9.RELEASE.jar
```

jdbc.properties

````xml
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/db_spring
jdbc.username=root
jdbc.password=123456
````

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
       
       	<context:property-placeholder location="jdbc.properties"/>
       	
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
       </bean>
    
       <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
       <property name="dataSource" ref="dataSource">       
       </property>
       </bean>
       
       <bean id="studentDao" class="com.spring.dao.Impl.StudentDaoImpl">
       <property name="jdbcTemplate" ref="jdbcTemplate"></property>       
       </bean>
       
       <bean id="studentService" class="com.spring.service.Impl.StudentServiceImpl">
       <property name="studentDao" ref="studentDao"></property>       
       </bean>
</beans>
```

Student实体类:

````java
package com.spring.po;

public class Student {
private int id;
private int age;
private String name;
public Student(int id, int age, String name) {
	this.id=id;
	this.age = age;
	this.name = name;
}

public Student( int age, String name) {
	this.age = age;
	this.name = name;
}
public Student() {
}
public int getId() {
	return id;
}
public void setId(int id) {
	this.id = id;
}
public int getAge() {
	return age;
}
public void setAge(int age) {
	this.age = age;
}
public String getName() {
	return name;
}
public void setName(String name) {
	this.name = name;
}

@Override
public String toString() {
	return "Student [id=" + id + ", age=" + age + ", name=" + name + "]";
}
}
````

Dao接口

```java
package com.spring.dao;

import java.util.List;

import com.spring.po.Student;

public interface StudentDao {
	
	public int addStudent(Student student);
	
	public int updateStudent(Student student);
	
	public int deleteStudent(int id);
	
	public List<Student> getStudents();

}

```

Dao接口实现类DaoImpl.java

```java
package com.spring.dao.Impl;

import java.util.List;

import org.springframework.jdbc.core.JdbcTemplate;

import com.spring.dao.StudentDao;
import com.spring.po.Student;

public class StudentDaoImpl implements StudentDao{

	private JdbcTemplate jdbcTemplate;
	
	//bean.xml通过setJdbcTemplate方法将spring的JDBC模板加载进来
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}
    
	@Override
	public int addStudent(Student student) {
		String sql="insert into t_student values (null,?,?)";
		Object []params=new Object[]{student.getName(),student.getAge()};
		return jdbcTemplate.update(sql, params);
	}

	@Override
	public int updateStudent(Student student) {
		String sql="update t_student set name=?,age=? where id=?";
		Object []params=new Object[]{student.getName(),student.getAge(),student.getId()};
		return jdbcTemplate.update(sql, params);
	}

	@Override
	public int deleteStudent(int id) {
	String sql="delete from t_student where id=?";
	Object []params=new Object[]{id};
	return jdbcTemplate.update(sql, params);
	}

	@Override
	public List<Student> getStudents() {
		String sql="select * from t_student";
		final List<Student> studentList=new ArrayList<Student>();
		jdbcTemplate.query(sql, new RowCallbackHandler() {
			
			@Override
			public void processRow(ResultSet rs) throws SQLException {
				Student student=new Student();
				student.setId(rs.getInt("id"));
				student.setAge(rs.getInt("age"));
				student.setName(rs.getString("name"));
				studentList.add(student);
			}
		});
		return studentList;
	}
}

```

Service接口：

```java
package com.spring.service;

import java.util.List;

import com.spring.po.Student;

public interface StudentService {
	
	public int addStudent(Student student);
	
	public int updateStudent(Student student);
	
	public int deleteStudent(int id);
	
	public List<Student> getStudents();
	
}
```

Service接口的具体实现

```java
package com.spring.service.Impl;

import java.util.List;

import com.spring.dao.StudentDao;
import com.spring.po.Student;
import com.spring.service.StudentService;

public class StudentServiceImpl  implements StudentService{

	private StudentDao studentDao;
	
	public void setStudentDao(StudentDao studentDao) {
		this.studentDao = studentDao;
	}

	@Override
	public int addStudent(Student student) {
		return studentDao.addStudent(student);
	}

	@Override
	public int updateStudent(Student student) {
		return studentDao.updateStudent(student);
	}

	@Override
	public int deleteStudent(int id) {
		return studentDao.deleteStudent(student);
	}

	@Override
	public List<Student> getStudents() {
		return studentDao.getStudents();
	}
```

测试代码：

````java
	@Test
	public void testAddStudent() {
	Student s1=new Student(11,"张三"); 
	StudentService ss= (StudentService) ac.getBean("studentService");
	int num=ss.addStudent(s1);
	if(num==1){
	System.out.println("添加成功");	
	}
}

	@Test
	public void testUpdateStudent() {
	Student s1=new Student(1,11,"张三222"); 
	StudentService ss= (StudentService) ac.getBean("studentService");
	int num=ss.updateStudent(s1);
	if(num==1){
	System.out.println("修改成功");	
	}
}
	
	@Test
	public void testDeleteStudent() {
	StudentService ss= (StudentService) ac.getBean("studentService");
	int num=ss.deleteStudent(1);
	if(num==1){
	System.out.println("删除成功");	
	}
      
    @Test
	public void testGetStudents() {
	StudentService ss= (StudentService) ac.getBean("studentService");
	List<Student> studentList=ss.getStudents();
	for(Student s:studentList){
		System.out.println(s);
	}
}
}
````



3，JdbcDaoSupport 的使用；

先看JdbcDaoSupport的源码有以下方法

```java
public final void setDataSource(DataSource dataSource) {
		if (this.jdbcTemplate == null || dataSource != this.jdbcTemplate.getDataSource()) {
			this.jdbcTemplate = createJdbcTemplate(dataSource);
			initTemplateConfig();
		}
	}
protected JdbcTemplate createJdbcTemplate(DataSource dataSource) {
		return new JdbcTemplate(dataSource);
	}
public final DataSource getDataSource() {
		return (this.jdbcTemplate != null ? this.jdbcTemplate.getDataSource() : null);
	}
public final void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
		initTemplateConfig();
	}
public final JdbcTemplate getJdbcTemplate() {
	  return this.jdbcTemplate;
	}
```

所以可以知道在Dao层继承JdbcDaoSupport后就可以跳过jdbcTemplate这一属性而直接

获取数据源

StudentDaoImpl.java

```java
package com.spring.dao.Impl;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowCallbackHandler;
import org.springframework.jdbc.core.RowCountCallbackHandler;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.support.JdbcDaoSupport;

import com.spring.dao.StudentDao;
import com.spring.po.Student;

public class StudentDaoImpl extends JdbcDaoSupport implements StudentDao{

	@Override
	public int addStudent(Student student) {
		String sql="insert into t_student values (null,?,?)";
		Object []params=new Object[]{student.getName(),student.getAge()};
		return this.getJdbcTemplate().update(sql, params);
	}

	@Override
	public int updateStudent(Student student) {
		String sql="update t_student set name=?,age=? where id=?";
		Object []params=new Object[]{student.getName(),student.getAge(),student.getId()};
		return this.getJdbcTemplate().update(sql, params);
	}

	@Override
	public int deleteStudent(int id) {
	String sql="delete from t_student where id=?";
	Object []params=new Object[]{id};
	return this.getJdbcTemplate().update(sql, params);
	}

	@Override
	public List<Student> getStudents() {
		String sql="select * from t_student";
		final List<Student> studentList=new ArrayList<Student>();
		this.getJdbcTemplate().query(sql, new RowCallbackHandler() {
			
			@Override
			public void processRow(ResultSet rs) throws SQLException {
				Student student=new Student();
				student.setId(rs.getInt("id"));
				student.setAge(rs.getInt("age"));
				student.setName(rs.getString("name"));
				studentList.add(student);
			}
		});
		return studentList;
	}
}
```

而且bean.xml中也无需再给StudentDaoImpl配置JdbcTemplate，而是配置dataSource即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
       
       	<context:property-placeholder location="jdbc.properties"/>
       	
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
       </bean>
      
       <bean id="studentDao" class="com.spring.dao.Impl.StudentDaoImpl">
       <property name="dataSource" ref="dataSource"></property>       
       </bean>
       
       <bean id="studentService" class="com.spring.service.Impl.StudentServiceImpl">
       <property name="studentDao" ref="studentDao"></property>       
       </bean>
</beans>
```

测试略

4，NamedParameterJdbcTemplate 的使用；支持命名参数变量；org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate

```java
public NamedParameterJdbcTemplate(DataSource dataSource) {
		Assert.notNull(dataSource, "DataSource must not be null");
		this.classicJdbcTemplate = new JdbcTemplate(dataSource);
	}
```

NamedParameterJdbcTemplate源码，NamedParameterJdbcTemplate有很多重载的构造，现以DataSource为例,需在bean.xml中往NamedParameterJdbcTemplate构造器配置中加上DataSource属性

````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
       
       	<context:property-placeholder location="jdbc.properties"/>
       	
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
       </bean>
    
       <bean id="nameParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
       <constructor-arg ref="dataSource"></constructor-arg>      
       </bean>
       
       <bean id="studentDao" class="com.spring.dao.Impl.StudentDaoImpl">
       <property name="nameParameterJdbcTemplate" ref="nameParameterJdbcTemplate"></property>       
       </bean>
       
       <bean id="studentService" class="com.spring.service.Impl.StudentServiceImpl">
       <property name="studentDao" ref="studentDao"></property>       
       </bean>
</beans>
````

StudentDaoImpl.java

```java
package com.spring.dao.Impl;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import org.springframework.jdbc.core.RowCallbackHandler;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import com.spring.dao.StudentDao;
import com.spring.po.Student;

public class StudentDaoImpl implements StudentDao{

	private NamedParameterJdbcTemplate nameParameterJdbcTemplate; 
	
    public void setNameParameterJdbcTemplate(NamedParameterJdbcTemplate nameParameterJdbcTemplate) {
		this.nameParameterJdbcTemplate = nameParameterJdbcTemplate;
	}
  // 必须要这个方法，不然会报org.springframework.beans.NotWritablePropertyException（考虑底层实现）
	@Override
	public int addStudent(Student student) {
        //注意这里的格式
		String sql="insert into t_student values (null,:name,:age)"; 
	    MapSqlParameterSource msps=new MapSqlParameterSource();
	    msps.addValue("name", student.getName());
	    msps.addValue("age", student.getAge());
		return nameParameterJdbcTemplate.update(sql, msps);
	}

	@Override
	public int updateStudent(Student student) {
		String sql="update t_student set name=:name,age=:age where id=:id";
		 MapSqlParameterSource msps=new MapSqlParameterSource();
		    msps.addValue("name", student.getName());
		    msps.addValue("age", student.getAge());
		    msps.addValue("id", student.getId());
		return nameParameterJdbcTemplate.update(sql, msps);
	}

	@Override
	public int deleteStudent(int id) {
	String sql="delete from t_student where id=:id";
	 MapSqlParameterSource msps=new MapSqlParameterSource();
	    msps.addValue("id", id);
	return nameParameterJdbcTemplate.update(sql, msps);
	}

	@Override
	public List<Student> getStudents() {
		String sql="select * from t_student";
		final List<Student> studentList=new ArrayList<Student>();
		nameParameterJdbcTemplate.query(sql, new RowCallbackHandler() {
			
			@Override
			public void processRow(ResultSet rs) throws SQLException {
				Student student=new Student();
				student.setId(rs.getInt("id"));
				student.setAge(rs.getInt("age"));
				student.setName(rs.getString("name"));
				studentList.add(student);
			}
		});
		return studentList;
	}
}
```

测试代码略



### 第二节：Spring 对Hibernate 的支持

后面Spring 整合Hibernate 的时候讲；





## 第五章  Spring 对事务的支持

### 第一节：事务简介

满足一下四个条件：
第一：原子性；
第二：一致性；
第三：隔离性；
第四：持久性；

先来看个模拟银行转账的例子放松一下

db_account

````mysql
CREATE TABLE `t_count` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) DEFAULT NULL,
  `user_name` varchar(20) DEFAULT NULL,
  `count` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
````

|  id  | user_id | user_name | count |
| :--: | :-----: | :-------: | :---: |
|  1   |    1    |    张三     |  100  |
|  2   |    2    |    李四     |  50   |



BankDao

````java
package com.spring.dao;

public interface BankDao {
public void inMoney(int money,int userId);
public void outMoney(int money,int userId);
}
````

BankDaoImpl

```java
package com.spring.dao.impl;

import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

import com.spring.dao.BankDao;

public class BankDaoImpl implements BankDao{

	private NamedParameterJdbcTemplate nameParameterJdbcTemplate;
	public void setNameParameterJdbcTemplate(NamedParameterJdbcTemplate nameParameterJdbcTemplate) {
		this.nameParameterJdbcTemplate = nameParameterJdbcTemplate;
	}
	
	@Override
	public void inMoney(int money,int userId) {
		String sql="update t_account set count=count+:money where user_id=:userId ";
		  MapSqlParameterSource msps=new MapSqlParameterSource();
		  msps.addValue("money", money);
		  msps.addValue("userId", userId);
	nameParameterJdbcTemplate.update(sql, msps);
	}

	@Override
	public void outMoney(int money,int userId) {
		String sql="update t_account set count=count-:money where user_id=:userId ";
		  MapSqlParameterSource msps=new MapSqlParameterSource();
		  msps.addValue("money", money);
		  msps.addValue("userId", userId);
	nameParameterJdbcTemplate.update(sql, msps);		
	}
}
```

BankService

```java
package com.spring.service;

public interface BankService {
	public void transferAccount(int count,int fromUser,int toUser);
}
```

BankServiceImpl

`````java
package com.spring.service.impl;

import com.spring.dao.BankDao;
import com.spring.service.BankService;

public class BankServiceImpl implements BankService{
	private BankDao bankDao;
	public void setBankDao(BankDao bankDao) {
		this.bankDao = bankDao;
	}
	@Override
	public void transferAccount(int count, int fromUser, int toUser) {
		bankDao.outMoney(count, fromUser);
		bankDao.inMoney(count, toUser);		
	}
}
`````

测试：

```java
package com.spring.test;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.spring.service.BankService;

public class Test1 {

	private ApplicationContext ac;
	@Before
	public void setUp() throws Exception {
	ac= new ClassPathXmlApplicationContext("beans.xml");
	}

	@Test
	public void testtransferAccount() {
	BankService bs= (BankService) ac.getBean("bankService");
	bs.transferAccount(30, 1, 2); //张三向李四转账30元
}
}
```

数据库

|  id  | user_id | user_name | count |
| :--: | :-----: | :-------: | :---: |
|  1   |    1    |    张三     |  70   |
|  2   |    2    |    李四     |  80   |



貌似不错对吧，但如果一时疏忽把BankDaoImpl中的inMoney的sql错写为



```java

	@Override
	public void inMoney(int money,int userId) {
		String sql="update t_account2 set count=count+:money where user_id=:userId ";
		  MapSqlParameterSource msps=new MapSqlParameterSource();
		  msps.addValue("money", money);
		  msps.addValue("userId", userId);
	nameParameterJdbcTemplate.update(sql, msps);
	}
```

那就尴尬了

|  id  | user_id | user_name | count |
| :--: | :-----: | :-------: | :---: |
|  1   |    1    |    张三     |  70   |
|  2   |    2    |    李四     |  50   |



### 第二节：编程式事务管理

Spring 提供的事务模版类：org.springframework.transaction.support.TransactionTemplate
事务管理器：org.springframework.jdbc.datasource.DataSourceTransactionManager

以上例子的解决方法是：

(1)在bean.xml中加上事事务管理器，将事务管理器加入到事务管理模板

```xml
 	    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       	<property name="dataSource" ref="dataSource"></property>
       	</bean>
       	
       	<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
       	<property name="transactionManager" ref="transactionManager"></property>
       	</bean>
```

然后再把事务管理模板作为属性加入到bankService,,，这样就配置好了

````xml
 <bean id="bankService" class="com.spring.service.impl.BankServiceImpl">
       <property name="bankDao" ref="bankDao"></property>   
       <property name="transactionTemplate" ref="transactionTemplate"></property>    
 </bean>
````

(2)将BankServiceImpl改为：

````java
package com.spring.service.impl;

import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;

import com.spring.dao.BankDao;
import com.spring.service.BankService;

public class BankServiceImpl implements BankService{
	
	private TransactionTemplate transactionTemplate;
	private BankDao bankDao;
	
  
    //setter是注入前的姿势
	public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
		this.transactionTemplate = transactionTemplate;
	}
	
	public void setBankDao(BankDao bankDao) {
		this.bankDao = bankDao;
	}
	@Override
	public void transferAccount(final int count,final int fromUser,final int toUser) {
		transactionTemplate.execute(new TransactionCallbackWithoutResult() {
						
			@Override
			protected void doInTransactionWithoutResult(TransactionStatus arg0) {
				bankDao.outMoney(count, fromUser);
				bankDao.inMoney(count, toUser);					
			}
		});
	}
}
````



## 第三节：声明式事务管理



看到以上代码你可能会留意到有一大串与业务逻辑无关的代码侵入到了BankServiceImpl，这不是我们想要的，以下引入声明式事务管理



1，使用XML 配置声明式事务；

bean.xml

`````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
      xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
       
       	<context:property-placeholder location="jdbc.properties"/>
       
        <!-- 配置事务管理器 -->
       	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       	<property name="dataSource" ref="dataSource"></property>
       	</bean>
       	
       	<!-- 配置事务通知 -->
       	<tx:advice id="txAdvice" transaction-manager="transactionManager">
       	<tx:attributes>
       	<tx:method name="*"/>
       	</tx:attributes>
       	</tx:advice>
       	
       	<!-- 配置事务切面 -->
       <aop:config>
    	<!-- 配置切点 -->
    	<aop:pointcut id="serviceMethod" expression="execution(* com.spring.service.*.*(..))" />
    	<!-- 配置事务通知 -->
    	<aop:advisor advice-ref="txAdvice" pointcut-ref="serviceMethod"/>
       </aop:config>
    	       	   	           	
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
       </bean>
    
       <bean id="nameParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
       <constructor-arg ref="dataSource"></constructor-arg>      
       </bean>
       
       <bean id="bankDao" class="com.spring.dao.impl.BankDaoImpl">
       <property name="nameParameterJdbcTemplate" ref="nameParameterJdbcTemplate"></property>          </bean>
       
       <bean id="bankService" class="com.spring.service.impl.BankServiceImpl">
       <property name="bankDao" ref="bankDao"></property>       
       </bean>
</beans>
`````

BanKService.java

`````java
package com.spring.service.impl;
import com.spring.dao.BankDao;
import com.spring.service.BankService;

public class BankServiceImpl implements BankService{
	
	private BankDao bankDao;

	public void setBankDao(BankDao bankDao) {
		this.bankDao = bankDao;
	}
	@Override
	public void transferAccount(int count,int fromUser,int toUser) {		
				bankDao.outMoney(count, fromUser);
				bankDao.inMoney(count, toUser);							
	}
}
`````

2，使用注解配置声明式事务；

在BankServiceImpl中加入@Transactional注解

````java
package com.spring.service.impl;
import org.springframework.transaction.annotation.Transactional;
import com.spring.dao.BankDao;
import com.spring.service.BankService;

@Transactional
public class BankServiceImpl implements BankService{
	
	private BankDao bankDao;

	public void setBankDao(BankDao bankDao) {
		this.bankDao = bankDao;
	}
	@Override
	public void transferAccount(int count,int fromUser,int toUser) {		
				bankDao.outMoney(count, fromUser);
				bankDao.inMoney(count, toUser);							
	}
}
````

在bean.xml在配置注解驱动

````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
      xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
       
       	<context:property-placeholder location="jdbc.properties"/>
       	
        <!-- 配置JDBC事务管理器 -->
       	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       	<property name="dataSource" ref="dataSource"></property>
       	</bean>

    	<tx:annotation-driven transaction-manager="transactionManager"/>
       	   	           	
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
       </bean>
    
       <bean id="nameParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
       <constructor-arg ref="dataSource"></constructor-arg>      
       </bean>
       
       <bean id="bankDao" class="com.spring.dao.impl.BankDaoImpl">
       <property name="nameParameterJdbcTemplate" ref="nameParameterJdbcTemplate"></property>       
       </bean>
       
       <bean id="bankService" class="com.spring.service.impl.BankServiceImpl">
       <property name="bankDao" ref="bankDao"></property>       
       </bean>
</beans>
````

两种方法比较：当Service比较多的时候使用XML 配置声明式事务（简单粗暴，不用加一堆）

## 第四节：事务传播行为

事务传播行为：Spring 中，当一个service 方法调用另外一个service 方法的时候，因为每个service 方法都有事
务，这时候就出现了事务的嵌套；由此，就产生了事务传播行为；
在Spring 中，通过配置Propagation，来定义事务传播行为；
PROPAGATION_REQUIRED--支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
PROPAGATION_SUPPORTS--支持当前事务，如果当前没有事务，就以非事务方式执行。
PROPAGATION_MANDATORY--支持当前事务，如果当前没有事务，就抛出异常。
PROPAGATION_REQUIRES_NEW--新建事务，如果当前存在事务，把当前事务挂起。
PROPAGATION_NOT_SUPPORTED--以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
PROPAGATION_NEVER--以非事务方式执行，如果当前存在事务，则抛出异常。

````xml
<tx:attributes>

<tx:method name="insert*" propagation="REQUIRED" />

<tx:method name="update*" propagation="REQUIRED" />

<tx:method name="edit*" propagation="REQUIRED" />

<tx:method name="save*" propagation="REQUIRED" />

<tx:method name="add*" propagation="REQUIRED" />

<tx:method name="new*" propagation="REQUIRED" />

<tx:method name="set*" propagation="REQUIRED" />

<tx:method name="remove*" propagation="REQUIRED" />

<tx:method name="delete*" propagation="REQUIRED" />

<tx:method name="change*" propagation="REQUIRED" />

<tx:method name="get*" propagation="REQUIRED" read-only="true" />

<tx:method name="find*" propagation="REQUIRED" read-only="true" />

<tx:method name="load*" propagation="REQUIRED" read-only="true" />

<tx:method name="*" propagation="REQUIRED" read-only="true" />

</tx:attributes>
````



## 第六章Spring4 整合Hibernate4，Struts2

第一节：S2SH 整合所需Jar 包

Struts2.3.16，Spring4.0.6，Hibernate4.3.5 整合所需jar 包；



![微信图片_20170715195937](C:\Users\Administrator\Desktop\微信图片_20170715195937.png)



Spring4.0.6 jar 包

![微信图片_20170715200049](C:\Users\Administrator\Desktop\微信图片_20170715200049.png)

Hibernate4.3.5 jar 包

![微信图片_20170715200127](C:\Users\Administrator\Desktop\微信图片_20170715200127.png)



