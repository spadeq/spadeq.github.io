---
layout: default
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
sudo apt install ruby
sudo apt install build-essentials ruby-dev zlib1g-dev
sudo gem install bundler
sudo gem install jekyll
```

特别注意要安装 build-essentials 和两个 dev 包，不然后面 gem 和 bundle 命令会各种出错。

# 创建 Github Pages

登录进入 Github，创建一个新的**公开** Repo，名称必须是 `<name>.github.io` 的形式，比如本站的 `spadeq.github.io`。gitignore 选择 Jekyll，协议自定。

# Jekyll 站点初始化

## 创建 repo

首先初始化一个 Jekyll 站点，注意将目录和 URL 中的相关名称替换成自己的站点名：

```shell
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

之所以先 pull，是因为先创建 jekyll 站点后 git init 可能会导致首次 push 不成功。Jekyll 不能在一个已经存在的目录中 new 站点，所以就先 new 到一个临时目录中，然后再移动到 git 根目录。

## 修改编译环境

修改 Gemfile，将其中的 `gem "jekyll", "~> 3.x.x"` 一行注释掉，增加一行：

```ruby
gem 'github-pages', group: :jekyll_plugins
```

然后执行：

```shell
bundle update
```

注意过程中可能会提示输入用户口令，因为个别依赖包需要安装到系统目录中。

# 编写内容

## 基本配置

站点的配置保存在 `_config.yml` 文件中，该文件的编写可以参考 Jekyll 的官方教程。

本地预览命令如下：

```shell
bundle exec jekyll serve
```

在浏览器中访问 `http://127.0.0.1:4000` 可以在本地查看站点。push 到 git 上只需要几秒钟就会重新生成新的网页。这比 hexo 节省了编译和发布的步骤。而且站点一旦初始化完成，如果不需要本地预览的话，都不需要安装 ruby 环境，只要一个纯文本编辑工具。甚至还可以直接在 Github 网页上写 blog。

## 主题

默认的 Jeykll 站点模板是使用 home、page、post 类型来组织站点的，而 Github Pages 推荐的几个主题，却只有 default 类型，而且没有常用的文章列表首页，所以建议不要直接用。。

# 自定义域名

如果要用自己的域名代替 name.github.io 的域名（比如本站使用 blog.spadeq.com），可以使用 Github Pages 提供的自定义域名功能。

前提当然是要在域名解析服务商那里配置 CNAME，不详说。国内建议使用 DNSPod 的免费服务。

在 Github 的 repo 设置中，找到 Github Pages，在 Custom domain 一栏中填入域名。

此外，还要在 Jekyll 站点根目录新建一个 `CNAME` 文件，内容也是域名。提交后，.github.io 的域名会自动重定向到自定义域名上。

# 从 Hexo 迁移

迁移过程其实非常简单，Jekyll 和 Hexo 都可以使用 markdown 来写文档，基本标记格式也相同。

首先，将 hexo 版 blog 目录下 `source/_posts` 中的所有文件复制到 jekyll 版 blog 目录的 `_posts` 中。然后在 `_posts` 目录下执行以下命令，修改文件后缀：

```shell
find ./ -name "*.md" | awk -F "." '{print $2}' | xargs -i -t mv ./{}.md ./{}.markdown
```

不需要修改文章的任何内容，即可平滑迁移。