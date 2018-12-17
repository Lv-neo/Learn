---
title: docker安装脚本
date: 2018-12-17 19:19:37
categories: docker
tags: docker
---

# 常用镜像源

国内访问 Docker Hub 有时会遇到困难，此时可以配置镜像加速器。国内很多云服务商都提供了加速器服务，例如：

* [阿里云加速器](https://cr.console.aliyun.com/#/accelerator)
* [DaoCloud 加速器](https://www.daocloud.io/mirror#accelerator-doc)
* [灵雀云加速器](http://docs.alauda.cn/feature/accelerator.html)

注册用户并且申请加速器，会获得如 https://xxxx.mirror.aliyuncs.com 这样的地址。我们需要将其配置给 Docker 引擎。

# 安装脚本

```
#!/bin/bash

yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum makecache fast

yum -y install docker-ce

mkdir -p /etc/docker

//替换成你的阿里源
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxxxxx.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload

service docker start

systemctl enable docker

yum -y install epel-release

yum -y install python-pip

pip install docker-compose
```