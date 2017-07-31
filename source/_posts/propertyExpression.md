---
title: Mybatis Dynamic Query 属性表达式
date: 2017-07-31 16:34:14
tags:
- Mybatis Dynamic Query
- java
---
项目地址：https://github.com/wz2cool/mybatis-dynamic-query  
文档地址：https://wz2cool.gitbooks.io/mybatis-dynamic-query-zh-cn/content/
## 简介 ##
作为.net 出生的我，一直被C#强大的语法所吸引，比如我们很容易获得一个属性的名字
```java
public static string GetPropertyName<T>(Expression<Func<T>> expression)
{
    MemberExpression memberExpression;
    if (expression.Body is UnaryExpression)
    {
        var unaryExpression = (UnaryExpression)expression.Body;
        memberExpression = (MemberExpression)unaryExpression.Operand;
    }
    else
    {
        memberExpression = (MemberExpression)expression.Body;
    }
    return memberExpression.Member.Name;
}

// call with lambda
var propertyPath = ObjectHelper.GetPropertyName(() => firstOrder.OrderDate),
```
在1.0.2 中我们同样支持了这个属性，当然我们是站在别人的肩膀上实现的（jodd的一个methodref实现）。

## 简单封装 ##
我们封装了jodd 的这个实现到了CommonsHelper中去，并且加上synchronized防止多线程，如果我们看到jodd 源码就知道，里面用到的是一个WeakHashMap做缓存，那么这个WeakHashMap线程不安全，所以我们还是让我们调用的方法线程安全。
### 约定 ###
1. T 就是我们实体类，而Class<T> 就是我们实体类的类型。
2. getMethodFunc 一定是与之对应field 的一个get方法。
```java
// 主要这里我们通过get方法来获取field 名字。
public static <T> String getPropertyName(final Class<T> target, 
    final Function<T, Object> getMethodFunc) {
        String methodName = obtainGetMethodName(target, getMethodFunc);
        if (methodName.startsWith("get")) {
            return java.beans.Introspector.decapitalize(methodName.substring(3, methodName.length()));
        }

        return methodName;
}

public static synchronized <T> String obtainGetMethodName(final Class<T> target, 
    final Function<T, Object> getMethodFunc) {
        Methref<T> methodRef = Methref.on(target);
        getMethodFunc.apply(methodRef.to());
        return methodRef.ref();
}
```
### 实际应用 ###
这里主要给 FilterDescriptor 和 SortDescriptor 设置propertyPath 提供了一个新的方法。
```java
public <T> FilterDescriptor(FilterCondition condition,
                                Class<T> entityClass,
                                Function<T, Object> getFieldFunc,
                                FilterOperator operator,
                                Object value) {
        this.setCondition(condition);
        this.operator = operator;
        this.propertyPath = CommonsHelper.getPropertyName(entityClass, getFieldFunc);
        this.value = value;
}
```
```java
public <T> SortDescriptor(Class<T> entityClass, Function<T, Object> getFieldFunc, SortDirection sortDirection) {
        this.propertyPath = CommonsHelper.getPropertyName(entityClass, getFieldFunc);
        this.sortDirection = sortDirection;
}
```
这样我们在创建时候可以这样子：
```java
@Test
public void lambdaNewInstanceTest() {
    FilterDescriptor filterDescriptor =
        new FilterDescriptor(FilterCondition.AND,
            Student.class, Student::getAge,
            FilterOperator.EQUAL, "3");
    // 验证我们设置的筛选属性是否就是age
    assertEquals("age", filterDescriptor.getPropertyPath());
}

@Test
public void lambdaSetPropertyTest() {
    FilterDescriptor filterDescriptor = new FilterDescriptor();
    filterDescriptor.setPropertyPath(Student.class, Student::getAge);
     // 验证我们设置的筛选属性是否就是age
    assertEquals("age", filterDescriptor.getPropertyPath());
      // 我们尝试使用lambda获取
    filterDescriptor.setPropertyPath(Student.class, (s)-> s.getNote());
    assertEquals("note", filterDescriptor.getPropertyPath());
}
```
## 结束 ##
这里我们类似于用lambda 表达式方法获取到属性名字，看似我们绕了很大一个圈子去做了这件事，当然我们写实一个String 也可以，但是这样做是有几个好处的：
1. 当编写的时候，不用特意去记住属性名字，IDE智能提醒找到对应的get方法。
2. 当修改属性的时候，你改变了属性名字，当你修改get方法，其他地方也会相应有提示改动。
3. 避免sonar 报错，sonar当你string 有三处一样的时候就会报错。（我去……是强凑了一个）

## 关注@我　##
最后大家可以关注我和 Mybatis-Dynamic-query项目 ^_^
<a class="github-button" href="https://github.com/wz2cool" data-size="large" data-show-count="true" aria-label="Follow @wz2cool on GitHub">Follow @wz2cool</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query" data-size="large" data-show-count="true" aria-label="Star wz2cool/mybatis-dynamic-query on GitHub">Star</a> <a class="github-button" href="https://github.com/wz2cool/mybatis-dynamic-query/fork" data-size="large" data-show-count="true" aria-label="Fork wz2cool/mybatis-dynamic-query on GitHub">Fork</a>