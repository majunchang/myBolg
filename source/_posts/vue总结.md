---
title: vue总结
date: 2018-03-04 21:31:58
tags:
- vue
categories: vue
---
## vue使用总结心得

### vue的安装

###### 在这里我们主要针对的是vue的单页面项目 如果仅仅是为了单个案例可以直接下载 然后script安装

> Vue 提供一个官方命令行工具，可用于快速搭建大型单页应用。该工具为现代化的前端开发工作流提供了开箱即用的构建配置。只需几分钟即可创建并启动一个带热重载、保存时静态检查以及可用于生产环境的构建配置的项目：

```
# 全局安装 vue-cli
$ npm install --global vue-cli
# 创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
# 安装依赖，走你
$ cd my-project
$ npm install
$ npm run dev
```

### vue的生命周期

#### vue的生命周期图示

![mark](http://upload-images.jianshu.io/upload_images/5703029-2053cda58f31f3e9..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

| header 1      | header 2                          |
| ------------- | --------------------------------- |
| beforeCreate  | 实例刚刚被创建 el和data并未初始化              |
| created       | 实例创建完成 data被初始化 但是el没有被初始化 dom不存在 |
| beforeMount   | 完成了el的初始化 模板编译之前                  |
| mounted       | 模板编译之后 完成挂载                       |
| beforeUpdate  | 组件更新之前                            |
| uodated       | 组件更新之后                            |
| beforeDestory | 组件销毁前调用                           |
| destoryed     | 组件销毁后调用                           |

### vue的核心思想（数据绑定和组件化）

- ==vue的双向数据绑定==

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <input type="text" v-model="text">
    {{ text }}

</div>
</body>
</html>
<script>
    /*
      1 数据监听器 首先有一个数据监听的函数 目的是 监听数据的变化 拿到最新值 并通知订阅者
      2 指令解析器 有一个编译的函数 对元素的节点进行扫描和解析 并绑定相关的更新函数
      3 watcher 作为连接observe和compile的敲了
      3 订阅者 这个订阅者 负责与watcher 配合收到属性变动的通知，执行相应的回调函数 完成视图的更新     */
    function observe(obj, vm) {
        //  对传入的对象 遍历 并分别添加 object.defineProperty
        Object.keys(obj).forEach((key) => {
            defineReactive(vm, key, obj[key])
        })
    }

    function defineReactive(vm, key, val) {
        var dep = new Dep();
        Object.defineProperty(vm, key, {
            get: function () {
                // 通过这一步 添加订阅者
                if (Dep.target) dep.addSub(Dep.target)
                return val;
            },
            set: function (newval) {
                if (newval === val) return
                val = newval;
                // 通知订阅者
                dep.notify()
            }
        })
    }

    // 需要实现一个消息订阅器
    function Dep() {
        // 消息订阅的让容器是一个数组 数组的每一项 都是指代一个view和mode的中间者
        this.subs = []
    }

    Dep.prototype = {
        addSub: function (sub) {
            this.subs.push(sub)
        },
        notify: function () {
            this.subs.forEach((sub) => {
                // 在这里 需要配合watcher进行更新
                sub.update()
            })
        }
    }

    //  在这里增加dom编译模板
    function nodeToFragment(node, vm) {
        var flag = document.createDocumentFragment();
        var child;
        while (child = node.firstChild) {
            compile(child, vm);
            // 将子节点劫持到文本节点中
            flag.appendChild(child)
        }
        return flag
    }

    function compile(node, vm) {
        var reg = /\{\{(.*)\}\}/;
        // 跟据节点类型去判断
        if (node.nodeType === 1) {
            var attr = node.attributes;
            for (var i = 0; i < attr.length; i++) {
                if (attr[i].nodeName === 'v-model') {
                    // 此时 name为text
                    var name = attr[i].nodeValue;
                    // 增加数据的变化监听
                    node.addEventListener('input', (e) => {
                        vm[name] = e.target.value;
                    });
                    // 在这里 因为 我们的数据监听器 已经封装了vm[name]  触发了 getter方法 完成了数据的初始化
                    node.value = vm[name];
                    node.removeAttribute('v-model')
                }
            }
            new Watcher(vm, node, name, 'input')
        }
        if (node.nodeType === 3) {
            if (reg.test(node.nodeValue)) {
                var name = RegExp.$1;
                name = name.trim();
                new Watcher(vm, node, name, 'text')
            }
        }
    }

    //订阅者 搭建数据监听变化和变异模板的桥梁
    function Watcher(vm, node, name, nodeType) {
        Dep.target = this;
        this.vm = vm;
        this.node = node;
        this.name = name;
        this.nodeType = nodeType
        this.update()
        Dep.target = null;
    }

    Watcher.prototype = {
        update: function () {
            this.get()
            if (this.nodeType === 'text') {
                this.node.nodeValue = this.value
            }
            if (this.nodeType === 'input') {
                this.node.value = this.value
            }
        },
        get: function () {
            this.value = this.vm[this.name];
        }
    }


    function Vue(options) {
        // 将options里面的data属性 放入数据监听器
        this.data = options.data;
        var data = this.data;
        observe(data, this); // this指代vm
        // 对指定id的dom 进行页面的渲染
        this.$el = options.el;
        var id = this.$el;
        var Dom = nodeToFragment(document.getElementById(id), this);
        // 编译完成之后 将dom 添加到节点中
        document.getElementById(id).appendChild(Dom)
    }

    var vm = new Vue({
        el: 'app',
        data: {
            text: 'hello,world'
        }
    })

    vm.text = 'ma'

</script>
```

![mark](http://upload-images.jianshu.io/upload_images/5703029-69cf49b5bc385b2d..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 组件化思想

1. 将实现页面的某一部分功能的结构，样式和逻辑封装为一个整体，使其高内聚，低耦合，达到分治和复用的目的

   ```
    为了提高代码复用性，减少重复性的开发，我们就把相关的代码按照 template、style、script 拆分，封装成一个个的组件。组件可以扩展
   ```

   HTML 元素，封装可重用的 HTML 代码，我们可以将组件看作自定义的 HTML 元素。在 Vue 里面，每个封装好的组件可以看成一个个的 ViewModel。

2. 组件的执行顺序

- 子组件先在父组件中的 components 中进行注册。
- 父组件利用 Vue.component 注册到全局。
- 当渲染父组件的时候，渲染到 <child-component> ，会把子组件也渲染出来。

### vue的路由分发（vue-router）

> #### vue的路由分发主要是使用vue-router 本质来说 使用了哈希路径和浏览器的history（html5新增api）

#### vue-router的安装和项目中的配置

```javascript
npm install vue-router --save
```

[vue-router的官网](https://router.vuejs.org/zh-cn/installation.html)

```javascript
//  在main.js中这样配置  
import Vue from 'vue'
import App from './App'
import router from './router'

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  template: '<App/>',
  components: {App},
  methods: {
    
  },
  created() {
    
  },
})

