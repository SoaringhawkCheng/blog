---
title: 《我的Python学习笔记》 
catalog: true
date: 2022-03-12 17:34:13
subtitle:
header-img:
tags:
- language
categories:
- 工程
> 书籍豆瓣链接：[《流畅的Python》](https://book.douban.com/subject/27028517/)
> 
> 参考资料链接 [Python面经]()
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：
---

[理解 Python 装饰器看这一篇就够了](https://foofish.net/python-decorator.html)

[Python进阶：浅析「垃圾回收机制」(上篇)](https://hackpython.com/blog/2019/07/05/Python%E8%BF%9B%E9%98%B6%EF%BC%9A%E6%B5%85%E6%9E%90%E3%80%8C%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6%E3%80%8D-%E4%B8%8A%E7%AF%87/)

[Python GIL全局解释器锁详解](http://c.biancheng.net/view/5537.html)

## python引用机制
在python中将类型分为了可变类型和不可变类型，分别有：
```
可变类型：列表，字典
不可变类型：int、float、string、tuple
对于不可变类型变量而言：因为不可变类型变量特性，修改变量需要新创建一个对象，形参的标签转而指向新对象，而实参没有变
对于可变类型变量而言，因为可变类型变量特性，直接在原对象上修改，因为此时形参和实参都是指向同一个对象，所以，实参指向的对象自然就被修改了
```

## python传参
python中函数的参数类型分为以下五种：位置参数、默认参数、可变参数`（*args）`、关键字参数`（**args）`、命名关键字参数(这个这次先不复习)

```
fun1(a,b,c) 位置参数
fun2(a=1,b=2,c=3) 默认参数
fun3(*args) 可变参数
fun4(**kargs) 关键字参数

1 按顺序把传给args的实参赋值给对应的行参
2 args = value 形式的实参赋值给行参
3 将多余出的即键值对行后的零散实参打包组成一个tuple传递给*args
4 将多余的key=value形式的实参打包正一个dicrionary传递给**kargs
5 和普通关键字参数不同，命名关键字参数需要一个用来区分的分隔符*，它后面的参数被认为是命名关键字参数 -- 这个先不看
```

## python赋值
直接赋值：其实就是对象的引用（别名）。
浅拷贝(copy)：拷贝父对象，不会拷贝对象的内部的子对象。
深拷贝(deepcopy)： copy 模块的 deepcopy 方法，完全拷贝了父对象及其子对象。

## python命名空间和作用域
[python3命名空间和作用域](https://www.runoob.com/python3/python3-namespace-scope.html)
### 命名空间
命名空间(Namespace)是从名称到对象的映射，大部分的命名空间都是通过 Python 字典来实现的。
```
内置名称（built-in names）， Python 语言内置的名称，比如函数名 abs、char 和异常名称 BaseException、Exception 等等。
全局名称（global names），模块中定义的名称，记录了模块的变量，包括函数、类、其它导入的模块、模块级的变量和常量。
局部名称（local names），函数中定义的名称，记录了函数的变量，包括函数的参数和局部定义的变量。（类中定义的也是）
```
命名空间的生命周期取决于对象的作用域，如果对象执行完成，则该命名空间的生命周期就结束。

### 作用域
作用域就是一个 Python 程序可以直接访问命名空间的文本区域。
在一个 python 程序中，直接访问一个变量，会从内到外依次访问所有的作用域直到找到

四种作用域
```
L（Local）局部作用域：最内层，包含局部变量，比如一个函数/方法内部。
E（Enclosing）嵌套作用域：包含了非局部(non-local)也非全局(non-global)的变量。比如两个嵌套函数，一个函数（或类） A 里面又包含了一个函数 B ，那么对于 B 中的名称来说 A 中的作用域就为 nonlocal。
G（Global）全局作用域：当前脚本的最外层，比如当前模块的全局变量。
B（Built-in）内建作用域： 包含了内建的变量/关键字等，最后被搜索。
```

## python闭包
闭包指延伸了作用域的函数，访问定义体之外的非全局变量，创建一个闭包必须满足一下几点：
```
必须有一个内嵌函数
内嵌函数必须引用外部函数中的变量
外部函数的返回值必须是内嵌函数
```

## 函数式编程
### 函数是一等公民
函数能作为参数传递，或者是作为返回值返回

### 匿名函数(lambda)
lambda内不要包含循环，如果有，定义函数来完成，lambda 是为了减少单行函数的定义而存在的

###  map函数
接收两个参数，一个是函数，一个是Iterable，传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回

### reduce函数
一个函数作用在一个序列上，reduce把结果继续和序列的下一个元素做累积计算 `reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)`

### filter函数
接收一个函数和一个序列，传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素

## 迭代器和生成器
[Python中的可迭代对象、迭代器和生成器的异同点](https://blog.csdn.net/SL_World/article/details/86507872)
### 容器
容器就是一个用来存储多个元素的数据结构，容器中的元素可通过迭代获取。list，tuple，dict，set，str
### 迭代器
Iterator实现了`__iter__()`和`__next__()`方法

### 生成器
含有`yield`关键字的函数就是生成器函数

## 协程
[Python协程深入理解](https://www.cnblogs.com/zhaof/p/7631851.html)
协程在运行过程中有四个状态：
```
GEN_CREATE:等待开始执行
GEN_RUNNING:解释器正在执行，这个状态一般看不到
GEN_SUSPENDED:在yield表达式处暂停
GEN_CLOSED:执行结束
```
x = yield这个表达式的计算过程是先计算等号右边的内容，然后在进行赋值，所以当激活生成器后，程序会停在yield这里，但并没有给x赋值。
当我们调用send方法后yield会收到这个值并赋值给x,而当程序运行到协程定义体的末尾时和用生成器的时候一样会抛出StopIteration异常