---
title: 2022.9.5 - 2022.9.9
date: 2022-09-06 14:07:14
password: 1062597351
tags:
---

> - 工作相关：写脚本做压测
>- 博客更新：[Docker](../../../09/06/docker/)

这周主要是在学习 [Docker](../../../09/06/docker/)，除此之外只有一个写压测脚本的小任务，所以顺带学了点 Linux 新命令（等有空再去看书系统学习）。

### 学习方面

同样的知识点，应当多看看不同的资料，获取不同人在不同角度下的不同理解，避免内容输入上的狭隘。

不同的知识点，在学习过程中应始终尝试挖掘知识点之间的联系，便于建立完备稳固的知识体系。

不应局限于看完一份资料即算任务完成的思维中，所有有价值的事情都应该是一个螺旋式的持续增长模式，而不是冲击式的通关模式。

### 工作方面

现在体会到中台和前台干活风格差异，但是现在这个阶段不应该局限于自身喜好，学都是要学的，需要补的知识点很多。

前几年的观念一直是认为上班时间应该做实际工作任务，不管是在实验室科研还是在外实习，都要有可量化的产出指标，真正的学习和反思都是工作之外自己再花时间去梳理。在这种观点下，时间安排的一直比较紧，节奏比较忙。

但是现在可以直接在上班时间去进行纯目的的学习，这种工作日常状态目前挺不习惯的，主要是会觉得忐忑，没有安全感。理论上这种不安全感会随着工作逐渐上手而逐渐减轻，希望事实如此。

### 附录

```sh
# sleep time interval
INTERVAL=5m

# url for test
url="https://xxx?id="

# id for test
ids=("xxx" "yyy" "zzz")

# number of invoking
num=0

#time spend of invoking
totalTime=0
averageTime=0

#invoke online service
while (true)
do
	for id in ${ids[@]}
	do
		echo -e "\n\n"
		start=`date "+%Y-%m-%d %H:%M:%S"`;
		curl -o /dev/null -w "\n\nTotal time taken of this invoking: %{time_total}" -k -d '{}' -H 'Content-Type: application/json' $url$id
		end=`date "+%Y-%m-%d %H:%M:%S"`;
	
		num=$((num+1))
		startTime=$(date --date="$start" +%s);
		endTime=$(date --date="$end" +%s);
		totalTime=$((totalTime + endTime - startTime));
		averageTime=$((totalTime/num));
	
		echo -e "\nNumber of invoking: "$num
		echo -e "\nTotalTime: "$totalTime
		echo -e "\nAverageTime: "$averageTime
	done
	sleep $INTERVAL
done
```

