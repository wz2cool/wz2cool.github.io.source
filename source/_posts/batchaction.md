---
title: MapperBatchAction 批量操作
date: 2021-06-27 11:26:58
tags:
  - Mybatis Dynamic Query
  - java
  - MDQ
---

# 前言

批量也是这个框架缺失的，因为没有 spring starter 封装，所以先弄一个帮助性质的方法，其实核心方法就是调用 sqlSession batch 提交，
还有需要注意的是查阅文档，批量操作应该只是作用于 insert, update, delete 三个操作。

# 使用

我们看一下怎么构建 MapperBatchAction

```java
// 注入sqlSessionFactory
@Resource
private SqlSessionFactory sqlSessionFactory;

@Test
public void testBatchInsert() {
    // 不能直接是你用 mapper , 而是使用 mapper 的class 才行， 这里我们设置每次批量是3
    MapperBatchAction<BugDao> insertBatchAction = MapperBatchAction.create(BugDao.class, this.sqlSessionFactory, 3);
    for (int i = 0; i < 10; i++) {
        Bug newBug = new Bug();
        newBug.setId(10000 + i);
        newBug.setAssignTo("frank");
        newBug.setTitle("title");
        // 缓存所有的插入操作,(PS: 当然也可以放入更新操作)
        insertBatchAction.addAction((mapper) -> mapper.insertSelective(newBug));
    }
    // 执行批量操作并返回批量结果
    final List<BatchResult> batchResults = insertBatchAction.doBatchActions();
    int effectRows = batchResults.stream().mapToInt(x -> Arrays.stream(x.getUpdateCounts()).sum()).sum();
    assertEquals(10, effectRows);
}
```

我们看一下返回值确实是 3 个为一批操作

```bash
JDBC Connection [HikariProxyConnection@806738808 wrapping conn0: url=jdbc:h2:mem:default user=SA] will not be managed by Spring
==>  Preparing: INSERT INTO bug ( id,title,assignTo ) VALUES( ?,?,? )
==> Parameters: 10000(Integer), title(String), frank(String)
==> Parameters: 10001(Integer), title(String), frank(String)
==> Parameters: 10002(Integer), title(String), frank(String)
==>  Preparing: INSERT INTO bug ( id,title,assignTo ) VALUES( ?,?,? )
==> Parameters: 10003(Integer), title(String), frank(String)
==> Parameters: 10004(Integer), title(String), frank(String)
==> Parameters: 10005(Integer), title(String), frank(String)
==>  Preparing: INSERT INTO bug ( id,title,assignTo ) VALUES( ?,?,? )
==> Parameters: 10006(Integer), title(String), frank(String)
==> Parameters: 10007(Integer), title(String), frank(String)
==> Parameters: 10008(Integer), title(String), frank(String)
==>  Preparing: INSERT INTO bug ( id,title,assignTo ) VALUES( ?,?,? )
==> Parameters: 10009(Integer), title(String), frank(String)
```

# 小结

这里和其他的 mapper 批量操作不一样 ，我们缓存操作会更具有灵活性。
