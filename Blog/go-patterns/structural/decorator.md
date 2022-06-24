# Decorator Pattern

Decorator 结构设计模式允许动态扩展现有对象的功能而不改变其内部结构。

Decorator 提供了一种灵活的方法来扩展对象的功能。

## Implementation

`LogDecorate` 用签名 `func(int) int` 来装饰一个函数，该函数处理整数并添加 输入/输出 日志记录功能。

```
type Object func(int) int

func LogDecorate(fn Object) Object {
    return func(n int) int {
        log.Println("Starting the execution with the integer", n)
    
        result := fn(n)
    
        log.Println("Execution is completed with the result", result)
    
        return result
    }
}
```

## Usage

```
func Double(n int) int {
    return n * 2
}

f := LogDecorate(Double)

f(5)
// Starting execution with the integer 5
// Execution is completed with the result 10
```

## Rules of Thumb

- 与 Adapter 模式不同的是，要装饰的对象是通过**注入**获得的。
- 装饰器不应改变对象的接口。
