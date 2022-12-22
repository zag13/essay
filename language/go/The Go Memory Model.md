# [The Go Memory Model](https://go.dev/ref/mem)

* [Introduction](#introduction)
    * [Advice](#advice)
    * [Informal Overview](#informal-overview)
* [Memory Model](#memory-model)
* [Implementation Restrictions for Programs Containing Data Races](#implementation-restrictions-for-programs-containing-data-races)
* [Synchronization](#synchronization)
    * [Initialization](#initialization)
    * [Goroutine creation](#goroutine-creation)
    * [Goroutine destruction](#goroutine-destruction)
    * [Channel communication](#channel-communication)
    * [Locks](#locks)
    * [Once](#once)
    * [Atomic Values](#atomic-values)
    * [Finalizers](#finalizers)
    * [Additional Mechanisms](#additional-mechanisms)
* [Incorrect synchronization](#incorrect-synchronization)
* [Incorrect compilation](#incorrect-compilation)
* [Conclusion](#conclusion)

## Introduction

Go内存模型描述的是"在一个groutine中对变量进行读操作能够侦测到在其他goroutine中对该变量的写操作"的条件。

### Advice

程序修改被多个goroutines同时访问的数据必须要序列化这种访问操作。

要序列化访问操作，请使用通道操作或其他同步原语（例如sync和sync/atomic包中的原语）保护数据。

如果您必须阅读本文档的其余部分才能理解程序的行为，那就太聪明了。

不要自作聪明。

### Informal Overview

Go使用与其他语言几乎相同的方式处理其内存模型，旨在保持语义简单、可理解和有用。
本节概述了该方法，应该足以满足大多数程序员的需要。
下一节将更正式地介绍内存模型。

数据竞争是对内存位置的写入与对同一位置的另一次读取或写入同时发生，除非涉及的所有访问都是sync/atomic包提供的原子数据访问。
如前所述，强烈建议程序员使用适当的同步来避免数据竞争。
在没有数据竞争的情况下，Go程序的行为就好像所有的goroutines都被多路复用到一个处理器上。
此属性有时称为DRF-SC：无数据竞争的程序以顺序一致的方式执行。

虽然程序员应该在没有数据竞争的情况下编写Go程序，但Go在响应数据竞争时可以做什么是有限制的。
一个实现是通过报告竞争和终止程序来对数据竞争作出响应。
否则，每次读取single-word-sized或 sub-word-sized的内存位置都必须观察到实际写入该位置的值（可能由并发执行的goroutine）且尚未被覆盖。
这些实现约束使Go更像Java或JavaScript，因为大多数竞赛的结果数量有限，而不像C和C++，在C和C++中，任何具有竞赛的程序的含义都是完全未定义的，编译器可能会做任何事情。
Go的方法旨在让出错的程序更可靠、更容易调试，同时仍然坚持认为竞争是错误的，工具可以诊断和报告它们。

## Memory Model

以下Go内存模型的正式定义严格遵循Hans-J. Boehm和Sarita V.
Adve在2008发布于PLDI的"[Foundations of the C++ Concurrency Memory Model](https://www.hpl.hp.com/techreports/2008/HPL-2008-56.pdf)"。
无数据竞争程序的定义和无竞争程序的顺序一致性保证与该工作中的相同。

内存模型描述了对程序执行的要求，这些由goroutine执行构成，而goroutine执行又由内存操作构成。

内存操作由四个细节构成：

- 它的种类，表示它是普通数据读取、普通数据写入还是同步操作，例如原子数据访问、互斥操作或通道操作，
- 它在程序中的位置，
- 正在访问的内存位置或变量，以及
- 操作读取或写入的值。

一些内存操作类似于读取，包括读取、原子读取、互斥锁和通道接收。
其他内存操作类似写，包括写、原子写、互斥锁解锁、通道发送和通道关闭。
有些，例如原子比较和交换，既像读又像写。

Goroutine执行由单个goroutine执行的一组内存操作所构成。

**要求1：**
给定从内存读取和写入内存的值，每个goroutine中的内存操作必须对应于该goroutine的正确顺序执行。
该执行必须与之前的顺序一致，[Go语言规范](https://go.dev/ref/spec)
的控制流构造以及[表达式的评估顺序](https://go.dev/ref/spec#Order_of_evaluation)。

Go程序执行是基于一组goroutine执行，以及一个映射W，它指定每个类读操作从中读取的类写操作。
（同一个程序的多次执行可以有不同的程序执行。）

**要求2：**
对于给定的程序执行，映射W，当仅限于同步操作时，必须可以通过与排序一致的同步操作的某种隐式总顺序以及这些操作读取和写入的值来解释。

Synchronized before关系是同步内存操作的偏序，从W派生。
如果同步读内存操作r观察到同步写内存操作w（即，如果 W(r) = w），则w在r之前同步。
通俗地讲，synchronized before关系是上一段中提到的隐含全序的子集，仅限于W直接观察到的信息。

Happens before关系被定义为sequenced before和synchronized before关系联合的传递闭包。

**要求3：**
对于内存位置x上的普通（非同步）数据读取r，W(r) 必须是对r可见的写入w，其中可见意味着以下两项均成立：

1. w发生在r前。
2. w不会在r之前发生的任何其他写入w'(to x)之前发生。

内存位置x上的读-写数据竞争由x上的类似读内存操作r和x上的类似写内存操作w组成，其中至少有一个是非同步的，它们的发生是无序的
(即，r既不会发生在w之前，也不会w发生在r之前）。

内存位置x上的写-写数据竞争由x上的两个类似写的内存操作w和w'组成，其中至少一个是非同步的，它们的发生是无序的。

请注意，如果内存位置x上没有读-写或写-写数据竞争，则x上的任何读取r都只有一个可能的W(r)：在先发生顺序中紧接在它之前的单个w。

更一般地说，可以证明任何无数据竞争的Go程序，这意味着它没有程序执行与读-写或写-写数据竞争，只能通过goroutine执行的一些顺序一致的交错来解释结果。
（证明与上面引用的Boehm和Adve论文的第7节相同。）
此属性称为DRF-SC。

正式定义的目的是匹配其他语言（包括 C、C++、Java、JavaScript、Rust和Swift）为无竞争程序提供的DRF-SC保证。

某些Go语言操作，如goroutine创建和充当同步操作的内存分配。
这些操作对synchronized-before偏序的影响记录在下面的"Synchronization"部分中。
各个软件包负责为自己的操作提供类似的文档。

## Implementation Restrictions for Programs Containing Data Races

上一节给出了无数据竞争程序执行的正式定义。
本节非正式地描述了实现必须为包含竞争的程序提供的语义。

首先，任何实现都可以在检测到数据竞争时报告竞争并停止程序的执行。
使用ThreadSanitizer的实现（通过"go build -race"访问）正是这样做的。

否则，对不大于机器字的内存位置x的读取r必须观察到一些写入w，使得r不会发生在w之前，并且没有写入w'使得w发生在w'之前并且w'发生在之前。
也就是说，每次读取都必须观察到先前或并发写入所写入的值。

此外，不允许观察非因果和"凭空"写入。

鼓励读取大于单个机器字的内存位置，但不要求满足与字大小内存位置相同的语义，观察单个允许的写入w。
出于性能原因，实现可能会将较大的操作视为一组未指定顺序的单个机器字大小的操作。
这意味着多字数据结构上的竞争会导致与单次写入不对应的值不一致。
当值取决于内部 (pointer, length) 或 (pointer, type) 对的一致性时，就像大多数 Go 实现中的接口值、映射、切片和字符串的情况一样，这种竞争反过来会导致任意内存损坏。

下面的"Incorrect synchronization"部分给出了不正确同步的示例。

下面的"Incorrect compilation"部分给出了编译失败的示例。

## Synchronization

### Initialization

程序初始化在单个goroutine中运行，但该goroutine可能会创建并发运行的其他goroutine。

如果包p导入包q，则q的初始化函数的完成发生在p开始之前。

所有init函数的完成在函数main.main开始之前是同步的。

### Goroutine creation

Go语句启动的新goroutine的开始执行之前是同步的。

例如，在这个程序中：

```
var a string

func f() {
	print(a)
}

func hello() {
	a = "hello, world"
	go f()
}
```

调用hello将在将来的某个时间打印"hello, world"（可能在hello返回之后）。

### Goroutine destruction

不保证goroutine的退出在程序中的任何事件之前是同步的。
例如，在这个程序中：

```
var a string

func hello() {
	go func() { a = "hello" }()
	print(a)
}
```

对a的赋值之后没有任何同步事件，因此不能保证任何其他goroutine都观察到它。
事实上，激进的编译器可能会删除整个go语句。

如果一个goroutine的效果需要被另一个goroutine观察到，则使用同步机制（例如锁或通道通信）来建立相对顺序。

### Channel communication

通道通信是goroutine之间同步的主要方式。
通道上的每个发送都与来自该通道的相应接收相匹配，通常在不同的goroutine中。

在从该通道完成相应接收之前，通道上的发送是同步的。

在这个程序中：

```
var c = make(chan int, 10)
var a string

func f() {
	a = "hello, world"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}
```

可以保证打印"hello, world"。
对a的写入在c上的发送之前排序，在c上的相应接收完成之前同步，在打印之前排序。

*通道的关闭在接收返回零值之前同步，因为通道已关闭。*

在前面的示例中，将c<-0替换为close(c)会产生具有相同保证行为的程序。

*来自无缓冲通道的接收在该通道上的相应发送完成之前同步。*

该程序（如上，但交换了发送和接收语句并使用无缓冲通道）：

```
var c = make(chan int)
var a string

func f() {
	a = "hello, world"
	<-c
}

func main() {
	go f()
	c <- 0
	print(a)
}
```

也保证打印"hello, world"。
对a的写入在c上的接收之前排序，在c上的相应发送完成之前同步，在打印之前排序。

如果是缓冲通道（例如，c=make(chan int, 1)），那么程序将不能保证打印"hello, world"。
（它可能会打印空字符串、崩溃或执行其他操作。）

*容量为C的通道上的第k个接收在从该通道完成的第k+C个发送完成之前同步。*

该规则将之前的规则推广到缓冲通道。
它允许通过缓冲通道对计数信号量进行建模：通道中的项目数量对应于活动使用的数量，通道的容量对应于同时使用的最大数量，发送一个项目获取信号量，并且接收一个项目释放信号量。
这是限制并发的常用习惯用法。

该程序为工作列表中的每个条目启动一个goroutine，但goroutines使用限制通道进行协调以确保一次最多运行三个工作函数。

```
var limit = make(chan int, 3)

func main() {
	for _, w := range work {
		go func(w func()) {
			limit <- 1
			w()
			<-limit
		}(w)
	}
	select{}
}
```

### Locks

sync包实现了两种锁数据类型，sync.Mutex和sync.RWMutex。

*对于任何sync.Mutex或sync.RWMutex变量l和n < m，l.Unlock()的调用n在l.Lock()的调用m返回之前同步。*

这个程序：

```
var l sync.Mutex
var a string

func f() {
	a = "hello, world"
	l.Unlock()
}

func main() {
	l.Lock()
	go f()
	l.Lock()
	print(a)
}
```

保证打印"hello, world"。
对l.Unlock()（在f中）的第一次调用在对l.Lock()（在 main 中）的第二次调用返回之前同步，后者在打印之前排序。

在sync.RWMutex变量l上对l.RLock的任何调用，存在一个n，使得对l.Unlock的第n次调用在从l.RLock返回之前同步，并且对l.RUnlock的匹配调用在返回之前同步从调用n+1返回到l.Lock。

成功调用l.TryLock（或l.TryRLock）等同于调用l.Lock（或 l.RLock）。
不成功的调用根本没有同步效果。
就内存模型而言，l.TryLock（或l.TryRLock）可能被认为即使在互斥锁l被解锁时也能够返回false。

### Once

sync包通过Once类型在存在多个goroutine的情况下提供了一种安全的初始化机制。
多个线程可以执行once.Do(f)，但只有一个会运行f()，而其他调用将阻塞直到f()返回。

once.Do(f)对f()的单个调用的完成在对once.Do(f)的任何调用返回之前同步。

在这个程序中：

```
var a string
var once sync.Once

func setup() {
	a = "hello, world"
}

func doprint() {
	once.Do(setup)
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

调用twoprint只会调用一次setup。
setup函数将在调用print之前完成。
结果是"hello, world"将被打印两次。

### Atomic Values

sync/atomic包中的API统称为"原子操作"，可用于同步不同goroutine的执行。
如果原子操作A的效果被原子操作B观察到，那么A在B之前同步。
程序中执行的所有原子操作的行为就好像以某种顺序一致的顺序执行一样。

前面的定义与C++的顺序一致原子和Java的volatile变量具有相同的语义。

### Finalizers

runtime包提供了一个SetFinalizer函数，该函数添加了一个终结器，当程序不再可以访问特定对象时将调用该终结器。
在完成调用f(x)之前同步调用SetFinalizer(x, f)。

### Additional Mechanisms

sync包提供了其他同步抽象，包括[condition variables](https://go.dev/pkg/sync/#Cond)、[lock-free maps](https://go.dev/pkg/sync/#Map)、[allocation pools](https://go.dev/pkg/sync/#Pool)
和[wait groups](https://go.dev/pkg/sync/#WaitGroup)。
每一个的文档都指定了它对同步所做的保证。

其他提供同步抽象的包也应该记录它们所做的保证。

## Incorrect synchronization

有竞争的程序是不正确的，并且可以表现出非顺序一致的执行。
特别要注意，读取r可能会观察到与r同时执行的任何写入w所写入的值。
即使发生这种情况，也不意味着发生在r之后的读取会观察到发生在w之前的写入。

在这个程序中：

```
var a, b int

func f() {
	a = 1
	b = 2
}

func g() {
	print(b)
	print(a)
}

func main() {
	go f()
	g()
}
```

g可能会打印2然后0。

这个事实使一些常见的习语无效。

双重检查锁定是一种避免同步开销的尝试。
例如，twoprint程序可能被错误地写成：

```
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func doprint() {
	if !done {
		once.Do(setup)
	}
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

但是这不能保证，在doprint中，观察对done的写入就意味着观察对a的写入。
这个版本可以（错误地）打印一个空字符串而不是"hello, world"。

另一个不正确的习语是忙于等待一个值，如：

```
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func main() {
	go setup()
	for !done {
	}
	print(a)
}
```

和以前一样，这也不能保证，在main中，观察对done的写入就意味着观察对a的写入，所以这个程序也可以打印一个空字符串。
更糟糕的是，不能保证对done的写入会被main观察到，因为两个线程之间没有同步事件。
不保证main中的循环完成。

这个主题有更微妙的变体，例如这个程序。

```
type T struct {
	msg string
}

var g *T

func setup() {
	t := new(T)
	t.msg = "hello, world"
	g = t
}

func main() {
	go setup()
	for g == nil {
	}
	print(g.msg)
}
```

即使main观察到g != nil并退出循环，也不能保证它会观察到g.msg的初始化值。

在所有这些示例中，解决方案都是相同的：使用显式同步。

## Incorrect compilation

Go内存模型像限制Go程序一样限制编译器优化。
一些在单线程程序中有效的编译器优化并非在所有Go程序中都有效。
特别是，编译器不能引入原程序中不存在的写，不能允许单次读观察多个值，不能允许单次写写多个值。

以下所有示例均假定`*p`和`*q`指的是多个goroutine可访问的内存位置。

不将数据竞争引入无竞争程序意味着不将它们从出现的条件语句中移出。
例如，编译器不得反转此程序中的条件：

```
*p = 1
if cond {
	*p = 2
}
```

也就是说，编译器不得将程序重写为以下程序：

```
*p = 2
if !cond {
	*p = 1
}
```

如果cond为false，另一个goroutine正在读取`*p`，那么在原来的程序中，另一个goroutine只能观察到`*p`和1的任何先验值。
在改写后的程序中，另一个goroutine可以观察到2，这在以前是不可能的。

不引入数据竞争也意味着不假设循环终止。
例如，编译器通常不得在该程序的循环之前移动对`*p`或`*q`的访问：

```
n := 0
for e := list; e != nil; e = e.next {
	n++
}
i := *p
*q = 1
```

如果list指向循环列表，那么原始程序将永远不会访问`*p`或`*q`，但重写后的程序会。 
（如果编译器可以证明`*p`不会崩溃，那么向前移动 `*p` 是安全的；向前移动`*q`也需要编译器证明没有其他goroutine可以访问`*q`。）

不引入数据竞争也意味着不假设被调用的函数总是返回或没有同步操作。
例如，编译器不得在该程序中的函数调用之前移动对`*p`或`*q`的访问（至少不能不直接了解f的精确行为）：

```
f()
i := *p
*q = 1
```

如果调用永远不会返回，那么原始程序将永远不会访问`*p`或`*q`，但重写后的程序会。
并且如果调用包含同步操作，则原始程序可以在访问`*p`和`*q`之前的边缘之前建立happens，但重写的程序不会。

不允许单次读取观察多个值意味着不从共享内存中重新加载局部变量。 
例如，在此程序中，编译器不得丢弃i并再次从`*p`重新加载它：

```
i := *p
if i < 0 || i >= len(funcs) {
	panic("invalid function index")
}
... complex code ...
// compiler must NOT reload i = *p here
funcs[i]()
```

如果复杂代码需要很多寄存器，单线程程序的编译器可以丢弃 i 而不保存副本，然后在funcs[i]()之前重新加载i = *p。
Go编译器不能，因为`*p`的值可能已经改变。
（相反，编译器可能会将i溢出到堆栈。）

不允许单次写入写入多个值也意味着在写入之前不使用将写入局部变量的内存作为临时存储。
例如，编译器不得在此程序中使用`*p`作为临时存储：

```
*p = i + *p/2
```

也就是说，它不能将程序改写成这个：

```
*p /= 2
*p += i
```

如果i和`*p`开始等于2，则原始代码确实`*p = 3`，因此一个快速线程只能从`*p`中读取2或3。
重写的代码执行`*p = 1`，然后`*p = 3`，从而允许竞速线程也读取1。

请注意，所有这些优化在C/C++编译器中都是允许的：与C/C++编译器共享后端的Go编译器必须注意禁用对Go无效的优化。

请注意，如果编译器可以证明数据竞争不会影响目标平台上的正确执行，则禁止引入数据竞争并不适用。
例如，在基本上所有CPU上，重写都是有效的

```
n := 0
for i := 0; i < m; i++ {
	n += *shared
}
```

进入：

```
n := 0
local := *shared
for i := 0; i < m; i++ {
	n += local
}
```

前提是可以证明`*shared`不会在访问时出错，因为潜在的添加读取不会影响任何现有的并发读取或写入。 
另一方面，重写在源到源翻译器中是无效的。

## Conclusion

编写无数据竞争程序的Go程序员可以依赖于这些程序的顺序一致执行，就像在所有其他现代编程语言中一样。

当谈到有竞争的程序时，程序员和编译器都应该记住这个建议：不要自作聪明。