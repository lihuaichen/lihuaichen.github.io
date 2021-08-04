---
title: Mysql学习第四课02-冗余--复制AB-replication
date: 2016-09-01 22:27:38
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52403959   
  20160901mysqlday4--2冗余  
 --------------------------  
 > 备份的优点 备份的缺点  
 > 冗余的优点 冗余的缺点  
 恢复速度快 只能解决硬件故障，人为误操作  
  
  
 1.如何构建冗余的环境（一主一从）  
 2.延迟问题  
1）主从同步本身的架构导致  
Msyql 5.5 从1sql线程 延迟问题最严重  
Mysql 5.6 1库1线程 没有解决问题  
Mysql 5.7 1组1线程 GTID   
2）读写分离导致的  
银行适用同步模式  
线上不是金钱的，半同步  
 同步 Wonderwoman-->app-->master---binlog--->I/O--relaylog--sql-->master--->app--->Wonderwoman-->app-->slave  
 异步 Wonderwoman-->app-->master---binlog--->app-->W--->app-->slave  
 --->I/O--relaylog--sql-->  
 半同步/半异步  
 Wonderwoman-->app-->master---binlog--->I/O--relaylog-->master--->app--->Wonderwoman-->app-->slave  
 -------------------------------------  
 db1.t1  
 10个sql1组  
 1sql 2sql 3sql  
 1 t1 1 t2 1  t3 1  
 2 t1 2 t1 10  t1 100  
 3 create t2 t1 11 t1 101  
 4 t1 3  
 5 t1 4  
 ...  
 10 create t3  
 --------------------------------------  
 3.冗余--复制AB-replication  
 1）主从 M-S、M-S-S  
 2）主主 M-M  
 3）多级主从+Multi-source M-M-S-S  
  
  
 4.单主从步骤  
