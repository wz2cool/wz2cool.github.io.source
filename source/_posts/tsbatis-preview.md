---
title: tsbatis-preview
date: 2017-10-24 15:14:20
tags:
- tsbatis
- typescript
---
# 前言
在发布了 [mybatis-dynamic-query](https://github.com/wz2cool/mybatis-dynamic-query)之后感觉基本功能稳定了，而且现在在工作项目开发效率大大提高，而且非常易于维护。 最近准备带几个小朋友以前用typescript 打通前后端，当写node写数据库的时候，我就发现非常麻烦，当时就想，为啥js 没有类似于mybatis 框架，然后。。。 就写了tsBatis (typescript + mybatis)

# TsBatis
tsbatis：诞生的目的是希望把mybatis一些思想借鉴过来（当然现在肯定没mybatis 强大）。  
typescript：我是蛮反对写普通js 的，我觉得会比较难维护，而且确实带来了惊喜，使用了decorator 类似于java中的aonnotion 才使得这一切成为可能

## Entity
这里和java(mybatis) 有点不一样，这里的Entity 不是一个注解，而是一个类，主要做一下标示，主要作用于视图

## TableEntity
这里和java(mybatis) 不一样，java 里面使用的是@Table 注解， 这里是一个虚类继承Entity，必须实现getTableName()方法， 主要作用于表

## @column
这是核心注解，里面的参数和mybatis里面@Column注解类似。  
两种构造：  
1. 表列（只要指定列名和是否是一个key，是否可以插入）
```java
import { column, TableEntity } from "../../../src";
export class Customer extends TableEntity {
    @column("id", true, false)
    public id: number;
    @column("compnay")
    public company: string;
    @column("last_name")
    public lastName: string;
    @column("first_ame")
    public firstName: string;
    @column("email_address")
    public emailAddress: string;
    @column("job_title")
    public jobTitle: string;

    public getTableName(): string {
        return "customers";
    }
}
```
2. 视图列 （指定列名和表名）
```java
import { column, Entity } from "../../../../src";
export class NorthwindProductView extends Entity {
    @column("Id", "Product")
    public productId: number;
    @column("ProductName", "Product")
    public productName: string;
    @column("UnitPrice", "Product")
    public unitPrice: number;
    @column("CategoryName", "Category")
    public categoryName: string;
}
```

## RowBounds 类
对的，你们没看错，这个就是类似于mybatis 中的 rowBounds, 多的我就不解释了

## PageRowBounds 类
对的这个同样来自于我们熟悉的abel533 大神写的PageHelper
项目地址： https://github.com/pagehelper/Mybatis-PageHelper

## Page<> 
我们可以直接使用PageRowRounds，查询返回这个Page，具体可以看实战分页
```java
export class Page<T extends Entity> {
    private pageNum: number;
    private pageSize: number;
    private total: number;
    private pages: number;
    private entities: T[];
    ...
}
```

## ISqlConnection
确实没有mybatis那么强大驱动，而且js数据库驱动都是来自其他库（千差万别），所以我们可能需要自己实现一下
```java
import { DatabaseType, Entity, SqlTemplate } from "../model";
export interface ISqlConnection {
    getDataBaseType(): DatabaseType;
    run(sql: string, params: any[], callback: (err: any, result?: any) => void);
    runTransaction(sqlTemplates: SqlTemplate[], callback: (err: any, result: any[]) => void);
    select(sql: string, params: any[], callback: (err: any, result: any[]) => void);
    selectCount(sql: string, params: any[], callback: (err: any, result: number) => void);
    selectEntities<T extends Entity>(
        entityClass: { new(): T },
        sql: string,
        params: any[],
        callback: (err: any, result: T[]) => void);
}
```

## SqliteConnection
实现ISqlConnection 接口，是我自己实现的一套sqlite版本主要用于测试。
数据库驱动：https://github.com/mapbox/node-sqlite3
```java
const dbPath = path.join(__dirname, "../../northwind.db");
 // 传入到SqliteConnection 构造方法
this.sqlitedb = new sqlite3.Database(dbPath);
```

## MysqlConnection
实现ISqlConnection 接口，mysql 用途广泛，当然要实现一下啦。
数据库驱动：https://github.com/mysqljs/mysql
```java
// 传入到MysqlConnection 构造方法
const pool = mysql.createPool({
    host: "sql12.freemysqlhosting.net",
    port: 3306,
    database: "sql12200910",
    user: "sql12200910",
    password: "XXXXXXXXXXXXXXX",
});
```

## BaseMapper
提供公共方法，直接用纯sql结果。

## BaseMybatisMapper
继承BaseMapper，提供mybatis风格查询，比如${}和#{}站位

## BaseTableMapper
继承BaseMybatisMapper，提供基本的CRUD功能，基本上就是通用mapper

# BaseTableMapper实战
注：测试都是基于Sqlite
1. 创建表
```sql
CREATE TABLE users (
    id          INTEGER PRIMARY KEY autoincrement,
    username    VARCHAR(64) NOT NULL,
    password    VARCHAR(64)
);
```
2. 创建表实体
```java
export class User extends TableEntity {
    @column("id", true, false)
    public id: number;
    @column("username")
    public username: string;
    @column("password")
    public password: string;

    public getTableName(): string {
        return "users";
    }
}
```
3. 创建UserMapper
```java
export class UserMapper extends BaseTableMapper<User> {
    constructor(sqlConnection: ISqlConnection) {
        super(sqlConnection);
    }

    // 这里需要返回这个User， 因为你懂的，typescript 泛型是编译的，
    // 那个泛型T是拿不到什么东西的
    public getEntityClass(): new () => User {
        return User;
    }
}
```
4. 组合SqliteConnection 到 UserMapper
```java
const dbPath = path.join(__dirname, "../sqlite.db");
const db = new sqlite3.Database(dbPath);
const connection = new SqliteConnection(db);
const userMapper = new UserMapper(connection);
```

5. 插入数据
```java
describe("#insert", () => {
        it("should return seq after inserting a new row", (done) => {
            const newUser = new User();
            newUser.username = "frankTest";
            newUser.password = "pwd";
            userMapper.insert(newUser)
                .then((id) => {
                    const searchUser = new User();
                    searchUser.id = id;
                    return userMapper.selectByExample(searchUser);
                })
                .then((users) => {
                    if (users.length === 0) {
                        done("cannot find user");
                        return;
                    }

                    const user = users[0];
                    if (newUser.username === user.username
                        && newUser.password === user.password) {
                        done();
                    } else {
                        done("user is invalid");
                    }
                })
                .catch((err) => {
                    done(err);
                });
        });
});
```
6. 输出, Sqlite 直接拿不到自增id，需要在查询一下 sqlite_sequence
```bash
sql:  INSERT INTO users (username,password) VALUES (?, ?)
params:  [ 'frankTest', 'pwd' ]
sql:  SELECT seq FROM sqlite_sequence WHERE name = ?
params:  [ 'users' ]
sql:  SELECT id AS id, username AS username, password AS password FROM users WHERE id = ?
params:  [ 3 ]
√ should return seq after inserting a new row (92ms)
```

# 依赖注入（inversify）实战
老实说inversify这个东西我还不是非常明白，如果有错误欢迎指正。
1. 构造一个可注入的sqlitedb 封装
```java
import { injectable } from "inversify";
import * as path from "path";
// 这个必须引入
import "reflect-metadata";
import * as sqlite3 from "sqlite3";

@injectable()
export class InjectableSqlitedb {
    public readonly sqlitedb: sqlite3.Database;

    constructor() {
        const dbPath = path.join(__dirname, "../../northwind.db");
        this.sqlitedb = new sqlite3.Database(dbPath);
    }
}
```
2. 构造一个可注入的SqliteConnection
```java
import { injectable } from "inversify";
import * as path from "path";
import "reflect-metadata";
import * as sqlite3 from "sqlite3";
import { SqliteConnection } from "../../../src";
import { InjectableSqlitedb } from "./injectableSqlitedb";

@injectable()
export class InjectableSqliteConnection extends SqliteConnection {
    constructor(db: InjectableSqlitedb) {
        super(db.sqlitedb);
    }
}
```
3. 建立视图实体
```java
export class NorthwindProductView extends Entity {
    @column("Id", "Product")
    public productId: number;
    @column("ProductName", "Product")
    public productName: string;
    @column("UnitPrice", "Product")
    public unitPrice: number;
    @column("CategoryName", "Category")
    public categoryName: string;
}
```
4. 构造一个可注入的mapper
```java
import { inject, injectable } from "inversify";
import "reflect-metadata";
import { BaseMybatisMapper, ISqlConnection } from "../../../src";
import { InjectableSqliteConnection } from "../connection/injectableSqliteConnection";
import { NorthwindProductView } from "../entity/view/NothwindProductView";

@injectable()
export class ProductViewMapper extends BaseMybatisMapper<NorthwindProductView> {
    constructor(connection: InjectableSqliteConnection) {
        super(connection);
    }

    public getEntityClass(): new () => NorthwindProductView {
        return NorthwindProductView;
    }
}
```

5. 创造一个专门生成sql 的js 文件 （类似于mybatis xml文件）
```java
import { CommonHelper, DynamicQuery, Entity, EntityHelper, SqlTemplate, SqlTemplateProvider } from "../../../src";
import { NorthwindProductView } from "../entity/view/NothwindProductView";

export class ProductViewTemplate {
    public static getSelectProductViewByDynamicQuery(dynamicQuery: DynamicQuery<NorthwindProductView>): SqlTemplate {
        const columnAs = SqlTemplateProvider.getColumnsExpression(NorthwindProductView);
        const filterSqlTemplate = SqlTemplateProvider.getFilterExpression(NorthwindProductView, dynamicQuery.filters);
        const sortSqlTemplate = SqlTemplateProvider.getSortExpression(NorthwindProductView, dynamicQuery.sorts);
        const params = [];
        const wherePlaceholder = CommonHelper.isNotBlank(filterSqlTemplate.sqlExpression)
            ? `WHERE ${filterSqlTemplate.sqlExpression}`
            : ``;
        const orderByPlaceholder = CommonHelper.isNotBlank(sortSqlTemplate.sqlExpression)
            ? `ORDER BY ${sortSqlTemplate.sqlExpression}`
            : ``;

        const query = `SELECT ${columnAs} FROM Product LEFT JOIN Category ` +
            `ON Product.CategoryId = Category.Id ${wherePlaceholder} ${orderByPlaceholder}`;

        const sqlTemplate = new SqlTemplate();
        sqlTemplate.sqlExpression = query;
        sqlTemplate.params = sqlTemplate.params.concat(filterSqlTemplate.params).concat(sortSqlTemplate.params);
        return sqlTemplate;
    }
}
```

6. 绑定
```java
import { Container } from "inversify";
import "reflect-metadata";
import { InjectableSqliteConnection } from "./connection/injectableSqliteConnection";
import { InjectableSqlitedb } from "./connection/injectableSqlitedb";
import { NorthwindProductView } from "./entity/view/NothwindProductView";
import { ProductViewMapper } from "./mapper/productViewMapper";
import { ProductViewTemplate } from "./template/productViewTemplate";

const myContainer = new Container();
myContainer.bind<InjectableSqliteConnection>(InjectableSqliteConnection).toSelf();
myContainer.bind<ProductViewMapper>(ProductViewMapper).toSelf();
myContainer.bind<InjectableSqlitedb>(InjectableSqlitedb).toSelf();
```

7. 测试注入
```java
describe("inject Test", () => {
    it("should get inject value", () => {
            const productViewMapper = myContainer.get<ProductViewMapper>(ProductViewMapper);
            expect(false).to.be.eq(CommonHelper.isNullOrUndefined(productViewMapper));
        });
});
```

# 分页实战
基于上面的注入实战，
1. 添加动态查询模板
```java
export class ProductViewTemplate {
    // 这个就是以前的动态查询，请查看 mybatis-dynamic-query
    public static getSelectProductViewByDynamicQuery(dynamicQuery: DynamicQuery<NorthwindProductView>): SqlTemplate {
        const columnAs = SqlTemplateProvider.getColumnsExpression(NorthwindProductView);
        const filterSqlTemplate = SqlTemplateProvider.getFilterExpression(NorthwindProductView, dynamicQuery.filters);
        const sortSqlTemplate = SqlTemplateProvider.getSortExpression(NorthwindProductView, dynamicQuery.sorts);
        const params = [];
        const wherePlaceholder = CommonHelper.isNotBlank(filterSqlTemplate.sqlExpression)
            ? `WHERE ${filterSqlTemplate.sqlExpression}`
            : ``;
        const orderByPlaceholder = CommonHelper.isNotBlank(sortSqlTemplate.sqlExpression)
            ? `ORDER BY ${sortSqlTemplate.sqlExpression}`
            : ``;

        const query = `SELECT ${columnAs} FROM Product LEFT JOIN Category ` +
            `ON Product.CategoryId = Category.Id ${wherePlaceholder} ${orderByPlaceholder}`;

        const sqlTemplate = new SqlTemplate();
        sqlTemplate.sqlExpression = query;
        sqlTemplate.params = sqlTemplate.params.concat(filterSqlTemplate.params).concat(sortSqlTemplate.params);
        return sqlTemplate;
    }

    public static getSelectPriceGreaterThan20(): string {
        const columnAs = SqlTemplateProvider.getColumnsExpression(NorthwindProductView);
        const query = `SELECT ${columnAs} FROM Product LEFT JOIN Category ` +
            `ON Product.CategoryId = Category.Id WHERE Product.UnitPrice > #{price}`;
        return query;
    }
}
```
2. 分页实战
```java
it("paging", (done) => {
            const priceFilter =
                new FilterDescriptor<NorthwindProductView>((u) => u.unitPrice, FilterOperator.LESS_THAN, 20);
            const nameSort =
                new SortDescriptor<NorthwindProductView>((u) => u.productName);
            const dynamicQuery = DynamicQuery.createIntance<NorthwindProductView>()
                .addFilters(priceFilter).addSorts(nameSort);

            const sqlTemplate = ProductViewTemplate.getSelectProductViewByDynamicQuery(dynamicQuery);
            const pageRowBounds = new PageRowBounds(1, 20);
            productViewMapper
                .selectEntitiesPageRowBounds(sqlTemplate.sqlExpression, sqlTemplate.params, pageRowBounds)
                .then((page) => {
                    const pageIndexEq = page.getPageNum() === pageRowBounds.getPageNum();
                    const pageSizeEq = page.getPageSize() === pageRowBounds.getPageSize();
                    const entitiesCountEq = page.getEntities.length <= pageRowBounds.getPageSize();
                    const pagesEq = page.getPages() === Math.ceil(page.getTotal() / page.getPageSize());
                    if (pageIndexEq && pageSizeEq && entitiesCountEq && pagesEq) {
                        done();
                    } else {
                        done("something is invalid");
                    }
                })
                .catch((err) => {
                    done(err);
                });
        });
```
3. 输出分页
```bash
sql:  SELECT Product.Id AS product_id, Product.ProductName AS product_name, Product.UnitPrice AS unit_price, Category.CategoryName AS category_name FROM Product LEFT JOIN Category ON Product.CategoryId = Categor
y.Id WHERE Product.UnitPrice < ? ORDER BY Product.ProductName ASC limit 0, 20
params:  [ 20 ]
selec Count sql:  SELECT COUNT(0) FROM (SELECT Product.Id AS product_id, Product.ProductName AS product_name, Product.UnitPrice AS unit_price, Category.CategoryName AS category_name FROM Product LEFT JOIN Catego
ry ON Product.CategoryId = Category.Id WHERE Product.UnitPrice < ? ORDER BY Product.ProductName ASC) AS t
params:  [ 20 ]
√ paging
```

## 结束 ##
为了这次1024，我真是有点赶想让大家早点看看这个项目，当然还有很多测试要做。
祝大家节日快乐。 


## 关注我　##
最后大家可以关注我和 tsbatis项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/tsbatis" data-size="large" data-show-count="true" aria-label="Star wz2cool/tsbatis on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/tsbatis" data-size="large" data-show-count="true" aria-label="Fork wz2cooltsbatis on GitHub">Fork</a>