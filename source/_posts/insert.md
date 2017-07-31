---
title: Mybatis Dynamic Query 插入
date: 2017-07-28 17:48:33
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/
## 问题 ##
mybatis 一个插入一个更新是最让我头痛的，不是多难而是很烦，每次加一个字段必定这两个地方要改，曾经也看过别人写的一个[mybatis-jpa](http://www.cnblogs.com/svili/p/7232323.html#3743019)，觉得不错，里面已经封装了一些常用的方法映射。  
如果用mybatis常用的插入方法有两个：insert和insertSelective，区别在于insert 每个值都会插入，insertSelective只是插非null的值。这里有个小问题，就是insertSelective粒度太大了，是对于所有的值都是插入非null。这里就可以用@Column 中的 insertIfNull 去对每个列进行控制，当然这个后面降到 @Column 用法再细细描述，这里还是用个简单例子来说明insert的用法。
## 准备工作 ##
这里我们沿用[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)里面的准备工作即可。
## 开始插入 ##
### insertSelective ###
默认插入就是insertSelective即，null的时候不插入，比如id 是自增列，我们可以不设置值
```java
 @Test
public void testInsertSelective() throws Exception {
        Product newProduct = new Product();
        newProduct.setCategoryID(1);
        String productName = "Product";
        newProduct.setProductName(productName);

        // insert.
        ParamExpression paramExpression = mybatisQueryProvider.getInsertExpression(newProduct);
        Map<String, Object> insertParam = new HashMap<>();
        insertParam.put("insertExpression", paramExpression.getExpression());
        insertParam.putAll(paramExpression.getParamMap());

        int result = northwindDao.insert(insertParam);
        assertEquals(1, result);
}
```
输出可以看到，我们只是插入有值的数据，price 为null 所以没有插入
```bash
==>  Preparing: INSERT INTO product (product_name, category_id) VALUES (?, ?) 
==> Parameters: Product(String), 1(Integer)
<==    Updates: 1
```
### insert null ###
刚才我们说到默认是null的时候不插入，那么我们无论某个列是否为null都要强制插入应该怎么做呢，这个时候就需要用的 @Column去控制。我们创建一个Product2实体类，唯一和Product不同的是我们强制price为null的时候也插入
```java
@Table(name = "product")
public class Product2 {
    @Column(name = "product_id")
    private Integer productID;
    private String productName;
    @Column(insertIfNull = true)
    private BigDecimal price;
    private Integer categoryID;

    // get/set
}
```
基本和之前插入测试代码一样
```java
@Test
public void testInsertNull() throws Exception {
        Product2 newProduct = new Product2();
        newProduct.setCategoryID(1);
        String productName = "Product";
        newProduct.setProductName(productName);

        // insert.
        ParamExpression paramExpression = mybatisQueryProvider.getInsertExpression(newProduct);
        Map<String, Object> insertParam = new HashMap<>();
        insertParam.put("insertExpression", paramExpression.getExpression());
        insertParam.putAll(paramExpression.getParamMap());

        int result = northwindDao.insert(insertParam);
        assertEquals(1, result);
}
```
输出我们看到，price是null也被强制放入插入字段中
```bash
==>  Preparing: INSERT INTO product (price, product_name, category_id) VALUES (?, ?, ?) 
==> Parameters: null, Product(String), 1(Integer)
<==    Updates: 1
```

## 结束 ##
因为没有写死所有的属性在XML中，所以以后添加字段，修改字段删除字段都不用再改XML文件，并且用@Column去控制InsertSelective，可以达到更小的粒度控制。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>