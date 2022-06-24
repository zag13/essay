# Observer Pattern

[Observer](https://en.wikipedia.org/wiki/Observer_pattern) 设计模式允许类型实例将事件 "publish" 给其他类型实例 ("observers")
（希望在特定事件发生时更新的实例）。

## Implementation

在长期运行的应用程序中——例如网络服务器——实例可以保存一组观察者，这些观察者被用于接收触发事件的通知。

实现方式各不相同，但可以使用接口来制作标准的 observers 和 notifiers：

```
type (
    // Event defines an indication of a point-in-time occurrence.
    Event struct {
        // Data in this case is a simple int, but the actual
        // implementation would depend on the application.
        Data int64
    }
    
    // Observer defines a standard interface for instances that wish to list for
    // the occurrence of a specific event.
    Observer interface {
        // OnNotify allows an event to be "published" to interface implementations.
        // In the "real world", error handling would likely be implemented.
        OnNotify(Event)
    }
    
    // Notifier is the instance being observed. Publisher is perhaps another decent
    // name, but naming things is hard.
    Notifier interface {
        // Register allows an instance to register itself to listen/observe
        // events.
        Register(Observer)
        // Deregister allows an instance to remove itself from the collection
        // of observers/listeners.
        Deregister(Observer)
        // Notify publishes new events to listeners. The method is not
        // absolutely necessary, as each implementation could define this itself
        // without losing functionality.
        Notify(Event)
    }
)
```

## Usage

具体用法，查阅 [observer/main.go](./observer/main.go) 或 [view in the Playground](https://play.golang.org/p/cr8jEmDmw0)。
