---
title: Oracle
date: 2022-12-15 21:00:21
tags:
---

# 存储单元

- **表空间（Table Space）**是存储例如表、索引、序列等对象的逻辑存储器。每个 Table Space 至少具备一个物理的 Data File 来在操作系统层面真实地存储 Table Space 中的内容，Table Space 和 Data File 是一对多的关系。
- **段（Segment）**用于存储 Table Space 中的对象，是一种消耗物理存储空间的任何逻辑单元。一个 Table Space 可以有多个 Segment，但一个 Segment 只属于一个 Table Space，Table Space 和 Segment 是一对多的关系。Segment 和 Data File 没有直接关系，一个 Segment 可能存在于多个 Data File 中。
- **扩展（Extent）**是连续的磁盘存储块的逻辑集合。Segment 和 Extent 是一对多的关系，Data File 和 Extent 也是一对多的关系。
- **数据块（Block）**是 Oracle 服务器管理数据的最小逻辑单元。Extent 和 Oracle Block 是一对多的关系，Oracle Block 和 OS Block 是一对多的关系，Data File 和 OS Block 是一对多的关系。

<img src="存储单元.jpg" alt="存储单元" style="zoom:100%;" />

# SQL 执行

- **Syntax Check**：确保 SQL 符合基本语法规则
- **Semantic Check**：确保 SQL 具备意义，比如涉及到的对象、表、列是存在的
- **Shared Pool Check**：判断 SQL 执行 Hard Parse 还是 Soft Parse（Soft Parse 会跳过 Optimization 和 Row Source Generation，直接 Execution），DDL 语句始终都会进行 Hard Parse
- **Optimization**：对于每个具备唯一哈希值的 DML都至少进行一次 Optimization，但 Oracle 从来不会优化 DDL（除非该 DDL 中存在需要优化的 DML 部分），目前 Oracle 采用的是基于代价的优化器（CBO），这种判断方式非常依赖于数据对象的统计分析信息
- **Row Source Generation**：从优化器接收最优执行计划，并生成可供数据库其余部分使用的迭代执行计划
- **Execution**：SQL 引擎执行 Row Source Generation 生成的树中的每个 Row Source（这个步骤是DML处理中唯一的强制步骤）

|                                |                              |
| :----------------------------: | :--------------------------: |
|       **Grammar Order**        |     **Execution Order**      |
|    1) SELECT DISTINCT <..>     | 1) FROM<..> ON<..> JOIN <..> |
| 2) FROM <..> JOIN <..> ON <..> |        2) WHERE <..>         |
|         3) WHERE <..>          |       3) GROUP BY <..>       |
|        4) GROUP BY <..>        |        4) HAVING <..>        |
|         5) HAVING <..>         |   5) SELECT DISTINCT <..>    |
|        6) ORDER BY <..>        |       6) ORDER BY <..>       |

# 执行计划

优化器给出的执行计划是一种推荐的用于执行 SQL 的方法，从执行计划中可以得到数据库在执行 SQL 时所采用的访问和连接表的方式。

执行计划所列出的方式不一定是最高效的，通过观察执行计划中的内容可以判断当前 SQL 的是否能够高效地执行。

比如可以观察优化器所估计的每一步操作将处理多少行，以及估计的行数（即基数）与实际数量的比较情况

- 如果基数估计是错误的，这意味着可以通过改写 SQL 来帮助优化器找到更优的执行计划
- 如果基数估计是准确的，但查询仍然太慢，则需要查看是否可以创建一些数据结构来更快地访问数据
- 如果表访问方式和表连接方式与期望的方式不同，则基数估计基本就是错误的
- 需要注意的是成本（Cost）是优化器用来比较同一 SQL 的不同执行计划的内部度量，不同 SQL 之间的 Cost 的比较是没有意义的，不应通过这种比较来对比 SQL 改写前后的性能优劣

