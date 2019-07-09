---
title: 使用 Let's Encrypt 为网站提供安全链接
tags: 技术 安全
---  

随着苹果要求APP必须使用https，chrome对没有使用https的网站发出警告，把网站搞成https就显得越来越必要了。虽然我个人Blog确实不在意这些，权且当作试用Let's Encrypt 的笔记吧。
<!--more-->

环境描述：
> - VPS： Ubuntu 16.04 LTE
- Nginx + PHP

Let's Encrypt 提供了一个证书机器人，可以相当方便完成证书生成到部署的工作。

## 安装 

    $ sudo apt-get install letsencrypt 
    
## 生成证书
    $ sudo letsencrypt certonly --webroot -w /var/www/example -d example.com -d www.example.com -w /var/www/thing -d thing.is -d m.thing.is
    
-w： 指定网站的根目录
-d： 网站的域名

自动申请和生成的证书，就在指定的目录下面。

## 配置Nginx
修改Nginx的配置，写好cert和key的位置，像这样：

    server {
        listen 443 ssl default_server;
        server_name my-domain;
        
        ssl_certificate /etc/letsencrypt/live/my-domain/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/my-domain/privkey.pem;
        ...
    }

然后重启你的Nginx：

    $ sudo nginx -t && sudo nginx -s reload

## 自动更新证书
Let's Encrypt 提供的证书虽然免费，但是时间有限，三个月就到期了。不过他们提供一种自动更新机制。

    $ sudo letsencrypt renew --dry-run --agree-tos
    
## 后记
### 备忘
- 更新站点到https之后，看看里面有没有硬写的链接到http的，js、css、图片什么的，因为这样会导致浏览器不加载这些“不安全”的内容。如果资源链接已经可以http&https了，那把代码里的 http:// 直接换成 // 就好了，就相当于不指定协议了。
- 毕竟多了一层加密，我的乞丐VPS上面，https站点速度比http差了挺远的。

### 参考
使用证书机器人制作证书：
https://certbot.eff.org/#ubuntuxenial-nginx

nginx下面配置SSL，就看server配置那段就好了，其他的写的是旧版本的let's encrypt用法。
https://www.nginx.com/blog/free-certificates-lets-encrypt-and-nginx/


