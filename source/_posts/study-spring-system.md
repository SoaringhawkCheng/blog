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


那么如果想要被Spring容器管理的Bean的路径不再Spring Boot 的包扫描路径下，怎么办呢？也就是如何去加载第三方的Bean 呢?

[ApplicationContext容器源码解析：spring启动时是如何解析xml配置文件并注册bean的？](https://zhuanlan.zhihu.com/p/358803204)

[spring boot自动装配原理](https://zhuanlan.zhihu.com/p/503007698)

[Spring基础组件的使用——@ComponentScan扫描规则及其源码分析](https://blog.csdn.net/qq_27610647/article/details/115704426)


