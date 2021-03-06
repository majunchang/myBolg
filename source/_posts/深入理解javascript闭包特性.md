---
title: 深入理解javascript闭包特性
categories: javascript
abbrlink: b4ae21a4
---

## 闭包的原理
* 在函数外部使用函数内部的数据 
* 上级作用域无法访问下级作用域中的变量，但是下级作用域可以访问上级作用域的变量
```javascript
function outer(){
    var data = "";
    function inner(){
        return data;
    }
    return inner;
}

```

## 闭包的应用举例 

* 可以将数据保护起来，外接想要修改数据，只能通过指定的渠道
* 给函数添加一个私有的变量

## 浅谈javascript闭包的特性 

闭包，是指语法域位于某个特定的区域，具有持续参照（读写）位于该区域内自身范围之外的执行域上的非持久型变量值能力的段落。
Javascript闭包的定义非常晦涩——闭包，是指语法域位于某个特定的区域，具有持续参照（读写）位于该区域内自身范围之外的执行域上的非持久型变量值能力的段落。
简单来说，Javascript闭包就是在另一个作用域中保存了一份它从上一级函数或作用域取得的变量

```javascript
 function f1(){
        var n=999;
        nAdd=function(){n+=1}
        function f2(){
            alert(n);
        }
        return f2;
    }
    var result=f1();
    result(); // 999
    nAdd();
    result(); // 1000
```

## 使用闭包的注意点 
1. 闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
2. 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value）
，这时一定要小心，不要随便更改父函数内部变量的值
  
