---
title: Gradle
date: 2023-07-29 10:45:10
tags:
---

Gradle 和 Maven 都是可以用于 Java 项目管理和构建的工具，Gradle 侧重于灵活快速的自定义构建，Maven 侧重于简单明了的依赖管理，对于构建流程较复杂的大型的多模块项目，可以用 Gradle 来替代 Maven。

# 依赖管理

Gradle 使用`build.gradle`（基于Groovy语言）来作为项目描述文件，与`pom.xml`的日益臃肿相比，`build.gradle`中进行依赖管理和插件管理所需的配置都变得更加简短。比如`build.gradle`的依赖管理只需将`groupId`，`artifactId`和`version`的 value 都用`:`连接起来，并根据不同的`scope`来调用不同函数即可

- compile scope 对应 implementation 或者 api
- runtime scope 对应 runtimeOnly
- test scope 对应 testImplementation 或者 testRuntimeOnly
- provided scope 对应 compileOnly

# 构建流程

Maven 构建流程中所需执行的每个 phase，都是通过某个插件（plugin）来执行的，Maven只是负责找到 phase 对应的内置标准化 plugin 或者`pom.xml`文件中声明并配置自定义的 plugin，Maven 把构建流程中几乎所有的工作都放在 plugin 中完成，且每次构建都是全量构建。而 Gradle 则是在`build.gradle`中定义 task 来执行构建流程，所有的 task 构成一个有向无环图 (Directed Acyclic Graphs , DAGs)，支持更加灵活强大的自定义构建方式，是增量构建。

# Wrapper 模式

由于 Gradle 的版本迭代很快，为了能更方便管理 Gradle 自身的版本，并且让不同项目之间的 Gradle 版本隔离不会互相影响，Gradle 项目最好使用 Wrapper 模式，此时项目根目录有一个`gradlew`脚本，用来启动 Gradle Wrapper，根据`gradle-wrapper.properties`的定义，Wrapper 会下载对应的 Gradle 版本并执行构建，免去了开发人员自己去下载和配置 Gradle 环境变量的负担。使用 Wrapper之后，本地也不需要事先安装和配置 Gradle 命令，直接用项目文件中的 gradlew 命令即可。

# 参考资料

- https://gradle.org/maven-vs-gradle/
- http://www.flydean.com/gradle-kick-off/
- http://www.flydean.com/gradle-vs-maven/
- http://www.flydean.com/gradle-build-script/
- https://zhuanlan.zhihu.com/p/372585663