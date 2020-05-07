---
title: 「Redis设计与实现」学习笔记[DOING]
catalog: true
date: 2019-12-24 14:38:09
subtitle: 
tags: 
- cache
categories:
- 工程
---
> 书籍豆瓣链接：[《Redis设计与实现》](https://book.douban.com/subject/25900156/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

# 数据结构与对象

## 简单动态字符串sds
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/1.png?raw=true)


```
1. 记录字符串长度len，常数时间将获得字符串长度
2. 杜绝缓冲区溢出，对sds修改时先检查空间是否满足修改所需的要求
3. 减少内存重分配次数
	空间预分配，空间扩展的时候，额外分配内存len(sds)>=1MB?1MB:len(sds)
	惰性空间释放，free属性记录空闲空间，提供api释放空间
4. 二进制安全，sds api以二进制的方式处理sds存放在buf数组里的数据
```
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/2.png?raw=true)
## 链表list

```
双端，prev和next指针 
无环，prev和next指针指向null
带表头表尾指针，head tail
带链表长度计数器，len
多态，通过void*指针来实现
```
## 字典dictht
hashtable

```
槽数量size，掩码sizemask=size-1，used槽使用数量,
负载因子 load_factor=used/size
哈希表ht包含两个hashtable，ht[1]用于rehash
```
rehash

```
时间
1. if BGSAVE和BGREWRITEAOF && 负载因子>=5
2. if not BGSAVE和BGREWRITEAOF && 负载因子>=1
3. 负载因子<=0.1
分配空间
1. 扩展操作，ht[1].size=第一个大于等于ht[0].used*2的2**n
2. 收缩操作，ht[1].size=第一个大于等于ht[0].used的2**n
```

渐进式rehash

```
步骤
1. 新分配的一个table ht[1]，维持一个索引计数器rehashidx，初始值为0，rehash工作正式开始
2. 每次对字典执行crud，将当前rehashidx索引上的所有键值对rehash，结束后rehashidx增一
3. rehash完成时，rehashidx属性的值-1
期间
1. create ht[1]
2. delete ht[0]和ht[1]都删除
3. retrieve 先ht[0]后ht[1]
4. update ht[0]和ht[1]都更新
```

### 
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/3.png?raw=true)

## 跳跃表skiplist

表数据结构

```
head tail
level 层数
length 跳跃表长度
头结点 长度为32的level数组
```

节点数据结构

```
level数组
	长度1-32随机
	前进指阵，跨度，跨度用于计算rank
backward指针
分值
成员
```

## 整数集合intset

```
contents 申明为int8数组
encoding 保存数组真正类型
contents item按从小到大排列
```

## 压缩列表ziplist

表数据结构

```

```

节点数据结构

```
previous_length 1 or 5字节
encoding 
content 
```

## 对象
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/4.png?raw=true)

对象数据结构

```
type
encoding
ptr 指向底层实现
```

字符串对象

```
int ptr指向long
raw
embstr object和sdshdr在连续空间，只读，如果要写，从embstr转换成raw
```

列表对象

```

```

对象共享

```
初始化服务器，创建0-9999，1万个对象
```

# 单机数据库
## 数据库
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/5.png?raw=true)

数据库

```
redisDb *db 维护一组数据库 select 下标选择
```

键空间

```
读写键空间的维护操作

读
更新键空间hit和miss次数
更新键的LRU时间
发现键过期，删除过期键

写
如果客户端用WATCH命令监视这个键，将键标记为dirty
修改键后dirty计数增1
```

过期删除策略

```
定时删除：设置键过期时间同时，创建定时器，定时删除
惰性删除：每次从键空间获取键，如果过期就删除，没过期就返回
定期删除：每隔一段时间，批量删除过期键
```

过期键处理

```
RDB
生成RDB文件，已过期的键不会被保存
载入RDB文件，主服务器

AOF
# 过期键不会对AOF写入和重写造成影响

复制
# 由主服务统一删除过期键，并向从服务器发送DEL链接

```

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/6.png?raw=true)
## RDB持久化

