# maven（自动化构建模式）

Tags : maven markdown

## 第一讲

### 1、maven

maven最大的好处就是不需要导入jar包，而是将jar包的gav，也即坐标写入pom.xml中，gav为dependency中的内容

各类jar包等资料都存储在仓库中（默认为C:\Users\wxs\.m2，可以改动，下一讲说明），当使用到仓库没有的jar包时，maven会自动下载相关的jar包并完成编译。[中央仓库](http://mvnrepository.com/)中含有各类jar包的坐标，可以在上面查找。



### 2、在cmd中的各类命令（进入项目目录）：

mvn compile：自动完成编译操作，将文件自动放到target的class文件中（main文件夹内的java文件）

mvn test：自动完成测试（test文件夹内的java文件），target文件内生成三个测试报告

mvn clean：将target文件删除

mvn package：编译测试并将项目打成jar包

mvn install：将jar包发到本地仓库中



## 第二讲

### 1、安装本地工厂

将D:\apache-maven-3.2.1-bin\apache-maven-3.2.1\conf中的setting.xml复制到某个文件夹中，在setting.xml中插入所改工厂的地址

（如图：  <localRepository>d:/java/maven/repository</localRepository>将setting.xml放入maven文件夹中，repository文件夹作为仓库）

![](http://ot0aou666.bkt.clouddn.com/%E8%AE%BE%E7%BD%AE%E6%9C%AC%E5%9C%B0%E5%B7%A5%E5%8E%82.png)

2、然后将conf中的setting.xml也做同样的修改



### 2、查找中央工厂（查找依赖）的路径

在D:\apache-maven-3.2.1-bin\apache-maven-3.2.1\lib中名为maven-model-builder-3.2.1的jar包，解压中在maven-model-builder-3.2.1.jar\org\apache\maven\model文件夹里有一个pom-4.0.0.xml，xml内含有网址http://repo.maven.apache.org/，此网址为中央工厂的路径。



另：[中央工厂](http://search.maven.org/)



### 3、在客户端（cmd）生成maven项目

archetpye骨架生成器

cmd中输入mvn archetype,然后输入mvn archetype:generate或mvn archetype:generate -DgroupId=test.maven -DartifactId=maven-ch03 -Dversion=0.0.1-SNAPSHOT（直接输入id名称等）创建项目

### 4、eclipse创建maven项目

创建maven项目时常用maven-archetype-quickstart和maven-archetype-webapp

groupId是自己部门网址倒过来 + 项目，项目名

ArtifactId是项目模块名

版本：x.x.x-里程碑 SNAPSHOT，alpha(内部测试版本)，beta（使用人员下载使用），release（RC,释放版本），GA（可靠版本）

在项目中建立src/main/resources放置资源的配置文件，建立src/main/resources放置测试的配置文件

每打开一个workspace就要对Preferences进行配置：installations、User settings（maven内的setting.xml文件）



## 第三讲

### 1、eclipse中的maven项目

在main文件夹写完每个包后要在test文件夹中测试

不同模块（即不同maven项目）要在其他项目中使用需要先进行clean package打包，然后clean install将包打进仓库中。

项目中pom对包的配置可使用隐式变量



### 2、隐式变量

*{basedir} 项目根目录

*${project.build.directory} 构建目录，缺省为target

*${project.build.outputDirectory} 构建过程输出目录，缺省为target/classes

*${project.build.finalName} 产出物名称，缺省为${project.artifactId}-${project.version}

*${project.packaging} 打包类型，缺省为jar

*${project.xxx} 当前pom文件的任意节点的内容如：

${project.groupId} 或

${project.artifactId}



### 3、某些包

dbuint隔离数据库原有的数据做数据库测试，测试时用dbuint的Connection，而不用main文件中的Connection



## 第四讲、maven的依赖、传递特性（标记图）



### 1、依赖的范围

<version></version>下面的<scope></scope>

<scope>，它主要管理依赖的部署(依赖范围)。

- compile（默认），编译、测试、打包时加入依赖。缺省值，适用于所有阶段，会随着项目一起发布。

- provided，编译测试加入依赖，打包时不会加入依赖（防止打包发生冲突）。类似compile，期望jdk、容器或使用者会提供这个依赖。如[servlet开发](http://www.makaidong.com/servlet%E5%BC%80%E5%8F%91/index.shtml) .jar。

- runtime，编译时不依赖，运行时依赖。只在运行时使用，如jdbc驱动，适用运行和测试阶段。

- test，只在测试时依赖，只在测试时使用，用于编译和运行测试代码。不会随项目发布，某些类基于测试时用放在test文件夹中。

- system，类似provided，需要显式提供包含依赖的jar，maven不会在repository中查找它

  ​

### 2、依赖的特性

（1）、依赖传递的包是依赖范围为compile的包，而test的包不会随某个maven项目打的包传递

（2）、直接依赖与间接依赖：

**依赖级别相同时（依赖于 —>）**：

假设user-core—>log4j 1.2.17

user-log—>log4j 1.2.9

user-service—>core、log，且pom部署为以下

```
<dependency>	
	<artifactId>user-log</artifactId>
</dependency>
<dependency>	
	<artifactId>user-core</artifactId>
</dependency>
```

则user-service依赖于log4j 1.2.9，即依赖于先部署的



**依赖级别不相同时**：

假设有

​            依赖关系1：dbunit —>common logging 1.1.1，core—>dbunit，

user service—>core

​	    依赖关系2：user log—>commons logging 1.0.4，

user service—>user log

那么，user service将依赖于commons logging 1.0.4



**结论**：当依赖级别相同时，最终模块依赖于先部署的；当依赖级别不相同时，	

​	    最终模块依赖于级别低的关系



### 3、排除依赖

在每个包的<version></version>部署如下，可排除依赖

```
<exclusion>
	<groupId></groupId>
	<artifactId></artifactId>
</exclusion>
```



## 第五讲 maven项目的聚合和继承



创建一个package为pom的maven项目

### 1、聚合

在新创建的maven项目中的pom.xml内部署<modules></modeules>

如：

```
<modules>
  <module>../test.maven.second</module>
  <module>../test.maven.third</module>
</modules>
```



### 2、继承

先创建parent模块（package为pom）

在每个模块的pom.xml内部署<parent></parent>

如：

```
<parent>
  		<groupId>com.topview.user</groupId>
  		<artifactId>test.maven.parent</artifactId>
  		<version>0.0.1-SNAPSHOT</version>
  		<relativePath>../test.maven.parent/pom.xml</relativePath>
</parent>
```



还可将依赖继承，即在parent模块的pom.xml中部署<dependencyManagement></dependencyManagement>

	<dependencyManagement>
	  <dependencies>
	  	<dependency>
	     <groupId>junit</groupId>
	     <artifactId>junit</artifactId>
	     <version>3.8.1</version>
	     <scope>test</scope>
	    </dependency>
	  <dependencies>
	<dependencyManagement>

在parent模块的pom.xml中聚合，部署<modules></modeules>



## 第六讲、插件与生命周期

### 1、生命周期

生命周期分为三套：clean、default、site，每个生命周期都有不同的阶段，阶段如下：

clean

​	pre-clean

​	clean

​	post-clean

default

​	validate  

​	initialize  

​	generate-sources  

​	process-sources:处理项目主资源、一般是将src/main/resources目录的内容进行变量替换工作后、复制到项目输出的主classpath目录中。  

​	generate-resources  

​	process-resources  

​	compile编译项目的主源码、一般是编译src/main/java目录下的Java文件至项目输出的主classpath目录中。  

​	process-classes  

​	generate-test-sources  

​	process-test-sources  

​	generate-test-resources  

​	process-test-resources  

​	test-compile:编译项目的测试源码、一般是编译src/test/java目录下的Java文件至项目输出的测试classpath目录中。  

​	process-test-classes  

​	test:使用单元测试框架运行测试、测试代码不会被打包或部署  

​	prepare-package  

​	package  

​	pre-intergration-test  

​	integration-test  

​	post-intergration-test  

​	verify  

​	install:将包安装到Maven本地仓库。  

​	deploy:将包复制到远程仓库  

site

​	pre-site  

​	site:生成项目站点文档  

​	post-site  

​	site-deploy：将生成的项目站点发布到服务器上  



### 2、插件

目标：目标是一个明确的任务，目标的例子包括Compiler插件中的compile目标，它用来编译项目中的所有源文件，或者Surefire插件中的test目标，用来运行单元测试



插件在pom.xml部署如下：

```
<build>
	<plugins>
		<plugin>//插件
			<groupId>some groupId</groupId>
			<artifactId>some artifactId</artifactId>
			<version>some version</version>
			<configuration></configuration>//某些插件所含的配置
			<executions>
				<execution>
					<phase></phase>//捆绑生命周期（在哪个生命周期后执行目标）
					<goals>
						<goal>some goal</goal>
					</goals>（目标）
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

注：可以与依赖一样有相同的聚合继承模式，需要的插件在[maven,apache的网站上找](https://maven.apache.org/plugins/maven-compiler-plugin/)



插件的部署还有一些另外的配置，在`<version></version>`后部署`<configuration></configuration>`

例如：surefile插件中有配置`<include></include>`等



### 3、测试

运行test文件夹的类时，默认只能将类名为Test******、    ******Test、**TestCase的文件进行运行，可由上述提到的surefile插件中的配置修改



## 第七、maven项目的分模块模式

### 1、依赖关系：

app-dao      --> app-util

app-service --> app-dao

app-web     --> app-service

### 2、模块化的目的

方便重用，如果你有一个新的swing项目需要用到app-dao和app-service，添加对它们的依赖即可，你不再需要去依赖一个WAR。而有些模块，如app-util，完全可以渐渐进化成公司的一份基础工具类库，供所有项目使用。这是模块化最重要的一个目的。

由于你现在划分了模块，每个模块的配置都在各自的pom.xml里，不用再到一个混乱的纷繁复杂的总的POM中寻找自己的配置。

如果你只是在app-dao上工作，你不再需要build整个项目，只要在app-dao目录运行mvn命令进行build即可，这样可以节省时间，尤其是当项目越来越复杂，build越来越耗时后。

某些模块，如app-util被所有人依赖，但你不想给所有人修改，现在你完全可以从这个项目结构出来，做成另外一个项目，svn只给特定的人访问，但仍提供jar给别人使用。

多模块的Maven项目结构支持一些Maven的更有趣的特性（如DepencencyManagement），这留作以后讨论。



## 第八、setting.xml内的一些设置

### 1、设置镜像（阿里云镜像）：

![](http://ot0aou666.bkt.clouddn.com/%E8%AE%BE%E7%BD%AE%E9%95%9C%E5%83%8F.png)

如图，在eclipse认定的maven路径下的setting.xml的<mirrors></mirrors>中设置

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
 </mirror>
```



### 2、将maven项目默认为jdk1.7

![](http://ot0aou666.bkt.clouddn.com/maven%E9%A1%B9%E7%9B%AE%E9%BB%98%E8%AE%A4%E4%B8%BAjdk1.7.png)

原先maven项目默认为javaSE1.5，现在需要在eclipse认定的maven路径下的setting.xml中的<profiles></profiles>中设置

```xml
<profile>
  <id>jdk-1.7</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.7</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
  </properties>
</profile>
```



**注：设置好后记得在eclipse中windows->preferences->maven->usersetting中update setting**

