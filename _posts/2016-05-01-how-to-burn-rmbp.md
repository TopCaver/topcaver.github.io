---
title: 我是如何烧坏了一台RMBP的
tags: 随笔 技术
---  

说实话，我也不知道我是怎么搞一台RMBP搞废掉的。坏的很彻底，主板、显示器，统统坏掉了。现在正在写字的这台RMBP，只有外壳、键盘和SSD是我的。

<!--more-->

#### 一、那一瞬间的“嘭！”

当时，我正在调一片MSP430的片上的程序，把关机中断处理的函数里面，加入一句切入低功耗状态的指令。我当时的连线是这样的:

![image](/illustration/how-to-burn-rmbp.png)

当我把编程器，插到我手里的电路板上的时候，砰地一声。随后一股焦糊味弥漫开来。

![image](/illustration/how-to-burn-rmbp-01.png)

连下面垫的纸都烧糊了。

办公室的总闸都跳开了。

当我们合闸之后，我发现了一件令人悲伤的事情……是的，我的macbook不亮了。


#### 二、Macbook Pro的维修之旅

微博上盛传苹果用户的三大错觉：iCloud能备份，AppStore能打开，GeniusBar能预约。

苹果维修需要预约苹果店的GeniusBar服务，确实不好约，首先是各种“抱歉，系统出错了”一直走不到预约时间那里。然后，走到预约时间那里的时候，可预约时间已经是第四天下午及之后的时间了。

当我按照预约来到GeniusBar的时候，苹果的天才们，鼓弄了一阵，告诉我主板坏了，维修需要¥3700。行，修吧……

过来一天，苹果来电告知，主板换完，你这还是不亮。屏幕也得换，维修需要¥7000+了。呃，算了吧……

回来之后，求助了一下大淘宝。找了在中关村附近，并且响应速度最快的一家，大概问了问价钱。主板维修200～700，更换主板1600，显示器1000。差不多是这个价钱。送去之前，我大概也拆开我的macbook看了看：

![image](/illustration/how-to-burn-rmbp-02.png)

如此规整，优雅。我盯着这块板看了一晚上。确定自己搞不定。

第二天送去修了，一个小作坊。在吹掉两块芯片之后，师傅手里的万用表依然碰哪哪响，提示板上各种短路。师傅放弃了，说：换吧。

换了一块主板之后，屏幕不亮，师傅坚定的认为是屏线坏了，要先换线试试。换完之后，师傅说：屏幕也换吧。

我说：您再帮忙看看SSD吧，要是也坏了，就算了。万幸……硬盘还在。

#### 三、亡羊补牢，防微杜渐

我已经不能再复现当时现场，也无从排查根本原因了。谁有这方面经验，希望可以指导我一下。

我初步怀疑，是一个高电压从usb的地线流入了macbook，反向击穿了几乎所有的接地组件。

为此，我做了一个叫做：USB-Safe-Guard的小硬件。简单说，就是在电源线、地线上分别串接了两个350mA的自恢复保险丝。电量超过500mA时，自动跳开。

![image](/illustration/how-to-burn-rmbp-03.png)

现在我也已经做出一块Demo了：

![image](/illustration/how-to-burn-rmbp-04.png)

嗯……

现在，就剩下**“测试”**这一步了。有人想帮忙么…… 就是，需要好好验证一下，外围电路短路之后，是保险丝先跳，还是笔记本先坏。-_-!

#### 好心人士补遗，2016/08/19
没想到还能有人看完这篇专门发来邮件，向我释惑并提出建议。全文抄录如下，感谢idaxjj，你让我再次感受到互联网的开放与无私帮助的伟大。

> Hi！我在搜索EAGLE使用问题的时候，发现了你的主页，看到了你的遭遇。
以下有几点疑问及建议：  
1.总体来说应该是大电流，经由电源VCC.GND进去主板的。  
2.关于变压器，首先该变压器是否为隔离变压器？即：初级与次级共地，如果是，那么请更换为隔离变压器；其次如果目前采用的是隔离变压器，那么建议采用隔离电压为AC4kV。  
3.关于你制作的USB-Safe-Guard小板，我觉得只能抗一些小电流的冲击。要实现真正的隔离，建议采用光耦隔离，低速率用LTV816等，高速率用高速光耦6N137等。  

> 来自一个电子爱好者idaxjj

> 发自坚果手机

