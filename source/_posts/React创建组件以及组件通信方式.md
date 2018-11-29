---
title: React创建组件以及组件通信方式
abbrlink: f1f1cb7d
date: 2018-10-11 22:42:44
tags:
---
### 无状态函数式组件
> 创建纯展示组件，无法使用State，也无法使用组件的生命周期方法，只负责根据传入的props来展示，不涉及到要state状态的操作，是一个只带有一个render方法的组件类

创建形式：

```js
import React from 'react'

function NoStatusComp (props) {
  console.log(props)
  return (
    <div>
      <h1>我是无状态组件</h1>
    </div>
  )
}

export default NoStatusComp

```

特点：
1. 不需要声明类，组件不会被实例化，整体渲染性能得到提升
2. 不需要显示声明this关键字，也就是说组件不能访问this对象
3. 不支持'ref'，同时也无法访问生命周期的方法


##### 无状态组件也是官方比较推荐的一种方式, 使得代码结构更加清晰，减少代码冗余，在开发过程中，尽量使用无状态组件。


### React.createClass方式创建组件

> createClass本质上是一个工厂函数,是ES5的原生的JavaScript来实现的React组件,是是react最初推荐的创建组件的方式。


```js
import React from 'react'
import {Button} from 'antd'

let createClassComp = React.createClass({
  getInitialState () {
    return {
      text: 'React早期创建组件的方式',
      title: '教师节快乐，感谢生命中出现过的老师，让似水年华没有苍白，教会我们听说读写思',
      num: 0
    }
  },
  componentDidMount () {
    console.log('cmd')
    console.log(this.props)
  },
  componentWillReceiveProps (nextProps) {
    console.log(this.props)
    console.log('下一次')
    console.log(nextProps)
  },
  add () {
    this.num++
  },
  render () {
    let {title, text} = this.state
    return (
      <div>
        <h1>{title}</h1>
        <span>{text}</span>
        <Button onClick={this.add}> 触发方法 </Button>
      </div>
    )
  }
})

export default createClassComp

```

######  新版的react  是16.4.1的  已经不再支持这种写法了

特点：

1. React.createClass会自绑定函数方法导致不必要的性能开销
2. React.createClass的mixins不够自然、直观


附一篇文章   介绍的比较详细  在此不做赘述了

