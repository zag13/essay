# Singleton Pattern

Singleton 设计模式将类型的实例化限制为但个对象。

## Implementation

```
package singleton

import "sync"

type singleton map[string]string

var (
    once sync.Once
    
    instance singleton
)

func New() singleton {
    once.Do(func() {
        instance = make(singleton)
    })
    
    return instance
}
```

## Usage

```
s := singleton.New()

s["this"] = "that"

s2 := singleton.New()

fmt.Println("This is ", s2["this"])
// This is that
```

## Rules of Thumb

- 单例模式代表全局状态，大多数时候会降低可测试性。