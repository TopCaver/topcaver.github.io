---
title: MakerBot Z18 加热打印机内部空间
tags: 技术 3D打印 MakerBot
---  

来自[官方文档](https://www.makerbot.com/support/new/03_Replicator_Z18/Knowledge_Base/Using_the_Heated_Chamber_in_the_MakerBot_Replicator_Z18)，MakerBot Z18 有一个选项可以选择加热打印机内腔空间。加温可以减少塑料层之间的裂缝。空间加温对大件物品打印更为有用，小件物品不使用加温既省电，也容易拿下来。

<!--more-->

加热打印机内部温度的方法：
1. 创建一个自定义配置文件（Profile）
2. 使用文本编辑器，编辑自定义配置文件，“chambertemp”
3. 将“0”改为“35”。35° C是官方推荐的内部温度
4. 保存并关闭配置文件
5. 点击“保存配置”（Save Settings）

如果MakerBot Desktop升级到 3.7.0.148之后，在settings里面可以直接设置ChamberTemp了。

![image](/illustration/makerbot_z18_%20heated_chamber.png)