//  在router文件夹下的index.js中 这样配置  

import Vue from 'vue';
import Router from 'vue-router';
import dashboard from '../pages/dashboard/dashboard.vue';


import index from '../pages/index.vue';
//系统设置页面
import systemSetting from '../components/systemSetting/systemSetting.vue'
// 引入登录页
import login from '../pages/auth/login/login.vue'
//引入注册页
import register from '../pages/auth/register/register.vue'

Vue.use(Router)

var router = new Router({
  routes: [
    {
      path: '/login',
      name: 'login',
      component: superTubeLogin,
    },
    {
      path: '/register',
      name: 'register',
      component: register
    },
   
    {
      path: '/',
      name: 'index',
      component: index,
      children: [
        {
          path: '/dashboard',
          name: 'dashboard',
          component: dashboard,
        },
        //系统设置的页面
        {
          path: '/systemSetting',
          name: 'systemSetting',
          component: systemSetting,
        },
        //工作组

        //授权用户管理
        {
          path: '/boxUserManageAllow',
          name: 'boxUserManageAllow',
          component: boxUserManageAllow
        },
      ]
    },
    {
      path: '*',
      component: notFoundComponent
    }
  ]
})


router.beforeEach((to, from, next) => {
  // 这里写的逻辑 任何路由跳转之前都会执行
})


export default router



```



#### 我们可以使用vue-router做那些事

1. 配置路由分发

2. 设置路由重定向

   - 典型的应用场景有 做登录前的禁止跳转
   - 在用户访问不存在的页面的时候 跳转到自定义的404页面

3.  在组件中进行路由的跳转 

4. 进行组件之间的传参

   ```javascript
   const router = new VueRouter({
     routes: [
       {
         // 
         path: '/user/:userId',
         name: 'user',
         component: User
       }
     ]
   })

   // 在组件中 应该这样写 router-link 在渲染时 会被转化为a标签
   <router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
   // 在vue的生命周期 或者 methods中 应该这样写  
   router.push({ name: 'user', params: { userId: 123 }})

   // 关于如何使用query进行传递参数  这里给出了一个示例
   // http://blog.csdn.net/k491022087/article/details/70214664
   ```

   > 向路由组件传递props  
   >
   > https://router.vuejs.org/zh-cn/essentials/passing-props.html

5. 根据路由元信息 设置组件的初始化或者区别组件

6. 设置过渡的动态效果

7. 路由信息对象

   https://router.vuejs.org/zh-cn/api/route-object.html

#### vue-router的使用注意事项

- 在组件中跳转的时候 和 获取路由元信息的时候 

  ```javascript
  //  组件中跳转
  this.$router.push({
            name: page,
            params: {id: 0, type: page, content: item.content, template: item.template}
   })
   
   //  获取路由元信息 
  this.$route.params.apiId
   
  ```

  ​

- 监听路由变化

  ```javascript
  wacth:{
    '$route'(to, from) {
         .....
        },
  }
  ```

### vue的复杂存储（vuex）


### vue通信

### 项目中遇到的比较通用的问题（易错点）

#### 1  如何阻止冒泡和事件的默认行为

#### 2  多层数据的嵌套以及更改vue遍历好的数组之后 如何进行实时显示

#### 3  如果遇到跨域问题 如何解决

#### 4 如何减少文件压缩体积

#### 5 如何进行文件的打包

