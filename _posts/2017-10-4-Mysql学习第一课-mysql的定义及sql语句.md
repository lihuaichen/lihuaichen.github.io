---
title: Mysql学习第一课-mysql的定义及sql语句
date: 2016-08-31 23:26:46
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390153   
  20160829-mysqlday1  
1.mysql的定义  
1）数据库管理系统  
管理数据，数据库当中的最小单位表  
文本方式保存：读文件从头开始，速度非常慢  
数据库：索引，（图片，视频）  
-----  
1  
2  
  
  
5000000  
-----  
读帖子---400000  
  
  
2)c/s----》b/s  
客户端浏览器  
麻烦方便  
安全  
稳定  
  
  
3)开放源码  
GNU--》GPL   
  
  
4)关系型  
表和表之间支持一定的关系  
  
  
5)sql  
结构化查询语言  
  
  
6)多os  
linux、windows  
  
  
7)mysqlab-->sun-->oracle--->mysql  
--mariadb分支  
  
  
===================================  
2.存储引擎  
数据库的读写速度的快慢  
数据库读写数据的方式---存储引擎  
mysql---插件式的  
两大知名存储引擎---》表   
默认事务锁机制适用场景  
MYISAM mysql5.5之前非事务表锁线下分析  
INNODBmysql5.5开始事务行锁线上金钱类  
-----------------------------  
# 事务 ACID  
----------------  
表1--余额表  
1000010000-5000  
  
  
表2--已购物表  
+1手机  
----------------------------------  
原子性，  
begin;  
10000-5000  
+1  
commit;  
===============================================  
# 锁  
  
  
写锁和读锁  
锁精度：表锁和行锁  
写不能写  
写不能读  
读能读  
读不能写  
-----------------------  
myisam表  
-----  
superman  
batman  
  
  
-----  
batman 发帖----写锁---表  
superman发帖---不能  
superman读写---能  
---------------------  
batman 读贴----读锁---表  
superman读贴---能  
supermna写贴---不能  
=================================  
innodb表  
batman 发帖----写锁---行  
superman发帖---写锁---行  
superman读写---能  
==============================================================  
3.安装适用mysql（mariadb）  
OS  
mysql5.1rhel6.5  
mysql5.5mariadb5.5rhel7.2  
mysql5.6mariadb10.0  
mysql5.7mariadb10.1  
mariadb10.2  
  
  
软件名mariadb-server 5.5  
servicemariadb  
daemonmysqld  
配置文件 /etc/my.cnf  
/etc/my.cnf.d/*.cnf  
数据文件/var/lib/mysql  
日志文件/var/log/mariadb/mariadb.log（错误日志，启动日志）  
端口号3306  
  
  
==============================================================  
# 安装必要的软件包  
[root@mastera0 ~]# yum install -y vim net-tools wget  
  
  
[root@mastera0 ~]# yum list|grep mariadb  
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast  
mariadb-libs.x86_64 1:5.5.44-2.el7 @anaconda/7.2  
mariadb.x86_64 1:5.5.44-2.el7 rhel_dvd   
mariadb-bench.x86_64 1:5.5.44-2.el7 rhel_dvd   
mariadb-devel.i686 1:5.5.44-2.el7 rhel_dvd   
mariadb-devel.x86_64 1:5.5.44-2.el7 rhel_dvd   
mariadb-libs.i686 1:5.5.44-2.el7 rhel_dvd   
mariadb-server.x86_64 1:5.5.44-2.el7 rhel_dvd   
mariadb-test.x86_64 1:5.5.44-2.el7 rhel_dvd   
[root@mastera0 ~]# yum install -y mariadb-server  
# 查看软件架构  
[root@mastera0 ~]# rpm -ql mariadb-server  
# 启动服务  
[root@mastera0 ~]# systemctl start mariadb  
# 查看守护进程  
[root@mastera0 ~]# ps -ef|grep mysqld  
mysql 2496 1 0 13:56 ? 00:00:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr  
mysql 2653 2496 0 13:56 ? 00:00:00 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock  
root 2694 2347 0 13:56 pts/0 00:00:00 grep --color=auto mysqld  
# 查看监听端口号  
[root@mastera0 ~]# netstat -luntp|grep mysqld  
tcp 0 0 0.0.0.0:3306 0.0.0.0:* LISTEN 2653/mysqld   
  
  
===========================================  
4.客户端  
  
  
软件名称mariadb 5.5  
命令mysql 登陆连接mysql服务器  
mysqladmin修改数据库服务器用户密码  
mysqldump备份  
mysqlbinlog二进制日志的查看  
  
-------------------------------------------  
命令的使用  
mysql  
1.服务启动后，mariadb5.5直接登陆，不需要密码  
2.退出 \q exit ctrl+d  
  
mysqladmin -uroot password 'uplooking'  
-u 用户名 空格可有可无 -u root;-uroot  
password 新密码一定要有空格  
  
  
mysql -uroot -puplooking  
-u 用户名 空格可有可无 -u root;-uroot  
-p 密码 不可以有空格 -puplooking  
  
  
mysqladmin -uroot -puplooking password 'uplooking123'  
-u 用户名可有可无  
-p 当前密码不能有  
password 新密码有  
  
  
mysql -uroot -puplooking123  
----------------------------------------------  
SQL语句  
* sql命令不区分大小，数据库，表，数据是区分大小写的  
* sql语句以分号提交  
  
  
# 分类  
DDL数据库定义语言create\drop\alter  
  
create database [dbname];  
drop database [dbname];  
create table [tbname] (col1 type,col2 type,col3....);  
drop table [tbname];  
  
DML数据库操作语言insert into\delete from\update  
insert into [tbname] set col1=value,col2=value,col3=value;  
insert into [tbname] values (1,'booboo'),(2,'batman'),(3,'superman');  
insert into [tbname] (name,id) values ();  
delete from [tbname] where id=1 and name='booboo';  
update [tbname] set col='superman' where id=2;  
  
DCL数据库控制语言grant revoke  
认证 + 授权  
grant all on *.* to booboo@'172.25.0.11' identified by 'uplooking';  
all 权限   
*.* 库.表  
flush privileges; 刷新授权表  
revoke [权限] on [库].[表] from booboo@'172.25.0.11';  
  
  
DQL数据库查询语言show databases;  
use mysql;  
show tables;  
desc mysql.user;  
select * from db1.t1;  
  
  
-------------------------------------------------  
课堂作业：  
1.查看mysql库中user表中的host,user,password列的值；  
2.删除mysql库中的user表中，user列为空或者password列为空的行；  
  
  
MariaDB [mysql]> select host,user,password from mysql.user;  
+----------------------+------+-------------------------------------------+  
| host | user | password |  
+----------------------+------+-------------------------------------------+  
| localhost | root | *6FF883623B8639D08083FF411D20E6856EB7D2BF |  
| mastera0.example.com | root | |  
| 127.0.0.1 | root | |  
| ::1 | root | |  
| localhost | | |  
| mastera0.example.com | | |  
+----------------------+------+-------------------------------------------+  
6 rows in set (0.00 sec)  
  
  
MariaDB [mysql]> delete from mysql.user where user=' ' or password=' ';  
Query OK, 5 rows affected (0.00 sec)  
  
  
MariaDB [mysql]> select host,user,password from mysql.user;  
+-----------+------+-------------------------------------------+  
| host | user | password |  
+-----------+------+-------------------------------------------+  
| localhost | root | *6FF883623B8639D08083FF411D20E6856EB7D2BF |  
+-----------+------+-------------------------------------------+  
1 row in set (0.00 sec)  
=====================================================================================  
实验2：添加授权用户  
实验要求:添加授权用户测试客户机优先使用的哪个密码  
user1@'172.25.X.%' uplooking  
user1@'172.25.X.12' uplooking123  
  
  
172.25.X.11 mastera  
  
  
masterb去登陆连接mastera的mariadb-server：  
  
  
1.install mariadb  
2.friewalld  
3.mysql -u -p -h   
  
  
======================================================  
如何破解mariadb5.5的root密码  
1.停止服务 systemctl stop mariadb  
2.跳过授权表启动服务 mysqld_safe --skip-grant-tables &   
 查看服务名称的方法：(ps -ef|grep mysqld;rpm -ql mariadb)  
3.mysql-->update mysql.user set password=password('uplooking') where user='root';   
4.停止跳过授权表启动服务 kill -9  
5.启动服务 systemctl start mariadb  
===============================  
  
  
晚自习作业：  
1.熟悉sql语句，bookshop  
2.查看授权表  
3.添加授权用户  
4.破解root密码  
  
  
-------------------------------------------------------  
   
 