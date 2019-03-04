---
title: PHP7 下无法使用 mb_strimwidth 函数的解决方案
date: 2016-05-05 12:37:36
categories: 
- Programming
- PHP
tags:
- Wordpress
---
升级到 PHP 7 后，原先国内 Wordpress 模板常用来截断文章的 mb\_strimwidth 函数就作废了，轻则乱码，重则整个页面无法显示。如果要在每个函数调用处都换成别的函数（比如 mb\_substr），工作量又太大。

解决的办法就是自己写一个 mb\_strimwidth，替换 PHP 内置的。修改主题根目录下的 function.php，添加一条函数：

``` php mb_strimwidth
function mb_strimwidth($str, $start, $width, $trimmarker) {
    $output = preg_replace('/^(?:[\x00-\x7F]|[\xC0-\xFF][\x80-\xBF]+){0,'.$start.'}((?:[\x00-\x7F]|[\xC0-\xFF][\x80-\xBF]+){0,'.$width.'}).*/s','\1',$str);
    return $output.$trimmarker;
}
```

主题就会使用这个函数进行截短。

如果服务器主机是自己可控的，那么还有更简单的办法，在 Ubuntu 下，可以安装 php-mbstring 包，这样就原生支持 mb_strimwidth 函数。