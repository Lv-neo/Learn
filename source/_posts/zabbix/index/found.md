---
title: zabbix主被动模式
date: 2018-12-17 18:39:17
categories: zabbix
tags: zabbix
---

# zabbix主被动模式
使用zabbix代理有很多好处，一方面可以监控不可达的远程区域；另一方面当监控项目数以万计的时候使用代理可以有效分担zabbix server压力，也简化分布式监控的维护。

## 区别
主动、被动模式都是相对于proxy来说的。proxy主动发送数据就是主动模式；proxy等待server的请求，再发送数据就是被动模式。因为主动模式可以有效减轻zabbix server压力，需要监控的东西很多时一定要把监控模式更改为主动监控

### zabbix客户端主被动模式
zabbix客户端分数据给服务端分为主被动两种模式,主动模式是zabbix客户端主动向服务端发送数据,被动模式是被动等待客户端来取数据

主动模式的流程是 客户端每隔一段时间主动向服务端发起连接请求-->服务端收到请求,查询客户端需要取的item信息,发送给客户端-->客户端收集数据发送服务端-->结束

而被动模式是客户端开一个端口默认10050,等待服务端来取数据,然后客户端收集数据发送到服务端,后结束

### 实践
zabbix_server端当主机数量过多的时候，由Server端去收集数据，Zabbix会出现严重的性能问题，主要表现如下：
1、当被监控端到达一个量级的时候，Web操作很卡，容易出现502
2、图层断裂
3、开启的进程（Pollar）太多，即使减少item数量，以后加入一定量的机器也会有问题

所以下面主要往两个优化方向考虑：
1、添加Proxy节点或者Node模式做分布式监控
2、调整Agentd为主动模式
由于第一个方案需要增加物理机器，所以首先尝试第二方案。

* 服务端配置

```
ListenPort=10051
LogFile=/usr/local/logs/zabbix/zabbix_server.log
DBHost=127.0.0.1
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
StartTrappers=200
Timeout=5
LogSlowQueries=3000
```

zabbix_server.conf 配置调整：
StartPollers=100 
首先把这个主动收集数据进程减少，原来开到700多
StartTrappers=200
然后把这个负责处理Agentd推送过来的数据的进程开大一些，就可以了

* 被监控端 zabbix_Agentd.conf 的配置调整：

```
LogFile=/usr/local/logs/zabbix/zabbix_agent.log
StartAgents=0          #客户端agent模式，仅为主动模式,值为0的时候，被监控端的zabbix_agentd 不监听本地端口，所以无法在 netstat -tunpl 中查看到zabbix_agentd进程
ServerActive=**.**.**.**    #zabbix_server的ip
Hostname=test_host    #重要：主机名,唯一/
RefreshActiveChecks=1800    #被监控端到服务器获取监控项的周期
BufferSize=200        #被监控端存储监控信息的空间大小
Timeout=10            #超时时间
```
比较重要的参数是ServerActive和Hostname，ServerActive是指定Agentd收集的数据往哪里发送，Hostname是必须要和Server端添加主机时的主机名一样，这样Server端接收到数据才能找到对应关系。



