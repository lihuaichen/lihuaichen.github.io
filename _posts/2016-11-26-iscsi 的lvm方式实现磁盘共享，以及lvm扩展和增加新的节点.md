---
title: iscsi 的lvm方式实现磁盘共享，以及lvm扩展和增加新的节点
date: 2016-10-12 22:10:04
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52801252   
  环境：  
 target:  
storage1:172.25.15.10  
 initiator:  
iscsi1:172.25.15.14  
iscsi2:172.25.15.15  
 用iscis方式实现共享存储   
  
  
 使用 iscsi-target 程序导出 iSCSI 设备需要完成如下步骤:  
 1. 下载 iscsi-target 程序,因为 RedHat 系统并没有自带这个程序的 rpm 包  
 2. 安装编译 iscsi-target 程序所需要的 gcc 、make 、kernel-devel 等编译环境 rpm 包  
 3. 编译 iscsi-target 并将其安装在系统中  
 4. 确定 iSCSI 导出的本地分区或本地存储设备已经就位  
 5. 编辑/etc/ietd.conf 配置文件,指定导出 iSCSI iqn 名和 LUN ID ,确定验证方式和  
 验证用用户名密码  
 6. 在/etc/initiator.allow 和/etc/initiator.deny 里设置访问控制规则  
 7. 启动/etc/init.d/iscsi-target 服务,导出 iSCSI 设备。  
  
  
 实验过程：  
 *******************************************************************  
1.开机自动挂载  
 *******************************************************************  
 [root@iscsi1-f15 ~]# ls /dev/sd*  
 /dev/sda /dev/sdb /dev/sdb1  
 [root@iscsi1-f15 ~]# fdisk /dev/sda #在sda上划分一个新的分区  
 [root@iscsi1-f15 ~]# ls /dev/sd*  
 /dev/sda /dev/sda1 /dev/sdb /dev/sdb1  
 [root@iscsi1-f15 ~]# makedir /iscsi  
 [root@iscsi1-f15 /]# mkfs.ext4 /dev/sda1  
 [root@iscsi1-f15 /]# blkid #查看UUID  
 /dev/mapper/VolGroup-lv_root: UUID="1ea6100d-d626-4ec4-909e-ad13df6912f1" TYPE="ext4"   
 /dev/vda1: UUID="583b5a56-a0c7-4e5e-82e2-6aa625a902c5" TYPE="ext4"   
 /dev/vda2: UUID="QtRr5i-Sb9q-S3a9-8lx8-IWrN-CD07-jd0ZyA" TYPE="LVM2_member"   
 /dev/mapper/VolGroup-lv_swap: UUID="b646bc01-35ce-49fe-b375-ae34f464c6bc" TYPE="swap"   
 /dev/sdb1: LABEL="storage-f15:test" UUID="0efa1906-cbf6-a68d-de1f-26f6c0e878ae" TYPE="gfs2"   
 /dev/sda1: UUID="684fff71-a24b-4042-af80-86f1ab6dba7a" TYPE="ext4"   
 [root@iscsi1-f15 ~]# vim /etc/fstab #设置开机挂载  
 [root@iscsi1-f15 ~]# cat /etc/fstab   
 /dev/mapper/VolGroup-lv_root / ext4 defaults 1 1  
 UUID=583b5a56-a0c7-4e5e-82e2-6aa625a902c5 /boot ext4 defaults 1 2  
 /dev/mapper/VolGroup-lv_swap swap swap defaults 0 0  
 tmpfs /dev/shm tmpfs defaults 0 0  
 devpts /dev/pts devpts gid=5,mode=620 0 0  
 sysfs /sys sysfs defaults 0 0  
 proc /proc proc defaults 0 0  
 UUID=684fff71-a24b-4042-af80-86f1ab6dba7a /mnt ext4 _netdev 0 0 #添加  
 UUID=0efa1906-cbf6-a68d-de1f-26f6c0e878ae /iscsi gfs2 defaults 0 0 #添加  
 [root@iscsi1-f15 ~]# sync   
 主机通过poweroff关闭iscsi1  
 重启后：  
 [root@iscsi1-f15 ~]# df -h   
 Filesystem Size Used Avail Use% Mounted on  
 /dev/mapper/VolGroup-lv_root 8.4G 2.0G 6.0G 25% /  
 tmpfs 246M 32M 215M 13% /dev/shm  
 /dev/vda1 485M 34M 426M 8% /boot  
 /dev/sda1 1011M 34M 926M 4% /mnt  
 /dev/sdb1 2.0G 259M 1.8G 13% /iscsi  
  
  
 可以看出sda和sdb实现了开机自动挂载  
 *******************************************************  
