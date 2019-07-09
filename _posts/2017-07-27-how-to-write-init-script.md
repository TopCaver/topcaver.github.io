---
title:  如何写一个Linux下的启动脚本 RC
tags: 技术 Linux
--- 

对于init脚本，其实可以参考 /etc/init.d/下面的已经有的脚步，仿照它们写一个。[这里有一篇文章](https://wiki.debian.org/LSBInitScripts) 描述了一些详情，本文记录一个最简单的init脚本。

<!--more-->

基本格式是这样的：


    #! /bin/sh
    ### BEGIN INIT INFO
    # Provides: TopCaver
    # Required-Start: $remote_fs $syslog    
    # Required-Stop: $remote_fs $syslog    
    # Default-Start: 2 3 4 5    
    # Default-Stop: 0 1 6    
    # Short-Description: TopCaver    
    # Description: This file starts and stops TopCaver server    
    # 
    ### END INIT INFO    
    case "$1" in
        start)
            cd /opt/topcaver && nohup /opt/topcaver/topcaver-server >/dev/null 2>&1 &
        ;;
        stop)
            killall topcaver-server
        ;;
        restart)
            killall topcaver-server
            cd /opt/topcaver && nohup /opt/topcaver/topcaver-server >/dev/null 2>&1 &
        ;;
        status)
            ps -ef | grep "qr-govf.com" | grep -v grep && exit 0 || exit $?
        ;;
        *)
            echo "Usage: $0 {start|stop|restart}" >&2
            exit 3
        ;;
    esac


最前面的一段注释头是必须的，除了一些基本信息，还有指定后于哪些服务启动停止之类的，然后需要实现的方法有：start, stop, restart, force-reload, 和 status。如果偷懒的话，把start和stop实现好就可以了。

然后，复制或软连接脚本到/etc/init.d/目录下

然后给执行权限：chmod a+x xxxx.sh

更新rc.d： sudo update-rc.d xxxx.sh defaults

如果不要这个启动脚本了，可以remove：sudo update-rc.d -f xxxx.sh remove


### 参考
https://mobiarch.wordpress.com/2014/05/16/creating-an-init-script-in-ubuntu-14-04/
http://www.linuxdiyf.com/linux/26896.html



