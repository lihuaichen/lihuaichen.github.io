---
title: linux下4种kill某个用户所有进程的方法
date: 2018-11-14 14:46:11
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/84066359   
  linux下4种kill某个用户所有进程的方法  
   
 这篇文章主要介绍了linux下4种kill某个用户所有进程的方法,需要的朋友可以参考下  
 在linux系统管理中，我们有时候需要kill掉某个用户的所有进程，初学者一般先查询出用户的所有pid，然后一条条kill掉，或者写好一个脚本，实际上方法都有现成的，这边有4种方法，我们以kill用户ttlsa为例.  
 1. pkill方式  
 # pkill -u ttlsa  
 2. killall方式  
 # killall -u ttlsa  
 3. ps方式  
 ps列出ttlsa的pid，然后依次kill掉，比较繁琐.  
 # ps -ef | grep ttlsa | awk '{ print $2 }' | sudo xargs kill -9  
 4. pgrep方式  
 pgrep -u参数查出用户的所有pid，然后依次kill  
 # pgrep -u ttlsa | sudo xargs kill -9

   
 