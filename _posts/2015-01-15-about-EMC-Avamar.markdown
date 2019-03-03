---
title: EMC Avamar 相关问题
date: 2015-01-15 17:10:15
categories:
- ITM
- Cloud
tags:
- Avamar
- Backup
---
用 EMC Avamar 与 VMware 搭配备份虚拟机的体验是很棒的。不过 Avamar 的技术支持就不那么令人愉悦，比 Documentum 的资料还少。知识点不好关联，我想到啥就写点啥。。算是一个拼凑合集，以作备忘。
<!-- more -->

# Avamar Proxy 设置

Proxy 的部署没什么好说的，一个 ova，用 vCenter 导入就行了。之后开机，打开控制台，进行设置。

网络方面也很简单，控制台里面就有一个 Network Config，照着提示一步步把 IP、网关、DNS、主机名都配好。回退到控制台，选择 Login。Avamar Proxy 的默认 root 口令是 avam@r，注意跟 Avamar Server 默认的 changeme 不同。登进去之后，需要做两件事。

一是将 Avamar Server 的地址，通常是节点一的地址，写进 hosts 里。而且一定要注意，hosts 里写的主机名，必须跟 Avamar Server 上配的一样，否则光是 IP 写对了还不行。我昨天因为重建 Windows 域，改掉了服务器所在的域名，结果就导致备份任务始终处于 waiting-client 的状态，直至超时。当然，如果 DNS 服务器给力的话，依靠 DNS 解析也是可以的。但最好还是交给本地 hosts 进行解析。

二是将 proxy 注册到 server 上。如果是首次启动 proxy，会自动进入这个向导，我建议是取消向导，因为这时你的网络还没有完全配好，特别是主机名，在 ova 模板部署的时候是不会要求你提供的，这样注册到 server 上的全都是 local 的名字，很混乱。取消向导，等网络设置全部完毕后，再进行注册。

注册之前，请在 avamar server 上创建一个独立的 domain，比如叫做 avproxy，用来存放 proxy。登录到 proxy 上，login。这时你可能会搜到教程告诉你 avregister 这个命令，很可惜，它已经过时了，请使用下面这条语句：

```shell
/usr/local/avamarclient/etc/initproxyappliance.sh start
```

这其实就是首次启动的向导。根据提示，依次输入 avamar server 的主机名（即 hosts 里的那条解析）、domain 名（avproxy），后面都默认。全部完成后，在 avamar server 的 avproxy domain 里就能看到这个 proxy 了。

Proxy 要怎么用呢？还是分两步。

1. 请在 Server 的 Administration 模块里，选择 avproxy domain 下的proxy，右键点击，选择 Edit Client...，在里面把它分管的存储给选中。只有选中存储上的虚拟机才可以用这个 proxy 进行备份！
1. 在 Policy 模块中，选中需要使用这个 proxy 的 policy，点击 Edit，然后在 proxies 选项卡里勾选 proxy 名称。随后可以手动启动 policy 进行验证。如果在 Activity 模块中没有报 no proxy 错，也没有一直 waiting-client，就表示 proxy 已正确部署。

# 备份错误解决 Q & A

Q：备份 Windows Server 2008、2008 R2 或更高版本虚拟机，任务错误信息提示：Failed to connect to virtual disk ..., (13) (13) You do not have access rights to this file

A：这是因为 Avamar 版本比较低（内核低于 VDP 5.1），不支持 diskUUID。解决方法是，在虚拟机关机状态下，编辑虚拟机设置，在 VM Options 的 Advanced 部分，找到 disk.EnableUUID 参数，设置为 false。

Q：备份 Linux 虚拟机，在升级到 ESXi 5.1 或 5.5 后，会提示 vSphere Task failed (snapshot error=1236): ‘An error occurred while saving the snapshot: Failed to quiesce the virtual machine.’.

A：这个问题与上面那个其实差不多，也是因为 diskUUID 引起的，而且一般只影响 RHEL 系的 Linux。解决办法有两种：

一者，用上面的方法，将 disk.EnableUUID 参数设为 false，然后登录该虚拟机，编辑 /etc/vmware-tools/tools.conf 文件，加上：

```conf
[vmbackup]
enableSyncDriver = false
```

二者，将 vmware-tools 降级至 ESXi 5.0 对应的版本（8.6.0-425873），可以绕过该问题。