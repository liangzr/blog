---
title: 用 Immutable 数据来优化 React 应用
tags: 
  - immutable
  - optimizing
categories:
  - javascript
  - react
date: 2018-05-29 00:00:00
toc: true
---


一直以来，Virtual DOM 都是 React 的一大特色，Facebook 宣称 React 借其能很大程度提高 SPA 的性能表现。但这就意味着 React 的性能一定优秀吗，可能并不是，在某些情况下，React 慢的令人抓狂，这时你可能就需要用一些正确的手段来优化它了。

React 的更新机制
---

我们不妨先简单了解下 React 的更新机制，如果能降低它的更新频率，自然能大大提高整体渲染速度。

### Props & State

props 和 state 的基本概念不再赘述，组件的 props 是只读的，只能通过上层 JSX 声明时的属性传递进来，state 则完全受组件自身控制，并且只存在于 `class` 语法声明的组件。

无论是 props 还是 state 发生变化都可以触发组件更新，下面这些生命周期方法会在组件重新渲染时被依次调用：

- componentWillReceiveProps*
- static getDerivedStateFromProps
- shouldComponentUpdate
- componentWillUpdate*
- render
- getSnapshotBeforeUpdate
- componentDidUpdate

<!-- more -->

> `*` 号标注的生命周期方法将会在 React 17 移除，一旦调用了新的生命周期方法，这些方法将不会被调用。

