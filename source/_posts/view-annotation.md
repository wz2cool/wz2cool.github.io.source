---
title: View 注解
date: 2020-04-19 22:58:16
tags:
  - Mybatis Dynamic Query
  - java
  - MDQ
---

# 前言

在之前，我们想连表查询，我们必须要把代码卸载 XML 中，这也是我们需要引入 @View 这个注解。

# 视图

## XML 连表查询

我们先看看以前的做法，我们需要在 XML 中定义

```xml
<select id="getProductViewsByDynamic" parameterType="java.util.Map"
        resultType="com.github.wz2cool.dynamic.mybatis.db.model.entity.view.ProductView">
  SELECT
  <choose>
      <when test="columnsExpression != null and columnsExpression !=''">
          ${columnsExpression}
      </when>
      <otherwise>
          *
      </otherwise>
  </choose>
  FROM product LEFT JOIN category ON product.category_id = category.category_id
  <if test="whereExpression != null and whereExpression != ''">WHERE ${whereExpression}</if>
  <if test="orderExpression != null and orderExpression != ''">ORDER BY ${orderExpression}</if>
</select>
```

在 Mapper 层定义

```java
@Mapper
public interface NorthwindDao {
    List<ProductView> getProductViewsByDynamic(Map<String, Object> params);
}
```

## View 注解

我们只在 Model 上定义即可

```java
@View("product LEFT JOIN category ON product.category_id = category.category_id")
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

    ....
}
```

然后在 mapper 继承一下即可拥有和单表一样的动态查询

```java
@Mapper
public interface ProductViewMapper extends SelectViewByDynamicQueryMapper<ProductView> {
}
```

# 小结

这个功能进一步弱化我们去写 XML，大家可以试用起来
