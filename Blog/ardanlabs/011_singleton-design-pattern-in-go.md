# Go 中的单例模式

> [Singleton Design Pattern in Go](https://www.ardanlabs.com/blog/2013/07/singleton-design-pattern-in-go.html)


多线程应用程序非常复杂，尤其是当您的代码没有组织并且与资源的访问、管理和维护方式不一致时。如果您想尽量减少错误，您需要遵循一些理念和规则。以下是我的一些：

1. 资源分配和释放应该在同一类型内抽象和管理。
2. 资源线程安全性应该在同一类型中抽象和管理。
3. 公共接口应该是访问共享资源的唯一方法。
4. 任何被分配资源的线程都应该释放相同的资源。

在 Go 中，我们没有 threads，只有 Go Routines。Go 运行时抽象了这些 routines 的线程和任务交换。无论如何，同样的理念和规则依旧适用。

我最喜欢的设计模式之一是 Singleton。当您只需要一个类型的一个实例且该类型管理共享资源时，它提供了一个很好的实现。Singleton
是一种设计模式，其中类型创建自己的实例并保持该引用私有。对该引用管理的共享资源的访问是通过静态公共接口抽象出来的。这些静态方法还提供线程安全性。使用 Singleton 的应用程序负责初始化和取消初始化
Singleton，但从不直接访问内部。

一段时间以来，我一直没有想到如何在 Go 中实现 Singleton，因为 Go 不是传统的面向对象编程语言，也没有静态方法。

我认为 Go 是一种轻量级的面向对象编程语言。是的，它确实具有封装和类型成员函数，但它缺乏继承，因此缺乏传统的多态性。在我使用过的所有 OOP 语言中，除非我想实现多态性，否则我从不使用继承。通过在 Go 中实现接口的方式，不需要继承。Go
采用了 OOP 中最好的部分，省略了其余部分，为我们提供了一种更好的方式来编写多态代码。

在 Go 中，我们可以通过利用包和类型的作用域和封装规则来实现单例。在这篇文章中，我们将探索我的 straps 包，因为它将为我们提供一个真实世界的示例。

straps 包提供了一种将配置选项（strap）存储在 XML 文档中并将它们读入内存以供应用程序使用的机制。strap 名字来自于配置网络设备的早期。这些设置被称为 straps，这个名字一直伴随着我。在 MacOS 中我们有
.plist 文件，在 .Net 中我们有 app.config 文件，在 Go 中我有 straps.xml 文件。

这是我的一个应用程序的 straps 示例文件：

```
<straps>
    <!– Log Settings –>
    <strap key="baseFilePath" value="/Users/bill/Logs/OC-DataServer">
    <strap key="machineName" value="my-machine">
    <strap key="daysToKeep" value="1">

    <!– ServerManager Settings –>
    <strap key="cpuMultiplier" value="100">
</straps>
```

straps 包知道如何读取这个 xml 文件并通过基于 Singleton 的公共接口提供对值的访问。因为这些值只需要被读入到内存一次，所以 Singleton 是一个很好的选择。

以下是 straps 包和类型的信息：

```
package straps

import (
    "encoding/xml"
    "io"
    "os"
    "path/filepath"
    "strconv"
)

.
. Types Removed
.

type straps struct {
    StrapMap map[string]string // The map of strap key value pairs
}

var st straps // A reference to the singleton
```

我不打算谈论读取 XML 文档的各个方面。如果您有兴趣，请阅读这篇博文 [reading-xml-documents-in-go](./008_reading-xml-documents-in-go.md)。

在上面的代码片段中，您将看到包名称（straps）、私有类型 *straps* 的定义和私有包变量 *st*。在 *st* 变量将包含 Singleton 值。

Go 的作用域规则规定，以大写字母开头的类型和函数是公共的，并且可以在包外访问。以小写字母开头的类型和函数是私有的，不能在包外访问。

我在函数范围内用小写字母命名定义变量。在函数范围之外定义的变量名，例如类型成员和包变量以大写字母开头。这样我查看代码时就可以立即知道引用变量的内存位置。

*straps* 类型和 *st* 变量都是私有的，只能从包中访问。

查看初始化 Singleton 所需要的 Load 函数：

```
func MustLoad() {
    // Find the location of the straps.xml file
    strapsFilePath, err := filepath.Abs("straps.xml")

    // Open the straps.xml file
    file, err := os.Open(strapsFilePath)
    if err != nil {
        panic(err.Error())
    }

    defer file.Close()

    // Read the straps file
    xmlStraps, err := readStraps(file)
    if err != nil {
        panic(err.Error())
    }

    // Create a straps object
    st = straps{
        StrapMap: make(map[string]string),
    }

    // Store the key/value pairs for each strap
    for _, strap := range xmlStraps {
        st.StrapMap[strap.Key] = strap.Value
    }
}
```

Load 函数是包的公共函数。应用程序可以通过包名访问该函数。您可以看到我如何使用以小写字母开头的名称作为局部变量。在 Load 函数的底部创建了一个 straps 对象，并将其引用设置为 *st* 变量。此时 Singleton
已存在，并且可以使用 straps。

使用公共函数 Strap 访问 straps：

```
func Strap(key string) string {
    return st.StrapMap[key]
}
```

公共函数 Strap 使用 Singleton 引用来访问共享资源。在这种情况下，straps 映射。如果该映射在应用程序的生命周期内可能发生变化，则需要使用互斥锁或其他同步对象来保护映射。幸运的是，straps 一旦加载就永远不会改变。

由于 straps 管理的资源只是内存，因此不需要 Unload 或 Close 方法。如果我们需要一个函数来关闭任何资源，则必须创建另一个公共函数。

如果 Singleton
包中需要私有方法来帮助组织代码，我喜欢使用类型函数。由于类型是私有的，我可以将成员函数设为公开，因为它们无法访问。我还认为类型函数有助于使代码更具可读性。通过查看该函数是否为类型函数，我知道该函数是私有的还是公共接口的一部分。

下面是一个使用类型函数的例子：

```
func SomePublicFunction() {
    .
    st.SomePrivateMemberFunction("key")
    .
}

func (straps *straps) SomePrivateMemberFunction(key string) {
    return straps.StrapMap[key]
    .
}
```

由于该函数是类型函数，我们需要使用 *st* 变量来进行函数调用。在类型函数中，我使用局部变量（straps）而不是 *st* 变量。类型函数是公共的，但引用是私有的，因此只有包可以引用成员函数。这只是我为自己建立的约定。

这是一个使用 straps 包的示例程序：

```
package main

import (
    "ArdanStudios/straps"
)

func main() {
    straps.MustLoad()
    cpu := straps.Strap("cpuMultiplier")
}
```

在 main 中，我们不需要分配任何内存或维护引用。通过包名调用 Load 来初始化 Singleton。然后再次通过包名访问公共接口，在本例中为 Strap 函数。

如果您同样需要通过公共接口管理共享资源，请尝试使用 Singleton。

与往常一样，我希望这可以帮助您编写更好的代码并减少错误。