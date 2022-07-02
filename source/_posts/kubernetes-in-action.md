---
title: 「Kubernetes in Action中文版」读书笔记 [DOING]
catalog: true
date: 2022-06-01 18:07:29
subtitle:
header-img:
tags:
- devops
categories:
- 工程
---

> 书籍豆瓣链接：[《Kubernetes in Action中文版》](https://book.douban.com/subject/30418855/)
> 
> 中文官方文档：[《Kubernetes文档》](https://kubernetes.io/zh-cn/docs/home/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## 相关资料

[书中重要的图](https://www.jianshu.com/p/fa66769ee551)

[Kubernetes介绍](https://www.jianshu.com/p/6957cc5a446d)

[部署第一个应用](https://www.jianshu.com/p/7bc34ff88d9d)

[理解容器技术](https://www.jianshu.com/p/8acfabeee26e)

[深入了解 Pod 概念](https://www.jianshu.com/p/e8c0ba64c8c4)

[管理Pod的生命](https://www.jianshu.com/p/d2147dd4970f)

[向容器挂载存储卷](https://www.jianshu.com/p/d2147dd4970f)

[使用 ConfigMaps、Secrets 和 Downward API 配置应用](https://www.jianshu.com/p/37c690589151)

[通过 Services 对象暴露 Pod 中的服务](https://www.jianshu.com/p/31dad7e92af6)

[通过 PersistentVolume 持久化数据](https://www.jianshu.com/p/5325121d6c31)

[《Kubernetes in action 读书笔记》：运维架构演进](https://xie.infoq.cn/article/2991b15a82c0974a827f9065a)

[《Kubernetes in action 读书笔记》：容器技术的发展](https://xie.infoq.cn/article/ff1dc14b1ad1062c90194eac9)

[《Kubernetes in action 读书笔记》：Kurbernetes 架构设计](https://xie.infoq.cn/article/09418e5a8b55a1a6259dd964a)

[《Kubernetes in action 读书笔记》：Kurbernetes 横空出世](https://xie.infoq.cn/article/a46bb80b61fb0ae052e5f3fdd)

[《你需要的 Docker 知识点都在这里了》](https://xie.infoq.cn/article/ba89764f2d51a549e589c311a)



# 概念

## 概述

### Kubernetes是什么

![](https://github.com/SoaringhawkCheng/blog/blob/a9a5f2b2a68727210ed433cb401883204506537a/source/_posts/kubernetes-in-action/container_evolution.svg)

#### 技术演变

早期，各机构是在物理服务器上运行应用程序。由于无法限制在物理服务器中运行的应用程序资源使用，因此会导致资源分配问题。

虚拟化技术允许你在单个物理服务器的 CPU 上运行多台虚拟机（VM），每个 VM 是一台完整的计算机，在虚拟化硬件之上运行所有组件，包括其自己的操作系统（OS）

容器类似于 VM但是更宽松的隔离特性，使容器之间可以共享操作系统（OS），每个容器都具有自己的文件系统、CPU、内存、进程空间等

### Kubernetes组件

一个 Kubernetes 集群是由一组被称作节点（node）的机器组成， 这些节点上会运行由 Kubernetes 所管理的容器化应用

工作节点会托管所谓的 Pods，而 Pod 就是作为应用负载的组件。 控制平面管理集群中的工作节点和 Pods

#### 控制平面组件

控制平面组件会为集群做出全局决策，比如资源的调度。 通常会在同一个计算机上启动所有控制平面组件， 并且不会在此计算机上运行用户容器

* kube-apiserver

控制平面的前端，设计上考虑了水平扩缩，可以运行多个实例，并在这些实例之间平衡流量

* etcd 

是兼顾一致性与高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库 [etcd官方文档-原理学习](https://doczhcn.gitbook.io/etcd/index/index-2)

* kube-scheduler

监视新创建的、未指定运行节点（node）的 Pods， 并选择节点来让 Pod 在上面运行

* kube-controller-manager

负责运行控制器进程，这些控制器包括：

```
节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
任务控制器（Job Controller）：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
端点控制器（Endpoints Controller）：填充端点（Endpoints）对象（即加入 Service 与 Pod）
服务帐户和令牌控制器（Service Account & Token Controllers）：为新的命名空间创建默认帐户和 API 访问令牌
```

*  cloud-controller-manager

```
看不懂
```

#### Node组件

节点组件会在每个节点上运行，负责维护运行的 Pod 并提供 Kubernetes 运行环境

* kubelet

kubelet 会在集群中每个节点（node）上运行。 它保证容器（containers）都运行在 Pod 中

* kube-proxy

kube-proxy 是集群中每个节点（node）所上运行的网络代理， 实现 Kubernetes 服务（Service） 概念的一部分

* Container Runtime

容器运行环境是负责运行容器的软件，Kubernetes 支持许多容器运行环境，例如 Docker

### Kubernetes API

### 使用 Kubernetes 对象

## Kubernetes架构

### 节点

通过将容器放入在节点（Node）上运行的 Pod 中来执行你的工作负载

节点上的组件包括 kubelet、 容器运行时以及 kube-proxy。

#### 管理

### 节点与控制面通信

#### 节点到控制面

### 控制器

#### 控制器模式

* 通过API服务控制：Job 是一种 Kubernetes 资源，Job 控制器是通知 API 服务器来创建或者移除 Pod
* 直接控制：例如，如果你使用一个控制回路来保证集群中有足够的 节点，那么控制器就需要当前集群外的 一些服务在需要时创建新节点。和外部状态交互的控制器从 API 服务器获取到它想要的状态，然后直接和外部系统进行通信 并使当前状态更接近期望状态。

#### 

### 云控制器管理器




