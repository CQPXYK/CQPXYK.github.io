---
title: 2022.11.21 - 2022.11.25
date: 2022-11-21 14:46:22
password: 1062597351
tags:
---

> - 工作相关：慢加载分页（前后端联调、需求遗漏、SQL 排序、时区问题、POST 和 GET）
> - 博客更新：[Test Driven Development](../../../09/30/tdd/)

关于前段时间开发的新接口，在后端开发工作收尾后，这周将代码部署到dit环境，开始与前端联调并进行性能测试。

- 在前后端联调的时候，发现三个问题，一个是需求遗漏，一个排序问题，另一个是时区转换问题。
- 在做性能测试时，发现虽然分页后的接口在多处使用了存储过程，但存储过程对性能的影响不大，是可以接受的程度，做了分页的新接口将原本平均一分钟的加载时间缩短至平均五秒，最多不超过十秒。测试脚本可参见 [2022.9.5 - 2022.9.9](../../../09/06/week-2/) 中的附录

### 需求遗漏

直到这周才被前端告知真正的需求与我原本被通知的有区别，简单来讲就是后端这边需要再增加两个接口用来做目前被漏掉的逻辑。

因为这个需求并不是产品提出的上新功能，而是对现有功能的优化，所以并不会有人来事无巨细的告知我具体要做哪些东西，所以再加上我对现有业务逻辑了解的还不够熟悉和全面，所以就很容易出现遗漏。

还好原本的上线排期被往后推了一段时间，不然临时加东西真的很容易出错。当我把被漏掉的需求补上之后，又要重新做全量的集成测试。

有一种无力感在于自我感觉是可以很快完成的工作却因为各种各样的原因拖到一个多月了还没有完成，目前看起来距离完成还会有很长一段时间（前端是这周才开始开发，目前是卡在前端了），现在的我依旧觉得团队合作比单独做事效率要低。

### SQL 排序

目前数据库表中的数据类型很多是字符串类型，之前为了令排序时忽视大小写，所以在 JAVA 程序中拼 SQL 时统一给所有需要排序的列加上不区分大小写的设置（Oracle 排序时默认是大小写敏感的），拼出来的大致 SQL 如下。

```sql
... ORDER BY NLSSORT( column1 , 'NLS_SORT=BINARY_CI') DESC NULLS LAST
```

但是对于一些数字类型的列，应当转换为数字后再进行排序，具体 SQL 如下

```sql
... ORDER BY TO_NUMBER( column1 ) DESC NULLS LAST
```

注意此时如果在`TO_NUMBER`外层继续加`NLSSORT`，就会自动将其转换为字符串，`TO_NUMBER`会失效。

### 时区问题

数据库是按照 GMT 时间存储的时间戳，在 UI 页面上显示的时间会按照美国东部时间进行转换，而美国东部时间又分为夏令时和冬令时，夏令时对应 GMT-4，冬令时对应 GMT-5，夏令时和冬令时之间差了一个小时，为了避免不同地区用户查看同一个时间出现不一致的情况，UI页面统一按照 GMT-4的标准进行时区转换。

页面有个功能是可以根据用户输入的关键词来对能够模糊匹配上的时间所对应的记录进行过滤，由于需要做分页，所以必须将这个根据模糊匹配结果从而进行过滤的逻辑放在 SQL 当中，这意味着需要将数据库中的时间戳转换为字符串然后在 WHERE 条件中进行模糊匹配。