```
BGSAVE派生出一个子进程，然后子进程负责创建RDB文件
因为写时复制，子进程带有服务器进程的数据副本，可以在避免使用锁的情况下，保证数据安全性
服务器启动，AOF更新频率高，优先使用RDB文件还原数据库
serviceCon每个一段时间执行一次，根据dirty和lastsave是否满足条件，执行BGSAVE
```

## AOF持久化

AOF持久化步骤

```
1. 命令追加
服务器实时将写命令追加到服务器状态的aof_buf缓冲区末尾

2. 文件写入
每次结束一个事件循环，会调用flushAppendOnlyFile，考虑是否将aof_buf写入到AOF文件

3. 文件同步
write只会更新页缓存，脏页的更新取决于os，系统提供fsync和fdatasync两个同步函数
appendfsync配置
	always 每个事件循环写入并同步
	everysec 每个事件循环写入，每个1s同步
	no 每个事件循环写入，同步由os
```

AOF载入与数据还原

```
创建一个不带网络链接的伪客户端
从AOF文件中分析并执行写命令
```

AOF重写步骤

```
跟RDB一样，也是使用子进程进程后端AOF重写

1. 设置重写缓冲区，服务端进程在执行客户端命令的时候，同时往AOF缓冲区和重写缓冲区追加命令
2. 重写完成后，子进程向父进程发送信号，父进程调用信号处理函数(中断)，将重写缓冲区写入新AOF文件
3. 对新AOF文件改名，原子的覆盖旧文件
```

## 事件
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/7.png?raw=true)

文件事件

```
reactor模式，I/O多路复用+单进程，简单，避免上下文切换带来的消耗
产生事件的套接字放到队列，每次把一个套接字传送给文件事件分派器
套接字编程可读或acceptable时，产生READABLE事件，编程可写，产生WRITEABLE事件，
允许同时监听READABLE和WRITEABLE事件，先读后写
具体看源码
```

时间事件

```
分为定时事件和周期性事件，redis只是用周期性事件
一般情况下只执行serverCron一个时间事件
调度规则
1. 事件循环的最大阻塞时间由最接近当前时间的时间事件决定
2. 先处理所有已产生的文件事件，再处理所有已产生的时间事件
```

## 客户端
服务器为连接的客户端保存了client列表

客户端属性
```
套接字描述符fd
fd=-1 伪客户端，一个载入AOF文件，一个执行lua脚本
flags 客户端状态
querybuf 保存客户端命令
argv argc 服务器将querybuf解析到argv，argc中
cmd 根据argv指向对应命令实现函数
buf 固定输出缓冲区
reply 可变输出缓冲区
```

客户端创建

```
使用connect函数连接到服务器
```

客户端关闭

```
客户端进程退出或杀死
设置了timeout，空转客户端
命令不符合协议格式或超出输入缓冲区大小
命令回复超出输入缓冲区大小
```

## 服务器
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/8.png?raw=true)

命令执行器step4 执行后续操作

```
if 开启慢查询 检查是否添加慢查询日志
更新cmd对应实现函数统计数据
if aof 将命令写入到aof缓冲区
if 有从服务器复制当前主服务器 将命令发给从服务器
```

命令回复给客户端

```
将命令回复保存到客户端输出缓冲区，为客户端套接字关联命令回复处理器
```

**TODO** 套接字什么时候可写

# 多机数据库
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/9.png?raw=true)
## 复制

旧版复制功能：同步和命令传播

```
同步
	1. 从服务器发送SYNC命令
	2. 主服务器BGSAVE，生成RDB文件，然后用一个缓冲区开始记录后续的执行命令
	3. RDB文件发给从服务器，从服务器加载命令
	4. 主服务器将缓冲区的写命令发送给从服务器
命令传播
	同步操作完毕后，服务器会将自己执行的命令发送给从服务器
问题缺陷
	不能高效处理断线重复制
```

新版复制功能：完整重同步，部分重同步

