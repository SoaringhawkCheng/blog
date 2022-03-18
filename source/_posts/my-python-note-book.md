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

### yield关键字
协程和生成器类似，在定义中包含yield关键字的函数

### yield在协程中的用法
```
在协程中yield通常出现在表达式的右边。比如 lhs = yield rhs，或者 lhs = yield(生成器产出None)
协程可能从调用方接受数据，调用方是通过send(datum)的方式把数据提供给协程使用
协程可以把控制器让给中心调度程序，从而激活其他的协程
```
### 协程的用法
`lhs = yield` 或 `lhs = yield rhs`，这个表达式的计算过程是先计算等号右边的内容，然后在进行赋值，所以当激活生成器后，程序会停在yield这里，但并没有给x赋值。

当我们调用send方法后yield会收到这个值并赋值给x，而当程序运行到协程定义体的末尾时和用生成器的时候一样会抛出StopIteration异常

```
#! /usr/bin/python3

from inspect import getgeneratorstate

def simple_coroutine():
    print('-->coroutine started')
    x = yield
    print('-->coroutine received:', x)

my_coro = simple_coroutine()
print(my_coro)
print(getgeneratorstate(my_coro))
print(next(my_coro))
print(getgeneratorstate(my_coro))
print(my_coro.send(24))
print(getgeneratorstate(my_coro))
```

如果协程没有通过next(...)激活(同样我们可以通过send(None)的方式激活)，但是我们直接send会报can't send non-None value to a just-started generator

```
##! /usr/bin/python3

from inspect import getgeneratorstate

def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield average
        total += term
        count += 1
        average = total/count

coro_avg = averager()
print(getgeneratorstate(coro_avg))
next(coro_avg)
print(getgeneratorstate(coro_avg))
print(coro_avg.send(10))
print(getgeneratorstate(coro_avg))
print(coro_avg.send(30))
print(getgeneratorstate(coro_avg))
print(coro_avg.send(40))
print(getgeneratorstate(coro_avg))
```

协程在运行过程中有四个状态：
```
GEN_CREATE:等待开始执行
GEN_RUNNING:解释器正在执行，这个状态一般看不到
GEN_SUSPENDED:在yield表达式处暂停
GEN_CLOSED:执行结束
```

### 预激协程的装饰器
关于调用next(...)函数这一步通常称为”预激(prime)“协程，即让协程向前执行到第一个yield表达式，准备好作为活跃的协程使用

```
##! /usr/bin/python3

from functools import wraps
from inspect import getgeneratorstate

def coroutine(func):
    @wraps(func) # 元信息还是func，不被替换
    def primer(*args, **kwargs):
        gen = func(*args, **kwargs)
        next(gen)
        return gen
    return primer

@coroutine
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield average
        total += term
        count += 1
        average = total/count

coro_avg = averager()
print(getgeneratorstate(coro_avg))
print(coro_avg.send(10))
print(getgeneratorstate(coro_avg))
print(coro_avg.send(30))
print(getgeneratorstate(coro_avg))
print(coro_avg.send(40))
print(getgeneratorstate(coro_avg))
```

### 终止协程和异常处理
协程中为处理的异常会向上冒泡,传给next函数或send函数的调用方(即触发协程的对象)

```
from inspect import getgeneratorstate
from operator import ne


class DemoException(Exception):
    """定义的异常类型"""

def demo_exc_handling():
    print('-> coroutine started')
    while True:
        try:
            x = yield
        except DemoException:
            print("DemoException handled, continuing...")
        else:
            print("-> coroutine received:{!r}".format(x))

    raise RuntimeError("this line should never run.")

ext_coro = demo_exc_handling()
next(ext_coro)
print(getgeneratorstate(ext_coro))
ext_coro.send(11)
print(getgeneratorstate(ext_coro))
ext_coro.send(22)
print(getgeneratorstate(ext_coro))
ext_coro.send(33)
print(getgeneratorstate(ext_coro))
ext_coro.throw(DemoException)
print(getgeneratorstate(ext_coro))
ext_coro.close()
print(getgeneratorstate(ext_coro))
```

从python2.5开始客户端代码在生成器对象上调用throw和close两个方法，显示的把异常发送给协程

* throw：生成器在暂时的yield表达式处抛出指定的异常

```
如果生成器处理了异常，代码会执行到下一个yield表达式，产出的值作为throw方法的返回值
如果没有处理异常，异常会向上冒泡，传到调用方的上下文中
```
	
* close：生成器在暂停的yield表达式处抛出GeneratorExit异常

```
 如果生成器没有处理这个异常，或者抛出了StopIteration异常，调用方不会报错
 收到GeneratorExit异常，生成器一定不能产出值，否则解释器会抛出RuntimeError异常，传给调用方
```

### 协程返回停止返回值 - yield from
```
from collections import namedtuple


Result = namedtuple("Result", "colunt average")

def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None:
            break
        total += term
        count += 1
        average = total/count
    return Result(count, average)

coro_avg = averager()
next(coro_avg)
coro_avg.send(10)
coro_avg.send(30)
coro_avg.send(5)
try:
    coro_avg.send(None)
except StopIteration as e:
    result = e.value
    print(result)
```
上面这种方式获取返回值比较麻烦，使用yield from，解释器不仅会捕获StopIteration异常，还会把value属性的值变成yield from表达式的值

### yield from 
yield from的主要功能是打开双向通道，把最外层的调用方与最内层的子生成器连接起来，这样二者可以直接发送和产出值，还可以直接传入异常，而不用再像之前那样在位于中间的协程中添加大量处理异常的代码
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/my-python-note-book/1.png?raw=true)

```
#! /usr/bin/python3

from collections import namedtuple


Result = namedtuple("Result", "count average")

# 子生成器
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None:
            break
        total += term
        count += 1
        average = total/count
    return Result(count, average)

# 委派生成器
def grouper(result, key):
    while True:
        result[key] = yield from averager()

# 客户端代码， 即调用方
def main(data):
    results = {}
    for key, values in data.items():
        group = grouper(results, key)
        next(group)
        for value in values:
            group.send(value)
        group.send(None)

    report(results)

# 输出报告
def report(results):
    for key, result in sorted(results.items()):
        group, unit = key.split(";")
        print("{} {} averaging {} {}".format(
            result.count, group, result.average, unit
        ))

data = {
    'girls;kg': [40.9, 38.5, 44.3, 42.2, 45.2, 41.7, 44.5, 38.0, 40.6, 44.5],
    'girls;m': [1.6, 1.51, 1.4, 1.3, 1.41, 1.39, 1.33, 1.46, 1.45, 1.43],
    'boys;kg': [39.0, 40.8, 43.2, 40.8, 43.1, 38.6, 41.4, 40.6, 36.3],
    'boys;m': [1.38, 1.5, 1.32, 1.25, 1.37, 1.48, 1.25, 1.49, 1.46],
}

if __name__ == '__main__':
    main(data)
```