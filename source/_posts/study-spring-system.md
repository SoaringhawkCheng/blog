---
title: 「Spring框架体系」学习笔记「TODO」
catalog: true
date: 2023-04-03 18:18:51
subtitle:
header-img:
tags:
- framework
categories:
- 工程
---

> 书籍豆瓣链接：
> [《Spring源码深度解析（第2版）》](https://book.douban.com/subject/30452948/)
> [《Spring Cloud Alibaba 微服务原理与实战》](https://book.douban.com/subject/35041576/)
> 
> 相关在线资料：

BeanDefinitionRegistryPostProcessor
所有常规bd已经加载完毕，然后可以再添加一些额外的bd

BeanFactoryPostProcessor
Bean实例化之前执行，所有的bd已经全部加载完毕，然后可以对这些bd做一些属性的修改或者添加工作

BeanPostProcessor
针对Bean实例化之后做一些逻辑处理

