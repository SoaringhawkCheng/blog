---
title: 「MySQL是怎样使用的&运行的&运维内参」学习笔记 [DOING]
catalog: true
date: 2023-01-12 15:44:50
subtitle:
header-img:
tags:
- storage
categories:
- 工程
---
> 书籍豆瓣链接：
> [《MySQL是怎样使用的》](https://book.douban.com/subject/35670862/)
> [《MySQL是怎样运行的》](https://book.douban.com/subject/35231266/)
> [《MySQL运维内参》](https://book.douban.com/subject/27044364/)
>
> 相关在线资料：
> [《MySQL是怎样运行的》](https://relph1119.github.io/mysql-learning-notes/#/)
> [《极客时间：MySQL45讲》](https://time.geekbang.org/column/intro/139)


<!--![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/how-mysql-use-and-work/page-summary.png?raw=true)-->

* const：在主键列或者唯一二级索引列和一个常数进行等值比较
* ref：搜索条件为二级索引列与常数等值比较，采用二级索引来执行查询的访问方法
* ref_or_null：ref的情况，同时还要查值NULL
* range：利用索引进行范围匹配的访问方法
* index：遍历二级索引记录，不用进行回表的执行方式
*  all：直接扫描聚簇索引
* eq_ref：对被驱动表使用主键值或者唯一二级索引列的值进行等值查找的查询执行方式

