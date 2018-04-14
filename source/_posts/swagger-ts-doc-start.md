---
title: 在typescript 编写swagger 文档
date: 2018-04-14 17:42:11
tags:
- typescript
- swagger
---
项目地址：https://github.com/wz2cool/swagger-ts-doc
# 动机
Swagger API 文档框架相信大家都使用过，并且真的很方便，但是大家应该都是用框架生成的出来swagger 文档，可能很少人会去写 yaml文档吧。  
确实我在使用nodejs 发现写接口还是很方便，但是唯独在写swagger文档时候发现nodejs中的框架并不好用，曾经用过swagger-jsdoc,写了一堆注释，然后自己崩溃了。  
好歹我自己也写了快2年的java了，为什么不照着java的方式写一套呢，于是写了 swagger-ts-doc，自用下来还是挺方便的。

# Swagger 简介
大家可打开一下 http://editor.swagger.io/ 看一下官方写的一个例子。
## info 
这个应该是一些对这个文档的描述信息，这个在swagger-ts-doc 是可以配置的, 对应的配置为 SwaggerInfoProperty。
## definitions
这个是一个重要的节点，这个节点其实是定义了我们所有的类，比如有个requestBody 其实是一个类，就可以通过  $ref: "#/definitions/User" 进行关联。  
在swagger-ts-doc 中，这里的是通过apiModelProperty进行描述的
## paths
这个节点就是最重要的节点，它描述了我们所有路由post,get,put,delete.   
在swagger-ts-doc 中，这里是通过registerRequestMapping 进行描述的。

# swagger-ts-doc 中类和方法
## apiModelProperty 装饰器
这个装饰器主要是为了生成 definitions 中的model，我们看代码可看到如何描述一个typescript中的一个类。
```javascript
import { apiModelProperty, DataType } from "swagger-ts-doc";

export class AddStudentDto {
    @apiModelProperty(
        DataType.STRING,  // 类型
        true, // 是否必填
        "学生姓名" // 描述
        )
    public name: string;
    @apiModelProperty(DataType.INTEGER, true, "学生年龄")
    public age: number;
}
```
最后会生成与之对应的swagger json 描述（这里我们不使用yaml语法，使用的json 语法）
```json
"AddStudentDto": {
    "type": "object",
    "required": ["name", "age"],
	"properties": {
		"name": {
			"type": "string",
			"description": "学生姓名"
		},
		"age": {
			"type": "integer",
			"description": "学生年龄"
		}
	}
},
```