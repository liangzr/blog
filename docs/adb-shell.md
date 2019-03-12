---
title: 一次折腾刷机的经历
tags:
  - android
date: 2016-02-06 17:48:40
---

这两天在折腾华为荣耀3C，型号是H30-T00/1G RAM，芯片是MediaTek6582，ROOT之后先是刷了个TWRP（Team Win Recovery Project）的recovery，之后又刷过CM12.1、MIUI7等第三方ROM，这些ROM大多基于华为官方的B268/Android4.4修改。
哦这里话说回来，手机是我爸的:P .于是在强权逼迫下，我不得不寻找把它刷回去的办法（非官方包在通话时均没有扬声器输入），官方的卡刷包各种不好找领导又催的紧，惹急了直接找了个据说是官方的线刷包，用MediaTek的Smart Phone Flash Tools直接把整个包刷进去，果不其然出问题了，各种系统刷之后都无法开机，卡在系统启动的动画里。
之后又找了个据说是官方的线刷包刷了进去，直接连recovery都被替换了，官方的各种救砖工具都能成功使用，却没有任何效果，真正的官方升级包无法输入，说是分区有问题。

<!-- more -->

## 前言有点儿长，正篇开始。

首先用SPflash工具把TWRP的Recovery又重新刷了进去，在里面擦除分区时发现，data分区和cust分区（华为自己的分区）都无法挂载，使用Recovery自己的工具把data分区重新格式化，正常挂载了，但是cust还是不行。

查看当前已有的分区

```console
~ # cat /proc/partitions 
major minor  #blocks  name

 179        0    3830784 mmcblk0
 179        1          1 mmcblk0p1
 179        2      10240 mmcblk0p2
 179        3      10240 mmcblk0p3
 179        4       6144 mmcblk0p4
 179        5    1048576 mmcblk0p5
 179        6     196608 mmcblk0p6
 179        7     129024 mmcblk0p7
 179        8    2375168 mmcblk0p8
 179       64       2048 mmcblk0boot1
 179       32       2048 mmcblk0boot0
 179       96    3872768 mmcblk1
 179       97    3870720 mmcblk1p1
```

这里能看到，共有两个disk，分别是mmcblk0和mmcblk1，其中前者被分为10个小分区。后者就是SD card

查看系统挂载的分区

```console
~ # df -am
Filesystem           1M-blocks      Used Available Use% Mounted on
tmpfs                      483         0       483   0% /dev
devpts                       0         0         0   0% /dev/pts
proc                         0         0         0   0% /proc
sysfs                        0         0         0   0% /sys
/dev/block/mmcblk0p8      2287      1157      1130  51% /data
/dev/block/mmcblk0p8      2287      1157      1130  51% /sdcard
/dev/block/mmcblk0p7       124         4       120   3% /cache
/dev/block/mmcblk1p1      3772      2751      1021  73% /external_sd
```

上面能看到已经挂载分区的大小和对应的文件系统分区，但是华为的cust无法挂载，查看/etc/fstab文件系统分区表

```console
/dev/block/mmcblk0p5 /system ext4 rw
/dev/block/mmcblk0p8 /data ext4 rw
/dev/block/mmcblk0p6 /cust ext4 rw
/dev/block/mmcblk0p7 /cache ext4 rw
/dev/block/mmcblk1p1 /external_sd vfat rw
```

手动挂载cust分区

```console
mount /dev/block/mmcblk0p6 /cust
或
mount /cust
```

提示无效参数，怀疑是mmcblk0p6分区损坏，于是把cache卸载，把cust挂载到mmcblk0p7上，果然成功挂载，于是执行

```sh
make_ext4fs /dev/block/mmcblk0p6
```

重新挂载/cust分区，果然成功挂载，至此Recovery已经没有任何报错，重新把/system、/data等等分区全部重新格式化，重新安装lollipop，然而结果没变，还是卡在开机画面。当时和某个QQ群里的“大神”交流了下，“大神”呵呵一笑：不用想了，字库坏了。（一般情况下，字库坏了=主板坏了，不能修）

说到字库这个词我也想呵呵，手机字库什么的就是Flash，叫的这么不近人情，然而我可以肯定的是，我的“字库肯定没坏啊”，因为我刚刚还在对它进行格式化的操作。

于是我用ADB进入手机的shell，用刚刚的格式化操作，对前面几个小分区全部格式化了一遍

```sh
make_ext4fs /dev/block/mmcblk0p2
make_ext4fs /dev/block/mmcblk0p3
make_ext4fs /dev/block/mmcblk0p4
```

重新开机，一切正常～～

### 后记

这次事情是在刷机的过程中，用一些简单的ADB Shell命令解决了手机系统无法开机的一些问题，Android                      Shell还有更多强大的功能还没用过，并且本文只是记载了自己遇到的问题，仅能做为参考，切勿照搬。