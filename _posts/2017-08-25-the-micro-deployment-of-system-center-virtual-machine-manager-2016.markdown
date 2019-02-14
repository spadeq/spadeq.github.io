---
title: System Center VMM 2016 微型部署
date: 2017-08-25 11:50:40
updated: 2017-10-01 16:50:40
categories:
- ITM
- Cloud
tags:
- Windows
- Hyper-V
- System Center
---
我之所以想写这一篇，是因为现在全网都找不到一个真正靠谱的，针对微型虚拟化环境的 SCVMM 2016 部署指南。各种言之不详，各种大部头，让人看了搞不清状况。那么我所说的微型部署，是希望能够像 vSphere 那样，hypervisor 主机加上一个虚拟化的管理服务器就把一切事情都搞定，除了域控外，所有主要服务都像 vCSA 那样，集成到一台虚拟机中。主机数量限制在个位数，所有的管理组件完全由一台服务器来实现，并且除了 hypervisor 主机外不需要任何物理服务器。
<!-- more -->
于是，我们需要做的事情就清晰了。

首先，在裸机上装好 Windows Server 2016，作为 hypervisor。开启 Hyper-V 功能即可。每台物理服务器至少需要两个物理网络接口，一个作为生产网络，一个作为集群内部通信网络（建议万兆），当然，这两个网络中都可以包含更多的物理网卡接口，然后在 Windows 中做 Team。这部分内容就不多展开，因为每个人的实际物理环境差别很大。我们就认为生产网络和集群内部网络各自具备一个接口。由于两台主机和十台主机本质上也是相同的，所以本文为了方面都说是两台物理主机。

# 准备工作

## 创建虚拟交换机

在建立虚拟机之前，必须先创建虚拟交换机。

1. 打开 HyperV 管理控制台，选择右侧的「Virtual Switch Manager」；
1. 点击「New virtual network switch」，右边选择「External」，点击「Create Virtual Switch」；
1. 取个名字，比如「VS_Product」，选择网卡，可以是单独的一个生产网卡，也可以是 Team 之后的虚拟网卡；
1. 如果生产和管理都走这个网卡，那么还需要勾选「Allow management opertating system to share this network adapter」；
1. 如果该网卡接入交换机的 trunk 口，管理地址需要打 vlan tag，则在下面启用 vlan 标识，填入 vlan 号；
1. 点击 OK。

在两台物理主机均要做此操作，且虚拟交换机的名称必须一致，否则会导致后面集群创建不成功。管理服务器的 IP 可能需要重新配置到 VS\_Product 这个虚拟交换机上。

## 安装第一台 Windows 虚拟机

此时，HyperV 主机还是比较弱鸡的，只能单机跑，而且虚拟机也只能存储在本地。但是我们必须从这一步开始做起，通过本地的 HyperV 管理器来安装一个 Windows Server 2016 的虚拟机，这台虚拟机就作为我们的域控制器来使用。启用域控也不在这里说了，假定域名为 spadeq.com。然后将外面的两台物理主机都加到域中，主机名分别叫做 hv01.spadeq.com 和 hv02.spadeq.com。

在这里约定一下地址的规划。所有生产网络（包括管理使用）的地址使用 11.64.19.128/25 网段，网关 11.64.19.254。集群内部通讯地址使用 192.168.1.0/24 网段，不需要网关。

两台物理主机的地址使用 11.64.19.211 和 11.64.19.212。

## SCVMM服务器组成

在其中一台物理机上安装第二个虚拟机，即 SCVMM 的管理服务器。本文针对的是常规非大规模部署私有云架构，且 VMM 本身不做高可用设计，所有 VMM 相关的服务包括 SQL Server 数据库均安装在一台 VM 中。共计涉及的服务器信息如下：

|服务器|主机名|地址|
|-|-|-|
|VMM 管理服务|vmm-mgmt.spadeq.com|11.64.19.215|
|AD 域控制器     | addc1.spadeq.com       | 11.64.19.180 |
|Hyper-V 主机1 | hv01.spadeq.com        | 11.64.19.211 |
|Hyper-V 主机2 | hv02.spadeq.com        | 11.64.19.212 |
|Hyper-V 集群  | hvc.spadeq.com         | 11.64.19.214 |

