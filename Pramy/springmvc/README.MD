# 一 mvc的基本配置

首先在web.xml里面配置先启动springmvc的servlet

```xml
<servlet>
  <servlet-name>springDispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
  </init-param>
  <!--如果不填配置文件的话，默认的配置文件为: /WEB-INF/<servlet-name>-servlet.xml-->
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>springDispatcherServlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

然后再controller写接收请求的bean

```java
@Controller
public class Hello {
  	//表示请求的路径
    @RequestMapping("/helloWorld")
    public String hello(){
        System.out.println("Hello");
        return "success";
    }
  //返回值相当于重定向，不过需要在spring.xml上面配置识图管理器
}
```

- 单独在类定义处使用：相对于WEB应用的根目录
- 单独在方法处使用：相对于WEB应用的根目录
- 如果在方法和类中同时使用，则方法的路径相当于类的路径的子路径



视图管理器

```xml
<context:component-scan base-package="com.pramy.handler"/>
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <!--前缀修饰，加后缀修饰最后变成/WEB-INF/views/success.jsp,相当于重定向到这-->
    <property name="prefix" value="/WEB-INF/views/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

## 指定请求方法，参数，头

请求参数

```java
@RequestMapping(value = "param",params = {"userName","age!=10","!sex"},method = RequestMethod.POST)
//意思就是参数一定要有userName而且age！=10而且不能带有sex,指定方法为post
//还可以用headers来指定请求头
```

## 路径匹配

- ?：匹配一个字符
- *：匹配多个字符
- **：匹配多层路径

## @PathVariable

```java
@RequestMapping(value = "/PathVariable/{id}")
public String PathVariable(@PathVariable("id")int id)
```

请求路径为/PathVariable/123，PathVariable可以将占位符数据映射为参数，注意数据类型要匹配，一般用来匹配基本数据类型

## 发送put请求和delete请求

1.首先需要在web.xml配置filter，而且必须发送post请求才可以转

```xml
  <!-- 配置HiddenHttpMethodFilter：可以把POST转化为DELETE和PUT请求 -->
  <filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

2.在表单提交的时候需要加上隐藏域

```xml
<form action="/testRequestMapping/testRestDelete/1" method="post">
    <input type="hidden" name="_method" value="DELETE">
    <input type="submit">
</form>
```

## 获取参数

@RequestParam

```java
public String put(@RequestParam(value = "userName")String userName,@RequestParam(value = "password")String password)
```

- value 值即为请求参数的参数名
- required 该参数是否是必须的，默认值是true
- defaultValue 请求参数的默认值

## 获取cookiesId

```java
public String cookies(@CookieValue(value = "JSESSIONID")String id)
```



# 二 数据模型

## POJO的封装

支持对pojo的封装，自动匹配属性，支持级联属性

```xml
<form action="/spring/pojo" method="post">
    <input type="hidden" value="pramy" name="userName">
    <input type="hidden" name="password" value="123">
    <input type="hidden" name="cat.catName" value="cat">
    <input type="hidden" name="cat.color" value="white">
    <input type="submit" value="提交">
</form>
```
cat是user里面的对象

```java
@RequestMapping(value = "/pojo")
public String pojo(User user){
```

不紧如此还可以用到servlet原生的API

- HttpServletRequest
- HttpServletResponse
- HttpSession
- java.security.Principal
- Locale
- InputStream
- OutputStream
- Reader
- Writer 

## ModelAndView对象

```java
ModelAndView mv = new ModelAndView();
mv.addObject("user",user);
```

这个方法相当于request.setAttribute();

底层原理就是其中有一个map，然后遍历这个map，把它setAttribute进request中

## Map作为参数

当传入值是map的时候

```java
public String testMap(Map<String,Object>map)
```

但这个map是BindingAwareModelMap类型，当你往里面添加东西的时候实际上是向ModeAndView中添加对象，相当于添加到request中

## 添加到session

```java
@SessionAttributes(value = "names",types = {String.class, User.class})
```

只能在类上面标记

- value：意思就是把key为names的值方法session中
- types：意思是把所有String类型和User类型的值都放到session中