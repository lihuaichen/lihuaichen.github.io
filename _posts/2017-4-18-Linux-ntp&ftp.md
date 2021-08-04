---
title: Linux-ntp&ftp
date: 2016-08-31 23:21:05
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390117   
  ntp  
1.作用:同步时间.  
2.原理:分层式结构.  
3.软件:el6:ntp  
 el7:chrony  
4.el6主配置文件 /etc/ntp.conf  
restrict 172.25.0.0 mask 255.255.255.0 -->允许谁来和我做同步  
server 172.25.0.10 -->我要找谁做同步  
  
  
特殊的表达方式 127.127.1.0 代表的是本地系统时间.  
和本地时间同步,指定层数的方式:  
server 127.127.1.0 fudge  
127.127.1.0 stratum 10  
  
  
在客户端上,使用ntpdate -u 172.25.0.11来同步时间  
当我们重启ntp服务后,需要等5-10分钟才能够被同步成功.  
5. el7 主配置文件 /etc/chrony.conf  
allow 172.25.0/24 --> 允许谁来和我做同步  
server -->和谁做同步  
  
  
和本地时间做同步,指定层数的方式  
server 172.25.0.10  
local stratum 10  
在客户端上,使用ntpdate -u 172.25.0.10来和服务端做同步  
6.启服务  
el6 service ntpd restart  
el7 systemctl restart chronyd.service  
  
  
ftp  
1.作用:共享文件  
2.软件 vsftpd  
3.工作原理:  
主动模式: 服务端21号端口处理链接请求,通过20号端口向客户端传输数据.  
被动模式: 服务端通过21端口处理链接请求,随后开启一个随即端口号通知客户端从该端口号获取数据.  
4.vsftpd支持的用户类型:匿名用户和本地用户  
匿名用户:在ftp服务器中没有指定账户,但是能访问ftp服务器相应资源的用户.  
本地用户:/etc/passwd用户  
5.vsftpd结构  
 1) 主配置文件/etc/vsftpd/vsftpd.conf  
 2) 数据文件 /var/ftp/pub 目录  
6.访问方式  
 1) 匿名访问 [1] 浏览器ftp://172.25.0.11/pub/  
 lftp工具 -->对应的软件名:lftp  
 [2] lftp ip地址   
[root@rhel7 ~]# lftp 172.25.0.11  
lftp 172.25.0.11:~> ls   
drwxr-xr-x 2 0 0 4096 Jan 06 06:43 pub  
lftp 172.25.0.11:/> cd pub/  
lftp 172.25.0.11:/pub> ls  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file1  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file10  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file2  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file3  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file4  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file5  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file6  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file7  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file8  
-rw-r--r-- 1 0 0 0 Jan 06 06:43 file9  
lftp 172.25.0.11:/pub> exit  
 2) 本地用户访问 lftp工具:  
 对于本地用户来说,它的共享目录是他的家目录.  
lftp testuser@172.25.0.11   
[root@rhel7 ~]# lftp testuser@172.25.0.11  
Password:   
lftp testuser@172.25.0.11:~> ls   
ls: Login failed: 500 OOPS: cannot change directory:/home/testuser  
lftp testuser@172.25.0.11:~>   
  
  
读取不到家目录下文件的原因是selinux造成的  
需要打开setsebool -P ftp_home_dir 1  
  
  
7.下载  
 lftp登陆以后,使用get去下载  
 lftp 172.25.0.11:/pub> get file1  
 下载的位置是在你当前位置.  
  
  
8.上传  
 本地用户的上传: lftp 登陆以后,使用put去上传  
lftp testuser@172.25.0.11:~> put testuserfile   
可以通过绝对路径上传我们当前位置所在目录下的文件  
 匿名用户的上传:  
 1) 程序限制   
vim /etc/vsftpd/vsftpd.conf  
打开anon_upload_enable=YES   
 2) UGO 给ftp用户进入目录所用的rwx权限.  
 3) selinux权限  
 semanage fcontext -a -t public_content_rw_t pub -->注意路径 /var/ftp/pub  
 restorecon -R -v pub -->针对的是 /var/ftp/pub  
 setsebool -P allow_ftpd_anon_write 1  
9.黑白名单  
黑名单:/etc/vsftpd/ftpusers  
  
  
在主配置文件里面有一行参数:userlist_enable=YES  
如果参数是YES,则代表/etc/vsftpd/user_list是黑名单.  
如果参数是NO,则代表/etc/vsftpd/user_list是白名单.  
如果没有该行配置,默认参数是NO.   
man 5 vsftpd.conf   
 