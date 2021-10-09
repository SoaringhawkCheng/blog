---
title: 「深入理解Java虚拟机」学习笔记[DOING]
catalog: true
date: 2021-11-24 23:53:35
subtitle:
header-img:
tags:
- os
categories:
- 工程
---
> 书籍豆瓣链接：[《深入理解Java虚拟机》](https://book.douban.com/subject/24722612/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：


<!--![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/understand-jvm/mind2.png?raw=true)-->

[第一篇](https://blog.csdn.net/weixin_40992982/article/details/104030759?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163376522916780264070543%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163376522916780264070543&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-104030759.first_rank_v2_pc_rank_v29&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0+NayelyAA&spm=1018.2226.3001.4187)
[第二篇](https://blog.csdn.net/weixin_40992982/article/details/104039324?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163376522916780264070543%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163376522916780264070543&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-10-104039324.first_rank_v2_pc_rank_v29&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0+NayelyAA&spm=1018.2226.3001.4187)
[第三篇](https://blog.csdn.net/weixin_40992982/article/details/104041449?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163376522916780264070543%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163376522916780264070543&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-7-104041449.first_rank_v2_pc_rank_v29&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0+NayelyAA&spm=1018.2226.3001.4187)
[第四篇](https://blog.csdn.net/weixin_40992982/article/details/104052638?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163376522916780264070543%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163376522916780264070543&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-104052638.first_rank_v2_pc_rank_v29&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0+NayelyAA&spm=1018.2226.3001.4187)
[第五篇](https://blog.csdn.net/weixin_40992982/article/details/104055126?ops_request_misc=&request_id=&biz_id=102&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%20NayelyA&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-3-104055126.first_rank_v2_pc_rank_v29&spm=1018.2226.3001.4187)
[第六篇](https://blog.csdn.net/weixin_40992982/article/details/104056639?ops_request_misc=&request_id=&biz_id=102&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%20NayelyA&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-104056639.first_rank_v2_pc_rank_v29&spm=1018.2226.3001.4187)
[第七篇](https://blog.csdn.net/weixin_40992982/article/details/104071278?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163376522916780264070543%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163376522916780264070543&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-3-104071278.first_rank_v2_pc_rank_v29&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0+NayelyAA&spm=1018.2226.3001.4187)
[第八篇](https://blog.csdn.net/weixin_40992982/article/details/104071298?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163376522916780264070543%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163376522916780264070543&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-4-104071298.first_rank_v2_pc_rank_v29&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0+NayelyAA&spm=1018.2226.3001.4187)
[第九篇](https://blog.csdn.net/weixin_40992982/article/details/104071318?ops_request_misc=&request_id=&biz_id=102&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%20NayelyA&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-104071318.first_rank_v2_pc_rank_v29&spm=1018.2226.3001.4187)
[第十篇](https://blog.csdn.net/weixin_40992982/article/details/104071348?ops_request_misc=&request_id=&biz_id=102&utm_term=%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%20NayelyA&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-6-104071348.first_rank_v2_pc_rank_v29&spm=1018.2226.3001.4187)