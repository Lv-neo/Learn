---
title: zabbix 监控nginx
date: 2018-12-17 18:43:17
categories: zabbix
tags: zabbix
---

## 配置nginx 
在nginx的配置文件中，添加status配置。

```
server {
    listen 40080;
    server_name _;
    allow 127.0.0.1;
    allow 10.24.166.208;
    deny all;
    access_log off;
    location /php-fpm_status {
        #fastcgi_pass unix:/tmp/php-fpm.sock;
        fastcgi_pass 127.0.0.1:9000;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
    }
    location /nginx_status {
        stub_status on;
    }
}
```

- 测试

```
curl 127.0.0.1:40080/nginx_status
```
- 测试结果

```
Active connections: 1 
server accepts handled requests
 90 90 89 
Reading: 0 Writing: 1 Waiting: 0 
```

- nginx Status 详细说明：

```
Activeconnections:对后端发起的活动连接数；

server accepts 2349542：nginx 总共处理了2349542个连接；

handled：成功创建了64603417次握手；

requests：总共处理了8798670请求。

Reading：nginx读取客户端的header数；

Writing: nginx 返回给客户端的header数；

Waiting: nginx 请求处理完成，正在等待下一请求指令的连接。
```

## 在agentd上编写监控脚本
- 编写脚本

```
#!/bin/bash

Active(){
        wget --quiet -O - http://localhost:40080/nginx_status?auto |awk 'NR==1 {print$3}'
}
Reading(){
        wget --quiet -O - http://localhost:40080/nginx_status?auto |awk 'NR==4 {print$2}'
}
Writing(){
        wget --quiet -O - http://localhost:40080/nginx_status?auto |awk 'NR==4 {print$4}'
}
Waiting(){
        wget --quiet -O - http://localhost:40080/nginx_status?auto |awk 'NR==4 {print$6}'
}
Accepts(){
  		wget --quiet -O - http://localhost:40080/nginx_status?auto |awk 'NR==3 {print $1}'
}
Handled(){
		wget --quiet -O - http://localhost:40080/nginx_status?auto |awk 'NR==3 {print $2}'
}
Requests(){
		wget --quiet -O - http://localhost:40080/nginx_status?auto |awk 'NR==3 {print $3}'
}
$1
```

- 并且设置属主和属组为zabbix，赋予执行权限。

```
chmod o+x nginx_status.sh
chown zabbix.zabbix nginx_status.sh
```

- 修改nginx服务器上zabbix客户端的zabbix_agentd.conf配置文件

```
grep -v "^[#;]" zabbix_agentd.conf | grep -v "^$"

PidFile=/tmp/zabbix_agentd.pid
LogFile=/usr/local/logs/zabbix/zabbix_agentd.log
Server=10.24.166.208
ServerActive=10.24.166.208
Hostname=120.76.24.60
UnsafeUserParameters=1
UserParameter=idle.processe,/usr/local/zabbix/scripts/php-fpm_status.sh idle
UserParameter=total.processes,/usr/local/zabbix/scripts/php-fpm_status.sh total
UserParameter=active.processes,/usr/local/zabbix/scripts/php-fpm_status.sh active
UserParameter=max.active.processes,/usr/local/zabbix/scripts/php-fpm_status.sh mactive
UserParameter=listen.queue.len,/usr/local/zabbix/scripts/php-fpm_status.sh listenqueuelen
UserParameter=listen.queue,/usr/local/zabbix/scripts/php-fpm_status.sh listenqueue
UserParameter=start.since,/usr/local/zabbix/scripts/php-fpm_status.sh since
UserParameter=accepted.conn,/usr/local/zabbix/scripts/php-fpm_status.sh conn
UserParameter=nginx.connections.active,/usr/local/zabbix/scripts/nginx_status.sh Active
UserParameter=nginx.connections.reading,/usr/local/zabbix/scripts/nginx_status.sh Reading
UserParameter=nginx.connections.writing,/usr/local/zabbix/scripts/nginx_status.sh Writing
UserParameter=nginx.connections.waiting,/usr/local/zabbix/scripts/nginx_status.sh Waiting
UserParameter=nginx.accepts,/usr/local/zabbix/scripts/nginx_status.sh Accepts
UserParameter=nginx.handled,/usr/local/zabbix/scripts/nginx_status.sh Handled
UserParameter=nginx.requests,/usr/local/zabbix/scripts/nginx_status.sh Requests
```

