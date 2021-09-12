---
title: NormPagingQuery 标准分页查询
date: 2021-09-12 12:46:38
tags:
  - java
  - Mybatis Dynamic Query
---

项目地址： [mybatis-dynamic-query](https://github.com/wz2cool/mybatis-dynamic-query)

# 前言

我们常规分页一般会使用 PageHelper 里面的分页的两次查询问题，这个之前已经在[逻辑分页](https://wz2cool.github.io/2020/05/02/logic-paging/)中讲过了，但是逻辑分页的代价又非常大，因为我们需要对排序字段构建一个 PageId 字段用于翻上一页和下一页。 那么有没有中间方案，就是我只是想去掉查询 count 那次查询，还是返回上一页和下一页，那么 NormPagingQuery 就可以做这件事。

# 设计

## 上下分页

和逻辑分页一样，我们还是只是告诉有没有上一页和有没有下一页，这样我们就可以保证不再查询总数来计算页码。 那么如何判断有没有上下页呢

- 如果 pageNum 不是第一页就是说明有上一页
- 如果查询的 pageSize 是 50， 那么我们就多查询一条 比如 查询 51 条， 如果是 51 条那么就是有下一页

```java
@Test
public void testNormPaging2() throws JsonProcessingException {
    // 传统分页
    bugDao.deleteByDynamicQuery(DynamicQuery.createQuery(Bug.class));
    for (int i = 0; i < 10; i++) {
        Bug newBug = new Bug();
        newBug.setId(10000 + i);
        newBug.setAssignTo("frank");
        newBug.setTitle("title");
        bugDao.insert(newBug);
    }
    // 这里我们calcTotal 是false 就会不计算数量
    NormPagingQuery<Bug> query1 = NormPagingQuery.createQuery(Bug.class, 2, 3, true, false);
    NormPagingResult<Bug> query1Result = bugDao.selectByNormalPaging(query1);
    ObjectMapper objectMapper = new ObjectMapper();
    final String json1 = objectMapper.writeValueAsString(query1Result);
    System.out.println(json1);
    assertEquals(10003, (int) query1Result.getList().get(0).getId());
    assertEquals(0, query1Result.getTotal());
    assertEquals(2, query1Result.getPageNum());
    assertEquals(0, query1Result.getPages());
    assertTrue(query1Result.isHasNextPage());
    assertTrue(query1Result.isHasPreviousPage());
    bugDao.deleteByDynamicQuery(DynamicQuery.createQuery(Bug.class));
}
```

这里我们看到 pages 和 total 因为我们在设置的时候 calcTotal 设置为了 false。

```json
{
  "hasPreviousPage": true,
  "hasNextPage": true,
  "pageSize": 3,
  "pageNum": 2,
  "pages": 0,
  "total": 0,
  "list": [
    {
      "id": 10003,
      "title": "title",
      "assignTo": "frank"
    },
    {
      "id": 10004,
      "title": "title",
      "assignTo": "frank"
    },
    {
      "id": 10005,
      "title": "title",
      "assignTo": "frank"
    }
  ]
}
```

## 自动回退如果为空

上面大家会看到一个参数 autoBackIfEmpty 是什么意思，这个意思是如果当前页面是空，会自动向上回退直到查询到有数据的那页。这个其实是一个对前端保护措施，如果我们只有上一页和下一页， 如果用户翻到中间，后面我们进行了一次批量删除操作，这样会非常尴尬，用户翻上一页和翻下一页都没有数据，并且还不能跳页，客户会非常奇怪， 所以我们这边可以自动回退到有数据的最后一页上面。

```java
@Test
public void testNormPaging3() throws JsonProcessingException {
    // 传统分页
    bugDao.deleteByDynamicQuery(DynamicQuery.createQuery(Bug.class));
    for (int i = 0; i < 10; i++) {
        Bug newBug = new Bug();
        newBug.setId(10000 + i);
        newBug.setAssignTo("frank");
        newBug.setTitle("title");
        bugDao.insert(newBug);
    }
    // 这里我们calcTotal 是false 就会不计算数量，并且设置pageNum = 5
    NormPagingQuery<Bug> query1 = NormPagingQuery.createQuery(Bug.class, 5, 3, true, false);
    NormPagingResult<Bug> query1Result = bugDao.selectByNormalPaging(query1);
    ObjectMapper objectMapper = new ObjectMapper();
    final String json1 = objectMapper.writeValueAsString(query1Result);
    System.out.println(json1);
    assertEquals(10009, (int) query1Result.getList().get(0).getId());
    assertEquals(0, query1Result.getTotal());
    // 因为只有4页数据，即使用户上面设置的是5页，我们也会归到第4页上
    assertEquals(4, query1Result.getPageNum());
    assertEquals(0, query1Result.getPages());
    // 一共只有4也所以没有下一页了
    assertFalse(query1Result.isHasNextPage());
    assertTrue(query1Result.isHasPreviousPage());
    bugDao.deleteByDynamicQuery(DynamicQuery.createQuery(Bug.class));
}
```

从结果看我们看到结果 的 pageNum 已经变成 4 而不是 5，因为第 5 页数据是没有数据的

```json
{
  "hasPreviousPage": true,
  "hasNextPage": false,
  "pageSize": 3,
  "pageNum": 4,
  "pages": 0,
  "total": 0,
  "list": [
    {
      "id": 10009,
      "title": "title",
      "assignTo": "frank"
    }
  ]
}
```

## 兼容传统分页

这里我们还可以兼容传统分页，就是说 NormPagingQuery 也可以查询总数来计算页码，默认其实设计就是传统分页

```java
@Test
public void testNormPaging1() throws JsonProcessingException {
    // 传统分页
    bugDao.deleteByDynamicQuery(DynamicQuery.createQuery(Bug.class));
    for (int i = 0; i < 10; i++) {
        Bug newBug = new Bug();
        newBug.setId(10000 + i);
        newBug.setAssignTo("frank");
        newBug.setTitle("title");
        bugDao.insert(newBug);
    }
    // default autoBackIfEmpty = false, calcTotal = true
    NormPagingQuery<Bug> query1 = NormPagingQuery.createQuery(Bug.class, 2, 3);
    NormPagingResult<Bug> query1Result = bugDao.selectByNormalPaging(query1);
    ObjectMapper objectMapper = new ObjectMapper();
    final String json1 = objectMapper.writeValueAsString(query1Result);
    System.out.println(json1);
    assertEquals(10003, (int) query1Result.getList().get(0).getId());
    assertEquals(10, query1Result.getTotal());
    assertEquals(2, query1Result.getPageNum());
    assertEquals(4, query1Result.getPages());
    assertTrue(query1Result.isHasNextPage());
    assertTrue(query1Result.isHasPreviousPage());
    bugDao.deleteByDynamicQuery(DynamicQuery.createQuery(Bug.class));
}
```

这里我们就可以看到我们可以把 total 和 pages 都给算出来了

```json
{
  "hasPreviousPage": true,
  "hasNextPage": true,
  "pageSize": 3,
  "pageNum": 2,
  "pages": 4,
  "total": 10,
  "list": [
    {
      "id": 10003,
      "title": "title",
      "assignTo": "frank"
    },
    {
      "id": 10004,
      "title": "title",
      "assignTo": "frank"
    },
    {
      "id": 10005,
      "title": "title",
      "assignTo": "frank"
    }
  ]
}
```

## 小结

因为逻辑分页比较复杂需要设计，面对数据量不是很大，又想少查询一次的时候我们就可以用 NormPagingQuery, 并且兼容两次查询算总数和页码，加上自动回退如果是空的功能，更加丰富了这个分页查询。
