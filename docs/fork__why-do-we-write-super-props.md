---
title: 「译」Why Do We Write super(props)?
toc: true
tags: 
  - react
categories:
  - translated
  - react
date: 2018-11-30 00:00:00
---

> 本文转载并翻译于 [Dan Abramov](https://twitter.com/dan_abramov) 的博客 [overreacted.io](https://overreacted.io/why-do-we-write-super-props/) 

[React Hooks](https://reactjs.org/docs/hooks-intro.html) 是当下社区的热门，但我却想从 *class* 组件的一些有趣实现讲起！

理解这些内容对你如何运用 React 来说并不重要，但如果你喜欢探寻事物动作的原理的话，就很有趣了。

---

我写过不计其数的 `super(props)` :

```js
class Checkbox extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOn: true };
  }
  // ...
}
```

当然，使用 [class fields proposal](https://github.com/tc39/proposal-class-fields) 可以让我省去这种写法：

```js
class Checkbox extends React.Component {
  state = { isOn: true };
  // ...
}
```

React 从 0.13 增加对普通 class 的支持开始，就计划要使用这种语法。现在这种定义 `constructor` 然后调用 `super(props)` 的做法只是 `class field` 来临之前的一种替代方案。

<!-- more -->

但是现在让我们回到这个例子，只使用 ES2015 的特性：

```js
lass Checkbox extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOn: true };
  }
  // ...
}
```

为什么我们要调用 `super`？可以省去吗？如果我们不得不调用它，不传 `props` 会怎么样？除了 `props` 还需要传递别的参数吗？我们来看一下这个问题。

---

在 JavaScript 中, `super` 指的是父类的 constructor（在我们的例子中，它指的就是 React.Component）

要注意的是，在 constructor 中，你只能在调用父类的 constructor 之后才能使用 `this`，JavaScript 不会让你这样做：

```js
class Checkbox extends React.Component {
  constructor(props) {
    // 🔴 Can’t use `this` yet
    super(props);
    // ✅ Now it’s okay though
    this.state = { isOn: true };
  }
  // ...
}
```

有一个很好的理由说明为什么 JavaScript 强制你在使用 `this` 之前要执行父类的 constructor，参考下面这个类结构：

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues(); // 🔴 This is disallowed, read below why
    super(name);
  }
  greetColleagues() {
    alert('Good morning folks!');
  }
}
```

想象一下，如果你在调用 `super` 之前就调用了 `this`，一个月之后，我们可能修改了 `greetColleagues` 方法：

```js
greetColleagues() {
  alert('Good morning folks!');
  alert('My name is ' + this.name + ', nice to meet you!');
}
```

但我们忘记了 `this.greetColleagues()` 是在 `super()` 之前调用的，这个时候 `this.name` 还没有被定义！所以像这样的代码就很不好维护。

为了避开这个陷阱，**JavaScript 强制你如果想要在 constructor 中使用 `this`，则必须先调用 `super`**，让父类先处理完该做的事情！这个限制也适用于使用类来定义 React 组件：

```js
constructor(props) {
  super(props);
  // ✅ Okay to use `this` now
  this.state = { isOn: true };
}
```

现在我们就只剩下另外一个问题，为什么要传递 `props` 参数？

---

你可能认为向 `super` 传递 `props` 是有必要的，以便 `React.Component` 的 constructor 可以初始化 `this.props`：

```js
// Inside React
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}
```

事实也差不多，准确地说，这是[它的过程](https://github.com/facebook/react/blob/1d25aa5787d4e19704c049c3cfa985d3b5190e0d/packages/react/src/ReactBaseClasses.js#L22)

但不知道为什么，即使你没有把 `props` 参数传递给 `super()`，你还是可以在 `render` 和其它方法中访问 `this.props`。（不信的话可以试试看）

所以它内部是如何运作的？事实上，React 会在你调用 constructor 之后重新赋值 `props`:

```js
// Inside React
const instance = new YourComponent(props);
instance.props = props;
```

所以即使你忘了把 `props` 传递给 `super()`，React 也会在之后正确的赋值，这就是原因。

当 React 支持 class 的时候，并不只是简单的添加了 ES6 的特性，它的目标是尽可能广泛抽象地支持 class。因为不能确定 ClojureScript、CoffeeScript、ES6、Fable、Scala.js、TypeScript 或其它的方式是如何定义一个组件的，所以 React 故意地不关心 `super()` 方法有没有被调用——尽管这是 ES6 class 规定的

所以，这是不是就意味着，你可以只写 `super()` 而不用 `super(props)` 了？

**恐怕并不是这样，因为它依然令人困惑。** 当然，React 会在 constructor 之后重新赋值 `this.props`，但是在 constructor 内部调用了 `super` 之后，`this.props` 依然是 `undefined`：

```js
// Inside React
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}

// Inside your code
class Button extends React.Component {
  constructor(props) {
    super(); // 😬 We forgot to pass props
    console.log(props);      // ✅ {}
    console.log(this.props); // 😬 undefined 
  }
  // ...
}
```

如果是在 constructor 中调用的某些方法中遇到了这种问题，就更难定位了。**这就是为什么要强调要坚持`super(props)`，即使没有严格限制**：

```js
class Button extends React.Component {
  constructor(props) {
    super(props); // ✅ We passed props
    console.log(props);      // ✅ {}
    console.log(this.props); // ✅ {}
  }
  // ...
}
```

这样可以保证 `this.props` 在 constructor 执行完之前就正确赋值。

---

还有最后一个 React 用户可能会困惑的问题。

你可能注意到了，当你使用 Context API 的时候（无论是旧的 `contextTypes` 或者在 React 16.6 中新添加的 `contextType` API），`context` 是 constructor 的第二个参数。

所以为什么我们不写成 `super(props, context)`？确实可以这样写，但是 context 用的相对比较少，所以也不容易碰到这个陷阱。

如果使用 class fields proposal 的话，这个陷阱就不复存在了。不需要特别地写一个 constructor，所以的变量都可以自动传递。这样也可以在 `state = {}` 中直接引用 `this.props` 或 `this.context` 了。

当然，使用 React Hooks 的话，我们甚至不需要使用 `super` 或 `this`，但这是以后的事儿了。

