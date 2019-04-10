---
title: HTML 的元信息标签
date: 2019-04-07 10:36:28
tags:
 - head meta
categories:
 - html
 - element
---

一个 HTML 文档通常有一组标签来描述文档自身的信息，供搜索引擎或浏览器去读取以更好地渲染页面，这就是元信息标签。

## 元信息标签的结构

一般来说，元信息标签都会放在 HTML 的 head 标签之内，其中又包括 base、meta 和 title 等具体的元信息，当然也包括和 script 和 style 相关的 script 和 link 等标签，下面我就逐一看下它们的使用场景。

### head

head 规定了自身必须为 html 的第一个元素，并且必须包含一个 title 标签，最多包含一个 base 标签。

> 如果文档作为一个 iframe 存在，或者有其它更高级地方式指定了文档的标题（比如 HTML 邮件的主题部分），则允许不包含 title 标签。

<!-- more -->

### base

base 标签用来为文档中所有的**相对路径的链接**指定了一个 baseURL，并且一个文档中只能有一个 base 标签。

base 标签有 `href` 和 `target` 两个属性：

- **href:** 用于相对路径的链接的 baseURL，如果指定了这个元素，则 base 必须放在任何具有 URL 值的标签之前。可以使用相对或绝对路径
- **target:** 用来指定当文档的链接或 form 导致的跳转时的位置，对没有指定 `target` 属性的元素有效
  - _self: 默认值，在当前页载入新页面
  - _blank: 在新标签（页面）载入新页面
  - _parent: 在父页面载入新页面，如果没有父页面，与 _self 同样在当前页面载入
  - _top: 同 _parent 行为类似，_top 指定的是最顶级的父页面

> 如果有多个 base 标签被指定，只有第一个出现的 href 和 target 会生效

例如有下面一个页面结构:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>test</title>
  <base href="http://baidu.com" target="_blank">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="shortcut icon" href="/favicon.ico" type="image/x-icon">
</head>
<body>
  <a href="/favicon.ico">
    <img src="/favicon.ico"> 
  </a>
</body>
</html>
```
<img style="max-width: 500px; " src="https://ws2.sinaimg.cn/large/006tNc79ly1g1u1cdr77mj30om0f6aec.jpg">

可以看到，网站的图标和 img 标签内的图片，都变成了百度的图标，a 链接复制出来也是 `http://baidu.com/favicon.ico`

**在实际使用中，并不建议使用 base 标签，它改变了全局相对链接的地址，容易在跟 JavaScript 配合时出现问题**

### title

title 标签作为文档的元信息，用来简单的概括文档的主题，显示在浏览器的头部或者标签上。title 标签内只允许普通文本，放入标签会被忽略。

### meta

meta 标签是一组键值对，作为其它元标签的补充，表示通用的文档的元信息

在 head 中可以有任意多个 mete 标签，一般的 mete 标签由 name 和 content 两个属性来定义。name 表示元信息的名，content 用来表示元信息的值。

name 是一种比较自由的约定，HTML 规定了一些 name 作为大家的共识，也鼓励大家发明自己的 name 来使用。除了基本用法，meta 标签还有一些变体用来简化书写方式或声明自动化武行为，例如 charset、http-equiv 等

## meta 标签变体

### viewport 作为 name 属性

viewport 属性并没有在 HTML 标准中定义，却是移动端开发的事实标准

| 键 | 取值 | 描述 |
| :--- | :--- | :--- |
| width | 正数，或者 'device-width' | 定义视图的像素宽度 |
| height | 正数，或者 'device-height' | 定义视图的像素高度 |
| initial-scale | 0.0-1.0 之间的数字 | 定义视力的初始缩放比例 |
| maximum-scale | 0.0-1.0 之间的数字 | 定义视力的最大缩放比例 |
| minimum-scale | 0.0-1.0 之间的数字 | 定义视力的最小缩放比例 |
| user-scalable | 'yes' 或 'no' | 如果定义为 'no'，则不允许用户缩放页面。浏览器可能会忽略此规则，iOS10+ 默认忽略 |

### charset 属性

从 HTML5 开始，为了简化写法 meta 新增了 charset 属性，可以指定 HTML 文档的编码格式，**建议将其放在整个 head 标签内的第一位**。

```html
<meta charset="UTF-8">
```
一般情况下，http 服务端会通过 http 头来指定正确的编码方式，但是有些特殊的情况如使用 file 协议打开一个 HTML 文件，则没有 http 头，这种时候，charset meta 就非常重要了。

### http-equiv 属性

http-equiv 属性用来指定一个编译命令。之所以叫 http-equiv，是因为所有允许的值，都是特定的 HTTP 头的名字

- **content-type**
  - 它用来定义文档的 MIME 类型，后面可以跟一个字符编码。
  - 🔴**不推荐**，由于 html 文档基本也只支持 **text/html** 这一种类型，所以不如用 charset 属性来代替这个 http 头
- **x-ua-compatible**
  - 声明 UA 兼容性 
  - ✅**推荐** 
- **content-language** 
  - 用来指定文档的语言 
  - 🔴**不推荐**，优先使用 html 标签的 **lang** 属性 |
- **content-security-policy** 
  - 允许页面作者为当前页面定义内容策略。内容策略主要指定允许的服务器源和脚本端点，以帮助防止跨站点脚本攻击。
  - ✅**推荐** 
- **refresh** 
  - 刷新或重定向页面
  - ✅**推荐** |
- **set-cookie** 
  - 为页面定义 Cookie，需要符合 HTTP 的标准 
  - 🔴**不推荐**，建议使用 HTTP 的 **set-cookie** 头来代替它 




## 应用场景示例

### 移动端设备缩放

