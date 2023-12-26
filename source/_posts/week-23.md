---
title: 2023.1.30 - 2023.2.3
date: 2023-02-04 12:41:25
password: 1062597351
tags:
---

> - 工作相关：Prod Issue (LISTAGG、SQL 参数限制)
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

我再也不期待所谓的 bug free 了，如果 QA 没有测出 bug，那就意味着得出 prod issue

### LISTAGG

`listagg` 是 Oracle 的一个聚合函数，用于拼接字符串（将所有行拼成一行），但如果带拼接的字符串超出 4000 个字符时，`listagg` 执行会出错。因此，在使用 `listagg` 时应当注意是否能确保需要拼接的字符串不超过 4000，如果不确定或者说数据会在未来不断增长的话，那么应当配合截断来使用，即

```sql
listagg(column, ';' on overflow truncate)
```

`listaggclob` 和 `xmlagg` 同样能够实现 `listagg` 的功能，区别在于

- `listaggclob` 和 `xmlagg` 会将需要字符转为二进制，与此同时也不存在 4000 个字符限制

- `listaggclob` 和 `xmlagg` 会占用更多的 PGA 内存，当应用频繁执行那些使用了 `listaggclob` 的 SQL 时，容易超出 Oracle 的 PGA 限制，从而导致 API 执行出错，且性能也不好

  

因此，在业务不严格要求完整的超出 4000 字符的拼接结果时，使用带截断的 `listagg` 函数即可，若这种不完整的结果无法满足业务要求，则应当由应用程序来完成字符串的拼接工作。

### SQL 参数限制

执行 SQL 时传入的参数个数不应超过1000个，超过1000时会报错，可以改成 `join` 临时表或者分批执行 SQL。