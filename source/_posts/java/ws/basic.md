---
title: spring-boot Ws Basic Authentication 
date: 2019-01-04 18:25:45
categories: java
tags:
    -java
    -webservice
    -spring-boot
---

# spring-boot ws basic authentication

## 什么是basic authentication

basic authentication 是通过HTTP user-agent通过提供username和password提供的一种验证方法。主要就是在HTTP头Authorization: Basic <credentials>，其中credentials是由冒号连接的id和密码的base64编码

* [相关阅读](https://en.wikipedia.org/wiki/Basic_access_authentication)

## 依赖描述

* Spring-WS 2.4
* Spring Security 4.2
* HttpClient 4.5
* Spring Boot 1.5
* Maven 3.5

## 