# Proxy Pattern

[proxy](https://en.wikipedia.org/wiki/Proxy_pattern) 设计模式提供一个对象，控制对另一个对象的访问，拦截所有调用。

## Implementation

代理可以连接到任何东西：网络连接、内存中的大对象、文件、其他昂贵的或不可能复制的资源。

```
// To use proxy and to object they must implement same methods
type IObject interface {
    ObjDo(action string)
}

// Object represents real objects which proxy will delegate data
type Object struct {
    action string
}

// ObjDo implements IObject interface and handel's all logic
func (obj *Object) ObjDo(action string) {
    // Action behavior
    fmt.Printf("I can, %s", action)
}

// ProxyObject represents proxy object with intercepts actions
type ProxyObject struct {
    object *Object
}

// ObjDo are implemented IObject and intercept action before send in real Object
func (p *ProxyObject) ObjDo(action string) {
    if p.object == nil {
        p.object = new(Object)
    }
    if action == "Run" {
        p.object.ObjDo(action) // Prints: I can, Run
    }
}
```

## Usage

更复杂的代理使用示例：用户创建 "Terminal" 授权和 PROXY 发送执行命令到真实终端对象。参见 [proxy/main.go](./proxy/main.go)
或 [在 Playground 中查看](https://play.golang.org/p/mnjKCMaOVE)。

