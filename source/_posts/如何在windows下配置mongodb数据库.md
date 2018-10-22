---
title: windows下配置mongodb
date: 2017-10-10 23:32:47
categories: javascript
tags: mongodb
---

###  如何在windows下配置mongodb数据库

#### 下载安装包

官网下载链接 https://www.mongodb.com/download-center?jmp=nav#community

下载以后进行解压 安装 （建议安装到c盘）



#### 如何启动和使用服务

在mongodb的安装目录bin下  打开命令行 输入 ./mongod  将数据库服务启动起来 默认在27017端口 

然后 在该目录下 重新打开一个命令行 输出mongo  将数据库连接起来  就可以使用nodejs操作数据库了



#### 如何配置环境变量 使其可以在全局范围呢 打开



1.![mark](http://oneg19f80.bkt.clouddn.com/blog/20171205/143701900.png)2.![mark](http://oneg19f80.bkt.clouddn.com/blog/20171205/143925480.png)3. ![mark](http://oneg19f80.bkt.clouddn.com/blog/20171205/143941245.png)

4.![mark](http://oneg19f80.bkt.clouddn.com/blog/20171205/143956889.png)

5.  经过以上步骤配置环境变量 我们可以在任意文件夹（包括桌面）中 以mongod 和mongo  来启动和连接数据库服务了
