---
title: Nginx web服务器搭建
date: 2016-09-10 13:58:07
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52494692   
  实验环境  
 网关 classroom 172.25.8.254  
 workstation 172.25.8.9   
 server a-j eth0 172.25.8.10- 外网  
eth1 192.168.8.x 内网  
eth2 192.168.8.x 备用  
 --------------------------------------------  
 项目-Nginx  
 需求：搭建一台Nginx的web服务器  
 设计：  
原理：  
处理用户请求：调度cpu 分配内存  
进程：内存独享  apache  
线程：内存共享 线程中相同的内容共用内存 优点：内存占用少 缺点：读写时的冲突 解决:锁机制 nginx  
选用：用户数量少，选择进程模式 用户数量高，线程模式  
/etc/nginx  
vim nginx.conf #主配置文件  
 ------------------------------------------------------------------------  
work_processes 4; 进程数  
work_connettions 4096; 每个进程的线程数，最大可以65535，但是不能开太高，冲突的概率增大，  
 ------------------------------------------------------------------------  
此时最大并发数4x4096  
提升可以增加进程数  
性能调优，一般能提升10%，所以用在服务器数量较多时，数千台以上的机器  
机器数量少时不必考虑性能调优  
nginx内核模型 进程是异步模式进行的  
 -------------------------------------------------------------------------------  
硬件：Linux服务器  
系统：rhel7  
软件：nginx  
服务：nginx  
 部署：  
 mount 172.25.254.250:/content /mnt #挂载远程目录  
 cd /mnt/ula/nginx/nginx-rpms/ #rpm包所在位置  
 ls  
 nginx-1.8.0-1.el7.ngx.x86_64.rpm php-fpm-5.4.16-23.el7_0.3.x86_64.rpm  
 nginx-1.8.1-1.el7.ngx.x86_64.rpm spawn-fcgi-1.6.3-5.el7.x86_64.rpm  
 yum install -y nginx-1.8.1-1.el7.ngx.x86_64.rpm  #安装最新1.8.1版本  
 systemctl start nginx #启动nginx服务  
 --------------------------------------------------------------  
 /etc/nginx  
vim nginx.conf #主配置文件  
 /etc/nginx/conf.d/  
