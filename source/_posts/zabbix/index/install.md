---
title: zabbix安装
date: 2018-12-17 18:37:47
categories: zabbix
tags: zabbix
---

＃安装说明

* agent安装
自己写了个脚本，适合简单安装

```
#!/bin/sh
version=3.0.4
zabbix_home="/usr/local/zabbix"
dir=$(pwd)
url="http://nchc.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/${version}/zabbix-${version}.tar.gz"
user=zabbix
group=zabbix

echo "the install dir is "$dir

#download zabbix
if [ -f "zabbix-${version}.tar.gz" ] ; then
echo "zabbix-${version}.tar.gz has been download"
else
echo "download zabbix-${version}.tar.gz"
wget $url
fi

#tar zabbix
if [ -d "zabbix-${version}" ] ; then
echo "the zabbix-${version} is exist"
else 
echo "tar zabbix-${version}.tar.gz"
tar -zxvf zabbix-${version}.tar.gz
fi

#create group if not exists  
egrep "^$group" /etc/group >& /dev/null  
if [ $? -ne 0 ]  
then  
    groupadd $group  
fi  
  
#create user if not exists  
egrep "^$user" /etc/passwd >& /dev/null  
if [ $? -ne 0 ]  
then  
    useradd -g $group $user  
fi  

#configure
if [ -f "${zabbix_home}/sbin/zabbix_agentd" ]; then
echo "install success"
else
cd "zabbix-${version}"
./configure --prefix=${zabbix_home} --enable-agent
make install
fi

echo "THE zabbix_agented:  ${zabbix_home}/sbin/zabbix_agentd"
echo "THE conf:   ${zabbix_home}/etc/zabbix_agentd.conf"


```


