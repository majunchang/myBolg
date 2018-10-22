---
title: node和mongdb
date: 2016-11-20 10:33:24
tags:
categories: Nodejs
---
###  1  原生操作数据库 和  使用插件操作数据库  (mongoose)

第一种方式 ：使用原生node

1 定义1个db模块 

```javascript
var  mongo = require('mongodb');
var mongoClient = mongo.MongoClient;
var url = 'mongodn://localhost:27017/itcast';
```

2 将db 暴露给其他的 处理模块 

使用的时候  这样使用就可以了：

db.find...// 传入响应的参数

第二种方式 ：使用插件mongoose

引入插件 连接数据库  声明数据模板

import mongoose from 'mongoose'

mongoose.conmect('mongodb://localhost/edu');

const advertSchema = new mongoose.Schema({//

    这个里面写单条数据的 数据结构

})

2 将这个模板 暴露出去 

export default mongoose.model('Advert',advertSchema)

3 在需要使用的地方进行接受  接受的时候 格式要对应

import {Advert} from '//相应的文件'

使用Advert 进行操作 增删差改



### 2 使用express 配置访问静态路径  这个 join 引用的是 path里面的join   这个path 是nodejs自带的东西

使用ejs 配置模板引擎 默认是找后缀名为ejs的文件 ，这个是需要单独安装的。

使用babel解析ecma6  这个具体的配置在阮一峰的博客上面

```javascript
    app.use('/node_modules', express.static(config.node_modules_path))
    app.use('/public', express.static(config.public_path))

    viewPath: join(__dirname, '../views'),
    node_modules_path: join(__dirname, '../node_modules'),
    public_path: join(__dirname, '../public')
```

### 3  详解express中间件

```javascript
    const express = require('express')
    const fs = require('fs')
    const path = require('path')
    const static = require('./middlwares/static')
    
    const app = express()
    
    app.use((req, res, next) => {
      const log = `请求方法：${req.method}  请求路径：${req.url} 请求时间：${+new Date()}\n`
      fs.appendFile('./log.txt', log, err => {
        if (err) {
          return console.log('记录日志失败了')
        }
        next()
      })
    })
 ```
``` javascript
    // 中间件：用来处理 http 请求的一个具体的环节（可能要执行某个具体的处理函数）
    //         中间件一般都是通过修改 req 或者 res 对象来为后续的处理提供便利的使用
    // 中间件分类：
    //     use(function () {req, res, next})  不关心请求方法和请求路径，没有具体路由规则，任何请求都会进入该中间件
    //     use('请求路径', function (req, res, next) {}) 不关心请求方法，只关心请求路劲的中间件
    //     get('请求路径', function (req, res, next) {}) 具体路由规则中间件
    //     post('请求路径', function (req, res, next) {})
    // 中间件的作用：
    //    
    
    // app.use('/public', express.static('开放目录的路径'))
    
    // 在 use 方法中，如果指定了第一个路径参数，则通过 req.path 获取到的是不包含该请求路径的字符串
    // 例如当前请求路劲是 /public/a.jpg 则通过 req.path 拿到的就是 a.jpg
    //                    /public/a/a.css a/a.css
    //                 目前已知传递给了 static 方法一个绝对路径 c:/project/public
    //                 假设目前请求是 /public/a/a.css 拿到的 req.path a/a.css
    //                 c:/project/public + a/a.css 拼接起来，读取
    app.use('/public', static(path.join(__dirname, 'public')))
    app.use('/node_modules', static(path.join(__dirname, 'node_modules')))
    
    app.get('/', (req, res, next) => {
      console.log('/ 111')
      res.end('hello')
      next()
    })

```

### 4  使用mongoose操作数据库的基本语法

