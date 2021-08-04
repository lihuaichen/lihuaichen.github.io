---
title: lemp(lnmp)web网站搭建
date: 2016-09-25 20:38:30
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52664129   
  实验环境  
 网关 classroom 172.25.8.254  
 workstation 172.25.8.9   
 server a-j eth0 172.25.8.10- 外网  
 eth1 192.168.0.x 内网  
 eth2 192.168.1.x 备用  
 --------------------------------------------  
 需求：  
 设计：  
 原理：用户在访问过程中如果访问动态页面,apache 和 nginx 都解释不了 php 页面,需要调用相应的 php 进程,apache 在调用过  
 程中使用的 libphp5.so 的模块,因为 apache 的配置文件中加载模块相应配置,nginx 没有,默认不能加载模块,所以需要安  
 装额外程序,php 进程管理器。Php 进程管理器比较常见的有 spawn-fcgi 和 php-fpm。  
 Spawn-cfgi 和 php-fpm 相比,后者性能更高,但后者必须和 php 程序版本完全一致,如果升级了 php,那么 php-fpm 程序  
 也要做相应的版本升级。  
 本例中使用的 spawn-fcgi 程序。  
 Php 进程管理器在 lnmp 环境中的作用:  
 (1)监听端口,nginx 把请求交给 php 管理器,php 管理器监听 9000 端口  
 (2)调用和管理 php 进程,管理程序去看你本地有没有 php 命令,有的话调用起来,php 命令运行之后再去处理刚才收到的  
 页面请求,处理 php 请求  
 fpm包不能独立编译，是在php编译的时候加入一些参数，是编译后产生两个rpm包，其中一个就是php-fpm  
 --------------------------------------  
 拆分  
 1）换性能更强的主机，升级后提升的性能和升级的成本是不成正比的  
 2）使用更多的主机  
 扩展的原则是使用更多的服务器  
 一台nginx 一组php服务器 一组db服务器  
 拆分（不同的服务运行在不同的服务器上，更高架构等级的是对应用进行拆分面向的是数千台及以上的服务器），  
 复制（性能不够时，用更多的服务器去跑相同的服务）  
 程序的迁移遵循的5个步骤：  
 1）程序，不同主机要相同的软件（mysql和mariadb就不能通用）  
 2）配置文件，配置的内容要相同（监听相同的端口等等）  
 3）数据，相同的数据库文件  
 4）指向，把用户请求指向到新配置的主机  
 5）其他 （用户权限问题）  
 硬件：linux主机  
 系统：rhel7  
 软件：nginx mariadb-server php spawn-fcgi-1.6.3-5.el7.x86_64.rpm  
 服务：spawn-fcgi.service nginx mariadb   
 部署：  
 安装spawn-fcgi进行处理php页面  
 cd /etc/nginx/conf.d  
 cp default.conf www.lemp.com.conf  
 vim www.lemp.com.conf   
 ---------------------------------  
 server {  
 listen 80;  
 server_name www.lemp.com; #虚拟主机名  
 #charset koi8-r;  
 #access_log /var/log/nginx/log/host.access.log main;  
 root /usr/share/nginx/lemp.com;  
 index index.php index.html index.htm;  
 #error_page 404 /404.html;  
 # redirect server error pages to the static page /50x.html  
 #  
 error_page 500 502 503 504 /50x.html;  
 location = /50x.html {  
 root /usr/share/nginx/html;  
 }  
 # proxy the PHP scripts to Apache listening on 127.0.0.1:80  
 #  
 #location ~ \.php$ {  
 # proxy_pass http://127.0.0.1;  
 #}  
 # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000  
 #  
 #location ~ \.php$ {  
 # root html;  
 # fastcgi_pass 127.0.0.1:9000;  
 # fastcgi_index index.php;  
 # fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;  
 # include fastcgi_params;  
 #}  
 location ~ \.php$ {  
 fastcgi_pass 127.0.0.1:9000; #指定spawn-fcgi主机地址和监听的端口号  
 fastcgi_index index.php;  
 fastcgi_param SCRIPT_FILENAME /usr/share/nginx/lemp.com$fastcgi_script_name; #指定网站的根目录  
 include fastcgi_params;   
 }   
 # deny access to .htaccess files, if Apache's document root  
 # concurs with nginx's one  
 #  
 #location ~ /\.ht {  
 # deny all;  
 #}  
 }  
 -----------------------------------------  
 mkdir /usr/share/nginx/lemp.com  
 mount 172.25.254.250:/content /mnt  
 cd /mnt/ula/nginx/nginx-rpms/   
 yum install spawn-fcgi-1.6.3-5.el7.x86_64.rpm #安装spawn-fcgi  
 cd /etc/sysconfig/  
 vim spawn-fcgi #修改spawn-fcgi配置文件  
 -------------------------------------------------------------------------  
 # You must set some working options before the "spawn-fcgi" service will work.  
 # If SOCKET points to a file, then this file is cleaned up by the init script.  
 #  
 # See spawn-fcgi(1) for all possible options.  
 #  
 # Example :  
 #SOCKET=/var/run/php-fcgi.sock  
 #OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"  
 OPTIONS="-u nginx -g nginx -p 9000 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi" #指定用户和组，-p监听端口号   
 --------------------------------------------------------------------------  
 yum install -y php #安装php  
 [root@servera sysconfig]# ls /usr/bin/php-cgi  
 /usr/bin/php-cgi  
 systemctl restart nginx #重启nginx服务  
 netstat -ntalp |grep 9000  
 systemctl restart spawn-fcgi.service #重启spawn-fcgi服务  
 ----------------  
 数据库：  
 systemctl start mariadb  
 MariaDB [(none)]> create database lemp;  
 MariaDB [(none)]> grant all on lemp.* to lemp@'172.25.8.10' identified by 'uplooking';  
 网站文件：  
 cd /mnt/ula/  
 cp Discuz_X2.5_SC_UTF8.zip /tmp  
 cd /tmp  
 unzip Discuz_X2.5_SC_UTF8.zip  
 cp -r upload/* /usr/share/nginx/lemp.com/  
 chown nginx. /usr/share/nginx/lemp.com/ -R  
 setenforce 0  
  
 -----------  
 测试：  
 ==========================================================================  
 程序的迁移遵循的5个步骤：  
 1）程序，不同主机要相同的软件（mysql和mariadb就不能通用）  
 2）配置文件，配置的内容要相同（监听相同的端口等等）  
 3）数据，相同的数据库文件,为保证数据的一致性，需要将原数据库中的数据导入到新的数据库服务器中  
 4）指向，把用户请求指向到新配置的主机  
 5）其他 （用户权限问题）  
 ---------------------------------  
 数据的迁移  
 数据库的导入导出：  
 旧mysqldump新mydql  
 php的页面代码指定连接数据库  
 db授权  
 ：安装mariadb  
 拷贝/etc/my.cnf  
 servera：只提供nginx服务器，  
 serverj：只负责数据库  
 ----------------------------------  
 servera：  
 mysqldump --all-database --single-transaction> /tmp/mysql.all.sql #将数据导出  
 scp mysql.all.sql root@172.25.8.19:/tmp #将备份的数据cp到数据库服务器serverj  
 scp /etc/my.cnf root@172.25.8.19:/etc #如果修改过mysql的主配置文件则需要将配置文件也要导入  
 cd /usr/share/nginx/lemp.com/ #修改网页数据文件中将localhost更改为数据库服务器的地址  
 [root@servera lemp.com]# for i in $(find ./ -name "*.php");do grep -q uplooking $i && echo $i; done  
 ./config/config_global.php  
 ./config/config_ucenter.php  
 ./uc_server/data/config.inc.php  
 [root@servera lemp.com]# vim ./config/config_global.php  
 [root@servera lemp.com]# vim ./config/config_ucenter.php  
 [root@servera lemp.com]# vim ./uc_server/data/config.inc.php  
 修改上述3个php文件，将文件中的localhost修改为mysql所在服务器的ip地址  
 [root@servera lemp.com]# systemctl stop mariadb #servera关闭mysqldb服务  
 --------------------------------------------------------------------  
 serverj：  
 systemctl start mariadb  
 mysql </tmp/mysql.all.sql #导入全备份数据  
 MariaDB [(none)]> drop user lemp@localhost; 取消原来对localhost的授权  
 MariaDB [(none)]> grant all on lemp.* to lemp@'172.25.8.10' identified by 'uplooking'; #增加新的授权  
 MariaDB [(none)]> flush privileges; #刷新授权  
 程序的迁移：  
 servera：只提供nginx服务器，  
 serverb：只负责处理php页面，安装spawn-fcgi，  
 serverj：只负责数据库  
 ------------------------------------  
 serverb:  
 mount 172.25.254.250:/content /mnt  
 cd /mnt/ula/nginx/nginx-rpms  
 rpm -ivh spawn-fcgi-1.6.3-5.el7.x86_64.rpm #安装spawn-fcgi  
 yum -y install php php-mysql #安装php php-mysql  
 servera:  
 scp /etc/sysconfig/spawn-fcgi 172.25.8.11:/etc/sysconfig/ #拷贝spawn-fcgi的配置文件  
 tar czf /tmp/lemp.com.tgz /usr/share/nginx/lemp.com/ #将网站文件通过tar打包压缩备份  
 scp /tmp/lemp.com.tgz root@172.25.8.11:~ #拷贝tar包  
 vim /etc/nginx/conf.d/www.lemp.com.conf #修改虚拟主机配置将fastcgi_pass 指向spawn-fcgi主机  
 location ~ \.php$ {  
 # fastcgi_pass 127.0.0.1:9000;  
 fastcgi_pass 172.25.8.11:9000;   
 fastcgi_index index.php;  
 fastcgi_param SCRIPT_FILENAME /usr/share/nginx/lemp.com$fastcgi_script_name;  
 include fastcgi_params;  
 }  
 systemctl restart nginx.service #重启服务  
 systemctl stop spawn-fcgi.service #关闭spawn-fcgi服务  
 serverb:  
 [root@serverb ~]# tar xf lemp.com.tgz -C / #解压tar包  
 ll /usr/share/nginx/lemp.com/ #显示的是uid和gid，所以要创建nginx用户和组  
 groupadd nginx -g 994  
 useradd nginx -u 996 -g nginx  
 systemctl start spawn-fcgi.service #启动spawn-fcgi服务  
 iptables -F #清空iptables  
 setenforce 0 #关闭selinux  
 serverj:  
 MariaDB [(none)]> drop user lemp@172.25.8.10;  
 MariaDB [(none)]> grant all on lemp.* to lemp@'172.25.8.11' identified by 'uplooking';  
 MariaDB [(none)]> grant all on lemp.* to lemp@'serverb.pod8.example.com' identified by 'uplooking';  
 #清空原来的授权，增加新的授权  
 授权时默认先对ip地址进行授权，有时也要对ip地址和hostname同时授权  
  
  
  
  
 问题与解答：  
 1)网页显示：  
 An error occurred.  
 Sorry, the page you are looking for is currently unavailable.  
 Please try again later.  
 If you are the system administrator of this resource then you should check the error log for details.  
 Faithfully yours, nginx.  
 原因：没有启动spawn-fcgi.service或者防火墙和selinux  
 systemctl start spawn-fcgi.service   
 2）网页显示  
 Discuz! Database Error  
 (2003) notconnect  
 原因：数据库服务没有启动或没有对spawn-fcgi服务器进行授权  
  


   
 