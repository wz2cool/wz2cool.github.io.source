---
title: Mybatis Dynamic Query 2.0 入门
date: 2017-08-15 14:55:54
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
## 简介 ##
2.0 主要是整合了tk.mybatis.mapper 到项目中去，所以和1.x比起来主要多了一个通用mapper。因为作者主要是使用springboot 这里讲一下Springboot 配法。
### 配置步骤 ###
添加依赖
```xml
<!-- 基本库 -->
<dependency>
    <groupId>com.github.wz2cool</groupId>
    <artifactId>mybatis-dynamic-query</artifactId>
    <version>2.0.0</version>
</dependency>
<!-- 主要注册通用mapper -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.1.3</version>
</dependency>
<!-- mybatis 最新版本 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.4</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
<!-- 如果有Spring boot web 自带jackson 这个可以不要,防止版本冲突 -->
<!--  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>-->
<!-- spring boot -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.2.0</version>
</dependency>
```