- 重启zabbix agent

- zabbix server测试

```
/usr/local/zabbix/bin/zabbix_get -s 10.24.166.235 -p 10050 -k "nginx.connections.waiting"
```


## 导入模版
选择仓库中的nginx_status.xml
导入成功后，添加应用集
设置监控模版

```
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version>2.0</version>
    <date>2013-11-13T06:24:34Z</date>
    <groups>
        <group>
            <name>Templates</name>
        </group>
    </groups>
    <templates>
        <template>
            <template>Template nginx_status</template>
            <name>Template nginx_status</name>
            <groups>
                <group>
                    <name>Templates</name>
                </group>
            </groups>
           <applications>
                <application>
                    <name>nginx_status</name>
                </application>
            </applications>
            <items>
                <item>
                    <name>Active_connections</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>nginx.connections.active</key>
                    <delay>60</delay>
                    <history>30</history>
                    <trends>90</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>0</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>0</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications/>
                    <valuemap/>
                </item>
                <item>
                    <name>Reading_connection</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>nginx.connections.reading</key>
                    <delay>60</delay>
                    <history>30</history>
                    <trends>90</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>0</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>0</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications/>
                    <valuemap/>
                </item>
                <item>
                    <name>Waiting_connection</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>nginx.connections.waiting</key>
                    <delay>60</delay>
                    <history>30</history>
                    <trends>90</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>0</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>0</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications/>
                    <valuemap/>
                </item>
                <item>
                    <name>Writing_connection</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>nginx.connections.writing</key>
                    <delay>60</delay>
                    <history>30</history>
                    <trends>90</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>0</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>0</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications/>
                    <valuemap/>
                </item>
                <item>
                    <name>Accepts</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>nginx.accepts</key>
                    <delay>60</delay>
                    <history>30</history>
                    <trends>90</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>0</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>0</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications/>
                    <valuemap/>
                </item>
                <item>
                    <name>Handled</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>nginx.handled</key>
                    <delay>60</delay>
                    <history>30</history>
                    <trends>90</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>0</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>0</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications/>
                    <valuemap/>
                </item>
                <item>
                    <name>Requests</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>nginx.requests</key>
                    <delay>60</delay>
                    <history>30</history>
                    <trends>90</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>0</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>0</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications/>
                    <valuemap/>
                </item>
            </items>
            <discovery_rules/>
            <macros/>
            <templates/>
            <screens/>
        </template>
    </templates>
    <graphs>
        <graph>
            <name>nginx_connections</name>
            <width>900</width>
            <height>200</height>
            <yaxismin>0.0000</yaxismin>
            <yaxismax>100.0000</yaxismax>
            <show_work_period>1</show_work_period>
            <show_triggers>1</show_triggers>
            <type>0</type>
            <show_legend>1</show_legend>
            <show_3d>0</show_3d>
            <percent_left>0.0000</percent_left>
            <percent_right>0.0000</percent_right>
            <ymin_type_1>0</ymin_type_1>
            <ymax_type_1>0</ymax_type_1>
            <ymin_item_1>0</ymin_item_1>
            <ymax_item_1>0</ymax_item_1>
            <graph_items>
                <graph_item>
                    <sortorder>0</sortorder>
                    <drawtype>0</drawtype>
                    <color>C80000</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template nginx_status</host>
                        <key>nginx.connections.active</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>1</sortorder>
                    <drawtype>0</drawtype>
                    <color>00C800</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template nginx_status</host>
                        <key>nginx.connections.reading</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>2</sortorder>
                    <drawtype>0</drawtype>
                    <color>0000C8</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template nginx_status</host>
                        <key>nginx.connections.waiting</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>3</sortorder>
                    <drawtype>0</drawtype>
                    <color>C800C8</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template nginx_status</host>
                        <key>nginx.connections.writing</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>4</sortorder>
                    <drawtype>0</drawtype>
                    <color>00EE00</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template nginx_status</host>
                        <key>nginx.accepts</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>5</sortorder>
                    <drawtype>0</drawtype>
                    <color>EE0000</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template nginx_status</host>
                        <key>nginx.handled</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>6</sortorder>
                    <drawtype>0</drawtype>
                    <color>EEEE00</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template nginx_status</host>
                        <key>nginx.requests</key>
                    </item>
                </graph_item>
            </graph_items>
        </graph>
    </graphs>
</zabbix_export>

```


