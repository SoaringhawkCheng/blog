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

# 一、初探Java虚拟机

[线程](https://www.jianshu.com/p/b0fc3d4bda5a)

# 二、Java内存模型

## 2.1 运行时数据区域

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/marea17.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/marea18.png?raw=true)

### 2.1.1 Java栈

#### 2.1.1.1 虚拟机栈

栈帧中都拥有：局部变量表、操作数栈、动态链接、方法返回地址

* 局部变量表：存放了编译期可知的各种数据类型、对象引用
* 操作数栈：方法执行过程中产生的中间计算结果、临时变量
* 动态链接主要服务一个方法需要调用其他方法的场景。在 Java 源文件被编译成字节码文件时，所有的变量和方法引用都作为符号引用（Symbilic Reference）保存在 Class 文件的常量池里。当一个方法要调用其他方法，需要将常量池中指向方法的符号引用转化为其在内存地址中的直接引用。动态链接的作用就是为了将符号引用转换为调用方法的直接引用。

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/jvm-stack.png?raw=true)

#### 2.1.1.2 本地方法栈

虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一

### 2.1.2 Java堆

此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存

JDK 1.7 开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存

java堆中的的Eden区可以划分出多个线程私有的缓冲区(Thread Local Allocation Buffer, TLAB)

### 2.1.3 直接内存

NIO是基于Channel和Buffer的I/O方式，直接用native方法分配native内存，通过DirectByteBuffer引用这块内存，避免了在java堆和native对之间复制数据

### 2.1.4 方法区

方法区会存储已被虚拟机加载的 类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码缓存等数据

运行时常量池、方法区、字符串常量池这些都是不随虚拟机实现而改变的逻辑概念，是公共且抽象的，Metaspace、Heap 是与具体某种虚拟机实现相关的物理概念，是私有且具体的。

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/method-area-jdk1.6.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/method-area-jdk1.7.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/method-area-jdk1.8.png?raw=true)

#### 2.1.4.1 版本变化

JDK1.7 之前，字符串常量池存放在永久代。JDK1.7 字符串常量池和静态变量从永久代移动了 Java 堆中，主要是因为永久代（方法区实现）的 GC 回收效率太低，只有在整堆收集 (Full GC)的时候才会被执行 GC

JDK 8 版本之后 PermGen(永久) 已被 Metaspace(元空间) 取代，作为方法区的实现，元空间使用的是直接内存。整个永久代有一个 JVM 本身设置的固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

#### 2.1.4.2 常量池

jvm中常量池分三种：

* 类文件常量池（静态常量池）

Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有用于存放编译期生成的各种字面量（Literal）和符号引用（Symbolic Reference）的 常量池表(Constant Pool Table) 

字面量是源代码中的固定值的表示法，即通过字面我们就能知道其值的含义。字面量包括整数、浮点数、字符串字面量和声明成final的常量

符号引用包括类符号引用、字段符号引用、方法符号引用和接口方法符号引用。

* 运行时常量池

类的常量池表会在类加载后存放到方法区的运行时常量池中

还会把由符号引用翻译出来的直接引用也存储在运行时常量池中

并不要求常亮一定在编译期产生，运行期间也可以将新的常量放入常量池中

* 字符串常量池

JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

HotSpot 虚拟机中字符串常量池是StringTable，底层的实现是一个hashSet，保存的是字符串对象的引用，字符串对象的引用指向堆中的字符串对象

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/string-pool.png?raw=true)

```
String str1 = "abcd"; //先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd"；

String str2 = new String("abcd"); //堆中创建一个新的对象

String str3 = str2.intern(); 
// 是一个Native方法，它的作用是：
// 如果运行时常量池中已经包含一个等于此String对象内容的字符串，则返回常量池中该字符串的引用；
// 如果没有，JDK1.7之前（不包含1.7）的处理方式是在常量池中创建与此String内容相同的字符串，
// 并返回常量池中创建的字符串的引用
// JDK1.7以及之后的处理方式是在常量池中记录此字符串的引用，并返回该引用

```

## 2.2 hotspot虚拟机对象

### 2.2.1 对象的创建

#### 2.2.1.1 类加载检查

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

#### 2.2.1.2 分配内存

内存分配方式

分配方式有`指针碰撞`和`空闲链表`，取决于Java堆是否规整也就是GC是否有压缩整理功能

