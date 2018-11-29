---
title: 对于jquery思想的理解初步的封装1
categories: JQuery
abbrlink: e13820e0
date: 2016-09-20 07:50:11
---
1. 本篇博客主要研究一下几个分类
    -> 选择模块
    -> 框架的核心结构
    -> DOM 操作
    -> 事件与属性和样式
    -> 插件机制
    -> 工具

2. 引入问题
    将下列 div 与 p 标签添加 1px solid red 样式
   ``` 

   <div>div</div>
    <span>span</span>
    <p>P</p>
    <span>span</span>
    <div>div</div>
    <span>span</span>
    <p>P</p>
    <span>span</span>
    <div>div</div>
    <span>span</span>
    <p>P</p>
    <span>span</span>
    <div>div</div>
    <span>span</span>
    <p>P</p>
    <span>span</span>
    <div>div</div>
    <span>span</span>
    
    ```

3. DOM 操作的要求和DOM 常用属性
     学会分析 DOM 树
        绘制 DOM 树( 标准 代码需要与图对象 )
     增删改查的方法
        查:
            document.getElementById()
            document.getElementsByTagName()
            document.getELementsByClassName() 
            document.getElementsByName()
            document.querySelector()
            document.querySelectorAll()
    访问亲属节点深度为 2 级
        如何访问父节点
        如果访问兄弟节点
        如何访问子节点
    DOM常用属性：
    .nodeName       标签名, 都是大写
    .nodeType       获得节点的类型: 1, 元素; 2, 属性; 3, 文本
    .nodeValue      常常用于文本节点, 表示文本的内容

4. 详解几个知识点：jq中会使用到的
    -> NodeList
        该数据就是一个伪数组的数据类型
        HTMLCollection
        NodeList
        用于描述一个集合数据类型( 伪数组 )

    ``` slice.call
        问
            [].slice.call
            Array.prototype.slice.call
            rest.slice.call
            有什么区别?
        
        var o = {
            method: function () {}
        };

        o.method();
        o.method.call( ... )

        var f = o.method;
        f.call( ... );
    ```

    apply call 参数
        1. 概念
            [].push.apply( rest, list );
            相当于 rest.push( "list" )

            [].push.call( rest, list );
            相当于 rest.push( "list" )
        2. call 与 apply 的参数
            第一个参数:
                如果是基本类型( string, number, boolean ) 会转换成包装类型( String, Number, Boolean )
                如果是引用类型( 非空 ), this 就是它( 相当于方法调用 )
                如果是空( null 与 undefined ) 那么 this 就是 window
             第二个+参数
                apply 只有两个参数, 第二个参数一定是一个数组( 伪数组 )
                    func.apply( obj, [ a, b, c, ..., n ] )
                    等价于
                    func( a, b, c, ..., n )
                
                call 将参数散列处理, 函数调用时怎么传参, call 就怎么传参
                    func.call( obj, a, b, c, ..., n )
                    等价于
                    func( a, b, c, ..., n )
            3. 既然有 call 为何还要有 apply
                ```
                var arr = [ 1, 2 ];
                arr.push( 3, 4, 5, 6, 7 );

                var weiArray = { 0:1, 1:2, 2:3, length: 3 };
                var zhenArray = [ 1, 2 ];

                [].push.call( zhenArray, weiArray );
                =>
                zhenArray.push( weiArray );

                [].push.apply( zhenArray, weiArray );
                =>
                zhenArray.push( ...weiArray )
                ```
    关联数组的用法
    ```
        var o = { name:'jim', age: 19, gender: '男' };

        // 点语法访问( 硬编码 )
        // o.name

        // 如果想要提供一个功能, 用户输入什么就访问什么
        if ( input === 'name' ) {
            o.name ....
        }
        else if ( input === 'age' ) {
            o.age ....
        }
        else if ( input === 'gender' ) {
            o.gender ....
        }
        ...
        ...
    ```
        // js 允许像使用数组一样使用对象, 只需要提供 属性的名字即可
        // o[ 名字 ]
        o[ input ]
        =>
        o[ 'name' ]
        o[ 'age' ]
        o[ 'gender' ]
        ...
        ...

5. 扩展DOM-Core 与 HTML DOM
    在 实际开发中, 有很多数据使用 XML 格式来进行表示的( win 新的技术( WPF ), 安卓界面, 苹果的界面, ... )
    在处理这数据的时候 通用方法称为 核心 DOM 方法

    ```<div>111</div><div>2<div>111</div>2<div>111</div>2</div>```

    在实际应用 HTML 飞速发展, 因此在处理 HTML 的时候给出了一些快速处理的 api, 这些 api 就称为 HTML-DOM

    HTML-DOM, 首先是 DOM 方法, 适用于 HTML 文档

    给 body 标签增加一个 itcast 属性


思考题:
    为什么 forEach 中的回调函数有第三个参数
    jq 中 map 与 each 中回调函数的参数为什么不同



