---
title: 使用localstorage作为本地数据库以及跨域問題的解決
date: 2017-04-16 11:32:59
tags:
---

深入浅出Nodejs-04

package.json 文件

package.json 文件一般用来描述项目的一些基本信息，例如入口文件、依赖项、项目介绍、开发作者等数据。

目前已知的两个非常重要的属性：

- main
  - main 和模块化中的第三方包加载规则有关系
- dependencies
  - dependencies 和 npm 命令行工具有关系
  - 当你安装包的时候，如果加上 --save 参数，则npm会自动把这个第三方包依赖信息写入到 package.json 文件中的 dependencies 字段中
  - 当你执行 npm install 的时候，npm 会找到当前项目中的 package.json 文件中的 dependencies 依赖项，然后依次将所有的依赖下载下来

这个文件最好每一个项目都有，保存一些项目的基本信息。

这个文件可以通过 npm init 以向导的形式生成，也可以加上 -y 参数，一步生成。

Express

一个基于 Node 开发的一个快速 Web 开发框架

主要用来构建 Server

- http://expressjs.com/
- http://www.expressjs.com.cn/

hello-world

    var express = require('express')
    
    // 1. 调用 express 方法，得到一个类似于 server 的实例
    //    一般称作 app
    var app = express()
    
    // 2. 通过 app 根据不同的请求方法及请求路径设定具体的处理函数
    
    // 当用户以 GET 请求 / 的时候，执行对应的回调处理函数
    app.get('/', function (req, res) {
      res.end('hello world')
    })
    
    // 当用户以 GET 请求 /login 的时候，执行对应的回调处理函数
    app.get('/login', function (req, res) {
      res.end('hello login')
    })
    
    // 3. 启动监听
    app.listen(3000, function () {
      console.log('服务器已启动，请访问：http://127.0.0.1:3000/')
    })

处理静态资源

参考文档：http://www.expressjs.com.cn/starter/static-files.html

通过 Express 内置的 express.static 可以方便地托管静态文件，

例如图片、CSS、JavaScript 文件等。

配置规则如下：

    app.use('路径访问前缀', express.static('资源目录路径'))

以下是一些配置示例：

- 将目录 static 资源暴露出来，可以通过 /static/* 的形式进行访问
  - app.use('/static/', express.static('static目录的绝对路径'))
  - /static/css/a.css
  - /static/css/b.css
  - /static/img/ab2.jpg
- 将目录 public 资源暴露出来，不需要任何前缀就可以访问
  - app.use(express.static('public 目录的绝对路径'))
  - /css/bb.css
  - /img/a.jpg
- 将目录 demo 资源暴露出来，可以通过 /aa/* /aa前缀的形式进行访问
  - app.use('/aa/', express.static('demo 目录的绝对路径'))
  - /aa/css/a.css
  - /aa/**/*.*
