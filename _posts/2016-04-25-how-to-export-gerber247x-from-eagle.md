---
title: 如何从Eagle导出gerber247x文件进行PCB制板
tags: 技术 硬件开发 PCB Eagle
---  

MAC上能画板子的软件少得可怜，Eagle画完的PCB，人家也不能之间打开文件。只能自己转换成gerber247x格式的，再交给板厂制板。

<!--more-->

#### 主要步骤
1. 运行run drillcfg命令，然后从弹出的“Drill Configuration”对话框中选择“inch”项，再单击“OK”按钮生成相应的钻孔配置文件。也可在ULP里面，选中drillcfg.ulp文件执行。两者是等价的。
2. 使用CAM处理器，选择gerber247x.cam，生成gerber 247x文件。
3. 使用CAM处理器，选择excellon.cam，生成钻孔坐标文件。

将上面生成出来的所有文件，一并打包给板厂制板打样.

#### 自定义一个作业文件

为了方便板厂明确知道哪个文件是哪层，我在eagle自带的作业文件基础上修改了一些。适用于两层板。

##### Gerger 247x 文件后缀和对应层关系

> *.GTL, 顶层元件和线路层，需要处理：1、17、18 共三层  
> *.GBL, 底层元件和线路层，需要处理：16、17、18 共三层  
> *.GTS, 顶层阻焊层，需要处理：29层  
> *.GBS, 底层阻焊层，需要处理：30层  
> *.GTO, 顶层丝印层，需要处理：20、21、25 共三层  
> *.GBO, 底层丝印层，需要处理：20、22、26 共三层  
> *.GKO, 外形轮廓层，需要处理：20层  
> *.drl,*.txt, 钻孔数据层，需要处理：44、45 共两层  

##### 以生成gtl文件，为例

![image](/illustration/how-to-export-gerber247x-from-eagle.png)

1. 设置当前作业的名称：例如叫做GTL
2. 输出设备选择：gerber247x
3. 输出文件：％N.gtl， （以 “当前文件名.gtl” 生成新的文件名。）
4. 选择需要处理的层，gtl为顶层线路层，需要选中：1、17、18三层。

注意：

* 其他的层数据，与此类似，主要修改输出文件的后缀名，和需要处理的层。不要搞错后缀名，这样会被覆盖的；不要搞错层，不然做出来不对。  
* 生成钻孔数据的时候，主要选择输出设备为：EXCELLON

#### 我做的生产gerber 247x的作业
[点击下载：gerb247x_L2_topcaver.cam](/attachment/gerb247x_L2_topcaver.cam)

#### 参考链接：

[http://www.flamingoeda.com/2007/11/11/在Eagle中生成Gerber文件/](http://www.flamingoeda.com/2007/11/11/%E5%9C%A8eagle%E4%B8%AD%E7%94%9F%E6%88%90gerber%E6%96%87%E4%BB%B6/)
