---
title: "树莓派4折腾记"
author: "iPattern"
date: 2021-02-16T04:30:00+08:00
description: "This is meta description"
categories: ["编程 Programming"]
tags: ["Raspberry Pi","Linux","OpenWrt"]
---

去年年中蒙朋友信赖，让俺做个展览项目，涉及到投影仪轮播设备时，想起来树莓派可以刷轮播固件，实现没有黑屏，没有多余播放文字信息的无限轮播。
而且更换视频内容只需拔下U盘，替换视频内容再插上即可。可以说是最方便的傻瓜轮播设备了。
再通过投影仪的USB A口来给树莓派供电，开机即播，关机即停，完美！
由于担心投影仪的USB供电不足，使用了树莓派3B，基本OK。

展览过后，俺对树莓派的热情又燃起来了。说起来，从树莓派2就开始折腾了，当时只是做XBMC播放器，时过境迁，如今XBMC也变成Kodi了。
其实树莓派4发布后没多久，俺就被种了草，不忍再剁手，倒是安利小伙伴买了一个，大内存4G甚至8G，高速USB3，外加双4K输出真是巨大的进步。
热情来了就不再忍了，先后入手了2G版，4G版，官方降价后很香了。

2020年是俺沉迷开源软件和Linux的一年，一系列命令行工具vim，ranger，youtube-dl，aria2让俺欣喜不已。
i3，sway等Tiling widows manager也是相见恨晚。
以树莓派上的Manjaro Sway为代表，pacman + AUR通吃各种软件，清爽，静音，几w的低功耗，就可以敲代码，跑跑QGIS，Gimp，Inkscape...
近期的新科技也就M1的MacBook Air能这么清爽了吧。

借新建博客的契机，就把这一年对树莓派4的折腾稍作记录吧。

## Desktop篇
树莓派4刚出来的时候，官方宣称它是一个功能完备的Desktop PC。

所言非虚。

### Raspberry Pi OS
似乎在树莓派4这一代，官方系统Rasbian也改名Raspberry Pi OS了，甚至还有了x86版，当然因为授权问题，阉割了Mathematica。
不太理解他们推出x86版本的意义，话说回来ARM的64位正式版支持迟迟未推出反倒令人着急。
好在USB启动支持做得更方便了，用Raspberry Pi Imager刷入bootloader EEPROM到microSD卡，插卡通电，
等树莓派指示灯绿灯闪烁，或外接显示器全绿色，即可完成升级。

官方系统，依然是最稳的树莓派系统，便利的VNC，豪奢的免费Mathemetica，没有Vulkan驱动搞不定2.8前依然堪用的Blender2.79...
2.8G的`Raspberry Pi OS with desktop and recommended software`用torrent拖下来即可开刷。

### VNC 配置
虽然ssh管理树莓派轻量便捷，但还是免不了个别情形下需要使用树莓派的桌面系统，或是使用一些窗口软件。
在不连接显示器的情况下，VNC viewer连接树莓派ip：`10.0.0.5`，默认用户名：`pi`，
会提示错误: **cannot currently show the desktop**
可先用ssh链接`ssh pi@10.0.0.5`，输入密码，然后运行`raspi-config`，在出现的配置选项找到`Display Option`然后`Resolution`，指定一个分辨率即可。（如1920*1080 60Hz）
完成后按提示重启，即可生效。

Ssh到这个ip如果之前有记录，会报错。
```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
6e:45:f9:a8:af:38:3d:a1:a5:c7:76:1d:02:f8:77:00.
Please contact your system administrator.
Add correct host key in /home/hostname /.ssh/known_hosts to get rid of this message.
Offending RSA key in /var/lib/sss/pubconf/known_hosts:4
RSA host key for pong has changed and you have requested strict checking.
Host key verification failed.
```
简单的解决办法是
```bash
ssh-keygen -R <host>
```
例如：
```bash
ssh-keygen -R 10.0.0.12
```

Removes all keys belonging to hostname from a known_hosts file.
这样会移除之前此ip的ssh密钥。

### exFAT硬盘格式支持
估计是由于Linux内核版本太低的缘故，树莓派的Raspberry Pi OS不能直接加载exFAT格式的外置硬盘，我当前使用的内核版本为`Kernel: 5.4.83-v7l+`。
按照检索到的方法安装下面两个包即可。
```bash
sudo apt-get install exfat-fuse
sudo apt-get install exfat-utils
```
### smb共享配置
说来惭愧，之前在树莓派的OpenWRT上一直没搞定smb共享文件夹，网上搜到的配置比较混乱，好些似乎因为版本过时不起作用。
好在这次找到了一个非常简单的配置方法，也不需要给树莓派静态ip，共享目录为挂载移动存储的目录`/home/pi/shared`。
安装以下两个包
```bash
sudo apt-get install samba samba-common-bin
```
然后sudo编辑smb配置文件
```bash
sudo nano /etc/samba/smb.conf
```
末尾加上
```conf
[share]
    path = /home/pi/shared
    available = yes
    valid users = pi
    read only = no
    browsable = yes
    public = yes
    writable = yes
```
测试参数是否正确
```bash
testparm
```

