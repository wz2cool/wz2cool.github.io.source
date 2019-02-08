---
title: TiDB入门（三）：简单测试
date: 2019-02-07 19:54:46
tags:
  - TiDB
---

# 前言

上一次我们搭建了[TiDB 环境](https://wz2cool.github.io/2019/02/07/tidb2/)，那么现在我想用跑跑数据，看看 TiDB 和 传统 MySQL 相比到底优越在哪里。

# 数据库配置

## TiDB 集群

由虚拟机搭建集群，共享 CPU 和内存

- CPU：Xeon(R) X5650 \* 2 （两个物理 CPU，逻辑 24 核）
- 内存：64G DDR3 1333MHz
- 主硬盘： 5.45 TB raid5 （DELL 服务器磁盘 7200 转）
- 固态：120GB \* 3 (非服务器 SSD)

## MySQL 单例

配置暂时缺失...

# 数据准备

## 下载测试数据

所有数据均来自于 [Stack Exchange Data Dump](https://archive.org/details/stackexchange)。  
当然下载的数据都是 xml 格式，使用不方便，我已经转化成 MySQL 能使用的 sql 文件了。

下载：

- 小数据：[math_stackchange](https://pan.baidu.com/s/1kTom0Eaq4iH_fM9m_22sZA)
- 大数据：[stackoverflow](https://pan.baidu.com/s/1ClLttJBnvrDaen1qcjteiQ)

## 插入数据

把对应的数据库建立好，导入数据即可。由于数据大概有 15G 左右，导入时间过长，可以看部电影耐性等待。  
<b>注：</b>导入数据到 TiDB 集群明显比传统 MySQL 数据库慢，这里我暂时还没有找到原因。

```java
$ mysql -uroot -h192.168.13.157 -P 4000 math_stackexchange < math_stackexchange.sql
$ mysql -uroot -h192.168.13.157 -P 4000 stackoverflow < stackoverflow.sql
```

![stackexchange_data](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/stackexchange_data.png)

# 测试

## 测试表信息

- comments 表

```java
CREATE TABLE `comments` (
	`Id` BIGINT(20) NOT NULL AUTO_INCREMENT,
	`PostId` BIGINT(20) NULL DEFAULT NULL,
	`Score` INT(11) NULL DEFAULT NULL,
	`Text` TEXT NULL,
	`CreationDate` DATETIME NULL DEFAULT NULL,
	`UserId` BIGINT(20) NULL DEFAULT NULL,
	`UserDisplayName` VARCHAR(100) NULL,
	PRIMARY KEY (`Id`),
	INDEX `CreationDate` (`CreationDate`)
)
COLLATE='utf8mb4_bin'
ENGINE=InnoDB
;
```

- users 表

```java
CREATE TABLE `users` (
	`Id` BIGINT(20) NOT NULL AUTO_INCREMENT,
	`Reputation` INT(11) NULL DEFAULT NULL,
	`CreationDate` DATETIME NULL DEFAULT NULL,
	`DisplayName` VARCHAR(100) NULL,
	`LastAccessDate` DATETIME NULL DEFAULT NULL,
	`WebsiteUrl` VARCHAR(255) NULL,
	`Location` VARCHAR(100) NULL,
	`AboutMe` TEXT NULL,
	`Views` INT(11) NULL DEFAULT NULL,
	`UpVotes` INT(11) NULL DEFAULT NULL,
	`DownVotes` INT(11) NULL DEFAULT NULL,
	`AccountId` BIGINT(20) NULL DEFAULT NULL,
	`ProfileImageUrl` VARCHAR(255) NULL,
	PRIMARY KEY (`Id`)
)
COLLATE='utf8mb4_bin'
ENGINE=InnoDB
;
```

| 数据库             | 表名     | 数量     | 量级         |
| ------------------ | -------- | -------- | ------------ |
| math_stackexchange | users    | 497023   | 十万级别     |
| math_stackexchange | comments | 4463392  | 百万级别     |
| stackoverflow      | users    | 9737247  | 接近千万级别 |
| stackoverflow      | comments | 60915000 | 5 千万级别   |

## 单表 Count

曾经我们常常会慢在 Count 所以统计表数量成为我优先测试的一项。

```java
# SELECT COUNT(0) FROM [表名]
```

|                             | MySQL      | TiDB      |
| --------------------------- | ---------- | --------- |
| math_stackexchange.users    | 0.234 sec  | 0.359 sec |
| math_stackexchange.comments | 1.031 sec  | 0.625 sec |
| stackoverflow. users        | 3.719 sec  | 0.953 sec |
| stackoverflow. comments     | 15.359 sec | 4.922 sec |

![tidb_single_select_count](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tidb_single_select_count.png)
