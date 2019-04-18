---
title: Redux 中间件
date: 2019-04-18 13:01:38
tags:
  - source code
categories: 
  - javascript
  - redux
---

Middleware 是 Redux 提供的一个增强功能，利用函数式编程的柯里化和组合特性，实现了一个和 koa 类似的洋葱结构的中间件。

## 初始化

我们再回到 `createStore` 方法的最开始，有这么一段代码

```js
export default function createStore(reducer, preloadedState, enhancer) {
  if (
    (typeof preloadedState === 'function' && typeof enhancer === 'function') ||
    (typeof enhancer === 'function' && typeof arguments[3] === 'function')
  ) {
    throw new Error(
      'It looks like you are passing several store enhancers to ' +
        'createStore(). This is not supported. Instead, compose them ' +
        'together to a single function.'
    )
  }

  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }
  
  // 主逻辑
  ...
}
```

它先是判断了如果第二个参数和第三个参数都为函数，或者第三个参数和第四个参数都为函数，则会警告只允许有一个 enhancer，实际上也只允许 applyMiddleware 的返回。

同时，如果第二个参数是 function 并且第三个参数是 undefined 时，第二个参数则为真正的 enhancer。这意味着 preloadedState 参数可以被省略，当它被省略的时候，值初始化为 undefined。

如果 enhancer 校验通过，则会调用它，并且把 `createStore` 本身作为参数，并且返回 enhancer 的结果。

<!-- more -->

## applyMiddleware

其实目前在 redux 中，唯一的 enhancer 就是 applyMiddleware 函数的返回，所以关键还在于这个函数。

applyMiddleware 主体：

```js
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

可以看到，它先是通过 `createStore` 创建了 store 实例，然后禁止在重新赋值之前使用 `dispatch` 方法。

```js
const middlewareAPI = {
  getState: store.getState,
  dispatch: (...args) => dispatch(...args)
}
```

随后定义了 middleware 的两个默认传参，一个是 store 中的 `getState` 方法，一个是封装的 `dispatch`。

```js
const chain = middlewares.map(middleware => middleware(middlewareAPI))
```

这一步遍历了每个中间件，把上一步的两个参数传进入后执行，并把执行后的结果以数组的形式赋给了 `chain`，所以 chain 其实是一个初始化过的中间件的数组。

```js
dispatch = compose(...chain)(store.dispatch)
```

这一步可以看到，通过一个 `compose` 方法，返回了一个函数，并且把 `store.dispatch` 传参进去，返回了一个新的 dispatch。

```js
return {
  ...store,
  dispatch
}
```

最后把新的 `dispatch` 替换第一步生成的，和 store 合并到一起并返回。

总的来说， `applyMiddleware` 的效果就是对 `dispatch` 做了处理，而具体的处理逻辑都做了什么呢？还要看 `compose` 方法。

## compose

compose 是函数式编程中常见的一个方法，常用来搭配柯里化（currying）来对原数据进行一系列处理。

在函数式编程的概念里，函数是一等公民。而在这个场景中，它处理的对象正是 `dispatch` 函数，

源码：

```js
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

看似非常简单，但理解起来不是特别容易，如果对 compose 和 currying 陌生的话。前两个都是边界处理，如果没有参数，则返回一个**返回自身**的函数。如果只有一个参数，则返回此参数。

最关键的是最后一行，使用了一个 `reduce` 方法，让它可以依次处理 `funcs` 中的所有函数，并把后一个执行的结果，做为参数传递给前一个函数。

## 执行过程

只是说可能有点拗口，我们来看个例子，假如有个 `funcs`：

```js
const funcs = [a, b, c, d, e]   // 有五项，每项都是函数
compose(funcs) // (...args) => a(b(c(d(e(...args)))))
```

所以上方返回的 dispatch 其实是：

```js
dispatch = compose(...funcs)(store.dispatch) // a(b(c(d(e(store.dispatch)))))
```

可以看到，它正是被中间件函数层层包裹之后处理得出的结果，即新的 dispatch。

那再来看一个典型的中间件应该是什么样的：

```js
const reduxLogger = ({ getState, dispatch }) => next => action => {
  console.log(action)
  return next(action)
}
```

一连串的箭头函数让人看的头皮发麻，不过我们知道，中间件在放进 `compose` 执行之前，先做了一步预处理，传递了两个 api 进去，所以简化一下真正被 `compose` 执行的中间件应该是这样的：

```js
let getState, dispatch // 闭包的变量

const reduxLoggerHander = next => action => {
  console.log(action)
  return next(action)
}
```

我们上方提到，后一个函数执行的结果，可以当作参数传递给前一个，所以假如 `reduxLogger` 在最后一个，则它执行完的结果应该是：

```js
let getState, dispatch // 闭包的变量

const dispatchWithLogger = action => {
  console.log(action)
  return store.dispatch(action)
}
```

走到这一步，拨开云雾见天明，这个中间件已经不再神秘了。经过 `componse` 处理后，它就是一个新的拥有日志功能的 dispatch，而它内部正是调用了 `createStore` 中原生的 `store.dispatch` 方法。

你可能会想，既然只是一层封装，重新写个方法不就好了，为什么要埋的这么深？这是因为，通过 `compose` 可以不只处理一个中间件，还可以直接应用一个中间件数组，并且让它们之间递归调用。

我们再回到 `dispatchWithLogger` 方法，它是最内层的执行结果，按照我们前面的分析，它将作为参数传递给下一个中间件。假如是一个 `Hello` 中间件，最终返回的应该是 `dispatchWithLoggerAndHello`。以此类推，最终我们就可以得到一个被中间件层层处理后的全新 `dispatch`。

还有一个问题是，我们刚刚讲了，中间件数组的最后一个方法先执行，返回一个全新的 `dispatch`，那执行顺序就是从最后一个到第一个吗？并不是，中间件函数返回的只是一个处理后的 dispatch 方法，而这个方法只有被上一级调用的时候，才会执行被修改后的 dispatch 的内容。

所以，**第一个中间件包裹在最外层，也是第一个执行的。**

类似于下面这种结构：

```js
action => {
  // a
  (actionB => {
    // b
    (actionC => {
      // c
      (actionD => {
        // d
        store.dispatch(actionD)
        // d
      })(actionC)
      // c
    })(actionB)
    // b
  })(action)
  // a
}
```

当然，因为很多中间件都返回了上一个 `dispatch` 方法的结果，所以不存在上图中的下半部分。

