---
title: 如何支持私有镜像
categories: k8s
tags:
  - k8s
---

# 如何支持私有镜像

```
kubectl create secret docker-registry regsecret -n namespace --docker-server=registry-internal.cn-hangzhou.aliyuncs.com --docker-username=abc@aliyun.com --docker-password=xxxxxx --docker-email=abc@aliyun.com
```

regsecret： 指定密钥的键名称，可自行定义。
—docker-server：指定 Docker 仓库地址。
—docker-username: 指定 Docker 仓库用户名。
—docker-password：指定 Docker 仓库登录密码。
—docker-email：指定邮件地址（选填）。
-n ：namespace 默认指向default

yml 文件加入密钥参数。
```
containers:
- name: channel
  image: registry-internal.cn-hangzhou.aliyuncs.com/abc/test:1.0
ports:
- containerPort: 8114
imagePullSecrets:
- name: regsecret
其中，
```
imagePullSecrets 是声明拉取镜像时需要指定密钥。
regsecret 必须和上面生成密钥的键名一致。
image 中的 Docker 仓库名称必须和 --docker-server 中的 Docker 仓库名一致。
还需要namespace一致，否则默认只能处理default的项目

参考阅读
https://help.aliyun.com/document_detail/86307.html?spm=a2c4g.11186623.6.560.750f5820y7pzUk



