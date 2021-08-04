---
title: rewrite、nginx proxy反向代理和缓存
date: 2016-09-10 13:59:49
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52494696   
  实验环境  
 网关 classroom 172.25.8.254  
 workstation 172.25.8.9   
 server a-j eth0 172.25.8.10- 外网  
eth1 192.168.0.x 内网  
eth2 192.168.1.x 备用  
 --------------------------------------------  
 rewrite地址重写技术  
 需求： 网页的地址也希望看起来尽量简洁明快  
搜索引擎更喜欢静态页面形式的网页,搜索引擎对静态页面的评分一般要高于动态页面。所以,UrlRewrite 可  
 以让我们网站的网页更容易被搜索引擎所收录。  
从安全角度上讲,如果在 url 中暴露太多的参数,无疑会造成一定量的信息泄漏,可能会被一些黑客利用,对你的系统造  
 成一定的破坏,所以静态化的 url 地址可以给我们带来更高的安全性。  
 设计：  
原理：1）匹配  
   
location：~正则  对path匹配  
if：if(匹配的内容) { }  可以匹配除路径以外的其他信息,如主机名、客户端 ip 等。  
if 可以支持如下条件判断匹配符号  
~ 为区分大小写匹配  
~* 为不区分大小写匹配  
!~和!~*分别为区分大小写不匹配及不区分大小写不匹配  
-f 和!-f 用来判断是否存在文件  
-d 和!-d 用来判断是否存在目录  
-e 和!-e 用来判断是否存在文件或目录  
-x 和!-x 用来判断文件是否可执行  
2）重写  
{rewrite 旧地址 新地址 标记位}  
每行 rewrite 指令最后应该根一个 flag 标记,支持的 flag 标记有,其中最为常用的有 last、break、permanent  
last  
相当于 Apache 里的[L]标记,表示完成 rewrite  
break  
本条规则匹配完成后,终止匹配,不再匹配后面的规则  
redirect 返回 302 临时重定向,浏览器地址会显示跳转后的 URL 地址  
permanent  
返回 301 永久重定向,浏览器地址会显示跳转后 URL 地址  
 --------------------------------------------------------  
