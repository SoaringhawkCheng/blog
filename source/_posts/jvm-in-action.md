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
> 书籍豆瓣链接：[《实战Java虚拟机》](https://book.douban.com/subject/34441840/)
> 
> 相关资料笔记：[参考笔记](https://blog.csdn.net/gaofubo/article/details/106110276)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## 一、初探Java虚拟机

## 二、认识Java虚拟机的基础结构

## 三、常见虚拟机参数

## 四、垃圾回收的概念与算法

## 五、垃圾收集器和内存分配

## 六、性能监控工具

## 七、分析Java堆

## 八、锁与并发

## 九、Class文件结构

## 十、Class装载系统

[笔记](https://note.youdao.com/ynoteshare/index.html?id=9cc3963222a194e90f1ac8a08ed7dd45&type=note&_time=1654517637002)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/class-load.png?raw=true)

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
