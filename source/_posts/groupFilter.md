---
title: Mybatis Dynamic Query 组筛选
date: 2017-07-28 14:27:08
tags: 
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/  
上次我们讲过[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/), 这次我们举例来说明 FilterGroupDescriptor用法。
FilterGroupDescriptor 定义可以参照： [FilterGroupDescriptor类](https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/filtergroupdescriptor.html)

## 准备工作 ##
这里我们沿用[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)里面的准备工作即可。
## 开始筛选 ###
测试放入两个简单的Id筛选到同一个Id组筛选中去。
```java
@Test
public void testGroupFilter() throws Exception {
        FilterGroupDescriptor groupIdFilter = new FilterGroupDescriptor();
        FilterDescriptor idFilter1 =
                new FilterDescriptor(FilterCondition.AND,
                        "productID", FilterOperator.GREATER_THAN, "1");
        FilterDescriptor idFilter2 =
                new FilterDescriptor(FilterCondition.AND,
                        "productID", FilterOperator.LESS_THAN, "4");
        // 把两个 id 筛选当成一个放到同一个组
        groupIdFilter.addFilters(idFilter1, idFilter2);
        // 单独一个简单筛选。
        FilterDescriptor priceFilter =
                new FilterDescriptor(FilterCondition.AND,
                        "price", FilterOperator.GREATER_THAN, 10);

        Map<String, Object> queryParams =
                mybatisQueryProvider.getWhereQueryParamMap(
                        Product.class, "whereExpression", groupIdFilter, priceFilter);
        
        northwindDao.getProductByDynamic(queryParams);
}
```
结果输出看到，两个id筛选被放到同一个括号中去了。
```bash
==>  Preparing: SELECT * FROM product WHERE ((product_id > ? AND product_id < ?) AND price > ?) 
==> Parameters: 1(String), 4(String), 10(String)
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE
<==        Row: 3, 2, Northwind Traders Cajun Seasoning, 16.5000
<==      Total: 1
```
## 结束 ##
有了前面[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)的基础，理解这个就非常容易了,
我常常是把同属性的筛选放到一起，这样看上去的筛选条件比较清晰。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>