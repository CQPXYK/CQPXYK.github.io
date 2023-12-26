---
title: Linux & Shell Scripting
date: 2023-10-14 11:29:47
tags:
---

# 1、概述

## 1.1 Linux 概述

Linux 操作系统分为四个部分

- 内核
  - 控制计算机系统的所有硬件和软件，在必要时分配硬件，根据需要执行软件，管理系统内存、软件程序、硬件设备以及文件系统
  - 较流行的选择是使用 systemd 作为初始化和进程管理系统，流行的原因在于 systemd 将事件与单元文件（定义了特定事件发生时要启动的程序）链接起来，从而根据不同的事件启动相应的进程
  - systemd 将单元文件按照 target 进行划分，target 定义了 Linux 系统的特定运行状态（比如`default.target`定义了系统启动的所有单元文件，`graphical.target`定义了多用户图形环境运行时要启动的单元文件），可以使用`systemctl`命令来启动、停止和列出系统中当前运行的单元文件
  - Linux 系统允许在无需重新编译内核的情况下将设备驱动程序插入运行中的内核，系统中安装的第一块硬盘为根驱动器，系统将硬件设备视为一种特殊文件，称为设备文件，并为每个设备文件创建一种称为“节点”的特殊文件，然后通过设备节点文件来与设备文件进行通信
  - 设备文件分为三类：1）字符设备文件，对应每次只能处理一个字符的设备，比如调制解调器；2）块设备文件，对应每次以块形式处理数据的设备，比如硬件驱动器；3）网络设备文件，对应采用数据包发送和接收数据的设备
  - Linux 内核支持通过不同类型的文件系统读写硬盘数据，还能够读写其他操作系统的文件系统，但内核必须在编译时就加入所有要用到的文件系统的支持，并要求硬件驱动器必须采用 Linux 所支持的文件系统进行格式化
  - Linux 内核采用虚拟文件系统（VFS）作为和各种其他类型文件系统交互的接口，当文件系统被挂载和使用时，VFS 会在内存中缓存相关信息，从而将计算机中所有存储设备的文件路径都纳入在名为虚拟目录的单个目录结构
  - 根驱动器包含了虚拟目录的核心，其他目录都是从这里开始构建的，Linux 虚拟目录结构只包含一个称为 root 目录的基础目录（不同于 Windows 为每个物理磁盘分区分配一个盘符，每个分区都有自己的目录结构），Linux 文件的绝对路径总是以`/`作为起始，`/`就是指 root 目录
  - 系统文件通常存储在根驱动器中，用户文件则存储在其他驱动器中，Linux 会使用根驱动器上一些特别的目录作为挂载点，并分配给额外存储设备的目录
- GNU 工具
  - 供 Linux 使用的核心 GNU 工具软件包由三部分构成：1）文件实用工具；2）文本实用工具；3）进程实用工具
  - GUN 核心工具集提供了 shell 这一特殊的交互式工具，从而为用户提供了启动程序、管理文件系统中的文件、管理运行在 Linux 系统中的进程的途径
  - Linux 系统中有很多种类的 shell，分别有不同的特性，有些适用于创建脚本，有些则适用于管理进程，默认的 shell 是 bash shell
- 图形化桌面窗口
  - Linux 有多种图形化桌面可供选择，图形化桌面包括图形化显示以及建立在图形化显示之上的桌面环境
  - 图形化显示软件是直接和 PC 显卡以及显示器打交道的底层程序，控制着 Linux 应用程序如何在计算机上呈现窗口和图形
  - 桌面环境运行用户在桌面的特定区域放置应用程序图标和文件图标，如今的桌面环境需要可观的系统资源才能正常运行
- 应用软件

完整的 Linux 系统包称为发行版，发行版通常分为完整的核心 Linux 发行版（内核、一个或多个图形化桌面环境以及预编译好的大部分可用的 Linux 应用程序）和基于核心发行版但仅包含特定部分的用于特定用途的 Linux 发行版。Linux 发行版采用 ISO 镜像文件形式发布，ISO 镜像是一个包含了完整的 DVD 镜像的文件，然后只需从 DVD 引导计算机即可安装 Linux。

## 1.2 shell 概述

shell 是一种复杂的交互式程序，其核心是命令行提示符，允许用户输入文本命令，然后解释命令并在内核中执行，或者将文本命令放入文件（称为 shell 脚本）中，构成一组命令在内核中执行（命令行最大字符数 255）。

用户输入的命令可以分为外部命令和内建命令，外部命令（也称文件系统命令，比如`ps`）并不属于 shell 程序，通常位于`/bin`、`usr/bin`、`/sbin`或`/usr/sbin`目录中，内建命令（比如`cd`、`exit`）则作为 shell 程序的组成部分存在，已经和 shell 编译成一体，无需借助外部程序文件来执行。

默认的系统 shell（`/usr/bin/sh`）是用于那些需要在系统启动时运行的系统 shell 脚本，而默认的交互式 shell（也称登录 shell）是在用户登录终端时启动的，为用户提供和系统间交互（对于 bash shell 脚本来说，默认的交互式 shell 和默认的系统 shell 这两种 shell 可能会导致问题）。

默认的交互式 shell 类型取决于`/etc/passwd`文件中用户账号的配置。大多数 Linux 系统中的`/etc/shells`文件中列出了各种已安装的 shell，这些 shell 中任意一个都可以被配置为用户的默认交互式 shell，通常被配置为是`/usr/bin/bash`，即 GUN bash shell（后续默认以 bash shell 为例介绍命令行和脚本）。

shell 提供命令行界面（CLI）来与 Unix 系统交互，在图形化桌面出现之前，CLI 是和 Unix 系统交互的唯一方式。

Linux 虚拟控制台就是一个简单的 shell CLI，是跟 Linux 系统交互的直接接口，也是唯一可以向 Linux 系统输入命令的地方。Linux 系统启动时会自动创建多个虚拟控制台，虚拟控制台是运行在 Linux 系统内存中的终端会话，在大多数 Linux 发行版中可以通过 Ctrl+Alt+功能键（F1~F7）来访问某个 Linux 虚拟控制台。Linux 虚拟控制台中是无法运行任何图形化程序的，因此输入密码时什么都不会显示，为了在访问 CLI 的同时也运行图形化程序，就需要使用终端仿真软件包，终端仿真软件包会在桌面图形化窗口中模拟控制台终端，这是在 GUI 中访问 shell CLI 的一种流行方式。

shell 程序启动后是否会自动出现 CLI 取决于用户所使用的登录方式，如果采用虚拟控制台终端登录，那么 CLI 提示符会自动出现；如果通过图形化桌面环境登录 Linux 系统，则需要启动图形化终端仿真器来访问 CLI。获得 CLI 提示符后，bash shell 会话会从用户主目录开始（在创建用户账户时系统为其分配的特有目录）。

# 2、shell 命令

## 2.1 环境变量

环境变量允许在内存中存储数据，以便 shell 中运行的程序或脚本能够访问到这些数据，在 shell 命令行和 shell 脚本中，可通过 Linux 环境变量来查看系统信息，存储临时数据和配置信息。

bash shell 中有全局变量和局部变量这两种环境变量，局部环境变量只对创建它的 shell 可见，全局环境变量则对于 shell 会话和其所有生成的子 shell 都是可见的，在子 shell 中改变全局环境变量值或者删除全局环境变量并不会影响父 shell 中的该变量，即这种改变仅在子 shell 中有效。

### 2.1.1 系统变量

Linux 系统在启动 bash 会话时就设置好了一些默认的全局系统环境变量（基本上会使用全大写字母命名，以区别于使用小写字母命名的用户自定义的环境变量），一些应当记住的全局系统环境变量如下

