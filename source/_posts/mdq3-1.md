---
title: mybatis-dynamic-query 3.0 更新
date: 2019-10-03 08:49:44
tags:
  - java
  - Mybatis Dynamic Query
---

项目地址： [mybatis-dynamic-query](https://github.com/wz2cool/mybatis-dynamic-query)

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

大家看到上面例子, 有 if 判断条件会断开一个查询，这个在阅读的时候非常不方便，有了 enable 可以设置这个筛选是否生效，这样我们写代码的可读性高了

```java
@Test
public void testGetProductListByQuery2() {
    BigDecimal startPrice = BigDecimal.valueOf(1.1);
    BigDecimal endPrice = BigDecimal.valueOf(10.1);
    // 根据参数添加筛选条件，这里就是我们看看开始价，结束价有没有，如果有才会放到一个组里面，
    DynamicQuery<ProductsDO> query = DynamicQuery.createQuery(ProductsDO.class)
            .select(ProductsDO::getProductName, ProductsDO::getListPrice, ProductsDO::getCategory)
            .and(group -> group
                    .and(ProductsDO::getListPrice, greaterThan(startPrice), Objects.nonNull(startPrice))
                    .and(ProductsDO::getListPrice, lessThan(endPrice), Objects.nonNull(endPrice)))
            .and(ProductsDO::getDescription, notEqual(null))
            .or(ProductsDO::getId, isEqual(1))
            .orderBy(ProductsDO::getId, desc())
            .orderBy(ProductsDO::getListPrice, asc());
    List<ProductsDO> result = productMapper.selectByDynamicQuery(query);
    Assert.assertFalse(result.isEmpty());
}
```

# 对比

开始我是不知道 mybatis-plus [博客园动态查询第一帖](https://www.cnblogs.com/wz2cool/p/7268428.html) 的不然的话，可能我就直接用了哈哈~，既然自己做了一个也和标杆对比一下吧，但还是期望大家选择自己合适的吧，这里我只对比 mybatis-plus 查询功能。

## 举例

### 复杂条件查询

基本和动态查询在写法上基本表现一致，不过新版的动态插叙加上了 enable 字段以后读起来会好一些

```java
@Test
public void testGetProductListByPlus() {
    BigDecimal startPrice = BigDecimal.valueOf(1.1);
    BigDecimal endPrice = BigDecimal.valueOf(10.1);
    LambdaQueryWrapper<ProductsDO> queryWrapper = new QueryWrapper<ProductsDO>().lambda()
            .select(ProductsDO::getListPrice, ProductsDO::getProductName, ProductsDO::getCategory);
    if (Objects.nonNull(startPrice) && Objects.nonNull(endPrice)) {
        // 没有找到如何将连个price 筛选放到一个组里面
        queryWrapper.and(obj -> obj.gt(ProductsDO::getListPrice, startPrice)
                .lt(ProductsDO::getListPrice, endPrice));
    } else if (Objects.nonNull(startPrice)) {
        queryWrapper.gt(ProductsDO::getListPrice, startPrice);
    } else if (Objects.nonNull(endPrice)) {
        queryWrapper.lt(ProductsDO::getListPrice, startPrice);
    }
    queryWrapper.ne(ProductsDO::getDescription, null)
            .or(obj -> obj.eq(ProductsDO::getId, 1))
            .orderByDesc(ProductsDO::getId)
            .orderByAsc(ProductsDO::getListPrice);
    List<ProductsDO> result = productPlusMapper.selectList(queryWrapper);
    Assert.assertFalse(result.isEmpty());
}
```

### 灵活性

mybatis-plus 是非常灵活的， api 特别多， 比如 queryWrapper 可以直接接上操作符比如 eq，gt, lt 也可以接 and， or， 但是动态查询对于筛选只能接 and / or 操作符必须在里面， 我个人喜欢统一性，这其实就是仁者见仁智者见智了。

### 安全性

#### 类型检查

可以说这个就是动态查询的优势了，设计之初就是怎么样让用户写出不会错的代码，所以所有的筛选值都是强类型，比如说字段 Price 是一个 BigDecimal, 如果写 Integer 就会报错，但是 mybatis-plus 就不会报错。

```java
@Test
public void testGetProductListByQuery2() {
  BigDecimal startPrice = BigDecimal.valueOf(1.1);
  BigDecimal endPrice = BigDecimal.valueOf(10.1);
  Integer integerStartPrice = 1;
  DynamicQuery<ProductsDO> query = DynamicQuery.createQuery(ProductsDO.class)
    .select(ProductsDO::getProductName, ProductsDO::getListPrice, ProductsDO::getCategory)
    .and(group -> group
            // 这段代码 会包错，因为integerStartPrice 不是BigDecimal 类型的
            .and(ProductsDO::getListPrice, greaterThan(integerStartPrice), Objects.nonNull(startPrice))
            .and(ProductsDO::getListPrice, lessThan(endPrice), Objects.nonNull(endPrice)))
    .and(ProductsDO::getDescription, notEqual(null))
    .or(ProductsDO::getId, isEqual(1))
    .orderBy(ProductsDO::getId, desc())
    .orderBy(ProductsDO::getListPrice, asc());
  List<ProductsDO> result = productMapper.selectByDynamicQuery(query);
  Assert.assertFalse(result.isEmpty());
}
```

### 链式调用

这个动态查询和 mybatis-plus 都是支持的，不一样的是，动态查询不会随意接后面的方法，是做过验证的，但是 mybatis-plus 比较灵活，把这个安全性检查交给用户了。
比如

```java
@Test
public void testGetProductListByPlus() {
    BigDecimal startPrice = BigDecimal.valueOf(1.1);
    BigDecimal endPrice = BigDecimal.valueOf(10.1);
    LambdaQueryWrapper<ProductsDO> queryWrapper = new QueryWrapper<ProductsDO>().lambda()
            .select(ProductsDO::getListPrice, ProductsDO::getProductName, ProductsDO::getCategory);
    if (Objects.nonNull(startPrice) && Objects.nonNull(endPrice)) {
        // 这里我随意在筛选后面添加了一个排序, 编译的时候没有错，执行的时候会报错
        queryWrapper.and(obj -> obj.gt(ProductsDO::getListPrice, startPrice)
                .lt(ProductsDO::getListPrice, endPrice).orderByAsc(ProductsDO::getListPrice));
    } else if (Objects.nonNull(startPrice)) {
        queryWrapper.gt(ProductsDO::getListPrice, startPrice);
    } else if (Objects.nonNull(endPrice)) {
        queryWrapper.lt(ProductsDO::getListPrice, startPrice);
    }
    queryWrapper.ne(ProductsDO::getDescription, null)
            .or(obj -> obj.eq(ProductsDO::getId, 1))
            .orderByDesc(ProductsDO::getId)
            .orderByAsc(ProductsDO::getListPrice);
    List<ProductsDO> result = productPlusMapper.selectList(queryWrapper);
    Assert.assertFalse(result.isEmpty());
}
```

# 小结

主要给大家看了一下 3.0 对查询的改动，主要也是给大家多一个选择， 稍微对比了一下 mybatis-plus, 自我感觉在查询写法上面有优势，但是 mybatis-plus 是功能非常多，大而全的一整套解决方案，文档非常完善，这也是动态查询不具备的，所以大家选择自己合适的。
