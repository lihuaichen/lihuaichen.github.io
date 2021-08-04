---
title: clusterinit初始化环境脚本
date: 2016-10-06 16:35:44
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52744308   
  ```
<pre name="code" class="html">#!/bin/bash

[ -z $1 ] && echo input hostname && exit


myname=$1

ret=0

if [[ $myname =~ ^node[1234]-f[0-9]*\.example\.com ]]
then
	ret=1
fi

if [[ $myname =~ ^lvs[12]-f[0-9]*\.example\.com ]]
then
        ret=1
fi



[ $ret -ne 1 ] && echo wrong hostname && exit

service NetworkManager stop
chkconfig NetworkManager off


fd=$(echo $myname | awk -F"-f" '{ print $2 }' | awk -F"." '{ print $1 }')


hostname $myname
sed -i "s/HOSTNAME=.*/HOSTNAME=$myname/" /etc/sysconfig/network

cat > /etc/hosts << ENDF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.$fd.10 node1-f$fd.example.com
172.25.$fd.11 node2-f$fd.example.com
172.25.$fd.12 node3-f$fd.example.com
172.25.$fd.13 node4-f$fd.example.com
172.25.$fd.14 lvs1-f$fd.example.com
172.25.$fd.15 lvs2-f$fd.example.com
ENDF

iptables -F
service iptables save

setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

myip=$(awk -v name=$myname '{ if ( $2 ~ name ) { print $1 } }' /etc/hosts)

cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << ENDF
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME="System eth0"
IPADDR=$myip
NETMASK=255.255.255.0
GATEWAY=172.25.$fd.254
ENDF

cat > /etc/sysconfig/network-scripts/ifcfg-eth2 << ENDF

DEVICE=eth2
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME="System eth2"
IPADDR=192.168.122.4${myip##*.1}
NETMASK=255.255.255.0

ENDF

cat > /etc/sysconfig/network-scripts/ifcfg-eth1 << ENDF
DEVICE=eth1
TYPE=Ethernet
ONBOOT=no
NM_CONTROLLED=no
BOOTPROTO=none
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no

ENDF

cat > /etc/yum.repos.d/cluster.repo << ENDF
[HighAvailability]
name = rhel6.5 repos HighAvailability
baseurl = http://classroom.example.com/content/rhel6.5/x86_64/dvd/HighAvailability
enable=1
gpgcheck=0

[LoadBalancer]
name = rhel6.5 repos LoadBalancer
baseurl = http://classroom.example.com/content/rhel6.5/x86_64/dvd/LoadBalancer
enable=1
gpgcheck=0

[ResilientStorage]
name = rhel6.5 repos ResilientStorage
baseurl = http://classroom.example.com/content/rhel6.5/x86_64/dvd/ResilientStorage
enable=1
gpgcheck=0

[ScalableFileSystem]
name = rhel6.5 repos ScalableFileSystem
baseurl = http://classroom.example.com/content/rhel6.5/x86_64/dvd/ScalableFileSystem
enable=1
gpgcheck=0

ENDF



```
  
  

```

```
   
 