|                  |                                                              |
| :--------------: | ------------------------------------------------------------ |
|     环境变量     | 作用                                                         |
|       `$0`       | 在 shell 命令行中为当前 shell 的名称，在 shell 脚本中为脚本的名称 |
|     `$SHELL`     | bash shell 的完整路径名                                      |
|     `$HOME`      | 当前用户的主目录                                             |
|      `$PWD`      | 当前工作目录                                                 |
|    `$TMPDIR`     | 保存 bash shell 创建的临时文件的目录名                       |
|     `$PATH`      | shell 查找命令时使用的目录列表（以冒号分割），当用户在 CLI 中输入一个外部命令时，shell 就会从这些目录中找到对应的外部命令程序，应当确保`PATH`包含所有存放应用程序的目录 |
| `$BASH_SUBSHELL` | 当前子 shell 环境的嵌套级别，初始值为 0 则表明当前没有子 shell，值为 1 则表明当前为子 shell，值为 2 则表明当前为子 shell 的子 shell... |
|   `$BASH_ENV`    | 定义 bash 脚本运行前需要运行的启动文件                       |
|      `$IFS`      | 定义了 bash shell 用作字段分隔符的一系列字符，默认会将空格、制表符以及换行符视为字段分隔符 |
|       `$$`       | shell 会将其设为当前 PID                                     |
|       `$?`       | 最后执行的命令的退出状态码                                   |

### 2.1.2 基本操作

- 通过`printenv`或`env`命令可以查看全局变量

- 通过`printenv variable_name`或`echo $variable_name`即可查看特定环境变量`variable_name`的值（在变量名前加上`$`是为了引用变量或是让变量作为命令的参数）

- 通过`set`命令可以显示特定进程的所有环境变量，既包含局部变量、全局变量，也包含系统变量、用户自定义变量，还有局部 shell 函数

- 通过`variable_name=variable_value`语句即可创建用户自定义的局部变量`variable_name`，在变量名、等号以及变量值之间不能有空格，如果字符串`variable_value`中含有空格，则需要通过单引号或者双引号来界定该字符串的起止

- 通过`variable_name=(v1 v2 v3)`语句即可创建为环境变量`variable_name`设置多个值，`variable_name`可以作为数组使用，接着可以通过`$variable_name[index]`来访问索引`index`处的数组值，通过`$variable_name[*]`来访问整个数组

- 通过`export variable_name`即可将创建好的用户自定义的局部变量导出为全局变量，通过`export variable_name=variable_value`即可直接创建一个用户自定义的全局变量

- 通过`unset variable_name`命令即可删除已有的环境变量`variable_name`

  

### 2.1.3 启动文件

bash shell 在启动时会在启动文件（也称做环境文件）中查找命令及环境变量，因此可以通过配置启动文件中的内容，来实现环境变量的持久化定义，bash shell 进程所运行的启动文件取决于启动 bash shell 的方式

- **用户登录时作为默认交互式 shell 启动**，此时启动文件分别是`/etc/profile`这一主启动文件以及`HOME`目录下的的`.bash_profile`、`.bashrc`、`.bash_login`以及`.profile`文件，用户可以对`HOME`目录下的这些启动文件进行编辑并添加自己的环境变量，其中的环境变量会在每次启动 bash shell 会话时生效

- **生成子 shell 时作为交互式 shell 启动**，此时启动文件是`HOME`目录下的`.bashrc`文件

- **脚本运行时作为非交互式 shell 启动**，此时启动文件由环境变量`BASH_ENV`定义，即 shell 启动一个非交互式 shell 进程时，会检查`BASH_ENV`以查看需要执行的启动文件名，并执行该启动文件中的命令，这通常包括 shell 脚本变量设置

  

## 2.2 文件管理

### 2.2.1 基本操作

如果需要在系统中维护同一文件的多个副本，可以使用单个物理副本加多个虚拟副本（链接）的方法代替创建多个物理副本，链接是目录中指向文件真实位置的占位符，只能对事先已存在的原始文件创建链接，Linux 中有符号链接（也称软链接）和硬链接两种

- 符号链接是一个真实存在的文件，该文件指向存放在虚拟目录结构中某个地方的另一个文件，因此这两个以符号方式链接在一起的文件彼此的 inode 号和文件内容并不相同，符号链接可以跨越不同的文件系统
- 硬链接是一个独立的虚拟文件，包含了原始文件的信息以及位置，但彼此其实就是同一个文件，即 inode 号和文件内容完全一致，硬链接不能跨文件系统，只能对处于同一存储设备的文件创建硬链接

在 Linux 中，重命名文件也称为移动，此时 inode 号和时间戳都保持不变；删除文件也称为移除，shell 没有回收站或者垃圾箱，文件一旦被删除就是彻底删除；删除目录时只能先删除目录中的文件，再删除空目录，不能直接删除非空目录；显示文件内容时，在显示之前应该先了解文件类型，否则可能会出现各种乱码甚至将终端仿真器挂起。

Linux 包含多种文件压缩工具，gzip 是 Linux 中最流行的压缩工具，用于处理文件扩展名为`.gz`的压缩文件；Linux 中将输出写入文件以归档数据所得到的文件就是`.tar`归档文件（又称为 tarball），即便是目录也可以归档为单个`.tar`文件，因此可以很方便地在系统之间传递数据，这是在 Linux 中分发开源程序源代码文件所采用的普遍方法；经 gzip 压缩后的`.tar`文件就成为`.tgz`文件，这就是开源软件包的文件格式。

大多数 Linux 发行版能自动挂载特定类型的可移动存储设备（DVD、U 盘等），如果不支持，则需要以 root 用户身份登录或以 root 用户身份运行 sudo 来手动挂载，设备挂载到虚拟目录后，root 用户就拥有了对该设备的所有访问权限，而其他用户的访问则会被限制，移除可移动设备时，应该先确保没有进程在访问该设备，再卸载设备，最后将设备拔除。

一些应当记住的常用命令如下（写命令时可使用 Tab 键自动补全文件名或目录名，文件名或目录名也可以使用通配符来进行多选）

|                    |                                                              |                           |                                                              |
| ------------------ | ------------------------------------------------------------ | ------------------------- | ------------------------------------------------------------ |
| `ls -F`            | 区分是目录还是文件，并用`*`标识可执行文件                    | `ls -R`                   | 递归显示子目录中的内容                                       |
| `ls -a`            | 显示隐藏文件                                                 | `ls -l`                   | 以长列表格式显示关于每个文件的更多详细信息                   |
| `ls keyword`       | 以关键字作为过滤器显示符合的文件信息                         | `ls -i`                   | 查看文件或目录的 inode 号                                    |
| `touch xxx`        | 创建空文件`xxx`，若文件已存在则无影响                        | `cp s d`                  | 复制`s`文件，并以`d`命名                                     |
| `cp s d/`          | 复制`s`文件到`d`目录下，文件名不变                           | `cp s .`                  | 复制`s`到当前工作目录下，文件名不变                          |
| `cp -i s d`        | 复制时强制询问是否覆盖已存在的`d`文件                        | `cp -R s/ d/`             | 创建目录`d`，并递归复制目录`s`中的内容到目录`d`中            |
| `ln -s s slink`    | 为已存在的文件`s`创建符号链接`slink`                         | `ln s slink`              | 为已存在的文件`s`创建硬链接`slink`                           |
| `mv a b`           | 将文件`a`移动到/重命名为文件`b`                              | `mv -i a b`               | 移动/重命名时强制询问是否覆盖已存在的`b`文件                 |
| `rm xx`            | 删除/移除文件`xx`                                            | `rm -i xx`                | 强制询问是否真的要删除/移除文件`xx`                          |
| `mkdir xxx`        | 创建空目录`xxx`                                              | `mkdir -p xxx/xx`         | “批量”创建目录`xxx`和子目录`xx`                              |
| `rmdir xxx`        | 删除空目录`xxx`                                              | `rm -r xxx`               | 删除`xxx`目录中的所有的文件并删除目录本身                    |
| `file xx`          | 区分`xx`是目录还是文件，若为则探测文件的内部，判断文件类型，以及`xx`链接到哪个文件 |                           |                                                              |
| `cat xx`           | 载入整个文件后显示文本文件`xx`中所有数据，如果是大文件内容则会一闪而过 | `tac xx`                  | 载入整个文件后倒序显示文本文件`xx`中所有数据，如果是大文件内容则会一闪而过 |
| `cat -n xx`        | 显示时给所有行加上行号，包括空行                             | `cat -b xx`               | 给非空行加上行号显示                                         |
| `more xx`          | 载入整个文件后分页显示文本文件`xx`中所有数据，并在每页暂停，可利用分页工具进行翻页 | `less xx`                 | 和`more`基本一样，但支持更多高级命令选项，比如可以在完成整个文件读取之前显示文件的内容 |
| `tail xx`          | 显示文件`xx`最后几行的内容，默认是最后10行                   | `tail -n 2 xx`            | 显示文件`xx`最后两行的内容                                   |
| `tail -f xx`       | 在其他进程修改文件`xx`时实时持续显示文件内容（就得到一个实时监测系统日志） | `head xx`                 | 显示文件`xx`开头几行的内容，默认是前10行，也支持`-n`选项来指定显示行数 |
| `sort xx`          | 将文件`xx`中的数据排序，默认以字符标准排序                   | `grep pattern xx`         | 显示文件`xx`中能够匹配 pattern 的行                          |
| `gzip xx`          | 压缩指定的文件`xx`                                           | `tar -cvf xx.tar d1/ d2/` | 创建了一个名为`xx.tar`的归档文件，包含目录`d1`和`d2`的内容   |
| `tar -tf xx.tar`   | 列出（但不提取）文件`xx.tar`中的内容                         | `tar -xvf xx.tar`         | 提取文件`xx.tar`中的内容（包括重建目录结构）                 |
| `tar -zxvf xx.tgz` | 提取文件`xx.tgz`中的内容                                     | `mount -t xx d1 d2`       | 将采用`xx`文件系统类型格式化的移动存储设备`d1`挂载到目录`d2`（`umount`卸载设备） |
| `df`               | 查看所有已挂载磁盘的使用情况，默认以磁盘块为单位，`-h`选项会以更加易读的形式显示磁盘空间 | `du`                      | 显示某个特定目录（默认当前目录）的磁盘使用情况，默认以磁盘块为单位，`-h`选项会以更加易读的形式显示磁盘空间 |
| `date +%y%m%d`     | `+%y%m%d`格式会告诉`date`命令以这种格式显示当前日期          | `date -d "$date1" +%s`    | `+%s`会将任意格式的日期变量`date1`以纪元时间（1970 年 1 月 1日午夜后的整数秒）显示 |

