---
title: 如何按原顺序打印出对象的属性？
toc: true
tags: 
  - javascript
  - v8
date: 2019-03-09 00:00:00
---

昨天在群里看到有人问:

> 网友：“**Object.keys会给值排序，那用哪个方法取对象属性能不排序的？**”
> 我：“对象的属性有顺序吗？”
> 网友：“这个就会按照从小到大排序，我只是想保持原样~~” (如下)
> 我："for...in 应该不会" 
> ......

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0wixmegiij30gk02vglu.jpg)

结果我试了下发现 `for..in` 也会，最终我试了六种方法：

```js
const obj = { 100: 'a', 2: 'b', 7: 'c' }
 
Object.keys(obj)                          // ["2", "7", "100"]
Object.values(obj)                        // ["b", "c", "a"]
Object.entries(obj)                       // "2,b,7,c,100,a", toString() 之后
for (key in obj) { console.log(key) }     // 2, 7, 10
Object.getOwnPropertyNames(obj)           // ["2", "7", "100"]
Reflect.ownKeys(obj)                      // ["2", "7", "100"]
```

可以看到，以上方法都无一例外地以 `{ 2: 'b', 7: 'c', 100: 'a' }` 的方式打印出了相关值，那这个问题的影响在哪里呢？

<!-- more -->

假如你从接口中获取一段 JSON 数据如下：

```js
{
  "100": { ... },
  "2": { ... },
  "7": { ... }
}
```

上面个数据可能是经过后端排序的，并且数据中并没有带有可供排序的信息，毫无疑问经过 JS 的重新排序后，它的排序信息就丢失了，假如我就是不想丢失呢？

欲知其然，先知其所以然。在了解它如果遍历属性之前，首先我们需要知道的是，在 V8 中对象是如何存储属性的呢？

## V8 中对象的属性

在 JavaScript 中，大部分时候对象的行为类似一个字典，它以字符串做为键名，以任意对象作为值。虽然在迭代的时候，规范约定了以不同的方式处理整数索引属性和其他属性。

下面我们先来解释下整数索引属性和命名属性的区别。

### Named properties vs. elements

先来假设一个简单的对象 `{a: 'foo', b: 'bar'}`。该对象有两个命名属性,`a` 和 `b`，它没有整数索引。整数索引属性（通常叫做元素element）在数组中比较常见，如 `['foo', 'bar']` 有两个整数索引，分别为 0 和 1。这是 V8 处理属性的第一个主要区别。

