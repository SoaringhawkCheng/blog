---
title: 「深入理解Kafka」学习笔记
catalog: true
date: 2020-05-20 12:08:36
subtitle:
header-img:
tags:
- mq
categories:
- 工程
---
> 书籍豆瓣链接：[《深入理解Kafka：核心设计与实践原理》](https://book.douban.com/subject/30437872/)
> 
> 相关资料笔记：
> [基础篇](https://blog.csdn.net/wk52525/article/details/96985534)
> [进阶篇](https://blog.csdn.net/wk52525/article/details/98070770)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## 第1章 初始Kakfa

### Kafka体系结构

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/kafka-architecture.png?raw=true)

服务代理节点broker：独立的服务节点或kafka实例

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/partition.png?raw=true)

**kafka引入partition解决机器IO性能瓶颈**，topic包含多个partition

partition包含的消息不同，offset是消息在分区中标识，kafka保证消息分区有序

partition是kafka的最小并行操作单元，单个partition数据写入可以并行化，partition消费只能被一个consumer线程消费

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/broker.png?raw=true)

**kafka为分区引入多副本机制，提升容灾能力**

一个topic可以横跨多个broker，每个broker包含全部partition副本

partition的每个副本必须分布在不同的broker上，才能实现有效冗余

producer和consumer只和leader副本进行交互，follower只负责同步，消息相对滞后

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/offset.png?raw=true)

分区中所有副本AR，同步中副本ISR，同步滞后副本OSR，AR=ISR+OSR

**LEO(Log End Offset)** 标识当前分区下一条待写入消息的offset

**HW(High Watermark)** 高水位，标识了一个特定的offset，消费者只能拉渠道这个offset之前的消息（不含HW）

只有ISR集合的副本才有资格被选为leader，ISR集合中最小的LEO即为分区的HW

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/offset-2.png?raw=true)

Kafka使用ISR有效权衡了数据可靠性和性能，既不是完全同步复制，也不是完全异步复制

## 第2章 生产者

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/producer-architecture.png?raw=true)

### 发送模式

**发后即忘(fire-and-forget)** 只管往kafka发送而不关心消息是否正确到达，不对发送结果进行判断处理

**同步(sync)** KafkaProducer.send()返回的是一个Future对象，使用Future.get()来阻塞获取任务发送的结果，来对发送结果进行相应的处理

**异步(async)** 向send()返回的Future对象注册一个Callback回调函数，来实现异步的发送确认逻辑
	
### 分区器

使用key计算消息的分区号
	
### acks参数

acks=1 leader副本成功写入就会收到服务端响应
	
acks=0 producer不需要服务端响应
	
acks=-1 or all ISR所有副本写入才能收到响应

## 第3章 消费者

### 消息消费

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/consumer-group.jpg?raw=true)

一个消费组能订阅topic上所有分区，每个分区只能被一个消费组的一个consumer消费

消息的消费一般是两种模式：推和拉，kafka消息消费是一个不断轮询poll的过程

### 位移提交

消费者位移存储在kafka内部主题__consumer_offsets中

消息位移的提交方式是自动提交，消费者每隔5s将拉取到的分区消息位移进行提交

自动提交会带来重复消费和消息丢失的问题，改为手动提交可以避免

### 多线程实现

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/consumer.png?raw=true)

KafkaProducer是线程安全的，但KafkaConsumer是非线程安全的

每个线程消费一个partition需要建立大量的TCP连接，推荐使用一个线程消费，多线程处理

## 第4章 主题与分区

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/log.png?raw=true)

## 第5章 日志存储

### 文件布局

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/io.png?raw=true)

不考虑多副本的情况，一个分区对应一个日志Log，Log被切分为多个LogSegment

Log在物理上只以文件夹的形式存储，LogSegment对应于磁盘上的一个日志文件和两个索引文件

### 消息压缩

生产者发送的压缩数据在broker中也是保持压缩状态进行存储的，消费者从服务端获取的也是压缩的消息

### 日志索引

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/index.jpeg?raw=true)

### 磁盘存储

Kafka使用mmap进行文件读写，大量使用了页缓存，这是实现高吞吐的重要因素之一

### 零拷贝

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/zero-copy.png?raw=true)

## 第6章 深入服务端

### 时间轮

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/time-wheel.png?raw=true)

### 延时操作

延时操作创建后会被加入延时操作管理器(DelayedOperationPurgatory)做专门处理，每个延时操作管理器配别一个定时器。延时操作除了满足时间条件执行，还支持外部事件触发，由一个监听池来监听每个分区的外部事件

## 第7章 深入客户端

### 事务

#### 消息传输保障