### 2.2.2 文件权限

Linux 为每个访问系统的用户分配一个唯一的用户账户 UID，多个用户还可以组成一个具备唯一 GID 的组，并在`/etc/passwd`文件中记录了所有用户的用户登录名、用户密码（在这里被设置为`x`，真正的密码保存在`etc/shadow`文件中，只有 root 用户和特定程序才能访问）、用户账户的 UID 、用户账户的 GID、备注字段、`HOME`变量、默认交互式 shell，用户对系统重各种对象的访问权限就取决于用户登录所使用的 UID 以及 GID。

Linux 文件权限表示示意图如下

<img src="文件权限.jpg" alt="文件权限" style="zoom:80%;" />

|      |        |        |                  |
| :--: | :----: | :----: | :--------------: |
| 权限 | 二进制 | 八进制 |       描述       |
| ---  |  000   |   0    |   没有任何权限   |
| --x  |  001   |   1    |   只有执行权限   |
| -w-  |  010   |   2    |    只有写权限    |
| -wx  |  011   |   3    | 有写入和执行权限 |
| r--  |  100   |   4    |    只有读权限    |
| r-x  |  101   |   5    | 有读取和执行权限 |
| rw-  |  110   |   6    | 有读取和写入权限 |
| rwx  |  111   |   7    |    有全部权限    |



文件的全权限值是 666（属主、属组和其他用户都有读取和写入权限），目录的全权限值是 777（属主、属组和其他用户都有读取、写入和执行权限）。启动文件`/etc/profile`中会设置一个八进制`umask`值，用于屏蔽不想授予该安全级别的权限，比如`umask`被设为 022，那么接下来新建的文件和目录的默认全权限值就变为 644（666 减去 022）和 755（777 减去 022）。

- 通过`chomd 760 xx`可以将文件/目录`xx`的权限修改为 760，`-R`选项可以递归地修改子目录和文件的权限

- 通过`chomd o+r xx`可以给文件/目录`xx`的其他用户加上读取权限，其中`o`代表其他用户（`u`代表用户，`g`代表组，`a`代表所有），`+`代表增加权限（`-`代表移除权限，`=`代表设置）

- 通过`chown new_user.new_group xx`可以将修改文件`xx`的属主修改为`new_user`，属组修改为`new_group`，`-R`选项可以递归地修改子目录和文件的所属关系，只有 root 用户能够修改属主，只有同时是原属组和新属组的成员的用户可以修改属组

- 通过`chgrp`可以修改文件/目录的默认属组

  

### 2.2.3 文件系统

从一个或多个磁盘或磁盘分区创建的存储池提供了生成虚拟磁盘（又称卷）的能力，分区的范围可以是整个硬盘，也可以是部分硬盘以包含虚拟目录的一部分。对存储设备创建好分区后，需要使用某种文件系统来对分区进行格式化，以便 Linux 能够使用分区，接着将分区挂载到 Linux 虚拟目录中的某个挂载点，以便在新分区中存储数据。

直接在存储设备分区上创建文件系统的局限在于无法轻易改变文件系统的大小，但 Linux 支持逻辑卷管理，Linux 的逻辑卷管理器（LVM）可以在无须重建整个文件系统的情况下，将另一块硬盘上的分区加入已有的文件系统，动态的管理磁盘存储空间

- 物理卷（PV），指一个未使用的磁盘分区（或整个磁盘驱动器）

- 卷组（VG），由多个 PV 组成的存储池，每个 PV 只能属于单个 VG

- 逻辑卷（LV），由单个 VG 的存储空间块组成（LV 不能跨 VG，但多个 LV 可以共享单个 VG），可以使用文件系统来格式化 LV，然后将其挂载，就像普通的磁盘分区那样使用，不同之处在于 LV 的容量可以动态调整，这赋予数据存储管理极大的灵活性

  

### 2.2.4 数据重定向

输出重定向`command > outputfile`会创建文件`outputfile`（使用默认的 umask 设置），并将`command`命令的输出重定向/保存至该文件而不只是在屏幕中显示如果输出文件已经存在，则会用新数据覆盖已有的文件数据，`>>`则是将输出追加到已有文件中而不覆盖，`&>`用于将输出重定向至`/dev/null`（有去无回的黑洞），从而令命令输出变得简洁，也可以通过`cat /dev/null > outputfile`来快速清除文件`outputfile`中的现有数据，无须先删除文件再重新创建。

输入重定向`command < inputfile`会将文件`inputfile`中的内容重定向至命令`command`（比如`wc < file`会统计文件`file`中的文本，默认会统计输出文本的行数、单词数以及字节数），`command << text`是一种内联输入重定向的方式，会将接下来通过命令行次提示所输入的数据重定向到`command`命令，这个过程中，次提示符会持续显示以获取用户输入，直到输入了作为文本标记的`text`字符串，`text`一般设为 EOF，bash shell 提供了 Ctrl+D 组合键来生成 EOF 字符，从而终止输入过程。

管道连接`command1 | command2`可以直接将命令`command1`的输出作为命令`command2`的输入，数据传输不会用到任何中间文件或缓冲区，管道可以串联的命令数量没有限制，可以持续地将命令输出通过管道传给其他命令。

Linux 系统使用特殊目录`/tmp`来存放那些不需要永久保留的临时文件，系统中的任何用户都有权限读写`/tmp`目录中的文件，大多数 Linux 系统在启动时会自动删除`/tmp`目录中的所有文件。命令`mktemp`会在本地目录中创建唯一的临时文件，命令的返回值就是该临时文件名（所创建的临时文件不使用默认的`umask`值），`-t`选项会强制在`/tmp`目录中创建临时文件，此时命令会返回临时文件的完整路径名；`-d`选项会创建一个临时目录。

### 2.2.5 文件描述符

Linux 会将每个对象（包括输入和输出）当做文件来处理，并用文件描述符（非负整数）来标识会话中打开的文件对象，bash shell 保留了前 3 个特殊的文件描述符用来处理脚本的输入和输出。

- 标准输入

  - 文件描述符为 0，缩写为`STDIN`

  - 对于终端界面来说，标准输入指向键盘，shell 会从`STDIN`对应的键盘获得输入并进行处理

  - 当使用输入重定向符`<`时，标准输入改为指向指定的文件，shell 就会从文件中获得输入数据

  - `exec 0< file`允许在脚本中指定脚本所使用的`STDIN`指向文件`file`

    

