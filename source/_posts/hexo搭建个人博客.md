---
title: hexo搭建个人博客
tags:
  - hexo
categories: hexo
abbrlink: 64e2e0e2
date: 2018-03-04 21:33:40
---

## 个人总结hexo博客搭建流程以及常用命令 
<!--more-->


### 大概流程：

- 搭建 Node.js 环境
- 搭建 Git 环境
- GitHub 注册和配置
- 安装配置 Hexo
- 关联 Hexo 与 GitHub Pages
- GitHub Pages 地址解析到个人域名
- Hexo 的常用操作

### 搭建 Node.js 环境
> 为什么要搭建 Node.js 环境？ - 因为 Hexo 博客系统是基于 Node.js 编写的

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境，可以在非浏览器环境下，解释运行 JS 代码。

在 Node.js 官网：https://nodejs.org/en/ 下载安装包 v6.10.3 LTS

保持默认设置即可，一路Next，安装很快就结束了。

然后打开命令提示符，输入 node -v、npm -v，出现版本号则说明 Node.js 环境配置成功，第一步完成！！！
![mark](http://upload-images.jianshu.io/upload_images/5703029-f130f03d84a4a404..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 搭建 Git 环境
> 为什么要搭建 Git 环境？ - 因为需要把本地的网页和文章等提交到 GitHub 上。

Git 是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

在 Git 官网：https://git-scm.com/ 下载安装包 Git-2.13.0-64-bit.exe


桌面右键，打开 Git Bush Here，输入 git --version，出现版本号则说明 Git 环境配置成功，第二步完成！！！

![mark](http://upload-images.jianshu.io/upload_images/5703029-ef4a28df576283e4..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
GitHub 注册和配置
GitHub 是一个代码托管平台，因为只支持 Git 作为唯一的版本库格式进行托管，故名 GitHub。

### Github注册：https://github.com/


创建仓库：Repository name 使用自己的用户名，仓库名规则：



```
yourname/yourname.github.io
```



到此搭建 Hexo博客的相关环境配置已经完成，下面开始讲解 Hexo 的相关操作

### 安装配置 Hexo
Hexo 是一个快速、简洁且高效的博客框架，使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

强烈建议你花20分钟区读一读 Hexo 的官方文档：https://hexo.io/zh-cn/
![mark](http://upload-images.jianshu.io/upload_images/5703029-9985bc5f5b697711..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 使用 npm 安装 Hexo：在命令行中输入

1
npm install hexo-cli -g
然后你将会看到下图，可能你会看到一个WARN，但是不用担心，这不会影响你的正常使用。
![mark](http://upload-images.jianshu.io/upload_images/5703029-b9039915983f4996..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看Hexo的版本

```
hexo version
```

安装 Hexo 完成后，请执行下列命令来初始化 Hexo，用户名改成你的，Hexo 将会在指定文件夹中新建所需要的文件。


```

hexo init majunchang.github.io
cd majunchang.github.io
npm install
```


新建完成后，指定文件夹的目录如下：


```
.
├── .deploy         #需要部署的文件
├── node_modules    #Hexo插件
├── public          #生成的静态网页文件
├── scaffolds       #模板
├── source          #博客正文和其他源文件，404、favicon、CNAME 都应该放在这里
| ├── _drafts       #草稿
| └── _posts        #文章
├── themes          #主题
├── _config.yml     #全局配置文件
└── package.json    #npm 依赖等
```


#### 运行本地 Hexo 服务


```
hexo server
或者
hexo s

```

您的网站会在 http://localhost:4000 下启动。如果 http://localhost:4000 能够正常访问，则说明 Hexo 本地博客已经搭建起来了，只是本地哦，别人看不到的。下面，我们要部署到Github。
![mark](http://upload-images.jianshu.io/upload_images/5703029-60d264211b26e4fb..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 注意1：执行hexo server提示找不到该指令

解决办法：在Hexo 3.0 后server被单独出来了，需要安装server，安装的命令如下：


```
sudo npm install hexo-server
或者
npm install hexo -server --save
```




#### 配置Git个人信息

现在你已经可以通过 SSH 链接到 GitHub 了，还有一些个人信息需要完善的。
Git 会根据用户的名字和邮箱来记录提交。GitHub 也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的。


```
git config --global user.name "majunchang"
git config --global user.email "2471978285@qq.com"
```


#### 配置 Deployment

在_config.yml文件中，找到Deployment，然后按照如下修改，用户名改成你的：

需要注意的是：冒号后面记得空一格！


```
//  https 后面跟的是 git的用户名的密码 截止到@符号之前 所以密码中 不要包含@符号 否则会报错
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://majunchang:*****@github.com/majunchang/majunchang.github.io.git
  branch: master
```


本地文件提交到 GitHub Pages


```
// 删除旧的 public 文件
hexo clean
// 生成新的 public 文件
hexo generate
或者
hexo g
// 开始部署
hexo deploye
或者
hexo d
```


### Hexo 的常用操作
#### 发表一篇文章

```
hexo new "文章标题"
D:\GitHub\Hexo\test>hexo new "文章标题"
INFO  Created: D:\GitHub\Hexo\test\source\_posts\文章标题.md

```

在本地博客文件夹 source\_posts 文件夹下看到我们新建的 markdown 文件。

当然，我们也可以手动添加Markdown文件在source->_deploy文件夹下，其效果同样可以媲美hexo new

文章编辑好之后，运行生成、部署命令：


```
hexo clean
hexo g
hexo d
```


当然你也可以执行下面的命令，相当于上面两条命令的效果


```
hexo clean
hexo d -g

```


#### 文章如何添加多个标签

有两种多标签格式

```
tags: [a, b, c]
或
tags:
  - a
  - b
  - c
```


<!--more-->
#### 更改主题
官方主题库：https://hexo.io/themes/

Hexo主题非常，推荐使用 Next 为主题，请阅读 Next 的官方文档（ http://theme-next.iissnan.com/ ），5 分钟快速安装。

再提示一点，大家可以hexo主题修改一步就hexo s看下变化，初次接触对参数不清楚。只有hexo s后在可以在本地浏览到效果，Ctrl+C 停止服务器。

#### 添加插件
添加 sitemap 和 feed 插件

切换到你本地的 hexo 目 CIA ，在命令行窗口，输入以下命令


```

npm install hexo-generator-feed -save
npm install hexo-generator-sitemap -save
```

修改 _config.yml，增加以下内容


```
# Extensions
Plugins:
- hexo-generator-feed
- hexo-generator-sitemap
#Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20
#sitemap
sitemap:
  path: sitemap.xml
```


再执行以下命令，部署服务端


```
hexo d -g
```


配完之后，就可以访问 https://majunchang.github.io/ ，发现这两个文件已经成功生成了。


。


