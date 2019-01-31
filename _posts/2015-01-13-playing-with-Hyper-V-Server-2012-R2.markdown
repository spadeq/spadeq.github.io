---
title: Hyper-V Server 2012 R2动手玩
date: 2015-01-13 17:13:19
categories:
- ITM
- Cloud
tags:
- Hyper-V
- Windows
---
请注意本文要玩的是“Microsoft Hyper-V Server 2012 R2”，而不是Windows Server 2012 R2中的Hyper-V。这两者是有区别的，前面那位是一个独立的sku，安装完成后只有hypervisor的功能，比Windows Server的core安装还要更“瘦”。由于它的瘦，很多事情就只能借助PowerShell命令来进行了。我觉得还是尽量不要使用第三方的图形化工具比较好。
<!-- more -->
本文是在一个比较高大上的环境和需求下来玩，而且会不断更新。至于安装过程这么简单的事情就不在这里说了。

# 关于网络

我们关注网络的两个方面：teaming和VLAN。一个合格的生产环境，核心路由肯定是有冗余，我这边就是把Hyper-V服务器的四个网卡做这样的设计：

|Port|Switch|Payload|
|-|-|-|
|NIC 1|Switch-A|Management|
|NIC 2|Switch-B|Management|
|NIC 3|Switch-A|VMs|
|NIC 4|Switch-B|VMs|

这样就是管理网和生产网分别都是接到两个交换机上。那么网卡1、2需要做teaming，3和4做另一个teaming，以实现LBFO，也就是Load Balancing + Failover。以网口1和2为例，在Hyper-V主机控制台上执行powershell，然后敲命令：

    PS> Get-NetAdapter

这条命令能够列出所有的网卡，记住要编组网卡的Name项，然后执行：

    PS> New-NetLbfoTeam "Mgmt" -TeamingMode SwitchIndependent

随后会提示要你提供TeamMembers，依次将网卡Name给它，当提示`"TeamMembers[2]:"`让你输入第三个网卡时，直接敲回车结束。这里的"Mgmt"是编组后的网络名称，可以自己定。teamingMode一共有三种：Static、SwitchIndependent和LACP，具体区别不在本文范围，应根据自己实际情况而定。交换机的负载均衡模式也有三种：Address Hash地址哈希、Hyper-V Port和Dynamic动态。现在Windows Server 2012 R2默认的是Dynamic方式。如果要修改，可以使用下面的命令：

    PS> Set-NetlbfoTeam Mgmt -LoadBalancingAlgorithm HyperVPort

当然最后那个参数也是要根据实际情况自定的。

编组完成后，要考虑VLAN。我这里四个口上联的交换机端口都是trunk，而管理网络的IP地址必然属于某一个VLAN，所以需要给管理网设置一个VLAN号，才能进行远程管理。命令如下：

    PS> Add-NetLbfoTeamNic Mgmt 190

此处的190就是管理网地址的VLAN号。其实这样做的真正含义是让经过Mgmt网络的数据包默认为VLAN 190，这个网络如果要走其它VLAN的话也是可以的。

接着，最好立即关闭Windows防火墙，省得后面SCVMM连不上。命令：

    netsh advfirewall set allprofiles state off

至此，服务器端的网络基础结构就做好了。执行sconfig，打开服务器配置工具，在里面选8，设置管理地址吧。之后再把远程管理、远程桌面、ping等等都启用，下面的操作都可以远程进行了。（是不是可以绕过域，我还在研究。。）

# 关于SCVMM

其实我的感觉，SCVMM的管理功能并不是Hyper-V Management的超集，M$在这块比起VMware真的差了不是一个档次，SCVMM用一个词概括就是不好用，秉承了M$一向弱爆了的业务逻辑组织能力。

首先，需要在AD里添加两个用户，姑且叫它们scvmm和hyperv，把scvmm加为SCVMM服务器的本地管理员，把hyperv加到Hyper-V主机的本地管理员列表里。安装SCVMM的时候使用scvmm作为服务账号，在添加主机的时候，【指定要用于发现的凭据】，也就是连接Hyper-V主机的账户使用hyperv。

这个所谓的System Center不能做teaming和vlan，无法启用MPIO，我一次次无语，很多事还得登到Hyper-V主机上操作。而HA、VM复制这些事没有SCVMM照样可以完成，所以我觉得真心没必要浪费钱买这么个废柴。。

# 关于SAN

## 磁盘MPIO

在Server Core模式下，管理本地硬盘只能通过命令。。所以最好是找一个带GUI的Server来远程管理之。2012R2的远程管理（不是远程桌面）还是很强大的。打开磁盘管理你会看到一个LUN被识别成好几个磁盘，这是因为没有启用多路径的缘故。我原本以为Hyper-V Server不支持MPIO多路径，事实证明我错了。用Windows Server远程服务器管理那套东西确实开启不了这个功能，但登录到本机通过PowerShell是可以的：

    PS> Install-WindowsFeature Multipath-IO

之后虽然提示无需重启，但还是应该重启一下。有可能你会搜到利用ocsetup安装MPIO的命令，很可惜在R2里面这个命令已经被移除了。。
在普通命令行执行下面命令，可以打开MPIO的图形化控制面板：

    mpiocpl

在【发现多路径】选项卡中，找到硬件ID，选中，点击添加，接受重启，你就会看到一个LUN只对应一个磁盘了。