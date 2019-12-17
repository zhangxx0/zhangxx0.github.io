---
layout:     post
title:      "Java集合（一）-Overview"
subtitle:   ""
date:       2017-09-24 02:00:00
author:     "zhang.xx"
header-img: "img/post-everydayguess.jpg"
catalog: true
tags:
    - Java进阶
---

> If more of us valued food and cheer and song above hoarded gold, it would be a merrier world. ——**John·Ronald·Reuel·Tolkien**

---

## 前言
Java集合是java提供的工具包，包含了常用的数据结构：集合、链表、队列、栈、数组、映射等。Java集合工具包位置是java.util.*  
Java集合主要可以划分为4个部分：List列表、Set集合、Map映射、工具类(Iterator迭代器、Enumeration枚举类、Arrays和Collections)

Java 容器里只能放对象，对于基本类型（int, long, float, double 等），需要将其包装成对象类型后（Integer, Long, Float, Double 等）才能放到容器里。很多时候拆包装和解包装能够自动完成。这虽然会导致额外的性能和空间开销，但简化了设计和编程。

## 概览
工具包框架图：  
![](https://gitee.com/zhangxx0/blog_image/raw/master/java/jhtotal02.jpg)

#### Collection接口(集合)
* List、Set和Queue接口的父接口  
* 定义了可用于操作List、Set和Queue的方法-增删改查  

#### Map接口(关联式容器)
* Map提供了一种映射关系，其中的元素是以键值对（key-value）的形式存储的，能够实现根据key快速查找value
* Map中的键值对以Entry类型的对象实例形式存在
* 键（key值）不可重复，value值可以
* 每个建最多只能映射到一个值
* Map接口提供了分别返回key值集合、value值集合以及Entry（键值对）集合的方法
* Map支持泛型


## 泛型-Generics
Java 容器能够容纳任何类型的对象，这一点表面上是通过泛型机制完成，Java 泛型不是什么神奇的东西，只是编译器为我们提供的一个 “语法糖”，泛型本身并不需要 Java 虚拟机的支持，只需要在编译阶段做一下简单的字符串替换即可。实质上 Java 的单继承机制才是保证这一特性的根本，因为所有的对象都是 Object 的子类，容器里只要能够存放 Object 对象就行了。  
事实上，所有容器的内部存放的都是 Object 对象，泛型机制只是简化了编程，由编译器自动帮我们完成了强制类型转换而已。

```
  //JDK 1.5 or latter
  ArrayList<String> list = new ArrayList<String>();//参数化类型
  list.add(new String("Monday"));
  list.add(new String("Tuesday"));
  list.add(new String("Wensday"));
  for(int i = 0; i < list.size(); i++){
      String weekday = list.get(i);//隐式类型转换，编译器自动完成
      System.out.println(weekday.toUpperCase());
  }
```
## 内存管理
跟 C\++ 复杂的内存管理机制不同，Java GC 自动包揽了一切，Java 程序并不需要处理令人头疼的内存问题，因此 JCF 并不像 C\++ STL 那样需要专门的空间适配器（alloctor）。  
另外，由于 Java 里对象都在堆上，且对象只能通过引用（reference，跟 C\++ 中的引用不是同一个概念，可以理解成经过包装后的指针）访问，容器里放的其实是对象的引用而不是对象本身，也就不存在 C\++ 容器的复制拷贝问题。

## 迭代器
```
  //visit a list with iterator
  ArrayList<String> list = new ArrayList<String>();
  list.add(new String("Monday"));
  list.add(new String("Tuesday"));
  list.add(new String("Wensday"));
  Iterator<String> it = list.iterator();//得到迭代器
  while(it.hasNext()){
      String weekday = it.next();//访问元素
      System.out.println(weekday.toUpperCase());
  }
```

JDK 1.5 引入了增强的 for 循环，简化了迭代容器时的写法:  
```
  //使用增强for迭代
  ArrayList<String> list = new ArrayList<String>();
  list.add(new String("Monday"));
  list.add(new String("Tuesday"));
  list.add(new String("Wensday"));
  for(String weekday : list){//enhanced for statement
      System.out.println(weekday.toUpperCase());
  }
```

## 总结与参考
参考了尚学堂高淇的集合视频15集，以及下面相关博客，代码示例均为手敲，jdk1.8下编译通过；

参考：

[Java集合干货系列-集合总体大纲](http://tengj.top/2016/04/12/javajhtotal/)  
[JCFInternals-1-Overview](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/1-Overview.md)  


2017年9月-by zhang.xx
