---
title: 2022.10.17 - 2022.10.21
date: 2022-10-17 16:44:10
password: 1062597351
tags:
---

> - 工作相关：慢加载优化（问题分析、Java 转 SQL、自测、SQL 优化）
>- 博客更新：无

### 问题分析

有一个 HTTP 请求执行时间很长，这个请求中涉及到很多次查询。

- 经过工具分析得到，请求执行时间长主要在于其中一个大查询的执行时间长（缓存过期时可能长达二三十秒）
- 经过测试得到，结果转化的执行时间与数据库返回数据量的行数成正比（每一万行大约需要一秒）
- 经过代码梳理得到，在得到第一次查询的结果和请求最终返回的数据之间，存在一系列逻辑处理（其中包含过滤逻辑）

综上所述，可以尝试的优化方案如下

- 结合执行计划来优化 SQL 语句本身，降低数据库执行 SQL 的时间
- 将代码中的过滤逻辑前置到 SQL 查询中，避免数据库返回不必要的数据，降低结果转化为Java 对象的时间

### Java 转 SQL

比如现在有段 Java 逻辑是两层 `for` 循环遍历一个 `listA` 中的每个元素的`String`类型字段 `strOFListB` 被切分后的所有元素值，根据该值是否存在于另一个查询得到的 `listC` 结果集来判定得到一个布尔型的`flag`取值，最终会根据该`flag`的取值来做过滤。

```java
public List<E> filter() {
    List<E> listA = getFromDB(query1);
    List<Long> listC = getFromDB(query2);
    for (E eOfA : listA) {
        boolean flag = true;
        String strOFListB = eOfA.getStrOFListB();
        String[] listB = strOFListB.split(";");
        for (String eOfB : listB) {
            if (StringUtils.isBlank(eOfB.trim()) || !NumberUtils.isDigits(eOfB.trim())) {
                continue;
            }
            if (!listC.contains(Long.parseLong(eOfB)) {
                flag = false;
            }
            else {
                flag = true;
                break;
            }
        }
        eOfA.setFlag(flag);
    }
    List<E> finalList = listA.stream().filter(o -> o.getFlag()).collect(Collectors.toList());
    return finalList;
}
```

其中，用于获取`listA`和`listC`的`queryOfA`和`queryOfC`大概内容如下

```SQL
-- queryOfA
SELECT ...,ta.eOfAID, ta.strOFListB,... FROM t
LEFT JOIN (
    SELECT
        ...,
        Listagg(eOfB, ';') Within Group (Order By eOfAID) strOFListB,
        ...
    FROM (
        SELECT ..., eOfB, ...
        FROM t1
        LEFT JOIN t2 ON ...
        ...
    )
    Group By eOfAID
) ta ON ta.eOfAID = t.eOfAID
WHERE ...
```

```sql
-- queryOfC
SELECT eOfC From t2 WHERE ...
UNION
SELECT eOfC From t3 WHERE ...
```

顾名思义，`eOfA`的字段`strOFListB`虽然是`String`类型，但是其实是一个被拼成`String`的`List`，之所以被拼成`String`，是因为在 SQL 的返回结果中需要将每个`eOfA`的多行`eOfB`合成一行`strOFListB`。

现在的任务相当于是将上文的 Java 代码和`queryOfC`的逻辑都改进`queryOfA`当中，从而直接从`queryOfA`获得过滤之后的`finalList`，修改后的`queryOfA`如下

