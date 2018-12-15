---
title: 客户端解决ssh连接断开的问题
date: 2018-12-13 16:34:08
categories: ssh
tags: ssh
---

# 客户端解决ssh连接断开的问题

今天有小伙伴又在问，客户端连服务器ssh总是断开，其实这个问题很简单。

## 核心问题点在哪里

其实这个问题，核心点在与客户端与服务端的心跳监听和防火墙

NAT/firewall在处理网络连接时候，对闲置连接会断开处理，减少空闲连接的占用。（具体请自己查阅资料，以前看的了）

另外客户端与服务端之间有心跳保持连接

看服务端ssh全局配置 /etc/ssh/sshd_config

```
TCPKeepAlive yes //核心保证连接
#UseLogin no
#UsePrivilegeSeparation sandbox
#PermitUserEnvironment no
#Compression delayed
ClientAliveInterval 60 //下发客户端心跳包
ClientAliveCountMax 5  //多少次未响应就断开
```

## 怎么解决

一般情况，我们将客户端在对应操作系统上设置下就好了

### 客户端配置
vim ~/.ssh/config

```
Host * //#表示需要启用该规则的服务端（域名或ip）
ClientAliveInterval 60 //下发客户端心跳包
ClientAliveCountMax 5  //多少次未响应就断开
```

### ssh增加连接参数

```
ssh -o ServerAliveInterval=30 user@host
```

### 修改连接工具的配置
通过改变连接工具的一些默认配置，把keepalive的配置打开起来即可：

* secureCRT：会话选项 - 终端 - 反空闲 - 发送NO-OP每xxx秒，设置一个非0值。
* putty：Connection - Seconds between keepalive(0 to turn off)，设置一个非0值。
* iTerm2：profiles - sessions - When idle - send ASCII code.
* XShell：session properties - connection - Keep Alive - Send keep alive message while this session connected. Interval [xxx] sec.