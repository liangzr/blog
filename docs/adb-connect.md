---
title: 尝试ADB连接
tags:
  - android
date: 2016-02-06 16:54:04
---

很早就会各种刷机，用各种工具，但始终没正式的接触过ADB（Android Debug Bridge），最近才熟悉了下它。首先是连接方式：USB和Wi-Fi

### USB连接

首先是显示当前连接的设备

```sh
adb devices
```
如果有设备通过USB成功连接，会显示如下：

```sh
List of devices attached
0123456789ABCDEF	device
```

左边的是设备的名字，右边的device代表设备的状态，目前见到过的状态

```sh
device            - 设备通过USB连接成功
offline           - 设备的adb usb服务没有启动
no permissions    - 需要通过root权限重启adb server
unauthorized      - Android4.4之后，需要在设备上对计算机授权才可调试
```

如果出现了上面的no permissions的状态，在teminal进入root用户，或者前面加sudo

<!-- more -->

```sh
adb kill-server
adb devices
```

其中，执行`adb devices`时，adb会首先默认执行`adb start-server`，之后可以`Ctrl+A+D`退出root用户，设备应该会是device状态。（调试OPPO U705T - Android4.1时遇到）

接下来，只要设备是device状态，就可以对它执行各种push、pull、install、shell的命令了。

### Wi-FI连接

USB一直连着，当然可以保证手机电量是满的，但是如果像我这样笔记本拿来拿去，手机一直挂在上面挺不方便的，这时候就可以用到Wi-Fi连接了，在CM12.1上的开发者选项里直接有**网络ADB调试**的选项，在Android4.1上并没有。这时候就需要我们手动改变它的连接方式了。

用USB把手机连接到电脑，前提是保证它是device状态，执行
`adb tcpip 5555`
5555是默认端口，可以自己修改，前提是没被其他的进程监听占用。这个命令，嗯我失败了，最保险的办法是在手机上的本地终端，输入su获取root权限，然后执行

```sh
stop adbd
setprop service.adb.tcp.port 5555
start adbd
```

然后就可以在电脑的teminal上

```sh
adb connect &lt;remotehost&gt;
```

其中`remotehost`是你手机在局域网中的ip地址，这里不需要特别标明端口，我这里是192.168.1.113，结果如下

```sh
liangzr@acer:~$ adb connect 192.168.1.113
connected to 192.168.1.113:5555
liangzr@acer:~$ adb devices
List of devices attached
192.168.1.113:5555	device
0123456789ABCDEF	device
```

这个时候就可以断开USB来调试了，用Wi-Fi调试和用USB一样一样的，只是在下载速度上会有些慢。

### 后记

其实上面的好多操作，都可以直接在手机的本地终端里完成，并且个别问题的原因并不是很清楚，只是在Google上找到了答案，另外，附送一个国外大神的Script切换USB和Wi-Fi方式。

[adbwifi.sh](https://gist.github.com/liangzr/3efa2aa4fec07fe60a83) - 脚本转自[Android通过Wifi来调试你的应用 - stormzhang博客](http://www.stormzhang.com/android/2014/08/27/adb-over-wifi/)

adbwifi.sh内容：

```sh
#!/bin/bash

#Modify this with your IP range
MY_IP_RANGE="192\.168\.1"

#You usually wouldn't have to modify this
PORT_BASE=5555

#List the devices on the screen for your viewing pleasure
adb devices
echo

#Find USB devices only (no emulators, genymotion or connected devices
declare -a deviceArray=(`adb devices -l | grep -v emulator | grep -v vbox | grep -v "${MY_IP_RANGE}" | grep " device " | awk '{print $1}'`)  

echo "found ${#deviceArray[@]} device(s)"
echo

for index in ${!deviceArray[*]}
do
echo "finding IP address for device ${deviceArray[index]}"
IP_ADDRESS=$(adb -s ${deviceArray[index]} shell ifconfig wlan0 | awk '{print $3}')

echo "IP address found : $IP_ADDRESS "

echo "Connecting..."
adb -s ${deviceArray[index]} tcpip $(($PORT_BASE + $index))
adb -s ${deviceArray[index]} connect "$IP_ADDRESS:$(($PORT_BASE + $index))"

echo
echo
done

adb devices -l
#exit</pre>
```