使用数据库自带的 TO_CHAR 方法在将 GMT 时间戳转换为美国东部时间的字符串时，会自动考虑到夏令时和冬令时的区别进行转换，那么这样转换得到的结果只有处于夏令时的时间能跟 UI 页面上保持一致，处于冬令时的时间会跟页面上显示的差一个小时，因此考虑写一个存储过程来区分时间戳到底处于夏令时还是冬令时，并进行时间戳到字符串的转换，令转换结果和 UI 页面上保持一致，具体代码如下（[参考](https://blog.csdn.net/weixin_39830020/article/details/116507097?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-116507097-blog-116507100.pc_relevant_3mothn_strategy_and_data_recovery&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-116507097-blog-116507100.pc_relevant_3mothn_strategy_and_data_recovery&utm_relevant_index=1)）。

```sql
FUNCTION CAL_UPDATE_ON_STR (
	update_time IN TIMESTAMP
) RETURN VARCHAR2 IS
	update_on VARCHAR2(500);
	start_day DATE;
	end_day DATE;
	loop_index INTEGER;
	
BEGIN
	start_day := TO_DATE(extract(year from update_time) || '-03-08 02:00:00','YYYY-MM-DD HH24:mi;ss');
	end_day := TO_DATE(extract(year from update_time) || '-11-01 02:00:00','YYYY-MM-DD HH24:mi;ss');
	
	FOR loop_index IN 0..6 LOOP
		IF (RTRIM(TO_CHAR(start_day + loop_index,'DAY')) = 'SUNDAY')
			THEN start_day := start_day + loop_index;
		END IF;
		IF (RTRIM(TO_CHAR(end_day + loop_index,'DAY')) = 'SUNDAY')
			THEN end_day := end_day + loop_index;
		END IF;
	END LOOP;
	
	IF update_time BETWEEN start_day AND end_day
		THEN RETURN TO_CHAR(update_time, 'YYYY-MM-DD HH24:mi;ss');
		ELSE RETURN TO_CHAR(update_time + INTERVAL '1' HOUR, 'YYYY-MM-DD HH24:mi;ss');
	END IF;
END CAL_UPDATE_ON_STR;
```

### POST 和 GET

在做性能测试时，无意中发现 POST 请求和 GET 请求的性能差别确实还挺明显的，比如通过相同的请求参数，并调用相同的请求逻辑时，发现使用 GET 请求第一次需要四五秒，跑一晚上脚本（每五分钟请求一次）算下来平均时间是零秒；而使用 POST 请求平均时间是八九秒。

（题外话：为什么要用不同的请求类型调用相同的请求逻辑呢，因为我一开始是写的 GET，并做了性能测试，但是在进一步熟悉业务逻辑后发现这里请求是一个动态变化的数据，有时会需要传一个请求体，所以又改成了 POST 并再次做了性能测试，但测试时传的请求体是一个空的 JSON，于是就这样发现了上文所述的现象）

而 GET 比 POST 快主要有两个原因

- GET 会将数据缓存起来，而 POST 不会，应该就是这个原因导致多次使用 GET 时平均时间是零，而 POST 每次请求时间都差不多
- GET 请求会在 TCP 第三次握手时发送请求头和数据，而 POST 请求在第三次握手时仅发送请求头，需要等待服务器返回100（continue）响应后再发送数据，即 POST 需要先将请求头发送给服务器并得到确认之后才会发送数据

这两个原因使得 POST 更加安全的同时，也使得 POST 变得更慢。

### 写在最后

- 发现一个有意思的地方是好几次在博客中记录下来的疑惑或者学习内容，在接下来的一个月内就在工作中实际碰到了，比如这个慢加载优化的需求，比如学习写sh脚本，比如学习写存储过程等

- 自己现在还处于一个资历浅经验少的阶段，所以这段时间总觉得现阶段应该多听听别人的建议，但这也导致了自己容易被带偏，这周就因为这种跑偏浪费了很多时间和精力，在做需求时应该考虑清楚什么事情前端做更合适，什么事情后端做更合适

- 最近在网上看到一个被安慰到的观点，原话大致如下

  > 人在不同时期的八维发展强度不同，但一个类型的内在运行机制是不变的，八维中的后四个功能当成兴趣适当培养是可以的，可以拓宽自己的能力范围，以及提供一些额外的视角，但要注意的是，后四个功能之所以是后四个，是因为在自己成长的过程中用不习惯、没有天赋或者根本不存在这个能力，发展起来比较困难，而且同一个维度的内外向功能是矛盾的，如果发展过度反而会给人造成困扰，在做决定时就会犹豫不决

  因为读研期间踩过的一些坑，导致自己在毕业时产生了比较强烈的改变意愿（大概就是在尝试发展后四个功能），毕业后紧接着开始工作、培训、进组，回想起自己那时刚开始写每周记录的时候就有提到“最近做事情总是不确定是否要这样做”