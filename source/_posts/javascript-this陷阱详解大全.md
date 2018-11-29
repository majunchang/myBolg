---
title: javascript-this陷阱详解大全
tags: javascript
categories: javascript
abbrlink: 384a3b7a
date: 2017-06-05 16:55:10
---
　JavaScript来自一门健全的语言，所以你可能觉得JavaScript中的this和其他面向对象的语言如java的this一样，是指存储在实例属性中的值。事实并非如此，在JavaScript中，最好把this当成哈利波特中的博格特的背包，有着深不可测的魔力。

　　下面的部分是我希望我的同事在使用JavaScript的this的时候应当知道的。内容很多，是我学习好几年总结出来的。

　　JavaScript中很多时候会用到this，下面详细介绍每一种情况。在这里我想首先介绍一下宿主环境这个概念。一门语言在运行的时候，需要一个环境，叫做宿主环境。对于JavaScript，宿主环境最常见的是web浏览器，浏览器提供了一个JavaScript运行的环境，这个环境里面，需要提供一些接口，好让JavaScript引擎能够和宿主环境对接。JavaScript引擎才是真正执行JavaScript代码的地方，常见的引擎有V8(目前最快JavaScript引擎、Google生产)、JavaScript core。JavaScript引擎主要做了下面几件事情：
一套与宿主环境相联系的规则;
JavaScript引擎内核（基本语法规范、逻辑、命令和算法);
一组内置对象和API;
其他约定。
但是环境不是唯一的，也就是JavaScript不仅仅能够在浏览器里面跑，也能在其他提供了宿主环境的程序里面跑，最常见的就是nodejs。同样作为一个宿主环境，nodejs也有自己的JavaScript引擎--V8。根据官方的定义:
Node.js is a platform built on Chrome’s JavaScript runtime for easily building fast, scalable network applications
### global this
在浏览器里，在全局范围内，this等价于window对象。
``` javascript
1 <script type="text/javascript">
2     console.log(this === window); //true
3 </script>

```
在浏览器里，在全局范围内，用var声明一个变量和给this或者window添加属性是等价的。
1 <script type="text/javascript">
2     var foo = "bar";
3     console.log(this.foo); //logs "bar"
4     console.log(window.foo); //logs "bar"
5 </script>
如果你在声明一个变量的时候没有使用var或者let(ECMAScript 6),你就是在给全局的this添加或者改变属性值。
 1 <script type="text/javascript">
 2     foo = "bar";
 3 
 4     function testThis() {
 5       foo = "foo";
 6     }
 7 
 8     console.log(this.foo); //logs "bar"
 9     testThis();
