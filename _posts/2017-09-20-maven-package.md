---
layout: post
title: "maven打包"
categories: java maven
# author: "炮灰哥"
---

### 前言

[上篇文章](/java/spring/https/2017/09/15/12306-java-https.html 'java导入cer证书')提到要做一个12306刷票应用，获得票之后短信提醒，这里短信是一个单独的应用，使用了Thrift框架，短信平台使用的是阿里大于。这篇主要写在该项目中，maven打包踩到的坑，其它内容就不展开了。

项目目录如下图

![project](/images/maven-package-01.png)

在maven项目中，我们的项目依赖都是取自本地或远程仓库，比如如下配置

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.3</version>
</dependency>
```

添加所需要的配置后，IDEA可以使用，但是发现不能用java -jar执行，会报classNotFound。

我们需要使用maven-assembly-plugin插件，在用_maven package_时，将依赖也一起放入jar包中，pom添加如下代码

```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <!-- 指示main方法所在的类 -->
                <mainClass>cn.wazitang.sms.MainServer</mainClass>
            </manifest>
        </archive>
    </configuration>

    <executions>
        <execution>
            <id>make-assembly</id>
            <!-- 在使用package命令时执行该插件 -->
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

这样，可以解决大部分的问题。
但是呢，项目依赖的jar包可能不属于任何仓库，比如使用的本地的jar文件，当然了，maven肯定可以结局这个问题的

```xml
<dependency>
    <groupId>local.lib.app-sms</groupId>
    <artifactId>taobao-sdk</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/taobao-sdk-java-auto-1.0.jar</systemPath>
</dependency>
```

使用上面配置后，用IDEA是可以正常运行项目了，但是用_maven package_打包，ClassNotFound又出现了，这样的话，不同的环境运行都要处理，不符合程序猿风格。

这里的解决方式是用maven插件，把本地jar安装到本地仓库，然后正常的依赖就可以了。内容如下

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-install-plugin</artifactId>
    <executions>
        <execution>
            <id>install-external</id>
            <phase>clean</phase>
            <configuration>
                <file>${basedir}/lib/taobao-sdk-java-auto-1.0.jar</file>
                <repositoryLayout>default</repositoryLayout>
                <groupId>local.lib.app-sms</groupId>
                <artifactId>taobao-sdk</artifactId>
                <version>1.0</version>
                <packaging>jar</packaging>
                <generatePom>true</generatePom>
            </configuration>
            <goals>
                <goal>install-file</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

这样，我们在执行clean命令的时候，就会把jar包放到本地仓库。在不同的环境，只要先clean，再package就可以了。

将jar解压出来如下图

![jar](/images/maven-package-02.png)

### 其它

- maven内容属性

|属性                                 |描述                                 |
|-------------------------------------|------------------------------------|
|${basedir}                           |项目根目录                           | 
|${version}                           |表示项目版本                          |
|${project.basedir}                   |同${basedir}                         |
|${project.version}                   |表示项目版本,与${version}相同         |
|${project.build.directory}           |构建目录，缺省为target               |
|${project.build.sourceEncoding}      |表示主源码的编码格式                  |
|${project.build.sourceDirectory}     |表示主源码路径                        |
|${project.build.finalName}           |表示输出文件名称                      |
|${project.build.outputDirectory}     |构建过程输出目录，缺省为target/classes |

- 依赖项的作用域

    - compile

        缺省值，适用于所有阶段，会随着项目一起发布。

    - runtime

        表示该依赖项只有在运行时才是需要的，在编译的时候不需要。这种类型的依赖项将在运行和test的类路径下可以访问。

    - test

        表示该依赖项只对测试时有用，包括测试代码的编译和运行，对于正常的项目运行是没有影响的。不会随项目分布。

    - provided

        表示该依赖项将由JDK或者运行容器在运行时提供，也就是说由Maven提供的该依赖项我们只有在编译和测试时才会用到，而在运行时将由JDK或者运行容器提供。如Servlet.jar

    - system

        当scope为system时，表示该依赖项是我们自己提供的，不需要Maven到仓库里面去找。指定scope为system需要与另一个属性元素systemPath一起使用，它表示该依赖项在当前系统的位置，使用的是绝对路径。

### 踩坑过程中参考的一些文章

- [关于maven打包外部依赖的问题](http://blog.csdn.net/demo_zj/article/details/47344505)