---
title: Linux-破解rhel7-root密码
date: 2016-08-31 23:22:06
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390124   
  破解7的密码  
1.linux16 rd.break  
2.mount -o remount,rw /sysroot  
3.chroot /sysroot  
4.passwd  
5.touch /.autorelabel  
exit  
exit  
7版本grub菜单加密  
1.grub2-mkpasswd-pbkdf2  
2.vi /etc/grub.d/40_custom  
set superusers="root"  
password_pbkdf2 root grub.pbkdf2.sha512.10000.7669206D8171A48FC040586CA67347D84BF910206DCCE86CC3D027DF844B023DB53648C25BD3337C2FF5F423D868E04D5DF84363D53F731D7E884D3CF6EB2943.F548BCE84B28829C3C06E302D6F2332D0FC607BCAE1EFE6936BD94BE7A61B90966DA0ADB2F17AE4EA3160D15DF6F68665F34490A108FCCBED1631C13B997F71A  
3.grub2-mkconfig -o /boot/grub2/grub.cfg   
 