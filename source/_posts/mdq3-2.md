---
title: mybatis-dynamic-query 3.2 更新
date: 2020-11-28 22:20:06
tags:
  - java
  - Mybatis Dynamic Query
---

项目地址： [mybatis-dynamic-query](https://github.com/wz2cool/mybatis-dynamic-query)

# 前言

主要这次更新是针对 GROUP BY 支持，一直以来都觉得分组不好支持，一个是聚合属性和表字段不一样，还有个就是分组是两个筛选，一个是数据筛选一个分组后聚合筛选。

# 设计

我们先分析一下一个 GROUP BY 语句，我们想统计分类下的产品数量，product_id > 0 就是想去掉无效的产品，并且每个分类下的产品数量必须大于 1, 然后再让数量多的排到前面。这里其实我们同时用到了 WHERE 和 HAVING 两个筛选，他们两有什么不同呢，简而言之 WHERE 分组前筛选，HAVING 分组后筛选（包含聚合函数）。所以后面的讨论一直会围绕两段式设计，即分组前和分组后

```sql
 SELECT COUNT(product_id) AS count, category_id AS category_id FROM product WHERE (product_id > 0) GROUP BY category_id HAVING (COUNT(product_id) > 1) ORDER BY COUNT(product_id)
```

## 两个 Entity 实体

既然筛选和查询会用到两个不同的属性

- TQuery 实体  
  分组前我们用 WHERE 筛选，这里我们会用到一个实体，这个实体一般是表或者视图，主要作用是帮我们在分组前过滤数据和 DynamicQuery 的查询类似。
- TSelect 实体  
  分组后的实体，为什么叫 TSelect 呢，一般分组后的实体就是我们查询后的结果，这个实体类一般都含有聚合属性。

```java
@RegisterMapper
public interface SelectByGroupedQueryMapper<TQuery, TSelect> { ... }
```

## GroupByQuery 分组前查询

分组前一般我们做什么呢, 1. 选择属性（SELECT）, 2. 过滤(WHERE)

- SELECT (选择属性)
  这里选择属性其实是选择分组后的属性，所以这里用到的实体其实是`TSelect`
  ```java
  GroupByQuery<Product, CategoryGroupCount> groupByQuery = GroupByQuery.createQuery(Product.class, CategoryGroupCount.class)
        .select(CategoryGroupCount::getCategoryId,
                CategoryGroupCount::getCount);
  ```
