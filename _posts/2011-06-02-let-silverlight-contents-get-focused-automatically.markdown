---
title: Silverlight内容自动获得焦点
date: 2011-06-02 16:44:46
categories:
- Programming
- .NET
tags:
- Silverlight
---
我们有时候希望一个网页刚打开的时候，里面的silverlight控件就自动获取焦点，比如登录窗口。这只需要在调用页面增加几行代码就可以了。

首先在onSilverlightError下面添加一个JavaScript函数：

``` javascript
function appLoad()
{
    var xamlObject = document.getElementById("silverlightControl");
    if (xamlObject != null)
        xamlObject.focus();
}
```

然后在页面上调用Silverlight的object标签里加上这个id：

``` html
id="silverlightControl"
```

最后，在object标签内部加一个参数：

``` html
<param name="onLoad" value="appLoad">
```

这样就完成了。