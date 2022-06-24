# [The Go Programming Language Specification](https://go.dev/ref/spec)

- [Introduction](#introduction)
- [Notation](#notation)
- [Source code representation](#source-code-representation)
    - [Characters](#characters)
    - [Letters and digits](#letters-and-digits)
- [Lexical elements](#lexical-elements)
    - [Comments](#comments)
    - [Tokens](#tokens)
    - [Semicolons](#semicolons)
    - [Identifiers](#identifiers)
    - [Keywords](#keywords)
    - [Operators and punctuation](#operators-and-punctuation)
    - [Integer literals](#integer-literals)
    - [Floating-point literals](#floating-point-literals)
    - [Imaginary literals](#imaginary-literals)
    - [Rune literals](#rune-literals)
    - [String literals](#string-literals)
- [Constants](#constants)
- [Variables](#variables)
- [Types](#types)
    - [Method sets](#method-sets)
    - [Boolean types](#boolean-types)
    - [Numeric types](#numeric-types)
    - [String types](#string-types)
    - [Array types](#array-types)
    - [Slice types](#slice-types)
    - [Struct types](#struct-types)
    - [Pointer types](#pointer-types)
    - [Function types](#function-types)
    - [Interface types](#interface-types)
    - [Map types](#map-types)
    - [Channel types](#channel-types)
- [Properties of types and values](#properties-of-types-and-values)
    - [Type identity](#type-identity)
    - [Assignability](#assignability)
    - [Representability](#representability)
- [Blocks](#blocks)
- [Declarations and scope](#declarations-and-scope)
    - [Label scopes](#label-scopes)
    - [Blank identifier](#blank-identifier)
    - [Predeclared identifiers](#predeclared-identifiers)
    - [Exported identifiers](#exported-identifiers)
    - [Uniqueness of identifiers](#uniqueness-of-identifiers)
    - [Constant declarations](#constant-declarations)
    - [Iota](#iota)
    - [Type declarations](#type-declarations)
    - [Variable declarations](#variable-declarations)
    - [Short variable declarations](#short-variable-declarations)
    - [Function declarations](#function-declarations)
    - [Method declarations](#method-declarations)
- [Expressions](#expressions)
    - [Operands](#operands)
    - [Qualified identifiers](#qualified-identifiers)
    - [Composite literals](#composite-literals)
    - [Function literals](#function-literals)
    - [Primary expressions](#primary-expressions)
    - [Selectors](#selectors)
    - [Method expressions](#method-expressions)
    - [Method values](#method-values)
    - [Index expressions](#index-expressions)
    - [Slice expressions](#slice-expressions)
    - [Type assertions](#type-assertions)
    - [Calls](#calls)
    - [Passing arguments to ... parameters](#passing-arguments-to--parameters)
    - [Operators](#operators)
    - [Arithmetic operators](#arithmetic-operators)
    - [Comparison operators](#comparison-operators)
    - [Logical operators](#logical-operators)
    - [Address operators](#address-operators)
    - [Receive operator](#receive-operator)
    - [Conversions](#conversions)
    - [Constant expressions](#constant-expressions)
    - [Order of evaluation](#order-of-evaluation)
- [Statements](#statements)
    - [Terminating statements](#terminating-statements)
    - [Empty statements](#empty-statements)
    - [Labeled statements](#labeled-statements)
    - [Expression statements](#expression-statements)
    - [Send statements](#send-statements)
    - [IncDec statements](#incdec-statements)
    - [Assignments](#assignments)
    - [If statements](#if-statements)
    - [Switch statements](#switch-statements)
    - [For statements](#for-statements)
    - [Go statements](#go-statements)
    - [Select statements](#select-statements)
    - [Return statements](#return-statements)
    - [Break statements](#break-statements)
    - [Continue statements](#continue-statements)
    - [Goto statements](#goto-statements)
    - [Fallthrough statements](#fallthrough-statements)
    - [Defer statements](#defer-statements)
- [Built-in functions](#built-in-functions)
    - [Close](#close)
    - [Length and capacity](#length-and-capacity)
    - [Allocation](#allocation)
    - [Making slices, maps and channels](#making-slices-maps-and-channels)
    - [Appending to and copying slices](#appending-to-and-copying-slices)
    - [Deletion of map elements](#deletion-of-map-elements)
    - [Manipulating complex numbers](#manipulating-complex-numbers)
    - [Handling panics](#handling-panics)
    - [Bootstrapping](#bootstrapping)
- [Packages](#packages)
    - [Source file organization](#source-file-organization)
    - [Package clause](#package-clause)
    - [Import declarations](#import-declarations)
    - [An example package](#an-example-package)
- [Program initialization and execution](#program-initialization-and-execution)
    - [The zero value](#the-zero-value)
    - [Package initialization](#package-initialization)
    - [Program execution](#program-execution)
- [Errors](#errors)
- [Run-time panics](#run-time-panics)
- [System considerations](#system-considerations)
    - [Package unsafe](#package-unsafe)
    - [Size and alignment guarantees](#size-and-alignment-guarantees)

## Introduction

这是 Go 编程语言的参考手册。相关更多信息和其他文档，请访问 [golang.org](https://golang.org)。

Go 是一种通用语言，设计时考虑了系统编程。它是强类型和垃圾收集的，并明确支持并发编程。程序由 *包* 构成，其属性可以来高效的管理依赖项。

语法紧凑，解析简单，便于集成开发环境等自动化工具分析。

## Notation

语法使用 Extended Backus-Naur Form  (EBNF) ：

```
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

Productions 是由项和以下运算符构造的表达式，优先级递增：

```
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

小写 production names 被用于识别 Lexical tokens 。在非终端中使用驼峰式大小写。Lexical tokens 被包裹在 `""` 或 ` `` ` 中。

a … b 表示从 a 到 b 的字符集。水平省略号 … 也用于规范中的其他地方，以非正式地表示未进一步指定的各种枚举或代码片段。字符 … （相对于三个字符 ... ）不是 Go 语言的标记。

## Source code representation

源代码是以 [UTF-8](https://en.wikipedia.org/wiki/UTF-8) 编码的 Unicode 文本。文本未规范化，因此单个重音代码点与由重音和字母组合构成的相同字符不同；
这些被视为两个代码点。为简单起见，本文档将使用 the unqualified term *character* 来指代源文本中的 Unicode code point。

每个代码点都是不同的；例如，大写和小写字母是不同的字符。

实现限制：为了与其他工具兼容，编译器可能不允许在源文本中使用 NUL 字符 (U+0000)。

实现限制：为了与其他工具兼容，如果 UTF-8 编码的字节顺序标记 (U+FEFF) 是源文本中的第一个 Unicode 代码点，则编译器可能会忽略它。在源代码中的任何其他地方都不允许使用字节顺序标记。

### Characters

以下术语用于表示特定的 Unicode 字符类：

```
newline        = /* the Unicode code point U+000A */ .
unicode_char   = /* an arbitrary Unicode code point except newline */ .
unicode_letter = /* a Unicode code point classified as "Letter" */ .
unicode_digit  = /* a Unicode code point classified as "Number, decimal digit" */ .
```

在 [The Unicode Standard 8.0](https://www.unicode.org/versions/Unicode8.0.0/) ，第 4.5 节 "General Category" 定义了一组字符类别。Go
将任何字母类别 Lu、Ll、Lt、Lm 或 Lo 中的所有字符视为 Unicode 字母，将数字类别 Nd 中的所有字符视为 Unicode 数字。

### Letters and digits

下划线字符 _ (U+005F) 被认为是一个字母。

```
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```

## Lexical elements

### Comments

注释被用来作为程序文档。有两种形式：

1. *行注释* 以字符序列 // 开始，并在行尾停止。
2. *一般注释* 以字符序列 /* 开始，以第一个后续字符序列 */ 结束。

注释不能在 [rune](#rune-literals) 或 [string literal](#string-literals) 或 注释内 开始。不包含换行符的一般注释就像一个空格，任何其他注释都像一个换行符。

### Tokens

Tokens 形成了 Go 语言的词汇表。有四类：*标识符 (identifiers)*、*关键字 (keywords)*、*运算符 (operators)* 和 *标点符号 (punctuation)*、
*字面量 (literals)*。由空格 (U+0020)、水平制表符 (U+0009)、回车 (U+000D) 和换行符 (U+000A)形成的空白被忽略，除非它分隔了原本会组合成单个的 token。
此外，换行符或文件结尾符可能会触发 [分号](#semicolons) 的插入。在将输入分解为 token 时，下一个 token 是形成有效 token 的最长字符序列。

### Semicolons

在许多 productions 中，形式文法使用 `;`  作为终止符。Go 程序会通过以下两条规则省略大部分分号：

1. 当输入被分解为 tokens 时，一个分号会立刻被插入到这个 token 流中最后一个 token 后面，如果这个 token 是

    - 一个 [标识符](#identifiers)
    - 一个 [整数](#integer-literals)，[浮点数](#floating-point-literals)，[复数](#imaginary-literals)，[字符](#rune-literals)
      ，或者 [字符串](#string-literals) 字面量
    - [关键词](#keywords) `break`，`continue`，`fallthrough`，或 `return` 中的一个
    - [运算符和标点符号](#operators-and-punctuation) `++`, `--`, `)`, `]`, 或 `}` 中的一个

2. 为了让复杂的语句占据一行，可以在结束 `)` 或 `}` 之前省略分号。

为了反映惯用用法，本文档中的示例代码使用这些规则来省略分号。

### Identifiers

标识符用来命名程序实体，例如变量和类型。标识符是一个或多个字母和数字的序列。标识符中的第一个字符必须是字母。

```
identifier = letter { letter | unicode_digit } .
a
_x9
ThisVariableIsExported
αβ
```

一些内置的 [标识符](#predeclared-identifiers)。

### Keywords

以下关键字是保留的，不能用作标识符。

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

### Operators and punctuation

以下字符序列表示 [运算符](#operators)（包括赋值运算符）和标点符号：

```
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

### Integer literals

整数字面量是用来表示 [整数常量](#constants) 的数字序列。可以选择前缀来设置非十进制基数：0b 或 0B 表示二进制，0、0o 或 0O 表示八进制，0x 或 0X 表示十六进制。单个 0
被视为十进制零。在十六进制文字中，字母 a 到 f 和 A 到 F 表示值 10 到 15。

为了可读性，下划线字符 _ 可能出现在前缀之后或连续数字之间；这样的下划线不会改变字面量的值。

```
int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .

decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
binary_digits  = binary_digit { [ "_" ] binary_digit } .
octal_digits   = octal_digit { [ "_" ] octal_digit } .
hex_digits     = hex_digit { [ "_" ] hex_digit } .
42
4_2
0600
0_600
0o600
0O600       // second character is capital letter 'O'
0xBadFace
0xBad_Face
0x_67_7a_2f_cc_40_c6
170141183460469231731687303715884105727
170_141183_460469_231731_687303_715884_105727

_42         // an identifier, not an integer literal
42_         // invalid: _ must separate successive digits
4__2        // invalid: only one _ at a time
0_xBadFace  // invalid: _ must separate successive digits
```

### Floating-point literals

浮点字面量是 [浮点数常量](#constants) 的十进制或十六进制表示。

十进制浮点字面量由整数部分（十进制数字）、小数点、小数部分（十进制数字）和指数部分（e 或 E 后跟可选的符号和十进制数字）组成。可以省略整数部分或小数部分之一；可以省略小数点或指数部分之一。指数值 exp 将尾数（整数和小数部分）缩放
10^exp。

十六进制浮点字面量由 0x 或 0X 前缀、整数部分（十六进制数字）、小数点、小数部分（十六进制数字）和指数部分（p 或 P 后跟可选符号和十进制数字）组成）。可以省略整数部分或小数部分之一；小数点也可以省略，但指数部分是必需的。
（此语法与 IEEE 754-2008 §5.12.3 中给出的语法匹配。）指数值 exp 将尾数（整数和小数部分）缩放 2^exp。

为了可读性，下划线字符 _ 可能出现在前缀之后或连续数字之间；这样的下划线不会改变字面量的值。

```
float_lit         = decimal_float_lit | hex_float_lit .

decimal_float_lit = decimal_digits "." [ decimal_digits ] [ decimal_exponent ] |
                    decimal_digits decimal_exponent |
                    "." decimal_digits [ decimal_exponent ] .
decimal_exponent  = ( "e" | "E" ) [ "+" | "-" ] decimal_digits .

hex_float_lit     = "0" ( "x" | "X" ) hex_mantissa hex_exponent .
hex_mantissa      = [ "_" ] hex_digits "." [ hex_digits ] |
                    [ "_" ] hex_digits |
                    "." hex_digits .
hex_exponent      = ( "p" | "P" ) [ "+" | "-" ] decimal_digits .
0.
72.40
072.40       // == 72.40
2.71828
1.e+0
6.67428e-11
1E6
.25
.12345E+5
1_5.         // == 15.0
0.15e+0_2    // == 15.0

0x1p-2       // == 0.25
0x2.p10      // == 2048.0
0x1.Fp+0     // == 1.9375
0X.8p-0      // == 0.5
0X_1FFFP-16  // == 0.1249847412109375
0x15e-2      // == 0x15e - 2 (integer subtraction)

0x.p1        // invalid: mantissa has no digits
1p-2         // invalid: p exponent requires hexadecimal mantissa
0x1.5e-2     // invalid: hexadecimal mantissa requires p exponent
1_.5         // invalid: _ must separate successive digits
1._5         // invalid: _ must separate successive digits
1.5_e1       // invalid: _ must separate successive digits
1.5e_1       // invalid: _ must separate successive digits
1.5e1_       // invalid: _ must separate successive digits
```

### Imaginary literals

虚数字面量表示 [复数常量](#constants) 的虚部。它由一个 [整数](#integer-literals) 或 [浮点数](#floating-point-literals) 字面量后跟小写字母 i
组成。虚数字面量的值是相应整数或浮点字面量的值乘以虚数单位 i。

```
imaginary_lit = (decimal_digits | int_lit | float_lit) "i" .
```

为了向后兼容，完全由十进制数字（可能还有下划线）组成的虚数字面量的整数部分被认为是十进制整数，即使它以 0 开头。

```
0i
0123i         // == 123i for backward-compatibility
0o123i        // == 0o123 * 1i == 83i
0xabci        // == 0xabc * 1i == 2748i
0.i
2.71828i
1.e+0i
6.67428e-11i
1E6i
.25i
.12345E+5i
0x1p-2i       // == 0x1p-2 * 1i == 0.25i
```

### Rune literals

字符字面量表示 [字符常量](#constants)，即 Unicode code point 对应的整数值。字符字面量表示为一个或多个用单引号括起来的字符，如 'x' 或 '
\n'。在引号内，除了换行符和未转义的单引号外，任何字符都可能出现。单引号字符表示字符本身的 Unicode 值，而以反斜杠开头的多字符序列以各种格式对值进行编码。

最简单的形式是引号内的单个字符；由于 Go 源文本是以 UTF-8 编码的 Unicode 字符，因此多个 UTF-8 编码的字节可能代表一个整数值。例如，'a' 包含单个字节，表示字面量 a，U+0061，值为 0x61，而 'ä'
包含两个字节（0xc3 0xa4），表示字面量 a-dieresis，U+00E4，值为 0xe4。

几个反斜杠转义允许将任意值编码为 ASCII 文本。有四种方法可以将整数值表示为数字常量：\x 后跟正好两个十六进制数字；\u 后跟四个十六进制数字；\U 后跟正好八个十六进制数字，和一个普通的反斜杠 \
后跟正好三个八进制数字。在每种情况下，字面量的值都是由相应基数中的数字表示的值。

尽管这些表示都产生一个整数，但它们具有不同的有效范围。八进制转义符必须表示介于 0 和 255 之间的值。十六进制转义符通过构造满足此条件。转义符 \u 和 \U 表示 Unicode 代码点，因此其中的一些值是非法的，特别是那些高于
0x10FFFF 和替代一半的值。

在反斜杠之后，一些单字符转义代表特殊值：

```
\a   U+0007 alert or bell
\b   U+0008 backspace
\f   U+000C form feed
\n   U+000A line feed or newline
\r   U+000D carriage return
\t   U+0009 horizontal tab
\v   U+000B vertical tab
\\   U+005C backslash
\'   U+0027 single quote  (valid escape only within rune literals)
\"   U+0022 double quote  (valid escape only within string literals)
```

所有其他以反斜杠开头的序列在字符字面量中都是非法的。

```
rune_lit         = "'" ( unicode_value | byte_value ) "'" .
unicode_value    = unicode_char | little_u_value | big_u_value | escaped_char .
byte_value       = octal_byte_value | hex_byte_value .
octal_byte_value = `\` octal_digit octal_digit octal_digit .
hex_byte_value   = `\` "x" hex_digit hex_digit .
little_u_value   = `\` "u" hex_digit hex_digit hex_digit hex_digit .
big_u_value      = `\` "U" hex_digit hex_digit hex_digit hex_digit
                           hex_digit hex_digit hex_digit hex_digit .
escaped_char     = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .
'a'
'ä'
'本'
'\t'
'\000'
'\007'
'\377'
'\x07'
'\xff'
'\u12e4'
'\U00101234'
'\''         // rune literal containing single quote character
'aa'         // illegal: too many characters
'\xa'        // illegal: too few hexadecimal digits
'\0'         // illegal: too few octal digits
'\uDFFF'     // illegal: surrogate half
'\U00110000' // illegal: invalid Unicode code point
```

### String literals

字符串字面量表示通过连接字符序列获得的 [字符串常量](#constants)。有两种形式：原始字符串字面量和可解释的字符串字面量。

原始字符串字面量是反引号之间的字符序列，如 `foo`。在引号内，除了反引号外，任何字符都可以出现。原始字符串字面量的值是由引号之间的未解释（隐式 UTF-8
编码）字符组成的字符串；特别是，反斜杠没有特殊意义，字符串可能包含换行符。原始字符串字面量中的回车符 ('\r') 从原始字符串值中被丢弃。

可解释的字符串字面量是双引号之间的字符序列，如 "bar"。在引号内，除了换行符和未转义的双引号外，任何字符都可能出现。引号之间的文本形成字面量的值，反斜杠转义被解释为 [字符字面量](#rune-literals)（\' 是非法的，而
\" 是合法的），具有相同的限制。三位数的八进制 (\nnn)和两位十六进制 (\xnn) 转义代表结果字符串的单个字节；所有其他转义代表单个字符的（可能是多字节）UTF-8 编码。因此在字符串字面量中 \377 和 \xFF 代表值为
0xFF=255 单字节，而 ÿ、\u00FF、\U000000FF 和 \xc3\xbf 代表 UTF-8 编码字符 U+00FF 的两个字节 0xc3 0xbf。

```
string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { unicode_char | newline } "`" .
interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
`abc`                // same as "abc"
`\n
\n`                  // same as "\\n\n\\n"
"\n"
"\""                 // same as `"`
"Hello, world!\n"
"日本語"
"\u65e5本\U00008a9e"
"\xff\u00FF"
"\uD800"             // illegal: surrogate half
"\U00110000"         // illegal: invalid Unicode code point
```

这些示例都表示相同的字符串：

```
"日本語"                                 // UTF-8 input text
`日本語`                                 // UTF-8 input text as a raw literal
"\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
"\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
```

如果源代码将一个字符表示为两个代码点，例如涉及重音和字母的组合形式，如果放在字符字面量中（它不是单个代码点），结果将是错误的，并且会在字符串文字中被替换为两个代码点。

## Constants

*布尔常量*、*字符常量*、*整数常量*、*浮点数常量*、*复数常量* 和 *字符串常量*。字符、整数、浮点数和复数常量统称为数字常量。

常量值由 [字符](#rune-literals)、[整数](#integer-literals)、[浮点数](#floating-point-literals)、[虚数](#imaginary-literals)
或 [字符串](#string-literals) 字面量、表示常量的标识符、[常量表达式](#constant-expressions)、结果为常量的 [转化](#conversions)
或某些内置函数例如 unsafe.Sizeof 等函数应用于任何值的结果，cap 或 len 应用于 [某些表达式](#length-and-capacity)，real 和 imag 应用于复数常量。布尔值由预先声明的常量 true 和
false 表示。预先声明的标识符 [iota](#iota) 表示一个整数常量。

一般而言，复数常量是 [常量表达式](#constant-expressions) 的一种形式，将在该部分进行讨论。

数字常量代表任意精度的精确值并且不会溢出。因此，没有表示 IEEE-754 负零、无穷大和非数字值的常量。

常量可以是 [有类型的](#types) 或 *无类型的*。字面量常量、true、false、iota 和某些仅包含无类型常量操作数的 [常量表达式](#constant-expressions) 是无类型的。

常量可以通过 [常量声明](#constant-declarations) 或 [转换](#conversions) 显式指定类型，或者在 [变量声明](#variable-declarations)、[赋值](#assignments)
、作为 [表达式](#expressions) 中的操作数时隐式指定类型。如果常量值不能 [表示](#representability) 为对应类型的值，就会出错。

无类型常量具有 *默认类型*，该类型是上下文中需要隐式转换的类型，例如，在没有显式类型的短变量声明中，例如 i := 0。无类型常量的默认类型分别为 bool、rune、int、float64、complex128 或
string，具体取决于它是 布尔、字符、整数、浮点数、复数 或 字符串常量。

实现限制：尽管数字常量在语言中具有任意精度，但编译器可以使用精度有限的内部表示来实现它们。也就是说，每个实现都必须：

- 表示至少 256 位的整数常量。
- 表示浮点数常量，包括复数常量的部分，尾数至少为 256 位，带符号的二进制指数至少为 16 位。
- 如果无法精确表示整数常量，则报错。
- 如果由于溢出而无法表示浮点或复数常量，则给出错误。
- 如果由于精度限制而无法表示浮点或复数常量，则舍入到最接近的可表示常量。

这些要求既适用于字面量常量，也适用于计算 [常量表达式](#constant-expressions) 的结果。

## Variables

变量是 *值* 的存储位置。值域由变量的 *[类型](#types)* 决定。

[变量声明](#variable-declarations)、函数参数值和结果值、[函数声明](#function-declarations) 或 [函数字面量](#function-literals)
的签名为命名变量保留存储空间。调用内置函数 new 或获取 [复合字面量](#composite-literals) 的地址会在运行时为变量分配存储空间。这种匿名变量是通过（可能是隐式的）[指针间接](#address-operators)
引用的。

[数组](#array-types)、[切片](#slice-types) 和 [结构体](#struct-types)类型的*结构化*变量具有可以单独 [寻址](#address-operators)
的元素和字段。每个这样的元素都像一个变量。

变量的 *静态类型*（或只是*类型*）是其声明中给出的类型、new 调用或复合字面量中提供的类型、结构化变量的元素类型。接口类型的变量也有一个独特的*动态*类型，它是在运行时分配给变量的值的具体类型（除非该值是预先声明的标识符
nil，它没有类型）。动态类型在执行期间可能会有所不同，但存储在接口变量中的值始终可以 [赋值](#assignability) 给变量的静态类型。

```
var x interface{}  // x is nil and has static type interface{}
var v *T           // v has value nil, static type *T
x = 42             // x has value 42 and dynamic type int
x = v              // x has value (*T)(nil) and dynamic type *T
```

变量的值可以通过 [表达式](#expressions) 来获取；它是 [赋值](#assignments) 给变量的最新值。如果变量尚未赋值，则其值为其类型的零值。

## Types

类型被用来确定一组值以及专属于于这些值的操作和方法。一个类型可以用一个*类型名称*来表示，如果它有的话，或者使用一个*类型字面量*来指定，它从现有的类型组成一个类型。

```
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
	    SliceType | MapType | ChannelType .
```

Go 语言 [预先声明](#predeclared-identifiers) 了一些类型名称，其他的则是通过 [类型声明](#type-declarations) 引入的。*复合类型*
——数组、结构体、指针、函数、接口、切片、映射和通道类型——可以使用类型字面量构造。

每个类型 T 都有一个*支撑类型*：如果 T 是预先声明的布尔、数字或字符串类型之一，或者类型字面量，则相应的支撑类型是 T 本身。否则，T 的支撑类型是 T 在其 [类型声明](#type-declarations)
中引用类型的支撑类型。

```
type (
	A1 = string
	A2 = A1
)

type (
	B1 string
	B2 B1
	B3 []B1
	B4 B3
)
```

string，A1，A2，B1 和 B2 的支撑类型是 string。[]B1，B3，B4的支撑类型是 []B1。

### Method sets

一个类型有一个（可能是空的）方法集与之关联。[接口类型](#interface-types) 的方法集就是它的接口。其它任何类型 T 的方法集由所有以接收者类型 T 声明的 [方法](#method-declarations)
组成。对应[指针类型](#pointer-types) *T 的方法集是所有以接收者 *T 或 T 声明的方法的集合（即，它还包含 T 的所有方法)
。更多规则适用于包含嵌入字段的结构体，如 [结构体类型](#struct-types) 章节所述。任何其他类型都有一个空的方法集。在一个方法集中，每个方法必须有一个[唯一](#uniqueness-of-identifiers)
的非[空](#blank-identifier)方法名。

类型的方法集决定了该类型 [实现](#interface-types) 的接口以及可以使用该类型的接收器 [调用](#calls) 的方法。

### Boolean types

*布尔类型*是由预先声明的常量 true 和 false 表示的一组布尔值。预先声明的布尔类型是 bool；它是一个[定义类型](#type-definitions)。

### Numeric types

*数字类型*表示整数或浮点数的集合。预先声明的与独立于架构的数字类型是：

```
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32
```

n-bit 整数的值是 n 位宽，并使用 [二进制补码算法](https://en.wikipedia.org/wiki/Two's_complement) 表示。

还有一组预先声明的数字类型具有特定于架构实现的大小：

```
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value
```

为避免可移植性问题，所有数字类型都是 [定义类型](#type-definitions)，因此除了 byte（uint8 的[别名](#alias-declarations)）和 rune（int32
的别名）外，它们是不同的。当表达式或赋值中混合不同的数字类型时，需要显式转换。例如，int32 和 int 不是同一类型，即使它们在特定架构上可能具有相同的大小。

### String types

*字符串*类型表示字符串值的集合。字符串值是一个（可能是空的）字节序列。字节数称为字符串的长度，永远不会为负数。字符串是不可变的：一旦创建，就不可能更改字符串的内容。预先声明的字符串类型是
string；它是一个 [定义类型](#type-definitions)。

可以使用内置函数 len 来查询字符串 s 的长度。如果字符串是常量，则长度是编译期常量。字符串的字节可以通过整数[索引](#index-expressions) 0 到 len(s)-1 访问。取这样一个元素的地址是非法的；如果 s[i]
是字符串的第 i 个字节，则 &s[i] 无效。

### Array types

数组是单一类型元素的集合，称为 element type。元素的数量称为数组的长度，并且永远不会为负数。

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

长度是数组类型的一部分；必须是一个可以由 int 类型[表示](#representability)的非负[常量](#constants)值。可以使用内置函数 len 来查询数组 a
的长度。元素可以通过整数[索引](#index-expressions) 0 到 len(a)-1 寻址。数组类型总是一维的，但可以组合成多维类型。

```
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

### Slice types

切片是其 *支撑数组* 连续段的描述符，并提供对来自该数组的元素的编号序列的访问。切片类型表示其所有数组切片的集合。元素的数量称为切片的长度，永远不会为负数。未初始化切片的值为 nil。

```
SliceType = "[" "]" ElementType .
```

切片 s 的长度可以通过内置函数 len 来查询；与数组不同，它可能会在运行过程中发生变化。元素可以通过整数[索引](#index-expressions) 0 到 len(s)-1
寻址。给定元素的切片索引可能小于支撑数组中相同元素的索引。

切片一旦初始化，就始终与持有其元素的支撑数组相关联。切片因此与其他切片共享支撑存储，拥有同一个数组；相比之下，不同的数组表示不同的存储。

切片的支撑数组可能会超出切片的末尾。*容量*是指：切片的长度与切片之外的数组长度的总和；可以通过从原始切片中[切出](#slice-expressions)一个新切片来创建长度达到该容量的切片。可以使用内置函数 cap(a) 来查询切片 a
的容量。

使用内置函数 make 为给定元素类型 T 生成一个新的初始化切片值，该函数使用切片类型和指定长度和可选的容量作为参数。使用 make 创建的切片总是分配一个新的、隐藏的数组，返回的切片值引用该数组。也就是说，执行

```
make([]T, length, capacity)
```

产生与分配数组相同的切片并对其进行[切片](#slice-expressions)，因此这两个表达式是等效的：

```
make([]int, 50, 100)
new([100]int)[0:50]
```

像数组一样，切片总是一维的，但可以组合起来构造更高维的对象。对于数组内的数组，内部数组在构造上总是相同的长度；但是对于切片中的切片（或切片中的数组），内部长度可能会动态变化。此外，内部切片必须单独初始化。

### Struct types

结构体是一系列命名元素（称为字段）的组合，每个字段都有一个名称和一个类型。字段名称可以显式指定 (IdentifierList) 或隐式指定 (EmbeddedField)。在结构体中，非[空](#blank-identifier)
字段名称必须是[唯一](#uniqueness-of-identifiers)的。

```
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName .
Tag           = string_lit .

// An empty struct.
struct {}

// A struct with 6 fields.
struct {
	x, y int
	u float32
	_ float32  // padding
	A *[]int
	F func()
}
```

使用类型声明但没有明确字段名称的字段称为*嵌入字段*。嵌入字段必须为类型 T 或指向非接口类型 *T 的指针，并且 T 本身可能不是指针类型。未限定类型名称用作字段名称。

```
// A struct with four embedded fields of types T1, *T2, P.T3 and *P.T4
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
	x, y int  // field names are x and y
}
```

以下声明是非法的，因为字段名称在结构类型中必须是唯一的：

```
struct {
	T     // conflicts with embedded field *T and *P.T
	*T    // conflicts with embedded field T and *P.T
	*P.T  // conflicts with embedded field T and *T
}
```

如果 x.f 是一个表示字段或方法 f 的合法[选择器](#selectors)，则结构体 x 嵌入字段的字段或[方法](#method-declarations) f 被称为*提升*。

提升的字段类似于结构体中的普通字段，只是它们不能用作结构体中的[复合字面量](#composite-literals)的字段名称。

给定一个结构体类型 S 和一个[定义类型](#type-definitions) T，提升的方法包含在结构的方法集中，如下所示：

- 如果 S 包含嵌入字段 T，则 S 和 *S 的[方法集](#method-sets)都包含接收者为 T 的提升方法。*S 的方法集还包括接收者为 *T 的提升方法。
- 如果 S 包含嵌入字段 *T，则 S 和 *S 的方法集都包含带有接收者为 T 或 *T 的提升方法。

字段声明后可以跟一个可选的字符串字面量*标签*，是相应字段的属性。空标签相当于不存在的标签。这些标签可以通过 [反射接口](https://go.dev/pkg/reflect/#StructTag)
获取，作为结构体的[类型标识](#type-identity)，但在其他方面会被忽略。

```
struct {
	x, y float64 ""  // an empty tag string is like an absent tag
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// A struct corresponding to a TimeStamp protocol buffer.
// The tag strings define the protocol buffer field numbers;
// they follow the convention outlined by the reflect package.
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```

### Pointer types

指针类型表示指向给定类型（指针的底层类型）[变量](#variables)的所有指针的集合。未初始化指针的值为 nil。

```
PointerType = "*" BaseType .
BaseType    = Type .
*Point
*[4]int
```

### Function types

函数类型表示具有相同参数和结果类型的所有函数的集合。函数类型的未初始化变量的值为 nil。

```
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

在参数或结果列表中，名称 (IdentifierList) 必须全部存在或全部不存在。如果存在，每个名称代表指定类型的一项（参数或结果），并且签名中的所有非[空](#blank-identifier)
名称必须是[唯一](#uniqueness-of-identifiers)的。如果不存在，则每种类型代表该类型的一项。参数和结果列表总是用括号括起来的，除非只有一个未命名的结果，可以被写成一个无括号的类型。

函数签名中的最后一个传入参数可能具有以 ... 为前缀的类型。具有此类参数的函数称为*可变参数*，并且可以使用该参数的零个或多个元素来调用。

```
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```

### Interface types

接口类型列举了*接口*的[方法集](#method-sets)。接口类型的变量可以存储任何类型的值，方法集是接口的超集。任意一种类型都*实现*了*接口*。接口类型的未初始化变量的值为 nil。

```
InterfaceType      = "interface" "{" { ( MethodSpec | InterfaceTypeName ) ";" } "}" .
MethodSpec         = MethodName Signature .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

接口类型可以通过方法规范*显式*指定方法，也可以通过接口类型名称*嵌入*其他接口的方法。

```
// A simple File interface.
interface {
	Read([]byte) (int, error)
	Write([]byte) (int, error)
	Close() error
}
```

接口中的每个方法的名字必须是[唯一的](#uniqueness-of-identifiers)且不能为[空](#blank-identifier)。

```
interface {
	String() string
	String() string  // illegal: String not unique
	_(x int)         // illegal: method must have non-blank name
}
```

多个类型可以实现同一个接口。例如，如果这是两种类型 S1 和 S2 的方法集

```
func (p T) Read(p []byte) (n int, err error)
func (p T) Write(p []byte) (n int, err error)
func (p T) Close() error
```

（其中 T 代表 S1 或 S2）那么 File 接口由 S1 和 S2 实现，而不管 S1 和 S2 可能拥有或共享其他什么方法。

一个类型实现了包含其方法的任何子集的任何接口，因此可以实现几个不同的接口。例如，所有类型都实现了*空接口*：

```
interface{}
```

类似地，考虑这个接口规范，在[类型声明](#type-declarations)中定义了一个名为 Locker 的接口：

```
type Locker interface {
	Lock()
	Unlock()
}
```

如果 S1 和 S2 也实现了

```
func (p T) Lock() { … }
func (p T) Unlock() { … }
```

它们就实现了 Locker 接口以及 File 接口。

接口 T 可以使用（可能限定的）另一个接口 E 代替原有的方法。这称为在 T 中*嵌入*接口 E。T 的[方法集](#method-sets)是 T 显式声明的方法和 T 的嵌入接口的方法集的*并集*。

```
type Reader interface {
	Read(p []byte) (n int, err error)
	Close() error
}

type Writer interface {
	Write(p []byte) (n int, err error)
	Close() error
}

// ReadWriter's methods are Read, Write, and Close.
type ReadWriter interface {
	Reader  // includes methods of Reader in ReadWriter's method set
	Writer  // includes methods of Writer in ReadWriter's method set
}
```

方法集的*并集*只包含每个方法集（导出和非导出）中的方法一次，并且具有[相同](#uniqueness-of-identifiers)名称的方法必须具有[相同](#type-identity)的签名。

```
type ReadCloser interface {
	Reader   // includes methods of Reader in ReadCloser's method set
	Close()  // illegal: signatures of Reader.Close and Close are different
}
```

接口类型 T 不能嵌入自身或递归的嵌入其它嵌入 T 的接口。

```
// illegal: Bad cannot embed itself
type Bad interface {
	Bad
}

// illegal: Bad1 cannot embed itself using Bad2
type Bad1 interface {
	Bad2
}
type Bad2 interface {
	Bad1
}
```

### Map types

映射是一种类型（称为元素类型）的无序元素集合，由另一种类型（称为键类型）的唯一键来索引。未初始化映射的值为 nil。

```
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = Type .
```

映射中的 key 必须可以使用[比较运算符](#comparison-operators) == 和 !=
来进行比较；因此键类型不能是函数、映射或切片。如果键类型是接口类型，则这些动态键必须可以使用比较运算符来进行比较；失败将导致[运行时恐慌](#run-time-panics)。

```
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

映射中元素的数量称为其长度。对于映射 m，可以使用内置函数 len 来查询长度，其长度在运行过程中可能会发生变化。可以在运行过程中使用 [赋值](#assignments)
添加元素并使用 [索引表达式](#index-expressions) 来获取元素；可以使用内置函数 delete 删除元素。

使用内置函数 make 创建一个新的空映射，该函数将映射类型和可选的容量作为参数：

```
make(map[string]int)
make(map[string]int, 100)
```

初始容量不限制其大小：映射会增长以容纳存储在其中的项目数量，除了 nil 映射。一个 nil 映射相当于一个空映射，只是不能添加任何元素。

### Channel types

通道为[并发执行函数](#go-statements)提供了一种通过[发送](#send-statements)和[接收](#receive-operator)指定元素类型的值来进行通信的机制。未初始化通道的值为 nil。

```
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

可选的 <- 运算符用于指定通道*方向*，发送或接收。如果没有给出方向，则通道是*双向的*。通道可以通过[赋值](#assignments)或显式[转化](#conversions)来指定只能发送或只能接受。

```
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints
```

<- 运算符与最左边的 chan 相结合：

```
chan<- chan int    // same as chan<- (chan int)
chan<- <-chan int  // same as chan<- (<-chan int)
<-chan <-chan int  // same as <-chan (<-chan int)
chan (<-chan int)
```

可以使用内置函数 make 创建一个新的初始化通道值，该函数使用通道类型和可选*容量*作为参数：

```
make(chan int, 100)
```

容量（元素的数量）是通道中缓冲区的大小。如果容量为零或不存在，则通道没有缓冲，只有当发送方和接收方都准备好时，通信才能成功。另外，如果缓冲区未满（发送时）或非空（接收时），则通道会进行缓冲并且通信成功不会阻塞。一个 nil
通道永远不会准备好进行通信。

可以使用内置函数 close 关闭通道。[接收操作符](#receive-operator)的多值赋值形式会在报告在通道关闭之前发送的值是否接收成功。

单个通道可以在任意数量中的 goroutine 中被用于[发送语句](#send-statements)、[接收操作](#receive-operator)以及调用内置函数 cap 和
len，而无需进一步的同步语句。通道充当先进先出队列。例如，如果一个 goroutine 在通道上发送值，而第二个 goroutine 接收它们，则按照发送的顺序接收值。

## Properties of types and values

### Type identity

两个类型要么*相同*，要么*不同*。

[定义类型](#type-definitions)总是不同于其他任何类型。此外，如果两个类型的[底层](#types)类型在结构上是相等的，则它们是相同的；也就是说，它们具有相同的字面量结构并且对应的部分具有相同的类型。具体如下：

- 如果两个数组类型具有相同的元素类型和相同的数组长度，则它们是相同的。
- 如果两个切片类型具有相同的元素类型，则它们是相同的。
- 如果两个结构体类型具有相同的字段，并且对应的字段具有相同的名称、相同的类型和相同的标签，则它们是相同的。来自不同包的[非导出](#exported-identifiers)字段名称总是不同的。
- 如果两个指针类型具有相同的底层类型，则它们是相同的。
- 如果两个函数类型相同，则它们具有相同数量的参数和结果值，对应的参数和结果类型相同，并且两个函数都是可变参数或两者都不是。参数和结果名称不需要匹配。
- 如果两个接口类型具有相同名称和相同函数类型的相同方法集，则它们是相同的。来自不同包的[非导出](#exported-identifiers)方法名称总是不同的。方法的顺序无所谓。
- 如果两个映射类型具有相同的键和元素类型，则它们是相同的。
- 如果两个通道类型具有相同的元素类型和相同的方向，则它们是相同的。

给出以下类型声明：

```
type (
	A0 = []string
	A1 = A0
	A2 = struct{ a, b int }
	A3 = int
	A4 = func(A3, float64) *A0
	A5 = func(x int, _ float64) *[]string
)

type (
	B0 A0
	B1 []string
	B2 struct{ a, b int }
	B3 struct{ a, c int }
	B4 func(int, float64) *B0
	B5 func(x int, y float64) *A1
)

type	C0 = B0
```

这些类型是相等的：

```
A0, A1, and []string
A2 and struct{ a, b int }
A3 and int
A4, func(int, float64) *[]string, and A5

B0 and C0
[]int and []int
struct{ a, b *T5 } and struct{ a, b *T5 }
func(x int, y float64) *[]string, func(int, float64) (result *[]string), and A5
```

B0 和 B1 不相等是因为它们是由不同的[定义类型](#type-definitions)创建的新类型；func(int, float64) *B0 和 func(x int, y float64) *[]string 不相等是因为 B0
和 []string 不相等。

### Assignability

值 x 可*赋值*给 T 类型的[变量](#variables)（"x 可赋值给 T"），如果以下条件之一适用：

- x 的类型与 T 相同。
- x 的类型 V 和 T 具有相同的底层类型，并且 V 或 T 中至少有一个不是[定义类型](#type-definitions)。
- T 是一个接口类型，x [实现](#interface-types)了 T。
- x 是双向通道值，T 是通道类型，x 的类型 V 和 T 具有相同的元素类型，并且 V 或 T 中至少有一个不是定义类型。
- x 是预先声明的标识符 nil 并且 T 是指针、函数、切片、映射、通道或接口类型。
- x 是一个可以用 T 类型的值[表示](#representability)的无类型[常量](#constants)。

### Representability

[常量](#constants) x 可由类型 T 的值*表示*，如果以下条件之一适用：

- x 在由 T [类型确定](#types)的一组值中。
- T 是浮点数类型，x 可以四舍五入到 T 的精度而不会溢出。舍入使用 IEEE 754 舍入到偶数规则，但 IEEE 负零进一步简化为无符号零。请注意，常量值永远不会导致 IEEE 负零、NaN 或无穷大。
- T 是一个复数类型，x 的[组成](#manipulating-complex-numbers) real(x) 和 imag(x) 可以用 T 的组成类型（float32 或 float64）的值表示。

```
x                   T           x is representable by a value of T because

'a'                 byte        97 is in the set of byte values
97                  rune        rune is an alias for int32, and 97 is in the set of 32-bit integers
"foo"               string      "foo" is in the set of string values
1024                int16       1024 is in the set of 16-bit integers
42.0                byte        42 is in the set of unsigned 8-bit integers
1e10                uint64      10000000000 is in the set of unsigned 64-bit integers
2.718281828459045   float32     2.718281828459045 rounds to 2.7182817 which is in the set of float32 values
-1e-1000            float64     -1e-1000 rounds to IEEE -0.0 which is further simplified to 0.0
0i                  int         0 is an integer value
(42 + 0i)           float32     42.0 (with zero imaginary part) is in the set of float32 values

x                   T           x is not representable by a value of T because

0                   bool        0 is not in the set of boolean values
'a'                 string      'a' is a rune, it is not in the set of string values
1024                byte        1024 is not in the set of unsigned 8-bit integers
-1                  uint16      -1 is not in the set of unsigned 16-bit integers
1.1                 int         1.1 is not an integer value
42i                 float32     (0 + 42i) is not in the set of float32 values
1e1000              float64     1e1000 overflows to IEEE +Inf after rounding
```

## Blocks

*块*是一对花括号内的可能为空的声明和语句序列。

```
Block = "{" StatementList "}" .
StatementList = { Statement ";" } .
```

除了源代码中的显式块，还有隐式块：

1. *Universe block* 包含所有 Go 源代码。
2. 每个[包](#packages)都有一个 *包块* 来包含该包的所有 Go 源代码。
3. 每个文件都有一个 *文件块*来包含该文件中的所有 Go 源代码。
4. 每个 "[if](#if-statements)", "[for](#for-statements)", 和 "[switch](#switch-statements)" 语句都被认为是在它自己的隐式块中。
5. "[switch](#switch-statements)" 或 "[select](#select-statements)" 语句中的每个子句都充当一个隐式块。

块嵌套影响[作用域](#declarations-and-scope)。

## Declarations and scope

*声明*将非[空](#blank-identifier)标识符绑定到[常量](#constant-declarations)、[类型](#type-declarations)、[变量](#variable-declarations)
、[函数](#function-declarations)、[标签](#labeled-statements)或[包](#import-declarations)
。程序中的每个标识符都必须声明。标识符不能在同一个块中声明两次，也不能在文件块和包块中都声明。

[空白标识符](#blank-identifier)可以像声明中的任何其他标识符一样使用，但它不引入绑定，因此不被声明。在包块中，标识符 init 可能仅用于 [init 函数](#package-initialization)
声明，并且与空白标识符一样，它不会引入新的绑定。

```
Declaration   = ConstDecl | TypeDecl | VarDecl .
TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .
```

声明标识符的*作用域*在源代码之内，其中标识符表示指定的常量、类型、变量、函数、标签或包。

Go 使用[块](#blocks)来约束作用域：

1. [预先声明的标识符](#predeclared-identifiers)的作用域是 Universe 块。
2. 在顶层（任何函数之外）声明的常量、类型、变量或函数（但不是方法）的标识符的作用域是包块。
3. 导入包的包名的作用域是包含导入声明的文件的文件块。
4. 表示方法接收器、函数参数或结果变量的标识符的作用域是函数体。
5. 在函数内声明的常量或变量标识符的作用域从 ConstSpec 或 VarSpec（短变量声明的 ShortVarDecl）的末尾开始，并在包含在最里面的块的末尾结束。
6. 在函数内声明的类型标识符的作用域从 TypeSpec 中的标识符开始，并在包含在最里面的块的末尾结束。

在块中声明的标识符可以在内部块中重新声明。虽然内部声明的标识符在块的作用域内，但它表示内部所声明的实体。

[package 子句](#package-clause)不是声明；包名不会出现在任何作用域内。其目的是识别文件是否属于同一个[包](#packages)并为导入声明指定默认包名称。

### Label scopes

标签由[标签语句](#labeled-statements)声明，用于 "[bread](#break-statements)"、"[continue](#continue-statements)"
和 "[goto](#goto-statements)" 语句。定义一个从未使用的标签是非法的。与其他标识符相比，标签不是块作用域的，并且不会与不是标签的标识符发生冲突。标签的作用域是声明它的函数的主体，不包括任何嵌套函数的主体。

### Blank identifier

*空白标识符*由下划线字符 _ 表示。它用作匿名占位符而不是常规（非空）标识符，并且在[声明](#declarations-and-scope)、作为[操作数](#operands)和[赋值](#assignments)中具有特殊含义。

### Predeclared identifiers

以下标识符是在 Universe 块中隐式声明的：

```
Types:
	bool byte complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr

Constants:
	true false iota

Zero value:
	nil

Functions:
	append cap close complex copy delete imag len
	make new panic print println real recover
```

### Exported identifiers

*导出的*标识符允许从另一个包访问它。如果同时满足以下条件，则该标识符可以导出：

1. 标识符名称的第一个字符是 Unicode 大写字母（Unicode class "Lu"）
2. 标识符在[包块](#blocks)中声明，或者它是[字段名称](#struct-types)或[方法名称](#method-sets)。

其它所有的标识符都是不可以导出的。

### Uniqueness of identifiers

给定一组标识符，如果一个标识符与其他标识符*不同*，则称之为*唯一*标识符。如果两个标识符的拼写不同，或者它们出现在不同的[包](#packages)中并且未[导出](#exported-identifiers)
，则它们是不同的。否则，它们是相同的。

### Constant declarations

常量声明将标识符列表（常量的名称）绑定到[常量表达式](#constant-expressions)列表的值。标识符的个数必须等于表达式的个数，左边第 n 个标识符绑定右边第 n 个表达式的值。

```
ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .

IdentifierList = identifier { "," identifier } .
ExpressionList = Expression { "," Expression } .
```

如果类型存在，则所有常量都采用指定的类型，并且表达式必须[可赋值](#assignability)给该类型。如果类型省略，则常量采用对应表达式的类型。如果表达式值是无类型[常量](#constants)
，则声明的常量保持无类型，常量标识符表示常量值。例如，如果表达式是一个浮点数字面量，常量标识符表示一个浮点数常量，即使字面量的小数部分为零。

```
const Pi float64 = 3.14159265358979323846
const zero = 0.0         // untyped floating-point constant
const (
	size int64 = 1024
	eof        = -1  // untyped integer constant
)
const a, b, c = 3, 4, "foo"  // a = 3, b = 4, c = "foo", untyped integer and string constants
const u, v float32 = 0, 3    // u = 0.0, v = 3.0
```

在 const 括号内的声明列表中，除了第一个 ConstSpec 其它都可以省略表达式。这样的空列表等效于第一个前面的非空表达式列表及其类型（如果有）的正文替代。因此，省略表达式列表等同于重复前面的列表。
标识符的数量必须等于前面列表中的表达式数量。与 iota [常量生成器](#iota)一起，该机制允许声明顺序值：

```
const (
	Sunday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Partyday
	numberOfDays  // this constant is not exported
)
```

### Iota

在[常量声明](#constant-declarations)中，预先声明的标识符 iota 表示连续的无类型整数[常量](#constants)。它的值是常量声明中相应 ConstSpec 的索引，从零开始。它可以用来构造一组相关的常量：

```
const (
	c0 = iota  // c0 == 0
	c1 = iota  // c1 == 1
	c2 = iota  // c2 == 2
)

const (
	a = 1 << iota  // a == 1  (iota == 0)
	b = 1 << iota  // b == 2  (iota == 1)
	c = 3          // c == 3  (iota == 2, unused)
	d = 1 << iota  // d == 8  (iota == 3)
)

const (
	u         = iota * 42  // u == 0     (untyped integer constant)
	v float64 = iota * 42  // v == 42.0  (float64 constant)
	w         = iota * 42  // w == 84    (untyped integer constant)
)

const x = iota  // x == 0
const y = iota  // y == 0
```

根据定义，iota 在同一个 ConstSpec 中的多次使用都具有相同的值：

```
const (
	bit0, mask0 = 1 << iota, 1<<iota - 1  // bit0 == 1, mask0 == 0  (iota == 0)
	bit1, mask1                           // bit1 == 2, mask1 == 1  (iota == 1)
	_, _                                  //                        (iota == 2, unused)
	bit3, mask3                           // bit3 == 8, mask3 == 7  (iota == 3)
)
```

最后一个示例在最后一个表达式中使用了非空表达式列表的[隐式重复](#constant-declarations)。

### Type declarations

类型声明将标识符（*类型名称*）绑定到[类型](#types)。类型声明有两种形式：别名声明和定义类型。

```
TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec = AliasDecl | TypeDef .
```

#### Alias declarations

别名声明将标识符绑定到给定的类型。

```
AliasDecl = identifier "=" Type .
```

在标识符的作用域内，它充当类型的*别名*。

```
type (
	nodeList = []*Node  // nodeList and []*Node are identical types
	Polar    = polar    // Polar and polar denote identical types
)
```

#### Type definitions

定义类型创建一个新的、不同的类型，其具有与给定类型相同的底层类型和操作，并为其绑定一个标识符。

```
TypeDef = identifier Type .
```

新类型称为*定义类型*。它[不同](#type-identity)于任何其他类型，包括创建它的类型。

```
type (
	Point struct{ x, y float64 }  // Point and struct{ x, y float64 } are different types
	polar Point                   // polar and Point denote different types
)

type TreeNode struct {
	left, right *TreeNode
	value *Comparable
}

type Block interface {
	BlockSize() int
	Encrypt(src, dst []byte)
	Decrypt(src, dst []byte)
}
```

定义类型可能具有与其关联的[方法集](#method-declarations)。它不继承任何绑定到原先类型的方法，但接口类型或复合类型元素的方法集保持不变：

```
// A Mutex is a data type with two methods, Lock and Unlock.
type Mutex struct         { /* Mutex fields */ }
func (m *Mutex) Lock()    { /* Lock implementation */ }
func (m *Mutex) Unlock()  { /* Unlock implementation */ }

// NewMutex has the same composition as Mutex but its method set is empty.
type NewMutex Mutex

// The method set of PtrMutex's underlying type *Mutex remains unchanged,
// but the method set of PtrMutex is empty.
type PtrMutex *Mutex

// The method set of *PrintableMutex contains the methods
// Lock and Unlock bound to its embedded field Mutex.
type PrintableMutex struct {
	Mutex
}

// MyBlock is an interface type that has the same method set as Block.
type MyBlock Block
```

定义类型可用于定义不同的布尔、数字或字符串类型并为它们关联方法：

```
type TimeZone int

const (
	EST TimeZone = -(5 + iota)
	CST
	MST
	PST
)

func (tz TimeZone) String() string {
	return fmt.Sprintf("GMT%+dh", tz)
}
```

### Variable declarations

变量声明创建一个或多个[变量](#variables)，将相应的标识符与之绑定，并赋予每个变量一个类型和一个初始值。

```
VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec     = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .
var i int
var U, V, W float64
var k = 0
var x, y float32 = -1, -2
var (
	i       int
	u, v, s = 2.0, 3.0, "bar"
)
var re, im = complexSqrt(-1)
var _, found = entries[name]  // map lookup; only interested in "found"
```

如果给出了表达式列表，变量将会被[赋值](#assignments)为对应表达式的值。否则，每个变量都被初始化为[零值](#the-zero-value)。

如果类型存在，则所有变量都采用指定的类型。否则，每个变量都会在赋值中被赋予相应类型的初始化值。如果该值是无类型常量，则首先将其隐式[转换](#conversions)为其[默认类型](#constants)
；如果它是一个无类型的布尔值，它首先被隐式转换为布尔类型。预先声明的值 nil 不能用于初始化没有显式类型的变量。

```
var d = math.Sin(0.5)  // d is float64
var i = 42             // i is int
var t, ok = x.(T)      // t is T, ok is bool
var n = nil            // illegal
```

实现限制：如果在[函数体](#function-declarations)中声明了一个从未使用的变量，编译器认为是非法的。

### Short variable declarations

*短变量声明* 使用以下语法：

```
ShortVarDecl = IdentifierList ":=" ExpressionList .
```

这是带有初始化表达式但没有类型的常规[变量声明](#variable-declarations)的简写：

```
"var" IdentifierList = ExpressionList .

i, j := 0, 10
f := func() int { return 7 }
ch := make(chan int)
r, w, _ := os.Pipe()  // os.Pipe() returns a connected pair of Files and an error, if any
_, y, _ := coord(p)   // coord() returns three values; only interested in y coordinate
```

与常规变量声明不同，短变量声明可以*重新声明*变量，前提是它们最初声明在同一块（或者是参数列表，如果块是函数体）且具有相同类型，并且至少有一个新的非[空](#blank-identifier)
变量。因此，重新声明只能出现在多变量短声明中。重新声明不会引入新变量；它只是为原始变量赋一个新值。

```
field1, offset := nextField(str, 0)
field2, offset := nextField(str, offset)  // redeclares offset
a, a := 1, 2                              // illegal: double declaration of a or no new variable if a was declared elsewhere
```

短变量声明只能出现在函数内部。在某些上下文中，例如 "[if](#if-statements)"、"[for](#for-statements)" 或 "[switch](#switch-statements)"
语句的初始值设定项，短变量声明可用于声明临时局部变量。

### Function declarations

函数声明将标识符（函数名称）绑定到函数。

```
FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = Block .
```

如果函数的[签名](#function-types)声明了返回参数，则函数体的语句列表必须以 [终止语句](#terminating-statements) 结尾。

```
func IndexRune(s string, r rune) int {
	for i, c := range s {
		if c == r {
			return i
		}
	}
	// invalid: missing return statement
}
```

函数声明可以省略主体。这样的声明为在 Go 之外实现的函数提供了签名，例如汇编例程。

```
func min(x int, y int) int {
	if x < y {
		return x
	}
	return y
}

func flushICache(begin, end uintptr)  // implemented externally
```

### Method declarations

方法是带有接收者的[函数](#function-declarations)。方法声明将标识符（方法名称）绑定到一个方法，并将该方法与接收者的 *底层类型* 相关联。

```
MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
Receiver   = Parameters .
```

接收者是由方法名称前面的额外参数指定的。该参数部分必须声明一个非可变参数，即接收者。它的类型必须是[定义类型](#type-definitions) T 或指向定义类型 T 的指针。T 称为接收器的 *底层类型* 。
接收器的底层类型不能是指针或接口类型，它必须与方法定义在同一个包中。该方法 *被绑定到* 接收器的底层类型，并且方法名称仅在类型 T 或 *T 的[选择器](#selectors)中可见。

非[空](#blank-identifier)的接收者标识符在方法签名中必须是[唯一](#uniqueness-of-identifiers)
的。如果方法体内部没有引用接收者的值，则可以在声明中省略其标识符。这同样适用于常规函数和方法的参数。

对于底层类型，绑定到它的方法的非空名称必须是唯一的。如果底层类型是[结构体类型](#struct-types)，则非空方法和字段名称必须不同。

对于给定类型 Point，以下声明：

```
func (p *Point) Length() float64 {
	return math.Sqrt(p.x * p.x + p.y * p.y)
}

func (p *Point) Scale(factor float64) {
	p.x *= factor
	p.y *= factor
}
```

将接收器类型为 *Point 的 Length 和 Scale 方法绑定到底层类型 Point。

方法的类型是以接收者作为第一个参数的函数的类型。例如，方法 Scale 有类型

```
func(p *Point, factor float64)
```

但是，以这种方式声明的函数不是方法。

## Expressions

表达式通过将运算符和函数应用于操作数来计算值。

### Operands

操作数表示表达式中的元素值。操作数可以是字面量、（可能[限定的](#qualified-identifiers)）非[空](#blank-identifier)标识符来表示[常量](#constant-declarations)
、[变量](#variable-declarations)、[函数](#function-declarations)、带括号的表达式。

[空白标识符](#blank-identifier)只能作为操作数出现在[赋值表达式](#assignments)的左侧。

```
Operand     = Literal | OperandName | "(" Expression ")" .
Literal     = BasicLit | CompositeLit | FunctionLit .
BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
OperandName = identifier | QualifiedIdent .
```

### Qualified identifiers

限定标识符是用包名称前缀作限定的标识符。包名和标识符都不能为[空](#blank-identifier)。

```
QualifiedIdent = PackageName "." identifier .
```

必须先[引入]((#import-declarations))，限定标识符才能访问不同包中的标识符。标识符必须是[可导出的](#exported-identifiers)且在[包级作用域](#blocks)中声明。

```
math.Sin	// denotes the Sin function in package math
```

### Composite literals

复合字面量为结构体、数组、切片和映射构建值，并在每次计算时创建一个新值。它们由字面量类型后跟花括号绑定的元素列表组成。每个元素之前可以有一个相应的键。

```
CompositeLit  = LiteralType LiteralValue .
LiteralType   = StructType | ArrayType | "[" "..." "]" ElementType |
                SliceType | MapType | TypeName .
LiteralValue  = "{" [ ElementList [ "," ] ] "}" .
ElementList   = KeyedElement { "," KeyedElement } .
KeyedElement  = [ Key ":" ] Element .
Key           = FieldName | Expression | LiteralValue .
FieldName     = identifier .
Element       = Expression | LiteralValue .
```

LiteralType 的底层类型必须是结构体、数组、切片或映射类型（语法强制执行此约束，除非类型作为 TypeName 给出）。元素和键的类型必须可[分配](#assignments)给字面量类型相应的字段、元素和键类型；没有额外的转换。
键被解释为结构体字面量的字段名称、数组和切片字面量的索引、映射字面量的键。对于映射字面量，所有元素都必须有一个键。在映射中，指定多个元素拥有相同的字段名称或常量键值是错误的。对于非常量映射键，请参阅 [求值顺序](#order-of-evaluation)
部分。

以下规则适用于结构体字面量：

- 键名必须是在结构体中声明的字段名称。
- 不包含任何键名的结构体必须按照字段声明的顺序为每个结构体字段赋值。
- 如果任何一个元素有键名，则其它元素都必须有键名。
- 具有键名的结构体不需要为每个结构体字段都赋值。被省略的字段默认为该字段的零值。
- 结构体字面量可以省略元素列表；这个结构体默认被赋值为该类型的零值。
- 为属于不同包的结构体的非导出字段赋值是错误的。

给出以下声明：

```
type Point3D struct { x, y, z float64 }
type Line struct { p, q Point3D }
```

一种可能的写法：

```
origin := Point3D{}                            // zero value for Point3D
line := Line{origin, Point3D{y: -4, z: 12.3}}  // zero value for line.q.x
```

以下规则适用于数组和切片字面量：

- 每个元素都有一个关联的整数索引来标记它在数组中的位置。
- 元素使用键作为索引。键必须是可由 int 类型的值[表示](#representability)的非负常量；如果它是被输入的，它必须是整数类型。
- 没有键的元素使用前一个元素的索引加一。如果第一个元素没有键，则其索引为零。

获取复合字面量的[地址](#address-operators)会生成一个指向唯一[变量](#variables)（使用字面量初始化）的指针。

```
var pointer *Point3D = &Point3D{y: 1000}
```

请注意，切片或映射类型的[零值](#the-zero-value)与同一类型的已初始化但为空的值不同。因此，获取空切片或映射复合字面量的地址与使用 new 分配新切片或映射的效果不同。

```
p1 := &[]int{}    // p1 points to an initialized, empty slice with value []int{} and length 0
p2 := new([]int)  // p2 points to an uninitialized slice with value nil and length 0
```

数组字面量的长度是字面量类型中规定的长度。如果字面量中提供的元素少于其长度，则缺少的元素将设置为数组元素类型的零值。提供索引值超出数组索引范围的元素是错误的。符号 ... 指定数组长度等于最大元素索引加一。

```
buffer := [10]string{}             // len(buffer) == 10
intSet := [6]int{1, 2, 3, 5}       // len(intSet) == 6
days := [...]string{"Sat", "Sun"}  // len(days) == 2
```

切片字面量描述了整个底层数组字面量。因此，切片字面量的长度和容量是最大元素索引加一。切片字面量具有以下形式

```
[]T{x1, x2, … xn}
```

这是数组生成切片的简写：

```
tmp := [n]T{x1, x2, … xn}
tmp[0 : n]
```

在数组、切片或映射类型为 T 的复合字面量中，如果其与 T 的元素或键类型相同，则本身是复合字面量的元素或映射键可以省略相应的字面量类型。类似地，当元素或键类型为 *T 时，复合字面量获取元素地址可能会省略 &T。

```
[...]Point{{1.5, -3.5}, {0, 0}}     // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}
[][]int{{1, 2, 3}, {4, 5}}          // same as [][]int{[]int{1, 2, 3}, []int{4, 5}}
[][]Point{{{0, 1}, {1, 2}}}         // same as [][]Point{[]Point{Point{0, 1}, Point{1, 2}}}
map[string]Point{"orig": {0, 0}}    // same as map[string]Point{"orig": Point{0, 0}}
map[Point]string{{0, 0}: "orig"}    // same as map[Point]string{Point{0, 0}: "orig"}

type PPoint *Point
[2]*Point{{1.5, -3.5}, {}}          // same as [2]*Point{&Point{1.5, -3.5}, &Point{}}
[2]PPoint{{1.5, -3.5}, {}}          // same as [2]PPoint{PPoint(&Point{1.5, -3.5}), PPoint(&Point{})}
```

当复合字面量使用 LiteralType 的 TypeName 形式在[关键词](#keywords)和 "if"，"for"，或 "switch"
语句块的左大括号之间作为操作数出现，且复合字面量是不包含在括号、方括号或花括号中，就会产生歧义。 在这种罕见的情况下，字面量的左大括号被错误地解析为引入语句块的大括号。为了解决歧义，复合字面量必须出现在括号内。

```
if x == (T{a,b,c}[i]) { … }
if (x == T{a,b,c}[i]) { … }
```

正确数组、切片和映射字面量的示例：

```
// list of prime numbers
primes := []int{2, 3, 5, 7, 9, 2147483647}

// vowels[ch] is true if ch is a vowel
vowels := [128]bool{'a': true, 'e': true, 'i': true, 'o': true, 'u': true, 'y': true}

// the array [10]float32{-1, 0, 0, 0, -0.1, -0.1, 0, 0, 0, -1}
filter := [10]float32{-1, 4: -0.1, -0.1, 9: -1}

// frequencies in Hz for equal-tempered scale (A4 = 440Hz)
noteFrequency := map[string]float32{
	"C0": 16.35, "D0": 18.35, "E0": 20.60, "F0": 21.83,
	"G0": 24.50, "A0": 27.50, "B0": 30.87,
}
```

### Function literals

函数字面量表示匿名[函数](#function-declarations)。

```
FunctionLit = "func" Signature FunctionBody .
func(a, b int, z float64) bool { return a*b < int(z) }
```

函数字面量可以分配给变量或直接调用。

```
f := func(x, y int) int { return x + y }
func(ch chan int) { ch <- ACK }(replyChan)
```

函数字面量是 *闭包* ：它们可以引用在周围函数中定义的变量。这些变量在周围函数和函数字面量之间共享，只要可以访问，它们就会一直存在。

### Primary expressions

基本表达式是操作数为一个和两个的表达式。

```
PrimaryExpr =
	Operand |
	Conversion |
	MethodExpr |
	PrimaryExpr Selector |
	PrimaryExpr Index |
	PrimaryExpr Slice |
	PrimaryExpr TypeAssertion |
	PrimaryExpr Arguments .

Selector       = "." identifier .
Index          = "[" Expression "]" .
Slice          = "[" [ Expression ] ":" [ Expression ] "]" |
                 "[" [ Expression ] ":" Expression ":" Expression "]" .
TypeAssertion  = "." "(" Type ")" .
Arguments      = "(" [ ( ExpressionList | Type [ "," ExpressionList ] ) [ "..." ] [ "," ] ] ")" .
x
2
(s + ".txt")
f(3.1415, true)
Point{1, 2}
m["foo"]
s[i : j + 1]
obj.color
f.p[i].x()
```

### Selectors

对于不是包名的基本表达式 x，*选择器表达式*：

```
x.f
```

表示值 x（或有时 *x；见下文）的字段或方法 f。标识符 f 称为（字段或方法）*选择器*；它不能是[空白标识符](#blank-identifier)。选择器表达式的类型就是 f 的类型。如果 x
是包名称，请参阅[限定标识符](#qualified-identifiers)的部分。

选择器 f 可以表示类型 T 的字段或方法，也可以表示 T 的嵌套[嵌入字段](#struct-types)的字段或方法。到达 f 所遍历的嵌入字段的数量称为其在 T 中的*深度*。在 T 中声明的字段或方法 f 的深度为零。在 T
中的嵌入字段 A 中声明的字段或方法 f 的深度是 A 中 f 的深度加一。

以下规则适用于选择器：

1. 对于 T 或 *T 类型的值 x，其中 T 不是指针或接口类型，x.f 表示 T 中最浅深度处的 f 字段或方法。如果没有[一个](#uniqueness-of-identifiers)深度最浅的 f ，则选择器表达式是非法的。
2. 对于类型 I 的值 x，其中 I 是接口类型，x.f 表示动态值 x 的实际方法 f 。如果 I 的[方法集](#method-sets)中没有名为 f 的方法，则选择器表达式是非法的。
3. 作为一个例外，如果 x 的类型是[定义](#type-definitions)的指针类型并且 (*x).f 是表示字段（但不是方法）的有效选择器表达式，则 x.f 是 (*x).f 的简写。
4. 在所有其他情况下，x.f 是非法的。
5. 如果 x 是指针类型且值为 nil 且 x.f 表示结构体字段，则给 x.f 赋值或计算会导致[运行时恐慌](#run-time-panics)。
6. 如果 x 是接口类型且值为 nil，则调用或[计算](#method-values) x.f 方法会导致[运行时恐慌](#run-time-panics)。

作为示例，给出以下声明：

```
type T0 struct {
	x int
}

func (*T0) M0()

type T1 struct {
	y int
}

func (T1) M1()

type T2 struct {
	z int
	T1
	*T0
}

func (*T2) M2()

type Q *T2

var t T2     // with t.T0 != nil
var p *T2    // with p != nil and (*p).T0 != nil
var q Q = p
```

可以这样写：

```
t.z          // t.z
t.y          // t.T1.y
t.x          // (*t.T0).x

p.z          // (*p).z
p.y          // (*p).T1.y
p.x          // (*(*p).T0).x

q.x          // (*(*q).T0).x        (*q).x is a valid field selector

p.M0()       // ((*p).T0).M0()      M0 expects *T0 receiver
p.M1()       // ((*p).T1).M1()      M1 expects T1 receiver
p.M2()       // p.M2()              M2 expects *T2 receiver
t.M2()       // (&t).M2()           M2 expects *T2 receiver, see section on Calls
```

下面这种是错误的：

```
q.M0()       // (*q).M0 is valid but not a field selector
```

### Method expressions

如果 M 在类型 T 的[方法集](#method-sets)中，则 T.M 是一个可视为与 M 相同（以方法接收器 T 为第一个参数，其余参数相同）的函数。

```
MethodExpr    = ReceiverType "." MethodName .
ReceiverType  = Type .
```

一个具有两个方法的结构体类型 T，Mv 的接收器是类型 T，Mp 的接收器是类型 *T。

```
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
```

表达式

```
T.Mv
```

产生一个等价于 Mv 的函数，但它的第一个参数为显式接收器；签名为

```
func(tv T, a int) int
```

可以使用显式接收器正常调用该函数，因此这五个调用是等效的：

```
t.Mv(7)
T.Mv(t, 7)
(T).Mv(t, 7)
f1 := T.Mv; f1(t, 7)
f2 := (T).Mv; f2(t, 7)
```

同样，表达式：

```
(*T).Mp
```

将产生一个等价于 Mp 的函数，签名如下：

```
func(tp *T, f float32) float32
```

对于使用值作为接收器的方法，也可以生成使用显式指针作为接收器的函数，所以

```
(*T).Mv
```

将产生一个等价于 Mv 的函数，签名如下：

```
func(tv *T, a int) int
```

这样的函数通过接收器间接创建一个值作为接收器传递给底层方法；该方法不会覆盖在函数调用中传递地址的值。

最后一种情况，对于指针接收器方法来说值接收器函数是非法的，因为指针接收器方法不在值类型的方法集中。

从方法衍生出来的函数可以使用函数调用语法进行调用；接收器作为调用函数的第一个参数。也就是说，给定 f := T.Mv，f 被调用为 f(t, 7) 而不是 t.f(7)
。要构造一个绑定接收器的函数，应使用[函数字面量](#function-literals)或[Method values](#method-values)。

从接口类型的方法生成函数是合法的。衍生出来的函数采用该接口类型作为显式接收器。

### Method values

如果表达式 x 是静态类型 T 且 M 在类型 T 的[Method sets](#method-sets)中，则 x.M 称为*方法值*。方法值 x.M 是一个函数，可以使用与调用 x.M 方法相同的参数进行调用。表达式 x
在计算方法值的过程中被计算并保存； 然后将保存的副本用作任意调用（可以在以后执行）中的接收器。

T 可以是接口或非接口类型。

正如上面对[方法表达式](#method-expressions)的讨论，考虑一个有两个方法的结构体类型 T：Mv，它的接收器是 T；Mp，它的接收器是 *T。

```
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
var pt *T
func makeT() T
```

表达式：

```
t.Mv
```

将生成一个函数值：

```
func(int) int
```

这两个调用是等效的：

```
t.Mv(7)
f := t.Mv; f(7)
```

同样，表达式：

```
pt.Mp
```

将生成一个函数值：

```
func(float32) float32
```

与[选择器](#selectors)一样，对使用指针作为接收器的非接口方法的引用将自动引用该指针：pt.Mv 等效于 (*pt).Mv。

与[方法调用](#calls)一样，对使用可寻址的值作为指针接收器的非接口方法的引用将自动获取该值的地址：t.Mp 等价于 (&t).Mp。

```
f := t.Mv; f(7)   // like t.Mv(7)
f := pt.Mp; f(7)  // like pt.Mp(7)
f := pt.Mv; f(7)  // like (*pt).Mv(7)
f := t.Mp; f(7)   // like (&t).Mp(7)
f := makeT().Mp   // invalid: result of makeT() is not addressable
```

虽然上面的例子使用了非接口类型，但从接口类型的值创建方法值也是合法的。

```
var i interface { M(int) } = myVal
f := i.M; f(7)  // like i.M(7)
```

### Index expressions

索引表达式形式为：

```
a[x]
```

表示指向由 x 索引的数组、指向数组的指针、切片、字符串或映射 a 中的元素。x 分别称为*索引*或*映射键*。以下规则适用：

如果 a 不是映射：

- 索引 x 必须是整数类型或无类型常量
- 常量索引必须是非负的并且可以用 int 类型的值[表示](#representability)
- 无类型的常量索引被赋予 int 类型
- 如果 0 <= x < len(a)，则索引 x 在范围内，否则*越界*

如果 a 是[数组类型](#array-types) A：

- [常量](#constants)索引必须在范围内
- 如果 x 在运行时越界，则会发生[运行时恐慌](#run-time-panics)
- a[x] 是索引 x 处的数组元素，a[x] 的类型是数组 A 的元素类型

如果 a 是数组类型的[指针](#pointer-types)：

- a[x] 就是 (*a)[x] 的缩写

如果 a 是[切片类型](#slice-types) S：

- 如果 x 在运行时越界，则会发生[运行时恐慌](#run-time-panics)
- a[x] 是索引 x 处的切片元素，a[x] 的类型是切片 S 的元素类型

如果 a 是[字符串类型](#string-types)：

- 如果字符串 a 也是常量，则[常量](#constants)索引必须在范围内
- 如果 x 在运行时越界，则会发生[运行时恐慌](#run-time-panics)
- a[x] 是索引 x 处的非常量字节值，a[x] 的类型是 byte
- a[x] 不能重新赋值

如果 a 是[映射类型](#map-types) M：

- x 的类型必须可[赋值](#assignability)给 M 的键类型
- 如果映射中存在键 x，则 a[x] 是键为 x 的映射元素，而 a[x] 的类型是 M 的元素类型
- 如果映射为 nil 或不存在键 x，则 a[x] 是映射 M 的元素类型的[零值](#the-zero-value)

其它情况下的 a[x] 都是非法的。

在一些情况下，为类型为 map[K]V 的映射 a [赋值](#assignments)或初始化会使用以下索引表达式：

```
v, ok = a[x]
v, ok := a[x]
var v, ok = a[x]
```

生成一个额外的无类型布尔值。如果键 x 存在于映射中，则 ok 的值为 true，否则为 false。

为 nil 映射赋值会导致[运行时恐慌](#run-time-panics)。

### Slice expressions

切片表达式从字符串、数组、指向数组的指针、切片中构造子字符串或切片。有两种变体：一种是只指定上下限的简单形式，另一种是同时指定容量的完整形式。

#### Simple slice expressions

对于字符串、数组、指向数组的指针、切片，一般表达式为：

```
a[low : high]
```

构造一个子字符串或切片。*索引* low 和 high 选择 a 中的哪些元素出现在最终结果中。最终结果的索引从 0 开始，长度等于 high - low。对数组 a 进行切片后

```
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]
```

切片 s 类型为 []int ，长度为 3 ，容量为 4，元素为：

```
s[0] == 2
s[1] == 3
s[2] == 4
```

为方便起见，可以省略索引。缺失的 low 默认为零；缺少的 high 默认为被操作数的长度：

```
a[2:] // same as a[2 : len(a)]
a[:3] // same as a[0 : 3]
a[:]  // same as a[0 : len(a)]
```

如果 a 是指向数组的指针，a[log : high] 则为 (*a)[low : high] 的简写。

对于数组或字符串，如果 0 <= low <= high <= len(a)，则索引*在范围内*，否则越界。对于切片，索引上限是切片容量 cap(a) 而不是长度。[常量](#constants)索引必须是非负的且可以使用 int
类型的值[表示](#representability)；对于数组或常量字符串，常量索引也必须在范围内。如果两个指数都是常量，则它们必须满足 low <=
high。如果索引在运行时越界，则会发生[运行时恐慌](#run-time-panics)。

除[无类型字符串](#constants)
外，如果被切片的操作数是字符串或切片，则切片操作会产生与操作数相同类型的非常量值。对于无类型字符串操作数，会产生一个字符串类型的非常量值。如果切片操作数是一个数组，它必须是[可寻址的](#address-operators)
，并且切片操作会产生一个与数组具有相同元素类型的切片。

如果有效切片表达式的操作数是 nil 切片，则会产生一个 nil 切片。否则，会产生一个与操作数共享底层数组的切片。

```
var a [10]int
s1 := a[3:7]   // underlying array of s1 is array a; &s1[2] == &a[5]
s2 := s1[1:4]  // underlying array of s2 is underlying array of s1 which is array a; &s2[1] == &a[5]
s2[1] = 42     // s2[1] == s1[2] == a[5] == 42; they all refer to the same underlying array element
```

#### Full slice expressions

对于数组、指向数组的指针、切片 a（但不是字符串），一般表达式：

```
a[low: high: max]
```

构造一个与简单切片表达式 a[low : high] 相同类型、相同长度和元素的切片。此外，它通过 max - low 来控制生成切片的容量。只有第一个索引可以省略，默认为 0。对数组 a 切片后

```
a := [5]int{1, 2, 3, 4, 5}
t := a[1:3:5]
```

切片 t 的类型为 []int ，长度为 2 ，容量为 4，元素：

```
t[0] == 2
t[1] == 3
```

对于简单的切片表达式，如果 a 是指向数组的指针，则 a[low : high : max] 是 (*a)[low : high : max] 的简写。如果被操作数是一个数组，它必须是[可寻址的](#address-operators)。

如果 0 <= low <= high <= max <= cap(a)，则索引*在范围内*，否则将*越界*。[常量](#constants)索引必须是非负的且可以用 int 类型的值[表示](#representability)
；对于数组，常量索引也必须在范围内。 如果多个索引是常量，则存在的常量必须在相对于彼此的范围内。如果索引在运行时超出范围，则会发生[运行时恐慌](#run-time-panics)。

### Type assertions

对于[接口类型](#interface-types)的表达式 x 和类型 T，一般表达式为：

```
x.(T)
```

判断 x 不是 nil 并且存储在 x 中的值的类型是 T。x.(T) 被称为 *类型断言* 。

更准确地说，如果 T 不是接口类型，则 x.(T) 断言 x 的动态类型是否与 T [相同](#type-identity)。在这种情况下，T 必须[实现](#method-sets) x 的（接口）类型；否则类型断言无效，因为 x
不可能存储 T 类型的值。如果 T 是接口类型，则 x.(T) 断言 x 的动态类型是否实现了接口 T。

如果类型断言成立，则表达式的值是存储在 x 中的值，类型为 T。如果类型断言为假，则会发生[运行时恐慌](#run-time-panics)。换句话说，即使 x 的动态类型只在运行时知道，x.(T) 的类型在正确的程序中也是已知的 T。

```
var x interface{} = 7          // x has dynamic type int and value 7
i := x.(int)                   // i has type int and value 7

type I interface { m() }

func f(y I) {
  s := y.(string)        // illegal: string does not implement I (missing method m)
  r := y.(io.Reader)     // r has type io.Reader and the dynamic type of y must implement both I and io.Reader
  …
}
```

一种在一些情况下被用来[赋值](#assignments)或初始化的类型断言

```
v, ok = x.(T)
v, ok := x.(T)
var v, ok = x.(T)
var v, ok interface{} = x.(T) // dynamic types of v and ok are T and bool
```

会生成一个额外的无类型布尔值。如果断言成立，则 ok 的值为 true。否则值为 false 且 v 的值是 T 类型的零值。在这种情况下不会发生[运行时恐慌](#run-time-panics)。

### Calls

给定函数类型 F 的表达式 f，

```
f(a1, a2, … an)
```

使用参数 a1, a2, ... an 调用 f。除了一种特殊情况外，参数必须是可[赋值](#assignability)给 F 的参数的类型的单值表达式，并在调用函数之前求值。表达式的类型是 F
的结果类型。方法调用与之类似，但方法本身被认为基于方法的接收器的值。

```
math.Atan2(x, y)  // function call
var pt *Point
pt.Scale(3.5)     // method call with receiver pt
```

在函数调用中，函数值和参数按[一般顺序](#order-of-evaluation)计算。在对它们求值后，调用的参数会被使用值语义传递给函数，被调用的函数开始执行。当函数返回时，函数的返回参数使用值语义传递回调用者。

调用 nil 函数值会导致[运行时恐慌](#run-time-panics)。

作为一种特殊情况，如果函数或方法 g 的返回值在数量上相等并且可以分配给另一个函数或方法 f ，那么调用 f(g(*parameters_of_g*)) 会将 g 的返回值绑定到 f 的参数后调用 f 。除了 g 的调用外，f
的调用不能包含任何参数，并且 g 必须至少有一个返回值。如果 f 有 ... 参数，它会在常规参数分配后，存储的 g 的返回值。

```
func Split(s string, pos int) (string, string) {
	return s[0:pos], s[pos:]
}

func Join(s, t string) string {
	return s + t
}

if Join(Split(value, len(value)/2)) != value {
	log.Panic("test fails")
}
```

如果 x （的类型）的方法集包含 m 并且实参可以赋值给 m 的形参，则方法调用 x.m() 是有效的。如果 x 是[可寻址的](#address-operators)且 &x 的方法集包含 m，则 x.m() 是 (&x).m()
的简写：

```
var p Point
p.Scale(3.5)
```

没有不同的方法类型，也没有方法字面量。

### Passing arguments to ... parameters

如果 f 最后一个参数 p 是 ...T 类型，则在 f 中 p 等效于 []T 类型。如果在 p 没有实参的情况下调用 f，则传递给 p 的值为 nil。否则，传递的值是一个 []T
类型的新切片，带有一个新的底层数组，其连续元素是实参，所有参数都必须可以[赋值](#assignability)给 T。因此，切片的长度和容量会依据实参 p 且每个调用方可能会不一样。

给定函数和调用

```
func Greeting(prefix string, who ...string)
Greeting("nobody")
Greeting("hello:", "Joe", "Anna", "Eileen")
```

在 Greeting 中，第一个调用中 who 的值为 nil，第二个调用中 who 的值为 []string{"Joe", "Anna", "Eileen"}。

如果最后一个参数可赋值给切片类型 []T 且后跟 ...，则将其作为 ...T 参数使用值语义传递。在这种情况下，不会创建新切片。

给定切片和调用

```
s := []string{"James", "Jasmine"}
Greeting("goodbye:", s...)
```

在 Greeting 中，who 将与 s 具有相同的值，并且具有相同的底层数组。

### Operators

运算符将操作数组合成表达式。

```
Expression = UnaryExpr | Expression binary_op Expression .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .

binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .

unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" .
```

比较运算符会在[别处](#comparison-operators)讨论。对于其他二元运算符，除非操作涉及移位或无类型[常量](#constants)，否则操作数类型必须[相同](#type-identity)
。对于仅涉及常量的操作，请阅读[常量表达式](#constant-expressions)。

除移位操作外，如果一个操作数是无类型[常量](#constants)而另一个操作数不是，则该常量将被隐式[转化](#conversions)为另一个操作数的类型。

移位表达式中的右操作数必须是整数类型或者是可由 uint 类型的值[表示](#representability)的无类型常量。如果非常量移位表达式的左操作数是无类型常量，则首先将其隐式转换为移位表达式仅被其左操作数替换时所假定的类型。

```
var a [1024]byte
var s uint = 33

// The results of the following examples are given for 64-bit ints.
var i = 1<<s                   // 1 has type int
var j int32 = 1<<s             // 1 has type int32; j == 0
var k = uint64(1<<s)           // 1 has type uint64; k == 1<<33
var m int = 1.0<<s             // 1.0 has type int; m == 1<<33
var n = 1.0<<s == j            // 1.0 has type int; n == true
var o = 1<<s == 2<<s           // 1 and 2 have type int; o == false
var p = 1<<s == 1<<33          // 1 has type int; p == true
var u = 1.0<<s                 // illegal: 1.0 has type float64, cannot shift
var u1 = 1.0<<s != 0           // illegal: 1.0 has type float64, cannot shift
var u2 = 1<<s != 1.0           // illegal: 1 has type float64, cannot shift
var v float32 = 1<<s           // illegal: 1 has type float32, cannot shift
var w int64 = 1.0<<33          // 1.0<<33 is a constant shift expression; w == 1<<33
var x = a[1.0<<s]              // panics: 1.0 has type int, but 1<<33 overflows array bounds
var b = make([]byte, 1.0<<s)   // 1.0 has type int; len(b) == 1<<33

// The results of the following examples are given for 32-bit ints,
// which means the shifts will overflow.
var mm int = 1.0<<s            // 1.0 has type int; mm == 0
var oo = 1<<s == 2<<s          // 1 and 2 have type int; oo == true
var pp = 1<<s == 1<<33         // illegal: 1 has type int, but 1<<33 overflows int
var xx = a[1.0<<s]             // 1.0 has type int; xx == a[0]
var bb = make([]byte, 1.0<<s)  // 1.0 has type int; len(bb) == 0
```

#### Operator precedence

一元运算符具有最高优先级。++ 和 -- 运算符构成语句而不是表达式，它们不属于运算符层级。值得注意的是，语句 *p++ 与 (*p)++ 相同。

二元运算符有五个优先级。乘法运算符优先级最高，其次是加法运算符、比较运算符、&&（ADN），最后是 || （OR）：

```
Precedence    Operator
    5             *  /  %  <<  >>  &  &^
    4             +  -  |  ^
    3             ==  !=  <  <=  >  >=
    2             &&
    1             ||
```

相同优先级的二元运算符的计算顺序是从左到右。例如，x / y * z 与 (x / y) * z 相同。

```
+x
23 + 3*x[i]
x <= f()
^a >> b
f() || g()
x == y+1 && <-chanInt > 0
```

### Arithmetic operators

算术运算符应用于数值并生成与第一个操作数相同类型的值。四个标准算术运算符（+、-、*、/）适用于整数、浮点数、复数类型；+ 也适用于字符串。位逻辑运算符和移位运算符仅适用于整数。

```
+    sum                    integers, floats, complex values, strings
-    difference             integers, floats, complex values
*    product                integers, floats, complex values
/    quotient               integers, floats, complex values
%    remainder              integers

&    bitwise AND            integers
|    bitwise OR             integers
^    bitwise XOR            integers
&^   bit clear (AND NOT)    integers

<<   left shift             integer << integer >= 0
>>   right shift            integer >> integer >= 0
```

#### Integer operators

对于两个整数 x 和 y，商 q = x / y 和余数 r = x % y 满足以下关系：

```
x = q*y + r  and  |r| < |y|
```

x / y 向零截断（[截断除法](https://en.wikipedia.org/wiki/Modulo_operation)）。

```
 x     y     x / y     x % y
 5     3       1         2
-5     3      -1        -2
 5    -3      -1         2
-5    -3       1        -2
```

此规则的一个例外是，如果被除数 x 是当前 int 类型的最小值，由于二进制补码[整数溢出](#integer-overflow)，商 q = x / -1 等于 x（且 r = 0）：

```
			 x, q
int8                     -128
int16                  -32768
int32             -2147483648
int64    -9223372036854775808
```

如果除数是[常量](#constants)，则它不能为零。如果运行时除数为零，则会发生[运行时恐慌](#run-time-panics)。如果被除数为非负且除数是 2 的常数幂，则除法可以用右移代替，余数的计算可以用按位与运算代替：

```
 x     x / 4     x % 4     x >> 2     x & 3
 11      2         3         2          3
-11     -2        -3        -3          1
```

移位运算符将左操作数移位右操作数指定的位数，该计数必须为非负数。如果运行时移位计数为负，则会发生[运行时恐慌](#run-time-panics)
。如果左操作数是有符号整数，则移位运算符实现算术移位，如果它是无符号整数，则执行逻辑移位。移位数量没有上限。 移位的行为就像左操作数被移位 n 次，移位计数为 n。因此，x << 1 与 x*2 相同，x >> 1 与 x/2
相同，但向负无穷大截断。

对于整数操作数，一元运算符 +、 -、 ^ 定义如下：

```
+x                          is 0 + x
-x    negation              is 0 - x
^x    bitwise complement    is m ^ x  with m = "all bits set to 1" for unsigned x
                                      and  m = -1 for signed x
```

#### Integer overflow

对于无符号整数，+、 -、 *、 << 操作以 2^n 为模进行计算，其中 n 是[无符号整数类型](#numeric-types)的位宽。粗略地说，这些无符号整数运算在溢出时丢弃高位，并且程序可能依赖于 "wrap around"。

对于有符号整数，+、 -、 *、 /、 << 操作可能会合法溢出，生成的值可由有符号整数、操作及其操作数确定性地定义。溢出不会导致[运行时恐慌](#run-time-panics)
。编译器可能不会在不发生溢出的假设下优化代码。例如，它可能不会假设 x < x + 1 总是正确的。

#### Floating-point operators

对于浮点数和复数，+x 与 x 相同，而 -x 是 x 的负数。浮点数或复数除以零的结果没有超出 IEEE-754 标准的规定；是否发生[运行时恐慌](#run-time-panics)是特定于具体实现的。

一个实现可能将多个浮点运算组合成一个单一的融合运算，可能跨语句，并产生与通过单独执行和舍入指令获得的值不同的结果。显式浮点数类型转换会舍入到目标类型的精度，从而防止该丢弃的溢出的融合。

例如，一些架构提供了 "fused multiply and add" (FMA) 指令，该指令计算 x*y + z 且不舍入中间结果 x*y。这些示例显示了 Go 实现何时可以使用该指令：

```
// FMA allowed for computing r, because x*y is not explicitly rounded:
r  = x*y + z
r  = z;   r += x*y
t  = x*y; r = t + z
*p = x*y; r = *p + z
r  = x*y + float64(z)

// FMA disallowed for computing r, because it would omit rounding of x*y:
r  = float64(x*y) + z
r  = z; r += float64(x*y)
t  = float64(x*y); r = t + z
```

#### String concatenation

字符串可以使用 + 或 += 运算符来进行拼接：

```
s := "hi" + string(c)
s += " and good bye"
```

字符串加法通过拼接操作数创建一个新字符串。

### Comparison operators

比较运算符通过比较两个操作数产生一个无类型的布尔值。

```
==    equal
!=    not equal
<     less
<=    less or equal
>     greater
>=    greater or equal
```

在任何比较中，第一个操作数必须是可[赋值](#assignability)给第二个操作数的类型，反之亦然。

相等运算符 == 和 != 适用于*可比较的*操作数。排序运算符 <、<=、> 和 >= 适用于*可排序的*操作数。这些术语和比较结果的定义如下：

- 布尔值具有可比性。如果两个布尔值都为 true 或 false，则它们相等。
- 整数值是可比较和可排序的，和以往一样。
- 根据 IEEE-754 标准的定义，浮点数值是可比较和可排序的。
- 复数值具有可比性。如果 real(u) == real(v) 和 imag(u) == imag(v) ，则两个复数值 u 和 v 相等。
- 字符串值是可比较和可排序的，按 byte-wise 排序。
- 指针值具有可比性。如果两个指针指向同一个变量或者两者都为 nil，则它们相等。指向不同的 [zero-size](#size-and-alignment-guarantees) 变量的指针可能相等，也可能不相等。
- 通道值具有可比性。如果两个通道值是由同一个 make 调用创建的，或值都为 nil，则它们相等。
- 接口值具有可比性。如果两个接口值具有[相同](#type-identity)的动态类型和相同的动态值，或值都为 nil，则它们相等。
- 非接口类型 X 的值 x 和接口类型 T 的值 t 具有可比性，当非接口类型 X 的值具有可比性且 X 实现 T 时。如果 t 的动态类型与 X 相同且 t 的动态值等于 x，则它们相等。
- 结构体类型的值具有可比性，当所有字段都具有可比性时。如果它们对应的非空白字段相等，则两个结构体相等。
- 数组类型的值具有可比性，当数组元素的值具有可比性时。如果它们对应的元素相等，则两个数组相等。

两个接口值（他们的动态类型值不能比较）的比较会导致[运行时恐慌](#run-time-panics)。此行为不仅适用于直接用接口值进行比较，还适用于接口类型的数组或接口类型的结构体。

切片、映射、函数值不可比较。然而，作为特殊情况，切片、映射、函数值可以与预先声明的标识符 nil 进行比较。也允许将指针、通道和接口值与 nil 进行比较，并遵循上述一般规则。

```
const c = 3 < 4            // c is the untyped boolean constant true

type MyBool bool
var x, y int
var (
	// The result of a comparison is an untyped boolean.
	// The usual assignment rules apply.
	b3        = x == y // b3 has type bool
	b4 bool   = x == y // b4 has type bool
	b5 MyBool = x == y // b5 has type MyBool
)
```

### Logical operators

逻辑运算符应用于[布尔值](#boolean-types)并生成与操作数相同类型的值。右操作数不一定会执行。

```
&&    conditional AND    p && q  is  "if p then q else false"
||    conditional OR     p || q  is  "if p then true else q"
!     NOT                !p      is  "not p"
```

### Address operators

对于 T 类型的操作数 x，&x 生成指向 x 的 *T 类型的指针。操作数必须是可寻址的，也就是说，要不是变量（指针间接）要不是切片（索引操作）；或可寻址结构体操作数的字段选择器；或可寻址数组的数组索引操作。
作为可寻址性要求的一个例外，x 也可以是（可能带括号的）复合文字。如果对 x 的求值会导致运行时恐慌，那么对 &x 的求值也会导致。

对于指针类型 *T 的操作数 x，指针间接 *x 表示 x 指向的类型 T 的变量。如果 x 为 nil ，则尝试运算 *x 将导致运行时恐慌。

```
&x
&a[f(2)]
&Point{2, 3}
*p
*pf(x)

var x *int = nil
*x   // causes a run-time panic
&*x  // causes a run-time panic
```

### Receive operator

对于[通道类型](#channel-types)的操作数 ch，接收操作 <-ch 获取的值是从通道 ch 接收到的值。通道的方向必须允许接收操作，接收操作的类型是通道的元素类型。表达式会阻塞，直到收到值。从 nil
通道接收永远阻塞。在一个[关闭的](#close)通道上的接收操作总是可以立即进行，在接收完之前发送的值后会得到元素类型的[零值](#the-zero-value)。

```
v1 := <-ch
v2 = <-ch
f(<-ch)
<-strobe  // wait until clock pulse and discard received value
```

一种在特殊情况下被用于[赋值](#assignments)或初始化的接收表达式

```
x, ok = <-ch
x, ok := <-ch
var x, ok = <-ch
var x, ok T = <-ch
```

产生一个额外的无类型布尔值来报告通信是否成功。如果收到一个由发送操作传递到通道的值，则 ok 的值为 true，如果收到由于通道关闭为空而生成的零值，则 ok 的值为 false。

### Conversions

转换将表达式的类型转换为指定的类型。转换可能会显式出现在源代码中，也可能会被上下文所*隐式*的表达。

*显式转换*是 T(x) 形式的表达式，其中 T 是一个类型，x 是一个可以转换为 T 类型的表达式。

```
Conversion = Type "(" Expression [ "," ] ")" .
```

如果类型以运算符 * 或 <- 开头，或者以关键字 func 开头且没有返回参数，则必须使用括号将其括起来以避免歧义：

```
*Point(p)        // same as *(Point(p))
(*Point)(p)      // p is converted to *Point
<-chan int(c)    // same as <-(chan int(c))
(<-chan int)(c)  // c is converted to <-chan int
func()(x)        // function signature func() x
(func())(x)      // x is converted to func()
(func() int)(x)  // x is converted to func() int
func() int(x)    // x is converted to func() int (unambiguous)
```

如果 x 可以被 T 类型的值[表示](#representability)，则[常量](#constants) x 可以转换为类型 T。作为特殊情况，整数常量 x 可以使用与非常量
x [相同的规则](#conversions-to-and-from-a-string-type)显式转换为[字符串类型](#string-types)。

转换一个常量会产生一个有类型的常量。

```
uint(iota)               // iota value of type uint
float32(2.718281828)     // 2.718281828 of type float32
complex128(1)            // 1.0 + 0.0i of type complex128
float32(0.49999999)      // 0.5 of type float32
float64(-1e-1000)        // 0.0 of type float64
string('x')              // "x" of type string
string(0x266c)           // "♬" of type string
MyString("foo" + "bar")  // "foobar" of type MyString
string([]byte{'a'})      // not a constant: []byte{'a'} is not a constant
(*int)(nil)              // not a constant: nil is not a constant, *int is not a boolean, numeric, or string type
int(1.2)                 // illegal: 1.2 cannot be represented as an int
string(65.0)             // illegal: 65.0 is not an integer constant
```

在以下任何一种情况下，非常量值 x 都可以转换为 T 类型：

- x 可[赋值](#assignability)给 T。
- 忽略结构体标签（见下文），x 的类型和 T 具有[相同的底层类型](#types)。
- 忽略结构体标签（见下文），x 的类型和 T 是未[定义类型](#type-definitions)的指针类型，它们的指针基类型具有相同的底层类型。
- x 的类型和 T 都是整数或浮点数类型。
- x 的类型和 T 都是复数类型。
- x 是一个整数、一个 bytes 或 runes 切片，且 T 是一个字符串类型。
- x 是一个字符串，且 T 是一个 bytes 或 runes 切片。
- x 是切片，T 是指向数组的指针，切片和数组类型具有[相同](#type-identity)的元素类型。

出于转换目的来比较结构体类型时，结构标签将被忽略：

```
type Person struct {
	Name    string
	Address *struct {
		Street string
		City   string
	}
}

var data *struct {
	Name    string `json:"name"`
	Address *struct {
		Street string `json:"street"`
		City   string `json:"city"`
	} `json:"address"`
}

var person = (*Person)(data)  // ignoring tags, the underlying types are identical
```

一些适用于数字类型之间或与字符串类型之间（非常量）转换的特殊规则。这些转换可能会改变 x 的表示并有运行时成本。所有其他转换仅更改类型而不更改 x 的表示。

没有在指针和整数之间转换的语言机制。包 unsafe 在受限情况下实现此功能。

#### Conversions between numeric types

对于非常量数值的转换，以下规则适用：

1. 在整数类型之间转换时，如果值为有符号整数，则符号扩展为隐式无限精度；否则为零扩展。然后将其截断以适应结果类型的大小。例如，如果 v := uint16(0x10F0)，则 uint32(int8(v)) ==
   0xFFFFFFF0。转换总是产生一个有效的值；没有溢出的迹象。
2. 将浮点数转换为整数时，分子将被丢弃（向零截断）。
3. 将整数或浮点数转换为浮点类型，或将复数转换为另一种复杂类型时，结果值将四舍五入到目标类型指定的精度。例如，float32 类型的变量 x 的值可以使用超出 IEEE-754 32 位数字的精度的额外精度进行存储，但 float32(
   x) 表示将 x 的值四舍五入到 32 位精度的结果。类似地，x + 0.1 可能使用超过 32 位的精度，但 float32(x + 0.1) 不会。

在所有涉及浮点数或复数值的非常量转换中，如果结果类型不能表示转换成功但结果值取决于实现的值。

#### Conversions to and from a string type

1. 将有符号或无符号整数值转换为字符串类型会生成一个包含整数的 UTF-8 表示的字符串。超出有效 Unicode 代码点范围的值将转换为 "\uFFFD"。

   ```
   string('a')       // "a"
   string(-1)        // "\ufffd" == "\xef\xbf\xbd"
   string(0xf8)      // "\u00f8" == "ø" == "\xc3\xb8"
   type MyString string
   MyString(0x65e5)  // "\u65e5" == "日" == "\xe6\x97\xa5"
   ```

2. 将字节切片转换为字符串类型会产生一个字符串，其连续字节是切片的元素。

   ```
   string([]byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'})   // "hellø"
   string([]byte{})                                     // ""
   string([]byte(nil))                                  // ""
   
   type MyBytes []byte
   string(MyBytes{'h', 'e', 'l', 'l', '\xc3', '\xb8'})  // "hellø"
   ```

3. 将符文切片转换为字符串类型会产生一个字符串，该字符串是各个符文值的串联。

   ```
   string([]rune{0x767d, 0x9d6c, 0x7fd4})   // "\u767d\u9d6c\u7fd4" == "白鵬翔"
   string([]rune{})                         // ""
   string([]rune(nil))                      // ""
   
   type MyRunes []rune
   string(MyRunes{0x767d, 0x9d6c, 0x7fd4})  // "\u767d\u9d6c\u7fd4" == "白鵬翔"
   ```

4. 将字符串类型的值转换为字节类型的切片会产生一个切片，其连续元素是字符串的字节。

   ```
   []byte("hellø")   // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
   []byte("")        // []byte{}
   
   MyBytes("hellø")  // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
   ```

5. 将字符串类型的值转换为符文类型的切片会产生一个包含字符串的各个 Unicode 代码点的切片。

   ```
   []rune(MyString("白鵬翔"))  // []rune{0x767d, 0x9d6c, 0x7fd4}
   []rune("")                 // []rune{}
   
   MyRunes("白鵬翔")           // []rune{0x767d, 0x9d6c, 0x7fd4}
   ```

#### Conversions from slice to array pointer

将切片转换为数组指针会产生一个指向切片底层数组的指针。如果切片的[长度](#length-and-capacity)小于数组的长度，则会发生[运行时恐慌](#run-time-panics)。

```
s := make([]byte, 2, 4)
s0 := (*[0]byte)(s)      // s0 != nil
s1 := (*[1]byte)(s[1:])  // &s1[0] == &s[1]
s2 := (*[2]byte)(s)      // &s2[0] == &s[0]
s4 := (*[4]byte)(s)      // panics: len([4]byte) > len(s)

var t []string
t0 := (*[0]string)(t)    // t0 == nil
t1 := (*[1]string)(t)    // panics: len([1]string) > len(t)

u := make([]byte, 0)
u0 = (*[0]byte)(u)       // u0 != nil
```

### Constant expressions

常量表达式可能只包含[常量](#constants)作为操作数并在编译时计算。

无类型的布尔值、数字、字符串常量可以用作操作数，只要使用布尔值、数字或字符串类型的操作数是合法的。

常量[比较](#comparison-operators)总是产生一个无类型的布尔常量。如果常量[移位表达式](#operators)
的左操作数是无类型常量，则结果为整型常量；否则它是一个与左操作数相同类型的常量，它必须是[整数类型](#numeric-types)。

对无类型常量的任何其他操作都会产生相同类型的无类型常量；即，布尔值、整数、浮点数、复数或字符串常量。
如果二元运算（移位除外）的无类型操作数属于不同类型，则结果属于此列表后面出现的操作数类型：整数、符文、浮点数、复数。例如，无类型整数常量除以无类型复合常量会产生无类型复合常量。

```
const a = 2 + 3.0          // a == 5.0   (untyped floating-point constant)
const b = 15 / 4           // b == 3     (untyped integer constant)
const c = 15 / 4.0         // c == 3.75  (untyped floating-point constant)
const Θ float64 = 3/2      // Θ == 1.0   (type float64, 3/2 is integer division)
const Π float64 = 3/2.     // Π == 1.5   (type float64, 3/2. is float division)
const d = 1 << 3.0         // d == 8     (untyped integer constant)
const e = 1.0 << 3         // e == 8     (untyped integer constant)
const f = int32(1) << 33   // illegal    (constant 8589934592 overflows int32)
const g = float64(2) >> 1  // illegal    (float64(2) is a typed floating-point constant)
const h = "foo" > "bar"    // h == true  (untyped boolean constant)
const j = true             // j == true  (untyped boolean constant)
const k = 'w' + 1          // k == 'x'   (untyped rune constant)
const l = "hi"             // l == "hi"  (untyped string constant)
const m = string(k)        // m == "x"   (type string)
const Σ = 1 - 0.707i       //            (untyped complex constant)
const Δ = Σ + 2.0e-4       //            (untyped complex constant)
const Φ = iota*1i - 1/1i   //            (untyped complex constant)
```

对无类型整数、符文或浮点数常量使用内置函数 complex 会生成无类型复数常量。

```
const ic = complex(0, c)   // ic == 3.75i  (untyped complex constant)
const iΘ = complex(0, Θ)   // iΘ == 1i     (type complex128)
```

常量表达式总是精确求值；中间值和常量本身可能需要比语言中任何预先声明的类型所支持的精度大得多的精度。以下是合法声明：

```
const Huge = 1 << 100         // Huge == 1267650600228229401496703205376  (untyped integer constant)
const Four int8 = Huge >> 98  // Four == 4                                (type int8)
```

常量除法或余数运算的除数不能为零：

```
3.14 / 0.0   // illegal: division by zero
```

类型常量的值必须始终可以由常量类型进行准确[表示](#representability)。以下常量表达式是非法的：

```
uint(-1)     // -1 cannot be represented as a uint
int(3.14)    // 3.14 cannot be represented as an int
int64(Huge)  // 1267650600228229401496703205376 cannot be represented as an int64
Four * 300   // operand 300 cannot be represented as an int8 (type of Four)
Four * 100   // product 400 cannot be represented as an int8 (type of Four)
```

一元按位补码运算符 ^ 使用的掩码匹配非常量的规则：无符号常量的掩码全为 1，有符号和无类型常量的掩码全为 -1。

```
^1         // untyped integer constant, equal to -2
uint8(^1)  // illegal: same as uint8(-2), -2 cannot be represented as a uint8
^uint8(1)  // typed uint8 constant, same as 0xFF ^ uint8(1) = uint8(0xFE)
int8(^1)   // same as int8(-2)
^int8(1)   // same as -1 ^ int8(1) = -2
```

实现限制：编译器在计算无类型浮点数或复数常量表达式时可能会舍入；请参阅[常量](#constants)部分中的实现限制。这种舍入可能导致浮点数常量表达式在整数上下文中无效，即使它在使用无限精度计算时是整数，反之亦然。

### Order of evaluation

在包级别，[初始化依赖](#package-initialization)决定了[变量声明](#variable-declarations)中各个初始化表达式的求值顺序。另外，在计算表达式的[操作数](#operands)
、赋值、[return 语句](#return-statements)、所有函数调用、方法调用、通信操作都按从左到右的顺序计算。

例如，在 (function-local) 赋值中

```
y[f()], ok = g(h(), i()+x[j()], <-c), k()
```

函数调用以 f()、h()、i()、j()、<-c、g() 和 k() 的顺序发生。但是，x 的索引以及 y 的计算，这些事件的顺序没有指定。

```
a := 1
f := func() int { a++; return a }
x := []int{a, f()}            // x may be [1, 2] or [2, 2]: evaluation order between a and f() is not specified
m := map[int]int{a: 1, a: 2}  // m may be {2: 1} or {2: 2}: evaluation order between the two map assignments is not specified
n := map[int]int{a: f()}      // n may be {2: 3} or {3: 3}: evaluation order between the key and the value is not specified
```

在包级别，初始化依赖覆盖单个初始化表达式的从左到右规则，但不覆盖每个表达式中的操作数：

```
var a, b, c = f() + v(), g(), sqr(u()) + v()

func f() int        { return c }
func g() int        { return a }
func sqr(x int) int { return x*x }

// functions u and v are independent of all other variables and functions
```

函数调用按 u()、sqr()、v()、f()、v() 和 g() 的顺序发生。

单个表达式中的浮点运算根据运算符的结合性进行计算。显式括号会覆盖默认的优先级。在表达式 x + (y + z) 中，加法 y + z 在 + x 之前执行。

## Statements

语句控制执行。

```
Statement =
	Declaration | LabeledStmt | SimpleStmt |
	GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
	FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
	DeferStmt .

SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | Assignment | ShortVarDecl .
```

### Terminating statements

*终止语句*阻止在同一[块](#blocks)中上出现在它之后所有语句的执行。以下是终止语句：

1. "[return](#return-statements)" 或 "[goto](#goto-statements)" 语句。
2. 内置函数 panic 的调用。
3. 语句列表以终止语句结尾的[块](#blocks)。
4. 一个 "[if](#if-statements)" 语句，其中：
    - 存在 "else" 分支，并且
    - 两个分支都是终止语句。
5. 一个 "[for](#for-statements)" 语句，其中：
    - 没有涉及到 "for" 语句的 "break" 语句，并且
    - 循环条件不存在。
6. 一个 "[switch](#switch-statements)" 语句，其中：
    - 没有涉及到 "switch" 语句的 "break" 语句，
    - 有一个默认 case ，并且
    - 该语句列出了每种情况，包括默认值、以终止语句或可能为 "[fallthrough](#fallthrough-statements)" 语句结尾。
7. 一个 "[select](#select-statements)" 语句，其中：
    - 没有涉及到 "select" 语句的 "break" 语句，并且
    - 每种情况下的语句列表，包括默认值（如果存在），都以终止语句结尾。
8. 标记终止语句的[标签](#labeled-statements)语句。

所有其他语句均不终止执行。

如果语句列表不为空且其最后的非空语句是终止语句，则[语句列表](#blocks)以终止语句结尾。

### Empty statements

空语句什么也不做。

```
EmptyStmt = .
```

### Labeled statements

标签语句可能是 goto、break 或 continue 语句的目标。

```
LabeledStmt = Label ":" Statement .
Label       = identifier .
Error: log.Panic("error encountered")
```

### Expression statements

一些内置函数、函数、[方法调用](#calls)、[接收操作](#receive-operator)都可以出现在语句上下文中。此类语句可以用括号括起来。

```
ExpressionStmt = Expression .
```

语句上下文中不允许使用以下内置函数：

```
append cap complex imag len make new real
unsafe.Add unsafe.Alignof unsafe.Offsetof unsafe.Sizeof unsafe.Slice

h(x+y)
f.Close()
<-ch
(<-ch)
len("foo")  // illegal if len is the built-in function
```

### Send statements

发送语句在通道上发送一个值。通道表达式必须是[通道类型](#channel-types)，通道的方向必须允许发送操作，并且要发送的值的类型必须可[赋值](#assignability)给通道元素。

```
SendStmt = Channel "<-" Expression .
Channel  = Expression .
```

在通信开始之前应该评估通道和值表达式。通信会一直阻塞直到发送可以继续。如果接收器准备好，则可以继续在无缓冲通道上发送。如果缓冲区中有空间，则可以继续在缓冲通道上发送。在一个已关闭通道上的发送会导致[运行时恐慌](#run-time-panics)
。在 nil 通道上发送永远阻塞。

```
ch <- 3  // send value 3 to channel ch
```

### IncDec statements

"++" 和 "--" 语句通过无类型[常量](#constants) 1 递增或递减它们的操作数。与赋值一样，操作数必须是[可寻址的](#address-operators)或映射索引表达式。

```
IncDecStmt = Expression ( "++" | "--" ) .
```

以下[赋值语句](#assignments)在语义上是相等的：

```
IncDec statement    Assignment
x++                 x += 1
x--                 x -= 1
```

### Assignments

```
Assignment = ExpressionList assign_op ExpressionList .

assign_op = [ add_op | mul_op ] "=" .
```

每个左侧操作数必须是[可寻址的](#address-operators)、映射键表达式或空白标识符（仅用于 = 赋值）。操作数可以用括号括起来。

```
x = 1
*p = f()
a[i] = 23
(k) = <-ch  // same as: k = <-ch
```

*赋值操作* x op= y 其中 op 是二元[算术运算符](#arithmetic-operators)等价于 x = x op (y) 但只对 x 求值一次。*op=*
是单独的标记。在赋值操作中，左边和右边的表达式列表都必须包含一个单值表达式，并且左边的表达式不能是空白标识符。

```
a[i] <<= 2
i &^= 1<<n
```

多值赋值将多值表达式的各个元素分配给变量。有两种形式。在第一种情况下，右侧操作数是单个多值表达式，例如函数调用、[通道](#channel-types)或[映射](#map-types)操作或[类型断言](#type-assertions)
。左侧操作数的数量必须与返回的数量相匹配。例如，如果 f 是一个返回两个值的函数，

```
x, y = f()
```

将第一个值分配给 x，将第二个值分配给 y。第二种形式，左边的操作数必须等于右边的表达式数，每个表达式都必须是单值的，右边的第 n 个表达式赋值给左边的第 n 个操作数：

```
one, two, three = '一', '二', '三'
```

[空白标识符](#blank-identifier)提供了一种忽略赋值中右侧值的方法：

```
_ = x       // evaluate x but ignore it
x, _ = f()  // evaluate f() but ignore second result value
```

赋值分两个阶段进行。首先，左边的[索引表达式](#index-expressions)和[间接指针](#address-operators)（包括[选择器](#selectors)
中的隐式间接指针）的操作数和右边的表达式都按[求值顺序](#order-of-evaluation)计算。其次，按从左到右的顺序进行赋值。

```
a, b = b, a  // exchange a and b

x := []int{1, 2, 3}
i := 0
i, x[i] = 1, 2  // set i = 1, x[0] = 2

i = 0
x[i], i = 2, 1  // set x[0] = 2, i = 1

x[0], x[0] = 1, 2  // set x[0] = 1, then x[0] = 2 (so x[0] == 2 at end)

x[1], x[3] = 4, 5  // set x[1] = 4, then panic setting x[3] = 5.

type Point struct { x, y int }
var p *Point
x[2], p.x = 6, 7  // set x[2] = 6, then panic setting p.x = 7

i = 2
x = []int{3, 5, 7}
for i, x[i] = range x {  // set i, x[2] = 0, x[0]
	break
}
// after this loop, i == 0 and x == []int{3, 5, 3}
```

在赋值操作中，每个值都必须可以[赋值](#assignability)给对应的操作数，有以下特殊情况：

1. 任何类型的值都可以赋值给空白标识符。
2. 如果将无类型常量赋值给接口类型的变量或空白标识符，常量将首先被隐式[转换](#conversions)为其[默认类型](#constants)。
3. 如果将无类型的布尔值赋值给一个接口类型的变量或空白标识符，它将首先被隐式转换为 bool 类型。

### If statements

"If" 语句根据布尔表达式的值在两个分支的条件中选择一个进行执行。如果表达式计算结果为 true，则执行 "if" 分支，否则，执行 "else" 分支（如果其存在）。

```
IfStmt = "if" [ SimpleStmt ";" ] Expression Block [ "else" ( IfStmt | Block ) ] .
if x > max {
	x = max
}
```

表达式前面可能有一个简单的语句，该语句在表达式计算之前执行。

```
if x := f(); x < y {
	return x
} else if x > z {
	return z
} else {
	return y
}
```

### Switch statements

"Switch" 语句提供 multi-way 执行方式。将 "switch" 中的表达式或类型与 "cases" 进行比较，以确定要执行的分支。

```
SwitchStmt = ExprSwitchStmt | TypeSwitchStmt .
```

有两种形式：expression switches 和 type switches 。在 expression switch 中，case 与 switch 中表达式的值进行比较。在 type switch 中，case 与 switch
中表达式的类型进行比较。switch 表达式在 switch 语句中只计算一次。

#### Expression switches

在 expression switch 中，对 switch 表达式求值，case 表达式（不必为常量）按从左到右和从上到下的顺序求值；第一个等于 switch 表达式的 case 语句将触发执行；其他 case 将被跳过。如果没有
case 匹配并且存在 "default"  case，则执行 default 语句。最多可以有一个 default 语句，它可能出现在 "switch" 语句中的任何地方。缺少 switch 表达式等效于布尔值 true。

```
ExprSwitchStmt = "switch" [ SimpleStmt ";" ] [ Expression ] "{" { ExprCaseClause } "}" .
ExprCaseClause = ExprSwitchCase ":" StatementList .
ExprSwitchCase = "case" ExpressionList | "default" .
```

如果 switch 表达式的计算结果为无类型常量，首先将其隐式[转换](#conversions)为其[默认类型](#constants)。预先声明的无类型值 nil 不能用作 switch 表达式。switch
表达式的类型必须是[可比较的](#comparison-operators)。

如果 case 表达式是无类型的，首先将其隐式[转换](#conversions)为 switch 表达式的类型。对于每个（可能已转换的）case 表达式 x 和 switch 表达式的值 t，x == t
必须[可比较的](#comparison-operators)。

换句话说，switch 表达式被视为用于声明和初始化一个没有显式类型的临时变量 t；用于测试每个 case 表达式的值 x 是否于 t 相等。

在 case 或 default 子句中，最后一个非空语句可能是（可能被[标记](#labeled-statements)）"[fallthrough](#fallthrough-statements)"
语句，以表明执行应从该子句的末尾流向下一个子句的第一个语句。否则执行应流到 "switch" 语句的末尾。"fallthrough"
语句可能作为 switch 表达式所有 case 语句的最后一个语句出现（除了最后一个 case）。

switch 表达式前面可能有一个简单的语句，该语句在表达式计算之前执行。

```
switch tag {
default: s3()
case 0, 1, 2, 3: s1()
case 4, 5, 6, 7: s2()
}

switch x := f(); {  // missing switch expression means "true"
case x < 0: return -x
default: return x
}

switch {
case x < y: f1()
case x < z: f2()
case x == 4: f3()
}
```

实现限制：编译器可能不允许多个 case 表达式对同一个常量求值。例如，目前编译器不允许在 case 表达式中出现重复的整数、浮点数或字符串常量。

#### Type switches

type switch 比较的是类型而不是值。在其他方面类似于 expression switch，使用一个特殊的 switch 表达式，该表达式使用关键字 type 而不是一个实际类型进行[类型断言](#type-assertions)：

```
switch x.(type) {
// cases
}
```

然后，case 将实际类型 T 与表达式 x 的动态类型进行匹配。与类型断言一样，x 必须是[接口类型](#interface-types)，并且 case 中列出的每个非接口类型 T 都必须实现 x 类型。在 type switch 中
case 的类型必须全部[不同](#type-identity)。

```
TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard "{" { TypeCaseClause } "}" .
TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
TypeCaseClause  = TypeSwitchCase ":" StatementList .
TypeSwitchCase  = "case" TypeList | "default" .
TypeList        = Type { "," Type } .
```

TypeSwitchGuard 可能包含一个[短变量声明](#short-variable-declarations)。使用该形式时，变量在每个 TypeSwitchCase 子句末尾的[隐式块](#blocks)中声明。在 case
只列出一种类型的子句中，变量为该类型；否则，该变量为 TypeSwitchGuard 中表达式的类型。

在 case 中可以使用预先声明的标识符 nil 代替类型；当 TypeSwitchGuard 中的表达式为 nil 接口值时，会选择这种 case。最多可能有一个 nil。

给定一个类型为 interface{} 的表达式 x，下面的 type switch：

```
switch i := x.(type) {
case nil:
	printString("x is nil")                // type of i is type of x (interface{})
case int:
	printInt(i)                            // type of i is int
case float64:
	printFloat64(i)                        // type of i is float64
case func(int) float64:
	printFunction(i)                       // type of i is func(int) float64
case bool, string:
	printString("type is bool or string")  // type of i is type of x (interface{})
default:
	printString("don't know the type")     // type of i is type of x (interface{})
}
```

可以改写成：

```
v := x  // x is evaluated exactly once
if v == nil {
	i := v                                 // type of i is type of x (interface{})
	printString("x is nil")
} else if i, isInt := v.(int); isInt {
	printInt(i)                            // type of i is int
} else if i, isFloat64 := v.(float64); isFloat64 {
	printFloat64(i)                        // type of i is float64
} else if i, isFunc := v.(func(int) float64); isFunc {
	printFunction(i)                       // type of i is func(int) float64
} else {
	_, isBool := v.(bool)
	_, isString := v.(string)
	if isBool || isString {
		i := v                         // type of i is type of x (interface{})
		printString("type is bool or string")
	} else {
		i := v                         // type of i is type of x (interface{})
		printString("don't know the type")
	}
}
```

type switch guard 可以在一个简单的语句之前，该语句在 guard 语句执行之前执行。

type switch 中不允许使用 "fallthrough" 语句。

### For statements

"for" 语句表示块的重复执行。共有三种形式：迭代可以由单个条件、"for" 子句或 "range" 子句控制。

```
ForStmt = "for" [ Condition | ForClause | RangeClause ] Block .
Condition = Expression .
```

#### For statements with single condition

最简单的形式，"for" 语句表示只要条件计算为 true 就重复执行块。在每次迭代之前计算条件。如果条件不存在，则相当于布尔值 true。

```
for a < b {
	a *= 2
}
```

#### For statements with for clause

带有 ForClause 的 "for" 语句也受其条件控制，但另外还可以指定 *init* 和 *post* 语句，例如赋值、增量或减量语句。init 语句可以是一个简短的变量声明，但 post 语句不能。init
语句声明的变量在每次迭代中都会重复使用。

```
ForClause = [ InitStmt ] ";" [ Condition ] ";" [ PostStmt ] .
InitStmt = SimpleStmt .
PostStmt = SimpleStmt .
for i := 0; i < 10; i++ {
	f(i)
}
```

如果非空，则在计算第一次迭代的条件之前执行一次 init 语句；post 语句在每次执行块后执行（并且仅当块被执行时）。ForClause 的任何元素都可以为空，但除非只有一个条件，否则[分号](#semicolons)
是必需的。如果条件不存在，则相当于布尔值 true。

```
for cond { S() }    is the same as    for ; cond ; { S() }
for      { S() }    is the same as    for true     { S() }
```

#### For statements with range clause

带有 "range" 子句的 "for" 语句遍历数组、切片、字符串、映射的所有元素、通道上接收到的值。对于每个元素，它会将*迭代值*赋值给相应的*迭代变量*（如果存在），然后执行该块。

```
RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range" Expression .
```

"range" 子句中右侧的表达式称为 *range 表达式*，它可以是数组、指向数组的指针、切片、字符串、映射、允许[接收操作](#receive-operator)
的通道。与赋值一样，如果左侧的操作数存在，必须是[可寻址的](#address-operators)或映射索引表达式；它们表示迭代变量。如果 range
表达式是一个通道，则最多允许一个迭代变量，否则最多可能有两个。如果最后一个迭代变量是[空白标识符](#blank-identifier)，则 range 子句等效于没有该标识符的相同子句。

range 表达式 x 在循环开始前计算一次，但有一个例外：如果最多存在一个迭代变量并且 len(x) 是[常量](#length-and-capacity)，则不计算 range 表达式。

左侧的函数调用每次迭代计算一次。对于每次迭代，如果存在相应的迭代变量，则迭代值生成如下：

```
Range expression                          1st value          2nd value

array or slice  a  [n]E, *[n]E, or []E    index    i  int    a[i]       E
string          s  string type            index    i  int    see below  rune
map             m  map[K]V                key      k  K      m[k]       V
channel         c  chan E, <-chan E       element  e  E
```

1. 对于数组、指向数组的指针或切片 a，索引迭代值按递增顺序生成，从元素索引 0 开始。如果最多存在一个迭代变量，则 range 循环从 0 到 len(a)-1 生成值且不索引到数组或切片本身。对于 nil 切片，迭代次数为 0。
2. 对于字符串值，"range" 子句从字节索引 0 开始迭代字符串中的 Unicode 代码点。在连续迭代中，索引值将是连续 UTF-8 编码代码点的第一个字节的索引字符串和符文类型的第二个值将是相应代码点的值。如果迭代遇到无效的
   UTF-8 序列，则第二个值将是 0xFFFD，即 Unicode 替换字符，下一次迭代将在字符串中前进一个字节。
3. 映射上的迭代顺序没有指定，不能保证每次迭代都是相同的。如果在迭代过程中删除了尚未访问的映射条目，则不会产生相应的迭代值。如果在迭代期间创建了映射条目，则该条目可能在迭代期间产生或可能被跳过。
   对于创建的每个条目以及从一个迭代到下一个迭代，选择可能会有所不同。如果映射为 nil，则迭代次数为 0。
4. 对于通道，产生的迭代值是在通道上发送的连续值，直到通道[关闭](#close)。如果通道为 nil，则 range 表达式将永远阻塞。

迭代值被赋值给相应的迭代变量，就像在[赋值语句](#assignments)中一样。

在 "range" 子句声明的迭代变量可以使用[短变量声明](#short-variable-declarations) (:=)
。在这种情况下，它们的类型被设置为各自迭代值的类型，[作用域](#declarations-and-scope)是 "for" 语句的块；它们在每次迭代中都被重复使用。如果迭代变量是在 "for"
语句之外声明的，则它们的值将是最后一次迭代的值。

```
var testdata *struct {
	a *[7]int
}
for i, _ := range testdata.a {
	// testdata.a is never evaluated; len(testdata.a) is constant
	// i ranges from 0 to 6
	f(i)
}

var a [10]string
for i, s := range a {
	// type of i is int
	// type of s is string
	// s == a[i]
	g(i, s)
}

var key string
var val interface{}  // element type of m is assignable to val
m := map[string]int{"mon":0, "tue":1, "wed":2, "thu":3, "fri":4, "sat":5, "sun":6}
for key, val = range m {
	h(key, val)
}
// key == last map key encountered in iteration
// val == map[key]

var ch chan Work = producer()
for w := range ch {
	doWork(w)
}

// empty a channel
for range ch {}
```

### Go statements

"go" 语句在同一地址空间内启动一个独立的并发控制线程（*goroutine*）来执行调用函数。

```
GoStmt = "go" Expression .
```

表达式必须是函数或方法调用；不能用括号括起来。内置函数的调用受限于[表达式语句](#expression-statements)。

函数值和参数在调用 goroutine 中[照常计算](#calls)，但与常规调用不同，程序执行不会等待调用的函数执行完成。相反，该函数在新的 goroutine 中独立执行。当函数终止时，它的 goroutine
也会终止。如果函数有任何返回值，它们会在函数完成时被丢弃。

```
go Server()
go func (ch chan<- bool) { for { sleep(10); ch <- true }} (c)
```

### Select statements

"select" 语句在一组可能的[发送](#send-statements)或[接收](#receive-operator)操作中选择一个来执行。它看起来类似于 "switch" 语句，但所有情况都涉及通信操作。

```
SelectStmt = "select" "{" { CommClause } "}" .
CommClause = CommCase ":" StatementList .
CommCase   = "case" ( SendStmt | RecvStmt ) | "default" .
RecvStmt   = [ ExpressionList "=" | IdentifierList ":=" ] RecvExpr .
RecvExpr   = Expression .
```

带有 RecvStmt 的 case 可以将 RecvExpr 的结果赋值给一个或两个变量，这些变量可以使用[短变量声明](#short-variable-declarations)来声明。RecvExpr
必须是（可能带括号的）接收操作。最多可以有一个默认 case ，它可能出现在 case 列表中的任何地方。

"select" 语句的执行分几个步骤：

1. 对于语句中的所有 case，通道的接收操作、通道、发送语句的右侧表达式在进入 "select"
   语句时按顺序仅计算一次。结果是一组要接收或发送的通道，以及要发送的相应值。无论选择哪个（如果有）通信操作来执行，该计算中的任何副作用都会发生。带有简短变量声明或赋值的 RecvStmt 左侧的表达式尚未计算。
2. 如果可以执行一个或多个通信，则通过统一伪随机选择来选取一个来进行通信。否则，如果存在 default case，则选择该 case。如果没有 default case，"select" 语句会阻塞，直到至少有一个通信可以继续。
3. 除非选定的 case 是 default case ，否则将执行相应的通信操作。
4. 如果选定的 case 是使用短变量声明或赋值的 RecvStmt，则计算左侧表达式并赋值接收到的值（或多个值）。
5. 被选择的 case 语句将被执行。

由于 nil 通道上的通信永远无法进行，因此只有 nil 通道且没有 default case 的选择会永远阻塞。

```
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
case i1 = <-c1:
	print("received ", i1, " from c1\n")
case c2 <- i2:
	print("sent ", i2, " to c2\n")
case i3, ok := (<-c3):  // same as: i3, ok := <-c3
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// same as:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}

for {  // send random sequence of bits to c
	select {
	case c <- 0:  // note: no statement, no fallthrough, no folding of cases
	case c <- 1:
	}
}

select {}  // block forever
```

### Return statements

函数 F 中的 "return" 语句终止 F 的执行，并可选择是否提供一个或多个结果值。任何被 F [延迟](#defer-statements)的函数都会在 F 返回到它的调用者之前执行。

```
ReturnStmt = "return" [ ExpressionList ] .
```

在没有返回类型的函数中，"return" 语句不得指定任何返回参数。

```
func noResult() {
	return
}
```

有三种方法可以从具有返回类型的函数中返回值：

1. 返回值（或值们）可能在 "return" 语句中明确列出。每个表达式必须是 single-valued 并且可以[赋值](#assignability)给函数返回参数的相应类型。

   ```
   func simpleF() int {
       return 2
   }
    
   func complexF1() (re float64, im float64) {
       return -7.0, -4.0
   }
   ```

2. "return" 语句中的表达式可能是对多返回值函数的调用。就好像从该函数返回的每个值都被分配给一个具有相应类型的临时变量，然后是一个列出这些变量的 "return" 语句，此时适用于前一种规则。

   ```
   func complexF2() (re float64, im float64) {
       return complexF1()
   }
   ```

3. 如果函数为[返回参数](#function-types)指定了名称，则表达式列表可以为空。返回参数是普通的局部变量，函数可以根据需要为它们赋值。"return" 语句返回这些变量的值。

   ```
   func complexF3() (re float64, im float64) {
   	re = 7.0
   	im = 4.0
   	return
   }
   
   func (devnull) Write(p []byte) (n int, _ error) {
   	n = len(p)
   	return
   }
   ```

不管它们是如何声明的，所有返回值在进入函数时都被初始化为其类型的[零值](#the-zero-value)。指定结果的 "return" 语句在执行任何延迟函数之前设置返回参数。

实现限制：与返回参数同名的不同实体（常量、类型或变量）在返回位置的[作用域](#declarations-and-scope)内，则编译器可能不允许 "return" 语句中为空表达式。

```
func f(n int) (res int, err error) {
	if _, err := f(n-1); err != nil {
		return  // invalid return statement: err is shadowed
	}
	return
}
```

### Break statements

"break" 语句终止同一函数内最里面的 "[for](#for-statements)"、"[switch](#switch-statements)" 或 "[select](#select-statements)" 语句的执行。

```
BreakStmt = "break" [ Label ] .
```

如果有一个标签，它必须是一个封闭的 "for"、"switch" 或 "select" 语句的标签，并且它就是用来执行终止的语句。

```
OuterLoop:
	for i = 0; i < n; i++ {
		for j = 0; j < m; j++ {
			switch a[i][j] {
			case nil:
				state = Error
				break OuterLoop
			case item:
				state = Found
				break OuterLoop
			}
		}
	}
```

### Continue statements

"continue" 语句在其 post 语句处开始最内层 "[for](#for-statements)" 循环的下一次迭代。"for" 循环必须在同一个函数中。

```
ContinueStmt = "continue" [ Label ] .
```

如果有一个标签，它必须是一个封闭的 "for" 语句的标签，并且它就是执行前进的语句。

```
RowLoop:
	for y, row := range rows {
		for x, data := range row {
			if data == endOfRow {
				continue RowLoop
			}
			row[x] = data + bias(x, y)
		}
	}
```

### Goto statements

"goto" 语句将控制权转移到同一函数内具有相应标签的语句。

```
GotoStmt = "goto" Label .
goto Error
```

执行 "goto" 语句不得导致任何变量进入作用域，这些变量在 goto 前的[作用域](#declarations-and-scope)声明。例如，这个例子：

```
	goto L  // BAD
	v := 3
L:
```

是错误的，因为跳转到标签 L 跳过了 v 的创建。

[块](#blocks)外的 "goto" 语句不能跳转到该块内的标签。例如，这个例子：

```
if n%2 == 1 {
	goto L1
}
for n > 0 {
	f()
	n--
L1:
	f()
	n--
}
```

是错误的，因为标签 L1 在 "for" 语句的块内，但 goto 不在。

### Fallthrough statements

"fallthrough" 语句将控制转移到 ["switch" 表达式](#expression-switches)中下一个 case 子句的第一个语句。它只能用作此类子句中的最后一个非空语句。

```
FallthroughStmt = "fallthrough" .
```

### Defer statements

使用 "defer" 语句调用一个函数，会将该函数的执行推迟到周围函数返回的那一刻，当周围函数执行了一个 [return 语句](#return-statements)、到达了它的[函数体](#function-declarations)
的末尾、对应的 goroutine 发生了[恐慌](#handling-panics)。

```
DeferStmt = "defer" Expression .
```

表达式必须是函数或方法调用；不能用括号括起来。内置函数的调用受限于[表达式语句](#expression-statements)。

每次执行 "defer" 语句时，函数值和调用的参数都会[像往常一样计算](#calls)
并重新保存，但不会调用实际的函数。而是在周围函数返回之前以延迟的相反顺序调用延迟函数。也就是说，如果周围的函数通过显式 [return 语句](#return-statements) 返回，则在该 return
语句设置任何返回参数之后但在函数返回到其调用者之前执行延迟函数。如果延迟函数值计算为 nil，则在调用函数时执行会出现[恐慌](#handling-panics)，而不是在执行 "defer" 语句时。

例如，如果延迟函数是[函数字面量](#function-literals)，并且周围的函数中有在字面量作用域内的[命名返回参数](#function-types)
，则延迟函数可以在返回之前访问和修改返回值。如果延迟函数有任何返回值，则在函数完成时将其丢弃。（另请参阅有关[处理恐慌](#handling-panics)部分。）

```
lock(l)
defer unlock(l)  // unlocking happens before surrounding function returns

// prints 3 2 1 0 before surrounding function returns
for i := 0; i <= 3; i++ {
	defer fmt.Print(i)
}

// f returns 42
func f() (result int) {
	defer func() {
		// result is accessed after it was set to 6 by the return statement
		result *= 7
	}()
	return 6
}
```

## Built-in functions

内置函数是[预先声明的](#predeclared-identifiers)。它们像任何其他函数一样被调用，但其中一些的第一个参数是类型而不是表达式。

内置函数没有标准的 Go 类型，所以只能出现在[调用表达式](#calls)中；它们不能用作函数值。

### Close

对于通道 c，内置函数 close(c) 标志不会在通道上发送更多值。如果 c 是只接收通道，这是一个错误。发送到或关闭已关闭的通道会导致[运行时恐慌](#run-time-panics)。关闭 nil
通道也会导致[运行时恐慌](#run-time-panics)。在调用 close 之后，且在接收到任何先前发送的值之后，接收操作将在不阻塞的情况下返回通道类型的零值。多值[接收操作](#receive-operator)
返回接收到的值以及通道是否关闭的标志。

### Length and capacity

内置函数 len 和 cap 接受各种类型的参数并返回 int 类型的值。实现保证返回结果总是一个 int 。

```
Call      Argument type    Result

len(s)    string type      string length in bytes
          [n]T, *[n]T      array length (== n)
          []T              slice length
          map[K]T          map length (number of defined keys)
          chan T           number of elements queued in channel buffer

cap(s)    [n]T, *[n]T      array length (== n)
          []T              slice capacity
          chan T           channel buffer capacity
```

切片的容量是在底层数组中分配空间的元素的数量。在任何时候，以下关系都成立：

```
0 <= len(s) <= cap(s)
```

nil 切片、映射、通道的长度为 0。nil 切片、映射、通道的容量为 0。

如果 s 是字符串常量，则表达式 len(s) 是[常量](#constants)。如果 s 的类型是数组或指向数组的指针，并且表达式 s 不包含[通道接收](#receive-operator)或（非常量）[函数调用](#calls)
，则表达式 len(s) 和 cap(s) 是常量；在这种情况下，不计算 s。否则，len 和 cap 的调用不是常量，s 会被计算。

```
const (
	c1 = imag(2i)                    // imag(2i) = 2.0 is a constant
	c2 = len([10]float64{2})         // [10]float64{2} contains no function calls
	c3 = len([10]float64{c1})        // [10]float64{c1} contains no function calls
	c4 = len([10]float64{imag(2i)})  // imag(2i) is a constant and no function call is issued
	c5 = len([10]float64{imag(z)})   // invalid: imag(z) is a (non-constant) function call
)
var z complex128
```

### Allocation

内置函数 new 的参数为 T 类型，在运行时为该类型的[变量](#variables)分配存储空间，并返回指向 *T 类型的值。变量按照[初始值](#the-zero-value)部分中的描述进行初始化。

```
new(T)
```

例如

```
type S struct { a int; b float64 }
new(S)
```

为 S 类型的变量分配存储空间，对其进行初始化 (a=0, b=0.0)，并返回包含位置地址的 *S 类型的值。

### Making slices, maps and channels

内置函数 make 的参数为 T 类型，它必须是切片、映射或通道类型，后面可以跟特定于类型的表达式列表。它返回一个类型为 T（不是 *T）的值。内存按照[初始值](#the-zero-value)部分中的描述进行初始化。

```
Call             Type T     Result

make(T, n)       slice      slice of type T with length n and capacity n
make(T, n, m)    slice      slice of type T with length n and capacity m

make(T)          map        map of type T
make(T, n)       map        map of type T with initial space for approximately n elements

make(T)          channel    unbuffered channel of type T
make(T, n)       channel    buffered channel of type T, buffer size n
```

每个数目参数 n 和 m 必须是整数类型或无类型[常量](#constants)。常量数目参数必须是非负的，并且可以用 int 类型的值[表示](#representability)；如果它是一个无类型常量，则被赋予 int 类型。如果 n
和 m 都存在并且是常量，则 n 不得大于 m。如果在运行时 n 为负数或大于 m，则会发生[运行时恐慌](#run-time-panics)。

```
s := make([]int, 10, 100)       // slice with len(s) == 10, cap(s) == 100
s := make([]int, 1e3)           // slice with len(s) == cap(s) == 1000
s := make([]int, 1<<63)         // illegal: len(s) is not representable by a value of type int
s := make([]int, 10, 0)         // illegal: len(s) > cap(s)
c := make(chan int, 10)         // channel with a buffer size of 10
m := make(map[string]int, 100)  // map with initial space for approximately 100 elements
```

使用映射类型和数目 n 调用 make 将创建一个具有初始空间的映射以容纳 n 个映射元素。具体的行为取决于实现。

### Appending to and copying slices

内置函数 append 和 copy 用来帮助常见的切片操作。对于这两个函数，结果与参数引用的内存是否重叠无关。

[可变参数](#function-types)函数 append 将零个或多个值 x 添加到 S 类型的 s 中，该类型必须是切片类型，并返回切片也是 S 类型。x 被传递给类型为 ...T 的参数，其中 T 是 S
的[元素类型](#slice-types)，适用相应的[参数传递规则](#passing-arguments-to--parameters)。作为一种特殊情况，append 还接受第一个是可赋值给 []byte 类型的参数和第二个是后跟
... 字符串类型的参数。这种形式为字符串添加字节。

```
append(s S, x ...T) S  // T is the element type of S
```

如果 s 的容量不足以容纳附加值，则 append 分配一个新的、足够大的底层数组，该数组既包括现有切片元素也包括新添加的。否则，append 会复用底层数组。

```
s0 := []int{0, 0}
s1 := append(s0, 2)                // append a single element     s1 == []int{0, 0, 2}
s2 := append(s1, 3, 5, 7)          // append multiple elements    s2 == []int{0, 0, 2, 3, 5, 7}
s3 := append(s2, s0...)            // append a slice              s3 == []int{0, 0, 2, 3, 5, 7, 0, 0}
s4 := append(s3[3:6], s3[2:]...)   // append overlapping slice    s4 == []int{3, 5, 7, 2, 3, 5, 7, 0, 0}

var t []interface{}
t = append(t, 42, 3.1415, "foo")   //                             t == []interface{}{42, 3.1415, "foo"}

var b []byte
b = append(b, "bar"...)            // append string contents      b == []byte{'b', 'a', 'r' }
```

函数 copy 将切片元素从 src 复制到 dst 并返回复制的元素数。两个参数必须具有[相同](#type-identity)的元素类型 T，并且必须可[赋值](#assignability)给 []T 类型的切片。复制的元素数是
len(src) 和 len(dst) 中的最小值。作为一种特殊情况，copy 还接受可赋值给 []byte 类型的 dst，src 参数为字符串类型。这种形式将字节从字符串复制到字节切片中。

```
copy(dst, src []T) int
copy(dst []byte, src string) int
```

示例：

```
var a = [...]int{0, 1, 2, 3, 4, 5, 6, 7}
var s = make([]int, 6)
var b = make([]byte, 5)
n1 := copy(s, a[0:])            // n1 == 6, s == []int{0, 1, 2, 3, 4, 5}
n2 := copy(s, s[2:])            // n2 == 4, s == []int{2, 3, 4, 5, 4, 5}
n3 := copy(b, "Hello, World!")  // n3 == 5, b == []byte("Hello")
```

### Deletion of map elements

内置函数 delete 从[映射](#map-types) m 中删除键为 k 的元素。k 的类型必须可[赋值](#assignability)给 m 的键类型。

```
delete(m, k) // remove element m[k] from map m
```

如果映射 m 为 nil 或元素 m[k] 不存在，则 delete 是空操作。

### Manipulating complex numbers

有三个函数用来组合或拆分复数。内置函数 complex 从浮点数实部和虚部构造复数值，而 real 和 imag 提取复数值的实部和虚部。

```
complex(realPart, imaginaryPart floatT) complexT
real(complexT) floatT
imag(complexT) floatT
```

参数的类型和返回值对应。对于 complex，两个参数必须是相同的浮点数类型，并且返回类型是具有相应浮点数的 complex 类型：float32 参数的 complex64 和 float64 参数的
complex128。如果其中一个参数的计算结果为无类型常量，首先将其隐式[转换](#conversions)为另一个参数的类型。如果两个参数都计算为无类型常量，则它们必须是非复数或其虚部必须为零，并且函数的返回值是无类型复数常量。

对于 real 和 imag，参数必须是复数类型，返回类型是对应的浮点类型：float32 参数的 complex64，float64 参数的
complex128。如果参数计算为无类型常量，则它必须是数字，并且函数的返回值是无类型浮点数常量。

real 和 imag 函数一起形成复数的逆，因此对于复数类型 Z 的值 z，z == Z(complex(real(z), imag(z)))。

如果这些函数的操作数都是常数，则返回值是常数。

```
var a = complex(2, -2)             // complex128
const b = complex(1.0, -1.4)       // untyped complex constant 1 - 1.4i
x := float32(math.Cos(math.Pi/2))  // float32
var c64 = complex(5, -x)           // complex64
var s int = complex(1, 0)          // untyped complex constant 1 + 0i can be converted to int
_ = complex(1, 2<<s)               // illegal: 2 assumes floating-point type, cannot shift
var rl = real(c64)                 // float32
var im = imag(a)                   // float64
const c = imag(b)                  // untyped constant -1.4
_ = imag(3 << s)                   // illegal: 3 assumes complex type, cannot shift
```

### Handling panics

两个内置函数 panic 和 recover 有助于报告和处理[运行时恐慌](#run-time-panics)和程序定义的错误情况。

```
func panic(interface{})
func recover() interface{}
```

在执行函数 F 时，显式调用 panic 或发生[运行时恐慌](#run-time-panics)会终止 F 的执行。任何被 F [延迟](#defer-statements)的函数都将照常执行。接下来，由 F
的调用者延迟的任何函数都会运行，依此类推，直到执行 goroutine 中的顶级函数所延迟的任何函数。此时，程序终止并报告错误条件，包括 panic 的参数值。这种终止序列称为*恐慌*。

```
panic(42)
panic("unreachable")
panic(Error("cannot parse"))
```

recover 函数允许程序管理发生恐慌的 goroutine 的行为。假设一个函数 G 推迟了一个 recover 函数 D，且在 G 正在执行的同一个 goroutine 上的一个函数中发生了一个恐慌。当延迟函数的运行到达 D 时，D
调用 recovery 并将返回值传递给 panic。如果 D 正常返回，没有发生新的恐慌，恐慌就会停止。在这种情况下，在 G 和调用 panic 之间调用的函数的状态被丢弃，并恢复正常执行。然后运行由 G 在 D 之前延迟的任何函数，并且
G 的执行通过返回到其调用者而终止。

如果以下任何一个条件成立，recover 的返回值为 nil：

- panic 的参数是 nil；
- goroutine 没有发生 panic；
- recover 不是由延迟函数直接调用的。

下面示例中的保护函数调用函数 g 并保护调用者免受 g 引发的运行时恐慌。

```
func protect(g func()) {
	defer func() {
		log.Println("done")  // Println executes normally even if there is a panic
		if x := recover(); x != nil {
			log.Printf("run time panic: %v", x)
		}
	}()
	log.Println("start")
	g()
}
```

### Bootstrapping

当前的实现提供了几个在引导过程中有用的内置函数。这些函数被记录为完整性，但不保证保留在语言中。他们不返回结果。

```
Function   Behavior

print      prints all arguments; formatting of arguments is implementation-specific
println    like print but prints spaces between arguments and a newline at the end
```

实现限制：print 和 println 不需要接受任意参数类型，但必须支持 boolean、numeric 和 string [类型](#types)的打印。

## Packages

Go 程序是通过将包链接在一起来构建的。包又由一个或多个源文件构造而成，这些源文件共同声明了属于该包的常量、类型、变量和函数，并且可以在同一包的所有文件中访问它们。这些元素可以[导出](#exported-identifiers)
并在另一个包中使用。

### Source file organization

每个源文件都包含一个 package 子句，定义它所属的包，后跟一组可能为空的导入声明，这些声明声明它希望使用其内容的包，后跟一组可能为空的函数、类型、变量声明、 和常数。

```
SourceFile       = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

### Package clause

package 子句开始每个源文件并定义文件所属的包。

```
PackageClause  = "package" PackageName .
PackageName    = identifier .
```

PackageName 不能是[空白标识符](#blank-identifier)。

```
package math
```

一组共享相同 PackageName 的文件构成了一个包的实现。一个实现可能要求一个包的所有源文件都位于同一目录中。

### Import declarations

导入声明指出包含该声明的源文件依赖于*被导入*包的功能（[§程序初始化和执行](#program-initialization-and-execution)），并允许访问该包的[可导出](#exported-identifiers)
标识符。导入命名用于访问的标识符 (PackageName) 和指定要导入的包的 ImportPath。

```
ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .
ImportSpec       = [ "." | PackageName ] ImportPath .
ImportPath       = string_lit .
```

PackageName 用于[限定标识符](#qualified-identifiers)以访问导入源文件中包的导出标识符。它在文件[块](#blocks)中声明。如果省略
PackageName，则默认为导入包的[包子句](#package-clause)中指定的标识符。如果出现显式句点 (.)
而不是名称，则在该包的包[块](#blocks)中声明的所有包的导出标识符可以在导入源文件的文件块中使用，并且必须在没有限定符的情况下访问。

ImportPath 的解释依赖于实现，但它通常是已编译包的完整文件名的子字符串，并且可能与已安装包的存储库相关。

实现限制：编译器可以将 ImportPaths 限制为仅使用属于 [Unicode](https://www.unicode.org/versions/Unicode6.3.0/) 的 L、M、N、P 和 S
通用类别（没有空格的图形字符）的字符的非空字符串，也可以排除字符 !"#$%&'()*,:;<=>?[\]^`{|} 和 Unicode 替换字符 U+FFFD。

假设我们编译了一个包含 package 子句 package math 的包，它导出函数 Sin，并将编译后的包安装在“lib/math”标识的文件中。此表说明了在各种类型的导入声明之后如何在导入包的文件中访问 Sin。

```
Import declaration          Local name of Sin

import   "lib/math"         math.Sin
import m "lib/math"         m.Sin
import . "lib/math"         Sin
```

导入声明声明了导入包和导入包之间的依赖关系。一个包直接或间接地导入自己，或者直接导入一个包而不引用它的任何导出标识符都是非法的。如果因为其 side-effects
（初始化）而导入包，请使用[空白标识符](#blank-identifier)作为显式包名称：

```
import _ "lib/math"
```

### An example package

这是一个完整的 Go 包，它实现了并发奇数筛选。

```
package main

import "fmt"

// Send the sequence 2, 3, 4, … to channel 'ch'.
func generate(ch chan<- int) {
	for i := 2; ; i++ {
		ch <- i // Send 'i' to channel 'ch'.
	}
}

// Copy the values from channel 'src' to channel 'dst',
// removing those divisible by 'prime'.
func filter(src <-chan int, dst chan<- int, prime int) {
	for i := range src { // Loop over values received from 'src'.
		if i%prime != 0 {
			dst <- i // Send 'i' to channel 'dst'.
		}
	}
}

// The prime sieve: Daisy-chain filter processes together.
func sieve() {
	ch := make(chan int) // Create a new channel.
	go generate(ch)      // Start generate() as a subprocess.
	for {
		prime := <-ch
		fmt.Print(prime, "\n")
		ch1 := make(chan int)
		go filter(ch, ch1, prime)
		ch = ch1
	}
}

func main() {
	sieve()
}
```

## Program initialization and execution

### The zero value

当通过声明或调用 new 为[变量](#variables)分配存储空间时，或者通过复合字面量或调用 make 创建新值时，并且未提供显式初始化，则该变量或值被赋予一个默认值。这个变量或值中的每个元素都为其类型的*零值*：布尔型为
false，数字类型为 0，字符串为 ""，指针、函数、接口、切片、通道、映射为 nil。此初始化是递归完成的，如果未指定任何值，则结构体数组的元素为对应的零值。

这两个简单的声明是等价的：

```
var i int
var i int = 0
```

之后

```
type T struct { i int; f float64; next *T }
t := new(T)
```

以下内容成立：

```
t.i == 0
t.f == 0.0
t.next == nil
```

之后也是如此

```
var t T
```

### Package initialization

在一个包中，包级变量初始化逐步进行，每一步选择声明顺序中最早的变量，该变量不依赖于未初始化的变量。

更准确地说，如果包级变量尚未初始化且没有初始化表达式或其初始化表达式不依赖于未[初始化的变量](#variable-declarations)，则认为它已准备好进行*初始化*
。初始化通过重复初始化下一个包级变量来进行，该变量在声明顺序中最早并准备初始化，直到没有准备好初始化的变量为止。

如果该过程结束时任何变量仍未初始化，则这些变量是一个或多个初始化周期的一部分，程序无效。

由右侧的单个（多值）表达式初始化的变量声明左侧的多个变量一起初始化：如果左侧的任何变量被初始化，则所有这些变量都被初始化在同一步骤中。

```
var x = a
var a, b = f() // a and b are initialized together, before x is initialized
```

出于包初始化的目的，[空白](#blank-identifier)变量与声明中的任何其他变量一样被处理。

在多个文件中声明的变量的声明顺序由文件呈现给编译器的顺序决定：在第一个文件中声明的变量在第二个文件中声明的任何变量之前声明，依此类推。

依赖分析不依赖于变量的实际值，只依赖于源中对它们的词法引用，并进行传递分析。例如，如果变量 x 的初始化表达式引用一个函数，其函数体引用变量 y，则 x 取决于 y。具体来说：

- 对变量或函数的引用是表示该变量或函数的标识符。
- 对方法 m 的引用是形式为 t.m 的[方法值](#method-values)或[方法表达式](#method-expressions)，其中 t 的（静态）类型不是接口类型，而方法 m 在 t
  的[方法集](#method-sets)中。是否调用结果函数值 t.m 无关紧要。
- 如果 x 的初始化表达式或主体（对于函数和方法）包含对 y 或对依赖于 y 的函数或方法的引用，则变量、函数或方法 x 依赖于变量 y。

例如，给定声明

```
var (
	a = c + b  // == 9
	b = f()    // == 4
	c = f()    // == 5
	d = 3      // == 5 after initialization has finished
)

func f() int {
	d++
	return d
}
```

初始化顺序为 d, b, c, a。请注意，初始化表达式中子表达式的顺序无关紧要：在此示例中，a = c + b 和 a = b + c 导致相同的初始化顺序。

每个包执行依赖分析；仅考虑对当前包中声明的变量、函数和（非接口）方法的引用。如果变量之间存在其他隐藏的数据依赖关系，则未指定这些变量之间的初始化顺序。

例如，给定声明

```
var x = I(T{}).ab()   // x has an undetected, hidden dependency on a and b
var _ = sideEffect()  // unrelated to x, a, or b
var a = b
var b = 42

type I interface      { ab() []int }
type T struct{}
func (T) ab() []int   { return []int{a, b} }
```

变量 a 将在 b 之后初始化，但是 x 是在 b 之前、b 和 a 之间还是在 a 之后初始化，因此也没有指定调用 sideEffect() 的时刻（在 x 初始化之前或之后）。

变量也可以使用在包块中声明的名为 init 的函数进行初始化，没有参数和结果参数。

```
func init() { … }
```

即使在单个源文件中，每个包也可以定义多个这样的函数。在 package 块中，init 标识符只能用于声明 init 函数，而标识符本身并没有[声明](#declarations-and-scope)。因此不能从程序的任何地方引用 init
函数。

一个没有导入的包是通过为它的所有包级变量分配初始值然后按照它们在源中出现的顺序调用所有 init
函数来初始化的，可能在多个文件中，如呈现给编译器的那样。如果包有导入，则在初始化包本身之前初始化导入的包。如果多个包导入一个包，则导入的包只会初始化一次。包的导入，通过构造，保证不会有循环初始化依赖。

包初始化——变量初始化和 init 函数的调用——发生在单个 goroutine 中，一次一个包。一个 init 函数可能会启动其他 goroutines，这些 goroutines 可以与初始化代码同时运行。然而，初始化总是对 init
函数进行排序：在前一个函数返回之前它不会调用下一个函数。

为了确保可重现的初始化行为，鼓励构建系统以词法文件名顺序向编译器呈现属于同一包的多个文件。

### Program execution

一个完整的程序是通过将一个称为 main 的单个未导入包与其导入的所有包链接起来创建的。main 包必须具有包名称 main 并声明一个不带参数且不返回值的函数 main。

```
func main() { … }
```

程序执行首先初始化主包，然后调用函数 main。当该函数调用返回时，程序退出。它不会等待其他 (non-main) goroutine 完成。

## Errors

预先声明的错误类型定义为

```
type error interface {
	Error() string
}
```

它是表示错误条件的常规接口，nil 值表示没有错误。例如，可以定义一个从文件中读取数据的函数：

```nilni l
func Read(f *File, b []byte) (n int, err error)
```

## Run-time panics

执行错误（例如尝试将数组索引越界）会触发运行时恐慌，相当于调用内置函数恐慌，其值为执行期定义的接口类型 runtime.Error。该类型满足预先声明的接口类型错误。表示不同运行时错误条件的确切错误值未指定。

```
package runtime

type Error interface {
	error
	// and perhaps other methods
}
```

## System considerations

### Package unsafe

可通过[导入路径](#import-declarations) "unsafe" 访问的内置包 unsafe ，为低级编程提供了便利，包括违反类型系统的操作。使用 unsafe 的包必须手动审查类型安全，并且可能不可移植。该包提供以下接口：

```
package unsafe

type ArbitraryType int  // shorthand for an arbitrary Go type; it is not a real type
type Pointer *ArbitraryType

func Alignof(variable ArbitraryType) uintptr
func Offsetof(selector ArbitraryType) uintptr
func Sizeof(variable ArbitraryType) uintptr

type IntegerType int  // shorthand for an integer type; it is not a real type
func Add(ptr Pointer, len IntegerType) Pointer
func Slice(ptr *ArbitraryType, len IntegerType) []ArbitraryType
```

Pointer 是一种[指针类型](#pointer-types)，但 Pointer 值可能不能[解引用](#address-operators)。任何[底层类型](#types) uintptr 的指针或值都可以转换为基础类型
Pointer 的类型，反之亦然。Pointer 和 uintptr 之间转换的效果是实现定义的。

```
var f float64
bits = *(*uint64)(unsafe.Pointer(&f))

type ptr unsafe.Pointer
bits = *(*uint64)(ptr(&f))

var p ptr = nil
```

函数 Alignof 和 Sizeof 接受任何类型的表达式 x 并分别返回假设变量 v 的对齐或大小，就好像 v 是通过 var v = x 声明的一样。

函数 Offsetof 接受一个（可能带括号的）[选择器](#selectors) s.f，表示由 s 或 *s 表示的结构的字段 f，并返回相对于结构地址的字节偏移量。如果 f 是一个[嵌入字段](#struct-types)
，它必须可以通过结构的字段在没有指针间接的情况下访问。对于具有字段 f 的结构体：

```
uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.f) == uintptr(unsafe.Pointer(&s.f))
```

计算机体系结构可能需要对齐内存地址；也就是说，对于一个变量的地址是一个因子的倍数，变量的类型的对齐方式。函数 Alignof 接受一个表示任何类型变量的表达式，并以字节为单位返回变量（的类型）的对齐方式。对于变量 x：

```
uintptr(unsafe.Pointer(&x)) % unsafe.Alignof(x) == 0
```

对 Alignof、Offsetof 和 Sizeof 的调用是 uintptr 类型的编译时常量表达式。

函数 Add 将 len 添加到 ptr 并返回更新后的指针 unsafe.Pointer(uintptr(ptr) + uintptr(len))。len 参数必须是整数类型或无类型[常量](#constants)。常量 len
参数必须可由 int 类型的值[表示](#representability)；如果它是一个无类型常量，则它被赋予类型 int。Pointer 的[使用规则](https://go.dev/pkg/unsafe#Pointer)仍然适用。

函数 Slice 返回一个切片，其底层数组从 ptr 开始，长度和容量为 len。Slice(ptr, len) 等价于

```
(*[len]ArbitraryType)(unsafe.Pointer(ptr))[:]
```

除了作为一种特殊情况，如果 ptr 为 nil 且 len 为零，Slice 返回 nil。

len 参数必须是整数类型或无类型[常量](#constants)。常量 len 参数必须是非负的并且可以用 int 类型的值[表示](#representability)；如果它是一个无类型常量，则它被赋予类型 int。在运行时，如果
len 为负，或者如果 ptr 为 nil 且 len 不为零，则会发生[运行时恐慌](#run-time-panics)。

### Size and alignment guarantees

对于[数字类型](#numeric-types)，保证以下大小：

```
type                                 size in bytes

byte, uint8, int8                     1
uint16, int16                         2
uint32, int32, float32                4
uint64, int64, float64, complex64     8
complex128                           16
```

保证以下最小对齐属性：

1. 对于任何类型的变量 x：unsafe.Alignof(x) 至少为 1。
2. 对于结构体类型的变量 x：unsafe.Alignof(x) 是 x 的每个字段 f 的所有值 unsafe.Alignof(x.f) 中的最大值，但至少为 1。
3. 对于数组类型的变量 x：unsafe.Alignof(x) 与数组元素类型的变量的对齐方式相同。

如果结构或数组类型不包含大小大于零的字段（或元素），则其大小为零。两个不同的零大小变量可能在内存中具有相同的地址。
