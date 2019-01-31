---
title: Windows 远程桌面不能使用保存的凭据
date: 2015-08-03 12:47:04
categories:
- ITM
- Windows
tags:
- RDP
---
我的操作系统是新装的 Windows 10，远程操作系统是 Windows Server 2012 R2。当然，本文对于所有的 NT6 系列（Vista、Win7、Win8、Win8.1 及其对应的服务器 OS）也是适用的。
<!-- more -->
在这种配置下，所保存的远程桌面凭据，在下一次登录时会提示错误：

> Your system administrator does not allow the use of saved credentials to log on to the remote computer XXXXXXXXXX because its identity is not fully verified. Please enter new credentials.

意思就是你所保存的这个凭据，不允许使用。这就让人很不爽，每次都输入凭据，我还保存干啥？那如果要规避这个错误，改个组策略就行了。而且好消息是，这个组策略是在本机修改，而不是目标机，你只需要改一处，就可以畅通登录所有的远程桌面。

运行 gpedit.msc 打开组策略编辑器，找到 Computer Configuration（计算机管理） > Administrative Template（管理模板） > System（系统） > Credentials Delegation（凭据分配），双击右边的 Allow delegating saved credentials with NTLM-only server authentication（允许分配保存的凭据用于仅 NTLM 服务器身份验证），弹出对话框。先选择 Enabled/启用，然后点击下面的 Show，在第一行写入 `TERMSRV/*`，连续 OK 保存即可。