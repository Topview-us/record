# JUnit4
## 注解
1. @Test : 测试方法，测试程序会运行的方法，后边可以跟参数代表不同的测试，如(expected=XXException.class) 异常测试，(timeout=xxx)超时测试
2. @Ignore : 被忽略的测试方法
3. @Before: 每一个测试方法之前运行
4. @After : 每一个测试方法之后运行
5. @BeforeClass: 所有测试开始之前运行
6. @AfterClass: 所有测试结束之后运行

## 测试流程
1. 包含必要地Package  
    - 最主要地一个Package就是org.junit.\*
    - 还有一个也非常地重要import static org.junit.Assert.* 
>我们在测试的时候使用的一系列assertEquals方法就来自这个包，这是一个静态包含（static），是JDK5中新增添的一个功能。也就是说，assertEquals是Assert类中的一系列的静态方法，一般的使用方式是Assert. assertEquals()，但是使用了静态包含后，前面的类名就可以省略了，使用起来更加的方便。

2. 测试类声明：任意
3. 创建一个待测试的对象  
    为了测试某个类，我们必须创建一个该类对象
```java
private static Calculator calculator = new Calculator();  
```
4. 测试方法的声明:注解开头
## Fixture：在某些阶段必须调用的代码
- 对被测试的类对象进行一个“复原”操作，以消除其他测试造成的影响
```java
      @Before
      public void setUp() throws Exception ...{
           calculator.clear();
      }
```
- @BeforeClass 和 @AfterClass在整个测试过程只执行一次，这些方法必须是public和static的

## 设置Runner（运行器）
在JUnit中有很多个Runner，他们负责调用测试代码，每一个Runner都有各自的特殊功能，要根据需要选择不同的Runner来运行你的测试代码
- 如果没有指定，那么系统自动使用默认TestClassRunner来运行代码

## 参数化测试：一次测试多组数据
```java
@RunWith(Parameterized.class)
public class SquareTest {
private static Calculator calculator = new Calculator();
private int param;
private int result;    

@Parameters   
public static Collection data() {
        return Arrays.asList(new Object[][]{
                {2, 4},
                {0, 0},
                {－3, 9},
        });
}
//构造函数，对变量进行初始化
public SquareTest(int param,int result) {
        this.param = param;
            this.result = result;
}

@Test   
public void square() {
        calculator.square(param);
        assertEquals(result, calculator.getResult());
    }
```
1. 为这种测试专门生成一个新的类，而不能与其他测试共用同一个类
2. 为这个类指定一个Runner(如ParameterizedRunner)
3. 定义一个待测试的类，并且定义两个变量，一个用于存放参数，一个用于存放期待的结果
4. 定义测试数据的集合，也就是上述的data()方法，该方法可以任意命名，但是必须使用@Parameters标注进行修饰
5. 构造函数，其功能就是对先前定义的两个参数进行初始化

## 打包测试
JUnit为我们提供了打包测试的功能，将所有需要运行的测试类集中起来，一次性的运行完毕
```java
import org.junit.runner.RunWith;
import org.junit.runners.Suite;

@RunWith(Suite.class)
@Suite.SuiteClasses(...{CalculatorTest.class, SquareTest.class})
public class AllCalculatorTests ...{}
```
- 这个功能也需要使用一个特殊的Runner，因此我们需要向@RunWith标注传递一个参数Suite.class
- 还需要另外一个标注@Suite.SuiteClasses，来表明这个类是一个打包测试类。
>我们把需要打包的类作为参数传递给该标注就可以了。有了这两个标注,下面的类已经无关紧要，随便起一个类名，内容全部为空既可。

## 补充
- 限时测试：常用于死循环
    @Test(timeout = 1000)
- 测试异常：检测某函数在特定情况下是否抛出正确的异常
```java
  @Test(expected = ArithmeticException.class)
  public void divideByZero() ...{
        calculator.divide(0);
  }
```

