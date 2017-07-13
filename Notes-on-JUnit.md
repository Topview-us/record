# Notes on JUnit 

标签（空格分隔）： 未分类

---

###测试
如写一个方法给别人用，保险起见应该对它进行测试，原始的方法是写一个main方法，但这太过费时费力，一方面是因为多个main不能同时运行，且测试结果需要认为观察，所以引入JUnit这一

###单元测试
原因：
重用测试，应付将来实现的变化
提高士气，明确我的代码没问题
降低后期维护成本

###静态引入
JDK1.5以后增加静态引入特性
即如
import static org.JUnit.Assert.*;
下面写代码时就不用“类名。方法名”这样调用，而是直接写方法名即可
eclipse中自带了JUnit的包（可能不是最新的），当然也可以自己导入外部包


##断言

###老断言：
常用方法
assertEquals(expected,actual);
assertTrue(String message, boolean condition) message为你自定义的想要看到的错误提示信息
.....
.....
.....

待测类
```Java
package com.tv.junit4;

public class T {
public int add (int x,int y){
	return x+y;
}
}
```

测试类(Test Case)
```Java
package com.tv.junit4.test;

import static org.junit.Assert.*;

import org.junit.Test;
import com.tv.junit4.T;

public class TTest {

	@Test
	public void testAdd() {
	int result=new T().add(1,1);
	assertEquals(2,result);
	}
}
```
此时将测试类run as JUnit Test 即可显示测试结果
（Keeps bar green to keep code clean）



###新断言
**assertThat(T actual, org.hamcrest.Matcher<T> matcher)**

该方法可以替代其他所有以assert开头的测试方法，该方法使用前提是有hamcrest的包

【注意】用eclipse自带的JUnit的Lib使用assertThat方法时，会报SecurityException，解决方法是将原来Junit4的库remove，再从外部导入JUnit的jar包即可

####assertThat的具体用法
```Java
assertThat( n, allOf( greaterThan(1), lessThan(15) ) );  //满足全部条件才能通过
assertThat( n, anyOf( greaterThan(16), lessThan(8) ) );
//满足一个条件即可通过
assertThat( n, anything() );  //无条件通过
a)assertThat( str, is( "abc" ) );  //str是abc才通过
assertThat( str, not( "bjxxt" ) ); //str不是abc才通过
ssertThat( str, containsString( "abc" ) ); //str包含abc才通过
assertThat( str, endsWith("abc" ) );   //str以abc结尾才通过
assertThat( str, startsWith( "bjsxt" ) );  //str以abc开头才通过
assertThat( n, equalTo( nExpected ) );  //n与nExpected相等才通过
assertThat( str, equalToIgnoringCase( "abc" ) );  // .....
assertThat( str, equalToIgnoringWhiteSpace( "abc" ) ); 
assertThat( d, closeTo( 3.0, 0.3 ) ); //d在误差为0.3情况下接近3.0时通过
assertThat( d, greaterThan(3.0) );  //....
assertThat( d, lessThan (10.0) ); //....
assertThat( d, greaterThanOrEqualTo (5.0) ); //大于或等于
assertThat( d, lessThanOrEqualTo (16.0) );//...

assertThat( map, hasEntry( "key", "value" ) ); //map里是否有key对应value
assertThat( iterable, hasItem ( "abc" ) ); //list里是否包含某元素
assertThat( map, hasKey ( "abc" ) ); //...
assertThat( map, hasValue ( "abc" ) ); //...
```

###注解  (after JUnit4)
1.@test:
在Test Case里的某个方法前加这个注解，表示这是一个测试方法
a)(expected=XXException.class)   //解决该方法可能会跑出的异常，如
```
@Test(expected=java.lang.ArithmeticException.class)
```
b)(timeout=xxx) 规定该方法在xxx毫秒内结束，否则kill掉，主要是为了测试某个方法的执行效率，可与expexted连用
```
@Test(expected=java.lang.ArithmeticException.class,timeout=1000)
```
2.@Ignore: 
被忽略的测试方法，实际意义是开发中可能有些方法还没有写好，这时候就可以忽略它去测试其他的模块

3.@Before: 
每一个测试方法之前运行

4.@After: 
每一个测试方法之后运行

5.@BeforeClass: 
所有测试开始之前运行，在类对象初始化前开始执行，标记的方法必须是static,实际意义是，如果测试需要搭建环境（建立数据库链接）或者配置文件时，需要用来搞好前提条件

6.@AfterClass: 
所有测试结束之后运行,在类对象初始化前开始执行，标记的方法必须是static，实际意义是用来关闭资源和卸载环境

###批量测试
点击要测试的包，右键Run As Configurations，在左侧工作区点击该包下任意一个文件，在右键工作区点选“Rub all tests in the selected project package or source folder”

###注意
1.遵守约定，比如：
a)类放在test包中
b)类名用XXXTest结尾
c)方法用testMethod命名 

###JUnit高级篇
.....
....
....


