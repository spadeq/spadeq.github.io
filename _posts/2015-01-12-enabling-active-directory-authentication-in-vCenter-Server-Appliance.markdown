---
title: 在 vCenter Server Appliance 中启用 Active Directory 验证
date: 2015-01-12 17:22:38
categories:
- ITM
- Cloud
tags:
- VMware
- Active Directory
---
vCSA 看发展趋势未来是要彻底取代 Windows 版 vCenter 的。不过域账号验证这个功能是企业无论如何不可能割舍的。好在 vCSA 具有更强大的 LDAP 集成功能，能够兼容 AD 的用户验证。
<!-- more -->
在 vCSA 的配置向导（5480 端口的那个）中，有个关于 AD Authentication 的内容，如果你在那使劲折腾，就上当了。。根本就不用在那做任何设置，一切都在 Web Client 中进行。。

用 administrator@vsphere.local 用户登录 Web Client（一定要用这个用户名，别的管理员用户是看不到下面提到的菜单项的），在导航菜单中选择「系统管理」，然后点击「Single Sign-On」下的「配置」。

在中间一大栏中，点击「标识源」选项卡，然后点击绿色的加号。

弹出添加标识源对话框。选择第二项「Active Directory 作为 LDAP 服务器」，然后填表：

* 名称：随便起一个
* 用户的基本 DN：填写LDAP格式的域名，比如 `dc=spadeq,dc=com`
* 域名：填写标准格式的域名，比如 `spadeq.com`
* 组的基本 DN：可以沿用用户基本 DN。如果用户位于特定的 ou 或者 cn 下，就在此制定全名。
* 主服务器 URL：例如 `ldap://dc.spadeq.com:3268`，替换成自己的 AD 控制器地址。
* 用户名和密码，以「域\用户」的格式，填入有访问域控制器权限的用户。

在确认前，可以点击「测试连接」进行验证。

之后在各个权限管理对话框中，就可以从这个域里面选择用户了。如果想要省点事，每次都在这里选用户，可以在标识源选项卡里将该域设置为默认域。