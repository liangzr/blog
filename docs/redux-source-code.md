---
title: Redux 源码分析
date: 2019-04-17 14:46:53
tags:
  - source code
categories: 
  - javascript
  - redux
---

Redux 起源于一个实验，作者 Dan Abramov 本想通过 Flux 思想解决他的热重载和时间旅行的问题，如今它已经是 React 技术栈中状态管理方案的不二之选。

本篇主要分析 Redux 的源码结构，对 Flux 架构的思想不再赘述，可以参考[官方的解释](https://facebook.github.io/flux/)

## 源码结构

Redux 源码的目录结构如下：

```
src
├── applyMiddleware.js
├── bindActionCreators.js
├── combineReducers.js
├── compose.js
├── createStore.js
├── index.js
└── utils
    ├── actionTypes.js
    ├── isPlainObject.js
    └── warning.js
```

<!-- more -->

可以看到 Redux 的源码还是非常简单的，入口文件则导出了关键函数：

`index.js`：

```js
...

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```

最关键的 `createStore` 就是我们用来用来创建 store 的，而 `combineReducers` 和 `bindActionCreators` 则是一些辅助方法，`applyMiddleware` 和 `compose` 则和中间件有关。

下面我们依次从创建 store、辅助方法、中间件三个方面了解下 Redux 的设计。

## createStore

先不关注 `createStore` 中其它函数，以及一些校验的方法，我们先来看看它本身都做了些什么：

```js
export default function createStore(reducer, preloadedState, enhancer) {
  let currentReducer = reducer          // 初始化 reducer
  let currentState = preloadedState     // 初始化状态
  let currentListeners = []             // 初始化监听器为空数组
  let nextListeners = currentListeners  // 缓存当前监听器数组
  let isDispatching = false             // 初始化是否正在 dispatch 为 false
  
  function ensureCanMutateNextListeners() {}
  function getState() {}
  function subscribe(listener) {}
  function dispatch(action) {}
  function replaceReducer(nextReducer) {}
  function observable() {}
  
  dispatch({ type: ActionTypes.INIT })
  
  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer，
    [$$observable]: observable,
  }
}
```

可以看到，`createStore` 定义了一堆局部变量来保存状态，而这些状态在内部的方法中可以访问到，这些方法又被返回了出来，由此因为闭包的原因，这些函数内的变量也得以保存。

在声明了一系列方法后，`createStore` 最后 dispatch 了一个初始化的 action，随后返回了这些方法。

`actionTypes.js`: 

```js
const randomString = () =>
  Math.random()
    .toString(36)
    .substring(7)
    .split('')
    .join('.')

const ActionTypes = {
  INIT: `@@redux/INIT${randomString()}`,
  REPLACE: `@@redux/REPLACE${randomString()}`,
  PROBE_UNKNOWN_ACTION: () => `@@redux/PROBE_UNKNOWN_ACTION${randomString()}`
}

export default ActionType
```

`randomString` 生成了一个随机的字符串，`Number.prototype.toString` 可以把数字转换成 2-36 进制的字符串，36 进制度则用 0-9、a-z 来表示，如此一来它产生的就是类似 `'p.w.1.s.r.h'` 这样的字符串。

另外可以注意到，INIT 和 REPLACE 是在模块加载的时候就已经生成了随机字符串，而 PROBE_UNKNOWN_ACTION 则会在调用的时候才调用 `randomString` 方法。

### getState

```js
 function getState() {
    if (isDispatching) {
      throw new Error(
        'You may not call store.getState() while the reducer is executing. ' +
          'The reducer has already received the state as an argument. ' +
          'Pass it down from the top reducer instead of reading it from the store.'
      )
    }

    return currentState
  }
```

`getState` 方法的逻辑非常简单，它直接返回了函数变量 `currentState`，即当前的的整个 store 树。

但要注意的是，`isDispatching` 为 `true` 的时候，则会抛出一个错误，这个问题我们先留着待会看。

### subscribe

```js
function subscribe(listener) {
    if (isDispatching) {
      throw new Error(
        'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      )
    }

    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
        )
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }
```

`subscribe` 方法将传递进来的监听器，push 到了 `nextListeners` 数组中，定义了内部变量 `isSubscribed` 为 `true`，并且返回了 `unsubscribe` 方法。

`unsubscribe` 方法则检查了 `isSubscribed` 变量，并且将其置为 `false`，这样做可以防止 `unsubscribe` 被多次调用，然后又从 `nextListener` 数组中删掉了这个监听器。

整个的逻辑非常简单，**将一个监听器方法加入或移除监听器列表**。

`ensureCanMutateNextListeners` 方法这里暂时不管它，后面我们会提到。

### dispatch

```js
function dispatch(action) {
  if (!isPlainObject(action)) {
    throw new Error(
      'Actions must be plain objects. ' +
        'Use custom middleware for async actions.'
    )
  }

  if (typeof action.type === 'undefined') {
    throw new Error(
      'Actions may not have an undefined "type" property. ' +
        'Have you misspelled a constant?'
    )
  }

  if (isDispatching) {
    throw new Error('Reducers may not dispatch actions.')
  }

  try {
    isDispatching = true
    currentState = currentReducer(currentState, action)
  } finally {
    isDispatching = false
  }

  const listeners = (currentListeners = nextListeners)
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners[i]
    listener()
  }

  return action
}
```

`dispatch` 方法是整个 Redux 的核心，不过它的过程也很简单。

首先它将 `isDispatching` 置于 `true`，然后调用了 `currentReducer` 方法，传入了 `currentState` 和 `action`，又返回了 `currentState`。也就是用当前的状态和 action，计算出了一次的状态。在 `try` 代码块执行完后，则会把 `isDispatching` 重置为 `false`。

> 这里的 try...finally 并没有 catch 代码块，所以并不会吃掉 Error

在这里终于明白上文中多次出现的 `isDispatching` 的具体含义，就是在计算新的 state 的过程中，也就是执行 reducer 方法时，不允许 `getState`、 `subscribe`、`unsubscribe` 以及 `dispatch`。

在 dispatch 方法的最后，遍历了监听器数组并且逐个执行，然后返回了 action 对象。

可以看到，dispatch 方法其实非常简单，它主要的内容就是**计算新 store 并且在执行完成后，执行监听器方法**。但对于 action 需要注意的是，首先它必须是一个常规对象，其次根据 Flux 的规范，**action 也必须有 type 属性。**

### other

还有一些其它的方法作为功能上的补充，这些简单介绍下它们

#### replaceReducer

```js
function replaceReducer(nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.')
  }

  currentReducer = nextReducer

  // This action has a similiar effect to ActionTypes.INIT.
  // Any reducers that existed in both the new and old rootReducer
  // will receive the previous state. This effectively populates
  // the new state tree with any relevant data from the old one.
  dispatch({ type: ActionTypes.REPLACE })
}
```

这个方法可以把当前的 reducer 整个替换掉，它的主要应用场景在于动态地加载 reducer，比如如果你想地你的应用启用代码分割时。

#### ensureCanMutateNextListeners

```js
function ensureCanMutateNextListeners() {
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice()
  }
}
```

通过 slice 方法遍历了 `currentListeners` 数组，并且缓存到 `nextListeners` 中。

## combineReducers

reducer 的定义是一个函数，这个函数接受一个旧的 state 和一个 action 对象，并且返回一个新对象。

我们可以在 reducer 内通过 `swtich...case` 语法，通过判断 `aciton.type` 来分别执行不同的 mutate 方法，最后返回一个新的 state。但如果一个应用中所有的 mutation 都写到一个 `switch...case` 里，这个 reducer 方法就容易变得难以维护。

redux 库为我们提供了一个 `combineReducers` 方法，帮助我们把许多 reducer 合并成一个总的 reducer，然后再传递给 `createStore` 方法去初始化 store。

我们先来简单地看下这个方法做了什么处理：

```js
export default function combineReducers(reducers) {
  /*
   * 第一部分: 过滤掉非函数的 reducer, 把结果放入 finanlReducers 中
   */
  const reducerKeys = Object.keys(reducers)          
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {   
    const key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)
  
  /*
   *  第二部分：判断 reducers 是否合法
   */

  let shapeAssertionError
  try {
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }


  /*
   *  第三部分：返回合并后的 reducer 方法
   */
  return function combination(state = {}, action) {
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```

这个函数大体上可以分为三部分。先是过滤掉了 reducers 中非函数的部分，然后对 reducer 做了个校验，最后返回了合并后的 reducer 方法。第一和第二部分较为简单不再赘述，主要来看下第三部分。

第二部分中校验失败时抛出的错误被 catch 并保存到 `shapeAssertionError` 中了，这样在创建 store 的时候就不会抛错，直到执行（任意） reducer 的时候才抛出错误。由于 createStore 方法的末尾 dispatch 了一个 INIT action，所以它也会在初始化的时候报错。

第三部分主要做了一个简单的遍历操作，把 reducers 中的每个 reducer 及其对应的 state 拿出来，加上传递进来的 action 对象，计算出了新的 state 并缓存起来。这里需要注意的是，combination 方法定义了一个 `hasChanged` 标志，如果这些 reducers 中没有任何一个计算后的 state 的引用发生改变，则返回旧的 state。在这里**对各个 reducer 对应的新旧 state 应用了浅比较**，这意味着你只修改 reducer 对应的 state 内部的属性，store 是不会发生改变的。

## bindActionCreators

这是 redux 库提供的一个易用的辅助方法，帮助我们简化 dispatch 和 actionCreators 的交互，可以方法地直接调用方法达到 dispatch action 的目的。

看下源码：

```js
export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  const boundActionCreators = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}

function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}
```

它的逻辑非常简单，一个 actionCreator 可能是这样的：

```js
const toggleLoading = () => ({
  type: 'TOGGLE_LOADING'
})
```

而一个 actionCreators 则是像下面这样：

```js
const actions = {
  toggleLoading() {
    return {
      type: 'TOGGLE_LOADING'
    }
  } 
}
```

所以 bindActionCreators 的效果则是接受一个 actionCreators 和 dispatch 方法，将它封装成一个对象，调用的时候可以直接把参数传给 actionCreators 并且把产生的 action 传递给 dispatch 来更新状态。

## 结语

如此一来 Redux 的基本功能我们也都了解的差不多了，它的实现可以说非常简洁，主要的设计思想围绕着 reducer 和 dispatch。

另外 Redux 还提供了中间件的功能，可以辅助我们在传递 action 的时候做处理，由于跟主逻辑无关，我们放到另外一篇文章来讲。