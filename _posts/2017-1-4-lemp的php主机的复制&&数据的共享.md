---
title: lemp的php主机的复制&&数据的共享
date: 2016-09-25 20:40:01
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52664142   
  实验环境  
 网关 classroom 172.25.8.254  
 workstation 172.25.8.9   
 server a-j eth0 172.25.8.10- 外网  
 eth1 192.168.0.x 内网  
 eth2 192.168.1.x 备用  
 --------------------------------------------  
 需求：  
 设计：  
 原理：  
 硬件：  
 系统：  
 软件：  
 服务：  
 部署：  
 问题与解答：  
 php主机的复制  
 -------------------  
 servera:  
 scp /etc/sysconfig/spawn-fcgi 172.25.8.11:/etc/sysconfig/  
 tar czf /tmp/lemp.com.tgz /usr/share/nginx/lemp.com/  
 scp /tmp/lemp.com.tgz root@172.25.8.11:~  
 vim /etc/nginx/conf.d/www.lemp.com.conf   
 systemctl restart nginx.service  
 systemctl stop spawn-fcgi.service   
 vim /etc/nginx/nginx.conf  
 http {  
 include /etc/nginx/mime.types;  
 default_type application/octet-stream;  
 log_format main '$remote_addr - $remote_user [$time_local] "$request" '  
 '$status $body_bytes_sent "$http_referer" '  
 '"$http_user_agent" "$http_x_forwarded_for"';  
 access_log /var/log/nginx/access.log main;  
 sendfile on;  
 #tcp_nopush on;  
 keepalive_timeout 65;  
 #gzip on;  
 upstream php_pool {  
 server 172.25.8.11:9000;  
 server 172.25.8.12:9000；  
 }  
 vim /etc/nginx/conf.d/www.lemp.com.conf  
 server {  
 listen 80;  
 server_name www.lemp.com;  
 root /usr/share/nginx/lemp.com;  
 index index.php index.html index.htm;  
 location ~ \.php$ {  
 #fastcgi_pass 127.0.0.1:9000;  
 #fastcgi_pass 172.25.8.11:9000;  
 fastcgi_pass php_pool; #  
 fastcgi_index index.php;  
 fastcgi_param SCRIPT_FILENAME /usr/share/nginx/lemp.com$fastcgi_script_name;  
 include fastcgi_params;  
 }  
 }  
 --------------------------  
 serverb:  
 scp /etc/sysconfig/spawn-fcgi 172.25.8.12:/etc/sysconfig/  
 tar czf /tmp/lemp.com.tgz /usr//share/nginx/lemp.com/  
 scp /tmp/lemp.com.tgz root@172.25.8.12:~  
 -------------------  
 serverc:  
 tar xf lemp.com.tgz -C /  
 ll /usr/share/nginx/lemp.com/  
 groupadd nginx -g 994  
 useradd nginx -u 996 -g nginx  
 systemctl start spawn-fcgi.service   
 mount 172.25.254.250:/content /mnt  
 cd /mnt/ula/nginx/nginx-rpms  
 rpm -ivh spawn-fcgi-1.6.3-5.el7.x86_64.rpm  
 yum -y install php php-mysql  
 systemctl start spawn-fcgi.service  
 测试：  
 serverb：  
 cd /usr/share/nginx/lemp.com/  
 echo bbbb>test.php  
 serverc：  
 cd /usr/share/nginx/lemp.com/  
 echo cccc>test.php  
 -----  
 www.lemp.com/test.php  
 cccc和bbbb交替显示代表成功  
 访问  
 www.lemp.com   
 发帖上传文件  
 图片会传到其中一个php服务器，另一个php服务器则无法访问到图片  
 find ./lemp.com/data/* -name "*.gif"  
 -------------------------------  
 数据的共享  
 serverb:  
 scp /tmp/lemp.com.tgz root@172.25.8.19:~  
 tar czf /tmp/lemp.com.tgz /usr/share/nginx/lemp.com/  
 serverj:  
 yum -y install nfs-utils  
 tar -xf lemp.com.tgz -C / 从b中网站根目录全备份拷贝过来  
 vim /etc/exports  
 /usr/share/nginx/lemp.com 172.25.8.0/24(rw,async)  
 systemctl start nfs  
 serverb:  
 mount 172.25.8.19:/usr/share/nginx/lemp.com /usr/share/nginx/lemp.com  
 serverc:  
 mount 172.25.8.19:/usr/share/nginx/lemp.com /usr/share/nginx/lemp.com  
   
 