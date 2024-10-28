---
title: '[译]Java内存泄露介绍'
enlink: 'the-introduction-of-memory-leak-what-why-and-how'
date: 2017-02-21 15:05:39
categories:
- 后端
tags:
- java
- translation
---
内存管理是Java最大的优势之一；你可以很简单的创建一个对象，内存的分配和释放则交给Java垃圾收集器处理；然而实际情况并非如此简单，因为在Java应用程序中会频繁的发生内存泄露。

这个教程将会说明内存泄露是什么？它为什么会发生？我们如何防止它？
## 内存泄露是什么
内存泄露的定义：对象不再被应用程序使用，但是由于它们还在被引用，垃圾收集器不能清除掉它们。

为了理解这个定义，我们需要理解对象在内存中的状态；下面的图表说明什么是未被使用和未被引用。

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/FgwzTgT7v_YeTJ0Y_pWnoJXyfeHg.jpeg)

图表中，有被引用的对象和未被引用的对象；未被引用的对象将会被当做垃圾回收，而被引用的对象将不会被当做垃圾回收；未被引用的对象由于没有被其他对象引用，它当然也是不被使用的对象，然而，不被使用的对象不全是不被引用的，它们中的一些是被引用的！这就是内存泄露的来源。

## 内存泄露为什么会发生
让我们来看一下下面这个例子，它说明了内存泄露为什么会发生。在下面这个列子中，对象A引用了对象B，A的生命周期（t1~t4）是比B（t2~t3）的长；当B不再被应用程序使用时，A仍然在引用它；在这种情况下，垃圾收集器不能从内存中移除B；如果A引用了很多类似B这样的对象，它们不能被回收，又消耗着内存空间的资源，这样很有可能造成内存不足的问题。

还有一种可能的事情，B又引用了一些对象，这些被B引用的对象也不能被回收，那所有这些不被使用的对象将消耗大量宝贵的内存空间。
![](https://cdn.jsdelivr.net/gh/yelog/assets/images/Fm3d2a94sdbY4mb5ua-_BjAusKbq.jpeg)

## 如何防止内存泄露
下面有一些防止内存泄露的快速实践技巧
1. 注意集合类，如：HashMap、ArrayList等等，因为它们是在常见的地方发生内存泄露；当它们被`static`声明时，它们和应用程序的生命周期是一样长的。
2. 注意事件监听和回调，当一个监听事件被注册，而这个类再也没有被使用时可能会发生内存泄露。
3. “如果一个类管理自己的内存，程序员应该被提醒内存泄漏了”，通常，一个对象的指向其他对象的成员变量需要被置为null。

References：
[1] Program Creek :[The Introduction of Java Memory Leaks](http://www.programcreek.com/2013/10/the-introduction-of-memory-leak-what-why-and-how/)
