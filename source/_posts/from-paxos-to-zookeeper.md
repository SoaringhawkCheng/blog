---
title: 「从Paxos到Zookeeper：分布式一致性原理与实践」学习笔记
catalog: true
date: 2019-08-27 17:00:38
subtitle:
header-img:
tags:
- gateway
categories:
- 工程
---
> 书籍豆瓣链接：[《从Paxos到Zookeeper》](https://book.douban.com/subject/26292004/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

# 前言

## 建议阅读顺序 

I. -ZooKeeper相关: 1、5、6、7章(建议中间穿插第8章阅读). 

II. -分布式一致性协议相关: 2、3、4. 

# 第1章 分布式架构

## 1.1 从集中式到分布式

### 1.1.1 集中式特点

部署结构简单（因为基于底层性能卓越的大型主机，不需考虑对服务多个节点的部署，也就不用考虑多个节点之间分布式协调问题）

### 1.1.2 分布式特点

分布式系统是一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统

分布式的特点：

1. 分布性：在空间随意分布
2. 对等性：没有主从之分，都是对等的
3. 并发性
4. 缺乏全局时钟：很难定义两个事件谁先谁后
5. 故障总是会发生
 
### 1.1.3 分布式环境的各种问题

1. 通信异常：主要是因为网络本身的不可靠性
2. 网络分区：当网络发生异常时，导致部分节点之间的网络延时不断增大，最终导致部分节点可以通信，而另一部分节点不能。
3. 三态（成功、失败与超时）
4. 节点故障：组成分布式系统的服务器节点出现宕机或“僵死”现象

### 1.1.4 分布式事务

分布式事务是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于分布式系统的不同节点之上。也可以被定义为一种嵌套型的事务，同时也具有了ACID事务特性。

## 1.2 一致性理论

### 1.2.1 ACID

### 1.2.2 CAP

一个分布式系统不可能同时满足一致性(Consistency)、可用性(Availability)和分区容错性(Parition tolerance)

#### 一致性

```
A read is guaranteed to return the most recent write for a given client
对某个指定客户端，读操作保证能够返回最新的写操作结果
```

数据在多个副本之间是否能够保持一致的特性。

#### 可用性

```
A non-failing node will return a reasonable response within a reasonable amount of time (no error or timeout)
非故障的节点在合理的时间内返回合理的响应(不是错误和超时的响应)
```

系统提供的服务必须一致处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果。

#### 分区容错性

```
The system will continue to function when network patitions occur
当发生网络分区后，系统能够继续“履行职责”
```

分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供服务，除非是整个网络环境都发生了故障。

#### CAP的矛盾

一个分布式系统不可能同时满足一致性、可用性和分区容错性这三个需求

对于一个分布式系统，P是一个最基本的需求，只能在C和A之间权衡

**CA** 优先保证一致性和可用性，放弃分区容错。这也意味着放弃系统的扩展性，系统不再是分布式的，有违设计的初衷。

**CP** 优先保证一致性和分区容错性，放弃可用性。在数据一致性要求比较高的场合(譬如:zookeeper,Hbase)是比较常见的做法，一旦发生网络故障或者消息丢失，就会牺牲用户体验，等恢复之后用户才逐渐能访问。

**AP** 优先保证可用性和分区容错性，放弃一致性。比如数据库的主从复制，Kafka的主从复制就是这种架构。跟CP一样，放弃一致性不是说一致性就不保证了，而是逐渐的变得一致

### 1.2.3 BASE

是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写。核心思想是即使无法做到强一致性，但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性

**基本可用** 分布式系统在出现不可预知故障的时候，允许损失部分可用性。

**弱状态** 也称软状态，允许系统在不同节点的数据副本之间进行数据同步存在延时。

**最终一致性** 系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。

# 第2章 一致性协议

## 2.1 2PC和3PC

协调者：用来统一调度所有分布式节点的执行逻辑

参与者：被调度的分布式节点

### 2.1.1 2PC
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/2pc.png?raw=true)

#### 流程

