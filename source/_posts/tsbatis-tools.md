---
title: tsbatis-tools 批量Entity导出
date: 2017-11-23 17:51:57
tags: 
- tsbatis
---
# 前言
tsbatis 现在还在preview 版本，大量的测试工作还没有完成，但是已经发现一个问题，就是每次写entity会很浪费时间，为了提高效率，我们开发了一个工具来解决这个问题。目前可以支持sqlite和mysql 批量导出entity。

# TsBatis-Tools
## 下载
tsbatis-tools 基于 express 开发，所以我们可以再github 上面下载源码：https://github.com/wz2cool/tsbatis-tools

## 运行
运行程序也非常简单, 和node 程序运行一致。  
`$ npm install`  
`$ npm start`  
然后我们就可以打开 http://localhost:3000 访问即可。


## sqlite 支持
### 文件选择
对于sqlite 支持，我们需要填写sqlite文件所在地址即可。  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tsbatis-tool-sqlite-url.png)

### 选择表
我们选择那些表需要导出成为tsbatis的Entity, 然后点击Export按钮即可。
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tsbatis-tool-sqlite-select.png)


## mysql 支持
### 链接地址
对于mysql 支持，我们需要填写mysql数据的链接信息。  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tsbatis-tool-mysql-url.png)

### 选择表
这里选择表和上面sqlite 操作是是一样的。

## 导出文件
最后我们看一下导出文件  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tsbatis-tool-result.png)

# 最后
今天是感恩节，谢谢大家长时间的支持和鞭策，希望在接下来的日子能更努力工作回报大家。