---
title: 解析jquery的思想初步的封装2
date: 2016-09-25 21:24:02
tags:
categories: JQuery
---
## 1. 自己封装一个函数完成数组遍历, 要求模拟数组的 forEach 方法
  

1.函数中如果没有代码 数组的每一项就什么也不做
2.函数中写什么代码 数组中的每一项就做什么事情
3.也就是说函数在处理数组的每一项
由于要处理每一项, 因此必须要遍历数组的每一个元素
使用函数处理每一项就是用函数处理 item. 就是说将 item 传入函数处理
```javascript
   function each( arr, callback ) {
        for ( var i= 0; i < arr.length; i++ ) {
            var item = arr[ i ];
            callback( item );
        }
    }

```
按照 数组的 forEach 方法实现 自己的 each 方法
要求 有两个参数, 一个是需要遍历的数组, 一个是回调函数, 要求回调函数有两个参数, 分别是 v, i
```javascript
   function each( arr, callback ) {
        for ( var i = 0; i < arr.length; i++ ) {
            callback( arr[ i ], i );
        }
    }
```
*    改良第一步, 添加 this 的引用, 参数向 jq 靠拢

``` javascript  
function each( arr, callback ) {
        for ( var i = 0; i < arr.length; i++ ) {
            callback.call( arr[ i ], i, arr[ i ] );
        }
    }

```

* 第二次改良, 在返回 false 的时候跳出循环, 注意, 返回 undefined, 0, 等不会跳出
* 所以这里一定有一个完全等于, 跳出循环就是跳出 each 函数内的 for 即需要 break

```javascript
   function each( arr, callback ) {
        for ( var i = 0; i < arr.length; i++ ) {
            if ( callback.call( arr[ i ], i, arr[ i ] ) === false ) break;
        }
    }

```
* 第三个改良就是返回被遍历的对象

```javascript
   function each( arr, callback ) {
        for ( var i = 0; i < arr.length; i++ ) {
            if ( callback.call( arr[ i ], i, arr[ i ] ) === false ) break;
        }
        return arr;
    }

```

#### 遍历对象与遍历数组 有什么区别?

1. 遍历对象是将对象的所有键值对遍历出来( 包括自然数项, 如果有的话 )
2. 遍历数组就是在遍历自然数项
3. 判断数组或伪数组的依据就是 length >= 0

``` javascript
    function each( arr, callback ) {
        if ( arr.length >= 0 ) {
            for ( var i = 0; i < arr.length; i++ ) {
                if ( callback.call( arr[ i ], i, arr[ i ] ) === false ) break;
            }
        } else {
            for ( var k in arr ) {
                if ( callback.call( arr[ k ], k, arr[ k ] ) === false ) break;
            }
        }
        return arr;
    }

```
##  2 封装 map

首先考虑, 只是处理 v 返回 结果( 实现一个 与 数组 的 map 一样的方法 )
    function map( arr, callback ) {
       var rest = [];
        for ( var i = 0; i < arr.length; i++ ) {
            // 要返回的数据是 回调函数的 返回值
            rest.push( callback( arr[ i ], i ) );
        }
        return rest;
    }
    // 向 jq 靠拢
    // 1> 不返回则不加到数组中
    // 2> 遍历对象

```javascript
    function map( arr, callback ) {
        var rest = [], tmp;
        if ( arr.length >= 0 ) {
            for ( var i = 0; i < arr.length; i++ ) {
                tmp = callback( arr[ i ], i );
                if ( tmp != null ) {
                    rest.push( tmp );
                }
            }
        } else {
            for ( var k in arr ) {
                tmp = callback( arr[ k ], k );
                if ( tmp != null ) {
                    rest.push( tmp );
                }
            }
        }
        return rest;
    }

```

## 3. 框架的核心结构

需求:
    1> 有一个函数, 可以实现元素的获取功能( select -> Array )
    2> 这个函数返回的对象具有 each 方法. 该方法只有一个 回调函数的参数, 用于遍历每一个对象.

先把函数写出来
    function I( selector ) {

    } 
该函数可以获取元素
    function I( selector ) {
        var list = document.querySelectorAll( selector ); // 伪数组
    }
    返回的数据应该就是这个 list, 但是又需要有 each 
    所以可以考虑直接让 list 带有 each 方法
