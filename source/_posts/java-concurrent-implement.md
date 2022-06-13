---
title: 「Java并发实现原理」读书笔记 [DOING]
catalog: true
date: 2022-06-12 15:51:25
subtitle:
header-img:
tags:
- language
categories:
- 工程
---

> 书籍豆瓣链接：[《Java并发实现原理：JDK源码剖析》](https://book.douban.com/subject/35013531/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

# 一、多线程基础

## 线程的优雅关闭

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/java-concurrent-implement/thread-state.png?raw=true)

stop、destroy之类的函数不建议使用，强制杀死线程，线程中所使用的资源如描述符和网络连接无法正常关闭，通过传递标志位来完成线程退出操作

JVM中线程分为守护线程和非守护线程，所有非守护线程退出后，整个JVM进程就会退出

## interrupt



## synchronized

## wait与notify

## volatile关键字

# Java内存模型

JVM为了跨平台，实现了Java内存模型用来屏蔽掉各种硬件和操作系统的内存访问差异

JMM的目标是定义程序中各个变量的访问规则，即在JVM中将变量从内存中取出和存入的底层细节

JMM的关键技术点都是围绕多线程的原子性、可见性和有序性来建立的

## 可见性

缓存同步协议MESI，规定每条缓存有个状态位，定义了四种状态M、E、S和I

单个CPU对缓存中数据进行了改动，需要通知给其他CPU；这意味着CPU不仅要控制自己的读写操作

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/java-concurrent-implement/?raw=true)



## 内存屏障

为了禁止编译器重排序和CPU重排序，在编译器和CPU层面都有内存屏障指令，也是JMM和happen-before规则的底层实现原理

### Linux内核例子

kfifo.c源代码中RingBuffer

在修改数据和更新指针之间，通过smp_wmb()插入了一个Store Barrier，从而确保两个操作：

* 更新指针操作不会被重排到修改数据前
* 数据的修改在更新指针前，已经完成Store Cache刷新，其他CPU可见

### JDK内存屏障





# Atomic类

# Lock和Condition
## 互斥锁

AbstractQueuedSynchronizer是用来构建锁或者其他同步组件的基础框架

### 基本原理

* 保存一个state变量标记该锁的状态，标识持有线程数量，对state变量的操作使用CAS保证线程安全
* 记录当前持有锁的线程
* 基于底层的park()和unpark()阻塞自己，或者唤醒线程
* 使用一个线程安全的无锁队列维护所有的阻塞的线程

### 无锁队列实现

双向链表，有head和tail指针

入队流程：

```
pred = tail // 获取tail的快照pred
node.prev = pred // 新节点node的prev指向pred
if (CAS(pred, node)) { // 如果tail==pred，将tail设置为pred
	pred.next=node
}
```

出队流程：release中调用unparkProcessor，由于release会判断是否持有锁，所以不需要CAS



## 读写锁

## Condition


# 并发容器

# 线程池与Future 

# ForkJoinPool

# CompleteFuture
