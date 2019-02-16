---
title: SonarQube入门（一）：Docker版本安装
date: 2019-02-16 18:08:44
tags:
  - sonarqube
---

# 前言

代码质量是我们一直关心的东西，首当其冲的我们需要 CodeReview，但是代码多了我们也会耗时耗力，所以我们需要借助工具来帮我们，这里就要推荐 SonarQube。
当然这篇教程还有个福利就是我们可以免费使用付费功能：branch。

# 安装

## 安装 Docker

- docker 安装

```java
$ yum install docker -y
```

- docker-compose 安装

```java
$ yum install docker-compose -y
```

- 启动 docker

```java
$ service docker start
```

## 创建数据库

这个数据库是为了下面安装 SonarQube 准备的，这里我使用的是 mysql

```java
$ CREATE DATABASE sonarqube
```

## 安装 SonarQube

如果不想看细节，直接看 [简便安装带 branch 插件 sonarqube](#简便安装带-branch-插件-sonarqube) 即可。  
这里重点来了，从 6.6 版本开始 sonarqube 把 branch 功能改成收费的了，导致的问题就是我们只能静态分析 master 分支了。  
![sonar_branch_need_pay](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/sonar_branch_need_pay.png)

那么我们如何解决：

1. 使用 7.1 版本镜像
2. 编译社区版本的 branch 插件的 jar 包

### 编译 branch 插件

因为 branch 插件从 sonarqube 的应用市场安装是需要付费的，但是有人弄出来一个社区插件：https://github.com/msanez/sonar-branch-community  
我们下载下来编译成为 jar 即可，当然我自己也编译好了一份，大家可以方便下载：[sonar-branch-plugin-2.0.0.jar](https://raw.githubusercontent.com/wz2cool/java-resource/master/jar/sonar-branch-community/2.0/sonar-branch-plugin-2.0.0.jar)

### 编写 docker 编排文件

这里我们要注意我们必须是用的是 7.1 版本，数据库 jdbc 地址用户名密码，根据自己的情况调整。保存文件为 docker-compose.yml

```java
version: '2'

services:
  sonarqube:
    image: sonarqube:7.1
    restart: always
    ports:
      - "9000:9000"
      - "9092:9092"
    environment:
      SONARQUBE_JDBC_USERNAME: ${username}
      SONARQUBE_JDBC_PASSWORD: ${password}
      SONARQUBE_JDBC_URL: jdbc:mysql://192.xxx.xxx.xxx/sonarqube
```

我们保存好 docker-compose.yml 文件后就可以启动了

```java
$ docker-compose up -d
```

### 使用编译好的 branch 插件

找到 sonarqube 容器 ID

```java
$ docker ps
```

进入容器中

```java
$ docker exec -it [容器ID] /bin/bash
```

到插件文件夹下

```java
$ cd extensions/plugins
```

下载已经编译好的插件

```java
$ wget https://raw.githubusercontent.com/wz2cool/java-resource/master/jar/sonar-branch-community/2.0/sonar-branch-plugin-2.0.0.jar
```

重启 sonarqube， 大功告成  
![sonar_restart](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/sonar_restart.png)

## 简便安装带 branch 插件 sonarqube

其实已经编译了一个镜像`wz2cool/sonarqube-branch:7.1`，直接用即可，是不是很方便。  
修改 docker-compose.yml 文件

```java
version: '2'

services:
  sonarqube:
    image: wz2cool/sonarqube-branch:7.1
    restart: always
    ports:
      - "9000:9000"
      - "9092:9092"
    environment:
      SONARQUBE_JDBC_USERNAME: ${username}
      SONARQUBE_JDBC_PASSWORD: ${password}
      SONARQUBE_JDBC_URL: jdbc:mysql://192.xxx.xxx.xxx/sonarqube
```

启动

```java
$ docker-compose up -d
```
