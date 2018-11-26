---
layout: post
title: Linux根目录爆满解决
description: /dev/mapper/centos-root 100%问题。
categories: linux
tags: 
---

这是很久之前遇到的问题了，参考的博客加了书签，最近闲下来，转成自己的。

先简述一下之前的情况，本地的一台测试服务器，一直只使用root用户，但安装系统时默认只给root用户分配了50G空间，使用一段时间后，docker启动容器，提示内存不足。

### 开始

1. 首先查看磁盘情况（这里已经扩到550g了）

    ```sh
    [root@test10 ~]# df -h
    Filesystem               Size  Used Avail Use% Mounted on
    /dev/mapper/centos-root  550G   32G  519G   6% /
    devtmpfs                 3.8G     0  3.8G   0% /dev
    tmpfs                    3.8G     0  3.8G   0% /dev/shm
    tmpfs                    3.8G  9.6M  3.8G   1% /run
    tmpfs                    3.8G     0  3.8G   0% /sys/fs/cgroup
    /dev/sda2                494M  188M  306M  39% /boot
    /dev/sda1                200M  9.8M  191M   5% /boot/efi
    /dev/mapper/centos-home  373G   33M  373G   1% /home
    overlay                  550G   32G  519G   6% /var/lib/docker/overlay2/a4213478d93f7dabc8d33adc9eadaf411c21c09cf225a91634ad5e5550134193/merged
    overlay                  550G   32G  519G   6% /var/lib/docker/overlay2/279d397a841cf2645862f73a21f0454cd77e372656c489b4d49c39d17d9f57c3/merged
    overlay                  550G   32G  519G   6% /var/lib/docker/overlay2/56945364e5b279ceb2f7961dedb2058f276f15a9f46aa636fd499bbe000e44de/merged
    shm                       64M     0   64M   0% /var/lib/docker/containers/ed11862c6b38097e64167dd11be6d0a99cd4604b235f16de784df88f00a221c2/mounts/shm
    shm                       64M     0   64M   0% /var/lib/docker/containers/cb82ec88b6e594494f808e989b12f4040aa9044b626f29d0f84fd3c2e7d96dbe/mounts/shm
    overlay                  550G   32G  519G   6% /var/lib/docker/overlay2/0db04f3c52d48bd07532ce452e23ef09d6d2d54654bba505f29ea3c039c68e1f/merged
    shm                       64M     0   64M   0% /var/lib/docker/containers/af0f2f29662b1cd495993e6693b19297e5cc4f014549855bc21f696f6d0a3b03/mounts/shm
    tmpfs                    770M     0  770M   0% /run/user/0
    overlay                  550G   32G  519G   6% 
    ```

1. 备份home分区文件

    ```sh
    tar cvf /tmp/home.tar /home
    ```

1. 卸载__/home__，无法卸载则停止相关进程

    ```sh
    fuser -km /home/  # 这里我没有用到，我的home目录为空，有需要使用
    umount /home
    ```

1. 删除/home所在的卷

    ```sh
    lvremove /dev/mapper/centos-home
    ```

1. 扩展/root所在卷，扩展文件系统(这里再增加50g，酌情修改)

    ```sh
    lvextend -L +50G /dev/mapper/centos-root
    xfs_growfs/dev/mapper/centos-root
    ```

1. 重建/home，创建文件系统，并挂载

    ```sh
    lvcreate -L 323G -n /dev/mapper/centos-home
    mkfs.xfs  /dev/mapper/centos-home
    mount /dev/mapper/centos-home
    ```

1. 文件恢复

    ```sh
    tar -xvf /tmp/home.tar -C /
    ```

### 参考博客

- [Linux 根目录爆满 解决](https://blog.csdn.net/e_wsq/article/details/79531493)