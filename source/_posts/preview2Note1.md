---
title: 开始构思2.0
date: 2017-08-08 14:16:56
tags:
- java
- MDQ 2.0 杂记
---
## 说说1.0 ##
&nbsp;&nbsp;&nbsp;&nbsp;Mybatis Dynamic Query(MDQ)发布两个版本了，其实MDQ已经在作者实际应用中用到了，并且极大的提高了生产效率。其实1.0 已经可以完全够用在我们Mybatis实际应用。并且极大规避了xml接口数量爆炸。但是限于本人java接触时间短，很多东西考虑不周全，加上有很多很好的框架没有仔细研究，也许知道不足了，才会有提高。

## tk.myhbatis.mapper VS MDQ ##
&nbsp;&nbsp;&nbsp;&nbsp;不得不说是个很好的框架，也发过邮件问过abel533作者一些问题。既然tk.mybatis这么优秀了，还要MDQ干什么呢？通用tk.mybatis.mapper相关操作都是单表, 因为insert/update/delete 这三个操作都是单表的，所以tk.mybatis.mapper的select必须也是单表操作。但是select可能是多表的，而且我们常用的80%的操作都是select操作，MDQ框架设计之初就是关注查询，并且可以支持多表join，我认为这个就是MDQ最大的意义所在。

## tk.mybatis.mapper 整合目的 ##
&nbsp;&nbsp;&nbsp;&nbsp;既然前面说到tk.mybatis.mapper 既然已经对单表支持的非常好了，我们insert/update/delete 完全可以交给它来处理，tk.mybatis.mapper 的查询筛选是使用Example 来进行筛选的，如果你使用MDQ，还要学习Example 这个对于使用者来说是一个负担，我们只扩展select部分，让他MDQ化。

## 整合方案 ##
### JPA 统一标准 ###
&nbsp;&nbsp;&nbsp;&nbsp;因为tk.mybatis.mapper 使用的jpa 标准库（javax.persistence），所以我也去除了@Column和@Table，既然insert/update/delete 交由tk.mybatis.mapper 处理了。

### 去除了DatabaseType ###
&nbsp;&nbsp;&nbsp;&nbsp;设计之初本来是想统一处理各个数据库不同的，比如大多数数据库对于查询条件都是隐形转化的，比如你一个DateTime列，你用一个String 可以查询到，但是到了postresql就有问题了，postresql必须保证类型一致性。再比如一个位运算，大多数数据库直接一个 & 符号就可以了，但是到了H2数据库必须调用BITAND方法才能够完成。这样的处理所有的情况就变成一个很大的工程。   
&nbsp;&nbsp;&nbsp;&nbsp;去除以后会加上一个CustomFilterDescriptor 让用户自己处理一个自定义查询，这样可扩展性高，而且框架也不用处理很多特殊情况了。