- 标准输出

  - 文件描述符为 1，缩写为`STDOUT`

  - 对于终端界面来说，标准输出指向终端显示器，shell 的会将运行的程序的输出送往显示器

  - 当使用输出重定向符`>`时，标准输出改为指向指定的文件，shell 会将输出重定向到文件

  - `exec 1> file`允许在脚本中指定脚本所使用的`STDOUT`指向文件`file`

    

- 标准错误

  - 文件描述符为 2，缩写为`STDERR`，shell 对于错误消息的处理跟普通输出是分开的

  - 默认情况下，`STDOUT`和`STDERR`都指向终端显示器，当使用输出重定向符`>`或`1>`时，`STDERR`并不会改变，当使用`2>`时，标准错误才会改为指定的错误日志文件

  - 可以使用特殊的重定向符`&<`来同时将`STDOUT`和`STDERR`指向同一个文件，此时 bash shell 自动赋予`STDERR`更高的优先级，因此错误消息会比正常输出消息在文件中显示的更靠前

  - `exec 2> file`允许在脚本中指定脚本所使用的`STDERR`指向文件`file`

    

文件描述符 3\~8（后面的文件描述符可能会与 shell 内部使用的文件描述符发生冲突，故不推荐用户使用）一方面可以临时存储 0\~2 的值（`exec 3>&1`），从而在修改 0~2 之后恢复其原始值（`exec 1>&3`）；另一个方面可以作为替代性文件描述符分配给文件，从而用在脚本中的输入或输出重定向。这些替代性文件描述符一旦分配后就会一直有效，直至重新分配或关闭（`exec 3>&-`），一旦关闭了文件描述符，就不能在脚本中向其输入或输出，否则 shell 会报错。

`lsof`命令会列出整个 Linux 系统打开的所有文件描述符，这包括所有后台进程以及登录用户打开的文件，`-p`选项允许指定 PID，`-d`选项允许指定要显示的文件描述符编号（多个编号之间以逗号分隔），`-a`选项允许对`-p`和`-d`选项的结果执行`AND`运算。

`tee`命令就像是连接管道的 T 型接头，能将来自`STDIN`的数据同时送往`STDOUT`和指定的文件，默认情况下会将覆盖指定文件中的原本数据，`-a`选项允许将新数据追加到指定文件中。

## 2.3 进程管理

### 2.3.1 基本操作

`ps`命令用于检测某个特定时间点的进程信息，不带选项的`ps`命令默认只显示运行在当前终端中属于当前用户的那些进程信息，其中 S 指进程的状态，PID 指进程 ID，UID 指启动该进程的用户 ID，PPID 指父进程的 PID，NI 指进程调度优先级，C 指进程生命周期中的 CPU 利用率，STIME 指进程启动时的系统时间，TTY 指进程是从哪个终端设备启动，TIME 指运行进程的累计 CPU 时间，CMD 指启动的程序名称。

`jobs`命令用于显示后台作业信息，即当前运行在后台模式中属于用户的所有进程信息，包括作业号、作业当前状态、对应的命令/作业内容这些默认信息，且最近启动的作业会标识出来，加上`-l`选项还可以显示作业的 PID。

`top`命令用于实时显示进程信息，包括当前时间、系统的运行时长、登录的用户数、系统的平均负载（值越大则系统的负载越高）、进程概况（处于运行、休眠、停止以及僵化状态的进程数）、CPU 概况、内存概况（物理内存使用情况，交换空间状态）、以及当前处于运行状态的进程的详细列表，很适用于观察那些被频繁换入和换出内存的进程，帮助找出占用系统大量资源的进程。

`kill`命令和`pkill`命令用于向进程发出进程信号（在 Linux 中进程通过信号来通信，进程的信号是预定义好的一个消息，进程能识别该消息并决定忽略还是做出反应），并通过`-s`选项支持指定信号类型（用信号名或信号值），`kill`命令通过 PID 向进程发送信号，默认情况下发送 `TERM`信号，`pkill`命令可以使用程序名代替 PID 来终止。

|        |        |                              |
| :----: | :----: | :--------------------------: |
| 信号值 | 信号名 |             描述             |
|   1    |  HUP   |             挂起             |
|   2    |  INT   |             中断             |
|   3    |  QUIT  |           结束运行           |
|   9    |  KILL  |          无条件终止          |
|   11   |  SEGV  |            段错误            |
|   15   |  TERM  |          尽可能终止          |
|   17   |  STOP  |   无条件停止运行，但不终止   |
|   18   |  TSTP  | 停止或暂停，但继续在后台运行 |
|   19   |  CONT  | 在 STOP 或 TSTP 之后恢复执行 |

### 2.3.2 父子 shell

用户登录后所启动的默认的交互式 shell 是一个父 shell，在父 shell 中可以通过`bash`命令（或其他任意一种已安装的 shell 名称）来创建新的子 shell，接着可以通过`exit`命令来退出子 shell 并注销当前的虚拟控制台终端或终端仿真器软件，从容退出子 shell CLI。这个过程中屏幕不会显示任何相关信息，可以在生成子 shell 前后通过`ps -f`来了解相关进程信息，父 shell 中导出的全局环境变量会被复制到子 shell。创建子 shell 的成本不菲，会明显拖慢任务进度，在 shell 命令行中，子 shell 的高效用法是后台模式；而在 shell 脚本中，子 shell 则经常用于多进程处理。

- CLI 在后台模式中运行命令（在输入命令后加一个`&`）等同于创建一个子 shell 来执行该命令，此时父 shell 依旧可以继续以供他用，但终端和子 shell 与的 I/O 绑定在了一起，如果退出终端会话那么后台子 shell 也会随之退出，这并非真正的多进程处理

- 如果将进程列表以后台模式运行（`(command1 ; command2)&`或者`coproc job_name { command1; command2 }`）则可以在子 shell 中进行大量的多进程处理，此时终端不再和子 shell 的 I/O 绑定在一起

- 此外，每当执行外部命令时，就会创建一个拥有全新环境的子进程，这种操作称为衍生（forking），需要耗费时间和资源来设置新子进程的环境，因此外部命令的系统开销较高；而使用内建命令时，不需要衍生子进程，因此内建命令的系统开销较低，速度更快，效率更高

  

在 Linux 系统中，由 shell 启动的所有进程的调度优先级默认都是相同的，调度优先级是一个整数值，取值范围从 -20（最高优先级）到 +19（最低优先级），bash shell 以优先级 0 来启动所有进程。

- `nice`命令允许在启动命令时设置其调度优先级，`-n`选项用于指定新的优先级，普通用户可以降低优先级，但只有 root 用户或特权用户才能提高优先级

- `renice`命令允许对已运行的命令重置其调度优先级，普通用户只能对属主为自己的进程使用`renice`且只能降低优先级，root 用户和特权用户可以使用`renice`对任意进程的优先级做任意调整

  

### 2.3.3 信号控制

bash shell 会话会处理收到的所有挂起信号（信号值为 1）和中断信号（信号值为 2）。

- 收到挂起信号时，bash shell 会退出，并在退出之前将挂起信号传给所有由该 shell 启动的进程，包括正在运行的 shell 脚本，如果 shell 会话中有已停止的进程作业，那么在退出 shell 时，bash 会发出提醒（但如果是以后台模式运行的进程，则不一定会有此提醒）

- 收到中断信号时（Ctrl+C 组合键会生成中断信号），bash shell 会被中断，Linux 不再为 shell 分配 CPU 时间片，shell 会将中断信号传给所有由该 shell 启动的进程

- 收到停止信号时（Ctrl+Z 组合键会生成停止信号），bash shell 会通知进程已经被停止了，但会让程序继续驻留在内存中（作业还能从上次停止的位置继续运行）

- `bg`命令会以后台模式重启被停止的作业，`fg`命令会以前台模式重启被停止的作业

  

shell 脚本的默认行为是忽略这些信号，完全由 bash shell 会话进行处理，但是可以在 shell 脚本中加入识别信号及处理的代码，当指定信号出现时，脚本会捕获该信号，此时无须 shell 会话处理该信号，而是由本地脚本自行处理（即执行指定命令）

- 在脚本中通过`trap commands signals`命令可以在信号`signals`（多个信号之间以空格分隔）出现时将其捕获并执行指定的`commands`命令

- 针对已设置的信号捕获，可以继续用`trap -- signals`来移除已设置好的信号捕获

