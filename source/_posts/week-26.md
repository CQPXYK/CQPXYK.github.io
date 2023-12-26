---
title: 2023.2.20 - 2023.2.24
date: 2023-02-25 13:07:15
password: 1062597351
tags:
---

> - 工作相关：日志配置调整、熟悉业务代码
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

### 日志配置

工作中是使用 kibana 来查看各个环境的日志的，有两个使用不便的问题是：

1）因为异常堆栈信息是在 error 那条日志之后换行，并一行一行打印的，当通过 error 作为关键字来查询过滤报错日志时，无法查询到报错的堆栈信息。

为了方便查询异常堆栈信息，希望将堆栈信息在报错日志之后紧跟着在同一行打印出来，并以`\n`作为分隔符将堆栈信息分隔开来（查询到之后可以通过 Sublime 将`\n`替换为回车效果）。

2）日志打印的服务器时间（美国东部时间），但是在使用 Kibana 按时间对日志进行过滤时，需要使用格林威治时间进行过滤，需要人为转换时区并更改为 Elasticsearch Query DSL 格式进行查询过滤。

为了方便按照时间过滤，希望在当前日志的基础上增加符合 Elasticsearch Query DSL 格式的 GMT 时间的打印。

更改前后的日志配置如下

```properties
logging.pattern.console=%clr($d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] [traceId: %X{traceId}] [%logger{50}: %line] - %msg%n)
```



```properties
logging.pattern.console=%clr($d{yyyy-MM-dd HH:mm:ss.SSS} [GMT: %d{yyyy-MM-dd,GMT}T%d{HH:mm:ss,GMT}] %-5level [%thread] [traceId: %X{traceId}] [%logger{50}: %line] - %m%replace(%ex{full}){"[\r\n]+", "\\\\n"}%n)
```

有空去熟悉一下 [Elasticsearch Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

### SQL Developer

在熟悉业务代码的过程中，为了调试一个存储过程的逻辑，我更改了存储过程代码并对存在语法问题的存储过程进行了编译，出现编译错误之后我想要恢复为原本的代码，但是发现一旦对函数体进行任何修改，界面就会完全卡住，更不可能对恢复后的函数体重新编译，但是查询`SYS.V_$session`时也并没有发现锁。

后来解决方式是，将存储过程函数体文件整个导出为`.sql`文件，然后通过 VSCode 或者 Sublime 对导出的`.sql`文件进行编辑并保存，最后在 SQL Developer 的 Worksheet 中执行该`.sql`文件，即可令更改后的存储过程生效。

### 写在最后

- 每周写工作记录确实做不到，不是因为我主观上懒得写，而是客观上确实没有值得记录的工作内容，这也是现在对工作和对自己都日益失望的原因之一
- 现在看书上看到某个点会回忆起当时绕了很大一个圈子才做成书里描述的样子，如果那个时候就已经看到了这个点，那会儿也就不必绕圈子了
- 当意识到某个地方有问题或者不好用的时候，要敢于解决问题并把力气用在解决问题上，不需要去抱怨问题或者消耗情绪