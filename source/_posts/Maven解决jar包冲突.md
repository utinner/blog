---
title: Maven解决jar包冲突
date: 2020.07.13 17:31:28
tag: Maven
categories: Java
---
<meta name="referrer" content="no-referrer" />

### 一、Maven中jar包冲突产生原因
MAVEN项目运行中如果报如下错误：
- Caused by:java.lang.NoSuchMethodError
- Caused by: java.lang.ClassNotFoundException
十有八九是Maven jar包冲突造成的。那么jar包冲突是如何产生的？
首先我们需要了解jar包依赖的传递性。
##### 1、依赖传递
当我们需要A的依赖的时候，就会在pom.xml中引入A的jar包；而引入的A的jar包中可能又依赖B的jar包，这样Maven在解析pom.xml的时候，会依次将A、B 的jar包全部都引入进来。

<!-- more -->

举个例子：
在Spring Boot应用中导入Hystrix和原生Guava的jar包：
```
        <!--原生Guava API-->
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>20.0</version>
        </dependency>

        <!--hystrix依赖（包含对Guava的依赖）-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>1.4.4.RELEASE</version>
        </dependency>
```
利用`Maven Helper`插件得到项目导入的jar包依赖树：
![Maven helper](https://upload-images.jianshu.io/upload_images/15200008-d6cc2115ee502001.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从图中可以看出Hystrix包含对Guava jar包依赖的引用： Hystrix -> Guava，所以在引入Hystrix的依赖的时候，会将Guava的依赖也引入进来。
##### 2.jar包冲突原理
假设有如下依赖关系：

- A->B->C->D1(log 15.0)：A中包含对B的依赖，B中包含对C的依赖，C中包含对D1的依赖，假设是D1是日志jar包，version为15.0

- E->F->D2(log 16.0)：E中包含对F的依赖，F包含对D2的依赖，假设是D2是同一个日志jar包，version为16.0

当pom.xml文件中引入A、E两个依赖后，根据Maven传递依赖的原则，D1、D2都会被引入，而D1、D2是同一个依赖D的不同版本。
当我们在调用D2中的method1()方法，而D1中是15.0版本（method1可能是D升级后增加的方法），可能没有这个方法，这样JVM在加载A中D1依赖的时候，找不到method1方法，就会报NoSuchMethodError的错误，此时就产生了jar包冲突。

注：
> 如果在调用method2()方法的时候，D1、D2都含有这个方法（且升级的版本D2没有改动这个方法，这样即使D有多个版本，也不会产生版本冲突的问题。）

举个例子：

利用Maven Helper插件分析得出：Guava这个依赖包产生冲突。
我们之前导入了Guava的原生jar包，版本号是20.0；而现在提示Guava产生冲突，且冲突发生位置是Hystrix所在的jar包，所以可以猜测Hystrix中包含了对Guava不同版本的jar包的引用。

为了验证我们的猜想，使用Maven Helper插件找到Hystrix依赖的jar：
![image.png](https://upload-images.jianshu.io/upload_images/15200008-5877bb2f0f136f64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![hystrix-javanica依赖](https://upload-images.jianshu.io/upload_images/15200008-a05b5161000a3e65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到：Hystrix jar中所依赖的Guava jar包是15.0版本的，而我们之前在pom.xml中引入的原生Guava jar包是20.0版本的，这样Guava就有15.0 与20.0这两个版本，因此发生了jar包冲突。
### 二、 Maven中jar包冲突的解决方案
Maven 解析 pom.xml 文件时，同一个 jar 包只会保留一个，那么面对多个版本的jar包，需要怎么解决呢？


##### 1、 Maven默认处理策略
**最短路径优先**

Maven 面对 D1 和 D2 时，会默认选择最短路径的那个 jar 包，即 D2。E->F->D2 比 A->B->C->D1 路径短 1。

**最先声明优先**

如果路径一样的话，如： A->B->C1, E->F->C2 ，两个依赖路径长度都是 2，那么就选择最先声明。

##### 2、移除依赖：用于排除某项依赖的依赖jar包
（1）我们可以借助Maven Helper插件中的Dependency Analyzer分析冲突的jar包，然后在对应标红版本的jar包上面点击execlude，就可以将该jar包排除出去。

再刷新以后冲突就会消失。

（2）手动排除
或者手动在pom.xml中使用<exclusion>标签去排除冲突的jar包（上面利用插件Maven Helper中的execlude方法其实等同于该方法）：
```
        <!--hystrix依赖（包含对Guava的依赖）-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>1.4.4.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```
##### 3 版本锁定原则：一般用在继承项目的父项目中
正常项目都是多模块的项目，如moduleA和moduleB共同依赖X这个依赖的话，那么可以将X抽取出来，同时设置其版本号，这样X依赖在升级的时候，不需要分别对moduleA和moduleB模块中的依赖X进行升级，避免太多地方（moduleC、moduleD….）引用X依赖的时候忘记升级造成jar包冲突，这也是实际项目开发中比较常见的方法。

首先定义一个父pom.xml，将公共依赖放在该pom.xml中进行声明：

```
   <properties>
        <spring.version>spring4.2.4</spring.version>
    <properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${spring.versio}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

这样如moduleA和moduleB在引用Spring-beans jar包的时候，直接使用父pom.xml中定义的公共依赖就可以：
moduleA在其pom.xml使用spring-bean的jar包(不用再定义版本)：

```
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>
    </dependencies>
```
moduleB在其pom.xml使用spring-bean的jar包如上类似：

```
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>
    </dependencies>
```
### 三、相关命令
如果不用`Maven Helper`这个从插件，maven还支持用命令的方式进行分析jar包冲突：
```
mvn dependency:tree
```
以上命令是输出到控制台，还可以输出到文件：
```
mvn dependency:tree>/Users/tinner/Desktop/1.txt
```
查询指定jar包的冲突:
```
 mvn dependency:tree -Dverbose -Dincludes=com.google.guava
```