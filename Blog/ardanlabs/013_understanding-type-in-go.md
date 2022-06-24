# 了解 Go 的类型

> [Understanding Type in Go](https://www.ardanlabs.com/blog/2013/07/understanding-type-in-go.html)


当我使用 C/C++ 进行编写代码时，必须理解类型。如果你不这样做，你会在编译器和运行代码方面遇到很多麻烦。不管是哪种语言，类型都涉及到编程语法的各个方面。在编程中，对类型和指针的良好理解至关重要。这篇文章将重点介绍类型。

作为初学者，记住这些内存字节：

| FFE4     | FFE3     | FFE2     | FFE1     |
|----------|----------|----------|----------|
| 00000000 | 11001011 | 01100101 | 00001010 |

地址 FFE1 的字节值是多少？如果你试图回答这个问题，那你就错了。为什么，因为我没有告诉你那个字节代表什么。我没有给你类型信息。

如果我说同一个字节代表一个数字怎么办？你的答案可能是 10，你又错了。为什么，因为当我说它是一个数字时，你假设是一个以 10 进制的数字。

```
Number Bases:

All numbering systems have a base that they function within. Since you were
a baby you were taught to count in base 10. This may be due to the fact that
most of us have 10 fingers and 10 toes. Also, it seems natural to perform
math in base 10.

Base defines the number of symbols a numbering system contains. In base 10
there are 10 distinct symbols we use to represent the infinite number of
things we can count. In base 10 the symbols are 0, 1, 2, 3, 4, 5, 6, 7, 8, 9.
Once we reach the symbol 9 we need to grow the length of the number. As an
example, 10, 100 and 1000.

There are two other bases we use all the time in computing. Base 2 or binary
numbers, such as the bits represented in the diagram above. Base 16 or
hexadecimal numbers, such as the addresses represented in the diagram above.

In a binary numbering system (base 2), there are only 2 symbols and those
symbols are 0 and 1.

In a hexadecimal number system (base 16), there are 16 symbols and those
symbols are 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F.

If there were apples sitting on a table, those apples could be represented
in any numbering system. We could say there are:

In Base 2:  10010001 apples
In Base 10: 145 apples
In Base 16: 91 apples

All of those answers are correct when given the correct base.

Notice the number of symbols required in each numbering system to represent
those apples. The larger the base, the more efficient the numbering system.

Using base 16 for computer addresses, IP addresses and color codes makes
a lot of sense now.

Look at the number for the HTML color white in all three bases:

In Base 2:  1111 1111 1111 1111 1111 1111 (24 characters)
In Base 10: 16777215 (10 characters)
In Base 16: FFFFFF (6 characters)

Which numbering system would you have chosen to represent colors?
```

现在，如果我告诉你地址 FFE1 的字节代表一个 10 进制的数字，那么你的答案 10 是正确的。

Type 提供了编译器和你所需的两条信息。

1. 要查看的内存量，以字节为单位
2. 这些字节的具体值

Go 语言提供了这些基础数字类型：

```
Unsigned Integers
uint8, uint16, uint32, uint64

Signed Integers
int8, int16, int32, int64

Real Numbers
float32, float64

Predeclared Integers
uint, int, uintptr
```

这些关键字的名称提供了类型的这两种信息。

uint8 包含一个 10 进制的数字，使用 1 个字节的内存。该值可以在 0 到 255 之间。

int32 包含一个 10 进制的数字，使用 4 个字节的内存。该值可以在 -2147483648 到 2147483647 之间。

Predeclared Integers 根据构建代码的平台来进行映射。在 64 位操作系统上，int 将映射到 int64，而在 32 位操作系统上，它将映射到 int32。

存储在内存中的所有内容都返回这些数字类型之中的一个。Go 中的字符串只是一系列的 uint8 类型，其规则将这些字节串在一起并识别字符串的结尾。

Go 中的指针是 uintptr 类型。同样，基于操作系统架构，这将是 uint32 或 uint64。Go 为指针创建了一种特殊类型，这很好。在过去，许多 C
程序员会编写代码，假设指针值总是无符号整数。随着时间的推移对语言和架构的升级，这最终不再适用。由于地址变得比预先声明的无符号整数大，很多代码都崩溃了。

结构体类型只是已知类型（最终解析为数字类型）的组合。

```
type Example struct{
    BoolValue bool
    IntValue  int16
    FloatValue float32
}
```

这个结构体代表了一个复杂类型。它用三种不同的数字类型表示 7 个字节。bool 为 1 个字节，int16 为 2 个字节，float32 为 4 个字节。但是，该结构体实际上在内存中分配了 8 个字节。

所有内存都是基于对齐边界进行分配的，以最大限度地减少内存碎片整理。要确定 Go 用于你的架构的对齐边界，你可以运行 unsafe.Alignof 函数。Go 中 64 位 Darwin 平台的对齐边界是 8 个字节。因此，当 Go
确定结构体的内存分配时，它将填充字节以确保最终的内存占用是 8 的倍数。编译器将确定在何处添加填充。

如果你想了解有关结构成员对齐和填充的更多信息，请查看此链接：

http://www.geeksforgeeks.org/structure-member-alignment-padding-and-data-packing/

这个程序显示了 Go 对 Example 结构体的填充情况：

```
package main

import (
    "fmt"
    "unsafe"
)

type Example struct {
    BoolValue bool
    IntValue int16
    FloatValue float32
}

func main() {
    example := &Example{
        BoolValue:  true,
        IntValue:   10,
        FloatValue: 3.141592,
    }

    exampleNext := &Example{
        BoolValue:  true,
        IntValue:   10,
        FloatValue: 3.141592,
    }

    alignmentBoundary := unsafe.Alignof(example)

    sizeBool := unsafe.Sizeof(example.BoolValue)
    offsetBool := unsafe.Offsetof(example.BoolValue)

    sizeInt := unsafe.Sizeof(example.IntValue)
    offsetInt := unsafe.Offsetof(example.IntValue)

    sizeFloat := unsafe.Sizeof(example.FloatValue)
    offsetFloat := unsafe.Offsetof(example.FloatValue)

    sizeBoolNext := unsafe.Sizeof(exampleNext.BoolValue)
    offsetBoolNext := unsafe.Offsetof(exampleNext.BoolValue)

    fmt.Printf("Alignment Boundary: %d\n", alignmentBoundary)

    fmt.Printf("BoolValue = Size: %d Offset: %d Addr: %v\n",
        sizeBool, offsetBool, &example.BoolValue)

    fmt.Printf("IntValue = Size: %d Offset: %d Addr: %v\n",
        sizeInt, offsetInt, &example.IntValue)

    fmt.Printf("FloatValue = Size: %d Offset: %d Addr: %v\n",
        sizeFloat, offsetFloat, &example.FloatValue)

    fmt.Printf("Next = Size: %d Offset: %d Addr: %v\n",
        sizeBoolNext, offsetBoolNext, &exampleNext.BoolValue)
}
```

输出：

```
Alignment Boundary: 8
BoolValue  = Size: 1  Offset: 0  Addr: 0x21015b018
IntValue   = Size: 2  Offset: 2  Addr: 0x21015b01a
FloatValue = Size: 4  Offset: 4  Addr: 0x21015b01c
Next       = Size: 1  Offset: 0  Addr: 0x21015b020
```

正如预期的那样，结构体的对齐边界是 8 个字节。

Size 是该字段被读取和写入的内存量。正如预期的那样，Size 是类型信息的一部分。

Offset 是在内存中找到该字段开头的字节数。

Addr 是在内存中找到该字段开头的位置。

我们可以看到 Go 在 BoolValue 和 IntValue 字段之间填充了 1 个字节。两个地址之间的偏移值的差为 2 个字节。你还可以看到，下一次内存分配开始于 4 字节的位置，这是结构体的最后一个字段。

让我们通过仅在结构体中保留字节数为 1 的 bool 字段来证明 8 字节对齐规则：

```
package main

import (
    "fmt"
    "unsafe"
)

type Example struct {
    BoolValue bool
}

func main() {
    example := &Example{
        BoolValue:  true,
    }

    exampleNext := &Example{
        BoolValue:  true,
    }

    alignmentBoundary := unsafe.Alignof(example)

    sizeBool := unsafe.Sizeof(example.BoolValue)
    offsetBool := unsafe.Offsetof(example.BoolValue)

    sizeBoolNext := unsafe.Sizeof(exampleNext.BoolValue)
    offsetBoolNext := unsafe.Offsetof(exampleNext.BoolValue)

    fmt.Printf("Alignment Boundary: %d\n", alignmentBoundary)

    fmt.Printf("BoolValue = Size: %d Offset: %d Addr: %v\n",
        sizeBool, offsetBool, &example.BoolValue)

    fmt.Printf("Next = Size: %d Offset: %d Addr: %v\n",
        sizeBoolNext, offsetBoolNext, &exampleNext.BoolValue)
}
```

和输出：

```
Alignment Boundary: 8
BoolValue = Size: 1 Offset: 0 Addr: 0x21015b018
Next      = Size: 1 Offset: 0 Addr: 0x21015b020
```

将这两个地址相减，你将看到在这两个结构体之间有 8 个字节差。此外，下一次内存分配从第一个示例的相同地址开始。Go 将 7 个字节填充到结构体中以保持对齐边界。

不管填充是什么样子，size 真正代表了我们可以为该字段读取和写入的内存量。

我们只能在使用数字类型时操作内存，而赋值运算符 (=) 就是我们这样做的方式。为了让我们的生活更轻松，Go
创建了一些直接支持赋值运算符的复杂类型。其中一些类型是字符串、数组和切片。要查看这些类型的完整列表，请查看此文档 http://golang.org/ref/spec#Types。

这些复杂类型抽象了可以在每个实现中找到的数字类型的操作。这样做，这些复杂类型可以像数字类型一样直接读写内存。

Go 是一种类型安全的语言。这意味着编译器将始终强制在赋值运算符的两侧校验类型是否可以相比较。这非常重要，因为它可以防止我们错误地读取或写入内存。

想象一下，如果我们能做到以下几点。如果你尝试编译此代码，你将收到错误消息。

```
type Example struct{
    BoolValue bool
    IntValue  int16
    FloatValue float32
}

example := &Example{
    BoolValue:  true,
    IntValue:   10,
    FloatValue: 3.141592,
}

var pointer *int32
pointer = *int32(&example.IntValue)
*pointer = 20
```

我要做的是获取 2 字节的 IntValue 字段的内存地址并将其存储在 int32 类型的指针中。然后我尝试使用指针将 4 字节整数写入该内存地址。如果我能够使用该指针，我将违反 IntValue 字段的类型规则并在此过程中破坏内存。

| FFE8 | FFE7 | FFE6 | FFE5 | FFE4 | FFE3 | FFE2 | FFE1 | | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | | 0
| 0 | 0 | 3.14 | 0 | 10 | 0 | true |

| pointer |
|:-------:|
|  FFE3   |

| FFE8 | FFE7 | FFE6 | FFE5 | FFE4 | FFE3 | FFE2 | FFE1 | | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | | 0
| 0 | 0 | 0 | 0 | 20 | 0 | true |

根据上面的内存占用，指针将在 FFE3 和 FFE6 之间的 4 个字节中写入值 20。IntValue 的值将如预期的那样为 20，但 FloatValue 的值现在为
0。想象一下，如果写入的这些字节超出了该结构体的分配内存并开始破坏应用程序其他区域的内存时，随之而来的错误将显得随机且不可预测。

Go 编译器将始终确保分配内存和强制转换是安全的。

在这个转换示例中，编译器会抱怨：

```
package main

import (
    "fmt"
)

// Create a new type
type int32Ext int32

func main() {
    // Cast the number 10 to a value of type Jill
    var jill int32Ext = 10

    // Assign the value of jill to jack
    // ** cannot use jill (type int32Ext) as type int32 in assignment **
    var jack int32 = jill

    // Assign the value of jill to jack by casting
    // ** the compiler is happy **
    var jack int32 = int32(jill)

    fmt.Printf("%d\n", jack)
}
```

首先，我们在系统中创建一个名为 int32Ext 的新类型，并告诉编译器这个新类型代表一个单独的 int32。接下来我们创建一个名为 jill 的新变量并赋值 10。编译器允许赋值，因为数值类型位于赋值运算符的右侧。编译器知道赋值是安全的。

现在我们尝试创建一个名为 jack 的 int32 类型的第二个变量，并赋值 jill 变量。这里编译器抛出一个错误：

"cannot use jill (type int32Ext) as type int32 in assignment"

编译器认为 jill 是 int32Ext 类型并且对赋值的安全性不做任何假设。

现在我们使用强制转换，编译器允许赋值并按预期打印值。当我们执行强制转换时，编译器会检查赋值的安全性。在我们的例子中，他的标识符是相同的类型并允许赋值。

这对你们中的一些人来说似乎是基本的东西，但它是使用任何编程语言工作的基础。即使类型是抽象的，因为你在操纵内存所以应该理解你在做什么。

有了这个基础，我们接下来就可以讨论 Go 中的指针以及将参数传递给函数。

和往常一样，我希望这篇文章能帮助你了解一些你可能不知道的事情。