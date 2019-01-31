---
title: CentOS 7部署Openfire纪实
date: 2015-02-27 16:28:15
categories:
- ITM
- Applications
tags:
- openfire
- centos
---
Openfire是比较流行的开源xmpp server，本文介绍了如何在CentOS 7下面部署Openfire服务器。
<!-- more -->

# Openfire Server安装

首先安装CentOS 7，选择Server with GUI的大类，然后右边把PostgreSQL选中，一步两步，一步两步，装好它。

请注意，因为CentOS 7已经变成纯64位的了，然而Openfire却是32位的，因此必须要安装32位的C++运行时。如果服务器能够访问外网，那么执行：

    # yum install libstdc++.i686

但是，如果服务器是内网的，就只能手动安装了。请下载下面四个rpm包：

* libstdc++.i686
* glibc
* libgcc
* libfreebl

把这四个rpm全部传到服务器上，然后执行：

    # rpm -ivh libstdc++-4.8.2-16.el7.i686.rpm \
    glibc-2.17-55.el7.i686.rpm \
    libgcc-4.8.2-16.el7.i686.rpm \
    nss-softokn-freebl-3.15.4-2.el7.i686.rpm

一定要放在一个命令里执行，不然glibc和freebl这对好基友都会装不上。

## Openfire程序安装

上传Openfire的安装程序。推荐使用`openfire-3.9.3-1.i386.rpm`这个安装包，里面已经自带对应的JRE，无需额外配置。执行：

    # rpm -ivh openfire-3.9.3-1.i386.rpm

为了后面操作的方便，请做一个准备操作：

    # cp /opt/openfire/resources/database/openfire_postgresql.sql /tmp
    # chmod 777 /tmp/openfire_postgresql.sql

注意使用下面命令确保openfire已经正常运行，且9090端口有侦听：

    # systemctl status openfire.service
    # netstat -an | grep 9090

如果openfire的status里有类似SYSV错误的提示，那么一般就是因为你没听我的话，事先安装好32位的C++运行时。

## 设置数据库

执行以下命令初始化PostgreSQL：
    # postgresql-setup initdb
    # systemctl enable postgresql.service
    # systemctl start postgresql.service

设置Linux的postgres用户口令：
    # passwd postgres

然后以该用户身份进入psql命令行：
    # su postgres
    $ psql

修改postgresql超级用户的口令。请注意，虽然这个用户也叫postgres，但它跟linux系统的postgres用户不是一回事，口令也是分开的。
    postgres=# alter user postgres with password 'password';

用\q命令退出这个命令行，创建用户和数据库：

    postgres=# \q
    $ createuser -P -D -R -e openfire
    $ createdb -O openfire openfire
    $ psql
    postgres=# alter user openfire with password 'password';
    postgres=# \q

过程中可能会有could not change directory之类的提示，不用管。退回到root下，修改/var/lib/pgsql/data/pg_hba.conf，将其中的local all all这一行最后的peer改为md5，host all all这一行最后的ident也改成md5。然后重启postgresql：
    # systemctl restart postgresql

接下来导入库表结构：
    # psql -U openfire -d openfire -f /tmp/openfire_postgresql.sql

之后会提示输入openfire用户的密码。执行完毕，用下面的命令确认创建成功：
    # psql -U openfire -d openfire
    postgres=> \l
    postgres=> \du
    postgres=> \d
    postgres=> \q

其中`\l`应该列出openfire的数据库名，\du显示openfire用户名，\d显示openfire数据库中所有表的名称。

# 配置Openfire

用服务器本机的浏览器访问<http://127.0.0.1:9090>，进入Openfire配置后台。

1. 第一步选择语言。
1. 第二步服务器设置，这里比较晦涩的就是“域”，其实指的就是机器名。建议将主机名配好并且写到DNS里，然后这里就填主机名。否则就用localhost也可以。
1. 第三步数据库设置，选择【标准数据库连接】，点击继续。在下一个页面中，数据库驱动选项请选择【PostgreSQL】，将数据库URL中方括号里面的项改为实际值，比如[host-name]改为127.0.0.1，[database-name]改为openfire。用户名密码就是之前建openfire库时创建的用户名和随后修改的密码。点击继续。
1. 第四步特性设置（左边怎么翻译成了外形设置-。-）。这里涉及到Windows AD的整合，我展开讲两句。

## Windows AD的整合

在特性设置里，如果不需要进行AD或者别的LDAP整合，选择初始设置（傻缺翻译写成了初使设置）就直接下一步了。但我要用到AD的整合，所以选择第二项，【目录服务器(LDAP)】，点继续。

1. 第一步，连接设置。Server类型选择【Active Directory】，主机填域控制器的主机名或IP，端口389。接下来的关键就是填好基础DN和管理员DN。比如说，这个域的域名叫做spadeq.com，所有的用户放在叫做“员工”的OU里面，那么基础DN就是【OU=员工,DC=spadeq,DC=com】。下面的管理员DN，如果用域管理员，就写【cn=Administrator,cn=Users,DC=spadeq,DC=com】。特别注意这里的cn=Users，很多人就卡在这里了。
1. 第二步，用户映射图。这里需要指定所谓的用户名和信息。这里最关键的信息就是用户名域、用户名和用户名全称。建议都保留默认值。
1. 第三步，用户组。这里也保留默认值，下一步。

最后要指定管理员。输入管理员的用户名，也就是AD里面的sAMAccount名。如果能添加进来，就没问题，找不到的用户是加不进来的。
再点继续，就结束整个安装向导了，可以登录管理控制台。

# Spark客户端

用了一圈下来发现，还是Spark最好用，别的非专门配合Openfire的IM客户端都或多或少存在一些问题，或者操作不方便。Spark登录时需要提供用户名、口令和Openfire服务器地址或主机名，唯一要注意的就是用户名，应该是前面AD整合时的“用户名域”，默认是sAMAccount。