## 创建用户

在域控制器上新建一个OU（比如叫做 Service Accounts），然后创建下列用户或用户组：

| 类型   | 名称                | 说明              |
| ---- | ----------------- | --------------- |
| 用户   | SPADEQ\vmm-svc    | SCVMM 的服务账户      |
| 用户   | SPADEQ\vmm-admin  | SCVMM的 Run As 用户  |
| 用户组  | SPADEQ\vmm-admins | SCVMM 管理员安全组     |
| 用户   | SPADEQ\sql-svc    | SQL 服务用户         |
| 用户   | SPADEQ\sql-admin  | SQL Server 系统管理员 |

每个用户在创建时，注意将「Password never expires」和「User cannot change password」勾选上（选不了的请查看域的组策略设置）。创建完成后，将 vmm-svc 用户添加到 vmm-admins 组里。

### 设置本地管理员权限

登陆 vmm-mgmt 主机，在计算机管理中，将 vmm-admins 组、vmm-admin、sql-svc 和 sql-admin 用户添加到本地的 Administrators 里，使之具备本地管理员的权限。

# 安装 SQL Server

以 sql-admin 用户登录 vmm-mgmt 服务器，然后安装 SQL Server 2016。

1. 勾选 SQL Server（下面子功能不需要）、Reporting Service 和 Client Tools Connectivity
1. 默认实例
1. 将几个服务账户都改成 SPADEQ\sql-svc，填入口令，启动类型都选自动。（Browser 无需修改用户名）
1. 排序方式选择「SQL\_Latin1\_General\_CP1\_CI\_AS」
1. SQL 管理员，先 Add Current User，然后把 sql-svc 也加进去
1. 安装必要的更新

# 安装 SCVMM

## 先决条件

以 vmm-admin 用户登录 vmm-mgmt 服务器。

首先需要安装 Windows 10 ADK。从微软网站下载的实际上是一个下载器，名为 adksetup.exe。执行之，安装以下两项：

* Deployment Tools
* Windows PE

另外还需要安装 SQL Server Command Line Utilities。

## 配置 DKM

DKM 是用来存储凭据的，增强服务器间交互的安全性。

1. 去域控制器上，运行 adsiedit.msc。
1. 右键点击，连接到，保持默认下一步
1. 展开默认命名，展开「DC=spadeq,DC=com」
1. 右键点击「DC=spadeq,DC=spadeq」，新建，对象
1. 选择 container，下一步
1. 值填写 VMMDKM，下一步，完成
1. 打开 Active Directory Users and Computers 管理控制台，主菜单选择 View -> Advanced Features
1. 右键点击 VMMDKM，属性
1. 安全选项卡，添加
1. 输入 vmm-admins，添加进去，然后把权限勾选读取、写入、创建所有子对象。
1. 点高级，选择 vmm-admins，点编辑，然后在应用于里面选择「这个对象及全部后代」（This object and all descendant objects）。
1. 一路点确定。
1. 重复 8 ~ 12，只是用户改成 vmm-admin，权限则是完全控制。

## 安装 VMM 管理服务

1. 以 vmm-admin 用户登陆服务器。以管理员方式运行 vmm 的安装文件。注意 ISO 里的那个文件只是个自解压程序，还要去对应目录执行 setup 才可以。
1. 点击 Install，勾选 VMM management server，下一步
1. 输入用户、单位和序列号，下一步
1. 我已阅读，下一步、下一步
1. 外网勾选更新，内网关闭之，下一步
1. 选择安装目录，下一步
1. 数据库配置。服务器名可以用本机的主机名，或者 localhost。勾选使用下面凭据，用户使用 SPADEQ\sql-admin，输入口令，其它不用改。下一步
1. 服务账户配置。服务账户使用 SPADEQ\vmm-svc。下面的 DKM 勾选 Store my keys in Active Directory，名称为 `CN=VMMDKM,DC=spadeq,DC=com`。下一步
1. 保持端口号不变，下一步
1. 保持库的选项不变，下一步
1. 下一步直到完成。