```sql
-- modified queryOfA

SELECT ...,ta.eOfAID, ta.strOFListB,... FROM t
LEFT JOIN (
    SELECT
        ...,
        Listagg(eOfB, ';') Within Group (Order By eOfAID) strOFListB,
        CASE WHEN (COUNT(flag_cnt) = 0 OR SUM(flag_cnt) > 0) THEN true
        ELSE false END AS flag,
        ...
    FROM (
        SELECT 
            ..., eOfB, ...,
            CASE WHEN eOfB == tc.eOfB THEN 1
            CASE WHEN (eOfB IS NOT NULL AND tc.eOfB IS NULL) THEN 0
            ELSE null END AS flag_cnt
        FROM t1
        LEFT JOIN t2 ON ...
        LEFT JOIN (
            SELECT eOfC From t2 WHERE ...
            UNION
            SELECT eOfC From t3 WHERE ...
        ) tc ON ...
        ...
    ) 
    Group By eOfAID
) ta ON ta.eOfAID = t.eOfAID
WHERE 
    ...
    AND ta.flag = true
    ...
```

目前 SQL 用的还不够熟，一开始还想在 SQL 里面对`strOFListB`进行切分，然后在这一步卡了很久，后来被同事提醒了才知道可以在通过`LEFT JOIN queryOfC`拿到`listC`之后，直接把`!listC.contains(Long.parseLong(eOfB)`判断逻辑加在`eOfB`被合成`strOFListB`之前就行。

### 结果测试

博客只是抽取了一部分的内容来记录，项目中实际的 Java 代码和 SQL 比上文所述要庞大很多，这次将 Java 转 SQL 涉及到的修改也很多，因此将修改前后的结果进行对比测试是很有必要的。

保险起见，给 QA 测之前，自己先写个脚本来依次使用所有的用户 ID 调用该 HTTP 请求，在请求代码中对修改前和修改后的两个已排序后的`finalList`结果进行逐一对比，然后就发现了上述修改后的 SQL 由于加了更多的`LEFT JOIN`，导致`tc`表返回的行数变多，进而导致合并得到的`strOFListB`中多了一些重复的`eOfB`，所以如果需要严格保持修改前后结果一致的话，就需要再加个去重，但是可以再去找同事讨论下有没有这个必要。

在用脚本从 csv 中拿数据这里卡了很久，从打印出来的内容看拿到的数据和拼出来的 url 都没问题，但是执行`curl`时一直显示 url 格式错误，我猜可能拿到的数据后面有换行之类的东西？改了很久没改出来后来直接换了一个读数据的方式总算跑通了，感觉学 Linux 迫在眉睫了。

```bash
url="http://localhost:xxx/xxx?id="
cnt=0
while read line
do
	cnt=$((cnt+1))
	id=$(echo ${line} | cut -d , -f 1)
	
	echo "cnt = "$cnt
	echo "id = "$id
	
	curl -H 'Content-Type: application/json' -d '{}' $url$id
	clear
done < IDs.csv
```

### SQL 优化

这段时间刚好在看《高性能MySQL》这本书，但是当我在根据书中的内容对 SQL 进行优化时，执行计划分析得到的 Cost 和 Time 并没有变化，可能 MySQL 的优化并不一定适用于 Oracle ，或者执行计划中显示的 Cost 和 Time 并不一定能反映 SQL 实际执行的消耗，目前是卡在了这里，下周可以继续和同事讨论这个问题。

### 写在最后

- 本周实现了第一种优化方案，带来的效果是对于过滤量比较大的用户，请求会快很多，但是对于过滤量比较小的用户，请求会比之前更慢，这个结果与理论是相符的
- 后续两种方案下周继续，尤其是根据执行计划来优化 SQL 语句本身，难得有这种机会可以给我试试
- 上周纠结是不是不应该将多个查询合并为一个查询，但还是要考虑实际需求，比如有的情况下数据库返回了太多不必要的数据，将过滤逻辑前置到 SQL 中虽然让查询更慢，但是节省了结果转换时间，针对实际情况做权衡吧
- 上周五接到需求后还在纠结自己不敢动老代码，这周在跟同事确认了这真的是我可以动的之后就开始撸起袖子干了，这是我入职以来最开心的一周
- 最近跟比较 senior 的同事讨论问题时总是有一种“原来可以这样，我怎么就没想到”的感觉，同事很棒！