---
title: 解析jquery的思想初步的封装final
categories: JQuery
abbrlink: 938bddae
date: 2016-11-01 20:24:01
tags:
---



##  jq 案例
    <div id="dv"></div>
    ...
 鼠标移入的时候有颜色高亮, 鼠标移开的时候颜色恢复
    $( '#dv' ).mouseenter(function () {
        $( this ).addClass...
    }).mouseleave(function () {
        $( this ). ...
    });
```javascript
    iQuery.select = function ( selector ) { 
        return slice.call( document.querySelectorAll( selector ) ); 
    }; 
    //这里直接return document.querySelectorAll( selector)行吗？一定要返回一个真数组吗？ 
    //构造函数中push.apply(this, iQuery.select(selector));传入伪数组也是可以的吧？

    var init = iQuery.prototype.init = function (selector) { 
        return [].push.apply(this, iQuery.select(selector)); 
    } 
    iQuery.select = function (selector) { 
        return [].slice.call(document.querySelectorAll(selector)); 
    } 
    为什么要分成两步，直接[].push.apply(this,document.querySelectorAll(selector))不就好了，
    为什么[].slice.call(document.querySelectorAll(selector))转换成真数组再
    [].push.apply(this, iQuery.select(selector))
    

```

核心结构中
        iQuery.fn = iQuery.prototype = {}; 
        iQuery.prototype.init.prototype = iQuery.fn; 
    但是这两句连写成iQuery.prototype.init.prototype = iQuery.fn = iQuery.prototype = {}; 
    会报错Cannot set property 'prototype' of undefined。是因为赋值的先后顺序。
    
疑问：
    iQuery.select = function(selector){
        slice.call(document.querySelectorAll(selector));
    }
    用来获取dom对象, 
    var init = iQuery.fn.init = function(){
        push.apply(this,iQuery);
    }
    用来初始化函数,是不是多此一举?为什么不直接: 
    var init = iQuery.fn.init = function(){
        slice.apply(this,document.querySelectorAll(selector));
    }
  





















