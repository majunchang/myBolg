---
title: 解析jquery的思想初步的封装3
tags: JQuery
categories: JQuery
abbrlink: 1aa46f22
date: 2016-10-10 09:10:48
---

### 对juqery核心结构的提炼以及封装完毕的map和each方法

1. jq中的forEach和map方法 和 原生js中的有所不同，功能更加完善。
```javascript
(function ( window ) {

var arr = [],
    push = arr.push,
    slice = arr.slice;

var iQuery = function ( selector ) {
    return new iQuery.fn.init( selector );
}

iQuery.fn = iQuery.prototype = {
    constructor: iQuery,
    each: function ( callback ) {
        return iQuery.each( this, callback );
    },
    map: function ( callback ) {
        return iQuery.map( this, callback );
    }
};


// 将所有的方法应该挂载到 iQuery 中
iQuery.select = function ( selector ) {
    return slice.call( document.querySelectorAll( selector ) );
};


// 工具方法 each map
iQuery.each = function ( arr, callback ) {
    var i;
    if ( arr.length >= 0 ) {
        for ( i = 0; i < arr.length; i++ ) {
            if ( callback.call( arr[ i ], i, arr[ i ] ) === false )  break;
        }
    } else {
        for ( i in arr ) {
            if ( callback.call( arr[ i ], i, arr[ i ] ) === false )  break;
        }
    }
    return arr;
};

iQuery.map = function ( arr, callback ) {
    var rest = [], tmp,
        i;
    if ( arr.length >= 0 ) {
        for ( i = 0; i < arr.length; i++ ) {
            tmp = callback( arr[ i ], i );
            if ( tmp != null )  {
                rest.push( tmp );
            }
        }
    } else {
        for ( i in arr ) {
            tmp = callback( arr[ i ], i );
            if ( tmp != null )  {
                rest.push( tmp );
            }
        }
    }
    return rest;
};

// 扩展能力
iQuery.extend = iQuery.fn.extend = function ( options ) {
    for ( var k in options ) {
        this[ k ] = options[ k ];
    }
};

window.iQuery = window.I = iQuery;

})( window );

```

##  框架的 扩展

所谓的扩展就是给原型或构造方法( iQuery ) 提供新的成员
    就是在给一个对象增加新的成员
    可以使用混合式. 其思想就是将几个对象的成员混合到一起
    extend 扩展

``` javascript
    // 对原型的处理
    iQuery.fn.extend = function ( options ) {
        for ( var k in options ) {
            this[ k ] = options[ k ];
        }
    }

    // 对 iQuery 处理
    iQuery.extend = function( options ) {
        for ( var k in options ) {
            this[ k ] = options[ k ];
        }
    } 
    iQuery.extend = iQuery.fn.extend = function ( options ) {
        for ( var k in options ) {
            this[ k ] = options[ k ];
        }
    };

```
将来在扩展功能的时候怎么用呢?
    1) 扩展实例方法
        iQuery.fn.extend({
            method: function ...
        });
    2) 扩展静态方法
        iQuery.extend( ... )
    
##. DOM 操作
    appendTo
    append
    prependTo
    prepend 
    insertBefore
    insertAfter
    before
    parent
    next
    nextAll
    prev
    prevAll
    siblings
    parseHTML   

* 产生了一个问题
    所有的 dom 元素只允许有一个父元素. 因此将一个 dom 元素
    追加到另一个元素里时, 会自动的从原有的父元素中移除.

    <div>
        <div>123</div>
        <div>456</div>
    </div>
    将 第 0 个 div 追加到 body 上的时候, 会从原有的位置移除
    <div>
        <div>456</div>
    </div>
    此时 childNodes 的长度也会自动的 -1, 原来排在第 1 号
    位置的 div 也会变成 第 0 号.

## 将构造函数进一步的完善
修改构造函数, 判断一下, 如果是 字符串结构, 就采用 parseHTML
    如果不是字符串结构就用 qsa 方法

    在 jq 构造函数中, 参数可以是很多东西
        1) 字符串: 1>选择器, 2>HTML 字符串
        2) DOM 元素
        3) 函数: 相当于 onload 事件
        4) jquery 元素
            $( '<div />' ).appendTo( $( 'body' ) );
            $( '<div />' ).appendTo( 'body' );
        5) 可以是 空
        ...
    在 init 构造函数中准备一套 if else 结构
    处理每一个情况

``` javascript
var init = iQuery.fn.init = function ( selector ) {
   
    // 处理: I( '' ), I( null ), I( undefiend ) 等
    if ( !selector ) return this;
    
    // 处理: 字符串
    if ( typeof selector == 'string' ) {
        // 处理多种
        //  \s*<.+>\s*
        if ( rquickExpr.test( selector ) ) {
            // HTML 格式的字符串
            push.apply( this, iQuery.parseHTML( selector ) );
        } else {
            // 选择器字符串
            push.apply( this, iQuery.select( selector ) );
        }
        return this;
    }

    // 处理函数
    if ( typeof selector == 'function' ) {

    }

    // 处理 DOM 元素
    if ( selector.nodeType ) {
        this[ 0 ] = selector;
        this.length = 1;
        return this;
    }

    // 处理 iQuery 对象
    if ( selector.type === 'iQuery' ) {
        // return selector;
        // 推荐的处理办法是将 selector 中的所有元素加到 this 中
        push.apply( this, selector );
        return this;
    }

    if ( selector.length >= 0 ) {
        push.apply( this, selector );
    } else {
        this[ 0 ] = selector;
        this.length = 1;
    }
    return this;
};

init.prototype = iQuery.fn;

```

    
