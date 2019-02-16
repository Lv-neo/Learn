---
title: springboot中的soap
date: 2018-12-29 23:42:02
categories: soap
tags:
    - java
    - soap
---

# springboot中的soap

## 什么是soap？

* 参考阅读[简单对象访问协议](https://zh.wikipedia.org/wiki/%E7%AE%80%E5%8D%95%E5%AF%B9%E8%B1%A1%E8%AE%BF%E9%97%AE%E5%8D%8F%E8%AE%AE)

## 什么是wsdl？

* 参考阅读[WSDL教程](http://www.w3school.com.cn/wsdl/index.asp)

## Linux下通过curl获取wsdl

* 通过curl方式获取
    * 如果需要用户名和密码通过--user设定
    * 如果要证书，使用-k忽略证书验证

```
curl --user "username:password" -k "https://domain/example?wsdl" > example.wsdl
```

## springboot构建soap服务

* [springboot编写soap服务端-官方示例](https://spring.io/guides/gs/producing-web-service/)
* [springboot编写soap客户端-官方示例](https://spring.io/guides/gs/consuming-web-service/)
<!-- * [springboot Ws Basic Authentication](/Learn/java/ws/basic/) -->
* [springboot Ws Basic Authentication](https://codenotfound.com/spring-ws-basic-authentication-example.html)
* [springboot WS - SOAP Header](https://codenotfound.com/spring-ws-soap-header-example.html)
* [springboot WS - HTTPS Client-Server Example](https://codenotfound.com/spring-ws-https-client-server-example.html)

## 系列阅读

* https://codenotfound.com/categories/#spring-ws
* https://memorynotfound.com/category/spring-framework/spring-ws/

## 延伸阅读

* [Springboot jar 环境下只能使用流引入resource 解决SSLContext证书问题](https://stackoverflow.com/questions/27724544/specifying-trust-store-information-in-spring-boot-application-properties)