---
title: AIX 小型机更换硬盘全攻略
date: 2015-01-07 10:24:08
categories:
- ITM
- Devices
tags:
- AIX
- Disk
- Unix
---
当 AIX 小机出现硬盘故障时，不能简单的热插拔更换磁盘，要通过 AIX 系统相关的命令来实现。

1、定位失效硬盘

    # lsvg -lp rootvg

得到类似下面的显示：

    rootvg:
    PV_NAME           PV STATE          TOTAL PPs   FREE PPs    FREE DISTRIBUTION
    hdisk0            missing           546         538         110..101..109..109..109
    hdisk1            active            546         11          00..00..00..00..11

这就说明 hdisk0 目前已损坏，需要更换。下文中提到的 hdisk0 均指该坏盘。

2、拆掉 rootvg 中的 hdisk0

    # unmirrorvg rootvg hdisk0

3、缩减卷组

    # reducevg -d rootvg hdisk0

    0516-914 rmlv：警告，所有属于逻辑卷 lg_dumplv、在物理卷
            hdisk0上的数据将被毁坏。
    rmlv：想要继续吗？y（是）n（否）？y
    rmlv：逻辑卷 lg_dumplv 已删除。

3x、dump 问题

在执行缩减卷组命令时，可能会遇到dump相关的错误：

    # reducevg -d rootvg hdisk0

    0516-914 rmlv：警告，所有属于逻辑卷 lg_dumplv、在物理卷
            hdisk0上的数据将被毁坏。
    rmlv：想要继续吗？y（是）n（否）？y
    0516-1251 rmlv：警告：不能删除逻辑卷 lg_dumplv。
            此逻辑卷还被用作主转储设备。
            复位转储设备并再试命令。
    0516-884 reducevg：不能删除物理卷 hdisk0。

这个时候，就需要修改 dump device 状态。执行以下命令查看 dump 情况：

    # sysdumpdev -l

    主要                 /dev/lg_dumplv
    辅助                 /dev/sysdumpnull
    复制目录             /var/adm/ras
    强制复制标志         TRUE
    总是允许转储         FALSE
    转储压缩             ON

可以看到，存在 /dev/lg_dumplv 这样一个 dump 设备。执行下面的命令查看其物理位置：

    # lslv -l lg_dumplv

    lg_dumplv:N/A
    PV                COPIES        IN BAND       DISTRIBUTION  
    hdisk0            008:000:000   100%          000:008:000:000:000

果然是位于 hdisk0，这就是无法缩减卷组的原因。下面我们把这个主 dump 设备也设成 null：

    # sysdumpdev -P -p /dev/sysdumpnull

    主要                 /dev/sysdumpnull
    辅助                 /dev/sysdumpnull
    复制目录             /var/adm/ras
    强制复制标志         TRUE
    总是允许转储         FALSE
    转储压缩             ON

再进行卷组缩减就没问题了。

4、修改启动顺序

这一步其实特别重要，因为更换硬盘可能会有意想不到的状况，将启动信息在hdisk1上也弄一份会更保险。

    # bosboot -ad hdisk1
    # bootlist -m normal hdisk1 cd0

5、删除磁盘设备

    # rmdev -l hdisk0 -d

    hdisk0 已删除

执行以下两条命令，确认输出信息中不包含任何hdisk0的信息。

    # lspv
    # lscfg -vl hdisk0

6、动手更换物理磁盘

千万要注意，**hdisk0 和 hdisk1 的物理位置不一定是 0 在左边，1 在右边，不要想当然的就把左边的盘给换了！需要通过下面的办法来确定：**

    # diag

然后依次选择：

* 【Enter】
* 【Task Selection (Diagnostics, Advanced Diagnostics, Service Aids, etc.)】
* 【RAID Array Manager】
* 【PCI-X SCSI Disk Array Manager】
* 【诊断及恢复选项】
* 【SCSI和SCSI RAID热插拔管理器】
* 【Identify a Device Attached to a SCSI Hot Swap Enclosure Device】

类似下面的显示：

                  U787B.001.DNW3B33-
      ses0            P1-T14-L15-L0
         slot  1      P1-T14-L8-L0         hdisk1
         slot  2      P1-T14-L5-L0         hdisk0
         slot  3                           [empty slot]
         slot  4                           [empty slot]

这就说明 hdisk0 在右边，要换右边那块！千万别弄错了！
更换完成之后回到 AIX 控制台上继续后面的操作。

7、重新扫描磁盘
    # cfgmgr
    # lspv

8、加入卷组并重建镜像

    # extendvg rootvg hdisk0
    # mirrorvg -c 2 rootvg

9、创建启动项

    # bosboot -ad hdisk0
    # bosboot -ad hdisk1
    # bootlist -m normal hdisk0 hdisk1 cd0

10、重建 dump

    # mklv -t sysdump -y lg_dumplv rootvg 8 hdisk0

这里的 8 是 pp 数，可用数量可以通过 lspv hdisk0 来查看，我这里 free pps 是 11，所以写 8，如果超过 16，可以写 16。

    # sysdumpdev -Pp /dev/lg_dumplv

这样就重新将主 dump 设备改为 lg_dumplv，之后再确认一下

    # lsvg -l rootvg

搞定收工。