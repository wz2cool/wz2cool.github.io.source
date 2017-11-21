---
title: Otter探索（一）：canal基本概念
date: 2017-11-21 10:29:21
tags: otter
---
# 前言
终于把otter 同步跑起来了，感觉还是应该需要做一下小结，第一是防止自己忘记，第二是方便他人，因为才疏学浅，希望不对的地方大家予以指正。这篇主要是介绍一下Canal，因为Otter设计基于Canal开发的。

# 工作原理
这里我们主要引用官网介绍  
## mysql主备复制实现
![](https://camo.githubusercontent.com/eec1605862fe9e9989b97dd24f28a4bc5d7debec/687474703a2f2f646c2e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038302f333038362f34363863316131342d653761642d333239302d396433642d3434616335303161373232372e6a7067)  
从上层来看，复制分成三步：  
1. master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events，可以通过show binlog events进行查看）；  
2. slave将master的binary log events拷贝到它的中继日志(relay log)；  
3. slave重做中继日志中的事件，将改变反映它自己的数据。
## canal的工作原理
![](https://camo.githubusercontent.com/46c626b4cde399db43b2634a7911a04aecf273a0/687474703a2f2f646c2e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038302f333130372f63383762363762612d333934632d333038362d393537372d3964623035626530346339352e6a7067)  
原理相对比较简单：  
1. canal模拟mysql slave的交互协议，伪装自己为mysql slave，向mysql master发送dump协议
2. mysql master收到dump请求，开始推送binary log给slave(也就是canal)  
3. canal解析binary log对象(原始为byte流)

# 讲解
这里我们从官网描述就可以看出来：
1. canal 是模拟slave， 发出dump binlog 请求下载到canal所在地。
2. 注意这里canal的原理中并没有去relay binlog到某个数据库，所以canal 应该只是一个dump master binlog 工具库，如何relay到自己的数据库，需要自己写代码。