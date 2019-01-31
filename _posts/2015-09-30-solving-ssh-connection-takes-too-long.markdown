---
title: 解决 SSH 登录建立连接耗时过长问题
date: 2015-09-30 12:42:43
categories:
- ITM
- Linux
tags:
- SSH
---
现在的 CentOS 7、SuSE 12 等系统，也包括以前的一些版本，在进行 SSH 登录的时候，会发现建立连接需要很长时间才能连上，而一旦连上之后速度并不慢。网上有些关于改 DNS、hosts 的方法，其实没那么复杂，只需要改动 ssh 配置文件的几个参数即可。
<!-- more -->
要改动的文件是 /etc/ssh/sshd_config。里面需要有三行设置：

``` bash sshd_config
UseDNS no
GSSAPIAuthentication no
GSSAPICleanupCredentials no
```

在你的配置文件中，如果它们是 yes，就改为 no，如果没有，就直接新增一条，被注释则取消注释。

然后重启 sshd：

* CentOS: service sshd restart
* SuSE: rcsshd restart

之后就是秒连秒开了。