---
title: mdq2.2
date: 2017-10-05 14:54:48
tags:
- Mybatis Dynamic Query
- java
- MDQ
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/
## 杂谈 ##
不知道有多少人在用MDQ(Mybatis Dynamic Query),反正我一直是在我们公司自己项目里用的，总之是不会坑大家的啊。还有毕竟作者也是入java坑不久，欢迎java大神前来指导。

## 面对质疑 ##
当然有人一开始真心接受不了我这套，无非就几点：
- 问题1：来个新人还要学习MDQ，只理解Mybatis上手要容易的多。  
反驳：MDQ没有产生一套新的东西，而是基于Mybatis的一个扩展，理解MDQ对Mybatis更加有帮助。
- 问题2：我直接在mybatis里面写sql感觉写起来比较快。
反驳：一开始你是写起来快，最开始你只写几个查询，随着业务扩展，你要写的筛选和排序会越来越多，更可怕的事，你全部写在xml中，想想你以后要维护一个弱类型的东西...
- 问题3：你写的东西感觉不靠谱。
反驳：大神写的同样有bug，好的代码需要反复重构，并且保持良好的单元测试，这个是不变的真理，有问题可以，大家讨论一下解决即可。

## 2.2 更新 ##
更新了三点：  
1. 原来是有自定义筛选的，现在添加了自定义排序。
2. 设计MDQ的目的就是为了可维护性，所以这次添加了获取查询列，即[tableName].[columnName] 这种形式。
3. 链式调用，等下看例子就明白了。

### 查询列 ###
这个查询列大家可能有点云里雾里，列就叫列，为什么叫查询列。其实MDQ工作的时候对数据库查询是[tableName].[columnName], 这样做有个好处就是当我们join表的时候，我们可以指定这个列是哪里来的，保证查询的正确性。  
当然你这个拼接工作是由MDQ完成的，我们只需要在@Column 这个注解设计 tableOrAlias 和name 两个属性即可。当然为什么要这个查询列，我在下面的自定义排序中说明。  
下面我们来看个例子，还是用我们经典的ProductView。
```java
public class ProductView {
    @Column(name = "product_id", table = "product")
    private Long productID;
    @Column(name = "product_name", table = "product")
    private String productName;
    @Column(name = "price", table = "product")
    private BigDecimal price;

    @Column(name = "category_id", table = "category")
    private Long categoryID;
    @Column(name = "category_name", table = "category")
    private String categoryName;
    @Column(name = "description", table = "category")
    private String description;
    
    ...
}
```
```java
@Test
public void testGetQueryColumn() throws Exception {
    String productIdColumn = MybatisQueryProvider.getQueryColumn(
            ProductView.class, productView -> productView.getProductID());
    // 这里我们就可以看到期望的输出结果就是 [tableName].[columnName]
    assertEquals("product.product_id", productIdColumn);
}
```

### 自定义筛选 ###
其实和自定义筛选类似，就是我们可以hardcode 一小段我们自己的sql（虽然觉得不是很好）。  
当然下面例子也会使用到查询列。  
现在我们有个需求，我们需要把ProductID是2的产品放到第一个，常规的升序降序都不能满足我们，我们该怎么办呢，恩 我们重新自定义一下排序权重即可。
```java
@Test
public void testCustomSort() throws Exception {
    String idQueryColumn = MybatisQueryProvider.getQueryColumn(ProductView.class, ProductView::getProductID);
    // NOTE: queryColumn cannot be parameter.
    // 这里注意：列不能当做参数，否则会报错，所以我们字符串拼接出来。
    String customSortExpression =
            String.format("CASE %s WHEN {0} THEN {1} ELSE product.product_id END DESC", idQueryColumn);
    CustomSortDescriptor id2TopSort = new CustomSortDescriptor();
    id2TopSort.setExpression(customSortExpression);
    // 当id是2的时候，我们权重直接给int 最大值
    id2TopSort.setParams(2, Integer.MAX_VALUE);
    Map<String, Object> queryParam =
        MybatisQueryProvider.createInstance(ProductView.class)
            .addSorts("orderExpression", id2TopSort)
            .toQueryParam();
    List<ProductView> productList = northwindDao.getProductViewsByDynamic(queryParam);
    assertEquals(Long.valueOf(2), productList.get(0).getProductID());
}
```
我们看一下输出结果，2的产品排序到了最上面去了。  
```hash
==>  Preparing: SELECT * FROM product LEFT JOIN category ON product.category_id = category.category_id ORDER BY CASE product.product_id WHEN ? THEN ? ELSE product.product_id END DESC 
==> Parameters: 2(Integer), 2147483647(Integer)
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE, CATEGORY_ID, CATEGORY_NAME, DESCRIPTION
<==        Row: 2, 2, Northwind Traders Syrup, 7.5000, 2, Condiments, test
<==        Row: 4, 3, Northwind Traders Olive Oil, 16.5000, 3, Oil, test
<==        Row: 3, 2, Northwind Traders Cajun Seasoning, 16.5000, 2, Condiments, test
<==        Row: 1, 1, Northwind Traders Chai, 18.0000, 1, Beverages, test
<==      Total: 4
```

