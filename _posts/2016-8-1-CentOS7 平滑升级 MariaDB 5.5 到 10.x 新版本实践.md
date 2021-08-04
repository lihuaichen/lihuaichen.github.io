---
title: CentOS7 平滑升级 MariaDB 5.5 到 10.x 新版本实践
date: 2018-12-12 14:44:25
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/84972757   
  ## 前言

 自从 CentOS 7 开始，自带的数据库就变成 MariaDB 了，yum 安装之后的默认版本是 5.5，但是这个数据版本已经比较老了，无论是安装全新的Percona还是升级MariaDB第一步始终是不要忘记备份。

 
> CentOS7平滑升级MariaDB 5.5到10.x新版本实践
> 
>  
 
## 更新历史

 2018年11月14日 - 初稿

 阅读原文 - [https://wsgzao.github.io/post/mariadb-upgrade/](https://wsgzao.github.io/post/mariadb-upgrade/)

 **扩展阅读**

 MariaDB - [https://mariadb.org/](https://mariadb.org/)

 
--------

## 备份数据库

 
> 重要的事情说三遍，备份，备份，备份
> 
>  
 
```
 # 备份数据库，如果升级顺利是不要实施备份还原的
mysqldump -u root -p --all-databases > alldb.sql
# 如果想保留自己的my.cof配置，则备份一下这个文件
cp /etc/my.cnf /etc/my.cnf.bak
# 停止数据库运行
systemctl stop mariadb
# 卸载MariaDB老版本
yum remove mariadb mariadb-server


```
 
## 添加 MariaDB Yum 库

 
> 建议使用MariaDB官方推荐的stable稳定版，
> 
>  
 [https://downloads.mariadb.org/mariadb/](https://downloads.mariadb.org/mariadb/)  
[http://yum.mariadb.org/](http://yum.mariadb.org/)

 
```
 # 添加MariaDB官方源
vi /etc/yum.repos.d/MariaDB.repo

# MariaDB 10.3 CentOS repository list
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

# 清除yum缓存
yum clean all 
yum makecache


```
 
## 升级已有数据库

 
```
 # 安装MariaDB新版本
yum install mariadb mariadb-server

# 启动新版MariaDB
systemctl start mariadb

# 升级已有数据库
mysql_upgrade -uroot -p 

[root@sg-gop-10-71-12-89 tmp]# mysql_upgrade -uroot -p
Enter password:
Phase 1/7: Checking and upgrading mysql database
Processing databases
mysql
mysql.columns_priv                                 OK
mysql.db                                           OK
mysql.event                                        OK
mysql.func                                         OK
mysql.help_category                                OK
mysql.help_keyword                                 OK
mysql.help_relation                                OK
mysql.help_topic                                   OK
mysql.host                                         OK
mysql.ndb_binlog_index                             OK
mysql.plugin                                       OK
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.proxies_priv                                 OK
mysql.servers                                      OK
mysql.tables_priv                                  OK
mysql.time_zone                                    OK
mysql.time_zone_leap_second                        OK
mysql.time_zone_name                               OK
mysql.time_zone_transition                         OK
mysql.time_zone_transition_type                    OK
mysql.user                                         OK
Upgrading from a version before MariaDB-10.1
Phase 2/7: Installing used storage engines
Checking for tables with unknown storage engine
Phase 3/7: Fixing views
Phase 4/7: Running 'mysql_fix_privilege_tables'
Phase 5/7: Fixing table and database names
Phase 6/7: Checking and upgrading tables
Processing databases
information_schema
performance_schema
zabbix
zabbix.acknowledges                                OK
zabbix.actions                                     OK
zabbix.alerts                                      OK
zabbix.application_discovery                       OK
zabbix.application_prototype                       OK
zabbix.application_template                        OK
zabbix.applications                                OK
zabbix.auditlog                                    OK
zabbix.auditlog_details                            OK
zabbix.autoreg_host                                OK
zabbix.conditions                                  OK
zabbix.config                                      OK
zabbix.corr_condition                              OK
zabbix.corr_condition_group                        OK
zabbix.corr_condition_tag                          OK
zabbix.corr_condition_tagpair                      OK
zabbix.corr_condition_tagvalue                     OK
zabbix.corr_operation                              OK
zabbix.correlation                                 OK
zabbix.dashboard                                   OK
zabbix.dashboard_user                              OK
zabbix.dashboard_usrgrp                            OK
zabbix.dbversion                                   OK
zabbix.dchecks                                     OK
zabbix.dhosts                                      OK
zabbix.drules                                      OK
zabbix.dservices                                   OK
zabbix.escalations                                 OK
zabbix.event_recovery                              OK
zabbix.event_suppress                              OK
zabbix.event_tag                                   OK
zabbix.events                                      OK
zabbix.expressions                                 OK
zabbix.functions                                   OK
zabbix.globalmacro                                 OK
zabbix.globalvars                                  OK
zabbix.graph_discovery                             OK
zabbix.graph_theme                                 OK
zabbix.graphs                                      OK
zabbix.graphs_items                                OK
zabbix.group_discovery                             OK
zabbix.group_prototype                             OK
zabbix.history                                     OK
zabbix.history_log                                 OK
zabbix.history_str                                 OK
zabbix.history_text                                OK
zabbix.history_uint                                OK
zabbix.host_discovery                              OK
zabbix.host_inventory                              OK
zabbix.hostmacro                                   OK
zabbix.hosts                                       OK
zabbix.hosts_groups                                OK
zabbix.hosts_templates                             OK
zabbix.housekeeper                                 OK
zabbix.hstgrp                                      OK
zabbix.httpstep                                    OK
zabbix.httpstep_field                              OK
zabbix.httpstepitem                                OK
zabbix.httptest                                    OK
zabbix.httptest_field                              OK
zabbix.httptestitem                                OK
zabbix.icon_map                                    OK
zabbix.icon_mapping                                OK
zabbix.ids                                         OK
zabbix.images                                      OK
zabbix.interface                                   OK
zabbix.interface_discovery                         OK
zabbix.item_application_prototype                  OK
zabbix.item_condition                              OK
zabbix.item_discovery                              OK
zabbix.item_preproc                                OK
zabbix.items                                       OK
zabbix.items_applications                          OK
zabbix.maintenance_tag                             OK
zabbix.maintenances                                OK
zabbix.maintenances_groups                         OK
zabbix.maintenances_hosts                          OK
zabbix.maintenances_windows                        OK
zabbix.mappings                                    OK
zabbix.media                                       OK
zabbix.media_type                                  OK
zabbix.opcommand                                   OK
zabbix.opcommand_grp                               OK
zabbix.opcommand_hst                               OK
zabbix.opconditions                                OK
zabbix.operations                                  OK
zabbix.opgroup                                     OK
zabbix.opinventory                                 OK
zabbix.opmessage                                   OK
zabbix.opmessage_grp                               OK
zabbix.opmessage_usr                               OK
zabbix.optemplate                                  OK
zabbix.problem                                     OK
zabbix.problem_tag                                 OK
zabbix.profiles                                    OK
zabbix.proxy_autoreg_host                          OK
zabbix.proxy_dhistory                              OK
zabbix.proxy_history                               OK
zabbix.regexps                                     OK
zabbix.rights                                      OK
zabbix.screen_user                                 OK
zabbix.screen_usrgrp                               OK
zabbix.screens                                     OK
zabbix.screens_items                               OK
zabbix.scripts                                     OK
zabbix.service_alarms                              OK
zabbix.services                                    OK
zabbix.services_links                              OK
zabbix.services_times                              OK
zabbix.sessions                                    OK
zabbix.slides                                      OK
zabbix.slideshow_user                              OK
zabbix.slideshow_usrgrp                            OK
zabbix.slideshows                                  OK
zabbix.sysmap_element_trigger                      OK
zabbix.sysmap_element_url                          OK
zabbix.sysmap_shape                                OK
zabbix.sysmap_url                                  OK
zabbix.sysmap_user                                 OK
zabbix.sysmap_usrgrp                               OK
zabbix.sysmaps                                     OK
zabbix.sysmaps_elements                            OK
zabbix.sysmaps_link_triggers                       OK
zabbix.sysmaps_links                               OK
zabbix.tag_filter                                  OK
zabbix.task                                        OK
zabbix.task_acknowledge                            OK
zabbix.task_check_now                              OK
zabbix.task_close_problem                          OK
zabbix.task_remote_command                         OK
zabbix.task_remote_command_result                  OK
zabbix.timeperiods                                 OK
zabbix.trends                                      OK
zabbix.trends_uint                                 OK
zabbix.trigger_depends                             OK
zabbix.trigger_discovery                           OK
zabbix.trigger_tag                                 OK
zabbix.triggers                                    OK
zabbix.users                                       OK
zabbix.users_groups                                OK
zabbix.usrgrp                                      OK
zabbix.valuemaps                                   OK
zabbix.widget                                      OK
zabbix.widget_field                                OK
Phase 7/7: Running 'FLUSH PRIVILEGES'
OK

# 配置服务自启动
systemctl enable mariadb

# 登录数据库验证
mysql -uroot -p

# 升级过程遇到错误记得先查看日志分析



```
 
## MariaDB官方升级文档

 Upgrading from MariaDB 5.5 to MariaDB 10.0  
[https://mariadb.com/kb/en/library/upgrading-from-mariadb-55-to-mariadb-100/](https://mariadb.com/kb/en/library/upgrading-from-mariadb-55-to-mariadb-100/)

 

   
 