* 阶段一：提交事务请求（投票阶段）

	协调者节点发送事务内容，询问是否可以执行事务提交操作
	
	参与者节点执行询问发起为止的所有事务操作，并将Undo信息和Redo信息写入日志
	
	各参与者节点响应协调者节点发起的询问。YES or NO

* 阶段二A：执行事务提交（执行阶段）

	所有询问响应都为“YES”，执行事物提交：
	
	协调者向参与者发送commit请求
	
	参与者节点接受到COMMIT请求后正式完成操作，并释放在整个事务期间内占用的资源
	
	参与者节点向协调者节点发送"ACK”消息
	
	协调者节点收到所有参与者节点反馈的"ACK”消息后，完成事务

* 阶段二B：中断事务（执行阶段）

	如果任一参与者节点在第一阶段返回的响应消息为“NO”，或者协调者节点在第一阶段的询问超时之前无法获取所有参与者节点的响应消息时,那么就会中断事务：
	
	协调者向参与者发送Rollback请求
	
	参与者节点利用之前写入的Undo信息执行回滚，并释放在整个事务期间内占用的资源
	
	参与者节点向协调者节点发送"ACK”消息
	
	协调者节点收到所有参与者节点反馈的"ACK”消息后，中断事务
	
#### 分析

* 优点

	原理简单，实现方便
	
* 缺点

	**同步阻塞问题** 事务阻塞型，锁定事务资源
	
	**单点问题** 阶段二commit没有执行，参与者会处在锁定事务资源的状态
	
	**脑裂** 分区两侧保持可用的状态称为脑裂，阶段二部分commit，会造成网络分区数据不一致
	
	**太过保守** 一个节点失败导致整个事务失败，没有容错机制

### 2.1.2 3PC

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/3pc.jpg?raw=true)

#### 流程

* 阶段一：CanCommit阶段

	协调者向参与者发送CanCommit请求

	参与者响应CanCommit请求 YES or NO？

* 阶段二A：PreCommit阶段 preCommit请求

　　发送预提交请求协调者向参与者发送PreCommit请求，并进入Prepared阶段
	
* 阶段二B：PreCommit阶段 abort请求

* 阶段三A：doCommit阶段 doCommit请求

* 阶段三B：doCommit阶段 abort请求

* 故障情况

	参与者没有收到来自协调者的doCommit或abort请求，会在等待超时后，继续进行事务提交

#### 分析

* 优点

	降低了参与者的阻塞范围，出单单点故障后继续打成一致

* 缺点

	网络分区，参与者依然会进行事务提交，出现数据不一致性

## 2.2 Paxos算法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-architecture.png?raw=true)

**Paxos** 解决的什么问题呢，解决的就是保证每个节点执行相同的操作序列

**Instance** 不断重试2PC，直到最终各方达成一致，最终确定一个值，是Paxos执行的过程，也就是一个Paxos Instance

**Multi Paxos** 重复这个过程，确定一系列值，也就是日志的每一条

### 2.2.1 概念与术语

**Proposer** 提议发起者，处理客户端请求，将客户端的请求发送到集群中，以便决定这个值是否可以被批准

**Acceptor** 提议批准者，负责处理接收到的提议，他们的回复就是一次投票，会存储一些状态来决定是否接收一个值

**Replica** 节点或者副本，分布式系统中的一个server，一般是一台单独的物理机或者虚拟机，同时承担paxos中的提议者和接收者角色

**ProposalId** 每个提议都有一个编号，编号高的提议优先级高

**acceptedProposal** 在一个Paxos Instance内，已经接收过的提议

**acceptedValue** 在一个Paxos Instance内，已经接收过的提议对应的值

**minProposal** 在一个Paxos Instance内，当前接收的最小提议值，会不断更新

**WARO** write all and read one，更新时写所有副本，只有在所有副本中更新才算成功

**Quorum** 当更新操作wi 在N个节点中的W个节点上更新成功，称之为“成功提交的更新操作”

### 2.2.2 Basic Paxos

#### 单点问题

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-single-point.png?raw=true)

Q: 如果acceptor挂掉怎么办？

A: 多个acceptors，选中被大多数acceptors的值

#### 平分提案

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-vote.png?raw=true)

