---
title: 临时解决浏览器的 Diffie-Hellman 问题
date: 2016-08-11 16:07:29
categories:
- ITM
tags:
- vPlex
- Browser
---
今天我登陆 vPlex 的管理页面时，浏览器无法打开页面。IE 报页面无法打开，而 Firefox 和 Chrome 则会详细地提示错误：

> Server has a weak ephemeral Diffie-Hellman public key

EMC 的官方答复是需要对 vPlex 做小版本升级，但其实这个问题可以临时修改浏览器参数进行绕过。以 Firefox 为例，在地址栏输入 about:config，然后找到下面两条配置项：

* security.ssl3.dhe\_rsa\_aes\_128\_sha
* security.ssl3.dhe\_rsa\_aes\_256\_sha

默认应该都是 `true`，改为 `false`，浏览器就会绕过这个错误，顺利打开页面。

不过还是要提醒一下，这样做是牺牲安全为前提的，因此，等版本升级过后，还是要尽快改回默认值。