# Gustavo 的 IEEE-754 脑筋急转弯

> [Gustavo's IEEE-754 Brain Teaser](https://www.ardanlabs.com/blog/2013/08/gustavos-ieee-754-brain-teaser.html)


早在 6 月，Gustavo Niemeyer 在他的Labix.org博客上发布了以下问题：

假设 uf 是一个 64 位的无符号整数，它包含 IEEE-754 表示的二进制浮点数。如何判断 uf
是否代表整数？我不能为你说话，但我编写业务应用程序。我只是没有背景来快速敲出这样一个问题的答案。幸运的是，古斯塔沃发布了答案的逻辑。我认为更好地理解这个问题并分解他的答案会很有趣。我将使用 32
位数字，只是为了让一切变得更容易。IEEE-754 标准如何以二进制格式存储浮点数？首先，我发现了这两个页面：

http://steve.hollasch.net/cgindex/coding/ieeefloat.html
http://class.ece.iastate.edu/arun/CprE281_F05/ieee754/ie5.html

IEEE-754 规范使用特殊的二进制格式表示以 2 为底的科学计数法的浮点数。如果你不知道我所说的 base 2
是什么意思，那么请查看我关于理解类型输入的帖子（https://www.ardanlabs.com/blog/2013/07/understanding-type-in​​-go.html）。

科学记数法是书写非常大或非常小的数字的一种有效方式。它通过使用带有乘数的十进制格式来工作。这里有些例子：

| Base 10 Number | Scientific Notation | Calculation   | Coefficient | Base | Exponent | Mantissa |
|----------------|---------------------|---------------|-------------|:----:|:--------:|:--------:|
| 700            | 7e+2                | 7 * 102       | 7           |  10  |    2     |    0     |
| 4,900,000,000  | 4.9e+9              | 4.9 * 109     | 4.9         |  10  |    9     |    .9    |
| 5362.63        | 5.36263e+3          | 5.36263 * 103 | 5.36263     |  10  |    3     |  .36263  |
| -0.00345       | 3.45e-3             | 3.45 * 10-3   | 3.45        |  10  |    -3    |   .45    |
| 0.085          | 1.36e-4             | 1.36 * 2-4    | 1.36        |  2   |    -4    |   .36    |

在正常的科学记数法形式中，小数点左侧总是只有一位数字。对于以 10 为底的数字，该数字必须介于 1 到 9 之间，对于以 2 为底的数字，该数字只能是 1。

科学计数法中的整个数字称为系数。尾数是小数点右边的所有数字。这些术语很重要，因此请花时间研究和理解上面的图表。

我们如何将小数点移动到第一个位置决定了指数的值。如果我们必须将小数点向左移动，指数是一个正数，向右移动，它是一个负数。查看上面的图表并查看每个示例的指数值。

Base 和 Exponent 在符号中一起工作。指数决定了我们需要在基础上执行的“Power Of”计算。在第一个示例中，数字 7 乘以 10（底数）乘以 2（指数）的幂，以返回原始以 10 为底的数字
700。我们将小数点向左移动两位以将 700 转换为 7.00 ，这使得指数 +2 并创建了 7e+2 的符号。

IEEE-754 标准不存储以 10 为底的科学记数法数字，而是以 2 为底的科学记数法数字。上图中的最后一个示例表示以 2 为底的科学计数法中以 10 为底的数字 0.085。让我们了解如何计算该符号。

| Base 10 Number | Scientific Notation | Calculation | Coefficient | Base | Exponent | Mantissa |
|----------------|---------------------|-------------|-------------|:----:|:--------:|:--------:|
| 0.085          | 1.36e-4             | 1.36 * 2-4  | 1.36        |  2   |    -4    |   .36    |

我们需要将基数为 10 的数字 (0.085) 除以 2 的某个幂，因此我们得到 1 + Fraction 值。1 + 分数值是什么意思？我们需要一个看起来像示例中的系数的数字，1 + .36。IEEE-754
标准要求我们有一个“1”。在系数中。这使我们只需要存储尾数并给我们额外的精度。

如果我们使用蛮力，您将看到我们最终获得 0.085 的 1 + 分数值：

0.085 / 2 -1 = 0.17 0.085 / 2 -2 = 0.34 0.085 / 2 -3 = 0.68 0.085 / 2 -4 = 1.36 **我们找到了 1 + 分数

-4 的指数为我们提供了所需的 1 + Fraction 值。现在我们拥有了以 IEEE-754 格式存储以 10 为底的数字 0.085 所需的一切。

让我们看看这些位在 IEEE-754 格式中是如何布局的。

| Precision        | Sign   | Exponent   | Fraction Bits | Bias |
|:-----------------|:-------|:-----------|:--------------|:-----|
| Single (32 Bits) | 1 [31] | 8 [30-23]  | 23 [22-00]    | 127  |
| Double (64 Bits) | 1 [63] | 11 [62-52] | 52 [51-00]    | 1023 |

这些位分为三个部分。符号位、指数位和称为分数位的位都保留了一些位。小数位是我们将尾数存储为二进制小数的地方。

如果我们使用单精度（32 位数字）存储 0.085 的值，IEEE-754 中的位模式将如下所示：

| Sign | Exponent (123) | Fraction Bits (.36)          |
|:-----|:---------------|:-----------------------------|
| 0    | 0111 1011      | 010 1110 0001 0100 0111 1011 |

最左边的符号位确定数字是正数还是负数。如果该位设置为 1，则该数字为负数，否则为正数。

接下来的 8 位代表指数。 在我们的示例中，以 10 为底的数字 0.085使用以 2 为底的科学计数法转换为 1.36 * 2 -4 。因此指数的值为-4。为了能够表示负数，有一个 Bias 值。我们的 32 位表示的偏差值为
127。要表示数字 -4，我们需要找到数字，当减去偏差时，我们得到 -4。在我们的例子中，这个数字是 123。如果您查看指数的位模式，您会看到它表示二进制数字 123。

剩下的 23 位是分数位。要计算小数位的位模式，我们需要对二进制小数进行计算和求和，直到我们得到尾数的值，或者一个尽可能接近的值。请记住，我们只需要存储尾数，因为我们总是假设“1”。存在价值。

要了解如何计算二进制分数，请查看下表。从左到右的每个位位置代表一个小数值：

| Binary | Fraction | Decimal | Power |
|:-------|:---------|:--------|:------|
| 0.1    | 1⁄2      | 0.5     | 2-1   |
| 0.01   | 1⁄4      | 0.25    | 2-2   |
| 0.001  | 1⁄8      | 0.125   | 2-3   |

我们需要设置正确数量的小数位数，这些位数加起来或使我们足够接近尾数。这就是为什么我们有时会失去精确度。

0**1**0 **111**0 000**1** 0**1**00 0**111** **1**0**11** = (0.36)

| Bit |  Value  | Fraction  |     Decimal      |      Total       |
|:---:|:-------:|:---------:|:----------------:|:----------------:|
|  2  |    4    |    1⁄4    |       0.25       |       0.25       |
|  4  |   16    |   1⁄16    |      0.0625      |      0.3125      |
|  5  |   32    |   1⁄32    |     0.03125      |     0.34375      |
|  6  |   64    |   1⁄64    |     0.015625     |     0.359375     |
| 11  |  2048   |  1⁄2048   |  0.00048828125   |  0.35986328125   |
| 13  |  8192   |  1⁄8192   | 0.0001220703125  | 0.3599853515625  |
| 17  | 131072  | 1⁄131072  | 0.00000762939453 | 0.35999298095703 |
| 18  | 262144  | 1⁄262144  | 0.00000381469727 | 0.3599967956543  |
| 19  | 524288  | 1⁄524288  | 0.00000190734863 | 0.35999870300293 |
| 20  | 1048576 | 1⁄1048576 | 0.00000095367432 | 0.35999965667725 |
| 22  | 4194304 | 1⁄4194304 | 0.00000023841858 | 0.35999989509583 |
| 23  | 8388608 | 1⁄8388608 | 0.00000011920929 | 0.36000001430512 |

您可以看到，设置这 12 位使我们得到 0.36 的值加上一些额外的分数。

让我们总结一下我们现在对 IEEE-754 格式的了解：

1. 任何要存储的以 10 为底的数字都转换为以 2 为底的科学计数法。
2. 我们使用的以 2 为底的科学计数值必须遵循 1 + Fraction 格式。
3. 格式中有三个不同的部分。
4. 符号位决定数字是正数还是负数。
5. 指数位代表一个需要减去偏差的数字。
6. 分数位使用二进制分数求和来表示尾数。

让我们证明我们对 IEEE-754 格式的分析是正确的。我们应该能够存储数字 0.85 并显示与我们所看到的所有内容相匹配的每个部分的位模式和值。

以下代码存储数字 0.085 并显示 IEEE-754 二进制表示：

```
package main

import (
    "fmt"
    "math"
)

func main() {
    var number float32 = 0.085

    fmt.Printf("Starting Number: %f\n\n", number)

    // Float32bits returns the IEEE 754 binary representation
    bits := math.Float32bits(number)

    binary := fmt.Sprintf("%.32b", bits)

    fmt.Printf("Bit Pattern: %s | %s %s | %s %s %s %s %s %s\n\n",
        binary[0:1],
        binary[1:5], binary[5:9],
        binary[9:12], binary[12:16], binary[16:20],
        binary[20:24], binary[24:28], binary[28:32])

    bias := 127
    sign := bits & (1 << 31)
    exponentRaw := int(bits >> 23)
    exponent := exponentRaw - bias

    var mantissa float64
    for index, bit := range binary[9:32] {
        if bit == 49 {
            position := index + 1
            bitValue := math.Pow(2, float64(position))
            fractional := 1 / bitValue

            mantissa = mantissa + fractional
        }
    }

    value := (1 + mantissa) * math.Pow(2, float64(exponent))

    fmt.Printf("Sign: %d Exponent: %d (%d) Mantissa: %f Value: %f\n\n",
        sign,
        exponentRaw,
        exponent,
        mantissa,
        value)
}
```

当我们运行程序时，我们得到以下输出：

```
Starting Number: 0.085000

Bit Pattern: 0 | 0111 1011 | 010 1110 0001 0100 0111 1011

Sign: 0 Exponent: 123 (-4) Mantissa: 0.360000 Value: 0.085000
```

如果您将显示的位模式与我们上面的示例进行比较，您会发现它匹配。我们学到的关于 IEEE-754 的一切都是真实的。

现在我们应该可以回答古斯塔沃的问题了。我们如何判断存储的值是否为整数？这是一个函数，感谢 Gustavo 的代码，它测试 IEEE-754 存储的值是否为整数：

```
func IsInt(bits uint32, bias int) {
    exponent := int(bits >> 23) - bias - 23
    coefficient := (bits & ((1 << 23) - 1)) | (1 << 23)
    intTest := (coefficient & (1 << uint32(-exponent) - 1))

    fmt.Printf("\nExponent: %d Coefficient: %d IntTest: %d\n",
        exponent,
        coefficient,
        intTest)

    if exponent < -23 {
        fmt.Printf("NOT INTEGER\n")
        return
    }

    if exponent < 0 && intTest != 0 {
        fmt.Printf("NOT INTEGER\n")
        return
    }

    fmt.Printf("INTEGER\n")
}
```

那么这个功能是如何工作的呢？

让我们从测试指数是否小于 -23 的第一个条件开始。如果我们使用数字 1 作为我们的测试数字，则指数将为 127，这与偏差相同。这意味着当我们将指数减去偏差时，我们将得到零。

```
Starting Number: 1.000000

Bit Pattern: 0 | 0111 1111 | 000 0000 0000 0000 0000 0000

Sign: 0 Exponent: 127 (0) Mantissa: 0.000000 Value: 1.000000


Exponent: -23 Coefficient: 8388608 IntTest: 0
INTEGER
```

测试函数增加了一个额外的减法 23，它表示 IEEE-754 格式中指数的起始位位置。这就是为什么您会看到来自测试函数的指数值 -23。

|    Precision     |   Sign   |    Exponent     |  Fraction Bits  |  Bias  |
|:----------------:|:--------:|:---------------:|:---------------:|:------:|
| Single (32 Bits) |  1 [31]  |  8 [30-**23**]  |   23 [22-00]    |  127   |

需要此减法以帮助进行第二次测试。因此，任何小于 -23 的值都必须小于一 (1)，因此不是整数。

为了理解第二个测试是如何工作的，让我们使用一个整数值。这次我们将代码中的数字设置为234523，再次运行程序。

```
Starting Number: 234523.000000

Bit Pattern: 0 | 1001 0000 | 110 0101 0000 0110 1100 0000

Sign: 0 Exponent: 144 (17) Mantissa: 0.789268 Value: 234523.000000


Exponent: -6 Coefficient: 15009472 IntTest: 0
INTEGER
```

第二个测试寻找两个条件来识别数字是否不是整数。这需要使用按位数学。让我们看看我们在函数中执行的数学运算：

```
    exponent := int(bits >> 23) - bias - 23
    coefficient := (bits & ((1 << 23) - 1)) | (1 << 23)
    intTest := coefficient & ((1 << uint32(-exponent)) - 1)
```

系数计算是将 1 + 添加到尾数，所以我们有以 2 为底的系数值。

当我们查看系数计算的第一部分时，我们会看到以下位模式：

```
coefficient := (bits & ((1 << 23) - 1)) | (1 << 23)

Bits:                   01001000011001010000011011000000
(1 << 23) - 1:          00000000011111111111111111111111
bits & ((1 << 23) - 1): 00000000011001010000011011000000
```

系数计算的第一部分从整个 IEEE-754 位模式中删除符号和指数位。

系数计算的第二部分将“1 +”添加到二进制位模式中：

```
coefficient := (bits & ((1 << 23) - 1)) | (1 << 23)

bits & ((1 << 23) - 1): 00000000011001010000011011000000
(1 << 23):              00000000100000000000000000000000
coefficient:            00000000111001010000011011000000
```

现在已经设置了系数位模式，我们可以计算 intTest 值：

```
exponent := int(bits >> 23) - bias - 23
intTest := (coefficient & ((1 << uint32(-exponent)) - 1))

exponent:                     (144 - 127 - 23) = -6
1 << uint32(-exponent):       000000
(1 << uint32(-exponent)) - 1: 111111

coefficient:                 00000000111001010000011011000000
1 << uint32(-exponent)) - 1: 00000000000000000000000000111111
intTest:                     00000000000000000000000000000000
```

我们在测试函数中计算的指数值用于确定我们将与系数进行比较的位数。在这种情况下，指数值为 -6。这是通过将存储的指数值 (144) 与偏差 (127) 相减，然后与指数 (23) 的起始位位置相减来计算的。这为我们提供了 6 个 1 (1)
的位模式。最后的操作采用这 6 位并将它们与系数的最右边 6 位相乘以获得 intTest 值。

第二个测试是寻找一个小于零 (0) 的指数值和一个非零 (0) 的 intTest 值。这将表明存储的数字不是整数。在我们的 234523 示例中，Exponent 小于零 (0)，但 intTest 的值为零 (0)。我们有一个整数。

我已将示例代码包含在 Go 操场中，因此您可以使用它。

http://play.golang.org/p/3xraud43pi

如果不是 Gustavo 的代码，我永远无法确定解决方案。这是他的解决方案的链接：

http://bazaar.launchpad.net/~niemeyer/strepr/trunk/view/6/strepr.go#L160

这是您可以复制和粘贴的代码副本：

```
package main

import (
    "fmt"
    "math"
)

func main() {
    var number float32 = 234523

    fmt.Printf("Starting Number: %f\n\n", number)

    // Float32bits returns the IEEE 754 binary representation
    bits := math.Float32bits(number)

    binary := fmt.Sprintf("%.32b", bits)

    fmt.Printf("Bit Pattern: %s | %s %s | %s %s %s %s %s %s\n\n",
        binary[0:1],
        binary[1:5], binary[5:9],
        binary[9:12], binary[12:16], binary[16:20],
        binary[20:24], binary[24:28], binary[28:32])

    bias := 127
    sign := bits & (1 << 31)
    exponentRaw := int(bits >> 23)
    exponent := exponentRaw - bias

    var mantissa float64
    for index, bit := range binary[9:32] {
        if bit == 49 {
            position := index + 1
            bitValue := math.Pow(2, float64(position))
            fractional := 1 / bitValue

            mantissa = mantissa + fractional
        }
    }

    value := (1 + mantissa) * math.Pow(2, float64(exponent))

    fmt.Printf("Sign: %d Exponent: %d (%d) Mantissa: %f Value: %f\n\n",
        sign,
        exponentRaw,
        exponent,
        mantissa,
        value)

    IsInt(bits, bias)
}

func IsInt(bits uint32, bias int) {
    exponent := int(bits>>23) - bias - 23
    coefficient := (bits & ((1 << 23) - 1)) | (1 << 23)
    intTest := coefficient & ((1 << uint32(-exponent)) - 1)

    ShowBits(bits, bias, exponent)

    fmt.Printf("\nExp: %d Frac: %d IntTest: %d\n",
        exponent,
        coefficient,
        intTest)

    if exponent < -23 {
        fmt.Printf("NOT INTEGER\n")
        return
    }

    if exponent < 0 && intTest != 0 {
        fmt.Printf("NOT INTEGER\n")
        return
    }

    fmt.Printf("INTEGER\n")
}

func ShowBits(bits uint32, bias int, exponent int) {
    value := (1 << 23) - 1
    value2 := (bits & ((1 << 23) - 1))
    value3 := (1 << 23)
    coefficient := (bits & ((1 << 23) - 1)) | (1 << 23)

    fmt.Printf("Bits:\t\t\t%.32b\n", bits)
    fmt.Printf("(1 << 23) - 1:\t\t%.32b\n", value)
    fmt.Printf("bits & ((1 << 23) - 1):\t\t%.32b\n\n", value2)

    fmt.Printf("bits & ((1 << 23) - 1):\t\t%.32b\n", value2)
    fmt.Printf("(1 << 23):\t\t\t%.32b\n", value3)
    fmt.Printf("coefficient:\t\t\t%.32b\n\n", coefficient)

    value5 := 1 << uint32(-exponent)
    value6 := (1 << uint32(-exponent)) - 1
    inTest := (coefficient & ((1 << uint32(-exponent)) - 1))

    fmt.Printf("1 << uint32(-exponent):\t\t%.32b\n", value5)
    fmt.Printf("(1 << uint32(-exponent)) - 1:\t%.32b\n\n", value6)

    fmt.Printf("coefficient:\t\t\t%.32b\n", coefficient)
    fmt.Printf("(1 << uint32(-exponent)) - 1:\t%.32b\n", value6)
    fmt.Printf("intTest:\t\t\t%.32b\n", inTest)
}
```

我要感谢 Gustavo 发布了这个问题，并给了我一些真正努力去理解的东西。