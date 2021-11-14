---
title: elasticsearch-dynamic-query 入门
date: 2021-11-13 22:38:35
tags:
  - elasticsearch Dynamic Query
  - java
---

项目地址：https://github.com/wz2cool/elasticsearch-dynamic-query  
demo 地址： https://github.com/wz2cool/elasticsearch-dynamic-query-demo

# 前言

作为我们先通过一个简单例子来把 elasticsearch-dynamic-query 用起来

# 使用

## 引入 pom

对于 springboot 项目，我们可以直接引入一个 starter 就好了

```xml
<dependency>
  <groupId>com.github.wz2cool</groupId>
  <artifactId>elasticsearch-dynamic-query-spring-boot-starter</artifactId>
  <version>0.1.11</version>
  <scope>test</scope>
</dependency>
```

## 配置 es 连接

因为我们实际是封装的 spring data elasticsearch 所以这里也是同样在 application.properties 配置

rest api

```bash
spring.elasticsearch.rest.uris=http://192.168.8.185:9200
```

transport (es7 后废弃建议不要使用)

```bash
spring.data.elasticsearch.cluster-name=dev-interes
spring.data.elasticsearch.cluster-nodes=192.168.8.185:9300
```

## 建立测试对象

一般

```java
@Document(indexName = "test_example", type = "testExample")
public class TestExampleES {

    @Id
    private Long id;
    @Field(type = FieldType.Text, analyzer = "hanlp_index")
    private String p1;
    private Integer p2;
    private Long p3;
    private Float p4;
    private Double p5;
    @Field(name = "p6", type = FieldType.Date)
    private Date aliasP6;
    @Field(type = FieldType.Keyword)
    private String p7;
    private BigDecimal p8;

    private Integer[] p9;

    @Transient
    private String p1Hit;
...
```