如果在这里遇到 DKM 的问题，请回到上一节，仔细检查用户权限的分配是否严格遵照我的指示进行。

## 安装更新

可以在微软官网下载 SCVMM 的离线更新（搜索 Rollup Update for Microsoft System Center 2016 VMM）。下回来的 .cab 文件可以解压成为标准的 msp，包括 vmmserver 和 adminconsole 两个，都要执行。完了重启服务器。

现在可以用桌面上的 VMM Console 连接本地的 VMM 服务器，所有信息都可以保持默认。在 VMware 早就全面转向 B/S 架构的时候，微软始终坚持 C/S 玩法，现在看来略土。

## 创建运行为（Run As）账户

1. 左面选择 Settings 标签
1. 点击上方 Ribbon 界面里的 Create Run As Account。
1. 起个名字，比如 Hyper-V 主机管理用户
1. 用户名 SPADEQ\vmm-admin，输入口令，确定

为了将来管理方便，可以再以一个域管理员的身份创建另一个 Run As 用户。

# 基础架构创建

基础架构指的是物理主机及其物理网络、虚拟网络、虚拟交换机等。

## 创建主机组

1. 左侧 Fabric 选项卡
1. Ribbon 的 Fabric Resources，在左侧 All Hosts 上右键，创建主机组
1. 输入主机组名称，确定

## 添加物理主机

1. 将 hv01 和 hv02 两台主机的 Administrators 组里，加入 vmm-admin 用户。
1. 左侧 Fabric 选项卡。Add Resources，选择 Hyper-V Hosts and Clusters
1. 第一项，下一步。
1. 选择之前创建的 Run As 用户，下一步
1. 输入计算机名：hv02.spadeq.com。为了保险起见，先添加 VMM 不在的那一台。
1. 打勾，下一步，OK
1. 选定一个主机组来存放这台主机，下一步。
1. 下一步直到完成。
1. 反复上述流程添加全部主机。

## 创建集群

1. 左侧 Fabric 选项卡，上面的 Create -> Hyper-V Cluster；
1. 给集群起个名字，注意，这个名字就是集群地址的主机名（我这里是 hvc.spadeq.com），写到 AD 里的，要确保不能有重名；
1. 「Host group」要选择这两台主机所在的主机组，不能是父组；
1. 若要启用存储空间直通功能，勾选下面的「Enable Storage Spaces Direct」，下一步；
1. 选择一个 Run As 账户。这个 Run As 账户最好是域管理员，否则无法创建集群的主机名。下一步；
1. 选中主机，下一步；
1. 勾选网络，然后提供一个集群使用的 IP 地址，下一步直到结束。

在等待进度的时候如果遇到远程桌面退出，主机和虚拟机都无法访问的话，不要紧张，这是因为服务器重启。虚拟机会挂起，不会丢失数据。

如果集群创建失败，大概率可能是在集群验证时就出错。可以登录到物理主机上，查看 %windir%\cluster\reports 目录下的

## 创建虚拟网络

我们假定现在有三个网络，分别是生产网络、集群心跳网络和主机管理网络。

### 创建逻辑网络

1. Fabric 选项卡
1. Ribbon 工具中的 Create Logical Network
1. 选择一个名称，比如 LN_Product，并填一个描述。
1. 下一步
1. 网络站点配置。点击添加。
1. 选择可以使用该网络站点的主机组（我们这里是 All Hosts）。为站点起一个名字，比如「生产网络-合肥」。
1. 在下面添加 VLAN 和对应的网络。比如 VLAN 80，对应 11.64.17.0/24。将所需的子网都加入。
1. 下一步，Finish。

### 创建IP地址池

1. Fabric 选项卡 -> Ribbon 工具栏 Fabric Resources，左边点 Logical Networks，然后选择一个逻辑网络。
1. 点击 Create IP Pool
1. 起个名字，选择对应的逻辑网络
1. 选择 IP 子网，下一步
1. 输入 IP 起止范围，下面把集群地址可能使用的地址以及保留的地址（比如网关地址）写进去
1. 输入网关地址等，下一步
1. 下一步直到结束。
1. 反复以上过程，添加所有 IP 地址池。