设置`pi`账户密码
```bash
sudo smbpasswd -a pi
```
重启 Samba server:
```bash
sudo service smbd restart
```

默认smb服务名称`RASPBERRYPI`，可在Mac，PC，Linux的文件浏览器的网络内发现。

### Manjaro Sway
除官方系统外，俺用的最久最爽的一个。
### Manjaro KDE
界面繁复，pacman和AUR清爽。
### Ubuntu Desktop 20.10
界面华丽但对树莓派4显得沉重了些，卡顿明显。
## Server篇
### Ubuntu Server 20.04
### OpenWrt
树莓派4用了半年的OpenWRT系统，源自GitHub上的`OpenWrt-Rpi`项目的`openwrt-bcm27xx-bcm2711-rpi-4-ext4-sysupgrade.img.gz`固件。
各类插件非常齐全，半年使用下来也非常稳定，似乎未出现过故障，体验堪比前年使用的3215u的x86软路由了。
只可惜树莓派只有一个网口，做主路由多有不便，只好拿来做旁路由。另外用USB3拖个移动硬盘也稍微凌乱了些。

#### 旁路由设置
刷完系统后，注意先不要对树莓派连接网线到主路由，不然会自动分配ip，不方便去初始的192.168.1.1去配置OpenWrt。
通电后，稍等片刻即可去电脑连接名为`OpenWrt`的Wi-Fi，在`192.168.1.1`进行设置，默认用户名密码分别为`root`和`password`。
在 网络 - 接口 LAN 进行以下设置：
- 静态协议
- ipv4地址： 10.0.0.233 （因为我的主路由是10.0.0.1，所以设置同一ip段内1-255之间的ip均可）
- ipv4网关：10.0.0.1
- ipv4广播：10.0.0.255
- 使用自定义的DNS服务器：10.0.0.1
完成后保存应用，重启，连接网线到主路由即可。
至于树莓派自带的无线网络，信号很弱，有些鸡肋，直接禁用，或者修改密码，留作管理使用。
#### SMB共享
网络存储 - 网络共享 - 编辑模版
注释掉这句，估计是因为OpenWrt只有一个root用户的缘故，必须启用。
`#invalid users = root`
其实在管理界面的 系统 - TTYD 终端 这里有个交互的终端可用，不需要ssh连接，浏览器内直接使用即可。
像是nano vim都可以拿来直接编辑配置文件，毕竟OpenWrt也是Linux，可以用命令行愉快地玩耍。
**共享目录**的设置：
"软件" 应该是“名称”才对
目录填写USB硬盘的挂载位置即可，如`/mnt/sda1`,
[*] 可浏览
[*] 允许匿名用户
创建权限掩码 777
目录权限掩码 777
#### Aria2 下载设置
网络存储 - Aria2 配置 - 文件和位置
默认下载目录： `/mnt/sda1/aria2`，主要在这里注意如果挂载的硬盘里没有`aria2`这个文件夹，需要去新建一个，或者修改到硬盘根目录`/mnt/sda1`亦可。
另外我的RPC端口6800不起作用，访问10.0.0.233:6800失败，而http://10.0.0.233/ariang没问题，暂时不明缘由。
## 游戏篇
### RetroPie

- Wolfanoz_Nostalgia_64GB_RPi4_Official
- [256gb]-Epic.Noir.RPi4.4.6.8-RickDangerous

## 翘首以盼的系统

Open Media Vault，可惜检索到需要先刷Raspberry Pi OS Lite，然后通过git安装OMV5测试版。
大量文件git + 测试版，这两项足以劝退了。。。

然后是令人心动的ESXi，这也是俺当初在3215u上使用的系统，体验优秀，稳如磐石。有消息它已经放出树莓派4的测试版固件了，可惜还要再等。


###### 参考链接：

- [OpenWrt-Rpi](https://github.com/SuLingGG/OpenWrt-Rpi)
- [Raspberry Pi 4 VNC Remote Connection Desktop Error - Raspberry Pi Stack Exchange](https://raspberrypi.stackexchange.com/questions/101796/raspberry-pi-4-vnc-remote-connection-desktop-error)

- [Build a Samba file server](https://magpi.raspberrypi.org/articles/raspberry-pi-samba-file-server)

- [Build a Raspberry Pi 4 NAS with Open Media Vault 5](https://linuxhint.com/raspberry_pi_open_media_vault/)