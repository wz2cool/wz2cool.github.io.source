---
title: mybatis-dynamic-query 3.0 更新
date: 2019-10-03 08:49:44
tags:
  - java
  - Mybatis Dynamic Query
---

# 前言

在 2.0 完成对 tk.mapper 集成，[为何 mybatis-dynamic-query 选择 tk.mapper 集成](https://wz2cool.github.io/2019/10/03/why-choose-tk/), 再 3.0 进一步对查询进行优化，当然这里可能会对比 mybatis-plus, 我觉得有对比大家才能选择自己合适的。

# 更新内容

1. 添加 DynamicQueryBuilder 步骤化生成 DynamicQuery 语句
2. 优化 DynamicQuery 添加，移除筛选和排序

## DynamicQueryBuilder 引入

这个在 3.0 引入，目的是为了让大家写查询的时候真的像写 sql （严格遵循 sql 查询顺序），最后通过 build 方法来 build 一个 DynamicQuery, 根据经验来看 DynamicQueryBuilder 适合筛选条件已知的情况下。

```java
public List<ProductsDO> getProductListByBuilder() {
  // select product_name, list_price, category
  // where (list_price > 1 and list_price < 10) and description is not null or id = 1
  // order by id desc, list_price asc
  DynamicQuery<ProductsDO> query = DynamicQueryBuilder.create(ProductsDO.class)
     .select(ProductsDO::getProductName, ProductsDO::getListPrice, ProductsDO::getCategory)
     .where(ProductsDO::getListPrice, greaterThan(BigDecimal.ONE),
                and(ProductsDO::getListPrice, lessThan(BigDecimal.TEN)))
     .and(ProductsDO::getDescription, notEqual(null))
     .or(ProductsDO::getId, isEqual(1))
     .orderBy(ProductsDO::getId, desc())
     .thenBy(ProductsDO::getListPrice, asc())
     .build();
  return productMapper.selectByDynamicQuery(query);
}
```

## DynamicQuery 筛选排序优化

DynamicQuery 的很多方法名被改了，和 DynamicQueryBuilder 基本保持一致，这样大家在使用的时候比较方便，从下面的例子大家可以看到可以在任何位置添加筛选或者排序并且和 if 判断语句结合

```java
@Test
public void testGetProductListByQuery() {
    BigDecimal startPrice = BigDecimal.valueOf(1.1);
    BigDecimal endPrice = BigDecimal.valueOf(10.1);
    DynamicQuery<ProductsDO> query = DynamicQuery.createQuery(ProductsDO.class)
            .select(ProductsDO::getProductName, ProductsDO::getListPrice, ProductsDO::getCategory);
    // 根据参数添加筛选条件，这里就是我们看看开始价，结束价有没有，如果有才会放到一个组里面，
    if (Objects.nonNull(startPrice) || Objects.nonNull(endPrice)) {
        FilterGroupDescriptor<ProductsDO> priceFilterGroup = new FilterGroupDescriptor<>();
        if (Objects.nonNull(startPrice)) {
            priceFilterGroup.and(ProductsDO::getListPrice, greaterThan(startPrice));
        }
        if (Objects.nonNull(endPrice)) {
            priceFilterGroup.and(ProductsDO::getListPrice, lessThan(endPrice));
        }
    }
    query.and(ProductsDO::getDescription, notEqual(null))
            .or(ProductsDO::getId, isEqual(1))
            .orderBy(ProductsDO::getId, desc())
            .orderBy(ProductsDO::getListPrice, asc());
    List<ProductsDO> result = productMapper.selectByDynamicQuery(query);
    Assert.assertFalse(result.isEmpty());
}
```

### enable 字段

大家看到上面例子没有 if 判断条件会断开一个查询，这个在阅读的时候非常不方便，有了 enable 可以设置这个筛选是否生效，这样我们写代码的可读性高了

```java
@Test
public void testGetProductListByQuery2() {
  BigDecimal startPrice = BigDecimal.valueOf(1.1);
  BigDecimal endPrice = BigDecimal.valueOf(10.1);
  // 根据参数添加筛选条件，这里就是我们看看开始价，结束价有没有，如果有才会放到一个组里面，
  // @formatter:off
  DynamicQuery<ProductsDO> query = DynamicQuery.createQuery(ProductsDO.class)
    .select(ProductsDO::getProductName, ProductsDO::getListPrice, ProductsDO::getCategory)
    // 如果开始价格和结束价格都没有这个价格筛选组都不会产生
    .andGroupBegin(Objects.nonNull(startPrice) || Objects.nonNull(endPrice))
        .and(ProductsDO::getListPrice, greaterThan(startPrice), Objects.nonNull(startPrice))
        .and(ProductsDO::getListPrice, lessThan(endPrice), Objects.nonNull(endPrice))
    .groupEnd()
    .and(ProductsDO::getDescription, notEqual(null))
    .or(ProductsDO::getId, isEqual(1))
    .orderBy(ProductsDO::getId, desc())
    .orderBy(ProductsDO::getListPrice, asc());
  List<ProductsDO> result = productMapper.selectByDynamicQuery(query);
  Assert.assertFalse(result.isEmpty());
  // @formatter:on
}
```
