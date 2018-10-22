---
title: gulp工具总结
date: 2017-03-26 09:39:30
tags:
---
# Gulp Tutorial

> Gulp: The streaming build system

![gulp](http://oneg19f80.bkt.clouddn.com/blog/20170326/100358707.png)

gulp是前端开发过程中一种基于流的代码构建工具，是自动化项目的构建利器；
她不仅能对网站资源进行优化，而且在开发过程中很多重复的任务能够使用正确的工具自动完成；
使用她，不仅可以很愉快的编写代码，而且大大提高我们的工作效率。

项目构建是指项目上线之前对项目源代码进行一系列处理，使其以最佳的形式运行于线上服务器。
常见处理任包括以下几方面：

1. 模块化开发可以实现功能的复用并解决模块间的依赖关系，但带来好处的同时也使得功能代码的碎片化（若干文件）程度增加。
2. 使用less、sass等预处理器，可以降低CSS的维护成本，最终需要将这些预处理器编译成css文件；
3. 对静态资源（css、js、html、images）压缩合并可以提升网页打开速度，提高性能；

以上任务完如果完全靠手动来完成是非常耗时耗力的且容易出错，实际开发通常借助构建工具来实现。
所谓构建工具是指通过一系简单配置就可以帮我们实现合并、压缩、校验、预处理等一系列任务的软件工具。
常见的构建工具包括：`Grunt、Gulp、F.I.S（百度出品）、webpack`等。

Gulp是基于Nodejs开发的一个构建工具，借助gulp插件可以实现不同的构建任务，
其以简洁的配置和卓越的性能成为目前主流的构建工具。

![gulpd的功能](http://oneg19f80.bkt.clouddn.com/blog/20170326/100510355.jpg)

## Introduction

- 官方：http://gulpjs.com/
- 中文官网：http://www.gulpjs.com.cn/
- npm：https://www.npmjs.com/package/gulp
- Github：https://github.com/gulpjs/gulp
- Gitbook：https://wizardforcel.gitbooks.io/gulp-doc/content/2.html

---

## Getting Started

> 官方文档：https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md

一：Install the gulp command

在项目中使用 gulp 首先需要确保全局有 gulp-cli 环境，如果有就不需要执行下面的命令了。

```bash
# npm install --global gulp-cli
yarn global add gulp-cli
```

二：Install gulp in your devDependencies

```bash
# npm install --save-dev gulp
yarn add -D gulp
```

三：Create a file called gulpfile.js in your project root with these contents:

```bash
var gulp = require('gulp');

gulp.task('default', function() {
  console.log('hello gulp')
})
```

四：Test it out: Run the gulp command in your project directory:

```bash
gulp
```

---

## API Documentation

> 官方文档：https://github.com/gulpjs/gulp/blob/master/docs/API.md

- gulp.task
- gulp.src
- gulp.dest
- gulp.watch

### gulp.task(name [, deps] [, fn])

作用：定义各种不同的任务

- gulp.task(name, fn)
- gulp.task(name, deps, fn)
- gulp.task(name, fn(cb))
- gulp.task(name, deps, fn(cb))

一：普通任务

```js
gulp.task('a', function () {
  console.log('1 aaa')
})

gulp.task('b', function () {
  console.log('2 bbb')
})
```

二：任务之间的依赖

```js
gulp.task('a', function (cb) {
  setTimeout(function () {
    console.log('1 aaa')
    cb()
  }, 1000)
})

// b 任务依赖的 a 任务中的回调函数如果不调用，b 任务是不会执行的
gulp.task('b', ['a'], function () {
  console.log('2 bbb')
})
```

三：gulp 流控制

```js
gulp.task('a', function () {
  // 当任务中是一个 gulp 流的时候则需要通过 return 来保证依赖中的执行顺序
  return gulp.src()
    .pipe()
    // ...
})

gulp.task('b', ['a'], function () {
  // doSomething
})
```

### gulp.src(globs[, options])

> gulp教程之gulp中文API：http://www.ydcss.com/archives/424

作用：根据路径（字符串或数组）读取需要构建的资源

#### globs

需要处理的源文件匹配符路径。

类型(必填)：String or StringArray，通配符路径匹配示例：

- `src/a.js` 指定具体文件；
- `*` 匹配所有文件    例：`src/*.js` (包含src下的所有js文件)；
- `**` 匹配0个或多个子文件夹    例：`src/**/*.js` (包含src的0个或多个子文件夹下的js文件)；
- `{}` 匹配多个属性    例：`src/{a,b}.js` (包含a.js和b.js文件)  src/*.{jpg,png,gif}(src下的所有jpg/png/gif文件)；
- `!` 排除文件    例：`!src/a.js` (不包含src下的a.js文件)；

#### options.base

options.base：类型：String  设置输出路径以某个路径的某个组成部分为基础向后拼接，具体看下面示例：

```js
gulp.src('client/js/**/*.js') 
  .pipe(minify())
  .pipe(gulp.dest('build'))  // Writes 'build/somedir/somefile.js'
 
gulp.src('client/js/**/*.js', { base: 'client' })
  .pipe(minify())
  .pipe(gulp.dest('build'))  // Writes 'build/js/somedir/somefile.js'
```

### gulp.dest(path[, options])

作用：构建任务完成后资源存放的路径

### gulp.watch(glob[, opts], tasks)

> 监视指定资源的改动，然后可以调用响应的任务处理

### gulp.watch(glob [, opts, cb])

---

## 常用插件

|          插件名称          |                        作用                        |
|----------------------------|----------------------------------------------------|
| del                        | 删除文件或文件夹                                   |
| gulp-less                  | 编译LESS文件                                       |
| gulp-rname                 | 重命名文件                                         |
| gulp-imagemin              | 图片压缩                                           |
| gulp-uglify                | 压缩Javascript                                     |
| gulp-concat                | 合并 js 文件                                       |
| gulp-concat-css            | 合并 css 文件                                      |
| gulp-cssnano               | 压缩 css                                           |
| gulp-htmlmin               | 压缩HTML                                           |
| gulp-rev                   | 添加版本号                                         |
| gulp-rev-collector         | 内容替换                                           |
| gulp-useref                | gulp-if                                            |
| gulp-load-plugins          | 依赖自动加载                                       |
| gulp-useref                | 自动合并打包处理                                   |
| gulp-wrap                  | 包装内容                                           |
| gulp-angular-templatecache | AngularJS 模板缓存                                 |
| browser-sync               | 和 gulp 配合使用实现文件改变执行某个任务后自动刷新 |
| yargs                      | 获取命令行参数                                     |
| gulp-if                    | 根据判断执行某个插件                               |
|                            |                                                    |


