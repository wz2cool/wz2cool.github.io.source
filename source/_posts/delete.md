---
title: Mybatis Dynamic Query 删除
date: 2017-07-31 15:13:50
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/
## 简介 ##
删除和更新真的是非常的相似,主要是通过动态查询去定位我们要删除的数据，唯一不同是参数，update 传入的是一个实体，delete 传入的是一个实体的类。一个简单例子就可以说明用法。
## 准备工作 ##
这里我们沿用[简单筛选](https://wz2cool.github.io/2017/07/25/filterBase/)里面的准备工作即可。
## 开始删除 ##
dao 中声明delete
```java
@Mapper
public interface NorthwindDao {
    int delete(Map<String, Object> params);
}
```
xml 中映射
```xml
<delete id="delete" parameterType="java.util.Map">
    ${deleteExpression}
</delete>
```
通过动态查询定位我们要删除的数据。
```java
@Test
public void testDelete() throws Exception {
        // 删除掉id 为1的产品。
        FilterDescriptor idFilter =
                new FilterDescriptor(FilterCondition.AND,
                        Product.class, Product::getProductID,
                        FilterOperator.EQUAL, 1);

        ParamExpression paramExpression =
                mybatisQueryProvider.getDeleteExpression(Product.class, idFilter);
        Map<String, Object> updateParam = new HashMap<>();
        updateParam.put("deleteExpression", paramExpression.getExpression());
        updateParam.putAll(paramExpression.getParamMap());

        int result = northwindDao.delete(updateParam);
        assertEquals(1, result);
}
```
输出结果
```bash
==>  Preparing: DELETE FROM product WHERE (product_id = ?) 
==> Parameters: 1(String)
<==    Updates: 1
```

## 结束 ##
删除基本上和更新一模一样，有了动态查询知识，我们就可以方便定位我们要删除的数据即可。

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>