2.ACL  
 ********************************************************  
 上述的做法，只要是与storage相连的及其都能实现发现共享磁盘，并进行登陆和挂载磁盘的操作，  
 出于安全考虑，通过acl认证，就是只有指定的ip才能发现共享的磁盘  
 [root@storage1-f15 ~]# vim /etc/tgt/targets.conf   
 <target iqn.2016-10.com.example.storage1-f15:vdb1.2G>  
 backing-store /dev/vdb1  
 initiator-address 172.25.15.14 #设置acl允许的ip  
 initiator-address 172.25.15.15  
 </target>  
  
  
 <target iqn.2016-10.com.example.storage1-f15:vdb2.2G>  
 backing-store /dev/vdb2  
 initiator-address 172.25.15.14  
 </target>  
  
  
 [root@storage1-f15 ~]# service tgtd force-reload  #重新加载服务  
 Force-updating SCSI target daemon configuration: [ OK ]  
 [root@storage1-f15 ~]# tgtadm --lld iscsi --mode target --op show #查看状态，可以看到acl的认证信息  
 -------------------------------------------------------------------  
 init端要先清空原来缓存的文件  
 [root@iscsi1-f15 ~]# rm -rf /var/lib/iscsi/node/*    
 [root@iscsi1-f15 ~]# rm -rf /var/lib/iscsi/send_targets/*  
 [root@iscsi1-f15 ~]# iscsiadm -m discovery -t st -p 172.25.15.10  
 172.25.15.10:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 172.25.15.10:3260,1 iqn.2016-10.com.example.storage1-f15:vdb2.2G  
 重新发现文件  
 [root@iscsi2-f15 ~]# rm -rf /var/lib/iscsi/node/*  
 [root@iscsi2-f15 ~]# rm -rf /var/lib/iscsi/send_targets/*  
 [root@iscsi2-f15 ~]# iscsiadm -m discovery -t st -p 172.25.15.10  
 172.25.15.10:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 只有被允许的ip才能发现共享磁盘  
 ****************************************************************  
