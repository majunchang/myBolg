---
title: vue双向绑定原理
date: 2018-03-05 21:51:42
tags: vue
categories: vue
---
> 本文采用了比较特殊的input和v-model指令 实际上vue的指令解析模板很复杂，本文重点是理解数据更新的思想

### 几种实现双向绑定的做法
目前几种主流的mvc(vm)框架都实现了单向数据绑定，而我所理解的双向数据绑定无非就是在单向绑定的基础上给可输入元素（input、textare等）添加了change(input)事件，来动态修改model和 view，并没有多高深。所以无需太过介怀是实现的单向或双向绑定。
实现数据绑定的做法有大致如下几种：
> 发布者-订阅者模式（backbone.js）

> 脏值检查（angular.js）

> 数据劫持（vue.js）

#### 发布者-订阅者模式:
一般通过sub, pub的方式实现数据和视图的绑定监听，更新数据方式通常做法是 vm.set('property', value)，这里有篇文章讲的比较详细，有兴趣可点这里

这种方式现在毕竟太low了，我们更希望通过 vm.property = value 这种方式更新数据，同时自动更新视图，于是有了下面两种方式

#### 脏值检查:
angular.js 是通过脏值检测的方式比对数据是否有变更，来决定是否更新视图，最简单的方式就是通过 setInterval() 定时轮询检测数据变动，当然Google不会这么low，angular只有在指定的事件触发时进入脏值检测，大致如下：

DOM事件，譬如用户输入文本，点击按钮等。( ng-click )

XHR响应事件 ( $http )

浏览器Location变更事件 ( $location )

Timer事件( $timeout , $interval )

执行 $digest() 或 $apply()

#### 数据劫持:
vue.js 则是采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。

### 思路整理
1. 实现一个数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者
2. 实现一个指令解析器Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
3. 实现一个Watcher，作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图
4. 入口函数，整合以上三者


### 流程图
![mark](http://oneg19f80.bkt.clouddn.com/blog/20171222/164830111.png)


#### 数据监听器

```js
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

```

#### 实现Compile


```js
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
                })
                    ;
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

```

#### 增加watcher 观察函数

```js
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

```

#### 入口函数

```js
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
            text: 'hello world',
            name: '你好，全世界'
        }
    });
    vm.data.text = 'majunchang'


    document.getElementsByClassName('btn')[0].onclick = function () {
        vm.text = 'majunchang'
        vm.name = '又疑瑶台镜，飞在青云端'
    }
```


