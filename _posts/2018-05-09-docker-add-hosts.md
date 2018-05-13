---
layout: post
title: docker容器添加自定义hosts
description: Docker容器修改hosts文件重启不变
categories: docker
tags: docker
---

### 方案一

启动时增加hosts，参考自[docker docs](https://docs.docker.com/edge/engine/reference/commandline/run/#description)

```sh
docker run -d --name test1 \
    --add-host test1.a:1.2.3.4 \
    local/test
```

### 方案二

docker-compose.yml文件指定，参考自[stackoverflow](https://stackoverflow.com/questions/29076194/using-add-host-or-extra-hosts-with-docker-compose?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

```yml
test2:
  build: local/test
  extra_hosts:
    test1.a: 1.2.3.4
    test1.b: 4.3.2.1
```

### 方案三

构建镜像时增加，参考自[docker docs](https://docs.docker.com/edge/engine/reference/commandline/build/#add-entries-to-container-hosts-file---add-host)，这个本人测试失败，不可用。

```sh
docker build \
    --add-host test.abc:1.2.3.4 \
    -t local/test \
    .
```

### 错误示例一

Dockerfile修改hosts文件，类似如下操作

```yml
RUN echo '1.2.3.4   test.a' >> /etc/hosts
```

### 错误示例二

容器启动后修改/etc/hosts，仅本次启动有效，重启就还原