3.认证  
 ***************************************************************  
 acl的方法是基于ip地址进行评定的，由于ip地址可以伪造，所以上述方法存在安全问题，  
 认证就是用用户名和密码来判定init端的身份  
 [root@storage1-f15 ~]# vim /etc/tgt/targets.conf   
 <target iqn.2016-10.com.example.storage1-f15:vdb2.2G>  
 backing-store /dev/vdb2  
 initiator-address 172.25.15.14  
 incominguser stadmin uplooking #用户名为stadmin， 密码为uplooking，注意密码后不能加额外的空格  
 </target>  
 [root@storage1-f15 ~]# service tgtd force-reload  #重新加载服务  
 Force-updating SCSI target daemon configuration: [ OK ]  
  
  
 [root@iscsi1-f15 ~]# vim /etc/iscsi/iscsid.conf  #在init端的主机在配置文件中增加认证  
 node.session.auth.authmethod = CHAP  
 # To set a CHAP username and password for initiator  
 # authentication by the target(s), uncomment the following lines:  
 node.session.auth.authmethod = CHAP #启用认证  
 node.session.auth.username = stadmin #用户名  
 node.session.auth.password = uplooking #密码  
 [root@iscsi1-f15 ~]# iscsiadm -m discovery -t st -p 172.25.15.10  
 172.25.15.10:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 172.25.15.10:3260,1 iqn.2016-10.com.example.storage1-f15:vdb2.2G  
  
  
 [root@iscsi1-f15 ~]# iscsiadm -m node -l #登入并导入设备  
 Logging in to [iface: default, target: iqn.2016-10.com.example.storage1-f15:vdb1.2G, portal: 172.25.15.10,3260] (multiple)  
 Logging in to [iface: default, target: iqn.2016-10.com.example.storage1-f15:vdb2.2G, portal: 172.25.15.10,3260] (multiple)  
 Login to [iface: default, target: iqn.2016-10.com.example.storage1-f15:vdb1.2G, portal: 172.25.15.10,3260] successful.  
 Login to [iface: default, target: iqn.2016-10.com.example.storage1-f15:vdb2.2G, portal: 172.25.15.10,3260] successful.  
  
  
 [root@iscsi1-f15 ~]# ls /dev/sd*  
 /dev/sda /dev/sda1 /dev/sdb /dev/sdb1  
 *********************************************************************  
 * 4.lvm方式共享磁盘  *  
 *********************************************************************  
 实现存储空间的动态扩展，过程中不用停止服务  
 vim /etc/sysconfig/network-scripts/ifcfg-eth2  
 ifup eth2  
  
  
 [root@storage1-f15 ~]# vim /etc/tgt/targets.conf   
  
  
 <target iqn.2016-10.com.example.storage1-f15:vdb1.2G>  
 backing-store /dev/vdb1  
 initiator-address 192.168.122.44 #一般使用内网地址  
 initiator-address 192.168.122.45  
 incominguser stadmin uplooking  
 </target>  
  
  
 [root@storage1-f15 ~]# service tgtd force-reload  #服务  
 Force-updating SCSI target daemon configuration: [ OK ]  
 [root@storage1-f15 ~]# tgtadm --lld iscsi --mode target --op show #查看  
  
  
 [root@iscsi1-f15 ~]# umount /mnt  
 [root@iscsi1-f15 ~]# iscsiadm -m node -u  
  
  
 [root@iscsi1-f15 ~]# iscsiadm -m discovery -t st -p 192.168.122.123  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 [root@iscsi1-f15 ~]# iscsiadm -m node -l  
 [root@iscsi1-f15 ~]# ls /dev/sd*  
 /dev/sda /dev/sda1  
 [root@iscsi1-f15 ~]# fdisk /dev/sda  
 [root@iscsi1-f15 ~]# ls /dev/sd*  
 /dev/sda  
 [root@iscsi1-f15 ~]# clustat  
 Cluster Status for storage-f15 @ Mon Oct 10 15:05:24 2016  
 Member Status: Quorate  
  
  
 Member Name ID Status  
 ------ ---- ---- ------  
 iscsi1-f15.example.com 1 Online, Local  
 iscsi2-f15.example.com 2 Online  
  
  
 [root@iscsi1-f15 ~]# service cman status  
 cluster is running.  
 [root@iscsi1-f15 ~]# service clvmd status  
 clvmd (pid 1538) is running...  
 Clustered Volume Groups: (none)  
 Active clustered Logical Volumes: (none)  
  
  
 [root@iscsi1-f15 ~]# pvcreate /dev/sda #创建pv  
 [root@iscsi1-f15 ~]# vgcreate cvg0 /dev/sda  #创建vg  
 [root@iscsi1-f15 ~]# lvcreate -L 1G -n clv0 cvg0  #创建lv  
 [root@iscsi1-f15 ~]# service clvmd status  
 clvmd (pid 1538) is running...  
 Clustered Volume Groups: cvg0  
 Active clustered Logical Volumes: clv0  
 [root@iscsi1-f15 ~]# mkfs.gfs2 -p lock_dlm -j 2 -t storage-f15:ctest /dev/cvg0/clv0 #指定文件系统类型，-j 节点数。（最短支持的主机数）  
  
  
 [root@iscsi1-f15 ~]# mount /dev/cvg0/clv0 /mnt  
 [root@iscsi1-f15 ~]# echo 111 > /mnt/file  
 [root@iscsi1-f15 ~]# cat /mnt/file  
  
  
 ----------------------------------  
 [root@iscsi2-f15 ~]# vim /etc/iscsi/iscsid.conf   
  
  
 [root@iscsi2-f15 ~]# rm -rf /var/lib/iscsi/nodes/*  
 [root@iscsi2-f15 ~]# rm -rf /var/lib/iscsi/send_targets/*  
 [root@iscsi2-f15 ~]# iscsiadm -m discovery -t st -p 192.168.122.123  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 [root@iscsi2-f15 ~]# iscsiadm -m node -l  
 [root@iscsi2-f15 ~]# ls /dev/sd*  
 /dev/sda  
 [root@iscsi2-f15 ~]# service clvmd status  
 clvmd (pid 1531) is running...  
 [root@iscsi2-f15 ~]# mount /dev/cvg0/clv0 /mnt  
 [root@iscsi2-f15 ~]# cat /mnt/file  
 111  
 *********************************************************************  
 * 5.添加一个新的磁盘对lvm进行扩展 *  
 *********************************************************************  
 所有主机启用网卡eth2  
 [root@storage1-f15 ~]# vim /etc/tgt/targets.conf  #新加一块磁盘   
 <target iqn.2016-10.com.example.storage1-f15:vdb2.2G>  
 backing-store /dev/vdb2  
 initiator-address 192.168.122.127  
 initiator-address 192.168.122.128  
 incominguser stadmin uplooking  
 </target>  
 [root@storage1-f15 ~]# service tgtd force-reload  #服务  
 Force-updating SCSI target daemon configuration: [ OK ]  
 [root@iscsi1-f15 ~]# iscsiadm -m discovery -t st -p 192.168.122.123  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb2.2G  
 [root@iscsi1-f15 ~]# iscsiadm -m node -T iqn.2016-10.com.example.storage1-f15:vdb2.2G -l  
 [root@iscsi1-f15 ~]# ls /dev/sd*  
 /dev/sda /dev/sdb /dev/sdb1  
 [root@iscsi1-f15 ~]# fidisk /dev/sdb #删除原来的分区  
 [root@iscsi1-f15 ~]# ls /dev/sd*  
 /dev/sda /dev/sdb  
 [root@iscsi1-f15 ~]# pvcreate /dev/sdb #创建pv  
 Physical volume "/dev/sdb" successfully created    
 [root@iscsi1-f15 ~]# vgextend cvg0 /dev/sdb  #扩展vg  
 Volume group "cvg0" successfully extended    
 [root@iscsi1-f15 ~]# pvs  
 PV VG Fmt Attr PSize PFree   
 /dev/sda cvg0 lvm2 a-- 2.00g 1020.00m  
 /dev/sdb cvg0 lvm2 a-- 2.00g 2.00g  
 /dev/vda2 VolGroup lvm2 a-- 9.51g 0   
 [root@iscsi1-f15 ~]# vgs  
 VG #PV #LV #SN Attr VSize VFree  
 VolGroup 1 2 0 wz--n- 9.51g 0   
 cvg0 2 1 0 wz--nc 3.99g 2.99g  
 [root@iscsi1-f15 ~]# lvextend -L +2G /dev/cvg0/clv0 # #lv的扩展  
 Extending logical volume clv0 to 3.00 GiB  
 Logical volume clv0 successfully resized  
 [root@iscsi1-f15 ~]# lvs  
 LV VG Attr LSize Pool Origin Data% Move Log Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51g   
 lv_swap VolGroup -wi-ao---- 1.00g   
 clv0 cvg0 -wi-ao---- 3.00g   
 [root@iscsi1-f15 ~]# df -h #查看挂载，没有变，因为只是对lv扩展，文件系统没有扩展，inode数目没有变  
 Filesystem Size Used Avail Use% Mounted on  
 /dev/mapper/VolGroup-lv_root 8.4G 2.0G 6.0G 25% /  
 tmpfs 246M 32M 215M 13% /dev/shm  
 /dev/vda1 485M 34M 426M 8% /boot  
 /dev/mapper/cvg0-clv0 1.0G 259M 766M 26% /mnt  
 [root@iscsi1-f15 ~]# gfs2_grow /dev/cvg0/clv0 #文件系统的扩展  
 [root@iscsi1-f15 ~]# df -h #变为了3G  
 Filesystem Size Used Avail Use% Mounted on  
 /dev/mapper/VolGroup-lv_root 8.4G 2.0G 6.0G 25% /  
 tmpfs 246M 32M 215M 13% /dev/shm  
 /dev/vda1 485M 34M 426M 8% /boot  
 /dev/mapper/cvg0-clv0 3.0G 259M 2.8G 9% /mnt  
 [root@iscsi2-f15 ~]# iscsiadm -m discovery -t st -p 192.168.122.123 #2上也变成的3G  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb2.2G  
 [root@iscsi2-f15 ~]# iscsiadm -m node -T iqn.2016-10.com.example.storage1-f15:vdb2.2G -l  
 [root@iscsi2-f15 ~]# ls /dev/sdb*  
 /dev/sdb /dev/sdb1  
 [root@iscsi2-f15 ~]# partx -d --nr 1 /dev/sdb  
 [root@iscsi2-f15 ~]# ls /dev/sdb*  
 /dev/sdb  
 [root@iscsi2-f15 ~]# df -h  
 Filesystem Size Used Avail Use% Mounted on  
 /dev/mapper/VolGroup-lv_root 8.4G 2.0G 6.1G 25% /  
 tmpfs 246M 32M 215M 13% /dev/shm  
 /dev/vda1 485M 34M 426M 8% /boot  
 /dev/mapper/cvg0-clv0 3.0G 259M 2.8G 9% /mnt  
  
  
 *****************************************************************  
