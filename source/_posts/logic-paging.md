---
title: 逻辑分页
date: 2020-05-02 20:03:52
tags:
  - Mybatis Dynamic Query
  - java
  - MDQ
---

# 前言

这个功能个人觉得是真的是一个非常有意思的功能，主要是为了解决大数据分页的问题。

# 分页对比

## 传统分页

比如我们常常用的 [PageHelper](https://github.com/pagehelper/Mybatis-PageHelper)

传统分页实际上是查询 2 次，输入参数为 pageNum, pageSize  
一般第一个查询为

```sql
SELECT * FROM ${tableName} LIMIT (pageNum -1) * pageSize, pageSize
```

第二个查询实际上是算总数(totalCount)

```sql
SELECT COUNT(*) FROM ${tableName}
```

当有了 pageNum, pageSize, totalCount, 我们就可以简答算出相关的分页信息  
pageNum  
pageSize  
totalCount  
totalPage = Math.ceiling(totalCount / pageSize)

分页逻辑看似没什么问题，但是当遇到海量数据的时候会出现两个性能问题

1. 注意第一查询， 中 `LIMIT ${offset}, ${pageSize}`， 当 offset 越大的时候整个查询会越来越慢 参考：[分页场景（limit,offset）为什么会慢？](https://blog.csdn.net/fengzongfu/article/details/103191867)

2. 注意第二个查询算总数，也是会因为数据量越来越大而越来越慢

## 逻辑分页

逻辑分页是一种权衡，就是说我们需要抛弃传统分页算总数（用户不知道到底有多少页），转而换成告诉用户，有没有上一页，有没有下一页。有一点点像 leetcode 里面有一种解决方法叫做窗口滑动。

那么具体我们怎么做呢？

- 首先我们需要多查一条记录，就是说我们判断有没有下一页，就是我们只需要多查一条数据就可以了，比如我们分页 pageSize 是 5， 那么实际上只需要 6 条数据就知道有没有下一页了  
  比如下图，就知道是有下一页的  
  [![logic1.md.png](https://wx2.sbimg.cn/2020/05/02/logic1.md.png)](https://sbimg.cn/image/mo7cw)  
  再比如我们再翻页，因为下一页不足 6 条数据我们就知道是没有下一页了   
  [![logic2.md.png](https://wx1.sbimg.cn/2020/05/02/logic2.md.png)](https://sbimg.cn/image/moHYo)
