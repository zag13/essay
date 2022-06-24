# Go 中的 time.Duration 类型详解

> [Go's time.Duration Type Unravelled](https://www.ardanlabs.com/blog/2013/06/gos-duration-type-unravelled.html)

我一直在挣扎着使用 Go 标准库中的 Time 包。 我的挣扎来自两个功能点。 首先，尝试捕获两个不同时间段之间的毫秒数。 其次，将持续时间（以毫秒为单位）与预定义的时间跨度进行比较。 这听起来很简单，但就像我说的，我一直在挣扎。

在 Time 包中有一个名为 Duration 的自定义类型和一组辅助常量：

```
type Duration int64

const (
  Nanosecond Duration = 1
  Microsecond = 1000 * Nanosecond
  Millisecond = 1000 * Microsecond
  Second = 1000 * Millisecond
  Minute = 60 * Second
  Hour = 60 * Minute
)
```

我大概看了 1000 次，但这对我来说没有任何意义。 我只想比较两个时间段，取回持续时间，比较持续时间或者经过了一定的时间再执行某些操作。我终其一生都无法理解这种结构将如何帮助我解开这个谜团。

我写了这个测试函数，但它不起作用。

```
func Test() {
  var waitFiveHundredMillisections int64 = 500

  startingTime := time.Now().UTC()
  time.Sleep(10 * time.Millisecond)
  endingTime := time.Now().UTC()

  var duration time.Duration = endingTime.Sub(startingTime)
  var durationAsInt64 = int64(duration)

  if durationAsInt64 >= waitFiveHundredMillisections {
    fmt.Printf("Time Elapsed : Wait[%d] Duration[%d]\n",
       waitFiveHundredMillisections, durationAsInt64)
  } else {
    fmt.Printf("Time DID NOT Elapsed : Wait[%d] Duration[%d]\n",
       waitFiveHundredMillisections, durationAsInt64)
  }
}
```

当我运行 Test 函数并查看输出时，代码认为已经过去了 500 毫秒。

```
Time Elapsed : Wait[500] Duration[10724798]
```

那么出了什么问题呢？ 我再次查看了 Duration 类型和相关常量。

```
type Duration int64

const (
  Nanosecond Duration = 1
  Microsecond = 1000 * Nanosecond
  Millisecond = 1000 * Microsecond
  Second = 1000 * Millisecond
  Minute = 60 * Second
  Hour = 60 * Minute
)
```

Duration 类型的基本时间单位是纳秒。 啊，这就是为什么将包含 10 毫秒的 Duration 类型转换为 int64 时我得到 10,000,000。

所以直接投射是行不通的。 我需要一个不同的策略，并更好地理解如何使用和转换 Duration 类型。

我知道最好直接使用 Duration 类型，这将最大限度地减少使用该类型时的问题。基于常量值，我可以通过以下方式创建 Duration 类型变量：

```
func Test() {
  var duration_Milliseconds time.Duration = 500 * time.Millisecond
  var duration_Seconds time.Duration = (1250 * 10) * time.Millisecond
  var duration_Minute time.Duration = 2 * time.Minute

  fmt.Printf("Milli [%v]\nSeconds [%v]\nMinute [%v]\n",
    duration_Milliseconds,
    duration_Seconds,
    duration_Minute)
}
```

我创建了 3 个 Duration 类型的变量。然后通过使用时间常数，我能够创建正确的持续时间跨度值。当我使用标准库 Printf 函数和 %v 运算符时，我得到以下输出：

```
Milli [500ms]
Seconds [12.5s]
Minute [2m0s]
```

这很酷。 Printf 函数知道如何自然的显示 Duration 类型。 根据每个 Duration 变量中的值，Printf 函数会选择恰当的时间单位来打印该值。 我也得到了我所期望的。

Duration 类型确实具有将 Duration 变量的值转换为原生 Go 类型的成员函数，返回值为 int64 或 float64：

```
func Test() {
  var duration_Seconds time.Duration = (1250 * 10) * time.Millisecond
  var duration_Minute time.Duration = 2 * time.Minute

  var float64_Seconds float64 = duration_Seconds.Seconds()
  var float64_Minutes float64 = duration_Minute.Minutes()

  fmt.Printf("Seconds [%.3f]\nMinutes [%.2f]\n",
    float64_Seconds,
    float64_Minutes)
}
```

我很快发现到没有 Milliseconds 函数。 除了毫秒之外，每个时间单位就有一个函数。 当我使用秒和分钟时，我按预期得到以下输出：

```
Seconds [12.500]
Minutes [2.00]
```

但是我需要毫秒，那么为什么缺少毫秒函数呢？

Go 的设计者不想在毫秒内将我锁定在一个单一的原生类型中。 他们希望我有选择。（译者注：不清楚作者的go版本，但现在的 time 包是有 Milliseconds 方法的 go version: 1.17）

以下代码将 Duration 变量的值作为 int64 和 float64 转换为毫秒：

```
func Test() {
  var duration_Milliseconds time.Duration = 500 * time.Millisecond

  var castToInt64 int64 = duration_Milliseconds.Nanoseconds() / 1e6
  var castToFloat64 float64 = duration_Milliseconds.Seconds() * 1e3

  fmt.Printf("Duration [%v]\ncastToInt64 [%d]\ncastToFloat64 [%.0f]\n",
    duration_Milliseconds,
    castToInt64,
    castToFloat64)
}
```

如果我将纳秒除以 1e6，我会得到一个 int64 的毫秒数。 如果我将秒数乘以 1e3，我会得到一个 float64 的毫秒数。

这是输出：

```
Duration [500ms]
castToInt64 [500]
castToFloat64 [500]
```

如果您想知道 1e6 或 1e3 代表什么，我也一样：

```
1e3 = 103 = One Thousand = 1 plus 3 0’s

1e6 = 106 = One Million  = 1 plus 6 0’s
```

现在我了解了 Duration 类型是什么以及如何最好地使用和操作它，我有我的最后一个使用毫秒的测试示例：

```
func Test() {
  var waitFiveHundredMillisections time.Duration = 500 * time.Millisecond

  startingTime := time.Now().UTC()
  time.Sleep(600 * time.Millisecond)
  endingTime := time.Now().UTC()

  var duration time.Duration = endingTime.Sub(startingTime)

  if duration >= waitFiveHundredMillisections {
    fmt.Printf("Wait %v\nNative [%v]\nMilliseconds [%d]\nSeconds [%.3f]\n",
       waitFiveHundredMillisections,
       duration,
       duration.Nanoseconds()/1e6,
       duration.Seconds())
  }
}
```

输出结果为下：

```
Wait 500ms
Native [601.001ms]
Milliseconds [601]
Seconds [0.601]
```

我没有比较原生类型以确定持续的时间，而是比较两种 Duration 类型。 这样干净多了。 显示值时，我使用 Duration 类型自定义格式并将 Duration 变量的值转换为毫秒作为 int64 和 float64。

花了一段时间，但最终使用 Duration 类型开始变得有意义。 一如既往，我希望这有助于其他人在他们的 Go 应用程序中使用 Duration 类型。
