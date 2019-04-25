---
title: Elasticsearch学习笔记
catalog: true
date: 2019-04-24 10:59:21
subtitle:
header-img:
tags:
- nosql
categories:
- 编程
---
> 书籍豆瓣链接：[ES 5.2 官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/index.html)
> 
> 开始学习日期：4-1
> 
> 预计完成时间：5-4
>
> 实际完成时间：

# 入门
## 基础概念
### 集群 cluster
集群是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能。
### 节点 node
节点是作为群集一部分的单个服务器，存储数据并参与群集的索引和搜索功能。
### 索引 index
索引是具有某些类似特征的文档集合
### 类型 type
类型是索引的逻辑类别/分区，其语义完全取决于您。 通常，具有一组公共字段的文档定义类型。
### 文档 document
文档是可以被索引的基础信息单元
### 分片 shards
每个分片本身都是一个功能齐全且独立的“索引”，可以托管在集群中的任何节点上。
* 水平分割/缩放内容量
* 跨分片（可能在多个节点上）分布和并行化操作

### 副本 replicas
* 分片/节点出现故障时提供高可用性。 副本永远不会和主分片在相同的节点
* 允许您扩展搜索量/吞吐量，可以在所有副本上并行执行搜索

## 健康
```
GET / _cat / health?v 
GET / _cat / nodes?v
```
### 集群健康
> * 绿色
> 
> 集群功能齐全
> 
> * 黄色 
> 
> 所有数据可用，但尚未分配一些副本
> 
> * 红色
> 
> 部分数据不可用
> 

### 三种通讯模式
> * 单播
>
> 主机之间“一对一”的通讯模式，网络中的交换机和路由器对数据只进行转发不进行复制
>
> * 广播
>
> 主机之间“一对所有”的通讯模式，网络对其中每一台主机发出的信号都进行无条件复制并转发，所有主机都可以接收到所有信息
>
> * 多播
> 
> 主机之间“一对一组”的通讯模式，也就是加入了同一个组的主机可以接受到此组内的所有数据，网络中的交换机和路由器只向有需求者复制并转发其所需数据

# 分析器
分析器分类：索引时分析和查询时分析器
分析器结构：字符过滤器、分词器、单词过滤器

## 测试分析器
```
GET index/_analyze # 获取index的分词器分词
{
	"analyzer": "",
	"text": ,
}
GET index/_analyze # 获取index的某个字段的分词器分词
{
	"field": "",
	"text": ,
}
POST _analyze  # 测试分词器分词 
```
## 分析器详解