- 在交互式 shell 会话中使用`trap -p`可以查看被捕获的信号，如果无显示，则说明没有捕获信号，shell 会话会按照默认方式处理信号，shell 脚本会按照默认行为忽略信号

  

通过`./xxx.sh &`命令以后台模式运行的`xxx.sh`子 shell 不和终端父 shell 会话的`STDIN`、`STDOUT`以及`STDERR`关联，但子 shell 的输出仍然会使用终端显示器来显示`STDOUT`和`STDERR`消息（最好是将后台脚本的`STDOUT`和`STDERR`进行重定向），如果终端收到挂起信号退出那么后台子 shell 也会随之退出

- `nohup ./xxx.sh &`命令能够阻断发给特定进程（以后台模式运行的`xxx.sh`脚本）的挂起信号，那么当退出终端 shell 时，该特定进程并不会收到挂起信号，不会退出

- 由于`nohup`会解除终端父 shell 和子 shell 进程之间的关联，子 shell 的输出不再使用终端显示器来显示`STDOUT`和`STDERR`消息，而是将`STDOUT`和`STDERR`重定向到一个名为`nohup.out`的文件（位于当前目录或者`HOME`目录）中

  

### 2.3.4 定时调度

`at`命令会将作业提交到队列中，并指定 shell 何时运行该作业，`-f`选项可以指定使用哪个脚本文件，`-q`选项可以指定作业的优先级队列，`-M`选项可以禁止作业产生输出信息

- 针对不同优先级，有 52 种作业队列（用`a~z`和`A~Z`来指代），默认情况下作业会被放入`a`队列

- Linux 会将提交该作业的用户 email 地址作为`STDOUT`和`STDERR`，任何送往`STDOUT`和`STDERR`的输出都会通过邮件系统传给该用户，如果系统没有安装`sendmail`，那就无法获得任何输出，因此最好对`STDOUT`和`STDERR`进行重定向

- `at`的守护进程`atd`在后台运行，`atd`默认情况下每隔 60 秒会检查一次系统的特殊目录`/var/spool/at`或`/var/spool/cron/atjobs`，从中获取`at`命令提交的作业，查看作业的运行时间，如果时间和当前时间一致则运行此作业，如果时间已经过去则在第二天的同一时间运行此作业

- `atq`命令可以查看系统中有哪些作业在等待，`atrm`命令可以删除等待中的作业

  

Linux 系统使用 cron 程序调度需要定期反复执行的作业（前提是 Linux 系统是不间断运行的），cron 在后台运行，会检查一个特殊的表（cron 时间表），从中获知已安排执行的作业，cron 不会运行已经过时的错过的作业。

anacron 程序同样用于调度需要定期反复执行的作业，但如果判断出某个作业错过了预设的运行时间，则会尽快运行该作业，那么即使 Linux 有过关闭重启，也能确保作业一定能运行。

# 3、shell 脚本

## 3.1 基本操作

### 3.1.1 脚本构建

在创建 shell 脚本时，必须在文件的第一行指定要使用的 shell 类型，比如`#!/bin/bash`指定要使用的 bash shell（除了`#!`开头的第一行以外，shell 不会解释以`#`开头的注释行），写在同一行的命令需要用分号隔开，写在独立行的命令可以不加分号，此外，由于 umash 变量设置可能会导致新文件的默认权限不包含执行权限，那么可以通过`chomd `命令进行授权，然后通过`./filename`来执行脚本。

- 用户自定义变量的名称可以是任何由字母、数字或下划线组成的字符串，长度不能超过 20 个字符，区分大小写

- 使用等号为变量赋值，在变量、等号和值之间不能出现空格，或者在双括号`(( statement ))`中进行赋值（此时可以出现空格）

- 对变量赋值时不加`$`，引用变量时要加`$`，脚本中只要使用`$x`，就会被替换为变量`x`的值，如果需要显示`$x`文本，则应该使用`\$x`

- shell 脚本会以字符串的形式存储所有的变量值，脚本中定义的变量在脚本的整个生命周期里会一直保存，在脚本结束时会被删除

- 命令替换通过反引号 `` 或者 $() 来将命令输出赋给变量，这一过程会创建出一个独立的子 shell 来运行指定命令，在这个子 shell 中运行的命令无法使用脚本中的变量

  

### 3.1.2 数学运算

bash shell 使用`expr operation`或`$[operation]`来进行整数数学运算（`${variable}`用于引用名为`variable`的变量），通过内建的 bash 计算器 bc 来进行浮点数运算。在命令行中可以通过`bc`命令进入 bc 计算器（`bc -q`则不显示冗长的欢迎信息），通过内建变量`scale`来控制保留小数位数，通过`quit`命令退出。在 shell 脚本中，最好是使用命令替换配合内联输入重定向，将计算输出赋给一个变量。

```bash
variable=$(bc << EOF
scale=4
var1=1
var2=2
$var1 / $var2
EOF
)
```

### 3.1.3 脚本退出

Linux 提供了专门的变量`?`来保存最后一个已执行命令的退出状态码，也可以通过将`exit num`作为脚本结束命令从而将退出状态码设置为`num`（介于 0~255）。

|            |                            |            |                                |
| :--------: | :------------------------: | :--------: | :----------------------------: |
| 退出状态码 |            描述            | 退出状态码 |              描述              |
|     0      |        命令成功结束        |    128     |         无效的退出参数         |
|     1      |       一般性未知错误       |   128+x    | 与 Linux 信号`x`相关的严重错误 |
|     2      |    不适合的 shell 命令     |    130     |     通过 Ctrl+C 终止的命令     |
|    126     | 用户没有权限，命令无法执行 |    255     |    正常范围之外的退出状态码    |
|    127     |         没找到命令         |            |                                |

### 3.1.4 脚本函数

函数是一个脚本代码块，可以为其命名并在脚本中的任何位置调用它

- 需要保证函数名是唯一的，在函数调用之前定义好函数，以及在调用函数时参数和函数名放在同一行

- 在函数内部，`$0`对应函数名，函数参数依次保存在`$1`、`$2`...等变量中，`$#`对应函数参数的个数

- 默认情况下，在脚本中定义的变量是全局变量，在脚本内任何地方（包括函数内部）都有效，函数内对全局变量的修改在函数外也是生效的（这个行为很危险），如果在函数中定义了全局变量，那么在函数外也可以访问

- 函数内部应当避免使用全局变量，在函数中定义的变量应当用`local`关键字声明，这样的变量是局部变量，即便在函数外有同名变量时 shell 也会保持变量间互不干扰，对脚本的其他部分而言，在函数中定义的局部变量是无效的

- 函数运行结束时会返回一个退出状态码（可以使用`$?`来查看），默认情况下为函数中最后一个命令返回的退出状态码，也可以在函数的最后通过`return`命令来返回一个指定的退出状态码

- 函数的最后通过`echo $variable`来返回变量值（区分`return`返回的是退出状态码）

- bash shell 允许创建函数库文件，然后在多个脚本中通过点号操作符`. ./funcs`引用当前目录下的库文件`funcs`，shell 并不运行库文件，但会使这些函数在运行该脚本的 shell 中生效

  

在 shell 命令行中也可以定义函数，接着就可以在整个系统的任意目录以及子 shell 中使用该函数，无须担心该函数是否位于`PATH`环境变量中，但应确保函数名不会和已有命令相同，否则新定义的函数会覆盖已有命令。一旦退出 shell，所定义的函数也会消失，因此，可以在`HOME`目录下的`.bashrc`启动文件中定义函数，那么每次启动交互式 shell 时都可以使用这些函数。

## 3.2 条件判断

bash shell 的`if`语句会运行`if`之后的命令，如果该命令的退出状态码为 0 则执行`then`之后的命令，非 0 则执行`else`或`elfi`之后的命令，`fi`标志`if-then`语句到此结束。

### 3.2.1 单方括号

`if`语句无法进行状态码之外的条件测试，因此可以使用`test condition`或`[ condition ]`（以及`[ condition1 ] && [ condition2 ]`、`[ condition1 ] || [ condition2 ]`）命令来测试某个条件，当条件满足时返回状态码 0，不满足时返回非 0。一些常用的简单`condition`如下所示

