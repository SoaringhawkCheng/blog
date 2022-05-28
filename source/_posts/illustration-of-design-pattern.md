---
title: 「图解设计模式」学习笔记
catalog: true
date: 2022-03-21 15:43:42
subtitle:
header-img:
tags:
- software
categories:
- 工程
---

> 书籍豆瓣链接：[《图解设计模式》](https://book.douban.com/subject/26933281/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

![](<!---->https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/illustration-of-design-pattern/mind.png?raw=true)

## 关于UML

统一建模语言(UML, Unified Modeling Language)

# UML类图

* 泛化关系

	![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/head-first-design-pattern/class_generalize.png?raw=true)
	
	继承类和接口，java中通过extends实现

* 实现关系

	![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/head-first-design-pattern/class_realization.png?raw=true)

	实现接口，java中使用implements表示

* 依赖关系

	![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/head-first-design-pattern/class_dependency.png?raw=true)
	
	是一种非常弱、临时性的关系。java中表现为，局部变量、方法中的参数和静态方法调用

* 关联关系

	![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/head-first-design-pattern/class_association.png?raw=true)

	java中使用成员变量实现，或者引用全局变量

* 聚合关系

	![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/head-first-design-pattern/class_aggregation.png?raw=true)
	
	整体和部分弱依赖，部分生命周期不依赖整体，比如指针成员变量

* 组合关系

	![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/head-first-design-pattern/class_composition.png?raw=true)
	
	整体和部分强依赖，部分生命周期依赖整体，如普通成员变量

## 第一部分 适应设计模式

## 第二部分 交给子类

## 第三部分 生成实例

## 第四部分 分开考虑

## 