Q: Acceptor只接受第一个到达的值？

A: 如果同时提出提案，不会选中任何值，所以Acceptors必须可以选择多个不同的值

#### 选择冲突

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-conflict-1.png?raw=true)

Q: Acceptor接受所有到达的值？

A: 可能会导致选择多个值，所以一旦一个值被选中，以后的提案必须提出/选择一样的值

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-conflict-2.png?raw=true)

Q: 后提出的提案被现提出的提案覆盖

A: 因此提案必须有序，拒绝旧的提案

#### 提案编号

如何产生唯一的编号呢？在《Paxos made simple》中提到的是让所有的Proposer都从不相交的数据集合中进行选择，例如系统有5个Proposer，则可为每一个Proposer分配一个标识j(0~4)，则每一个proposer每次提出决议的编号可以为5*i+j(i可以用来表示提出议案的次数)

Q: 每个提案的值递增且唯一

A: 时间戳+IP

#### 算法流程

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-basic-paxos.png?raw=true)

* prepare阶段

	**prepare请求** 
	
	**promise响应** 找到已经被选中的值，通过minProposal阻止旧的提案

* accept阶段

	**accept请求** 请求acceptors接受值

	**accepted响应**

#### 提案覆盖

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-accept-value-1.png?raw=true)

旧的值被选中，新的proposer发现并使用它

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-accept-value-2.png?raw=true)

旧的值没有被钻中，但发现被接受了，新的proposer发现并使用它

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-accept-value-3.png?raw=true)

旧的值没有被选中，也没有被发现接受了，新的proposer使用自己的值

#### 算法缺陷

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-live-lock.png?raw=true)

**活锁问题** 旧的提案被接受未被选中时，新的提案已被接受，因此旧提案不能被接受，循环往复

**性能问题** 每选中一个值，至少需要两次RTT+两次写盘

**状态同步** 被选中的日志，状态如何同步给其他机器

### 2.2.3 Multi Paxos

[微信团队PhxPaxos项目的wiki](https://github.com/Tencent/phxpaxos/wiki)
[Paxos相关博客](https://www.cnblogs.com/richaaaard/p/6297104.html)

**Multi Paxos** 需要运行多次Paxos Instance，使用logID标识每条log日志以及对应的Paxos Instance，logID全局唯一且自增

multi-paxos协议中有一个Leader。Leader是系统中唯一的Proposal，在lease租约周期内所有提案都有相同的ProposalId，可以跳过prepare阶段，议案只有accept过程

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos.png?raw=true)

#### 选择日志项

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-select-entry.jpg?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-select-log-entry-1.jpg?raw=true)

#### 提高效率

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-leader-0.png?raw=true)

众所周知Basic Paxos算法的Latency很高，串行运行Basic Paxos效率很差

Multi-Paxos算法希望找到多个Instance的Paxos算法之间的联系，从而尝试在某些情况去掉Prepare步骤

首先我们定义Multi-Paxos的参与要素：

1. 3个参与节点 A/B/C.
2. **Prepare(b)** NodeA节点发起Prepare携带的编号，即**proposalId**
3. **Promise(b)** NodeA节点承诺的编号，即**minProposal**
4. **Accept(b)** NodeA节点发起Accept携带的编号，即**acceptedProposal**

下面图中：

1(A)的意思是A节点产生的编号1，2(B)代表编号2由B节点产生。绿色表示Accept通过，红色表示拒绝。

* A/B/C三节点并行提交

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-leader-1.png?raw=true)

这种情况下NodeA节点几乎每个Instance都收到其他节点发来的Prepare，导致Promise编号过大，迫使自己不断提升编号来Prepare。这种情况并未能找到任何的优化突破口

* 只有A节点提交

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-leader-2.png?raw=true)

这种情况我们会立刻发现，在没有其他节点提交的干扰下，每次Prepare的编号都是一样的。于是乎我们想，为何不把Promised(b)变成全局的？来看下图：

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-leader-3.png?raw=true)

