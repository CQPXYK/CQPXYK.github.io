---
title: 2023.6.5 - 2023.6.9
date: 2023-06-06 21:24:38
password: 1062597351
tags:
---

> - 工作相关：Upstream data extracts (Optimization/Feelings/Code Review/SIT Support/Work Style/Env Deploy & Promotion)
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

关于近期所做的优化，其实没有用到什么新技巧，就还是之前总结到的一些方式，但还是觉得优化意识不够敏锐，并没有及时意识到某个逻辑处理方式不合适，应当尽快形成条件反射，避免后期频繁进行方案变动。

关于 Coding，越来越意识到业务逻辑划分很大程度上决定了 Coding 方式，即便最后实现的功能一致，但好的结构和不好的结构相比，与业务逻辑相吻合的前者明显更加优雅流畅得多，且有利于项目在后续的需求迭代中朝着良性的方向发展演进。

关于 Code Review，同事给我提的 comments 渐渐少到可以在工作记录中忽略，不确定这算不算好事，就像我不确定 release 前的 test bug free 算不算好事，谁知道到底是真的 bug free 还是 prod issue 呢。不过最近从一位 lead 那听到一句话给了我很大的安慰“好的架构一定要踩过足够多的坑”，想到我工作以来一直在踩坑，四舍五入也能算是朝着目标前进了。

关于 SIT Support，我好像总是能在 SIT 阶段靠自己发现 QA 都没有发现的隐蔽 bug，入职以来就做了两个比较大一点的需求，两个都是这样，简直怀疑我去做 QA 会更容易出业绩，一个 DEV 天天操着 QA 的心。

关于 Work Style，我性子确实比较急，但急与慢并没有好坏之分，重要的是凡事留证，不软弱也不糊弄，既自我保护也不冤枉他人。

关于 Env Promote，杂事真多！这周遇到的相关问题及解决方式如下：

- openshift 上没有规律的 pod 部署失败或者成功，看一下是不是因为繁忙导致的超时，试着把相关参数调大

- 使用 SpringBoot 的`RestTemplate` 来调用 https 服务 [参考](https://blog.csdn.net/m0_46583587/article/details/126469020)

- jenkins 部署失败，首先看是不是 sonar 没过，其次再看配置（一般不会是因为配置）

  

  
