---
title: 使用 NetBackup 备份虚拟化平台
date: 2015-02-04 17:02:29
categories:
- ITM
- Cloud
tags:
- VMware
- NetBackup
- Backup
---
本文将要讨论的是使用两台 Windows Serve 安装 NetBackup 7.6.1 备份环境，用来进行 ESXi 和 Hyper-V 虚拟化平台的整体备份。

两台服务器分别为 nbumaster 和 nbumedia，很显然一台是 Master Server，另一台作为 Media Server。这种设计是为了能够利用 Media Server 的重复数据删除功能。如果想把这两台服务器装成虚拟机，可以参照之前我介绍 NPIV 的文章，对磁带机、FC LUN 进行映射，本文不再提及基础环境的搭建，只专注于 NetBackup 本身。
<!-- more -->

# NetBackup 的安装

NBU 7.6.1 的安装介质已经支持直接安装在 Windows Server 2012 R2 上，建议有条件的都使用这样的组合。Linux 下配置 NBU 简直堪比 AIX 上配 HA，太麻烦，太不靠谱。有 DNS 的情况下，请将 master 和 media 两台服务器的 IP 进行解析，否则请写 hosts，因为 NBU 需要使用域名来进行相互通信。

Master Server 的安装无需多言，完全按照默认值就可以了。Media Server 要注意安装时填写 Master Server 的域名，别的也一切按照默认。

都装好之后，在 Master Server 启动控制台，选择「NetBackup Management」->「Host Properties」->「Master Servers」，找到 nbumaster，双击之。

在「Servers」中的「Media Servers」选项卡里，点击「Add」，然后输入 Media Server 的域名，点击添加然后确定。

请注意，如果在这一步出现了「unable to save data on some hosts」、「file write failed (14)」错误，可能是因为开启了 UAC 的缘故。请关掉 NBU 控制台，然后以管理员权限重新运行。

上面工作做完之后，打开管理员模式命令行，在 nbu 安装目录 bin 下执行 `bpdown /f /v`（两台机器都要做），然后先后在 Master 和 Media 上执行 `bpup /v`，重启 nbu 各项服务。

在 Master Server 的主机属性里，建议把「Global Attributes」中的 Maximum jobs per client 设成 99，「Timeouts」里的各项时间也适当调大。

# 备份介质配置

用来备份虚拟机的备份介质常见的是三种：磁带、本地磁盘和 PDDE 去重池。磁带建议还是算了。。而 PDDE 虽然是最佳选择但我的 key 没有这功能的授权。。所以就只能用基本磁盘的方式了。。

首先，把备份池磁盘挂载到 Media Server 上，无论是 FC san 还是 IP san 还是本地盘都可以，注意一下如果分区大于 2 TB，则需要把磁盘弄成 GPT 分区格式。Server 2012 R2 可以将其格式化为 ReFS 文件系统，分配一个盘符（比如 N:）。

然后登录 Master Server（注意以后对 NBU 的操作一般都是在 Master 上操作），打开 NBU 控制台，选择左边的「NetBackup Management」->「Storage」，右键点击「Storage Units」，选择「New Storage Unit」。

在弹出的对话框中，为这个存储单元设一个名字，然后选择 Storage unit type 为 Disk，Disk type 为 BasicDisk。下面 Media Server 选择我们的 Media Server 主机名，填入目录绝对路径（比如 N:\，也可以点击 Browse 手动选择）。再下面那些参数可以保留默认值，点击 OK。这样，我们就拥有了一个磁盘备份介质。

# vSphere 备份设置

## 建立 NBU 与 vCenter 的连接

登录 Master Server 的 NBU 控制台，选择左边的「Media and Device Management」->「Credentials」->「Virtual Machine Servers」。右键点击右边的空白处，选择 New，新增一个凭据。

在弹出的对话框中，选择 Virtual machine server type 为 VMware Virtual Center Server，然后输入 vCenter 的用户名、密码。注意，NBU 很傻缺的认为用户名不应该有「@」，所以 administrator@vsphere.local 不能用，带有后缀的域账号不能用，必须用 domain\\username 的形式。点击 OK 后 NBU 会去跟 vCenter 做验证，验证通过后，这一条目就添加完成了。

在左边选择「NetBackup Management」->「Host Properties」->「Master Servers」，双击我们的 Master Server，选择「VMware Access Host」，然后点击 Add。

~~（先占坑）~~

## 开启 NBU Web 服务

在 Master Server 上，创建 nbwebgrp 组和 nbwebsvc 用户。既可以是域账号也可以是本地的（建议后者）。不要给 nbwebsvc 任何额外的权限，比如远程登录。然后进入管理工具->本地安全策略，找到「安全设置」->「本地策略」->「用户权限分配」，然后在右边右键点击「作为服务登录」，选择「属性」，将 nbwebsvc 添加进去，然后关闭这个对话框。

下面启动WMC服务。打开命令行（管理员方式），进入 nbu 安装目录 \NetBackup\wmc\bin\install\，运行：

    setupWmc.bat -password PASSWORD

后面那个 PASSWORD 就是 nbwebsvc 的密码。完了应该提示在 8443 端口启动了服务。可以找一台机器访问 <https://[server]:8443/nbwebservice/application.wadl>，如果出现 HTTP 401，表示服务已启用。

继续在上面的路径执行命令：

    manageClientCerts.bat -create vCenter_plugin_host

其中 vCenter-plugin_host 是 vCenter 的 FQDN。执行完毕会生成一个证书的 zip 包，并且提示你 zip 文件存放的位置（一般是 nbu 安装目录 \NetBackup\var\global\wsl\credentials\clients\）。把它复制出来。

打开 vSphere Client，在「主页」中打开「Symantec NetBackup」，然后点击「Add / Remove Servers」。在弹出的对话框中，输入 Master Server 的 FQDN，然后选择那个 zip 文件，点击后面的「Add Server」。如果前面操作都没有错误，应该提示成功。这样就可以在这个页面里进行虚拟机的恢复了。