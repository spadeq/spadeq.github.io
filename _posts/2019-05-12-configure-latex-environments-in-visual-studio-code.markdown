---
layout: post
title: 配置 Visual Studio Code 的 LaTeX 环境
date: 2019-05-12 22:40:00
categories: 
- Documents
tags:
- LaTeX
- Visual Studio Code
---

本文介绍如何在 Windows 下使用 Visual Studio Code 搭配 MikTeX 构建 LaTeX 编辑与编译环境。

## 基本软件

最基本的当然是安装 Visual Studio Code 和 MikTeX，都使用基于用户的默认安装。

## 插件配置

在 VSC 中唯一需要安装的就是 LaTeX Workshop 插件，这是个 all-in-one 的十全大补包，包含了几乎所有需要的功能。

### LaTeX Workshop 必要配置

LW 默认是调用 latexmk 进行编译的，这与 TeXlive 配合比较好。在 MikTeX 环境下，更好的选择是调用 texify，同时兼顾中文文字，将引擎改为 xelatex。打开 VSC 的设置，在 User Settings 中搜索 recipe，然后点击 Edit in settings.json，加入下面的代码：

```json
"latex-workshop.latex.recipes": [
    {
        "name": "texify",
        "tools": [
            "texify"
        ]
    }
],
"latex-workshop.latex.tools": [
    {
        "name": "texify",
        "command": "texify",
        "args": [
            "--synctex",
            "--pdf",
            "--engine=xetex",
            "--tex-option=\"-interaction=nonstopmode\"",
            "--tex-option=\"-file-line-error\"",
            "%DOC%.tex"
        ],
        "env": {}
    }
],
```

## 辅助功能

有两项比较实用的辅助功能，一是自动格式化，二是字数统计。这两项功能在 MikTeX 发行版中都要依赖 Perl 环境。

首先安装 Perl 环境，从 [Strawberry](http://strawberryperl.com/) 下载安装包，例如 x64 msi 版本。

安装完后从开始菜单启动 Perl 命令行，执行：

```cmd
perl -MCPAN -e shell
fforce install Log::Log4perl
install Log::Dispatch::File
```

注意由于 Log4perl 包在检查的时候有 bug，而且两年多都没有修复，因此需要强强制安装（fforce）。

然后在 MikTeX 控制台安装以下两个 tex 包：

* latexindent。在编辑时，按下 Alt + Shift + F 进行代码格式化（自动缩进）。
* texcount。按下 Ctrl + P，然后输入 `>`、`count`，就可以对文字进行计数。
