---
title: 警惕利用Windows远程桌面传播的3389蠕虫
date: 2015-01-08 11:14:23
updated: 2017-08-16 10:12:00
categories:
- ITM
- Security
tags:
- Windows
- RDP
---
安装使用Windows Installer封装的软件时，总是提示类似“另一个程序要求重新启动此服务器”的提示。一般导致该问题发生的原因是，在注册表的`HKLM\System\CurrentControlSet\Control\Session Manager`子项里，存在PendingFileRenameOperations项，将其删除即可正常进行安装。但此次处理问题时，该注册表项在删除后还会不断自动重新产生，遂起疑。

# 情况分析

不断产生的PendingFileRenameOperations的内容类似于：

    \??\c:\windows\offline web pages\cache.txt
    !\??\c:\windows\system32\sens32.dll

经检查c:\windows\system32下确实存在sens32.dll文件，其特征为：

> File: C:\WINDOWS\system32\sens32.dll
Size: 37376 bytes
File Version: 5.2.3790.3959 (srv03_sp2_rtm.070216-1710)
Modified: 2007年2月17日, 6:43:54
MD5: 38AF469884AC6D461C46DDAD2235225B
SHA1: BFC62E35309EE732E07E9411AACF313FCC18A86D
CRC32: 2053062B

网络上有人发现该文件有其它变种，MD5可能不同。请注意，该注册表项的重要特点是删除后会自动重建。

# 问题定性

该蠕虫被F-Secure命名为Morto（Backdoor:W32/Morto.A and Worm:W32/Morto.B）。它通过Windows的RDP协议扫描局域网内所有服务器，穷举Administrator用户的口令，字典包括但不限于以下内容：

> admin
password
server
test
user
pass
letmein
1234qwer
1q2w3e
1qaz2wsx
aaa
abc123
abcd1234
admin123
111
123
369
1111
12345
111111
123123
123321
123456
654321
666666
888888
1234567
12345678
123456789
1234567890

凡是开启远程桌面且Administrator口令在其字典中的服务器，均有可能被蠕虫感染。

# 解决方法

一般来说具有防病毒软件的机器，基本对此蠕虫免疫，因为毕竟这是几年前流行的了。但如果服务器未安装防病毒软件，或者装的杀软不靠谱（比如SEP 11），那就不好说了。如果查看服务器注册表发现该蠕虫行踪，请务必首先对服务器做快照（虚拟机）或者手动备份重要数据（物理机），然后进行查杀。对于能够连接互联网的服务器，可以安装靠谱的杀软（比如MSE）扫全盘；内网机器，可以使用360或金山的急救箱。但请注意，查杀该病毒有可能会导致服务器崩溃（比如重启后win32k.sys蓝屏等），所以一定要事先做好备份。