---
layout: post
title: "成丰实时数仓2.0"
description: "基于lambda架构、flink+clickhouse"
categories: [实时数仓架构]
tags: [clickhouse,redis,lambda]
redirect_from:
  - /2021/01/03/
---



# 成丰实时数仓2.0

**基于成丰实时数仓1.0留下的问题，后续对架构提出了升级**

1.由于flink支持的mysqlCDC源表存储的数据是在内存是比较浪费的，因此我们通过实现RichAsyncFunction做到异步关联redis维表数据，大大提升了关联的效率和资源的浪费。并且通过flink读取ods层写到redis实现redis的CDC（Change Data Capture）。

2.由于ecs服务器和flink所在的region不同，导致vpc访问的限制，且要做到即席查询还要依赖impala、phoenix等即席查询组件。借鉴了腾讯看点的flink+clickhouse架构，以其物化视图和mergeTree及其家族表引擎的预处理、并行向量化处理、全程无锁等特点，做到即席查询、olap分析的压秒级响应。

![image-20210402191805679](/photo/jg3.jpg)