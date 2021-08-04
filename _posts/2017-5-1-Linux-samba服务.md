---
title: Linux-samba服务
date: 2016-08-31 23:23:10
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390131   
  1.samba 作用：windows unix 共享  
2.服务端 samba samba-common  
 客户端 samba samba-common samba-client  
服务端的主配置 /etc/samba/smb.conf  
两种1.本地用户  
用户名 /etc/passwd 密码不是/etc/shadow  
密码是samba自身维护的密码  
  
  
默认家目录共享目录，能够访问其他共享目录  
  
  
 2.匿名用户  
服务端起服务service smb restart 445 139  
 service nmb restart 137 138  
  
  
客户端：  
samba samba-common samba-client  
一：查看远程主机提供的共享名称  
1.匿名用户  
smbclient -L 172.25.0.11  
2.本地用户 需要服务端赋予密码  
服务端：  
第一次赋予密码：smbpasswd -a student  
后期修改密码 smbpasswd student 即可，但是要求知道第一次密码  
客户端：smbclient -L 172.25.0.11 -U student   
二：登陆  
smbclient //172.25.0.11/student -U student  
 //远程主机/共享名   
smbclient //172.25.0.11/test  
setsebool -P samba_export_all_rw on  
三：上传  
1.匿名用户  
ugo : nobody  
2.本地用户  
tom tom  
student student  
  
  
  
  
  
  
tom student alice nobody --> 组 --> acl  
  
  
server端共享的配置  
[root@rhel6 smbshare]# tail -n5 /etc/samba/smb.conf   
[test] #共享名  
comment = testing #说明  
path = /smbshare #共享目录  
public = yes #是否所有人可访问  
writable = yes#是否所有人可写  
  
  
======================================  
题目 el6 server el7 client  
共享名：public  
共享目录: /public  
要求：匿名可以上传文件，student用户可以上传文件 UGO权限或者acl权限去满足。  
selinux -> enforcing   
  
  
mount 以本地用户的方式将public挂接到 /mnt  
  
  
  
  
=================================================================  
通过mount来访问远程主机   
yum -y install cifs-utils  
1.匿名用户挂载  
mount -t cifs //172.25.0.11/test /mnt -o guest  
2.本地用户挂载  
mount -t cifs //172.25.0.11/test /mnt -o username=tom  
  
  
fstab  
1.匿名用户  
//172.25.0.11/test /mntcifs guest 0 0  
2.本地用户  
1) //172.25.0.11/test /mntcifs cred=/etc/samba.txt 0 0  
2) vim /etc/samba.txt   
username=tom  
password=tom  
3) chmod 400 /etc/samba.txt  
  
  
  
  
  
  
  
  
windows登陆  
运行  
\\172.25.0.11\test  
   
 