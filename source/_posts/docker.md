---
title: Docker
date: 2022-09-06 13:44:49
tags:
---

# [英文版](../../../../2023/09/01/week-53/)

# 问题与解决

**容器**：应用可以被部署并运行在 VM 上，但是考虑到 VM 需要使用过多的硬件资源来运行一整套操作系统（是对硬件的封装隔离），而容器则仅包含一些应用运行所必要的软件和配置来运行一个进程（是对进程的封装隔离），实现了比 VM 更宽松的隔离特性，更加充分的硬件资源利用以及更快速轻量的应用运行环境。

**云服务**：随着应用变得越来越庞大，流量负载越来越高，则倾向于使用一组互相独立且富余的服务器系统（云服务），而非单体服务来运行应用。

**Docker**：考虑到在每一个服务机器上都进行应用环境配置是很繁琐的工作，版本更新时还需要重新配置，而且不能跨平台。因此，考虑在发布一个项目时，将项目应用带上环境安装一起打包，即发布的 jar 包中是包含环境的，这就是现在的 Docker 镜像（image）。

# Docker

Docker 是一个可以在云平台上构建，运行以及管理容器的软件框架，采用了 `C/S` 架构，包括客户端和服务端。

Docker 守护进程 （`Daemon`）作为服务端接受来自客户端的请求，并处理这些请求（创建、运行、分发容器）一般在宿主主机后台运行，Docker 客户端则为用户提供一系列可执行命令，用户用这些命令实现跟 Docker 守护进程交互。客户端和服务端既可以运行在一个机器上，也可通过 `socket` 或者 `RESTful API` 来进行通信。

基于 Docker：

- 不再需要费力配置应用环境、交付、部署、迁移、运维、升级和扩容都变得更加快速便捷；
- 简化了容器的创建和维护，并实现了进一步的封装和共享，比如容器间独立的文件系统，网络隔离，共享的操作系统，网络互联等；
- 镜像非常小巧，只包含必要的核心环境，生成的容器运行轻便，占据较少的硬件资源，性能接近原生，秒级启动。

# 镜像与容器

镜像就是一个轻量级、可执行的独立软件包，用来打包软件的完整运行环境和基于运行环境开发的软件，它包含某个软件所需的所有内容（代码、库、环境变量和配置文件），但不包含任何动态数据，其内容在构建之后不会被改变。一般使用`<仓库名>:<标签>`的格式来指定具体是哪个软件哪个版本的镜像。

Docker镜像实际上由一层一层的文件系统组成，这种层级的文件系统称为联合文件系统（Union File System）。所有的Docker镜像都起始于一个基础镜像层（rootfs），当进行修改或者增加新的内容时，就会在当前镜像层之上，创建新的镜像层，而非修改现有镜像。每一个镜像层与它之下的镜像层都可以组成一个镜像，因此，镜像与镜像之间，可能会复用某些镜像层。

Docker镜像都是只读的，当通过镜像新建容器（container）并启动时，一个新的可写层被加载到镜像的顶部，这一层就是容器存储层（可用于存储容器运行时产生的临时数据），容器之下都叫镜像层。当开发者对容器存储层的操作结束后，可以将容器层与容器之下的镜像层一起打包发布自己定义的新镜像（但尽量不用使用这种方式来定制镜像）。

容器是一种对进程进行隔离的软件运行环境，也提供了资源控制的功能。一个应用进程对应一个容器（不建议在一个容器中运行多个应用）。容器能够让应用程序间互不干扰的独立运行，能够对进程在运行中所使用的资源进行干预。运行在容器虚拟化中的应用程序直接运行于宿主机的内核，其运行效率与真实运行在物理平台上的应用程序不相上下，远超虚拟机等其他虚拟化技术实现。

# Dockerfile

`Dockerfile`是用于自动化构建docker镜像的指令脚本，不管是开发还是运维都能通过`Dockerfile`来清晰的理解应用运行条件及环境，实现透明化的构建流程，脚本中的每一个指令都对应一层镜像，通过这一个个指令，构建出一层层镜像。

通过`docker build xxx .`构建镜像时，并非是在本地构建，而是在服务端（也就是Docker引擎）中构建的。应当将`Dockerfile`以及构建镜像所需的所有文件都置于上下文目录，构建时会通过指定的上下文路径（`.`）来将路径下的所有内容打包，上传给Docker引擎，从而使得服务端获得构建时所需的本地文件。

编写`Dockerfile`脚本指令时：

- 一般使用空目录或者项目根目录作为上下文目录，仅仅将构建镜像所以的文件放置于上下文目录中，不要引入无关文件

- 每个保留关键字（指令）都必须是大写字母

- 指令执行顺序从上到下，每个指令都会创建一个新的镜像层

