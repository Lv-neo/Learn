---
title: kubernetes 滚动更新发布及回滚
date: 2018-12-12 14:36:59
categories: k8s
tags: k8s
---

# kubernetes 滚动更新发布及回滚

可以处理的资源
  * deployments  
  * daemonsets  
  * statefulsets

```
Available Commands:
  history     View rollout history
  pause       Mark the provided resource as paused
  resume      Resume a paused resource
  status      Show the status of the rollout
  undo        Undo a previous rollout
```

## 针对deployment记录record

```
kubectl  apply -f **** --record
```

## 查看当前状态
```
kubectl rollout status deployment/[DEPLOYMENT NAME] -w -n=[NAMESPACE]
```

## 查看历史
```
kubectl rollout history deployment/[DEPLOYMENT NAME] -n=[NAMESPACE]
```

## 回滚到指定版本
```
kubectl rollout undo deployment/[DEPLOYMENT NAME] --to-revision=[VERSION ID] -n=[NAMESPACE]
```