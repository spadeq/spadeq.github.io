---
title: Outlook 客户端连接 Domino Notes 收发邮件
date: 2016-03-10 15:21:18
categories:
- ITM
- Office
tags:
- Outlook
- Domino
---
没有开启 SMTP 和 POP3 的 Domino 服务器，理论上只有 Lotus Notes 客户端可以连接。Outlook 在 2003 版本时，由微软官方推出了一个 Connector，可兼容 Domino Notes 服务器。但从 2007 开始，微软已经停止提供这一服务。IBM 为了解决这个问题，又推出了 DAMO（Domino Access for Microsoft Outlook），以 MAPI 的形式提供互通。结果到了 09 年，IBM 也不再更新，所以新版的 Office 看起来又没法和 Domino 愉快玩耍了。但事实上，从 Office 2007 到现在，API 并没有变，DAMO 其实仍然可以用。本文以 Outlook 2016 和 Domino 8.5 连接为例，说明整个的过程。
<!-- more -->

# 软件准备

首先，你的 Office 必须是 32 位，DAMO插件只支持 32 位的 Outlook。如果你装了 64 位，请卸载，安装 32 位版。

然后，你得有 DAMO 的最新版（8.0.2 Fix 6），IBM 官方停止了下载，你得自己搜索一下，实在找不到可以发邮件找我要。

# 安装与配置

重要提示：**凡是我提到重启系统的，不要偷懒，务必要重启。还有，只要我没说启动 Outlook，也请绝对不要擅自启动，否则大概率失败。**

1. 如果之前你曾经装过 DAMO 并且配置失败，那么请务必卸载之，并将其安装目录和数据目录均删除。数据目录的可能位置是在你用户的 AppData\Local 里。之后重启一次系统。
1. 以管理员方式运行 DAMO 安装程序。在安装快结束的时候会提示你建立一个 profile。给 profile 选择一个合适的位置存放（我是放在 Documents 下，这样重装系统时方便迁移）。请在这里直接填写服务器 IP，姓名不重要。下一步就会让你选择 id 并输入密码。这都是常规的 Domino 登录，不再赘述。在点击 finish 后，请密切关注 profile 目录，必须生成 pst 和 nsf 文件，才是真正创建完成。
1. 找到 **mapisvc.inf** 文件所在的目录。（以下均以 64 位 Windows 上安装的 Office 2016 为例）
    * 对于 Click2Run 版本的 Office（比如 Office 365 用户），它的位置在 C:\Program Files (x86)\Microsoft Office\root\vfs\ProgramFilesCommonX86\System\MSMAPI\[LANG-ID]\
    * 对于 MSI 安装版的 Office（比如 Vol 光盘安装），它的位置在 C:\Program Files (x86)\Common Files\System\MSMAPI\[LANG-ID]\
    LANG-ID 视 Office 语言版本而定，简体中文为 2052，英文为 1033。
1. 在 DAMO 安装目录下，可以找到一个 nwnspR32.dll 的文件。将该文件复制到上一步所找到的那个目录里。
1. 以管理员权限开启 Outlook（这应该是目前为止第一次启动），会提示你输入 id 对应的密码，请勾选 Save this password，并等待同步结束。
1. 等同步完毕之后，重启系统。
1. 重启完成后，就可以正常使用 Outlook 啦。从现在开始启动 Outlook 无需管理员权限。

# 小技巧

## 优化体验

修改 profile 目录中的 notes.ini 文件（不是 DAMO 安装目录里的那个）。

默认 DAMO 是定时将发件箱里的邮件发出，如果需要立即发送，请在配置文件中加上一条：

    REPL_PST_FORCE_SUBMIT=1

默认 DAMO 只会同步邮箱内容，如果同时需要同步日历内容，请找到 OUTLOOK_TYPE=0 这一条，将0改成1。

## 去除90天限制

默认情况下，Outlook 只会同步 Domino 邮箱中近 90 天的邮件。对于 Outlook 2007 的用户，可以参考 [IBM官方给出的解决方案](http://www-01.ibm.com/support/docview.wss?uid=swg21171979)。然而这个加载项在 Outlook 2010 和更高版本中，因为 ribbon界面的关系是找不到的。

不过好在天无绝人之路。DAMO 的设置既然存放在 notes.ini 中，那么这个同步天数设置自然也不例外。在该文件中找到 REPLTEST_CACHE 这一项，比如：

    REPLTEST_CACHE=XXXXXX.nsf

复制这个 .nsf 的文件名，在配置文件下面添加：

    XXXXXX_Purge=365
    XXXXXX_Threshold=1
    XXXXXX_Interval=10

这里的 365 可以改成任意你需要的天数，下面两行请原文照粘。

## 和本地收件箱联动

一般 Domino 服务器都是有容量限制的，那么我们可以在本地新建一个数据文件，然后将可以归档的邮件直接拖放到本地，就等于拥有了一个无限大小的邮箱。

## 收件人自动补全

Notes 客户端一个强大的功能就是回车键自动提示补全收件人。这个功能在 Outlook 中就比较麻烦点，必须将服务器上的通讯簿中对应的联系人添加到本地。只要建立了本地联系人，那么在收件人一栏中输入人名，就可以自动提示了。否则，就必须点击收件人，然后在通讯簿中进行查找。
另外，由于字符集的关系，Outlook 中看到的用户组名可能是乱码，这个只要重新命名即可。