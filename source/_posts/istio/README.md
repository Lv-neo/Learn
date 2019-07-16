---
title: Sidecar
date: 2019-04-13 10:37:47
categories:
tags:
---

# Sidecar模式

一种单节点、多容器的应用设计形式。

Sidecar主张以额外的容器来扩展或增强主容器，而这个额外的容器被称为Sidecar容器。

Web-server容器可以与一个sidecar容易共同部署，该sidecar容器从文件系统中读取由Web-server容器生成的web-server日志，并将日志/stream发送到原称服务器（remote server）。Sidecar容器通过处理web-server日志来作为web-server容器的补充。当然，可能会有人说，为什么web-server不自己处理自己的日志？答案在于以下几点：

隔离（separation of concerns）：让每个容器都能够关注核心问题。比如web-server提供网页服务，而sidecar则处理web-server的日志，这有助于在解决问题时不与其他问题的处理发生冲突；
单一责任原则（single responsibility principle）：容器发生变化需要一个明确的理由。换句更容易理解的话来说，每个容器都应该是能够处理好“一件”事情的，而根据这一原则，我们应该让不同的容器分别展开工作，应该让它们处理的问题足够独立；
内聚性/可重用性（Cohesiveness/Reusability）：使用sidecar容器处理日志，这个容器按道理应该可以应用的其他地方重用；
