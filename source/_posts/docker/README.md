---
title: docker
date: 2018-12-17 19:16:58
categories: docker
tags: docker
---

- [alpine](/Learn/ops/docker/alpine)
- [busybox](/Learn/ops/docker/busybox)
- [docker安装脚本](/Learn/ops/docker/install)

# docker常用命令

## docker system

* 类似于Linux上的df命令，用于查看Docker的磁盘使用情况:
```
docker system df
```

* docker system prune命令可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)。docker system prune -a命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了。
```
docker system prune -a
```

## docker rm

```
docker rmi $(docker images | grep "none" | awk '{print $3}')
```

```
docker rm $(sudo docker ps -aq -f status=exited)
```