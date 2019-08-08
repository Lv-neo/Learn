---
title: minikube
date: 2019-07-16 15:46:51
categories: k8s
tags:
  -- k8s
  -- minikube
---

# minikube

## 安装好docker

## 安装脚本(MAC版本)

```
# !/bin/sh

echo "install kubectl"

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

kubectl version

echo "install minikube"

curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube

sudo cp minikube /usr/local/bin && rm minikube

minikube version

echo "minikube start"

minikube start --registry-mirror=https://registry.docker-cn.com
```
