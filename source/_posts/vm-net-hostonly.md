---
title: virualbox 安装 otter 准备环境
date: 2017-11-01 15:43:09
tags:
- centos7
---
# 前言
最近研究了一下阿里otter项目（分布式数据库同步），所以就在virualbox 上开始准备，遇到了不少坑，所以记录一下啊。   
otter 项目：https://github.com/alibaba/otter
- win10 无法运行virualbox 5.x版本以上运行
- 使用Host-only 主机和虚拟机互通，（win10 更新导致桥接不可用）
- otter 必要软件准备

# win10 安装virualbox
## 安装virualbox 4.3.6
这个是第一个坑，安装5.x版本一直报错，请使用virualbox 4.3.6版本。
## 关闭 360
无法运行镜像，这个是由于360导致的，请关闭360安全卫士。
## 创建 Host-only Network
1. File -> Preferences... -> Network -> Host-only Networks  
点击添加  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/addHostOnly.png)
2. 配置Adapter 和 DHCP Server 如下
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/hostonlyAdapter.png)
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/hostonlyDHCP.png)

# 安装centos7 mini
## 配置网络
1. 第一个网卡为NAT，主要访问外网。
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/netnat.png)
2. 第二块为host-only 主要和主机互通
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/nethostonly.png)
## 安装centos7 mini
这里一步一步，就不赘述了...
## 配置上网
已进入系统很奇怪，什么网都上不去，这个就需要我们自动获取ip 地址  
1. 输入命令 `$ nmtui`  
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/nmtui.png)
2. enp0s3应该是我们那块NAT 网卡, 把状态都改成 Automatic，
（这里有个X 真是坑，是用空格选中的！！！）
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/enp3.png)
3. 其他的网卡一样自动获取
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/othereth.png)
4. 使用 `$ ip a` 查看链接状态
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/ipa.png)

# 安装 java
这里也是一个坑，一定要用oracle 的JDK，千万不要用openJDK。因为otter中node节点在openJDK 会报错 SHA找不到错误。  
请去oracle 官网下载：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html  
下载完成以后执行
`$ yum localinstall [JDK.rpm]`

# 安装 mysql
和java 一样，centos 默认提供是mariadb，为了防止不必要的意外，我们还是使用mysql5.7, 依次执行语句。  
`$ wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm`  
`$ sudo yum localinstall mysql57-community-release-el7-11.noarch.rpm
`  
`$ yum repolist enabled | grep "mysql.*-community.*"
`  
`$ sudo yum install mysql-community-server`

## 修改my.conf
`$ vim /etc/my.conf`    
1. 修改默认字符集为utf8, 这个不改的话以后配置 otter 会报错。  
添加 charater-set-server=utf8   
2. 开启binlog, otter 主要使用binlog, 添加：   
log-bin=mysql-bin   
server-id=1   
![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/binlog.png)
3. 重启service   
`$ service mysqld restart`

## 外网访问
这个大家根据自己需求建立访问权限，这里就不赘述了。

# 安装 aria2c
主要是otter node 节点需要使用这个库，官方文档是说要下源码进行编译，这里我们可以直接用安装包安装。   
`$ yum install epel-release -y`   
`$ yum install aria2 -y`
最后可以用命令确定安装成功  
`$ aria2c -v`

# 安装 zookeeper
大概就是这几条命令就好了。
`$ mkdir /tmp`   
`$ wget http://ftp.jaist.ac.jp/pub/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz`    
`$ tar -xvf zookeeper-3.4.9.tar.gz -C /tmp/`  
`$ cd /tmp/zookeeper-3.4.9/conf`
`$ mv zoo.example.cfg zoo.cfg`
`$ cd..`  
`$ ./bin/zkServer.sh start`

嗯 基本上otter 要的东西都准备好了，后面开始otter 安装。