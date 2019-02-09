---
title: Centos7 安装jmeter
date: 2019-02-09 11:27:21
tags:
  - jmeter
---

# 前言

jmeter 是一个 apache 组织的免费测试工具，我们可以比较简单的做一些测试, 因为自己要做 TiDB 的接口性能测试，所以需要把 jmeter 安装到 Centos7 上。  
[jmeter 官网](http://jmeter.apache.org/index.html)

# 安装

## 安装 Java 8

为了保险我没用用 openJDK 我安装的是 Oracle JDK 8 的版本， 大家可以去 Oracle 官网下载 [rpm 安装包](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)。

```java
$ yum localinstall jdk-8u201-linux-x64.rpm
$ java -version
```

## 安装 jmeter

- 从官网下载最新版本：http://jmeter.apache.org/download_jmeter.cgi

```java
$ wget http://mirrors.hust.edu.cn/apache//jmeter/binaries/apache-jmeter-5.0.tgz
$ tar -xvf apache-jmeter-5.0.tgz
$ mv apache-jmeter-5.0.tgz jmeter
$ mv jmeter /usr/local
```

- 添加环境变量
  添加 `export PATH=/usr/local/jmeter/bin/:$PATH` 到 /etc/profile 末尾。

```java
$ vi /etc/profile
$ source /etc/profile
```

- 检查安装

```java
$ jmeter -v
    _    ____   _    ____ _   _ _____       _ __  __ _____ _____ _____ ____
   / \  |  _ \ / \  / ___| | | | ____|     | |  \/  | ____|_   _| ____|  _ \
  / _ \ | |_) / _ \| |   | |_| |  _|    _  | | |\/| |  _|   | | |  _| | |_) |
 / ___ \|  __/ ___ \ |___|  _  | |___  | |_| | |  | | |___  | | | |___|  _ <
/_/   \_\_| /_/   \_\____|_| |_|_____|  \___/|_|  |_|_____| |_| |_____|_| \_\ 5.0 r1840935

Copyright (c) 1999-2018 The Apache Software Foundation
```
