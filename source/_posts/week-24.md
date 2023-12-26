---
title: 2023.2.6 - 2023.2.10
date: 2023-02-11 09:23:26
password: 1062597351
tags:
---

> - 工作相关：Code Review、JMeter、一点想法
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)、[Apache JMeter](../../../../2023/02/11/jmeter/)

### Code Review

记录一下最近别人帮我做 cr 时指出的一个问题。

当需要从 `Map` 中获取一个 `value`，如果没有就创建一个 `value` 并放进 `Map` 时，我往常会这样写

```java
DTO item = hashmap.getOrDefault(key, new DTO());
// 处理逻辑
hashmap.put(key, item);
```

为了避免每次获取 `value` 的时候都创建新的 `DTO` 对象，可以改成

```java
DTO item = Optional.ofNullable(hashmap.get(key)).orElseGet(DTO::new);
// 处理逻辑
hashmap.put(key, item);
```

如果可以将 `get` 和 `put` 逻辑写到一起而不影响这之间的处理逻辑时，更加优雅的写法是

```java
DTO item = hashmap.computeIfAbsent(key, k -> new DTO());
// 处理逻辑
```

### Apache JMeter

最近去试用了一个测试工具 JMeter，然后重新对之前优化的一个接口测试了优化后的性能，根据使用教程和测试 demo 产出了一篇文档（在工作电脑上，懒得重写就直接截图搬到 [JMeter](../../../../2023/02/11/jmeter/) 上了）。

在没使用这个工具之前，我是通过 shell 脚本来批量测试接口并计算平均响应时间的，这样需要额外写代码，这个工具的优点在于图形化界面可以很简单方便的配置测试场景，尤其是多线程并发的场景（那么也很适合用来做压测），并且生成一个正式详细美观的测试报告（那么就挺适合用来作为 PPT 的素材）。

有了这个工具之后，下周可以快速尝试验证更多的性能优化方案。

### 写在最后

- 有时候思路会局限在一些自以为是的地方，比如自以为让我去熟悉这个测试工具是为了让我产出文档做分享，然后就产生了抗拒心理，但后来才意识到熟悉它是为了将来可以更加高效的验证优化方案的有效性，低成本的尝试更多优化方案，意识到这里就有热情了
- 过去几年一直在锻炼写文档和做 PPT 的能力，锻炼到本末倒置的程度并产生了很强的逆反心理和羞耻感，现在只要有可能的话都会故意避开这种风格，有时候反而杯弓蛇影了
- 将项目视作是一种有机的、进化着的生命体而去培养，而非追求快速见效