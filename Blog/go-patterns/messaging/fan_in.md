# Fan-In Messaging Patterns

Fan-In 是一种消息传递模式，用于在工作人员之间创建工作漏斗 (clients: source, server: destination)。

## Implementation

```
// Merge different channels in one channel
func Merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    
    out := make(chan int)
    
    // Start an send goroutine for each input channel in cs. send
    // copies values from c to out until c is closed, then calls wg.Done.
    send := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    
    wg.Add(len(cs))
    for _, c := range cs {
        go send(c)
    }
    
    // Start a goroutine to close out once all the send goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

`Merge` 函数通过为每个通道启动一个 goroutine 将通道列表转换为单个通道将值复制到唯一的出站通道中。

一旦所有的发送 goroutine 都被启动，`Merge` 会启动一个 goroutine 以关闭主通道。

