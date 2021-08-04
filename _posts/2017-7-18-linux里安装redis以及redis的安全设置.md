---
title: linux里安装redis以及redis的安全设置
date: 2018-06-06 15:48:42
tags: CSDN迁移
---
  发表此篇文章是由于redis有一个小白式错误：redis默认匿名访问，导致redis安全性堪忧。下面从redis安装说起：  


安装Redis: 请参考这儿;[https://redis.io/download](https://redis.io/download)

[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. <span style="font-family:Verdana;font-size:12px;color:#666666;">$ wget http://download.redis.io/releases/redis-3.2.7.tar.gz 
  2. $ tar xzf redis-3.2.7.tar.gz 
  3. $ cd redis-3.2.7 
  4. $ make 
  5. 
  6. $ src/redis-server</span>   






ps :如果以上有报错，可能是你的服务器没有安装依赖：

CentOS7：

[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. yum install -y gcc-c++ tcl   


安装完成后

在目录 redis-3.2.7中有一个redis.conf的配置文件，按照默认习惯我们将其复制到/etc目录下：

[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. [root@MyCloudServer ~]# cp redis-3.2.7/redis.conf /etc   


PS：请使用复制（cp）而不要使用移动（mv）；毕竟你要弄错了还可以再拷贝一份儿过去用不是？

使用vim编辑刚刚拷贝的redis.conf

[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. vim /etc/redis.conf   


PS:使用vim需要先安装：

CentOS7：

[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. yum install vim   
我们需要注意以下几项：

1.注释掉47行的bind 127.0.0.1（这个意思是限制为只能 127.0.0.1 也就是本机登录）PS：个人更建议 将你需要连接Redis数据库的IP地址填写在此处，而不是注释掉。这样做会比直接注释掉更加安全。  


2.更改第84行port 6379 为你需要的端口号（这是Redis的默认监听端口）PS：个人建议务必更改

3.更改第128行 daemonize no 为 daemonize yes（这是让Redis后台运行） PS:个人建议更改  


4.取消第 480 # requirepass foobared 的#注释符（这是redis的访问密码） 并更改foobared为你需要的密码 比如 我需们需要密码为123456 则改为 requirepass 123456。PS：密码不可过长否则Python的redis客户端无法连接  


以上配置文件更改完毕，需要在防火墙放行：

[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. firewall-cmd --zone=public --add-port=xxxx/tcp --permanent   
请将xxxx更改为你自己的redis端口。

重启防火墙生效：

[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. systemctl restart firewalld.service   
指定配置文件启动redis:  
[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. [root@MyCloudServer ~]# redis-3.2.7/src/redis-server /etc/redis.conf   
加入到开机启动:[plain] [view plain](https://blog.csdn.net/Prety_Boy/article/details/56487187#) [copy](https://blog.csdn.net/Prety_Boy/article/details/56487187#)  
  
  
  

  1. echo "/root/redis-3.2.6/src/redis-server /etc/redis.conf" >> /etc/rc.local   
一个较为安全的redis配置完毕。

redis的桌面客户端我推荐：RedisDesktopManager

去下面这个地址下载：[https://github.com/uglide/RedisDesktopManager/releases](https://github.com/uglide/RedisDesktopManager/releases)

   
 