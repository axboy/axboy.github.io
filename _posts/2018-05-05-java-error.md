---
layout: post
title: NoSuchFieldError处理
description: 程序运行时报java.lang.NoSuchFIeldError，什么原因呢？
categories: java
tags: java
---

今天公司项目发布上线，可是上线后商品列表不能请求了，看日志错误如下。

```log
Caused by: java.lang.NoSuchFieldError: TYPE
  at com.cm.admin.service.goods.GoodsTagService.listAllTagBySpu(GoodsTagService.java:216)
  at com.cm.admin.service.goods.GoodsTagService$$FastClassBySpringCGLIB$$c44fbbaa.invoke(<generated>)
  at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
```

来来来，抓重点，NoSuchFieldError，定位到指定行，这个字段是有的，而且在本地和测试服务器都正常。如果这个字段不存在的话，应用应该是不能启动的，那么是什么原因呢？这里先用个demo复现这个错误。

首先，我们创建两个类，代码如下:

- TestClass.java

```java
public class TestClass {
    public static String str = "Hello By TestClass";
}
```

- Example.java

```java
public class Example {
    public static void main(String[] args){
        System.out.println(TestClass.str);
    }
}
```

用命令行编译并运行，命令如下

```shell
javac Example.java
java Example
```

输出如下:

```text
Hello By TestClass
```

一切正常，并未出现NoSuchFieldError。

接下来我们把TestClass类做如下修改，注释第二行。

```java
public class TestClass {
    //public static String str = "Hello By TestClass";
}
```

只编译TestClass类，而不把Example类编译，然后运行程序

```shell
javac TestClass.java
java Example
```

运行结果如下:

```text
Exception in thread "main" java.lang.NoSuchFieldError: str
	at Example.main(Example.java:3)
```

好了，NoSuchFieldError出现了。

要处理这个错误，我们要清除所有的 _.class_ 文件，重新编译，以保证所有的文件都是最新的。
如果这个错误运行时还存在，可能是编译时引用的依赖和运行时的版本不同，这里就要检查各个路径、版本是否错误了。
maven项目一般执行 _mvn clean_ 就可以了。

项目已成功上线，当时原因除了这个外，还有docker镜像也不是最新的(docker部署)，两错误一起，多绕了点弯，这里就不细说了。

### 参考文章

- [java.lang.NoSuchFieldError - How to solve SuchFieldError | Examples Java Code Geeks - 2018](https://examples.javacodegeeks.com/java-basics/exceptions/java-lang-nosuchfielderror-how-to-solve-suchfielderror/)

- [NoSuchFieldError (Java Platform SE 7 )](https://docs.oracle.com/javase/7/docs/api/java/lang/NoSuchFieldError.html)
