---
title: Mysql学习第二课 tar、lvm、mysqldump全备份
date: 2016-08-31 23:27:20
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390157   
  20160830mysqlday2  
# 复习  
1.什么是mysql？  
1） 2） 3） 4） 5） 6） 7）  
2.存储引擎的概念  
3.知名的两大存储引擎MYISAM和INNODB的区别  
事务锁机制适用场景  
MYISAM  
INNODB  
4.红帽企业版rhel7.2默认的数据库软件版本为？  
5.mariadb5.5的服务名service为？ 守护进程名daemon为？默认监听的端口号为？  
6.mariadb5.5的rpm包安装后默认的数据文件存放路径为？ 启动日志（排错日志）存放的目录为？  
7.sql语句的分类：1） 2） 3） 4）  
8.如何破解mariadb5.5的root密码？  
9.判断题：先授权越优先？  
10.判断题：授权越精确越优先？  
11.通过客户端程序连接登陆mariadb服务器，客户端软件名为？登陆的命令为？  
12.第一次修改数据库root密码的命令为  
----------------------------------------  
同学记录：34  
========================================================================================  
备份  
1.分类：冷备、热备、异地灾备  
2.冷备：将数据以隔离的方式保存，备份数据不受源数据的影响。  
优点：1）硬件故障 2）人为误操作  
缺点：还原（恢复）速度太慢   
3.热备：冗余 多余的重复的 人为地增加重复的部分  
冗余环境：应用程序---》主服务器 从服务器  
优点：还原（恢复）速度快，能够瞬间还原  
缺点：只能解决硬件故障，无法解决人为误操作  
4.在线上生产环境里，冷备和热备同时使用。  
5.mysql数据库中冷备的方法  
1)冷备的分类：按照备份的数据来分类  
完全备份  
增量备份  
2)冷备的分类：备份的内容的分类  
物理备份 :备份的是真的数据  
逻辑备份 :备份的是sql语句  
create database db1;------>/var/lib/mysql/db1  
create table db1.t1 (id int);/var/lib/mysql/db1/t1.frm  
/var/lib/mysql/db1/t1.MYD  
/var/lib/mysql/db1/t1.MYI  
insert into db1.t1 value (1);/var/lib/mysql/db1/t1.MYD---1  
  
  
6.完全备份  
1)物理备份：cp tar lvm快照方式  
2)逻辑备份：mysqldump  
  
  
7.备份中两大要素  
数据一致性：从备份开始的时间点，你的备份数据就已经确定了。  
服务可用性：数据库是否可以读写，既能读也能写才是服务可用。  
===================================  
# tar备份和还原  
## tar备份  
  
  
1）停止服务 systemctl stop mariadb  
2）备份数据 tar -cf /tmp/mysql.all.tar /var/lib/mysql   
3）启动服务 systemctl start mariadb  
  
  
## tar还原  
  
