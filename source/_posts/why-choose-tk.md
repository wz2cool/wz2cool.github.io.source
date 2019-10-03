---
title: 为何mybatis-dynamic-query 选择tk.mapper集成
date: 2019-10-03 08:14:22
tags:
  - java
  - Mybatis Dynamic Query
---

# 前言

mybaits-dynamic-query 到了第三个大版本了，之前一直没有讲为什么我在 2.0 集成了 tk.mapper, 这里也和大家聊聊原因吧。

## 专注查询

在第一版本中，我其实常规的增删改都是自己弄得，但是这有个问题就是可能并没有别人做的好，比如 tk.mapper 或者是 mybatis-plus, 选择与其中一个集成是一个非常好的方案，这样可以省时间，而且这两个插件非常优秀。

## license 问题

让位选择 tk.mapper 第一个主要原因就是 license 问题， tk.mapper 的是 MIT 协议，而 mybatis-plus 是 apache 协议， tk.mapper 比起来更加开放，因为不想有任何侵权嫌疑，所以 tk.mapper 非常友好。

## 功能

mybatis-plus 是一个全方位的，重量型产品，也就是说你要的基本他都有，防 sql 注入防火墙，分页，自动生成工具，也有动态查询（既生瑜何生亮！！！），而 tk.mapper 非常轻量，并且提供了扩展的 mapper，DynamicQueryMapper 也是扩展于此， 并且作者也有 PageHelper 插件可以集成，至于 sql 防火墙，我们用的是 druid 连接池也有这个功能，所以大概可以这样看 mybatis-plus = tk.mapper + PageHelper + druid + mybatis-dynamic-query, 但是自己还是比较喜欢轻量可扩展的。

## 良性竞争

适合才是最好的，大家可以自行对比两家阵营的产品，选择自己喜欢的，都非常不错，做动态查询这个项目也是自己业余时间喜欢，并且真正用在我们自己项目中了。
