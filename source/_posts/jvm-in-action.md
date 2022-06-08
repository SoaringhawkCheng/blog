---
title: 「实战Java虚拟机：JVM故障诊断与性能优化」[DOING]
catalog: true
date: 2021-11-06 19:57:01
subtitle:
header-img:
tags:
- os
categories:
- 工程
---
> 书籍豆瓣链接：
> [《实战Java虚拟机》](https://book.douban.com/subject/34441840/) 
> [《深入理解Java虚拟机》](https://book.douban.com/subject/34907497/)
> 
> 相关资料笔记：[参考笔记](https://blog.csdn.net/gaofubo/article/details/106110276)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## 一、初探Java虚拟机

## 二、Java内存模型

### 运行时数据区域

#### Java虚拟机栈

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/jvm-stack.png?raw=true)

#### Java堆

java堆中的的Eden区可以划分出多个线程私有的缓冲区(Thread Local Allocation Buffer, TLAB)

#### 方法区

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/marea17.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/marea18.png?raw=true)

#### 直接内存

NIO是基于Channel和Buffer的I/O方式，直接用native方法分配native内存，通过DirectByteBuffer引用这块内存，避免了在java堆和native对之间复制数据

#### 运行时常量池

静态常量池中的内容，在类加载后会被存放到方法区的运行时常量池中；字符串常量池也存在运行时常量池之中，String.intern()会把new出来的字符串添加到常量池中

### 对象内存分配

#### 内存分配方式

分配方式有`指针碰撞`和`空闲链表`，取决于Java堆是否规整也就是GC是否有压缩整理功能

并发问题采用两种解决方式：CAS+失败重试、TLAB，优先使用TLAB

#### 内存布局

对象内存中包括三部分：对象头，实例数据和对齐填充

* 对象头包括两部分信息，第一部分用于存储对象自身的运行时数据（哈希码、GC 分代年龄、锁状态标志等等），另一部分是类型指针，即对象指向它的类元数据的指针
* 实例数据部分是对象真正存储的有效信息
* Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍

#### 内存引用

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/object-handler.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/object-pointer.png?raw=true)

使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。

## 三、常见虚拟机参数

## 四、垃圾回收的概念与算法

## 五、垃圾收集器和内存分配

## 六、性能监控工具

## 七、分析Java堆

## 八、锁与并发

## 九、Class文件结构

## 十、Class装载系统

[笔记](https://note.youdao.com/ynoteshare/index.html?id=9cc3963222a194e90f1ac8a08ed7dd45&type=note&_time=1654517637002)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/life-cycle.png?raw=true)

### 触发

一个类或接口在初次使用前，必须进行初始化：

* 当创建一个类的实例时，比如使用new关键字或者反射、克隆、反序列化
* 当调用类的静态方法时，即使用了字节码invokestatic指令
* 当使用类或接口的静态字段时（final修饰、编译期放入常量池的静态字段除外，且是定义字段的类），比如使用getstatic或者putstatic指令
* 当使用java.lang.reflect包中的方法反射类的方法时
* 当初始化子类时，要先初始化父类
* 作为启动虚拟机，含有main()方法的那个类
* 一个接口定义了jdk8新加入的默认方法，实现类初始化时，接口也要初始化
* jdk7动态语言相关的。。。具体见《深入理解Java虚拟机》

其他一些特殊情况：

* 使用使用final常量修饰静态字段，类不会初始化
* 使用父类的静态字段，子类不会初始化，至于是否加载和验证取决于具体实现

### 加载

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/class-load.png?raw=true)

### 链接

#### 验证

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/link.png?raw=true)

语法分析的任务是判断源程序在结构上是否正确，是上下文无关的；

语义分析的任务是判断结构正确的源程序所表达的意义是否正确，是上下文有关的。

#### 准备

为类中定义的静态变量创建零值，但是对于final静态变量，准备阶段会直接设置value

#### 解析

### 初始化

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，

### 10.1 class loader分类
String.class.getClassLoader()

#### Bootstrap classLoader 

启动类加载器 负责的目录如下:

* 环境变量 classpath
* -cp
* 系统属性java.class.path

#### ExtClassLoader 

扩展类加载器 负责的目录如下:

* %JAVA_HOME%/jre/lib/ext
* 系统属性java.ext.dirs指定的类库

#### AppClassLoader(SystemClassLoader)

系统类加载器 负责的目录如下:

* %JAVA_HOME%/jre/lib
* -Xbootclasspath 参数指定的目录
* 系统属性sun.boot.class.path

#### 10.1.1 自定义ClassLoader 

自定义类加载器用于加载用户自定义路径下的类包

### 双亲加载机制





## 十一、字节码执行

[cms](https://blog.csdn.net/wangyy130/article/details/88758055)
