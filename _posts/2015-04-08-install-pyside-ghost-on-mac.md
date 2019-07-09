---
title: Mac安装PySide、Ghost.py
tags: 技术 Python PySide Ghost
---  

在试用PySide 和 Ghost.py的时候，碰到几个问题，整理记录一下安装的过程。这个安装方法，是不需要用到brew的。

1. 安装Qt 4.8
2. 安装PySide
3. 安装Ghost.py

<!--more-->

####1、安装Qt
因为PySide的QtCore依赖了Qt 4的库文件，现在Qt最新版是5，Mac上安装路径也不一样。好在Qt提供了老版本的下载。[http://download.qt.io/archive/qt/4.8/4.8.5/](http://download.qt.io/archive/qt/4.8/4.8.5/)  


如果没有安装Qt，或者版本被坑了，大概会得到这样一个错误：
> ImportError: dlopen(/Library/Python/2.7/site-packages/PySide/QtCore.so, 2): Library not loaded: /usr/local/lib/QtCore.framework/Versions/4/QtCore

装4.8.5之前的版本吧，我开始装了一下4.8.6，QtCore总是会崩溃。

####2、安装PySide
建议这样的：

    sudo -H pip install pyside
    

然后， 安装之后，看看有什么错误，如果不顺利的话，执行一下pyside_postinstall.py，据说这货应该是自己执行的。处理一些路径之类的问题。

    pyside_postinstall.py -install

可能碰到的错误，大概是这样的：

> ImportError: dlopen(/Library/Python/2.7/site-packages/PySide/QtCore.so, 2): Library not loaded: libpyside-python2.7.1.2.dylib

如果装完，可以这样验证一下：

    from Pyside import QtCore
    
如果没有错误，就是可以了。

####3、安装Ghost.py
这个按照文档的说法，就可以了：

    pip install ghost.py --pre

***
Ghost.py 文档：[http://ghost-py.readthedocs.org/en/latest/#](http://ghost-py.readthedocs.org/en/latest/#)

这里还有一个非brew的Python安装brew的pyside的帖子：[http://django-china.cn/topic/690/](http://django-china.cn/topic/690/)