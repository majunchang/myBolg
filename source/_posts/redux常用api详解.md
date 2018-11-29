---
title: react-redux常用api详解
tags: React
categories: React
abbrlink: 8843d73f
date: 2018-07-09 23:10:55
---
### createStore(reducer, [preloadedState], enhancer)


> 创建一个 Redux store 来以存放应用中所有的 state。
应用中应有且仅有一个 store。

**参数：**
1. reducer (Function): 接收两个参数，分别是当前的 state 树和要处理的 action，返回新的 state 树。

2. [preloadedState] (any): 初始时的 state。 在同构应用中，你可以决定是否把服务端传来的 state 水合（hydrate）后传给它，或者从之前保存的用户会话中恢复一个传给它。如果你使用 combineReducers 创建 reducer，它必须是一个普通对象，与传入的 keys 保持同样的结构。否则，你可以自由传入任何 reducer 可理解的内容。

3. enhancer (Function): Store enhancer 是一个组合 store creator 的高阶函数，返回一个新的强化过的 store creator。这与 middleware 相似，它也允许你通过复合函数改变 store 接口。

**返回值**
 保存了应用中所有state的对象，改变state的唯一方法是dispatch action。 你也可以subscribe监听state的变化 然后更新ui
 
 ###  createStore的简易版源码
 
 
>  enhancer是一个高阶函数  运行结果表明 enhancer之后代码 直接进入applymiddleware

```
const store = createStore(counter, applyMiddleware(thunk, arrThunk))
```


```js
export function createStore (reducer, enhancer) {
  if (enhancer) {
    console.log('enhancer', enhancer)
    return enhancer(createStore)(reducer)

    //  return applymiddleware(thunk)(createStore)(reducer)
  }
  let currentState = {}
  let currentListeners = []

  function getState () {
    return currentState
  }

  function subscribe (listener) {
    currentListeners.push(listener)
  }

  function dispatch (action) {
    currentState = reducer(currentState, action)
    currentListeners.forEach(v => {
      v()
    })
    return action
  }

  //  初次调用的时候 首先执行一次 dispatch
  dispatch({type: '@@redux/firstTime'})

  return {getState, subscribe, dispatch}
}
```

 
 
 ###  Store  
>  Store 就是用来维持应用所有的 state 树 的一个对象。 改变 store 内 state 的惟一途径是对它 dispatch 一个 action。

> Store 不是类。它只是有几个方法的对象。 要创建它，只需要把根部的 reducing 函数 传递给 createStore。

> 注意

**Store的方法**
- **getState()**       返回应用当前的state树
- **dispatch(action)**  分发action 这是触发state变化的唯一途径 
- **subscribe(listener)**   添加一个变化监听器 每当dispatch(action)的时候 就会执行 state 树中的一部分可能已经变化。你可以在回调函数里调用 getState() 来拿到当前 state。

```js
const createStoreWithMiddleware = applyMiddleware(...middlewares)(createStore)
const store = createStoreWithMiddleware(rootReducer)

sagaMiddleware.run(rootSaga)

const action = type => store.dispatch({ type })

function render () {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => action('INCREMENT')}
      onDecrement={() => action('DECREMENT')}
      onIncrementAsync={() => action('INCREMENT_ASYNC')} />,
    document.getElementById('root')
  )
}

render()

store.subscribe(render)
```

###  applyMiddleware(...middlewares)
> 使用包含自定义功能的 middleware 来扩展 Redux 是一种推荐的方式。Middleware 可以让你包装 store 的 dispatch 方法来达到你想要的目的。同时， middleware 还拥有“可组合”这一关键特性。多个 middleware 可以被组合到一起使用，形成 middleware 链。其中，每个 middleware 都不需要关心链中它前后的 middleware 的任何信息。

#### redux-thunk 举例
> 例如，redux-thunk 支持 dispatch function，以此让 action creator 控制反转。被 dispatch 的 function 会接收 dispatch 作为参数，并且可以异步调用它。这类的 function 就称为 thunk。


```js
function addIfOdd() {
    return (dispatch, getState) => {
        const currentValue = getState();
        if (currentValue % 2 == 0) {
            return false;
        }
        dispatch(add())
    }
}
function addAsy(delay = 2000) {
    return (dispatch, getState) => {
        setTimeout(() => {
            dispatch(add())
        }, delay)
    }
}
```


