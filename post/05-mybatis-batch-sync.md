---
title: mybatis batch sync
date: 2020-09-26
permalink: mybatis-batch-sync
categories:
  - 技术
tags:
  - mybatis
toc: true
---

# mybatis 批量同步数据

有个数据同步的任务，需要每天将 oracle9i 部分表的全量数据同步到 oracle12c。当然可以直接用 jdbc，但是太繁琐，我可以利用 mybatis 和 spring 自身的一些特性来完成这个功能。

大致的思路是，配置多个数据源，源数据源利用 `ResultHandler` 分批查出来数据，然后目前数据源直接使用 jdbc 将数据插入到目标数据库里面。

整个过程有多个知识点：

1. 如何配置多个数据源
2. 如何配置多个 mybatis SqlSessionTemplate
3. 如何分批来处理数据，按批次查询并插入到目标数据库
4. 如何动态的生成 insert sql
