# Go 中的面向对象编程

> [Object Oriented Programming in Go](https://www.ardanlabs.com/blog/2013/07/object-oriented-programming-in-go.html)


今天论坛上有人问了一个问题，关于如何在不嵌入的情况下获得继承的好处。对于每个人来说，以 Go 而不是他们离开的语言来考虑问题是非常重要的。我无法告诉你我从早期的 Go
实现中删除了多少代码，因为这不是必需的。语言设计师拥有多年的经验和知识。后见之明正在帮助创建一种快速、精简且编写代码非常有趣的语言。

我认为 Go 是一种轻量级的面向对象编程语言。是的，它确实具有封装和类型成员函数，但它缺乏继承，因此缺乏传统的多态。对我来说，除非你想实现多态，否则继承是没有用的。使用 Go 中实现接口的方式，不需要继承。Go 采用了 OOP
中最好的部分，省略了其余部分，为我们提供了一种更好的方式来编写多态代码。

这是 Go 中 OOP 的快速视图。从这三个结构体开始：

```
type Animal struct {
    Name string
    mean bool
}

type Cat struct {
    Basics Animal
    MeowStrength int
}

type Dog struct {
    Animal
    BarkStrength int
}
```

以下是您可能会在任何 OOP 示例中看到的三个结构体。我们有一个基础结构体和另外两个以此为基础的结构体。Animal 结构体包含所有动物共享的属性，另外两个结构体特定于猫和狗。

除 mean 字段外，所有成员属性（字段）都是公开的。mean 字段在 Animal 结构体中并以小写字母开头。在 Go 中，变量、结构体、字段、函数等的第一个字母的大小写决定了访问规范。使用大写字母是公开的，使用小写字母是私有的。

*注意：Go 中公共和私有访问的观念并不完全正确：https://www.ardanlabs.com/blog/2014/03/exportedunexported-identifiers-in-go.html*

由于 Go 中没有继承，因此组合是您唯一的选择。Cat 结构体中有一个名为 Basics 的字段，其类型为 Animal。Dog 结构体使用了一个未命名的结构体（嵌入），类型为 Animal。由您决定哪个更适合您，我将向您展示这两种实现。

我要感谢 John McLaughlin 对未命名结构体的评论！

为 Cat 和 Dog 创建一个成员函数（方法），语法如下：

```
func (dog *Dog) MakeNoise() {
    barkStrength := dog.BarkStrength

    if dog.mean == true {
        barkStrength = barkStrength * 5
    }

    for bark := 0; bark < barkStrength; bark++ {
        fmt.Printf("BARK ")
    }

    fmt.Println("")
}

func (cat *Cat) MakeNoise() {
    meowStrength := cat.MeowStrength

    if cat.Basics.mean == true {
        meowStrength = meowStrength * 5
    }

    for meow := 0; meow < meowStrength; meow++ {
        fmt.Printf("MEOW ")
    }

    fmt.Println("")
}
```

在方法名之前，我们指定一个接收器，它是指向每种类型的指针。现在 Cat 和 Dog 都有名为 MakeNoise 的方法。

这两种方法都做同样的事情。每只动物都会根据它们的吠叫或喵喵声的强度来使用它们的母语说话。很傻，但它向您展示了如何访问引用的对象（值）。

使用 Dog 引用时，我们直接访问 Animal 字段。对于 Cat 引用，我们使用名为 Basics 的命名字段。

到目前为止，我们已经介绍了封装、组合、访问规范和成员函数。剩下的就是如何创建多态行为。

我们使用接口来创建多态行为：

```
type AnimalSounder interface {
    MakeNoise()
}

func MakeSomeNoise(animalSounder AnimalSounder) {
    animalSounder.MakeNoise()
}
```

这里我们添加一个接口和一个参数为接口类型的公共函数。实际上，该函数将引用实现此接口的类型的值。接口不是可以被实例化的类型。接口是由其他类型实现的行为声明。

在 Go 中，有一个关于接口的约定：当接口只包含一个方法时，使用 "er" 作为名称后缀。

在 Go 中，任何通过方法实现接口的类型都可以表示该接口类型。在我们的例子中，Cat 和 Dog 都使用指针接收器实现了 AnimalSounder 接口，因此被认为是 AnimalSounder 类型。

这意味着 Cat 和 Dog 的指针都可以作为参数传递给 MakeSomeNoise 函数。MakeSomeNoise 函数通过 AnimalSounder 接口实现多态行为。

如果你想减少 Cat 和 Dog 的 MakeNoise 方法中的代码重复，你可以为 Animal 类型创建一个方法来处理它：

```
func (animal *Animal) PerformNoise(strength int, sound string) {
    if animal.mean == true {
        strength = strength * 5
    }

    for voice := 0; voice < strength; voice++ {
        fmt.Printf("%s ", sound)
    }

    fmt.Println("")
}

func (dog *Dog) MakeNoise() {
    dog.PerformNoise(dog.BarkStrength, "BARK")
}

func (cat *Cat) MakeNoise() {
    cat.Basics.PerformNoise(cat.MeowStrength, "MEOW")
}
```

现在，Animal 类型有一个具有业务逻辑的方法来制造噪音。业务逻辑保持在其所属类型的范围内。另一个很酷的好处是我们不需要将 mean 字段作为参数传递，因为它已经属于 Animal 类型。

这是完整的工作示例程序：

```
package main

import (
    "fmt"
)

type Animal struct {
    Name string
    mean bool
}

type AnimalSounder interface {
    MakeNoise()
}

type Dog struct {
    Animal
    BarkStrength int
}

type Cat struct {
    Basics Animal
    MeowStrength int
}

func main() {
    myDog := &Dog{
        Animal{
           "Rover", // Name
           false,   // mean
        },
        2, // BarkStrength
    }

    myCat := &Cat{
        Basics: Animal{
            Name: "Julius",
            mean: true,
        },
        MeowStrength: 3,
    }

    MakeSomeNoise(myDog)
    MakeSomeNoise(myCat)
}

func (animal *Animal) PerformNoise(strength int, sound string) {
    if animal.mean == true {
        strength = strength * 5
    }

    for voice := 0; voice < strength; voice++ {
        fmt.Printf("%s ", sound)
    }

    fmt.Println("")
}

func (dog *Dog) MakeNoise() {
    dog.PerformNoise(dog.BarkStrength, "BARK")
}

func (cat *Cat) MakeNoise() {
    cat.Basics.PerformNoise(cat.MeowStrength, "MEOW")
}

func MakeSomeNoise(animalSounder AnimalSounder) {
    animalSounder.MakeNoise()
}
```

这是输出：

```
BARK BARK
MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW
```

有人在论坛中发布了一个关于在结构体中嵌入接口的示例。这是一个例子：

```
package main

import (
    "fmt"
)

type HornSounder interface {
    SoundHorn()
}

type Vehicle struct {
    List [2]HornSounder
}

type Car struct {
    Sound string
}

type Bike struct {
   Sound string
}

func main() {
    vehicle := new(Vehicle)
    vehicle.List[0] = &Car{"BEEP"}
    vehicle.List[1] = &Bike{"RING"}

    for _, hornSounder := range vehicle.List {
        hornSounder.SoundHorn()
    }
}

func (car *Car) SoundHorn() {
    fmt.Println(car.Sound)
}

func (bike *Bike) SoundHorn() {
    fmt.Println(bike.Sound)
}

func PressHorn(hornSounder HornSounder) {
    hornSounder.SoundHorn()
}
```

在此示例中，Vehicle 结构体维护了一个实现 HornSounder 接口的值列表。在 main 中，我们创建一个新车辆并将一个 Car 和 Bike 指针分配给列表。这种分配是可能的，因为 Car 和 Bike
都实现了该接口。然后使用一个简单的循环，我们使用界面来发出喇叭声。

在应用程序中实现 OOP 所需的一切都在 Go 中。正如我之前所说，Go 采用了 OOP 的最佳部分，省略了其余部分，并为我们提供了编写多态代码的更好方法。

要了解有关相关主题的更多信息，请查看以下帖子：

https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html
https://www.ardanlabs.com/blog/2013/07/how-packages-work-in -go-language.html
https://www.ardanlabs.com/blog/2013/07/singleton-design-pattern-in-go.html

我希望这个小例子对你未来的 Go 编程有所帮助。
