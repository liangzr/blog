---
title: PlantUML 简介
date: 2019-04-17 09:07:56
tags:
categories:
  - tools
---

相信很多人都尝试过不少画图工具，但要么是扩展性不强，要么支持的类型单一，有些在线的画图工具还要收费，而且一些框框线线拖来拖去，还要调整大小、字体、线条和位置各种繁琐的操作，回过头来才发现，你不就是为了梳理内容才要画图吗，怎么就不能专注于内容呢？


## 画图工具对比

下面我先介绍几个我尝试过的画图工具，以及使用体验：

| 工具 | 支持类型 | 操作性 | 外观 | 导出格式 | 价格 |
| :--- | :--- | :--- | :--- | :--- |
| ProcessOn | 流程图、类图、思维导图、状态图等 | 一般/拖拽 | 一般、高度自定义 | png/jpg/pdf/pos/svg | 免费保存 5 个图 |
| MindNode/XMind | 思维导图 | 简单/快捷键 | 优秀 | png/pdf/text等 | 免费 |
| Markdown | 时序图、流程图 | 简单/代码 | 一般、固定样式 | 无 | 扩展语法，免费但有限支持 |
| Keynote | 任意 | 繁琐/拖拽 | 优秀、高度自定义 | key | 免费，限定 macOS |
| PlantUML | 时序图、用例图、类图、流程图、状态图、对思维导图等 | 简单/代码 | 一般、有限自定义 | png/svg/text | 开源 & 免费 |

<!-- more -->

就我自己的使用经验来说，ProcessOn 之类的专门画流程图的工具，线条框框拖拽起来太繁琐，很难专注于内容，而且样式一般。XMind 和 MindNode 虽然不错但只能画思维导图，列在这里是因为思维导图也是很重要的一种图。Keynote 虽然可以画出挺不错的图，但因为它本来最多算个作画工具而不是作图工具，所以线框和箭头之类都要自己画出来备用，操作繁琐。

Markdown 和 PlantUML 都是基于代码实时渲染绘图，对于一般人来说可能有点门槛，但对于程序员来说就不存在了。相比来说我比较喜欢这种方式，不用自己去拖拽线框的位置，逻辑顺序修改起来也非常方便。不过 Markdown 本身不支持作图，只有个别支持扩展语法的才能渲染出来，并且支持的类别也较少，语法也过于简单。所以接下来我们就要重点讲一下 PlantUML 和它的常规用法～

## PlantUML 简介

