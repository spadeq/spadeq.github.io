---
title: 按日期清理 Linux 文件
date: 2017-08-16 10:24:20
categories:
- ITM
- Linux
tags:
- Linux
---
在运维过程中，我们经常会遇到「删除某目录下7天前的所有文件」之类的运维任务。在 Linux 中，实现这一目标仅需要一条命令即可。该命令的形式如下：

``` shell
find <path> -mtime +<N> -name "<name pattern>" -exec rm -v {} \;
```

其中：

* path 指的是待删除文件位于的路径，比如 /db_arclog
* N 指的是天数，比如 +7 表示 7 天
* name pattern 表示文件名的匹配信息，比如 *.LOG

如果不是删除而是别的命令，可以替换 -exec 后面的 rm -v。