假设我们在Instance i进行Prepare(b)，我们要求对这个b进行Promise的生效范围是Instance[i, ∞)，那么在i之后我们就无需在做任何Prepare了。可想而知，假设上图Instance 1之后都没有任何除NodeA之外其他节点的提交，我们就可以预期接下来Node A的Accept都是可以通过的。那么这个去Prepare状态什么时候打破？我们来看有其他节点进行提交的情况：

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-leader-4.png?raw=true)

Instance 4出现了B的提交，使得Promised(b)变成了2(B), 从而导致Node A的Accept被拒绝。而NodeA如何继续提交？必须得提高自己的Prepare编号从而抢占Promised(b)。这里出现了很明显的去Prepare的窗口期Instance[1,3]，而这种期间很明显的标志就是只有一个节点在提交。

* 优化内容

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-efficiency.jpg?raw=true)

Multi-Paxos通过改变Promised(b)的生效范围至全局的Instance，从而使得一些唯一节点的连续提交获得去Prepare的效果。Leader的作用是希望大部分时间都只有一个节点在提交，这样才能最大发挥Mulit-Paxos的优化效果

#### 选主方法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-leader-election.jpg?raw=true)

实际上大多数系统都不会采用这种选举方式，它们会采用基于租约的方式

#### 减少prepare

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-eliminate-prepare.jpg?raw=true)

#### 复制完整性

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-full-disclosure.jpg?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-full-disclosure-1.jpg?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-full-disclosure-2.jpg?raw=true)

#### 配置变更

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-configuration-change.jpg?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-configuration-change-1.jpg?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/multi-paxos-configuration-change-2.jpg?raw=true)

# 第3章 Paxos的工程世界

# 第4章 Zookeeper与Paxos

## 4.1 初识ZooKeeper

　　ZooKeeper作为一个分布式协调框架，主要用于解决分布式数据一致性的问题。分布式应用程序可以基于它实现诸如数据发布/订阅，分布式锁，Master选举等功能。

### 4.1.1 支持特性

**顺序一致性** 同一客户端发起的事务请求，会严格地按照其发起的顺序应用到ZooKeeper

**原子性** 要么所有节点都成功执行了事务，要么全都没有执行

**单一视图** 无论客户端连接的是哪个ZooKeeper服务器，看到的数据模型都是一样的

**可靠性** 事务引起的服务端状态变更会被一直保留下来

**实时性** ZooKeeper仅仅保证一定时间内，客户端能读到最新数据

### 4.1.2 设计目标

**简单的数据模型** 使得分布式程序能通过一个共享的，树形结构的名字空间来相互协调。其数据模型类似于一个文件系统

**可以构建集群** 集群的每台机器都会在内存中维护服务器状态，并且每台机器互相保持通信，只要有半数以上的机器正常工作，就能正常对外服务

**顺序访问** 每个更新请求，都有有一个全局唯一的递增编号

**高性能** ZooKeeper将全量数据存储在内存中，适用于以读为主的应用场景

### 4.1.3 基本概念

#### 集群角色

ZooKeeper集群的所有机器通过一个Leader选举过程来选定一台被称为Leader的机器，Leader服务器为客户端提供读和写服务

Follower和Observer都能提供读服务，不能提供写服务。两者唯一的区别在于，Observer机器不参与Leader选举过程，也不参与写操作的『过半写成功』策略，因此Observer可以在不影响写性能的情况下提升集群的读性能

#### 会话

一个客户端连接是指客户端和ZooKeeper服务器之间的TCP长连接

通过这个连接，客户端能够通过心跳检测和服务器保持有效的会话，也能够向ZooKeeper服务器发送请求并接受响应，同时还能通过该连接接收来自服务器的Watch事件通知

#### 数据节点

而ZooKeeper中的数据节点是指数据模型中的数据单元，称为ZNode

ZooKeeper将所有数据存储在内存中，数据模型是一棵树(ZNode Tree)，由斜杠(/)进行分割的路径，就是一个ZNode，如/hbase/master,其中hbase和master都是ZNode

每个ZNode上都会保存自己的数据内容，同时会保存一系列属性信息

在ZooKeeper中，ZNode可以分为持久节点和临时节点两类。持久节点需要主动进行ZNode移除操作，临时节点的生命周期与客户端会话绑定

