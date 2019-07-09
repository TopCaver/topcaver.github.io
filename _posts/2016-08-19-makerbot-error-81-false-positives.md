---
title: Makerbot Z18 频繁误报堵料错误(Error 81)的应对方法
tags: 技术 3D打印 Makerbot
---  

在较长时间使用Makerbot Z18之后，我们较为频繁的遇到了打印机报警 Filament Jam（Error 81）错误。但是点击进料可以正常进料，表示喷头并没有真正的“堵住”，甚至不需要重新进料，只要点击“Resume”继续，就可以继续打印。这说明堵料报警是个误报，传感器出了问题。Makerbot官方给出了一个旧喷头的解决方法。

<!--more-->

>  Makerbot 官方支持原文：
>  http://support.makerbot.com/troubleshooting/makerbot-replicator/error-codes/error-81_10422/8483

### 针对旧喷头 Extruder
针对旧喷头（Extruder）官方的说法是这样的，找一个“一字”螺丝刀或者类似的东西，把检测堵料的滚轮，向触点方向撬一下。

![image](/illustration/makerbot-error-81-false-positives.png)


### 针对新喷头 Extruder+
针对新喷头（Extruder ＋），官方的说法是，需要提交反馈(open case)由他们处理。

但是，从旧喷头给出的方案看，makerbot检测堵料的方法，（我猜）是靠料丝带动橡胶滚轮，然后使用光感应器，检测滚轮是否在滚动，以判断是否发生了堵料。这个检测有点类似于鼠标滚轮。 频繁发生错误，那么可能是 1.料丝没有带动橡胶滚轮，或者 2.橡胶管轮没有正确触发光电感应器。

针对新喷头（Extruder +），如果方便送修的话，可以之间去找Makerbot的售后。如果不方便的话，可以自行拆开，清理一下滚轮。

![image](/illustration/makerbot-error-81-false-positives-extruder-plus.png)

［1］ 机牙内六角螺丝； ［2］带反光贴纸的塑胶滚轮； ［3］光电感应器

主要是清理塑胶滚轮[2]的反光贴纸，不要被污损；光电感应器头[3]不要有污物，废料渣之类的；然后锁紧滚轮中心轴的机牙螺丝。可以稍向下按压滚轮，让滚轮反光面更靠近光电感应器。

旧喷头按照官方手法，目前正常；新喷头我已经清理，还没有测试。


