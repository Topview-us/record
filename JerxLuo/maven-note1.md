# maven概览

## pom.xml

### pom.xml的位置
   **应该放在与src同级的目录下**
###   pom.xml的组成
  - project的schema文件用于约束maven的写法
  - 标签
    - `groupId`某公司开发的某个项目
    - `artifactId`该项目某个具体的模块
    - `version`该项目版本
       0.0.1-SNAPSHOT 快照

```xml
<?xml version="1.0" encoding="utf-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemalocation="htttp://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">		<modelVersion>4.0.0</modelVersion>
		<groupId>zttc.itat.maven</groupId>
		<artifactId>maven-ch01</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<dependencies>
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version></version>
			</denpendency>
		</dependencies>
</project>
```
## 命令
在命令执行的时候，maven会根据
- `mvn compile`  编译
  要编译的java文件会要导入一些jar包，比如junit的包，这个时候应该要把junit的denpency标签(关键是里面的gav三个标签)加入pom.xml
  dependency(依赖标签)
- `mvn test` 测试
- `mvn clean`清除target文件夹
  **target文件夹**
  - 存放着测试等命令的结果报告和class文件，以及编译产生的.class文件放在一个class文件夹里面
- `mvn package`打包，打的包放在target里面
  （先编译再测试然后才打包）
- `mvn instal`将这个包install进maven的仓库，然后其他项目要引用这个项目的某个类的时候就根据依赖标签在maven的仓库查找或者在中央工厂
  **仓库**
  maven的默认仓库会放在用户主目录下的.m2文件夹下
  需要学会自定义自己的maven仓库位置。
##使用maven常见步骤：

1. 安装maven插件
2. 设定maven仓库位置
  **在客户端插件项目**
3. 设定骨架相关的程序：工具：`mvn archetype`
  调用工具archetype：`mvn archetype:generate -DgroupId=zttc -Dzttc` 
  过程中要求输入 groupId、artcifactId、version
  `非常重要`**在eclipse创建项目**
  a.maven配置
  b.usersetting 设置成为仓库的setting


仓库------>我的文档中
根据pom.xml去仓库里面或者线上查找jar包
mvn package是打包，mvn install是传到仓库里面
设定工厂到自己的位置
本地工厂默认是~/.m2/repository
先在目录中新建一个文件夹

## 临时小结

1. 安装工厂
2. 找到路径
3. archetype骨架生成工具的`generate`命令
4. IDE 中使用maven设置：每次打开一个workspace,都要设置maven的Installation和usersetting
5. 查看中央仓库的url

## 核心概念

- POM

- Maven插件

- Maven生命周期

- Maven依赖管理

- Maven库

  ### POM

  关键的有：

  - 定义项目的类型
  - 名字
  - 管理依赖关系
  - 定制插件的行为
  ###Maven插件
  archetype插件
  generate是目标（goal）的名字
  `mvn archetype:generate`是告诉maven执行archetype插件的generate目标
  **目标和插件**
  一个目标是一个工作单元，而插件是一个或多个目标的集合
  由此知道：Jar插件包含建立Jar文件的目标
  Compile插件包含编译代码和单元测试代码的目标
  Surefire插件运行单元测试
  mvn不会做太多事，而是把构建任务交给插件做。**插件定义了常用的构建逻辑**，能够被重复使用
###Maven生命周期
process-resources 阶段：resources:resources
compile 阶段：compiler:compile
process-classes 阶段：(默认无目标)
process-test-resources 阶段：resources:testResources
test-compile 阶段：compiler:testCompile
test 阶段：surefire:test
prepare-package 阶段：(默认无目标)
package 阶段：jar:jar
命令mvn package的 package 是一个maven的生命周期阶段 (lifecycle phase )。生命周期指项目的构建过程，它包含了一系列的有序的阶段 (phase)，而一个阶段就是构建过程中的一个步骤。插件目标可以绑定到生命周期阶段上。一个生命周期阶段可以绑定多个插件目标
###Maven依赖管理
  Maven坐标
  依赖关系是在dependencies部分中定义
scope标签：
  scope决定依赖关系的适用范围

- compile（默认）（整个生命周期阶段该依赖都有效）
  **编译和打包**的时候将这个依赖加进去
  编译测试打包都有效

- provided（最后打包阶段该依赖无效）

  **编译和测试**的时候将这个依赖加进去，打包的时候不会
  ​

- runtime（运行时）
  **运行时 **依赖范围有效，编译测试的时候不依赖
  实例：mysql的connector的jar包

- test
  测试范围有效，
  实例：JUnit的jar包

  ### 依赖的传递特性

  - 只会传递compile范围的依赖，而不传递test范围的依赖

  - **所以必须在自己的包里依赖测试范围的jar包而不应该间接依赖**

  - 同级间接依赖两次同一个jar包的不同版本就根据依赖顺序来依赖此jar包

  - 不同级间接依赖按级别最短的版本去依赖该jar

    #### 依赖冲突：

    ​	1.最短级别依赖优先

    ​	2.同级间接依赖下，本包pom.xml里面的依赖顺序在前的优先

  exclusions排除标签

  ```xml
  <exclusions>
  	<groupId>[]</groupId>
    	<artifactId>[]</artifactId>
  </exclusions>
  ```

  比如test，只会在执行compiler:testCompile and surefire:test 目标的时候才会被加到 classpath 中，在执行 compiler:compile 目标时是拿不到 junit 的。

###Maven库
本地库：指maven下载了插件或者jar文件放在本地机器的copy
`mvn install`可以将自己的项目安装到本地库

### 中央仓库镜像

https://skyao.gitbooks.io/learning-maven/content/installation/mirror.html

原来的中央仓库访问问题较多，可以改用阿里云的镜像