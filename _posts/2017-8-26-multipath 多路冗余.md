---
title: multipath 多路冗余
date: 2016-10-12 22:14:25
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52801274   
  multipath 多路冗余

 **一、什么是multipath**

 普通的电脑主机都是一个硬盘挂接到一个总线上，这里是一对一的关系。而到了有光纤组成的SAN环境，由于主机和存储通过了光纤交换机连接，这样的话，就构成了多对多的关系。也就是说，主机到存储可以有多条路径可以选择。主机到存储之间的IO由多条路径可以选择。

 既然，每个主机到所对应的存储可以经过几条不同的路径，如果是同时使用的话，I/O流量如何分配？其中一条路径坏掉了，如何处理？还有在[操作系统](http://lib.csdn.net/base/operatingsystem)的角度来看，每条路径，操作系统会认为是一个实际存在的物理盘，但实际上只是通向同一个物理盘的不同路径而已，这样是在使用的时候，就给用户带来了困惑。多路径软件就是为了解决上面的问题应运而生的。多路径的主要功能就是和存储设备一起配合实现如下功能：

 1. 故障的切换和恢复

 2. IO流量的负载均衡

 3. 磁盘的虚拟化

 **二、为什么使用multipath**

 由于多路径软件是需要和存储在一起配合使用的，不同的厂商基于不同的操作系统，都提供了不同的版本。并且有的厂商，软件和硬件也不是一起卖的，如果要使用多路径软件的话，可能还需要向厂商购买license才行。比如EMC公司基于[Linux](http://lib.csdn.net/base/linux)下的多路径软件，就需要单独的购买license。

 其中，EMC提供的就是PowerPath，HDS提供的就是HDLM，更多的存储厂商提供的软件，可参考这里。

 当然，使用系统自带的免费多路径软件包，同时也是一个比较通用的包，可以支持大多数存储厂商的设备，即使是一些不是出名的厂商，通过对配置文件进行稍作修改，也是可以支持并运行的很好的。

 ※ 请与IBM的RDAC、Qlogic的failover驱动区分开，它们都仅提供了Failover的功能，不支持Load Balance方式。但multipath根据选择的策略不同，可支持多种方式，如：Failover、Multipath等。

 **三、multipath的组成**

 我这里以红帽x86_64为例，虽然版本比较老，但下面的配置方式基本适用后面的所有版本。

 # cat /etc/redflag-release

 Red Flag DC Server release 5.0 (Trinity SP2)

 # uname -a

 Linux localhost.localdomain 2.6.18-164.el5 #1 SMP Tue Aug 18 15:51:48 EDT 2009 x86_64 x86_64 x86_64 GNU/Linux

 # rpm -qa|grep device

 device-mapper-event-1.02.32-1.el5

 device-mapper-1.02.32-1.el5

 device-mapper-multipath-0.4.7-30.el5

 device-mapper-1.02.32-1.el5

 可见，一套完整的multipath由下面几部分组成：

 **1. device-mapper-multipath**

 提供multipathd和multipath等工具和multipath.conf等配置文件。这些工具通过device mapper的ioctr的接口创建和配置multipath设备（调用device-mapper的用户空间库。创建的多路径设备会在/dev/mapper中）；

 **2. device-mapper**

 device-mapper包括两大部分：内核部分和用户部分。

 内核部分由device-mapper核心（multipath.ko）和一些target driver（dm-multipath.ko）构成。dm-mod.ko是实现multipath的基础，dm-multipath其实是dm的一个target驱动。核心完成设备的映射，而target根据映射关系和自身特点具体处理从mappered device 下来的i/o。同时，在核心部分，提供了一个接口，用户通过ioctr可和内核部分通信，以指导内核驱动的行为，比如如何创建mappered device，这些device的属性等。

 用户空间部分包括device-mapper这个包。其中包括dmsetup工具和一些帮助创建和配置mappered device的库。这些库主要抽象，封装了与ioctr通信的接口，以便方便创建和配置mappered device。device-mapper-multipath的程序中就需要调用这些库；

 **3. scsi_id**

 其包含在udev程序包中，可以在multipath.conf中配置该程序来获取scsi设备的序号。通过序号，便可以判断多个路径对应了同一设备。这个是多路径实现的关键。scsi_id是通过sg驱动，向设备发送EVPD page80或page83 的inquery命令来查询scsi设备的标识。但一些设备并不支持EVPD 的inquery命令，所以他们无法被用来生成multipath设备。但可以改写scsi_id，为不能提供scsi设备标识的设备虚拟一个标识符，并输出到标准输出。

 multipath程序在创建multipath设备时，会调用scsi_id，从其标准输出中获得该设备的scsi id。在改写时，需要修改scsi_id程序的返回值为0。因为在multipath程序中，会检查该直来确定scsi id是否已经成功得到。

 **四、配置multipath**

 原理看了一堆，实际配置还是比较简单的。配置文件只有一个：/etc/multipath.conf 。配置前，请用fdisk -l 确认已可正确识别盘柜的所有LUN，HDS支持多链路负载均衡，因此每条链路都是正常的；而如果是类似EMC CX300这样仅支持负载均衡的设备，则冗余的链路会出现I/O Error的错误。

 详细步骤：  
 [root@iscsi1-f15 ~]# iptables -F  
 [root@iscsi1-f15 ~]# chkconfig iptables on  
 [root@iscsi1-f15 ~]# setenforce 0  
 [root@iscsi1-f15 ~]# cd /etc/sysconfig/network-scripts/  
 1. 启用 eth1 和 eth2 两块网卡,并设置 ip  
 [root@iscsi1-f15 network-scripts]# vim ifcfg-eth1  
 [root@iscsi1-f15 network-scripts]# vim ifcfg-eth2  
 [root@iscsi1-f15 network-scripts]# cat ifcfg-eth1  
 DEVICE=eth1  
 TYPE=Ethernet  
 ONBOOT=yes  
 NM_CONTROLLED=yes  
 BOOTPROTO=static  
 IPADDR=10.0.1.14  
 NETMASK=255.255.255.0  
 [root@iscsi1-f15 network-scripts]# cat ifcfg-eth2  
 DEVICE=eth2  
 TYPE=Ethernet  
 ONBOOT=yes  
 NM_CONTROLLED=yes  
 BOOTPROTO=static  
 IPADDR=10.0.2.14  
 NETMASK=255.255.255.0  
 [root@iscsi1-f15 network-scripts]# ifup eth1  
 Determining if ip address 10.0.1.14 is already in use for device eth1...  
 [root@iscsi1-f15 network-scripts]# ifup eth2  
 Determining if ip address 10.0.2.14 is already in use for device eth2...  
 [root@iscsi1-f15 network-scripts]# ifconfig  
 ------------------------------------------------------------------------  
 [root@storage1-f15 ~]# iptables -F  
 [root@storage1-f15 ~]# chkconfig iptables on  
 [root@storage1-f15 ~]# setenforce 0  
 [root@storage1-f15 ~]# cd /etc/sysconfig/network-scripts/  
 [root@storage1-f15 network-scripts]# vim ifcfg-eth1  
 [root@storage1-f15 network-scripts]# cat ifcfg-eth1  
 DEVICE=eth1  
 TYPE=Ethernet  
 ONBOOT=yes  
 NM_CONTROLLED=yes  
 BOOTPROTO=static  
 IPADDR=10.0.1.10  
 NETMASK=255.255.255.0  
 [root@storage1-f15 network-scripts]# vim ifcfg-eth2  
 [root@storage1-f15 network-scripts]# cat ifcfg-eth2  
 DEVICE=eth2  
 TYPE=Ethernet  
 ONBOOT=yesNM_CONTROLLED=yes  
 BOOTPROTO=static  
 IPADDR=10.0.2.10  
 NETMASK=255.255.255.0  
 [root@storage1-f15 network-scripts]# ifup eth1  
 Determining if ip address 10.0.1.10 is already in use for device eth1...  
 [root@storage1-f15 network-scripts]# ifup eth2  
 Determining if ip address 10.0.2.10 is already in use for device eth2...  
 [root@storage1-f15 network-scripts]# ifconfig  
 -------------------------------------------------------------------------  
 2. 创建一个新的分区作为共享设备  
 [root@iscsi1-f15 network-scripts]# fdisk /dev/vdb  
 Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel  
 Building a new DOS disklabel with disk identifier 0x279bb9f4.  
 Changes will remain in memory only, until you decide to write them.  
 After that, of course, the previous content won't be recoverable.  
 Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)  
 WARNING: DOS-compatible mode is deprecated. It's strongly recommended to  
 switch off the mode (command 'c') and change display units to  
 sectors (command 'u').  
 Command (m for help): p  
 Disk /dev/vdb: 10.7 GB, 10737418240 bytes  
 16 heads, 63 sectors/track, 20805 cylinders  
 Units = cylinders of 1008 * 512 = 516096 bytes  
 Sector size (logical/physical): 512 bytes / 512 bytes  
 I/O size (minimum/optimal): 512 bytes / 512 bytes  
 Disk identifier: 0x279bb9f4  
 Device Boot  
 Start  
 End  
 Blocks Id System  
 Command (m for help): n  
 Command action  
 e extended  
 p primary partition (1-4)  
 p  
 Partition number (1-4): 1  
 First cylinder (1-20805, default 1):  
 Using default value 1  
 Last cylinder, +cylinders or +size{K,M,G} (1-20805, default 20805): +2G  
 Command (m for help): w  
 The partition table has been altered!  
 Calling ioctl() to re-read partition table.  
 Syncing disks.[root@iscsi1-f15 network-scripts]# ls -l /dev/vdb1  
 brw-rw----. 1 root disk 252, 17 Oct 12 15:01 /dev/vdb1  
 3. 安装 scsi* 软件,修改配置文件指定共享磁盘  
 [root@iscsi1-f15 network-scripts]# yum install scsi-target*  
 [root@iscsi1-f15 network-scripts]# chkconfig tgtd on  
 [root@iscsi1-f15 network-scripts]# service tgtd start  
 Starting SCSI target daemon:  
 [ OK ]  
 [root@iscsi1-f15 network-scripts]# vim /etc/tgt/targets.conf  
 <target iqn.2016-10.com.example.iscsi1-f15:disk1>  
 backing-store /dev/vdb1  
 scsi_id f15_disk1_id  
 scsi_sn f15_disk1_sn  
 </target>  
 [root@iscsi1-f15 network-scripts]# tgtadm --lld iscsi --mode target --op show  
 [root@iscsi1-f15 network-scripts]# service tgtd reload  
 # 重新加载服务  
 Updating SCSI target daemon configuration:  
 [ OK ]  
 [root@iscsi1-f15 network-scripts]# tgtadm --lld iscsi --mode target --op show  
 Target 1: iqn.2016-10.com.example.iscsi1-f15:disk1  
 System information:  
 Driver: iscsi  
 State: ready  
 I_T nexus information:  
 LUN information:  
 LUN: 0  
 Type: controller  
 SCSI ID: IET 00010000  
 SCSI SN: beaf10  
 Size: 0 MB, Block size: 1  
 Online: Yes  
 Removable media: No  
 Prevent removal: No  
 Readonly: No  
 Backing store type: null  
 Backing store path: None  
 Backing store flags:  
 LUN: 1  
 Type: disk  
 SCSI ID: f15_disk1_id  
 SCSI SN: f15_disk1_sn  
 Size: 2148 MB, Block size: 512  
 Online: Yes  
 Removable media: No  
 Prevent removal: No  
 Readonly: No  
 Backing store type: rdwr  
 Backing store path: /dev/vdb1  
 Backing store flags:  
 Account information:  
 # 查看共享ACL information:  
 ALL  
 ----------------------------------------------------  
 4. 服务端安装 iscsi*  
 [root@storage1-f15 network-scripts]# yum install iscsi-initiator-utils -y #init 端安装 iscsi 软件  
 [root@storage1-f15 network-scripts]# chkconfig iscsi on  
 [root@storage1-f15 network-scripts]# service iscsi start  
 [root@storage1-f15 network-scripts]# lsmod | grep iscsi  
 5. 发现并登陆和导入共享设备  
 [root@storage1-f15 network-scripts]# iscsiadm -m discovery -t st -p 10.0.1.14  
 Starting iscsid:  
 [ OK ]  
 10.0.1.14:3260,1 iqn.2016-10.com.example.iscsi1-f15:disk1  
 [root@storage1-f15 network-scripts]# iscsiadm -m node -l  
 Logging in to [iface: default, target: iqn.2016-10.com.example.iscsi1-f15:disk1, portal: 10.0.1.14,3260]  
 (multiple)  
 Login to [iface: default, target: iqn.2016-10.com.example.iscsi1-f15:disk1, portal: 10.0.1.14,3260]  
 successful.  
 [root@storage1-f15 network-scripts]# iscsiadm -m discovery -t st -p 10.0.2.14  
 10.0.2.14:3260,1 iqn.2016-10.com.example.iscsi1-f15:disk1  
 [root@storage1-f15 network-scripts]# iscsiadm -m node -l  
 Logging in to [iface: default, target: iqn.2016-10.com.example.iscsi1-f15:disk1, portal: 10.0.2.14,3260]  
 (multiple)  
 Login to [iface: default, target: iqn.2016-10.com.example.iscsi1-f15:disk1, portal: 10.0.2.14,3260]  
 successful.  
 [root@storage1-f15 network-scripts]# ls /dev/sd* # 查看导入成功  
 /dev/sda /dev/sdb  
 [root@storage1-f15 network-scripts]# lsscsi  
 [2:0:0:0] storage IET Controller  
 0001 -  
 [2:0:0:1] disk IET VIRTUAL-DISK 0001 /dev/sda  
 [3:0:0:0] storage IET Controller  
 0001 -  
 [3:0:0:1] disk IET VIRTUAL-DISK 0001 /dev/sdb  
 [root@storage1-f15 network-scripts]# scsi_id -h  
 Usage: scsi_id OPTIONS <device>  
 --device=  
 device node for SG_IO commands  
 --config=  
 location of config file  
 --page=0x80|0x83|pre-spc3-83 SCSI page (0x80, 0x83, pre-spc3-83)  
 --sg-version=3|4  
 use SGv3 or SGv4  
 --blacklisted  
 threat device as blacklisted  
 --whitelisted  
 threat device as whitelisted  
 --replace-whitespace  
 replace all whitespaces by underscores  
 --verbose  
 verbose logging  
 --version  
 print version  
 --export  
 print values as environment keys  
 --help  
 print this help text  
 [root@storage1-f15 network-scripts]# scsi_id --whitelisted --device=/dev/sda  
 # 识别共享设备  
 1f15_disk1_id  
 [root@storage1-f15 network-scripts]# scsi_id --whitelisted --device=/dev/sdb # 识别共享设备  
 1f15_disk1_id  
 6. 启动 multipathd 多路冗余服务[root@storage1-f15 network-scripts]# yum list device-mapper-multipath*  
 [root@storage1-f15 network-scripts]# chkconfig multipathd on  
 [root@storage1-f15 network-scripts]# service multipathd start  
 # 启动 multipathd 多路冗余服务  
 Starting multipathd daemon:  
 [ OK ]  
 [root@storage1-f15 network-scripts]# multipath -lld  
 Oct 12 15:29:07 | /etc/multipath.conf does not exist, blacklisting all devices.  
 Oct 12 15:29:07 | A sample multipath.conf file is located at  
 Oct 12 15:29:07 | /usr/share/doc/device-mapper-multipath-0.4.9/multipath.conf  
 Oct 12 15:29:07 | You can run /sbin/mpathconf to create or modify /etc/multipath.conf  
 7. 修改 mpathconf 配置  
 [root@storage1-f15 network-scripts]# mpathconf -h  
 usage: /sbin/mpathconf <command>  
 Commands:  
 Enable: --enable  
 Disable: --disable  
 Set user_friendly_names (Default n): --user_friendly_names <y|n>  
 Set find_multipaths (Default n): --find_multipaths <y|n>  
 Load the dm-multipath modules on enable (Default y): --with_module <y|n>  
 start/stop/reload multipathd (Default n): --with_multipathd <y|n>  
 chkconfig on/off multipathd (Default y): --with_chkconfig <y|n>  
 [root@storage1-f15 network-scripts]# mpathconf --user_friendly_names y --find_multipaths y  
 --with_module y --with_multipathd y --with_chkconfig y # 修改 mpathconf 配置  
 Reloading multipathd:  
 [ OK ]  
 [root@storage1-f15 network-scripts]# ls -l /dev/mapper/*  
 crw-rw----. 1 root root 10, 58 Oct 12 14:41 /dev/mapper/control  
 lrwxrwxrwx. 1 root root 7 Oct 12 15:30 /dev/mapper/mpatha -> ../dm-2  
 lrwxrwxrwx. 1 root root 7 Oct 12 14:41 /dev/mapper/VolGroup-lv_root -> ../dm-0  
 lrwxrwxrwx. 1 root root 7 Oct 12 14:41 /dev/mapper/VolGroup-lv_swap -> ../dm-1  
 [root@storage1-f15 network-scripts]# multipath -lld  
 # 查看状态  
 mpatha (1f15_disk1_id) dm-2 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=1 status=active  
 | `- 2:0:0:1 sda 8:0 active ready running  
 `-+- policy='round-robin 0' prio=1 status=enabled  
 `- 3:0:0:1 sdb 8:16 active ready running  
 [root@storage1-f15 network-scripts]# fdisk /dev/mapper/mpatha # 新建一个分区  
 [root@storage1-f15 network-scripts]# ls -l /dev/mapper/  
 total 0  
 crw-rw----. 1 root root 10, 58 Oct 12 14:41 control  
 lrwxrwxrwx. 1 root root 7 Oct 12 15:33 mpatha -> ../dm-2  
 lrwxrwxrwx. 1 root root 7 Oct 12 15:33 mpathap1 -> ../dm-3  
 lrwxrwxrwx. 1 root root 7 Oct 12 14:41 VolGroup-lv_root -> ../dm-0  
 lrwxrwxrwx. 1 root root 7 Oct 12 14:41 VolGroup-lv_swap -> ../dm-1  
 [root@storage1-f15 network-scripts]# ls /dev/sd*  
 /dev/sda /dev/sdb  
 [root@storage1-f15 network-scripts]# kpartx -a /dev/sda  
 # 强制刷新[root@storage1-f15 network-scripts]# kpartx -a /dev/sdb  
 [root@storage1-f15 network-scripts]# ls /dev/sd*  
 /dev/sda /dev/sdb  
 [root@storage1-f15 network-scripts]# mkfs.ext4 /dev/mapper/mpathap1 # 格式化指定文件系统  
 [root@storage1-f15 network-scripts]# mount /dev/mapper/# 挂载  
 [root@storage1-f15 network-scripts]# cd /opt/  
 [root@storage1-f15 opt]# echo 111 > file  
 [root@storage1-f15 opt]# cat file  
 [root@storage1-f15 opt]# (while true ; do echo A >> foo ; sleep 1 ;done ) &  
 [1] 24126  
 ------------------------------------------------------------------  
 停掉一个网卡,服务一直有效  
 [root@iscsi1-f15 network-scripts]# ifdown eth1  
 [root@storage1-f15 opt]# tail foo  
 A  
 A  
 A  
 A  
 [root@storage1-f15 opt]# multipath -lld  
 mpatha (1f15_disk1_id) dm-2 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=0 status=active  
 | `- 2:0:0:1 sda 8:0 active faulty running  
 `-+- policy='round-robin 0' prio=1 status=enabled  
 `- 3:0:0:1 sdb 8:16 active ready running  
 [root@iscsi1-f15 network-scripts]# ifup eth1  
 Determining if ip address 10.0.1.14 is already in use for device eth1...  
 [root@storage1-f15 ~]# multipath -lld  
 mpatha (1f15_disk1_id) dm-2 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=1 status=enabled  
 | `- 2:0:0:1 sda 8:0 active ready running  
 `-+- policy='round-robin 0' prio=1 status=active  
 `- 3:0:0:1 sdb 8:16 active ready running  
 *************************************************************************  
 增加一个存储,保证一个存储故障后服务继续可用  
 [root@iscsi2-f15 ~]# iptables -F  
 [root@iscsi2-f15 ~]# chkconfig iptables off  
 [root@iscsi2-f15 ~]# setenforce 0  
 [root@iscsi2-f15 ~]# cd /etc/sysconfig/network-scripts/  
 [root@iscsi2-f15 network-scripts]# vim ifcfg-eth1  
 [root@iscsi2-f15 network-scripts]# vim ifcfg-eth2  
 [root@iscsi2-f15 network-scripts]# ifup eth1  
 Determining if ip address 10.0.1.15 is already in use for device eth1...  
 [root@iscsi2-f15 network-scripts]# ifup eth2  
 Determining if ip address 10.0.2.15 is already in use for device eth2...  
 1. 安装 storage 端软件 scsi*  
 [root@iscsi2-f15 network-scripts]# yum install scsi-target*2. 指定共享磁盘  
 [root@iscsi2-f15 network-scripts]# chkconfig tgtd on  
 [root@iscsi2-f15 network-scripts]# fdisk /dev/vdb # 建立一个新的分区  
 [root@iscsi2-f15 network-scripts]# vim /etc/tgt/targets.conf  
 <target iqn.2016-10.com.example.iscsi2:disk1>  
 backing-store /dev/vdb1  
 scsi_id f15_iscsi2_id  
 scsi_sn f15_iscsi2_sn  
 </target>  
 [root@iscsi2-f15 network-scripts]# service tgtd start  
 # 启动服务  
 Starting SCSI target daemon:  
 [ OK ]  
 [root@iscsi2-f15 network-scripts]# tgtadm --lld iscsi --mode target --op show  
 Target 1: iqn.2016-10.com.example.iscsi2:disk1  
 ------------------------------------------------------------  
 3. 服务端重新发现磁盘  
 [root@storage1-f15 opt]# iscsiadm -m discovery -t st -p 10.0.1.15  
 10.0.1.15:3260,1 iqn.2016-10.com.example.iscsi2:disk1  
 [root@storage1-f15 opt]# iscsiadm -m discovery -t st -p 10.0.2.15  
 10.0.2.15:3260,1 iqn.2016-10.com.example.iscsi2:disk1  
 [root@storage1-f15 opt]# iscsiadm -m node -l  
 # 登陆并导入共享磁盘  
 Logging in to [iface: default, target: iqn.2016-10.com.example.iscsi2:disk1, portal: 10.0.1.15,3260]  
 (multiple)  
 Logging in to [iface: default, target: iqn.2016-10.com.example.iscsi2:disk1, portal: 10.0.2.15,3260]  
 (multiple)  
 Login to [iface: default, target: iqn.2016-10.com.example.iscsi2:disk1, portal: 10.0.1.15,3260]  
 successful.  
 Login to [iface: default, target: iqn.2016-10.com.example.iscsi2:disk1, portal: 10.0.2.15,3260]  
 successful.  
 [root@storage1-f15 opt]# ls /dev/sd*  
 /dev/sda /dev/sdb /dev/sdc /dev/sdd  
 [root@storage1-f15 opt]# multipath -lld  
 # 新导入的磁盘会自动加入到 multipath  
 mpathb (1f15_iscsi2_id) dm-6 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=1 status=active  
 | `- 5:0:0:1 sdc 8:32 active ready running  
 `-+- policy='round-robin 0' prio=1 status=enabled  
 `- 4:0:0:1 sdd 8:48 active ready running  
 mpatha (1f15_disk1_id) dm-2 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=1 status=enabled  
 | `- 2:0:0:1 sda 8:0 active ready running  
 `-+- policy='round-robin 0' prio=1 status=active  
 `- 3:0:0:1 sdb 8:16 active ready running  
 ----------------- 对存储做镜像 ---------------------------  
 两个存储要保证数据的一致,所以要多两个存储做镜像  
 [root@storage1-f15 ~]# umount /opt/  
 [root@storage1-f15 ~]# fdisk /dev/mapper/mpatha # 删除 mpathap1  
 [root@storage1-f15 ~]# fdisk /dev/mapper/mpatha # 新建两个分区  
 [root@storage1-f15 ~]# fdisk /dev/mapper/mpathb # 新建两个分区[root@storage1-f15 ~]# ls /dev/mapper/mpath*  
 /dev/mapper/mpatha /dev/mapper/mpathbp1  
 /dev/mapper/mpathb /dev/mapper/mpathbp2  
 [root@storage1-f15 ~]# partprobe # 刷新  
 Warning: WARNING: the kernel failed to re-read the partition table on /dev/vda (Device or resource  
 busy). As a result, it may not reflect all of your changes until after reboot.  
 [root@storage1-f15 ~]# ls /dev/mapper/mpath*  
 /dev/mapper/mpatha /dev/mapper/mpathap2 /dev/mapper/mpathbp1  
 /dev/mapper/mpathap1 /dev/mapper/mpathb /dev/mapper/mpathbp2  
 [root@storage1-f15 ~]# pvcreate /dev/mapper/mpathap1 # 创建 pv  
 Physical volume "/dev/mapper/mpathap1" successfully created  
 [root@storage1-f15 ~]# pvcreate /dev/mapper/mpathap2  
 Physical volume "/dev/mapper/mpathap2" successfully created  
 [root@storage1-f15 ~]# pvcreate /dev/mapper/mpathbp1  
 Physical volume "/dev/mapper/mpathbp1" successfully created  
 [root@storage1-f15 ~]# pvcreate /dev/mapper/mpathbp2  
 Physical volume "/dev/mapper/mpathbp2" successfully created  
 [root@storage1-f15 ~]# pvs  
 Found duplicate PV cHwUBQGkhEdQ0Nqt1oltZ04DKhsRdhoS: using /dev/mapper/sda1 not  
 /dev/mapper/mpathap1  
 Found duplicate PV cHwUBQGkhEdQ0Nqt1oltZ04DKhsRdhoS: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 PV  
 VG  
 Fmt Attr PSize PFree  
 /dev/mapper/mpathap1  
 lvm2 a-- 509.84m 509.84m  
 /dev/mapper/mpathap2  
 lvm2 a-- 509.88m 509.88m  
 /dev/mapper/mpathbp1  
 lvm2 a-- 509.84m 509.84m  
 /dev/mapper/mpathbp2  
 lvm2 a-- 509.88m 509.88m  
 /dev/mapper/sdb1  
 lvm2 a-- 509.84m 509.84m  
 /dev/vda2  
 VolGroup lvm2 a-- 9.51g  
 0  
 [root@storage1-f15 ~]# vgcreate myvg0 /dev/mapper/mpathap[1,2] /dev/mapper/mpathbp[1,2]# 创建 vg  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sda1 not  
 /dev/mapper/mpathap1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 Volume group "myvg0" successfully created  
 [root@storage1-f15 ~]# vgs  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 VG  
 #PV #LV #SN Attr VSize VFree  
 VolGroup 1 2 0 wz--n- 9.51g 0  
 myvg0  
 4 0 0 wz--n- 1.98g 1.98g创建 lv  
 [root@storage1-f15 ~]# lvcreate -m 1 -L 256M myvg0 -n mylvm /dev/mapper/mpathap1  
 /dev/mapper/mpathbp1  
 -m 使用镜像方式, ap1 做存储, bp1 做镜像  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 Logical volume "mylvm" created  
 [root@storage1-f15 ~]# lvs  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 LV VG  
 Attr  
 LSize Pool Origin Data% Move Log  
 Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51g  
 lv_swap VolGroup -wi-ao---- 1.00g  
 mylvm myvg0 mwi-a-m--- 256.00m  
 mylvm_mlog 10.94  
 [root@storage1-f15 ~]# mkfs.ext4 /dev/myvg0/mylvm  
 # 创建文件系统  
 [root@storage1-f15 ~]# mount /dev/myvg0/mylvm /opt # 挂接  
 [root@storage1-f15 ~]# kpartx /dev/mapper/mpatha  
 mpatha1 : 0 1044162 /dev/mapper/mpatha 63  
 mpatha2 : 0 1044225 /dev/mapper/mpatha 1044225  
 [root@storage1-f15 ~]# kpartx /dev/mapper/mpathb  
 mpathb1 : 0 1044162 /dev/mapper/mpathb 63  
 mpathb2 : 0 1044225 /dev/mapper/mpathb 1044225  
 [root@storage1-f15 ~]# cd /opt  
 [root@storage1-f15 opt]# (while true ; do echo B >> foo ; sleep 1 ; done) &  
 [1] 25648  
 [root@storage1-f15 opt]# tail -f foo  
 B  
 B  
 B  
 .............  
 [root@iscsi2-f15 network-scripts]# ifdown eth1  
 [root@iscsi2-f15 network-scripts]# ifdown eth2  
 停止两块网卡相当于 iscsi2 存储设备故障无法访问,可以看出服务继续可用  
 --------------------------------------------------  
 [root@storage1-f15 opt]# multipath -lld  
 # 查看状态  
 mpathb (1f15_iscsi2_id) dm-6 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=0 status=active  
 | `- 5:0:0:1 sdc 8:32 active faulty running  
 `-+- policy='round-robin 0' prio=0 status=enabled  
 `- 4:0:0:1 sdd 8:48 active faulty running  
 mpatha (1f15_disk1_id) dm-2 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=1 status=enabled  
 | `- 2:0:0:1 sda 8:0 active ready running`-+- policy='round-robin 0' prio=1 status=active  
 `- 3:0:0:1 sdb 8:16 active ready running  
 [root@storage1-f15 ~]# tail -f /opt/foo  
 B  
 B  
 B  
 B  
 B  
 -------------------------------  
 停止执行的脚本  
 [root@storage1-f15 opt]# ps -ef |grep bash  
 root  
 1481 1477 0 14:42 pts/0 00:00:00 -bash  
 root 25648 1481 0 16:59 pts/0 00:00:00 -bash  
 root 28188 28184 0 17:40 pts/2 00:00:00 -bash  
 root 28320 1481 0 17:42 pts/0 00:00:00 grep bash  
 -bash: tpy: command not found  
 [root@storage1-f15 opt]# tty  
 /dev/pts/0  
 [root@storage1-f15 opt]# kill 25648  
 -------------------------------------------  
 iscsi2 存储重新加入  
 ot@iscsi2-f15 network-scripts]# ifup eth1  
 Determining if ip address 10.0.1.15 is already in use for device eth1...  
 [root@iscsi2-f15 network-scripts]# ifup eth2  
 Determining if ip address 10.0.2.15 is already in use for device eth2...  
 lvs 会报错,因为 iscsi2 离线后有加入, mpathbp1 和 mpathbp2 不能自动的加入到 lvm 中,要执行  
 lvconvert 命令手动加入到 lv  
 [root@storage1-f15 ~]# lvs  
 /dev/mapper/mpathbp1: read failed after 0 of 1024 at 534511616: Input/output error  
 /dev/mapper/mpathbp1: read failed after 0 of 1024 at 534601728: Input/output error  
 /dev/mapper/mpathbp1: read failed after 0 of 1024 at 0: Input/output error  
 /dev/mapper/mpathbp1: read failed after 0 of 1024 at 4096: Input/output error  
 /dev/mapper/mpathbp1: read failed after 0 of 2048 at 0: Input/output error  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 /dev/mapper/mpathb: read failed after 0 of 512 at 2147876864: Input/output error  
 /dev/mapper/mpathb: read failed after 0 of 512 at 2147950592: Input/output error  
 /dev/mapper/mpathb: read failed after 0 of 512 at 0: Input/output error  
 /dev/mapper/mpathb: read failed after 0 of 512 at 4096: Input/output error  
 /dev/mapper/mpathb: read failed after 0 of 2048 at 0: Input/output error  
 /dev/mapper/mpathbp2: read failed after 0 of 512 at 534577152: Input/output error  
 /dev/mapper/mpathbp2: read failed after 0 of 512 at 534634496: Input/output error  
 /dev/mapper/mpathbp2: read failed after 0 of 512 at 0: Input/output error  
 /dev/mapper/mpathbp2: read failed after 0 of 512 at 4096: Input/output error  
 /dev/mapper/mpathbp2: read failed after 0 of 2048 at 0: Input/output error  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 Couldn't find device with uuid mtSCjr-WDfY-H28k-a90X-0SFp-N2K6-kuZFbS.  
 Couldn't find device with uuid P6hSoX-VPNc-ElMD-r5Zn-rmh5-vVUP-MVOFeh.LV VG  
 Attr  
 LSize Pool Origin Data% Move Log Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51g  
 lv_swap VolGroup -wi-ao---- 1.00g  
 mylvm myvg0 -wi-ao---- 256.00m  
 lvconvert 命令手动加入到 lv  
 [root@storage1-f15 ~]# lvconvert -m 1 /dev/myvg0/mylvm /dev/mapper/mpathbp1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 WARNING: Inconsistent metadata found for VG myvg0 - updating to use version 8  
 Missing device /dev/mapper/mpathbp1 reappeared, updating metadata for VG myvg0 to version 8.  
 Missing device /dev/mapper/mpathbp2 reappeared, updating metadata for VG myvg0 to version 8.  
 myvg0/mylvm: Converted: 0.0%  
 myvg0/mylvm: Converted: 51.6%  
 myvg0/mylvm: Converted: 100.0%  
 [root@storage1-f15 ~]# lvs  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 LV VG  
 Attr  
 LSize Pool Origin Data% Move Log  
 Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51g  
 lv_swap VolGroup -wi-ao---- 1.00g  
 mylvm myvg0 mwi-aom--- 256.00m  
 mylvm_mlog 100.00  
 ***********************************************************************************  
 **************  
 保证服务的可用性,增加一台提供服务的服务器,保证一台服务器故障,服务继续可用  
 需要搭建 HA 集群  
 [root@storage2-f15 ~]# iptables -F  
 [root@storage2-f15 ~]# chkconfig iptables off  
 [root@storage2-f15 ~]# setenforce 0  
 [root@storage2-f15 ~]# cd /etc/sysconfig/network-scripts/  
 [root@storage2-f15 network-scripts]# vim /ifcfg-eth1  
 [root@storage2-f15 network-scripts]# vim ifcfg-eth1  
 [root@storage2-f15 network-scripts]# vim ifcfg-eth2  
 [root@storage2-f15 network-scripts]# ifup eth1  
 Determining if ip address 10.0.1.11 is already in use for device eth1...  
 [root@storage2-f15 network-scripts]# ifup eth2  
 Determining if ip address 10.0.2.11 is already in use for device eth2...  
 1. 安装 init 端软件发现并登陆和导入共享磁盘  
 [root@storage2-f15 network-scripts]# yum install iscsi* -y  
 [root@storage2-f15 network-scripts]# chkconfig iscsid on  
 [root@storage2-f15 network-scripts]# iscsiadm -m discovery -t st -p 10.0.1.14  
 Starting iscsid:  
 [ OK ]  
 10.0.1.14:3260,1 iqn.2016-10.com.example.iscsi1-f15:disk1  
 [root@storage2-f15 network-scripts]# iscsiadm -m discovery -t st -p 10.0.2.1410.0.2.14:3260,1 iqn.2016-10.com.example.iscsi1-f15:disk1  
 [root@storage2-f15 network-scripts]# iscsiadm -m discovery -t st -p 10.0.1.15  
 10.0.1.15:3260,1 iqn.2016-10.com.example.iscsi2:disk1  
 [root@storage2-f15 network-scripts]# iscsiadm -m discovery -t st -p 10.0.2.15  
 10.0.2.15:3260,1 iqn.2016-10.com.example.iscsi2:disk1  
 [root@storage2-f15 network-scripts]# iscsiadm -m node -l  
 [root@storage2-f15 network-scripts]# ls /dev/sd?  
 /dev/sda /dev/sdb /dev/sdc /dev/sdd  
 # 识别磁盘  
 [root@storage2-f15 network-scripts]# scsi_id --whitelisted --device=/dev/sda  
 1f15_disk1_id  
 [root@storage2-f15 network-scripts]# scsi_id --whitelisted --device=/dev/sdb  
 1f15_iscsi2_id  
 [root@storage2-f15 network-scripts]# scsi_id --whitelisted --device=/dev/sdc  
 1f15_disk1_id  
 [root@storage2-f15 network-scripts]# scsi_id --whitelisted --device=/dev/sdd  
 1f15_iscsi2_id  
 2. 配置导入的磁盘  
 [root@storage2-f15 network-scripts]# mpathconf -h  
 [root@storage2-f15 network-scripts]# mpathconf --user_friendly_names y --find_multipaths y  
 --with_module y --with_multipathd y --with_chkconfig y # 配置  
 [root@storage2-f15 network-scripts]# service multipathd start  
 Starting multipathd daemon:  
 [ OK ]  
 [root@storage2-f15 network-scripts]# multipath -lld  
 mpathb (1f15_iscsi2_id) dm-5 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=1 status=active  
 | `- 4:0:0:1 sdb 8:16 active ready running  
 `-+- policy='round-robin 0' prio=1 status=enabled  
 `- 5:0:0:1 sdd 8:48 active ready running  
 mpatha (1f15_disk1_id) dm-2 IET,VIRTUAL-DISK  
 size=2.0G features='0' hwhandler='0' wp=rw  
 |-+- policy='round-robin 0' prio=1 status=active  
 | `- 3:0:0:1 sda 8:0 active ready running  
 `-+- policy='round-robin 0' prio=1 status=enabled  
 `- 2:0:0:1 sdc 8:32 active ready running  
 新加入的服务器发现不了 lv ,因为它与 storage1 没有建立通信,要建立通信要建立 HA  
 [root@storage2-f15 network-scripts]# lvs  
 LV VG  
 Attr  
 LSize Pool Origin Data% Move Log Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51g  
 lv_swap VolGroup -wi-ao---- 1.00g  
 [root@storage2-f15 network-scripts]# pvs  
 PV  
 VG  
 Fmt Attr PSize PFree  
 /dev/vda2 VolGroup lvm2 a-- 9.51g 0  
 [root@storage2-f15 network-scripts]# vgs  
 VG  
 #PV #LV #SN Attr VSize VFree  
 VolGroup 1 2 0 wz--n- 9.51g 0  
 [root@storage2-f15 network-scripts]# fdisk -l /dev/mapper/mpathaDisk /dev/mapper/mpatha: 2147 MB, 2147959296 bytes  
 255 heads, 63 sectors/track, 261 cylinders  
 Units = cylinders of 16065 * 512 = 8225280 bytes  
 Sector size (logical/physical): 512 bytes / 512 bytes  
 I/O size (minimum/optimal): 512 bytes / 512 bytes  
 Disk identifier: 0x9ca54ace  
 Device Boot Start  
 End  
 Blocks Id System  
 /dev/mapper/mpathap1  
 1  
 65  
 522081 83 Linux  
 /dev/mapper/mpathap2  
 66  
 130  
 522112+ 83 Linux  
 [root@storage2-f15 network-scripts]# fdisk -l /dev/mapper/mpathb  
 Disk /dev/mapper/mpathb: 2147 MB, 2147959296 bytes  
 255 heads, 63 sectors/track, 261 cylinders  
 Units = cylinders of 16065 * 512 = 8225280 bytes  
 Sector size (logical/physical): 512 bytes / 512 bytes  
 I/O size (minimum/optimal): 512 bytes / 512 bytes  
 Disk identifier: 0x36277aa2  
 Device Boot Start  
 End  
 Blocks Id System  
 /dev/mapper/mpathbp1  
 1  
 65  
 522081 83 Linux  
 /dev/mapper/mpathbp2  
 66  
 130  
 522112+ 83 Linux  
 3. 移除原来的 lv , pv , vg  
 [root@storage1-f15 ~]# umount /opt  
 [root@storage1-f15 ~]# lvremove myvg0/mylvm  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 Do you really want to remove active logical volume mylvm? [y/n]: y  
 Logical volume "mylvm" successfully removed  
 [root@storage1-f15 ~]# vgremove myvg0  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 Volume group "myvg0" successfully removed  
 [root@storage1-f15 ~]# pvs  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 PV  
 VG  
 Fmt Attr PSize PFree  
 /dev/mapper/mpathap1  
 lvm2 a-- 509.84m 509.84m  
 /dev/mapper/mpathap2  
 lvm2 a-- 509.88m 509.88m  
 /dev/mapper/mpathbp1  
 lvm2 a-- 509.84m 509.84m  
 /dev/mapper/mpathbp2  
 lvm2 a-- 509.88m 509.88m  
 /dev/vda2  
 VolGroup lvm2 a-- 9.51g  
 0[root@storage1-f15 ~]# pvremove /dev/mapper/mpathap1 /dev/mapper/mpathap2  
 /dev/mapper/mpathbp1 /dev/mapper/mpathbp2  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sda1 not  
 /dev/mapper/mpathap1  
 Found duplicate PV IA3EwlsJ2AwpC9S8Pf1WEw6fHSemZLQy: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Labels on physical volume "/dev/mapper/mpathap1" successfully wiped  
 Labels on physical volume "/dev/mapper/mpathap2" successfully wiped  
 Labels on physical volume "/dev/mapper/mpathbp1" successfully wiped  
 Labels on physical volume "/dev/mapper/mpathbp2" successfully wiped  
 [root@storage1-f15 ~]# pvs  
 PV  
 VG  
 Fmt Attr PSize PFree  
 /dev/vda2 VolGroup lvm2 a-- 9.51g 0  
 4. 组建 HA 实现 lv 的共享  
 [root@storage1-f15 ~]# yum install ricci luci -y  
 [root@storage1-f15 ~]# chkconfig ricci on  
 [root@storage1-f15 ~]# chkconfig luci on  
 [root@storage1-f15 ~]# passwd ricci  
 [root@storage1-f15 ~]# service ricci start  
 [root@storage1-f15 ~]# service luci start  
 Start luci...  
 [ OK ]  
 Point your web browser to https://storage1-f15.example.com:8084 (or equivalent) to access luci  
 -------------------------------------------------------------------  
 [root@storage2-f15 network-scripts]# yum install ricci -y  
 [root@storage2-f15 network-scripts]# chkconfig ricci on  
 [root@storage2-f15 network-scripts]# passwd ricci  
 Changing password for user ricci.  
 New password:  
 BAD PASSWORD: it is based on a dictionary word  
 Retype new password:  
 passwd: all authentication tokens updated successfully.  
 [root@storage2-f15 network-scripts]# service ricci start  
 Starting oddjobd:  
 [ OK ]  
 generating SSL certificates... done  
 Generating NSS database... done  
 Starting ricci:  
 [ OK ]  
 --------------------------------------------------------------------------  
 在浏览器中配置 HA  
 firefox:  


 https://storage1-f15.example.com:8084

 ![]()  


 --------------------------------------------------------------------------

 

 [root@storage1-f15 ~]# clustat

 Cluster Status for cluster-f15 @ Wed Oct 12 20:00:54 2016  
 Member Status: Quorate  
 Member Name  
 ID Status  
 ------ ----  
 ---- ------  
 storage1-f15.example.com  
 1 Online, Local  
 storage2-f15.example.com  
 2 Online  
 5. 安装 cmirror 实现镜像的管理  
 [root@storage1-f15 ~]# yum install cmirror -y  
 [root@storage2-f15 ~]# yum install cmirror -y  
 [root@storage2-f15 ~]# rpm -ql cmirror  
 /etc/rc.d/init.d/cmirrord  
 /usr/sbin/cmirrord  
 /usr/share/man/man8/cmirrord.8.gz  
 [root@storage2-f15 ~]# chkconfig cmirrord on  
 [root@storage2-f15 ~]# service cmirrord start  
 Starting cmirrord:  
 [ OK ][root@storage1-f15 ~]# chkconfig cmirrord on  
 [root@storage1-f15 ~]# service cmirrord start  
 Starting cmirrord:  
 [ OK ]  
 6. 重新创建 lvm  
 [root@storage1-f15 ~]# pvcreate /dev/mapper/mpathap[1,2]  
 Physical volume "/dev/mapper/mpathap1" successfully created  
 Physical volume "/dev/mapper/mpathap2" successfully created  
 [root@storage1-f15 ~]# pvcreate /dev/mapper/mpathbp[1,2]  
 Physical volume "/dev/mapper/mpathbp1" successfully created  
 Physical volume "/dev/mapper/mpathbp2" successfully created  
 [root@storage1-f15 ~]# pvs  
 PV  
 VG  
 Fmt Attr PSize PFree  
 /dev/mapper/mpathap1  
 lvm2 a-- 509.84m 509.84m  
 /dev/mapper/mpathap2  
 lvm2 a-- 509.88m 509.88m  
 /dev/mapper/mpathbp1  
 lvm2 a-- 509.84m 509.84m  
 /dev/mapper/mpathbp2  
 lvm2 a-- 509.88m 509.88m  
 /dev/vda2  
 VolGroup lvm2 a-- 9.51g  
 0  
 [root@storage1-f15 ~]# vgcreate mycvg0 /dev/mapper/mpathap1 /dev/mapper/mpathap2  
 /dev/mapper/mpathbp1 /dev/mapper/mpathbp2  
 Found duplicate PV hkcFzeaQIjEP9flRaBTJ2BdVc5WhvUua: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV hkcFzeaQIjEP9flRaBTJ2BdVc5WhvUua: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 Clustered volume group "mycvg0" successfully created  
 [root@storage1-f15 ~]# vgs  
 Found duplicate PV hkcFzeaQIjEP9flRaBTJ2BdVc5WhvUua: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV hkcFzeaQIjEP9flRaBTJ2BdVc5WhvUua: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 VG  
 #PV #LV #SN Attr VSize VFree  
 VolGroup 1 2 0 wz--n- 9.51g 0  
 mycvg0 4 0 0 wz--nc 1.98g 1.98g  
 ----------------------------------------------------------------------------------------------------  
 [root@storage2-f15 ~]# lvcreate -m 1 -L 300M mycvg0 -n clvm0 /dev/mapper/mpathap1  
 /dev/mapper/mpathap2  
 Logical volume "clvm0" created  
 [root@storage2-f15 ~]# lvs  
 LV VG  
 Attr  
 LSize Pool Origin Data% Move Log  
 Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51g  
 lv_swap VolGroup -wi-ao---- 1.00g  
 clvm0 mycvg0 mwi-a-m--- 300.00m  
 clvm0_mlog 77.33  
 [root@storage1-f15 ~]# lvs  
 Found duplicate PV hkcFzeaQIjEP9flRaBTJ2BdVc5WhvUua: using /dev/mapper/sdb1 not  
 /dev/mapper/sda1  
 Found duplicate PV hkcFzeaQIjEP9flRaBTJ2BdVc5WhvUua: using /dev/mapper/mpathap1 not  
 /dev/mapper/sdb1  
 LV VG  
 Attr  
 LSize Pool Origin Data% Move Log  
 Cpy%Sync Convert  
 lv_root VolGroup -wi-ao---- 8.51glv_swap VolGroup -wi-ao---- 1.00g  
 clvm0 mycvg0 mwi-a-m--- 300.00m  
 clvm0_mlog 100.00  
 7. 格式化,指定文件系统,挂接 lvm  
 [root@storage2-f15 ~]# mkfs.gfs2 -j 2 -t cluster-f15:webfs -p lock_dlm /dev/mycvg0/clvm0  
 This will destroy any data on /dev/mycvg0/clvm0.  
 It appears to contain: symbolic link to `../dm-11'  
 Are you sure you want to proceed? [y/n] y  
 Device:  
 /dev/mycvg0/clvm0  
 Blocksize:  
 4096  
 Device Size  
 0.29 GB (76800 blocks)  
 Filesystem Size:  
 0.29 GB (76800 blocks)  
 Journals:  
 2  
 Resource Groups:  
 2  
 Locking Protocol:  
 "lock_dlm"  
 Lock Table:  
 "cluster-f15:webfs"  
 UUID:  
 fdd5290c-2005-f6ea-1607-38e829d5c5bd  
 [root@storage1-f15 ~]# mount /dev/mycvg0/clvm0 /opt   
 