硬件：Linux  
系统：rhel7  
软件：nginx  
服务：nginx  
 部署：  
  location ~ /new {  
 rewrite /new/.* /new/index.html break;  
 }  
 -----------------------------------------------------------  
 location ~ /uplook {  
 rewrite ^/uplook/([0-9]+)-([0-9]+)-([0-9]+)\.html /uplook/$1/$2/$3.html last;  
 }  
 if:  
 --------------------------------  
 root@serverb ~]# vim /etc/nginx/conf.d/www.joy.com.conf  
 server {  
 listen  
 80;  
 server_name ~.*.joy.com;  
 root /usr/share/nginx/joy.com;  
 index index.html index.htm;  
 if ( $http_host ~* ^www\.joy\.com$ ) {  
 29Nginx 配置  
 break;  
 }  
 #如果用户访问的是 www.joy.com,则不做 rewrite  
 if ( $http_host ~* ^(.*)\.joy\.com$ ) {  
 set $domain $1;  
 rewrite /.* /$domain/index.html break;  
 }  
 #将用户输入的.joy.com 之前的字符串保存并将保存的内存赋值给 domain 变量  
 创建测试文件  
 [root@serverb ~]# cd /usr/share/nginx/joy.com/  
 [root@serverb joy.com]# mkdir tom  
 [root@serverb joy.com]# echo tom > tom/index.html  
 [root@serverb ~]# systemctl restart nginx.service  
 客户端测试  
 [root@workstation ~]# echo 172.25.41.10 tom.joy.com >> /etc/hosts  
 -----------  
 代理：正向代理、反向代理、透明代理  
 正向代理：一般用在客户端，当client与server无法直接通信时，客户端可以用一个可以和cilent和server正常通信的服务器作为代理来实现client与server之间的通信，一般用作翻墙  
 反向代理(Reverse Proxy)方式是指以代理服务器来接受 internet 上的连接请求,然后将请求转发给内部网络上的服务  
 器,并将从服务器上得到的结果返回给 internet 上请求连接的 客户端,此时代理服务器对外就表现为一个服务器。  
 反向代理又称为 Web 服务器加速,是针对 Web 服务器提供加速功能的。它作为代理 Cache,但并不针对浏览器用户,  
 而针对一台或多台特定 Web 服务器(这也是反向代理名称的由来)。代理服务器可以缓存一些 web 的页面,降低了 web 服务  
 器的访问量,所以可以降低 web 服务器的负载。web 服务器同时处理的请求数少了,响应时间自然就快了。同时代理服务器也  
 存了一些页面,可以直接返回给客户端,加速客户端浏览。实施反向代理,只要将反向代理设备放置在一台或多台 Web 服务器  
 前端即可。当互联网用户访问某个 WEB 服务器时,通过 DNS 服务器解析后的 IP 地址是代理服务器的 IP 地址,而非原始 Web  
 服务器的 IP 地址,这时代理服务器设备充当 Web 服务器,浏览器可以与它连接,无需再直接与 Web 服务器相连。因此,大量  
 Web 服务工作量被转载到反向代理服务上。不但能够很大程度上减轻 web 服务器的负担,提高访问速度,而且能够防止外部  
 网主机直接和 web 服务器直接通信带来的安全隐患。  
 透明代理：一般用在网关  
 -------------------------------------------------  
  
  
 Nginx proxy 是 Nginx 的王牌功能,利用 proxy 基本可以实现一个完整的 7 层负载均衡,它有这 些特色:  
 1.  
 功能强大,性能卓越,运行稳定。  
 2. 配置简单灵活。  
 3. 能够自动剔除工作不正常的后端服务器。  
 4. 上传文件使用异步模式。  
 5. 支持多种分配策略,可以分配权重,分配方式灵活。  
  
  
 servera:  
 /etc/nginx/conf.d  
 vim www.ccc.com.conf   
 server {  
 listen 80;  
 server_name www.ccc.com;  
 location / {  
 proxy_pass http://192.168.0.11;  #反向代理指向ip,确定需要代理的 URL,端口或 socket。  
   
proxy_set_header Host $host;  
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504 http_404;  
proxy_set_header X-Real-IP $remote_addr;  
proxy_redirect off;  
  
  
  
  
 }  
 }  
 +++++++++++++++++++++++++++++++++++++++++++++++  
 proxy_pass http://192.168.0.11;  
 #确定需要代理的 URL,端口或 socket。  
 proxy_set_header Host $host;  
 #这个指令允许将发送到后端服务器的请求头重新定义或者增加一些字段。 这个值可以是一个文本,变量或者它们的组合。  
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
 proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504 http_404;  
 proxy_set_header X-Real-IP $remote_addr;  
 #确定在何种情况下请求将转发到下一个服务器:  
 #error - 在连接到一个服务器,发送一个请求,或者读取应答时发生错误。  
 #timeout - 在连接到服务器,转发请求或者读取应答时发生超时。  
 #invalid_header - 服务器返回空的或者错误的应答。  
 #http_500 - 服务器返回 500 代码。  
 #http_502 - 服务器返回 502 代码。  
 #http_503 - 服务器返回 503 代码。  
 #http_504 - 服务器返回 504 代码。  
 #http_404 - 服务器返回 404 代码。  
 #off - 禁止转发请求到下一台服务器。  
 proxy_redirect off;  
 #如果需要修改从后端服务器传来的应答头中的"Location"和"Refresh"字段,可以用这个指令设置。  
  
  
 ------------------------------------------------  
 serverb：  
 yum -y install httpd vim  
 vim /etc/httpd/conf.d/ccc.com.com  
 <VirtualHost *:80>  
serverName www.ccc.com  
DocumentRoot /var/www/ccc.com  
 </VirtualHost>  
 mkdir /var/www/ccc.com  
 echo serverb> /var/www/ccc.com/index.html  
 systemctl start httpd  
  
  
  
  
 upstream  
 作用:HTTP 负载均衡模块。upstream 这个字段设置一群服务器,可以将这个字段放在 proxy_pass 和 fastcgi_pass 指令中作  
 为一个单独的实体,它们可以是监听不同端口的服务器,并且也可以是同时监听 TCP 和 Unix socket 的服务器。  
 分配方式:  
  
  
 轮询(默认)  
 每个请求按时间顺序逐一分配到不同的后端服务器,如果后端服务器 down 掉,能自动剔除。  
  
  
 weight  
 指定轮询几率,weight 和访问比率成正比,用于后端服务器性能不均的情况。  
  
  
  
  
  
  
 轮询：  
 servera:  
 /etc/nginx  
 vim nginx.conf   
 upstream ccc_pool { #定义一个组 组名  
 server 192.168.0.11:80 weight=1; 组成员 weight：权重  
 server 192.168.0.12:80 weight=4; 组成员 weight：权重，权重越大越容易被访问到  
 ip_hash; #会话一致性，同一个用户不进行轮询  
 }  
 -----------------------  
 /etc/nginx/conf.d  
 vim www.ccc.con.conf  
 server {  
 listen 80;  
 server_name www.ccc.com;  
 location / {  
 proxy_pass http://ccc_pool;  #反向代理指向组  
 proxy_set_header Host $host;  
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504 http_404;  
proxy_set_header X-Real-IP $remote_addr;  
proxy_redirect off;  
}  
 }  
 ----------------------------------------------  
 serverb：  
 yum -y install httpd vim  
 vim /etc/httpd/conf.d/ccc.com.com  
 <VirtualHost *:80>  
serverName www.ccc.com  
DocumentRoot /var/www/ccc.com  
 </VirtualHost>  
 mkdir /var/www/ccc.com  
 echo serverb> /var/www/ccc.com/index.html  
 systemctl start httpd  
 --------------------------------------------------  
 serverc：  
 yum -y install httpd vim  
 vim /etc/httpd/conf.d/ccc.com.com  
 <VirtualHost *:80>  
serverName www.ccc.com  
DocumentRoot /var/www/ccc.com  
 </VirtualHost>  
 mkdir /var/www/ccc.com  
 echo serverc> /var/www/ccc.com/index.html  
 systemctl start httpd  
 *********************************************************************  
 缓存：  
 nginx 缓存  
 修改全局配置选项,指定缓存文件放置位置  
 servera:  
 /etc/nginx  
 vim nginx.conf   
proxy_temp_path /usr/share/nginx/proxy_temp_dir 1 2;  
proxy_cache_path /usr/share/nginx/proxy_cache_dir levels=1:2 keys_zone=cache_web:50m inactive=1d max_size=30g;  
 upstream ccc_pool { #定义一个组 组名  
 server 192.168.0.11:80 weight=1; 组成员 weight：权重  
 server 192.168.0.12:80 weight=4; 组成员 weight：权重，权重越大越容易被访问到  
 ip_hash; #会话一致性，同一个用户不进行轮询  
 }  
 ------------------------------------------------  
 建立全局配置项中的缓存文件放置位置,并将目录的拥有者设置为 nginx  
 mkdir -p /usr/share/nginx/proxy_temp_dir /usr/share/nginx/proxy_cache_dir  
 chown nginx /usr/share/nginx/proxy_temp_dir/ /usr/share/nginx/proxy_cache_dir/  
  
  
  
  
  
  
 ------------------------------------------------  
 vim /etc/nginx/conf.d/www.ccc.com.conf  
  
  
 server {  
 listen 80;  
 server_name www.ccc.com;  
 location / {  
 # proxy_pass http://192.168.0.11;  
 proxy_pass http://ccc_pool;  
 proxy_set_header Host $host;  
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
 proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504 http_404;  
 proxy_set_header X-Real-IP $remote_addr;  
 proxy_redirect off;  
 client_max_body_size 10m;  
 client_body_buffer_size 128k;  
 proxy_connect_timeout 90;  
 proxy_send_timeout 90;  
 proxy_read_timeout 90;  
 proxy_cache cache_web;  
 proxy_cache_valid 200 302 12h; #200 302指网页的类型  
 proxy_cache_valid 301 1d;  
 proxy_cache_valid any 1h;  
 proxy_buffer_size 4k;  
 proxy_buffers 4 32k;  
 proxy_busy_buffers_size 64k;  
 proxy_temp_file_write_size 64k;  
 #对缓存做相关设置,如缓存文件大小等  
 }  
 }  
 ------------------------------  
  
  
 systemctl restart nginx.service  
 没访问之前查看缓存目录,目录为空  
 ll /usr/share/nginx/proxy_cache_dir/  
 total 0  
 访问完成后再次查看缓存目录,有相应缓存文件,也可将后台服务停掉,对于缓存文件,客户端仍然可以访问到。  
  
  
  
  
  
  
  
  
 问题与解答：   
 