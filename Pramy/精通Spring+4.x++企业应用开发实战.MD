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