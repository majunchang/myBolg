---
title: 详解js原型，构造函数以及class之间的原型关系
abbrlink: cf19f4c8
date: 2018-10-11 22:47:27
tags:
---
## 原型

##### 概念
> 在构造函数创建的时候，系统默认的帮构造函数创建并关联一个对象 这个对象就是原型
##### 作用
>  在原型中的所有属性和方法，都可以被和其关联的构造函数创建出来的所有的对象共享
##### 访问原型
>  构造函数名.prototype   实例化的对象.__proto __
##### 原型的简单使用
1.  利用对象的动态特性为原型对象增加成员
2.  直接替换原型对象（jq核心方法的实现 就是使用原型替换的思想）


```js
 function Person (name) {
      this.name = name
    }

    Person.prototype.fn = function () {
      console.log('hello world')
      console.log(this.name)
    }

    var p = new Person('小红')

    p.fn()
    p.name = '晓丽'
    p.fn()
    console.log(Person.prototype)
    console.log(p)
```
![image](http://upload-images.jianshu.io/upload_images/5703029-f6f93658cfcdabc8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image](http://upload-images.jianshu.io/upload_images/5703029-4b7c22281abecf3e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###  1. prototype
含义： 是一个函数的属性，这个属性是一个指针，指向一个对象
作用： 构造函数调用  访问该构造函数所关联的原型对象

###  2. __proto__
含义： 是一个对象拥有的内置属性，是js内部使用寻找原型链的属性，通过该属性可以允许实例对象直接访问到原型

###  3. constructor

含义：原型对象的constructor 指向其构造函数,如果替换了原型对象之后，这个constructor属性就不准确，需要手动补充一下
![image](http://upload-images.jianshu.io/upload_images/5703029-f0cdca6c86b70cd0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 原型链 
![image](http://upload-images.jianshu.io/upload_images/5703029-f94e98852e196457.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 构造函数以及js原生Object对象之间的原型关系
![image](http://upload-images.jianshu.io/upload_images/5703029-8548b5bdb83b82fb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##  原型的注意事项

- 当对象在访问属性和方法的时候，会现在自身查找，如果没有才回去原型中找。（一级一级传递 形成了原型链）
- 替换原型对象的时候，替换之前构造函数创建的对象A和替换之后创建的对象B，A和B的原型是不一致的。
- 对象能够访问的原型，就是在对象创建的那一刻，和构造函数关联的那个原型


> ## 扩展以及延伸


![image](http://upload-images.jianshu.io/upload_images/5703029-badb132db05176ea.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 构造函数 
> 在很多编程语言中，如java，objectC,c++等，都存在类的概念，类中有私有属性，私有方法等，通过类来实现面对对象的继承，但是，在ES5以及以前中不像上面这几种语言一样，有严格的类的概念。js通过构造函数以及原型链来实现继承。

##### 特点
-  首字母必须为大写，用来区分普通函数
-  内部使用的this对象，来指向即将要生成的实例对象
-  使用new 关键字来生成实例对象（下面为new关键字的具体实现）

```js
    var obj = new Date()
    // 可以分解为
    var obj = {};
    obj.__proto__ = Date.prototype;
    Base.call(obj)

```

##### 缺点

- 所有的实例对象都可以继承构造器函数中的属性和方法，但是同一个对象实例之间，无法共享属性。
- 如果方法在构造函数内部，每次new一个实例对象的时候，都会创建内部的这些方法，并且不同的实例对象之间，不能共享这些方法，造成了资源的浪费（于是有了原型这个概念）


###  实现方式 （简单列举几种）

#### 构造函数模式（自定义构造函数）

>  构造函数与普通函数的区别
```js
//构造函数
function Egperson (name,age) {
    this.name = name;
    this.age = age;
    this.sayName = function () {
        alert(this.name);
     }
}
var person = new Egperson('mike','18'); //this-->person
person.sayName();  //'mike'


//普通函数
function egPerson (name,age) {
    this.name = name;
    this.age = age;
    this.sayName = function () {
        alert(this.name);
     }
}
egPerson('alice','23'); //this-->window
window.sayName();  //'alice'

```

#### 工厂模式

```js
function CreatePerson(name, age, gender){
    var obj = {};
    obj.name = name;
    obj.age = age;
    obj.gender = gender;
    //由于是函数调用模式，所以this打印出来是window
    console.log(this);
    return obj;
}
var p = CreatePerson("小明", 18, "male");  // 调用方式是函数的调用方式
```

#### 寄生模式

```
function CreatePerson(name, age, gender){
    var obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.gender = gender;
    //这里的this指向new 创建出来的对象
    console.log(this);
    return obj;
}
var p = new CreatePerson("小明", 18, "male");  // 调用方式是函数的调用方式

```
#### 动态原型模式  

```js
function Person(name, age, job) {
    //属性
    this.name = name;
    this.age = age;
    this.job = job;
    
    //方法
    if(typeof this.sayName != "function") {
        //所有的公有方法都在这里定义
        Person.prototype.sayName = function() {
            alert(this.name);
        }；

        Person.prototype.sayJob = function() {
            alert(this.job);
        };


        Person.prototype.sayAge = function() {
            alert(this.age);
        };
    }
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");

person1.sayName();        //Nicholas
person2.sayName();        //Greg


```


>  js实现继承的方式: 混入式继承，原型继承以及经典继承，ES6的Class也可以实现继承


## Class 详解
> 基本上，ES6 的class可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。


```js
// ES5
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);
// ES6

//定义类
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}


```

#### 特点
- class中的constructor函数相当于ES5中的构造函数（声明属性以及静态方法，这种类创建属性和创建方法参照上面动态原型模式的构造函数。个人感觉有很多相似之处
- 类中定义方法的时候，前面不加function，后面不加，可被实例对象也就是子类继承的方法 定义在类的prototype属性中。
- 类的内部定义的所有方法都是不可枚举的
- 类和模块内部默认采用严格模式
- 子类继承父类以后，必须在constructor中调用时super方法，否则不能新建实例，因为子类没有属于自己的this对象，而是继承了父类的this对象对其进行加工


#### 类中的原型链关系
> 每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。Class 作为构造函数的语法糖，同时有prototype属性和__proto__属性，因此同时存在两条继承链。

1. 子类的__proto__属性，表示构造函数的继承，总是指向父类。
2. 子类prototype属性的__proto__属性，表示实例方法的继承，总是指向父类的prototype属性。

```js
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```
类的继承内部实现


```js
class A {
}

class B {
}

// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);

// B 继承 A 的静态属性
Object.setPrototypeOf(B, A);

const b = new B();

Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```
**作为一个对象，子类（B）的原型（__proto__属性）是父类（A）；**

**作为一个构造函数，子类（B）的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。**


```
Object.create(A.prototype);
// 等同于
B.prototype.__proto__ = A.prototype;
```
#### 实例的 __proto__ 属性

> 子类实例的__proto__属性的__proto__属性，指向父类实例的__proto__属性。也就是说，子类的原型的原型，是父类的原型。


```
var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__ === p1.__proto__ // false
p2.__proto__.__proto__ === p1.__proto__ // true

```



#### 类中this指向问题
> 类的方法内部含有this，默认指向类的实例。但是当类中的实例方法提取出来使用的时候，this指向运行时所在环境。

解决方法（新版react中，在声明绑定方法的时候 三种方式与此相同）


```js
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }

  // ...
}

class Logger {
  constructor() {
    this.printName = (name = 'there') => {
      this.print(`Hello ${name}`);
    };
  }

  // ...
}

```


#### ES5与ES6 实现继承的区别

1. 在ES5中，继承实质上是子类先创建属于自己的this，然后再将父类的方法添加到this（也就是使用Parent.apply(this)的方式

2. 而在ES6中，则是先创建父类的实例对象this，然后再用子类的构造函数修改this。









