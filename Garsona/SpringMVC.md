## 配置web.xml

web.xml

```xml
<!-- 配置DispatcherServlet  -->
<servlet>
  <servlet-name>springDispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- 
         配置DispatcherServlet的一个初始化参数：配置SpringMVC配置文件的位置和名称 
         也可以不配置以下的init-param，默认的配置文件为：/WEB-INF/<servlet-name>-servlet.xml
		 也即可以将spring配置文件改名为<servlet-name>-servlet.xml并放入WEB-INF文件夹中
  -->
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>springDispatcherServlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

## 配置springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd  
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd  
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/jee 
        http://www.springframework.org/schema/jee/spring-jee-4.0.xsd  
        http://www.springframework.org/schema/tx 
        http://www.springframework.org/schema/tx/spring-tx-4.0.xsd"> 
  
  </beans>
```



## 一、`RequestMapping`注解

### 1、基础用法及配置

spring配置文件

```xml
<!-- 扫描注解 -->
<context:component-scan base-package="handlers" />

<!-- 配置视图解析器：如何把handler方法返回值解析为实际的物理视图 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="prefix" value="/WEB-INF/" /><!-- 为返回值设置前缀 -->
  <property name="suffix" value=".jsp" /><!-- 为返回值设置后缀 -->
</bean>
```

控制层类：

```java
@Controller
public class HelloWorld {

	/**
	 * 映射请求的url
	 * 返回值会通过视图解析器解析为实际的视图，对于InternalResourceViewResolver解析器，会做如下操作：
	 * prefix + returnValue + 后缀 得到实际的物理视图，然后做转发操作
	 * @return
	 */
	@RequestMapping("/helloworld")//此方法映射超链接的请求，执行此方法并执行跳转
	public String hello(){
		System.out.println("hello world");
		return "Success";//返回的值应该与jsp页面的名称相同，相当于跳转到Success.jsp
	}
	
}
```

jsp上：

```jsp
<a href="helloworld">HelloWorld</a>
```

### 2、修饰类

@RequestMapping

类定义处：提供初步的请求映射信息，相对于web应用的根目录，即webapp

方法处：提供进一步的细分映射信息，相对于类定义处的URL。

若类定义处未标注，则方法处标记的url相对于web应用根目录



控制层类：

```java
@Controller
@org.springframework.web.bind.annotation.RequestMapping("/springmvc")
public class RequestMapping {

	@org.springframework.web.bind.annotation.RequestMapping("/testRequestMapping")
	public String testRequestMapping(){
		System.out.println("111");
		return "Success";
	}
	
}
```

则访问的url应该为springmvc/testRequestMapping

```jsp
<a href="springmvc/testRequestMapping">测试</a>
```

另：WEB-INF文件夹下的jsp无法通过客户端访问，只能由服务端访问，无法在浏览器里直接访问，因此可以通过请求转发的方式访问

### 3、指定请求方式

```java
/**
 * 使用method属性来指定请求方式
 * @return
 */
@org.springframework.web.bind.annotation.RequestMapping(value="/testMethod" ,method=RequestMethod.POST)
  public String testMethod(){
  return "Success";
}
```

### 4、指定映射请求的参数、请求头

（1）`params={param1}`

param1：表示请求必须包含名为param1的请求参数

!param1：表示请求不能包含名为param1的请求参数

param1!=value1：表示请求包含名为param1的请求参数但其值不能为value1

```java
@org.springframework.web.bind.annotation.RequestMapping(value="/testParamsAndHeaders" ,
				params={"userName" ,"age!=10"})
public String testParamsAndHeaders(){
  return "Success";
}
```

jsp中：

```jsp
<a href="springmvc/testParamsAndHeaders?userName=chenjiasheng&age=11">testParamsAndHeaders</a>
```

（2）headers：设定请求头信息，在浏览器工具中Header中有各类信息

