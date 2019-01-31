---
title: 在vCenter Server Appliance中启用Active Directory验证
date: 2015-01-12 17:22:38
updated: 2017-08-14 17:28:00
categories:
- ITM
- Cloud
tags:
- VMware
- Active Directory
---
vCSA看发展趋势未来是要彻底取代Windows版vCenter的。不过域账号验证这个功能是企业无论如何不可能割舍的。好在vCSA具有更强大的LDAP集成功能，能够兼容AD的用户验证。
<!-- more -->
在vCSA的配置向导（5480端口的那个）中，有个关于AD Authentication的内容，如果你在那使劲折腾，就上当了。。根本就不用在那做任何设置，一切都在Web Client中进行。。

用administrator@vsphere.local用户登录Web Client（一定要用这个用户名，别的管理员用户是看不到下面提到的菜单项的），在导航菜单中选择【系统管理】，然后点击【Single Sign-On】下的【配置】。
在中间一大栏中，点击【标识源】选项卡，然后点击绿色的加号。

弹出添加标识源对话框。选择第二项【Active Directory作为LDAP服务器】，然后填表：

* 名称：随便起一个
* 用户的基本DN：填写LDAP格式的域名，比如`dc=spadeq,dc=com`
* 域名：填写标准格式的域名，比如`spadeq.com`
* 组的基本DN：可以沿用用户基本DN。如果用户位于特定的ou或者cn下，就在此制定全名。
* 主服务器URL：例如`ldap://dc.spadeq.com:3268`，替换成自己的AD控制器地址。
* 用户名和密码，以【域\用户】的格式，填入有访问域控制器权限的用户。

在确认前，可以点击【测试连接】进行验证。

之后在各个权限管理对话框中，就可以从这个域里面选择用户了。如果想要省点事，每次都在这里选用户，可以在标识源选项卡里将该域设置为默认域。