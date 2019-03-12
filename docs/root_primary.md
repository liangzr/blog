---
title: Android Root原理学习（初级）
tags:
  - android
date: 2016-03-15 11:05:04
---

偶然在慕课网上看到了有关Root原理的视频，一直也挺感兴趣的，学习了下。
玩儿过Linux的应该都明白Root代表了什么，获取Root权限你就能控制系统的一切，甚至还可以执行 `rm -rf /` ，反正我没试过，不如你试试？

那么一般情况下如何切换到Root用户呢，在大多数的Linux发行版中，在终端输入 `su` 就可以进入Root用户，当然如果Root用户有密码，你必须输入密码才能切换过去。

Android系统本质上还是属于Linux，它有着Linux和内核和文件系统，它同样可以输入su来切换到Root用户，但为了安全起见，Google一开始就规定Android系统只有两个用户能获取Root权限，一个是Root用户本身，另一个是Shell用户。Shell用户是通过ADB（Android Debug Bridge）登录的，但如果你其他的App想获取Root权限，就没办法通过Shell用户（这里倒是没试过在Shell用户里，用am命令启动App是否能获取Root权限，没办法测试，我的设备已经Root了）。

所以如果我们想让我们登录手机的用户启动的App，来获取到Root权限，我们就要修改su（SuperUser）文件。

<!-- more -->

#### Root所需条件

- Android手机（最好是Nexus系列） × 1
- 修改后的su文件 × 1
- 强大的Recovery × 1

#### 提取Root权限步骤

- 刷入一个合适的Recovery
- 修改su命令
- Recovery刷机文件
- 执行su命令提取Root权限
- 让ROM本身拥有Root权限

## 刷入一个强大的Recovery

对于一个没有Root权限的手机，如果想修改/替换手机系统内部文件，有两种办法

- 通过Bootloader模式复制整个文件系统（也就是我们常说的刷机）
- 在Recovery模式下通过Recovery升级包的方式将文件复制到指定的目录中

很明显一般我们理解的获取Root权限 !=  重新刷机，所以我们选择第二种方式，但Android手机默认的Recovery不够强大，我们需要寻找一个好用的Recovery来替换它。

### 下载Recovery

目前比较流行的强大的Recovery有：