|                   |                                    |                   |                                           |
| :---------------: | :--------------------------------: | :---------------: | :---------------------------------------: |
|   **数值比较**    |                                    |                   |                                           |
|    `n1 -eq n2`    |        检查`n1`是否等于`n2`        |    `n1 -ne n2`    |          检查`n1`是否不等于`n2`           |
|    `n1 -gt n2`    |        检查`n1`是否大于`n2`        |    `n1 -ge n2`    |        检查`n1`是否大于或等于`n2`         |
|    `n1 -lt n2`    |        检查`n1`是否小于`n2`        |    `n1 -le n2`    |        检查`n1`是否小于或等于`n2`         |
|  **字符串比较**   |                                    |                   | （使用字符的 Unicode 编码值进行大小比较） |
|   `str1 = str2`   |     检查`str1`是否和`str2`相同     |  `str1 != str2`   |        检查`str1`是否和`str2`不同         |
|  `str1 \< str2`   |      检查`str1`是否小于`str2`      |  `str1 \> str2`   |         检查`str1`是否大于`str2`          |
|     `-n str1`     |      检查`str1`长度是否不为 0      |     `-z str1`     |          检查`str1`长度是否为 0           |
|   **文件比较**    |                                    |                   |                                           |
|     `-e file`     |         检查`file`是否存在         |     `-s file`     |         检查`file`是否存在且非空          |
|     `-f file`     |     检查`file`是否存在且为文件     |     `-d file`     |        检查`file`是否存在且为目录         |
|     `-r file`     |      检查`file`是否存在且可读      |     `-w file`     |         检查`file`是否存在且可写          |
|     `-x file`     |     检查`file`是否存在且可执行     |                   |                                           |
|     `-O file`     | 检查`file`是否存在且属当前用户所有 |     `-G file`     | 检查`file`是否存在且默认组与当前用户相同  |
| `file1 -nt file2` |     检查`file1`是否比`file2`新     | `file1 -ot file2` |        检查`file1`是否比`file2`旧         |

### 3.2.2 单括号

使用单括号`(command)`作为`if`之后的命令，会使用子 shell 来执行`command`命令，子 shell 的退出码取决于单括号中最后一个命令的退出码。

### 3.2.3 双括号

使用双括号命令`(( expression ))`作为`if`之后的命令，在比较过程中进行一些高级数学表达式运算，双括号中的表达式无需使用`\`转义（双括号命令也可以在脚本中的普通命令里用来赋值）。

|         |             |         |            |
| :-----: | :---------: | :-----: | :--------: |
|  符号   |    描述     |  符号   |    描述    |
| `val++` |    后增     | `val--` |    后减    |
| `++val` |    先增     | `--val` |    先减    |
|   `!`   |  逻辑求反   |   `~`   |   位求反   |
|  `**`   |   幂运算    |         |            |
|  `<<`   |   左位移    |  `>>`   |   右位移   |
|   `&`   | 位布尔`AND` |  `\|`   | 位布尔`OR` |
|  `&&`   |  逻辑`AND`  | `\|\|`  |  逻辑`OR`  |

### 3.2.4 双方括号

使用双方括号命令`[[ expression ]]`作为`if`之后的命令，除了可以进行一些基本条件判断以外，还可以对字符串进行模式匹配（不是所有的 shell 都支持双方括号）。

## 3.3 用户输入

bash shell 提供了一些方法从用户处获取数据来作为脚本运行时变量。

### 3.3.1 命令行参数

- 命令行参数是在命令/脚本之后出现的各个单词，bash shell 会将所有的命令行参数都指派给称作位置参数的特殊变量

- 位置参数是用于保存命令行参数的变量，位置参数的名称都是标准数字，`$0`对应脚本路径名（可以通过`basename $0`获取不包含路径的脚本名），`$1`对应第一个命令行参数...

- `$#`对应命令行参数的个数，`${!#}`对应最后一个参数，`$*`和`$@`都包含了所有命令行参数，区别在于`"$*"`会被扩展为`"$1c$2..."`（其中`c`为`IFS`变量值的第一个字符分隔），`"$@"`会被扩展为`"$1""$2"...``

- `shift`命令会将除了`$0`以外的每个命令行参数都向左移动一个位置，即`$1`的值会被删除，`$2`的值会移入`$1`，`$3`的值会移入`$2`...，可以通过给`shift`命令提供参数来指定要移动的步数

  

### 3.3.2 命令行选项

- 选项是在连字符之后出现的单个字母，长选项是在双连字符之后紧跟着的一个字符串，选项可以理解为一种以`-`或`--`起始的特殊形式的命令行参数

- 当命令行参数和选项混合使用时，Linux 通过`--`这一特殊字符来将两者分开，`--`表明命令行选项部分结束，后面是普通命令行参数

- 当多个选项被合并使用时，可以通过`getopt`命令进行解析，并通过`set`命令将原始命令行参数替换为解析后的格式化的命令行参数，最后就可以用常规的方式来解析非合并使用的命令行选项

- `getopt`和`set`命令配合使用方式为`set -- $(getopt -q optstring "$@")`，其中`optstring`定义了有效的命令行选项字母以及哪些选项需要参数值，`-q`选项用于在`optstring`定义不完整（缺乏用户实际输入的选项）时忽略错误消息显示

- `getopts`命令可以直接在循环中依次解析单个命令行选项，无须像`getopt`这样先格式化再解析，也可以解析`getopt`解析不了的带引号和空格的参数

- `getopts`命令使用方式为`while getopts :optstring opt`（`:`是为了忽略错误消息显示），对于每个正在被解析的选项变量`opt`，环境变量`OPTAGR`会保存该选项的参数值，`OPTIND`会保存该选项在整个参数列表中所处的索引位置

- 命令行选项解析完成后，变量`OPTIND`配合`shift`命令（`shift $[ $OPTIND - 1 ]`）即可解析剩下的命令行参数

  

常用的标准化选项含义如下

|      |                          |      |                              |
| :--: | :----------------------: | :--: | :--------------------------: |
| 选项 |           描述           | 选项 |             描述             |
| `-a` |       显示所有对象       | `-c` |           生成计数           |
| `-d` |         指定目录         | `-e` |           扩展对象           |
| `-f` |    指定读入数据的文件    | `-h` |      显示命令的帮助信息      |
| `-i` |      忽略文本大小写      | `-l` |        产生长格式输出        |
| `-n` | 使用非交互模式（批处理） | `-o` | 将所有输出重定向至指定的文件 |
| `-q` |      以静默模式运行      | `-r` |      递归处理目录和文件      |
| `-s` |      以静默模式运行      | `-v` |         生成详细输出         |
| `-x` |       排除某个对象       | `-y` |      对所有问题回答 yes      |

### 3.3.3 运行时输入

- bash shell 提供`read`命令来在脚本运行时询问用户并等待用户回答

- `read`命令会将提示符后输入的所有数据分配给所指定的单个变量，如果不指定变量则会将接收到的所有数据都放进特殊环境变量`REPLY`中

- `-p`选项可以用来指定多个变量，此时输入的每个数据值都会分配给指定的变量列表中的下一个变量，如果变量数据不够，那么剩下的数据就全部分配给最后一个变量

- `-t`选项可以用来指定等待用户输入的秒数，一旦超时`read`命令会返回非 0 退出状态码

- `-n`选项可以用来指定等待用户输入的字符个数，只要用户输入达到指定字符数`read`命令就会将其传给变量，无需按 Enter 键

- `-s`选项可以避免在`read`命令中输入的数据出现在屏幕上（其实是将文本颜色设成了跟背景色一样）

- 通过`cat`和管道可以将文件数据作为`read`命令的输入（`cat file | while read line`）

  

# 4、文本处理

## 4.1 正则表达式

正则表达式是一种可供 Linux 工具对数据进行模式匹配从而过滤文本的自定义模板，正则表达式使用元字符来描述数据流中的一个或多个具体内容不确定的字符数据，然后通过正则表达式引擎（一种底层软件）来负责解释正则表达式并用这些模式进行文本匹配。在 Linux 中，不同的应用程序可能使用不同类型的正则表达式引擎，最常用是 POSIX 基础正则表达式（BRE）引擎和 POSIX 扩展正则表达式（ERE）引擎，大多数 Linux 工具至少符合 BRE 引擎规范。

