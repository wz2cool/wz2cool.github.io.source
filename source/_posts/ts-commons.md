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
| hasValue          | 判断当前对象是否有值, 是isNullOrUndefined 取反           |
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

## RegexUtils 正则工具类

常用的正则表达式

| 方法名           | 描述             |
| ---------------- | ---------------- |
| escapeRegExp     | 转义正则特殊符号 |
| validateUsername | 验证用户名       |
| validatePassword | 验证密码         |
| validateEmail    | 验证邮箱         |

## StringUtils 字符串工具类

核心帮助类，主要参照了 java 中的 Apache Commons Lang 3.8 API，每个方法都是经常用的。

| 方法名              | 描述                                                           |
| ------------------- | -------------------------------------------------------------- |
| isEmpty             | 判断字符串是否为空                                             |
| isNotEmpty          | 判断字符串不为空                                               |
| isBlank             | 判断字符串是否为空，并且空格也理解为空                         |
| isNotBlank          | 判断字符串是否不为空，并且空格也理解为空                       |
| trim                | 去掉前后空格                                                   |
| trimToNull          | 去掉前后空格，如果为空返回 Null                                |
| trimToEmpty         | 去掉前后空格，如果为空返回 ""                                  |
| strip               | 去掉前后待选字符                                               |
| equals              | 比较两个字符串是否相等，接受 null 和 undefined                 |
| equalsIgnoreCase    | 比较两个字符串是否相等，并且忽略大小写，接受 null 和 undefined |
| indexOf             | 第一个满足查找字符串的位置                                     |
| lastIndexOf         | 最后一个满足查找字符串的位置                                   |
| contains            | 是否包含查询字符串                                             |
| containsIgnoreCase  | 是否包含查询字符串，并且忽略大小写                             |
| subString           | 从当前字符串中截取一段子集                                     |
| startWith           | 是否以某个字符串开头                                           |
| startWithIgnoreCase | 是否以某个字符串开头, 并且忽略大小写                           |
| endWith             | 是否以某个字符串结尾                                           |
| endWithIgnoreCase   | 是否以某个字符串结尾， 并且忽略大小写                          |
| endWith             | 是否以某个字符串结尾                                           |
| isWhitespace        | 是否是一个空格类型                                             |
| newGuid             | 创建一个唯一表示字符串，类似 uuid， （推荐经常使用）           |
