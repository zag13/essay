## 安装 Go、Gocode、GDB 和 LiteIDE

> [Installing Go, Gocode, GDB and LiteIDE](https://www.ardanlabs.com/blog/2013/06/installing-go-gocode-gdb-and-liteide.html)

查看我的新安装文档：
https://www.ardanlabs.com/blog/2016/05/installing-go-and-your-workspace.html

*我删除了关于 gocode 和 GDB 的部分。这些不再需要了。我还添加了更多编辑器的链接。*

**Windows**
这是[Wade Wegner 的](https://twitter.com/WadeWegner)一篇很棒的帖子，用于在您的 Windows 机器上安装 Go：
http://www.wadewegner.com/2014/12/easy-go-programming-setup-for-windows/

**Mac OS X**

以下说明将指导您在 Mac 上安装 Go

**第 1 步：下载 Go**

打开您喜欢的浏览器并访问以下网站：http://golang.org/dl/

这个网站会向您展示适用于不同操作系统的所有最新版本。Darwin 是 Mac OS 的名称。下载适用于您的操作系统版本和架构的安装程序包。

下载后，转到 Finder 中的“Download”文件夹，然后双击 pkg 文件开始安装。安装会将 Go 放在***/usr/local/go*** 中。安装完成后，您将需要检查安装。

*注意：下一步我们将同时使用 Terminal 和 Finder ，这样可以在 Finder 中查看所有文件。默认情况下，Finder 不会向您显示硬盘驱动器上的所有内容。*

***在 Finder 中显示所有文件：***

打开终端会话。 如果您不知道终端程序在哪里，请转到 Finder 中的应用程序文件夹，然后转到实用程序。 单击终端，将打开一个 bash shell 命令窗口。

在终端中执行以下命令。

```
defaults write com.apple.finder AppleShowAllFiles TRUE
killall Finder
```

`killall Finder`  会重启 Finder ，然后你就会看到所有文件了。

**第二步：校验安装**

打开终端对话，输入一下命令  `go version`

如果一切都顺利的话，你会看到  `go version go1.5 darwin/amd64`

现在输入  `which`  来查看安装路径，是否符合预期  `which go`

屏幕输出  `/usr/local/go/bin/go`

**第三步：设置 GOPATH**

您需要一个地方来存储和处理您的 Go 项目。 从您的主文件夹中创建一个名为 projects 的文件夹：

```
cd $HOME
mkdir projects
cd projects
mkdir src
mkdir bin
```

现在将 projects 设置为您的 GOPATH。 从 $HOME 文件夹中打开 .bash_profile 文件并将以下项目添加到最后。

```
export GOPATH=“$HOME/projects”
export PATH=$PATH:$GOPATH/bin
```

然后重启终端，检查 Go 环境现在是否有新的 GOPATH。  `go env`

您应该会看到所有与 Go 相关的环境变量，包括 GOPATH。 这里是其中的一些：

```
GOARCH=“amd64”
GOHOSTARCH=“amd64”
GOHOSTOS=“darwin”
GOOS=“darwin”
GOPATH=“/Users/you/projects”
GOROOT=“/usr/local/go”
GOTOOLDIR=“/usr/local/go/pkg/tool/darwin_amd64”
```

**第四步：安装一些基础包**

这是一些你可能会用到的包。

```
// gocode is used by many editors to provide intellisense
go get github.com/nsf/gocode

// goimports is something you should run when saving code to fill in import paths
go get golang.org/x/tools/cmd/goimports

// gorename is used by many editors to provide identifier rename support
go get golang.org/x/tools/cmd/gorename

// oracle is a tool that help with code navigation and search
go get golang.org/x/tools/cmd/oracle

// golint should be run after every build to check your code
go get github.com/golang/lint/golint
```

**第五步： Debugger**

使用 GDB 不是唯一的选择。 如果您想尝试使用 Delve，这里是一个链接：https://github.com/derekparker/delve

**第六步：安装编译器**

- **[Sublime](http://www.sublimetext.com/)**
    - DisposaBoy 写了这个插件 https://github.com/DisposaBoy/GoSublime
    - Mark Wolfe 写了这篇文章 http://www.wolfe.id.au/2015/03/05/using-sublime-text-for-go-development/
- **[VIM](http://www.vim.org/download.php)**
    - Victor Farazdagi 写了这篇文章 http://farazdagi.com/blog/2015/vim-as-golang-ide/
- **[Atom](https://atom.io/)**
    - Joe Fitzgerald 写了这个插件 https://github.com/joefitzgerald/go-plus
- **[LiteIDE](http://sourceforge.net/projects/liteide/files/)**
- **[Emacs](https://github.com/creack/dotfiles
  )**

**一些很有用的网站**

这些网页非常有用：

http://golang.org/
https://github.com/golang/go/wiki
http://blog.golang.org/
http://www.youtube.com/user/gocoding
http://dave.cheney.net/
http://gophervids.appspot.com/

关于 Go Concurrency Patterns 你必须要看的视频：

http://www.youtube.com/watch?v=QDDwwePbDtw
http://www.youtube.com/watch?v=f6kdp27TYZs