**sequential属性** 节点被创建的时候，ZooKeeper会自动在其节点名后面追加上一个整型数字，这个整型数字是一个父节点维护的自增数字

#### Watcher

ZooKeeper允许用户在指定节点上注册一些Watcher(事件监听器)

并且在一些特定事件触发的时候，ZooKeeper服务端会将事件通知到感兴趣的客户端上去

#### ACL

ZooKeeper采用ACL(Access Control Lists)策略来进行权限控制

## 4.2 ZooKeeper的ZAB协议

ZooKeeper采用ZooKeeper Atomic Broadcast(ZAB 原子消息广播协议)算法

ZooKeeper使用一个单一主进程来处理所有事物请求，并采用ZAB，将服务器数据的状态变更广播到所有副本进程上

### 4.2.1 协议介绍

ZAB包括两种基本模式，分别是崩溃恢复和消息广播

当Leader服务器出现网络中断或者崩溃，ZAB协议会进入恢复模式并选举新的Leader

当集群中过半的Follower完成了Leader的状态同步，整个服务框架就可以进入消息广播模式了

如果非Leader节点收到客户端的事务请求，会转发给Leader

### 4.2.2 消息广播

#### 取消中断

在ZAB的二阶段提交过程中，移除了中断逻辑

意味着可以在过半节点反馈ACK之后提交事务

需要崩溃恢复模式来解决Leader崩溃带来的数据不一致问题

#### 顺序一致性

在消息广播的过程中，Leader会为每一个事务Proposal分配一个全局递增唯一的ID

Leader会为每个Follower分配一个单独队列，按zxid的先后顺序以FIFO的策略进行消息广播

Follower在接收到这个事务的Proposal之后，会将其以事务日志的形式写入到磁盘中去，并向Leader反馈ACK

当Leader收到半数节点的ACK时，会广播一个commit消息通知其他节点进行事务提交，同时自己也完成事务提交

### 4.2.3 崩溃恢复

Leader选举算法应该保证：已经在Leader上提交的事务最终也被其他节点都提交，即使出现了Leader挂掉，Commit消息没发出去这种情况。

确保丢弃只在Leader上被提出的事务。Leader提出一个事务后挂了，集群中别的节点都没收到，当挂掉的节点恢复后，要确保丢弃那个事务。

让Leader选举算法能够保证新选举出来的Leader拥有最大的事务ID的Proposal。 

### 4.2.4 数据同步

在完成Leader选举后，Leader会首先确认事务日志中所有Proposal是否都被集群中过半的节点提交了，即是否完成数据同步。

### 4.2.5 事务

是一个64位数，低32位是一个单调递增的计数器，每一个新的Proposal产生，该计数器都会+1.

高32位代表Leader周期epoch编号，每当选举出新Leader，会取得这个Leader的最大事务ID，对其高32位+1.低32位清零。

ZAB通过epoch来区分Leader周期变化的策略，简化和提升了数据恢复的流程。

### 4.2.6 心跳检测 

Leader和Follower通过心跳检测来感知彼此的情况。

如果Leader在指定时间内无法从过半的Follower那收到心跳检测，或者是TCP连接本身断开了，会重新选举Leader。

心跳是从follower向leader发送心跳

# 第5章 使用Zookeeper

# 第6章 ZooKeeper的典型应用场景