- 将目录 static 资源暴露出来，可以通过 /static/* 或者 /aa/* 的形式进行访问 
  - app.use('/aa/', express.static('static 目录的绝对路径'))
  - app.use('/static/', express.static('static 目录的绝对路径'))
  - 上面的形式就是把 static 目录中的资源提供了两种形式，既能以 /static/ 为前缀进行访问也可以以 /aa/ 的前缀进行访问

路由系统

在 Express 配置使用 ejs 模板引擎

Express 这个框架很精简，默认是不支持模板引擎的，需要配合一些第三方的模板引擎来结合使用，

例如这里将 ejs 和 express 结合起来使用：

第一：安装 ejs：

    npm install --save ejs

第二：在代码中配置：

    app.set('views', 模板文件存储路径) // 注意，这里可以不配置，因为 Express 默认会去项目中的 `views` 目录进行查找
    app.set('view engine', 'ejs') // 这里表示让 Express 中的 res.render 方法使用 ejs 模板引擎，这里的 ejs 就是你安装的那个模板引擎的包名

只要经过了上面这种配置，然后 res 对象上就会自动多出一个方法：res.render ,使用方式和咱们之前

自己封装的一样：res.render('视图名称', {要解析替换的对象数据})

注意：使用了 ejs 模板引擎，默认视图文件后缀名必须是 .ejs，否则 render 方法找不到。

如果想要修改，可以像下面这样：

    // app.set('view enginge', 'ejs')
    // 将上面这句配置改为下面的形式，就修改了默认的 .ejs 后缀名
    app.engine('.html', require('ejs').__express)
    app.set('view engine', 'html')

在 Express 中配置使用 body-parser 插件解析处理表单 POST 请求体

第一步：安装 body-parser

    npm install --save body-parser

第二步，在代码中进行配置：

    app.use(bodyParser.urlencoded({ extended: false }))
    app.use(bodyParser.json())

只要经过上面的安装配置，则在任意的 post 处理函数中都可以直接通过 req.body 来获取表单 POST 请求体数据。

例如：

    app.post('login', function (req, res) {
      // 这里可以直接通过 req.body 来获取表单 POST 请求体数据
      console.log(req.body)
    })

---

MongoDB

数据库概念

数据库：一个电子化的文件柜

数据库就是为我们方便的管理数据的一个平台，例如对数据的存储、修改、查询等都非常的方便。

数据库分类

数据库产品有很多，以下是一些常见的数据库产品：

- MySQL
- Oracle
- DB2
- SqlServer
- MongoDB
- etc.

MongoDB 数据库（内存型的）

为了更好的学习 MongoDB 数据库，大家可以参考菜鸟教程上的 MongoDB 数据库教程文档，

链接地址：http://www.runoob.com/mongodb/mongodb-tutorial.html

MongoDB 数据库与mysql 等老牌数据库相比的优劣势









安装与配置 MongoDB 数据库环境

启动 MongoDB 服务实例

可以通过使用安装程序中的 mongod CLI应用程序来启动 MongoDB 服务。

直接在控制台输入：mongod 敲回车即可。

在启动的时候，可以通过 --dbpath 指定数据服务存储数据的目录，

如果不指定该目录，默认 mongod 会去 c:/data/db 作为自己的数据存储目录。

64 位版本启动 MongoDB 数据服务：

    mongod --dbpath C:\data\db

32 位版本使用下面的命令启动数据服务：

    mongod --dbpath 数据存储路径 --journal --storageEngine=mmapv1

提示：如果不加 --dbpath, mongod 会自动使用 C:\data\db 目录作为自己的数据存储路径，

所以，如果你已经有了 C:\data\db 目录了，可以省略 --dbpath。

执行完上面的命令并成功开启 MongoDB 数据服务实例之后，就把该控制台最小化到一边就可以了，

千万不要关闭，否则无法连接，如果对数据库的操作结束，可以打开该控制台通过 Ctrl + C 关闭。

连接 MongoDB 数据服务

注意：在进行连接之前请确保你的服务实例是开启状态的（不要关闭刚才开启的数据服务实例）。

打开一个新的控制台，在控制台输入以下命令用来连接本机的 MongoDB 服务实例：

    mongo

mongo 命令默认去连接本机上的 MongoDB 服务实例：127.0.0.1:27017，可以通过下面的命令

指定连接的主机名和端口号：

    mongo --host 127.0.0.1 --port 27017

如果看到类似于如下的字样说明连接成功：

    MongoDB shell version v3.4.0
    connecting to: mongodb://127.0.0.1:27017
    MongoDB server version: 3.4.0
    Server has startup warnings:
    2017-01-18T18:49:53.865+0800 I CONTROL  [initandlisten]
    2017-01-18T18:49:53.865+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
    2017-01-18T18:49:53.866+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
    2017-01-18T18:49:53.866+0800 I CONTROL  [initandlisten]
    >

如果提示 “无法连接主机”，请检查你的 MongoDB 数据服务实例是否开启。

基本操作命令

- show dbs
  - 查看当前服务实例上所有的数据库
- use 数据库名称
  - 这个命令表示切换到指定的数据库
  - 如果没有，也不会创建
  - 如果已经有了，则表示切换到这个数据库对该数据库进行操作
- db
  - 查看当前所处的数据库
- db.集合名称.insert(数据文档)
- show collections
  - 查看当前数据库中所有的集合
- db.集合名称.find()
  - 查询指定集合中所有的数据
  - 可以通过 db.集合名称.find().pretty() 美化输出格式
  - 默认是查询所有，可以通过：db.集合名称.find({查询条件}) 按条件查询集合中的数据
- db.集合名称.update({更新条件}, {要更新的字段})
  - 更新指定集合数据
- db.集合名称.remove({删除条件})
  - 删除指定集合中的数据

使用 Node 操作 MongoDB

项目结构如下 db为封装的Mongodb的增删改查方法，student为学生管理系统中分散的路由：



安装 MongoDB 官方提供的驱动包：

    npm install --save mongodb

具体操作方式请参考官方文档：https://www.npmjs.com/package/mongodb

使用原生方式操作MongoDB

db

    var mongo = require('mongodb');
    var MongoClient = mongo.MongoClient;
    
    var url = 'mongodb://localhost:27017/itcast';
    
    // 获取到 数据库 单条数据本身的ObjectID，并把这个通过接口暴露出去
    exports.ObjectID = mongo.ObjectID;
    exports.insertOne = function (collectionName, doc, callback) {
        MongoClient.connect(url, function (err, db) {
            if (err) {
                // 如果报错了  酒吧这个错误传递给外边的回调函数
                return callback(err);
            }
            db.collection(collectionName)
                .insertOne(doc, function (err, result) {
                    if (err) {
                        return callback(err);
                    }
    
                    callback(null, result);
    
                    db.close()
                })
        })
    }
    
    exports.findOne = function (collectionName, conditon, callback) {
        MongoClient.connect(url, function (err, db) {
            if (err) {
                // 如果报错了  酒吧这个错误传递给外边的回调函数
                return callback(err);
            }
            db.collection(collectionName)
                .findOne(conditon, function (err, result) {
                    if (err) {
                        return callback(err);
                    }
                    callback(null, result);
                    db.close()
                })
        })
    }
    
    
    exports.find = function (collectionName, condition, callback) {
        MongoClient.connect(url, function (err, db) {
            if (err) {
                // 如果报错了  酒吧这个错误传递给外边的回调函数
                return callback(err);
            }
            db.collection(collectionName)
                .find(condition)
                .toArray(function (err, docs) {
                    if (err) {
                        return callback(err)
                    }
                    callback(null, docs);
                    db.close();
                })
        })
    }
    exports.deleteOne = function (collectionName, condiction, callback) {
        MongoClient.connect(url, function (err, db) {
            if (err) {
                return callback(err);
            }
            db.collection(collectionName)
                .deleteOne(condiction, function (err, result) {
                    if (err) {
                        return callback(err);
                    }
                    callback(null, result);
                })
        })
    }
    
    // 做编辑功能
    exports.updateOne = function (collectionName, condition, changeDoc, callback) {
        MongoClient.connect(url,function (err, db) {
            if (err) {
                return callback(err)
            }
            db.collection(collectionName)
                .updateOne(condition, changeDoc, function (err, result) {
                if(err)
                {
                    return callback(err);
                }
                callback(null, result);
                })
        })
    }

student:

    var db = require('./db');
    
    // db  就是我们封装号的对象 通过require  来暴露接口
    
    exports.index = function (req,res) {
        db.find('students',{},function (err,docs) {
            if(err)
            {
                // res.end(JSON.stringify({对象}))
                // res.json 是 Express 帮你扩展的一个方法，传入一个对象，自动帮你转为字符胡之后再去发送给客户端
                return res.json({
                    err_no:500,
                    message:'查询数据失败，请稍后重试'
                })
            }
            console.log(docs)
            res.render('student/list',{
                students:docs
            })
        })
    }
    
    exports.list = function (req, res) {
        db.find('students', {}, function (err, docs) {
            if (err) {
                return res.json({
                    err_no: 500,
                    message: '查询数据失败，请稍后重试'
                })
            }
            res.json({
                err_no: 0,
                data: docs
            })
        })
    }
    exports.new = function (req,res) {
        res.render('student/new');
    }
    
    exports.doNew = function (req,res) {
        db.insertOne('students',req.body,function (err,result) {
            if(err)
            {
                return res.json({
                    err_no:500,
                    message:'服务器正忙，请稍后重试'
                })
            }
            res.json({
                err_no:0,
                message:'insert success'
            })
        })
    }
    
    // 这是对查看按钮 做的操作
    
    exports.info = function (req,res) {
        var id = req.query.id;
        db.findOne('students',{
            _id:db.ObjectID(id)
        },function (err,doc) {
            if(err)
            {
                return res.json({
                    err_no:500,
                    message:'操作数据库失败了'
                })
            }
            console.log(doc);
            res.render('student/info',{
                student:doc
            })
        })
    }
    
    
    exports.remove = function (req,res) {
        var id = req.query.id;
        db.deleteOne('students',{
            _id:db.ObjectID(id)
        },function (err,result) {
            if(err)
            {
                return res.json({
                    err_no:500,
                    message:'操作数据库失败'
                })
            }
            res.json({
                err_no:0,
                message:'success'
            })
            /*
            res.redirect  就是重定向方法
            告诉客户端浏览器 你重新请求这个路径把  对于异步请求来说
            服务端的 redirect 无效的
            red.redirect('/students')
             */
        })
    }
    
    // 制作编辑功能
    exports.edit = function (req,res) {
        var id = req.query.id;
        console.log(id)
        db.findOne('students', {
            _id: db.ObjectID(id)
        }, function (err, doc) {
            if (err) {
                return res.json({
                    err_no: 500,
                    message: '读取数据失败，请稍后重试'
                })
            }
            res.render('student/edit', {
                student: doc,
                majors: [
                    'java',
                    'ui',
                    'ios',
                    '前端与移动开发',
                    '全栈'
                ]
            })
        })
    }
    
       exports.doEdit = function (req,res) {
           var body = req.body;
           var id  = body.id;
           delete body.id;
           db.updateOne('students',{
               _id:db.ObjectID(id)
           },{
               $set:body
           },function (err,result) {
               if (err) {
                   return res.json({
                       err_no: 500,
                       message: '更新失败，请稍后重试'
                   })
               }
               res.json({
                   err_no: 0,
                   message: 'success'
               })
           })
       }
    

