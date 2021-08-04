---
title: Mysql学习第四课01-percona-xtrabackup
date: 2016-09-01 22:25:54
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52403947   
  20160901mysqlday4  
  
  
 复习  
 --------------  
 1.增量备份---实时增量备份与还原（基于时间点的恢复）  
 1）如何打开二进制日志，详细步骤  
1>   
2>  
3>  
 2）如何查看二进制日志  
1> 命令  
2> 通过时间来截取  
3> 通过位置编号来截取  
 3）全备份数据时记录备份开始后的所有写操作记录在新的日志中，并记录该日志的日志名，参数分别为  
1> 刷新日志  
2> 记录日志名和位置编号  
 4）如何导入二进制日志  
  
  
 48  
 ==================================  
 今天的内容  
1.第三方备份工具Percona Xtrabackup  
2.冗余环境的搭建和原理  
 -------------------------------------------------  
 冷备---备份  
全备份：  
物理分备--tar\lvm snapshot  
逻辑备份--mysqldump  
增量备份：  
实时备份--mysqlbinlog二进制日志  
增量备份--Percona Xtrabackup（全备份和增量备份）  
  
  
  
  
 Percona Xtrabackup  
 -------------------------------  
 create db1.t1  
 insert 1  
 2  
 3  
 4  
 ----------------------全备份 all 周一：8:00  
 5  
 ----------------------周二 增量备份1 5 增量备份1 5  
 6   
 ----------------------周三 增量备份2 6 增量备份2 5 6  
 7  
 ----------------------周四 增量备份3 7 增量备份3 5 6 7  
 8  
 ----------------------周五 增量备份4 8 增量备份4 5 6 7 8  
 innobackupex  
   
 针对的存储引擎：innodb\xtradb  
 # 如何获取软件？  
官网 percona  
 # git版本控制软件 github---mysql---mariadb  
  
  
 # 教室环境已经下载好软件，存放路径为：  
http://classroom.example.com/content/MYSQL/04-others/soft/Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar  
 # 缺少libev.so.4()(64bit)  
http://classroom.example.com/content/MYSQL/04-others/soft/libev-4.15-6.el7.x86_64.rpm  
 # install libev-4.15-6.el7.x86_64.rpm  
 # install percona-xtrabackup-2.3.4-1.el7.x86_64.rpm  
 -----------------  
 # 查看软件架构  
 软件名 percona-xtrabackup  
 命令 xtrabackup  innodb/xtradb  
innobackupex  innodb/xtradb myisam  
 选项 --user=name 用户名  
----password=name 密码  
--apply-log  
--copy-back  
--redo-only  
  
 示例 全备份  innobackupex --user=root --password=uplooking /tmp/backup   
还原全备份 innobackupex --apply-log /tmp/backup/2016----- #重演和回滚同时进行  
innobackupex --copy-back /tmp/backup/2016----- #数据还原  
  
  
1）停止服务  
2）请环境  
3）导入数据  
1> apply-log  innobackupex --apply-log /tmp/backup/2016-----  
2> copy-back  innobackupex --copy-back /tmp/backup/2016-----  
4）修改权限  
5）启动服务  
6）测试  
  
  
 ------------------------------------------------------------------------------  
全备份 innobackupex --user=root --password=uplooking /tmp/backup  
增量备份1 innobackupex --user=root --password=uplooking --incremental-basedir=/tmp/backup/2016-09-01_11-32-43 --incremental /tmp/backup  
增量备份2 innobackupex --user=root --password=uplooking --incremental-basedir=/tmp/  
 backup/增量1 --incremental /tmp/backup  
增量备份3 innobackupex --user=root --password=uplooking --incremental-basedir=/tmp/  
 backup/增量2 --incremental /tmp/backup  
  
  
全备份还原innobackupex --apply-log --redo-only /tmp/backup/全备份 #只重演不回滚  
innobackupex --apply-log --redo-only /tmp/backup/全备份 --incremental-dir=增量1 #只重演不回滚  
innobackupex --apply-log --redo-only /tmp/backup/全备份 --incremental-dir=增量2 #只重演不回滚  
innobackupex --apply-log --redo-only /tmp/backup/全备份 --incremental-dir=增量3 #只重演不回滚  
innobackupex --apply-log /tmp/backup/全备份 #重演和回滚同时进行  
innobackupex --copy-back /tmp/backup/全备份 #数据还原  
  
  
  
  
 --------------------  
db1.t1 1 2 3  redolog begin;insert 123;commit; ibdata 1 2 3  
4 begin;insert 4; ibdata 4  
5 commit;begin;insert 5; ibdata 5  
6 commit;begin;insert 6; ibdata 6  
 -----------------------------------------------------------------  
还原 1.准备数据 全备份+增1+增2+增3  
2.检查事务日志（重演）全备份+增1+增2+增3---》检查  
3.回滚  
4.数据还原 copy-back  
  
  
 =================================================================  
 # Percona xtrabackup的原理  
 1.先备份innodb存储引擎的表，再备份MYISAM的表；  
 2.innodb--数据一致，服务可用，myisam--数据一致，全局读锁，时间点是结束的时间点  
 3.备份的内容：/var/lib/mysql/ 在备份的时候，物理备份  
 4.innodb存储引擎的表是存放在表空间---》ibdata1  
 5.innodb的事务日志ib_logfile 重做日志 redolog  
 6.innodb的表如何还原  
