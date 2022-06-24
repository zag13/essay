# Strategy Pattern

Strategy 行为设计模式允许在运行时选择要使用的算法。

定义、封装、并可以相互替换着使用这些算法。

## Implementation

```
type Operator interface {
    Apply(int, int) int
}

type Operation struct {
    Operator Operator
}

func (o *Operation) Operate(leftValue, rightValue int) int {
    return o.Operator.Apply(leftValue, rightValue)
}
```

## Usage

### Addition Operator

```
type Addition struct{}

func (Addition) Apply(lval, rval int) int {
    return lval + rval
}
```

```
add := Operation{Addition{}}
add.Operate(3, 5) // 8
```

### Multiplication Operator

```
type Multiplication struct{}

func (Multiplication) Apply(lval, rval int) int {
    return lval * rval
}
```

```
mult := Operation{Multiplication{}}

mult.Operate(3, 5) // 15
```

## Rules of Thumb

- Strategy 模式与 Template 模式相似，只是粒度不同。
- Strategy 模式让你改变一个对象的内部结构。Decorator 模式让你改变一个对象的皮肤。
