---
title: Linux安装 VMware tools 工具的方法
date: 2018-12-18 14:11:29
tags: CSDN迁移
---
  Linux安装 VMware tools 工具的方法

 VMware虚拟机中如何安装VMWare-Tools详解好处：可以支持图形界面，可以支持共享文件功能等

 1 工具/原料   
 1）安装过虚拟机软件的计算机   
 2）linux操作系统   
 3）虚拟机配置VMware tools文件，   
 点击工具栏上的【虚拟机】，然后选择【虚拟机设置】，再选择CD/DVD(IDE)硬件，使用ISO映象文件，选择如下目录（我虚拟机是安装在C盘的）   
 C:\Program Files (x86)\VMware\VMware Workstation\linux.iso

 2 方法/步骤

 1)以root身份登陆计算机   
 2)开始安装Vmware 选择VM–>install VMware Tools   
 3)输入如下命令   
 [root@localhost ~]# mkdir /mnt/cdrom   
 输入 注意空格   
 [root@localhost ~]#mount /dev/cdrom /mnt/cdrom/   
 4)   
 [root@localhost ~]# cd /mnt/cdrom/   
 [root@localhost cdrom]# ls 显示其下有哪些文件，类似 windows中的dir   
 5)将文件拷贝至根目录下的tmp这个临时目录下   
 //拷贝到/tmp下这里如果VMware的版本不同出现的数字也是不同的 ，不过差不过，至于这串字符你要是怕输错了，可以复制粘贴 右击   
 [root@localhost cdrom]# cp VMwareTools-7.8.4-126130.tar.gz /tmp   
 6)进入tmp文件夹   
 [root@localhost cdrom]# cd /tmp/   
 [root@localhost tmp]# tar zxvf VMwareTools-6.5.0-118166.tar.gz //解压文件   
 7)安装开始   
 [root@localhost tmp]# cd vmware-tools-distrib   
 [root@localhost tmp]# ls   
 [root@localhost vmware-tools-distrib]# ./vmware-install.pl //安装开始   
 8)   
 最后用“./install.pl”命令来运行该安装程序，然后根据屏幕提示一路回车。到此整个安装过程算是完成了。

 最后一步 好像提示有错 我把虚拟机重启了一次 然后就可以复制了

   
 