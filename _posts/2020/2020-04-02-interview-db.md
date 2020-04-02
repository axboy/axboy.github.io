---
layout: post
title: 备战面试--数据库知识点整理
description: 关于数据库的简单复习
categories: java
tags: intervirew java
---

## 问题
- MyBatis中，#{}和${}的区别是什么
```
#{}是预编译处理，${}是字符串替换
MyBatis在处理#{}时，会将sql中的#{}替换成?号，调用PreparedStatement的set方法赋值
使用#{}可以有效的防止sql注入，提高系统安全性
```

- MyBatis是如何进行分页的?分页插件的原理是什么?
```
MyBatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页。
分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。
```

- MySQL中，哪些情况下不会用到索引
```

```
