---
title: 回滚方案和思路
date: 2018-12-12 15:59:29
categories: k8s
tags: k8s
---

# k8s回滚方案和思路

* deployment（官方推荐）
    * apply imgaeId
* ReplicationController
    * RollingUpdate

如果你使用ReplicationController创建的pod可以使用kubectl rollingupdate命令滚动升级，如果使用的是Deployment创建的Pod可以直接修改yaml文件后执行kubectl apply即可。

## ReplicationController
Replication Controller为Kubernetes的一个核心内容，应用托管到Kubernetes之后，需要保证应用能够持续的运行，Replication Controller就是这个保证的key，主要的功能如下：

* 确保pod数量：它会确保Kubernetes中有指定数量的Pod在运行。如果少于指定数量的pod，Replication Controller会创建新的，反之则会删除掉多余的以保证Pod数量不变。
* 确保pod健康：当pod不健康，运行出错或者无法提供服务时，Replication Controller也会杀死不健康的pod，重新创建新的。
* 弹性伸缩 ：在业务高峰或者低峰期的时候，可以通过Replication Controller动态的调整pod的数量来提高资源的利用率。同时，配置相应的监控功能（Hroizontal Pod Autoscaler），会定时自动从监控平台获取Replication Controller关联pod的整体资源使用情况，做到自动伸缩。
* 滚动升级：滚动升级为一种平滑的升级方式，通过逐步替换的策略，保证整体系统的稳定，在初始化升级的时候就可以及时发现和解决问题，避免问题不断扩大。

Deployment
Deployment同样为Kubernetes的一个核心内容，主要职责同样是为了保证pod的数量和健康，90%的功能与Replication Controller完全一样，可以看做新一代的Replication Controller。但是，它又具备了Replication Controller之外的新特性：

* Replication Controller全部功能：Deployment继承了上面描述的Replication Controller全部功能。
* 事件和状态查看：可以查看Deployment的升级详细进度和状态。
* 回滚：当升级pod镜像或者相关参数的时候发现问题，可以使用回滚操作回滚到上一个稳定的版本或者指定的版本。
* 版本记录: 每一次对Deployment的操作，都能保存下来，给予后续可能的回滚使用。
* 暂停和启动：对于每一次升级，都能够随时暂停和启动。
* 多种升级方案：Recreate：删除所有已存在的pod,重新创建新的; RollingUpdate：滚动升级，逐步替换的策略，同时滚动升级时，支持更多的附加参数，例如设置最大不可用pod数量，最小升级间隔时间等等。

## 参考

[Rolling update机制解析](http://dockone.io/article/328)

[Running a Stateless Application Using a Deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)

[Simple Rolling Update](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/simple-rolling-update.md)

[使用kubernetes的deployment进行RollingUpdate](https://segmentfault.com/a/1190000008232770)

[Kubernetes Deployment Concept](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)