---
title: bugzilla迁移需要修改的文件
date: 2017-11-28 15:36:40
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/78655716   
  ```
数据库授权：
grant all on bugs.* to root@localhost identified by "root"; #授权


flush privileges; # 刷新
------------------------------------------
/var/www/html/bugzilla/localconfig
$db_host = 'localhost'
$db_name = 'bugs'
$db_user = 'root’
$db_pass = 'root'
-----------------------------------------
/etc/httpd/conf/httpd.conf


<VirtualHost *:80>
     DocumentRoot /var/www/html/bugzilla/
</VirtualHost>
 
<Directory /var/www/html/bugzilla>
     AddHandler cgi-script .cgi
     Options +Indexes +ExecCGI
     DirectoryIndex index.cgi
     AllowOverride Limit FileInfo Indexes
</Directory>
-----------------------------------------
vi ./bugzilla/.htaccess


#Options –Indexes
-------------------------------


Redhat搭建bugzilla提示 Couldn't connect to smtp server




1.问题


最近搭建系统集成系统（jenkins），问题跟踪系统（bugzilla）配置SMTP发送邮件时，jenkins发送邮件功能OK，bugzilla发送邮件提示 Couldn't connect to smtp




2.解决


根据各种检索到的解决方案，均无答案，后来回到redhat网络连接本身上来，通过多次检索，发现redhat对于网络还存在其他的设定， httpd_can_network_connect属性默认为off，需要开启才能正常处理网络连接问题。执行如下语句后，bugzilla邮件发送功能即OK。


setsebool httpd_can_network_connect on


参考：https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Booleans-Configuring_Booleans.html



```
  
  
   
 