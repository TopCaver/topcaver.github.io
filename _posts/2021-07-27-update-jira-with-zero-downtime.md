---
title: 零停机升级Jira Data Center
tags: 运维 Jira
--- 

由于Jira出现了一个CVE-2020-36239 ， 虽然可以通过防火墙的方式，限制可信IP访问，但是内部还是期望尽快升级修复漏洞。通过查找官方文档，Jira Data Center多节点的部署，可以进行零停机的升级。

运行环境是Linux x64 ， 升级是从8.13.2(LTS) 升级到 8.13.9(LTS)

本文收集、整理、翻译了部分Jira官方文档。主要包括三个文档：

1. [Upgrading Jira Data Center with zero downtime](https://confluence.atlassian.com/adminjiraserver/upgrading-jira-data-center-with-zero-downtime-938846953.html)
2. [Preparing for the upgrade](https://confluence.atlassian.com/adminjiraserver0813/preparing-for-the-upgrade-1027137569.html)
3. [Upgrading Jira Data Center (installer)](https://confluence.atlassian.com/adminjiraserver/upgrading-jira-data-center-installer-966063332.html)

<!--more-->

[TOC]

注意： 零停机升级并不适用于从 Jira 7.x 升级到 8.x ，大版本的升级需要使用传统的升级方式。 如果已经是Jira 8.x了，可以使用零停机升级到更新的版本。



>  **关于零停机升级（ZDU， Zero downtime upgrades）**
>
>  零停机升级是Jira Data Center特有的升级方式。本质上是不同的Jira节点上运行不同版本的Jira，依次升级Jira的节点，在此期间，Jira可以提供完整的功能和服务。
>
>  你可以使用这种升级方式升级Jira Software Data Center和Jira Service Management Data Center。 Jira Software 7.3 或者 Jira Service Management 3.6是可以升级的最低的版本。
>
>  零停机升级可能比较繁琐（取决于你有多少节点）， 我们提供了一个[检查列表](https://confluence.atlassian.com/adminjiraserver/zero-downtime-upgrade-checklist-938846955.html)，方便用来确认每一个步骤都正确完成了。我们建议你按照本文的步骤进行升级操作， 检查列表只是一个辅助工具来帮助跟踪升级的进展。
>
>  **技术概述**
>
>  集群升级的更多技术细节，可以参考  [ZDU technical overview.](https://developer.atlassian.com/server/jira/platform/zero-downtime-upgrades-for-jira-data-center-applications/) 
>
>  @TopCaver注释： 可以先看一下这个技术概览的文档， 不太长，讲述了集群升级过程中的状态迁移：从稳定状态-准备升级-混杂版本-升级至同版本-运行升级任务-回到稳定状态。 这个一系列的状态过程。
>  **FAQs**
>
>  还有疑问的话， 参考：[Zero downtime upgrade FAQs.](https://confluence.atlassian.com/adminjiraserver/zero-downtime-upgrade-faqs-962971483.html)



# 准备工作

## Step 1: 升级前准备

确保完成了 [Preparing for the upgrade](https://confluence.atlassian.com/adminjiraserver/preparing-for-the-upgrade-966063325.html) 这个文档中要求的全部步骤。这些非常重要，是顺利升级的前提。



你还要确保没有正在执行的数据导出 [data pipeline export](https://confluence.atlassian.com/adminjiraserver/data-pipeline-1027142324.html)



## Step 2: 选择要升级的版本

如果你对选择版本还有一些疑惑，可以参考

[upgrade matrix](https://confluence.atlassian.com/adminjiraserver/upgrade-matrix-966063322.html) 文档，了解一下不同版本之间的特性、支持的平台，以及升级说明。 



# 设置Jira模式为升级模式(upgrade mode)

将Jira 设置为升级模式 upgrade mode ， 才能允许在节点依次升级的过程中， 整个集群的不同节点可以在不同的Jira版本下运行。

1. 点击 **Administration  > Applications > Jira upgrades.**
2. 点击 **Put Jira into upgrade mode** 。这个按钮只有在集群节点运行相同版本的时候，才能够点击。 

>  取消升级 Canceling the upgrade
>  *  在你开始升级之前，你可以点击取消升级(cancel the upgrade)，退出升级模式(upgrade mode)。一旦你开始升级了， 这个选项也不能点击了。
>  * 升级过程中需要取消的话，你需要回滚所有节点到之前的版本。



# 升级  Jira Service Management

这个步骤只有你同时安装了Jira Software 和  Jira Service Management时才需要，如果只有一个的话，可以忽略这一步。

（略）

>  @TopCaver注释：我没JSM，就忽略这步了。 需要的话，参考原文。



# 升级  Jira

当Jira被设置为升级模式（Upgrade Mode）之后， 就可以依次升级Jira节点了。升级一个节点将会执行：停止Jira、升级安装、启动Jira。 停止Jira会从集群中移除这个节点，登录到这个节点的用户，就会丢掉现在的登录会话，直到他们重新登录。作为管理员，你要决定以何种顺序，每次升级多少个节点，并且还要保证至少一个节点可以正常工作。以保证集群可以持续提供服务——零停机。

*  [DATA CENTER Upgrading Jira Data Center (installer)](https://confluence.atlassian.com/adminjiraserver/upgrading-jira-data-center-installer-966063332.html)
*  [DATA CENTER Upgrading Jira Data Center (manual)](https://confluence.atlassian.com/adminjiraserver/upgrading-jira-data-center-manual-938846951.html)

@TopCaver注释： 我选择了使用Installer， 看了一下两个方法，其实差别不大，看个人习惯和之前的安装方式吧。 **务必看对应版本的升级说明，别被我误导了**



## 升级Jira Data Center （Installer）

### 升级第一个节点
想要避免重复操作升级，你可以只升级一个节点，然后把安装目录作为一个模板。 这样你直接把这个安装模板复制到其他节点，就可以了。 至于先升级哪个节点，其实无所谓。

#### Step 1. 下载jira
从官方网站下载Jira 

* [Jira Software](https://www.atlassian.com/software/jira/download/data-center)

* [Jira Service Management](https://www.atlassian.com/software/jira/service-desk/download/data-center)

#### Step 2. 使用升级向导
使用安装程序，会有一个向导，帮助完成升级过程。 
1. 运行installer

   ```bash
   $ chmod a+x atlassian-jira-X.X.X-x64.bin
   $ sudo ./atlassian-jira-X.X.X-x64.bin
   ```

2. 根据提示升级
    a.  This will install Jira Software 8.13.9 on your computer. 选择 OK [o, Enter]
    b. Choose the appropriate installation or upgrade options. 选择 Upgrade an existing Jira installation [3, Enter]
    c. Existing installation directory: 确认安装目录之后，按回车即可
    d. Back up Jira Home directory. 如果已经有备份的话，可以选择n，如果选择备份的话，注意下磁盘空间的问题。
    e. Checking for local modifications. **这个务必要记下， 因为安装后，有一些修改需要升级之后手动重新改回来。 **
    f. Have you complete all these steps. 确认下检查检查、app兼容性、数据库备份都完成的话， 就选择 Yes [y]
    g. Do you want to proceed? 如果没有什么问题， 就选择Upgrade [u, Enter]
  
3. 等到安装程序执行到最后的时候， 会提示你是否启动Jira？
    h. Start Jira Software 8.13.9 now?
    **这里先什么都不要选， 停在这里！**

#### Step 3. 安装数据库驱动
Oracle 或 MySQL数据用户，需要更新JDBC driver， 其他的数据库跳过此步骤。

#### Step 4. 合并之前的修改
有一些改动，Jira的安装程序会合并，但是有一些还是需要手动合并到新版本的配置文件里。
在新版本的配置上进行改动，不要直接把旧文件直接拷进去。
如果不记得哪些重要文件的话，可以参考 [Important files in Jira
](https://confluence.atlassian.com/adminjiraserver/important-directories-and-files-938847744.html)

#### Step 5. 禁用自动reindex
从Jira 7.x 到 8.x 需要，8.x的升级不需要这个步骤。

### 升级后步骤（仅第一个节点）
这些步骤仅需要在第一个节点完成即可（就是你正在升级的这个节点）， 其他的节点不需要做这一步。

#### Step 1. 首次启动Jira
1. 回到刚才那个安装程序向导， Start Jira Software 8.13.9 now? 选择 Yes [y, Enter]
2. 打开浏览器
3. 根据提示，完成设置。

#### Step 2. （可选）升级Jira Service Management

#### Step 3. 升级app（插件）
@TopCaver注释： 看需要吧，我也没升。 

#### Step 4. 重建索引
@TopCaver注释： 看需要吧，我也没重建。 

#### Step 5. 复制已经升级的Jira安装目录，作为模板
如果在这个节点上升级，并且重建了索引，会非常方便。可以把升级后的这个节点的安装目录保存下来作为模板。用来升级其他的节点。

### 升级其他节点
1. 把模板复制到新的节点上
2. 如果有一些本地目录在不同的节点间有差异的话，修改 setenv.sh 
3. 启动这个节点
4. 重复1-3 步骤，继续升级其他的节点。

### 完成节点升级
至此，节点的升级就完成了。

接下来要完成Jira集群的升级最后阶段  



# 完成升级

最后阶段Jira需要运行一系列的升级任务，并将Jira退出升级模式（out of upgrade mode）一旦这些必要的任务完成，整个升级就完成了。
1.  点击 **Administration > Applications > Jira upgrades.**
2.  点击 结束升级（ **Finalize upgrade**）。 这个按钮只有你所有节点都升级到相同的新版本才可以点击。



# 万一需要回滚

你可以在**完成升级**之前，通过点击**取消升级**取消正在进行的升级。 
取消升级之后，还需要停止那些已经升级的节点，并且重新安装之前的Jira版本，再重新加入到集群中。

