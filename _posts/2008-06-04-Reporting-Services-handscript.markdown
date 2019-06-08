---
title: Reporting Services Handscript
date: 2008-06-04 16:53:11
categories:
- Programming
- SQL
tags:
- Reporting Service
- SQL Server
---
SQL Server Reporting Services（简称 SSRS），是一个功能强大的报表服务。在 MSDN 的相关文档以及 Google 的结果中，可以找到它的一般性描述。在当前 SQL Server 2008 还在酝酿，2000 基本过时的情况下，我们理所当然地讨论其 2005 版本。

可惜的是，对这项功能的应用，目前似乎仍不广泛，以至于想找一本非常全面而又充满技巧性的教程都如此困难。但好消息是，这是微软开发的产品！秉承微软产品一贯的易上手、帮助翔实的优良传统，对 SSRS 的探索还是相当值得大书特书的。
<!-- more -->

# 起

这一部分主要说说关于 SSRS 的安装、部署和配置问题。

## SSRS 的安装

Reporting Services 作为 SQL Server 的一个组件，自然是要伴随 SQL Server 一起安装了。目前 SQL Server 的诸多版本，只有Enterprise 和 Development 版本有着对 SSRS 的完全支持，Standard 版本提供了大部分支持，具有高级功能的 Express 版只支持一些最基本的功能（不含设计器）。因此，在企业部署的时候应该选用 Enterprise 版，作为开发者应选择 Development 版。

在安装 SQL Server 的过程中选中 Reporting Services 的相关组件，或者更改一个 SQL Server 的安装以添加 SSRS 都是可行的。

注意上面说的是服务器端的安装。SQL Server 2005 Development Edition 的组件分为服务器和工作站两个部分。在安装完服务器端的相关组件后，还需要在进行开发的机器（可以就在服务器上，也可以是另外的工作站）安装工作站组件。其中的 Business Intelligence Development Studio 必须安装，这是一个 Visual Studio 2005 的扩展，如果机器已经安装过 VS2005，那它就会直接将BI的开发模板集成进 VS 中，如果没有装过，那它则会自动替你安装一个 VS2005 外壳（没有 C#、VWD 等组件）。

## SSRS 在服务器端的配置

服务器端配置 SSRS 有两种方法，一是通过 SQL Server Management Studio，登录到 Server 进行操作；二是通过 web 访问服务器的 Report Manage 页面，比如 <http://IP/Reports>。两种方法在功能上略有差别，具体操作过程可以查看相关文档。

# 承

这一部分来讨论一下 SSRS 的一些基本功能，即报表的建立、发布和引用。

## 创建报表

### 设计环境

报表设计环境就是那个 Business Intelligence Development Studio，以下简称 BI。如果项目是在 VS2005 下进行的，那么就非常方便了，因为可以在一个 Solution 里像添加普通 Project 一样来添加 BI 的工程。事实上我们也是这么做的。

### 报表创建的基本步骤

在正式开始利用 BI 开发 SSRS 之前，强烈建议大家把随机附带的 Book Online 里的相关教程全部手动完成一遍。

总的来说，一个报表的设计可以归纳为下面的步骤：

1. 创建报表工程；
1. 建立数据源（Data Source，rds 文件），这是报表与数据库进行通信的桥梁；
1. 创建数据集，也就是报表的数据来源，报表从数据集获取数据，并不直接访问数据库；
1. 进行页面布局（layout）；
1. 预览结果（preview），并根据结果进行进一步修改，直到完全满足要求。

