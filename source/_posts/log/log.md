---
title: 日志服务
p: /ops/log
date: 2018-12-01 14:28:13
categories: log
tags: log
---

# 日志服务

日志服务的特点

* 访问量大
* 访问频率低
* 写问题
    * 缓存策略
        * 多集缓存策略
        * 数量纬度
        * 时间纬度
    * 内存使用
    * 多组线程池
    * 顺序性
        * 命名格式
        * hashkey
            * hostip
            * containerName
        * partition
    * 调优复杂度
    * 压缩
        * gzip
* 存储消费问题
    * 消费
        hive
    * hdfs
* 无状态
* 套件/库的支持
    * scarl
    * kafka

