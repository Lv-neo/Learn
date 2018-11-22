---
title: namespace简介
categories: k8s
tags:
  - k8s
---

# namespace简介

Namespace（命名空间）是kubernetes系统中的另一个重要的概念，通过将系统内部的对象“分配”到不同的Namespace中，形成逻辑上分组的不同项目、小组或用户组，便于不同的分组在共享使用整个集群的资源的同时还能被分别管理。

   Kubernetes集群在启动后，会创建一个名为“default”的Namespace，如果不特别指明Namespace，则用户创建的Pod、RC、Service都被系统创建到“default”的Namespace中。

## 查询

```
# kubectl get namespaces
NAME      STATUS    AGE
default   Active    6d
```

## 创建

yml模式
```
apiVersion: v1
kind: Namespace
metadata:
   name: xxx
   labels:
     name: xxx

```

shell模式
```
# kubectl create namespace xxx
namespace "xxx" created
```

## 创建k8s服务指定namespace

yml
```
...
metadata:
  name: php
  labels:
    name: php
  namespace: xxx
...
```

## 根据namespcae查询服务

```
kubectl get [po|svc|ing|rs|secrets|psp|deploy|hap|describe|...] [name] [--namespace|-n]=[namespacename]
```