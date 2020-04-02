---
layout: post
title: 备战面试--集合
description: 关于集合的简单复习
categories: java
tags: intervirew java
---

# Collection
## List
- ArrayList
- LinkedList

## Set
- HashSet
- LinkedHashSet
- TreeSet
- EnumSet

## Queue
- LinkedList
- PriorityQueue
- BlockingQueue

## Map
- HashMap
- EnumMap
- ConcurrentHashMap
- WeakHashMap
- IdentityHashMap

## Deprecated
- Hashtable
- Stack
- Vector

### 常见问题

- Java中容器有哪些?哪些是同步容器?哪些是并发容器?
```
同步容器：Vector,Stack,Hashtable,Collections.synchronized
并发容器：ConcurrentHashMap,CopyOnWriteAyyayList,CopyOnWriteArraySet,ArrayBlockingQueue,LinkedBlockingQueue
```

- ArrayList、LinkedList的插入、访问的时间复杂度？
```
ArrayList: get(): O(1), add(E): O(1), add(index, E): O(n)
LinkedList: get(): O(n), add(E): O(1), add(index, E): O(n)
```

- HashMap检测到hash冲突后，将元素插入在链表末尾还是开头？
```
jdk1.7之前，包括1.7，头插入，
jdk8是尾插入
```

- HashMap和Hashtable的区别
```
1. Hashtable是同步的，HashMap不是
2. Hashtable的键值都不能为null，HashMap可以，所以HashMap的get(key)方法不能用于判断key是否存在，应使用containsKey(key)方法
3. hash值不同，Hashtable用hashCode()，HashMap重新计算
4. 扩容方式不同，Hashtable默认容量11，HashMap默认容量16
```

- HashSet添加重复元素会如何?
```
添加失败，不会替换元素。
这要看HashMap的put方法的实现，相同的key不会替换key
```