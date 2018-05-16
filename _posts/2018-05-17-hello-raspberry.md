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

### 系统安装

1. 各种下载

    - [官方镜像下载](https://www.raspberrypi.org/downloads/raspbian/)
    - [刻录工具 etcher](https://etcher.io/)
    - [Microsoft Remote Desktop for Mac2.1.1 from pchome.net](https://download.pchome.net/internet/server/remote/download-67720.html)

1. 刻录镜像

    略

1. 允许远程ssh连接

    在存储卡根目录新建一个空文件命名为ssh即可，无需后缀

    ```sh
    touch /Volumes/boot/ssh
    ```

1. 远程桌面连接

    todo

### 参考教程

[官方安装教程](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)