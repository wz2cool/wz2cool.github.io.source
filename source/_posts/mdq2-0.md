---
title: mdq2.0
date: 2017-08-15 14:09:23
tags:
- java
- MDQ 2.0 杂记
---
## tk.mybatis.mapper 整合 ##
tk.mybatis.mapper 是一套非常棒的mybatis通用框架，主要license 还是 MIT 所以很放心的引入进来，基本上单表很强大的功能tk.mybatis.mapper都有了。  
项目地址：https://github.com/abel533/Mapper
### 使用 javax.persistence 实体类注解###
原来注解都是我们自己定义的，而tk.mybatis.mapper 自带javax.persistence，我们必须删除掉原来的注解，因为@Column/@Id/@Table 有冲突了。

### DynamicQueryMapper ###
DynamicQueryMapper 是我们扩展了 tk.mybatis.mapper 里面的mapper，主要以前tk.mybatis.mapper 筛选是使用Example来做的，现在我们使用我们自己的Filter表述器来做。   
设置一下我们扫描mapper包
```java
@SpringBootApplication
@MapperScan(basePackages = "com.github.wz2cool.mdqtest.mapper")
@EnableSwagger2
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
继承一下通用mapper
```java
public interface ProductDao extends DynamicQueryMapper<Product> {
}
```
然后我们在使用ProductDao 的时候就会有很多自带的方法了。
 ![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/20170815143538.png)