6.添加一个新的节点  
 ******************************************************************  
 又有一个新的节点主机要使用共享磁盘，考虑，先加入HA集群还是先导入磁盘  
 做法：先导入磁盘，再加入集群  
 因为集群的作用是通过组播的方式同步信息，必须要先有磁盘存在，才能做同步  
 所以先导入磁盘  
 [root@storage1-f15 ~]# vim /etc/tgt/targets.conf   
 <target iqn.2016-10.com.example.storage1-f15:vdb1.2G>  
 backing-store /dev/vdb1  
 initiator-address 192.168.122.127  
 initiator-address 192.168.122.128  
 initiator-address 192.168.122.126 #增加新的节点ip  
 incominguser stadmin uplooking  
 </target>  
  
  
 <target iqn.2016-10.com.example.storage1-f15:vdb2.2G>  
 backing-store /dev/vdb2  
 initiator-address 192.168.122.127  
 initiator-address 192.168.122.128  
 initiator-address 192.168.122.126 #增加新的节点ip  
 incominguser stadmin uplooking  
 </target>  
  
  
 [root@storage1-f15 ~]# service tgtd force-reload  #服务  
 Force-updating SCSI target daemon configuration: [ OK ]  
  
  
 [root@storage4-f15 ~]# iptables -F  
 [root@storage4-f15 ~]# service iptables save  
 iptables: Saving firewall rules to /etc/sysconfig/iptables:[ OK ]  
 [root@storage4-f15 ~]# setenforce 0  
 [root@storage4-f15 ~]# date  
 Mon Oct 10 15:29:33 CST 2016  
 环境一致为集群做准备  
 [root@storage4-f15 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth2  
 [root@storage4-f15 ~]# ifup eth2 #启用网卡eth2  
 [root@storage4-f15 ~]# yum install ricci -y  #安装客户端  
 [root@storage4-f15 ~]# service ricci start #启动服务  
 Starting oddjobd: [ OK ]  
 generating SSL certificates... done  
 Generating NSS database... done  
 Starting ricci: [ OK ]  
 [root@storage4-f15 ~]# passwd ricci #设置密码  
 [root@storage4-f15 ~]# yum install iscsi* -y  #安装init端软件  
 [root@storage4-f15 ~]# vim /etc/iscsi/iscsid.conf #增加认证  
 [root@storage4-f15 ~]# iscsiadm -m discovery -t st -p 192.168.122.123 #发现  
 Starting iscsid: [ OK ]  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb1.2G  
 192.168.122.123:3260,1 iqn.2016-10.com.example.storage1-f15:vdb2.2G  
 [root@storage4-f15 ~]# iscsiadm -m node -l #登入并导入  
 [root@storage4-f15 ~]# ls /dev/sd*  
 /dev/sda /dev/sdb  
 ----------------------------------------------------------  
 /etc/hosts #修改集群中所有主机的hosts  
 172.25.15.14 iscsi1-f15.example.com  
 172.25.15.15 iscsi2-f15.example.com  
 172.25.15.13 storage4-f15.example.com  
 ~   
 [root@iscsi1-f15 ~]# service luci start #启动luci服务  
 Start luci...   
 [ OK ]  
 ---------------------------------------------------  
 浏览器https://iscsi1-f15.example.com:8084  
 将storage4加入到HA集群  
 勾选，Download Packages Enable Share Stroage Support  
 ----------------------------------------------------   
 [root@storage4-f15 ~]# mount /dev/cvg0/clv0 /mnt  #挂载发现挂载不上，因为之前设置节点数是2个，  
 Too many nodes mounting filesystem, no free journals  
 [root@iscsi1-f15 ~]# gfs2_jadd -j 1 /dev/cvg0/clv0  #增加一个新的节点  
 Filesystem: /mnt  
 Old Journals 2  
 New Journals 3  
 [root@storage4-f15 ~]# lvs  
 LV VG Attr LSize Pool Origin Data% Move Log Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51g   
 lv_swap VolGroup -wi-ao---- 1.00g   
 clv0 cvg0 -wi-a----- 3.00g   
 [root@storage4-f15 ~]# mount /dev/cvg0/clv0 /mnt  #挂载成功，测试  
 [root@storage4-f15 ~]# cat /mnt/file   
 111  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
   
 