---
layout: post
title: Docker部署XX-Net
# description: 
categories: docker
tags: docker
# author: "炮灰哥"
---

[XX-Net](https://github.com/XX-net/XX-Net 'github/XX-Net')，易用的翻墙工具，包含的GAE_proxy和X-Tunnel。

### 目录

- [安装docker](#安装docker)

- [获取镜像](#获取镜像)

- [构建镜像](#构建镜像)

- [运行](#运行)

- [允许远程连接](#允许远程连接)

- [修改配置](#修改配置)

- [创建appid](#创建appid)

- [远程配置appid](#远程配置appid)

- [导出证书](#导出证书)

- [其它](#其它)

### 安装docker

 参考[官方说明](https://docs.docker.com/engine/installation/)

### 获取镜像

关于镜像，可看本人[github](https://github.com/zengchw/xx-net)

```sh
docker pull zengchw/xx-net
```

### 运行

建议映射数据卷，方便修改配置和导出证书，以下为参考代码。

```sh
docker run -d --name xx-net \
    -p 8085-8087:8085-8087 \
    -v `pwd`/data:/data/xx-net/data \
    zengchw/xx-net
```

### 允许远程连接

1. 编辑容器内的/data/xx-net/data/launcher/config.yaml文件

2. 把allow_remote_connect的值改为1

3. 重启容器

### 修改配置

1. 在容器内创建/data/xx-net/data/gae_proxy/manual.ini文件

2. 添加以下内容，允许指定网段访问。

```
[listen]
ip = 0.0.0.0
port = 8087
visible = 1
debuginfo = 0

[pac]
ip = 0.0.0.0
```

### 创建appid

 参考[这里](https://github.com/XX-net/XX-Net/wiki/how-to-create-my-appids)

### [远程配置appid](https://github.com/XX-net/XX-Net/tree/master/code/default/gae_proxy/server '官方教程')

```
docker exec -it ${CONTAINER_NAME} bash
cd /data/xx-net/code/default/gae_proxy/server
python uploader.py "appid1|appid2" -debug
#下面的看控制台输出，用浏览器访问链接
```

### 导出证书

把容器内/data/xx-net/data/gae_proxy/CA.crt文件复制出来，要使用xx-net服务的客户端都安装该证书

证书错误参考[这里](https://github.com/XX-net/XX-Net/wiki/%E8%AF%81%E4%B9%A6%E9%94%99%E8%AF%AF)，把证书都删除，重启服务，重新安装证书即可。


### 其它

参考[wiki](https://github.com/XX-net/XX-Net/wiki)