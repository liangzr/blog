---
title: JavaScript 的类型
date: 2019-04-08 08:24:12
categories:
 - javascript
 - basic
---

在接触一门语言之前，首先要了解的就是这门语言的语法和数据类型，在 JavaScript 中，变量都是动态类型的，每个变量的值都属于一种语言类型。

## 类型

JavaScript 规定了七种语言类型:

1. Undefined
2. Null
3. Boolean
4. String
5. Number
6. Symbol
7. Object

其中前六种为基本类型，Symbol 是 ES6 新加入的。可能你已经对这些基础的基础已经耳熟能详了，但是不是还有一些你不知道的细节呢？

<!-- more -->

> 下面的内容会经常涉及值和变量，比如 undefined 值与 undefined 变量，关于这两种表示的区分请看[附录A](#附录A)

### Undefined、Null

Undefined 类型表示未定义，任何变量在声明之后都是 Undefined 类型，Undefined 类型也只有一个值，就是 undefined。一般我们可以使用全局变量 `undefined` 来引用这个值

然而，在 JavaScript 中，`undefined` 是一个全局变量，而不是一个保留字。尽管在**现代的浏览器中，这个全局变量是不可配置也不可写的**，但它确实可以在内层作用域中被覆盖，比如：

```js
function foo () {
  var undefined = 'defined'
  console.log(undefined)
}
foo() // 'defined'
```

那如何保证我们引用的 undefined 一定没有被上层作用域修改过呢？

#### 使用全局变量

我们可以使用 `window.undefined` 来获得 undefined 值，这样就可以直接跳过作用域链的查找，直接获取 JavaScript 引擎预定义的 `undefined` 变量。但这样就缺点就是每次都要使用 widnow 来引用这个值

#### 利用语法特性

Undefined 类型在 JavaScript 代码中处处可见，引擎在很多时候也会返回 Undefined 类型来表示未定义，那我们就可以利用这种机制，来获取 undefined 值：

- **void 0**：void 表示无返回，用 void 修饰在一个 RHS 表达式的前面，这个表达式最后返回的值就会是 undefined
- **未初始化变量**： 上面提到变量在声明之后，都是 Undefined 类型，所以我们可以声明一个变量，并且不对它初始化值
- **无返回的函数**：任何无返回的函数，都会返回一个 Undefined 类型

例如：

```js
console.log(void 0) // undefined

var no_value
console.log(no_value) // undefined

console.log((function () {})()) // undefined
```

Null 和 Undefined 在 JavaScript 中的含义是有差别的，它表示一个空引用的对象

Null 类型只有一个值 null，并且这个 JavaScript 中 null 是一个保留字，所以可以放心的引用它。

### Boolean

Boolean 类型只有两个值，true 和 false，用于表示逻辑上的真和假，同样有两个关键字 true 和 false 来表示这两个值。

### String

String 用于表达文本数据，一段 String 数据其实是由一组 UTF16 编码的元素组成的，这样的元素最长有 2^53 - 1 个。

JavaScript 字符把每个 UTF16 单元当作一个字符来处理，而一个 UTF16 字符的编码范围在 0-65536（U+0000 - U+FFFF）之间，所以处理基本字符区域（BMP）之外的字符时，要格外小心长度问题。

### Number

Number 类型表示通常意义上的数字，对应数学中的有理数，但对于计算机来说，这个数值是有精度限制的。

JavaScript 中的 Number 类型有 2^64 - 2^53 + 3 个值，基本符合 IEEE 754-2008 规定的双精度浮点数规则，但是 JavaScript 为了表达几个额外的场景，规定了几个例外的情况：

- NaN，占用了 9007199254740990，这个数字原本是符合 IEEE 规则的数字
- Infinity，无穷大
- -Infinity，负无穷大

#### 浮点精度

在 IEEE 754 标准的 64 位 double 双精度浮点数中，符号（Sign）位占 1 位，指数（Exponent）位占 11 位，尾数（Mantissa）位占 52 位。由于是二进制的科学计数表示，实数位永远为 1，例如：

```none
27 -> 11011 -> 1.1011 * 2^4

S: 1
E: 1023 + 4
M: 1011
```
在舍去了整数部分的 1 之后，尾数位为 1011，由此可以看出，Number 能表示的最大精度是 2^(52+1) - 1 ，也就是 9007199254740991（Number.MAX_SAFE_INTEGER），超过这个数字的整数就将丢失精度。

同样根据浮点数的定义，非整数的 Number 类型无法用 ==（或 ===） 来比较，下面看一个著名的问题：

```js
console.log(0.1 + 0.2 == 0.3) // false
```

这是因为如上所说的浮点精度的问题导致的，但这里错误的不是结论，因为这种误差在浮点运算中本来就存在，正确比较它们相等的办法是：

```js
console.log(Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON) // true
```

检查等式左右两边差的绝对值是否小于最小精度，才是正确比较浮点数的方法。

### Symbol

Symbol 是 ES6 中引入的类型，它是一切非字符串的对象 key 的集合，在 ES6 规范中，整个对象系统被用 Symbol 重塑。

Symbol 类型可以具体字符串类型的描述，但即使描述是相同的，Symbol 也不相等。可以用全局的 Symbol 函数创建 Symbol：

```js
var s = Symbol('symbolA')
```

在一些标准中提到的 Symbol，可以在全局的 Symbol 函数的属性中找到。例如，我们可以用 `Symbol.iterator` 来自定义迭代器的行为：

```js
var o = new Object

o[Symbol.iterator] = function() {
    var v = 0
    return {
        next: function() {
            return { value: v++, done: v > 10 }
        }
    }        
};

for(var v of o) 
    console.log(v); // 0 1 2 3 ... 9
```

代码中我们定义了对象的 `Symbol.iterator` 属性后，就可以使用 for...of 迭代这个对象了

### Object

Object 是 JavaScript 中最复杂的类型，也是 JavaScript 的核心机制之一，它表示对象，是一切有形和无形物体的总称。

在 JavaScript 中，对象的定义是**属性的集合**。属性分为数据属性和访问器属性，二者都是 key-value 结构，key 可以是字符串或者 Symbol 类型。



## 类型转换

因为 JavaScript 是弱类型的语言，所以在很多情况下都会进行类型转换，尤其是一些运算场景。

虽然大部分的转换符合我们的直觉，但隐式转换带来的失误依然不可忽视，其实最著名的就是 `==` 运算符，它试图跨类型比较，规划复杂到没人记得住，甚至还有人专门总结了 JavaScript 真值表来辅助记忆。但这里我认为，在大多数场景下，我们应该尽量避免类型的隐式转换，隐式转换的过程较为隐匿，可读性也差。

除此之外，如加减乘除和比较运算，也都会涉及类型转换，这种情况的类型转换还是相对简单的：

![](https://ws2.sinaimg.cn/large/006tNc79gy1g1xljiojnwj30vb0cftai.jpg)

在这里面较为复杂的是 Number 和 String 之间的转换，以前对象跟基本类型之间的转换，我们来分别看一下这几种转换的规则。

### StringToNumber

字符串到数字的类型转换，存在一个语法结构，类型转换支持十进制、二进制、八进制和十六进制，比如：

- 30
- 0b111
- 0o12
- 0xFF

此外，JavaScript 支持的字符串语法还包括正负号的科学计数法，可以用 E 或 e 来表示：

- 1e3
- -1e-2

需要注意的是，parseInt 和 parseFloat 并不支持这个转换，支持的语法也不相同。在不传入第二个参数的情况下，parseInt 只支持 16 进制的 0x 前缀，而且会忽略非数字字符，也不支持科学计数法，所以在任何情况下，都建议传入 parseInt 的第二个参数，而 parseFloat 则只支持十进制。

所以在多数情况下，使用 Number 方法是比 parseInt 和 parseFloat 更好的选择。

### NumberToString

在较小的范围里，数字到字符串的转换完全符合你直觉的表示，而当 Number 绝对值较大或较小时，则会用科学计数法表示。

### 装箱转换

Number、String、Boolean、Symbol 这几种基本类型在对象中都有其对应的类，所以装箱转换，正是把基本类型转换成对应的对象，这在很多语言中都能见到。

Symbol 函数无法使用 new 来调用，但我们也可以利用装箱机制来得到一个 Symbol 对象，比如用函数的 call 方法来强迫产生装箱。

```js
var symbolObj = (function () { return this }).call(Symbol('a'))

console.log(typeof symbolObj) // 'object'
console.log(symbolObj.description) // 'a'
console.log(symbolObj instanceof Symbol) // true
```

装箱机制会频繁地产生临时对象，在一些对性能有要求的场景下，我们应该尽量避免对基本类型做装箱转换

使用内置的 Object 函数，我们可以在 JavaScript 代码中显式的调用装箱能力

```js
var symbolObj = Object(Symbol('a'))

console.log(typeof symbolObj) // 'object'
console.log(symbolObj.description) // 'a'
console.log(symbolObj instanceof Symbol) // true
```

每一类装箱对象都有私有的 Class 属性，这个属性可以用 Object.prototype.toString 获取：

```js
var symbolObj = Object(Symbol('a'))

console.log(Object.prototype.toString.call(symbolObj)) // [object Symbol]
```

Class 属性是 JavaScript 引擎的私有属性，没有任何方法可以改变这个值，相比 typeof 和 instanceof，它可以准确地显示对象对应的类型，更为准确。

不过需要注意的是，由于 call 本身会导致装箱操作，如果要判断一个值是不是基本类型，以及是哪个基本类型，还需要配置 typeof 来区分。

```js
var symbolObj = Object(Symbol('a'))
var symbol = Symbol('a')

console.log(typeof symbolObj) // 'object'
console.log(Object.prototype.toString.call(symbolObj)) // [object Symbol]

console.log(typeof symbol) // 'symbol'
console.log(Object.prototype.toString.call(symbol)) // [object Symbol]
```

### 拆箱转换

在 JavaScript 标签中，规定了 ToPrimitive 函数，它是对象类型到基本类型的转换。

对象到 String 和 Number 的转换都遵循先**先拆箱再转换**的原则。通过拆箱转换，把对象转换为基本类型，再从基本类型转换为对应的 String 或 Number。

拆箱转换会尝试调用 valueOf 或 toString 来获取拆箱后的基本类型。如果 valueOf 和 toString 都不存在，或者没有返回基本类型，则会抛出类型错误 TypeError。

```js
var o = {
  valueOf ()  {
    console.log('valueOf')
    return {}
  }
  toString ()  {
    console.log('toString')
    return {}
  }
}

o * 2
// valueOf
// toString
// TypeError
```

如果是到 String 的拆箱，则会优先调用 toString:

```js
var o = {
  valueOf ()  {
    console.log('valueOf')
    return {}
  },
  toString ()  {
    console.log('toString')
    return {}
  },
}

String(o)
// toString
// valueOf
// TypeError
```

在 ES6 之后，我们可以通过 Symbol 显式的覆盖默认行为：

```js
var o = {
  valueOf ()  {
    console.log('valueOf')
    return {}
  },
  toString ()  {
    console.log('toString')
    return {}
  },
  [Symbol.toPrimitive] () {
    console.log('toPrimitive')
    return 'hello'
  }
}

console.log(o + '')
// toPrimitive
// hello
```

## 结语

以上七种类型就是 JavaScript 的七种语言类型，除此之外，还有七种规范类型：

- List 和 Record：用于描述函数的传参过程
- Set：主要用于解释字符集等
- Completion Record：用于描述异常、跳出语句执行过程
- Reference：用于描述对象属性访问、delete 等
- Property Descriptor：用于描述对象的属性
- Lexical Environment 和 Enviorment Record：用于描述变量和作用域
- Data Block：用于描述二进制数据


## 附录

### 附录A

在 JavaScript 语法中，LHS/RHS 是 Left/Right-Hand Side 的缩写，意思是要被赋值的变量（LHS）和变量要被赋的值（RHS）

```js
var a = 1
var b = a + 1
```

在上面这两行语句中，`var a = 1` 可以分为 `var a; a = 1` 在这里 `a` 就是一个 LHS，而 `1` 和 `a + 1` 是一个 RHS。所以如果是 undefined

```js
console.log(a === undefined)
```
undefined 并不是 JavaScript 的保留字，所以这里的 undefined 只是一个变量，而 null 则是保留字，不可作为变量名

```js
null = 1 // Uncaught ReferenceError: Invalid left-hand side in assignment
undefined = 1 // 1
1 = 1 // Uncaught ReferenceError: Invalid left-hand side in assignment 
```

可以看到所有值类型都不可作为 LHS

更多有关 LHS/RHS 的理解请参考[Scope & Closures - You Dont Know JS](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch1.md)


## 参考

- [JavaScript类型：关于类型，有哪些你不知道的细节？- 重学前端](https://time.geekbang.org/column/article/78884)
- [6.1 ECMAScript Language Types - ECMA262](https://tc39.github.io/ecma262/#sec-ecmascript-language-types)
- [JavaScript data types and data structures - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