#### applymiddleware 源码解析 


官方源码

```
function applyMiddleware() {
    //1
  for (var _len = arguments.length, middlewares = Array(_len), _key = 0; _key < _len; _key++) {
    middlewares[_key] = arguments[_key];
  }
  //2
  return function (createStore) {
      //3
    return function (reducer, preloadedState, enhancer) {
      //4
      var store = createStore(reducer, preloadedState, enhancer);
      var _dispatch = store.dispatch;
      var chain = [];
      //5
      var middlewareAPI = {
        getState: store.getState,
        dispatch: function dispatch(action) {
          return _dispatch(action);
        }
      };
      chain = middlewares.map(function (middleware) {
        return middleware(middlewareAPI);
      });
      //6
      _dispatch = _compose2['default'].apply(undefined, chain)(store.dispatch);

      return _extends({}, store, {
        dispatch: _dispatch
      });
    };
  };
}
```

简化版源码

```js
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    // 生成一个store 
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []
    //简陋版的store，里面包含了`getState`和`dispatch`两个方法。
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    //将middlewareAPI作为参数注入到每个middleware中去，执行middleware, 返回一个新的链。
    //中间件函数(store)=>next=>action.
    // chain  [next=>action,...];
    //这个 next 其实store.dispatch.   而`action`就是`dispatch`的action
    chain = middlewares.map(middleware => middleware(middlewareAPI))

    //我们假设有三个中间件,fn1,fn2,fn3,那么下面代码等同于  dispatch=fn1(fn2(fn3(store.dispatch)));
    //可以发现，中间件所组成的dispatch 其实就是一个执行过fn1,fn2,fn3的函数。
    //所以，每个中间件在遇到不是自己处理范围之内的action的时候，会使用next(action)，将其传递给下一个中间件。
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

>搭配一个redux-thunk的源码


```js
const thunk = ({dispatch, getState}) => next => action => {
  if (typeof action === 'function') {
    //  如上面所示
    return action(dispatch, getState)
  }
  //  默认情况下
  return next(action)
}

export default thunk
```

###  appliymiddleware的总结


```
createStore(reducer,initState,applyMiddleware(ThunkMiddleware));
```


```
createStore(reducer,applyMiddleware(ThunkMiddleware));
```
这两种设置中间件的方式是一致的 



```
const store = applyMiddleware(...middlewares)(createStore)(reducer, initialState)。
```


```
const store = createStore(reducer, initialState, applyMiddleware(...middlewares))
```



### compose的作用 


> 从右到左来组合多个函数。
这是函数式编程中的方法，为了方便，被放到了 Redux 里。
当需要把多个 store 增强器 依次执行的时候，需要用到它。


 附示例代码：
```js
const store = createStore(reducers, compose(
  applyMiddleware(thunk),
  window.devToolsExtension ? window.devToolsExtension() : f => f
))
```

### compose的源码解析 


```js
function compose (...fns) {
  if (fns.length === 0) {
    return args => args
  }
  if (fns.length === 1) {
    return fns[0]
  }
  return fns.reduce((ret, item) => (...args) => ret(item(...args)))
}
```

####  附 reduce方法 
> reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。

> reduce() 可以作为一个高阶函数，用于函数的 compose。

>  reduce() 对于空数组是不会执行回调函数的

###### 参数 

```
 array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
    total 必需。初始值, 或者计算结束后的返回值。
    currentValue  必需。当前元素
    currentIndex  可选。当前元素的索引
    arr 可选。当前元素所属的数组对象。
    initialValue  可选   代表total的初始类型和初始值 
```

   
###### 基本使用


```js
   var arr = [1,2,3,4,5,6]
    var result = arr.reduce((sum,val)=>{
        return sum+val
    },100)

    console.log(result);
```

###### 高级使用
如何知道一串字符串中每个字母出现的次数？
```js
var arrString = 'abcdaabc';

arrString.split('').reduce(function(res, cur) {
    res[cur] ? res[cur] ++ : res[cur] = 1
    return res;
}, {})
```