![](https://v8.dev/_img/fast-properties/jsobject.png)

元素和属性存储在两个独立的数据结构中，这使得添加和访问属性或元素，在不同的场景下都更有效率。

元素主要用于 `Array.prototype` 的各种方法，鉴于这些函数访问的是连范围内的属性，V8 在内部也将他们表示为简单数组（在大多数情况下是这样的，有时会切换到基于稀疏字典的形式来节省内存）

命名属性以类似的方式存储在单独的数组中。但是与元素不同的是，我们不能使用简单的键来推断他们在属性数组中的位置，我们需要一些额外的元数据。在 V8 中，每个 JavaScript 对象都有一个关联的 HiddenClass，它用来存储对象的结构信息，以及从属性名到属性数组的索引的一个映射关系。对于复杂的情况，通常会使用一个字典来存储属性信息，而不是一个简单的数组。

> 更详细的内容请阅读 V8 博客的文章 [Fast properties in V8](https://v8.dev/blog/fast-properties)

## 如何遍历对象的属性

通过查询 ECMA 262 规范我们可以看到，第一节中我们使用的六种遍历属性的方法，在类似的情况下，最终都会返回 `Obj.[[OwnPropertyKeys]]` 的结果。

按照 ECMA 262 中对 `[[OwnPropertyKeys]]` 的[定义](https://tc39.github.io/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots-ownpropertykeys)：

> When the `[[OwnPropertyKeys]]` internal method of O is called, the following steps are taken:
> 1. Return ! OrdinaryOwnPropertyKeys(O).

它返回了一个 `OrdinaryOwnPropertyKeys(O)` 的处理结果，而 `OrdinaryOwnPropertyKeys(O)` 的执行过程则是：

> When the abstract operation OrdinaryOwnPropertyKeys is called with Object O, the following steps are taken:
> 1. Let `keys` be a new empty List.
> 2. For each own property key `P` of `O` that is an array index, in ascending numeric index order, do
>   a. Add `P` as the last element of `keys`.
> 3. For each own property key `P` of `O` that is a String but is not an array index, in ascending chronological order of property creation, do
>   a. Add `P` as the last element of `keys`.
> 4. For each own property key `P` of `O` that is a Symbol, in ascending chronological order of property creation, do
>   b. Add P as the last element of `keys`.
> 5. Return `keys`.

我们来简单描述下上述过程就是：首先创建一个名为 `keys` 的空数组，然后先遍历对象中的数组索引的属性，结果以升序排列，并逐个放入 `keys` 中；再遍历字符串属性（但不是数组索引），以属性创建时间升序排列，并逐个放入 `keys` 中去；然后再遍历 Symbol 类型的属性名，同样以属性创建时间升序排列，放入 `keys` 中，最后返回 `keys` 数组。

下来我们来验证一下：

```js
var a = {
  b: 1,
  a: 2,
  c: 3,
  7: 4,
  1: 5,
  10: 6,
  [Symbol('a')]: 7
}
a.d = 8
a[Symbol('b')] = 9

Reflect.ownKeys(a)
```

output(devtools):

```js
(9) ["1", "7", "10", "b", "a", "c", "d", Symbol(a), Symbol(b)]
  0: "1"
  1: "7"
  2: "10"
  3: "b"
  4: "a"
  5: "c"
  6: "d"
  7: Symbol(a)
  8: Symbol(b)
  length: 9
```

Chrome 的实现与规范的约定完全一致😕，所以至此我们知道它为什么打印出来是升序的了。

另外引用 Chromium 社区上 [Issue 164: Wrong order in Object properties interation](https://bugs.chromium.org/p/v8/issues/detail?id=164) 的讨论所述：

> There seems to be a widespread feeling that this used to work the way people expected it, but then the V8 team broke it in order to be mean.
> 
> What actually happened was that originally the order was completely arbitrary in V8.  At a later point it was changed so that non-numeric indices were in insertion order, and numeric indices were sometimes in insertion order.  Whether or not the numeric indices were in in insertion order was dependent on internal V8 heuristics that decide whether to use an array or a hash map implementation for the numeric indices.  Making heuristics in the V8 implementation visible in this way was felt to be undesirable so it was normalized so that numeric indices were always iterated in numeric order regardless of the internal representation.  Numeric iteration order was always a possibility, but with the last change it was made predictable.
> 
> There has never been any difference between the internal representation or iteration order of arrays vs. other objects in V8.
>
> Here is an independent test of the way arrays and objects perform in various engines (a little out of date now): http://news.qooxdoo.org/javascript-array-performance-oddities-characteristics  If this bug ever gets 'fixed' you can wave goodbye to some of the nice performance results in that graph.

结合前面介绍的 V8 属性一节我们知道，数组属性总是存储在一个单独的空间（可能是数组，也可能是字典）。在这种情况下，始终以有序数组的状态输出键值，这样的结果是可预测的（始终一致）。并且在 V8 内部，数组的内部表示和迭代方式，和其它对象没有任何不同。

综上所讲，这样的内部实现，有性能的因素，也有历史原因。

## 有没有办法按原顺序打印？

讲了那么多，我就是想按原顺序打印怎么办？

首先如果目标结构已经是 JavaScript 对象，应该是没有办法了。我们回到最终的问题，如果我们有一串 JSON 数组，想把它按原序获得键值，可以怎么做？假如我们有串数据：

```json
{"100":"foo","2":"bar","7":"baz"}
```

首先能想到的一个简单的方法就是，自己写一个简单的 json-parser。

下面是一个简单的实现：

```js
const jsonString = '{"100":"foo","2":"bar","7":"baz"}'

const parseKeys = str => {
  const out = []
  const tokens = str.slice(1, -1).split(',')
  for (let i = 0; i < tokens.length; i += 1) {
    out.push(tokens[i].split(':')[0].slice(1, -1))  
  } 
  return out
}

// try
console.log(parseKeys(jsonString))  // ✅ ["100", "2", "7"]
```

看起来我们得到了想要的结果（yeah），但是如果 json 数组稍微复杂点儿呢？

```json
{"100":{"b":"foo"},"2":[1,2],"7":200}
```

我们再来重构下这个解析器：

```js
var parseKeys = (str, lvl = 1) => {
  let out = []
  let level = 0
  let matching = false
  let pair = []
  for (let i = 0; i < str.length; i += 1) {
    if (str.charAt(i) === '"' && level === lvl) {
      if (!matching) {
        pair[0] = i 
      }  else {
        pair[1] = i
        out.push([...pair])
      }
      matching = ~matching
    } else if (['{', '['].indexOf(str.charAt(i) > 0)) {
      level += 1
    } else if (['}', ']'].indexOf(str.charAt(i) > 0)) {
      level -= 1
    }
  }
  return out.map(pair => str.slice(pair[0], pair[1]))
}
```

output(devtools):

```js
["100", "2", "7"]
```

上面这个方法执行效率并不高，只是提出一种思路，当然我们的目标还是解析出 key，而不是完整的引入一个 json 解释器，那样可能得不偿失。

更高效的解决方法，我们之后再补充...

## TL; DR;

V8 在内部将命名属性和数组索引属性分开存储，并且数组和其它对象的内部实现和迭代机制是完全一致的。

由规范定义，对象在迭代的时候，总是以升序输出数组索引的属性。如果要解决这个问题，目前可能自己去解析 JSON 字符串。

更多问题的延伸讨论，请参考 Chromium 社区的 **Issue: 164** 讨论。

## Reference

- [ECMA262 Specification - OwnPropertyKeys](https://tc39.github.io/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots-ownpropertykeys)
- [Fast properties in V8](https://v8.dev/blog/fast-properties)
- [Fast for-in in V8](https://v8.dev/blog/fast-for-in)
- [Issue 164: Wrong order in Object properties interation](https://bugs.chromium.org/p/v8/issues/detail?id=164)
