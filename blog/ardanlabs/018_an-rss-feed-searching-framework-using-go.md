# 使用 Go 的 RSS 搜索框架

> [An RSS Feed Searching Framework Using Go](https://www.ardanlabs.com/blog/2013/07/an-rss-feed-searching-framework-using-go.html)


_这篇文章是为 Safari Books Online 编写和发布的_

早在 5 月，我终于决定是时候从 Microsoft 堆栈转移到 Linux 了。在 Microsoft 堆栈上开发、许可和购买云计算的成本太高了。我的公司获得了 BizSpark
的批准，这使得像我这样的小公司的发展成为可能，但三年后我要做什么？我的另一个问题是我如何要求客户为微软云计算和许可支付数千美元，尤其是当在 Linux 上运行的开源技术只需花费数百美元，而且这些技术可以更好地扩展时。

我们在 Web 项目中使用 Ruby 已经有一段时间了。不幸的是，我们需要继续使用 .NET 进行服务器开发。我不想再次开始编写 C/C++，而 Ruby、Erlang
和我看过的所有其他语言都觉得不适合这种类型的开发。然后我的搭档让我看看 Go。

您可以使用 Go 编程语言和范例实现非常健壮的代码。几个小时后，我就可以编写一个名为 NewsSearch 的程序。该程序下载 RSS 提要文档列表并使用不同的搜索插件对其进行处理。第一个搜索插件称为 Expression。它搜索每个 RSS
文档以寻找匹配的关键字或正则表达式。它显示任何匹配的文档以及匹配的位置。