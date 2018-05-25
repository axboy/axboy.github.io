---
layout: post
title: Hello dubbo
description: dubbo学习，第一篇，hello world
categories: java
tags: java
---

根据业务需求，最近研究一下负载均衡和不宕机更新服务(其实并发量不大，负载均衡是可以不需要的)，第一版已经完成，通过nginx + docker 实现，java程序启动两份，更新时一个一个处理。

之前未接触过duboo，spring cloud 有了解，也已经忘了，这次打算把dubbo和spring cloud学习一下，还是要有笔记啊，一段时间不用，就忘了。

闲话不多说，开始正题，程序猿入门，都是从hello world开始的，这里大部分代码都是摘抄自官方说明文档。

### 工具

- Intellij idea 社区版
- maven 3.5.0
- jdk 8

### 开始

- 创建空maven工程

新建maven工程，加入相关依赖，未全部使用，本人直接从[apache/dubbo-demo](https://github.com/apache/incubator-dubbo)复制，父项目只用于控制依赖版本。

定义的dubbo相关的依赖版本为2.6.1

#### 定义api服务接口

创建api子模块，新建如下接口

```java
public interface IDemoService {

    String sayHello(String name);
}
```

#### 服务提供方

创建provider子模块

- 添加maven依赖

```xml
<dependency>
    <groupId>cn.axboy.demo.dubbo</groupId>
    <artifactId>api</artifactId>
    <version>${parent.version}</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo-config-spring</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo-registry-zookeeper</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo-registry-multicast</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo-rpc-dubbo</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo-remoting-netty</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo-serialization-hessian2</artifactId>
</dependency>
```

- 实现上述接口

```java
public class DemoServiceImpl implements IDemoService {

    public String sayHello(String name) {
        System.out.println("[" + new SimpleDateFormat("HH:mm:ss").format(new Date()) + "] Hello " + name + ", request from consumer: " + RpcContext.getContext().getRemoteAddress());
        return "Hello " + name + ", response from provider: " + RpcContext.getContext().getLocalAddress();
    }
}
```

- 编写主函数

```java
public class Provider {
    public static void main(String[] args) throws Exception {
        System.setProperty("java.net.preferIPv4Stack", "true");
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/hello-dubbo-provider.xml"});
        context.start();
        System.in.read(); // press any key to exit
    }
}
```

- 编辑配置文件

注意，该文件必须在 META-INF/spring 路径下，具体原因还要深入学习

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
       <!-- 这里dubbo的地址有修改，未使用官方demo的 http://dubbo.apache.org/，
       本人使用这个地址一直不能启动，但官方demo可以启动，可能是dubbo版本不同吧 -->

    <!-- 应用名，用于追踪依赖关系 -->
    <dubbo:application name="demo-provider"/>

    <!-- 使用ZooKeeper为注册中心 -->
    <dubbo:registry address="zookeeper://localhost:2181"/>

    <!-- dubbo服务端口配置 -->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!-- 服务的具体实现 -->
    <bean id="demoService" class="cn.axboy.demo.dubbo.hello.provider.DemoServiceImpl"/>

    <!-- 声明暴露的服务 -->
    <dubbo:service interface="cn.axboy.demo.dubbo.hello.api.IDemoService" ref="demoService"/>
</beans>
```

#### 服务消费者

创建consumer子模块，这里maven依赖和provider一样，不再提供

- 编写主函数

```java
public class Consumer {
    public static void main(String[] args) {
        System.setProperty("java.net.preferIPv4Stack", "true");
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/hello-dubbo-consumer.xml"});
        context.start();
        IDemoService demoService = (IDemoService) context.getBean("demoService"); // get remote service proxy

        while (true) {
            try {
                Thread.sleep(1000);
                String hello = demoService.sayHello("world");
                System.out.println(hello); // get result
            } catch (Throwable throwable) {
                throwable.printStackTrace();
            }
        }
    }
}
```
- 编辑配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 应用名，用于追踪依赖关系，不要和服务提供者同名 -->
    <dubbo:application name="demo-consumer"/>

    <!-- 使用zookeeper注册中，服务发现 -->
    <dubbo:registry address="zookeeper://localhost:2181"/>

    <!-- 生成远程服务代理，在本地能当做普通方法调用 -->
    <dubbo:reference id="demoService" check="false" interface="cn.axboy.demo.dubbo.hello.api.IDemoService"/>
</beans>
```

#### zookeeper配置、启动

TODO

### 下载

[github/axboy](https://github.com/axboy/zcw.learn/tree/master/java/dubbo/hello-dubbo/)

### 参考

- [dubbo user book](http://dubbo.apache.org/books/dubbo-user-book/)