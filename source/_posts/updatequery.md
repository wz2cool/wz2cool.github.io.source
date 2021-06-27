---
title: Update Query
date: 2021-06-27 11:04:44
tags:
  - Mybatis Dynamic Query
  - java
---

项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/

# 前言

项目中其实有个很大的问题就是对于更新操作，这个问题在 tk.mapper 里面也会有，目前支持的两个更新是根据对象来的两个方法

- update: 方法是全部进行赋值，即使为 null 也赋值为 null
- updateSelective: 方法是如果 null 就忽略，有值才赋值

大家看到了没，少了一个中间状态，就是说我就想更新里面某几个值，但是 null 我们也想赋值，这个我们发现就不太好弄了，所以我们新增了 UpdateQuery 对象进行处理

# 使用

## UpdateByUpdateQueryMapper

这个就是对应的 Mapper, 里面主要提供了 updateByUpdateQuery 方法，参数为 UpdateQuery 这里不做赘述，并且已经放入 DynamicQueryMapper 里面，所以只要继承了 DynamicQueryMapper 就会默认有 里面主要提供了 updateByUpdateQuery 方法

## UpdateQuery

这个就是我们的新类，这个新的查询类是集成筛选组的，所以以前的筛选方法都是可以用的，主要添加 set 方法，可以对某个属性进行赋值。

```java
 @Test
public void testUpdateByUpdateQuery() {
    User user = new User();
    user.setId(19);
    user.setUsername("frank19");
    user.setPassword("frank");
    int result = userDao.insert(user);
    assertEquals(1, result);
    UpdateQuery<User> userUpdateQuery = UpdateQuery.createQuery(User.class)
            // 设置有值
            .set(User::getUsername, "Marry")
            // 设置null
            .set(User::getPassword, null)
            .and(User::getId, isEqual(19));
    result = userDao.updateByUpdateQuery(userUpdateQuery);
    assertEquals(1, result);
    final User user1 = userDao.selectByPrimaryKey(19);
    // 有值的设置成功更新
    assertEquals("Marry", user1.getUsername());
    // 设置为null 也成功
    assertNull(user1.getPassword());
    userDao.deleteByPrimaryKey(19);
}
```

我们同样可以看一下 output 验证

```bash
JDBC Connection [HikariProxyConnection@1220747354 wrapping conn0: url=jdbc:h2:mem:default user=SA] will not be managed by Spring
==>  Preparing: INSERT INTO users ( id,username,password ) VALUES( ?,?,? )
==> Parameters: 19(Integer), frank19(String), frank(String)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@771db12c]
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@1ba05e38] was not registered for synchronization because synchronization is not active
JDBC Connection [HikariProxyConnection@179060558 wrapping conn0: url=jdbc:h2:mem:default user=SA] will not be managed by Spring
==>  Preparing: UPDATE users SET password=?,username=? WHERE (id = ?)
==> Parameters: null, Marry(String), 19(Integer)
<==    Updates: 1
```

# 结束

这样发现这个真的很方便了，完全弥补了 tk.mapper 缺失更新的灵活性
