---
title: Android Studio 安装 ( Linux )
tags:
  - android
date: 2016-03-25 15:25:22
---


##### 本文撰写时的系统环境：  

- 系统：Ubuntu 15.10
- JDK版本：1.8.0_65（64位）
- Android Studio 版本：2.0
- 开发机：Nexus 6 - 6.0.1

## JDK（Java Development Kit）安装

### JDK or JRE ？

首先要说清一点，JDK 是 Java 工具包，包含了很多编译所要用到的命令工具和运行环境（JRE - Java Runtime Environment）。所以 JDK 是包含了 JRE 的，但如果一个电脑只需要运行已经编译好的 Java 软件程序，只需要在这个电脑上安装 JRE 即可。

### 下载与安装

- [Oracle JDK](http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html##javasejdk)
- [OpenJDK](http://openjdk.java.net/install/)

Oracle JDK 是拥有 Java 版权的官方 JDK , 而 OpenJDK 是原 Sun 公司发布的开源版本，均按引导安装即可。

<!-- more -->

### 添加环境变量

JDK安装成功后，如果不在环境变量中声明 Java 工具包的地址，仍无法从命令行直接调用 Java 命令，如果是 OpenJDK 可跳过这一步。

#### 1, 更改当前用户环境变量

首先确保当前用户非 Root 用户，因为我们一般在调用它的时候是以非 Root 用户，如果需要对 Root 用户也添加 JDK 的支持，切换到 Root 用户重复步骤即可。

```
cd ~ && vi .bashrc
```

在用户配置文件中的最后面添加 Java 的配置，按键盘 `i` 切换到输入模式。

```
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_65
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

复制上面的代码并 `CTRL + SHIFT + V` 粘贴在 `.bashrc` 文件的末尾，`ESC` 退出编辑模式并输入 `:wq` 保存文件。

#### 2, 生效配置

只是更改了用户配置文件，系统并不知道它的变化，重启或者手动更新以让系统识别到。

```
source ~/.bashrc
```

#### 3, 检查是否添加成功

打开 Teminal 输入 `java -version` 如果显示如下，则表示环境变量配置成功，否则重新试一下或者 Google it！。

```
java version "1.8.0_65"
Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)
```

## Android Studio 安装

Android Studio 是编写 Android 应用的最佳 IDE (Integrated Drive Electronics - 集成开发环境)，和臃肿的 Eclipse 甚至简陋的文本编辑器说再见吧！

### 下载

- [Google 官方下载](http://developer.android.com/intl/zh-cn/sdk/index.html)
- [AndroidDevTools](http://www.androiddevtools.cn/)

前者就不用介绍了，因为一些你懂的原因，我们访问不到，推荐大家到第二个地址下载 Android Studio，除了 SDK 这个站点还有许多你想都想不到的工具，简直无所不有。

### 安装

下载下来后，把压缩包解压到用户目录你放软件的地方，打开 Teminal，切换到 Android Studio 目录下的 `bin` 文件夹，给 `studio.sh` 添加可执行权限，并且执行它。

```
sudo chmod +x studio.sh
./studio.sh
```

第一次执行这个文件，脚本会自动检测你是否安装过 Android Studio ，如果没有就会进行初始化安装。

在安装过程中，可能需要你设置代理，或者提示你无法下载 SDK platform tool 的报错，这还是因为在国内 `dl.google.com` 被墙的原因，最简单直接的办法是用论坛里的 “[黑科技](http://www.studyjamscn.com/thread-172-1-1.html)”，这里不再重复。

### 其他问题

#### 没有桌面图标
在 Android Studio 的欢迎界面，最下面的配置，可以生成图标，选择为所有用户生成

![](http://7xq464.com1.z0.glb.clouddn.com/icon.png)

生成后直接按 `Win` 键然后在 Commend 搜索里面多半是搜不到的……这是 Ubuntu 的 Bug，但是图标确实已经创建了，你可以在 `usr/share/applications/`下找到 Android Studio 的图标，然后拖到任务栏固定就好。

#### 找不到 tool.jar ?

以 JDK 的安装目录下，找到 tool.jar 直接复制到 Android Studio 根目录下的 `lib` 文件夹里就行了。

#### 后续追加……

以上是我在配置 Android Studio 中遇到的问题，具体问题还会有个例，遇到问题多去 Google 一下，StackOverflow 上有你（假设新手）遇到的 95% 的问题的答案。

## 使用前的配置: SDK Manager

在开始尝试第一个 HelloWorld 前，还有一些遗留问题要解决，那就是 SDK 和 AVD。首先打开SDK Manager

![](http://7xq464.com1.z0.glb.clouddn.com/sdk_manager.png)

没错就是第三个小人儿，然后点击下面的 Launch Standalone SDK Manager，打开后可以看到

![](http://7xq464.com1.z0.glb.clouddn.com/sdk_manager_2.png)

在这里可以选择下载需要的 SDK 文件。

### SDK Upgrade

在国内我们无法直接连接到 Google 来更新我们的 SDK 库，但有很多镜像站可供我们选择，依然是刚刚下载 Android Studio 的 **[AndroidDevTools](http://www.androiddevtools.cn/)** 网站，首页就向我们介绍了如何使用国内的镜像站，我们可以从中选择一个使用。

![](http://7xq464.com1.z0.glb.clouddn.com/sdk_proxy.png)

注意！注意！注意！在这里记得选中 `Force https://... sources to be fetched using htpp://`

### AVD 建立

Linux 下创建 AVD 还是比较简单的，不用多考虑驱动等各种问题，只要你把 SDK 该下载的都下载了。新版的 Android Sutdio 的创建 AVD 的引导越来越智能化，然而我在用 Android Studio 创建时选择 Nexus 之类的模板会缺少必要的参数，卡在开机界面，最好选择比如 5.1 寸这样的配置。

![](http://7xq464.com1.z0.glb.clouddn.com/avd.png)

不过话说回来，强烈建议大家使用真机调试，即使快如 Genymotion 依然不如一个 500 块的真机来的流畅。如果有朋友是因为真机调试需要总带着 USB 线，大可不必担心，因为 ADB 是提供了通过 Wi-Fi 连接手机的，有意向的同学可以看看这篇 [Android通过Wifi来调试你的应用](http://stormzhang.com/android/2014/08/27/adb-over-wifi/)。

## 开始第一个应用：HappyBirthday

### 新建项目引导

在欢迎界面或者 Android Studio 主界面，都能很容易找到 **New Project** ，点击新建

![](http://7xq464.com1.z0.glb.clouddn.com/new_project.png)

填上项目的名称，公司域名如果没有自己的就写 `android.example.com` ，然后下一步

#### 选择合适的 API Level 

![](http://7xq464.com1.z0.glb.clouddn.com/api_select.png)

![](http://7xq464.com1.z0.glb.clouddn.com/api_select_2.png)

关于 **API Level** 的选择，目前推荐用 Android 4.1 也就是 API 15 ，Android Studio 为我们提供了目前激活设备的系统分布，[友盟](http://www.umindex.com/)有更详细的介绍。

#### 选择默认 Activity 模板

![](http://7xq464.com1.z0.glb.clouddn.com/activity_select.png)

以前的 Android Studio 有两个默认 Activity 分别叫 BlankActivity 和 EmptyActivity ，傻傻分不清。机智的小伙伴应该已经发现了，在这里 BlankActivity 已经变成了 BasicActivity，对于新手来说，我们还是选择 EmptyActivity 就好。

### Android Studio 界面

![](http://7xq464.com1.z0.glb.clouddn.com/as.png)

到此我们的 Linux - Android Studio 教（bi）程（ji）就基本完成了，其他部分和 Windows 大同小异。

## 后记

本来觉得安装 Android Studio 的教程到处都是，没啥写的，但这次参加 Google Jams 的活动，刚好有这个任务，顺便就写的认真点儿，也算总结下吧。
  
