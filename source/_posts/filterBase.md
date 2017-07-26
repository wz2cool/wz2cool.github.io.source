---
title: Mybatis Dynamic Query 简单筛选
date: 2017-07-25 23:52:00
tags: Mybatis Dynamic Query
author: wz2cool
---
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>
在框架中，筛选描述类有两种（FilterDescriptor, FilterGroupDescriptor），这里我们主要举例来说明FilterDescriptor用法。  
FilterDescriptor 定义可以参照：[FilterDescriptor类](https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/filterdescriptor.html)

## 准备工作 ##
创建一张产品表
```sql
CREATE TABLE product (
  product_id    INT PRIMARY KEY,
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