# 主服务器  
1）修改配置文件  vim /etc/my.cnf  
[mysqld]  
log-bin   
server-id=1  
（重启服务）  
2）授权从机  grant replication slave on *.* to slave@172.25.8.12 identified by 'uplooking';  
  
  
3）初始化数据一致    
mysqldump -uroot -puplooking -A --single-transaction --master-data=2 --flush-logs > /tmp/mysql.11.mysql  
---》传输给从机器  
scp mysql.all.sql 172.25.8.12:/tmp  
  
  
# 从服务  
1）install   yum -y install mariadb-server   
2）修改配置文件  server-id=2  
3）初始化数据一致  导入全备数据  
rm -rf /var/lib/mysql/*  
systemctl start mariadb  
mysql </tmp/mysql.all.mysql  
mysql  
>flush privileges;  
4）> change master to master_host='172.25.0.11'  
master_user='slave'  
master_password='uplooking'  
master_log_file=''  
master_log_pos=''  
 ##change master to master_host='172.25.8.11',master_user='slave',master_password='uplooking',master_log_file='mastera.000028',MASTER_LOG_POS=245;  
5)> start slave;  
6)> show slave status\G;  
---------------------------------------------------------------------  
 脚本实现：  
 #!/bin/bash  
 if ! rmp -q mariadb &> /dev/null  
 then  
 yum -y install mariadb-server &>/dev/null  
 fi  
 systemctl stop firewalld  
 ip add |grep dynamic > file.ip  
 A=$(awk -F'/' '{print$1}' file.ip| awk -F'.' '{print$4}')  
 sed -i "2aserver-id=$A" /etc/my.cnf  
 systemctl stop mariadb  
 rm -rf /var/lib/mysql/*  
 systemctl start mariadb  
 mysql </tmp/mysql.all.mysql  
 mysql <<END  
 flush privileges;  
 change master to master_host='172.25.8.11',master_user='slave',master_password='uplooking',master_log_file='mastera.000028',MASTER_LOG_POS=245;  
 start slave;  
 END  
 =======================================================================================================  
 5.双主步骤  
  
 -------------------------------------------------  
 # 详细步骤  
 ## 主服务器  
 [root@mastera0 mysql]# vim /etc/my.cnf  
 [root@mastera0 mysql]# grep -v '^#' /etc/my.cnf|grep -v '^$'  
 [mysqld]  
 datadir=/var/lib/mysql  
 socket=/var/lib/mysql/mysql.sock  
 symbolic-links=0  
 log-bin=/var/lib/mysql-log/mastera  
 server-id=1  
 [mysqld_safe]  
 log-error=/var/log/mariadb/mariadb.log  
 pid-file=/var/run/mariadb/mariadb.pid  
 !includedir /etc/my.cnf.d  
  
  
 [root@mastera0 mysql]# systemctl restart mariadb  
 [root@mastera0 mysql]# mysql -uroot -puplooking  
 Welcome to the MariaDB monitor. Commands end with ; or \g.  
 Your MariaDB connection id is 2  
 Server version: 5.5.44-MariaDB-log MariaDB Server  
  
  
 Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
 MariaDB [(none)]> grant replication slave on *.* to slave@'172.25.0.12' identified by 'uplooking';  
 Query OK, 0 rows affected (0.00 sec)  
  
  
 MariaDB [(none)]> flush privileges;  
 Query OK, 0 rows affected (0.00 sec)  
  
  
 MariaDB [(none)]> \q  
 Bye  
 [root@mastera0 mysql]# mysqldump -uroot -puplooking -A --single-transaction --master-data=2 --flush-logs > /tmp/mysql.91.sql  
 [root@mastera0 mysql]# scp /tmp/mysql.91.sql root@172.25.0.12:/tmp  
 The authenticity of host '172.25.0.12 (172.25.0.12)' can't be established.  
 ECDSA key fingerprint is 91:2b:bd:df:0e:17:da:a0:f6:01:ff:5b:09:50:e8:ad.  
 Are you sure you want to continue connecting (yes/no)? yes  
 Warning: Permanently added '172.25.0.12' (ECDSA) to the list of known hosts.  
 root@172.25.0.12's password:   
 mysql.91.sql 100% 505KB 504.7KB/s 00:00   
  
  
 ## 从服务器  
 [root@masterb0 ~]# yum install -y mariadb-server  
 [root@masterb0 ~]# vi /etc/my.cnf  
 [root@masterb0 ~]# systemctl start mariadb  
 [root@masterb0 ~]# mysql < /tmp/mysql.91.sql   
 [root@masterb0 ~]# mysql   
 Welcome to the MariaDB monitor. Commands end with ; or \g.  
 Your MariaDB connection id is 3  
 Server version: 5.5.44-MariaDB MariaDB Server  
  
  
 Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
 MariaDB [(none)]> flush privileges;  
 Query OK, 0 rows affected (0.00 sec)  
  
  
 MariaDB [(none)]> \q  
 Bye  
 [root@masterb0 ~]# mysql   
 ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)  
 [root@masterb0 ~]# mysql -uroot -puplooking  
 Welcome to the MariaDB monitor. Commands end with ; or \g.  
 Your MariaDB connection id is 5  
 Server version: 5.5.44-MariaDB MariaDB Server  
  
  
 Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.  
  
  
 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
  
 MariaDB [(none)]>   
  
  
  
  
 MariaDB [(none)]> change master to master_host='172.25.0.11',master_user='slave',master_password='uplooking',master_log_file='mastera.000028',MASTER_LOG_POS=245;  
 Query OK, 0 rows affected (0.21 sec)  
  
  
 MariaDB [(none)]> start slave;  
 Query OK, 0 rows affected (0.00 sec)  
  
  
 MariaDB [(none)]> show slave status\G;  
 *************************** 1. row ***************************  
 Slave_IO_State: Waiting for master to send event  
 Master_Host: 172.25.0.11  
 Master_User: slave  
 Master_Port: 3306  
 Connect_Retry: 60  
 Master_Log_File: mastera.000028  
 Read_Master_Log_Pos: 245  
 Relay_Log_File: mariadb-relay-bin.000002  
 Relay_Log_Pos: 527  
 Relay_Master_Log_File: mastera.000028  
 Slave_IO_Running: Yes  
 Slave_SQL_Running: Yes  
 Replicate_Do_DB:   
 Replicate_Ignore_DB:   
 Replicate_Do_Table:   
 Replicate_Ignore_Table:   
 Replicate_Wild_Do_Table:   
 Replicate_Wild_Ignore_Table:   
 Last_Errno: 0  
 Last_Error:   
 Skip_Counter: 0  
 Exec_Master_Log_Pos: 245  
 Relay_Log_Space: 823  
 Until_Condition: None  
 Until_Log_File:   
 Until_Log_Pos: 0  
 Master_SSL_Allowed: No  
 Master_SSL_CA_File:   
 Master_SSL_CA_Path:   
 Master_SSL_Cert:   
 Master_SSL_Cipher:   
 Master_SSL_Key:   
 Seconds_Behind_Master: 0  
 Master_SSL_Verify_Server_Cert: No  
 Last_IO_Errno: 0  
 Last_IO_Error:   
 Last_SQL_Errno: 0  
 Last_SQL_Error:   
 Replicate_Ignore_Server_Ids:   
 Master_Server_Id: 1  
 1 row in set (0.00 sec)  
  
  
 ERROR: No query specified  
  
  
 MariaDB [(none)]>   
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
 | 777 |  
 +-----+  
 17 rows in set (0.00 sec)  
  
  
 MariaDB [(none)]> \q  
 Bye  
  
  
 -------------------------------------------------  
 课后习题  
 1.配置1主多从  
 服务器 ip  作用  
 mastera 172.25.X.11 主服务器   
 masterb 172.25.X.12 从服务器  
 slavea 172.25.X.13 从服务器  
 slaveb 172.25.X.14 从服务器  
 脚本实现：  
 #!/bin/bash  
 if ! rmp -q mariadb &> /dev/null  
 then  
 yum -y install mariadb-server &>/dev/null  
 fi  
 systemctl stop firewalld  
 ip add |grep dynamic > file.ip  
 A=$(awk -F'/' '{print$1}' file.ip| awk -F'.' '{print$4}')  
 sed -i "2aserver-id=$A" /etc/my.cnf  
 systemctl stop mariadb  
 rm -rf /var/lib/mysql/*  
 systemctl start mariadb  
 mysql </tmp/mysql.all.mysql  
 mysql <<END  
 flush privileges;  
 change master to master_host='172.25.8.11',master_user='slave',master_password='uplooking',master_log_file='mastera.000028',MASTER_LOG_POS=245;  
 start slave;  
 END  
  
  
 2.配置双主从  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
   
 