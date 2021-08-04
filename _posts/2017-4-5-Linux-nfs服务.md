---
title: Linux-nfs服务
date: 2016-08-31 23:20:14
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390108   
  1.nfs  
网络文件系统 -->适用范围 unix unix共享的访问方式  
原理：nfs将对应端口号注册给本地rpc，客户端通过111端口链接到rpc获取相应的nfs注册的随即端口号  
注意：服务端rpc重启了，nfs必须重启。  
 nfs重启，rpc不需要重启  
2.软件  
rpc --> rpcbind --> 服务名称 rpcbind  
nfs --> nfs-utils 驱动 客户端|服务端都要进行安装 --> nfs  
服务端 --> setup --> /etc/exports nfs的主配置文件  
3./etc/exports 配置方法  
共享目录 主机定位(选项)  
  
  
需求：el6 server el7 client  
server /share 172.25.0.10 只读  
==============================  
题目：el6 server el7 client  
server /nfsshare 172.25.x.0/24 172.25.0.11 只读  
client 挂载到/mnt 目录下  
==============================  
权限问题1.如果要赋予客户端uid=1000的用户rwx，那么实际上就是赋予服务器uid=1000的用户rwx权限，用户名称不一致也无所谓。  
问题2.默认情况下，客户端以root用户身份访问时，在服务端会被映射成nfsnobody（uid=65534,gid=65534)用户。  
/share 172.25.0.0/24(rw,no_root_squash) 不做root用户映射  
3.all_squash 全部用户作映射 默认情况下nfsnobody  
4.主机的定位(rw,all_squash,anonuid=150,anongid=100) anonuid 映射的uid号码 anongid 映射的gid号码  
  
  
=============================  
题目：el6 server el7 client  
server /nfsshare 所有用户映射成nfsnobody，rw 172.25.x.0/24  
/share 所有用户不映射 rw 172.25.x.0/24  
client 将以上两个目录开机自动挂载，/nfsshare /mnt   
 /share/media  
  
  
  
  
  
  
========================================  
autofs的配置   
[root@rhel7 /]# yum -y install autofs ^C  
[root@rhel7 /]# grep nfs /etc/auto.master  
/srv /etc/auto.nfs  
[root@rhel7 /]# cp /etc/auto.misc /etc/auto.nfs ^C  
[root@rhel7 /]# tail -n1 /etc/auto.nfs   
test -fstype=nfs172.25.0.11:/share  
[root@rhel7 /]# systemctl restart autofs  
[root@rhel7 /]# cd /srv/  
[root@rhel7 srv]# ls  
[root@rhel7 srv]# cd test  
[root@rhel7 test]# df -h  
Filesystem Size Used Avail Use% Mounted on  
/dev/mapper/vg--rhel7-root 18G 3.0G 15G 18% /  
devtmpfs 488M 0 488M 0% /dev  
tmpfs 498M 80K 497M 1% /dev/shm  
tmpfs 498M 6.9M 491M 2% /run  
tmpfs 498M 0 498M 0% /sys/fs/cgroup  
/dev/mapper/vg--rhel7-home 497M 26M 472M 6% /home  
/dev/vda1 497M 119M 379M 24% /boot  
172.25.0.11:/share 17G 2.9G 14G 18% /srv/test  
  
  
5分钟不使用则自动卸载  
[root@rhel7 /]# grep TIMEOUT /etc/sysconfig/autofs | grep -v ^#  
TIMEOUT=300  
==========================================  
题目：  
EL7 NFS-CLIENT  
远程/nfsshare 自动挂载/test/nfsshare  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
   
 