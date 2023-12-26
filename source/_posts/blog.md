---
title: 搭建个人博客
date: 2022-05-06 15:25:39
tags:
---

# 1、工具

- [Git](../../07/git/)：一种分布式版本控制系统（DVCS）；

- GitHub：一个只支持Git进行版本控制的代码托管平台；

- GitHub Pages：提供免费静态网站托管服务，它能够对GitHub中的静态文件资源进行处理并展示成一个网站，同时还会给这个网站提供一个特定的域名，但用户也可以将域名改成自己想要的；

- Node.js：一个开源和跨平台的JavaScript运行时环境；

- Hexo：一个将用户编辑的md格式的文件生成HTML静态资源文件的博客框架。

# 2、前期准备

## 2.1 Git、GitHub及配置

1）在[Git](https://git-scm.com/downloads)下载安装包并安装；

2）鼠标右键可以看到：`Git Bash Here`，单击打开；

3）输入`git --version`，可以看到安装好的版本信息；

4）在[GitHub](https://github.com/)注册账号；

5）设置Git用户信息；

```bash
git config --global user.name "GitHubName"
git config --global user.email "GitHubEmail"
```

6）生成SSH key

```bash
ssh-keygen -t rsa -C "GitHubEmail"
# -t rsa 指定要创建的秘钥类型为 RSA
# -C "GitHubEmail" 指定追加到公钥文件末尾以便于识别的注释（通常以电子邮件地址用作注释，但也可以使用任何最适合你基础结构的事物）
```

7）连续一路回车直到生成一个矩形图案，此时，可以看到用户目录下生成了`.ssh`文件夹，该文件夹的`id_rsa`和`id_rsa.pub`两个文件分别对应SSH key的私钥和公钥；

<img src="文件目录截图1.png" alt="文件目录截图1" style="zoom:70%;" />

<img src="文件目录截图2.png" alt="文件目录截图2" style="zoom:70%;" />

8）打开GitHub主页 --> `Settings` --> `SSH and GPG keys` --> `New SSH key`，将`id_rsa.pub`中的内容复制粘贴到key中，title可以随便取，点击`Add SSH key`；

<img src="项目设置截图1.png" alt="项目设置截图1" style="zoom:70%;" />

9）验证是否成功；

```bash
ssh -T git@github.com
```

10）SSH key配置完毕后，就可以将本地的源码和资源上传到GitHub，而无需每次都输账号和密码。

## 2.2 GitHub Page和域名

1）打开GitHub主页 --> `+`按钮 --> `New respository`，在`Repository name`输入框中填入`GitHubName.github.io`，勾选`Initialize this repository with a README`选项；

2）在浏览器中输入https://GitHubName.github.io/ 即可看到如下页面，代表成功开启了GitHub Pages服务；

<img src="浏览器截图1.png" alt="浏览器截图1" style="zoom:70%;" />

3）在[万网-阿里云](https://wanwang.aliyun.com/)注册账号或直接登录，选择还未被注册的域名，进入结算页面，创建持有者信息模本，验证并等待审核，付钱；

<img src="域名注册截图1.png" alt="域名注册截图1" style="zoom:70%;" />

4）由于GitHub推荐在DNS中使用A记录和CNAME相结合的方式，在注册好的个人域名后点击解析添加如下两条解析记录，其中，IPv4地址可以通过`ping`得到；

```bash
ping GitHubName.github.io
```

<img src="域名注册截图2.png" alt="域名注册截图2" style="zoom:70%;" />

5）打开GitHubName.github.io仓库 --> `Add file` --> `Create new file`，文件名为`CNAME`，内容为注册好的个人域名，然后提交；

<img src="项目设置截图2.png" alt="项目设置截图2" style="zoom:70%;" />

6）打开GitHubName.github.io仓库 --> `Settings` --> `Pages`，勾选`Enforce HTTPS`选项即可强制走Https；

<img src="项目设置截图3.png" alt="项目设置截图3" style="zoom:70%;" />

7）在浏览器中输入个人域名即可看到如下页面，代表成功绑定了域名。

<img src="浏览器截图2.png" alt="浏览器截图2" style="zoom:70%;" />

## 2.3 Node.js和Hexo

1）在[Node.js](https://nodejs.org/en/download/)下载安装包并安装；

2）查看安装好的版本信息；

```bash
node -v
npm -v
```

3）在`nodejs`文件夹中新建两个空文件夹 `node_cache`、`node_global`；

<img src="文件目录截图3.png" alt="文件目录截图3" style="zoom:70%;" />

4）配置（如果不配置的话，后续安装模块时会安装到C盘，不方便管理，甚至有可能安装好hexo后却无法使用）；

```bash
npm config set prefix "D:\nodejs\node_global"
npm config set cache "D:\nodejs\node_cache"
```

5）打开控制面板 --> 系统 --> 高级系统设置 --> 环境变量，在系统变量中新建一个变量名为`ODE_PATH`，值为`D:\nodejs\node_global\node_modules`的变量；

<img src="系统设置截图1.png" alt="系统设置截图1" style="zoom:70%;" />

6）编辑用户变量里的`Path`，将npm的路径改为：`D:\nodejs\node_global`；

<img src="系统设置截图2.png" alt="系统设置截图2" style="zoom:70%;" />

7）安装webpack并查看版本信息；

```bash
npm install webpack –g
npm webpack –v
```

至此，npm在安装全局模块时的路径和环境变量均设置完成；

8）安装hexo并查看版本信息。

```bash
npm install -g hexo-cli
hexo –v
```

# 3、博客搭建

## 3.1 搭建和预览

1）在想要放置的地方建立一个文件夹`GitHubName.github.io`作为博客根目录；

2）在博客根目录下初始化hexo，可以看到博客根目录中有了以下内容；

```bash
hexo init
npm install
```

<img src="文件目录截图4.png" alt="文件目录截图4" style="zoom:70%;" />

3）在`source`文件夹下新建一个`CNAME`文件，然后在此文件中写入注册好的个人域名；

<img src="文件目录截图5.png" alt="文件目录截图5" style="zoom:70%;" />

4）在博客根目录下静态部署，生成博客，此时会将`source\_posts`文件夹下的`hello-world.md`文件解析到一个`public`文件夹下，并生成`db.json`文件；

```bash
hexo g
```

<img src="文件目录截图6.png" alt="文件目录截图6" style="zoom:70%;" />

5）在博客根目录下本地运行

```bash
hexo s
```

至此，在http://localhost:4000/ 即可本地预览博客，预览结束之后按`Ctrl+C`来停止运行服务器，每当修改完网站里的东西就可以通过这种方式来预览，而不是直接发布到网上。

## 3.2 配置和部署

1）在博客根目录下的`_config.yml`文件中作以下配置，`_config.yml`是整个博客的配置文件，每项配置参数在中[hexo官网](https://hexo.io/zh-cn/docs/configuration)有详细的介绍；

<img src="配置文件截图1.png" alt="配置文件截图1" style="zoom:70%;" />

2）在博客根目录下安装一个帮助我们上传文件的Git部署插件；

```bash
npm install hexo-deployer-git --save
```

3）将博客部署到GitHub服务器，这一步如果没有反应的话，就在`_config.yml`文件中所配置`repo`的冒号后再增加一个空格；

```bash
hexo d
```

4）在浏览器中输入个人域名即可看到如下页面，代表成功部署了博客。

<img src="浏览器截图3.png" alt="浏览器截图3" style="zoom:70%;" />

## 3.3 个性化主题

1）在[hexo themes](https://hexo.io/themes/)中挑一个想要的hexo主题，在博客根目录下执行以下操作，即可将xxx主题保存到`themes\xxx`文件夹下；

```bash
git clone xxxURL themes\xxx
```

2）在博客根目录下的`_config.yml`文件中作以下配置；

<img src="配置文件截图2.png" alt="配置文件截图2" style="zoom:70%;" />

3）关于个人图标、打赏、评论等功能需要在xxx主题的`_config.yml`等文件中进行配置，不同主题的配置方法可能略有不同，可自行根据所设置主题的`README.md`文件所介绍的内容进行配置；

4）在博客根目录下重复以下步骤即可在http://localhost:4000/ 中预览新主题下的博客；

```bash
hexo clean
hexo g
hexo s
```

5）将博客部署到GitHub服务器并等待一段时间后，在浏览器中输入个人域名即可看到所设置主题下的博客。

```bash
hexo d
```

## 3.4 菜单和写作

1）在`themes\xxx`目录下的`_config.yml`文件中作以下配置，即可设置想要的博客菜单，下图设置了首页、分类以及归档菜单；

<img src="配置文件截图3.png" alt="配置文件截图3" style="zoom:70%;" />

2）在博客根目录下执行以下操作即可在`source`目录下生成一个`categories`文件夹，编辑（[markdown](https://markdown.com.cn/)）该文件夹中的`index.md`文件，那么在网页上点击`categories`菜单后就能显示出所编辑的内容；

```bash
hexo new page categories
```

<img src="文件目录截图7.png" alt="文件目录截图7" style="zoom:70%;" />

3）为了在静态部署生成博客时能够将`source\_posts`中的图片资源也解析到`public`文件夹中，需要在博客根目录下的`_config.yml`文件中作以下配置；

<img src="配置文件截图4.png" alt="配置文件截图4" style="zoom:90%;" />

4）此时执行以下操作即可在`source\_post`目录新建一个`xxx.md`文件和一个`xxx`文件夹，那么在编辑`xxx.md`文件时所使用的图片资源就放置在`xxx`文件夹中；

```bash
hexo new xxx
```

5）有时候需要在不同页面跳转，只需在其他md文件中插入需要的跳转的md文件被解析后的地址，即可完成跳转；

6）重复之前的预览、调整和部署操作，在浏览器中输入个人域名即可看到编辑后的博客内容；

7）在博客根目录下安装以下插件可分别令博客支持搜索功能和文章加密功能。

```bash
npm install hexo-generator-searchdb
npm install --save hexo-blog-encrypt
```

# 4、问题记录

1）**插入图片的路径问题：**静态部署前`xxx.md`文件中所插入的图片位于`xxx`文件夹中，静态部署后图片会被解析到与html文件相同的文件夹下，因此，在编辑完成后静态部署前需要将插入图片的路径更改为正确的形式，这一点可自行观察解析后的`public`文件夹结构

2）**代码显示问题：**无序列表文本中采用反引号包裹的代码在网页上没有显示出代码形式，而是普通文本形式，此时，需要在无序列表的某一条中的最后依次输入回车和后退（不要输入连续两次回车），此时反引号包裹的代码即可在网页上正常显示了（导致这个问题的原因暂时还不知道）

3）**浏览器上的更新问题：**有时候需要将浏览数据清除，再重新打开网页才能正确显示出更新后的博客内容

4）**GitHub Page 不显示问题**：仓库设为私有后，博客将无法正常显示，即便再改为公有后还是无法显示，此时需要在 GitHub Page 中将域名移除重新添加即可

5）**命令执行权限问题**：有的命令需要管理员身份才能执行，比如`hexo d`

6）**部署时报错**：先删除 `.deploy_git` 文件夹，然后输入 `git config --global core.autocrlf false`，最后重新部署即可

<img src="报错截图1.png" alt="文件目录截图7" style="zoom:70%;" />

# 5、参考资料

- [《A-Guide-Of-Making-Your-Personal-Blog》](https://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-1/)
- [《about-custom-domains-and-github-pages》](https://docs.github.com/cn/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)
- https://juejin.cn/post/6844904131266609165
- https://zhuanlan.zhihu.com/p/102592286
- http://lowrank.science/archives/
- https://github.com/D0n9X1n/hexo-blog-encrypt/blob/master/ReadMe.zh.md
- https://www.itfanr.cc/2021/04/16/hexo-blog-article-encryption/

