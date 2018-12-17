---
title: zabbix解决中文乱码
date: 2018-12-17 18:41:27
categories: zbixabbix
tags: zab
---

# 解决中文乱码

## 选择字体
在windows下C:\Windows\Fonts\simkai.ttf（楷体）上传到服务器zabbix网站目录fonts目录下。
[SIMKAI.ttf](zabbix/files/SIMKAI.ttf)本站下载

## 修改zabbix php配置文件
//define('ZBX_FONT_NAME', 'DejaVuSans');
define('ZBX_FONT_NAME', 'SIMKAI');
 
//define('ZBX_GRAPH_FONT_NAME', 'DejaVuSans'); // font file name
define('ZBX_GRAPH_FONT_NAME', 'SIMKAI'); 
// font file name

## 刷新页面
中文乱码就解决了


