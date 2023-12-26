---
title: Maven
date: 2022-05-07 20:24:08
tags:
---

Maven是一个用于Java项目管理和构建的工具，提供了一套标准化的项目结构，标准化自动化的构建流程（编译、测试、打包、发布），以及项目依赖管理机制。

# 项目结构

一个使用Maven管理的Java项目目录结构如下：

- `pom.xml`：项目描述文件 (Project Object Model)；

- `src/main/java`：Java源码；

- `src/main/resources`：资源文件；

- `src/test/java`：测试源码；

- `src/test/resources`：测试资源文件；

- `target`：所有编译、打包生成的文件。

  

注：

- 在`pom.xml`文件中，`groupId`定义隶属的实际项目；`artifactId`定义项目中的一个模块；

- 一个Maven工程是由`groupId`，`artifactId`和`version`作为唯一标识，在引用其他第三方库的时候，也是通过这3个变量来唯一定位要依赖的jar包。

  

# 依赖管理

Maven会自动解析并判断真正需要的依赖，避免了繁琐的人工管理。Maven定义了几种依赖关系（scope）如下：

- compile：编译、运行和测试时需要用到的jar包（最常用的，直接放入classpath），打包时也要包含进去；
- test：编译UT时需要用到的jar包，不会随项目发布，比如junit；
- runtime：编译时不需要，但运行和测试时需要用到的jar包，比如mysql；
- provided：编译时需要用到，但运行时不需要（运行时由JDK或某个服务器提供），打包时不需要包含进去，比如servlet-api。

# jar包的发布与下载

- 一个jar包一经发布就无法修改，修改已发布jar包的唯一方法是发布一个新版本；
- Maven维护了一个中央仓库，所有第三方库将自身的jar以及相关信息上传至中央仓库，Maven就可以从中央仓库把所需依赖下载到本地；
- 一个jar包一旦被下载过，就会被Maven自动缓存在本地目录（用户主目录的`.m2`目录），避免重复下载；
- `version`以`-SNAPSHOT`结尾的jar包会被Maven视为开发版本，开发版本每次都会重复下载，这种开发版本只能用于内部私有的Maven repo，公开发布的版本不允许出现SNAPSHOT；

- 当中央仓库速度较慢时，可以选择一个速度较快的Maven的镜像仓库（镜像仓库定期从中央仓库同步），使用Maven镜像仓库需要在用户主目录的`.m2`目录创建一个`settings.xml`配置文件，进行配置；

- 进入到`pom.xml`所在的目录，输入命令`mvn clean package`，即可在`target`目录下获得编译后自动打包的jar包。

  

# 构建流程

- Maven有两个生命周期（lifecycle）： `default`和`clean`，每个lifecycle都由一系列阶段（phase）构成；

- mvn命令后面的参数是指phase；

- 当命令中只有一个phase时，就运行`default`这个lifecycle，并运行到指定的phase；

- 当命令中有两个phase时，就依次先后运行`clean`和`default`这两个lifecycle到分别指定的phase；

- 常用的phase按运行顺序分别是clean（清理所有生成的class和jar）、compile（编译）、test（运行测试）、package（打包）；

- 每执行一个phase，又会触发一个或多个goal，goal是最小任务单元，goal的名称是`a:b`这种形式；

- 大多数情况下只要指定phase，就默认执行这些phase默认绑定的goal，只有少数情况可以直接指定运行一个goal，例如，启动Tomcat服务器的命令是`mvn tomcat:run`；

- 执行每个phase，都是通过某个插件（plugin）来执行的，maven只是负责找到phase对应的plugin；

- maven内置了一些常用的标准插件，当标准插件无法满足要求时，还可以通过在`pom.xml`文件中声明并配置自定义的插件。

  - maven-source-plugin插件用于创建源码
  
  - maven-javadoc-plugin插件用于创建javadoc
  
  - spring-boot-maven-plugin插件用于将应用打包成一个可执行的jar包或war包（可直接使用`java -jar`命令执行）
  
    
  
  

# 模块管理

- 将一个项目拆分为多个模块可以降低软件复杂度，拆分后的项目中的每个模块都有自己的`pom.xml`和`src`，可以把每个模块当做独立的Maven项目，每个模块也可以在`pom.xml`引入其他模块；

- 当模块之间存在大量重复时，可以提取出公共的部分作为parent，parent的`<packaging>`是`pom`，module的`<packaging>`是`jar`，parent本身不含任何Java代码；

- 在根目录输入命令`mvn clean package`，即可将包括parent在内的所有module一次性全部编译打包。

  

<img src="模块结构.jpg" alt="模块结构" style="zoom:100%;" />

# 参考资料

- https://www.liaoxuefeng.com/wiki/1252599548343744/1255945359327200