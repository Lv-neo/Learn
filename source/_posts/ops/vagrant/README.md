---
title: vagrant
date: 2018-12-17 18:48:21
categories: vagrant
tags: vagrant
---

# 什么是Vagrant

Vagrant是一款用来构建虚拟开发环境的工具，底层支持VirtualBox,VMware,AWS作为虚拟机。

## 为什么要用vagrant？

* 统一开发环境。一次配置打包，统一分发给团队成员，统一团队开发环境，解决诸如“编码问题”，“缺少模块”，“配置文件不同”带来的问题；
* 避免重复搭建开发环境。新员工加入，不用浪费时间搭建开发环境，快速加入开发，减少时间成本的浪费；
* 多个相互隔离的开发环境，可以在不同的box中跑不同的环境，卸载清除也快捷。
* 有人说docker好，我举双手同意，不过docker用于生产更灵活，开发测试。构建一个vagrant纯净系统，一键安装oneinstack，改个防火墙就好了。如果是一直用docker的大牛可以无视

## vagrant 官网
https://www.vagrantup.com

## virtualbox 官网
https://www.virtualbox.org/wiki/Downloads

## vagrant box
https://www.vagrantup.com/docs/boxes.html
http://www.vagrantbox.es/
巨慢，建议用百度云盘下载，你懂的


## vagrant安装和使用
[vagrant安装和使用](/Learn/ops/vagrant/install/)