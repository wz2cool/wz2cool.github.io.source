---
title: Mybatis Dynamic Query 更新
date: 2017-07-31 15:13:43
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/
## 简介 ##
更新和插入的问题其实是一样的，基本上我们可以解决方案也是类似的，唯一的不同就是，一般更新的时候我们都是带筛选条件的。常用我们都是通过ID筛选去找记录，但是现在有了前面的知识，这个筛选条件真的是so easy!!!   
关于粒度的控制，同样可以使用@Column 中的 updateIfNull 标记来达到 updateSelective效果。  
废话不多说上代码
## 准备工作 ##
这里我们沿用[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)里面的准备工作即可。
### update ###
和insert 相反，默认是null的时候我们不更新。
```java
@Test
public void testUpdate() throws Exception {
        // 查找ID筛选
        // 这里我们使用表达式去设置propertyPath.
        FilterDescriptor idFilter =
                new FilterDescriptor(FilterCondition.AND,
                        Product.class, Product::getProductID,
                        FilterOperator.EQUAL, 1);

        Product newProduct = new Product();
        // only update product name
        // 这里我们只更新产品名字，所以只设置产品名。
        String productName = "modifiedName";
        newProduct.setProductName(productName);

        ParamExpression paramExpression =
                mybatisQueryProvider.getUpdateExpression(newProduct, idFilter);
        Map<String, Object> updateParam = new HashMap<>();
        updateParam.put("updateExpression", paramExpression.getExpression());
        updateParam.putAll(paramExpression.getParamMap());

        int result = northwindDao.update(updateParam);
        assertEquals(1, result);
}
```
在XML写好与之对应的update
```xml
<update id="update" parameterType="java.util.Map">
    ${updateExpression}
</update>
```
输出结果我们可以看到，我们只更新了产品名称，并且是通过id 找到对应的记录。
```bash
==>  Preparing: UPDATE product SET `product_name`=? WHERE (product_id = ?) 
==> Parameters: modifiedName(String), 1(String)
<==    Updates: 1
```
### update Null ###
更新的时候默认是null的时候不更新，那么我们可以强制更新null的列。我们创建一个Product3实体类，唯一和Product不同就是在price列上加上了强制更新。
```java
@Table(name = "product")
public class Product3 {
    // id 为null 的时候不插入
    @Column(name = "product_id")
    private Integer productID;
    private String productName;
    // 强制更新price列，无论是否为null
    @Column(updateIfNull = true)
    private BigDecimal price;
    private Integer categoryID;
    // get/set...
}
```
更新操作不变
```java
@Test
public void testUpdateNull() throws Exception {
        // 查找ID筛选
        // 这里我们使用表达式去设置propertyPath.
        FilterDescriptor idFilter =
                new FilterDescriptor(FilterCondition.AND,
                        Product.class, Product::getProductID,
                        FilterOperator.EQUAL, 1);

        Product3 newProduct = new Product3();
        // only update product name
        // 这里我们只更新产品名字，所以只设置产品名。
        String productName = "modifiedName";
        newProduct.setProductName(productName);

        ParamExpression paramExpression =
                mybatisQueryProvider.getUpdateExpression(newProduct, idFilter);
        Map<String, Object> updateParam = new HashMap<>();
        updateParam.put("updateExpression", paramExpression.getExpression());
        updateParam.putAll(paramExpression.getParamMap());

        int result = northwindDao.update(updateParam);
        assertEquals(1, result);
}
```
输出结果发现price被更新成为了null
```bash
==>  Preparing: UPDATE product SET `price`=?, `product_name`=? WHERE (product_id = ?) 
==> Parameters: null, modifiedName(String), 1(String)
<==    Updates: 1
```

## 结束 ##
更新和插入基本操作是一样的，唯一就是多了后面支持动态查询。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>