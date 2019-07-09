---
title: Android逆向中用到的几个工具
tags: 技术 Android 逆向
--- 

在最近的工作中，用到了几个Android／Java相关的小工具。
<!--more-->

### Android设备抓包工具

**tPacketCapture**

https://play.google.com/store/apps/details?id=jp.co.taosoftware.android.packetcapture

这个工具原理是在本地起了一个VPN服务，重定向所有流量。工具简单，无需ROOT。导出*.pcap，用wireshark就行了。


### JAR反编译工具

**Procyon & Luyten**

https://bitbucket.org/mstrobel/procyon/wiki/Java%20Decompiler

https://github.com/deathmarine/Luyten

这个是Jar的反编译工具，Luyten是Procyon的GUI的壳。个人感觉比JD-GUI的好处在于没有出现JD-GUI解不出的enum。