- 正则表达式区分大小写，能识别的特殊字符为`.*[]^${}\+?|()`，如果需要将某个特殊字符以及`/`视为普通字符，则必须使用`\`进行转义

- 默认情况下正则表达式可以匹配数据流中的任何地方，位于正则表达式开头的脱字符`^`可以指定位于数据流中文本行行首的模式匹配，即如果模式出现在非行首的位置是无法匹配的，如果`^`放到正则表达式开头之外的位置，那么就是没有特殊含义的普通字符

- 脱字符`^`用于锚定行首时，无需`\`转义，`^`作为普通字符进行匹配且后面没有其他文本时，无需`\`转义，`^`做普通字符且和后面的文本一起进行匹配时，需要`\`转义

- 位于正则表达式末尾的`$`指定位于数据流文本行行尾的模式匹配，即如果模式出现在非行尾的位置是无法匹配的，直接将行首锚点和行尾锚点组合在一起可以过滤出空白行

- 点号字符`.`可以匹配除换行符之外的任意单个字符，但必须匹配一个字符，如果该位置没有可匹配的字符则模式不匹配

- `[]`用于定义限定匹配范围的字符组，在`[]`中可以使用`0-3`来代替`0123`，使用`0-35-7`来代替`0123567`，`[^]`用于定义限定匹配范围以外的字符组，除了这些自定义字符组，还有一些特殊字符组

  |               |                                    |               |                                                              |
  | :-----------: | :--------------------------------: | :-----------: | :----------------------------------------------------------: |
  |    字符组     |                描述                |    字符组     |                             描述                             |
  | `[[:alpha:]]` |   匹配任意字母字符，不区分大小写   | `[[:print:]]` |                      匹配任意可打印字符                      |
  | `[[:alnum:]]` | 匹配任意字母数字字符，不区分大小写 | `[[:punct:]]` |                         匹配标点符号                         |
  | `[[:blank:]]` |          匹配空格或制表符          | `[[:space:]]` | 匹配任意空白字符：空格、制表符、换行符、分页符、垂直制表符和回车符 |
  | `[[:digit:]]` |  匹配 0~9 中的数字，等价于`[0-9]`  | `[[:upper:]]` |                     匹配任意大写字母字符                     |
  | `[[:lower:]]` |        匹配任意小写字母字符        |               |                                                              |

- 星号`*`表明前面的字符必须出现 0 次或多次，问号`?`表明前面的字符可以出现 0 次或 1 次，`+`表明前面的字符必须出现 1 次或多次

- 花括号`{}`允许为前面的字符指定可重复出现次数，`{1}`代表出现 1 次，`{2,3}`代表至少出现 2 次，至多出现 3 次

- 竖线`|`允许以逻辑`OR`方式指定正则表达式引擎要使用的两个或多个模式，正则表达式和`|`之间不能有空格

- 圆括号`()`可以对正则表达式分组，每一组会被视为一个整体（可以将其看作是一个字符），然后对该组应用特殊字符

  

## 4.2 vim 编辑器

vi 编辑器使用控制台图形模式来模拟文本编辑窗口，复杂但功能强大，开源后被改进为 vim，通过`vim`命令和要编辑的文件名就可以启动 vim 编辑器。vim 使用全屏模式来将整个控制台窗口作为编辑器区域，在内存缓冲区中处理数据，如果在启动 vim 时未指定文件名或者指定文件不存在，则会开辟一段新的缓冲区进行编辑，如果指定的是已有文件的名称，则会将文件的整个内容都读入缓冲区以备编辑。

刚打开要编辑的文件或新建文件时，vim 编辑器会进入命令模式（又称普通模式）

- 命令模式下按下 i 键后进入插入模式，vim 会将当前光标位置输入的字符、数字或符号都放入缓冲区，按下 Esc 键退出插入模式并返回命令模式
- 命令模式下按下 : 键后进入 Ex 模式（提供了交互式命令行），此时光标会移动到屏幕底部的消息行处，出现冒号，接着可以输入命令来控制 vim 的操作
- 命令模式下按下 v 键，进入可视模式，接下来可以移动光标来覆盖想要复制的文本，vim 会高亮显示复制区域的文本，按下 y 键完成复制，并回到命令模式
- 命令模式下按下 / 键，光标会去到屏幕底部的消息行，并显示出一个正斜线，接着就可以输入需要查找的文本内容，按下 Enter 键，光标会定位到正确位置或者输出没有找到的错误信息，若需要继续查找则按 / 键，再按 Enter 键或 n 键

|                                               |                                                              |                                                              |
| --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 命令模式                                      | 命令模式                                                     | Ex 模式                                                      |
| `h`：光标左移一个字符                         | PageDn/Ctrl+F：下翻一屏                                      | `q`：如果未修改缓冲区数据则退出                              |
| `j`：光标下移一个行                           | PageUp/Ctrl+B：上翻一屏                                      | `q!`：放弃对缓冲区数据的所有修改并退出                       |
| `k`：光标上移一行                             | `G`：移到缓冲区的最后一行                                    | `w filename`：将文件另存为其他名称                           |
| `l`/Backspace：光标右移一个字符               | `num G`：移到缓冲区的第`num`行                               | `wq`：将缓冲区数据保存到文件中并退出                         |
| 方向键：在文本区域移动光标                    | `gg`：移到缓冲区的第一行                                     | `:s/old/new/g`：替换当前行内出现的所有`old`                  |
| `x`/delete：删除光标当前所在位置的字符        | `u`：撤销上一个编辑命令                                      | `:n, ms/old/new/g`：替换第`n`行和第`m`之间出现的所有`old`    |
| `dd`：删除光标当前所在行                      | `a`：在光标当前位置后追加数据                                | `:%s/old/new/g`：替换整个文件中出现的所有`old`               |
| `dw`：删除光标当前所在位置的单词              | `A`：在光标当前位置所在行结尾追加数据                        | `:%s/old/new/gc`：替换整个文件中出现的所有`old`，并在每次替换时提示 |
| `d$`：删除光标当前所在位置至行尾的内容        | `r char`：用`char`替换光标当前所在位置的单个字符             |                                                              |
| `J`：删除光标当前所在行结尾的换行符（合并行） | `R text`：用`text`覆盖光标当前所在位置的内容                 |                                                              |
| `yw`：复制一个单词                            | `p`：粘贴上一次被删除/复制的文本，可以与任何删除/复制文本的命令搭配使用 |                                                              |
| `y$`：复制到行尾                              |                                                              |                                                              |

## 4.3 sed 编辑器

### 4.3.1 sed 概述

不同于 vim 这样通过命令或鼠标单击来处理文本的交互式编辑器，有时候需要一种可以自动格式化、插入、修改或删除文本元素的命令行编辑器。sed 编辑器被称作流编辑器，可以根据命令行中输入的或脚本中预先定义好的一组规则编辑数据流，支持 POSIX BRE 正则表达式引擎

- 默认情况下，sed 编辑器会将指定的命令（用单引号`''`定义）应用于`STDIN`输入流或者指定数据文件中的数据流（比如`sed 's/a/b/'`使用了替换命令`s`，代表使用`b`替换文本中的`a`）

- 如果需要应用多个命令，则要么使用`-e`选项在命令行中指定并用分号分隔（比如`sed -e 's/a/b/; s/c/d/' xx.text`对`xx.text`文件中的数据使用了两次替换命令），要么使用`-f`选项在单独的sed 脚本文件中指定

- 这种 sed 脚本文件区别于 bash shell 脚本文件，一般使用`.sed`作为 sed 脚本文件的扩展名

- 每当匹配一行数据并针对该行执行所有命令之后，就会继续读取下一行数据并重复这个过程直到处理完数据流中的所有行

- sed 编辑其并不会修改原始数据，而是将修改后的新数据输出到`STDOUT`，使用`-n`选项则不产生命令输出

- `-s`选项可以告知 sed 将指定目录内的各个文件作为单独的流，这样就可以分别处理目录中的每个文件，可以配合`1F`命令一起使用，`F`命令用于打印当前正在处理的文件名（不受`-n`选项的影响），`F`前的`1`代表只需显示一次文件名，否则所处理的每个文件的每一行都会显示一次文件名

  

### 4.3.2 行寻址

默认情况下，sed 命令会应用于所有的文本行，可以通过行寻址来将命令只应用于特定的某一行或某些行

- 基于行号的行寻址可以用行号来引用文本流中的特定行，比如`sed '2s/a/b/'`只对第 2 行的文本进行替换，`sed '2,4s/a/b/'`只对第 2~4 行的文本进行替换，`sed '2$s/a/b/'`只对第 2 行开始到结尾的文本进行替换

- 基本文本模式匹配的行寻址通过正则表达式来指定需要匹配的行，且必须将匹配放入正斜线内，比如`sed '/pattern/s/a/b/'`只对能够匹配`pattern`的行进行替换

- 如果需要对单行执行多条命令，可以用花括号将其组合在一起，每种行寻址都可以对应一个花括号组合到一起的命令块

- 命令之前加上`!`就成为排除命令，旨在指示命令不应用于行寻址所指定的行，而是应用于行寻址以外的所有行

- 在行寻址后加上`b`和`label`参数就可以定义指定行需要跳转执行的`label`处的指定命令，如果不指定`label`，那么指定行就无须指定命令脚本

  

### 4.3.3  sed 命令

替换命令`s`默认情况下只替换每行中出现的第一处匹配文本，可以通过设置替换标志`flags`（`s/pattern/replacement/flags`）来进一步指定替换方式

- `flag`为数字时，指明新文本将替换行中的第几处匹配

- `flag`为`g`时，指明新文本将替换行中所有的匹配

- `flag`为`p`时，指明打印出替换后的行，一般与`-n`选项配合使用，此时只输出修改后的行

- `flag`为`w file`时，指明将替换的结果写入文件，同样与`-n`选项配合使用，此时只在文件中保存修改后的行

  

删除命令`d`用于删除文本行

- 使用行寻址时只删除指定的所有行，没有使用行寻址时会删除所有文本行

- 可以使用两个基于文本模式的行寻址来删除某个区间内的行（包括被指定的行）

- 由于 sed 编辑其并不会修改原始数据，被删除的行仅仅只是从输出中消失了

  

插入命令`i`会在指定行前增加一行，附件命令`a`会在指定行后增加一行

- 必须使用行寻址指定数据被插入/附加在什么位置，

- 只能指定一个行地址（基本行号或文本模式匹配都可），不能使用行区间

- 必须在要插入/附加的每行新文本末尾使用反斜线

  

修改命令`c`允许修改整行文本的内容

- 必须使用行寻址指定要修改的数据行

- 可以指定行区间，但此时会将区间内的多行修改为指定的一行文本数据

  

转换命令`y`是唯一可以处理单个字符的 sed 编辑器命令，会对指定的`inchars`和`outchars`进行一对一的映射和替换

- `inchars`中的第 n 个字符会被`outchars`中的第 n 个字符替换

- 这个映射替换过程会一直持续到处理`inchars`中存在的所有字符

- `inchars`和`outchars`的长度必须相同，否则 sed 编辑器会产生错误消息

  

打印命令`p`用于打印 sed 编辑器输出中的文本行，打印命令`=`用于打印行号，列出命令`l`用于列出行

- 和`-n`选项（抑制其他行的输出）配合使用就能只打印指定的行

- 可以在替换或修改命令做出改动之前先用打印命令查看相应的行

- 列出命令可以打印数据流中的文本和不可打印字符（比如`\t`表示制表符，`$`表示换行）

  

写入命令`w`用来向数据文件写入行

- 写入命令要求运行 sed 编辑器的用户拥有该文件的写权限

- 可以行寻址指定单个行或者行区间

  

读取命令`r`允许将一条独立文件中的数据插入数据流

- 一般与插入/附加/修改命令配合使用，将一条独立文件中的数据插入/附加/修改到数据流

- 读取命令中无法使用行区间，只能通过行寻址指定确定的行，并将数据文件的所有内容都插入数据流的指定地址之后

  

### 4.3.4 多行命令

上一节介绍的 sed 命令都是单行版本命令，有时候需要对跨多行的数据执行特定的操作，sed 编辑器提供了三个可用于处理多行文本的特殊命令。

`N`命令

- 通常 sed 编辑器只有每次在当前行中执行完所有定义好的命令之后会自动的移动到数据流中的下一行并再次重头开始执行命令

- 单行`n`命令会将数据流中的下一行移入 sed 编辑器的模式空间（上一行已经被移出模式空间），而多行`N`命令则是将下一行添加到模式空间中已有文本之后，文本行之间仍然用换行符分隔，将换行符分隔的两行当做一行放入模式空间中合并处理

- 单行`n`命令和多行`N`命令都是位于命令列表中的定义好的令编辑器移动到下一行的 sed 命令，都不会不影响 sed 编辑器每次执行完命令列表后再继续自动换行的一系列行为

- 如果 sed 编辑器在处理最后一行时（此时没有下一行），那么在执行`N`命令时就会叫停 sed 编辑器，即命令列表中位于`N`命令后面的命令就不会再执行了

  

`D`命令

- `N`命令使得 sed 编辑器将换行符分隔的当前行和下一行当做一行来一起合并处理，此时单行`d`命令会删除整个合并行（换行符分隔的两行）

- 多行`D`命令只删除合并行中的第一行（换行符及其之前的所有字符），合并行中的第二行虽然被`N`命令加入模式空间，但仍然完好

- 每当执行完`D`命令后，会强制 sed 编辑器在不读取数据流下一行的情况返回到命令脚本的顶部，重新开始执行命令列表

  

`P`命令

- `N`命令使得 sed 编辑器将换行符分隔的当前行和下一行当做一行来一起合并处理，此时单行`p`命令会打印整个合并行（换行符分隔的两行）

- 多行`P`命令只打印合并行中的第一行（换行符及其之前的所有字符）

  

模式空间时一块活跃的缓冲区，在 sed 编辑器执行命令时保存着待检查的文本，此外，sed 编辑器还有另一块称为保留空间的缓冲区，可以将模式空间复制（`h`）/附加（`H`）到保留空间，以便清空模式空间，载入其他要处理的字符串，最后再将保留空间复制（`g`）/附加（`G`）模式空间，或者交换（`x`）两者的内容，从而将保存的字符串移回模式空间。这种机制可以很灵活地来回移动文本行。

## 4.4 gawk 编辑器

gawk 是比 sed 更加强大的命令行编辑器，提供了一种编程语言，而不仅仅是编辑器命令，可以定义变量来保存数据、使用算术和字符串运算符来处理数据、使用结构化编程概念（比如`if-then`语句和循环）为数据处理添加处理逻辑、提取文件中的数据将其重新排列组合，最后生成格式化可读性报告，最完美的应用案例是格式化日志文件。

- 默认情况下，gawk 编辑器同样会将指定的命令（用单引号和花括号`'{}'`定义）应用于`STDIN`输入流或者指定数据文件中的数据流（比如`gawk '{print "hello"}'`使用了打印命令`print`，代表将`hello`输出到`STDOUT`打印出来）

- 如果需要应用多个命令，则要么直接在命令行`'{}'`中指定并用分号分隔，要么使用`-f`选项在单独的 gawk 脚本文件中指定

- gawk 命令默认可以使用`$0`代表当前整个文本行，`$1`代表文本行中的第一个数据字段，...，`$n`代表文本行中的第 n 个数据字段，`-F`选项指定了行中划分数据字段的字段分隔符，默认情况下是任意的空白字符（比如空格或制表符）

- gawk 脚本文件中可以定义变量来保存数据，引用 gawk 脚本自定义的变量的值时无须像 bash shell 脚本那样使用`$`符号，gawk 脚本文件中可以通过特殊变量`FS`来定义字段分隔符，此时就无须设置`-F`选项

- gawk 允许通过关键字`BEGIN`和`END`来定义在处理数据流开始之前和结束之后要执行的命令，即先执行`BEGIN`定义的命令，再依次匹配数据并处理数据，最后再执行`END`定义的命令，这就是生成格式化可读性报告的关键

- 可以在 gawk 脚本文件中依次定义`BEGIN`命令块，数据处理命令块以及`END`命令块

- gawk 支持 POSIX ERE 正则表达式引擎，并且能够提供一些 sed 所不具备的额外过滤功能，因此处理数据时往往比较慢

- gawk 默认情况下不识别正则表达式的`{}`，需要指定 gawk 的命令行选项`--re-interval`才能识别

  

# 5、参考资料

- 《Linux Command Line and Shell Scripting Bible, 4th Edition》（看到 488 页，就此告一段落吧......）