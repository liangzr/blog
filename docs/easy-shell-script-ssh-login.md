---
title: 简易shell script实现ssh登陆
tags:
  - ssh
categories:
  - linux
  - shell
date: 2016-01-07 11:17:03
---

因为想每次管理服务器都要手动输入`ssh user@remotehost`然后输入密码才能进去，并且经常隔2-3分钟不去管它，控制台就无响应了，所以我就想写个脚本实现自动登陆和无响应重连。于是

- 想写个脚本去
- 找到篇Advanced Bash-scripting Guide, 想创建一个Github repository来整理
- 在2048KB创建个page来收录Github repository, 嫌网站字体太难看要更改字体
- 在`.css`文件里更改font-family由于文件太多又去学了正则表达式
- 修改完后发现没效果, 更改`.css`的font-face实现修改
好，还是说正事儿。

<!-- more -->

## Option 1: 仅实现登陆

* * *

最简单的实现手动的登陆，可以用expect工具来实现，在Ubuntu下需要下载：
`sudo apt-get install expect`
Expect中最关键的四个命令是：

- send：用于向进程发送字符串
- xpect：从进程接收字符串
- spawn：启动新的进程
- interact：允许用户交互

实现如下：

```sh
#!/usr/bin/expect
spawn ssh user@remotehost
expect "*password:"
send "userpwd\r"
expect "*#"
interact
```

添加执行权限

```sh
sudo chmod +x ssh-script.sh
```

运行以上ssh-script.sh就可以登陆了，也可以把用户命令放入`～/bin`，在Linux的很多发行版本中默认PATH都加入了这个目录，在Ubuntu上只要在用户根目录下建立了bin文件夹，下次登陆时会自动添加进PATH，如果要手动修改则打开.bashrc文件

```sh
sudo vi ~/.bashrc
```

文件末添加

```sh
export PATH = ~/bin:$PATH
```
生效改动

```sh
source ~/.bashrc
```

这样在teminal里输入ssh-script.sh就能一键登陆VPS了，如果嫌后缀麻烦，可以……删掉好了：P

未完