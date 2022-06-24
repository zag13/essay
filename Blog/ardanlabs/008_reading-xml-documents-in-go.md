# 在 Go 中读取 XML 文档

> [Reading XML Documents in Go](https://www.ardanlabs.com/blog/2013/06/reading-xml-documents-in-go.html)

我是真的被震惊到了，通过标准库中的 encoding/xml 包来读取 XML 文档是多么的简单啊。该包通过定义映射 XML 文档的结构体来工作。如果您需要更多的灵活性，请使用 Gustavo Niemeyer
的 [xmlpath](http://godoc.org/launchpad.net/xmlpath) 包。

这是我们将要读取和反序列化的 XML 文档：

```
<straps>
    <strap key="CompanyName" value="NEWCO" />
    <strap key="UseEmail" value="true" />
</straps>
```

我们首先要做的事情是定义一个用来映射文档的结构体：

```
type XMLStrap struct {
    XMLName  xml.Name  ̀xml:"strap"̀
    Key      string    ̀xml:"key,attr"̀
    Value    string    ̀xml:"value,attr"̀
}

type XMLStraps struct {
    XMLName  xml.Name    ̀xml:"straps"̀
    Straps   []XMLStrap  ̀xml:"strap"̀
}
```

定义了两个结构体，一个用于整个文档 (<strap>)，一个用于每个单独的子节点 (<strap>)。 如果你仔细观察这些结构，你可能会看到一些新的东西。每个字段都有一个与之关联的标签。这些标签绑定到每个单独的字段。Go
的反射包允许你访问这些标签。

这些标签格式被特定于 encoding/xml 包内的解码支持。 标签将 XML 文档的节点和属性映射到结构体中。

以下代码解码 XML 文档并返回 strap 节点数组：

```
func ReadStraps(reader io.Reader) ([]XMLStrap, error) {
    var xmlStraps XMLStraps
    if err := xml.NewDecoder(reader).Decode(&xmlStraps); err != nil {
        return nil, err
    }

    return xmlStraps.Straps, nil
}
```

该函数需要一个 io.Reader。 我们将向这个方法传递一个 os.File 变量。 该函数将返回一个从文件中读取的包括每个 strap 的指针数组。

首先我们创建一个 XMLStraps 变量并获取它的地址。 接下来，我们使用传递 io.Reader 对象的 xml.NewDecoder 方法创建解码器。 然后我们调用 Decode，它读取文件并将文件反序列化为 XMLStraps
变量。 然后我们只返回 strap 数组。

以下为完整示例代码：

```
/*
straps.xml should be located in the default working directory

<straps>
    <strap key="CompanyName" value="NEWCO" />
    <strap key="UseEmail" value="true" />
</straps>
*/
package main

import (
    "encoding/xml"
    "fmt"
    "io"
    "os"
    "path/filepath"
)

type XMLStrap struct {
    XMLName  xml.Name ̀ xml:"strap"̀
    Key      string   ̀ xml:"key,attr"̀
    Value    string   ̀ xml:"value,attr"̀
}

type XMLStraps struct {
    XMLName  xml.Name    ̀ xml:"straps"̀
    Straps   []XMLStrap ̀ xml:"strap"̀
}

func ReadStraps(reader io.Reader) ([]XMLStrap, error) {
    var xmlStraps XMLStraps
    if err := xml.NewDecoder(reader).Decode(&xmlStraps); err != nil {
        return nil, err
    }

    return xmlStraps.Straps, nil
}

func main() {
    // Build the location of the straps.xml file
    // filepath.Abs appends the file name to the default working directly
    strapsFilePath, err := filepath.Abs("straps.xml")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }

    // Open the straps.xml file
    file, err := os.Open(strapsFilePath)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }

    defer file.Close()

    // Read the straps file
    xmlStraps, err := ReadStraps(file)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }

    // Display The first strap
    fmt.Printf("Key: %s  Value: %s", xmlStraps[0].Key, xmlStraps[0].Value)
}
```

我希望这个示例能帮助您阅读 Go 应用程序的 XML 文档。