在遵循执行计划来执行 SQL 时，是根据中序遍历来执行计划中的所有操作的，即从执行计划的顶部开始，沿着树向下执行第一个叶子操作，然后沿着树返回到第一个带有未访问子节点的节点，重复这个过程直到执行了执行计划中的所有操作（UNION 操作是后序遍历，存在子查询时情况有变，详细见参考资料）。

# 表访问方式

- TABLE ACCESS FULL（全表扫描）
  - 读取表中所有的行，并检查每一行是否满足 SQL 语句中的 WHERE 限制条件
  - 除非真正需要返回的数据较多，不建议使用全表扫描来读取数据量太大的表
- TABLE ACCESS BY ROWID（通过 ROWID 的表存取）
  - ROWID 是由 Oracle 自动加在表中每行最后的一列伪列，表中并不会物理存储 ROWID 的值，因为不能对该列的值进行增删改操作，但是可以进行查询
  - 行的 ROWID 指出了该行所在的数据文件、数据块以及行在该块中的位置，因此通过 ROWID 来定位目标行是非常高效的，是 Oracle 中存取单行数据最快的方法
  - Oracle 要么从WHERE 条件要么从索引扫描来获取需要访问的行的 ROWID 
- TABLE ACCESS BY INDEX SCAN（索引扫描）
  - 索引既存储每个索引的键值，也存储具有该键值的行的ROWID 
  - 索引扫描分两步，先扫描索引得到对应的 ROWID（内存读取，速度较快），再通过 ROWID 定位到具体的行读取数据
  - 当只需要返回部分行的数据时，对于无范围操作符的唯一索引则使用索引唯一扫描，对于非唯一索引或者有范围操作符的唯一索引则使用索引范围扫描
  - 当需要返回所有行的数据时，对于排序操作则使用索引全扫描，对于无排序操作则使用索引快速扫描

# 表连接方式

在使用 JOIN 关键字来对两张表作连接，表（row source）之间的连接顺序对于查询效率有很大的影响。

根据连接顺序的不同，将作连接的两张表分别定义为驱动表和匹配表

- 驱动表（Driving Table）
  - 驱动表指连接时首先存取的表，又称外层表
  - 驱动表的全表数据集会作为循环基础数据，其中每一条数据都会作为过滤条件到被驱动表中查询数据，最后合并
  - 如果驱动表返回比较多的数据行，则对所有的后续操作有负面影响
  - 驱动表需要全表扫描，不走索引，所以建不建索引都无法优化性能
  - 一般选择小表（应用 WHERE 限制条件后返回较少行数的表）作为驱动表
- 匹配表（Probed Table）
  - 又称内层表
  - 从驱动表获取一行具体数据后，会到内层表中寻找符合连接条件的行
  - 可以通过将匹配表中的与驱动表连接的字段建立索引，从而优化性能
  - 一般选择大表（应用 WHERE 限制条件后返回较多行数的表）作为匹配表

内连接时小表为驱动表，左外连接时左表为驱动表，右外连接时右表为驱动表。

表连接方式有以下几种

- SORT MERGE JOIN（排序 - 合并连接）
  - 一般用于两个表之间的连接条件为不等于条件的情况，例如<、<=、>或>=
  - 在联合大数据集时，性能比 NESTED LOOPS 更好
- NESTED LOOPS（嵌套循环）
  - 一般用于联合小数据集且能高效访问匹配表的情况
  - 可以看作是两个嵌套的 FOR 循环
- HASH JOIN（哈希连接）
  - 一般用于联合大数据集的情况
  - 优化器选择两个表中较小的表来在内存中建立哈希表，接着扫描更大的表，并在 JOIN 的列上执行相同的哈希算法
- CARTESIAN PRODUCT（笛卡尔积）
  - 通常只只用于涉及的表很小，或者语句中一个或多个表没有到任何其他表的连接条件的情况
  - 优化器将来自一个数据源的每一行与来自另一个数据源的每一行连接起来，创建两个集合的笛卡尔积
  - 笛卡尔连接并不常见，因此如果它是由于任何其他原因而选择的，那么它可能是基数估计有问题的标志（严格地说，笛卡尔积不是一个连接）

