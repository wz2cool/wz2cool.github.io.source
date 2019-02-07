---
title: (二) 虚拟机搭建 TiDB-Ansible 部署方案
date: 2019-02-07 15:48:05
tags:
  - TiDB
---

# 前言

如果对 TiDB 一无所知同学可以简单看一下 [TiDB 简介](https://wz2cool.github.io/2019/02/07/tidb1/)。  
如果对 TiDB 只是想简单看一下的效果，对性能无要求的，可以参考官网 [Docker Compose 快速构建集群](https://www.pingcap.com/docs-cn/op-guide/docker-compose/), 三个命令即可。  
这一次我们要用到其推荐的方式进行搭建，对机器硬件是有所要求的。当然我自己也是穷啊，所以也只是用了一台二手服务器进行搭建, 和官网推荐配置差距很多。

# 环境准备

## 硬件

- CPU：Xeon(R) X5650 \* 2 （两个物理 CPU，逻辑 24 核）
- 内存：64G DDR3 1333MHz
- 主硬盘： 5.45 TB raid5 （DELL 服务器磁盘 7200 转）
- 固态：120GB \* 3 (非服务器 SSD)

![dell-server](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/dell-server.png)

## 虚拟机模板配置

虚拟机有器好处，就是我们可以通过模板进行快速复制部署，这里我设置了两个模板：

- tidb_template 这里设置磁盘大小为 16G
- tikv_template 这里设置磁盘大小为 50G
  这里曾经有个坑就是，TiKV 存储满了，我不知道如何扩展（待处理），所以对 TiKV 单独建立了一个 50G 大容量模板  
  下载 ova： [tidb_template](https://pan.baidu.com/s/1_4IQqaSz1XrgqVUZCyP1pg) , [tikv_temlate](https://pan.baidu.com/s/1E8NuJW2zSTEziNp44UM-0A)

| 组件            | 模板          | 数量 | CPU | 内存 | 硬盘    |
| --------------- | ------------- | ---- | --- | ---- | ------- |
| Manager(中控机) | tidb_template | 1    | 4   | 4G   | SAS 16G |
| TiDB            | tidb_template | 1    | 8   | 8G   | SAS 16G |
| PD              | tidb_template | 3    | 8   | 4G   | SSD 16G |
| TiKV            | tikv_template | 3    | 8   | 12G  | SSD 50G |

## 网络配置

这里网络配置只是我自己的配置, 这里写出来是方便后面的配置文件。

| Server          | IP             | 作用           | hostname        |
| --------------- | -------------- | -------------- | --------------- |
| ansible_Manager | 192.168.13.158 | 中央配置，监控 | ansible_Manager |
| ansible_TiDB_1  | 192.168.13.157 | TiDB           | ansible_TiDB_1  |
| ansible_PD_1    | 192.168.13.151 | PD             | ansible_PD_1    |
| ansible_PD_2    | 192.168.13.152 | PD             | ansible_PD_2    |
| ansible_PD_3    | 192.168.13.153 | PD             | ansible_PD_3    |
| ansible_TiKV_1  | 192.168.13.154 | TiKV           | ansible_TiKV_1  |
| ansible_TiKV_2  | 192.168.13.155 | TiKV           | ansible_TiKV_2  |
| ansible_TiKV_3  | 192.168.13.156 | TiKV           | ansible_TiKV_3  |

![tidb_constructor](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tidb_constructor.png)

## 其他重要配置

### 修改 hostname

这里我们需要修改每个 Server 的 hostname, 因为都是模板部署所以 hostname 是一样的，不改的话同名以后部署会报错的。

```java
$ hostnamectl set-hostname your-new-hostname
```

### 关闭 swap

这里似乎是 TiDB 为了更好性能，否则会报错, 我们在每个 server 关闭即可。

```java
// 查看交互分区
$ cat /etc/fstab
// 交互分区可能不一样
$ swapoff /dev/mapper/centos-swap
```

# 安装

## 在中控机上安装系统依赖包

因为使用模板都是 centOS 7, 执行一下命令：

参考：  
https://www.pingcap.com/docs-cn/op-guide/ansible-deployment/
