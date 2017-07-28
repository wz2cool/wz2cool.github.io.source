---
title: Mybatis Dynamic Query 排序
date: 2017-07-28 16:07:47
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/  
排序和筛选类似，主要使用 SortDescriptor 来做描述
关于 SortDescriptor定义可以参照：[SortDescriptor类](https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/sortdescriptorlei.html)

## 准备工作 ##
这里我们沿用[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)里面的准备工作即可。
## 开始排序 ##
### 简单排序 ###
让价格倒叙排列
```java
@Test
public void testSort() throws Exception {
        SortDescriptor priceSort =
                new SortDescriptor("price", SortDirection.DESC);

        Map<String, Object> queryParams =
                mybatisQueryProvider.getSortQueryParamMap(
                        Product.class, "orderExpression", priceSort);
        northwindDao.getProductByDynamic(queryParams);
}
```
输出结果可以看到价格已经按照倒叙排列了
```bash
==>  Preparing: SELECT * FROM product ORDER BY price DESC 
==> Parameters: 
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE
<==        Row: 1, 1, Northwind Traders Chai, 18.0000
<==        Row: 3, 2, Northwind Traders Cajun Seasoning, 16.5000
<==        Row: 4, 3, Northwind Traders Olive Oil, 16.5000
<==        Row: 2, 2, Northwind Traders Syrup, 7.5000
<==      Total: 4
```
### 多排序 ###
我们可以加入多排序，注意这里的顺序是很重要的，排序优先级是按照加入的优先级别来的，就是第一个加入的优先级别最高。
```java
@Test
public void testMultiSort() throws Exception {
        SortDescriptor priceSort =
                new SortDescriptor("price", SortDirection.DESC);
        SortDescriptor idSort =
                new SortDescriptor("productID", SortDirection.DESC);

        Map<String, Object> queryParams =
                mybatisQueryProvider.getSortQueryParamMap(
                        Product.class, "orderExpression", priceSort, idSort);
        northwindDao.getProductByDynamic(queryParams);
}
```
和之前的测试结果有一点不一样了，id 为3,4的产品（价格相同），顺序变化了。
```bash
==>  Preparing: SELECT * FROM product ORDER BY price DESC, product_id DESC 
==> Parameters: 
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE
<==        Row: 1, 1, Northwind Traders Chai, 18.0000
<==        Row: 4, 3, Northwind Traders Olive Oil, 16.5000
<==        Row: 3, 2, Northwind Traders Cajun Seasoning, 16.5000
<==        Row: 2, 2, Northwind Traders Syrup, 7.5000
<==      Total: 4
```
## 结束 ##
和筛选一样，同样可以控制你排序条件的插入。理解筛选了，那么排序也就非常容易类比了。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>