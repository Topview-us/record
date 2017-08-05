# 代理

## 1. 静态代理

1.1.角色分析：

抽象角色---------使用接口或者抽象类来实现

```java
public interface Rent {
	public void rent();
}
```

真实角色---------被代理的角色

```java
public class Host implements Rent{
	@Override
	public void rent(){
		System.out.println("房屋出租");
	}
}
```

代理角色----------代理真实角色-----代理真实角色后一般会做一些附属操作

```java
import javax.xml.parsers.FactoryConfigurationError;

public class Proxy implements Rent {
  	private Host host;
	public void setHost(Host host) {
		this.host = host;
	}

	public Proxy(Host host) {
		super();
		seeHouse();
		this.host = host;
		fare();
	}
	private void seeHouse(){ 
		System.out.println("带顾客看房");
	}
	private void fare(){
		System.out.println("收取中介费");
	}
	@Override
	public void rent(){
		host.rent();
	}
}
```

客户-----使用代理角色来进行一些操作

```java
public class Client {
	public static void main(String[] args) {
		Host host = new Host();
		Proxy proxy =new Proxy(host);
		proxy.rent();
	}
}
```

关系图
![](images/proxy.png)

代理就是将真实角色中的公共业务交于代理类，这使得真实角色变得更加纯粹，专注于自身的领域业务

代理的目的：实现业务的分工：领域业务和公共业务的分离
​			公共业务发生扩展时变得更加集中和方便

静态代理的缺点：
​	多了代理类

​	公共业务

## 2. 动态代理

2.1.分类

代理类：1)基于接口的动态代理：jdk

​		2)基于类的动态代理：cglib

现在javasjst来生成动态代理

2.2.jdk动态代理
InvocationHandler是 代理类的 实例调用处理程序 实现的接口

每个代理实例

其方法：invoke
入参：proxy代理实例，method对应代理实例上调用的接口方法

Proxy 提供用于创建动态代理类和实例的静态方法，还是由这些方法常见的所有动态代理类的父类

如何创建某一接口foo的代理“

```java
//得到handler
InvocationHandler handler = new MyInvocationHandler();
//得到代理类，该类实现了Foo的接口
Class proxyClass  = Proxy.getProxyClass(Foo.class.getClassLoader(),new Class[]{Foo.class});
Foo f = (Foo)proxyClass.getConstructor(new Class[]{InvocationHandler,class})
  						.newInstance(new Object[]{handler});
```

这里的方法newProxyInstance()，入参：类加载器
能返回一个带有代理类的指定调用处理程序的代理实例，它由指定类加载器所定义，并实现指定接口

2.3.角色分析

代理角色

```java
package dynamicproxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyInvocationHandler implements InvocationHandler {
/**
 * proxy是代理类
 * method 代理类的调用处理程序的方法对象
 */
	private Object target;
	public void setTarget(Object target) {
		this.target = target;
	}
	public Object getProxy(){
		return Proxy.newProxyInstance(this.getClass().getClassLoader()
				, target.getClass().getInterfaces(), this);
      //入参：类加载器，目标对象的接口，指派方法调用的调用处理程序
      //返回：一个带有代理类的指定调用处理程序的代理实例，它由类加载器定义，并实现指定的接口
	}
	@Override
	public Object invoke(Object arg0, Method arg1, Object[] arg2) 
			throws Throwable {
		System.out.println("带房客看房");
		Object result = arg1.invoke(target, arg2);
		System.out.println("");
		return result;			
	}
}
```

真实角色

```java
package dynamicproxy;

public class Host implements Rent{
	@Override
	public void rent() {
		System.out.println("房屋出租");	
	}
}
```

接口

```java
package dynamicproxy;

public interface Rent {
	public void rent();
}

```

客户

```java
package dynamicproxy;

public class Client {
	public static void main(String[] args) {
      //1.创建真实对象
		Host host =new Host();
      //2.创建invocationHandler（调用处理程序）
		ProxyInvocationHandler proxyInvocationHandler = new ProxyInvocationHandler();
      //3.将目标对象传入调用处理程序
		proxyInvocationHandler.setTarget(host);
      //4.得到代理实例，他实现了目标对象的接口，因此可以强转
		Rent proxy = (Rent) proxyInvocationHandler.getProxy();
		proxy.rent();
	}
}
```



一般而言：一个动态代理一般代理某一类业务。

​	

