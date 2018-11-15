---
title: mybatis-dynamic-query 2.0.3 更新
date: 2018-11-15 21:45:03
tags:
  - Mybatis Dynamic Query
  - java
  - MDQ
---

项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/

## 杂谈

前段时间一直忙项目，而且一在直用 typescript 写动态查询，结果一看 java 版本已经一年没有更新了，感觉要加点东西了。

## 2.3 更新

更新了两点：

1. 支持选择字段查询
2. 支持链式调用在 DynamicQuery 类中

### 选择字段查询

默认来说，我们会把所有的字段全部返回，有些字段我们不想返回，比如 password

下面我们来看个例子：

1. 创建一个实体 Entity

```java
@Table(name = "product")
public class Product {
    @Column(name = "product_id", insertable = false, updatable = false)
    private Integer productID;
    private String productName;
    private BigDecimal price;
    private Integer categoryID;

    ...
}
```

2. 创建我们 Dao 继承自 DynamicQueryMapper

```java
public interface ProductDao extends DynamicQueryMapper<Product> {
}
```

3. 利用 addSelectField 方法来描述我们需要哪些字段，这里例子中，我们只需要：productName, price.

```java
@Test
public void testSelectFields() {
    DynamicQuery<Product> dynamicQuery = DynamicQuery.createQuery(Product.class)
            .addSelectField(Product::getProductName)
            .addSelectField(Product::getPrice);

    List<Product> products = PageHelper.startPage(0, 3, false)
                .doSelectPage(() -> productDao.selectByDynamicQuery(dynamicQuery));

    for (Product p : products) {
        // categoryID ignore to select
        assertEquals(null, p.getCategoryID());
        assertEquals(true, StringUtils.isNotBlank(p.getProductName()));
    }
}

```

5. 我们同样可以看一下 mybatis 控制台输出结果, 确实我们只选择了 productName 和 price

```bash
==>  Preparing: SELECT price AS price, product_name AS product_name FROM product LIMIT 3
==> Parameters:
<==    Columns: PRICE, PRODUCT_NAME
<==        Row: 18.0000, Northwind Traders Chai
<==        Row: 7.5000, Northwind Traders Syrup
<==        Row: 16.5000, Northwind Traders Cajun Seasoning
<==      Total: 3
```

### 链式调用

为了让代码写的比较舒服，不用再手动 new FilterDescritpor 了。

```java
@Resource
private ProductDao productDao;

@Test
public void testLinkOperation() {
    DynamicQuery<Product> dynamicQuery = DynamicQuery.createQuery(Product.class)
            .addSelectField(Product::getProductID)
            .addSelectField(Product::getProductName)
            .addSelectField(Product::getPrice)
            .addFilterDescriptor(Product::getPrice, FilterOperator.GREATER_THAN, 16)
            .addSortDescriptor(Product::getPrice, SortDirection.DESC)
            .addSortDescriptor(Product::getProductID, SortDirection.DESC);

    List<Product> products = PageHelper.startPage(0, 100, false)
            .doSelectPage(() -> productDao.selectByDynamicQuery(dynamicQuery));

    for (Product p : products) {
        // categoryID ignore to select
        assertEquals(null, p.getCategoryID());
        assertEquals(true, StringUtils.isNotBlank(p.getProductName()));
        // price > 16
        assertEquals(1, p.getPrice().compareTo(BigDecimal.valueOf(16)));
    }
}
```

看一下输出结果和用 new FilterDescriptor，new SortDesciptor 效果是一样的, 并且在价格一样的时候，ID 是倒叙排列的。

```bash
==>  Preparing: SELECT product_id AS product_id, price AS price, product_name AS product_name FROM product WHERE (price > ?) ORDER BY price DESC, product_id DESC LIMIT 100
==> Parameters: 16(Integer)
<==    Columns: PRODUCT_ID, PRICE, PRODUCT_NAME
<==        Row: 1, 18.0000, Northwind Traders Chai
<==        Row: 4, 16.5000, Northwind Traders Olive Oil
<==        Row: 3, 16.5000, Northwind Traders Cajun Seasoning
<==      Total: 3
```

## 结束

时隔一年已经在项目中一直使用，感觉还是蛮顺手的，希望大家能支持一下。

## 关注@我　

最后大家可以关注我和 Mybatis-Dynamic-query 项目 ^\_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>

## 文章整合

[Mybatis Dynamic Query 简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)
[Mybatis Dynamic Query 组筛选](https://wz2cool.github.io/2017/07/28/groupFilter/)
[Mybatis Dynamic Query 排序](https://wz2cool.github.io/2017/07/28/sort/)
[Mybatis Dynamic Query 筛选+排序](https://wz2cool.github.io/2017/07/28/filterSort/)
[Mybatis Dynamic Query 插入](https://wz2cool.github.io/2017/07/28/insert/)
[Mybatis Dynamic Query 更新](https://wz2cool.github.io/2017/07/31/update/)
[Mybatis Dynamic Query 删除](https://wz2cool.github.io/2017/07/31/delete/)
[Mybatis Dynamic Query 属性表达式](https://wz2cool.github.io/2017/07/31/propertyExpression/)
[Mybatis Dynamic Query join 视图](https://wz2cool.github.io/2017/07/31/joinView/)
[Mybatis Dynamic Query 2.0 入门](https://wz2cool.github.io/2017/08/15/howToUse2/)
