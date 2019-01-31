---
title: 使用NetBackup备份虚拟化平台
date: 2015-02-04 17:02:29
categories:
- ITM
- Cloud
tags:
- VMware
- NetBackup
- Backup
---
本文将要讨论的是使用两台Windows Server安装NetBackup 7.6.1备份环境，用来进行ESXi和Hyper-V虚拟化平台的整体备份。

两台服务器分别为nbumaster和nbumedia，很显然一台是Master Server，另一台作为Media Server。这种设计是为了能够利用Media Server的重复数据删除功能。如果想把这两台服务器装成虚拟机，可以参照之前我介绍NPIV的文章，对磁带机、FC LUN进行映射，本文不再提及基础环境的搭建，只专注于NetBackup本身。
<!-- more -->

# NetBackup的安装

NBU 7.6.1的安装介质已经支持直接安装在Windows Server 2012 R2上，建议有条件的都使用这样的组合。Linux下配置NBU简直堪比AIX上配HA，太麻烦，太不靠谱。有DNS的情况下，请将master和media两台服务器的IP进行解析，否则请写hosts，因为NBU需要使用域名来进行相互通信。

Master Server的安装无需多言，完全按照默认值就可以了。Media Server要注意安装时填写Master Server的域名，别的也一切按照默认。

都装好之后，在Master Server启动控制台，选择【NetBackup Management】->【Host Properties】->【Master Servers】，找到nbumaster，双击之。

在【Servers】中的【Media Servers】选项卡里，点击【Add】，然后输入Media Server的域名，点击添加然后确定。

请注意，如果在这一步出现了“unable to save data on some hosts”、“file write failed (14)”错误，可能是因为开启了UAC的缘故。请关掉NBU控制台，然后以管理员权限重新运行。

上面工作做完之后，打开管理员模式命令行，在nbu安装目录bin下执行`bpdown /f /v`（两台机器都要做），然后先后在Master和Media上执行`bpup /v`，重启nbu各项服务。

在Master Server的主机属性里，建议把【Global Attributes】中的Maximum jobs per client设成99，【Timeouts】里的各项时间也适当调大。

# 备份介质配置

用来备份虚拟机的备份介质常见的是三种：磁带、本地磁盘和PDDE去重池。磁带建议还是算了。。而PDDE虽然是最佳选择但我的key没有这功能的授权。。所以就只能用基本磁盘的方式了。。

首先，把备份池磁盘挂载到Media Server上，无论是FC san还是IP san还是本地盘都可以，注意一下如果分区大于2TB，则需要把磁盘弄成GPT分区格式。Server 2012 R2可以将其格式化为ReFS文件系统，分配一个盘符（比如N:）。

然后登陆Master Server（注意以后对NBU的操作一般都是在Master上操作），打开NBU控制台，选择左边的【NetBackup Management】->【Storage】，右键点击【Storage Units】，选择【New Storage Unit】。

在弹出的对话框中，为这个存储单元设一个名字，然后选择Storage unit type为Disk，Disk type为BasicDisk。下面Media Server选择我们的Media Server主机名，填入目录绝对路径（比如N:\，也可以点击Browse手动选择）。再下面那些参数可以保留默认值，点击OK。这样，我们就拥有了一个磁盘备份介质。

# vSphere备份设置

## 建立NBU与vCenter的连接

登录Master Server的NBU控制台，选择左边的【Media and Device Management】->【Credentials】->【Virtual Machine Servers】。右键点击右边的空白处，选择New，新增一个凭据。

在弹出的对话框中，选择Virtual machine server type为VMware Virtual Center Server，然后输入vCenter的用户名、密码。注意，NBU很傻缺的认为用户名不应该有@，所以administrator@vsphere.local不能用，带有后缀的域账号不能用，必须用domain\\username的形式。点击OK后NBU会去跟vCenter做验证，验证通过后，这一条目就添加完成了。

在左边选择【NetBackup Management】->【Host Properties】->【Master Servers】，双击我们的Master Server，选择【VMware Access Host】，然后点击Add。

[先占坑]

## 开启NBU Web服务

在Master Server上，创建nbwebgrp组和nbwebsvc用户。既可以是域账号也可以是本地的（建议后者）。不要给nbwebsvc任何额外的权限，比如远程登录。然后进入管理工具->本地安全策略，找到【安全设置】->【本地策略】->【用户权限分配】，然后在右边右键点击“作为服务登录”，选择【属性】，将nbwebsvc添加进去，然后关闭这个对话框。

下面启动WMC服务。打开命令行（管理员方式），进入nbu安装目录\NetBackup\wmc\bin\install\，运行：

    setupWmc.bat -password PASSWORD

后面那个PASSWORD就是nbwebsvc的密码。完了应该提示在8443端口启动了服务。可以找一台机器访问<https://[server]:8443/nbwebservice/application.wadl>，如果出现HTTP 401，表示服务已启用。

继续在上面的路径执行命令：

    manageClientCerts.bat -create vCenter\_plugin\_host

其中vCenter-plugin_host是vCenter的FQDN。执行完毕会生成一个证书的zip包，并且提示你zip文件存放的位置（一般是nbu安装目录\NetBackup\var\global\wsl\credentials\clients\）。把它复制出来。

打开vSphere Client，在【主页】中打开【Symantec NetBackup】，然后点击【Add/Remove Servers】。在弹出的对话框中，输入Master Server的FQDN，然后选择那个zip文件，点击后面的【Add Server】。如果前面操作都没有错误，应该提示成功。这样就可以在这个页面里进行虚拟机的恢复了。