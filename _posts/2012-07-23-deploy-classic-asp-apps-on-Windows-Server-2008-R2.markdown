---
title: Windows Server 2008 R2下部署经典ASP程序
date: 2012-07-23 16:34:49
updated: 2017-08-15 16:36:00
categories:
- Programming
tags:
- ASP
- Windows
---
单位里有个十年前部署的老程序，Windows Server 2000 + SQL Server 2000 + ASP的架构。现在要把程序重新部署在全新的环境下，Windows Server 2008 R2 + SQL Server 2012，折腾了许久终于搞定，拿出来分享。
<!-- more -->

# 系统环境安装

Windows安装不说了，需要添加Application Server和Web Server(IIS)两个角色，注意Application Development部分要把ASP选中，否则只能hold住ASP.NET了。ASP和ASP.NET不一样，你懂的。

SQL Server 2012安装只要注意开启混合登陆模式。经典ASP不支持Windows验证，只能用传统的用户名密码方式。

# IIS配置

## 添加应用程序池

1. 新建一个应用程序池，.NET Framework version选择No Managed Code，mode选Classic，勾选立即启动，确定。
1. 之后右键点击之，选择高级设置
1. 将启用32位应用程序设为True：

## 添加网站

1. 在Web Site中添加Application。注意将应用程序池设为我们刚添加的那一个。
1. 添加完毕之后，在Features View里的IIS板块找到ASP，双击之。
1. 将其中的“启用父路径”设置为True。
1. 在Features View的Default Document中可以修改默认文档，请按实际情况进行设置。

## 迁移数据库

SQL Server的迁移绝对是所有DBMS中最简单方便的，只需要在原数据库管理器中做一次detach，将mdf和ldf两个文件拷到新服务器上重新attach就可以了。但实际上从SQL2000到2012跨度太大，可能会出现attach失败的情况。我经过实践发现了一个必杀解决方案，那就是先把从2000分离出来的文件用SQL Server 2005 attach一次，再detach掉，然后再用2012进行attach -。-

OK，到这里，整个部署宣告完毕。

# 启用详细错误信息

迁移过来的网站，很难保证百分之百能够无错运行，可能还需要对代码进行一些微调，那么错误信息就非常重要了。但是默认情况下，IIS7上的ASP只会显示“500 – 内部服务器错误”，那么我们就需要再进行一番设置，让详细信息显示出来。

还记得站点的Features View吧，还是双击那个ASP。这一次，是将Compliant分支中的Send Errors to Browser设为True。
接下来还是在Features View，这次双击Error Pages，选中“详细错误”，并确认。

对于使用IE的用户，还需要在IE选项中把“显示友好HTTP错误信息” 给去掉。OK，去发现错误解决错误吧~~