---
title: 2023.7.3 - 2023.7.7
date: 2023-07-07 21:31:16
password: 1062597351
tags:
---

> - 工作相关：JSON File Parsing and Cached by Redis
> - 博客更新：[Test Driven Development](../../../../2022/09/30/tdd/)，[Java Concurrency](../../../../2022/05/08/java-concurrency/)

### JSON File Paring

<img src="1.png" alt="" style="zoom:80%;" />

<img src="2.png" alt="" style="zoom:80%;" />

<img src="3.png" alt="" style="zoom:80%;" />

### Redis Configuration

<img src="4.png" alt="" style="zoom:80%;" />

<img src="5.png" alt="" style="zoom:80%;" />

<img src="6.png" alt="" style="zoom:80%;" />

<img src="7.png" alt="" style="zoom:80%;" />

<img src="8.png" alt="" style="zoom:80%;" />

<img src="9.png" alt="" style="zoom:80%;" />

<img src="10.png" alt="" style="zoom:80%;" />

<img src="11.png" alt="" style="zoom:80%;" />

<img src="12.png" alt="" style="zoom:80%;" />

### Cached by Redis

<img src="13.png" alt="" style="zoom:80%;" />

<img src="14.png" alt="" style="zoom:80%;" />

<img src="15.png" alt="" style="zoom:80%;" />

<img src="16.png" alt="" style="zoom:80%;" />

<img src="17.png" alt="" style="zoom:80%;" />

以上`RedisUtil`所采用的 redis 数据导入导出的方式效率并不高，如果数据量比较大时，可以考察使用`Pipline`进行性能上的优化（[参考](https://blog.csdn.net/zhanglong_4444/article/details/87921162)）。