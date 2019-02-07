---
title: TiDB入门（一）：TiDB 简介
date: 2019-02-07 10:50:58
tags:
  - TiDB
---

# TiDB 简介

## 什么是 TiDB

简单来说两点：

1. 是一个分布式数据库，支持无限水平扩展。
2. 是一个高度兼容 MySQL，基本可以做到无缝切换。  
   具体可参考官方简介：https://www.pingcap.com/docs-cn/

## 适用场景

### 适用

1. 原业务的 MySQL 的业务遇到单机容量或者性能瓶颈时，可以考虑使用 TiDB 无缝替换 MySQL。TiDB 可以提供如下特性：
   - 吞吐量、存储和计算能力的水平扩展
   - 水平伸缩时不停服务
   - 强一致性分布式 ACID 事务
2. 大数据量下，MySQL 复杂查询很慢。
3. 大数据量下，数据增长很快，接近单机处理的极限，不想分库分表或者使用数据库中间件等对业务侵入性较大、对业务有约束的 Sharding 方案。
4. 大数据量下，有高并发实时写入、实时查询、实时统计分析的需求。
5. 有分布式事务、多数据中心的数据 100% 强一致性、auto-failover 的高可用的需求。

### 不适用

1. 单机 MySQL 能满足的场景也用不到 TiDB。
2. 数据条数少于 5000w 的场景下通常用不到 TiDB，TiDB 是为大规模的数据场景设计的。
3. 如果你的应用数据量小（所有数据千万级别行以下），且没有高可用、强一致性或者多数据中心复制等要求，那么就不适合使用 TiDB。

## TiDB 整体架构

![tidb-architecture](https://www.pingcap.com/images/docs-cn/tidb-architecture.png)

### TiDB Server

TiDB Server 负责接收 SQL 请求，处理 SQL 相关的逻辑，并通过 PD 找到存储计算所需数据的 TiKV 地址，与 TiKV 交互获取数据，最终返回结果。  
TiDB Server 是无状态的，其本身并不存储数据，只负责计算，可以无限水平扩展，可以通过负载均衡组件（如 LVS、HAProxy 或 F5）对外提供统一的接入地址。  
<em>注： 我们连接 Mysql 的地址其实就是连接到 TiDB Server 上</em>

### PD Server

Placement Driver (简称 PD) 是整个集群的管理模块，其主要工作有三个：一是存储集群的元信息（某个 Key 存储在哪个 TiKV 节点）；二是对 TiKV 集群进行调度和负载均衡（如数据的迁移、Raft group leader 的迁移等）；三是分配全局唯一且递增的事务 ID。  
PD 是一个集群，需要部署奇数个节点，一般线上推荐至少部署 3 个节点。

### TiKV Server

TiKV Server 负责存储数据，从外部看 TiKV 是一个分布式的提供事务的 Key-Value 存储引擎。  
存储数据的基本单位是 Region，每个 Region 负责存储一个 Key Range（从 StartKey 到 EndKey 的左闭右开区间）的数据，每个 TiKV 节点会负责多个 Region。TiKV 使用 Raft 协议做复制，保持数据的一致性和容灾。副本以 Region 为单位进行管理，不同节点上的多个 Region 构成一个 Raft Group，互为副本。数据在多个 TiKV 之间的负载均衡由 PD 调度，这里也是以 Region 为单位进行调度。
<em>注： 这是放数据的地方，所以线上必须是 SSD 保证其速度。</em>

### TiSpark

TiSpark 作为 TiDB 中解决用户复杂 OLAP 需求的主要组件，将 Spark SQL 直接运行在 TiDB 存储层上，同时融合 TiKV 分布式集群的优势，并融入大数据社区生态。  
至此，TiDB 可以通过一套系统，同时支持 OLTP 与 OLAP，免除用户数据同步的烦恼。

参考：  
简介/教程：https://www.pingcap.com/docs-cn/  
适用场景：https://blog.csdn.net/u011782423/article/details/80940766  
TiDB 的正确使用姿势：https://pingcap.com/blog-cn/how-to-use-tidb/
