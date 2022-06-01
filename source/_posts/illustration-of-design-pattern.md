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

<!--![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/illustration-of-design-pattern/mind.png?raw=true)-->

## 零、前言

统一建模语言(UML, Unified Modeling Language)，具体见[「Head first设计模式」学习笔记](https://blog.soaringhawkcheng.tech/%E5%B7%A5%E7%A8%8B/head-first-design-pattern/)

## 一、适应设计模式

### 1.1 迭代器模式 Iterator

### 1.2 适配器模式 Adapter

## 二、交给子类

### 2.1 模板方法模式 Template Method

### 2.2 工厂方法模式 Factory Method

#### 2.2.1 简单工厂简介

简单工厂模式不属于23种GOF设计模式，是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类

* 优点：实现了对象创建和使用的分离，不修改客户端代码即可更改具体产品
* 缺点：更改具体产品需要修改工厂类，违反了开放——封闭原则

#### 2.2.2 工厂方法

工厂方法将工厂抽象化，增加新产品只需增加对应的具体实现工厂类
factory定义工厂接口，product定义产品接口，concrete product和对应的concrete factory实现接口

* 优点：解决了简单工厂不符合开放——封闭原则的问题
* 缺点：增加新产品需要增加工厂类

## 三、生成实例

### 3.1 单例模式 Singleton

### 3.2 原型模式 Prototype

### 3.3 建造者模式 Builder

builder定义构建接口，concrete builder实现接口

director调用builder的接口生成实例

### 3.4 抽象工厂模式 Abstract Factory

#### 3.4.1 对比工厂方法

工厂方法模式中，我们使用一个工厂创建一个产品，一个具体工厂对应一个具体产品

抽象工厂模式拥有多个抽象产品类（产品族）和一个抽象工厂类， 一个工厂能够提供多个产品对象，即一个产品族

#### 3.4.2 产品概念

产品等级结构：产品等级结构指的是产品的继承结构，例如一个空调抽象类，它有海尔空调、格力空调、美的空调等一系列的子类，那么这个空调抽象类和他的子类就构成了一个产品等级结构。

产品族：产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品。比如，海尔工厂生产海尔空调、海尔冰箱，那么海尔空调则位于空调产品族中

#### 3.4.3 优缺点

易于增加新的具体工厂，难以增加新的零件

## 四、分开考虑

### 4.1 桥接模式 Bridge

将类的功能层次结构和实现层次结构分离，不使用继承而使用组合委托

#### 4.1.1 类的层次结构

功能层次结构：extends，实现层次结构：implement

#### 4.1.2 组成结构

* abstraction：持有implementer的实例，并简单封装了implementer的方法
* refined abstraction：在abstraction基础上新增了功能
* implementor：定义了abstraction接口依赖的方法
* concrete implementer：实现了implementer的接口

#### 4.1.3 优缺点

桥接模式可以取代多层继承方案，多层继承方案违背了“单一职责原则”，复用性较差，且类的个数非常多，桥接模式是比多层继承方案更好的解决方法，它极大减少了子类的个数，如下面的示意。

```
                   ----Shape---
                  /            \
         Rectangle              Circle
        /         \            /      \
BlueRectangle  RedRectangle BlueCircle RedCircle
```
```
          ----Shape---                        Color
         /            \                       /   \
Rectangle(Color)   Circle(Color)           Blue   Red
```

### 4.2 策略模式 Strategy

strategy负责定义策略所必须的接口，concrete strategy实现这些接口

context使用组合，持有concrete strategy实例

将策略实现解耦出来，使用委托的方式，可以动态替换算法

## 五、一致性

### 5.1 混合模式 Composite

#### 5.1.1 详细原理

目录树结构使用的设计模式，目录下即有文件也有子目录

容器和内容具有一致性，提供一致性的方法，创造出递归结构的模式

#### 5.1.2 组成结构

* Leaf：叶子节点，不能放入其他叶子
* Composite：容器节点，可以放入Leaf和Composite
* Component：使Leaf和Composite具有一致性的父类

### 5.2 装饰模式 Decorator

* component：定义了被装饰的对象的接口
* decorator：实现并组合了component
* concrete component：实现了被装饰component的接口
* concrete decorator：实现decorator的方法，使用委托

## 六、访问数据结构

### 6.1 访问者模式 Visitor

访问者模式基于混合模式的目录，将组件的处理从组件数据结构中分离出来，提高了组件的独立性

* component：访问者模式中增加accept方法调用visitor.visit(this)
* visitor：visitor对混合模式中的leaf和composite各自声明一个visitor接口
* concrete visitor：实现visitor中的接口

易于增加concrete visitor，不易于增加component，因为每个concrete visitor都需要适配新component

### 6.2 职责链模式 Chain of Responsibility

* handler：有next指针，定义了处理请求的接口
* concrete handler：实现处理请求的接口

弱化了调用方和处理方的关系，可以动态改变职责链

## 七、简单化

### 7.1 外观模式 Facade

facade为错综复杂无法解耦的部分提供统一的高层接口

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

## 附录一、延伸阅读