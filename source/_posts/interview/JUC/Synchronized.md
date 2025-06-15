---
title: Synchronized
date: 2025-05-20T21:44:32+08:00
tags: [Concurrent]
categories: [java面试]
index_img: 
banner_img: 

excerpt: 自己面试用的对Synchronized的从头至尾的总结
author: GENCO 
---
## 为什么要有Synchronized
+ 在解决这个问题之前:我们得先了解一下整个java的内存模型也就是JMM

{% note primary %}
这里有个有趣的点:为什么有JMM的存在呢?
> 由于java语言本身是希望做跨平台支持的,但是面对的是各种硬件上的不统一,CPU,内存,访存方式上不一致,用的缓存协议不一致,从而多线程环境下不同的线程读取变量可能不统一,**就需要人为去定义一个JAVA的内存模型了,主要为了便于跨平台上的并发是一致的**
+ 一句话核心:为了更方便的跨平台而人为定义的java线程内存模型
{% endnote %}


{% fold primary @java内存模型(可以先自行回忆一下) %}
![](https://65728-1316358396.cos.ap-beijing.myqcloud.com/imgs/postUse/JMM%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.webp?imageSlim)
[^1] 引用Java Guide图片
{% endfold %}  
## JMM内存模型工作方式:
+ 线程1可以读取主存当中的共享变量,并可以将其拷贝到线程一的本地内存当中,作为副本参与线程一的操作 
+ 线程2可以把本地修改过的内存变量副本给同步到主内存当中去

{% note danger %}
这里一定要分清JVM和JMM,对于JMM来说主要是针对并发的模型,而JVM则是整个Java运行时的内存虚拟机
{% endnote %}