```javascript
    const mongoose = require('mongoose')
    
    mongoose.connect('mongodb://localhost/test')
    
    // 1. 创建一个模型架构，设计数据结构和约束
    const studentSchema = mongoose.Schema({
      name: String,
      age: Number
    })
    
    // 2. 通过 mongoose.model() 将架构发布为一个模型（可以把模型认为是一个构造函数）
    //    第一个参数就是给你的集合起一个名字，这个名字最好使用 帕斯卡命名法
    //        例如你的集合名 persons ，则这里就命名为 Person，但是最终 mongoose 会自动帮你把 Person 转为 persons
    //    第二个参数就是传递一个模型架构
    const Student = mongoose.model('Student', studentSchema)
    
    //3. 通过操作模型去操作你的数据库
    //   保存实例数据对象
    const s1 = new Student({
      name: 'Mike',
      age: 23
    })
    s1.save((err, result) => {
      if (err) {
        throw err
      }
      console.log(result)
    })
```
###  5  使用nunjucks模板引擎 

    // 配置使用 nunjucks 模板引擎
    // nunjucks 模板引擎没有对模板文件名的后缀名做特定限制
    // 如果文件名是 a.html 则渲染的时候就需要传递 a.html
    // 如果文件名是 b.nujs 则传递 b.nujs
    // nunjucks 模板引擎默认会缓存输出过的文件
    // 这里为了开发方便，所以把缓存禁用掉，可以实时的看到模板文件修改的变化
    nunjucks.configure(config.viewPath, {
      autoescape: true,
      express: app,
      noCache: true
    })

### 6 使用body-parser处理post请求  将其挂载在req.body上 

``` javascript
    // 如果是普通表单POST，则咱们自己处理 application/x-www-form-urlencoded
    // 如果是有文件的表单POST，则咱们不处理
    if( req.headers['content-type'].startsWith('multipart/form-data') ) {
      return next()
    }

    import queryString from 'querystring'
    
    export default (req, res, next) => {
      // req.headers 可以拿到当前请求的请求报文头信息
      if (!req.headers['content-length']) {
        console.log('进入 body-parser 了')
        return next()
      }
      let data = ''
      req.on('data', chunk => {
        data += chunk
      })
      req.on('end', () => {
          // 将查询字符串 转化为对象
        req.body = queryString.parse(data)
        next()
      })
    }
```

### 7  使用formidable处理文件上传 比如说 图片这种 

``` javascript
npm install --save formidable

yarn add formidable

    const form = new formidable.IncomingForm()
    form.uploadDir = config.uploadDir // 配置 formidable 文件上传接收路径
    form.keepExtensions = true // 配置保持文件原始的扩展名
    form.parse(req, (err, fields, files) => {
      if (err) {
        return next(err)
      }
    
      const body = fields // 普通表单字段
      body.image = basename(files.image.path) // 这里解析提取上传的文件名，保存到数据库
    
      const advert = new Advert({
        title: body.title,
        image: body.image,
        link: body.link,
        start_time: body.start_time,
        end_time: body.end_time,
      })
    
      advert.save((err, result) => {
        if (err) {
          return next(err)
        }
        res.json({
          err_code: 0
        })
      })
    }
```
#### 还有一种 就是 使用promise对象 处理回调函数

``` javascript
    pmFormidable(req)
      .then((result) => {
        const [fields, files] = result
        const body = fields // 普通表单字段
        body.image = basename(files.image.path) // 这里解析提取上传的文件名，保存到数据库
        const advert = new Advert({
          title: body.title,
          image: body.image,
          link: body.link,
          start_time: body.start_time,
          end_time: body.end_time,
        })
        return advert.save()
      })
      .then(result => {
        res.json({
          err_code: 0
        })
      })
      .catch(err => {
        next(err)
      })
    
    function pmFormidable(req) {
      return new Promise((resolve, reject) => {
        const form = new formidable.IncomingForm()
        form.uploadDir = config.uploadDir // 配置 formidable 文件上传接收路径
        form.keepExtensions = true // 配置保持文件原始的扩展名
        form.parse(req, (err, fields, files) => {
          if (err) {
            reject(err)
          }
          resolve([fields, files])
        })
      })
    }

````
#### 同步分页 和 异步无刷新分页 

``` javascript
 //同步分页  这个是在服务器端 就进行分页 

    {
        const page = Number.parseInt(req.query.page,10);
        const pageSize = 5;
        Advert
            .find()
            .skip((page - 1) * 5)
            .limit(10)
            .exec((err, adverts) => {
                if (err) {
                    return next(err)
                }
                // https://docs.mongodb.com/manual/reference/method/db.collection.count/
                // 这个的api 又回到了 db.collection.count 这里的ccount 就是指的总条数
                Advert.count((err,count)=>{
                    if(err)
                    {
                        return next(err)
                    }
                    const totalPage = Math.ceil(count/pageSize);
                    res.render('advert_list.html',{
                        adverts,
                        totalPage,
                        page
                    })
                })
            })
    }

