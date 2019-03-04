---
title: 给 Twenty Sixteen 模板加上访问量
date: 2016-05-19 15:06:25
categories:
- Programming
- PHP
tags:
- Wordpress
---
Twenty Sixteen 默认是不显示访问量的，只要修改以下几个文件，就可以让这个数字展现出来。
<!-- more -->
当然，前提是要安装 WP-PostViews 插件，这个不赘述。

# 首页

依序进入 WP 后台「外观」–>「编辑」，于最右一列下拉找到 template-tags.php，点击之，在正文编辑区找到 `twentysixteen_entry_meta()` 函数，这里所列出的就是各项 meta 信息。在合适的位置，例如 `$format = get_post_format();` 上方，添加：

``` php
if ( function_exists('the_views')) {
    the_views();
}
```

此位置所展示的访问量位于文章基本信息（发帖日期、类别等）之上。

# 文章

依照前一节所述相仿，唯一不同即修改的文件换为 content-single.php。