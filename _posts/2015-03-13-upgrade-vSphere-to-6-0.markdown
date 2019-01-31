---
title: 升级 vSphere 到 6.0
date: 2015-03-13 14:33:53
categories:
- ITM
- Cloud
tags:
- VMware
---
3.12 守了一整天，终于守到了 vSphere 6.0 的正式发布！那么今天当然第一件事情就是做升级了。下面分享一下升级过程中的一点经验。
<!-- more -->

# vCenter Server Appliance

我用的是 vCSA 架构，首先应该升级 vCSA。这次的 vCSA 没有 ova 格式了，只有一个 iso，而且这个 iso 是在本地执行的，算是一个大创新。

挂载 iso 之后，先别忙着打开根目录下那个 html 文件，先进入 vcsa 目录，把 VMware-ClientIntegrationPlugin-6.0.0.exe 安装一下，这样浏览器才能正确执行后面的步骤。

装好插件，打开根目录的 vcsa-setup.html，选择升级，然后一步步按照提示来。在第4步，【连接到源设备】中，可能会遇到一个错误提示：

> vCenterServer FQDN xxxx does not match DNS servers "localhost.localdom, localhost" and ip addresses "" from VC certifacate
> Examine the VC certificate and make sure it is valid and point to vCenterServer FQDN.

这可能是因为最初部署 vCSA 时，没有事先更改主机名造成的。解决方案一：

1. 登录 <https://[vcenter-ip]:5480>
1. 在 Admin 选项卡里，将 Certificate regeneration enabled 设为 Yes
1. 重启 vCSA
1. 重启完成后，再进入 Admin 选项卡，将那个选项重新设成 No（不然每次重启都会重新生成证书）

解决方案二（能登陆 vCSA 的时候推荐这样做）：

1. 打开 vCSA 控制台或 SSH 登录之
1. `touch /etc/vmware-vpx/ssl/allow_regeneration`
1. `echo only-once > /etc/vmware-vpx/ssl/allow_regeneration`
1. 重启 vCSA

这时候再回到安装页面，就不会报错了。

还有一种可能发生的错误，cannot upload upgraderunner via ssh tunnel，这是因为没有开启旧 vCSA 的 SSH 登录。也是在 5480 的 Admin 页面，将【Administrator SSH login enabled】设为 Yes 即可。一路继续，直到升级完成。

# vSphere Update Manager

为了后续的升级，先要升级 Update Manager。特别是上一步骤中重新生成了 ssl 证书的同学，um 更是必须重新配置。其实我建议彻底重新安装一个 um，因为既然升到 6，以前积累的补丁都没用了，没有保留的价值。如果真要升级，就把 vCenter for Windows 的那张 iso 挂载到 um 服务器上，选择 vSphere Update Manager 服务器，并且点击安装。安装程序会自动检测当前已安装过的 um，并进行升级。升级结束后重新打开 vSphere Client，并且在插件管理器中手动安装 vSphere Update Manager 扩展。

升级之后，之前导入过的修补补丁全部还可以用，但 5.5 的 umds 导出的目录是不能用的，需要在外网用新版的 umds 重新下载。

# ESXi

升级 ESXi 就需要用到前面的 Update Manager。如果不想用 um，可以把 iso 直接刻盘去服务器引导重装。我还是采用 Update Manager 的方式。

首先在 Update Manager 的管理界面，选择「ESXi 映像」模块，导入 6.0 的 ISO，官方原版或者 OEM 定制版都可以。

导入的过程中，会提示你创建基准，可以为每个 ISO 创建一个不同名称的基准。去主机或者集群，附加该基准，然后就可以应用了。等着升级完毕吧。

# vSphere Data Protection

这次一大利好就是 VDP 不再有 Advanced 的单独授权，这样一来等于可以节省一大笔费用，享受更高级的 VDP 功能。VDP 升级比较复杂一点，得展开说一说。

## 升级前的准备

1. 进入 Web Client，在主机与集群清单里，关掉 VDP 的虚拟机。请注意，一定要用「关闭客户机操作系统」的菜单指令，而不能用「关闭电源」，否则很可能造成 appliance 损坏；
1. 编辑 VDP 虚拟机设置，把从硬盘 2 开始的所有硬盘的模式，改成「从属」（原来应该是「独立-持久」）；
1. 给 VDP 虚拟机做一个快照；
1. 把 VDP 6.0 的 ISO 传到存储上，一定要放在存储上，如果用远程文件系统，会慢的让你无法忍受。

## 开始升级 appliance

1. 启动 VDP 虚拟机；
1. 将 VDP 6.0 的 ISO 挂载给 VDP 虚拟机；
1. 打开 <https://[vdp-ip]:8543/vdp-configure/>，输入用户名密码登录；
1. 切换到【升级】选项卡，等待检查结束，应该会显示 VSphereDataProtection7181；
1. 选中它，点击【升级 VDP】按钮，等待进度走完；
1. 完成之后，VDP 虚拟机会自动关机。

## 升级的后续

现在 appliance 已经升级完毕，但我们还需要做一些后续工作，先别急着启动 VDP 虚拟机。

1. 在 vCenter 清单中，把之前给 VDP 虚拟机做的快照删掉；
1. 编辑该虚拟机的设置，把从硬盘 2 开始的所有硬盘模式改回【独立-持久】；
1. 把 CD/DVD 改成主机设备；
1. 打开虚拟机，等待启动完成（出现蓝色屏幕）；
1. 重新登陆 Web Client，你可以看到左边多出了 vSphere Data Protection 6.0。点击它，选中 VDP 设备，点击连接；
1. 在【配置】一栏中，点击右边的齿轮，选择完整性检查。这一步虽然不是必须，但为了以后备份的正确，最好是做一次。