# [How to Write Go Code](https://go.dev/doc/code)

- [Introduction](#introduction)
- [Code organization](#code-organization)
- [Your first program](#your-first-program)
    - [Importing packages from your module](#importing-packages-from-your-module)
    - [Importing packages from remote modules](#importing-packages-from-remote-modules)
- [Testing](#testing)
- [What's next](#whats-next)
- [Getting help](#getting-help)

## Introduction

本文档演示了使用模块开发一个简单的 Go 包，并介绍了 [go tool](https://go.dev/cmd/go/)（获取、构建和安装 Go 模块、包和命令的标准方法）。

注意：本文档假设你使用的是 Go 1.13 或更高版本，并且环境变量 GO111MODULE 未设置。如果你正在寻找本文档的较旧版本，它被存档在[这里](https://go.dev/doc/gopath_code.html)。

## Code organization

Go 程序以*包*为基本单元。*包*是在同一目录中编译在一起的源文件的集合。源文件中定义的函数、类型、变量和常量对同一包中的所有其他源文件都是可见的。

存储库包含一个或多个*模块*。*模块*是一起发布的相关 Go 包的集合。Go 存储库通常只包含一个模块，位于存储库的根目录。一个名为`go.mod`的文件声明了*模块路径*
：模块中所有包的导入路径前缀。该模块包括`go.mod`文件所在的目录中以及其子目录，直到另一个`go.mod`文件（如果有）所在的下一个子目录。

注意，你无需在构建代码之前将代码发布到远程存储库。模块可以在本地定义而不属于存储库。但是，像有一天会发布它一样来组织代码是一个好习惯。

每个模块的路径不仅用作其包的导入路径前缀，而且还指示`go`命令应该从哪里下载它。例如，为了下载模块`golang.org/x/tools`，`go`
命令将下载由`https://golang.org/x/tools`（详细描述在[此](https://go.dev/cmd/go/#hdr-Remote_import_paths)）指引的存储库。

*导入路径*是用于导入包的字符串。包的导入路径是其模块路径与其在模块中的子目录相拼接。例如，包`cmp/`在模块`github.com/google/go-cmp`中。该包的导入路径是
`github.com/google/go-cmp/cmp`。标准库中的包没有模块路径前缀。

## Your first program

要编译和运行一个简单的程序，首先选择一个模块路径（我们将使用`example/user/hello`）并创建一个`go.mod`文件：

```
$ mkdir hello # Alternatively, clone it if it already exists in version control.
$ cd hello
$ go mod init example/user/hello
go: creating new go.mod: module example/user/hello
$ cat go.mod
module example/user/hello

go 1.16
$
```

Go 源文件中的第一条语句必须是`package name`。可执行命令必须使用`package main`。

接下来，在该目录中创建一个名为`hello.go`的文件，其中代码如下：

```
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```

现在你可以使用`go`工具构建和安装该程序：

```
$ go install example/user/hello
$
```

此命令构建`hello`命令，生成可执行二进制文件。然后它将该二进制文件安装为`$HOME/go/bin/hello`（在 Windows 下，`%USERPROFILE%\go\bin\hello.exe`）。

安装目录由[环境变量](https://go.dev/cmd/go/#hdr-Environment_variables)`GOPATH`和`GOBIN`控制。如果`GOBIN`设置，二进制文件将安装到该目录。 如果`GOPATH`
设置，二进制文件将安装到`GOPATH`列表中第一个`bin`子目录。否则，二进制文件将安装到默认`GOPATH`下的`bin`子目录 (`$HOME/go` 或 `%USERPROFILE%\go`)。

你可以使用该`go env`命令为以后的`go`命令设置环境变量的默认值：

```
$ go env -w GOBIN=/somewhere/else/bin
$
```

要取消由`go env -w`设置的变量，请使用`go env -u`：

```
$ go env -u GOBIN
$
```

类似`go install`的命令适用于包含当前工作目录的模块的上下文中。如果工作目录不在example/user/hello模块内，go install可能会失败。

为方便起见，go命令接受相对于工作目录的路径，如果没有给出其他路径，则默认为当前工作目录中的包。所以在我们的工作目录中，下面的命令都是等价的：

```
$ go install example/user/hello
$ go install .
$ go install
```

接下来，让我们运行程序以确保其正常工作。为了方便起见，我们将安装目录添加到我们的目录中，PATH以便轻松运行二进制文件：

```
# Windows users should consult https://github.com/golang/go/wiki/SettingGOPATH
# for setting %PATH%.
$ export PATH=$PATH:$(dirname $(go list -f '{{.Target}}' .))
$ hello
Hello, world.
$
```

如果你使用的是源代码控制系统，那么现在是初始化存储库、添加文件并提交你的第一个更改的好时机。同样，这一步是可选的：你不需要使用源代码管理来编写 Go 代码。

```
$ git init
Initialized empty Git repository in /home/user/hello/.git/
$ git add go.mod hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 7 insertion(+)
 create mode 100644 go.mod hello.go
$
```

该go命令通过请求相应的 HTTPS URL 并读取嵌入在 HTML 响应中的元数据来定位包含给定模块路径的存储库（请参阅 参考资料 go help importpath）。许多托管服务已经为包含 Go
代码的存储库提供了该元数据，因此使你的模块可供其他人使用的最简单方法通常是使其模块路径与存储库的 URL 匹配。

### Importing packages from your module

让我们编写一个morestrings包并从hello程序中使用它。首先，为名为 的包创建一个目录 $HOME/hello/morestrings，然后 reverse.go在该目录中创建一个名为的文件，其内容如下：

```
// Package morestrings implements additional functions to manipulate UTF-8
// encoded strings, beyond what is provided in the standard "strings" package.
package morestrings

// ReverseRunes returns its argument string reversed rune-wise left to right.
func ReverseRunes(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

因为我们的ReverseRunes函数以大写字母开头，所以它被导出，并且可以在导入我们morestrings 包的其他包中使用。

让我们测试一下这个包是否可以编译go build：

```
$ cd $HOME/hello/morestrings
$ go build
$
```

这不会产生输出文件。相反，它将编译的包保存在本地构建缓存中。

确认morestrings包构建后，让我们从hello程序中使用它。为此，请修改你的原始 $HOME/hello/hello.go文件以使用 morestrings 包：

```
package main

import (
	"fmt"

	"example/user/hello/morestrings"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
}
```

安装hello程序：

```
$ go install example/user/hello
```

运行新版本的程序，你应该会看到一条新的反转消息：

```
$ hello
Hello, Go!
```

### Importing packages from remote modules

导入路径可以描述如何使用 Git 或 Mercurial 等修订控制系统获取包源代码。该go工具使用此属性自动从远程存储库中获取包。例如，要github.com/google/go-cmp/cmp在你的程序中使用：

```
package main

import (
	"fmt"

	"example/user/hello/morestrings"
	"github.com/google/go-cmp/cmp"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
	fmt.Println(cmp.Diff("Hello World", "Hello Go"))
}
```

现在你已经依赖于外部模块，你需要下载该模块并将其版本记录在你的go.mod文件中。该go mod tidy命令为导入的包添加缺少的模块要求，并删除不再使用的模块的要求。

```
$ go mod tidy
go: finding module for package github.com/google/go-cmp/cmp
go: found github.com/google/go-cmp/cmp in github.com/google/go-cmp v0.5.4
$ go install example/user/hello
$ hello
Hello, Go!
  string(
- 	"Hello World",
+ 	"Hello Go",
  )
$ cat go.mod
module example/user/hello

go 1.16

require github.com/google/go-cmp v0.5.4
$
```

模块依赖项会自动下载到环境变量pkg/mod
指示的目录的子目录中。GOPATH给定版本的模块的下载内容在该版本的所有其他模块之间共享require，因此该go命令将这些文件和目录标记为只读。要删除所有下载的模块，你可以将-modcache标志传递给go clean：

```
$ go clean -modcache
$
```

## Testing

Go 有一个由go test 命令和testing包组成的轻量级测试框架。

你可以通过创建一个名称以 结尾的文件来编写测试，该文件_test.go 包含以TestXXXsignature 命名的函数func (t *testing.T)。测试框架运行每个这样的功能；如果函数调用失败函数，例如t.Erroror
t.Fail，则认为测试失败。

morestrings通过创建 $HOME/hello/morestrings/reverse_test.go包含以下 Go 代码 的文件， 将测试添加到包中。

```
package morestrings

import "testing"

func TestReverseRunes(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := ReverseRunes(c.in)
		if got != c.want {
			t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

然后运行测试go test：

```
$ cd $HOME/hello/morestrings
$ go test
PASS
ok  	example/user/hello/morestrings 0.165s
$
```

运行go help test并查看 测试包文档以获取更多详细信息。

## What's next

订阅 golang-announce 邮件列表，以便在发布新的稳定版 Go 时收到通知。

有关编写清晰、惯用的 Go 代码的技巧，请参阅Effective Go。

参加 A Tour of Go 以学习正确的语言。

访问文档页面，获取一组关于 Go 语言及其库和工具的深入文章。

## Getting help

如需实时帮助，请在社区运行的 gophers Slack 服务器中询问有帮助的 gophers（在此处 获取邀请）。

讨论 Go 语言的官方邮件列表是 Go Nuts。

使用Go 问题跟踪器 报告错误。