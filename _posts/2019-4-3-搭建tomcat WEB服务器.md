---
title: 搭建tomcat WEB服务器
date: 2016-09-25 20:40:44
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52664147   
  实验环境  
 网关 classroom 172.25.8.254  
 workstation 172.25.8.9   
 server a-j eth0 172.25.8.10- 外网  
 eth1 192.168.0.x 内网  
 eth2 192.168.1.x 备用  
 --------------------------------------------  
 需求：  
 设计：  
 原理：  
 硬件：  
 系统：  
 软件：  
 服务：  
 部署：   
 安装与部署tomcat服务器  
 cd /mnt/ula/tomcat  
 ls   
 rpm -ivh jdk-7u79-linux-x64.rpm  
 id tomcat  
 groupadd tomcat -g 518  
 useradd tomcat -u 518 -g tomcat   
 id tomcat  
 tar -xf apache-tomcat-8.0.24.tar.gz -C /home/tomcat  
 cd /home/tomcat/  
 ls  
 chown tomcat. apache-tomcat-8.0.24/ -R  
 cd apache-tomcat-8.0.24/  
 ls  
 rpm -ql jdk |head -n 20  
 export JAVA_HOME="/usr/java/jdk1.7.0_79"  
 ls bin  
 bin/startup.sh   
 测试：172.25.8.11:8080  
 以上是以root身份启动的tomcat  
 -------------------------------------  
 以tomcat用户的身份启动tomcat服务  
 第 2.2 节 优化 tomcat 服务启动  
 上 述 启 动 方 法 存 在 以 下 问 题 ( 1 ) 启 动 和 关 闭 tomcat 服 务 需 进 入 到 tomcat 主 程 序 中 bin 目 录 找  
 startup.sh 和 shutdown.sh 脚本,不方便。(2)tomcat 进程是由 root 用户打开并维护的,从安全角度  
 考虑存在缺陷。  
 如果希望可以通过 serveice 命令等方式方便的启动,并且希望是 tomcat 用户维护 tomcat 服务正常运行,  
 需安装 jsvc 命令。  
 (1)进入 tomcat 主程序所在目录的子目录 bin,该目录下有 common-daemon-native 软件包,通过  
 该软件安装 jsvc 命令。该软件需通过源代码安装方式安装。  
 1.1 解压该源代码压缩包至当前目录,进入解压目录中子目录 unix 目录,执行 configure 检测程序。  
 [root@serverc bin]# cd /home/tomcat/apache-tomcat-8.0.24/bin/  
 [root@serverc bin]# tar -xf commons-daemon-native.tar.gz  
 [root@serverc bin]# cd commons-daemon-1.0.15-native-src/unix/  
 [root@serverc unix]# ./configure  
 1.2 检测过程中可能会出现很多问题,需逐一解决。 例如:下图中由于没有 c 语言编译器导致检测未完成,  
 出现 error。可通过 yum 程序安装 gcc 软件包解决。  
 [root@ser verc unix]# ./configure  
 *** Current host ***checking build system type... x86_64-unknown-linux-gnu  
 checking host system type... x86_64-unknown-linux-gnu  
 checking cached host system type... ok  
 *** C-Language compilation tools ***  
 checking for gcc... no  
 checking for cc... no  
 checking for cc... no  
 checking for cl... no  
 configure: error: no acceptable C compiler found in $PATH  
 See `config.log' for more details.  
 [root@ser verc unix]# yum -y install gcc  
 1.3 修改上述问题后重新检测,检测已通过。但是中途出现 warning,出现 warning 不会中断检测程序,  
 故可以忽略。如果想解决,可查看报错,找除解决方法。 例如:下图中由于没有 libcap 相关程序导致警告。  
 [root@serverc unix]# ./configure  
 *** C-Language compilation tools ***  
 checking for gcc... gcc  
 checking for C compiler default output file name... a.out  
 checking for sys/capability.h... no  
 configure: WARNING: cannot find headers for libcap  
 *** Writing output files ***  
 configure: creating ./config.status  
 config.status: creating Makefile  
 config.status: creating Makedefs  
 config.status: creating native/Makefile  
 *** All done ***  
 Now you can issue "make"  
 1.4 安装 libcap-devel 软件包解决该问题。一般在源代码安装软件过程中,需安装的大多为 devel 包,即  
 程序的开发包。  
 [root@serverc unix]# yum -y install libcap-devel  
 1.5 解决该报警之后,再次执行 configure 检测程序完成检测。检测通过后会生成 Makefile 文件以记录检  
 测结果。(此处省略)  
 1.6 执行 make 程序编译,将源代码编译成二进制。编译过程中会调用 Makefile 文件。  
 [root@serverc unix]# make  
 (cd native; make all)  
 make[1]:  
 Entering  
 directory  
 `/home/tomcat/apache-tomcat-8.0.24/bin/commons-daemon-1.0.15-native-  
 src/unix/native'  
 gcc  
 -g  
 -O2  
 -DOS_LINUX  
 -DDSO_DLFCN  
 -DCPU=\"amd64\"  
 -Wall  
 -Wstrict-prototypes  
 -I/usr/java/jdk1.7.0_79//include -I/usr/java/jdk1.7.0_79//include/linux -c jsvc-unix.c -o jsvc-unix.o  
 1.7 编译完成后,该目录下会安装 jsvc 命令,将该命令拷贝至/home/tomcat/apache-tomcatxxx/bin 目  
 录下。因为该目录下有 daemon.sh 文件,执行该文件时会自动调用 jsvc 命令。  
 -DHAVE_LIBCAP[root@serverc unix]# cp jsvc /home/tomcat/apache-tomcat-8.0.24/bin/  
 [root@serverc unix]# cd /home/tomcat/apache-tomcat-8.0.24/bin/  
 [root@serverc bin]# grep jsvc daemon.sh  
 # Setup parameters for running the jsvc  
 # If not explicitly set, look for jsvc in CATALINA_BASE first then CATALINA_HOME  
 JSVC="$CATALINA_BASE/bin/jsvc"  
 JSVC="$CATALINA_HOME/bin/jsvc"  
 1.8 编辑 daemon.sh 文件,将之前临时声明的 JAVA_HOME、CATALINA_HOME、CATALINA_BASE 变量  
 再该文件中声明,以便永久生效,并且可以在启动 tomcat 服务时自动声明。  
 [root@serverc ~]# vim /home/tomcat/apache-tomcat-8.0.24/bin/daemon.sh  
 # resolve links - $0 may be a softlink  
 export JAVA_HOME="/usr/java/jdk1.7.0_79/"  
 export CATALINA_HOME="/home/tomcat/apache-tomcat-8.0.24/"  
 export CATALINA_BASE="/home/tomcat/apache-tomcat-8.0.24/"  
 ARG0="$0"  
 (2)将文件复制到 /etc/init.d/目录下,以后就可以方便使用 rhel6 版本中服务的状态控制方式控制  
 tomcat 服务。  
 前一章节 tomcat 服务器基础搭建操作步骤中我们启动过 tomcat 服务,该进程已经存在,所以在重启之  
 前先将 tomcat 进程停掉。  
 再通过/etc/init.d/tomcat start 方式或者 service tomcat start 方式启动该服务。  
 对比发现,通过这种方式启动之后,tomct 进程的拥有者为 tomcat 用户。(当前,root 用户需要先打开  
 tomcat 进程,然后再由 tomcat 用户创建子进程。root 用户打开的 tomcat 进程不参与服务运行。)  
 [root@serverc ~]# cp /home/tomcat/apache-tomcat-8.0.24/bin/daemon.sh /etc/init.d/tomcat  
 [root@serverc ~]# ps -ef | grep java  
 root  
 1102  
 1  
 0  
 14:24  
 ?  
 00:00:06  
 /usr/java/jdk1.7.0_79//bin/java  
 -Djava.util.logging.config.file=/home/tomcat/apache-tomcat-8.0.24//conf/logging.properties  
 -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endor ...  
 [root@serverc ~]# kill 1102  
 [root@serverc ~]# ps -ef | grep java  
 root  
 4146 3979 0 14:40 pts/1  
 00:00:00 grep --color=auto java  
 [root@serverc ~]# /etc/init.d/tomcat start  
 [root@serverc ~]# ps -ef | grep java  
 root  
 4157  
 1 0 14:40 ?  
 00:00:00 jsvc.exec -java-home /usr/java/jdk1.7.0_79/ -user tomcat -pidfile  
 /home/tomcat/apache-tomcat-8.0.24//logs/catalina-daemon.pid -wait 10 -outfile ...  
 tomcat  
 4158 4157 58 14:40 ?  
 00:00:02 jsvc.exec -java-home /usr/java/jdk1.7.0_79/ -user tomcat -pidfile  
 /home/tomcat/apache-tomcat-8.0.24//logs/catalina-daemon.pid -wait 10 -outfile /home/tomcat/apache-tomcat-  
 8.0.24//logs/catalina-daemon.out -errfile &1 -classpath /ho...  
 (3)在 client 端 workstation 机器上访问 tomcat 服务器。访问后发现出现空白页(图省略)。此处是由  
 于 tomcat 主程序所在目录下 UGO 权限导致的,该目录需要 tomcat 用户有完整权限。可以通过改变目录拥有者方式达到此要求。重启 tomcat 服务后,再次测试,即看到 tomcat 默认首页文件。  
 [root@serverc ~]# cd /home/tomcat/  
 [root@serverc tomcat]# chown tomcat * -R  
 [root@serverc tomcat]# /etc/init.d/tomcat stop  
 [root@serverc tomcat]# /etc/init.d/tomcat start  
 ---------------------------------  
 虚拟主机基本配置  
 (1)修 改 tomcat 服 务 的 主 配 置 文 件 。 该 文 件 位 于 tomcat 主 程 序 下 conf 目 录 中 , 文 件 名 称 为  
 server.xml。  
 [root@serverc ~]# cd /home/tomcat/apache-tomcat-8.0.24/conf/  
 [root@serverc conf]# ls -l server.xml  
 -rw------- 1 tomcat root 6458 Jul 2 04:23 server.xml  
 (2)编辑该文件,添加虚拟主机。host 字段为 tomcat 服务的虚拟主机配置字段。下图中配置了两个虚拟  
 主机,www.tomcat1.com 和 www.tomcat2.com,两台虚拟主机有自己的 appbase(即网页文件根目  
 录,下图中使用的是相对路径表示虚拟主机网页根目录,该路径是相对于 tomcat 服务主程序所在目录来  
 说,该路径现不存在,后续再创建)和相关日志前缀。注意:严格遵循该文件的语法格式,每个虚拟主机  
 的配置都是<host>开始,</host>结束。  
 <Host name="www.tomcat1.com" appBase="tomcat1.com"  
 unpackWARs="true" autoDeploy="true">  
 <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"  
 prefix="localhost_access_log" suffix=".txt"  
 pattern="%h %l %u %t "%r" %s %b" />  
 </Host>  
 <Host name="www.tomcat2.com" appBase="tomcat2.com"  
 unpackWARs="true" autoDeploy="true">  
 <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"  
 prefix="localhost_access_log" suffix=".txt"  
 pattern="%h %l %u %t "%r" %s %b" />  
 </Host>  
 (3)进入 tomcat 服务主程序所在目录,创建上述步骤中虚拟主机所指定的 appBase,分别进入每一个  
 虚拟主机的 appBase 目录下创建 ROOT 目录,在 ROOT 目录写每一个虚拟主机对应的首页文件。。  
 [root@serverc conf]# cd /home/tomcat/apache-tomcat-8.0.24/  
 [root@serverc apache-tomcat-8.0.24]# mkdir tomcat1.com/  
 [root@serverc apache-tomcat-8.0.24]# cd tomcat1.com/  
 [root@serverc tomcat1.com]# mkdir ROOT  
 [root@serverc tomcat1.com]# echo tomcat1 > ROOT/index.html  
 [root@serverc tomcat1.com]# cd ../  
 [root@serverc apache-tomcat-8.0.24]# mkdir tomcat2.com  
 [root@serverc apache-tomcat-8.0.24]# cd tomcat2.com  
 [root@serverc tomcat2.com]# mkdir ROOT  
 [root@serverc tomcat2.com]# echo tomcat2 > ROOT/index.html  
 (4)重启 tomcat 服务  
 [root@serverc tomcat2.com]# /etc/init.d/tomcat stop[root@serverc tomcat2.com]# /etc/init.d/tomcat start  
 (5)在 client 端 workstation 机器上测试  
 [root@workstation ~]# echo 172.25.41.10 www.tomcat1.com >> /etc/hosts  
 [root@workstation ~]# echo 172.25.41.10 www.tomcat2.com >> /etc/hosts  
 ------------------------------------  
 nginx 代理访问 tomcat 服务  
 通过上述配置,我们会发现,在客户端访问过程中,需要在$host 后添加上需要访问的端口号,略麻烦,  
 我们希望客户端访问的都是 http 协议默认的 80 端口,但是可以访问到后台多台服务器。这个时候我们可  
 以引入 nignx,让 nginx 作为代理服务器去代理用户请求。也就是说客户端找的是 nginx 服务,nginx 服  
 务默认监听 80 端口,然后 nginx 将用户请求转交给后台监听不同端口的 tomcat 虚拟主机。  
 [root@serverb ~]# cd /etc/nginx/conf.d/  
 [root@serverb conf.d]# cp default.conf www.tomcat.com.conf  
 [root@serverb conf.d]#vim /etc/nginx/conf.d/www.tomcat.com.conf  
 server {  
 listen  
 80;  
 server_name ~www\.tomcat.*\.com;  
 #charset koi8-r;#access_log /var/log/nginx/log/host.access.log main;  
 location / {  
 index index.html index.htm;  
 proxy_pass http://172.25.0.12:8080;  
 proxy_set_header Host $host;  
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
 proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504 http_404;  
 proxy_set_header X-Real-IP $remote_addr;  
 proxy_redirect off;  
 }  
 ---------------------------------------  
 用tomcat搭建一个论坛：  
 Tomcat+db  
 第 4.1 节 cgi 连接 db 过程  
   
 有关数据库服务器信息写到文件中(php 程序需要写到 php 代码文件中,jsp 可以写到代码  
 文件中,也可以写到配置文件中):数据库服务器的 ip 地址,主机名、用户名、密码、库名  
  驱动:php+db 使用 php-mysql 驱动,tomcat+db 使用 java-connector-java 驱动  
  db 授权:允许 cgi 所在服务器读取 db 中信息  
 第 4.1 节 配置过程  
 (1) 进入服务器公共目录,下载 ejforum 软件包  
 [root@serverc ~]# cd /mnt/items/tomcat/  
 [root@serverc tomcat]# cp ejforum-2.3.zip /opt/  
 (2)解压 ejforum 软件包,并将 jsp 文件拷贝到网站根目录下。  
 [root@serverc opt]# unzip ejforum-2.3.zip  
 [root@serverc opt]# cd ejforum-2.3/  
 [root@serverc ejforum-2.3]# cp -r ejforum/* /home/tomcat/apache-tomcat-8.0.24/tomcat1.com/ROOT/  
 (3)设置 UGO 权限,使 tomcat 用户对网站根目录下文件具有完整权限。  
 [root@serverc ejforum-2.3]# chown -R tomcat /home/tomcat/apache-tomcat-8.0.24/tomcat1.com/ROOT/  
 (4)安装驱动。mysql-connector-java 程序解压后,把 mysql-connector-java-5.1.36-bin.jar 文件拷  
 贝到 tomcat 安装主目录下的 lib 目录。  
 [root@serverc ~]# cd /mnt/items/tomcat/  
 [root@serverc tomcat]# cp mysql-connector-java-5.1.36.tar.gz /opt/  
 [root@serverc opt]# cd mysql-connector-java-5.1.36/  
 [root@serverc  
 mysql-connector-java-5.1.36]#  
 cp  
 mysql-connector-java-5.1.36-bin.jar  
 /home/tomcat/apache-tomcat-  
 8.0.24/lib/  
 (5)配置数据库相关信息  
 [root@serverc ~]# cd /home/tomcat/apache-tomcat-8.0.24/tomcat1.com/ROOT/WEB-INF/  
 [root@serverc WEB-INF]# vim conf/config.xml  
 <database maxActive="10" maxIdle="10" minIdle="2" maxWait="10000"  
 username="javaadmin" password="uplooking"  
 driverClassName="com.mysql.jdbc.Driver"  
 url="jdbc:mysql://172.25.0.18:3306/javabbs?  
 characterEncoding=gbk&autoReconnect=true&autoReconnectForPools=true&zeroDateTimeBehavior=conve  
 rtToNull"  
 sqlAdapter="sql.MysqlAdapter"/>  
 (6)将 ejforum 软件包中 easyjforum_mysql 中的数据导入到数据库服务器 serveri 的 javabbs 库中。[root@serveri ~]#mysqladmin -u root password "uplooking  
 [root@serveri ~]# mysqladmin -uroot -puplooking create javabbs  
 [root@serveri ~]# mysql -uroot -puplooking javabbs < /tmp/install/script/easyjforum_mysql.sql  
 (7)在数据库服务器上给 tomcat 服务器授权  
 mysql  
 >create database javabbs;  
 >grant all on javabbs.* to javaadmin@'172.25.0.12' identified by 'uplooking';  
 根据网站的要求在数据库中建立表，建表通过网页文件中的脚本进行  
 serverb:  
 cd /tmp/ejforum-2.3/install/script/  
 scp easyjforum_mysql.sql 172.25.8.19:/tmp  
 serverj:  
 mysql javabbs</tmp/easyjforum_mysql.sql  
 (8)重启 tomcat 服务  
 [root@serverc ~]# /etc/init.d/tomcat stop  
 [root@serverc ~]# /etc/init.d/tomcat start  
 (9)客户端访问 tomcat 服务器,可以看  
  
   
 