并发问题采用两种解决方式：CAS+失败重试、TLAB，优先使用TLAB

#### 2.2.1.3 初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

#### 2.2.1.4 设置对象头


#### 2.2.1.5 执行 init 方法 

执行 new 指令之后会接着执行 <init> 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

### 2.2.2 对象内存布局

对象内存中包括三部分：对象头，实例数据和对齐填充

* 对象头包括两部分信息

第一部分用于存储对象自身的运行时数据（哈希码、GC 分代年龄、锁状态标志等等）

另一部分是类型指针，即对象指向它的类的元数据的指针

* 实例数据部分是对象真正存储的有效信息
* Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍

### 2.2.3 对象内存引用

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/object-handler.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/object-pointer.png?raw=true)

使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。

# 三、常见虚拟机参数

## 3.1 jvm参数分类

* 标准参数（-）：所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
* 非标准参数（-X）：默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
* 非Stable参数（-XX）：此类参数各个jvm实现会有所不同（用的最多：JVM调优），将来可能会随时取消，需要慎重使用；
* -D：是jvm启动时给系统参数赋值用的(可以是系统默认有的参数，也可以是自己定义的参数)，这个过程会在jvm开始java应用程序之前执行。这个参数赋值也可以通过使用System.setProperty(key, value)；来完成。

## 3.2 


# 四、垃圾收集器和内存分配

