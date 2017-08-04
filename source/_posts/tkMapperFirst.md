---
title: 初识 tk.mybatis.mapper
date: 2017-08-04 15:23:48
tags:
- tk.mybatis.mapper
- java
---
在博客园发表Mybatis Dynamic Query后，一位园友问我知不知道通用mapper，仔细去找了一下，还真的有啊，比较好的就是abel533写的[tk.mybatis.mapper](https://github.com/abel533/Mapper)。  
本次例子地址：https://github.com/wz2cool/tk-mybatis-demo

## 传统Mybatis用法 ##
### Spring boot ###
引用基本的jar到pom
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>1.5.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>1.5.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```
### sql 数据准备 ###
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
  (4, 3, 'Northwind Traders Olive Oil', 16.5000),
  (5, 3, 'Northwind Traders Olive Oil2', 16.5000);
```
### entity ###
```java
public class Product {
    private Integer productID;
    private String productName;
    private BigDecimal price;
    private Integer categoryID;
    // get/set...
}

@Table(name = "category")
public class Category {
    @Id
    @Column(name = "category_id")
    private Integer categoryID;
    private String categoryName;
    private String description;
    // get /set...
}
```
### Dao ###
这里的ProductDao 是传统的mybatis的用法。
```java
@Mapper
public interface ProductDao {
    List<Product> getProducts();
}
```
### mapper ###
传统mybatis 我们必须有个xml 文件和Dao 对应起来, tk.maybatis.mapper无需此文件。
```xml
<mapper namespace="com.github.wz2cool.demo.tk.mybatis.mapper.ProductDao">
    <select id="getProducts" resultType="com.github.wz2cool.demo.tk.mybatis.model.entity.table.Product">
        SELECT * FROM product
    </select>
</mapper>
```
### 设置扫描包 ###
```java
@SpringBootApplication
@MapperScan(basePackages = "com.github.wz2cool.demo.tk.mybatis.mapper")
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
```
### application.properies ###
注意这里的配置只是针对我们传统mybatis配置，对于tk.maybatis.mapper可以省略这里的配置。
```bash
mybatis.type-aliases-package=com.github.wz2cool.demo.tk.mybatis.mapper
mybatis.mapper-locations=classpath:com.github.wz2cool.demo.tk.mybatis.mapper/*.xml
```
### 测试 ###
我们可以简单调用一下
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes = TestApplication.class)
public class SimpleTest {

    @Autowired
    private ProductDao productDao;

    @Test
    public void testSelect() throws Exception {
        List<Product> productList = productDao.getProducts();
        assertEquals(true, productList.size() > 0);
    }
}
```
我们可以看到输出结果，基本上就通了
```bash
==>  Preparing: SELECT * FROM product 
==> Parameters: 
<==    Columns: PRODUCT_ID, CATEGORY_ID, PRODUCT_NAME, PRICE
<==        Row: 1, 1, Northwind Traders Chai, 18.0000
<==        Row: 2, 2, Northwind Traders Syrup, 7.5000
<==        Row: 3, 2, Northwind Traders Cajun Seasoning, 16.5000
<==        Row: 4, 3, Northwind Traders Olive Oil, 16.5000
<==        Row: 5, 3, Northwind Traders Olive Oil2, 16.5000
<==      Total: 5
```
## tk.mybatis.mapper 用法 ##
### 添加引用 ###
需要多添加两个引用 (ps:曾经少了一个mapper-spring-boot-starter,报错死活找不到为什么--!!!)
```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>3.4.2</version>
</dependency>
<!--mapper-->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.1.3</version>
</dependency>
```
### 继承通用mapper ###
嗯 是的没有xml 文件。
```java
public interface CategoryDao extends Mapper<Category> {
}
```
### 测试调用 ###
dao 里面自带很多方法，比如 selectAll(), insert().
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes = TestApplication.class)
public class SimpleTkMapperTest {

    @Autowired
    private CategoryDao categoryDao;

    @Test
    public void selectAllTest() {
        List<Category> categories = categoryDao.selectAll();
        assertEquals(true, categories.size() > 0);
    }

    @Test
    public void insertTest() {
        Category newCategory = new Category();
        newCategory.setCategoryID(1000);
        newCategory.setCategoryName("test");
        newCategory.setDescription("for test");
        int result = categoryDao.insert(newCategory);
        assertEquals(1, result);
    }
}
```
输出结果
```bash
==>  Preparing: SELECT category_id,category_name,description FROM category 
==> Parameters: 
<==    Columns: CATEGORY_ID, CATEGORY_NAME, DESCRIPTION
<==        Row: 1, Beverages, test
<==        Row: 2, Condiments, test
<==        Row: 3, Oil, test
<==      Total: 3


==>  Preparing: INSERT INTO category ( category_id,category_name,description ) VALUES( ?,?,? ) 
==> Parameters: 1000(Integer), test(String), for test(String)
<==    Updates: 1
```
## tk.mybatis.mapper 初步使用感受 ##
### 优势 ###
1. 无需xml文件和Dao 对应。
2. 无需指定xml 文件位置（因为就没有xml文件）。
3. 自带多种常用方法。 

### 困惑（仅代表个人观点） ###
1. 自带的方法多但是有些方法理解很晦涩，比如 deleteByExample(Object var1)，这里给一个object 我都不知道要填什么，是填写主键呢还是这个实体呢？(后来了解其实example 应该是一个筛选条件)

## tk.mybatis.mapper(Criteria) 筛选 ##
在理解Example 以后，就开始了解了 Criteria 筛选，当然Example 还是可以做排序的。
来看看 tk.mybatis.mapper 筛选是如何做的
```java
@Test
public void selectByExampleTest() {
    Example example = new Example(Category.class);
    Example.Criteria criteria = example.createCriteria();
    criteria.andEqualTo("categoryID", 1);
    criteria.orEqualTo("categoryID", 2);
    categoryDao.selectByExample(example);
}
```
输出结果
```bash
==>  Preparing: SELECT category_id,category_name,description FROM category WHERE ( category_id = ? or category_id = ? ) 
==> Parameters: 1(Integer), 2(Integer)
<==    Columns: CATEGORY_ID, CATEGORY_NAME, DESCRIPTION
<==        Row: 1, Beverages, test
<==        Row: 2, Condiments, test
<==      Total: 2
```

## 结束 ##
这个只是我初次使用tk.mybatis.mapper 确实不错，可以减少工作量，好像还有可以生成Dao和mapper 的工具，这个我以后再试试看。  
### 要有自己的Mapper ###
看了 tk.mybatis.mapper 以后，觉得通用mapper 也应该加入到 Mybatis Dynamic Query 中去，当然实现可能有点不一样。
### 关于 tk.mybatis.mapper 问题 ###
这里真的想问问园友了
1. tk.mybatis.mapper 支持多表筛选么（join），我自己好像没有找到。
2. tk.mybatis.mapper 支持组么，就是类似于 (id > 1 and id < 5) and price = 10. 


## 关注我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>