```javascript

    function I( selector ) {
        var list = document.querySelectorAll( selector ); // 伪数组
        list.each = function ( callback ) {
            // 调用 该方法就是在 遍历 list 并将每一个元素用 callback 进行处理
            each( this, callback );
        }
        return list
    }

```
### 4.引入构造函数( init )

知道 list 是一个有 qsa 方法返回的对象, 每次提供 each 方法不合理, 而且 jq 有很多方法
所以借助 oop 的知识应该使用 继承. 利用构造函数来创建 "list 对象", 但是继承自提供 
each 等方法的原型对象.
```javascript
    function F( selector ) {
        // 给 this 加 伪数组项
    }
    F.prototype.each = function () {

    } 
```
* 考虑到 list 中每次都添加方法不合理, 而且冗余, 因此使用构造函数创建对象( 伪数组 ).
* 让构造函数.prototype 提供各种方法, 那么实例对象( 伪数组 )就可以使用这些方法了.

```javascript
    function F( selector ) {
        // 给 this 加 伪数组项
        var list = document.querySelectorAll( selector );
        for( var i = 0; i < list.length; i++ ) {
            this[ i ] = list[ i ];
        }
        this.length = list.length;
    }
    F.prototype.each = function ( callback ) {
        each( this, callback );
    };

    升级一下
    function F( selector ) {
        // 给 this 加 伪数组项
        [].push.apply( this, document.querySelectorAll( selector ) );
    }
    F.prototype.each = function ( callback ) {
        each( this, callback );
    };

    如何使用呢?
    new F( 'div, p' ).each( ... )   

```

##  5.隐藏 new 关键字
1. 考虑引入函数
    function iQuery( selector ) {
        return new F( selector );
    }
    var I = iQuery;

2. 引入沙箱
    从沙箱中如果要暴露一些函数或对象采用方法有三种( 常用 )
    1> 利用 window 参数
        (function ( window ) {

            window.xxx = vvv;
        })( window );
    2> 利用 返回值( 例如 Sizzle )
        var Sizzle =
        (function () {

            return xxx;
        })();
    3> 利用 this 映射
        (function () {
            this.xxx = vvv;
        })();

3. 缺点:
    1> 代码结构凌乱. 优化结构.
    2> 无法实现扩展, 只有在该文件中实现代码结构.

##  6. 可扩展性

``` javascript
    1> 可以将 iQuery 的一个属性指向构造函数
        var iQuery = function ( selector ) {
            return new F( selector );
        }; 

        ...
        iQuery.F = F;
        ...

        此时对外的扩展
        iQuery.F.prototype.xxx = vvv;

    2> 在 jq 中采用是原型映射的办法, jq 中将 对外公开的 jquery 函数 与 构造函数 的原型用一个对象表示
        // 类比
        F.prototype = {};
        iQuery.prototype = F.prototype;

        //===============================================================

        var jQuery = function ( selector ) {
            return ...( selector );
        };

        jQuery.prototype = {
            // 对象
            constructor: jQuery,
            each: function () ....
        };  

        var init = function ( selector ) {
            // ... 构造函数 ...
            // [].push.apply( this, document.querySelectorAll( selector ) );
        };

        init.prototype = jQuery.prototype;

        //===============================================================

        // 在 jq 中又 进一步处理
        // 将 init 函数作为 jquery 的原型的一个方法
        // 将所有的方法挂载到 jQuery 上, 便于优化与更新

        // jquery 1.7 以后的写法
        var jQuery = function ( selector ) {
            return new jQuery.fn.init( selector );
        };

        jQuery.fn = jQuery.prototype = {
            // 对象
            constructor: jQuery
        };  

        var init = jQuery.fn.init = function ( selector ) {
            // ...
        };

        init.prototype = jQuery.fn;
        等价的写法
        // jquery 1.7 以前的写法
        var jQuery = function ( selector ) {
            return new jQuery.fn.init( selector );
        };

        jQuery.fn = jQuery.prototype = {
            // 对象
            constructor: jQuery,
            init: function ( selector ) {
                // ...
            }
        };  

        jQuery.fn.init.prototype = jQuery.fn;

```