[zookeeper实战系列](https://segmentfault.com/a/1190000012185452)

## 6.1 典型应用场景及实现

### 6.1.1 数据发布/订阅

数据发布/订阅系统，即所谓的配置中心。发布/订阅系统有两种模式：推模式和拉模式

1. 推模式：服务端主动将数据更新发送给所有订阅的客户端
2. 拉模式：客户端主动发起请求来获取最新数据

ZooKeeper采用推拉结合的方式：客户端向服务端注册自己需要关注的节点，一旦节点的数据发生变更，那么服务端就会向相应的客户端发送Watcher事件通知。客户端收到这个通知之后，需要主动到服务端获取最新的数据

### 6.1.2 负载均衡 
　　
ZooKeeper实现动态DNS方案(Dynamic DNS, DDNS)，应用从域名节点获取一份IP地址和端口的配置，进行自行解析

每个应用还会在域名节点注册一个数据变更Watcher监听，以便及时收到域名变更的通知

### 6.1.3 命名服务

客户端应用能够根据指定名字来获取资源或服务的地址，提供者等信息。被命名的实体通常可以是集群中的机器，提供的服务，远程对象等等——这些我们都可以统称他们为名字（Name）。

其中较为常见的就是一些分布式服务框架（如RPC、RMI）中的服务地址列表。通过在ZooKeepr里创建顺序节点，能够很容易创建一个全局唯一的路径，这个路径就可以作为一个名字。

ZooKeeper的命名服务即生成全局唯一的ID

### 6.1.4 分布式协调/通知

基于ZooKeeper实现分布式协调与通知功能，将分布式协调的职责从应用中分离出来

不同客户端对ZooKeeper上同一个数据节点进行Watcher注册，监听数据节点变化

### 6.1.5 集群管理

利用ZooKeeper两大特性

### 6.1.6 Master选举

ZooKeeper将会保证客户端无法重复创建一个已经存在的数据节点

客户端定时往ZooKeeper上创建临时节点，比如`/master_election/2000-01-01/binding`

只有一个客户端能成功创建这个节点，这个客户端称为master

其他客户端在节点`/master_election/2000-01-01`上注册子节点变更的watcher

一旦发现master机器挂了，其余的客户端将会重新进行master选举

### 6.1.7 分布式锁

分布式锁是控制分布式系统之间同步访问共享资源的一种方式

#### 排他锁

* 定义锁

	通过ZooKeeper上的一个数据节点来表示一个锁，例如`/exclusive_lock/lock`节点就可以被定义为一个锁

* 获取锁

	在需要获取锁时，所有客户端都会试图调用create（）接口，在`/exclusive_lock`节点下创建临时节点`/exclusive_lock/lock`

	最终只有一个客户端能成功，就可以认为该客户端获取了锁。其他没获取锁的客户端会在/exclusive_lock节点上注册监听，实时监听lock节点的变更情况

* 释放锁

	获取锁的客户端宕机了，临时节点会被移除

	正常执行完业务逻辑后，客户端会主动删除临时节点

#### 共享锁

* 定义锁

	通过ZooKeeper上的一个临时顺序节点来表示一个锁
	
	例如`/shared_lock/host1-R-000001`

* 获取锁

	在需要获取锁时，所有客户端都会到`/shared_lock`下创建临时顺序节点。

	如果是读请求，就创建例如`/shared_lock/host1-R-000001`的节点

	如果是写请求，就创建例如`/shared_lock/host2-W-000002`的节点

	写操作必须在当前没有任何事务进行读写操作的情况下进行。

	对于读请求，如果比自己序号小的节点中有写请求，就进入等待。

	对于写请求，如果自己不是序号最小的子序号，就进入等待。

* 释放锁

	和排他锁一样
	
* 羊群效应

　　客户端无端收到过多和自己不相关的事件通知

　　每个节点应该只需要关注比自己小的那个节点的变更。而不需要关心全局

* 改进

	读请求：向比自己序号小的最后一个写请求节点，注册监听
	
	写请求 向比自己序号小的节点注册监听

### 6.1.8 分布式队列

#### FIFO

所有客户端都会到节点`/queue_fifo`下面创建一个临时顺序节点

如果自己不是序号最小的节点对比自己序号小的最后一个节点，注册watcher监听

如果是，执行操作后删除节点

#### Barrier 分布式屏障

通过在Zookeeper设置栅栏节点实现Barrier，节点的名字我们叫做`/zookeeper/barrier`

1. 客户端在Barrier节点上调用exists()方法，并设置观察器
2. 如果Barrier节点不存在，则线程继续执行
3. 如果Barrier节点存在，则线程等待，直到触发观察器事件，并且事件是Barrier节点消失的事件，唤起线程

# 第7章 ZooKeeper技术内幕

# 第8章 ZooKeeper运维