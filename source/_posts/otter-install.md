---
title: virualbox 搭建 otter
date: 2017-11-01 18:03:43
tags:
---
# 前言
为了学习otter，上一篇我们讲到了 otter 必要软件的安装，参考：[virualbox 安装 otter 必备软件](https://wz2cool.github.io/2017/11/01/vm-net-hostonly/)，现在安装otter，相比官方文档，我们尽量简化安装步骤。     

# virualbox clone
之前我们安装的一台虚拟机，我们现在就可以clone 一台出来。这样我们把一台当做manager， 另外一台当做node。
## 网络链接
同样参照之前 [virualbox 配置网络](https://wz2cool.github.io/2017/11/01/vm-net-hostonly/#配置网络)，确保NAT，和host-only 网卡正常工作。

# manager 安装
## 环境准备
先到我们第一台虚拟机，我们定义它为manager虚拟机。前面我们已经装好了mysql了，现在就可以用到了，这里我是用工具跑的，不是用命令跑的。直接点开在浏览器打开：https://raw.githubusercontent.com/alibaba/otter/master/manager/deployer/src/main/resources/sql/otter-manager-schema.sql   
然后打开我们IDE， 复制sql 代到IDE，然后在最上面加上 `SET sql_mode = '';`
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/runsql.png)

## 启动步骤
1.  直接去https://github.com/alibaba/otter/releases 下载编译好的文件。
这里我们使用的是   
`$ wget https://github.com/alibaba/otter/releases/download/v4.2.14/manager.deployer-4.2.14.tar.gz`
2. 解压缩  
`$ mkdir /tmp/manager`  
`$ tar zxvf manager.deployer-4.2.14.tar.gz  -C /tmp/manager`  
3. 配置修改  
把 127.0.0.1 改成当前虚拟机内网host-only ip, 这里我的ip 为 192.168.56.101.
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/managerOtterconfig.png)
4. 准备启动  
`$ ./bin/startup.sh`
5. 查看日志  
`$ vim logs/manager.log`  
```code
2013-08-14 13:19:45.911 [] WARN  com.alibaba.otter.manager.deployer.JettyEmbedServer - ##Jetty Embed Server is startup!   
2013-08-14 13:19:45.911 [] WARN  com.alibaba.otter.manager.deployer.OtterManagerLauncher - ## the manager server is running now ......
```
6. 验证  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/managerUI.png)   
初始密码为: admin/admin

# node 
## 环境准备
1. 安装完成 manager，添加一个zookeeper 集群，zookeeper 虚拟机安装过了。
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/zoomanager.png)  
2. 在manager 中添加一个node，为的是产生一个唯一id，
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/nodemanage.png)    
添加完成
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/nodeid.png)    

## 启动步骤
1. 下载otter node, 可访问：https://github.com/alibaba/otter/releases ，这里我们直接用最新的。
`$ wget https://github.com/alibaba/otter/releases/download/v4.2.14/node.deployer-4.2.14.tar.gz`
2. 解压缩   
`$ mkdir /tmp/node`   
`$ tar zxvf node.deployer-4.2.14.tar.gz  -C /tmp/node`   
3. 配置修改   
a. 把nid 写入进去  
`$ cd /tmp/node`   
`$ echo 1 > conf/nid`    
b. 修改otter.properies    
基本上什么都不用改，只需填写manager地址要改otter.manager.address=192.168.56.101:1099   
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/nodeconfig.png)  
4. 启动  
`$ ./bin/startup.sh`
5. 查看日志  
`$ vim logs/node/node.log`
```code
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
2017-11-01 06:21:54.027 [main] INFO  com.alibaba.otter.node.deployer.OtterLauncher - INFO ## the otter server is running now ......
```
6. 验证  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/noderun.png) 


# 操作演示
## 建表
manager 和 node 数据库都跑这个   
```code
CREATE DATABASE `test`;
CREATE TABLE  `test`.`example` (
  `id` int(11)  NOT NULL AUTO_INCREMENT,
  `name` varchar(32) COLLATE utf8_bin DEFAULT NULL ,
   PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
## 配置
1. 添加 canal
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/addcanal.png) 
2. 添加数据源 managedb，然后点击验证数据源，这里编码是UTF-8,这个地方是个有个坑的，但是我们已经在之前的必要安装里面设置过了mysql默认编码为UTF-8。  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/managedb.png) 
3. 再添加一个node1db.  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/nodedb.png) 
4. 添加数据表   
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/managetable.png)   
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/nodetable.png) 
5. 添加channel  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/addchannel.png) 
6. 添加Pipeline  
这里需要点击一下 channel 的名字   
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/clickChannel.png)   
添加一下  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/pipline.png)  
7. 添加映射关系  
首先需要点击一下 pipeline 的名字  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/createtableMapping.png)  
到这里就是很简单的选择，源数据表选择managedb中 test数据库的exmple表，目标数据表选择node1db 中的example 表。  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tablemapping.png)  
8. 执行channel   
回到channl 管理点击启用按钮。   
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/runchanenl.png) 

# 最终验证
你在mangerdb 中任何改动都会同步到node上，  
1. 添加数据
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/result.png) 
2. 添加一列  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/addcolumn.png) 

# 最后
终于把otter 入门测试环境搭建好了，里面省去教程很多不必要步骤，而且还提醒大家在搭建时候遇到的坑，希望大家喜欢，有任何问题欢迎指教。  

参考：https://github.com/alibaba/otter/wiki/QuickStart