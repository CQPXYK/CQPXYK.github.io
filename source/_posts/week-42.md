---
title: 2023.6.12 - 2023.6.16
date: 2023-06-15 18:58:56
password: 1062597351
tags:
---

> - 工作相关：Upstream data extracts (UAT Support/File Backup & Recovery/MockMultipartFile)
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

### UAT Support

在 UAT 上测试时发现，相同 SQL 在不同 DB 环境的执行情况可能非常不同，比如一下 SQL 执行时可能会造成 session 长时间锁住，改成另一种写法后就解决了，以后记得要去不同的 DB 都测试下

```sql
UPDATE T1 SET COL1 = 'VAL1'
WHERE ID IN (SELECT T1.ID FROM T1 LEFT JOIN T2 ON T1.ID = T2.ID WHERE T2.ID IS NULL)

-- 上面的 SQL 可能存在执行过慢/死锁，改写为
UPDATE T1 SET COL1 = 'VAL1'
WHERE NOT EXISTS (SELECT T2.ID FROM T2 WHERE T1.ID = T2.ID)
```

### File Backup & Recovery

关于上游文件备份与恢复的补充功能，开发难度不大，但是由于自己对 Java IO 这块知识点不熟悉，借鉴了很多别人的示例代码，故留存一下代码示例

XML:

<img src="1.png" alt="" style="zoom:80%;" />

<img src="2.png" alt="" style="zoom:80%;" />

<img src="3.png" alt="" style="zoom:80%;" />

<img src="4.png" alt="" style="zoom:80%;" />

<img src="5.png" alt="" style="zoom:80%;" />

<img src="6.png" alt="" style="zoom:80%;" />

EXCEL:

<img src="7.png" alt="" style="zoom:80%;" />

<img src="8.png" alt="" style="zoom:80%;" />

<img src="9.png" alt="" style="zoom:80%;" />

<img src="10.png" alt="" style="zoom:80%;" />

<img src="11.png" alt="" style="zoom:80%;" />

<img src="12.png" alt="" style="zoom:80%;" />

<img src="13.png" alt="" style="zoom:80%;" />

### MockMultipartFile

关于上游数据获取需求的单测，大量使用了`MockMultipartFile`来进行测试，为以后方便使用，故留存以下创建各种类型`MockMultipartFile`的工具类代码

<img src="14.png" alt="" style="zoom:80%;" />

<img src="15.png" alt="" style="zoom:80%;" />

<img src="16.png" alt="" style="zoom:80%;" />

<img src="17.png" alt="" style="zoom:80%;" />

<img src="18.png" alt="" style="zoom:80%;" />

<img src="19.png" alt="" style="zoom:80%;" />