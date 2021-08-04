---
title: Linux-dns服务器
date: 2016-08-31 23:18:45
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390094   
  服务  
1.服务名 守护进程名  
at atd  
cron  crond  
2.基于daemon可以分为两类  
 1) stand alone daemon 独立启动服务  
 2) super daemon 超级服务  
3. stand alone daemon  
 1) 什么是独立启动的服务: 不借助计算机其它资源,独立启动的服务  
 2) 特征:   
1.状态控制脚本存放的位置/etc/init.d目录下,  
 el6:可以使用sevice atd restart 命令去变更服务的状态  
 /etc/init.d/atd restart 去变更服务的状态  
 el7:systemctl restart atd 变更服务状态的方法 -->可以通过man systemctl查看相应的一些帮助.  
  2.程序的初始化配置文件:/etc/sysconfig/  
 3.数据文件 /var  
  4.日志 /var/log,也可以根据程序自身的设定而变更位置.  
5.主配置文件 /etc 以.conf结尾,或者.cf结尾  
6. el6: 进程文件 /var/run 以pid结尾.  
 el7: 进程 /run 以pid结尾  
7.永久占用系统资源,优势在于能够及时响应客户请求.  
8.会有监听端口号,用来区别客户端访问的不同服务.  
1-1023是知名端口号  
1024-65535 随机端口号  
 查看端口号的方法:  
netstat -ntlup n:不做解析  
t:tcp  
l:监听的端口号  
u:udp  
p:协议  
4.super daemon --> xinetd服务  
用来唤醒其它服务的程序  
  
  
  
  
======================================================  
DNS  
1.作用:用来解析主机名和IP地址 和/etc/hosts文件类似  
2.实现机制:分层式结构  
 . 根域  
 com cn org edu 二级域  
 baidu sina 子域  
 www 主机  
全球共有13台根域  
3.fqdn 完全合格的域名=主机名+域名  
www.baidu.com.   
4.查找顺序   
优先读取本地的/etc/hosts文件,再去读取dns  
由/etc/nsswitch.conf定义先后顺序  
DNS的查询方式 1) 递归查询 A -->B -->C  
 A <--B <--C   
 2) 迭代查询  
A -->B   
A C<--B  
A -->C  
A <--C0  
5./etc/resolv.conf DNS服务器指向文件,找谁来帮我做解析.  
nameserver 172.25.254.250  
  
  
==============================================  
DNS服务的软件  
1.bind DNS服务器的主程序.   
2.bind-chroot 出于安全性考虑,用来挂机目录和配置文件的辅助程序  
===============================================  
配置文件  
1.主配置文件 /etc/named.conf  
2.zone的定义文件/etc/named.rfc1912.zones  
3.数据文件 /var/named目录下,由zone的定义文件file字段决定文件名称  
  
  
  
  
===============================================  
实验  
1.主配置文件 vim /etc/named.conf  
 listen-on port 53 { any; };  
 allow-query { any; };  
2.定义域 vim /etc/named.rfc1912.zones  
zone "abc.com" IN {  
type master;  
file "abc.com.zone";  
allow-update { none; };  
};  
3.生成数据文件  
cd /var/named  
cp -p named.localhost abc.com.zone  
vim abc.com.zone  
NS @ --> @代表继承域名  
A 172.25.0.11 -->   
www A 172.25.0.254 --> www.abc.com是172.25.0.254主机  
MX 5 mail  
mail A 172.25.0.253 --> mail服务器必须要指定MX优先级 .mail服务器是172.25.0.253  
ftp CNAME bbs  --> CNAME代表别名,ftp又叫做bbs  
bbs A 172.25.0.252 --> bbs是172.25.0.252  
  
  
4.启动服务.  
service named restart   
5. vim /etc/resolv.conf 指定谁来做解析  
nameserver 172.25.0.11  
6. 检测 nslookup  
  
  
如果发现在service named restart的时候rndc.key这里启不来  
rndc-confgen -a -r /etc/named.conf  
  
  
反解  
1.主配置文件  
2.定义域 vim /etc/named.rfc1912.conf  
zone "0.168.192.in-addr.arpa" IN {  
 type master;  
 file "test.arpa";  
 allow-update { none; };  
};  
3.生成数据文件  
cd /var/named  
cp -p named.localhost test.arpa  
NS test.com.  
254 PTR test.com.  
253 PTR www.test.com.  
252 PTR mail.test.com.  
251 PTR ftp.test.com.  
4.重启服务  
5.vim /etc/resolv.conf 指定谁来做解析  
nameserver 172.25.0.11  
6.nslookup  
  
  
=================================  
  
  
DNS主辅同步  
  
  
1.作用:  
1)安全性  
2)负载均衡  
2.实验  
master机器上的配置  
 1) 变更zone 定义  
 allow-transfer { 172.25.0.10; }; 允许谁来和我做同步  
 2) 变更数据文件  
$TTL 1D --> 缓存周期1D代表一天  
@ IN SOA @ 用户名.域名#这个不改也没关系 ( --> SOA 起始授权记录  
 20160106 ; serial -->序列号,比对工具  
 1D ; refresh -->多久做一次同步  
 1H ; retry -->同步失败后,多久重试  
 1W ; expire -->同步失败后,多久失效  
 3H ) ; minimum -->最小缓存时间  
slave机器上的配置:  
 1) 装包  
 2) 主配置文件修改  
listen-on port 53 { any; };  
allow-query { any; };  
 3) 定义域  
zone "test.com" IN {  
 type slave;  
 masters { 172.25.0.11; };  
 file "slaves/test.com.zone";  
 allow-update { none; };  
};  
zone "0.168.192.in-addr.arpa" IN {  
 type slave;  
 masters { 172.25.0.11; };  
 file "slaves/test.arpa";  
 allow-update { none; };  
};  
  
  
关防火墙的命令 service iptables stop  
systemctl stop firewalld  
排错思路  
1.重启服务看报错  
2.看日志  
 1)/var/log/messages -->tail -f messages 去重启服务,看重启服务过程当中出了什么问题  
 2) /var/named/data/named.run 程序的日志.  
3.权限  
 1) 程序权限  
 2) ugo权限  
 3) selinux权限 -->通过分析selinux的日志来获取相应的信息.使用sealert工具分析  
setsebool -P named_write_master_zones 1    
 