一般消息中间件的消息传输保障有3个层级：

1. at most once：至多一次。消息可能丢失，但不会重复
2. at least once：至少一次。消息不会丢失，但可能重复
3. exactly once：恰好一次。每条消息肯定且仅传输一次

kafka提供的消息传输保障为at least once

#### 幂等

Kafka的幂等只能保证单个生产者会话（session）中单分区的幂等

对于每个PID（producer id），消息发送到的每一个partition都有对应的序列号

broker在内存中为每一对<PID, partition>维护一个序列号，进行对比

#### 事务

kafka的事务可以使应用程序将消费消息，生产消息，提交消费位移当做原子操作来处理，同时成功或失败，即使该生产或消费会跨多个分区

transactionId与PID一一对应，但是transactionId是由用户显式设置，而PID是kafka内部分配。如果使用同一个transactionId开启两个producer，则前一个producer会报错并不再工作

## 第8章 可靠性研究

### 副本剖析

#### 失效副本

同步失效或功能失效的副本成为失效副本，失效副本对应的分区成为同步失效分区（under-replicated）

同步失效判定：根据broker参数 replica.lag.time.max.ms 作为标准，当ISR中的follower副本滞后leader副本时间超过此时间则判定同步失败

#### LEO和HW

多副本消息追加过程：

1. 生产者客户端发送消息至leader副本
2. 消息追加到leader副本的本地日志，并更新日志偏移量
3. follower副本向leader副本请求同步数据
4. leader副本所在的服务器读取本地日志，并更新对应拉取的follower副本信息
5. leader副本所在服务器将拉取结果返回follower副本
6. follower副本收到结果，将消息追加到本地日志，并更新日志的偏移量信息

LEO和HW更新过程：

1. follower向leader拉取消息时，带有自己的LEO信息(fetch_offset)
2. leader更新HW(取leader的HW和follower的LEO中的最小值)，保存副本LEO，返回follower相应消息，并带有自身的HW
3. follower收到新消息后，更新LEO和HW

在一个分区中，leader副本所在的节点会记录所有副本的LEO(用于计算主HW)

而follower副本所在的节点只会记录自身的LEO，而不会记录其他副本的LEO

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/leo-hw-update.png?raw=true)

如上图，从LEO和HW的更新过程可以看到，follower更新完消息1，下一轮fetch才能把HW更新为1

leader和follower的HW更新不是同步的，且滞后于follower消息写入，会造成数据丢失和不一致问题

#### 数据丢失问题

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/data-loss-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/data-loss-2.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/data-loss-3.png?raw=true)

follower副本恢复后会做两件事情：

1. 使用HW截断自身
2. 向leader发送fetch请求

#### 数据不一致问题

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/hw-data-inconsistent-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/hw-data-inconsistent-2.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/hw-data-inconsistent-3.png?raw=true)

### 使用epoch

ack=-1只能解决日志写入同步，epoch解决了HW不同步带来的问题

#### 解决数据丢失

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/epoch-data-loss-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/epoch-data-loss-2.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/epoch-data-loss-3.png?raw=true)

#### 解决数据不一致

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/epoch-data-inconsistent-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/epoch-data-inconsistent-2.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/epoch-data-inconsistent-3.png?raw=true)

### 可靠性分析

ack=-1，保证所有副本日志写入成功后再返回，即使leader宕机，消息也不会丢失

设置min.insync.replicas参数，指定了ISR集合中的最小副本数，不满足条件就会抛出错误

将auto commit设置为false，执行手动位移提交，宁可重复消费也不应该因为消费异常导致消息丢失

死信队列，将一直不能成功被消费的消息暂存到死信队列

## 第9章 Kafka应用

## 第10章 Kafka监控

## 第11章 高级应用

**Kafka不支持重试机制也就不支持消息重试，也不支持死信队列**

**原生kafka并没有提供延时队列、消息轨迹等功能**

### 过期时间

给消息添加timeStamp和超时时间，并在消费时使用拦截器，判断是否超时后进行消费

### 延时队列

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/delay-service.png?raw=true)

先投递到内部主题，按照不同的延时等级来划分的，比如设定5s、10s、30s、1min、2min、5min、10min、20min、30min、45min、1hour、2hour这些按延时时间递增的延时等级

针对不同延时级别的主题，在 DelayService 的内部都会有单独的线程来进行消息的拉取，以及单独的 DelayQueue（这里用的是 JUC 中 DelayQueue）进行消息的暂存

### 死信和重试队列

重试队列一般分成多个重试等级，超过投递次数进入死信队列

### 消息路由

### 消息轨迹

### 消息审计

### 消息代理

### 消息中间件选型