[PlantUML](http://plantuml.com/zh/) 是一个[开源项目](https://github.com/plantuml/plantuml)，支持从文本（代码）快速绘制 UML 图。

> UML-Unified Model Language 统一建模语言，又称标准建模语言。是用来对软件密集系统进行可视化建模的一种语言。

目前已经支持的 UML 图：

- **时序图**
- 用例图
- 类图
- **活动图**（流程图）
- 组件图
- 状态图
- 对象图
- 部署图 
- 定时图 

非 UML 图：

- 线框图形界面
- 架构图
- 规范和描述语言 (SDL)
- Ditaa 图
- 甘特图 
- **思维导图**
- 工作分解结构图
- 以 AsciiMath 或 JLaTeXMath 符号的数学公式


## 常用图绘制

### 时序图

时序图可以很直观地看到一个有序地操作流程

git-flow 流程图：

![](/diagrams/git-flow/git-flow.svg)

下面我们来跟着一个 TCP 握手的时序图来学习它的一般用法

```diagram
@startuml tcp-connect

header overloading
footer copyright liangzr

title TCP 握手协议

actor client 

participant server order 2
participant client order 1

autonumber
client -> server : SYN
note left
  客户端向服务端发送连接请求报文段，
  该报文段包含自身的数据通讯初始序号
end note

server -> client : SYN + ACK
note right
  服务端收到连接请求报文段后，如果同
  意连接，刚会发送一个应答，该应答中
  也会包含自身的数据通讯初始序号
end note

client -> server : ACK
note left
  客户端收到连接同意的应答后，还要向
  服务端发送一个确认报文
end note

@enduml
```

![](/diagrams/tcp/tcp-connect.svg)

`@startuml` 和 `@enduml` 用来表示大多数情况下 UML 图的开始与结束。

**`角色A -> 角色B : 过程` 这种语法表示了角色之间的过程信息**

`actor client` 定义了 client 的角色：

- actor
- boundary
- control
- entity
- database
- collections

一共六种角色可以选择，也可以不指定角色

`participant server order 2` 可以指定角色的排列顺序，数字小的排在左边

`header`、`footer` 和 `title` 可以定义一些页面信息, `note` 可以给节点添加注释

### 流程图

流程图是最常用的图之一，可以直观形象地描述一个流程的详细过程

先来看一个 React 生命周期的简单流程图：

![](/diagrams/react-lifestyle/react-lifestyle.svg)

这是一个最简单的流程图，下面我们来跟着一个 React 的组件挂载流程来认识一下完整的流程图写法

```diagram
@startuml react-mounting

|mountComponent挂载组件|
start

:初始化props和context;
:执行构造函数获得实例;

if (无状态组件OK) then (yes) 
  :渲染无状态组件;
  note left
    无状态组件没有状态更新队列
    只专注于渲染
  end note
endif

:重新初始化类属性;

note right
  为类抽象兜底，即使用户
  没有正确地调用 super()
end note

:初始化state;
:初始化更新队列;
note right
  _pendingStateQueue = null
  _pendingReplaceState = false
  _pendingForceUpdate = false
end note

|#FAEBD7|performIntialMount执行挂载|

#82F49C:componentWillMount;

if (更新队列不为空) then (yes) 
  :合并state;    
  note left
    此时不会多次render
  end note

endif

if (无状态组件) then (no) 
  :创建Component实例;    
endif

#82F49C:递归render;
note right
  调用 ReactReconciler.mountComponent
  递归地渲染子组件，子组件渲染成功后，父组件
  才渲染完成
end note

|mountComponent挂载组件|

#82F49C:componentDidMount;

end

@enduml
```

![](/diagrams/react-mounting/react-mounting.svg)

在流程图中，**`:过程;` 代表一个流程节点**

在 `#82F49C:过程;` 前面加颜色可以修改节点的默认背景色

`|泳道A|` 语法可以创建一个泳道，需要注意的是，泳道的切换都需要主动声明

`if` 逻辑判断语义化很好理解，不再赘述

### 思维导图

思维导图虽然我还是更喜欢用 MindNode 来写，但是 PlantUML 生成的 svg 也许更适合放在文章中保存

它的语法也非常简单，我们直接来看个例子：

```diagram
@startmindmap
title JavaScript 结构

* JavaScript
** 运行时
*** 数据结构
**** 类型
***** 7 种语言类型
***** 7 种规范类型
**** 实例（内置对象）
*** 执行过程
**** 事件循环
**** 微任务执行
**** 函数执行
**** 语句执行
** 文法
*** 词法
*** 语法
** 语义

@endmindmap
```

![](/diagrams/fed-mind/javascript.svg)



## 插件支持

PlantUML 的官方列举了很多在使用它的工具和网站，在这里我们以方便使用为目的，向大家介绍一个 VSCode 的 PlantUML 插件。

在 VSCode 市场或者软件内，搜索 [`PlantUML` 插件 / jebbs](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml)，安装后需要在全局或者工作区配置参数：

`.vscode/settings.json`:

```json
{
  "plantuml.diagramsRoot": "diagrams",
  "plantuml.exportOutDir": "hexo/source/diagrams"
}
```

`plantuml.diagramsRoot` 代表放源文件的地方，而 `plantuml.exportOutDir` 则是输出渲染后的图片的地方

`/diagrams` 目录结构:

```console
├── git-flow.wsd
├── react-lifestyle.wsd
├── react-mounting.wsd
├── react-receive-update.wsd
├── react-set-state-pitfall.wsd
├── react-unmounting.wsd
└── tcp.wsd
```

`/hexo/source/diagrams` 目录结构：

```console
├── git-flow
│   └── git-flow.svg
├── react-lifestyle
│   └── react-lifestyle.svg
├── react-mounting
│   └── react-mounting.svg
├── react-receive-update
│   └── react-receive-update.svg
├── react-set-state-pitfall
│   └── react-set-state-pitfall.svg
├── react-unmounting
│   └── react-unmounting.svg
└── tcp
    ├── tcp-connect.svg
    └── tcp-intro.svg
```

按照插件官方的说明，如果需要在本地渲染，还需要指定
配置完还不能马上使用，把 plantuml 代码渲染到图形，你还需要一个渲染器。

有两种办法，一是使用本地渲染，但是需要安装以下两个依赖：

- Java : 运行 PlantUML
- Graphviz : PlnatUML 需要用它来计算位置信息

另一个办法是使用服务器渲染，建议在 VSCode 的**全局配置**中添加如下修改：

```json
{
  "plantuml.server": "http://server_host:port",
  "plantuml.render": "PlantUMLServer"
}
```

`plantuml.server` 这一项替换成你自己的渲染服务器，如果还没有，你可以[搭建一个](#搭建私服)

### 使用

在配置的源文件路径中（也可以在任意位置的文件，但这样不能保存），新建一个 `demo.wsd` 文件，然后就可以编写 PlantUML 代码了

在编辑 PlantUML 代码块的时候，可以打开 `PlantUML: Preview Current Diagram` 窗口，它会根据你的代码**实时渲染**结果，使用私服的话，渲染速度非常的快

### 使用渲染服务的优缺点

优点：
- 15 倍的导出速度和更快的预览速度
- 不需要设置本地变量
- 不需要设置并发量

缺点：
- 不能渲染特别大的图
- 不能渲染有 `!include` 的图
- 只支持 svg, png, txt 格式
- 一些自定义 plantuml 版本的选项将失效


## 搭建私服

不管怎么说，从稳定性和渲染效率上来说，我都倾向于搭建一个自己的渲染服务，而官方也提供了现成的 Docker 镜像可以一键启用（关于 Docker 如何使用不在本文章讨论范围内）

获取镜像：

```shell
docker pull plantuml/plantuml-server:jetty
```

启动服务：

```shell
docker run -d -p 8001:8080 plantuml/plantuml-server:jetty
```

使用 -p 参数将服务默认的 8080 端口转发到对应宿主机的端口上即可

## 参考

本文只是 PlantUML 的简单入门介绍，可以带你快速上手，但对于它的功能的描述也只是冰山一角，更多的 UML 特性和 API 请直接查看[PlantUML 官网文档](http://plantuml.com)