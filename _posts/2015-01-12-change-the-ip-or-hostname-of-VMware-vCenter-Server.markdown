---
title: 更改 VMware vCenter Server 的 IP 或主机名
date: 2015-01-12 11:09:16
updated: 2019-03-03 21:07:00
categories:
- ITM
- Cloud
tags:
- VMware
---
无论是基于 Windows 的 vCenter Server，还是基于 Linux 的 vCenter Server Appliance，一旦部署完毕，想更改 IP 或是主机名（加入不同的 Windows 域中），都会导致服务无法启动。本文以 vCenter Server 5.5 为例，完整介绍了更改其网络信息的正确方法。
<!-- more -->

# 周边服务

首先，对于网络架构变化，需要将域控制器、DNS、数据库服务器等周边服务的地址、IP 理顺。随后就可以直接更改掉 vCenter 的主机名或 IP。为了表述方便，本文以 vCenter 使用 FQDN 为例说明。

# 重新安装SSO

登陆到已经修改好 FQDN 的 vCenter 服务器，修改 lookup service 的服务地址。该地址存放在 %programdata%\VMware\ls_url.txt 文件中。其中内容只有一个类似于 <https://[FQDN]:7444/lookupservice/sdk> 的地址，修改之，然后重新启动服务器。

卸载 Single Sign-On 5.5，然后清理下列文件夹（删除或改名）：

* `%programdata%\VMware\CIS`
* `%programdata%\MIT`

重新安装 Single Sign-On 5.5 组件。重装的目的是为了重新生成 SSL 证书。

# 重注册 vCenter

打开命令提示符（管理员），进入下面的文件夹 C:\Program Files\VMware\Infrastructure\Inventory Service\scripts

运行下面的命令，注意替换 SSO 管理员口令：

    is-change-sso.bat https://<FQDN>:7444/lookupservice/sdk "administrator@vsphere.local" "SSO_pw1@"

重启服务器，查看 vCenter 是否工作正常。

参考：
> <http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2062921>
> <http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2050273>