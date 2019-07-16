---
title: ngx_stream_core_module模块
date: 2019-04-29 11:50:55
categories:
tags:
---

# 官方文档

* http://nginx.org/en/docs/stream/ngx_stream_core_module.html

## 使用说明

> HTTP负载均衡，也就是我们通常所有"七层负载均衡"，工作在第七层"应用层"。而TCP负载均衡，就是我们通常所说的"四层负载均衡"，工作在"网络层"和"传输层"。例如，LVS（Linux Virtual Server，Linux虚拟服务）和F5（一种硬件负载均衡设备），也是属于"四层负载均衡"


nginx-1.9.0 已发布，该版本增加了stream 模块用于一般的TCP 代理和负载均衡，ngx_stream_core_module 这个模块在1.90版本后将被启用。但是并不会默认安装，
需要在编译时通过指定 --with-stream 参数来激活这个模块。
 
### 配置Nginx编译文件参数

```
./configure --with-http_stub_status_module --with-stream
```

## 编译、安装，make && make install

### 配置nginx.conf文件

```
stream {
    upstream kevin {
        server 192.168.10.10:8080;     #这里配置成要访问的地址
        server 192.168.10.20:8081;
        server 192.168.10.30:8081;     #需要代理的端口，在这里我代理一一个kevin模块的接口8081
    }
    server {
        listen 8081;               #需要监听的端口
        proxy_timeout 20s;
        proxy_pass kevin;
    }
}
```
 
* 创建最高级别的stream（与http同一级别），定义一个upstream组 名称为kevin，由多个服务组成达到负载均衡 定义一个服务用来监听TCP连接（如：8081端口），并且把他们代理到一个upstream组的kevin中，配置负载均衡的方法和参数为每个server；配置些如：连接数、权重等等。

* 首先创建一个server组，用来作为TCP负载均衡组。定义一个upstream块在stream上下文中，在这个块里面添加由server命令定义的server，指定他的IP地址和主机名（能够被解析成多地址的主机名)和端口号。下面的例子是建立一个被称之为kevin组，两个监听1395端口的server ，一个监听8080端口的server。

```
upstream kevin {
    server 192.168.10.10:8080;     #这里配置成要访问的地址
    server 192.168.10.20:8081;
    server 192.168.10.30:8081;     #需要代理的端口，在这里我代理一一个kevin模块的接口8081
}
```
 
> 需要特别注意的是：你不能为每个server定义协议，因为这个stream命令建立TCP作为整个 server的协议了。
 
配置反向代理使Nginx能够把TCP请求从一个客户端转发到负载均衡组中(如：kevin组)。在每个server配置块中 通过每个虚拟server的server的配置信息和在每个server中定义的监听端口（客户端需求的代理端口号，如我推流的的是kevin协议，则端口号为：8081）的配置信息和proxy_passs 命令把TCP通信发送到upstream的哪个server中去。下面我们将TCP通信发送到kevin 组中去。

```
server {
        listen 8081;    #需要监听的端口
        proxy_timeout 20s;
        proxy_pass kevin;
    }
```


当然我们也可以采用单一的代理方式：
 
```
server {
        listen 8081;  #需要监听的端口
        proxy_timeout 20s;
        proxy_pass  192.168.10.30:8081;  #需要代理的端口，在这里我代理一一个kevin模块的接口8081
}
```

### 改变负载均衡的方法：

默认nginx是通过轮询算法来进行负载均衡的通信的。引导这个请求循环的到配置在upstream组中server端口上去。 因为他是默认的方法，这里没有轮询命令，
只是简单的创建一个upstream配置组在这儿stream山下文中，而且在其中添加server。
 
#### least-connected 

>对于每个请求，nginx plus选择当前连接数最少的server来处理:

```
upstream kevin {
　　 least_conn;
    server 192.168.10.10:8080;     #这里配置成要访问的地址
    server 192.168.10.20:8081;
    server 192.168.10.30:8081;     #需要代理的端口，在这里我代理一一个kevin模块的接口8081
}
```
 
#### least time 

>对于每个链接,nginx pluns 通过几点来选择server的： 最底平均延时：通过包含在least_time命令中指定的参数计算出来的：

* connect：连接到一个server所花的时间
* first_byte：接收到第一个字节的时间
* last_byte：全部接收完了的时间 最少活跃的连接数:

``` 
upstream kevin {
　　　　least_time first_byte;
    server 192.168.10.10:8080;     #这里配置成要访问的地址
    server 192.168.10.20:8081;
    server 192.168.10.30:8081;     #需要代理的端口，在这里我代理一一个kevin模块的接口8081
}
```

#### 普通的hash算法

>nginx plus选择这个server是通过user_defined 关键字，就是IP地址：$remote_addr;
 
```
upstream kevin {
　　 hash $remote_addr consistent;
    server 192.168.10.10:8080 weight=5;    #这里配置成要访问的地址
    server 192.168.10.20:8081 max_fails=2 fail_timeout=30s;
    server 192.168.10.30:8081 max_conns=3;    #需要代理的端口，在这里我代理一一个kevin模块的接口8081
}
```

### 使用官方docker镜像实现

default.conf

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}

stream {
    upstream backend {
        server 192.168.222.111:8080; 
    }
    server {
        listen 80;
        # proxy_connect_timeout 1s;
        proxy_pass backend;
    }
}
```

docker-compose.yml

```
version: '3'

services:
  nginx:
    restart: always
    image: nginx
    volumes:
      - "./default.conf:/etc/nginx/nginx.conf"
    ports:
      - "80:80"
    labels:
      - nginx
    
```