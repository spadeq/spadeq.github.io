---
layout: post
title: 使用 Windows Server 提供 SFTP 服务
date: 2021-10-15 14:31:00
categories:
  - IT
tags:
  - SFTP
  - SSH
  - Windows
---

本文介绍如何在 Windows Server 2022 上开启 SFTP 服务，并进行一些必要的配置。

## 安装 OpenSSH Server

不建议安装任何第三方 SFTP 或 OpenSSH 软件，都存在问题。

能够访问互联网的情况下，进 PowerShell 执行：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

不能够访问互联网，则需要先[下载 Windows FOD 镜像](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)，从镜像中提取 OpenSSH 安装包，例如 `OpenSSH-Server-Package~31bf3856ad364e35~amd64~~.cab`，上传到内网 Windows Server 上，然后在同一目录下执行：

```powershell
dism /online /add-package /packagepath:.
```

如果执行命令和安装包在不同目录，需要在 `/packagepath` 参数中指明绝对路径。

安装完成后，将其设为自动启动：

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
```

## 创建 SFTP 用户

SFTP 用户通过 Windows 自身的用户来管理，就是个普通的 Windows 用户。对于 Windows Server 2022，可右键点击开始按钮，选择 Computer Manager、Local Users and Groups。创建用户的时候注意取消选择首次强制修改密码，勾选用户不能修改密码。

## 设置权限和文件目录

默认情况下创建的用户还可以通过 SSH 登录服务器，而且主目录是它的 Windows 用户目录。但对于一个 SFTP 服务来说，我们不希望它还能登录 SSH，可能还要将它的主目录设置为一个专门的 FTP 目录。这些设置都在 `%programdata%\ssh\sshd_config` 文件中配置。

```text
Match User foo
    ChrootDirectory d:/bar
    ForceCommand internal-sftp
```

上面的配置就是将 foo 用户的主目录设为 D:\bar，并且它只能使用 SFTP 服务，不能通过 SSH 登录服务器。注意尽量不要在 C 盘设置主目录，避免出现用户权限的麻烦问题。

修改配置后，需要重启 SSH 服务：

```powershell
Restart-Service sshd
```
