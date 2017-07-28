---
title: Mybatis Dynamic Query 筛选+排序
date: 2017-07-28 16:32:30
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/  
有了上次我们讲过[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)和[排序](https://wz2cool.github.io/2017/07/28/sort/)的知识，我们就可以组合它们了。
## 准备工作 ##
这里我们沿用[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)里面的准备工作即可。
### 筛选+排序 ###
其实productID 为3和4的价格一样，我们去掉一个，然后再按照价格升序排列。排序和筛选各生成一个map，我们把他们放到同一个map即可。
```java
@Test
public void testFilterSort() throws Exception {
        FilterDescriptor idFilter =
                new FilterDescriptor(FilterCondition.AND,
                        "productID", FilterOperator.NOT_EQUAL, 4);
        SortDescriptor priceSort =
                new SortDescriptor("price", SortDirection.ASC);

        Map<String, Object> filterParams = mybatisQueryProvider.getWhereQueryParamMap(
                Product.class, "whereExpression", idFilter);
        Map<String, Object> sortParams =
                mybatisQueryProvider.getSortQueryParamMap(
                        Product.class, "orderExpression", priceSort);

        Map<String, Object> queryParams = new HashMap<>();
        // 放入到同一个map中去。
        queryParams.putAll(filterParams);
        queryParams.putAll(sortParams);
        northwindDao.getProductByDynamic(queryParams);
}
```
输出结果也是我们期望的，没有id为4的产品了并且价格是按照升序排列的。
```bash
==>  Preparing: SELECT * FROM product WHERE (product_id <> ?) ORDER BY price ASC 
==> Parameters: 4(String)
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE
<==        Row: 2, 2, Northwind Traders Syrup, 7.5000
<==        Row: 3, 2, Northwind Traders Cajun Seasoning, 16.5000
<==        Row: 1, 1, Northwind Traders Chai, 18.0000
<==      Total: 3
```
## 结束 ##
我们大部分对数据库都是查询操作，至此我相信我们已经可以解决大部分查询问题了。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>