---
layout: post
title: 配置 Jekyll 与 Github Pages
date: 2019-01-31 20:49:00
categories: 
- Article
tags:
- Github Pages
- Jekyll
- Ruby
---
本 blog 以前通过 Hexo 引擎生成，说实话不太好用，所以更新也很不及时（哈哈哈，借口）。经过一番奋战，终于顺利切换到 Github Pages 原生的 Jekyll 引擎。目前网上搜到的教程，包括 Gitpages 和 Jekyll 官方的教程，都令人不满意。所以在这记下来各种排坑的过程。

<!-- more -->

# Ruby 环境

Jekyll 基于 Ruby，所以我使用的是 Ubuntu on Windows。执行：

```shell
sudo apt install ruby ruby-dev zlib1g-dev
sudo gem install bundler
```

# 创建 Github Pages

登录进入 Github，创建一个新的公开 Repo，名称必须是 `<name>.github.io` 的形式，比如本站的 `spadeq.github.io`。gitignore 选择 Jekyll。

# Jekyll 本地 repo

首先初始化一个 Jekyll 站点，注意将目录和 URL 中的相关名称替换成自己的站点名：

```shell
sudo gem install jekyll
mkdir spadeq.github.io
cd spadeq.github.io
git init
git remote add origin https://github.com/spadeq/spadeq.github.com
git add .
git pull
jekyll new temp
mv temp/* .
rm -rf temp
git commit -m "initial"
git push -u origin master
```

之所以先 pull，是为了防止直接创建 jekyll 站点后首次 push 不成功。

