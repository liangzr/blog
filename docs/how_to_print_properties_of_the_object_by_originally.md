---
title: 如何按原顺序打印出对象的属性？
toc: true
tags: 
  - javascript
  - v8
description: 从一个问题出发，简单介绍了 V8 中存储对象属性的方式，以及命名属性和元素的区别
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



## 有没有办法按原顺序打印？

## TL; DR;

## Reference

- [Fast properties in V8](https://v8.dev/blog/fast-properties)
- [Fast for-in in V8](https://v8.dev/blog/fast-for-in)
- [Issue 164: Wrong order in Object properties interation](https://bugs.chromium.org/p/v8/issues/detail?id=164)