10     console.log(this.foo); //logs "foo"
11 </script>
在node环境里，如果使用REPL(Read-Eval-Print Loop，简称REPL:读取-求值-输出,是一个简单的，交互式的编程环境)来执行程序,this并不是最高级的命名空间，最高级的是global.
> this
{ ArrayBuffer: [Function: ArrayBuffer],
  Int8Array: { [Function: Int8Array] BYTES_PER_ELEMENT: 1 },
  Uint8Array: { [Function: Uint8Array] BYTES_PER_ELEMENT: 1 },
  ...
> global === this
true
在node环境里，如果执行一个js脚本，在全局范围内，this以一个空对象开始作为最高级的命名空间，这个时候，它和global不是等价的。
 1 test.js脚本内容：
 2 
 3 console.log(this);
 4 console.log(this === global);
 5 
 6 REPL运行脚本：
 7 
 8 $ node test.js
 9 {}
10 false
在node环境里，在全局范围内，如果你用REPL执行一个脚本文件，用var声明一个变量并不会和在浏览器里面一样将这个变量添加给this。
1 test.js:
2 
3 var foo = "bar";
4 console.log(this.foo);
5 
6 $ node test.js
7 undefined
但是如果你不是用REPL执行脚本文件，而是直接执行代码，结果和在浏览器里面是一样的(神坑)
1 > var foo = "bar";
2 > this.foo
3 bar
4 > global.foo
5 bar
在node环境里，用REPL运行脚本文件的时候，如果在声明变量的时候没有使用var或者let，这个变量会自动添加到global对象，但是不会自动添加给this对象。如果是直接执行代码，则会同时添加给global和this
1 test.js
2 
3 foo = "bar";
4 console.log(this.foo);
5 console.log(global.foo);
6 
7 $ node test.js
8 undefined
9 bar
上面的八种情况可能大家已经绕晕了，总结起来就是：在浏览器里面this是老大，它等价于window对象，如果你声明一些全局变量(不管在任何地方)，这些变量都会作为this的属性。在node里面，有两种执行JavaScript代码的方式，一种是直接执行写好的JavaScript文件，另外一种是直接在里面执行一行行代码。对于直接运行一行行JavaScript代码的方式，global才是老大，this和它是等价的。在这种情况下，和浏览器比较相似，也就是声明一些全局变量会自动添加给老大global，顺带也会添加给this。但是在node里面直接脚本文件就不一样了，你声明的全局变量不会自动添加到this，但是会添加到global对象。所以相同点是，在全局范围内，全局变量终究是属于老大的。
function this
无论是在浏览器环境还是node环境， 除了在DOM事件处理程序里或者给出了thisArg(接下来会讲到)外，如果不是用new调用，在函数里面使用this都是指代全局范围的this。
 1 <script type="text/javascript">
 2     foo = "bar";
 3 
 4     function testThis() {
 5       this.foo = "foo";
 6     }
 7 
 8     console.log(this.foo); //logs "bar"
 9     testThis();
10     console.log(this.foo); //logs "foo"
11 </script>
test.js

foo = "bar";

function testThis () {
  this.foo = "foo";
}

console.log(global.foo);
testThis();
console.log(global.foo);
$ node test.js
bar
foo
除非你使用严格模式，这时候this就会变成undefined。
 1 <script type="text/javascript">
 2     foo = "bar";
 3 
 4     function testThis() {
 5       "use strict";
 6       this.foo = "foo";
 7     }
 8 
 9     console.log(this.foo); //logs "bar"
10     testThis();  //Uncaught TypeError: Cannot set property 'foo' of undefined 
11 </script>
如果你在调用函数的时候在前面使用了new，this就会变成一个新的值，和global的this脱离干系。
 1 <script type="text/javascript">
 2     foo = "bar";
 3 
 4     function testThis() {
 5       this.foo = "foo";
 6     }
 7 
 8     console.log(this.foo); //logs "bar"
 9     new testThis();
10     console.log(this.foo); //logs "bar"
11 
12     console.log(new testThis().foo); //logs "foo"
13 </script>
我更喜欢把新的值称作一个实例。

函数里面的this其实相对比较好理解，如果我们在一个函数里面使用this，需要注意的就是我们调用函数的方式，如果是正常的方式调用函数，this指代全局的this，如果我们加一个new，这个函数就变成了一个构造函数，我们就创建了一个实例，this指代这个实例，这个和其他面向对象的语言很像。另外，写JavaScript很常做的一件事就是绑定事件处理程序，也就是诸如button.addEventListener(‘click’, fn, false)之类的，如果在fn里面需要使用this，this指代事件处理程序对应的对象，也就是button。
prototype this
你创建的每一个函数都是函数对象。它们会自动获得一个特殊的属性prototype，你可以给这个属性赋值。当你用new的方式调用一个函数的时候，你就能通过this访问你给prototype赋的值了。
1 function Thing() {
2       console.log(this.foo);
3 }
4 
5 Thing.prototype.foo = "bar";
6 
7 var thing = new Thing(); //logs "bar"
8 console.log(thing.foo);  //logs "bar"
当你使用new为你的函数创建多个实例的时候，这些实例会共享你给prototype设定的值。对于下面的例子，当你调用this.foo的时候，都会返回相同的值，除非你在某个实例里面重写了自己的this.foo
复制代码

 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () {
 5     console.log(this.foo);
 6 }
 7 Thing.prototype.setFoo = function (newFoo) {
 8     this.foo = newFoo;
 9 }
10 
11 var thing1 = new Thing();
12 var thing2 = new Thing();
13 
14 thing1.logFoo(); //logs "bar"
15 thing2.logFoo(); //logs "bar"
16 
17 thing1.setFoo("foo");
18 thing1.logFoo(); //logs "foo";
19 thing2.logFoo(); //logs "bar";
20 
21 thing2.foo = "foobar";
22 thing1.logFoo(); //logs "foo";
23 thing2.logFoo(); //logs "foobar";
实例里面的this是一个特殊的对象。你可以把this想成一种获取prototype的值的一种方式。当你在一个实例里面直接给this添加属性的时候，会隐藏prototype中与之同名的属性。如果你想访问prototype中的这个属性值而不是你自己设定的属性值，你可以通过在实例里面删除你自己添加的属性的方式来实现。
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () {
 5     console.log(this.foo);
 6 }
 7 Thing.prototype.setFoo = function (newFoo) {
 8     this.foo = newFoo;
 9 }
10 Thing.prototype.deleteFoo = function () {
11     delete this.foo;
12 }
13 var thing = new Thing();
14 thing.setFoo("foo");
15 thing.logFoo(); //logs "foo";
16 thing.deleteFoo();
17 thing.logFoo(); //logs "bar";
18 thing.foo = "foobar";
19 thing.logFoo(); //logs "foobar";
20 delete thing.foo;
21 thing.logFoo(); //logs "bar";
或者你也能直接通过引用函数对象的prototype 来获得你需要的值。
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () {
 5     console.log(this.foo, Thing.prototype.foo);
 6 }
 7 
 8 var thing = new Thing();
 9 thing.foo = "foo";
10 thing.logFoo(); //logs "foo bar";
通过一个函数创建的实例会共享这个函数的prototype属性的值，如果你给这个函数的prototype赋值一个Array，那么所有的实例都会共享这个Array，除非你在实例里面重写了这个Array，这种情况下，函数的prototype的Array就会被隐藏掉。
1 function Thing() {
2 }
3 Thing.prototype.things = [];
4 
5 
6 var thing1 = new Thing();
7 var thing2 = new Thing();
8 thing1.things.push("foo");
9 console.log(thing2.things); //logs ["foo"]
给一个函数的prototype赋值一个Array通常是一个错误的做法。如果你想每一个实例有他们专属的Array，你应该在函数里面创建而不是在prototype里面创建。
 1 function Thing() {
 2     this.things = [];
 3 }
 4 
 5 
 6 var thing1 = new Thing();
 7 var thing2 = new Thing();
 8 thing1.things.push("foo");
 9 console.log(thing1.things); //logs ["foo"]
10 console.log(thing2.things); //logs []
实际上你可以通过把多个函数的prototype链接起来的从而形成一个原型链，因此this就会魔法般地沿着这条原型链往上查找直到找你你需要引用的值。
 1 function Thing1() {
 2 }
 3 Thing1.prototype.foo = "bar";
 4 
 5 function Thing2() {
 6 }
 7 Thing2.prototype = new Thing1();
 8 
 9 
10 var thing = new Thing2();
11 console.log(thing.foo); //logs "bar"
一些人利用原型链的特性来在JavaScript模仿经典的面向对象的继承方式。任何给用于构建原型链的函数的this的赋值的语句都会隐藏原型链上游的相同的属性。
 1 function Thing1() {
 2 }
 3 Thing1.prototype.foo = "bar";
 4 
 5 function Thing2() {
 6     this.foo = "foo";
 7 }
 8 Thing2.prototype = new Thing1();
 9 
10 function Thing3() {
11 }
12 Thing3.prototype = new Thing2();
13 
14 
15 var thing = new Thing3();
16 console.log(thing.foo); //logs "foo"
我喜欢把被赋值给prototype的函数叫做方法。在上面的例子中，我已经使用过方法了，如logFoo。这些方法有着相同的prototype，即创建这些实力的原始函数。我通常把这些原始函数叫做构造函数。在prototype里面定义的方法里面使用this会影响到当前实例的原型链的上游的this。这意味着你直接给this赋值的时候，隐藏了原型链上游的相同的属性值。这个实例的任何方法都会使用这个最新的值而不是原型里面定义的这个相同的值。
 1 function Thing1() {
 2 }
 3 Thing1.prototype.foo = "bar";
 4 Thing1.prototype.logFoo = function () {
 5     console.log(this.foo);
 6 }
 7 
 8 function Thing2() {
 9     this.foo = "foo";
10 }
11 Thing2.prototype = new Thing1();
12 
13 
14 var thing = new Thing2();
15 thing.logFoo(); //logs "foo";
在JavaScript里面你可以嵌套函数，也就是你可以在函数里面定义函数。嵌套函数可以通过闭包捕获父函数的变量，但是这个函数没有继承this
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () {
 5     var info = "attempting to log this.foo:";
 6     function doIt() {
 7         console.log(info, this.foo);
 8     }
 9     doIt();
10 }
11 
12 
13 var thing = new Thing();
14 thing.logFoo();  //logs "attempting to log this.foo: undefined"
　　 在doIt里面的this是global对象或者在严格模式下面是undefined。这是造成很多不熟悉JavaScript的人深陷 this陷阱的根源。在这种情况下事情变得非常糟糕，就像你把一个实例的方法当作一个值，把这个值当作函数参数传递给另外一个函数但是却不把这个实例传递给这个函数一样。在这种情况下，一个方法里面的环境变成了全局范围，或者在严格模式下面的undefined。

 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () {  
 5     console.log(this.foo);   
 6 }
 7 
 8 function doIt(method) {
 9     method();
10 }
11 
12 
13 var thing = new Thing();
14 thing.logFoo(); //logs "bar"
15 doIt(thing.logFoo); //logs undefined
一些人喜欢先把this捕获到一个变量里面，通常这个变量叫做self，来避免上面这种情况的发生。
博主非常喜欢用这种方式
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () {
 5     var self = this;
 6     var info = "attempting to log this.foo:";
 7     function doIt() {
 8         console.log(info, self.foo);
 9     }
10     doIt();
11 }
12 
13 
14 var thing = new Thing();
15 thing.logFoo();  //logs "attempting to log this.foo: bar"
但是当你需要把一个方法作为一个值传递给一个函数的时候并不管用。
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () { 
 5     var self = this;
 6     function doIt() {
 7         console.log(self.foo);
 8     }
 9     doIt();
10 }
11 
12 function doItIndirectly(method) {
13     method();
14 }
15 
16 
17 var thing = new Thing();
18 thing.logFoo(); //logs "bar"
19 doItIndirectly(thing.logFoo); //logs undefined
你可以通过bind将实例和方法一切传递给函数来解决这个问题，bind是一个函数定义在所有函数和方法的函数对象上面
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () { 
 5     console.log(this.foo);
 6 }
 7 
 8 function doIt(method) {
 9     method();
10 }
11 
12 
13 var thing = new Thing();
14 doIt(thing.logFoo.bind(thing)); //logs bar
你同样可以使用apply和call来在新的上下文中调用方法或函数。
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () { 
 5     function doIt() {
 6         console.log(this.foo);
 7     }
 8     doIt.apply(this);
 9 }
10 
11 function doItIndirectly(method) {
12     method();
13 }
14 
15 
16 var thing = new Thing();
17 doItIndirectly(thing.logFoo.bind(thing)); //logs bar
你可以用bind来代替任何一个函数或者方法的this，即便它没有赋值给实例的初始prototype。
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 
 5 
 6 function logFoo(aStr) {
 7     console.log(aStr, this.foo);
 8 }
 9 
10 
11 var thing = new Thing();
12 logFoo.bind(thing)("using bind"); //logs "using bind bar"
13 logFoo.apply(thing, ["using apply"]); //logs "using apply bar"
14 logFoo.call(thing, "using call"); //logs "using call bar"
15 logFoo("using nothing"); //logs "using nothing undefined"
你应该避免在构造函数里面返回任何东西，因为这可能代替本来应该返回的实例。
 1 function Thing() {
 2     return {};
 3 }
 4 Thing.prototype.foo = "bar";
 5 
 6 
 7 Thing.prototype.logFoo = function () {
 8     console.log(this.foo);
 9 }
10 
11 
12 var thing = new Thing();
13 thing.logFoo(); //Uncaught TypeError: undefined is not a function
奇怪的是，如果你在构造函数里面返回了一个原始值，上面所述的情况并不会发生并且返回语句被忽略了。最好不要在你将通过new调用的构造函数里面返回任何类型的数据，即便你知道自己正在做什么。如果你想创建一个工厂模式，通过一个函数来创建一个实例，这个时候不要使用new来调用函数。当然这个建议是可选的。

你可以通过使用Object.create来避免使用new，这样同样能够创建一个实例。
 1 function Thing() {
 2 }
 3 Thing.prototype.foo = "bar";
 4 
 5 
 6 Thing.prototype.logFoo = function () {
 7     console.log(this.foo);
 8 }
 9 
10 
11 var thing =  Object.create(Thing.prototype);
12 thing.logFoo(); //logs "bar"
在这种情况下并不会调用构造函数
 1 function Thing() {
 2     this.foo = "foo";
 3 }
 4 Thing.prototype.foo = "bar";
 5 
 6 
 7 Thing.prototype.logFoo = function () {
 8     console.log(this.foo);
 9 }
10 
11 
12 var thing =  Object.create(Thing.prototype);
13 thing.logFoo(); //logs "bar"
因为Object.create不会调用构造函数的特性在你继承模式下你想通过原型链重写构造函数的时候非常有用。
 1 function Thing1() {
 2     this.foo = "foo";
 3 }
 4 Thing1.prototype.foo = "bar";
 5 
 6 function Thing2() {
 7     this.logFoo(); //logs "bar"
 8     Thing1.apply(this);
 9     this.logFoo(); //logs "foo"
10 }
11 Thing2.prototype = Object.create(Thing1.prototype);
12 Thing2.prototype.logFoo = function () {
13     console.log(this.foo);
14 }
15 
16 var thing = new Thing2();
object this
在一个对象的一个函数里，你可以通过this来引用这个对象的其他属性。这个用new来新建一个实例是不一样的。
1 var obj = {
2     foo: "bar",
3     logFoo: function () {
4         console.log(this.foo);
5     }
6 };
7 
8 obj.logFoo(); //logs "bar"
注意，没有使用new，没有使用Object.create，也没有使用函数调用创建一个对象。你也可以将对象当作一个实例将函数绑定到上面。
1 var obj = {
2     foo: "bar"
3 };
4 
5 function logFoo() {
6     console.log(this.foo);
7 }
8 
9 logFoo.apply(obj); //logs "bar"
当你用这种方式使用this的时候，并不会越出当前的对象。只有有相同直接父元素的属性才能通过this共享变量
 1 var obj = {
 2     foo: "bar",
 3     deeper: {
 4         logFoo: function () {
 5             console.log(this.foo);
 6         }
 7     }
 8 };
 9 
10 obj.deeper.logFoo(); //logs undefined

你可以直接通过对象引用你需要的属性
var obj = {
    foo: "bar",
    deeper: {
        logFoo: function () {
            console.log(obj.foo);
        }
    }
};

obj.deeper.logFoo(); //logs "bar"
DOM event this
在一个HTML DOM事件处理程序里面，this始终指向这个处理程序被所绑定到的HTML DOM节点
 1 function Listener() {
 2     document.getElementById("foo").addEventListener("click",
 3        this.handleClick);
 4 }
 5 Listener.prototype.handleClick = function (event) {
 6     console.log(this); //logs "<div id="foo"></div>"
 7 }
 8 
 9 var listener = new Listener();
10 document.getElementById("foo").click();
除非你自己通过bind切换了上下文
 1 function Listener() {
 2     document.getElementById("foo").addEventListener("click", 
 3         this.handleClick.bind(this));
 4 }
 5 Listener.prototype.handleClick = function (event) {
 6     console.log(this); //logs Listener {handleClick: function}
 7 }
 8 
 9 var listener = new Listener();
10 document.getElementById("foo").click();
HTML this
在HTML节点的属性里面，你可以放置JavaScript代码，this指向了这个元素
1 <div id="foo" onclick="console.log(this);"></div>
2 <script type="text/javascript">
3 document.getElementById("foo").click(); //logs <div id="foo"...
4 </script>
override this
你不能重写this，因为它是保留字。

1 function test () {
2     var this = {};  // Uncaught SyntaxError: Unexpected token this 
3 }
eval this
你可以通过eval来访问this
function Thing () {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    eval("console.log(this.foo)"); //logs "bar"
}

var thing = new Thing();
thing.logFoo();
这会造成一个安全问题，除非不用eval，没有其他方式来避免这个问题。

在通过Function来创建一个函数的时候，同样能够访问this
function Thing () {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = new Function("console.log(this.foo);");

var thing = new Thing();
thing.logFoo(); //logs "bar"
with this
你可以通过with来将this添加到当前的执行环境，并且读写this的属性的时候不需要通过this
 1 function Thing () {
 2 }
 3 Thing.prototype.foo = "bar";
 4 Thing.prototype.logFoo = function () {
 5     with (this) {
 6         console.log(foo);
 7         foo = "foo";
 8     }
 9 }
10 
11 var thing = new Thing();
12 thing.logFoo(); // logs "bar"
13 console.log(thing.foo); // logs "foo"
许多人认为这样使用是不好的因为with本身就饱受争议。

jQuery this
和HTML DOM元素节点的事件处理程序一样，在许多情况下JQuery的this都指向HTML元素节点。这在事件处理程序和一些方便的方法中都是管用的，比如$.each
 1 <div class="foo bar1"></div>
 2 <div class="foo bar2"></div>
 3 <script type="text/javascript">
 4 $(".foo").each(function () {
 5     console.log(this); //logs <div class="foo...
 6 });
 7 $(".foo").on("click", function () {
 8     console.log(this); //logs <div class="foo...
 9 });
10 $(".foo").each(function () {
11     this.click();
12 });
13 </script>
thisArg this
如果你用过underscore.js 或者 lo-dash 你可能知道许多类库的方法可以通过一个叫做thisArg 的函数参数来传递实例，这个函数参数会作为this的上下文。举个例子，这适用于_.each。原生的JavaScript在ECMAScript 5的时候也允许函数传递一个thisArg参数了，比如forEach。事实上，之前阐述的bind，apply和call的使用已经给你创造了传递thisArg参数给函数的机会。这个参数将this绑定为你所传递的对象。

 1 function Thing(type) {
 2     this.type = type;
 3 }
 4 Thing.prototype.log = function (thing) {
 5     console.log(this.type, thing);
 6 }
 7 Thing.prototype.logThings = function (arr) {
 8    arr.forEach(this.log, this); // logs "fruit apples..."
 9    _.each(arr, this.log, this); //logs "fruit apples..."
10 }
11 
12 var thing = new Thing("fruit");
13 thing.logThings(["apples", "oranges", "strawberries", "bananas"]);
这使得代码变得更加简介，因为避免了一大堆bind语句、函数嵌套和this暂存的使用。
