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
当然下载的数据都是 xml 格式，方便大家使用，已经转化成 MySQL 能使用的 sql 文件了。

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
	PRIMARY KEY (`Id`),
	INDEX `CreationDate` (`CreationDate`)
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

## 单表查询

### 单表 Count

```java
$ SELECT COUNT(0) FROM [表名]
```

|                             | MySQL      | TiDB      |
| --------------------------- | ---------- | --------- |
| math_stackexchange.users    | 0.234 sec  | 0.359 sec |
| math_stackexchange.comments | 1.031 sec  | 0.625 sec |
| stackoverflow.users         | 3.719 sec  | 0.953 sec |
| stackoverflow.comments      | 15.359 sec | 4.922 sec |

![tidb_single_select_count](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tidb_single_select_count.png)

### 带索引

```java
$ select count(0) from [表名] where CreationDate > '2017-01-01'
```

|                             | MySQL     | TiDB      |
| --------------------------- | --------- | --------- |
| math_stackexchange.users    | 0.063 sec | 0.219 sec |
| math_stackexchange.comments | 0.422 sec | 0.813 sec |
| stackoverflow.users         | 0.922 sec | 1.312 sec |
| stackoverflow.comments      | 2.891 sec | 1.437 sec |

![tidb_single_select_count2](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tidb_single_select_count2.png)

### 无索引

```java
$ select count(0) from math_stackexchange.users where views > 10
$ select count(0) from math_stackexchange.comments where score > 3
$ select count(0) from stackoverflow.users where views > 10
$ select count(0) from stackoverflow.comments where score > 3
```

|                             | MySQL     | TiDB       |
| --------------------------- | --------- | ---------- |
| math_stackexchange.users    | 0.203 sec | 0.484 sec  |
| math_stackexchange.comments | 2.219 sec | 1.297 sec  |
| stackoverflow.users         | 4.312 sec | 2.297 sec  |
| stackoverflow.comments      | 61.42 sec | 11.313 sec |

![tidb_single_select_count3](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tidb_single_select_count3.png)

## 连表查询

```java
$ select count(0) from math_stackexchange.comments as a join math_stackexchange.users as b on a.UserId = b.Id where a.CreationDate > '2017-01-01' and  b.views > 10
$ select count(0) from stackoverflow.comments as a join stackoverflow.users as b on a.UserId = b.Id where a.CreationDate > '2017-01-01' and  b.views > 10
```

|                    | MySQL      | TiDB       |
| ------------------ | ---------- | ---------- |
| math_stackexchange | 5.297 sec  | 4.468 sec  |
| stackoverflow      | 180.55 sec | 27.515 sec |

![tidb_single_select_count4](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/tidb_single_select_count4.png)

# 小结

1. 由于硬件所限，无法发挥 TiDB 比较好的性能。
2. 在表数据量比较小的时候 MySQL 的速度其实是好于 TiDB 的。
   - 因为 TiDB 组件之间需要网络传输 （这也是 TiDB 生产环境需要万兆网卡的原因）
   - 对于小表数据 MySQL 会保存在缓存中
3. TiDB 在大数据量上基本是是好于 MySQL （官方推荐表数据在 5000w 以上）
