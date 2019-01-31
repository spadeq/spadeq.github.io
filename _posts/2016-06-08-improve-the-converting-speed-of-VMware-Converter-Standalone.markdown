---
title: 提高 VMware Converter Standalone 的转换速度
date: 2016-06-08 16:20:05
categories:
- ITM
- Virtulization
tags:
- VMware
- Cloud
---
很多人会使用 VMware Converter Standalone 作为 P2V 的工具，但这个工具的默认设置，会使得直连 vCenter 的转换速度在某些场合下慢的离谱。VMware 的 KB 其实已经 [提供了解决的方法](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2107851)，本文对其精华进行提炼，供参考。

<!-- more -->

# 提高连接并发数

默认情况下，每个任务会限制为仅使用一个连接。修改方法为，在 VMware Converter Standalone 的界面中，打开菜单 [Administration]–>[Data connections per task]，选中第一项 Maximum，确认退出。

# 取消 SSL 加密流

如果是在可信任网络环境中，可以取消数据流加密，提高性能。VMware 的 KB 也对此有 [专门的描述](https://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=2020517)。

首先，找到 converter-work.xml 文件，其位置在：

* Windows 7/Server 2008 及更新版本 – `C:\ProgramData\VMware\VMware vCenter Converter Standalone`
* Window Vista, XP and 2003 Server – `%ALLUSERSPROFILE%\VMware\VMware vCenter Converter Standalone`
* 更早期的 Windows 版本 – `%ALLUSERSPROFILE%\Application Data\VMware\VMware vCenter Converter Standalone`

在其中找到 `<nfc>` 标签中的 `<useSsl>true</useSsl>`，将其中的 `true` 改为 `false`，保存。

之后运行 services.msc，找到 VMware vCenter Converter Standalone Worker 服务，重启之。

经过以上配置，Converter 就可以用较快的速度进行 P2V 了。