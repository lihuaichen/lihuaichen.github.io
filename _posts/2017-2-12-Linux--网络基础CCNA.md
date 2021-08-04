---
title: Linux--网络基础CCNA
date: 2016-08-31 23:16:04
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/52390068   
  网络拓卜结构  
让我们很快直观的对当前环境的网络结构进行了解  
总线型，星型，环型，蜂窝型  
  
  
CCNA  
  
  
> //当前处于用户模式  
>enable //进入特权模式的命令  
# //当前处于特权模式  
#show running-config //查看当前设备的内存配置文件  
#show mac-address-table //查看mac地址表  
#show vlan //查看VLAN的全部信息  
#show vlan brief //简介的方式查看vlan信息  
#write //保存配置文件  
#copy running-config startup-config //将内存配置文件保存到启动配置文件中（相当于保存配置文件）  
#configure terminal //进入全局配置模式的命令  
(config)# //当前处于全局配置模式  
(config)#hostname 名字 //设定设备名称  
(config)#enable password 密码 //设置进入特权模式是的明文认证（密码）  
(config)#enable secret 密码 //设置进入特权模式是的密文认证（密码）  
(config)#no ip domain-lookup //取消特权模式下输错时的一分钟等待  
(config)#interface f0/1 //进入f0/1接口  
(config-if)# //当前处于接口模式  
(config)#interface range f0/1-f0/10 //进入组接口模式  
(config)#interface range f0/1,f0/3,f0/5 //进入组接口模式  
(config-if-range)# //当前处于组接口模式  
exit //返回上一级模式  
end //返回特权模式  
命令补全 tab 命令帮助 ? 命令简写  命令缓存 上 下  
取消配置命令要在源有命令的前面加上 no  
要在全局配置模式和接口模式下查看配置文件或查看其他配置需要在show命令前加do  
例：（config-if）#do show show vlan brief  
-----------------------------------------------------------------------------------------------  
VLAN（虚拟局域网） 不同VLAN之间无法通信  
#vlan database //进入vlan数据库模式  
(vlan)# //当前处于VLAN数据库模式  
(vlan)#vlan 10 //创建ID为10的VLAN（创建VLAN 10）  
(vlan)#vlan 10 name caiwu //将vlan10的名字设定为caiwu（更改vlan的名字）  
(config-if)#switchport access vlan 10 //将该接口加入到VLAN10中  
TRUNK（主干，中继） 让不同设备相同VLAN进行通信  
(config-if)#switchport mode trunk //将该接口设置为trunk（中继）模式  
  
  
VTP（VLAN TRUNK protocal中继协议）  
方便管理VLAN的信息：信息包含VLAN_ID,vlan的名字  
VTP的三种模式：服务模式，客户模式，透明模式  
服务模式  
提供VTP信息  
学习并转发同域名下的VTP信息  
不学习也不转发不同域名下的VTP信息  
可以对VLAN进行添加，修改及删除  
  
  
客户模式  
请求VTP信息  
学习并转发同域名下的VTP信息  
不学习也不转发不同域名下的VTP信息  
不可以对VLAN进行添加，修改及删除  
  
  
透明模式  
不提供VTP信息也不请求VTP信息  
转发VTP信息  
可以对VLAN进行添加，修改及删除（但只对本地生效）  
  
  
(config)#vtp domain 名字 //设置VTP域名  
(config)#vtp password 密码 //设置vtp的认证  
(config)#vtp version 1or2（1或2）//设置vtp的版本号默认2  
(config)#vtp mode server //设置vtp模式为server（服务端）  
(config)#vtp mode client //设置vtp模式为client（客户端）  
(config)#vtp mode transparent //设置vtp模式为transparent（透明）  
#show vtp status //查看vtp信息  
-------------------------------------------------------------------------  
路由  
静态路由  
(config-if)#ip address IP地址 子网掩码 //设置该接口的IP地址及子网掩码  
(config-if)#no shutdown //开启该接口  
(config)#ip route 目标网段 目标网段子网 下一跳地址 //手动添加一条路由到路由表  
目标网段：现在路由表里面缺哪个网段，那么哪个网段就是目标  
下一跳地址：1.是现在已知的网段地址；2.通过这个地址可以找到我们的目标网段  
  
  
#show ip route //查看路由表  
C：直连路由  
S：静态路由  
R：以rip路由协议学习到的  
O：以OSPF路由协议学习到的  
B：以BGP路由协议学习到的  
---------------------------------------------------------------------------  
动态路由  
根据路由协议，让相邻的路由器之间交换路由表信息  
内部协议和外部协议  
内部协议：  
1.距离矢量路由协议  
rip：只支持15台路由以下  
  
  
2.链路状态路由协议  
ospf  
外部协议：  
BGP  
  
  
  
  
(config)#router rip //开启rip协议  
(config-router)# //动态路由协议模式  
(config-router)#network 自身网段 //宣告自身网段  
---------------------------------------------------------------------------  
单臂路由  
让不同VLAN之间进行通信  
(config)#interface f0/0.1 //创建子接口  
(config-subif)#encapsulation dot1Q 10 //将当前子接口和vlan10进行封装（绑定）  
(config-subif)#ip address IP地址 子网掩码 //为该VLAN设置出口（网关）  
  
  
使用三层交换机来实现不同VLAN之间的通信  
(config)#interface vlan 10 //创建vlan10这个接口  
这步相当于单臂路由中的创建子接口加封装VLAN两步  
---------------------------------------------------------------------------  
DHCP（动态IP地址自动分配）  
(config)#ip dhcp pool 1 //创建名为1的地址池  
(dhcp-config)# //DHCP配置模式  
(dhcp-config)#default-router IP地址 //设定获取到网关  
(dhcp-config)#network IP网段 子网掩码 //设定获取到的IP地址范围  
(dhcp-config)#dns-server IP地址 //设定获取到的DNS的地址  
(config)#ip dhcp excluded-address IP地址 //设定该地址不会被DHCP分配到  
中继DHCP  
(config-if)#ip help-address IP地址 //该接口寻求有DHCP服务的IP地址的帮助  
  
  
DNS（域名解析服务）网络的基石  
www.baidu.com  
  
  
访问控制列表  
格式：(config)#access 编号 （deny or permit） 网段或IP地址或通配符  
deny //拒绝数据包  
permit //允许数据包  
通配符：any //代表所有网段及IP  
 host //精确到主机位  
格式(config-if)#ip access-group 编号 （in or out）  
在接口上生效访问控制列表的编号，in为进入的数据包；out为出去的数据包  
例：(config)#access 1 deny any //创建编号为1的访问控制列表拒绝所有人  
(config)#access 2 permit 172.16.0.0 0.0.255.255（反掩码）  
创建编号为2的访问控制列表条件为允许172.16.0.0网段的所有人数据包通过  
如果要拒绝某个IP地址  
(config)#access 1 deny host 172.16.0.1 //拒绝172.16.0.1  
(config)#access 1 permit any //允许所有人  
(config-if)#ip access-group 1 in //让编号为1的控制在该接口进入时生效  
  
  
#access-lists //查看访问控制列表   
 