# Timing Functions

优化代码时，有时需要快速而随意的时间测量，而不是使用分析工具/框架来验证假设。

时间测量可以通过使用 `time` 包和 `defer` 语句来执行。

## Implementation

```
package profile

import (
    "time"
    "log"
)

func Duration(invocation time.Time, name string) {
    elapsed := time.Since(invocation)

    log.Printf("%s lasted %s", name, elapsed)
}
```

## Usage

```
func BigIntFactorial(x big.Int) *big.Int {
    // Arguments to a defer statement is immediately evaluated and stored.
    // The deferred function receives the pre-evaluated values when its invoked.
    defer profile.Duration(time.Now(), "IntFactorial")

    y := big.NewInt(1)
    for one := big.NewInt(1); x.Sign() > 0; x.Sub(x, one) {
        y.Mul(y, x)
    }

    return x.Set(y)
}
```