- 镜像层数应尽可能少，通过`&&`将可以合并的多条指令合并为一条指令（过长的指令通过`\`换行）

- 确保每一层只添加真正需要的添加的东西，任何无关的东西都应该清理掉

  

# 容器数据卷

如果数据放在容器存储层中，那么删除容器时，数据就会丢失。因此，考虑将容器中产生的数据持久化到本地，实现本地与容器，容器与容器之间的数据同步与共享。在 Docker 中，通过挂载来进行数据共享或持久化的文件或目录，都称为容器数据卷（volume）。数据卷的生存周期独立于容器。

在挂载时，可以给该挂载指定名字（容器内路径一定要指定），指定名字的挂载称为具名挂载，没有指定名字的挂载称为匿名挂载。没有被任何容器所使用的数据卷可以删除。具名挂载可以通过名字来删除数据卷，而匿名挂载的数据卷可以看成它们与对应的容器产生了绑定，因此可以在容器删除时将它们一并删除。

可以专门用一个容器来做其他容器之间的数据共享与备份，这个容器称为数据卷容器。数据卷容器没有具体指定的应用，也不需要运行，使用它的目的就是为了定义一个或多个数据卷那并持有它们的引用。

# Docker网络

对于大部分的程序来说，它们的运行都不会是孤立的，而是要与其他程序进行交互的，网络通讯是目前最常用的一种程序间的数据交换方式。

当启动Docker服务时，它会创建一个默认的bridge网络，容器在不专门指定网络的情况都会连接到这个网络上。

Docker可以对每个容器的网络进行配置，可以在容器间建立虚拟网络来包裹多个容器并与其他网络环境隔离，让应用从宿主机操作系统的网络环境中独立出来，形成容器自由的网络设备。


# Docker Compose

一个相对完整的应用系统往往由数十个甚至上百个应用程序或微服务组成，需要多个应用（多个容器）协作完成工作。

如果说`Dockerfile`是将容器内运行环境的搭建固化下来，那么Docker Compose就可以理解为一个管理容器的工具，是将多个相关联容器的合作运行的方式和配置固化下来。

Docker Compose的配置文件是一个基于 YAML 格式的文件，version这一配置代表当前`docker-compose.yml`文件内容所采用的版本（建议使用最新的版本），services这一配置是整个`docker-compose.yml`的核心部分，其中每个service里的配置代表的是一个应用集群（多个相同镜像的容器）的配置。

Docker Compose 能够对这个集群做到黑盒效果，让其他的应用和容器无法感知它们的具体结构。

# Docker Swarm

当使用Docker Compose来定义集群时，可以通过Docker Swarm来部署和编排集群。对于Docker Swarm来说，每个Docker daemon实例都可以成为集群中的一个节点。

在任意一个Docker实例上都可以通过`docker swarm init`来初始化集群，初始化后，这个Docker实例就自动成为了集群的管理节点，而其他的Docker实例可以通过运行`docker swarm join`来加入集群，逐渐建立出用于服务开发的Docker集群。

在 `Swarm` 集群中也可以使用 `compose` 文件 （`docker-compose.yml`） 来配置、启动多个服务。

# Kubernetes

在分布式系统中，服务集群的部署、调度、伸缩一直是最为重要也最为基础的功能。除了 Docker Swarm，[Kubernetes](../../../10/03/kubernetes/) 同样可以解决这一系列容器编排问题。

K8S 是基于 Docker 的开源容器集群管理系统，目标是管理跨多个主机的容器，提供基本的部署，维护以及应用伸缩。OpenShift 是企业版的 K8S 集成式开发运维平台（集成了 K8S 以及身份验证、网络、安全、监控、日志管理和其他工具），可通过 CI/CD 来构建和部署自动化。

一个`Dockerfile`用于定义一个 Docker 镜像，一个Docker 镜像用于部署基于同一个镜像的多个容器，一个`docker-compose.yml`用于定义基于不同镜像的一系列互相合作的容器所构成的集群，即一个项目或者说业务单元，Docker Swarm/K8S 用于部署编排各种容器集群。

# CI/CD Jenkins

持续集成(Continuous integration) 是一种软件开发实践，每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误。持续部署(continuous deployment) 是通过自动化的构建、测试和部署循环来快速交付高质量的产品。

DevOps 是一个软件开发生命周期，具备 CI/CD 工作流（pipeline），加速了一个想法从开发到部署的过程，可以帮助频繁地向客户交付应用程序，并以最少的人为干预来验证软件质量。

Jenkins 是一个基于 Java 的 MIT 许可下的开源 CI/CD 工具，Jenkins 本身并不是流水线，不执行任何功能，但能够和许多不同的服务器和工具联合使用并统一安排起来，比如版本控制管理工具 [Git](../../../05/07/git/)，自动化构建工具 [Maven](../../../05/07/maven/)，网页应用服务器 Tomcat，代码测试框架、代码质量改进工具及 Docker 容器等。

开发人员可以在低版本环境通过`Dockerfile`来打包 Docker 镜像，实现持续集成，运维人员可以在高版本的线上环境通过 Docker 镜像来部署容器内运行的服务，实现持续部署。

# 入门命令

- `docker version`

  - 显示所安装docker的版本信息

- `docker info`

  - 显示docker的系统信息，包括镜像和容器的数量

- `docker system df`

  - 查看镜像、容器、数据卷所占用的空间

- `docker images`

  - 查看本地主机上的所有镜像

  - `-q`只显示镜像的Id

    

- `docker search xxx`

  - 搜索docker镜像

- `docker pull xxx`

  - 下载docker镜像

- `docker rmi xxx`

  - 删除docker镜像

- `docker image prune`

  - 删除因同名而被取消名称的旧镜像

- `docker run`

  - 本机寻找镜像，没有的话去Docker Hub上下载镜像到本地，Docker Hub上找不到的话返回错误，找到后使用本地镜像新建容器并运行

  - `-it`交互式操作终端

  - `--rm`容器退出后将其删除

  - `-d`后台运行

  - `-p`端口映射

  - `-v`数据卷挂载（必须使用绝对路径，不能使用相对路径）

  - `-e`环境配置

  - `--name`容器名字

  - `--volumes-from xxx`挂载备份`xxx`容器中的数据

  - `--expose xxx`端口暴露

  - `--net xxx`指定容器走得网络

  - `--link xxx`指定容器要连接的容器（只能连接同一网络中的其他容器）

    

- `exit`

  - 停止运行当前容器并退出

- `Ctrl + P + Q`

  - 退出当前容器但不停止运行

- `docker ps`

  - 查看正在运行的容器

  - `docker ps -a`查看所有状态的容器

    

- `docker rm xxx`

  - 删除容器，但不能删除正在运行的容器

  - `docker rm -f`删除所有容器，包括正在运行的容器

    

- `docker container prune`

  - 删除所有处于终止状态的容器

- `docker start/restart/stop/kill xxx`

  - 启动/重启/停止/强制停止容器

- `docker logs xxx`

  - 查看容器日志

- `docker top xxx`

  - 查看容器进程信息

- `docker inspect xxx`

  - 查看容器的元数据

- `docker exec -it xxx`

  - 进入当前正在运行的容器，并开启一个新的终端

  - `docker attach xxx`进入容器，不会启动新的终端

    

- `docker cp 容器id: 容器内路径 目的地`

  - 从容器内拷贝文件到主机上

- `docker volume rm xxx`

  - 删除指定名称的数据卷

  - `-v xxx`删除容器关联的数据卷

  - `docker volume prune`删除没有被容器引用的数据卷

    

- `docker network create`

  - 创建自定义的docker网络，连接不同的容器

  - 与网络相关的命令都以`docker network`开头

    

- `docker commit`

  - 提交修改后的容器，获得一个新的自定义镜像

- `docker build xxx .`

  - 根据编写的dockerfile文件构建一个镜像
  - `.`表示当前目录，这个路径用于指定上下文路径

- `docker push`

  - 发布自己构建的镜像

- `docker history xxx`

  - 查看镜像是如何生成的（会给出dockerfile中的所有指令）
  
- Dockerfile脚本中的常用指令如下

  - `FROM`指定基础镜像

  - `MAINTAINER`指定镜像的作者

  - `RUN`指定镜像构建时需要运行的命令

  - `COPY`指定需要复制到镜像中的文件

  - `ADD`指定需要向镜像中添加的文件（适合用于需要自动解压缩的场合，不需要自动解压则使用`COPY`更佳）

  - `WORKDIR`指定镜像的工作目录

  - `USER`指定执行`RUN`、`CMD`等这类命令的身份

  - `VOLUME`指定镜像挂载的宿主机目录

  - `EXPOSE`指定对外暴露的端口

  - `CMD`指定容器启动时主程序需要执行的启动命令，替换式
  
  - `ENTRYPOINT`指定容器启动时需要执行的初始化相关的命令，追加式
  
  - `ENV`指定构建镜像时需要设置的环境变量
  
  - `ARG`与`ENV`同理，但`ARG`所设置的变量在将来容器运行时是不会存在的，并且可以在构建命令`docker builder`中用`--build-arg <参数名>=<值>`来覆盖变量值
  
    

# 参考资料

- [开发者必备的Docker 实践指南- 有明- 掘金小册](https://juejin.cn/book/6844733746462064654)
- [Docker — 从入门到实践](https://yeasy.gitbook.io/docker_practice/)
- https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.337.search-card.all.click&vd_source=0b01a9c17ef5ab558386031fc8124c56
- [使用开源工具构建 DevOps 流水线的初学者指南](https://linux.cn/article-11307-1.html)