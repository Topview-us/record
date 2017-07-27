- 建立本地仓库：修改conf/setting.xml的localRepository为自己建立的仓库地址,并且复制setting.xml到自己新建的repository同级目录下
- 中央工厂：maven安装目录下lib的maven-model-builder-3.5.0.jar里面找到pom-4.0.0.xml,里面<url>https://repo.maven.apache.org/maven2</url>是中央工厂
- mvnrepository.com,可以在里面找到gav(groupId,artifactId,versionId)
- 架构maven项目：mvn archettpe:generate

## 常用指令
- mvn -version/-v  显示版本信息  
- mvn archetype:generate 创建mvn项目   
- mvn archetype:create -DgroupId=com.oreilly -DartifactId=my-app   创建mvn项目  
- mvn package            生成target目录，编译、测试代码，生成测试报告，生成jar/war文件  
- mvn compile 编译   
- mvn test 编译并测试 
- mvn clean 清空生成的文件 
- mvn site 生成项目相关信息的网站

## 基本目录结构
```
|-- pom.xml
|-- src
|   |-- main
|   |   `-- java
|   |   `-- resources
|   |   `-- filters
|   `-- test
|   |   `-- java
|   |   `-- resources
|   |   `-- filters
|   `-- it
|   `-- assembly
|   `-- site
`-- LICENSE.txt
`-- NOTICE.txt
`-- README.txt
```

    src/main/java 项目的源代码所在的目录
    src/main/resources 项目的资源文件所在的目录
    src/main/filters 项目的资源过滤文件所在的目录
    src/main/webapp 如果是web项目，则该目录是web应用源代码所在的目录，比如html文件和web.xml等
    src/test/java 测试代码所在的目录
    src/test/resources 测试相关的资源文件所在的目录
    src/test/filters 测试相关的资源过滤文件所在的目录
不常用：

    src/it 集成测试代码所在的目录，主要是供别的插件使用的。
    src/assembly 组件（Assembly）描述符所在的目录
    src/site 站点文件
    LICENSE.txt 项目的许可文件
    NOTICE.txt 该项目依赖的库的注意事项
    README.txt 项目的readme文件

## pom.xml
### maven协作的相关属性
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
                      http://maven.apache.org/maven-v4_0_0.xsd">  
  <modelVersion>4.0.0</modelVersion>  
  <groupId>org.codehaus.mojo</groupId>  
  <artifactId>my-project</artifactId>  
  <version>1.0</version>  
  <packaging>war</packaging>  
</project>
```
- groupId : 组织标识，例如：org.codehaus.mojo，在M2_REPO目录下，将是: org/codehaus/mojo目录。
- artifactId : 项目名称，例如：my-project，在M2_REPO目录下，将是：org/codehaus/mojo/my-project目录。
- version : 版本号，例如：1.0，在M2_REPO目录下，将是：org/codehaus/mojo/my-project/1.0目录。
- packaging : 打包的格式，可以为：pom , jar , maven-plugin , ejb , war , ear , rar , pa
### 依赖配置：
依赖会被继承
```xml
    <dependencies>  
       <dependency>  
         <groupId>junit</groupId>  
         <artifactId>junit</artifactId>  
         <version>4.0</version>  
         <scope>test</scope>  
       </dependency>  
       …  
     </dependencies>
```
### 继承
在子pom.xml中配置
```
<parent>  
    <groupId>xxx</groupId>  
    <artifactId>xxx</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</parent> 
```
### 聚合
在总pom.xml中配置
```xml
  <modules>
    <module>demo001</module>
    <module>demo002</module>
    <module>demo003</module>
  </modules>
```
- 依赖管理  
  dependencyManagement的特性：在dependencyManagement中配置的元素既不会给parent引入依赖，也不会给它的子模块引入依赖，仅仅是它的配置是可继承的
    - 所以在子文件只需要表明groupId和artifactId就可以，详细信息在父文件

## 依赖特性
- 依赖范围\<scope>  
  pom文件的\<dependency>\</dependency>标签对可以设置一个\<scope>\</scope>标签对，值如下
    - compile(默认)：在编译、测试、打包的时候都把依赖加进去
    - test：仅测试范围内依赖有效
    - privided：编译、测试的时候会把依赖加进去，但是打包的时候就不会把依赖加进去，例如servlet-api
    - runtime：运行时依赖范围，比如mysql的jar
    - system：provided依赖范围完全一致。但是，使用system范围依赖时必须通过systemPath元素显式地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植，因此应该谨慎使用

- 依赖传递，test的范围的不会依赖于之前依赖的文件

- 依赖传递优先级
    - 同级依赖，先被声明的被依赖
    - 不同级依赖，依赖于更直接的依赖
    - 排除依赖，  
---
        <exclusions>  
            <exclusion>  
                groupId  
                artifactId  
            </exclusion>  
        </exclusions>

> 冲突（还没搞清楚原因）：依赖net.sf.json这个包的时候，json-b直接依赖于commons-lang的2.5版本，然后通过直接依赖于ezmorph来间接依赖commons-lang的2.3版本会有冲突，不加以解决compile能通过，但是将2.3修改为2.5冲突会消失。此外依赖这个包的时候要父子项目同时指定<classfied>参数为jdk15才能正常compile

## 生命周期
Maven的有三套生命周期：clean、default、site。++生命周期的各个阶段由插件控制。++
- clean: pre-clean, clean, post-clean
- default: 
```
validate
initialize
generate-sources
process-sources
generate-resources
process-resources：复制和处理资源文件到target目录，准备打包；
compile：编译项目的源代码；
process-classes
generate-test-sources
process-test-sources
generate-test-resources
process-test-resources
test-compile：编译测试源代码；
process-test-classes
test：运行测试代码；
prepare-package
package：打包成jar或者war或者其他格式的分发包；
pre-integration-test
integration-test
post-integration-test
verify
install：将打好的包安装到本地仓库，供其他项目使用；
deploy：将打好的包安装到远程仓库，供其他项目使用；

```
- site  
    pre-site
    site：生成项目的站点文档；  
    post-site  
    site-deploy：发布生成的站点文档
