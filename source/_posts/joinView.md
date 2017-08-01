---
title: Mybatis Dynamic Query join视图
date: 2017-07-31 23:44:01
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/
## 简介 ##
前面基本增删改查完成，现在可以到渐进阶段。在实际查询中，我们不可能只查询一张表，我们肯定要连表查询，所以我们相当于在查询一个视图，那么和简单的单表查询有什么区别呢？这里我们就必须要用到 @Column 中tableOrAlias去指定这个列来自于哪张表，具体我们还是看例子吧。
## 准备工作 ##
创建两张表，一张产品类型表，一张产品表
```sql
DROP TABLE IF EXISTS category;
CREATE TABLE category (
  category_id   INT PRIMARY KEY,
  category_name VARCHAR (50) NOT NULL,
  description   VARCHAR (100)
);

DROP TABLE IF EXISTS product;
CREATE TABLE product (
  product_id    INT PRIMARY KEY auto_increment,
  category_id   INT NOT NULL,
  product_name  VARCHAR (50) NOT NULL,
  price         DECIMAL
);
DELETE FROM category;
INSERT INTO category (category_id, category_name, description) VALUES
  (1, 'Beverages', 'test'),
  (2, 'Condiments', 'test'),
  (3, 'Oil', 'test');

DELETE FROM product;
INSERT INTO product (product_id, category_id, product_name, price) VALUES
  (1, 1, 'Northwind Traders Chai', 18.0000),
  (2, 2, 'Northwind Traders Syrup', 7.5000),
  (3, 2, 'Northwind Traders Cajun Seasoning', 16.5000),
  (4, 3, 'Northwind Traders Olive Oil', 16.5000);
```
关键的地方来了，我们创建个ProductView的实体类, 这里我们指定了每列来自于那长表
```java
public class ProductView {
    // 这三个列来自于产品表
    @Column(name = "product_id", tableOrAlias = "product")
    private Long productID;
    @Column(name = "product_name", tableOrAlias = "product")
    private String productName;
    @Column(name = "price", tableOrAlias = "product")
    private BigDecimal price;
    // 这三个列来自于类别表
    @Column(name = "category_id", tableOrAlias = "category")
    private Long categoryID;
    @Column(name = "category_name", tableOrAlias = "category")
    private String categoryName;
    @Column(name = "description", tableOrAlias = "category")
    private String description;
    // get/set...
}
```
Dao 接口, 和其他动态查询是一样的。
```java
@Mapper
public interface NorthwindDao {
    List<ProductView> getProductViewsByDynamic(Map<String, Object> params);
}
```
xml 中这里注意了，我们并没有给表别别名，所以在 on 的时候一定是 [表名].[列名]
```xml
<select id="getProductViewsByDynamic" parameterType="java.util.Map"
            resultType="com.github.wz2cool.dynamic.mybatis.db.model.entity.view.ProductView">
        SELECT * FROM product LEFT JOIN category ON product.category_id = category.category_id
        <if test="whereExpression != null and whereExpression != ''">WHERE ${whereExpression}</if>
        <if test="orderExpression != null and orderExpression != ''">ORDER BY ${orderExpression}</if>
</select>
```
## 开始查询 ##
其实当我们定义好类中每个属性来自于哪张表以后，其他的和单表动态查询就一样了
```java
@Test
public void testEndWith() throws Exception {
        FilterDescriptor nameFilter =
                new FilterDescriptor(FilterCondition.AND, "categoryName", FilterOperator.END_WITH, "l");
        Map<String, Object> queryParams =
                mybatisQueryProvider.getWhereQueryParamMap(
                        ProductView.class, "whereExpression", nameFilter);
        List<ProductView> productViews = northwindDao.getProductViewsByDynamic(queryParams);
        assertEquals("Oil", productViews.get(0).getCategoryName());
}
```
输出结果我们发现，每个查询的的列都带上了表名（别名也可以）
```bash
==>  Preparing: SELECT * FROM product LEFT JOIN category ON product.category_id = category.category_id WHERE (category.category_name LIKE ?) 
==> Parameters: %l(String)
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE, CATEGORY_ID, CATEGORY_NAME, DESCRIPTION
<==        Row: 4, 3, Northwind Traders Olive Oil, 16.5000, 3, Oil, test
<==      Total: 1
Closing non transact
```

## 结束 ##
在类中指定每个属性来自于哪个表中的那个列的映射是这个的核心。这里并不推荐使用别名类似（t1,t2）这种去设置 tableOrAlias，为什么呢？因为没有必要而且会影响你阅读代码，t1 是哪个表你要去猜测，所以推荐还是直接使用表名即可。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>