---
title: java basics - box and unbox
date: 2019-09-20 22:54:25
tags:
---

## java 基础之拆箱、装箱

先看看下面的代码，有疑问再继续看。

```java
int a = 10;
Integer b = 10;
System.Out.println(a == b); // true or false ?
Integer c = 10;
System.out.println(b == c); // true or false ?
b = c  = 150;
System.out.println(b == c); // true of false ?
b = 150;
c = 150;
System.out.println(b == c); // true or false ?
```

看一看，想一想，答案在最后。

## 什么是基本类型

java中有8个基本类型，都有对应的包装类型。

| 基本类型 | 包装类型 | 字节数 |
|---|---|---|
|byte| Byte | 1|
|short | Short | 2|
|int | Integer| 4|
|float| Float| 4|
|long | Long| 8|
|double| Double| 8|
|char| Char | 2|
|boolean| Boolean | 单独使用是4个字节，数组中又是1个字节。|

## 基本类型和包装类型的区别

1. 基本类型存在栈中，对象存储在堆中。
2. 初始值不同，包装类型都为null，int为0.
3. 集合中操作的只能是对象，所以必须要做装箱。

## 装箱和拆箱

基本类型转包装类型（装箱）： valueOf()
包装类型转基本类型（拆箱）： xxxValue() [xxx代表基本数据类型]

valueOf()在sdk中解释如下，注意[-128,127]范围的值将缓存

```
public static Integer valueOf(int i)
Returns an Integer instance representing the specified int value. If a new Integer instance is not required, this method should generally be used in preference to the constructor Integer(int), as this method is likely to yield significantly better space and time performance by caching frequently requested values. 
This method will always cache values in the range -128 to 127, inclusive, and may cache other values outside of this range.
Parameters:
i - an int value.
Returns:
an Integer instance representing i.
Since:
1.5
```






## 这里是答案

```
true  //拆箱对比值相等
true  //[-127,128]范围内整数做了缓存对比。
true  //传引用，指向同一个对象。
false //超出缓存值范围，不同对象引用的比较
```