路由router：

    var express = require('express');
    var index = require('./controllers/index');
    var student = require('./controllers/student');
    
    var router = express.Router();
    
    // 解析这句话 就是 当路径地址是 '/'的时候 会调用后来的这个方法来进行处理
    router.get('/',index.index);
    
    // 一个是 求情主页 一个是请求new 页
    router
        .get('/students',student.index)
        .get('/students/new',student.new)
        .get('/students/info',student.info)
        .post('/students/new',student.doNew)
        .get('/students/list',student.list)
        .get('/students/remove',student.remove)
        //  在这个地方设置编辑信息
    // 有两种形式 一种是利用get的形式  一种是利用post的形式
        .get('/students/edit',student.edit)
        .post('/students/edit',student.doEdit)
    // 暴露接口
    module.exports = router;



app：

    var express = require('express');
    var path = require('path');
    var router = require('./router');
    var bodyParser = require('body-parser');
    var app = express();
    
    // 配置静态资源的访问路径
    // 将node_module开放为公共资源 ，可以通过路径的形式直接访问该目录中的任意资源
    app.use('/node_modules/',express.static(path.join(__dirname,'node_modules')));
    
    app.set('views',path.join(__dirname,'views'));
    //这一步设置 要注意 是view
    app.set('view engine','ejs');
    
    // 配置 body-parser插件 用来解析 表单post请求体
    app.use(bodyParser.urlencoded({extended:false}));
    app.use(bodyParser.json());
    
    // 挂载路由
    app.use(router);
    
    app.listen(3000,function () {
        console.log('地狱之门已经开启，尽情的杀戮吧');
    })



使用插件操作

http://www.nodeclass.com/api/mongoose.html






