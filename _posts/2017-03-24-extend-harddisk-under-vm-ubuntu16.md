---
title: 为VMware下Linux扩展硬盘空间（Ubuntu 16.04)
tags: 技术 VMWare
---  

团队内部的共享服务器空间不够了，这个共享服务器是在EXSi上一个Ubuntu 16.04虚拟机。把记录扩展VM下linux的磁盘空间过程做一个记录，以备日后查阅。

<!--more-->

## 第一步：扩展VM虚拟机设置
不同的虚拟机软件版本可能略有不同，基本就是编辑虚拟机的磁盘选项，把磁盘空间调大。
![imges](/illustration/extend-harddisk-under-vm-ubuntu16.png)

## 第二步：对新增的磁盘空间写分区，进行格式化

    # fdisk /dev/sda
    //在fdisk交互当中：
可以先看一下当前的磁盘结构：    

    Device     Boot   Start      End  Sectors  Size Id Type
    /dev/sda1  *       2048   999423   997376  487M 83 Linux
    /dev/sda2       1001470 33552383 32550914 15.5G  5 Extended
    /dev/sda5       1001472 33552383 32550912 15.5G 8e Linux LVM

在fdisk交互环境中：

    [Command (m for help): n	//创建新的分区
    Partition type
        p   primary (1 primary, 1 extended, 2 free)
        l   logical (numbered from 5)    
    Select (default p): p  // 选择p创建主分区
    
    Partition number (3,4, default 3): // 默认即可，创建sda3分区
    
    First sector (999424-134217727, default 999424): 33552384 // 此处注意从上面最后一个分区End+1开始，因为默认扇区开始址的值是在sda1和sda2的缝隙里面。
    
    [Command (m for help): t // 修改分区类型
    Partition number (1-3,5, default 5): 3 //选择刚才创建的分区编号3
    Partition type (type L to list all types): 8e  // 创建8e（LVM）分区类型。
    
    [Command (m for help): w // 保存分区表
    The partition table has been altered.
    The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8). // 系统提示重启

重启吧：

    # reboot

## 将分区加入到逻辑卷

    # lvm  //进入lvm配置

可以先lvdisplay一下，看看当前的逻辑卷：

    lvm> lvdisplay
      --- Logical volume ---
      LV Path                /dev/GoVF-NAS-vg/root
      LV Name                root
      VG Name                GoVF-NAS-vg
      LV UUID                XQYNxA-e1Qq-VE95-Xbj6-F7LU-mTiY-tfrACZ
      LV Write Access        read/write
      LV Creation host, time GoVF-NAS, 2017-03-24 11:32:36 +0800
      LV Status              available
      # open                 1
      LV Size                14.52 GiB
      Current LE             3717
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           252:0
   
      --- Logical volume ---
      LV Path                /dev/GoVF-NAS-vg/swap_1
      LV Name                swap_1
      VG Name                GoVF-NAS-vg
      LV UUID                0miZOI-NEJf-CPR9-FjFb-Xdxm-If3v-GM0lRZ
      LV Write Access        read/write
      LV Creation host, time GoVF-NAS, 2017-03-24 11:32:37 +0800
      LV Status              available
      # open                 2
      LV Size                1.00 GiB
      Current LE             256
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           252:1
  
  可以看到现在有两个逻辑卷/dev/GoVF-NAS-vg/root 和 /dev/GoVF-NAS-vg/swap_1 ，他们同属于一个逻辑卷组 GoVF-NAS-vg
  
  然后将上一步创建的分区sda3加入到这个卷组：
  
    lvm> vgextend GoVF-NAS-vg /dev/sda3
    Volume group "GoVF-NAS-vg" successfully extended
  
  扩展逻辑卷，因为我当时只有root和swap，所有存储都在root上，所以扩展root逻辑卷：
  
    lvm> lvextend -L +48G /dev/GoVF-NAS-vg/root
    Size of logical volume GoVF-NAS-vg/root changed from 14.52 GiB (3717 extents) to 62.52 GiB (16005 extents).
    Logical volume root successfully resized.
 
 可以再lvdisplay确认一下，然后quit退出即可：
 
    lvm> lvdisplay
    lvm> quit

## 扩展逻辑卷的文件系统的大小

运行resize2fs，来扩展文件系统：

    # resize2fs /dev/GoVF-NAS-vg/root 

    // 成功之后会有下面的提示    
    resize2fs 1.42.13 (17-May-2015)
    Filesystem at /dev/GoVF-NAS-vg/root is mounted on /; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 4
    The filesystem on /dev/GoVF-NAS-vg/root is now 16389120 (4k) blocks long.

## 参考：
http://www.cnblogs.com/sixiweb/p/3360008.html
http://www.joomlaworks.net/blog/item/168-resizing-the-disk-space-on-ubuntu-server-vms-running-on-vmware-esxi-5

