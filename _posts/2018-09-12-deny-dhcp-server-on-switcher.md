---
title: 在交换机上屏蔽特定DHCP服务器
tags: 技术 路由交换
--- 

最近在使用PXE安装系统时，遇到机房有若干DHCP服务器提供DHCP服务，折腾了一通，在接入交换机上，用Mac ACL把另外不需要的DHCP服务器屏蔽了，交换机型号：Quanta LB6M  
<!--more-->

-------

## 如何发现网络内几个DHCP服务？
发一个DHCP DISCOVER数据包，然后抓DHCP OFFER 数据包。

1. 使用nmap发送DHCP请求：  
```
sudo nmap -script broadcast-dhcp-discover -e eth
```

2. 使用tcpdump监听DHCP服务器的响应  
```
sudo tcpdump -e -vvv -i eth0  port bootps
```

-------

## 在Quanta LB6M上配置MAC ACL，屏蔽特定DHCP服务

#### 1.进入全局配置，创建一个mac acl策略  

    # enable
    # config
    (Config)# mac access-list extended deny-dhcp-192.168.1.1 //创建一条mac acl
    (Config-mac-access-list)# deny 00:B0:2C:12:E7:5B 00:00:00:00:00:00 any //阻止指定mac源地址到任意地址的访问 ，注意mac mask为wildmask，置成FF会导致所有阻断
    (Config-mac-access-list)# exit

    

#### 2.再创建一个允许访问的默认acl策略


    (Config)# mac access-list extended permit-all //再创建一个运行访问的默认acl  
    (Config-mac-access-list)#  permit any any  
    (Config-mac-access-list)# exit  


#### 3.在某个接口上绑定两个mac acl策略


    (Config)# interface 0/26 //编辑某个接口的策略  
    (Interface 0/26)# mac access-group deny-dhcp-192.168.1.1 in 1 //绑定mac阻止的策略到接口 in方向，序号1  
    (Interface 0/26)# mac access-group permit-any in 99 //绑定默认允许策略到接口in方向，序号99  
    (Interface 0/26)# exit  


-------

## 附：
### 可能的其他方法
+ 方法1: 使用MAC绑定，把不需要的DHCP服务器的MAC绑定到不用的接口上。
+ 方法2: 在交换机上启动DHCP Reply到指定的服务器。

上面这两个方法，未作验证。