在我另一篇日志里有一个 [简单的例子](http://ustc.blog.hexun.com/18707734_d.html)，来源于 SQL Book Online，可以作为一个参考。

## 报表的发布

设计完报表之后，就要将其发布到 Reporting Services 服务中，以供调用。我们可以把这个发布称作 deploy。

经过实际应用，发现可以有下面三种方法来进行报表的发布

### 在 BI 中编译发布

我们设计报表是在 BI 中进行的，可以利用它来一次性将整个报表工程 deploy 到服务器上。大致步骤如下：

1. 菜单执行 Project -> Properties，将 Configuration 改为 Production，即编辑 Production 模式的参数；
1. 在右边分别填入相应的属性值，一般来说 TargetDataSourceFolder 的内容 Data Sources 不变，如果数据源有更新，那么就必须将上面的 OverwriteDataSources 设为 True；
1. 设置 TargetReportFolder。这个值是在 Report Server 中的一个虚拟目录，该工程的所有 rdl 文件都将存放在这个目录下；
1. 设置 TargetServerUrl。这里就是 Reporting Services 所在的 URL 地址，比如本地部署可以用 <http://localhost/ReportServer>。注意后面的那个路径是默认的安装路径，在 IIS 中打开默认站点后可以看到它，是一个虚拟目录；
1. 都填写完毕之后，在编译环境中切换编译模式为 Deploy，再 Start Debugging，这时 BI 就会自动向 Report Server 部署这一系列的报表。

完了之后会显示 <http://localhost/ReportServer> 这个页面，在这个页面中显示的就是该报表服务器上所有的 ReportFolder，而报表则会按照 deploy 时的设置，分别保存在这些 folder 内。进入 folder 之后，点击报表即可查看，系统已经为我们生成了一个带有 ReportViewer 的 aspx 页面。

### 通过 Web 下的 Report Manager

下面这两种方法均是用来管理报表服务器，发布报表只是它们的一部分功能。

使用 Report Manager 的大致步骤如下：

1. 打开 Report Manager 的页面，一般为 <http://ServerUrl/Reports>；
1. 进入 Data Sources 文件夹，上传数据源的 rds 文件；
1. 回到根文件夹，建立一个 ReportFolder，名称即 TargetReportFolder 中的值；
1. 进入该文件夹，把 rdl 文件逐一上传，它会自动给报表起名，一般接受默认值。

这样就 OK 了，之后也可以在 ReportServer 页面下查看内容。

### 通过 SQL Server Management Studio

在 SQL Server 的配置中，这个工具无疑是最强大的。在登录 SSMS 的时候，选择 Server Type 为 Reporting Services，然后指定 Server 的名称，以及登录方式。登录成功后，在 Home 目录下就是我们在 Report Manager 中看到的内容，后面的操作大同小异，就不浪费文字了。

## 利用 ReportViewer 控件引用报表

建立、发布报表的最终目的还是为了在程序中引用它们，在此我们选择的是最简单的方法——使用 ReportViewer 控件。

### WinForm 环境下的 ReportViewer

WinForm 下的 ReportViewer 控件，位于 Microsoft.Reporting.WinForms 命名空间下，在 VS2005 中默认会出现在 ToolBar 中，直接将其拖放进窗体即可对其操作。

一般来说，所有报表都必须设置的参数有以下几个：

* ProcessingMode：这个属性用来设置 ReportViewer 的数据来源是本地还是远程，在这里我们设为 Remote；
* ServerReport.ReportServerUrl：就是我们前面看到的 TargetServerUrl，即报表服务器的 URL 地址。注意这个地址包含了「ReportServer」，比如 <http://ServerUrl/ReportServer> 这样；
* ServerReport.ReportPath：是 ReportFolder 和 ReportName 的组合，比如「/Test/Report1.rdl」，注意注意千万注意，最开始的那个「/」一定不能省略！
* 对于实际应用，采用代码来控制 ReportViewer 要比设计时设置属性更加常用，下面就是一个简短的例子，概括了这样一个过程：

```C#
this.reportViewer1.ServerReport.ReportPath = "/Test/Report1";
List<ReportParameter> parameters = new List<ReportParameter>();
parameters.Add(new ReportParameter("params",textQueryString.Text));
this.reportViewer1.ServerReport.SetParameters(parameters);
this.reportViewer1.ShowParameterPrompts = false;
this.reportViewer1.RefreshReport();
```

在上面的过程中，先是设置 ReportPath（ReportServerUrl 在本示例中已经指定，实际上应该通过 App.config 的设置字符串来设置）。然后创建报表参数列表（这个 params 的名称是在设计报表的时候设置的报表参数，在 SQL 语句中通过 @params 进行引用），进而调用 ServerReport 的 SetParameters 方法，将参数传递给报表。紧接着，将报表的 ShowParameterPrompts 属性设为 false，即不在 ReportViewer 的头部显示参数输入提示。最后执行 RefreshReport() 方法，刷新报表页面。

### ASP.NET 环境下的 ReportViewer

微软的统一性工作无疑是相当出色的，Web 下的 ReportViewer 在使用起来与 WinForm 下完全相同，唯一不同的就是控件位于 Microsoft.Reporting.WebForms 下，而诸如 ReportParameter 等类也改为此命名空间之下。在代码控制报表方面，不需要进行改动即可移植。

### Visual WebGUI 下的 ReportViewer

在项目中，我们是采用 VWG 来作为程序的框架的。Gizmox 开发团队也为 ReportViewer 设计了相应版本，控件位于 Gizmox.WebGUI.Reporting 命名空间下，但要注意，它的属性诸如 ReportParameter、ProcessingMode 等仍然是位于 Microsoft.Reporting.WebForms 下的，这一点不要搞错。

## 直接通过 ReportServer 访问报表

还记得前面提到过的 <http://ServerUrl/ReportServer> 吗？SSRS 已经为我们准备了一个用来查看报表的方法，即通过 URL 访问，比如要查看在 localhost/ReportServer 服务器中，位于 Test 下的 Report1 报表，可以直接在浏览器中输入  <http://localhost/ReportServer?Test/Report1>，SSRS 会自动调用一个系统内置的页面来显示它。在这个带参数的 URL 后面，我们可以通过附加 URL 参数的方法来对报表进行控制。比如上面的那个例子，在 ASP.NET 中可以使用 Response.Write() 向页面写入下面的代码来弹出窗口显示报表：

```html
"<script language=\"JavaScript\"> window.open('http://localhost/ReportServer?Test/Report1&params=" + textQueryString.Text + "&rc:Parameters=false&rs:Command=Render'; </script>"
```

其中 URL 参数的构造方法请参阅 MSDN 相关文档。

# 转

上面说的都是常规应用，但实际上一个报表从提出需求到最后部署，绝大部分都不是会了那个示例就能做的，中间会遇到各种各样的问题。在这一部分中，我以问答的形式，将开发过程中遇到的问题以及解决方法分门类地列举出来，并且不断更新。

## 部署与调试

Q：我在 ASP.NET 下使用 ReportViewer 载入报表，为什么会出现 {用户"NT AUTHORITY\NETWORK SERVICE"授予的权限不足，无法执行此操作。 (rsAccessDenied)} 的错误？
A：这是由于 IIS 下 ReportServer 虚拟目录的访问权限没有设置正确。解决问题的方法有三种：

* 在服务器端访问 <http://localhost/Reports>，进入 Report Manager，然后点击「属性」标签页下的「新建角色分配」，在「组或用户名」中填入「NT AUTHORITY\NETWORK SERVICE」（没有两边的引号），在下面勾选 Browser，确定。这是给该账户赋以浏览报表的权限，我强烈推荐这种方法；
* 在 IIS 中，修改默认站点下 ReportServer 虚拟目录的属性，在 Directory Security 选项卡中，点击 Authentication and access control 中的 Edit，开启匿名访问，将匿名访问帐户设为管理员账号，本地登录的就设为 Administrator，域账号登录的就设为具有管理员权限的域账号。这样可以使访问 ReportServer 的连接以管理员权限浏览报表。这也是网上流传最广的方法，但存在严重安全隐患，开发调试的时候没问题，真正部署的时候不建议使用；
* 专门为 Reporting Services 建立一个匿名帐户，比如 IUSR_ReportView，然后在 Report Manager 里分配 Browser 角色，还有等等后续步骤。这是最麻烦的，步骤之多我都懒得在这里写全，同样在网上流传很广，但我觉得只有实在闲着没事干的人才会用。。。

Q：我在 Visual Studio 里开发 VWG，以 debug 方式运行，然后在 ReportViewer 导出 PDF 时就报 Session Expired 错误，这是怎么回事？
A：其实我也不知道为什么。。。解决的方法是不用 debug 方式，直接在浏览器访问站点，就 OK 了。至于其原因，呼唤高人来解释~~~

## 报表数据相关

Q：我现在不仅仅想向报表传递传统的 SQL 参数，比如 @ID、@Count，而是想整条 WHERE 子句以至整个 SQL 语句的任何地方都可以用参数的形式来控制，可以吗？
A：当然可以，这都不行那 SSRS 也太圡了。。在这里需要用到报表语句。用过 Excel 吧，单元格的值如果用「=」开头，那后面就可以跟表达式，报表语句也一样。一个最简单的例子，比如像我现在想用参数传递一整条 WHERE 子句，就这么做：

1. 菜单中 Report -> Report Parameters，Add 一个新参数，假定 Name 叫做 WhereString，类型 String，别的不动
1. 在 Data 页中修改需要接受参数的 DataSet，假设原来 SQL 语句的内容为 `select * from table1 where age between 10 and 15 and id>40`
1. 修改为 `="select * from table1 where " + Parameters!WhereString.Value`
1. 好，切换到 Preview 中，看结果吧。

注意这里的 where 后面有个空格，尤其是在构造 SQL 语句的时候，一定要特别注意空格的使用。

Q：我在那个 Parameters 提示框中什么都没填，直接点击 View Report，出错了耶……
A：-_-|| 当然会出错，SQL 语句中的 where 后面没有条件，语法不正确啊！像这种情况，你得在表达式语句里做一个判断，看是否传入的参数为空。上面那个例子，就可以改为：`="select * from table1″ + IIF(Parameters!WhereString.Value<>"", " where " + Parameters!WhereString.Value, "")` 这里的 IIF 是报表语句的一种，属于 Program Flow 语句。至于更博大精深的用法，参阅文档~

Q：这太神奇了，那你能告诉我，如果我想在一个页面里加入「第 X 页/共 Y 页」这种，又该怎么做呢？
A：这还是要用到报表语句。进入 Layout 视图，在任意一个文本框右键点击然后选 Expression，就是前面有个「fx」图标的那个，就能看到表达式编辑器。在左下的那个框中列出的就是报表语句的类别，看到 Globals 了吗？点击一下，右边就会有一些关于报表本身的常量，那个 PageNumber 和 TotalPages 就是这里要用的。前面用到的 Parameters 和 IIF 在这里也都能找到。灵活应用这些元素可以极大丰富报表的表现力。哦，忘了说一点，PageNumber 和 TotalPages 这两个量都只能在页眉或者页脚处才可使用，所以在用它们之前需要在 Layout 视图中，菜单 Report -> Page Header（或Page Footer），开启页眉或页脚，然后在里面引用。

## 报表 Layout 相关

Q：为什么我的报表打印出来之后水平方向跨页啦？我在设计页面中明明没有越界啊！
A：cft……你的眼睛欺骗了你。设计界面的那个页面边界其实跟你的纸张页面完全没有关系，SSRS 也没有提供整页居中的功能，所以你必须一点点地微调你的页面布局。首先我建议在菜单 Report -> Report Properties -> Layout 选项卡中，将左右 Margin 都改为0。然后回到设计界面，通过手动指定控件的 Locate 属性的方法，来确定左侧边界的大小。比如现在的纸张是 A4 大小（21cm * 29.7cm），我的正文表格总宽度设置为 17.75cm，剩下的 3.25cm 是左右页边距总和，按理说应该各分配 1.625cm，但考虑到表格里的表格线也是有宽度的（我设的是 1pt），因此将左页边距设为 1.6cm，也就是把控件摆放在距离设计页面左侧 1.6cm 的位置，再说的明白点就是把靠左的各控件的 Locate 属性中的 Left 元素的值设为 1.6cm（好多定语啊……我不是故意的……）。进行预览的时候建议用 ReportViewer 的导出功能导出为 pdf 查看，也可以打印到 pdf、xps，要是你再土豪一点，直接打印到纸上看是最好。这个微调是个痛苦的过程。

# 合

经过这一段时间的开发经历，个人感觉 SSRS 还是非常好用的，与 SQL Server 的无缝集成，layout 的灵活多变以及报表语言的强大功能，加上微软产品的亲和性，带来的是清爽的使用体验。不过 SSRS 的执行效率并不高，尤其是 SQL 2005 版的 SSRS 在建立 DataSet 的时候还只能用 SQL 语句来查询，实现复杂的业务逻辑还不是很方便。并且在动态列显示以及动态页面设置还有待提高。期待 SQL Server 2008 版 SSRS 能够更上一层楼~
