---
title: 2022.12.19 - 2022.12.23
date: 2022-12-19 13:41:45
password: 1062597351
tags:
---

> - 工作相关：慢加载分页（UAT）、代码规范（拼接字符串、动态生成 SQL、线程池的使用、重构大段 else if）
> - 博客更新：[ORACLE](../../../12/15/oracle/)

这周 QA 给了 sign off，实现了 bug free，接下来就是上 UAT，此外同事帮我做了 cr，现记录主要建议如下。

### 拼接字符串

如果是简单的静态字符串拼接，可以直接使用`+`来拼接`String`类型的字符串，因为编译器在编译阶段会直接计算出结果

如果是拼接动态计算得到的字符串，在多线程并发情况下，只能使用`StringBuffer`，在不存在多线程并发的情况下，一般使用`StringBuilder`

- 如果直接使用`+`来拼接`String`类型的字符串，JDK 底层会创建`StingBuilder`对象，并通`StingBuilder`的`append`方法来执行拼接，优化了原本通过不断创建新`String`对象的拼接字符串的方式，但此时对于每一行字符串拼接的代码（一个分号），都需要在底层创建一个`StingBuilder`对象

- 如果是在一个循环当中循环执行字符串拼接，使用`String`意味着每次循环都需要在底层创建`StingBuilder`对象，此时性能远远低于直接使用`StringBuilder`，而且显式地创建`StringBuilder`还允许预先指定其大小，避免多次重新分配缓存

- 在引入了偏向锁和锁升级后，无锁竞争时的`StringBuffer`和`StringBuilder`的性能是一致的，所以非线程并发时使用`StringBuilder`其实也可以，并不会带来性能上的下降

  

### 动态生成 SQL

在 [ORACLE](../../../12/15/oracle/) 中有记录到，对于每个具备唯一哈希值的 SQL 语句来说，都需要进行一次 Optimization 来生成最优的执行计划，因此在需要动态的生成 SQL 语句时，在动态传参和动态拼接之间会更推荐前者的方式来实现，代码如下

```java
if (StringUtils.isNotEmpty(paraDTO.getName())) {
    builder.append(" AND NAME LIKE concat('%', concat(:name, '%')) ");
    paraSource.addValue("name", paraDTO.getName());
}
```

此时，即便动态计算得到的参数发生了变化，拼接得到的 SQL 语句并不会随着参数的变化而变化，那么就能够复用之前生成的执行计划，而不需要反复进行 Hard Parse。

另外需要注意的是，用于模糊匹配的`%`在 SQL 中进行拼接更加合适，而不是在传入的参数中拼接，写代码应该多思考什么逻辑由应用程序做更合适，什么逻辑由数据库做更合适，比如日期时间转换的逻辑就更适合在应用程序中实现。

### 线程池的使用

使用线程池是为了避免频繁的创建和销毁线程，那么在代码中使用线程池时就应当注意是否发挥了线程池的优势，比如声明一个无法被多个线程共享的线程池是毫无意义的，因此，一般会将线程池声明为一个服务类（单例）的成员变量，或者一个工具类的静态变量

- 在创建一个线程池时，应当根据应用的并发情况，合理设置线程池的参数，比如并发密集时增大`maximumPoolSize`和`keepAliveTime`

- 当`corePoolSize`为零时，池内线程在执行完任务后会在达到`keepAliveTime`后被 JVM 回收，当`corePoolSize`大于零时，应该在`destory`方法中调用线程池的`shutdown`方法，否则池内线程将不会被回收，并产生内存泄漏问题

  

### 重构大段 else if

如果代码在迭代过程中渐渐出现大段的`else if`时，可根据 [Design Patterns](../../../05/11/design-patterns/) 中的策略模式进行重构，此外，如果`else if`中的逻辑非常简单（比如只有一行代码），也可以使用`Map`进行重构，代码如下

```java
private void fun1(User user, String type, String id) {
    if ("type1".equals(type)) {
        user.setId1(id);
    }
    else if ("type2".equals(type)) {
        user.setId2(id);
    }
    else if ("type3".equals(type)) {
        user.setId3(id);
    }
}

// 重构为

private void fun2(User user, String type, String id) {
    Map<String, BiConsumer<User, String>> typeMap = new HashMap<>();
    typeMap.put("type1", (obj,value) -> obj.setId1(value));
    typeMap.put("type2", (obj,value) -> obj.setId2(value));
    typeMap.put("type3", (obj,value) -> obj.setId3(value));
    typeMap.get(type).accept(user, id);
}
```

