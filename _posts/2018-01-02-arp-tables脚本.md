---
title: arp-tables脚本
date: 2016-10-06 16:37:03
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52744322   
  ```
#!/bin/bash

VIP=172.25.0.100
VIPCAST=172.25.0.255
RIP=192.168.88.80
DGW=172.25.0.254
DGWMAC="52:54:00:00:00:fe"

arptables -F
arptables -A IN -d $VIP -j DROP
arptables -A OUT -s $VIP -j mangle --mangle-ip-s $RIP

/sbin/ifconfig eth0:1 $VIP broadcast $VIPCAST netmask 255.255.255.0 up
arp -s $DGW $DGWMAC

/sbin/route add default gw $DGW

```
  
   
 