1> 准备数据 ibdata  
2> 重演日志 apply-log redo-only（重演不回滚） 检查事务的完整性  
3> 回滚  apply-log(包括了重演和回滚) 发现不完整的是事务，滚回到不完整事务的begin之前  4> 还原数据 copy-back   
  
  
 binlog 所有的存储引擎 写操作 完整的事务  
 redolog innodb存储引擎 写操作 所有的语句  
  
  
  
  
  
  
  
  
  
 ------------------------------------------  
 sql redolog  数据ibdata1  
 begin; begin;    
 insert into db1.t1 values (0); 0  
 insert 2 2  
 commit; commit; 0,2  
  
  
 -----------------------------------------------  
 begin;  
 insert 0 0  
 insert 2 2  
  
  
 ---------------------------------------------  
 # 详细步骤  
 ## install  
 [root@mastera0 ~]# setenforce 0  
 [root@mastera0 ~]# systemctl stop firewalld  
 [root@mastera0 ~]# wget http://classroom.example.com/content/MYSQL/04-others/soft/Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar  
 --2016-09-01 10:41:05-- http://classroom.example.com/content/MYSQL/04-others/soft/Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar  
 Reusing existing connection to classroom.example.com:80.  
 HTTP request sent, awaiting response... 200 OK  
 Length: 25548800 (24M) [application/x-tar]  
 Saving to: ‘Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar’  
  
  
 100%[===========================================>] 25,548,800 122MB/s in 0.2s   
  
  
 2016-09-01 10:41:05 (122 MB/s) - ‘Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar’ saved [25548800/25548800]  
  
  
 FINISHED --2016-09-01 10:41:05--  
 Total wall clock time: 0.3s  
 Downloaded: 1 files, 24M in 0.2s (122 MB/s)  
 [root@mastera0 ~]# ls  
 anaconda-ks.cfg Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar  
 [root@mastera0 ~]# tar -xf Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar   
 [root@mastera0 ~]# ls  
 anaconda-ks.cfg  
 percona-xtrabackup-2.3.4-1.el7.x86_64.rpm  
 Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar  
 percona-xtrabackup-debuginfo-2.3.4-1.el7.x86_64.rpm  
 percona-xtrabackup-test-2.3.4-1.el7.x86_64.rpm  
 [root@mastera0 ~]# yum localinstall -y percona-xtrabackup  
 Loaded plugins: product-id, search-disabled-repos, subscription-manager  
 This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.  
 Skipping: percona-xtrabackup, filename does not end in .rpm.  
 Nothing to do  
 [root@mastera0 ~]# yum localinstall -y percona-xtrabackup-2.3.4-1.el7.x86_64.rpm   
 Loaded plugins: product-id, search-disabled-repos, subscription-manager  
 This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.  
 Examining percona-xtrabackup-2.3.4-1.el7.x86_64.rpm: percona-xtrabackup-2.3.4-1.el7.x86_64  
 Marking percona-xtrabackup-2.3.4-1.el7.x86_64.rpm to be installed  
 Resolving Dependencies  
 --> Running transaction check  
 ---> Package percona-xtrabackup.x86_64 0:2.3.4-1.el7 will be installed  
 --> Processing Dependency: rsync for package: percona-xtrabackup-2.3.4-1.el7.x86_64  
 rhel_dvd | 4.1 kB 00:00:00   
 --> Processing Dependency: libev.so.4()(64bit) for package: percona-xtrabackup-2.3.4-1.el7.x86_64  
 --> Running transaction check  
 ---> Package percona-xtrabackup.x86_64 0:2.3.4-1.el7 will be installed  
 --> Processing Dependency: libev.so.4()(64bit) for package: percona-xtrabackup-2.3.4-1.el7.x86_64  
 ---> Package rsync.x86_64 0:3.0.9-17.el7 will be installed  
 --> Finished Dependency Resolution  
 Error: Package: percona-xtrabackup-2.3.4-1.el7.x86_64 (/percona-xtrabackup-2.3.4-1.el7.x86_64)  
 Requires: libev.so.4()(64bit)  
 You could try using --skip-broken to work around the problem  
 You could try running: rpm -Va --nofiles --nodigest  
 [root@mastera0 ~]# wget http://classroom.example.com/content/MYSQL/04-others/soft/libev-4.15-6.el7.x86_64.rpm  
 --2016-09-01 10:43:37-- http://classroom.example.com/content/MYSQL/04-others/soft/libev-4.15-6.el7.x86_64.rpm  
 Resolving classroom.example.com (classroom.example.com)... 172.25.254.254  
 Connecting to classroom.example.com (classroom.example.com)|172.25.254.254|:80... connected.  
 HTTP request sent, awaiting response... 200 OK  
 Length: 44964 (44K) [application/x-rpm]  
 Saving to: ‘libev-4.15-6.el7.x86_64.rpm’  
  
  
 100%[===========================================>] 44,964 --.-K/s in 0s   
  
  
 2016-09-01 10:43:37 (360 MB/s) - ‘libev-4.15-6.el7.x86_64.rpm’ saved [44964/44964]  
  
  
 [root@mastera0 ~]# ls  
 anaconda-ks.cfg  
 libev-4.15-6.el7.x86_64.rpm  
 percona-xtrabackup-2.3.4-1.el7.x86_64.rpm  
 Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar  
 percona-xtrabackup-debuginfo-2.3.4-1.el7.x86_64.rpm  
 percona-xtrabackup-test-2.3.4-1.el7.x86_64.rpm  
 [root@mastera0 ~]# yum localinstall -y libev-4.15-6.el7.x86_64.rpm  
 Loaded plugins: product-id, search-disabled-repos, subscription-manager  
 This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.  
 Examining libev-4.15-6.el7.x86_64.rpm: libev-4.15-6.el7.x86_64  
 Marking libev-4.15-6.el7.x86_64.rpm to be installed  
 Resolving Dependencies  
 --> Running transaction check  
 ---> Package libev.x86_64 0:4.15-6.el7 will be installed  
 --> Finished Dependency Resolution  
  
  
 Dependencies Resolved  
  
  
 =====================================================================================  
 Package Arch Version Repository Size  
 =====================================================================================  
 Installing:  
 libev x86_64 4.15-6.el7 /libev-4.15-6.el7.x86_64 86 k  
  
  
 Transaction Summary  
 =====================================================================================  
 Install 1 Package  
  
  
 Total size: 86 k  
 Installed size: 86 k  
 Downloading packages:  
 Running transaction check  
 Running transaction test  
 Transaction test succeeded  
 Running transaction  
 Installing : libev-4.15-6.el7.x86_64 1/1   
 Verifying : libev-4.15-6.el7.x86_64 1/1   
  
  
 Installed:  
 libev.x86_64 0:4.15-6.el7   
  
  
 Complete!  
 [root@mastera0 ~]# yum localinstall -y percona-xtrabackup-2.3.4-1.el7.x86_64.rpm Loaded plugins: product-id, search-disabled-repos, subscription-manager  
 This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.  
 Examining percona-xtrabackup-2.3.4-1.el7.x86_64.rpm: percona-xtrabackup-2.3.4-1.el7.x86_64  
 Marking percona-xtrabackup-2.3.4-1.el7.x86_64.rpm to be installed  
 Resolving Dependencies  
 --> Running transaction check  
 ---> Package percona-xtrabackup.x86_64 0:2.3.4-1.el7 will be installed  
 --> Processing Dependency: rsync for package: percona-xtrabackup-2.3.4-1.el7.x86_64  
 --> Running transaction check  
 ---> Package rsync.x86_64 0:3.0.9-17.el7 will be installed  
 --> Finished Dependency Resolution  
  
  
 Dependencies Resolved  
  
  
 =====================================================================================  
 Package Arch Version Repository Size  
 =====================================================================================  
 Installing:  
 percona-xtrabackup x86_64 2.3.4-1.el7 /percona-xtrabackup-2.3.4-1.el7.x86_64 21 M  
 Installing for dependencies:  
 rsync x86_64 3.0.9-17.el7 rhel_dvd 359 k  
  
  
 Transaction Summary  
 =====================================================================================  
 Install 1 Package (+1 Dependent package)  
  
  
 Total size: 22 M  
 Total download size: 359 k  
 Installed size: 22 M  
 Downloading packages:  
 rsync-3.0.9-17.el7.x86_64.rpm | 359 kB 00:00:00   
 Running transaction check  
 Running transaction test  
 Transaction test succeeded  
 Running transaction  
 Installing : rsync-3.0.9-17.el7.x86_64 1/2   
 Installing : percona-xtrabackup-2.3.4-1.el7.x86_64 2/2   
 Verifying : rsync-3.0.9-17.el7.x86_64 1/2   
 Verifying : percona-xtrabackup-2.3.4-1.el7.x86_64 2/2   
  
  
 Installed:  
 percona-xtrabackup.x86_64 0:2.3.4-1.el7   
  
  
 Dependency Installed:  
 rsync.x86_64 0:3.0.9-17.el7   
  
  
 Complete!  
 [root@mastera0 ~]# ls  
 anaconda-ks.cfg  
 libev-4.15-6.el7.x86_64.rpm  
 percona-xtrabackup-2.3.4-1.el7.x86_64.rpm  
 Percona-XtraBackup-2.3.4-re80c779-el7-x86_64-bundle.tar  
 percona-xtrabackup-debuginfo-2.3.4-1.el7.x86_64.rpm  
 percona-xtrabackup-test-2.3.4-1.el7.x86_64.rpm  
 [root@mastera0 ~]# rpm -ql percona-xtrabackup  
 /usr/bin/innobackupex  
 /usr/bin/xbcloud  
 /usr/bin/xbcloud_osenv  
 /usr/bin/xbcrypt  
 /usr/bin/xbstream  
 /usr/bin/xtrabackup  
 /usr/share/doc/percona-xtrabackup-2.3.4  
 /usr/share/doc/percona-xtrabackup-2.3.4/COPYING  
 /usr/share/man/man1/innobackupex.1.gz  
 /usr/share/man/man1/xbcrypt.1.gz  
 /usr/share/man/man1/xbstream.1.gz  
 /usr/share/man/man1/xtrabackup.1.gz  
  
  
 ## backup  
 [root@mastera0 mysql-log]# innobackupex --user=root --password=uplooking /tmp/backup  
 160901 11:32:43 innobackupex: Starting the backup operation  
  
  
 IMPORTANT: Please check that the backup run completes successfully.  
 At the end of a successful backup run innobackupex  
 prints "completed OK!".  
  
  
 Can't locate Digest/MD5.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at - line 693.  
 BEGIN failed--compilation aborted at - line 693.  
 160901 11:32:43 Connecting to MySQL server host: localhost, user: root, password: set, port: 0, socket: /var/lib/mysql/mysql.sock  
 Using server version 5.5.44-MariaDB-log  
 innobackupex version 2.3.4 based on MySQL server 5.6.24 Linux (x86_64) (revision id: e80c779)  
 xtrabackup: uses posix_fadvise().  
 xtrabackup: cd to /var/lib/mysql  
 xtrabackup: open files limit requested 0, set to 1024  
 xtrabackup: using the following InnoDB configuration:  
 xtrabackup: innodb_data_home_dir = ./  
 xtrabackup: innodb_data_file_path = ibdata1:10M:autoextend  
 xtrabackup: innodb_log_group_home_dir = ./  
 xtrabackup: innodb_log_files_in_group = 2  
 xtrabackup: innodb_log_file_size = 5242880  
 160901 11:32:43 >> log scanned up to (1607773)  
 xtrabackup: Generating a list of tablespaces  
 160901 11:32:43 [01] Copying ./ibdata1 to /tmp/backup/2016-09-01_11-32-43/ibdata1  
 160901 11:32:43 [01] ...done  
 160901 11:32:44 >> log scanned up to (1607773)  
 160901 11:32:44 Executing FLUSH NO_WRITE_TO_BINLOG TABLES...  
 160901 11:32:44 Executing FLUSH TABLES WITH READ LOCK...  
 160901 11:32:44 Starting to backup non-InnoDB tables and files  
 160901 11:32:44 [01] Copying ./mysql/tables_priv.frm to /tmp/backup/2016-09-01_11-32-43/mysql/tables_priv.frm  
 160901 11:32:44 [01] ...done  
 160901 11:32:44 [01] Copying ./mysql/tables_priv.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/tables_priv.MYI  
 160901 11:32:44 [01] ...done  
 160901 11:32:44 [01] Copying ./mysql/time_zone.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone.MYD  
 160901 11:32:44 [01] ...done  
 160901 11:32:44 [01] Copying ./mysql/ndb_binlog_index.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/ndb_binlog_index.MYD  
 160901 11:32:44 [01] ...done  
 160901 11:32:44 [01] Copying ./mysql/plugin.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/plugin.MYD  
 160901 11:32:44 [01] ...done  
 160901 11:32:44 [01] Copying ./mysql/proc.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/proc.MYI  
 160901 11:32:44 [01] ...done  
 160901 11:32:44 [01] Copying ./mysql/procs_priv.frm to /tmp/backup/2016-09-01_11-32-43/mysql/procs_priv.frm  
 160901 11:32:44 [01] ...done  
 160901 11:32:44 [01] Copying ./mysql/procs_priv.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/procs_priv.MYD  
 160901 11:32:44 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/proxies_priv.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/proxies_priv.MYI  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/servers.frm to /tmp/backup/2016-09-01_11-32-43/mysql/servers.frm  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/servers.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/servers.MYD  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone.MYI  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_name.frm to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_name.frm  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_name.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_name.MYI  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_name.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_name.MYD  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_transition.frm to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_transition.frm  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_transition.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_transition.MYI  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 >> log scanned up to (1607773)  
 160901 11:32:45 [01] Copying ./mysql/time_zone_transition.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_transition.MYD  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_transition_type.frm to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_transition_type.frm  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_transition_type.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_transition_type.MYI  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_transition_type.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_transition_type.MYD  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/user.frm to /tmp/backup/2016-09-01_11-32-43/mysql/user.frm  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/user.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/user.MYI  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/user.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/user.MYD  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_leap_second.frm to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_leap_second.frm  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/time_zone_leap_second.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_leap_second.MYI  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/general_log.frm to /tmp/backup/2016-09-01_11-32-43/mysql/general_log.frm  
 160901 11:32:45 [01] ...done  
 160901 11:32:45 [01] Copying ./mysql/general_log.CSM to /tmp/backup/2016-09-01_11-32-43/mysql/general_log.CSM  
 160901 11:32:45 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/general_log.CSV to /tmp/backup/2016-09-01_11-32-43/mysql/general_log.CSV  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/slow_log.frm to /tmp/backup/2016-09-01_11-32-43/mysql/slow_log.frm  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/slow_log.CSM to /tmp/backup/2016-09-01_11-32-43/mysql/slow_log.CSM  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/slow_log.CSV to /tmp/backup/2016-09-01_11-32-43/mysql/slow_log.CSV  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/tables_priv.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/tables_priv.MYD  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/columns_priv.frm to /tmp/backup/2016-09-01_11-32-43/mysql/columns_priv.frm  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/columns_priv.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/columns_priv.MYI  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/columns_priv.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/columns_priv.MYD  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/db.frm to /tmp/backup/2016-09-01_11-32-43/mysql/db.frm  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/db.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/db.MYI  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/db.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/db.MYD  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 >> log scanned up to (1607773)  
 160901 11:32:46 [01] Copying ./mysql/event.frm to /tmp/backup/2016-09-01_11-32-43/mysql/event.frm  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/event.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/event.MYI  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/event.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/event.MYD  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/func.frm to /tmp/backup/2016-09-01_11-32-43/mysql/func.frm  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/func.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/func.MYI  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/func.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/func.MYD  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/help_category.frm to /tmp/backup/2016-09-01_11-32-43/mysql/help_category.frm  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/help_category.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/help_category.MYI  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/help_category.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/help_category.MYD  
 160901 11:32:46 [01] ...done  
 160901 11:32:46 [01] Copying ./mysql/help_keyword.frm to /tmp/backup/2016-09-01_11-32-43/mysql/help_keyword.frm  
 160901 11:32:46 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/help_keyword.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/help_keyword.MYI  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/help_keyword.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/help_keyword.MYD  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/help_relation.frm to /tmp/backup/2016-09-01_11-32-43/mysql/help_relation.frm  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/help_relation.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/help_relation.MYI  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/help_relation.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/help_relation.MYD  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/help_topic.frm to /tmp/backup/2016-09-01_11-32-43/mysql/help_topic.frm  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/help_topic.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/help_topic.MYI  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 >> log scanned up to (1607773)  
 160901 11:32:47 [01] Copying ./mysql/help_topic.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/help_topic.MYD  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/host.frm to /tmp/backup/2016-09-01_11-32-43/mysql/host.frm  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/host.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/host.MYI  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/host.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/host.MYD  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/ndb_binlog_index.frm to /tmp/backup/2016-09-01_11-32-43/mysql/ndb_binlog_index.frm  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/ndb_binlog_index.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/ndb_binlog_index.MYI  
 160901 11:32:47 [01] ...done  
 160901 11:32:47 [01] Copying ./mysql/plugin.frm to /tmp/backup/2016-09-01_11-32-43/mysql/plugin.frm  
 160901 11:32:47 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/plugin.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/plugin.MYI  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/proc.frm to /tmp/backup/2016-09-01_11-32-43/mysql/proc.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/proc.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/proc.MYD  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/procs_priv.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/procs_priv.MYI  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/proxies_priv.frm to /tmp/backup/2016-09-01_11-32-43/mysql/proxies_priv.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/proxies_priv.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/proxies_priv.MYD  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/servers.MYI to /tmp/backup/2016-09-01_11-32-43/mysql/servers.MYI  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/time_zone.frm to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./mysql/time_zone_leap_second.MYD to /tmp/backup/2016-09-01_11-32-43/mysql/time_zone_leap_second.MYD  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 >> log scanned up to (1607773)  
 160901 11:32:48 [00] Writing test/db.opt  
 160901 11:32:48 [00] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/db.opt to /tmp/backup/2016-09-01_11-32-43/performance_schema/db.opt  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/cond_instances.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/cond_instances.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/events_waits_current.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/events_waits_current.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/events_waits_history.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/events_waits_history.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/events_waits_history_long.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/events_waits_history_long.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/events_waits_summary_by_instance.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/events_waits_summary_by_thread_by_event_name.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/events_waits_summary_global_by_event_name.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:48 [01] Copying ./performance_schema/file_instances.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/file_instances.frm  
 160901 11:32:48 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/file_summary_by_event_name.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/file_summary_by_instance.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/file_summary_by_instance.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/mutex_instances.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/mutex_instances.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/performance_timers.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/performance_timers.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/rwlock_instances.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/rwlock_instances.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/setup_consumers.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/setup_consumers.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/setup_instruments.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/setup_instruments.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/setup_timers.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/setup_timers.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./performance_schema/threads.frm to /tmp/backup/2016-09-01_11-32-43/performance_schema/threads.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 >> log scanned up to (1607773)  
 160901 11:32:49 [01] Copying ./db1/db.opt to /tmp/backup/2016-09-01_11-32-43/db1/db.opt  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./db1/t1.frm to /tmp/backup/2016-09-01_11-32-43/db1/t1.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./db2/db.opt to /tmp/backup/2016-09-01_11-32-43/db2/db.opt  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 [01] Copying ./db2/t1.frm to /tmp/backup/2016-09-01_11-32-43/db2/t1.frm  
 160901 11:32:49 [01] ...done  
 160901 11:32:49 Finished backing up non-InnoDB tables and files  
 160901 11:32:49 [00] Writing xtrabackup_binlog_info  
 160901 11:32:49 [00] ...done  
 160901 11:32:49 Executing FLUSH NO_WRITE_TO_BINLOG ENGINE LOGS...  
 xtrabackup: The latest check point (for incremental): '1607773'  
 xtrabackup: Stopping log copying thread.  
 .160901 11:32:49 >> log scanned up to (1607773)  
  
  
 160901 11:32:50 Executing UNLOCK TABLES  
 160901 11:32:50 All tables unlocked  
 160901 11:32:50 Backup created in directory '/tmp/backup/2016-09-01_11-32-43'  
 MySQL binlog position: filename 'mastera.000021', position '426'  
 160901 11:32:50 [00] Writing backup-my.cnf  
 160901 11:32:50 [00] ...done  
 160901 11:32:50 [00] Writing xtrabackup_info  
 160901 11:32:50 [00] ...done  
 xtrabackup: Transaction log of lsn (1607773) to (1607773) was copied.  
 160901 11:32:50 completed OK!  
 [root@mastera0 mysql-log]# cd /tmp/backup/  
 [root@mastera0 backup]# ll  
 total 4  
 drwx------. 7 root root 4096 Sep 1 11:32 2016-09-01_11-32-43  
 [root@mastera0 backup]# cd 2016-09-01_11-32-43/  
 [root@mastera0 2016-09-01_11-32-43]# ll  
 total 18460  
 -rw-r-----. 1 root root 386 Sep 1 11:32 backup-my.cnf  
 drwx------. 2 root root 32 Sep 1 11:32 db1  
 drwx------. 2 root root 32 Sep 1 11:32 db2  
 -rw-r-----. 1 root root 18874368 Sep 1 11:32 ibdata1  
 drwx------. 2 root root 4096 Sep 1 11:32 mysql  
 drwx------. 2 root root 4096 Sep 1 11:32 performance_schema  
 drwx------. 2 root root 19 Sep 1 11:32 test  
 -rw-r-----. 1 root root 19 Sep 1 11:32 xtrabackup_binlog_info  
 -rw-r-----. 1 root root 113 Sep 1 11:32 xtrabackup_checkpoints  
 -rw-r-----. 1 root root 473 Sep 1 11:32 xtrabackup_info  
 -rw-r-----. 1 root root 2560 Sep 1 11:32 xtrabackup_logfile  
  
  
 ## copyback  
  
  
 [root@mastera0 mysql]# systemctl stop mariadb  
 [root@mastera0 mysql]# rm -rf /var/lib/mysql/*  
 [root@mastera0 mysql]# innobackupex --apply-log /tmp/backup/2016-09-01_11-32-43/  
 160901 11:36:33 innobackupex: Starting the apply-log operation  
  
  
 IMPORTANT: Please check that the apply-log run completes successfully.  
 At the end of a successful apply-log run innobackupex  
 prints "completed OK!".  
  
  
 innobackupex version 2.3.4 based on MySQL server 5.6.24 Linux (x86_64) (revision id: e80c779)  
 xtrabackup: cd to /tmp/backup/2016-09-01_11-32-43/  
 xtrabackup: This target seems to be not prepared yet.  
 xtrabackup: xtrabackup_logfile detected: size=2097152, start_lsn=(1607773)  
 xtrabackup: using the following InnoDB configuration for recovery:  
 xtrabackup: innodb_data_home_dir = ./  
 xtrabackup: innodb_data_file_path = ibdata1:10M:autoextend  
 xtrabackup: innodb_log_group_home_dir = ./  
 xtrabackup: innodb_log_files_in_group = 1  
 xtrabackup: innodb_log_file_size = 2097152  
 xtrabackup: using the following InnoDB configuration for recovery:  
 xtrabackup: innodb_data_home_dir = ./  
 xtrabackup: innodb_data_file_path = ibdata1:10M:autoextend  
 xtrabackup: innodb_log_group_home_dir = ./  
 xtrabackup: innodb_log_files_in_group = 1  
 xtrabackup: innodb_log_file_size = 2097152  
 xtrabackup: Starting InnoDB instance for recovery.  
 xtrabackup: Using 104857600 bytes for buffer pool (set by --use-memory parameter)  
 InnoDB: Using atomics to ref count buffer pool pages  
 InnoDB: The InnoDB memory heap is disabled  
 InnoDB: Mutexes and rw_locks use GCC atomic builtins  
 InnoDB: Memory barrier is not used  
 InnoDB: Compressed tables use zlib 1.2.7  
 InnoDB: Using CPU crc32 instructions  
 InnoDB: Initializing buffer pool, size = 100.0M  
 InnoDB: Completed initialization of buffer pool  
 InnoDB: Highest supported file format is Barracuda.  
 InnoDB: The log sequence numbers 0 and 0 in ibdata files do not match the log sequence number 1607773 in the ib_logfiles!  
 InnoDB: Database was not shutdown normally!  
 InnoDB: Starting crash recovery.  
 InnoDB: Reading tablespace information from the .ibd files...  
 InnoDB: Restoring possible half-written data pages   
 InnoDB: from the doublewrite buffer...  
 InnoDB: 128 rollback segment(s) are active.  
 InnoDB: Waiting for purge to start  
 InnoDB: 5.6.24 started; log sequence number 1607773  
 xtrabackup: Last MySQL binlog file position 426, file name /var/lib/mysql-log/mastera.000021  
  
  
 xtrabackup: starting shutdown with innodb_fast_shutdown = 1  
 InnoDB: FTS optimize thread exiting.  
 InnoDB: Starting shutdown...  
 InnoDB: Shutdown completed; log sequence number 1607783  
 xtrabackup: using the following InnoDB configuration for recovery:  
 xtrabackup: innodb_data_home_dir = ./  
 xtrabackup: innodb_data_file_path = ibdata1:10M:autoextend  
 xtrabackup: innodb_log_group_home_dir = ./  
 xtrabackup: innodb_log_files_in_group = 2  
 xtrabackup: innodb_log_file_size = 5242880  
 InnoDB: Using atomics to ref count buffer pool pages  
 InnoDB: The InnoDB memory heap is disabled  
 InnoDB: Mutexes and rw_locks use GCC atomic builtins  
 InnoDB: Memory barrier is not used  
 InnoDB: Compressed tables use zlib 1.2.7  
 InnoDB: Using CPU crc32 instructions  
 InnoDB: Initializing buffer pool, size = 100.0M  
 InnoDB: Completed initialization of buffer pool  
 InnoDB: Setting log file ./ib_logfile101 size to 5 MB  
 InnoDB: Setting log file ./ib_logfile1 size to 5 MB  
 InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0  
 InnoDB: New log files created, LSN=1607783  
 InnoDB: Highest supported file format is Barracuda.  
 InnoDB: 128 rollback segment(s) are active.  
 InnoDB: Waiting for purge to start  
 InnoDB: 5.6.24 started; log sequence number 1608204  
 xtrabackup: starting shutdown with innodb_fast_shutdown = 1  
 InnoDB: FTS optimize thread exiting.  
 InnoDB: Starting shutdown...  
 InnoDB: Shutdown completed; log sequence number 1608214  
 160901 11:36:37 completed OK!  
 [root@mastera0 mysql]# innobackupex --copy-back /tmp/backup/2016-09-01_11-32-43/  
 160901 11:36:56 innobackupex: Starting the copy-back operation  
  
  
 IMPORTANT: Please check that the copy-back run completes successfully.  
 At the end of a successful copy-back run innobackupex  
 prints "completed OK!".  
  
  
 innobackupex version 2.3.4 based on MySQL server 5.6.24 Linux (x86_64) (revision id: e80c779)  
 160901 11:36:56 [01] Copying ib_logfile0 to /var/lib/mysql/ib_logfile0  
 160901 11:36:56 [01] ...done  
 160901 11:36:56 [01] Copying ib_logfile1 to /var/lib/mysql/ib_logfile1  
 160901 11:36:56 [01] ...done  
 160901 11:36:56 [01] Copying ibdata1 to /var/lib/mysql/ibdata1  
 160901 11:36:56 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/tables_priv.frm to /var/lib/mysql/mysql/tables_priv.frm  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/tables_priv.MYI to /var/lib/mysql/mysql/tables_priv.MYI  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/time_zone.MYD to /var/lib/mysql/mysql/time_zone.MYD  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/ndb_binlog_index.MYD to /var/lib/mysql/mysql/ndb_binlog_index.MYD  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/plugin.MYD to /var/lib/mysql/mysql/plugin.MYD  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/proc.MYI to /var/lib/mysql/mysql/proc.MYI  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/procs_priv.frm to /var/lib/mysql/mysql/procs_priv.frm  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/procs_priv.MYD to /var/lib/mysql/mysql/procs_priv.MYD  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/proxies_priv.MYI to /var/lib/mysql/mysql/proxies_priv.MYI  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/servers.frm to /var/lib/mysql/mysql/servers.frm  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/servers.MYD to /var/lib/mysql/mysql/servers.MYD  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/time_zone.MYI to /var/lib/mysql/mysql/time_zone.MYI  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/time_zone_name.frm to /var/lib/mysql/mysql/time_zone_name.frm  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/time_zone_name.MYI to /var/lib/mysql/mysql/time_zone_name.MYI  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/time_zone_name.MYD to /var/lib/mysql/mysql/time_zone_name.MYD  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/time_zone_transition.frm to /var/lib/mysql/mysql/time_zone_transition.frm  
 160901 11:36:57 [01] ...done  
 160901 11:36:57 [01] Copying ./mysql/time_zone_transition.MYI to /var/lib/mysql/mysql/time_zone_transition.MYI  
 160901 11:36:57 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/time_zone_transition.MYD to /var/lib/mysql/mysql/time_zone_transition.MYD  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/time_zone_transition_type.frm to /var/lib/mysql/mysql/time_zone_transition_type.frm  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/time_zone_transition_type.MYI to /var/lib/mysql/mysql/time_zone_transition_type.MYI  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/time_zone_transition_type.MYD to /var/lib/mysql/mysql/time_zone_transition_type.MYD  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/user.frm to /var/lib/mysql/mysql/user.frm  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/user.MYI to /var/lib/mysql/mysql/user.MYI  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/user.MYD to /var/lib/mysql/mysql/user.MYD  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/time_zone_leap_second.frm to /var/lib/mysql/mysql/time_zone_leap_second.frm  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/time_zone_leap_second.MYI to /var/lib/mysql/mysql/time_zone_leap_second.MYI  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/general_log.frm to /var/lib/mysql/mysql/general_log.frm  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/general_log.CSM to /var/lib/mysql/mysql/general_log.CSM  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/general_log.CSV to /var/lib/mysql/mysql/general_log.CSV  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/slow_log.frm to /var/lib/mysql/mysql/slow_log.frm  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/slow_log.CSM to /var/lib/mysql/mysql/slow_log.CSM  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/slow_log.CSV to /var/lib/mysql/mysql/slow_log.CSV  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/tables_priv.MYD to /var/lib/mysql/mysql/tables_priv.MYD  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/columns_priv.frm to /var/lib/mysql/mysql/columns_priv.frm  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/columns_priv.MYI to /var/lib/mysql/mysql/columns_priv.MYI  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/columns_priv.MYD to /var/lib/mysql/mysql/columns_priv.MYD  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/db.frm to /var/lib/mysql/mysql/db.frm  
 160901 11:36:58 [01] ...done  
 160901 11:36:58 [01] Copying ./mysql/db.MYI to /var/lib/mysql/mysql/db.MYI  
 160901 11:36:58 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/db.MYD to /var/lib/mysql/mysql/db.MYD  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/event.frm to /var/lib/mysql/mysql/event.frm  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/event.MYI to /var/lib/mysql/mysql/event.MYI  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/event.MYD to /var/lib/mysql/mysql/event.MYD  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/func.frm to /var/lib/mysql/mysql/func.frm  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/func.MYI to /var/lib/mysql/mysql/func.MYI  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/func.MYD to /var/lib/mysql/mysql/func.MYD  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_category.frm to /var/lib/mysql/mysql/help_category.frm  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_category.MYI to /var/lib/mysql/mysql/help_category.MYI  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_category.MYD to /var/lib/mysql/mysql/help_category.MYD  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_keyword.frm to /var/lib/mysql/mysql/help_keyword.frm  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_keyword.MYI to /var/lib/mysql/mysql/help_keyword.MYI  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_keyword.MYD to /var/lib/mysql/mysql/help_keyword.MYD  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_relation.frm to /var/lib/mysql/mysql/help_relation.frm  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_relation.MYI to /var/lib/mysql/mysql/help_relation.MYI  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_relation.MYD to /var/lib/mysql/mysql/help_relation.MYD  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_topic.frm to /var/lib/mysql/mysql/help_topic.frm  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_topic.MYI to /var/lib/mysql/mysql/help_topic.MYI  
 160901 11:36:59 [01] ...done  
 160901 11:36:59 [01] Copying ./mysql/help_topic.MYD to /var/lib/mysql/mysql/help_topic.MYD  
 160901 11:36:59 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/host.frm to /var/lib/mysql/mysql/host.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/host.MYI to /var/lib/mysql/mysql/host.MYI  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/host.MYD to /var/lib/mysql/mysql/host.MYD  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/ndb_binlog_index.frm to /var/lib/mysql/mysql/ndb_binlog_index.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/ndb_binlog_index.MYI to /var/lib/mysql/mysql/ndb_binlog_index.MYI  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/plugin.frm to /var/lib/mysql/mysql/plugin.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/plugin.MYI to /var/lib/mysql/mysql/plugin.MYI  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/proc.frm to /var/lib/mysql/mysql/proc.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/proc.MYD to /var/lib/mysql/mysql/proc.MYD  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/procs_priv.MYI to /var/lib/mysql/mysql/procs_priv.MYI  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/proxies_priv.frm to /var/lib/mysql/mysql/proxies_priv.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/proxies_priv.MYD to /var/lib/mysql/mysql/proxies_priv.MYD  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/servers.MYI to /var/lib/mysql/mysql/servers.MYI  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/time_zone.frm to /var/lib/mysql/mysql/time_zone.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./mysql/time_zone_leap_second.MYD to /var/lib/mysql/mysql/time_zone_leap_second.MYD  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./test/db.opt to /var/lib/mysql/test/db.opt  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./performance_schema/db.opt to /var/lib/mysql/performance_schema/db.opt  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./performance_schema/cond_instances.frm to /var/lib/mysql/performance_schema/cond_instances.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./performance_schema/events_waits_current.frm to /var/lib/mysql/performance_schema/events_waits_current.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:00 [01] Copying ./performance_schema/events_waits_history.frm to /var/lib/mysql/performance_schema/events_waits_history.frm  
 160901 11:37:00 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/events_waits_history_long.frm to /var/lib/mysql/performance_schema/events_waits_history_long.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /var/lib/mysql/performance_schema/events_waits_summary_by_instance.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /var/lib/mysql/performance_schema/events_waits_summary_by_thread_by_event_name.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /var/lib/mysql/performance_schema/events_waits_summary_global_by_event_name.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/file_instances.frm to /var/lib/mysql/performance_schema/file_instances.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /var/lib/mysql/performance_schema/file_summary_by_event_name.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/file_summary_by_instance.frm to /var/lib/mysql/performance_schema/file_summary_by_instance.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/mutex_instances.frm to /var/lib/mysql/performance_schema/mutex_instances.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/performance_timers.frm to /var/lib/mysql/performance_schema/performance_timers.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/rwlock_instances.frm to /var/lib/mysql/performance_schema/rwlock_instances.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/setup_consumers.frm to /var/lib/mysql/performance_schema/setup_consumers.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/setup_instruments.frm to /var/lib/mysql/performance_schema/setup_instruments.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/setup_timers.frm to /var/lib/mysql/performance_schema/setup_timers.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./performance_schema/threads.frm to /var/lib/mysql/performance_schema/threads.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./db1/db.opt to /var/lib/mysql/db1/db.opt  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./db1/t1.frm to /var/lib/mysql/db1/t1.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./db2/db.opt to /var/lib/mysql/db2/db.opt  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./db2/t1.frm to /var/lib/mysql/db2/t1.frm  
 160901 11:37:01 [01] ...done  
 160901 11:37:01 [01] Copying ./xtrabackup_info to /var/lib/mysql/xtrabackup_info  
 160901 11:37:01 [01] ...done  
 160901 11:37:02 [01] Copying ./xtrabackup_binlog_pos_innodb to /var/lib/mysql/xtrabackup_binlog_pos_innodb  
 160901 11:37:02 [01] ...done  
 160901 11:37:02 completed OK!  
 [root@mastera0 mysql]#   
 [root@mastera0 mysql]# ll /var/lib/mysql  
 total 28688  
 drwx------. 2 root root 32 Sep 1 11:37 db1  
 drwx------. 2 root root 32 Sep 1 11:37 db2  
 -rw-r-----. 1 root root 18874368 Sep 1 11:36 ibdata1  
 -rw-r-----. 1 root root 5242880 Sep 1 11:36 ib_logfile0  
 -rw-r-----. 1 root root 5242880 Sep 1 11:36 ib_logfile1  
 drwx------. 2 root root 4096 Sep 1 11:37 mysql  
 drwx------. 2 root root 4096 Sep 1 11:37 performance_schema  
 drwx------. 2 root root 19 Sep 1 11:37 test  
 -rw-r-----. 1 root root 38 Sep 1 11:37 xtrabackup_binlog_pos_innodb  
 -rw-r-----. 1 root root 473 Sep 1 11:37 xtrabackup_info  
 [root@mastera0 mysql]# ll /var/lib/mysql -d  
 drwxr-xr-x. 7 mysql mysql 4096 Sep 1 11:37 /var/lib/mysql  
 [root@mastera0 mysql]# chown mysql. /var/lib/mysql/ -R  
 [root@mastera0 mysql]# ll /var/lib/mysql  
 total 28688  
 drwx------. 2 mysql mysql 32 Sep 1 11:37 db1  
 drwx------. 2 mysql mysql 32 Sep 1 11:37 db2  
 -rw-r-----. 1 mysql mysql 18874368 Sep 1 11:36 ibdata1  
 -rw-r-----. 1 mysql mysql 5242880 Sep 1 11:36 ib_logfile0  
 -rw-r-----. 1 mysql mysql 5242880 Sep 1 11:36 ib_logfile1  
 drwx------. 2 mysql mysql 4096 Sep 1 11:37 mysql  
 drwx------. 2 mysql mysql 4096 Sep 1 11:37 performance_schema  
 drwx------. 2 mysql mysql 19 Sep 1 11:37 test  
 -rw-r-----. 1 mysql mysql 38 Sep 1 11:37 xtrabackup_binlog_pos_innodb  
 -rw-r-----. 1 mysql mysql 473 Sep 1 11:37 xtrabackup_info  
 [root@mastera0 mysql]# getenforce  
 Permissive  
 [root@mastera0 mysql]#   
  
  
 [root@mastera0 mysql]# systemctl start maraidb  
 Failed to start maraidb.service: Unit maraidb.service failed to load: No such file or directory.  
 [root@mastera0 mysql]# systemctl start mariadb  
 [root@mastera0 mysql]# mysql -uroot -puplooking  
 Welcome to the MariaDB monitor. Commands end with ; or \g.  
 Your MariaDB connection id is 2  
 Server version: 5.5.44-MariaDB-log MariaDB Server  
  
  
 Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
 MariaDB [(none)]> select * from db1.t1 ;  
 +-----+  
 | id |  
 +-----+  
 | 3 |  
 | 4 |  
 | 5 |  
 | 6 |  
 | 7 |  
 | 8 |  
 | 9 |  
 | 10 |  
 | 11 |  
 | 12 |  
 | 13 |  
 | 14 |  
 | 100 |  
 +-----+  
 13 rows in set (0.00 sec)  
  
  
 MariaDB [(none)]> \q  
 Bye  
 ===========================================  
 # 增量备份并还原  
 [root@mastera0 mysql]# mysql -uroot -puplooking  
 Welcome to the MariaDB monitor. Commands end with ; or \g.  
 Your MariaDB connection id is 4  
 Server version: 5.5.44-MariaDB-log MariaDB Server  
  
  
 Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
 MariaDB [(none)]> select * from db1.t1;  
 +----+  
 | id |  
 +----+  
 | 3 |  
 | 4 |  
 | 5 |  
 | 6 |  
 | 7 |  
 | 8 |  
 | 9 |  
 | 10 |  
 | 11 |  
 | 12 |  
 | 13 |  
 | 14 |  
 +----+  
 12 rows in set (0.00 sec)  
  
  
 MariaDB [(none)]>   
  
  
  
  
 MariaDB [(none)]> insert into db1.t1 values (100);  
 Query OK, 1 row affected (0.13 sec)  
 ## 全备份  
 [root@mastera0 ~]# rm -rf /tmp/backup/*  
 [root@mastera0 ~]# innobackupex --user=root --password=uplooking /tmp/backup  
  
  
 MariaDB [(none)]> select * from db1.t1;  
 +-----+  
 | id |  
 +-----+  
 | 3 |  
 | 4 |  
 | 5 |  
 | 6 |  
 | 7 |  
 | 8 |  
 | 9 |  
 | 10 |  
 | 11 |  
 | 12 |  
 | 13 |  
 | 14 |  
 | 100 |  
 +-----+  
 13 rows in set (0.00 sec)  
  
  
 MariaDB [(none)]> insert into db1.t1 values (101);  
 Query OK, 1 row affected (0.22 sec)  
  
  
 # 增量1  
 [root@mastera0 ~]# innobackupex --user=root --password=uplooking --incremental-basedir=/tmp/backup/2016-09-01_14-07-43/ --incremental /tmp/backup   
  
  
  
  
 MariaDB [(none)]> insert into db1.t1 values (102);  
 Query OK, 1 row affected (0.06 sec)  
  
  
 # 增量2  
 [root@mastera0 ~]# innobackupex --user=root --password=uplooking --incremental-basedir=/tmp/backup/2016-09-01_14-08-35 --incremental /tmp/backup   
  
  
 MariaDB [(none)]> insert into db1.t1 values (103);  
 Query OK, 1 row affected (0.04 sec)  
 # 增量3  
 [root@mastera0 ~]# innobackupex --user=root --password=uplooking --incremental-basedir=/tmp/backup/2016-09-01_14-09-39 --incremental /tmp/backup  
  
  
 ==============  
 # 还原  
 [root@mastera0 mysql]# systemctl stop mariadb  
 [root@mastera0 mysql]# rm -rf /var/lib/mysql/*  
  
  
 [root@mastera0 mysql]# innobackupex --apply-log --redo-only /tmp/backup/2016-09-01_14-07-43  
 [root@mastera0 mysql]# innobackupex --apply-log --redo-only /tmp/backup/2016-09-01_14-07-43 --incremental-dir=/tmp/backup/2016-09-01_14-08-35/  
 [root@mastera0 mysql]# innobackupex --apply-log --redo-only /tmp/backup/2016-09-01_14-07-43 --incremental-dir=/tmp/backup/2016-09-01_14-09-39  
 [root@mastera0 mysql]# innobackupex --apply-log --redo-only /tmp/backup/2016-09-01_14-07-43 --incremental-dir=/tmp/backup/2016-09-01_14-10-09  
 [root@mastera0 mysql]# innobackupex --apply-log /tmp/backup/2016-09-01_14-07-43   
 [root@mastera0 mysql]# innobackupex --copy-back /tmp/backup/2016-09-01_14-07-43   
 [root@mastera0 mysql]# ll /var/lib/mysql  
 total 28688  
 drwx------. 2 root root 32 Sep 1 14:14 db1  
 drwx------. 2 root root 32 Sep 1 14:14 db2  
 -rw-r-----. 1 root root 18874368 Sep 1 14:14 ibdata1  
 -rw-r-----. 1 root root 5242880 Sep 1 14:14 ib_logfile0  
 -rw-r-----. 1 root root 5242880 Sep 1 14:14 ib_logfile1  
 drwx------. 2 root root 4096 Sep 1 14:14 mysql  
 drwx------. 2 root root 4096 Sep 1 14:14 performance_schema  
 drwx------. 2 root root 19 Sep 1 14:14 test  
 -rw-r-----. 1 root root 41 Sep 1 14:14 xtrabackup_binlog_pos_innodb  
 -rw-r-----. 1 root root 550 Sep 1 14:14 xtrabackup_info  
 [root@mastera0 mysql]# chown mysql. /var/lib/mysql -R  
 [root@mastera0 mysql]# ll /var/lib/mysql  
 total 28688  
 drwx------. 2 mysql mysql 32 Sep 1 14:14 db1  
 drwx------. 2 mysql mysql 32 Sep 1 14:14 db2  
 -rw-r-----. 1 mysql mysql 18874368 Sep 1 14:14 ibdata1  
 -rw-r-----. 1 mysql mysql 5242880 Sep 1 14:14 ib_logfile0  
 -rw-r-----. 1 mysql mysql 5242880 Sep 1 14:14 ib_logfile1  
 drwx------. 2 mysql mysql 4096 Sep 1 14:14 mysql  
 drwx------. 2 mysql mysql 4096 Sep 1 14:14 performance_schema  
 drwx------. 2 mysql mysql 19 Sep 1 14:14 test  
 -rw-r-----. 1 mysql mysql 41 Sep 1 14:14 xtrabackup_binlog_pos_innodb  
 -rw-r-----. 1 mysql mysql 550 Sep 1 14:14 xtrabackup_info  
 [root@mastera0 mysql]# systemctl start mariadb  
 [root@mastera0 mysql]# mysql -uroot -puplooking  
 Welcome to the MariaDB monitor. Commands end with ; or \g.  
 Your MariaDB connection id is 2  
 Server version: 5.5.44-MariaDB-log MariaDB Server  
  
  
 Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
 MariaDB [(none)]> select * from db1.t1;  
 +-----+  
 | id |  
 +-----+  
 | 3 |  
 | 4 |  
 | 5 |  
 | 6 |  
 | 7 |  
 | 8 |  
 | 9 |  
 | 10 |  
 | 11 |  
 | 12 |  
 | 13 |  
 | 14 |  
 | 100 |  
 | 101 |  
 | 102 |  
 | 103 |  
 +-----+  
 16 rows in set (0.00 sec)  
  
  
 MariaDB [(none)]> \q  
 Bye  
 ～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～  
 ----------------------  
 晚自习作业  
 1.上网Percona官网、github官网 percona-xtrabackup-2.3.4-1.el7.x86_64.rpm  
 2.通过innobackupex工具做全备份，并还原  
 3.通过innobackupex工具做增量备份，并还原  
 4.制订备份计划，并尝试用脚本完成备份。  
 数据库 172.25.0.15 /var/lib/mysql/*   
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
   
 