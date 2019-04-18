---
title: 为什么要使用 redux-thunk
date: 2019-04-18 13:09:06
tags:
---

以前我一直不理解，为什么说 `redux-thunk` 是用来处理异步 action 的，没有它就不能处理异步 action 吗？

答案是可以的。

实际上，只要你能拿到 dispatch 方法，你可以在任何时候 dispatch 一个 action 对象。dispatch 方法是一个同步操作，它传递了 action 并且更新了 store，可以参考前面的文章[《Redux 源码分析》](https://blog.liangzr.tech/2019/04/17/redux-source-code/)。

这样我就更不明白了，既然这样，`redux-thunk` 究竟做了什么？

<!-- more -->

## redux-thunk 源码

`redux-thunk` 是一个标准的 Redux 中间件，源码也十分简单：

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
``` 

默认情况下，`redux-thunk` 返回的是一个标准的 Redux 中间件，它暴露了一个方法让你自己创建一个带参数的 thunk 中间件，我们先忽略这部分：

```js
const thunk =  ({ dispatch, getState }) => next => action => {
  if (typeof action === 'function') {
    return action(dispatch, getState, extraArgument);
  }

  return next(action);
};
```

通过[《Redux 中间件》](https://blog.liangzr.tech/2019/04/18/redux-middleware/) 一文我们得知。第一个箭头前面的 `dispatch` 和 `getState` 做为 **middlewareAPI** 传递进来，初始化中间件的。

下面的 `return next(action)` 也是调用了上层中间件返回的 `dispatch` 方法，所以关键在于这个判断：

```js
if (typeof action === 'function') {
  return action(dispatch, getState, extraArgument);
}
```

如果 action 是一个函数，则调用 action，并把初始化中间件时的两个参数，以及额外的参数，传递给了 action 函数。

源码就这些内容，想要彻底理解，还是要跟着例子来看。

## 异步 Action

假如我们有个 action 是这样的：

```js
const fetchUserAvatar = name => ({getState, dispatch}) => {
  fetch(`https://api.github.com/users/${name}`)
    .then(res => res.json())
    .then(({ avatar_url }) => {
      dispatch({
        type: 'UPDATE_USER_AVATAR',
        payload: avatar_url,
      })
    })
}
```

然后我们 dispatch 这个 action:

```js
dispatch(fetchUserAvatar('liangzr'))
```

因为 action 是一个方法，`redux-thunk` 就把相应的参数传给了 `fetchUserAvatar`，而 api 在返回后也成功 dispatch 了 action。

但是到这里，我起初有个疑惑，如果 `redux-thunk` 没有调用 `next` 方法，也就是下层生成的 dispatch 方法的话，后面不就不会经过中间件了吗？

## 到底是哪个 dispatch

这个问题就又要回到 `applyMiddleware` 方法内了：

```js
let dispatch = () => {}

const middlewareAPI = {
  getState: store.getState,
  dispatch: (...args) => dispatch(...args)
}
const chain = middlewares.map(middleware => middleware(middlewareAPI))
dispatch = compose(...chain)(store.dispatch)
```

可以看到，`dispatch` 变量在一开始就声明了，并且 `middlewareAPI` 的 dispatch 属性的方法中，通过闭包持有它的引用。最后 `dispatch` 变量又被赋值了 `compose` 处理后的最终 dispatch 方法。

也就是说，`middlewareAPI` 中的 dispatch 方法，最终会调用被层层中间件处理后的 dispatch。

那 `dispatch(fetchUserAvatar('liangzr'))` 的执行过程应该是：

![](/diagrams/redux-thunk/dispatch.svg)

可以看到 `redux-thunk` 其实执行了两次，第二次才真正地处理了中间件。并且由此也可以看出，因为我们上次说过，第一个中间件会被第一个执行，而 `redux-thunk` 也必须第一个执行，所以在使用的时候，始终要把它放到中间件的第一位。

## 为何不用 store.dispatch

首先，如果从外部引用 `store.dispatch` 的话，有两个问题：

1. **它强制了 store 必须是一个单例**：这对服务端渲染很不友好，因为对于不同的用户或者说请求，它都应该有个独立的 store
2. **不易于测试**：因为使用了外部引用的 store，你没办法方便地 mock 一个 store 用来测试，也不能控制 store 的状态

其次，使用 `redux-thunk` 中间件，赋予了 redux 来 dispatch 异步 action 的能力，你不用担心 dispatch 的 action 到底是同步还是异步的，通过 thunk 中间件，异步 action 自然可以惰性处理。

Redux 的作者 Dan Abramov 曾对这个问题进行过多次释疑，可以参考下面两个链接：

- [How to dispatch a Redux action with a timeout?](https://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559#35415559)
- [Why do we need middleware for async flow in Redux?](https://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux/34599594#34599594)