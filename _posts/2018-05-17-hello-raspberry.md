---
layout: post
title: Hello 树莓派
description: 树莓派入门
categories: other
tags: other
---

### 条件

- Raspberry Pi 3b+
- 32G存储卡
- mac
- 网线
- 面包板、led灯、杜邦线、传感器若干

### 系统安装

#### 各种下载

- [官方镜像下载](https://www.raspberrypi.org/downloads/raspbian/)
- [刻录工具 etcher](https://etcher.io/)
- [Microsoft Remote Desktop for Mac2.1.1 from pchome.net](https://download.pchome.net/internet/server/remote/download-67720.html)

#### 刻录镜像

![images](/images/rpi-image.gif)

#### 允许远程ssh连接

在存储卡根目录新建一个空文件命名为ssh即可，无需后缀

```sh
touch /Volumes/boot/ssh
```

#### 未开机前设置wifi

```sh
touch /Volumes/boot/wpa_supplicant.conf
```

内容如下

```text
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="WiFi-Name"
    psk="12345678"
    key_mgmt=WPA-PSK
    priority=1
}
```

#### 远程桌面连接

```sh
sudo apt-get install xrdp
sudo update-rc.d xrdp defaults
```

### 其它

#### docker

```
curl -sSL get.docker.com |sh
```

仅仅是安装，x86、x64的镜像都是不能用的，要使用arm架构的，很少。

#### nodejs

```sh
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
$ sudo apt install nodejs
```

#### nginx

- 安装

```sh
sudo apt-get install nginx
```

- 配置

建议放在/etc/nginx/con.d/下

#### gpio

- 引脚定义

    ![树莓派40Pin引脚对照表](/images/rpi-pins-40-0.png)

- 面包板

    ![面包板的使用](/images/rpi-01.jpg)

- led控制demo

这里使用nodejs控制led

```sh
$ mkdir gpio-demo & cd gpio-demo
$ npm init -y
$ npm install -S rpio
$ node
> let rpio = require('rpio')
> rpio.open(11, rpio.OUTPUT)    //rpio.OUTPUT == 1，填1也行，打开11号针脚作为输出
> rpio.write(11, rpio.HIGH)     //rpio.HIGH == 1, 表示11号针脚输出高电平，打开led灯
> rpio.write(11, rpio.LOW)      //rpio.LOW == 0, 表示11号针脚输出低电平，关闭led灯
```

### 推荐文章

- [官方安装教程](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

- [无屏幕和键盘配置树莓派WiFi和SSH](http://shumeipai.nxez.com/2017/09/13/raspberry-pi-network-configuration-before-boot.html)

- [树莓派新手入门教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2017/06/raspberry-pi-tutorial.html)

- [关于Docker在树莓派上的5件事 - DockOne.io](http://dockone.io/article/1732)

- [Beginner’s Guide to Installing Node.js on a Raspberry Pi](http://thisdavej.com/beginners-guide-to-installing-node-js-on-a-raspberry-pi/#install-node)

- [Linux/Raspbian 每个目录用途说明 \\| 树莓派实验室](http://shumeipai.nxez.com/2018/01/05/directory-introduction-in-raspbian.html#more-3769)

- [面包板的怎么使用_百度经验](https://jingyan.baidu.com/article/851fbc37a8b5053e1f15abb0.html)