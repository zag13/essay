# Object Pool Pattern

The object pool 设计模式被用于根据需求准备和维护多个实例。

## Implementation

```
package pool

type Pool chan *Object

func New(total int) *Pool {
    p := make(Pool, total)
    
    for i := 0; i < total; i++ {
        p <- new(Object)
    }
    
    return &p
}
```

## Usage

下面给出了一个对象池的简单生命周期示例。

```
p := pool.New(2)

select {
case obj := <-p:
    obj.Do( /*...*/ )
    
    p <- obj
default:
    // No more objects left — retry later or fail
    return
}
```

## Rules of Thumb

- 对象池模式在初始化对象比维护对象更昂贵的情况下很有用。
- 如果需求不是稳定（有峰值），则维护的开销可能会超过对象池的好处。
- 由于事先初始化了对象，对提升性能是有帮助的。

