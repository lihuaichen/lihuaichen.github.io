---
title: 项目实训：搭建一台LAMP （Linux+Apache+Mysql+Php)架构的web服务器
date: 2016-09-07 22:17:15
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52463923   
  实验环境  
 网关 classroom 172.25.8.254  
 workstation 172.25.8.9   
 server a~j eth0 172.25.8.10- 外网  
 eth1 192.168.8.x 内网  
 eth2 192.168.8.x 备用  
 项目一 搭建apache服务器  
 需求：搭建一台LAMP （Linux+Apache+Mysql+Php)架构的服务器  
 --------------------------------------------------------------------------------------------------  
 设计：  
原理：1)webserver ：apache 进行请求的处理 使用http协议 监听80端口 能处理html静态页面，动态页面要转交给php处理  
网页分类 php jsp asp html  
 2)访问过程 client ---->www.xxx.com --->server   
apache对一个php页面请求会产生一个php进程 加载php模块 Load modules  
 3)php编译，编译的过程需要连接数据库 a.需要连接数据库的驱动 b.登陆认证的配置，mysql地址，用户名，密码，连接的数据库名  
 4）database产生库表 手动的在mydql中创建库 和对php进行授权  
硬件：Linux服务器  
系统：rhel7  
软件：httpd php php-mysql（驱动程序） mariadb-server  
服务：httpd mariadb  
 ---------------------------------------------------------------------------------------------------  
 部署：  
 yum install httpd php php-mysql mariadb-server -y #安装软件  
 配置虚拟主机  
 /etc/httpd/conf.d   
 vim www.abc.com.conf 新建配置文件  
 <VirtualHost *:80>  
ServerName www.abc.com  #虚拟主机名  
DocumentRoot /var/www/abc.com #虚拟主机目录  
 </VirtualHost>  
  
  
 mkdir /var/www/abc.com #创建网站目录  
 cd /etc/httpd/conf.modules.d/ 模块配置  
 cd /etc/httpd/modules/ 模块  
 xxx.conf   
 abc.com index.php页面  
 cd /var/www/abc.com  
 vim index.php #新建一个测试页面  
 <?php  
 phpinfo();  
 ?>  
 --------------------------------------------------  
 项目实施  
 -------------------------搭建Discuz论坛-------------  
 servera：  
 rpm -ql php-mysql # 检查连接数据库的驱动  
 /usr/lib64/php/modules/mysql.so #连接数据库驱动  
  
  
 rm -f /var/www/abc.com/index.php #删除测试页  
 mount 172.25.254.250:/content /mnt #挂载远程目录  
 cp /mnt/ula/Discuz_X2.5_SC_UTF8.zip /tmp #cp网站源文件  
 cd /tmp  
 unzip Discuz_X2.5_SC_UTF8.zip #解压缩  
 ls upload  
 cp -r upload/* /var/www/abc.com #复制网页文件到网站根目录  
 chown apache. /var/www/abc.com -R #对目录和文件修改权限  
 setenforce 0 #关闭selinux  
 vi /etc/selinux/config Permissive #修改配置文件更改selinux  
 systemctl start mariadb #启动数据库  
 mysql  
 >create database discuz; #创建discuz库  
 >grant all on discuz.* to discuz@'localhost' identified by 'uplooking'; #对php进行授权  
 -----------------------------------------------------  
 测试：  
 workstation：  
 vim /etc/hosts #修改hosts  
 172.25.8.10 www.abc.com  
 firefox  #打开浏览器www.abc.com进行网站初始化安装和测试  
 --------------------------------------------------------  
 问题解答  
 1）、访问出现Unable to connect  
 检查firewalld和selinux  
 2）、数据库找不到  
 检查是否安装了驱动  
  
  
  
   
 