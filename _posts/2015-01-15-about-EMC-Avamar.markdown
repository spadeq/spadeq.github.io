---
title: EMC Avamar相关问题
date: 2015-01-15 17:10:15
categories:
- ITM
- Cloud
tags:
- Avamar
- Backup
---
用EMC Avamar与VMware搭配备份虚拟机的体验是很棒的。不过Avamar的技术支持就不那么令人愉悦，比Documentum的资料还少。知识点不好关联，我想到啥就写点啥。。算是一个拼凑合集，以作备忘。
<!-- more -->

# Avamar Proxy设置

Proxy的部署没什么好说的，一个ova，用vCenter导入就行了。之后开机，打开控制台，进行设置。

网络方面也很简单，控制台里面就有一个Network Config，照着提示一步步把IP、网关、DNS、主机名都配好。回退到控制台，选择Login。Avamar Proxy的默认root口令是avam@r，注意跟Avamar Server默认的changeme不同。登进去之后，需要做两件事。

一是将Avamar Server的地址，通常是节点一的地址，写进hosts里。而且一定要注意，hosts里写的主机名，必须跟Avamar Server上配的一样，否则光是IP写对了还不行。我昨天因为重建Windows域，改掉了服务器所在的域名，结果就导致备份任务始终处于waiting-client的状态，直至超时。当然，如果DNS服务器给力的话，依靠DNS解析也是可以的。但最好还是交给本地hosts进行解析。

二是将proxy注册到server上。如果是首次启动proxy，会自动进入这个向导，我建议是取消向导，因为这时你的网络还没有完全配好，特别是主机名，在ova模板部署的时候是不会要求你提供的，这样注册到server上的全都是local的名字，很混乱。取消向导，等网络设置全部完毕后，再进行注册。

注册之前，请在avamar server上创建一个独立的domain，比如叫做avproxy，用来存放proxy。登录到proxy上，login。这时你可能会搜到教程告诉你avregister这个命令，很可惜，它已经过时了，请使用下面这条语句：

    /usr/local/avamarclient/etc/initproxyappliance.sh start

这其实就是首次启动的向导。根据提示，依次输入avamar server的主机名（即hosts里的那条解析）、domain名（avproxy），后面都默认。全部完成后，在avamar server的avproxy domain里就能看到这个proxy了。

Proxy要怎么用呢？还是分两步。

一，请在Server的Administration模块里，选择avproxy domain下的proxy，右键点击，选择Edit Client…，在里面把它分管的存储给选中。只有选中存储上的虚拟机才可以用这个proxy进行备份！

二，在Policy模块中，选中需要使用这个proxy的policy，点击Edit，然后在proxies选项卡里勾选proxy名称。随后可以手动启动policy进行验证。如果在Activity模块中没有报no proxy错，也没有一直waiting-client，就表示proxy已正确部署。

# 备份错误解决Q&A

Q：备份Windows Server 2008、2008R2或更高版本虚拟机，任务错误信息提示：Failed to connect to virtual disk …, (13) (13) You do not have access rights to this file

A：这是因为Avamar版本比较低（内核低于VDP 5.1），不支持diskUUID。解决方法是，在虚拟机关机状态下，编辑虚拟机设置，在VM Options的Advanced部分，找到disk.EnableUUID参数，设置为false。

Q：备份Linux虚拟机，在升级到ESXi 5.1或5.5后，会提示vSphere Task failed (snapshot error=1236): ‘An error occurred while saving the snapshot: Failed to quiesce the virtual machine.’.
A：这个问题与上面那个其实差不多，也是因为diskUUID引起的，而且一般只影响RHEL系的Linux。解决办法有两种。
一者，用上面的方法，将disk.EnableUUID参数设为false，然后登录该虚拟机，编辑/etc/vmware-tools/tools.conf文件，加上：

    [vmbackup]
    enableSyncDriver = false

二者，将vmware-tools降级至ESXi 5.0对应的版本（8.6.0-425873），可以绕过该问题。