### 链式调用 ###
这个应该是我在哪本java 文章看过，反正感觉不错啊。  
以前的调用我们非要组织一个DynamicQuery类，然后调用一个静态方法生成一个参数Map，有了链式调用我们写代码就方便了很多。    
看我们一个步骤，创建帮助类-》添加筛选-》添加排序-》转化成参数。
```java
@Test
public void testMultiTablesFilter() throws Exception {
        FilterDescriptor priceFilter1 =
                new FilterDescriptor(ProductView.class, ProductView::getPrice,
                        FilterOperator.GREATER_THAN_OR_EQUAL, 6);
        FilterDescriptor priceFilter2 =
                new FilterDescriptor(ProductView.class, ProductView::getPrice,
                        FilterOperator.LESS_THAN, 10);
        FilterDescriptor categoryNameFilter =
                new FilterDescriptor(ProductView.class, ProductView::getCategoryName,
                        FilterOperator.START_WITH, "Co");

        SortDescriptor idDescSort =
                new SortDescriptor(ProductView.class, ProductView::getProductID, SortDirection.DESC);

        Map<String, Object> params =
                // NOTE: we recommend you to set "columnsExpressionPlaceholder"
                // in case of duplicated column name in two tables.
                // 这里你也可以不给列的站位，但是推荐使用，防止两个表有重复的名字
                MybatisQueryProvider
                        .createInstance(ProductView.class, "columnsExpression")
                        .addFilters("whereExpression",
                                priceFilter1, priceFilter2, categoryNameFilter)
                        .addSorts("orderExpression", idDescSort)
                        .toQueryParam();

        List<ProductView> result = northwindDao.getProductViewsByDynamic(params);
        assertEquals(true, result.size() > 0);
}
```
看一下输出结果
```hash
==>  Preparing: SELECT product.product_id AS product_id, product.price AS price, category.description AS description, category.category_name AS category_name, product.product_name AS product_name, category.category_id AS category_id FROM product LEFT JOIN category ON product.category_id = category.category_id WHERE (product.price >= ? AND product.price < ? AND category.category_name LIKE ?) ORDER BY product.product_id DESC 
==> Parameters: 6(Integer), 10(Integer), Co%(String)
<==    Columns: PRODUCT_ID, PRICE, DESCRIPTION, CATEGORY_NAME, PRODUCT_NAME, CATEGORY_ID
<==        Row: 2, 7.5000, test, Condiments, Northwind Traders Syrup, 2
<==      Total: 1
```

## 结束 ##
利用十一的时间更行了2.0.2，欢迎大家节后回来使用。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>

## 文章整合 ##
[Mybatis Dynamic Query 简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)
[Mybatis Dynamic Query 组筛选](https://wz2cool.github.io/2017/07/28/groupFilter/)
[Mybatis Dynamic Query 排序](https://wz2cool.github.io/2017/07/28/sort/)
[Mybatis Dynamic Query 筛选+排序](https://wz2cool.github.io/2017/07/28/filterSort/)
[Mybatis Dynamic Query 插入](https://wz2cool.github.io/2017/07/28/insert/)
[Mybatis Dynamic Query 更新](https://wz2cool.github.io/2017/07/31/update/)
[Mybatis Dynamic Query 删除](https://wz2cool.github.io/2017/07/31/delete/)
[Mybatis Dynamic Query 属性表达式](https://wz2cool.github.io/2017/07/31/propertyExpression/)
[Mybatis Dynamic Query join视图](https://wz2cool.github.io/2017/07/31/joinView/)
[Mybatis Dynamic Query 2.0 入门](https://wz2cool.github.io/2017/08/15/howToUse2/)