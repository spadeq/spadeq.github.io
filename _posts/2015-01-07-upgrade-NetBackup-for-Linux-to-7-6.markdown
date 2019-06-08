---
title: Netbackup for Linux升级到7.6
date: 2015-01-07 11:18:26
categories:
- ITM
tags:
- NetBackup
- Backup
---
1. 停用所有备份策略
1. 修改 /etc/security/limits.conf，加入两行：
        *            hard nofile             8000
        *            soft nofile             8000
1. 修改/etc/sysctl.conf，加一行：
        kernel.sem = 300        307200  32      1024
1. 执行 `sysctl -p`，然后再执行 `sysctl -a | grep kernel.sem`，看是不是刚才加上的值
1. 重启。之后用 `ulimit -n`，看一下是不是8000
1. 备份出 /var/NBU_DRFILE 的最后几个，以及 Media Server 上的 /usr/openv/scripts
1. `kill_all`。完了之后用 `bpps -a` 看一下是不是都停了。
1. 修改 /usr/openv/var/global/server.conf，将 `-ch 512M` 改为 `-ch 1G`
1. `tar -zxvf NetBackup_7.6.0.1_LinuxS_x86_64.tar.gz`
1. `./install -s`。到了是否添加附加 license 的时候，选 no，其它全部保留默认值。
1. 启动 `jnbSA &`，看是否正常。
1. 在同一目录解压 NB\_7.6.0.4.linuxS\_x86.tar和NB\_CLT\_7.6.0.4.tar
1. `./NB_update.install -s`，然后输入 NB_7.6.0.4，回车。后面一路默认。
1. 最后显示两个带 * 的产品名，即为成功。