![Update lifecycle](https://ws3.sinaimg.cn/large/006tKfTcgy1frj8i1dy0jj30aq0g9dgp.jpg)

从上面的生命周期中我们可以看到，`shouldComponentUpdate` 方法将在组件接收到新的 props 或者 state 时被调用。然而在默认情况下， 每次更新，React 都会去调用 **render** 方法重新生成 Virtual DOM 并通过 diff 算法计算出需要变动的部分，然后操作 DOM 完成这部分更新。

对于一些简单的 React 应用来说，每次 **render** 带来的消耗不会特别大，不过一旦你的应用有了一定规模，尤其是复杂的树形结构时，每次更新都会消耗不少的系统资源。

### shouleComponentUpdate（SCU）

我们先来看下[官方文档](https://reactjs.org/docs/optimizing-performance.html#shouldcomponentupdate-in-action)里的示意图。

![](https://ws3.sinaimg.cn/large/006tNc79gy1frsifbmx0kj30tc0oitci.jpg)

从图中可以看到，在这个简单的树形结构中，仅仅是 c7 的状态发生了改变，所有的组件都要进行一次 **render**，那如果我这个树下有 10 个组件呢，50 个呢？尤其当这个 c7 的状态变化与鼠标移动这种高频操作相关时，所有的组件不停的重新生成 Virtual DOM，这样能有多卡顿你能想象的到吗？不要问我是怎么知道的，某天 Leader 叫我写了个表单设计器……

如果不用 **SCU** 对 React 的更新进行限制，你可能像我之前一样，对着 Chrome 的 Perfomance 工具里锯齿般的火焰图束手无策。那假如 **SCU** 可以正确的感知数据变化并返回你期待的结果，实际情况又会如何呢？

![](https://ws2.sinaimg.cn/large/006tNc79gy1frsixpjyzij30sv0oi0w9.jpg)

如上所示，如果 **SCU** 正常工作，只会发生 3 次 Virtual DOM 的比较，换言之，只有发生改变的 c7 以及它的父级组件会进入 **render** 方法，生成 Virtual DOM。那这次如果我们有 100 个子组件，但 c7 的深度还是 3 呢？没错，它依然是只会调用 3 次 **render** 方法，在大型树形结构里，这样的渲染效率无疑是成几何倍提升。

那么问题又来了，**SCU** 是一定要实现的，但在每个组件中都手写 **SCU**，手动地比较复杂的对象中每个键的值，难度非同一般，那么如何轻松地让 **SCU** 返回你期待的结果？

解决思路
---

虽然完全手写 **SCU** 不现实，但这里依然有一些组合方案可以助我们实现目标。

### PureComponent

PureComponent 是 React 提供的另一个组件，它默认帮你实现了 **SCU** 方法，其实在它出现之前，它的前身是 React 的 addons 提供的 PureRenderMixin，它的源码如下：

```javascript
var shallowEqual = require('fbjs/lib/shallowEqual');

module.exports = {
  shouldComponentUpdate: function(nextProps, nextState) {
    return (
      !shallowEqual(this.props, nextProps) ||
      !shallowEqual(this.state, nextState)
    );
  }
};
```

 我们可以看到它帮我们实现了 **SCU** 方法，实现的机制是浅比较（Shallow Compare），也就是说，它只简单的比较了 `this.props` 和 `nextProps` 两个变量（以及他们的第一层子属性）引用的是否为同一个地址，如果是则返回 **false**，否则返回 **true**。

> `shallowEqual` 的具体实现请查阅[源码](https://github.com/facebook/fbjs/blob/master/packages/fbjs/src/core/shallowEqual.js)

同样的我们也来看下使用 PureComponent 时的具体实现：

```javascript
function checkShouldComponentUpdate(
  ...
) {
  const instance = workInProgress.stateNode;
  const ctor = workInProgress.type;
  // 用户自己实现
  if (typeof instance.shouldComponentUpdate === 'function') {
    const shouldUpdate = instance.shouldComponentUpdate(
      newProps,
      newState,
      newContext,
    );
    return shouldUpdate;
  }

  if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    return (
      !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
    );
  }

  return true;
}
```

可以看到，如果用户不定义 **SCU** 方法，并且当前组件为 PureComponent 时，最终依然会对新旧 Props 和 State 进行一个浅比较。

虽然 PureComponent 帮我们实现了 **SCU** 方法，但这并不意味着我们已经达到目标了，别忘了它只是实现了浅比较，在 JavaScript 中，Primitive 数据能直接的用 `=` 号简单的浅比较，而 Object 数据仅仅表示两个变量引用的堆地址相同，但这块儿内存中的数据有没有改动过，就无从得知了，看个简单的例子：

```javascript
oldState = { expand: true };
oldState.expand = false;
newState = oldState;

shallowEqual(newState, oldState) // true
```

  如上我们更新了 state 的 expand 的值，但 PureComponent 在比较时会认为 state 并没有更新返回 **SCU** 返回 `false`，这样我们的组件就得不到正确的更新了。

### 深拷贝就行了，是这样吗

可能比较有经验的童鞋会说，只要用深拷贝就行了，那我们来看下几种常见的深拷贝实现

#### JSON 之 stringify + parse

这个原理比较简单，序列化之后，对象变成了一个字符串，`JSON.parse` 会从字符串重新生成对象，很明显这已经不是之前那个对象了，实现了完全的深拷贝。但是别忘了，JSON 只有 6 种基本数据类型，这样转换很显然不少对象会出现问题，比如 Function 对象，Date 对象等等，都无法正常转换。可见这种方案的适用场景也是比较少的。

```javascript
const o = {
  a: 1,
  b: false,
  c: 'react',
  d: null,
  e: [1, 2],
  f: function () { console.log('Forget me!') },
  g: new Date(),
  h: /forget me/g,
  i: [1, new Date()],
  j: Symbol('Forget me'),
}
console.log(JSON.stringify(JSON.parse(o)));
```

Output:

![](https://wx2.sinaimg.cn/large/006tKfTcgy1frt9x393n1j30gf054t94.jpg)

#### lodash.cloneDeep

相较于用 JSON 粗暴的转换，lodash 的处理更为细致，Primitive 数据直接返回，Object 数据则逐一处理。

还是上面的例子，lodash 的输出结果：

![](https://ws4.sinaimg.cn/large/006tKfTcgy1frt9wmakpgj30fh05zmxp.jpg)

类似的还有 jQuery 的 extend 方法（第一个参数为 true 时为深拷贝）。虽然深拷贝帮我们重新处理了浅比较的问题，但当你使用的时候可能会发现，每次修改树形结构的里的一个值，所有的组件依然会全部渲染。这是因为树形结构中所有的对象引用地址都被改变了，PureComponent 在浅比较时，自然所有的 **SCU** 都会返回 ture，我们似乎又回到了起点，那如何只让变动的部分改变引用呢？

### 优雅的 Immutable 数据

Immutable 即不可变的，意思是对象创建后，无法通过简单的赋值更改值或引用。Facebook 推出了 ImmutableJS 来实现这套机制，它有自己的一套 API 来对已有的 Immutable 对象进行修改并返回一个全新的对象，但与深拷贝不同，这个对象只修改了变动的部分，示意如下：

![](https://ws2.sinaimg.cn/large/006tKfTcgy1frt9vy605zj30lz0ctq47.jpg)

#### ImmutableJS

Facebook 推荐使用 ImmutableJS 来优化 React 应用，但使用它的同时也意味需要重新学习大量的 API

#### Immutability-helper

Immutability-helper 原来是 React 的 addons 里面的 update 模块，独立出来后又新增了拓展模块，它提供了一种语法糖，你可以直接描述需要修改的对象，并且用预置命令对这部分进行修改，最后返回一个修改后的对象，以此来模拟 Immutable 数据的行为

extend 的行为与 Object.assign 一致：

```javascript
const newData = extend(myData, {
  x: extend(myData.x, {
    y: extend(myData.x.y, {z: 7}),
  }),
  a: extend(myData.a, {b: myData.a.b.concat(9)})
});
```

使用 immutability-helper:

```javascript
import update from 'immutability-helper';

const newData = update(myData, {
  x: {y: {z: {$set: 7}}},
  a: {b: {$push: [9]}}
});
```

可以看到通过这个库提供的语法糖，我们可以更快速清晰便捷的修改对象，而不用一层一层地用 Object.assign 之类的包起来。这种方式相较于 ImmutableJS 比较没有侵入性，性能也不比 Immutable 差多少（有待测试），没有学习成本，比较**推荐**！

### 小结

其实说到这里，本篇基本已经结束了，在 PureComponent 和 Immutable Data 的搭配使用下，**SCU** 能很大程度提高 React 应用的性能，不过这也只是从组件更新的角度来优化 React，实际上我们能做的事还有很多。

问题与建议
---

上文只是作者本人在 React 优化中的实践，翻阅网上的资料与源码总结而出的一篇分享，如有谬误欢迎指正！

参考
---

1. [Optimizing Performance - reactjs.org](https://reactjs.org/docs/optimizing-performance.html)
2. [React is Slow, React is Fast: Optimizing React Apps in Practice - Daily JS](https://medium.com/dailyjs/react-is-slow-react-is-fast-optimizing-react-apps-in-practice-394176a11fba)

### 图形素材

[optimizing-react-app-with-immutable-data.key](https://github.com/liangzr/blog/blob/master/assets/optimizing-react-app-with-immutable-data.key)