1）停止服务 systemctl stop mariadb  
2）清环境 rm -rf /var/lib/mysql/*  
3）导入数据 tar -xf /tmp/mysql.all.tar -C /  
4）启动服务 systemctl start mariadb  
5）测试 > select * from db1.t1;   
  
  
# lvm snapshot 快照方式备份和还原  
## lvm快照的优点  
1.备份速度快  
2.不区分存储引擎  
## 缺点  
必须支持lvm  
  
/var/lib/mysql/--->lv ---->snapshot  
/dev/vdb---fdisk---/dev/vdb1 1G---->mkfs.ext4--->目录树  
/dev/vdb---fdisk---/dev/vdb1 1G--pv--vg--lv--->mkfs.ext4--->目录树  
  
  
## os支持lvm方式 lv1---/var/lib/mysql  
1)fdisk   
2)pv  
3)vg  
4)lv  
5)mkfs ext4 (xfs)  
6)停止服务  
7)全备份tar  
8)挂接/var/lib/mysql/  
9)导入数据(注意权限，ugo，selinux)  
10)启动服务  
  
  
## lvm快照备份数据  
### 与数据库服务相关的操作  
1)添加全局的读锁（只能读不能写---》数据不会变）> flush tables with read lock;  
2)创建快照lvcreate -s -L 1G -n snap1 /dev/vgmysql/lv1 --->/dev/vgmysql/snap1  
3)解锁> unlock tables;  
  
  
### 与数据库服务无关的操作  
4)挂接快照mount /dev/vgmysql/snap1 /mnt (如果是xfs，mount -o nouuid /dev/vgmysql/snap1 /mnt )  
5)tar打包cd /mnt;tar -cf /tmp/mysql.2.tar ./*  
6)umountumount /mnt  
7)删除快照lvremove /dev/vgmysql/snap1  
  
  
------------------------------------------------------------  
## lvm快照还原数据  
1）停止服务 systemctl stop mariadb  
2）清环境 rm -rf /var/lib/mysql/*  
3）导入数据 tar -xf /tmp/mysql.2.tar -C /var/lib/mysql  
4）启动服务 systemctl start mariadb  
5）测试 > select * from db1.t1;   
  
  
  
  
  
  
# mysqldump备份和还原  
数据一致服务可用  
MYISAMokno锁表  
INNODBokokMVCC版本控制机制  
----------------------------------------------------  
MVCC版本控制机制  
不重复，一次递增1 2 3 4 。。。  
----------------------------------------------------  
mysqldump 备份数据---逻辑备份sql语句  
-u 用户名  
-p 密码  
-A 所有的库   
--single-transaction INNODB存储引擎的表备份时能够做到数据一致，服务可用  
mysqldump -uroot -puplooking -A --single-transaction > /tmp/mysql.201608301600.sql  
--lock-all-tables MYISAM存储引擎的表备份时能够做到数据一致，服务不可用  
mysqldump -uroot -puplooking -A --lock-all-tables > /tmp/mysql.xxx.sql  
  
  
===============================  
maraidb-server 5.5 INNODB (mysql)  
# mysqldump备份步骤  
INNODBmysqldump -uroot -puplooking -A --single-transaction > /tmp/mysql.201608301600.sql  
MYISAMmysqldump -uroot -puplooking -A --lock-all-tables > /tmp/mysql.xxx.sql  
  
# mysqldump还原步骤  
1）停止服务  
2）清空环境  
3）启动服务（启动之后，/var/lib/mysql/目录下会产生初始化的文件）  
4）导入数据  
5）刷新授权  
6）测试  
==========================================================  
============================  
tar详细步骤：  
# 备份  
[root@mastera0 ~]# systemctl stop mariadb  
[root@mastera0 ~]# tar -cf /tmp/mysql.all.tar /var/lib/mysql/  
tar: Removing leading `/' from member names  
[root@mastera0 ~]# systemctl start mariadb  
[root@mastera0 ~]# ll /tmp  
total 29772  
-rw-r--r--. 1 root root 30484480 Aug 30 11:28 mysql.all.tar  
  
  
  
  
# 还原  
[root@mastera0 ~]# systemctl stop mariadb  
ot@mastera0 ~]# rm -rf /var/lib/mysql/*  
[root@mastera0 ~]# ll /var/lib/mysql  
total 0  
[root@mastera0 ~]# tar -xf /tmp/mysql.all.tar -C /  
[root@mastera0 ~]# ll /var/lib/mysql  
total 28700  
-rw-rw----. 1 mysql mysql 16384 Aug 30 11:27 aria_log.00000001  
-rw-rw----. 1 mysql mysql 52 Aug 30 11:27 aria_log_control  
drwx------. 2 mysql mysql 32 Aug 30 11:24 db1  
-rw-rw----. 1 mysql mysql 18874368 Aug 30 11:27 ibdata1  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 11:27 ib_logfile0  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 11:23 ib_logfile1  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 mysql  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 performance_schema  
drwx------. 2 mysql mysql 6 Aug 30 11:23 test  
[root@mastera0 ~]# systemctl start mariadb  
[root@mastera0 ~]# echo "select * from db1.t1"|mysql -uroot -puplooking  
id  
1  
2  
  
  
============================  
lvm snapshot详细步骤：  
# lv1-->/var/lib/mysql  
[root@mastera0 mysql]# fdisk -l  
  
  
Disk /dev/vda: 10.7 GB, 10737418240 bytes, 20971520 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
Disk label type: dos  
Disk identifier: 0x000deb17  
  
  
 Device Boot Start End Blocks Id System  
/dev/vda1 * 2048 411647 204800 83 Linux  
/dev/vda2 411648 20971519 10279936 8e Linux LVM  
  
  
Disk /dev/vdb: 21.5 GB, 21474836480 bytes, 41943040 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
  
  
  
  
Disk /dev/mapper/rhel-root: 9458 MB, 9458155520 bytes, 18472960 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
  
  
  
  
Disk /dev/mapper/rhel-swap: 536 MB, 536870912 bytes, 1048576 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
  
  
  
  
Disk /dev/mapper/rhel-home: 524 MB, 524288000 bytes, 1024000 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
  
  
[root@mastera0 mysql]#   
[root@mastera0 mysql]# fdisk /dev/vdb  
Welcome to fdisk (util-linux 2.23.2).  
  
  
Changes will remain in memory only, until you decide to write them.  
Be careful before using the write command.  
  
  
Device does not contain a recognized partition table  
Building a new DOS disklabel with disk identifier 0xb9ec589f.  
  
  
Command (m for help): n  
Partition type:  
 p primary (0 primary, 0 extended, 4 free)  
 e extended  
Select (default p): p  
Partition number (1-4, default 1): 1  
First sector (2048-41943039, default 2048):   
Using default value 2048  
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039): +1G  
Partition 1 of type Linux and of size 1 GiB is set  
  
  
Command (m for help): n  
Partition type:  
 p primary (1 primary, 0 extended, 3 free)  
 e extended  
Select (default p): p  
Partition number (2-4, default 2): 2  
First sector (2099200-41943039, default 2099200):   
Using default value 2099200  
Last sector, +sectors or +size{K,M,G} (2099200-41943039, default 41943039): +1G  
Partition 2 of type Linux and of size 1 GiB is set  
  
  
Command (m for help): p  
  
  
Disk /dev/vdb: 21.5 GB, 21474836480 bytes, 41943040 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
Disk label type: dos  
Disk identifier: 0xb9ec589f  
  
  
 Device Boot Start End Blocks Id System  
/dev/vdb1 2048 2099199 1048576 83 Linux  
/dev/vdb2 2099200 4196351 1048576 83 Linux  
  
  
Command (m for help): w  
The partition table has been altered!  
  
  
Calling ioctl() to re-read partition table.  
Syncing disks.  
[root@mastera0 mysql]# ls /dev/vdb*  
/dev/vdb /dev/vdb1 /dev/vdb2  
[root@mastera0 mysql]# pvcreate /dev/vdb{1,2}  
 Physical volume "/dev/vdb1" successfully created  
 Physical volume "/dev/vdb2" successfully created  
[root@mastera0 mysql]# vgcreate vgmysql /dev/vdb{1,2}  
 Volume group "vgmysql" successfully created  
[root@mastera0 mysql]# lvcreate -L 1G -n lv1 vgmysql  
 Logical volume "lv1" created.  
[root@mastera0 mysql]# lvs  
 LV VG Attr LSize Pool Origin Data% Meta% Move Log Cpy%Sync Convert  
 home rhel -wi-ao---- 500.00m   
 root rhel -wi-ao---- 8.81g   
 swap rhel -wi-ao---- 512.00m   
 lv1 vgmysql -wi-a----- 1.00g   
[root@mastera0 mysql]# mkfs.ext4 /dev/vgmysql/lv1   
mke2fs 1.42.9 (28-Dec-2013)  
Filesystem label=  
OS type: Linux  
Block size=4096 (log=2)  
Fragment size=4096 (log=2)  
Stride=0 blocks, Stripe width=0 blocks  
65536 inodes, 262144 blocks  
13107 blocks (5.00%) reserved for the super user  
First data block=0  
Maximum filesystem blocks=268435456  
8 block groups  
32768 blocks per group, 32768 fragments per group  
8192 inodes per group  
Superblock backups stored on blocks:   
32768, 98304, 163840, 229376  
  
  
Allocating group tables: done   
Writing inode tables: done   
Creating journal (8192 blocks): done  
Writing superblocks and filesystem accounting information: done  
  
  
[root@mastera0 mysql]#   
[root@mastera0 mysql]# systemctl stop mariadb  
[root@mastera0 mysql]# tar -cf /tmp/mysql.1.tar /var/lib/mysql/  
tar: Removing leading `/' from member names  
[root@mastera0 mysql]# mount /dev/vgmysql/lv1 /var/lib/mysql^C  
[root@mastera0 mysql]# ll -dZ /var/lib/mysql  
drwxr-xr-x. mysql mysql system_u:object_r:mysqld_db_t:s0 /var/lib/mysql  
[root@mastera0 mysql]# mount /dev/vgmysql/lv1 /var/lib/mysql  
[root@mastera0 mysql]# ll -dZ /var/lib/mysql  
drwxr-xr-x. root root system_u:object_r:unlabeled_t:s0 /var/lib/mysql  
[root@mastera0 mysql]# ll /var/lib/mysql  
total 16  
drwx------. 2 root root 16384 Aug 30 13:57 lost+found  
[root@mastera0 ~]# ll -d /var/lib/mysql  
drwxr-xr-x. 3 root root 4096 Aug 30 13:57 /var/lib/mysql  
[root@mastera0 ~]# tar -xf /tmp/mysql.1.tar -C /  
[root@mastera0 ~]# ll -d /var/lib/mysql  
drwxr-xr-x. 7 mysql mysql 4096 Aug 30 14:01 /var/lib/mysql  
[root@mastera0 ~]# ll /var/lib/mysql  
total 28724  
-rw-rw----. 1 mysql mysql 16384 Aug 30 14:01 aria_log.00000001  
-rw-rw----. 1 mysql mysql 52 Aug 30 14:01 aria_log_control  
drwx------. 2 mysql mysql 4096 Aug 30 11:24 db1  
-rw-rw----. 1 mysql mysql 18874368 Aug 30 14:01 ibdata1  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 14:01 ib_logfile0  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 11:23 ib_logfile1  
drwx------. 2 root root 16384 Aug 30 13:57 lost+found  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 mysql  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 performance_schema  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 test  
[root@mastera0 ~]# systemctl start mariadb  
Job for mariadb.service failed because the control process exited with error code. See "systemctl status mariadb.service" and "journalctl -xe" for details.  
[root@mastera0 ~]# setenforce 0  
[root@mastera0 ~]# systemctl start mariadb  
[root@mastera0 ~]# ll /var/lib/mysql -Z  
-rw-rw----. mysql mysql unconfined_u:object_r:unlabeled_t:s0 aria_log.00000001  
-rw-rw----. mysql mysql unconfined_u:object_r:unlabeled_t:s0 aria_log_control  
drwx------. mysql mysql unconfined_u:object_r:unlabeled_t:s0 db1  
-rw-rw----. mysql mysql unconfined_u:object_r:unlabeled_t:s0 ibdata1  
-rw-rw----. mysql mysql unconfined_u:object_r:unlabeled_t:s0 ib_logfile0  
-rw-rw----. mysql mysql unconfined_u:object_r:unlabeled_t:s0 ib_logfile1  
drwx------. root root system_u:object_r:unlabeled_t:s0 lost+found  
drwx------. mysql mysql unconfined_u:object_r:unlabeled_t:s0 mysql  
srwxrwxrwx. mysql mysql system_u:object_r:unlabeled_t:s0 mysql.sock  
drwx------. mysql mysql unconfined_u:object_r:unlabeled_t:s0 performance_schema  
drwx------. mysql mysql unconfined_u:object_r:unlabeled_t:s0 test  
  
  
# lvm快照备份  
MariaDB [(none)]> flush tables with read lock;  
Query OK, 0 rows affected (0.00 sec)  
## 数据库加上全局读锁后，立刻在新终端中创建快照  
[root@mastera0 ~]# lvcreate -s -L 500M -n snap1 /dev/vgmysql/lv1   
 Logical volume "snap1" created.  
## 快照创建之后，解锁，服务可以正常适用了  
MariaDB [(none)]> unlock tables;  
Query OK, 0 rows affected (0.00 sec)  
  
  
## 挂接快照使用  
[root@mastera0 ~]# mount /dev/vgmysql/snap1 /mnt  
[root@mastera0 ~]# ls /mnt  
aria_log.00000001 db1 ib_logfile0 lost+found mysql.sock test  
aria_log_control ibdata1 ib_logfile1 mysql performance_schema  
[root@mastera0 ~]# cd /mnt  
[root@mastera0 mnt]# tar -cf /tmp/mysql.2.tar ./*  
tar: ./mysql.sock: socket ignored  
[root@mastera0 mnt]# ll /tmp  
total 89316  
drwxr-xr-x. 2 root root 6 Aug 30 11:31 a  
-rw-r--r--. 1 root root 30484480 Aug 30 14:01 mysql.1.tar  
-rw-r--r--. 1 root root 30484480 Aug 30 14:58 mysql.2.tar  
-rw-r--r--. 1 root root 30484480 Aug 30 11:28 mysql.all.tar  
[root@mastera0 mnt]# cd   
[root@mastera0 ~]# umount /mnt  
[root@mastera0 ~]# lv  
lvchange lvdisplay lvmchange lvmdiskscan lvmpolld lvreduce lvresize  
lvconvert lvextend lvmconf lvmdump lvmsadc lvremove lvs  
lvcreate lvm lvmconfig lvmetad lvmsar lvrename lvscan  
[root@mastera0 ~]# lvremove /dev/vgmysql/snap1   
Do you really want to remove active logical volume snap1? [y/n]: y  
 Logical volume "snap1" successfully removed  
  
  
# 还原数据  
[root@mastera0 ~]# systemctl stop mariadb  
[root@mastera0 ~]# rm -rf /var/lib/mysql/*  
[root@mastera0 ~]# tar -xf /tmp/mysql.2.tar -C /var/lib/mysql  
[root@mastera0 ~]# ll /var/lib/mysql  
total 28712  
-rw-rw----. 1 mysql mysql 16384 Aug 30 14:01 aria_log.00000001  
-rw-rw----. 1 mysql mysql 52 Aug 30 14:01 aria_log_control  
drwx------. 2 mysql mysql 4096 Aug 30 11:24 db1  
-rw-rw----. 1 mysql mysql 18874368 Aug 30 14:42 ibdata1  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 14:42 ib_logfile0  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 11:23 ib_logfile1  
drwx------. 2 root root 4096 Aug 30 13:57 lost+found  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 mysql  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 performance_schema  
drwx------. 2 mysql mysql 4096 Aug 30 11:23 test  
[root@mastera0 ~]# systemctl start mariadb  
[root@mastera0 ~]# echo "select * from db1.t1" | mysql -uroot -puplooking  
id  
1  
2  
  
  
--------------------------------  
# mysqldump备份还原详细步骤  
[root@mastera0 ~]# mysqldump -uroot -puplooking -A --single-transaction > /tmp/mysql.all.1.sql  
[root@mastera0 ~]# ll /tmp  
total 89820  
drwxr-xr-x. 2 root root 6 Aug 30 11:31 a  
-rw-r--r--. 1 root root 30484480 Aug 30 14:01 mysql.1.tar  
-rw-r--r--. 1 root root 30484480 Aug 30 14:58 mysql.2.tar  
-rw-r--r--. 1 root root 515980 Aug 30 16:02 mysql.all.1.sql  
-rw-r--r--. 1 root root 30484480 Aug 30 11:28 mysql.all.tar  
[root@mastera0 ~]# systemctl stop mariadb  
[root@mastera0 ~]# rm -rf /var/lib/mysql/*  
[root@mastera0 ~]# ll /var/lib/mysql  
total 0  
[root@mastera0 ~]# systemctl start mariadb  
[root@mastera0 ~]# ll /var/lib/mysql  
total 28704  
-rw-rw----. 1 mysql mysql 16384 Aug 30 16:10 aria_log.00000001  
-rw-rw----. 1 mysql mysql 52 Aug 30 16:10 aria_log_control  
-rw-rw----. 1 mysql mysql 18874368 Aug 30 16:10 ibdata1  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 16:10 ib_logfile0  
-rw-rw----. 1 mysql mysql 5242880 Aug 30 16:10 ib_logfile1  
drwx------. 2 mysql mysql 4096 Aug 30 16:10 mysql  
srwxrwxrwx. 1 mysql mysql 0 Aug 30 16:10 mysql.sock  
drwx------. 2 mysql mysql 4096 Aug 30 16:10 performance_schema  
drwx------. 2 mysql mysql 4096 Aug 30 16:10 test  
[root@mastera0 ~]# mysql  
Welcome to the MariaDB monitor. Commands end with ; or \g.  
Your MariaDB connection id is 2  
Server version: 5.5.44-MariaDB MariaDB Server  
  
  
Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
MariaDB [(none)]> \q  
Bye  
[root@mastera0 ~]#   
  
  
  
  
  
  
[root@mastera0 ~]# mysql < /tmp/mysql.all.1.sql   
[root@mastera0 ~]# mysql  
Welcome to the MariaDB monitor. Commands end with ; or \g.  
Your MariaDB connection id is 4  
Server version: 5.5.44-MariaDB MariaDB Server  
  
  
Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
MariaDB [(none)]> flush privileges;  
Query OK, 0 rows affected (0.00 sec)  
  
  
MariaDB [(none)]> \q  
Bye  
[root@mastera0 ~]# mysql  
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)  
[root@mastera0 ~]# mysql -uroot -puplooking  
Welcome to the MariaDB monitor. Commands end with ; or \g.  
Your MariaDB connection id is 6  
Server version: 5.5.44-MariaDB MariaDB Server  
  
  
Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
MariaDB [(none)]> select * from db1.t1;  
+----+  
| id |  
+----+  
| 1 |  
| 2 |  
| 3 |  
+----+  
3 rows in set (0.00 sec)  
  
  
MariaDB [(none)]> show databases;  
+---------------------+  
| Database |  
+---------------------+  
| information_schema |  
| db1 |  
| db2 |  
| #mysql50#lost+found |  
| mysql |  
| performance_schema |  
| test |  
+---------------------+  
7 rows in set (0.00 sec)  
  
  
MariaDB [(none)]> exit  
  
  
  
  
-------------------------  
-----------------------  
晚自习作业  
0.发一封邮件给weiyaping@uplooking.com  
1.tar打包全备份数据；模拟人为误操作；tar还原全备份数据；  
2.创建逻辑卷lv1并挂接给数据库适用；通过lvm快照方式作备份；模拟人为误操作；还原数据；  
3.mysqldump备份数据；模拟人为误操作；还原数据；   
 