[haha](http://www.voycn.com/article/guanyugcxiacmsheg1gcdebijiao)

## 4.1 垃圾回收的概念与算法

### 4.1.1 死亡对象判断方法

#### 4.1.1.1 引用计数法

方法实现简单，效率高，很难解决对象之间相互循环引用的问题

####  4.1.1.2 可达性分析算法

通过一系列的称为 “GC Roots” 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的，需要被回收。

哪些对象可以作为 GC Roots 呢？

* 虚拟机栈(栈帧中的本地变量表)中引用的对象
* 本地方法栈(Native 方法)中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 所有被同步锁持有的对象

#### 4.1.1.3 可触及性

对象可以被回收，就代表一定会被回收吗？事实上，一个无法被触及的对象有可能在某个条件下使自己“复活”，如果是这样的情况则不应该回收。可触及性包含以下3种状态：

* 可触及的：从根节点开始，可以到达这个对象。
* 可复活的：对象的所有引用都被释放，但是对象有可能在finalize()函数中复活。
* 不可触及的：对象的finalize()函数被调用，并且没有复活，那么就会进入不可触及状态，不可触及的对象不可能被复活，因为finalize()函数只会被调用一次。只有对象不可触及时才会被回收。

#### 4.1.1.4 引用类型

暂时略

#### 4.1.1.5 回收方法区

方法区的垃圾收集主要回收两部分内容：废弃的常量和不再使用的类型

* 废弃常量判断

与回收堆中对象类似

* 无用类判断

该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。

加载该类的 ClassLoader 已经被回收。

该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

## 4.1.2 Hotspot具体实现

### 4.1.2.1 根节点枚举

GC开始的时候，就通过OopMap这样的一个映射表知道，在对象内的什么偏移量上是什么类型的数据，而且特定的位置记录下栈和寄存器中哪些位置是引用。

但是可能导致引用关系变化或OopMap内容变化的指令非常多，每一条指令都生成OopMap成本非常高，只是在称为安全点的特定位置才会生成OopMap，并且程序只有在安全点才能暂停开始垃圾收集

### 4.1.2.2 安全点

Safe Point 的选择很重要，如果太少可能导致GC等待的时间太长，如果太频繁可能导致运行时的性能问题。大部分指令的执行的时间都非常短暂，通常会根据“是否具有让程序长时间执行的特征”为标准。比如：选择一些执行时间较长的指令作为Safe Point，如方法调用、循环跳转和异常跳转等

### 4.1.2.3 安全区域

例如线程Sleep 状态或Blocked 状态，这时候线程无法响应JVM的中断请求，“走”到安全点去中断挂起，JVM也不太可能等待线程被唤醒，对于这种情况，就需要安全区域（Safe Region）来解决。

安全区域是指在一段代码片段中，对象的引用关系不会发生变化，在这个区域中的任何位置开始GC都是安全的。我们也可以把Safe Region 看做是被扩展了的SafePoint。

#### 4.1.2.4 记忆集

使用记忆集来缩减部分区域收集时的GC Roots扫描范围，但没有解决卡表如何维护的问题

#### 4.1.2.5 写屏障

虚拟机层面对引用类型字段赋值动作的切面，分写前屏障和写后屏障

#### 4.1.2.6 并发可达性分析

三色标记法


## 4.3 经典垃圾收集器

### 4.3.1 串行收集器

两个特点：1）仅仅使用单线程进行垃圾回收。2）独占式的垃圾回收方式，分为新生代串行回收器和老年代串行回收器。新生代串行回收器使用复制算法，老年代串行回收器使用的是标记压缩法。

### 4.3.2 并行回收器
#### 4.3.2.1 ParNew收集器

将串行回收器多线程化，回收策略、算法及参数和新生代串行回收器一样

#### 4.3.2.2 Parallel Scavenge收集器

新生代收集器，采用标记-复制算法，关注点是吞吐量（高效率的利用 CPU），就是 CPU 中用于运行用户代码的时间与 CPU 总消耗时间的比值

#### 4.3.2.3 Parallel Old 收集器

ParallelOldGC回收器使用标记压缩法

### 4.3.3 CMS收集器

### 4.3.4 G1收集器

### 4.2.4 ZGC收集器

## 4.3 


## 5.4 G1




# 六、性能监控工具

# 七、分析Java堆

jhat虚拟机堆转储快照分析工具

## 7.1 oom异常

见《深入理解jvm虚拟机》p54



# 八、锁与并发

# 九、Class文件结构

# 十、Class装载系统

[笔记](https://note.youdao.com/ynoteshare/index.html?id=9cc3963222a194e90f1ac8a08ed7dd45&type=note&_time=1654517637002)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/life-cycle.png?raw=true)

## 流程

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

加载类时，Java虚拟机必须完成一下工作：

* 通过类的全名获取类的二进制数据流。
* 解析类的二进制数据流为方法区内的数据结构。
* 创建java.lang.Class类的实例，表示该类型。


### 链接

#### 验证

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/link.png?raw=true)

语法分析的任务是判断源程序在结构上是否正确，是上下文无关的；

语义分析的任务是判断结构正确的源程序所表达的意义是否正确，是上下文有关的。

#### 准备

为类中定义的静态变量创建零值，但是对于final静态变量，准备阶段会直接设置value

#### 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，直接引用与虚拟机的内存布局直接相关

### 初始化

初始化阶段的重要工作是执行类的初始化方法<clinit>。方法<clinit>是由编译器自动生成的，它是由类静态成员的赋值语句及static语句块共同产生的。

<clinit>方法是带锁安全的，所以多线程下进行类初始化时，可能会引起死锁，如下面代码所示
```
class StaticA {
    static {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            Class.forName("org.example.StaticB");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println("StaticA init ok!");
    }
}

class StaticB {
    static {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            Class.forName("org.example.StaticA");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println("StaticB init ok!");
    }
}
```

## 加载机制

### 类加载器

#### Bootstrap classLoader

启动类加载器，负责的目录如下: 

* 环境变量 classpath
* -cp
* 系统属性java.class.path

#### ExtClassLoader
扩展类加载器 负责的目录如下:

* %JAVA_HOME%/jre/lib/ext
*系统属性java.ext.dirs指定的类库

#### AppClassLoader(SystemClassLoader)

系统类加载器 负责的目录如下:

* %JAVA_HOME%/jre/lib
* -Xbootclasspath 参数指定的目录
* 系统属性sun.boot.class.path

#### 自定义ClassLoader 

自定义类加载器用于加载用户自定义路径下的类包

### 双亲委托模式

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/classloader.png?raw=true)

String.class.getClassLoader()获取对应的类加载器，如果得到的是null，那么这个类的类加载器是启动类加载器

查找和委托都是单向的，上层的ClassLoader无法访问下层的ClassLoader所加载的类。`getContextClassLoader()`和`setContextClassLoader(ClassLoader cl)`两个方法分别是取得设置在线程中的上下文加载器和设置一个线程的上下文加载器

### 热替换

两个不同ClassLoader加载同一个类，虚拟机内部会认为这两个类是完全不同的

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/jvm-in-action/hotswap.png?raw=true)



# 十一、字节码执行

[cms](https://blog.csdn.net/wangyy130/article/details/88758055)
