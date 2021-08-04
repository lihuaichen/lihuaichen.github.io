---
title: rhel7如何通过rpm包的方式升级python
date: 2017-11-28 14:04:58
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/78654227   
  ```
测试环境为centos7.2
因某些需要需要把系统的python版本从2.7.5升级到3.6.2，升级方式选择rpm包的方式
注意：python的升级不需要把原本的python卸载，也一定不要卸载，否则会给系统带来严重的问题

1.下载rpm包，通过http://rpm.pbone.net/，搜索相应的包，因为网站属于外国站点，访问速度会比较慢，稍等片刻就行，下面我提供了搜索后提取的下载链接
wget ftp://mirror.switch.ch/pool/4/mirror/centos/7.4.1708/sclo/x86_64/rh/rh-python36/rh-python36-python-3.6.2-3.el7.x86_64.rpm
----------------------------------------------------------------------
2.rpm -ivh rh-python36-python-3.6.2-3.el7.x86_64.rpm
提示安装依赖包
----------------------------------------------------------------------
3.根据提示下载相关的依赖包
wget ftp://mirror.switch.ch/pool/4/mirror/centos/7.4.1708/sclo/x86_64/rh/rh-python36/rh-python36-runtime-2.0-1.el7.x86_64.rpm
wget ftp://mirror.switch.ch/pool/4/mirror/centos/7.4.1708/sclo/x86_64/rh/rh-python36/rh-python36-python-libs-3.6.2-3.el7.x86_64.rpm
wget ftp://mirror.switch.ch/pool/4/mirror/centos/7.4.1708/sclo/x86_64/rh/rh-python36/rh-python36-python-pip-9.0.1-2.el7.noarch.rpm
wget ftp://mirror.switch.ch/pool/4/mirror/centos/7.4.1708/sclo/x86_64/rh/rh-python36/rh-python36-python-setuptools-36.5.0-1.el7.noarch.rpm


[root@localhost ~]# ls   
rh-python36-python-3.6.2-3.el7.x86_64.rpm  
rh-python36-python-libs-3.6.2-3.el7.x86_64.rpm  
rh-python36-python-pip-9.0.1-2.el7.noarch.rpm  
rh-python36-python-setuptools-36.5.0-1.el7.noarch.rpm  
rh-python36-runtime-2.0-1.el7.x86_64.rpm  
-----------------------------------------------------------------
4.rpm -ivh rh*
[root@localhost ~]# rpm -ivh rh*  
警告：rh-python36-python-3.6.2-3.el7.x86_64.rpm: 头V4 RSA/SHA1 Signature, 密钥 ID f2ee9d55: NOKEY  
准备中...                          ################################# [100%]  
正在升级/安装...  
   1:rh-python36-runtime-2.0-1.el7    ################################# [ 20%]  
   2:rh-python36-python-libs-3.6.2-3.e################################# [ 40%]  
   3:rh-python36-python-pip-9.0.1-2.el################################# [ 60%]  
   4:rh-python36-python-setuptools-36.################################# [ 80%]  
   5:rh-python36-python-3.6.2-3.el7   ################################# [100%]  
[root@localhost ~]# python  
Python 2.7.5 (default, Aug  4 2017, 00:39:18)   
[GCC 4.8.5 20150623 (Red Hat 4.8.5-16)] on linux2  
Type "help", "copyright", "credits" or "license" for more information.  
>>> 
版本没有变是因为没有设置环境变量
----------------------------------------------------------------------
5.提示安装成功后，默认安装到opt目录下
如果找不到可以使用find查找
-----------------------------------------------------------------------
6.cat /opt/rh/rh-python36/enable          //环境变量文件，source这个文件会立马生效，如果需要永久生效需要写入到profile文件中。


export PATH=/opt/rh/rh-python36/root/usr/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/opt/rh/rh-python36/root/usr/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export MANPATH=/opt/rh/rh-python36/root/usr/share/man:$MANPATH
export PKG_CONFIG_PATH=/opt/rh/rh-python36/root/usr/lib64/pkgconfig${PKG_CONFIG_PATH:+:${PKG_CONFIG_PATH}}
export XDG_DATA_DIRS="/opt/rh/rh-python36/root/usr/share:${XDG_DATA_DIRS:-/usr/local/share:/usr/share}"
选中复制
---------------------------------------------------------------------
7.vi /etc/profile


将上方的复制到该文件中，保存退出
8.source /etc/profile
测试：
[root@localhost rh-python36]# python --version
Python 3.6.2


升级成功！！！



```
  
   
 