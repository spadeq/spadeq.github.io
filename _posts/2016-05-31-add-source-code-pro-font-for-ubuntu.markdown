---
title: 为 Ubuntu 增加 Source Code Pro 字体
date: 2016-05-31 16:20:05
categories:
- ITM
- Linux
tags:
- Linux
- Ubuntu
- 字体
---
本文其实介绍的不仅仅是安装这一款字体，也是对 Ubuntu 的字体管理做一个总体的介绍。

<!-- more -->

# 字体存放

本文主要关注 TTF、OTF 这一类的字体。在 Ubuntu 中，字体分为系统全局字体和用户独有字体这两类。

系统级的字体，存放地点为 /usr/share/fonts。如果在下面还有目录结构，其意义仅仅是便于管理和分类，易于查看，并不会对字体的引用带来什么影响。一个 ttf 文件扔进去，无论放在怎样的目录下，最终用起来是一样的。

用户级的字体，存放地点为 ~/.local/share/fonts，原理和系统级是相同的

# 安装字体

安装字体主要分为两个步骤。

1. 将字体文件根据需要（全局还是用户）复制到上面提到的目录中。为了便于管理，可建立目录。
1. 执行下面的语句，刷新字体缓存。 执行这条语句并不需要 root 权限。

        fc-cache -f -v

1. 确认安装完成。使用系统自带的 Font Viewer，看其中是否存在新增的字体。（这个破程序还不支持搜索，只能拖滚动条人工搜索）。

# 安装 Source Code Pro 字体

该字体可通过 git 下载：

        git clone --depth 1 --branch release https://github.com/adobe-fonts/source-code-pro.git ~/Downloads/scp

随后就是按照上一节的方式进行安装

        mkdir ~/.local/share/fonts/scp
        cp ~/Downloads/scp/TTF/*.ttf ~/.local/share/fonts/scp
        fc-cache -f -v

P.S. Visual Studio Code 真心好用，而且这一基于 Atom 的文本编辑器的意义不仅仅是为我们带来了一款很棒的跨平台码砖神器，而且其 io.js + Chromium 的架构为客户端 app 开发起到了很好的示范作用。