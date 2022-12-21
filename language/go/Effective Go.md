# [Effective Go](https://go.dev/doc/effective_go)

- [Introduction](#introduction)
    - [Examples](#examples)
- [Formatting](#formatting)
- [Commentary](#commentary)
- [Names](#names)
    - [Package names](#package-names)
    - [Getters](#getters)
    - [Interface names](#interface-names)
    - [MixedCaps](#mixedcaps)
- [Semicolons](#semicolons)
- [Control structures](#control-structures)
    - [If](#if)
    - [Redeclaration and reassignment](#redeclaration-and-reassignment)
    - [For](#for)
    - [Switch](#switch)
    - [Type switch](#type-switch)
- [Functions](#functions)
    - [Multiple return values](#multiple-return-values)
    - [Named result parameters](#named-result-parameters)
    - [Defer](#defer)
- [Data](#data)
    - [Allocation with new](#allocation-with-new)
    - [Constructors and composite literals](#constructors-and-composite-literals)
    - [Allocation with make](#allocation-with-make)
    - [Arrays](#arrays)
    - [Slices](#slices)
    - [Two-dimensional slices](#two-dimensional-slices)
    - [Maps](#maps)
    - [Printing](#printing)
    - [Append](#append)
- [Initialization](#initialization)
    - [Constants](#constants)
    - [Variables](#variables)
    - [The init function](#the-init-function)
- [Methods](#methods)
    - [Pointers vs. Values](#pointers-vs-values)
- [Interfaces and other types](#interfaces-and-other-types)
    - [Interfaces](#interfaces)
    - [Conversions](#conversions)
    - [Interface conversions and type assertions](#interface-conversions-and-type-assertions)
    - [Generality](#generality)
    - [Interfaces and methods](#interfaces-and-methods)
- [The blank identifier](#the-blank-identifier)
    - [The blank identifier in multiple assignment](#the-blank-identifier-in-multiple-assignment)
    - [Unused imports and variables](#unused-imports-and-variables)
    - [Import for side effect](#import-for-side-effect)
    - [Interface checks](#interface-checks)
- [Embedding](#embedding)
- [Concurrency](#concurrency)
    - [Share by communicating](#share-by-communicating)
    - [Goroutines](#goroutines)
    - [Channels](#channels)
    - [Channels of channels](#channels-of-channels)
    - [Parallelization](#parallelization)
    - [A leaky buffer](#a-leaky-buffer)
- [Errors](#errors)
    - [Panic](#panic)
    - [Recover](#recover)
- [A web server](#a-web-server)

## Introduction

Go是一个新语言。
虽然借鉴了现有语言的一些概念，但它不同寻常的特性使Go程序与它的近亲语言存在很大的不同。
将C++或Java程序直接翻译成Go不太可能产生令人满意的结果——Java程序是用Java编写的，而不是Go。
另一方面，从Go的角度思考问题可能会产生一个成功但完全不同的程序。
换句话说，要写好Go，重要的是要了解它的特性和习语。
了解Go语言的通俗约定也很重要，例如命名、格式化、程序结构等，以便您编写的程序易于其他Go程序员理解。

本文档提供了编写清晰、符合语言习惯的的Go代码的技巧。
它扩充了[language specification](https://go.dev/ref/spec)、
[Tour of Go](https://go.dev/tour/)和[How to Write Go Code](https://go.dev/doc/code.html)，您应该首先阅读这些内容。

2022年1月添加的注释：本文档是为Go于2009年发布而编写的，此后没有进行过重大更新。
尽管作为了解如何使用语言本身它是一个很好的指南，但由于语言的稳定性，它很少提及库，也没有提及自编写以来Go生态系统的重大变化，例如构建系统、测试、模块和多态性。
没有更新本文档的计划，因为已经发生了很多事情，而且越来越多的文档、博客和书籍很好地描述了现代Go的用法。
Effective Go仍然有用，但读者应该明白它远非完整的指南。
查看[issue 28782](https://github.com/golang/go/issues/28782)获取更多信息。

### Examples

[Go package sources](https://go.dev/src/)不仅是核心库，而且可以作为如何使用该语言的示例。
此外，许多包都包含有效的、独立的可执行示例，您可以直接从[golang.org](https://golang.org/)
网站运行，例如[这个](https://go.dev/pkg/strings/#example_Map)（如有必要，单击 "Example" 将其打开）。
如果您对如何解决问题或如何实现一些东西有疑问，依赖库中的文档、代码和示例可以提供答案、想法和背景介绍。

## Formatting

格式问题是最有争议但最不重要的。
人们可以适应不同的格式风格，但最好不必这样做，如果每个人都坚持相同的风格，那么花在这个问题上的时间就会更少。
问题是如何在没有冗长的规范风格指南的情况下接近这个乌托邦。

在Go中，我们采取了一种不同寻常的方法，让机器处理大多数格式问题。
Gofmt程序（也可用作go fmt，它在包级别而不是源文件级别运行）读取Go程序并以标准缩进和垂直对齐样式更改源代码，保留并在必要时重新格式化注释。
如果关于一些新的布局情况你想知道如何处理，运行gofmt；如果答案似乎不正确，请修改您的程序（或提交有关gofmt的错误），不要忽略它。

例如，无需花时间排列结构体字段上的注释。
Gofmt将为您做到这一点。
声明如下

```
type T struct {
    name string // name of the object
    value int // its value
}
```

gofmt将其排列为：

```
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

标准包中的所有Go代码都已使用gofmt格式化。

关于格式的一些细节仍然存在。
非常简短：

- 缩进
    - 我们使用制表符进行缩进，gofmt默认使用。仅在必须时使用空格。
- 行长
    - Go没有行长限制。不用担心溢出。如果感觉一行太长，请将其包裹起来并用额外的制表符缩进。
- 括号
    - 与C和Java相比，Go需要的括号更少：控制结构（if、for、switch）的语法中没有括号。此外运算符优先级层次结构更短更清晰，所以
      ```
      x<<8 + y<<16
      ```
      间距是有其暗示的含义，这与与其他语言不同。

## Commentary

Go提供C风格的块注释/* */和C++风格的行注释//。
行注释是常态；块注释主要用于包注释，但在表达式中或禁用大量代码时很有用。

出现在顶级声明之前的注释，中间没有换行符，被认为是解释文档。
这些"文档注释"是Go包或命令的主要文档。
有关文档注释的更多信息，请参阅[Go Doc Comments](https://go.dev/doc/comment)。

## Names

命名在Go中与在任何其他语言中一样重要。
它们甚至具有语义效果：变量在包外的可见性取决于它的第一个字符是否为大写。
因此，值得花一点时间讨论Go程序中的命名约定。

### Package names

导入包时，包名成为其内容的访问器。
在

```
import "bytes"
```

之后，可以使用 bytes.Buffer。
如果每个使用包的人都可以使用相同的名称来引用，这将很有帮助，这意味着包名称应该是好的：简短、简洁、令人回味。
按照惯例，包名被赋予小写的单词；不需要下划线或混合大写字母。
首先是要简洁，因为每个使用您的包的人都会输入该名称。
并且不要担心先验证其是否冲突。
包名只是导入的默认名称；它不需要在所有源代码中都是唯一的，并且在极少数发生冲突的情况下，导入包可以选择不同的名称以在本地使用。
无论如何，冲突很少见，因为导入中的文件名决定了正在使用哪个包。

另一个约定是包名是其源目录的名称；
src/encoding/base64 被导入为"encoding/base64"其名称为base64，而不是encoding_base64或encodingBase64。

包的导入器将使用包名来引用其内容，因此包中的导出名称可以使用该事实来避免重复。
（不要使用 import . 符号，它可以在测试包时使得一些测试变简单，但在其他方面应该避免。）
例如，bufio包中的缓冲读取器类型称为Reader，而不是BufReader，因为用户将其视为bufio.Reader，这是一个简洁明了的名称。
此外，由于导入的实体总是以其包名来寻址，因此bufio.Reader不会与io.Reader冲突。
类似地，创建新实例的函数ring.Ring——Go中构造函数的定义——一般会被叫做NewRing，但是因为Ring是该包唯一的导出类型，
并且该包被称为ring，所以它只是被叫做New，使用该包的客户端将其视为ring.New。
使用包结构来帮助您选择好名称。

另一个简短的例子是once.Do；once.Do(setup)读起来很好，也不会因为被叫做once.DoOrWaitUntilDone(setup)而看起来更好。
长名称不会自动使内容更具可读性。
有用的文档注释通常比超长的名称更有价值。

### Getters

Go不提供对getters和setters的自动支持。
自己提供getters和setters并没有什么问题，而且这样做通常是合适的，但是将Get放入getter的名称中既不是惯用的也不是必需的。
如果您有一个名为owner（小写，未导出）的字段，则相应的getter应为方法Owner（大写，已导出），而不是GetOwner。
使用大写名称进行导出已经提供了将字段与方法区分开来的钩子。
如果需要对应的setter函数，名称应为SetOwner。这两个名字在实践中都很好读：

```
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

### Interface names

按照惯例，单方法接口由方法名称加上-er后缀或类似修饰来命名，以构造代理名词：Reader，Writer，Formatter，CloseNotifier等。

有许多这样的名称，尊重它们以及它们存储的函数名称是很有用的。
Read，Write，Close，Flush，String等具有规范的签名和含义。
为避免混淆，除非它具有相同的签名和含义，否则不要为您的方法提供这些名称之一。
相反，如果您的类型实现了与知名类型上的方法具有相同含义的方法，请为其赋予相同的名称和签名；将你的字符串转换方法称为String而不是ToString。

### MixedCaps

最后，Go中的约定是使用MixedCaps或者mixedCaps而不是下划线来编写多单词名称。

## Semicolons

与C一样，Go使用分号来终止语句，但与C不同的是，这些分号不会出现在源代码中。
词法分析器使用一个简单的规则在扫描时自动插入分号，因此输入文本中大部分没有分号。

规则是这样的。
如果换行符之前的最后一个标记是标识符（包括int和float64之类的词）、基本字符（例如数字或字符串常量）或下面标记中的一个

```
break continue fallthrough return ++ -- ) }
```

词法分析器总是在标记后插入一个分号。
可以概括为"如果换行符出现在可以结束语句的标记之后，则插入分号"。

分号也可以在右大括号之前省略，所以像这样的语句

```
   go func() { for { dst <- <-src } }()
```

不需要分号。
一般Go程序仅在诸如for循环子句之类的地方使用分号来分隔初始值设定项、条件和延续元素。
如果您以这种方式编写代码，还必须在一行中分隔多个语句。

分号插入规则的一个后果是您不能将控制结构的左大括号（if、for、switch 或 select）放在下一行。
如果这样做，将在大括号前插入分号，这可能会导致不良影响。
你应该像这样编写

```
if i < f() {
    g()
}
```

而不是这样

```
if i < f()  // wrong!
{           // wrong!
    g()
}
```

## Control structures

Go的控制结构与C的控制结构相似，但在一些方面有所不同。
没有do或者while循环，只有一个有点概括性质的for；switch变得更灵活；if和switch接受一个可选的初始化语句，如for;
break和continue语句采用可选标签来标识要中断或继续的内容；并且有新的控制结构，包括type switch和多路通信复用器，select。
语法也略有不同：没有括号，正文必须始终用大括号分隔。

### If

在Go中，一个简单的if看起来像这样：

```
if x > 0 {
    return y
}
```

强制大括号鼓励if在多行上编写简单的语句。
无论如何，这样做是一种很好的风格，尤其是当主体包含控制语句时，例如return或者break。

因为if和switch可以接受一个初始化语句，通常会被用来设置局部变量。

```
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在Go库中，您会发现当一条if语句没有流入下一条语句时——即if主体以break、continue、goto 或return结尾——else语句就可以被省略了。

```
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

这是代码必须防范一系列错误条件的常见情况的示例。
如果成功语句顺着页面向下运行，则代码可读性很好，从而消除了可能出现的错误情况。
由于错误情况往往以return语句结尾，因此生成的代码不需要else语句。

```
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### Redeclaration and reassignment

顺便说一句：上一节中的最后一个示例展示了:=短声明的工作原理。
调用os.Open如下：

```
f, err := os.Open(name)
```

该语句声明了两个变量，f和err。
几行之后，调用f.Stat读取，

```
d, err := f.Stat()
```

看起来好像它声明了d和err。
但是请注意，err在这两个语句都出现了。
这种重复是合法的：err由第一个语句声明，在第二个语句中重新分配。
这意味着f.Stat使用上面声明的现有err变量，并给它一个新值。

在:=声明中，可能会出现已经声明过了的变量v，前提是：

- 此声明与现有声明在同一范围内（如果 v 已在外部范围内声明，则该声明将创建一个新变量 §），
- 初始化中有相应值可分配给 v，并且
- 该声明至少创建了一个其他变量。

这种不寻常的特性是纯粹的实用主义，这样使用单个err值变得容易，例如，在长if-else链中。
你会看到它经常被使用。

§ 这里值得注意的是，在Go中，函数参数和返回值的作用域与函数体相同，即使它们在词法上出现在包围函数体的大括号之外。

### For

Go中for循环类似于——但不同于——C的循环。
它结合了for和while并且没有do-while。
共有三种形式，其中只有一种有分号。

```
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

短声明让在循环中声明索引变量变得更加容易。

```
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

如果您正在循环遍历array、slice、string或map，或者从channel读取，range子句可以管理循环。

```
for key, value := range oldMap {
    newMap[key] = value
}
```

如果您在range中只需要第一项（键或索引），请删除第二项：

```
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

如果您在range中只需要第二项（值），请使用空白标识符（_）来丢弃第一项：

```
sum := 0
for _, value := range array {
    sum += value
}
```

空白标识符有很多用途，会在[后面部分](https://go.dev/doc/effective_go#blank)讲述。

对于字符串，range可以为您做更多的工作，通过解析UTF-8来分解单个Unicode码点。
错误的编码消耗一个字节用于生成替换符文U+FFFD。
（名称（带有关联的内置类型）rune是Go术语，用于单个Unicode码点。
请参阅[语言规范](https://go.dev/ref/spec#Rune_literals)获取更多相关信息。）
循环

```
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```

打印

```
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```

最后，Go没有逗号运算符，++和--是语句而不是表达式。
因此，如果您想在for中运行多个变量，应该使用并行赋值（尽管这排除了++和--）。

```
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

### Switch

Go中的switch比C中的更通用。
表达式不必是常量或整数，case会从上到下进行评估，直到找到匹配项，如果switch没有表达式，则选择true。
因此，有可能——而且是惯用的——将if-else-if-else写为switch。

```
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

没有自动fall through，但case可以使用逗号分隔的列表。

```
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

尽管它们在Go中不像其他一些类似C的语言那样常见，但break语句可用于提前终止switch。
有时，有必要跳出周围的循环，而不是switch，在Go中，这可以通过在循环前放置一个标签并"breaking"该标签来完成。
此示例显示了这两种用法。

```
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

当然，continue语句也接受一个可选标签，但它仅适用于循环。

为了结束本节，这里有一个使用两个switch语句的字节切片比较程序：

```
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

### Type switch

switch也可用于判断interface变量的动态类型。
这样的type switch使用括号内带有关键字的类型断言的语法。
如果switch在表达式中声明了一个变量，则该变量将在每个子句中具有相应的类型。
在这种情况下重用名称也是惯用的，实际上是在每种情况下声明一个具有相同名称但类型不同的新变量。

```
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

## Functions

### Multiple return values

Go一个不同寻常的特性是函数和方法可以返回多个值。
这种形式可用于改进C程序中的几个笨拙的习惯用法：把错误绑定到返回参数，例如-1代表EOF和修改的由地址传递的参数。

在C中，write错误由负数表示，错误代码隐藏在不易察觉的位置。
在Go中，write可以返回一个数字和一个错误："是的，你写了一些字节，但不是全部，因为你的设备满了"。
os包中Write方法的签名是：

```
func (file *File) Write(b []byte) (n int, err error)
```

正如文档所说，当n!=len(b)时它返回写入的字节数和非nil的error。
这是一种常见的风格；有关更多示例，请参见错误处理部分。

类似这样的方法不需要传递一个指针并将返回参数绑定到这个指针上。
这是一个简单的函数，用于从字节切片中的某个位置获取一个数字，返回该数字和下一个位置。

```
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

您可以使用它来扫描输入slice b中的数字，如下所示：

```
    for i := 0; i < len(b); {
        x, i = nextInt(b, i)
        fmt.Println(x)
    }
```

### Named result parameters

Go函数的返回或结果"parameters"可以指定名称并用作常规变量，就像传入参数一样。
使用命名返回参数时，它们在函数开始时会被初始化为对应类型的零值；如果函数执行return返回不带参数的语句，则使用返回参数的当前值作为返回值。

这些名称不是强制性的，但它们可以使代码更短更清晰：它们是文档。
如果我们为nextInt命名返回参数，返回的是哪个int就很明显了。

```
func nextInt(b []byte, pos int) (value, nextPos int) {
```

因为命名返回参数被初始化并被返回，所以它们可以简单和清晰。
这是一个使用 io.ReadFull 还不错的版本：

```
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### Defer

Go的defer语句会使一个函数调用（延迟函数）在执行defer的函数返回之前立即运行。
这是一种不寻常但有效的方式来处理类似无论函数采用哪条路径返回都必须释放资源的情况。
典型示例是解锁互斥锁或关闭文件。

```
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

推迟对类似Close函数的调用有两个优点。
首先，它保证您永远不会忘记关闭文件，如果您稍后编辑该函数以添加新的返回路径，则很容易犯这个错误。
其次，这意味着close位于open附近，这比将其放在函数的末尾要清楚得多。

Defer函数（如果函数是方法，则包括接收者）的参数在*defer*时赋值，而不是在*call*时赋值。
避免了在函数执行时变量会发生变化的担心，这意味着延迟调用可以延迟在多个函数后执行。
这是一个愚蠢的例子。

```
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

延迟函数按LIFO顺序执行，因此在函数返回时此代码会打印出4 3 2 1 0。
一个更合理的例子是通过执行trace函数。
我们可以编写几个简单的跟踪程序，如下所示：

```
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

我们可以通过利用延迟函数的参数在defer执行时赋值这一事实来做得更好。
trace的返回值可以作为untrace函数的参数。例如：

```
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

打印如下

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

对于习惯于使用block-level进行资源管理的程序员来说，defer可能看起来很奇怪，但它最有趣和最强大的地方恰恰来自于它不是基于块而是基于函数。
在panic和recover章节中，我们将看到它的另一个例子。

## Data

### Allocation with new

Go有两个分配语句，内置函数new和make。
它们用来做不同的事情并适用于不同的情景，这可能会令人困惑，但规则很简单。
先来说说new。
它是一个分配内存的内置函数，但与其他一些语言中的同名函数不同，它不会*初始化*内存，它只会将其*归零*。
也就是说，new(T)为T类型的新项目分配为零的内存并返回其地址，即类型为T的值*T。
使用Go术语，它返回一个指向类型为T的重新分配为零值的指针。

由于new返回的内存为零，在设计数据结构时可以使用每种类型的零值而无需进一步初始化，这是很有帮助的。
这意味着用户可以通过new创建一个数据结构并开始工作。
例如，bytes.Buffer说明 "Buffer的零值是一个可以使用的空缓冲区"。
同样，sync.Mutex没有显式构造函数或Init方法。
取而代之的是，零值被sync.Mutex定义为未锁定的互斥体。

Zero-value-is-useful的属性是可传递的。
考虑这种类型声明。

```
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

SyncedBuffer类型的值也可以在new或声明后立即使用。
在下面，p和v无需进一步赋值都可以正常工作。

```
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### Constructors and composite literals

有时零值不够好，初始化构造函数是必要的，正如从os包衍生出来的下例。

```
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

这里有很多赋值。
我们可以使用复合字面量来简化它，这是一个在每次赋值时都会创建一个新实例的表达式。

```
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

请注意，与C不同，返回局部变量的地址是完全可以的；变量在函数返回后仍然存在。
事实上，每次获取复合字面量的地址并赋值时都会分配一个新实例，因此我们可以将最后两行组合起来。

```
    return &File{fd, name, nil, 0}
```

复合字面量的字段要按顺序排列，且必须全部存在。
但是将元素显式标记为*field:value*对，字段可以按任何顺序出现，缺少的元素则为对应类型的零值。
因此我们可以这样写：

```
    return &File{fd: fd, name: name}
```

作为一种极限情况，如果复合字面量根本不包含任何字段，它会为该类型创建一个零值。
表达式new(File)和&File{}是等价的。

为array、slice和map创建复合字面量，字段标签是索引或映射键。
在这些示例中，无论Enone，Eio和Einval的值如何，只要字段标签不同，初始化就会起作用。

```
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

### Allocation with make

回到分配。
内置函数make(T, args)的用途不同于new(T)。
它只用来创建slice、map和channel，并返回*已初始化的*（非零）T（非*T）类型的值。
不一样的原因是这三种类型必须要在使用前初始化其引用的数据结构。
例如，slice是一个three-item描述符，包括指向数据的指针（在数组内）、长度和容量，在这些项目被初始化之前，slice为nil。
对于slice、map和channel，要先初始化其引用的内部数据结构才能使用。
例如，

```
make([]int, 10, 100)
```

创建一个包含100个整数的array，然后创建一个长度为10、容量为100的slice，指向数组的前10个元素。
（创建slice时，可以省略容量；请参阅slices部分获取更多信息。）
相反，new([]int)返回指向新分配的切片零值的指针，即指向值为nil的指针。

下面这些例子说明了new和make之间的区别。

```
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

请记住，make仅适用于map、slice和channel，并且不返回指针。
要显式获得指针，请使用new或显式获取变量的地址。

### Arrays

Arrays在规划内存的详细布局时很有用，有时可以用来避免分配，但它们主要用于构建slices，下一节的主题。
为了奠定该主题的基础，这里有几句话是关于数组的。

数组在Go和C中的工作方式之间存在重大差异。
在Go中，

- 数组是值。将一个数组分配给另一个会复制所有元素。
- 特别是，如果将数组传递给函数，它将收到数组的副本，而不是指向它的指针。
- 数组的大小是其类型的一部分。类型[10]int和[20]int是不同的。

值属性很有用但也很昂贵；如果你想要类似C的行为和效率，你可以传递一个指向数组的指针。

```
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```

但这种不是Go常用的风格。
请使用切片代替。

### Slices

Slices将数组包装，为数据序列提供更通用、更强大、更方便的接口。
除了具有显式维度的项（例如转换矩阵）之外，Go中的大多数数组编程都是使用切片而不是数组完成的。

Slices是对数组的引用，如果将一个切片赋值给另一个，两者都引用同一个数组。
如果函数的参数是切片类型，它对切片元素所做的更改对调用者可见的，类似于传递底层数组的指针。
因此Read函数可以接受一个切片参数，而不是一个指针和一个计数; 切片的长度设置了读取数据量的上限。
这是os包中File类型的Read方法的签名：

```
func (f *File) Read(buf []byte) (n int, err error)
```

该方法返回读取的字节数和错误值（如果有）。
为了读入一个较大缓存buf的前32个字节，分割缓存。

```
    n, err := f.Read(buf[0:32])
```

这种分割是常见且有效的。
事实上，暂时不考虑效率，下面的代码也会读取缓存的前32个字节。

```
    var n int
    var err error
    for i := 0; i < 32; i++ {
        nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
        n += nbytes
        if nbytes == 0 || e != nil {
            err = e
            break
        }
    }
```

Slice的长度可以更改，只要它符合底层数组的限制；只是分配自身的一部分给切片。
切片的容量，可通过内置函数cap查询，会返回切片可以使用的最大长度。
这里有一个将数据追加到切片的函数。
如果数据超出容量，则重新分配切片。
最后返回切片。
该函数使用了len和cap在应用于nil切片时是合法的特性，返回0。

```
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

我们必须返回切片，尽管Append可以修改slice的元素，但切片本身（持有指针、长度和容量的run-time数据结构）是按值传递的。

追加数据到切片的想法非常有用，内置函数append因此而出现。
但是，要了解该函数的设计，我们需要更多信息，所以稍后再讨论。

### Two-dimensional slices

Go的数组和切片是一维的。
要创建2D数组或切片，需要定义一个array-of-arrays或slice-of-slices，如下所示：

```
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

因为切片的长度是可变的，所以可以让每个内部切片具有不同的长度。
这是一种常见情况如LinesOfText所示：每行都有不同的长度。

```
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

有时分配2D切片是必要的，例如，处理像素扫描线时可能出现的情况。
有两种方法可以实现这一点。
一种是独立分配每个slice；另一种是分配一个array并将slice放入其中。
使用哪个取决于您的应用。
如果切片可能会增长或缩小，则应独立分配它们以避免覆盖下一行；如果没有，使用单次分配来构造会更有效。
作为参考，这里是这两种方法的简单实现。
首先，一次一行：

```
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

现在是分配一次，并将其填入：

```
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

### Maps

Maps是一种方便且强大的内置数据结构，它将一种类型的值（键）与另一种类型的值（元素或值）相关联。
键可以是可以使用相等运算符的任何类型，例如integers，floating point和complex
numbers，strings，pointers，interfaces（只要动态类型支持比较）、structs和arrays。
Slices不能用作映射键，因为切片不能进行比较。
像切片一样，maps是对底层数据结构的引用。
如果您将map传递给函数并更改map中的内容，这些更改在调用者中是可见的。

Maps可以使用带有冒号分隔的键值对的复合字面量语法来构造，因此在初始化期间很容易构建它们。

```
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

设置和获取映射值在语法上看起来就像对数组和切片做的一样，只是索引不需要是整数。

```
offset := timeZone["EST"]
```

尝试使用映射中不存在的键获取映射值将返回映射中条目类型的零值。
例如，如果值为整数的映射，则查找不存在的键时将返回0。
集合可以通过值为bool的映射来实现。
将映射的值设置true，然后通过简单的索引对其进行测试。

```
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

有时您需要将缺失的值与零值区分开来。
是否存在一个值为0的键"UTC"或者因为它根本不在映射中所以为0？
您可以通过多值赋值的形式来进行区分。

```
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

显而易见，这被称为“comma ok”。
在这个例子中，如果tz存在，seconds将被赋值且ok为 true；如果不存在，seconds则为0且ok为false。
这是一个利用这个规则进行错误报告的函数：

```
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

只是测试映射中键的存在而不考虑实际值，可以使用[空白标识符](https://go.dev/doc/effective_go#blank)（_）作为值的变量。

```
_, present := timeZone[tz]
```

删除映射中的条目，请使用内置函数delete，其参数是映射和要删除的键。
即使映射中不存在该键，这样做也是安全的。

```
delete(timeZone, "PDT")  // Now on Standard Time
```

### Printing

Go中的格式化打印类似于C家族，但更丰富、更通用。
这些函数在fmt包中，并具有大写的名称：fmt.Printf，fmt.Fprintf，fmt.Sprintf等。
字符串函数（Sprintf等）返回一个字符串，而不是填充一个提供的缓冲区。

格式字符串不是必须的。
对于Printf，Fprintf和Sprintf中的每一个都还有另一对与之对应的函数，例如Print和Println。
这些函数不使用格式字符串，而是为每个参数生成默认格式。
Println还在参数之间插入一个空格并在输出中附加一个换行符，Print版本仅在两边的操作数都不是字符串时才添加空格。
下面示例中，每一行都产生相同的输出。

```
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

格式化的打印函数fmt.Fprint和其小伙伴们将实现了io.Writer接口的任何对象作为第一个参数；变量os.Stdout和os.Stderr是常见的例子。

从这里开始，与C变得有些不一样了。
首先，例如%d等数字格式不带符号或大小的标志；取而代之的是，打印线程以参数的类型来决定这些。

```
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

打印如下

```
18446744073709551615 ffffffffffffffff; -1 -1
```

如果只想要默认转换，例如整数的十进制，您可以使用通用格式%v（用于 “value”）；结果正是Print和Println会产生的结果。
此外，该格式可以打印*任何*值，甚至是数组、切片、结构体和映射。
这是打印上一节中定义的timeZone映射的语句。

```
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

输出如下：

```
map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

对于映射，Printf和其小伙伴们，将键按字典顺序排序并输出。

对于结构体，%+v会用名称注释结构体的字段，对于任意值，%#v格式会以完整的Go语法打印。

```
type T struct {
    a int
    b float64
    c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
```

输出如下：

```
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
```

（注意&符号。）
对于字符串或[]byte类型的值，可以通过%q输出带引号的字符串。
如果可以的话，%#q将使用反引号。（%q也适用于integer和rune，生成单引号rune常量。）
此外，%x适用于字符串、字节数组和字节切片以及整数，生成十六进制字符串，（% x）允许字节之间填充空格。

另一种常用的格式是%T，打印出值的类型。

```
fmt.Printf("%T\n", timeZone)
```

输出如下：

```
map[string]int
```

如果您想控制自定义类型的默认格式，只需定义一个签名为String() string的方法即可。
对于T类型，看起来可能像这样。

```
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

格式化打印如下：

```
7/-2.35/"abc\tdef"
```

（如果您需要打印T的类型的值以及指向T的指针，则接收器String必须是值类型；此示例使用指针，因为这对于结构体类型更有效且更惯用。
参阅[pointers vs. value receivers](https://go.dev/doc/effective_go#pointers_vs_values)以获取更多信息。）

因为打印线程是完全可重入的，所以Sprintf可以调用我们的String方法。
然而，关于这种方法有一个重要的细节需要理解：通过调用Sprintf来构造String方法，有可能会使你的String方法陷入死循环。
如果Sprintf调用尝试将接收器直接打印为字符串，则可能会发生这种情况，这反过来又会再次调用该方法。
如本例所示，这是一个常见且容易犯的错误。

```
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

也很容易修复：将参数转换为基本字符串类型，因为它没有相应的方法。

```
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

在[初始化](https://go.dev/doc/effective_go#initialization)部分，我们将看到另一种避免这种递归的技术。

另一种打印技术是将打印线程的参数直接传递给另一个这样的线程。
Printf的签名使用类型...interface{}作为其最后的参数，以指定任意数量的参数（任意类型）可以出现在format之后。

```
func Printf(format string, v ...interface{}) (n int, err error) {
```

在函数Printf中，v类似于 []interface{} 类型的变量，但如果将其传递给另一个可变参数函数，则它又类似于常规的参数列表。
下面是我们上面使用的函数log.Println的实现。
它将其参数直接传递给fmt.Sprintln以进行格式化。

```
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

我们在对Sprintln的嵌套调用中在v之后使用...来告诉编译器将v视为参数列表；否则它只会将v作为单个切片参数传递。

有关打印的内容比我们在这里介绍的还要多。
有关详细信息，请参阅fmt包的文档。

顺便说一句，...参数可以是特定类型，例如用于选择最小整数列表的min函数使用...int：

```
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // largest int
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```

### Append

现在我们开始解释内置函数append的设计的缺失部分。
append的签名与我们上面自定义的Append函数不同。
从示意图上看，它是这样的：

```
func append(slice []T, elements ...T) []T
```

其中T是任何给定类型的占位符。
您实际上不能在Go中编写由调用者确定的T类型的函数。
这是内置append的原因：它需要编译器的支持。

append所做的是将元素附加到切片的末尾并返回结果。
需要返回结果，因为与我们手写的Append一样，底层数组可能会发生变化。
这个简单的例子：

```
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

输出为[1 2 3 4 5 6]。
所以append有点像Printf，参数可以是任意数量。

但是，如果我们想做Append所做的事情并将切片附加到切片怎么办？
简单：调用时使用...，就像我们在上面对Output的调用中所做的那样。
下面代码段拥有与上面相同的输出。

```
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

如果没有...，就不会通过编译，因为它的类型是错误的；y不是int类型。

## Initialization

虽然表面上看起来与C或C++中的初始化没有太大区别，但Go中的初始化更强大。
可以在初始化期间构建复杂的结构体，并且可以正确处理初始化对象之间甚至不同包之间的排序问题。

### Constants

常量在Go中就只是常量。
它们是在编译时创建的，即使在函数中定义的，而且只能是numbers，characters (runes)，strings或booleans。
由于编译时的限制，定义它们的表达式必须是常量表达式，可以由编译器计算。
例如，1<<3是一个常量表达式，而math.Sin(math.Pi/4)不是，因为math.Sin需要在运行时发才能调用、。

在Go中，可以使用iota创建枚举常量。
由于iota是表达式的一部分，并且可以隐式重复，所以用来构建复杂的集合比较容易。

```
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

将诸如String之类的方法添加到任何用户定义的类型使得其可以格式化打印。
尽管它最常应用于结构体，但这种技术对于数量类型（如ByteSize等浮点类型）也很有用。

```
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

表达式YB打印为1.00YB，而ByteSize(1e13)打印为9.09TB。

这里使用Sprintf来实现ByteSize的String方法是安全的（要避免无限重复），不是因为类型转换，而是因为它使用%f调用Sprintf，这不是字符串格式：
Sprintf只会在字符串时调用String方法，%f需要一个浮点值。

### Variables

变量可以像常量一样被初始化，其表达式是在运行时可以计算的通用表达式。

```
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

### The init function

最后，每个源文件都可以定义自己的无参数init函数来设置所需的任何状态。（实际上每个文件可以有多个init函数。）
最后：当包中级别的变量都赋值后且所有导入的包都初始化后才调用init。

除了初始化那些不能使用声明的变量外，init函数的一个常见用途是在实际执行开始之前修复或验证程序状态的正确性。

```
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## Methods

### Pointers vs. Values

正如我们在ByteSize中看到的，可以为任何命名类型（指针或接口除外）定义方法；接收者不必是结构体。

在上面对切片的讨论中，我们编写了一个Append函数。
我们可以将其定义为切片上的方法。
为此，我们首先声明一个可以将方法绑定到的命名类型，然后将方法的接收者设为该类型的值。

```
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```

这仍然需要方法返回更新后的切片。
我们可以通过重新定义该方法将指向ByteSlice的指针作为其接收者来消除这种繁琐，该方法可以覆盖调用者的切片。

```
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

事实上，我们可以做得更好。
如果我们修改我们的函数，让它看起来像一个标准的Write方法，像这样：

```
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

那么*ByteSlice类型满足标准接口io.Writer，这很方便。
例如，我们可以像下面这样打印。

```
    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

我们传递ByteSlice的地址，因为只有*ByteSlice满足io.Writer。
接收者是指针或值的规则是：值方法可以在指针和值上调用，但指针方法只能在指针上调用。

出现这个规则是因为指针方法可以修改接收者；在一个值上调用它们会导致该方法接收该值的副本，因此任何修改都将被丢弃。
因此，Go不允许这种错误。
不过，有一个为了简便的特例。
当值是可寻址的时，Go通过自动插入地址运算符来处理对值调用指针方法的情形。
在我们的示例中，变量b是可寻址的，因此我们可以只使用b.Write调用它的Write方法。
编译器会为我们重写为(&b).Write。

顺便说一句，bytes.Buffer实现的核心是在字节切片上使用Write。

## Interfaces and other types

### Interfaces

Go中的接口提供了一种指定对象行为的方法：如果有东西可以做到这些，那么它就可以在这里使用。
我们已经看过几个简单的例子；自定义打印可以通过String方法实现，而Fprintf可以使用Write方法生成任何输出。
只有一两个方法的接口在Go代码中很常见，并且通常被赋予方法的名称，例如io.Writer可以是任何实现Write方法的东西。

一个类型可以实现多个接口。
例如，如果一个集合实现了sort.Interface，它可以通过sort包中的线程进行排序，其中包含Len()、Less(i, j int) bool和Swap(i, j int)
，它还可以有自定义格式。
在这个例子中，Sequence满足了这两者。

```
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Copy returns a copy of the Sequence.
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    s = s.Copy() // Make a copy; don't overwrite argument.
    sort.Sort(s)
    str := "["
    for i, elem := range s { // Loop is O(N²); will fix that in next example.
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

### Conversions

Sequence的String方法正在重构原先Sprint已经为切片所做的工作。（复杂性还为 O(N²) ）
如果我们在调用Sprint之前将Sequence转换为普通的[]int，我们可以分担工作（并加快速度）。

```
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

此方法是自定义String方法中安全调用Sprintf方法的另一个示例。
因为如果我们忽略类型名称，这两种类型（Sequence和[]int）是相同的，所以在它们之间进行转换是合法的。
转换不会创建新值，它只是暂时充当现有值具有新类型的作用。
（还有其他合法的转换，例如从整数到浮点，确实会创建一个新值。）

转换表达式的类型以访问不同的方法集是Go程序中的一个习惯用法。
例如，我们可以使用现有类型sort.IntSlice将整个示例简化为：

```
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

现在，我们不是让Sequence实现多个接口（排序和打印），而是让数据项转换为多种类型（Sequence、sort.IntSlice和[]int），每个类型都执行某些部分工作。
这在实践中不常见，但是有效。

### Interface conversions and type assertions

[Type switches](https://go.dev/doc/effective_go#type_switch)是一种转换形式：一个接口类型的值，尝试将其转换为case下的类型。
这是fmt.Printf下的代码如何使用type switch将值转换为字符串的简化版本。
如果它已经是一个字符串，我们想要接口保存的实际字符串值，而如果它有一个String方法，我们想要调用该方法的结果。

```
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

第一个case找到具体值；第二个将接口转换为另一个接口。
以这种方式混合类型非常好。

如果我们只关心一种类型怎么办？
如果我们知道该值包含一个字符串并且我们只想提取它？
单个case的type switch是可以的，*type assertion*也可以。
Type assertion接受接口值并从中提取指定显式类型的值。
该语法借用了type switch 的子句，但使用了显式类型而不是type关键字：

```
value.(typeName)
```

结果是静态类型typeName的值。
该类型必须是此value持有的具体类型，或者是value可以转换为的第二个接口类型。
要提取我们知道值是字符串，我们可以这样写：

```
str := value.(string)
```

但如果该值不是字符串，则程序将因运行时错误而崩溃。
为了防止这种情况，请使用 "comma, ok" 来安全地测试该值是否为字符串：

```
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

如果类型断言失败，str仍然存在并且是字符串类型，但它是零值，一个空字符串。

作为该功能的说明，这里有一个if-else语句，它等效于本节开始的type switch。

```
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

### Generality

如果一个类型只是为了实现一个接口并且永远不会在该接口之外导出方法，则不需要导出该类型本身。
仅导出接口就可以清楚地表明该值没有超出接口中描述的行为。
它还避免了对通用方法的每个实例重复书写文档。

在这种情况下，构造函数应该返回一个接口而不是具体实现方法的类型。
例如，在哈希库中crc32.NewIEEE和adler32.New都返回接口类型hash.Hash32。
在Go程序中用CRC-32算法替换Adler-32只需要更改构造函数调用即可；其余代码不受算法更改的影响。

类似的方法允许将各种加密包中的流密码算法与它们链接在一起的块密码分开。
Crypto/cipher包中的Block interface指定了分组密码的行为，它提供了对单个数据块的加密。
那么，类比bufio包，实现该接口的cipher包可以用来构造流式密码，以Stream interface为代表，无需了解分组加密的细节。

Crypto/cipher interface如下所示：

```
type Block interface {
    BlockSize() int
    Encrypt(dst, src []byte)
    Decrypt(dst, src []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}
```

这是计数器模式 (CTR) 流的定义，它将分组密码转换为流密码；请注意，分组密码的详细信息已被抽象掉：

```
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```

NewCTR不仅适用于一种特定的加密算法和数据源，还适用于 Block interface 和任何 Stream 的任何实现。
因为它们返回接口值，所以用其他加密模式替换CTR加密是本地化的变化。
必须编辑构造函数调用，但由于周围的代码必须仅将结果视为Stream，因此不会注意到差异。

### Interfaces and methods

几乎所有东西都可以附加方法，因此几乎任何东西都可以满足接口。
一个说明性示例是在http包中，它定义了Handler接口。
任何实现Handler的对象都可以服务于HTTP请求。

```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

ResponseWriter本身就是一个接口，它提供对将响应返回给客户端所需的方法的访问。
这些方法包括标准的Write方法，因此可以在任何可以使用io.Writer的地方使用http.ResponseWriter。
Request是一个结构体，其中包含来自客户端的请求的解析表示。

为简洁起见，让我们忽略POST并假设HTTP请求始终是GET；这种简化不会影响处理程序的设置方式。
这是一个处理程序的简单实现，用于计算页面被访问的次数。

```
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

（与我们的主题保持一致，请注意Fprintf如何打印到http.ResponseWriter。）
在真实服务器中，对ctr.n的访问需要防止并发访问。
请参阅sync和atomic包以获取建议。

作为参考，以下是如何将此类服务器附加到URL树上的节点。

```
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

但是为什么要让Counter成为一个结构体呢？
这儿需要的只是一个整数。
（接收者需要是一个指针，以便调用者可以看到增量）

```
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

如果您的程序有一些内部状态需要通知页面已被访问怎么办？
将channel绑定到网页。

```
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

最后，假设我们想在/args上显示调用服务器二进制文件时使用的参数。
编写一个打印参数的函数很容易。

```
func ArgServer() {
    fmt.Println(os.Args)
}
```

我们如何把它变成一个HTTP服务器？
我们可以让ArgServer成为我们忽略其值的某种类型的方法，但有一种更简洁的方法。
由于我们可以为除指针和接口之外的任何类型定义方法，因此我们可以为函数编写方法。
Http包中包含以下代码：

```
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

HandlerFunc具有方法ServeHTTP，因此该类型的值可以服务于HTTP请求。
看方法的实现：接收者是一个函数，f，方法内部调用f。
这可能看起来很奇怪，但它与接收器是一个channel和在channel上发送的方法没有什么不同。

为了使ArgServer成为HTTP服务器，我们首先将其修改为具有正确的签名。

```
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

ArgServer现在与HandlerFunc具有相同的签名，因此可以将其转换类型以访问其方法，就像我们将Sequence转换为IntSlice以访问IntSlice.Sort一样。
设置它的代码很简洁：

```
http.Handle("/args", http.HandlerFunc(ArgServer))
```

当有人访问/args页面时，安装在该页面上的处理程序具有值ArgServer并键入HandlerFunc。
HTTP服务器将调用方法ServeHTTP，以ArgServer作为接收者，接收者将依次调用ArgServer（通过HandlerFunc.ServeHTTP调用f(w, req)）。
参数将会随之显示。

在本节中，我们从一个结构体、一个整数、一个通道和一个函数创建了一个HTTP服务器，所有这些都是。
因为接口只是一组方法，可以为（几乎）任何类型定义。

## The blank identifier

在[for](https://go.dev/doc/effective_go#for)和[maps](https://go.dev/doc/effective_go#maps)的上下文中，我们已经多次提到空白标识符。
可以使用任何类型的任何值分配或声明空白标识符，并且可以无害地丢弃该值。
这有点像写入Unix /dev/null文件：它表示一个只写值，用作需要变量但实际值无关的占位符。
它的用途超出了我们已经看到的范围。

### The blank identifier in multiple assignment

在for range循环中使用空白标识符一般是为了：多重赋值。

如果赋值需要左侧有多个值，但其中一个值不会被程序使用，则赋值左侧的空白标识符避免了创建虚拟变量的需要，并清楚地表明该值将被丢弃。
例如，当调用返回值和错误但只有错误重要的函数时，使用空白标识符丢弃不相关的值。

```
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```

有时您会看到丢弃错误值以忽略错误的代码；这是可怕的做法。
始终检查错误返回；提供它们是有原因的。

```
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### Unused imports and variables

导入包或声明变量而不使用它是错误的。
未使用的导入会使程序膨胀并降低编译速度，而已初始化但未使用的变量至少是浪费的计算，并且可能表明存在更大的错误。
然而，当一个程序正在积极开发中时，经常会出现未使用的导入和变量，并且为了继续编译而删除它们可能很烦人，只是为了以后再次需要它们。
空白标识符提供了一种解决方法。

这个写了一半的程序有两个未使用的导入（fmt和io）和一个未使用的变量（fd），所以它不会编译，但很高兴看看到目前为止的代码是否正确。

```
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

要消除有关未使用导入的编译失败，请使用空白标识符来引用导入包中的符号。
类似地，将未使用的变量fd分配给空白标识符将使未使用的变量错误消失。
这个版本的程序可以通过编译。

```
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

按照惯例，消除导入错误的全局声明应在导入之后立即出现并进行注释，以使它们易于查找并提醒以后清理。

### Import for side effect

最终应该使用或删除前面示例中未使用的导入（如 fmt 或 io）：空白分配将代码标识为正在进行的工作。
但有时导入一个包只是为了它的副作用而不显式使用是有用的。
例如，在它的init函数中，net/http/pprof包注册了提供调试信息的HTTP处理程序。
它有一个导出的API，但大多数客户端只需要处理程序注册并通过网页访问数据。
要仅出于副作用导入包，请将包重命名为空白标识符：

```
import _ "net/http/pprof"
```

这种形式的导入清楚地表明，因为它的副作用，所以导入该包，该包没有其他可能的用途：在这个文件中，它没有名称。
（如果是这样，并且我们没有使用该名称，编译器将拒绝该程序。）

### Interface checks

正如我们在上面对[接口](https://go.dev/doc/effective_go#interfaces_and_types)的讨论中看到的，一个类型不需要明确声明它实现了一个接口。
相反，类型仅通过实现接口的方法来实现接口。
实际上，大多数接口转换都是静态的，因此在编译时进行检查。
例如，将*os.File传递给需要io.Reader的函数将无法编译，除非*os.File实现了io.Reader接口。

不过，一些接口检查确实发生在运行时。
一个实例在[encoding/json](https://go.dev/pkg/encoding/json/)
包中，它定义了一个[Marshaler](https://go.dev/pkg/encoding/json/#Marshaler) 接口。
当JSON编码器接收到实现该接口的值时，编码器会调用该值的编组方法将其转换为JSON，而不是执行标准转换。
编码器在运行时使用如下[type assertion](https://go.dev/doc/effective_go#interface_conversions)检查此属性：

```
m, ok := val.(json.Marshaler)
```

如果只需要询问类型是否实现了接口，而不实际使用接口本身，可能作为错误检查的一部分，请使用空白标识符来忽略类型断言的值：

```
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

这种情况出现的一个地方是当有必要在包中保证它实际满足接口的类型时。
如果一个类型，例如[json.RawMessage](https://go.dev/pkg/encoding/json/#RawMessage)
需要自定义JSON表示，它应该实现json.Marshaler，但没有静态转换会导致编译器自动验证这一点。
如果类型无意中无法满足接口，JSON编码器仍然可以工作，但不会使用自定义实现。
为保证实现正确，可以在包中使用使用空白标识符的全局声明：

```
var _ json.Marshaler = (*RawMessage)(nil)
```

在此声明中，涉及将*RawMessage转换为Marshaler的赋值要求*RawMessage实现 Marshaler，并且将在编译时检查该属性。
如果json.Marshaler接口发生变化，这个包将不再编译，我们会注意到它需要更新。

此构造中出现空白标识符表明该声明仅用于类型检查，而不是用于创建变量。
但是，不要对满足接口的每种类型都这样做。
按照惯例，此类声明仅在代码中不存在静态转换时使用，这是一种罕见的情况。

## Embedding

Go不提供典型的、类型驱动的子类化概念，但它确实能够通过在结构或接口中嵌入类型来实现。

接口嵌入非常简单。
我们之前提到过io.Reader和io.Writer接口；这是他们的定义。

```
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

io包还导出了几个其他接口，这些接口指定了可以实现多个此类方法的对象。
例如，有io.ReadWriter，一个包含Read和Write的接口。
我们可以通过显式列出这两个方法来指定io.ReadWriter，但是嵌入这两个接口以形成新的接口更容易也更令人回味，如下所示：

```
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

这说明：ReadWriter可以做Reader和Writer所做的事情；它是嵌入式接口的联合。
只有接口可以嵌入到接口中。

相同的基本思想适用于结构，但具有更深远的影响。
bufio包有两种结构类型，bufio.Reader和 bufio.Writer，它们当然都实现了包io中的类似接口。
并且bufio还实现了一个缓冲的读取器/写入器，它通过使用嵌入将读取器和写入器组合到一个结构中来实现：它列出了结构中的类型，但不给它们字段名称。

```
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

嵌入的元素是指向结构的指针，当然必须先初始化以指向有效的结构，然后才能使用它们。
ReadWriter结构可以写成

```
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

但是为了提升字段的方法并满足io接口，我们还需要提供转发方法，如下所示：

```
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

通过直接嵌入结构，我们避免了这种记账。
嵌入式类型的方法是免费的，也就是说bufio.ReadWriter不仅有bufio.Reader和bufio.Writer的方法，它还满足io.Reader、io.Writer、io.ReadWriter这三个接口。

嵌入与子类化有一个重要的区别。
当我们嵌入一个类型时，该类型的方法成为外部类型的方法，但是当它们被调用时，方法的接收者是内部类型，而不是外部类型。
在我们的示例中，当调用bufio.ReadWriter的Read方法时，它的效果与上面写的转发方法完全相同；接收者是ReadWriter的reader字段，而不是ReadWriter本身。

嵌入也可以很方便。
此示例表示了一个嵌入字段以及一个常规的命名字段。

```
type Job struct {
    Command string
    *log.Logger
}
```

Job类型现在具有*log.Logger的Print、Printf、Println和其他方法。
当然，我们可以给Logger一个字段名称，但没有必要这样做。
现在，一旦初始化，我们就可以登录到Job：

```
job.Println("starting now...")
```

Logger是Job结构体的常规字段，因此我们可以在Job的构造函数中以通常的方式对其进行初始化，如下所示，

```
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

或使用复合字面量，

```
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

如果我们需要直接引用嵌入字段，则忽略包限定符的字段类型名称用作字段名称，就像在我们的ReadWriter结构的Read方法中所做的那样。
在这里，如果我们需要访问Job变量job的*log.Logger，我们将编写job.Logger，如果我们想改进Logger的方法，这将很有用。

```
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

嵌入类型引入了名称冲突的问题，但解决它们的规则很简单。
首先，字段或方法X将任何其他X隐藏在类型的更深嵌套部分中。
如果log.Logger包含一个名为Command的字段或方法，则Job的Command字段会占据这个名字。

其次，如果同名出现在同一个嵌套层级，通常是报错；如果Job结构包含另一个名为Logger的字段或方法，则嵌入log.Logger将是错误的。
但是，如果在类型定义之外的程序中从不提及重复名称，则没有问题。
此限定提供了一些保护，防止对从外部嵌入的类型进行更改；如果添加的字段与另一个子类型中的另一个字段冲突，如果两个字段都未使用过，则没有问题。

## Concurrency

### Share by communicating

并发编程是一个很大的话题，这里有一些属于Go的亮点。

由于实现对共享变量的正确访问所需的微妙之处，许多环境中的并发编程变得困难。
Go鼓励一种不同的方法，在这种方法中，共享值在channel上传递，实际上，从不由单独的执行线程主动共享。
在任何给定时间，只有一个goroutine可以访问该值。
设计上不会发生数据竞争。
为了鼓励这种思维方式，我们将其简化为一个口号：

Do not communicate by sharing memory; instead, share memory by communicating.

这种方法可能走得太远。
例如，引用计数最好通过在整数变量周围放置互斥锁来完成。
但是作为一种高级方法，使用通道来控制访问可以更容易地编写清晰、正确的程序。

考虑此模型的一种方法是考虑在一个CPU上运行的典型单线程程序。
它不需要同步原语。
现在运行另一个这样的实例；它也不需要同步。
现在让这两个交流；如果通信是同步器，则仍然不需要其他同步。
例如，Unix管道就非常适合这个模型。
尽管Go的并发方法起源于Hoare的通信顺序进程 (CSP)，但它也可以看作是Unix管道的类型安全泛化。

### Goroutines

称为goroutines因为现有的术语——threads、coroutines、processes等——传达了不准确的含义。
Goroutine有一个简单的模型：它是一个在同一地址空间中与其他goroutine并发执行的函数。
它是轻量级的，成本仅比堆栈空间的分配多一点。
堆栈开始时很小，因此它们很便宜，并且通过根据需要分配（和释放）堆存储来增长。

Goroutine被多路复用到多个OS线程上，因此如果一个线程阻塞，例如在等待I/O时，其他线程会继续运行。
他们的设计隐藏了线程创建和管理的许多复杂性。

使用go关键字为函数或方法调用添加前缀，以在新的goroutine中运行调用。
调用完成后，goroutine静默退出。（效果类似于在后台运行命令的Unix shell的&符号。）

```
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

函数字面量在goroutine调用中很方便。

```
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

在Go中，函数字面量是闭包：实现确保函数引用的变量只要它们处于活动状态就可以存活。

这些示例不太实用，因为这些函数无法发出完成信号。
为此，我们需要 channel。

### Channels

和maps一样，channels用make初始化，是对底层数据结构的引用。
如果提供了可选的整数参数，它会设置通道的缓冲区大小。
对于无缓冲或同步通道，默认为0。

```
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

无缓冲通道将通信（值的交换）与同步相结合，确保两个计算（goroutines）处于已知状态。

有很多使用channel的好习惯。
让我们开始第一个。
在上一节中，我们在后台启动了排序。
Channel可以允许启动goroutine等待排序完成。

```
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

接收器总是阻塞，直到有数据要接收。
如果通道没有缓冲，发送方会阻塞，直到接收方收到该值。
如果通道有缓冲区，发送方只会阻塞，直到值被复制到缓冲区；如果缓冲区已满，这意味着要等到某个接收器检索到一个值。

缓冲通道可以像信号量一样使用，例如限制吞吐量。
在这个例子中，传入的请求被传递给 handle，它向通道发送一个值，处理请求，然后从通道接收一个值，为下一个消费者准备“semaphore”。
通道缓冲区的容量限制了同时处理的调用数。

```
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```

一旦MaxOutstanding处理程序正在执行进程，任何其他处理程序都将阻止尝试发送到填充的通道缓冲区，直到现有处理程序之一完成并从缓冲区接收。

但是，这种设计有一个问题：Serve为每个传入请求创建一个新的goroutine，即使它们中只有MaxOutstanding可以随时运行。
因此，如果请求来得太快，程序可能会消耗无限的资源。
我们可以通过改变Serve来控制goroutines的创建来解决这个缺陷。
这是一个明显的解决方案，但要注意它有一个我们将在随后修复的错误：

```
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```

错误在于，在for循环中，循环变量在每次迭代中都被重用，因此req变量在所有goroutine之间共享。
这不是我们想要的。
我们需要确保req对于每个goroutine都是唯一的。
这是一种方法，将req的值作为参数传递给goroutine中的闭包：

```
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```

将此版本与上一个版本进行比较，以了解闭包的声明和运行方式的差异。
另一种解决方案是创建一个具有相同名称的新变量，如下例所示：

```
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // Create new instance of req for the goroutine.
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```

写起来可能很奇怪

```
req := req
```

但在Go中这样做是合法且惯用的。
你会得到一个新版本的同名变量，故意在本地隐藏循环变量，但在每个goroutine中都是唯一的。

回到编写服务器的一般问题，另一种很好地管理资源的方法是启动固定数量的句柄goroutine，它们都从请求通道中读取。
Goroutine的数量限制了同时调用处理的数量。
这个Serve函数还接受一个将被告知退出的通道；启动goroutine后，它会阻止从该通道接收。

```
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

### Channels of channels

Go最重要的属性之一是channel，是一个可以像其他任何东西一样分配和传递的一流值。
此属性的一个常见用途是实现安全的并行解复用。

在上一节的示例中，handle是一个理想化的请求处理程序，但我们没有定义它正在处理的类型。
如果该类型包含要回复的通道，则每个客户端都可以提供自己的回复路径。
这是Request类型的示意图定义。

```
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

客户端提供了一个函数及其参数，以及请求对象内的一个通道，用于接收答案。

```
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

在服务器端，处理函数是唯一发生变化的东西。

```
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

显然还有很多工作要做才能使它成为现实，但这段代码是一个限速、并行、非阻塞RPC系统的框架，而且看不到互斥锁。

### Parallelization

这些想法的另一个应用是跨多个CPU内核并行计算。
如果计算可以分解为可以独立执行的独立部分，则可以并行化，并在每个部分完成时使用通道发出信号。

假设我们对项目向量执行昂贵的操作，并且每个项目的操作值是独立的，就像在这个理想化的例子中一样。

```
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```

我们在一个循环中独立启动这些片段，每个CPU一个。
他们可以按任何顺序完成，但这没关系；我们只是在启动所有goroutine后通过排空通道来计算完成信号。

```
const numCPU = 4 // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

我们可以询问运行时什么值是合适的，而不是为numCPU创建一个常量值。
[runtime.NumCPU](https://go.dev/pkg/runtime#NumCPU)函数返回机器中硬件CPU内核的数量，所以我们可以这样写

```
var numCPU = runtime.NumCPU()
```

还有一个函数[runtime.GOMAXPROCS](https://go.dev/pkg/runtime#GOMAXPROCS)，它报告（或设置）Go程序可以同时运行的用户指定的内核数量。
它默认为runtime.NumCPU的值，但可以通过设置类似命名的shell环境变量或使用正数调用函数来覆盖。
用零调用它只是查询值。
因此，如果我们想尊重用户的资源请求，我们应该写

```
var numCPU = runtime.GOMAXPROCS(0)
```

确保不要混淆concurrency（将程序构造为独立执行的组件）和parallelism（在多个 CPU 上并行执行计算以提高效率）的概念。
尽管Go的并发特性可以使一些问题易于构建为并行计算，但Go是一种并发语言，而不是并行语言，并且并非所有并行化问题都适合Go的模型。
有关区别的讨论，请参阅[此博客文章](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)。

### A leaky buffer

并发编程的工具甚至可以让非并发的想法更容易表达。
这是一个从RPC包中抽象出来的示例。
客户端goroutine循环从某个源（可能是网络）接收数据。
为了避免分配和释放缓冲区，它保留了一个空闲列表，并使用一个缓冲通道来表示它。
如果通道为空，则分配一个新缓冲区。
一旦消息缓冲区准备好，它就会发送到serverChan上的服务器。

```
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

服务器循环接收来自客户端的每条消息，对其进行处理，并将缓冲区返回到空闲列表。

```
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

客户端尝试从freeList中检索缓冲区；如果没有可用，它会分配一个新的。
服务器发送到freeList会将b放回空闲列表，除非列表已满，在这种情况下，缓冲区会被丢弃在地板上，由垃圾收集器回收。
（选择语句中的默认子句在没有其他情况准备好时执行，这意味着选择永远不会阻塞。）
这个实现只用几行就构建了一个漏桶空闲列表，依靠缓冲通道和垃圾收集器进行记账。

## Errors

Library routines必须经常向调用者返回某种错误指示。
如前所述，Go的多值返回使得在返回正常返回值的同时返回详细的错误描述变得很容易。
使用此功能提供详细的错误信息是一种很好的方式。
例如，正如我们将看到的，os.Open不仅在失败时返回一个nil指针，它还返回一个错误值来描述发生了什么错误。

按照惯例，errors有error类型，一个简单的内置接口。

```
type error interface {
    Error() string
}
```

库编写者可以在后台使用更丰富的模型自由地实现此接口，从而不仅可以查看错误，还可以提供一些上下文。
如前所述，除了通常的*os.File返回值之外，os.Open 还返回一个错误值。
如果文件打开成功，则error为nil，但是当出现问题时，它会持有一个os.PathError：

```
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

PathError's Error生成一个这样的字符串：

```
open /etc/passwx: no such file or directory
```

这样的错误，包括有问题的文件名、操作和它触发的操作系统错误，即使在远离导致它的调用的地方打印也是有用的；它比简单的 "no such
file or directory"提供更多信息。

在可行的情况下，错误字符串应标识其来源，例如通过使用前缀命名生成错误的操作或包。
例如，在包图像中，由于未知格式导致的解码错误的字符串表示为 "image: unknown format"。

关心精确错误详细信息的调用者可以使用类型开关或类型断言来查找特定错误并提取详细信息。
对于PathErrors，这可能包括检查内部Err字段是否存在可恢复的故障。

```
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

这里的第二个if语句是另一种 [type assertion](https://go.dev/doc/effective_go#interface_conversions)。
如果失败，ok将为false，e将为nil。
如果成功，ok将为true，这意味着错误的类型是*os.PathError，然后e也是，我们可以检查它以获取有关错误的更多信息。

### Panic

向调用者报告错误的常用方法是将错误作为额外的返回值返回。
Read方法是一个众所周知的例子。
它返回一个字节数和一个错误。
但是，如果错误无法恢复怎么办？
有时程序根本无法继续。

为此，有一个内置函数panic，它实际上会创建一个运行时错误，从而停止程序（但请参阅下一节）。
该函数采用任意类型的单个参数（通常是字符串）在程序终止时打印。
这也是一种表示不可能发生的事情的方法，例如退出无限循环。

```
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

这只是一个例子，但真正的库函数应该避免恐慌。
如果问题可以被掩盖或解决，最好让事情继续运行而不是取消整个程序。
一个可能的反例是在初始化期间：如果库真的无法设置自己，那么恐慌可能是合理的，可以这么说。

```
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

### Recover

当panic被调用时，或运行时错误时，例如索引切片越界或类型断言失败，它会立即停止当前函数的执行并展开goroutine的堆栈，任何被延迟的函数，如果到达堆栈的顶部，程序就会终止。
但是，可以使用内置函数recover重新获得对goroutine的控制并恢复正常执行。

调用recover会停止展开并返回传递给panic的参数。
因为在展开时运行的唯一代码是在延迟函数内部，recover仅在延迟函数内部有用。

recover的一个应用是关闭服务器内失败的goroutine而不杀死其他正在执行的goroutine。

```
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

在这个例子中，如果do(work)出现恐慌，结果将被记录下来，goroutine将干净地退出而不会打扰其他人。
在延迟关闭中不需要做任何其他事情；调用recover可以完全处理这种情况。

因为除非直接从延迟函数调用，否则恢复总是返回nil，延迟代码可以调用库例程，这些库例程本身使用恐慌并恢复而不会失败。
例如，safelyDo中的延迟函数可能会在调用恢复之前调用日志记录函数，并且该日志记录代码将不受恐慌状态的影响。

有了我们的恢复模式，do函数（以及它调用的任何东西）可以通过调用panic来干净利落地摆脱任何糟糕的情况。
我们可以使用这个想法来简化复杂软件中的错误处理。
让我们看一个regexp包的理想化版本，它通过使用本地错误类型调用panic来报告解析错误。
下面是Error定义、Error方法和Compile函数。

```
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

如果doParse发生恐慌，恢复块会将返回值设置为nil——延迟函数可以修改命名的返回值。
然后它将在对err的赋值中通过断言它具有本地类型Error来检查问题是否是解析错误。
如果没有，类型断言将失败，导致运行时错误继续堆栈展开，就好像没有任何东西中断它一样。
此检查意味着如果发生意外情况，例如索引越界，即使我们使用恐慌和恢复来处理解析错误，代码也会失败。

有了错误处理，错误方法（因为它是一个绑定到类型的方法，很自然，因为它与内置错误类型具有相同的名称）使得报告解析错误变得容易，而不必担心展开手动解析堆栈：

```
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

尽管这种模式很有用，但它只能在包中使用。
Parse将其内部恐慌调用转换为错误值；它不会向其客户公开恐慌。
这是一个很好的规则。

顺便说一句，如果发生实际错误，这个re-panic习惯用法会更改panic值。
但是，原始故障和新故障都将显示在崩溃报告中，因此问题的根本原因仍然可见。
因此，这种简单的重新恐慌方法通常就足够了——毕竟这是一个崩溃——但如果你只想显示原始值，你可以编写更多代码来过滤意外问题并重新恐慌原始错误。
这留给读者作为练习。

## A web server

让我们完成一个完整的Go程序，一个Web服务器。
这实际上是一种网络重新服务器。
Google在chart.apis.google.com上提供了一项服务，可以将数据自动格式化为图表和图形。
但是，它很难以交互方式使用，因为您需要将数据作为查询放入URL。
这里的程序为一种形式的数据提供了一个更好的接口：给定一小段文本，它调用图表服务器来生成一个二维码，一个对文本进行编码的框矩阵。
该图像可以用您手机的摄像头抓取并解释为例如URL，而无需您在手机的小键盘上输入URL。

这是完整的程序。
下面是一个解释。

```
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET">
    <input maxLength=1024 size=70 name=s value="" title="Text to QR Encode">
    <input type=submit value="Show QR" name=qr>
</form>
</body>
</html>
`
```

主要的部分应该很容易理解。
one标志为我们的服务器设置默认的HTTP端口。
模板变量templ是有趣的地方。
它构建了一个HTML模板，将由服务器执行以显示页面；稍后再详细介绍。

主函数解析标志，并使用我们上面讨论的机制，将函数QR绑定到服务器的根路径。
然后调用http.ListenAndServe启动服务器；它在服务器运行时阻塞。

QR只是接收到包含表单数据的请求，并对名为s的表单值中的数据执行模板。

模板包html/template功能强大；该程序仅涉及其功能。
本质上，它通过替换从传递给templ.Execute的数据项派生的元素（在本例中为表单值）来动态重写一段HTML文本。
在模板文本(templateStr)中，双大括号分隔的部分表示模板操作。
从{{if .}}到{{end}}的部分仅在当前数据项的值（称为 . （点），是非空的。）
即当字符串为空时，这块模板被抑制。

两个片段{{.}}表示在网页上显示呈现给模板的数据（查询字符串）。
HTML模板包会自动提供适当的转义，以便安全地显示文本。

模板字符串的其余部分只是页面加载时显示的HTML。
如果这解释得太快，请参阅模板包的[文档](https://go.dev/pkg/html/template/)以进行更深入的讨论。

你就拥有了：几行代码加上一些数据驱动的 HTML 文本，就成了一个有用的 Web 服务器。
Go足够强大，可以在几行代码中完成很多事情。