---
title: CentOS下VNC配置和安装
date: 2017-11-28 15:32:21
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/78655663   
  CentOS下VNC配置和安装  
 [日期：2013-05-07] 来源：CSDN 作者：rentiansheng  [字体：大 中 小]  
  
  
 文档声明：CentOS5.4的桌面版上的安装过程。本文仅是个人的安装过程，其中软件的配置方案仅是根据个人需要选择，可能无法满足所有的需要。其中的配置也可能因为个人能力问题造成配置是错误、不是最优或者配置了多余的选项，请大家谅解。  
   
 端口，  
 端口默认是从5900开始，再加上你的桌面号。  
 比如你的桌面号为1，则vnc的连接端口号为5900+1=5901  
 比如你的桌面号为10000，则vnc的连接端口号为5900+10000=15900  
 ======================================================================  
 下面配置VNC服务器，使用户（root）能够通过vnc客户端远程连接到linux系统的图形界面(前提是你的服务器要安装桌面)  
 1. 检查linux系统是否安装VNC  
 rpm -q vnc-server  
   
 出现 “package vnc-server is not installed”说明vnc服务器没有安装  
 如果没有安装使用下面命令安装vnc  
   
 yum install vnc-server  
   
 2. 启动vnc服务  
   
 vncserver  
   
 You will require a password to access yourdesktops.  
   
 Password:  
   
 Verify:  
   
 会提示输入密码，这个密码是远程登录时所需要输入的密码，输入密码，回车  
   
 4、切换到root账号:su root然后输入root账号的密码   
   
  
  
 vim /etc/sysconfig/vncservers #vnc配置文件  
   
 将下面两行注释去掉。  
   
 VNCSERVERS="1:root" # 1:root （桌面号:用户），配置启动一号桌面  
   
 VNCSERVERARGS[2]="-geometry 1204x768-nolisten tcp -localhost"  
  
  
 // 800x600表示桌面的分辨率  
   
 最后保存退出  
   
 5、重启vnc服务器   
   
 方法一：etc/init.d/vncserver restart  
   
 方法二：service vncserver restart  
   
 6、 设置vnc服务器开机自动启动  
 vi/etc/rc.local  
   
 在文件中添加下面内容  
   
 /etc/init.d/vncserverstart  
   
 7、更改vnc连接密码  
 vncpasswd   
   
   
 8、连接远程桌面   
   
   
 使用SecureCRT连接到目标机器。  
   
 执行iptables –F命令，  
   
 然后使用VNC Viewer连接即可   
   
  
  
 在远程桌面使用接受后，重启防火墙就可以进制远程vnc连接。   
   
 注意：如果在连接上之后，出现灰屏，可以按照下面的方法设置   
   
 进入用户的home目录  
   
 cd /home/user  
   
 如果是用root账号登录的，那么当前目录就是用户根目录  
   
 cd ~/.vnc  
 vi xstartup #编辑  
   
 #twm & #注释掉这一行  
   
 gnome-session & #添加这一行   
 