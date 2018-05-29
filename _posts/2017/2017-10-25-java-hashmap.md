---
layout: post
title: HashMap源码分析
description: HashMap基于哈希表的 Map 接口的实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键。
categories: java
tags: java HashMap
---

### 概述

> 基于哈希表的 Map 接口的实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键。（除了非同步和允许使用 null 之外，HashMap 类与 Hashtable 大致相同。）此类不保证映射的顺序，特别是它不保证该顺序恒久不变。 此实现假定哈希函数将元素适当地分布在各桶之间，可为基本操作（get 和 put）提供稳定的性能。迭代 collection 视图所需的时间与 HashMap 实例的“容量”（桶的数量）及其大小（键-值映射关系数）成比例。所以，如果迭代性能很重要，则不要将初始容量设置得太高（或将加载因子设置得太低）。

### 源码分析

- 内部类

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;     //key的hash值
    final K key;        //key
    V value;            //值
    Node<K,V> next;     //下一个节点的引用
    ...
}

//注意 LinkedHashMap.Entry extends HashMap.Node，即HashMap.TreeNode extends HashMap.Node
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
}
```

### 属性

```java
private static final long serialVersionUID = 362498820763181265L;       //序列号
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;         //初始默认容量，必须为2的幂次倍
static final int MAXIMUM_CAPACITY = 1 << 30;                //最大容量
static final float DEFAULT_LOAD_FACTOR = 0.75f;             //默认填充因子
static final int TREEIFY_THRESHOLD = 8;                     //当bucket上的结点数大于该值时会转成红黑树
static final int UNTREEIFY_THRESHOLD = 6;                   //当bucket上的结点数小于该值时树转链表
static final int MIN_TREEIFY_CAPACITY = 64;                 //树形最小容量

transient Node<K,V>[] table;                                //存储元素的数组，长度总是2的幂次倍
transient Set<Map.Entry<K,V>> entrySet;                     //存放元素的集合
transient int size;                         //存放元素的个数
transient int modCount;                     //当前HashMap的修改次数
int threshold;                              //阈值，当实际大小(容量*填充因子)超过临界值时，会进行扩容
final float loadFactor;                     //填充因子
```

### 构造函数

```java
//带两参数的构造函数，另外几个构造函数不做介绍
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;         //当初始容量大于最大容量，设为最大容量
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;                   //初始化填充因子
    this.threshold = tableSizeFor(initialCapacity); //初始化临界值
}

//返回cap的最小二次幂值，把cap的低位全部置1
static final int tableSizeFor(int cap) {
    //>>> 无符号右移，高位补0
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### 获取hash值

```java
//hashCode右移16位，减少冲突
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

由于哈希表的容量是2幂次倍，元素的hashCode很多时候低位是相同的，更容易导致冲突，故而将key的hashCode无符号右移16位，再与本身按位疑惑，减少冲突。

### put方法，增加元素

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);      //获取hashCode，调用putVal方法
}

/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;                 //
    if ((tab = table) == null || (n = tab.length) == 0)     //如果当前table为空
        n = (tab = resize()).length;                        //重置table，n赋值为table的长度
    if ((p = tab[i = (n - 1) & hash]) == null)              //p赋值为当前的链表头，如果当前node为null
        tab[i] = newNode(hash, key, value, null);           //创建新节点
    else {
        Node<K,V> e; K k;                   //
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;                          //如果为同一个key，e赋值为链表头
        else if (p instanceof TreeNode)     //如果为红黑树,进入红黑树赋值判断，这里不做展开
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {  //其它情况，遍历链表 
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1)  //当链表节点数大于设定的值，转为红黑树
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;          //存在一样的key，跳出循环
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;        //值替换
            afterNodeAccess(e);         //空方法
            return oldValue;
        }
    }
    ++modCount;                 //计数器加1
    if (++size > threshold)
        resize();               //容量大于阈值，扩容
    afterNodeInsertion(evict);  //空方法
    return null;
}
```

- 代码流程如下

    1. 调用hash()方法获取hash值

    2. 调用putVal()方法进行实际操作

    3. 首先对哈希表进行空判断，若为空则新建哈希表

    4. 根据hash值获取当前的bucket

    5. 如果bucket为空，则把当前元素放进去，否则从第一个元素开始匹配

        1. 首先判断新插入的key是否和第一个元素的key一样，若一样，则替换值

        2. 若不一样，继续查找，直到遍历完成或者找到相同的key

        3. 遍历过程中，节点数大于预定的值，转为红黑树

    6. 若存在相同的key，值替换，返回原来的值

    7. 若不存在相同的key，计数器和容量加1

    8. 判断容量是否大于阈值，达到阈值，扩容

### resize方法，扩容

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;                         //备份
    int oldCap = (oldTab == null) ? 0 : oldTab.length;  //原bucket数量
    int oldThr = threshold;                             //原阈值
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {               //超过最大容量
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;   // double threshold,双倍容量
    }
    else if (oldThr > 0)            // initial capacity was placed in threshold，初始化
        newCap = oldThr;
    else {                          // zero initial threshold signifies using defaults,初始化
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {      //若阈值为0
        float ft = (float)newCap * loadFactor;      //容量 * 填充因子
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {      //遍历原table
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {      //若链表首元素有值
                oldTab[j] = null;
                if (e.next == null)             //若链表无下一节点引用
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode) //若bucket为树节点
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {        //遍历链表，重新赋值
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### get方法，获取元素

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;      //获取hashCode,调用getNode方法
}

/**
 * Implements Map.get and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @return the node, or null if none
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
            return first;                       //链表头为所需元素
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)      //如果为红黑树，获取红黑树节点,这里不做展开
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {                                //遍历链表
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}       
```

### 鸣谢

- [百度百科 HashMap](https://baike.baidu.com/item/Hashmap)

- [百度百科 哈希表](https://baike.baidu.com/item/%E5%93%88%E5%B8%8C%E8%A1%A8)

- [http://www.importnew.com/7099.html](http://www.importnew.com/7099.html)

- [http://blog.csdn.net/u011240877/article/details/53351188](http://blog.csdn.net/u011240877/article/details/53351188)