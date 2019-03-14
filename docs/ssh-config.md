---
title: SSH-Key 配置
tags:
  - ssh-key
categories:
  - linux
  - ssh
date: 2016-08-20 10:45:11
---

换台电脑，或者换个服务器，甚至换个手机啥的的，总是要重新配置 SSH 密钥，又老是记不住还要去网上查，所以今天总结一下配置的过程。

#### 环境

- Unix 系统（客户端）
- CentOS （服务器端）

<!-- more -->

## SSH 介绍

Secure Shell（缩写为ssh），由IETF的网络工作小组（Network Working Group）所制定；ssh为一项创建在应用层和传输层基础上的安全协议，为计算机上的Shell（壳层）提供安全的传输和使用环境。

以上是官方解释，用个人的话来说，就是一种通讯协议，通过 ssh 你在和远程服务器通讯时更加的安全，现在不管是一般的 VPS、Github、甚至路由器后台，都是通过 ssh 协议登录的，如果能熟悉的使用它就方便的多了，这次我要说的是通过配置 ssh 密钥，ssh 密钥对可以让你无需输入密码即可登录到 ssh 服务器。

### SSH 密钥

SSH 密钥对总是成双出现的，一把公钥，一把私钥。公钥可以自由的放在您所需要连接的 SSH 服务器上，而私钥必须稳妥的保管好。

所谓"公钥登录"，原理很简单，就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录 shell，不再要求密码。这样子，我们即可保证了整个登录过程的安全，也不会受到中间人攻击。

## 客户端

### SSH-Key 生成

首先打开终端 terminal ，默认路径为当前用户的用户目录下，输入

```sh
ssh-keygen -t rsa -b 4096 -C "youemail@hostserver.com"
```

其中 `youremail@hostserver.com` 是你的邮件地址，这里并不会对它进行验证，如果没有出错，你看到的应该是

```console
Enter file in which to save the key (/Users/<User>/.ssh/id_rsa):
```

>默认情况下，ssh-key 会存放在用户目录下的 `.ssh` 目录下，这是一个隐藏文件夹，你可以输入 `ls -al` 查看它，也可以直接 `cd .ssh` 进入到这个文件夹。同时输入 `mkdir github` 用来存放我们的密钥，当然这里纯属个人的强迫行为，看起来整洁一些～

所以现在屏幕上出现的一段文字就是让你输入密钥的完整路径，其实最后面的 `id_rsa` 是密钥文件名，这个路径可以自己定义，文件名也一样，这里我们就拿 Github 来举例，比如输入

```console
Enter file in which to save the key (/Users/<User>/.ssh/id_rsa):/Users/<User>/.ssh/github/id\_rsa.github
```

这样它就会在 `~/.ssh/github/` 目录下生成名为 `id_rsa.github` 的私钥文件和 `id_rsa.github.pub` 的公钥文件。接下来它会让你输入密码，这里也可以留空，但建议还是加上密码——并且记住，因为这个密码一旦忘了，就找不回来的，只能重新配置密钥了。

同理按照以上方便可以生成多个密钥。

### 编辑 SSH 配置文件

首先确保当前目录在 `~/.ssh` 下，在此目录下新建一个文件名为 `config` ，注意这里没有后缀。使用 `vi config` 可以直接新建并编辑。接下来我们添加配置如下

```sh
Host Github
   User git
   HostName github.com
   IdentityFile ~/.ssh/github/id_rsa.github
   
Host Sugar
   User root
   HostName liangzr.me
   IdentityFile ~/.ssh/vultr/id_rsa.sugar
```

这里分别管理了两个 SSH 密钥，一个是用来登录 Github 的，一个是我的服务器。这里的格式代表：

```console
HOST 			主机名称，相当于配置标识，可以自己随意编辑
User 			登录用户名
HostName		主机域名，这里是顶级域名，比如 `www.github.com` 的域名就是 `github.com` 
IdentityFile	公钥地址，输入对应配置的公钥地址
```

按照如上配置，SSH 就可以识别到以上两个配置了，github 的话，可以验证一下，输入

```sh
ssh git@github.com
```

如果有返回以下返回，就代表设置成功了！当然，现在你不会成功的，因为你还没有配置 Github （服务器）端。

```console
Hi liangzr! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

## 服务器端

服务器端一般指的是 **正经** 的服务器，这里先只拿 CentOS 服务器来举例，github 是非一般情况。

### 配置 authorized_keys

CentOS 也一样，同样是类 Unix 系统，默认路径为 当前用户的用户目录下，并且目录下有个 `.ssh` 文件夹来配置 ssh 服务器，进入到 `.ssh` 文件夹后，可以看到一个叫 `authorized_keys` 的文件，这个文件就是用来存放授权密钥的，也就是你之前在客户端生成的公钥。现在找到你刚刚生成的密钥对中的公钥，比如 `id_rsa.github.pub` 打开并复制其中的内容到 `authorized_keys` 文件中去，编辑完保存即可

### 配置 Github 

Github 不用配置上面的 authorized_keys ，因为是 Github 官方管理的，我们可以在 Web 上添加

### 打开个人设置

在 Github 的个人主页，进入右上角的设置

![](http://7xq464.com1.z0.glb.clouddn.com/github1.jpeg)

进入到 **SSH and GPG keys** ，并且点击 **New SSH key** 

![](http://7xq464.com1.z0.glb.clouddn.com/github2.jpeg)

**Title** 随便填入什么方便你记忆和名字，把上面也提到的公钥（注意是 github 的公钥）内容复制进 **Key** 编辑框内。然后点击 **Add SSH key** 即可。

然后你就可以测试下前面设置的 Github 到底有没有成功了

## 登录服务器

配置好了 SSH-Key ，就可以通过 key 直接登录服务器了，服务器验证过密钥就不会再问你密码了。不过你还是输入这样的指令登录吗？

```sh
ssh root@liangzr.me -p 22
```

还记得我们在编辑配置文件时填写的 ***HOST*** 属性名吗，现在你可以通过 **HOST 名** 直接登录服务器了

```sh
ssh Sugar
```

以上，就配置好了 SSH 的密钥，当然还仅限于一般使用，无论是 `ssh` 指令，还是 `config` 文件都还有很多配置可供挖掘，有兴趣或有需求的可以自行研究一下。

## Reference
1、[NERDERATIBLOG | Simplify Your Life With an SSH Config File](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)
2、[ArchLinux | SSH keys ](https://wiki.archlinux.org/index.php/SSH_keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
3、[Github Help | Generating an SSH key](https://help.github.com/articles/generating-an-ssh-key/)