[关于React.createClass方法与class App extends Component方法的区别](https://blog.csdn.net/wbiokr/article/details/73027398?utm_source=itdadao&utm_medium=referral)


### React.Component创建组件  

> React.Component是以ES6的形式来创建react的组件的，是React目前极为推荐的创建有状态组件的方式，其成员函数不会自动绑定this，需要手动绑定，否则this不能获取当前组件实例对象。


```js
import React from 'react'
import { Button } from 'antd'

export default class ExtendsComp extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      data: props.data
    }
  }
  render () {
    return (
      <div>
        <h1>我是新版创建组件的方法</h1>
        <Button> 点我回去</Button>
      </div>
    )
  }
}

```

特点:  

1. 其成员函数不会自动绑定this，需要手动绑定，否则this不能获取当前组件实例对象, 绑定this 有三种方法  比较推荐在构造函数中绑定
2. 类的constructor需要接收props并且调用super(props)。这是createClass所没有的一点。
3. 在调用super之后，可以直接设置state。




####  react的组件更新

1.  react的父组件 更新的时候  触发了render方法
2.  父组件下面的所有子组件都被重新渲染  
3.  可以通过使用immutatble的这种数据结构 去节省这种渲染（只渲染数据改动的子组件  数据没有改动的 维持原状）


[Immutable 常用API简介](https://segmentfault.com/a/1190000010676878)

[Immutable 详解及 React 中实践](https://github.com/camsong/blog/issues/3)


##  react组件的通信方式 


####  父组件传递给子组件 
> React数据流动是单向的,父组件向子组件通信也是最常见的;父组件通过props向子组件传递需要的信息

```
import React, { Component } from 'react';

import Child from './Child';

class Parent extends Component {
    render() {
        return (
            <div>
                <Child name="apple" />
            </div>
        );
    }
}

export default Parent;
```



####  子组件传递给父组件 
> 将父组件的方法  通过props传递给子组件   然后子组件调用方法 （也就是调用了父组件的方法 进而发生改变）


```
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class List3 extends Component {
    static propTypes = {
        hideConponent: PropTypes.func.isRequired,
    }
    render() {
        return (
            <div>
                我是字组件
                <button onClick={this.props.show}>点我 展示</button>
            </div>
        );
    }
}

export default List3;
```


####  较深层级的父子组件通信  

-  层层组件传递props  （不推荐）

-  使用context  
> context是一个全局变量,像是一个大容器,在任何地方都可以访问到,我们可以把要通信的信息放在context上,然后在其他组件中可以随意取到;

官方说明：

Context 通过组件树提供了一个传递数据的方法，从而避免了在每一个层级手动的传递 props 属性。

在一个典型的 React 应用中，数据是通过 props 属性由上向下（由父及子）的进行传递的，但这对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI主题），这是应用程序中许多组件都所需要的。 Context 提供了一种在组件之间共享此类值的方式，而不必通过组件树的每个层级显式地传递 props 

- parent.jsx

```js
import React, { Component } from 'react'

import Child from './childOneComp'
import ChildTwoComp from './childTwoComp'
import PropTypes from 'prop-types';

class Parent extends Component {
  constructor () {
    super()
    this.state = {
      listArr: [ {
        text: '题目一'
      },
      {
        text: '题目二'
      } ]
    }
  }
  // 提供一个函数,用来返回相应的context对象
  getChildContext () {
    return {
      color: 'red'
    }
  }
  // 父组件声明自己支持context
  static childContextTypes = {
    color: PropTypes.string,
  }
  render () {
    let {listArr} = this.state
    return (
      <div>
        <Child name='Sara' />
        {
          listArr.map((item, index) => {
            return <ChildTwoComp value={item.text} key={index} />
          })
        }

      </div>
    )
  }
}

export default Parent

```


- child.jsx

```js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import emitter from '../../utils/event'

class ChildTwoComp extends Component {
    // 子组件声明自己要使用context
    static contextTypes = {
        color: PropTypes.string,
    }
    static propTypes = {
        value: PropTypes.string,
    }
    constructor(){
      super()
      this.state ={
        color:'pink'
      }
    }
    componentDidMount() {
      this.eventEmitter = emitter.addListener('colorChange', (color) => {
        console.log('接受了方法');
        this.setState({
          color,
        },()=>{
          console.log(color);
        });
        
    });
    }
    componentWillUnmount() {
      if(typeof this.eventEmitter === 'function'){
        this.eventEmitter.removeListener(this.eventEmitter);
      }
      
  }
    
    render() {
        const { value } = this.props;
        return (
            <div>
              <h1 style={{ background: this.state.color }}>1234567</h1>
               <li style={{ background: this.context.color }}>
                <span>{value}</span>
            </li>
            </div>
           
        );
    }
}

export default ChildTwoComp;

```


####  兄弟组件之间通信  

以常用的发布/订阅模式举例,借用Node.js Events模块的浏览器版实现
要求组件A的数据 传递给组件B  但是 组件A和组件B 必须要同时渲染的时候 才能使用这种方法（有些类似vue的eventBus的功能，但是没有vue的强大）

- child.js

```
import React from 'react'
import PropTypes from 'prop-types'
import emitter from '../../utils/event'
import { Button } from 'antd'

export default class Child extends React.Component {
  constructor (props) {
    super(props)
    this.changeColor = this.changeColor.bind(this)
  }
  changeColor () {
    emitter.emit('colorChange', 'deepskyblue')
  }
  render () {
    let {name} = this.props
    return (
      <div>
        <h1>Hello, {name}</h1>
        <Button onClick={this.changeColor}> 11111点我回去</Button>
      </div>
    )
  }
}

Child.propTypes = {
  name: PropTypes.string.isRequired
}

```


- event.js


```js
import { EventEmitter } from 'events'

export default new EventEmitter()

```

###  react组件通信的第三方库 


1. [pubsub](https://www.npmjs.com/package/pubsub-js)  类似于发布订阅模式这样
2. [redux](http://www.redux.org.cn/)  （比较推荐）






