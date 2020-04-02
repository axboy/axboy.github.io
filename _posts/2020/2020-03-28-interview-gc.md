---
layout: post
title: 备战面试--Java垃圾回收
description: 关于垃圾回收的简单复习
categories: java
tags: intervirew java
---

之前有段时间学习jvm，只是时间断断续续，看过就忘，刚好近期准备面试，以文章形式备忘，笔记为主。

## 如何确定垃圾

主要有如下两种算法

### ~~引用计数法~~

> 给对象增加一个引用计数器，每当有一个地方引用它时，计数器就+1；当引用失效时，计数器就-1；计数器为0的对象就是不能再被使用的，即对象已”死”。

需要说明一下，在主流的JVM中没有选用引用计数法来管理内存，最主要的原因就是引用计数法无法解决对象的循环引用问题。

### 可达性分析法

此算法的核心思想是通过一系列“根”对象作为起始点，从根出发开始遍历，可以被访问到称为可达对象，即有用的，反之，则是要回收的。

根对象是认为一定有用的东西，有以下4类:
- 局部变量
- 静态变量
- Native方法引用的对象
- 活动中的线程

## 回收算法

### 标记清除算法(mark and sweep)

#### 原理

顾名思义，算法分为两个阶段，标记和清除。
在标记阶段，收集器从“根”对象出发，把能访问到的都打上标识。
而在清除阶段，收集器对堆内存从头到尾进行线性遍历，发现某个对象没有标记，则将其回收。
标记和清除阶段，应用程序都将暂停，Stop the world.

#### 缺点

标记清除后存在大量碎片

### 标记压缩算法(mark and compact)

#### 原理

同样，算法也是两个阶段。
标记阶段和标记清除算法一样。
区别在于压缩阶段，它的目的是把可达对象移动到堆内存的同一块区域中，使其紧凑排列，以减少内存碎片。

#### 缺点

需遍历多次以迁移对象，需要额外的开销，实现复杂

### 半区复制算法(copying)

#### 原理

之所以叫半区复制，是因为它将堆内存分为两个半区，只用其中一个半区来进行对象内存分配。
当这个半区内存不够时，则开始垃圾收集，将可达对象复制到另一个半区。

#### 效果

该算法目的也是为了解决内存碎片问题，对比标记压缩算法，它不需要遍历多次，节约时间。
但有一个主要缺点就是可用堆内存减小一半，对于大对象，复制比标记代价更高。

## 分代回收算法

首先一个基础假设，大部分对象只存在很短时间。

然后将内存分为新生代和老生代，新生代是经常需要垃圾回收的(Minor GC)，
将新生代再进一步划分，分为Eden、Survior1、Survior2，在新生代用半区复制算法，
每次将eden和一个survior的对象回收后拷贝到另一个survior中，并标记存活次数，因为有前面的基础假设，拷贝的对象一般不会太多。

当存活次数到一定次数后（比如15次），将对象放入老生代中，老生代发生GC的概率更小，在老生代就采用标记压缩算法。

除了新生代和老生代外，还有个持久代(Permanent Generation)，java8以后为MetaSpace，功能如下。
```
> Permanent Generation
1. 放置ClassLoader读进来的class，除系统class外
2. 放置String.intern后的结果
3. 可能出现OutOfMemoryError:PermGen Space

> Meta Space
1. java8 之后使用Meta space,取消Permanent space
2. String.intern的结果放入堆
3. meta space默认不限制，使用系统内存
```
综上，分带回收分类如下：

```
- 新生代
  - Eden
  - Survior1
  - Survior2
- 老生代
- 持久代/Meta space
```

说了这么多，每一块内存到底多大。不要慌，这些都是参数，可以配置的。

```
-XX:NewRatio，老生代/新生代比例，默认2
-XX:SurvivorRatio，Eden/Survivor比例，默认8，survior有两块，所以８:1:1
-XX:MaxTenuringThreshold，新生代转至老生代的阈值，默认15
```

## 调试

### 获取信息
```
-verbose:gc
-XX:+HeapDumpOnOutOfMemoryError
-XX:+PrintGCDetails -Xloggc:gc-log-file-path
Spring Actuator
```

### 工具
```
官方：visualvm,jmap
第三方：Eclipse Memory Analyzer(MAT)
在线服务：gceasy.io,fastthread.io
```