![](http://ot0aou666.bkt.clouddn.com/%E8%AF%B7%E6%B1%82%E5%A4%B4%E4%BF%A1%E6%81%AF.png)

```java
@org.springframework.web.bind.annotation.RequestMapping(value="/testParamsAndHeaders" ,
				headers="Accept-Encoding=gzip, deflate, br")
public String testParamsAndHeaders(){
  return "Success";
}
```

### 5、Ant风格的url

？：匹配文件名的一个字符

*：匹配文件名中的任意字符

**：匹配多层路径

例如：

“/helloworld/index?”可以匹配“/helloworld/indexA”、“/helloworld/indexB”，但不能匹配“/helloworld/index”也不能匹配“/helloworld/indexAA”；

“/helloworld/index*”可以匹配“/helloworld/index”、“/helloworld/indexA”、“/helloworld/indexAA”但不能匹配“/helloworld/index/A”；

“/helloworld/index/*”可以匹配“/helloworld/index/”、“/helloworld/index/A”、“/helloworld/index/AA”、“/helloworld/index/AB”但不能匹配    “/helloworld/index”、“/helloworld/index/A/B”;

“/helloworld/index/**”可以匹配“/helloworld/index/”下的多有子路径，比如：“/helloworld/index/A/B/C/D”;

如果现在有“/helloworld/index”和“/helloworld/*”，如果请求地址为“/helloworld/index”那么将如何匹配？Spring MVC会按照最长匹配优先原则（即和映射配置中哪个匹配的最多）来匹配，所以会匹配“/helloworld/index”

## 二、关于参数属性的注解

### 1、PathVariable注解

可通过此注解将url中占位符参数 {xxx} 绑定到控制器处理方法的入参中

```java
/**
 * 可以映射url中的占位符到目标方法的参数上，占位符内的属性名要与注解内的相同
 * @param id
 * @return
 */
@org.springframework.web.bind.annotation.RequestMapping("/testPathVariable/{id}")
public String testPathVariable(@PathVariable("id") int id){
  	System.out.println(id);
  	return "Success";
}
```

```jsp
<a href="springmvc/testPathVariable/1">testPathVariable</a>
```

### 2、关于REST风格

http://www.ruanyifeng.com/blog/2011/09/restful.html ,可查看此博文

关于rest风格，看了网上的资料，总结一下就是将url当作一种资源实体化，对其进行增删查改，其中的PUT、DELETE、GET、POST分别对应，这是http为区别对待资源而预设的几个method，但其实PUT以及GET就能够满足我们的需求，因此其余两个method更多的是要我们符合语义化，个人感觉真正采用rest风格可能会带来更惨的结果

首先在web.xml中配置过滤器

```xml
<!-- 配置HiddenHttpMethodFilter：可以把POST请求转为DELETE或PUT请求 -->
<filter>
  <filter-name>HiddenHttpMethodFilter</filter-name>
  <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>HiddenHttpMethodFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

以DELETE为例：

jsp中：

```jsp
<form action="springmvc/testRest/1" method="post">
  <input type="hidden" name="_method" value="DELETE" /><!-- 其中_method为过滤器中的属性值，将POST转变为DELETE -->
  <input type="submit" value="TestRestDELETE" />
</form>
```

java中：

```java
@org.springframework.web.bind.annotation.RequestMapping(value="/testRest/{id}" ,method=RequestMethod.DELETE)
public String testRestDelete(@PathVariable int id){
  return "Success";
}
```

### 3、`@RequestParam`注解

与`@PathVariable`作用相当

```java
/**
 * @RequestParam 来映射请求参数
 * value 值即请求参数的参数名
 * required 该参数是否必须，默认为true
 * defaultValue 请求参数的默认值
 * @param un
 * @param age
 * @return
 */
@org.springframework.web.bind.annotation.RequestMapping("/testParam")
  public String testParam(@RequestParam(value="username") String un ,
                          @RequestParam(value="age" ,required=false ,defaultValue="0") int age){
  return "Success";
}
```

### 4、`@RequestHeader`注解

与`@RequestParam`类似，设置请求头属性

```java
@org.springframework.web.bind.annotation.RequestMapping("/testRequestHeader")
  public String testParam(@RequestHeader(value="Accept-Encoding=gzip, deflate, br") String al){
  return "Success";
}
```

### 5、`@CookieValue`获取cookie

```java
@org.springframework.web.bind.annotation.RequestMapping("/testCookie")
public String testCookie(@CookieValue(value="JSESSIONID") String sessionId){
  System.out.println(sessionId);
  return "Success";
}
```

### 6、使用POJO为参数

User类中有类型为Address的属性

user:

```java
public String name;
public int age;
public Address address;
```

jsp：

```jsp
<form action="springmvc/testPoJo" method="post">
  username:<input type="text" name="username">
  age:<input type="text" name="age">
  city:<input type="text" name="address.city">
  <input type="submit" name="submit">
</form>
```

java:

```java
@org.springframework.web.bind.annotation.RequestMapping("/testPoJo")
public String testPoJo(User user){
  System.out.println(user);
  return "Success";
}
```

### 7、使用Servlet原生API作参数

```java
@org.springframework.web.bind.annotation.RequestMapping("/testPoJo")
public String testServletApi(HttpServletRequest request ,
				HttpServletResponse response ,Writer out) throws IOException{
  System.out.println("request : " + request + "response : " + response);
  out.write("hello mvc");//此处的out其实是response.getWriter()
  return "Success";
}
```



## 三、处理模型数据

### 1、ModelAndView

```java
/**
 * 目标方法的返回值可以是ModelAndView类型
 * 其中可以包括视图和模型信息
 * SpringMVC会把ModelAndView的model中的数据放入到request域对象中
 * @return
 */
@org.springframework.web.bind.annotation.RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
  String viewName = "Success";//跳转页面的名称
  ModelAndView modelAndView = new ModelAndView(viewName);
  modelAndView.addObject("time", new Date());//在该页面上加上数据,可以相当于request.setAttribute
  return modelAndView;
}
```

Success.jsp

```jsp
<h2>time: ${requestScope.time }</h2>
```

另：requestScope是EL表达式中的一个隐含对象，类似request，如${requestScope.username }表示从request域中获取username属性对应的值，相当于request.getAttribute(“username”);

### 2、Map

```java
/**
 * 目标方法可以添加Map类型（实际上也可以是Model或ModelMap类型）的参数
 * @param map
 * @return
 */
@org.springframework.web.bind.annotation.RequestMapping("/testMap")
  public String testMap(Map<String ,Object> map){
  map.put("names", Arrays.asList("chen" ,"hong"));
  return "Success";
}
```

Success.jsp

```jsp
<h2>name: ${requestScope.names }</h2>
```

### 3、将模型数据置入session

在类开头使用`@SessionAttributes`

```java
@SessionAttributes(value={"user"})
@org.springframework.web.bind.annotation.RequestMapping("/springmvc")
@Controller
public class RequestMapping {
  /**
   * @SessionAttributes可以通过属性名指定需要放到会话中的属性外，
   * 还可以通过模型属性的对象类型指定哪些模型属性需要放到会话中
   * @param map
   * @return
   */
	@org.springframework.web.bind.annotation.RequestMapping("/testSession")
	public String testSession(Map<String ,Object> map){
		User user = new User("chen", 15);
		map.put("user", user);
		return "Success";
	}
```

## 四、@ModelAttribute 注解

### 1、使用

@ModelAttribute标记的方法会在每个目标方法执行之前被springMVC调用，多用于在执行目标方法前向数据库获取信息

```java
/**
 * @ModelAttribute 标记的方法会在每个目标方法执行之前被SpringMVC调用
 * @param id
 * @param map
 */
	@ModelAttribute
	public void getUser(@RequestParam(value="id" ,required=false) Integer id,
				Map<String ,Object> map){
		if(id != null){
			System.out.println("modelAttribute");
			User user = new User(1 ,"chen", 15);
			System.out.println("从数据库中取一个对象 ： " + user);
			map.put("user", user);//map的key应该要与目标方法的POJO类型参数的参数名一致
		}
	}

	@org.springframework.web.bind.annotation.RequestMapping("/testModelAttribute")
	public String testModelAttribute(User user){
		System.out.println("修改：" + user);
		return "Success";
	}
```

### 2、springMVC确定目标方法pojo型入参过程

![](http://ot0aou666.bkt.clouddn.com/SpringMVC%E7%A1%AE%E5%AE%9A%E7%9B%AE%E6%A0%87%E6%96%B9%E6%B3%95POJO%E7%B1%BB%E5%9E%8B%E5%85%A5%E5%8F%82.png)

## 五、超级浅析视图解析器执行流程



![](http://ot0aou666.bkt.clouddn.com/springMVC%E8%A7%86%E5%9B%BE%E8%A7%A3%E6%9E%90%E5%99%A8.png)



另：使用mvc: view-controller标签

```xml
<!-- 配置直接转发的页面 -->
<!-- 可以直接响应转发的页面，无需再经过Handler的方法 -->
<mvc:view-controller path="/Success" view-name="Success"/>

<!-- 实际开发中通常都需要配置此标签 -->
<mvc:annotation-driven />
```

重定向或请求转发：

```java
@org.springframework.web.bind.annotation.RequestMapping("/testRedirect")
	public String testRedirect(){
		return "redirect:/helloworld.jsp";//或者return "forward:/helloworld.jsp"请求转发
	}
```



## 六、HttpMessageConverter

### 1、原理

[点击此博文查看](http://www.cnblogs.com/winner-0715/p/6512806.html)，该博文前段概述了消息转换器的用途，后段阐述了实现代码

### 2、使用

![](http://ot0aou666.bkt.clouddn.com/%E4%BD%BF%E7%94%A8HttpMessageConverter.png)



`@ResponseBody`与`@RequestBody`

在链接网页上输出返回值

```java
@ResponseBody
@RequestMapping("/testHttpMessageConverter")
public String testHttpMessageConverter(@RequestBody String body){
  System.out.println(body);
  return "helloworld! " + new Date();
}
```

下载文件(以aaa.png为例)：

```java
@RequestMapping("/testResponseEntity")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException{
  ServletContext servletContext = session.getServletContext();
  InputStream in = servletContext.getResourceAsStream("/image/aaa.png");
  byte[] body = new byte[in.available()];
  in.read(body);

  HttpHeaders headers = new HttpHeaders();
  headers.add("Content-Disposition" ,"attachment;filename=aaa.png");

  HttpStatus statusCode = HttpStatus.OK;

  ResponseEntity<byte[]> response = new ResponseEntity<byte[]>(body, headers, statusCode);

  return response;
}
```

## 七、上传文件

springMVC.xml配置：

```xml
<!-- 配置上传文件的解析器，其中可配置属性来定义各种条件，如文件上传的最大字节数等等 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="defaultEncoding" value="UTF-8" />
	<property name="maxUploadSize" value="1024000" />
</bean>
```

```java
/**
 * 可在方法中获取文件的各种信息，如输入流信息等，然后来进行文件上传
 * @param desc
 * @param file
 * @return
 * @throws IOException
 */
@RequestMapping("/testFileUpload")
public String testFileUpload(@RequestParam(value="desc" ,required=false) String desc,
		@RequestParam("file") MultipartFile file) throws IOException{
	System.out.println(desc);
	System.out.println("fileName: " + file.getOriginalFilename());
	System.out.println(file.getInputStream());
	return "Success";
}
```



## 八、拦截器

### 1、配置自定义拦截器

自定义拦截器：

```java
//实现HandlerInterceptor接口
public class FirstInterceptor implements HandlerInterceptor {
  	
  	/**
	 * 该方法在目标方法之前被调用
	 * 若返回值为true，则继续调用后续的拦截器和目标方法
	 * 若返回值为false，则不会再调用后续的拦截器和目标方法
	 * 
	 * 可以考虑做权限、日志、事务等等
	 */
	@Override
	public boolean preHandle(HttpServletRequest request,
			HttpServletResponse response, Object handler) throws Exception {
		System.out.println("pre");
		return true;
	}

    /**
	 * 调用目标方法之后，渲染视图之前被调用
	 * 
	 * 可以对请求域中的属性或视图做出修改
	 */
	@Override
	public void postHandle(HttpServletRequest request,
			HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("post");
	}

  	/**
	 * 渲染视图之后
	 * 
	 * 释放资源
	 */
	@Override
	public void afterCompletion(HttpServletRequest request,
			HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		// TODO Auto-generated method stub
		System.out.println("after");
	}

}
```

Spring.xml中配置拦截器：

```xml
<mvc:interceptors>
  <!-- 配置自定义拦截器 -->
  <bean class="interceptors.FirstInterceptor" />
  <!-- 配置拦截器（不）作用的路径       不作用：<mvc:exclude-mapping> -->
  <mvc:interceptor>
    <mvc:mapping path="/testCookie"/>
    <!-- 指定拦截器 -->
    <bean class="interceptors.SecondInterceptor" />
  </mvc:interceptor>
</mvc:interceptors>
```

### 2、多个拦截方法的执行顺序

![](http://ot0aou666.bkt.clouddn.com/%E5%A4%9A%E4%B8%AA%E6%8B%A6%E6%88%AA%E5%99%A8%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F.png)

preHandle：按配置正序执行

postHandle、afterCompletion：按配置反序执行

若preHandle返回false，则直接执行FirstInterceptor的afterCompletion

## 九、异常处理

### 1、

定义异常：

```java
@ResponseStatus(value=HttpStatus.FORBIDDEN ,reason="密码不正确")
public class notMatchPasswordException extends RuntimeException {

	/**
	 * 
	 */
	private static final long serialVersionUID = 1L;

}
```

其中`@ResponseStatus`可定义异常的状态以及原因

测试类：

```java
@RequestMapping("/testException")
public String testException(@RequestParam("id") int id){
  if(id == 13){
    //若请求传入的参数id值为13，则抛异常
    throw new notMatchPasswordException();
  }else{
    return "Success";
  }
}
```

### 2、

`DefaultHandlerExceptionResolver`对一些特殊的异常进行处理，比如：`NoSuchRequestHandlingMethodException`等



## 十、MVC运行流程

![](http://ot0aou666.bkt.clouddn.com/SpringMVC%E8%BF%90%E8%A1%8C%E6%B5%81%E7%A8%8B.png)









善用扫描包中的`<context:include-filter type="annotation" expression="">`

SpringMVC的IOC容器中的bean可以来引用Spring IOC容器中的bean，反之则不行。Spring IOC容器中的bean却不能来引用SpringMVC IOC容器中的bean