### 绑定物理网卡

做这一步的时候可能有人会担心导致网络中断。其实不会的，Logical Network 绑定网卡的时候，并不会取消原先单机下 vSwitch 对网卡的绑定，故管理地址仍可用，甚至原先用 vSwitch 的虚拟机也不会受到影响。

1. 左边选择 Fabric 选项卡，然后找到 Servers 下的主机组；
1. 从右边的面板中选中一个服务器，右键点击一台服务器，选择 Properties；
1. Hardware，然后找到我们需要用到的网卡；
1. 在 Logical network 中，勾选允许通过的逻辑网络；
1. 点击 OK 确认。

### 转换为逻辑交换机

之前创建的标准交换机是无法在 SCVMM 中进行高级管理的，需要转换为逻辑交换机才能用到各种 SDN 的功能。

1. 左侧 Fabric 选项卡，选择上面的「Create Logical Switch」；
1. 创建一个逻辑交换机，名称**必须与之前用的标准交换机相同**；
1. Uplink Mode，这要看之前你创建标准虚拟交换机的时候有没有做双网卡绑定，如果做过 Teaming，这里就要选择「Team」；
1. 下一步的 Minimum bandwidth mode 要选择「Absolute」。是否启用 SR-IOV 也要与之前标准交换机的选择一致，一般是不选的；
1. 一直下一步到 Uplinks，选择 Add，创建新的端口档案，名字可以随便起，但是一定要注意负载均衡算法这里要跟之前做网卡 Teaming 的时候一致。对于 Windows Server 2016 来说，默认是 Dynamic，下面的 Teaming Mode 一般是 Switch Independent。Network Sites 也要选择标准交换机一样的 Site。
1. 前面的都做完后，还是在 Fabric 选项卡中，选取 Servers，然后在右侧右键点击其中一台服务器，选择属性；
1. 选择 Virtual Switches，选中之前那个标准交换机，然后点击下方的 Convert 按钮。如果这个按钮是灰色的，请检查下面几项：
    * 逻辑网络是不是绑定到物理网卡上了，而且物理网卡就是标准交换机使用的网卡；
    * 逻辑交换机和标准交换机名字是不是一致；
    * 最小带宽模式是否一致（一般的标准交换机都是 Absolute）；
    * Uplink 端口模式与主机的 Teaming 参数是否一致。

### 修复虚拟交换机

如果此时给通过这一虚拟交换机的 VM 设置 VLAN，有可能网络是无法联通的。仔细看 Jobs 会发现有一条错误：

> **Error (26908)**
>
> The virtual network adapter cannot be connected because the virtual switch 'xxxxxx' on the host is not in compliance with the settings specified in the logical switch.

这时候就需要对虚拟交换机进行修复。操作步骤如下：

1. Fabric 选项卡，选择「Logical Switch」项，然后在 Ribbon 界面里找到 Hosts 按钮，切换到主机视图；
1. 找到主机下面的虚拟交换机名称，可以看到右边的 Network Compliance 栏会显示「Not Compliant」；
1. 右键点击这个虚拟交换机，然后选择 Remediate；
1. 同样的，对于集群中的所有服务器都进行相同的操作。

# 存储管理

对于 FC 存储，SCVMM 表面上提供了非常齐全的接口，比如 SMI-S，能直接管理到磁盘阵列，实现划 LUN 等操作。但实际上多数实践中，映射 LUN 给主机这件事我们并不希望由云管理平台去完成，而对于已经映射过来的 LUN，SCVMM 就没有提供像 vCenter 那样的管理界面，你就只能在物理主机上配置，然后做成 CSV 的形式。

在这里，我用的是 Storage Spaces Direct，即 S2D，做了一个超融合架构。下面我来介绍一下如何在 SCVMM 里管理 S2D 存储。

## 添加 S2D 提供者

