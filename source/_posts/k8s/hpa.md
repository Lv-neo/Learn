---
title: Horizontal Pod Autoscaling
p: k8s/hpa
date: 2018-11-30 15:52:04
categories: k8s
tags: [k8s,hap]
---

# Horizontal Pod Autoscaling

Horizontal Pod Autoscaling可以根据CPU使用率或应用自定义metrics自动扩展Pod数量（支持replication controller、deployment和replica set）。

* 控制管理器每隔30s（可以通过–horizontal-pod-autoscaler-sync-period修改）查询metrics的资源使用情况
* 支持三种metrics类型
  * 预定义metrics（比如Pod的CPU）以利用率的方式计算
  * 自定义的Pod metrics，以原始值（raw value）的方式计算
  * 自定义的object metrics
* 支持两种metrics查询方式：Heapster和自定义的REST API
* 支持多metrics

# 附录

[官方文档](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

# 配置示例

```
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: ${JOB_NAME}-hpa
  namespace: php
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    name: ${JOB_NAME}
    kind: Deployment
  minReplicas: 4
  maxReplicas: 6
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 300
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ${JOB_NAME}
  namespace: php
spec:
  rules:
  - host: ${JOB_NAME}.com
    http:
      paths:
      - backend:
         serviceName: ${JOB_NAME}
         servicePort: 80
```