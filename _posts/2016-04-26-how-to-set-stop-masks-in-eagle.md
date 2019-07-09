---
title: Eagle如何设置过孔盖油
tags: 技术 硬件开发 PCB Eagle
--- 

通过设置stop masks的limit设计规则，来设置过孔盖油。
<!--more-->
##### Q：如何为Eagle设置过孔盖油？
##### A：
* 方法1: 在29.tStop层、30.bStop层上编辑删除那些阻焊层的开窗。这样阻焊层将会覆盖所有没有标记的区域。
* 方法2: 在“设计规则”中，masks标签下，设置Limit超过过孔直径的尺寸。按照说明，eagle只会对超过limit限制的区域，增加stop masks。如下图：

![image](posthow-to-set-stop-masks-in-eagle.png)