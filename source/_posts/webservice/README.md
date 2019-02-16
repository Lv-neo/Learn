---
title: webservice
date: 2018-12-18 12:16:54
categories: java
tags: java
---

# 什么是webservice

其实计算机的工作无非是数据的处理，程序来完成这些处理，具体涉及到cpu内存的就不再这里赘述，我们把计算机程序提供的功能，称之为服务（"service"），那么不需要网络的服务，我们称之为本地服务。那么需要网络的就是网路服务，本质就是通过网络调用网络上其他计算机的服务和资源。

## 看看维基百科的说明

Web服务是一种[服务导向架构](https://zh.wikipedia.org/wiki/%E6%9C%8D%E5%8B%99%E5%B0%8E%E5%90%91%E6%9E%B6%E6%A7%8B)的技术，通过标准的Web协议提供服务，目的是保证不同平台的应用服务可以互操作。

根据W3C的定义，Web服务（Web service）应当是一个软件系统，用以支持网络间不同机器的互动操作。网络服务通常是许多应用程序接口（API）所组成的，它们透过网络，例如国际互联网（Internet）的远程服务器端，执行客户所提交服务的请求。

尽管W3C的定义涵盖诸多相异且无法介分的系统，不过通常我们指有关于主从式架构（Client-server）之间根据SOAP协议进行传递XML格式消息。无论定义还是实现，WEB服务过程中会由服务器提供一个机器可读的描述（通常基于WSDL）以辨识服务器所提供的WEB服务。另外，虽然WSDL不是SOAP服务端点的必要条件，但目前基于Java的主流WEB服务开发框架往往需要WSDL实现客户端的源代码生成。一些工业标准化组织，比如WS-I，就在WEB服务定义中强制包含SOAP和WSDL。

## 核心定义

考虑到并没某个独立文档包含一切相关内容，可采用模块化的方式给出对WEB服务的描述，但不能给出一个“绝对全面和准确”的定义。受外部环境和实现技术影响，各方给出的核心定义可能稍有出入，但通常包括：

* [SOAP](https://zh.wikipedia.org/wiki/SOAP)
一个基于XML的可扩展消息信封格式，需同时绑定一个网络传输协议。这个协议通常是HTTP或HTTPS，但也可能是SMTP或XMPP。
* [WSDL](https://zh.wikipedia.org/wiki/WSDL)
一个XML格式文档，用以描述服务端口访问方式和使用协议的细节。通常用来辅助生成服务器和客户端代码及配置信息。
* [UDDI](https://zh.wikipedia.org/wiki/UDDI)
一个用来发布和搜索WEB服务的协议，应用程序可借由此协议在设计或运行时找到目标WEB服务。

## 常用web服务方式

* 远程过程调用

WEB服务提供一个分布式函数或方法接口供用户调用，这是一种比较传统的方式。通常，在WSDL中对RPC接口进行定义（类似于早期的XML-RPC）。

尽管最初的WEB服务广泛采用RPC方式部署，但针对其过于紧密之耦合性的批评声也随之不断。这是因为RPC式WEB服务实质上是利用一个简单的映射，以把用户请求直接转化成为一个特定语言编写的函数或方法。如今，多数服务提供商认定此种方式在未来将难有作为，在他们的推动下，WS-I基本协议集（WS-I Basic Profile）已不再支持远程过程调用。

* 服务导向架构

现在，业界比较关注的是遵从服务导向架构（Service-oriented architecture，SOA）概念来构筑WEB服务。在服务导向架构中，通讯由消息驱动，而不再是某个动作（方法调用）。这种WEB服务也被称作面向消息的服务。

SOA式WEB服务得到了大部分主要软件供应商以及业界专家的支持和肯定。作为与RPC方式的最大差别，SOA方式更加关注如何去连接服务而不是去特定某个实现的细节。WSDL定义了联络服务的必要内容。

* 表述性状态转移

表述性状态转移式（Representational state transfer，REST）WEB服务类似于HTTP或其他类似协议，它们把接口限定在一组广为人知的标准动作中（比如HTTP的GET、PUT、DELETE）以供调用。此类WEB服务关注与那些稳定的资源的互动，而不是消息或动作。

此种服务可以通过WSDL来描述SOAP消息内容，通过HTTP限定动作接口；或者完全在SOAP中对动作进行抽象。

# Web service的发展趋势

* resuful还是越来越多占据主导
* json格式会比xml要更多

# 外部链接

https://zh.wikipedia.org/wiki/Web%E6%9C%8D%E5%8A%A1

http://www.ruanyifeng.com/blog/2009/08/what_is_web_service.html