1. Fabric 页面，点击上面的「Add Resources」->「Storage Devices」；
1. 第一项「Windows-based file server」，下一步；
1. IP 填集群管理地址或者域名 hvc.spadeq.com，选择管理的 Run As 用户，下一步；
1. 会显示出整个 S2D 的基本情况，Clustered Windows Storage on hvc，下一步；
1. 创建一个对应的 Classification，比如叫 Platinum Pool，随你喜欢；
1. 下一步直到完成。

如果存储池没有创建，则这一步中会比较恶心，只能选择其中一台物理主机上的磁盘。所以建议在故障转移群集中创建 S2D 的 pool，到这里来加入，就没有这个问题了。

## 创建卷

从 SCVMM 2016 起，操作不像以前那样可以右键点击 pool 然后新增卷了。如果直接 Create LU，则无法映射给主机。正确做法如下：

1. Fabric 页面，右键点击 Servers 列表里的集群，点「Properties」；
1. 左边选择「Shared Volumes」；
1. 弹出对话框，给该共享卷起个名字，比如「VD_Test」；
1. 选择一个 Classification，就是前面创建的；
1. 指定共享卷的大小，以及文件系统。如果选择 NTFS，还可以启用重复数据删除功能；
1. 下一步到结束。

再次进入到 Shared Volumes 这个配置页，可以看到 CSV 的路径列表。在服务器上看到的地址就是类似「C:\ClusterStorage\Volume1」的样子，也没法在 SCVMM 里面进行改名。将就用吧。

# 集群优化

现在我们的虚拟网络和共享存储都做好了，原先临时性的集群设置需要进行优化配置。

## 选择集群实时迁移网络

默认情况下，集群的虚拟机做实时迁移会使用所有的网络接口。但如果集群内部通信接口是高速网络，则可以禁止实时迁移流量走生产网络，一方面提速，另一方面也降低对生产流量的影响。

1. 登录其中一台物理主机，打开故障转移群集管理控制台；
1. 展开群集，点击网络；
1. 点击右面的「Live Migration Settings」；
1. 取消勾选生产网络，确定。

## 为集群添加仲裁

三台机器以上的集群，仲裁并不是必须的。但我们这里的两台机器如果没有仲裁，可能会引起脑裂而导致整个集群挂掉。由于采用 S2D 的超融合架构，所以没有共享的 FC 磁盘可以使用。因此，只能通过额外设置一台服务器，通过 SMB 共享实现文件仲裁，或者通过 iSCSI 映射共享卷实现磁盘仲裁。具体的方式不在此展开。

## 虚拟机高可用化

之前我们的虚拟机都是在单机上创建的，现在一方面要将虚拟机从本地存储转移到共享存储，另一方面，要开启虚拟机的高可用。

1. 右键点击 AD 的虚拟机，选择「Migrate Virtual Machine」；
1. 勾选「Make this VM highly available」；
1. 选择另一个主机（正好借机测试集群网络），下一步；
1. 选择一个共享的 CSV 卷，比如之前创建的那个 Volume1，下一步；
1. 保证虚拟交换机和 VLAN 值正确，下一步；
1. 点击 Move 开始。

建议刚开始使用 AD 的机器来试验，而不要直接用 VMM 的虚拟机，否则万一出现网络问题，可能会导致未知后果。

## 创建库

SCVMM 的设计中，库（Library）是很重要的一个组成部分。库甚至能存放虚拟机。当然我们使用中最大的用处是放一些 ISO。

在默认安装中，会在 vmm-mgmt 主机创建一个叫做 MSSCVMMLibrary 的库，对于微型的架构，用 VMM 本机作为库就可以应付使用要求了。下面举例如何将一个 ISO 导入到库里。（直接复制到库所在的目录是不行的）

1. 先将 ISO 复制到 VMM 服务器上，或者用 SMB；
1. 左侧选择「Library」，然后点击上方的「Import Physical Resource」；
1. 点击「Add Resource」，选择 ISO 文件；
1. 点击「Browse」，选择放置 ISO 文件的目录。如果想新建一个，比如 ISOs，那么就展开 MSSCVMMLibrary 节点，然后点击下方的「Explore directory」，在里面直接新建文件夹；
1. 重复点击「Add resource」可以一次性添加多个 ISO 文件；
1. 点击「Import」按钮，等待导入成功。