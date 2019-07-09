---
title: MAC下，使用expect，以密码方式自动登录SSH
tags: 技术 Mac expect
--- 

在搜索MAC下自动登录SSH的时候，大多数方案都是建议使用一对公私秘钥来达到自动登录的目的。
下面这是一种借用expect脚本实现的以密码方式自动登录到ssh服务器的方法。当然建议还是使用公私秘钥的方式。

<!--more-->

    !/usr/bin/expect

    set timeout 60          #设置超时时间为60s
    set host 192.168.1.2    #设置服务器的ip
    set name root           #设置登录的用户名
    set password  ******    #设置root用户的密码

    spawn ssh $host -l $name       #spawn一个ssh进程

    expect {    # 等待响应，第一次登录往往会提示是否永久保存 RSA 到本机的 know hosts 列表中；等到回答后，在提示输出密码；
          "(yes/no)?" {
               send "yes\n"
               expect "password:"
               send   "$password\n"
          }
          "password:"  {
              send "$password\n"
          }
    }

    expect "#"
    #下面检测是否登录到host
    send "uname\n"
    expect "Linux"
    send_user  " Now you can do some operation on this terminal\n"
    interact

#### 参考
[http://bbs.chinaunix.net/thread-4117940-1-1.html](http://bbs.chinaunix.net/thread-4117940-1-1.html)
