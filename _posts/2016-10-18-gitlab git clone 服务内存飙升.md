---
title: gitlab git clone 服务内存飙升
date: 2018-07-10 17:23:05
tags: CSDN迁移
---
  内部的glitlab服务器在git clone 某个repository时很慢，服务器响应变慢。

上服务器一看2g物理内存几乎都被git进程沾满了，交换内存都用上了，cpu load值很高， %wa值在90%以上了。

加到3g物理内存问题依旧。

原因： 当客户端个git clone时 服务端会先压缩然后才发送，但git是为代码管理设计的，在compressing packages 时会缓存在内存，在处理单个大文件（比如1g左右）时会

超量使用内存。

解决办法： 对大文件不启用压缩： 在xxxxxx.git/下新建 info/attributes(如果不存在的话)，在attributes 中添加  *.xx -delta (xx为大文件的扩展名) 

   
 