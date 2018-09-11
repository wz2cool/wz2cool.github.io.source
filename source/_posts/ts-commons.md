---
title: ts-commons 简介
date: 2018-09-11 22:10:44
tags:
  - typescript
---

# 前言

作为一个不经常写 js 的我来说，前端 js 真的是太让我抓狂了，因为真的 js 太随意了，而 typescript 真的是我们这些人的福音啊，但是 typescript 仍然有些东西不习惯，比如我习惯了 C# 和 java 的方法，但是 ts 没有，这样的情况下 [ts-commons](https://github.com/wz2cool/ts-commons) 孕育而生。  
项目地址：https://github.com/wz2cool/ts-commons

# Utils 工具类

集成了常用的帮助方法，方便使用。

## ArrayUtils 数组工具类

主要参照了 C# 中 List 的方法。

| 方法名      | 描述                                            |
| ----------- | ----------------------------------------------- |
| isEmpty     | 判断当前集合是否为空，接受 null 和 undefined    |
| contains    | 判断 item 是否在 array 中                       |
| containsAny | 判断 candidates 中任意一个 item 是否在 array 中 |
| insert      | 向 array 中的索引某个位置 index 插入 item       |
| remove      | 从 array 数组删除 item                          |

## DateUtils 时间工具类

最初设计主要是解决了，springboot 后台给的时间是 timestamp, 需要转化成为 js 的 Date 类型。

| 方法名          | 描述                                                            |
| --------------- | --------------------------------------------------------------- |
| dateToTimestamp | 把当前 js 中的 Date 类型变成 timestamp                          |
| timestampToDate | timestamp 变成 js 中 Date 类型                                  |
| toString        | 参照 C# 时间 toString 格式化当前时区 (yyyy/MM/dd HH:mm:ss.SSS)  |
| toUTCString     | 参照 C# 时间 toString 格式化 UTC 时区 (yyyy/MM/dd HH:mm:ss.SSS) |

## HttpUtils 网络工具类

主要解决获取 cookie 和 url 参数

| 方法名         | 描述                                     |
| -------------- | ---------------------------------------- |
| getCookies     | 从一个 cookie 字符串中得到所有 cookie 值 |
| getQueryParams | 从一个 URL 中获取参数                    |

## NumberUtils 数字工具类

主要设计目的是为了检查是否是整型。

| 方法名        | 描述               |
| ------------- | ------------------ |
| isInteger     | 是否为一个整型     |
| isSafeInteger | 是否为一个有效整型 |

## ObjectUtils 对象工具类

判断一些 any 类型和一些通用对象操作，举个例子 js 有个比较坑的地方就是，我想判断是否为可用的，一般 if(value), 这样可能会有个问题就是 0 也是相当于 false，就被坑到了, 所以可以用 ObjectUtils.isNullOrUndefinend(value) 替代。

| 方法名            | 描述                                                      |
| ----------------- | --------------------------------------------------------- |
| isNull            | 判断当前对象是否为 Null                                   |
| isUndefinend      | 判断当前对象是否为 Undefinend                             |
| isNullOrUndefined | 判断当前对象是否为 Null 或者 Undefinend (推荐经常使用)    |
| isArray           | 判断当前对象是否为数组类型                                |
| isDate            | 判断当前对象是否为时间类型                                |
| isString          | 判断当前对象是否为字符串类型                              |
| isNumber          | 判断当前对象是否为数字类型                                |
| isBoolean         | 判断当前对象是否为布尔类型                                |
| toSafeString      | toString 操作，但是对于 Null 或者 Undefinend 返回空字符串 |
| getProperty       | 利用 key 或者对象的 value 值                              |
| setProperty       | 设置值到对象上                                            |
| createObject      | 利用类型创建对象                                          |
| getPropertyName   | 利用 key 获取对象属性名字 (推荐经常使用, 有类型强检查)    |
