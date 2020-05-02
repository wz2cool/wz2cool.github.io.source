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

- 首先我们需要确认一个分页 id，因为没有这个分页 id 我们无法知道下一页从哪里开始。
- 然后我们需要多查一条记录，就是说我们判断有没有下一页，就是我们只需要多查一条数据就可以了，比如我们分页 pageSize 是 5， 那么实际上只需要 6 条数据就知道有没有下一页了

有了上面两个理论基础，我们假设一个场景：我们有 9 名学生，每个学生有自己的 id 和名字， 我们分页查询，每页 5 条数据  
首先我们多查一条数据，那么就是说我们要查 6 条， 然后 6 > 5 我们就知道有下一页了  
这里我们要记录一下这个窗口的分页 id (startPageId: 1, endPageId: 5), pageId 是为了记录位置

```sql
SELECT * FROM `student` LIMIT 6
```

[![logic1.md.png](https://wx1.sbimg.cn/2020/05/02/logic1.md.png)](https://sbimg.cn/image/moNYY)

再比如我们再翻页, 其实我们后面只有 4 名学生了， 4 < 5 我们就知道没有下一页了  
这里我们也要记录一下这个结果集的分页 id (startPageId: 6, endPageId: 9), pageId 是为了记录位置

```sql
-- 这里的5 是上一次endPageId
SELECT * FROM `student` WHERE ID > 5 Limit 6
```

[![logic2.md.png](https://wx1.sbimg.cn/2020/05/02/logic2.md.png)](https://sbimg.cn/image/moHYo)

好了大家已经能看到优势了对吧，一般来说 id 是有索引的，这样避免了 offset 过大导致语句慢，还有就是其实这里用多查一条来代替查询 count

## 在框架中应用

这里面我们需要使用一个新的 LogicPagingQuery, 专门来做逻辑分页，注意逻辑分页是不允许非 PageId 字段进行排序的，因为我们需要根据这个逻辑的分页 ID 来进行记录位置

```java
@Test
public void testGetDataAscDown() {
    // 用 student 表中的id 作为分页id，升序并且向下翻页
    LogicPagingQuery<Student> logicPagingQuery =
            LogicPagingQuery.createQuery(Student.class, Student::getId, SortDirection.ASC, UpDown.DOWN);
    logicPagingQuery.setPageSize(5);
    LogicPagingResult<Student> result = productDao.selectByLogicPaging(logicPagingQuery);
}

@Test
public void testGetDataAscDown2() {
     // 用 student 表中的id 作为分页id，升序并且向下翻页
    LogicPagingQuery<Student> logicPagingQuery =
            LogicPagingQuery.createQuery(Student.class, Student::getId, SortDirection.ASC, UpDown.DOWN);
    logicPagingQuery.setPageSize(5);
    // 我们第二次翻页要填上上次pageId位置信息
    logicPagingQuery.setLastStartPageId(1L);
    logicPagingQuery.setLastEndPageId(5L);
    LogicPagingResult<Student> result = productDao.selectByLogicPaging(logicPagingQuery);
}
```
