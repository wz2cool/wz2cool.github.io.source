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

以 root 登录 ansible_Manager 中控机  
因为使用模板都是 centOS 7, 执行以下命令：

```java
$ yum -y install epel-release git curl sshpass
$ yum -y install python-pip
```

## 在中控机上创建 tidb 用户，并生成 ssh key

以 root 用户登录中控机，执行以下命令  
创建 tidb 用户

```java
$ useradd -m -d /home/tidb tidb
```

设置 tidb 用户密码

```java
$ passwd tidb
```

配置 tidb 用户 sudo 免密码，将 tidb ALL=(ALL) NOPASSWD: ALL 添加到文件末尾即可。

```java
$ visudo
tidb ALL=(ALL) NOPASSWD: ALL
```

生成 ssh key: 执行 su 命令从 root 用户切换到 tidb 用户下。

```java
# su - tidb
```

创建 tidb 用户 ssh key， 提示 Enter passphrase 时直接回车即可。执行成功后，ssh 私钥文件为 /home/tidb/.ssh/id_rsa， ssh 公钥文件为 /home/tidb/.ssh/id_rsa.pub。

```java
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/tidb/.ssh/id_rsa):
Created directory '/home/tidb/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/tidb/.ssh/id_rsa.
Your public key has been saved in /home/tidb/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:eIBykszR1KyECA/h0d7PRKz4fhAeli7IrVphhte7/So tidb@172.16.10.49
The key's randomart image is:
+---[RSA 2048]----+
|=+o+.o.          |
|o=o+o.oo         |
| .O.=.=          |
| . B.B +         |
|o B * B S        |
| * + * +         |
|  o + .          |
| o  E+ .         |
|o   ..+o.        |
+----[SHA256]-----+
```

## 在中控机器上下载 TiDB-Ansible

以 tidb 用户登录中控机并进入 /home/tidb 目录, 下载 TiDB-Ansible

```java
$ git clone -b release-2.1 https://github.com/pingcap/tidb-ansible.git
```

<b>注</b>：请务必按文档操作，将 tidb-ansible 下载到 /home/tidb 目录下，权限为 tidb 用户，不要下载到 /root 下，否则会遇到权限问题。

## 在中控机器上安装 Ansible 及其依赖

以 tidb 用户登录中控机，请务必按以下方式通过 pip 安装 Ansible 及其相关依赖的指定版本，否则会有兼容问题。安装完成后，可通过 ansible --version 查看 Ansible 版本。目前 release-2.0、release-2.1 及 master 版本兼容 Ansible 2.4 及 Ansible 2.5 版本，Ansible 及相关依赖版本记录在 tidb-ansible/requirements.txt 文件中

```java
$ cd /home/tidb/tidb-ansible
$ sudo pip install -r ./requirements.txt
$ ansible --version
  ansible 2.6.12
```

## 在中控机上配置部署机器 ssh 互信及 sudo 规则

以 tidb 用户登录中控机，将你的部署目标机器 IP 添加到 hosts.ini 文件 [servers] 区块下。

```java
[servers]
192.168.13.151
192.168.13.152
192.168.13.153
192.168.13.154
192.168.13.155
192.168.13.156
192.168.13.157
192.168.13.158

[all:vars]
username = tidb
ntp_server = pool.ntp.org
```

执行以下命令，按提示输入部署目标机器 root 用户密码。该步骤将在部署目标机器上创建 tidb 用户，并配置 sudo 规则，配置中控机与部署目标机器之间的 ssh 互信。

```java
$ ansible-playbook -i hosts.ini create_users.yml -u root -k
```

## 在部署目标机器上安装 NTP 服务

以 tidb 用户登录中控机，执行以下命令：

```java
$ cd /home/tidb/tidb-ansible
$ ansible-playbook -i hosts.ini deploy_ntp.yml -u tidb -b
```

## 分配机器资源，编辑 inventory.ini 文件

以 tidb 用户登录中控机，inventory.ini 文件路径为 /home/tidb/tidb-ansible/inventory.ini。

```java
## TiDB Cluster Part
[tidb_servers]
192.168.13.157

[tikv_servers]
192.168.13.154
192.168.13.155
192.168.13.156

[pd_servers]
192.168.13.151
192.168.13.152
192.168.13.153

[spark_master]

[spark_slaves]

[lightning_server]

[importer_server]

## Monitoring Part
# prometheus and pushgateway servers
[monitoring_servers]
192.168.13.158

[grafana_servers]
192.168.13.158

# node_exporter and blackbox_exporter servers
[monitored_servers]
192.168.13.151
192.168.13.152
192.168.13.153
192.168.13.154
192.168.13.155
192.168.13.156
192.168.13.157
192.168.13.158

[alertmanager_servers]
192.168.13.158
```

## 部署任务

1. 确认 tidb-ansible/inventory.ini 文件中 ansible_user = tidb，本例使用 tidb 用户作为服务运行用户，配置如下：

```java
## Connection
# ssh via normal user
ansible_user = tidb
```

执行以下命令如果所有 server 返回 tidb 表示 ssh 互信配置成功。

```java
$ ansible -i inventory.ini all -m shell -a 'whoami'
```

执行以下命令如果所有 server 返回 root 表示 tidb 用户 sudo 免密码配置成功。

```java
$ ansible -i inventory.ini all -m shell -a 'whoami' -b
```

2. 执行 local_prepare.yml playbook，联网下载 TiDB binary 到中控机：

```java
$ ansible-playbook local_prepare.yml
```

3. 初始化系统环境，修改内核参数  
   <b>注:</b> 这里即使错了也没有关系，这里会检查最低配置，所以这里报错也没有关系。

```java
$ ansible-playbook bootstrap.yml
```

4. 部署 TiDB 集群软件

```java
$ ansible-playbook deploy.yml
```

5. 启动 TiDB 集群

```java
$ ansible-playbook start.yml
```

## 测试集群

测试连接 TiDB 集群，推荐在 TiDB 前配置负载均衡来对外统一提供 SQL 接口。

- 使用 MySQL 客户端连接测试，TCP 4000 端口是 TiDB 服务默认端口。

```java
$ mysql -u root -h 192.168.13.157 -P 4000
```

- 通过浏览器访问监控平台。
  地址：http://192.168.13.158:3000 默认帐号密码是：admin/admin

参考：  
https://www.pingcap.com/docs-cn/op-guide/ansible-deployment/
