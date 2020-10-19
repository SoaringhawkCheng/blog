---
title: 「软件架构设计：大型网站技术架构与业务架构融合之道」学习笔记[DOING]
catalog: true
date: 2020-05-24 21:06:15
subtitle:
header-img:
tags:
- architecture
categories:
- 工程
---
> 书籍豆瓣链接：[《软件架构设计：大型网站技术架构与业务架构融合之道》](https://book.douban.com/subject/30443578/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

[分布式共识博客](http://blog.kongfy.com/2016/05/%E5%88%86%E5%B8%83%E5%BC%8F%E5%85%B1%E8%AF%86consensus%EF%BC%9Aviewstamped%E3%80%81raft%E5%8F%8Apaxos/)

[零拷贝](https://www.jianshu.com/p/193cae9cbf07)
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/%E5%88%86%E5%B8%83%E5%BC%8F.jpg?raw=true)

## 第4章 操作系统

### 零拷贝

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-0.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-context-0.png?raw=true)

上下文切换是CPU密集型的工作，数据拷贝是I/O密集型的工作。如果一次简单的传输就要像上面这样复杂的话，效率是相当低下的。零拷贝机制的终极目标，就是消除冗余的上下文切换和数据拷贝，提高效率

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-context-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-1.png?raw=true)

可见，不仅拷贝的次数变成了3次，上下文切换的次数也减少到了2次，效率比传统方式高了很多。但是它还并非完美状态，下面看一看让它变得更优化的方法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-context-2.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-2.png?raw=true)

在“基础”零拷贝方式的时序图中，有一个“write data to target socket buffer”的回环，在框图中也有一个从“Read buffer”到“Socket buffer”的大箭头。这是因为在一般的Block DMA方式中，源物理地址和目标物理地址都得是连续的，所以一次只能传输物理上连续的一块数据，每传输一个块发起一次中断，直到传输完成，所以必须要在两个缓冲区之间拷贝数据。

而Scatter/Gather DMA方式则不同，会预先维护一个物理上不连续的块描述符的链表，描述符中包含有数据的起始地址和长度。传输时只需要遍历链表，按序传输数据，全部完成后发起一次中断即可，效率比Block DMA要高。也就是说，硬件可以通过Scatter/Gather DMA直接从内核缓冲区中取得全部数据，不需要再从内核缓冲区向Socket缓冲区拷贝数据。

## 问题分类

### 侧重于高并发读

### 侧重于高并发写

## 高并发问题

### 高并发读

* 策略1 加缓存

	缓存

	mysql从库

	动静分离，cdn静态文件加速
	
* 策略2 并发读

	rpc异步调用
	
* 策略3 重写轻度

	不是查询的时候再去聚合，而是将计算逻辑从读的一端移到写的一端
	
### 高并发写

* 策略1 数据分片

	分库分表

* 策略2 任务分片

* 策略3 异步化

	操作系统层面的异步IO

	收到请求不立即处理，落盘(数据库消息中间件)
	
* 策略4 批量处理

	kafka如果是异步发送，支持批量操作

* 策略5 串行化+多进程单线程+异步IO

	多线程两大问题：锁竞争，线程切换开销大

### CQRS架构

* 读和写设计不同的数据结构

* 写一端，DB分库分表

* 读一端，使用kv缓存，join宽表，ES等

* 读和写的串联，binlog，消息队列

* 读比写有延迟，保证最终一致性

## 如何保证分布式系统中接口调用的顺序性？

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/call-in-sequence-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/call-in-sequence-2.jpeg?raw=true)

## Raft算法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-log-index.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-replicated-state-machine.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-state-machine-safety.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-state-transfer.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-term.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-two-disjoint-majorities.png?raw=true)