```
主从服务器分别维护一个复制偏移量offset
复制积压缓冲区，命令传播时将命令写入复制挤压缓冲区里面，并为每个字节记录偏移量
从服务器通过PSYNC将offset发给主服务器
如果offset后面的数据在积压缓冲区，进行部分重同步，如果不在，进行完整全同步
```

心跳检测REPLCONF

```
从服务器1s一次，向主服务器发送命令：
检测主从服务器的网络连接状态
检测命令丢失，通过发送offset
```

## 哨兵(高可用方案)

故障转移

```
sentinel是redis的高可用方案
当前server1下线时长超过用户设定的下线时长上限，sentinal就会对server1执行故障转移
1. 挑选server1下的一个从服务器，并将被选中的从服务器升级为新的主服务器
2. sentinel向server1下的所有从服务器发送复制指令，让它们称为新主服务器的从服务器，并进行复制
3. 继续监视下线的server1，重新上线时，设置为从服务器

```

网络连接

```
sentinel与服务器的连接
	1. 命令连接 向服务器发送命令，并接收命令回复
	2. 订阅连接 订阅__sentinel__:hello频道

sentinel之间的连接 通过channel发现新sentinel，互相建立命令连接
```

sentinel命令类型

```
INFO命令 10s一次，通过命令连接向服务器发送INFO命令
PING命令 根据回复判断是否在线，回复类型+PONG，-LOADING，-MASTERDOWN
```

下线判断

```
一定时间内，服务器连续向sentinel返回无效回复，判断为主观下线状态
足够数量的已下线判断之后，将从服务器定位客观下线
主服务器客观下线后，监视这个下线服务器的各个sentinel会选举出领头sentinel，对进行故障转移
```

## 集群(分布式数据库方案)
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/10.png?raw=true)

集群

数据结构

```
节点使用clusterNode记录自己状态
节点使用clusterState保存集群状态
clusterState中
	slots[16384]记录所有指派信息
	slots_to_keys跳跃表 保存槽与键的关系，每个链表节点的分值是槽号
```

重定向

```
MOVED 槽的负责权转移到另一个节点，永久重定向
ASK 迁移槽的过程中使用的一种临时措施，客户端只会在接下来一次命令请求重定向到ASK指定节点
```

复制和故障转移

```
故障检测
集群每个节点定期向其他节点发送PING消息，检测对方是否在线
当一半以上主节点将某个主节点x报告为疑似下线的时候
会想集群广播主节点的FAIL消息，收到消息的节点将x标记为已下线

故障转移
新主节点向集群广播一条PONG消息
```



# 独立功能
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/11.png?raw=true)
## 发布与订阅

## 事务

事务的实现

```
事务的阶段
1. 事务开始
MULTI，客户端状态的flags属性打开REDIS_MULTI标识
2. 命令入队
客户端切换到事务状态后，
如果发送EXEC DISCARD WATCH MULTI，服务器会立刻执行
如果是其他命令，不会立刻执行，而是放入事务队列里，返回QUEUED回复
3. 执行事务
```

WATCH

```
WATCH是乐观锁，可以在EXEC命令执行之前，监视若干数据库键
如果被修改，服务器拒绝执行事务
```

redis事务的ACID性质

```
1. 	原子性 Atomic 事务中的多个操作，要么全部都执行，要么一个都不执行
	redis不支持回滚，即是出错，也会继续执行下去
2. 一致性 Consistency 无论事务是否执行成功，数据库是一致的，一致指的是符合数据库本身的定义和要求
	redis进行了错误检测，入队错误，执行错误。另外服务器停机也具有一致性
3. 隔离性 Isolation 并发执行的事务不会相互影响，结果完全相同
	redis使用单线程执行事务，且不会中断
4. 持久性 Duration 事务执行完毕，结果被持久化到硬盘中
	服务器在无持久化的内存模式以及RDB持久化模式下的事务不具有持久性
	AOF持久化模式，当且仅当appendfsync=always，具有持久性
```

## 慢查询日志