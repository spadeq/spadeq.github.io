---
title: 使用 SSH 公钥登陆 Linux 服务器
date: 2017-08-21 15:23:55
updated: 2017-08-21 15:23:55
categories:
- ITM
- Linux
tags:
- SSH
- RSA
---
Linux 服务器的 SSH 是个好东西，长久以来管理员习惯了使用「用户名 + 密码」的形式进行远程登录。然而为了更加安全的管理服务器，现在更建议的方式是采用公钥。本文介绍如何改用公钥管理 Linux 服务器的 SSH 登陆。
<!-- more -->

# 公钥登陆原理

这里我们使用的工具是 Xmanager，而其它的 SSH 工具比如 putty，或者 Linux 自己的命令行，其原理是相同的。

首先，在客户端，也就是发起远程访问的计算机上，生成一对 RSA（或 DSA）的公钥与私钥对。私钥自己保存，然后将公钥以文件或者文本的形式交给受管服务器。当客户端发起远程访问时，服务器会进行验证，如果公私钥匹配，则允许登陆。

单一客户端的公钥，可以发放给多个服务器，这样，客户端只需要一个或少数几个私钥，就可以管理很多的服务器。

# 配置方式

## 客户端

### Xmanager

1. 选择 Tools -> New User Key Wizard
1. Key Type 选择 RSA，长度可以选择 2048 或更高，下一步，下一步
1. 为 key 取个名字，可以是任意的名称而且以后还可以改，并不重要。Passphrase 是一个密码，访问私钥的密码，可以防止私钥泄露被人滥用。而你每次用私钥的时候，都要输入这个短语。如果确信私钥安全，也可以留空。下一步
1. 这里文本框里显示的，就是公钥的文本形式。点击 Save as a file，可将其保存为一个 .pub 后缀的文件（其实文件内容也就是这串文本。Public Key Format 请选择「SSH2 - OpenSSH」，供我们后面使用。
1. 如果需要备份这个私钥，可以选择 Tools -> User Key Manager，选择刚生成的私钥，然后点击 Export 导出。**请一定要妥善保管私钥和密码短语！**

### Linux

执行以下命令：

``` shell
ssh-keygen -t rsa -b 2048
```

然后会提示 passphrase。

这个时候在 ~/.ssh 目录下，会生成 id\_rsa 和 id\_rsa.pub 两个文件，前者是私钥，后者是公钥。.pub文件里的内容就是公钥字符串。如果要备份私钥，就将 id\_rsa 妥善保存。

## 服务器端

以 SuSE Linux 为例。

1. 登录服务器，就以远程登陆所使用的用户来登录，尽量不要直接用 root
1. 首先查看用户目录下有没有 ~/.ssh 这个目录，如果没有就创建一个，权限为 700（`chmod 700 .ssh`）
1. 进入 .ssh 目录，创建一个文件叫做 authorized\_keys，然后将刚才生成的公钥字符串粘贴进去。也可以将之前生成的 .pub 文件上传到此处，然后改名为 authorized_keys。注意，如果已经存在这个文件，并且里面已经有内容了，就在后面加一个空行，然后把自己的公钥字符串加在后面。这也是允许多客户端登录的方法。
1. 将 authorized\_keys 的权限设为 600（`chmod 600 authorized_keys`）
1. 修改 /etc/ssh/sshd\_config 文件，注意修改下面几项（取消注释、更改值或者增加新行）
    * `PasswordAuthentication no`
    * `PermitEmptyPasswords no`
    * `ChallengeResponseAuthentication no` 禁用 Keyboard Interactive 形式的用户密码登录
    * `PubkeyAuthentication yes` 这一条默认是注释掉的，即便不写 yes，在 SuSE 下默认也是开启的
    * `UsePAM no` 可改可不改，因为上面已经禁用了密码登录，这里就算是 yes，也不能密码登录
1. 执行 `rcsshd restart`，重启 ssh 服务，并试着用用户名口令进行登录，看是否被强制断开。

# 登录操作

1. 在 Xmanager 的 Xshell 项目里，右键点击然后选择 Properties，
1. Authentication 一节，将 Method 改为 Public Key，输入远程登录用户名
1. 在 User Key 里面选择刚使用的私钥，然后输入 Passphrase
1. OK，双击登录看是否正确

如果是 Linux 的 shell，那么直接执行 ssh 命令即可。