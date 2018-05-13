---
layout: post
title: nginx的基本使用
description: nginx的安装及基本使用
categories: docker
tags: docker nginx
---

### docker安装

```sh
docker run -d -p 80:80 -p 443:443 --name nginx \
  -v `pwd`/conf.d:/etc/nginx/conf.d \
  -v `pwd`/logs:/var/log/nginx \
  -v `pwd`/data:/data \
  --restart=always \
  --privileged \
  nginx
```

### 文件服务器配置

```yml
# conf.d/file.local.test.conf
autoindex on;
autoindex_exact_size off;
autoindex_localtime on;

#http
server {
    server_name file.local.test;
    listen *:80;

    root /data/file/;       # 指定文件路径，这里为容器内的/data/目录

    limit_rate_after 1G;    # 超过1G之后限速
    limit_rate 100k;        # 限速为100k
}
```

- autoindex 

    表示是否列出整个目录，默认为false

- autoindex_exact_size 

    默认on，显示文件的确切大小，单位为bytes。
    改为off，显示文件的大致大小，单位为KB,MB,GB。

- autoindex_localtime

    默认off，显示的文件时间为GMT时间。 
    改为on，显示的文件时间为文件的服务器时间

### 服务转发 & 负载均衡

```yml
# conf.d/local.test.conf
# 服务列表
upstream local_test {
    server 172.16.1.4:8080;
    server 172.16.1.5:8080;
}
# http
server {
    server_name local.test;
    listen *:80;

    location / {
        proxy_pass              http://local_test;
    }
}
```

### location的配置

```text
~ 表示执行一个正则匹配，区分大小写
~* 表示执行一个正则匹配，不区分大小写
^~ 表示普通字符匹配。使用前缀匹配。如果匹配成功，则不再匹配其他location。
= 进行普通字符精确匹配。也就是完全匹配。
@ 它定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
```

- location优先级说明

在nginx的location和配置中location的顺序没有太大关系。与location表达式的类型有关。相同类型的表达式，字符串长的会优先匹配。

- 优先级顺序

1. 等号类型（=）的优先级最高。一旦匹配成功，则不再查找其他匹配项。
2. ^~类型表达式。一旦匹配成功，则不再查找其他匹配项。
3. 正则表达式类型（~ ~*）的优先级次之。如果有多个location的正则能匹配的话，则使用正则表达式最长的那个。
4. 常规字符串匹配类型。按前缀匹配。

### 参考文章

- [nginx documentation](http://nginx.org/en/docs/)

- [使用nginx作为文件服务器 - CSDN博客](https://blog.csdn.net/u013410747/article/details/63262561)

- [NGINX location 配置 - zlingh - 博客园](https://www.cnblogs.com/zlingh/p/6288994.html)