# 优化技巧

- 建立数据结构

  - 当 SQL 很慢且无法通过改写 SQL 来提升性能时，就可以创建一些数据结构来更快地访问数据
  - 比如创建索引、临时表、实体视图、约束、分区、甚至优化表的设计

- 去掉冗余

  - 不要 SELECT 不需要的字段
  - 不要 JOIN 不需要的表

- 尽可能减少需要重复的操作/查询

- EXISTS 和 IN

  - 两者之间并不一定是前者性能更好，和连接查询类似，这里也存在驱动表和匹配表的关系

  - 应当根据表的大小来选择驱动表和匹配表，从而确定使用 EXISTS 还是 IN

    ```sql
    SELECT * FROM A WHERE id IN (SELECT id FROM B); -- B 是驱动表
    SELECT * FROM A WHERE EXISTS (SELECT 1 FROM B WHERE B.id = A.id); -- A 是驱动表
    ```

- 使用连接而不是子查询，因为它为优化器提供了更多的空间

- 添加逻辑上多余的过滤条件，这些条件可能仍然有助于搜索最佳执行计划，特别是对于外部连接。

- 避免数据类型的隐式转换，特别是在 WHERE 子句中

- 尽可能避免对条件中的索引列使用<>、NOT IN、NOT EXISTS 和 LIKE 等不带前导 '%' 的比较操作符

- 不要对 WHERE 子句中的索引列应用函数

- 不要在聚合之前滥用 HAVING 过滤行

- 避免不必要的排序，包括当使用 UNION ALL 而不是 UNION 时

- 除非必须使用，否则避免使用DISTINCT

TBC

# 语法小抄

- 不区分 `""` 和 null

- 分页 `... OFFSET 0 ROWS FETCH NEXT 50 ROWS ONLY`（不是使用 `LIMIT`）

- 针对某一列进行去重（`DISTINCT`是对返回的所有列进行去重，此时返回的某一列仍旧存在重复列）

  ```sql
  SELECT * FROM (
      SELECT 
          DISTINCT column1,column2,
          row_number() OVER(PARTITION BY column1 ORDER BY column2) AS row_cnt 
      FROM tableA
      ...
  ) WHERE row_cnt = 1
  ```

- 排序时忽视大小写 `ORDER BY NLSSORT(column1, 'NLS_SORT=BINARY_CI')`

- 定义一个 SQL 片断 `WITH AS`

- 多行拼接为一行 `listagg(column, ';' on overflow truncate)`

- 查看执行计划 `EXPLAIN PLAN FOR SQL; SELECT * FROM table(dbms_xplan.display);`

- 查看当前 session 锁住的对象

  ```sql
  SELECT c.owner, c.object_name, c.object_type, b.sid, b.serial#, b.status, b.osuser, b.machine 
  FROM v$locked_object a, v$session b, dba_objects c 
  WHERE b.sid = a.session_id AND a.object_id = c.object_id ORDER BY object_name;
  ```

- kill 当前 session 

  ```sql
  ALTER SYSTEM KILL SESSION ':sid, :serial#, @1';
  ```

  

# 参考资料

- https://www.oracletutorial.com/oracle-administration/oracle-database-architecture
- https://docs.oracle.com/database/121/TGSQL/tgsql_sqlproc.htm#TGSQL175
- https://docs.oracle.com/database/121/TGSQL/tgsql_optcncpt.htm#TGSQL196
- https://docs.oracle.com/database/121/TGSQL/tgsql_interp.htm#TGSQL277
- https://blogs.oracle.com/connect/post/how-to-read-an-execution-plan
- https://blog.csdn.net/weixin_42156231/article/details/124920365
- https://blog.csdn.net/weixin_42156231/article/details/124749986
- https://www.oracle.com/docs/tech/database/technical-brief-explain-the-explain-plan-052011.pdf
- https://blog.csdn.net/weixin_44390164/article/details/119970753
- https://oracle.readthedocs.io/en/latest/sql/hints/optimization.html

