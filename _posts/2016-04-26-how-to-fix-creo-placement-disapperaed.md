---
title: Creo中“放置(placement)”页签下项目消失，如何恢复？
tags: 技术 Creo
---  
通过清空Creo缓存文件夹，恢复Creo的窗口设置。
<!--more-->
##### 问题：
在Windows 10 (64位)的Creo Parametric 3.0时，会偶尔出现Placement等小窗口不知所踪的情况。但是使用ATL+TAB确实又可以看到这个窗口。像是被移动到了某个不可见的区域。

> I've got a problem with Creo Parametric 3.0 on Windows 10 (64bit)
> 
> The last weeks Creo works fine, but today I grabed accidentally my "Platzierung/Placement" button and now all the parameters disappeared.

##### 解决方法：
1. 重命名C:\Users\%username%\AppData\Roaming\PTC\ProENGINEER\Wildfire下的.wf文件夹，重置缓存文件夹。
2. 并重启Creo

> I have seen this problem in the past and those were due to corrption in cache folder, rename .wf folder (default location for that is in C:\Users\%username%\AppData\Roaming\PTC\ProENGINEER\Wildfire) and restart Creo. I hope this will help you. 

##### 参考：
https://www.ptcusercommunity.com/thread/129873