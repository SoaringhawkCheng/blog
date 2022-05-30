---
title: 「图解设计模式」学习笔记 [DOING]
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

## 一、适应设计模式

### 1.1 迭代器模式 Iterator

### 1.2 适配器模式 Adapter

## 二、交给子类

### 2.1 模板方法模式 Template Method

### 2.2 工厂方法模式 Factory Method

## 三、生成实例

### 3.1 单例模式 Singleton

### 3.2 原型模式 Prototype

### 3.3 建造者模式 Builder

builder定义构建接口，concrete builder实现接口

director调用builder的接口生成实例

### 3.4 抽象工厂模式 Factory

## 四、分开考虑

### 4.1 桥接模式 Bridge

### 4.2 策略模式 Strategy

## 五、一致性

### 5.1 组合模式 Composite

### 5.2 装饰模式 Decorator

## 六、访问数据结构

### 6.1 访问者模式 Visitor

### 6.2 职责链模式 Chain of Responsibility

## 七、简单化

### 7.1 外观模式 Facade

### 7.2 中介者模式 Mediator

## 八、管理状态

### 8.1 观察者模式 Observer

### 8.2 备忘录模式 Memento

### 8.3 状态模式 State

## 九、避免浪费

### 9.1 享元模式 Flyweight

### 9.2 代理模式 Proxy

## 十、用类来表现

### 10.1 命令模式 Command

### 10.2 解释器模式 Interpreter