---
title: "JVM 堆内存模型"
date: 2020-08-10T10:41:54+08:00
categories: 
 - Java
tags: 
  - JVM
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - JVM
description: 
---
## JVM 堆内存模型

`jk8` 

## Survivorv <新生代>

> 程序运行过程中新产生的对象都会分配在新生代，如果jvm采用的是分代收集算法 则会在新生代中的伊甸园去，随着时间的推移伊甸园区存放的对象会越来越多，直到伊甸园内存占满，这个时候就会进行jvm会执行轻GC`Minor GC`清理对象 ，清理过程有的对象被清除，有的对象会继续存活下去。存活下来的对象会被放入S0 或 S0 (From  or  TO)

## Old <老年代>

> 伴随着程序长时间的运行 每次轻GC都会有一部分对象存活下来，每次躲过轻GC 的清理，对象的年龄都会+1，当这个对象的 年轻达到15岁时 再次进行轻GC时,这个时候 jvm会将这些年轻达到15岁的对象放入 Old 老年区中

## MetaSpace <元空间>

> jdk7以前 是方法区 占用的内存是jvm分配的，jdk8之后是直接使用宿主机的内存

工具 jvisualvm

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528191934274.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

**example**

这段代码会一直运行 直到OOM

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052819452068.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528195039193.png)

## 垃圾回收算法

### 复制算法

>  