---
title: jmeter 分布式压力测试
date: 2019-02-09 14:26:49
tags:
  - jmeter
---

# 前言

当单机测试有局限的时候，我们需要用多台机器来做测试。 [jmeter 分布式](http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html)  
当然我们需要用到之前已经安装好的系统 [Centos7 安装 jmeter](https://wz2cool.github.io/2019/02/09/jmeter1/),
这里再次提醒大家使用 jmeter 版本是 3.3，重要的事情说三遍。

# 准备

## 分布测试基本要求

- 所有系统上的防火墙需要关闭，并且对应的端口必须是开放的。
- 所有机器必须在同一子网段上。比如 一台机器用 192.x.x.x 那么另外的机器必须也是 192.x.x.x。
- 所有机器必须能够互相访问。
- 所用的 JMeter 和 Java 的版本需要是一样的。

## 机器配置

需要 2 台 slave 机器（Centos7）, 一台中控机器 （Window 10）

| 机器    | IP            |
| ------- | ------------- |
| master  | 192.168.10.53 |
| slave_1 | 192.168.8.14  |
| slave_2 | 192.168.8.123 |

# 步骤

## slave 配置

- 关闭防火墙

```java
$ service firewalld stop
$ service iptables stop
```

- 运行 jmeter-server

这里只有一个地方需要注意， 我们需要指定一下 hostname 否则会报错。

```java
$ cd /usr/local/jmeter/bin
$ ./jmeter-server -Djava.rmi.server.hostname=192.168.8.14
```

## master 配置

- 添加远程机器 IP
  修改 bin 文件夹下的 jmeter.properties 文件, `remote_hosts=192.168.8.14,192.168.8.123`

```java
# Remote Hosts - comma delimited
remote_hosts=192.168.8.14,192.168.8.123
#remote_hosts=localhost:1099,localhost:2010
```

- 运行 jmeter UI 界面
  直接点击 jmeter.bat 文件， 因为在 window 下， linux 用户可以运行 jmeter.sh

# 测试

## 运行

- 单台运行  
  选择 Remote start 菜单  
  ![jmeter_remote_start](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/jmeter_remote_start.png)

- 全部一起运行
  选择 Remote start all 菜单

## 结果

- 在 slave 可以看到任务被执行

```java
Created remote object: UnicastServerRef [liveRef: [endpoint:[192.168.8.123:59906](local),objID:[-3b632aff:168d21e3af0:-7fff, 7021558400555619249]]]
Starting the test on host 192.168.8.123 @ Sat Feb 09 19:57:03 CST 2019 (1549713423638)
Finished the test on host 192.168.8.123 @ Sat Feb 09 19:57:09 CST 2019 (1549713429469)
Starting the test on host 192.168.8.123 @ Sat Feb 09 19:59:20 CST 2019 (1549713560420)
Finished the test on host 192.168.8.123 @ Sat Feb 09 19:59:26 CST 2019 (1549713566724)
```

- 在 master 看到汇聚后的结果
  给了 500 线程，但是我们可以看到统计结果是两台的合并 1000。  
  ![jmeter_master_1](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/jmeter_master_1.png)  
  ![jmeter_master_2](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/jmeter_master_2.png)

参考：
https://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html
