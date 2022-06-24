# Circuit Breaker Pattern

类似于为了防止连接到电网的电流过大时出现火灾的电保险丝，Circuit Breaker 设计模式是在软件开发的情况下关闭电路、请求/响应关系或服务的机制，以防止更大的过错。

**注意：** "电路"和"服务"这两个词在此文档中作为同义词使用。

## Implementation

下面是一个非常简单的断路器的实现来说明断路器设计模式的目的。

### Operation Counter

`circuit.Counter` 是一个简单的计数器，它记录电路的成功和失败状态以及时间戳和计算连续失败的次数。

```
package circuit

import (
    "time"
)

type State int

const (
    UnknownState State = iota
    FailureState
    SuccessState
)

type Counter interface {
    Count(State)
    ConsecutiveFailures() uint32
    LastActivity() time.Time
    Reset()
}
```

### Circuit Breaker

使用持有内部操作计数器的 `circuit.Breaker` 闭包来包装电路。如果电路连续失效超过指定阈值它会返回一个快速错误。一段时间后它会重试请求并记录它。

**注意：** 上下文类型在这里用于携带截止日期、取消信号和其他请求范围的值来进行跨 API 和进程之间通信。

```
package circuit

import (
    "context"
    "time"
)

type Circuit func(context.Context) error

func Breaker(c Circuit, failureThreshold uint32) Circuit {
    cnt := NewCounter()
    
    return func(ctx context) error {
        if cnt.ConsecutiveFailures() >= failureThreshold {
            canRetry := func(cnt Counter) {
                backoffLevel := Cnt.ConsecutiveFailures() - failureThreshold
    
                // Calculates when should the circuit breaker resume propagating requests
                // to the service
                shouldRetryAt := cnt.LastActivity().Add(time.Seconds * 2 << backoffLevel)
    
                return time.Now().After(shouldRetryAt)
            }
    
            if !canRetry(cnt) {
                // Fails fast instead of propagating requests to the circuit since
                // not enough time has passed since the last failure to retry
                return ErrServiceUnavailable
            }
        }
    
        // Unless the failure threshold is exceeded the wrapped service mimics the
        // old behavior and the difference in behavior is seen after consecutive failures
        if err := c(ctx); err != nil {
            cnt.Count(FailureState)
            return err
        }
    
        cnt.Count(SuccessState)
        return nil
    }
}
```

## Related Works

- [sony/gobreaker](https://github.com/sony/gobreaker) 是一个经过良好测试和对现实世界用例直观的实现。