```
#### 异步无刷新分页  使用的   twbs-pagination  插件 

``` javascript
    $.ajax({
        url: '/advert/count',
        type: 'get',
        success: function (data) {
            if (data.err_code === 0) {
                $('#pagination').twbsPagination({
                    totalPages: Math.ceil(data.result / pageSize),
                    visiblePages: 5,
                    first: '首页',
                    prev: '上一页',
                    next: '下一页',
                    last: '尾页',
                    onPageClick: function (event, page) {
                        // 当你点击该页的时候 进行的行为
                        // 在这里我们要加载数据
                        loadData(page);
                    }
                })
            }
        }
    })
```
###  用户登录状态的保持 cookie和session

``` javascript
     有一个cookie-parser插件 

    // 关于从 HTTP 协议角度使用 Cookie
    // 1. 通过响应报文在报文头中加入一个字段：Set-Cookie 值就是：key=value 的格式
    // 2. 当浏览器收到服务端响应回来的数据的时候，发现响应头中有一个 Set-Cookie 
    //    然后浏览器会将该 Cookie 对应的数据保存起来（会话Cookie，持久化）
    // 3. 当下一次浏览器再次访问该服务器的时候，浏览器会自动将保存的所有 Cookie 放到请求报文头中都带上来
    // 4. 然后服务端就可以根据当前客户端的请求报文头中的 cookie 字段来获取进行判断

    // 引入express中处理cookie的中间件
    import cookieParser from 'cookie-parser'

    // 挂载 cookie-parser 中间件：专门用来处理对 cookie 的操作的
    // cookie-parser 中间件做两件事儿：
    //    req.cookies 通过该中间件会自动将当前请求报文中的 cookie 解析为一个对象挂载到 req.cookies 属性上
    //    res.cookie(name, value) 通过 res.cookie 可以向当前请求客户端发送 cookie 数据  // 这是设置
    app.use(cookieParser())
```
### 手写session

``` javascript
    const sessionStorage = {}
    
    export default (options) => {
      return (req, res, next) => {
        const { sidName } = options
        // 当客户端请求过来的时候，先判断一下是否有 sessionid
        // 如果有，则直接跳过去
        // 如果没有，则生成一把钥匙发送给用户
        const sidCookie = req.cookies[sidName]
    
        // req.getSession = function () {
        //   !sessionStorage[sidCookie] && (sessionStorage[sidCookie] = {})
        //   return sessionStorage[sidCookie]
        // }
    
        // req.setSession = function (name, value) {
        //   console.log('setSession')
        //   req.getSession()[name] = value
        // }
    
        // 添加一个属性成员 req.session
        // 作用：用来根据当前请求客户端的 sessionid 对 Session 数据的取值和赋值
        Object.defineProperty(req, 'session', {
          get: function () {
            !sessionStorage[sidCookie] && (sessionStorage[sidCookie] = {})
            return sessionStorage[sidCookie]
          }
        })
        
        // 如果没有 sessionid ，则生成一个，发送给客户端
        if (!sidCookie) {
         res.cookie(sidName, generateSid())
         return next()
        }
    
        // 如果有 sessionid ，则直接跳过
        next()
      }
    }
    
    function generateSid() {
      return `it_${Math.random().toString().substr(2)}cast${Math.random().toString().substr(2)}`
    }
    
    // export default (req, res, next) => {
    
    // }
```


