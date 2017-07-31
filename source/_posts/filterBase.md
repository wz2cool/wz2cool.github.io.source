---
title: Mybatis Dynamic Query 简单筛选
date: 2017-07-25 23:52:00
tags: 
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/  
在框架中，筛选描述类有两种（FilterDescriptor, FilterGroupDescriptor），这里我们主要举例来说明FilterDescriptor用法。  
FilterDescriptor 定义可以参照：[FilterDescriptor类](https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/filterdescriptor.html)

## 准备工作 ##
创建一张产品表
```sql
CREATE TABLE product (
  product_id    INT PRIMARY KEY auto_increment,
  category_id   INT NOT NULL,
  product_name  VARCHAR (50) NOT NULL,
  price         DECIMAL
);
```
添加测试数据
```sql
INSERT INTO product (product_id, category_id, product_name, price) VALUES
  (1, 1, 'Northwind Traders Chai', 18.0000),
  (2, 2, 'Northwind Traders Syrup', 7.5000),
  (3, 2, 'Northwind Traders Cajun Seasoning', 16.5000),
  (4, 3, 'Northwind Traders Olive Oil', 16.5000);
```
添加Entity，Entity定义参照： [基本概念](https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/definition.html) 
```java
// 类名和表名做了映射 
@Table(name = "product")
public class Product {
    // 这里的productID 属性做了和数据库列的映射
    @Column(name = "product_id")
    private Integer productID;
    private String productName;
    private BigDecimal price;
    private Integer categoryID;

    // get/set ...
}
```
添加Mapper
```java
@Mapper
public interface NorthwindDao {
    List<Product> getProductByDynamic(Map<String, Object> params);
}
```
添加到xml
```xml
<select id="getProductByDynamic" parameterType="java.util.Map"
         resultType="com.github.wz2cool.dynamic.mybatis.db.model.entity.table.Product">
    SELECT * FROM product
    <if test="whereExpression != null and whereExpression != ''">WHERE ${whereExpression}</if>
    <if test="orderExpression != null and orderExpression != ''">ORDER BY ${orderExpression}</if>
</select>
```
## 开始筛选 ##
### 简单id 筛选 ###
```java
@Test
public void simpleDemo() throws Exception {
    FilterDescriptor idFilter =
        new FilterDescriptor(FilterCondition.AND, 
            "productID", FilterOperator.GREATER_THAN_OR_EQUAL, 2);
    Map<String, Object> queryParams =
        mybatisQueryProvider.getWhereQueryParamMap(
            roduct.class, "whereExpression", idFilter);
    northwindDao.getProductByDynamic(queryParams);
}
```
结果输出，这里其实已经可以看到了动态查询拼接的sql其实是站位符（防止sql注入）。
```bash
==>  Preparing: SELECT * FROM product WHERE (product_id >= ?) 
==> Parameters: 2(String)
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE
<==        Row: 2, 2, Northwind Traders Syrup, 7.5000
<==        Row: 3, 2, Northwind Traders Cajun Seasoning, 16.5000
<==        Row: 4, 3, Northwind Traders Olive Oil, 16.5000
<==      Total: 3
```
### 多筛选 ###
```java
@Test
public void multiFilterDemo() throws Exception {
        FilterDescriptor idFilter =
                new FilterDescriptor(FilterCondition.AND,
                        "productID", FilterOperator.GREATER_THAN_OR_EQUAL, 2);
        FilterDescriptor priceFilter =
                new FilterDescriptor(FilterCondition.AND,
                        "price", FilterOperator.LESS_THAN, 15);

        Map<String, Object> queryParams =
                mybatisQueryProvider.getWhereQueryParamMap(
                        Product.class, "whereExpression", idFilter, priceFilter);
        Product productView =
                northwindDao.getProductByDynamic(queryParams).stream().findFirst().orElse(null);
        assertEquals(Integer.valueOf(2), productView.getProductID());
}
```
很容易多加了一个priceFilter, 结果输出如下：
```bash
==>  Preparing: SELECT * FROM product WHERE (product_id >= ? AND price < ?) 
==> Parameters: 2(String), 15(String)
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE
<==        Row: 2, 2, Northwind Traders Syrup, 7.5000
<==      Total: 1
```
## 结束 ##
用最简单两个例子大概入了一个门，是不是很简单？能想到的有几个应用场景：
1. Filter 是最后动态加上去的，所以你可以在你的代码中任意地方根据你的条件生成Filter。
2. 可以剥离生成Filter以达到Filter复用性。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>