浏览器的 viewport 是可以看到Web内容的窗口区域，通常与渲染出的页面的大小不同，这种情况下，浏览器会提供滚动条以滚动访问所有内容。Apple 在 Safari iOS 中引入了“viewport meta 标签”，让Web开发人员控制视口的大小和比例。

移动端常用 viewport 写法示例：

```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no">
```

### SEO 优化

SEO（Search Engine Optimization）:汉译为搜索引擎优化。搜索引擎优化是一种利用搜索引擎的搜索规则来提高目前网站在有关搜索引擎内的自然排名的方式。

#### keywords 和 description

keywords 和 decription 是最常见的两个 meta 标签，用来标识一个页面的基本信息与关键字，方便搜索引擎来检索，可以提高被搜索引擎收录的效果。

[IT 之家](https://www.ithome.com) 页面的 meta:

```html
<meta name="description" content="IT之家，青岛软媒旗下，IT科技门户网站。快速精选IT新闻，实时报道科技周边，关注苹果iOS、谷歌Android、微软Windows Phone，紧盯iPhone/iPad、安卓智能设备、WP手机等数码潮流。技术资讯、攻略教程、资源下载 - IT人的生活，尽在IT之家，爱IT，爱这里。" />
<meta name="keywords" content="IT新闻,互联网,Internet,数码,科技,科普,通信,智能手机,IT之家,ithome,软媒" />
```

百度搜索 IT 之家：

![](https://ws1.sinaimg.cn/large/006tNc79gy1g1u47z6xjwj30w407gq5c.jpg)

Google 搜索 IT 之家：

![](https://ws1.sinaimg.cn/large/006tNc79gy1g1u48pgncgj30y006eac4.jpg)

可以看到搜索引擎对网站的描述信息都来自 meta 里的 description。（🐷百度还把中文逗号转换成了英文逗号）

#### 社交媒体优化

我们先来看下 hexo 为这篇文章生成的 meta 信息

```html
<meta name="description" content="一个 HTML 文档通常有一组标签来描述文档自身的信息，供搜索引擎或浏览器去读取以更好地渲染页面，这就是元信息标签。 元信息标签的结构一般来说，元信息标签都会放在 HTML 的 head 标签之内，其中又包括 base、meta 和 title 等具体的元信息，当然也包括和 script 和 style 相关的 script 和 link 等标签，下面我就逐一看下它们的使用场景。 headhead">
<meta name="keywords" content="head meta">
<meta property="og:type" content="article">
<meta property="og:title" content="HTML 的元信息标签">
<meta property="og:url" content="https://blog.liangzr.tech/2019/04/07/html-head-meta/index.html">
<meta property="og:site_name" content="OVERLOADING 🔥">
<meta property="og:description" content="一个 HTML 文档通常有一组标签来描述文档自身的信息，供搜索引擎或浏览器去读取以更好地渲染页面，这就是元信息标签。 元信息标签的结构一般来说，元信息标签都会放在 HTML 的 head 标签之内，其中又包括 base、meta 和 title 等具体的元信息，当然也包括和 script 和 style 相关的 script 和 link 等标签，下面我就逐一看下它们的使用场景。 headhead">
<meta property="og:locale" content="default">
<meta property="og:image" content="https://ws2.sinaimg.cn/large/006tNc79ly1g1u1cdr77mj30om0f6aec.jpg">
<meta property="og:updated_time" content="2019-04-07T07:15:56.797Z">
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="HTML 的元信息标签">
<meta name="twitter:description" content="一个 HTML 文档通常有一组标签来描述文档自身的信息，供搜索引擎或浏览器去读取以更好地渲染页面，这就是元信息标签。 元信息标签的结构一般来说，元信息标签都会放在 HTML 的 head 标签之内，其中又包括 base、meta 和 title 等具体的元信息，当然也包括和 script 和 style 相关的 script 和 link 等标签，下面我就逐一看下它们的使用场景。 headhead">
<meta name="twitter:image" content="https://ws2.sinaimg.cn/large/006tNc79ly1g1u1cdr77mj30om0f6aec.jpg">
```

可以看到有两种标签我们没有见过，分别是 `name="twitter:x"` 和 `property="og:x"`，下面就分别看下这两个是什么用途

[Open Graph协议](ogp.me)是由 Facebook 推出的，使任何网页都成为社交图中的丰富对象。例如，这在Facebook上用于允许任何网页具有与Facebook上的任何其他对象相同的功能。

可以理解为，当你分享一个网页到 Facebook 中时，即使它不是 Facebook 自己的网页，也可以有足够的信息像展示预览 Facebook 自己的网页具有更丰富的信息。

同理 twitter 也推出了自己的[卡片优化协议](https://developer.twitter.com/en/docs/tweets/optimize-with-cards/guides/getting-started.html)，提升了 Twitter 上的整体卡片体验，每种卡片类型都支持并需要一组特定的属性。

### 其它 meta 功能

- application-name: 定义应用名称
- author: 页面作者
- generator: 生成页面所使用的工具，主要用于可视化编辑器，如果是手写 HTML 的网页，不需要加这个 meta
- referrer: 跳转策略，是一种安全考量
- theme-color: 页面风格颜色，实际并不会影响页面，但是浏览器可能据此调整页面之外的 UI（如窗口边框或者 tab 的颜色）
- robots: 定义页面的爬虫行为

## 参考

- [HTML元信息类标签：你知道head里一共能写哪几种标签吗？- 重学前端](https://time.geekbang.org/column/article/82711)
- [在移动浏览器中使用viewport元标签控制布局 - MDN](https://developer.mozilla.org/zh-CN/docs/Mobile/Viewport_meta_tag)
- [meta - MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta)


