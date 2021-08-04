---
title: Linux-apache服务器
date: 2016-08-31 23:17:15
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390081   
  虚拟主机的操作步骤  
2.2版本  
vi /etc/httpd/conf/httpd.conf  
NameVirtualHost *:80  
vi /etc/httpd/conf.d/www.efg.com.conf  
<VirtualHost *:80>  
 ServerAdmin root@efg.com  
 DocumentRoot /var/www/html/efg.com  
 ServerName www.efg.com  
 ErrorLog logs/efg.com-error_log  
 CustomLog logs/efg.com-access_log common  
</VirtualHost>  
  
  
service httpd restart  
  
  
2.4版本  
vi /etc/httpd/conf.d/test.conf  
<VirtualHost *:80>  
 ServerAdmin root@test.com  
 DocumentRoot "/var/www/html/test.com"  
 ServerName www.test.com  
 ErrorLog "/var/log/httpd/test.com-error_log"  
 CustomLog "/var/log/httpd/test.com-access_log" common  
</VirtualHost>  
  
  
  
  
2.4版本的该配置字段可以从/usr/share/doc/httpd-2.4.6/httpd-vhosts.conf文件中找到。  
  
  
如果是通过错误的解析，或者是IP访问到主机，则将第一个匹配到的虚拟主机结果反馈给客户端  
  
  
  
  
  
  
  
  
  
  
  
  
-------  
  
  
单独开辟一个分区，用来存放网页文件。  
lvm --> 可在线扩展  
www.lvm.com  
el6  
 fdisk /dev/vdb  
pvcreate /dev/vdb1  
vgcreate vgweb /dev/vdb1  
lvcreate -n lvweb -L 300M vgweb  
mkfs.ext4 /dev/vgweb/lvweb  
mount /dev/vgweb/lvweb /lvm  
  
  
vim /etc/httpd/conf.d/www.lvm.com.conf  
<VirtualHost *:80>  
 ServerAdmin root@lvm.com  
 DocumentRoot /lvm  
 ServerName www.lvm.com  
 ErrorLog logs/lvm.com-error_log  
 CustomLog logs/lvm.com-access_log common  
</VirtualHost>  
semanage fcontext -a -t httpd_sys_content_t '/lvm(/.*)?'  
restorecon -F -R -v '/lvm'  
echo lvm > index.html  
  
  
el7  
<Directory "目录">  
 Require all granted -->允许所有人访问  
</Directory>  
<Directory "目录">  
 Require all denied -->不允许任何人访问  
</Directory>  
  
  
  
  
  
  
访问控制  
1.server   
el7 require all granted 允许所有人访问  
 require all denied 不允许任何人访问  
Require ip 172.25.0.11  
  
  
el6 order deny,allow -> 允许大于拒绝  
 deny from ip 网段  
 allow from ip 网段 all  
  
  
2.认证  
<Directory "/var/www/html/efg.com/doc">  
AllowOverride AuthConfig  
AuthName "info" # 说明信息  
AuthType basic   
AuthUserFile "/etc/httpd/.htpasswd" # 认证文件：用户名密码  
Require valid-user  
</Directory>  
  
  
htpasswd -cm /etc/httpd/.htpasswd carol  
chmod 400 /etc/httpd/.htpasswd   
chown apache /etc/httpd/.htpasswd  
  
  
  
  
Alias /download/ "/var/www/download/" -->别名的机制，当访问文件路径部分匹配到/download/ 则将/var/www/download/目录下的文件反馈给客户端  
  
  
  
  
<Directory "/var/www/download">  
 Options Indexes --> 如果对应目录下没有index.html ，则将文件名称罗列出来  
</Directory>  
  
  
<Directory "/var/www/download/carol">  
 Options -Indexes  
</Directory>  
   
 