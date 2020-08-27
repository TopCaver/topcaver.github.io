---
title: Linux命令行发送邮件（mutt + msmtp）

tags: Linux mutt msmtp
--- 

试了一圈，对于命令行发送邮件，还是 mutt + msmtp 这个比较方便。可以发送附件。

<!--more-->

[TOC]

-----

### 安装 mutt 和 msmtp
```bash
    yum install mutt msmtp
```

### 配置msmtp
**1.  在当前用户~目录，创建 ~/.msmtprc 配置发件SMTP服务器和密码**

```ini
defaults
account <发件账户>
host <smtp服务器>
port 25
protocol smtp
auth login
from <发件人邮箱>
user <SMTP发件人的登录用户名>
password <SMTP发件人的密码>

account default : <默认使用的发件账号>
```

**2.  测试msmtp，发送邮件**

```bash
msmtp <收件人的邮箱>
hello,msmtp
Ctrl+D退出
```



### 配置mutt

**1. 在当前用户~目录，创建 ~/.muttrc 配置mutt发送选项**
```ini
set sendmail="/usr/bin/msmtp"
set from="<发送邮箱>"
set realname="<显示的发件人名称>"
```



**2. 测试mutt，发送邮件**

发送文本

```bash
echo "内容" | mutt -s "主题" <收件人@邮箱>
```

发送附件

```bash
echo "内容" | mutt -s "主题" <收件人@邮箱>  -a "附件文件名"
```



### 参考
http://cn.linux.vbird.org/linux_server/0380mail.php#mua_mutt
https://faceghost.com/article/562751
https://wiki.archlinux.org/index.php/Mutt_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)