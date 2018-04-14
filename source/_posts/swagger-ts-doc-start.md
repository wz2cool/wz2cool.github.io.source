---
title: 在typescript 编写swagger 文档
date: 2018-04-14 17:42:11
tags:
- typescript
- swagger
---
项目地址：https://github.com/wz2cool/swagger-ts-doc
demo代码地址：https://github.com/wz2cool/swagger-ts-doc-demo
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

export class UpdateStudentDto {
    @apiModelProperty(
        DataType.STRING,  // 类型
        false, // 是否必填
        "学生姓名" // 描述
        )
    public name: string;
    @apiModelProperty(DataType.INTEGER, false, "学生年龄")
    public age: number;
}
```
最后会生成与之对应的swagger json 描述（这里我们不使用yaml语法，使用的json 语法）
```json
"UpdateStudentDto": {
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
## Request参数
参考swagger 文档：     
https://swagger.io/docs/specification/describing-parameters/   
https://swagger.io/docs/specification/describing-request-body/   
* RequestBody 类对应文档 requestBody
* PathVariable 类对应文档 path parameters （in: path)
* RequestParam 类对弈文档 query parameters （in: query)

## Reponse
参考swagger 文档：
https://swagger.io/docs/specification/describing-responses/  
我们看一下定义多个返回相应
```javascript
[
    new Response(HttpStatusCode.OK, DataType.STRING, "ok"),
    new Response(HttpStatusCode.INTERNAL_SERVER_ERROR, DataType.STRING, "内部错误"),
    new Response(HttpStatusCode.NOT_FOUND, DataType.STRING, "学生未找到"),
],
```

## registerRequestMapping 方法
这里就是我们要去生成swagger中paths节点调用的方法，这里我们举一个修改学生的一个例子。
```javascript
 registerRequestMapping(
    StudentApi, // tags 类似于把一些路由放到一个组里面
    "/students/{id}", // 路由
    RequestMethod.PUT,
    [
        new PathVariable("id", DataType.STRING, "学生ID"),
        new RequestBody("student", DataType.OBJECT, UpdateStudentDto, "学生"),
    ],
    [
        new Response(HttpStatusCode.OK, DataType.STRING, "ok"),
        new Response(HttpStatusCode.INTERNAL_SERVER_ERROR, DataType.STRING, "内部错误"),
        new Response(HttpStatusCode.NOT_FOUND, DataType.STRING, "学生未找到"),
    ],
    "修改学生"); // 对这个路由的描述
route.put("/:id", (req, res, next) => {
    const input = req.body;
    const id = req.params.id;
    if (!id) {
        res.status(HttpStatusCode.INTERNAL_SERVER_ERROR);
        res.json("学生ID不能为空");
        return;
    }

    if (lodash.findIndex(this.students, (x) => x.uuid === id) < 0) {
        res.status(HttpStatusCode.NOT_FOUND);
        res.json(`未能找到学生`);
        return;
    }

    const student = new Student();
    student.uuid = id;
    student.name = input.name;
    student.age = input.age;
    this.modifyStudent(student);
    res.json("ok");
});
```
我们最后看一下效果
 ![](https://raw.githubusercontent.com/wz2cool/markdownPhotos/master/res/swagger-result.png)


# 结束
利用typescript 现在我们可以轻松用强类型来写swagger文档了，swagger-jsdoc 虽然很好但是写起来太痛苦了，希望大家可以关注swagger-ts-doc 并提出宝贵的意见。