vim www.abc.com.conf #新建从配置文件新建虚拟主机  
 server {  
listen  80;  
server_name www.abc.com;  
root /usr/share/nginx/abc.com;  
index index.html index.htm;  
 }  
 server_name xx.xxx.xxx #网站名支持通配符和正则表达式  
 ~正则  #正则要加～号  
 例如 *.abc.com #当bbs.abc.com也能访问到  
 mkdir /usr/share/nginx/abc.com #建立网站主目录  
 echo hello >/usr/share/nginx/abc.com/index.html  #建立index.html  
 systemctl start nginx #启动服务  
 -----------------------------------------------------------------------  
 vim noservername.conf #  
 server{  
listen 80 default;  
root /usr/share/nginx/noserver.com;  
index index.html;  
 }  
 mkdir /usr/share/nginx/noserver.com  
 echo noserver > noserver.com/index.html  
 -------------------------------------------------------------------------  
 server {  
 listen 80;  
 server_name *.abc.com;  
 location / {  
 root /usr/share/nginx/abc.com;  
 index index.html index.htm;  
 }  
  
  
 location ~ /test1 {  
 root /usr/share/nginx/test1.com;  
 index index.html index.htm;  
 }  
  
  
 location ~ /test2 {  
 root /usr/share/nginx/test2.com;  
 index index.html index.htm;  
 }  
 }  
 ---------------  
 访问不同目录有不同页面  
 www.abc.com/test1/index.html  
 www.abc.com/test2/index.html  
 location最先匹配到 ～ 最后匹配 /  
 -------------------------------------------------  
 server {  
 listen 80;  
 server_name *.abc.com;  
 root /usr/share/nginx/abc.com;  
 index index.html index.htm;  
  
  
 location ~ /test1 {  
 root /usr/share/nginx/test1.com;  
 index index.html index.htm;  
 }  
  
  
 location ~ /test2 {  
 root /usr/share/nginx/test2.com;  
 index index.html index.htm;  
 }  
 }  
 --------------------  
 localtion外的root / 对全局生效  
 ---------------------------------------------------  
 访问控制  
 vim /etc/nginx/conf.d/www.abc.com.conf  
 -------------  
 server {  
 listen 80;  
 server_name *.abc.com;  
location /{  
 root /usr/share/nginx/abc.com;  
 index index.html index.htm;  
auth_basic "xxxx";   
auth_basic_user_file /etc/nginx/passwd.db;  #访问控制文件  
}  
  
  
 location ~ /test1 {  
 root /usr/share/nginx/test1.com;  
 index index.html index.htm;  
 }  
  
  
 location ~ /test2 {  
 root /usr/share/nginx/test2.com;  
 index index.html index.htm;  
 }  
 }  
 ----------------------  
 [root@servera conf.d]# htpasswd -c /etc/nginx/passwd.db chen  
 New password:   
 Re-type new password:   
 Adding password for user chen  
 [root@servera conf.d]# systemctl restart nginx #重启服务  
  
  
 想阻止别人访某些目录下的特定文件,比如我网站主目录下有个 test 目录,我不想让别人访问我 test 目录下的”.txt”  
 和”.doc”的文件,那么我们可以通过 deny 的方式来做拒绝。  
 location ~* \.(txt|doc)$ {  
 deny all;  
 }  
 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
 https  
 -------------------------------  
 众所周知,我们在互联网上冲浪,一般都是使用的 http 协议(超文本传输协议),默认情况下数据是明文传送的,这些数据在  
 传输过程中都可能会被捕获和窃听,因此是不安全的。https 可以说是 http 协议的安全版,就是为了满足对安全性要求比较高  
 的用户而设计的。  
 如果您的邮件中有敏感数据,不希望被人窃听;如果您不希望被钓鱼网站盗用帐号信息,如果您希望您在使用邮箱的过程中更  
 安全,那么我们推荐您使用 https 安全连接。  
 HTTPS 在 HTTP 的基础上加入了 SSL 协议,SSL 依靠证书来验证服务器的身份,并为浏览器和服务器之间的通信加密。SSL  
 (Secure Socket Layer 安全套接层)为 Netscape 所研发,用以保障在 Internet 上数据传输之安全,利用数据加密技术,可确保  
 数据在网络上之传输过程中不会被截取及窃听。数据密码方式常见的有三种  
 (1)对称加密:采用单钥密码系统的加密方法,同一个密钥可以同时用作信息的加密和解密,这种加密方法称为对称加密,也  
 称为单密钥加密。服务端生成公钥和私钥,服务端将公钥传递给客户端,客户端通过公钥加密自己的数据后传递给服务器。  
 举例来说:甲和乙是一对生意搭档,他们住在不同的城市。由于生意上的需要,他们经常会相互之间邮寄重要的货物。为了保  
 证货物的安全,他们商定制作一个保险盒,将物品放入其中。他们打造了两把相同的钥匙分别保管,以便在收到包裹时用这个  
 钥匙打开保险盒,以及在邮寄货物前用这把钥匙锁上保险盒。只要甲乙小心保管好钥匙,那么就算有人得到保险盒,也无法打  
 开。  
 对称加密算法的优点是算法公开、计算量小、加密速度快、加密效率高。  
 对称加密算法的缺点是在数据传送前,发送方和接收方必须商定好秘钥,然后使双方都能保存好秘钥。其次如果一方的秘钥被  
 泄露,那么加密信息也就不安全了。另外,每对用户每次使用对称加密算法时,都需要使用其他人不知道的唯一秘钥,这会使  
 得收、发双方所拥有的钥匙数量巨大,密钥管理成为双方的负担。  
 (2)非对称加密  
 非对称加密算法需要两个密钥来进行加密和解密,公钥和私钥。还是上述例子,乙方生成一对密钥并将公钥向甲方公开,得到  
 该公钥的甲方使用该密钥对机密信息进行加密后再发送给乙方。乙方再用自己保存的私钥对加密后的信息进行解密。乙方只能  
 用私钥解密由对应的公钥加密后的信息。同样,如果乙要回复加密信息给甲,那么需要甲先公布甲的公钥给乙用于加密,甲自  
 己保存甲的私钥用于解密。在传输过程中,即使攻击者截获了传输的密文,并得到了乙的公钥,也无法破解密文,因为只有乙  
 的私钥才能解密密文。  
 非对称加密与安全性更好,使用一对秘钥,一个用来加密,一个用来解密,而且公钥是公开的,秘钥是自己保存的。  
 19Nginx 配置  
 非对称加密的缺点是加密和解密花费时间长、速度慢,只适合对少量数据进行加密。  
 (3)单向加密  
 加密方式：非对称式加密 服务器：私钥 客户端：公钥  
 CA认证也叫做证书  
 客户端通过https访问时会检测证书，经过认证的才能够被访问  
 如何进行CA认证  
 1）创建server的私钥  
 2）创建证书颁发的请求  
 -----------------  
 3）创建CA中心  
 4）CA去签名证书  
 -----------------  
 5）服务器配置https  
 ##生产环境中需要做的是1）2）5）这三步  
  
  
 1）创建server的私钥  
 openssl genrsa 2048 > serverb-web.key  
  
  
 2）创建证书颁发的请求  
 [root@servera ca]# openssl req -new -key servera-web.key -out servera-web.csr  
 You are about to be asked to enter information that will be incorporated  
 into your certificate request.  
 What you are about to enter is what is called a Distinguished Name or a DN.  
 There are quite a few fields but you can leave some blank  
 For some fields there will be a default value,  
 If you enter '.', the field will be left blank.  
 -----  
 Country Name (2 letter code) [XX]:CN #国家  
 State or Province Name (full name) []:shanghai  
 Locality Name (eg, city) [Default City]:shanghai  
 Organization Name (eg, company) [Default Company Ltd]:uplooking  
 Organizational Unit Name (eg, section) []:uplooking  
 Common Name (eg, your name or your server's hostname) []:www.abc.com 要认证的虚拟主机名  
 Email Address []:root@xxx.com  
  
  
 Please enter the following 'extra' attributes  
 to be sent with your certificate request  
 A challenge password []: #不用写  
 An optional company name []: #不用写  
 ----  
 将请求提交给ca  
 scp servera-web.csr root@172.25.8.9:/root/ca  
  
  
 -----------------  
 3）创建CA中心  
 [root@workstation ~]# mkdir ca  
 [root@workstation ~]# cd ca  
 [root@workstation ca]# pwd  
 /root/ca  
 [root@workstation ca]# openssl genrsa -des3 -out ca.key 4096  
 Generating RSA private key, 4096 bit long modulus  
 ......................................................................................................++  
 ...................................................................................................................................................................................++  
 e is 65537 (0x10001)  
 Enter pass phrase for ca.key:  
 Verifying - Enter pass phrase for ca.key:  
 [root@workstation ca]# ls  
 ca.key servera-web.csr  
  
  
 [root@workstation ca]# openssl req -new -x509 -days 3650 -key ca.key -out ca.crt  
 Enter pass phrase for ca.key:  
 You are about to be asked to enter information that will be incorporated  
 into your certificate request.  
 What you are about to enter is what is called a Distinguished Name or a DN.  
 There are quite a few fields but you can leave some blank  
 For some fields there will be a default value,  
 If you enter '.', the field will be left blank.  
 -----  
 Country Name (2 letter code) [XX]:CN  
 State or Province Name (full name) []:shanghai  
 Locality Name (eg, city) [Default City]:shanghai  
 Organization Name (eg, company) [Default Company Ltd]:shanghai  
 Organizational Unit Name (eg, section) []:shanghai ca  
 Common Name (eg, your name or your server's hostname) []:ca.shanghai.com  
 Email Address []:root@xxx.com  
  
  
 4）CA去签名证书  
  
  
 [root@workstation ca]# openssl x509 -req -days 365 -in servera-web.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out servera-web.crt  
 Signature ok  
 subject=/C=CN/ST=shanghai/L=shanghai/O=uplooking/OU=uplooking/CN=www.abc.com/emailAddress=root@xxx.com  
 Getting CA Private Key  
 Enter pass phrase for ca.key:  
  
  
 scp servera-web.crt root@172.25.8.10:/root/ca  #将签名过的公钥发送给servera  
 -----------------  
 5）服务器配置https  
 cp servera-web.crt servera-web.key /etc/nginx #将公钥和私钥cp到/etc/nginx/目录  
 chmod 400 servera-web.key #修改私钥的权限为只有root可读  
 /etc/nginx/conf.d  
 cp example_ssl.conf https.abc.com.conf  
 vim https.abc.com.conf  
 --------------------------------  
 # HTTPS server  
 #  
 server {  
 listen 443 ssl;  
 server_name www.abc.com; #虚拟主机名  
  
  
 ssl_certificate /etc/nginx/servera-web.crt;  
 ssl_certificate_key /etc/nginx/servera-web.key;  
  
  
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #启用TLs加密，并关闭ssl加密 因为ssl存在安全漏洞  
 ssl_session_cache shared:SSL:1m;  
 ssl_session_timeout 5m;  
  
  
 ssl_ciphers HIGH:!aNULL:!MD5;  
 ssl_prefer_server_ciphers on;  
  
  
 location / {  
 root /usr/share/nginx/abc.com;  
 index index.html index.htm;  
 }  
 }  
 -------------------------  
 systemctl restart nginx #重启服务  
 测试（浏览器导入ca认证 ca.crt)  
  
  
  
  
  
  
   
  
  
  
  
  
  
 问题解答：  
 1)、  
 [root@servera nginx-rpms]#systemctl start nginx #启动服务时报错  
 Job for nginx.service failed. See 'systemctl status nginx.service' and 'journalctl -xn' for details.  
 ---------------------------------------------------------------------------------------------------  
 [root@servera nginx-rpms]# systemctl status nginx.service #查看调试信息  
 Sep 07 22:53:06 servera.pod8.example.com nginx[2176]: nginx: configuration file /etc/nginx/nginx.conf test...ful  
 Sep 07 22:53:06 servera.pod8.example.com nginx[2178]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Addr...se)  
 Sep 07 22:53:06 servera.pod8.example.com nginx[2178]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Addr...se)  
 Sep 07 22:53:07 servera.pod8.example.com nginx[2178]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Addr...se)  
 Sep 07 22:53:07 servera.pod8.example.com nginx[2178]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Addr...se)  
 Sep 07 22:53:08 servera.pod8.example.com nginx[2178]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Addr...se)  
 Sep 07 22:53:08 servera.pod8.example.com nginx[2178]: nginx: [emerg] still could not bind()  
 Sep 07 22:53:08 servera.pod8.example.com systemd[1]: nginx.service: control process exited, code=exited status=1  
 Sep 07 22:53:08 servera.pod8.example.com systemd[1]: Failed to start nginx - high performance web server.  
 Sep 07 22:53:08 servera.pod8.example.com systemd[1]: Unit nginx.service entered failed state.  
 Hint: Some lines were ellipsized, use -l to show in full.  
 错误显示80端口无法绑定  
 原因：apache占用80端口  
 解决方法：停止apache服务httpd  
 systemctl stop httpd #停止httpd服务  
 systemctl start nginx #启动服务  
  
   
 