- [ClockWorkMod](https://www.clockworkmod.com/rommanager) （CWM）
- [Team Win Recovery Project](https://twrp.me/Devices/) （TWRP）

先去找下有没有自己的设备，如果没有，只能到国内的各大手机论坛，找下自己的手机版块，看有没有民间大神把这些强大的Recovery移植到你的手机上，这里另外提一个，国内低端手机用的比较多的MTK芯片的机子，可以到移动叔叔论坛，移动叔叔自产的Recovery也不错，推荐下国产～

### 刷入Recovery

下载好Recovery之后，我们就可以想办法用它来替换我们手机里的原装Recovery。

#### Option1：通过fastboot命令刷入Recovery

先将手机切换到Bootloader模式，用USB连接手机到电脑，并且确认它已经处于待调试的状态，比如输入 `adb devices` ，显示出你的设备，并且状态是device，输入命令

```sh
adb reboot bootloader
```

等待手机重启到bootloader模式，大概在你准备看的时候，它已经准备好了。

>这里需要提醒的是，bootloader模式下的操作非常危险，bootloader程序是手机在装载系统时运行的程序，同时它也承担着通过软件方式自我更新系统的任务，比较类似我们常见的BIOS，但BIOS好在一般是固件程序。总之，弄坏了bootloader，要么换主板，要么让厂家通过JTAG之类的硬件的方式重新刷入bootloader，简而言之，就是废了。不过也没有那么可怕，只要不执行fastboot命令中有关bootloader的命令，一般也不会有事儿。

用fastboot命令，刷入你已经准备好的recovery

```sh
fastboot flash recovery [你的recovery路径]
```

随后就等待Recovery刷入完成吧！

### Option2：Recovery下通过命令直接刷入（需要Root）

这种方法适合已经Root过，但想更换Recovery的朋友，这里也顺便说下，就一个命令，在 adb shell 下或者手机本地终端su进入Root用户后使用

```sh
dd if=/sdcard/recovery.img of=/dev/recovery
```

下面还是来解释下这个命令吧，dd是Linux自带的一个复制文件的命令，并在复制的同时可以进行指定的转换，`if ` 后跟的是源路径， `of` 后跟的是目标路径，这个命令即是把/sdcard/目录下的recovery.img文件复制到 /dev/recovery .

到这里，相信Recovery这里没啥疑问啦。

## 制作Recovery升级包

有了强大的Recovery之后，我们就需要制作一个Recovery的升级包，来直接替换掉系统自带的su文件，这里Recovery升级包主要是靠一个脚本语言——updater-script，来实现替换系统文件的自动化操作。

### updater-script

updater-script目前的格式是Edify语言，它几乎每一条语句都是一个函数，我们主要看看Edify语言的语法格式。

#### Edify语法

我们主要看看这次制作升级包所用到的函数，更详细语法有兴趣的可以查看[tody_guo的专栏 - Android updater-scripts(Edify Script)各函数详细说明](http://blog.csdn.net/tody_guo/article/details/7948083)

***1. ui_print***  

- 原型：uiprint(msg1, ..., msgN);
- 功能：该函数用于在Recovery界面输出字符串， 其中msg1 - msgN表示N个字符串参数，它至少要指定一个参数，如果指定多个，会将这些参数值连接起来输出。
- 用法：ui_print(" hello world ");

***2. run_program*** 

- 原型：run_program(prog, arg1, ..., argN);
- 功能：该函数用于执行程序，其中prog参数表示要执行的程序文件（完整路径）， arg1 - argN 表示要执行程序的参数发。prog参数是必须的，其他参数可选
- 用法：run_program("/sbin/busybox", "mount", "/system");

***3. delete***

- 原型：delete(file1, file2, ..., fileN);
- 功能：该函数用于删除一个或多个文件。其中file、file2、...、fileN表示要删除文件的路径，至少需要指定一个文件。
- 用法：delete("/system/xbin/su");

***4. package_extract_dir***

- 原型：package_extract_dir(package_path, destination_path);
- 功能：用于提取刷机包中package_path指定目录的所有文件到destination_path指定的目录。其中package_path参数表示刷机包中的目录，destination_path参数表示目标目录。
- 用法：package_extract_dir("system", "/system");

***5. set_perm***

- 原型：set_perm(uid, gid, mode, file1, file2, ..., fileN);
- 功能：用于设置一个或多个文件的权限。其中uid参数表示用户ID，gid参数表示用户组ID。如果想让文件的用户和用户组都是Root，uid和gid需要为0。mode参数表示设置的权限。与chmod命令相似。
- 用法：set_perm(0, 0, 0777, "/system/xbin/su");

***6. mount***

- 原型：mount(fs_type, partition_type, location, mount_point);
- 功能：挂载分区
- 用法：mount("ext4", "EMMC", "/dev/block/platform/s3c-sdhci.0/by-name/system", "/system");

***7. unmount***

- 原型：unmount(mount_point);
- 功能：用于解除文件系统的挂载。其中mount_point参数表示文件系统。
- 用法：unmount("/system");

#### 编写脚本文件

了解了基本的语法后，就可以来编写个简单的替换系统文件的脚本了，我们主要进行的操作如下

- 以读写模式挂载/system
- 删除旧的su文件
- 复制新的su文件
- 修改su文件的权限
- 卸载/system

根据以上步骤，这里直接给上脚本

```sh
ui_print("----------------------");
ui_print("Recovery Upgrade Package");
ui_print("----------------------");

ui_print("--- Mounting /system ---");
#以读写模式挂载/system
run_program("/sbin/busybox", "mount", "-o", "rw", "/system");

ui_print(--- Delete /system/xbin/su ---);
#删除旧的su文件
delete("/system/xbin/su");

ui_pirnt("--- Extracting system to /system ---");
#将刷机包中的system目录的所有文件复制到/system目录中的相应位置
package_extract_dir("system", "/system");

#给su命令添加可执行权限
set_perm(0, 0, 0777, "/system/xbin/su");

#卸载/system
unmount(/system);

ui_print("--- finished ---");
```

### 升级包制作

制作Recovery升级包需要两个目录

- META-INF/com/google/android
- system/xbin

前者用来放我们制作的updater-script文件，在 `META-INF/com/google/android` 目录下还有一个 `update-binary` 的文件，它是用来解析我们制作的updater-script文件，把su放在 `system/xbin` 目录下。

>目录中的其他东西，可以找一个现成任意的Recovery升级包，把内容复制过来就行了

最后，把这两个目录压缩成 `.zip` 文件， 升级包就制作完成了。

# 替换su文件

有了升级包之后，我们要做的就是在Recovery里面安装这个升级包，通过adb shell进入recovery模式

```sh
adb reboot recovery
```

在Recovery模式下，我们可以直接用 adb 操作这个升级包，比如先push到sdcard里面，然后在Recovery里面手动的选择在sdcard里面找到并安装这个升级包，另外我们也可以通过一个adb命令直接push并安装这个升级包

```sh
adb sideload update.zip
```

它会首先将updata.zip下载到手机中，并且执行安装，这里的updata.zip是升级包的路径，而非单单名字。


## 使用su命令提取Root权限

su文件替换完之后，我们就可以通过各种方法获取到Root权限

### 在终端中执行su命令提取Root权限

无论在pc上的adb shell命令还是手机的本地终端，正如我们开篇所说的，直接输入su，即可获取Root权限。

### 在App中使用Root权限

```sh
Runtime.getRuntime().exec("su");
OutputStream os = process.getOutputStream();
os.write("ls /system/app".getBytes());
os.flush();
os.close();
```

这段代码还没有进行尝试，之后会写个删除系统自带软件的程序来验证下。

## su命令源代码解析

Android源代码上su.c文件

```c
/* 
** 
** Copyright 2008, The Android Open Source Project 
** 
** Licensed under the Apache License, Version 2.0 (the "License");  
** you may not use this file except in compliance with the License.  
** You may obtain a copy of the License at  
** 
**     http://www.apache.org/licenses/LICENSE-2.0  
** 
** Unless required by applicable law or agreed to in writing, software  
** distributed under the License is distributed on an "AS IS" BASIS,  
** WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
** See the License for the specific language governing permissions and  
** limitations under the License. 
*/  
  
#define LOG_TAG "su"  
  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <sys/types.h>  
#include <dirent.h>  
#include <errno.h>  
  
#include <unistd.h>  
#include <time.h>  
  
#include <pwd.h>  
  
#include <private/android_filesystem_config.h>  
  
/* 
 * SU can be given a specific command to exec. UID _must_ be 
 * specified for this (ie argc => 3). 
 * 
 * Usage: 
 * su 1000 
 * su 1000 ls -l 
 */  
int main(int argc, char **argv)  
{  
    struct passwd *pw;  
    int uid, gid, myuid;  
  
    if(argc < 2) {  
        uid = gid = 0;  
    } else {  
        pw = getpwnam(argv[1]);  
  
        if(pw == 0) {  
            uid = gid = atoi(argv[1]);  
        } else {  
            uid = pw->pw_uid;  
            gid = pw->pw_gid;  
        }  
    }  
  
    /* Until we have something better, only root and the shell can use su. */  
    myuid = getuid();  
    if (myuid != AID_ROOT && myuid != AID_SHELL) {  
        fprintf(stderr,"su: uid %d not allowed to su\n", myuid);  
        return 1;  
    }  
      
    if(setgid(gid) || setuid(uid)) {  
        fprintf(stderr,"su: permission denied\n");  
        return 1;  
    }  
  
    /* User specified command for exec. */  
    if (argc == 3 ) {  
        if (execlp(argv[2], argv[2], NULL) < 0) {  
            fprintf(stderr, "su: exec failed for %s Error:%s\n", argv[2],  
                    strerror(errno));  
            return -errno;  
        }  
    } else if (argc > 3) {  
        /* Copy the rest of the args from main. */  
        char *exec_args[argc - 1];  
        memset(exec_args, 0, sizeof(exec_args));  
        memcpy(exec_args, &argv[2], sizeof(exec_args));  
        if (execvp(argv[2], exec_args) < 0) {  
            fprintf(stderr, "su: exec failed for %s Error:%s\n", argv[2],  
                    strerror(errno));  
            return -errno;  
        }  
    }  
  
    /* Default exec shell. */  
    execlp("/system/bin/sh", "sh", NULL);  
  
    fprintf(stderr, "su: exec failed\n");  
    return 1;  
}  
```

从main函数的源代码中可以看到

```c
/* Until we have something better, only root and the shell can use su. */  
myuid = getuid();  
if (myuid != AID_ROOT && myuid != AID_SHELL) {  
    fprintf(stderr,"su: uid %d not allowed to su\n", myuid);  
    return 1;  
}  
```

这个函数可以看出，su文件检测当前用户如果不是Root用户或者Shell用户，将会直接退出su命令。

```c
if(argc < 2) {  
    uid = gid = 0;  
} else {  
    pw = getpwnam(argv[1]);  

    if(pw == 0) {  
        uid = gid = atoi(argv[1]);  
    } else {  
        uid = pw->pw_uid;  
        gid = pw->pw_gid;  
    }  
}  
```

如果参数小于2，uid（将要切换的用户id）和gid（将要切换的用户组id）都会切换到Root用户和Root用户组，C语言中如果一个命令不加任何参数，argc值就等于1，也就是说，如果你只输入了su命令，将自动切换到Root用户和Root用户组。

```c
if(setgid(gid) || setuid(uid)) {  
    fprintf(stderr,"su: permission denied\n");  
    return 1;  
}
```

setgid和setuid函数是最关键的获取Root权限的函数，如果设置成功则返回0，所以只有当两个函数返回都为0的时候，才算成功获取Root权限。

```c
/* Default exec shell. */  
execlp("/system/bin/sh", "sh", NULL);  
```

这段代码将当前进程替换成一个新进程，也就是以Root用户登录的一个新的Shell。

## 后记

有关Android设备提取Root权限最基本的原理就是这些了，但一般情况下往往没这么简单，我们也知道不同的机型Root的方式也不一样，因为厂商对待Root设备的态度不一样，有的坚决反对，有的不鼓励不支持，有的想方设法阻止你Root，或者只有Nexus系列的原生系统才会如此的简单吧，但万变不离其宗，原理上不会差太多，对以后的学习也是一个基础。




