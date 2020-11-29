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

  ```java
  public class Product {
    @Column(name = "product_id")
    private Long productId;
    @Column(name = "product_name")
    private String productName;
    @Column(name = "price")
    private BigDecimal price;
    @Column(name = "category_id")
    private Integer categoryId;

    ...
  }
  ```

- TSelect 实体  
  分组后的实体，为什么叫 TSelect 呢，一般分组后的实体就是我们查询后的结果，这个实体类一般都含有聚合属性。

  ```java
  public class CategoryGroupCount {
    // 产品分类
    @Column(name = "product.category_id")
    private Integer categoryId;
    // 聚合属性，product 表中计算数量COUNT
    @Column(name = "COUNT(product.product_id)")
    private Integer count;

    ...
  }
  ```

## GroupByQuery 分组前查询

分组前一般我们做什么呢, 1. 选择属性（SELECT）, 2. 过滤(WHERE)

- SELECT (选择属性)
  这里选择属性其实是选择分组后的属性，所以这里用到的实体其实是`TSelect`
  ```java
  GroupByQuery<Product, CategoryGroupCount> groupByQuery =
    GroupByQuery.createQuery(Product.class, CategoryGroupCount.class)
        .select(CategoryGroupCount::getCategoryId,
                CategoryGroupCount::getCount);
  ```
- WHERE (过滤)
  这里筛选就会用到分组前的属性，所以这里用到的实体是`TQuery`
  ```java
  GroupByQuery<Product, CategoryGroupCount> groupByQuery =
    GroupByQuery.createQuery(Product.class, CategoryGroupCount.class)
        // 这里用到是分组后实体 TSelect
        .select(CategoryGroupCount::getCategoryId,
                CategoryGroupCount::getCount)
        // 这里用到是表实体TQuery
        .and(Product::getProductId, greaterThan(0L));
  ```

## GroupedQuery 分组后查询

分组后查询是不能独立存在的，他是在分组前查询（GroupByQuery）调用了方法（groupBy）以后自动会变成分组后查询。他做了哪些事情呢： 1. 对分组结果进行筛选（HAVING），2. 对分组结果进行排序（ORDER BY）

- HAVING (对分组结果进行筛选)  
  后面用到的实体都是 TSelect
  ```java
   GroupedQuery<Product, CategoryGroupCount> groupedQuery =
      GroupByQuery.createQuery(Product.class, CategoryGroupCount.class)
          .select(CategoryGroupCount::getCategoryId,
                  CategoryGroupCount::getCount)
          // 这里是Where 对数据筛选
          .and(Product::getProductId, greaterThan(0L))
          // 分组过后就变成了 GroupedQuery
          .groupBy(Product::getCategoryId)
          // 这里是having 对分组筛选
          .and(CategoryGroupCount::getCount, greaterThan(1))
  ```
- ORDER BY (对分组后进行排序)
  这里实体也是 TSelect
  ```java
   GroupedQuery<Product, CategoryGroupCount> groupedQuery =
      GroupByQuery.createQuery(Product.class, CategoryGroupCount.class)
        .select(CategoryGroupCount::getCategoryId,
                CategoryGroupCount::getCount)
        // 这里是Where 对数据筛选
        .and(Product::getProductId, greaterThan(0L))
        // 分组过后就变成了 GroupedQuery
        .groupBy(Product::getCategoryId)
        // 这里是having 对分组筛选
        .and(CategoryGroupCount::getCount, greaterThan(1))
        // 数量大的排在上面
        .orderBy(CategoryGroupCount::getCount, desc());
  ```
  这里注意，为了性能，我们为排序增加了`orderByNull`这个方法为了提高性能防止 filesort, [参考: mysql 语句：group by 后显示 using filesort 之解决方法](https://blog.csdn.net/weixin_43329834/article/details/93979663)
  ```java
   GroupedQuery<Product, CategoryGroupCount> groupedQuery =
      GroupByQuery.createQuery(Product.class, CategoryGroupCount.class)
        .select(CategoryGroupCount::getCategoryId,
                CategoryGroupCount::getCount)
        // 这里是Where 对数据筛选
        .and(Product::getProductId, greaterThan(0L))
        // 分组过后就变成了 GroupedQuery
        .groupBy(Product::getCategoryId)
        // 这里是having 对分组筛选
        .and(CategoryGroupCount::getCount, greaterThan(1))
        // 为了性能防止filesort
        .orderByNull();
  ```

## SelectByGroupedQueryMapper

这个 mapper 也没什么好赘述的了，就是针对分组专门设计的一个 mapper

```java
public interface CategoryGroupCountMapper extends SelectByGroupedQueryMapper<Product, CategoryGroupCount> {
}
```

## 结合

我们最终看一下把所有东西结合在一起是什么样子的, 还有就是这个分组对视图同样有效，类似做法就不举例了。

```java
@Test
public void testGroupBy() {
  GroupedQuery<Product, CategoryGroupCount> groupedQuery =
      GroupByQuery.createQuery(Product.class, CategoryGroupCount.class)
          .select(CategoryGroupCount::getCategoryId,
                  CategoryGroupCount::getCount)
          // 这里是Where 对数据筛选
          .and(Product::getProductId, greaterThan(0L))
          // 分组过后就变成了 GroupedQuery
          .groupBy(Product::getCategoryId)
          // 这里是having 对分组筛选
          .and(CategoryGroupCount::getCount, greaterThan(1))
          .orderBy(CategoryGroupCount::getCount, desc());
      List<CategoryGroupCount> categoryGroupCountList =
          categoryGroupCountMapper.selectByGroupedQuery(groupedQuery);
      for (CategoryGroupCount categoryGroupCount : categoryGroupCountList) {
        assertTrue(categoryGroupCount.getCount() > 1);
      }
}
```

我们看一下输出结果和我们想象的一致

```bash
==>  Preparing: SELECT COUNT(product.product_id) AS count, product.category_id AS category_id FROM product WHERE (product_id > ?) GROUP BY category_id HAVING (COUNT(product.product_id) > ?) ORDER BY COUNT(product.product_id) DESC
==> Parameters: 0(Long), 1(Integer)
<==    Columns: COUNT, CATEGORY_ID
<==        Row: 3, 3
<==        Row: 2, 2
<==      Total: 2
```

## 小结

以前困扰我许久的功能终于在两段式设计上面完成了，同时保证了智能提示和强类型校验，希望大家也不吝赐教。
