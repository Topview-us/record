# maven的聚合

模块同级添加pom.xml

eclipse里面建立一个simple的maven project 

packaging选择pom（管理类）

```xml
<!--导入了三个模块，模块相对此pom文件的位置，-->
<packaging>pom</packaging>
<modules>
	<module>../user-core</module></module>
  	<module>../user-log</module></module>
  	<module>模块名3</module>
</modules>
```

module标签

# maven的聚合和继承

（一）创建多模块工程
user-core和user-service继承user-parent模块

(继承操作：在子类加入parent标签)

子类上要加入

```xml
<parent>
	<groupId>zttc.itat.user</groupId>
	<artifactId>user-parent</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  	<relativePath>../user-parent/pom.xml</relativePath>
  <!--相对路径：相对模块去查找父类pom文件，是从此模块出发查找父类pom-->
</parent>
```

（聚合操作：在父类加入modules标签、packaging要为pom）

父类

```xml
<!--子类就不会将父类所有的依赖继承，而是在management里面检索需要的依赖的版本号，再依赖进去
	子类只需要写groupId和artifactId-->
<packaging>pom</packaging>
<modules>
	<module>user-core</module>
  	<module>user-service</module>
</modules>
```

`非常重要`子模块和父模块的关系：子模块对父模块存在依赖关系，即**子模块依赖父模块**

（二）父模块的特性

（1）作用：配置共享

```xml

<dependencyManagement>
  <dependencies>
	<dependency>
    	<groupId>junit</groupId>
      	<artifactId>junit</artifactId>
      	<version>4.10</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

a.子模块里面只需添加dependency的ga标签，不需添加version标签

b.Maven会根据子模块所需要的依赖的ga标签，然后在父模块里面的查找对应的version然后在加进去

（2）父模块install的时候执行顺序：Maven会根据模块之间的依赖顺序来install，举个例子：service依赖了core（也就是在service的pom.xml中加入dependency标签），那install的顺序是:user-parent------>user-core------>user-service

## 复习

pom.xml

src

  main

​	java

​		package

​	resources

  test

​	java	

​		package

​	resources
target

​	bin




1.命令

`mvn compile`将src里面进行编译,编译到target里面的classes

`mvn test`将test里面的进行编译测试,编译到target里面的test-classes

`mvn clean`将target清空

`mvn install`将编译好的东西安装到本地仓库当中

`mvn package`将文件进行打包

2.pom.xml

​	groupId----->项目公司

​	artifactId----->

​	version----->0.0.1-SNAPSHOT

​	0.1.2-里程碑

​	0：大版本

​	1：分支

​	2：多少次更新

​	里程碑：SNAPSHAOT(开发中版本),	alpha(内测),beta(外测),Release(RC),GA(可靠版本)

cms0.0.1-SNAPSHOT ---->cms0.0.1-Release----->cms1.0.1-SNAPSHOT

​						---->cms0.1.1-SNAPSHOT     	-----cms1.0.1-RELEASE

​						---->cms0.1.1-Release

​	3.依赖

```xml
	<dependencies>
      	
    </dependencies>
```

​	4.nexus (sonatype)

​	5.执行`clean package` 会经过整个生命周期

​	6.三套生命周期(独立)

​			clean

​				pre-clean
​				clean
​				post-clean

​			compile
​				validate（验证）
​				generate-sources（）
​				process-sources
​				**compile**(执行compile是到这一步)
​				process-classes
​				generate-test-sources
​				process-test-sources
​				generate-test-resources
​				process-test-resources
​				test-compile
​				process-test-classes
​				**test**（执行test是到这一步）
​				prepare-package
​				**package**(执行package是到这一步)
​				pre-integration-test
​				integration-test
​				post-integration-test
​				verify
​				**install**(安装至本地仓库是到这一步)
​				deploy

​			site
​				pre-site
​				site
​				post-site

​		7.插件（plugins）
单个插件可以捆绑多个目标，`插件:目标`，目标是插件的一个操作单位，在plugin开发中，一个goal通常对应一个方法。
**一个goal可以绑定到一个生命周期阶段**：Jar plugin的jar这个goal默认绑定在package。

build标签类似于dependencymanagement（里面的插件不一定都要依赖）

```	xml
<build>
	<plugins>
      <plugin>
      	<groupId></groupId>
        <artifactId></artifactId>
        <version></version>
		<execution>
        	<phase>compile</phase>
          	<goals>
          		<goal>jar</goal>
              	<goal>test-jar</goal>
          </goals>
        </execution>
      </plugin>
  </plugins>
</build>
```

phase表示该插件的这两个目标所绑定的生命周期的compile阶段

#### 插件surefire

只会运行Test** 或者 ** Test 又或者  ** TestCase
该插件有test目标，查看这个的各种配置，典型有：excludes、includes、skip

cobertura

### 插件的小结

1. gav  : 插件的gav标签，应该在maven的plugins里面查找Example
2. configuration  : 插件的配置设置，可以点进插件绑定的目标，查看配置的标签
3. excutions:插件的执行标签，里面可以设置绑定的周期，还设置使用该插件特定的目标
4. id

