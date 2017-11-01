---
title: virualbox安装centos7
date: 2017-11-01 15:43:09
tags:
- centos7
---
# 前言
最近研究了一下阿里otter 项目， 在准备虚拟机的的时候真的很坑。记录一下吧。
- win10 无法运行virualbox 5.x版本以上
- 使用Host-only 主机和虚拟机互通，（win10 更新导致桥接不可用）

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