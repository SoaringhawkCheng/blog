---
title: 「Redis深度历险」读书笔记 [TODO]
catalog: true
date: 2022-05-14 12:18:12
subtitle:
header-img:
tags:
- storage
categories:
- 工程
---

> 书籍豆瓣链接：[《Redis 深度历险：核心原理与应用实践》](https://book.douban.com/subject/30386804/
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

redis占用内存评估

[高并发redis学习笔记](https://so.51cto.com/?sever_type=2&keywords=%E9%AB%98%E5%B9%B6%E5%8F%91redis%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0)

# 一、基础篇

## 1.1 分布式锁
[Redis实现分布式锁的7种方案](https://blog.csdn.net/fengyuyeguirenenen/article/details/123752418)

### 1.1.1 SETNX + EXPIRE

先用setnx来抢锁，如果抢到之后，再用expire给锁设置一个过期时间

不是原子操作，容易产生未释放死锁的问题

### 1.1.2 SETNX + value值是(系统时间+过期时间)

为了解决方案一，发生异常锁得不到释放的场景

过期时间放到setnx的value值里面。如果加锁失败，再拿出value值校验一下

# 二、原理篇

# 三、集群篇

# 四、拓展篇

# 五、源码篇