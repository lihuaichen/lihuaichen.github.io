---
title: bugzilla备份的方法
date: 2017-11-28 15:31:07
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/78655640   
  ```
1、导出：首先去需要备份的机器上导出数据库(我安装时数据库的名称为bugs)：

mysqldump -uroot -p 数据库名称 >导出数据库存放的位置/导出后你给这个数据库命名.sql
例如：mysqldump -uroot -p bugs >/root/2016bugs.sql
输入密码
即可。
到你输入的导出位置即可看到你导出的文件：2016bugs.sql
导入：将其拷贝到你需要导入的机器的某一个位置，例如/root下
输入：mysql -uroot -p   bugs < /tmp/2017bugs.sql

重启服务：
systemctl restart httpd.service    
systemctl restart mariadb.service
再次打开导入机器的bugzilla，即可看到导入的全部内容。
```
  
   
 