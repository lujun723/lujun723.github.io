---
layout: post
title:  "飞凌i.MX6UL开发板填坑记"
categories: Technique
tags:  Technique
author: Sam
---

* content
{:toc}

## 飞凌i.MX6UL开发板填坑记

### 第一坑 这年头谁还用DB9的串口啊...
飞凌的15年设计的板子，竟然还用DB9串口232，送了一根特别长的双头DB9串口线，说实话，这年头用这个的人越来越少了吧，飞凌技术团队需要更新自我啊！
还好我开发板多，翻腾28335盒子，里面有一个USB转232，而且最近玩Arduino又成功给Macbook Pro装上了CH340的驱动，所以，终于能终端连上开发板的调试串口了，这里需要记录一点，**如何通过Mac终端连接开发板调试串口：**
```
我用的是 Z-TEK ZE551A，在 OS X 里即插即用，连接后在 /dev/ 里就能看到 tty.usbserial-XXXXXXXX，后面的8位字符似乎是随机生成的。
　　接好串口线，在终端里执行代码：
　　screen /dev/tty.usbserial-FTB3L8FS 9600
　　就能连上了，就这么简单，从screen返回终端的快捷键是Ctrl+A,D
　　如果再次连接串口时出错可直接杀掉screen进程：
　　kill -9 `pidof SCREEN`
```


### 第二坑 这年头谁还用WEP方式加密wifi啊...

要用板载wifi，需要：

- 先加载网卡驱动：
```
insmod /lib/modules/3.14.38-6UL_ga\+ge4944a5/kernel/drivers/ne
t/wireless/realtek/rtl8723BU/8723bu.ko
```
- 第一次配置路由器时，需要这一条（会生成一个wpa.conf的文件，里面就是ssid和sn）
```
wpa_passphrase "你路由的名字" >wpa.conf 你的密码
```

- 然后加载配置文件：
```
wpa_supplicant -Dwext -cwpa.conf -iwlan0 &
```
- 最后用dhcp找路由器要ip：
```
udhcpc -iwlan0
```

这样就可以上网了

### 第三坑 没有yum怎么整
当你手里的linux没有apt-get或yum时，你要往根源追踪下，debian系的是dpkg，redhat系的是rpm，我们手里的板子有rpm，那么应该是redhat系的，要装一个yum
然而，需要装yum需要先装python支持库，安装python支持库需要先让wget能get到https链接的文件，我们先重新安装一遍wget，因为默认的wget是简化版
####安装wget
- 去这里找一个喜欢的版本：http://ftp.gnu.org/gnu/wget/
- wget下来，然后
```
tar zxvf wget-1.xx.tar.gz
cd wget-1.xx
./configure
make
make install
```

这个过程会发现时钟还需要设置，可以网络同步，然后又发现没有gcc

问题来了，安装gcc需要下源码，重新编译内核，**卒**