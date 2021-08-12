---
title: MySQL max_connect_errors 报错
date: 2021-08-12 15:22:35
categories: Code
tags: MySQL
---
{% cq %} 
一次关于MySQL max_connect_errors报错的排查记录
{% endcq %}
<!-- more -->
## 现象
数据同步服务未正常工作，查看docker日志`docker logs --tail 100`输出报错信息
```LOG
Mysql2::Error: Host '10.xx.x.xxx' is blocked because of many connection errors;
```
分析为应用服务器多次连接数据库服务器失败，导致应用服务器ip被ban
## 临时解决方案
登录MySQL，使用`flush hosts`命令重置host的报错记录
修改`max_connect_errors`参数，设置一个较大值
```
show variables like 'max_connect_errors';
set global max_connect_errors = 1000;
```
重启应用服务后即可恢复
## 问题分析
参考文章[https://www.cnblogs.com/kerrycode/p/8405862.html](https://www.cnblogs.com/kerrycode/p/8405862.html)说明
MySQL会在performance_schema数据库下的host_cache表中记录客户端的访问情况，包括连接失败和密码登录失败等
```SQL
use performance_schema;
select * from host_cache where ip='10.xx.x.xxx' \G;
```
```SQL
Connection id:    390771
Current database: performance_schema

*************************** 1. row ***************************
                                        IP: 10.xx.x.xxx
                                      HOST: NULL
                            HOST_VALIDATED: YES
                        SUM_CONNECT_ERRORS: 134
                 COUNT_HOST_BLOCKED_ERRORS: 0
           COUNT_NAMEINFO_TRANSIENT_ERRORS: 0
           COUNT_NAMEINFO_PERMANENT_ERRORS: 1
                       COUNT_FORMAT_ERRORS: 0
           COUNT_ADDRINFO_TRANSIENT_ERRORS: 0
           COUNT_ADDRINFO_PERMANENT_ERRORS: 0
                       COUNT_FCRDNS_ERRORS: 0
                     COUNT_HOST_ACL_ERRORS: 0
               COUNT_NO_AUTH_PLUGIN_ERRORS: 0
                  COUNT_AUTH_PLUGIN_ERRORS: 0
                    COUNT_HANDSHAKE_ERRORS: 163
                   COUNT_PROXY_USER_ERRORS: 0
               COUNT_PROXY_USER_ACL_ERRORS: 0
               COUNT_AUTHENTICATION_ERRORS: 0
                          COUNT_SSL_ERRORS: 0
         COUNT_MAX_USER_CONNECTIONS_ERRORS: 0
COUNT_MAX_USER_CONNECTIONS_PER_HOUR_ERRORS: 0
             COUNT_DEFAULT_DATABASE_ERRORS: 0
                 COUNT_INIT_CONNECT_ERRORS: 0
                        COUNT_LOCAL_ERRORS: 0
                      COUNT_UNKNOWN_ERRORS: 0
                                FIRST_SEEN: 2021-08-12 15:04:22
                                 LAST_SEEN: 2021-08-12 15:43:13
                          FIRST_ERROR_SEEN: 2021-08-12 15:04:22
                           LAST_ERROR_SEEN: 2021-08-12 15:43:13
1 row in set (0,08 sec)

ERROR: 
No query specified
```
其中`SUM_CONNECT_ERRORS`的值超过`max_connect_errors`值时，则会导致文章开头报错

根据文章测试

登录时输入密码错误并不会导致ERRORS次数增加

而尝试TCP连接且连接最终未成功时，则会导致ERRORS增加
## 问题修复
服务器近期有启用端口扫描脚本，该脚本通过`nc`命令每隔10秒访问目标ip:端口以判断连通性

经验证该命令作用在MySQL对外服务端口上时，会导致`SUM_CONNECT_ERRORS`增加

其扫描原理是通过建立TCP连接来判断连通性，因此可增加参数`-u`建立UDP连接避免

或扫描22 ssh服务端口替代3306 MySQL服务端口即可解决问题
