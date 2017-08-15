---
title: Mybatis Dynamic Query 2.0 入门
date: 2017-08-15 14:55:54
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
## 简介 ##
2.0 主要是整合了tk.mybatis.mapper 到项目中去，所以和1.x比起来主要多了一个通用mapper。因为作者主要是使用springboot 这里讲一下Springboot 配法。有问题的话可以参照本章demo。
本章demo地址: https://github.com/wz2cool/mdq2.0test
## 配置步骤 ##
添加依赖
```xml
<!-- 基本库 -->
<dependency>
    <groupId>com.github.wz2cool</groupId>
    <artifactId>mybatis-dynamic-query</artifactId>
    <version>2.0.0</version>
</dependency>
<!-- 主要注册通用mapper -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.1.3</version>
</dependency>
<!-- mybatis 最新版本 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.4</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
<!-- 如果有Spring boot web 自带jackson 这个可以不要,防止版本冲突 -->
<!--  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>-->
<!-- spring boot -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.2.0</version>
</dependency>
```
在applicaiton.properties 注册DynamicQueryMapper, （这里也可以注册自己写的mapper）。
```java
mapper.mappers[0]=com.github.wz2cool.dynamic.mybatis.mapper.DynamicQueryMapper
```
在Application 类中设置扫描mapper包的路径
```java
@SpringBootApplication
@MapperScan(basePackages = "com.github.wz2cool.mdqtest.mapper")
@EnableSwagger2
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
## 2.0 新特性 ##
### 单表无XML ###
主要是tk.mybatis.mapper 已经实现了很多默认通用模板，所以我们无需再去写XML，DynamicQueryMapper 在tk.mybatis.mapper 基础上对筛选使用了我们自己的筛选器进行筛选。  
实体类Product
```java
@Table(name = "product")
public class Product {
    @Id
    @Column(name = "product_id")
    private Integer productId;
    @Column(name = "name")
    private String productName;
    private BigDecimal price;
    private Integer categoryId;
    // get/set...
}
```
对于mapper 我们只要继承一下DynamicQueryMapper 即可。
```java
public interface ProductDao extends DynamicQueryMapper<Product> {
}
```
然后我们就可以看到通用方法已经可使用了
 ![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/20170815143538.png)

### 全程强类型编写 ###
当我们使用最常用的筛选表述器FilterDescriptor/SortDescriptor 的时候，我们需要填写我们对哪个属性进行操作，以前是写一个String，现在我们可以使用表达式编写。
```java
FilterDescriptor nameFilter = new FilterDescriptor(
                User.class, User::getUsername,
                FilterOperator.CONTAINS, "18");
```

### 添加自定义筛选 ###
当FilterDescriptor 和 FilterGroupDescriptor 不能满足我们时候我们需要使用自定义筛选，比如H2数据库在做位运算的时候，需要调用Bitand 方法。
```java
@Test
public void testBitand() throws Exception {
        if ("h2".equalsIgnoreCase(active)) {
            CustomFilterDescriptor bitFilter =
                    new CustomFilterDescriptor(FilterCondition.AND,
                            "Bitand(product_id, {0}) > {1}", 2, 0);
            Map<String, Object> filterParams = MybatisQueryProvider.getWhereQueryParamMap(
                    Product.class, "whereExpression", bitFilter);
            List<Product> products = northwindDao.getProductByDynamic(filterParams);
            assertEquals(true, products.size() > 0);
        } else {
            CustomFilterDescriptor bitFilter =
                    new CustomFilterDescriptor(FilterCondition.AND,
                            "product_id & {0} > {1}", 2, 0);
            Map<String, Object> filterParams = MybatisQueryProvider.getWhereQueryParamMap(
                    Product.class, "whereExpression", bitFilter);
            List<Product> products = northwindDao.getProductByDynamic(filterParams);
            assertEquals(true, products.size() > 0);
        }
}
```
### AS 枚举列 ###
以前我们都是直接 `SELECT * XXX` 来读取数据，这样做有两个非常不好的地方
1. 选择了所有的列，不是每个列都是我们需要的，增加传输时间。
2. 当多表查询的时候，如果你两个表里面有列重名了，这样会有问题。  

MybatisQueryProvider 帮助类中增加columsExpression占位。
```java
Map<String, Object> params = MybatisQueryProvider.getQueryParamMap(dynamicQuery,
    "whereExpression",
    "sortExpression",
    "columnsExpression");
return productViewDao.getProductViewByDynamicQuery(params);
```
在 xml 中的写法
```xml
<select id="getProductViewByDynamicQuery" resultType="com.github.wz2cool.mdqtest.model.entity.view.ProductView">
        SELECT ${columnsExpression} FROM product LEFT JOIN category on product.category_id = category.category_id
        <if test="whereExpression != null and whereExpression != ''">WHERE ${whereExpression}</if>
        <if test="orderExpression != null and orderExpression != ''">ORDER BY ${orderExpression}</if>
</select>
```
输出的时候我们可以看到每个列都被AS 成为对应的属性列了
```bash
==>  Preparing: SELECT product.product_id AS product_id, product.price AS price, category.description AS description, category.name AS category_name, product.name AS product_name, category.category_id AS category_id FROM product LEFT JOIN category on product.category_id = category.category_id WHERE ((product.price > ? AND product.price < ?) AND category.name = ?) 
==> Parameters: 10(Integer), 20(Integer), Condiments(String)
<==    Columns: PRODUCT_ID, PRICE, DESCRIPTION, CATEGORY_NAME, PRODUCT_NAME, CATEGORY_ID
<==        Row: 3, 16.5000, test, Condiments, Northwind Traders Cajun Seasoning, 2
<==        Row: 8, 17.4375, test, Condiments, Northwind Traders Walnuts, 2
<==      Total: 2
```

### json 序列化支持 ###
我们筛选器现在已经支持json 序列化，这就意味着，我们查询可以通过接口完全动态化。当然你也可以把json 放入数据库，当做一个配置来用。
可以运行一下我们的demo，打开swagger: http://localhost:8080/swagger-ui.html
1. 先去获取一下我们示例的json （GET /serialize/getGroupPriceFilters）  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/20170815155941.png)
2. 我们把的出来的json 调用 （POST /data/getProductsByDynamicQuery）  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/20170815160053.png)

## 结束 ##
2.0 更新概括一下：
1. 整合tk.mybatis.mapper, 并自定义 DynamicQueryMapper 通用mapper。
2. 添加自定义筛选 CustomFilterDescriptor。
3. AS 枚举列。
4. json 序列化支持。

## 关注我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>