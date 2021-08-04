---
title: Linux-lvm
date: 2016-08-31 23:19:30
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390102   
  ===========================================  
  
  
创建LVM  
  
  
  
  
 1.创建物理卷 分区 (逻辑分区|主分区)   
 pvcreate /dev/vdb1   
 pvcreate /dev/vdb2 /dev/vdb3   
 pvs pvdisplay   
  
  
 2.创建卷组   
 vgcreate vgtest /dev/vdb1 /dev/vdb2 /dev/vdb3   
 vgs vgdisplay   
  
  
 3.创建逻辑卷   
 lvcreate -n lvtest -L 400M vgtest   
 lvs lvdisplay   
 lvcreate -n lvkevin -l 50 vgtest   
 -L KMG指定大小 -l 通过块的数量来指定大小   
 lvcreate -n lvmark -l 100%FREE vgtest   
 4.创建文件系统 挂载   
 mkfs.ext4 /dev/vgtest/lvcarol   
 mkfs.ext4 /dev/mapper/vgtest-lvcarol   
 mount /dev/vgtest/lvcarol /mnt   
======================================  
  
  
PE的大小定制  
  
  
 vgcreate -s 8M vgcarol /dev/vdb5  
 vgchange -s 2M vgcarol  
  
  
===============================  
扩展  
  
  
  
  
1.lv的扩展 (对应vg空间足够)  
  
  
 lvextend -L 500M /dev/vgweb/lvweb --> 扩展至500M  
 lvextend -L +300M /dev/vgweb/lvweb --> 扩展300M   
  
  
  
  
2.文件系统的扩展   
ext  
 resize2fs /dev/mapper/vgweb-lvweb  
xfs  
 xfs_growfs /web  
  
  
3.vg的扩展   
 vgextend vgweb /dev/vdb9  
  
  
 lvextend -L 1.5G /dev/vgweb/lvweb  
resize2fs /dev/mapper/vgweb-lvweb  
  
  
==============================================  
  
  
缩小  
  
  
  
  
1.文件系统的缩小  
xfs不允许  
ext可以  
 umount /lvtest  
 e2fsck -f /dev/mapper/vgweb-lvweb  
 resize2fs /dev/mapper/vgweb-lvweb 800M  
  
  
2.lv缩小  
 lvreduce -L 800M /dev/vgweb/lvweb  
 lvreduce -L -300M /dev/vgweb/lvweb   
 lvreduce -f -L -200M /dev/vgweb/lvweb   
  
  
3.vg缩小   
 vgreduce vgweb /dev/vdb9  
  
  
=================================================  
  
删除  
1.卸载  
umount   
2.删除lv  
lvremove -f /dev/vgcarol/lvcarol1  
3.删除vg  
vgremove vgweb  
4.删除pv  
pvremove /dev/vdb9  
5.删除分区   
fdisk d  
  
  
=================================================  
  
pv移动  
  
  
前提:移动前后的pv存放在同一个vg当中  
建议移动前后pv大小保持一致  
pvmove /dev/vdb1 /dev/vdb7  
 | !  
 |  |  
 - - - - - -   
  
  
===============================================   
 