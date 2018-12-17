---
title: zabbix 监控php-fpm
date: 2018-12-17 18:43:21
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
curl 127.0.0.1:40080/php-fpm_status
```
- 测试结果

```
pool:                 www
process manager:      dynamic
start time:           13/May/2016:18:57:26 +0800
start since:          73244
accepted conn:        10780
listen queue:         0
max listen queue:     1
listen queue len:     65535
idle processes:       11
active processes:     1
total processes:      12
max active processes: 7
max children reached: 0
slow requests:        9
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
idle(){
        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "idle processes" |awk '{print$3}'
}

total(){
        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "total processes" |awk '{print$3}'
}

active(){
        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "active" |awk '{print$3}'|grep -v "process"
}

mactive(){

        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "max active processes:" |awk '{print$4}'
}

listenqueuelen(){
        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "listen queue len" |awk '{print$4}'
}

listenqueue(){
        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "listen queue:"|grep -vE "len|max"|awk '{print$3}'
}

since(){
        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "start since: " |awk '{print$3}'
}

conn(){
        wget --quiet -O - http://127.0.0.1:40080/php-fpm_status?auto |grep "accepted conn" |awk '{print$3}'
}
$1

```

- 并且设置属主和属组为zabbix，赋予执行权限。

```
chmod o+x php-fpm_status.sh
chown zabbix.zabbix php-fpm_status.sh
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
/usr/local/zabbix/bin/zabbix_get -s 10.24.166.235 -p 10050 -k "idle.processe"
```


## 导入模版
选择仓库中的php-fpm_status.xml
导入成功后，添加应用集
设置监控模版

```
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version>2.0</version>
    <date>2013-11-13T06:26:56Z</date>
    <groups>
        <group>
            <name>Templates</name>
        </group>
    </groups>
    <templates>
        <template>
            <template>Template php-fpm</template>
            <name>Template php-fpm</name>
            <groups>
                <group>
                    <name>Templates</name>
                </group>
            </groups>
            <applications>
                <application>
                    <name>php-fpm</name>
                </application>
            </applications>
            <items>
                <item>
                    <name>accepted conn</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>accepted.conn</key>
                    <delay>60</delay>
                    <history>90</history>
                    <trends>365</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units>k</units>
                    <delta>1</delta>
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
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
                <item>
                    <name>active processes</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>active.processes</key>
                    <delay>60</delay>
                    <history>90</history>
                    <trends>365</trends>
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
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
                <item>
                    <name>listen queue</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>listen.queue</key>
                    <delay>60</delay>
                    <history>90</history>
                    <trends>365</trends>
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
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
                <item>
                    <name>listen queue len</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>listen.queue.len</key>
                    <delay>60</delay>
                    <history>90</history>
                    <trends>365</trends>
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
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
                <item>
                    <name>max active processes</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>max.active.processes</key>
                    <delay>30</delay>
                    <history>90</history>
                    <trends>365</trends>
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
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
                <item>
                    <name>php-fpm_idle_processes</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>idle.processe</key>
                    <delay>60</delay>
                    <history>90</history>
                    <trends>365</trends>
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
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
                <item>
                    <name>start since</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>start.since</key>
                    <delay>60</delay>
                    <history>90</history>
                    <trends>365</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units/>
                    <delta>1</delta>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privpassphrase/>
                    <formula>1</formula>
                    <delay_flex/>
                    <params/>
                    <ipmi_sensor/>
                    <data_type>2</data_type>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
                <item>
                    <name>total processes</name>
                    <type>0</type>
                    <snmp_community/>
                    <multiplier>0</multiplier>
                    <snmp_oid/>
                    <key>total.processes</key>
                    <delay>60</delay>
                    <history>90</history>
                    <trends>365</trends>
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
                    <applications>
                        <application>
                            <name>php-fpm</name>
                        </application>
                    </applications>
                    <valuemap/>
                </item>
            </items>
            <discovery_rules/>
            <macros>
                <macro>
                    <macro>{$PHP_FPM_STATUS_URL}</macro>
                    <value>http://127.0.0.1:10061/php-fpm_status</value>
                </macro>
            </macros>
            <templates/>
            <screens/>
        </template>
    </templates>
    <graphs>
        <graph>
            <name>php-fpm_status</name>
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
                    <color>C800C8</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template php-fpm</host>
                        <key>active.processes</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>1</sortorder>
                    <drawtype>0</drawtype>
                    <color>00C8C8</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template php-fpm</host>
                        <key>listen.queue</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>2</sortorder>
                    <drawtype>0</drawtype>
                    <color>C8C800</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template php-fpm</host>
                        <key>listen.queue.len</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>3</sortorder>
                    <drawtype>0</drawtype>
                    <color>C8C8C8</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template php-fpm</host>
                        <key>max.active.processes</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>4</sortorder>
                    <drawtype>0</drawtype>
                    <color>960000</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template php-fpm</host>
                        <key>idle.processe</key>
                    </item>
                </graph_item>
                <graph_item>
                    <sortorder>5</sortorder>
                    <drawtype>0</drawtype>
                    <color>000096</color>
                    <yaxisside>0</yaxisside>
                    <calc_fnc>2</calc_fnc>
                    <type>0</type>
                    <item>
                        <host>Template php-fpm</host>
                        <key>total.processes</key>
                    </item>
                </graph_item>
            </graph_items>
        </graph>
    </graphs>
</zabbix_export>


```


