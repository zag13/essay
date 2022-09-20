## Golang 中的线程池

> [Thread Pooling in Go Programming](https://www.ardanlabs.com/blog/2013/05/thread-pooling-in-go-programming.html)

在使用 go 工作了一段时间后，我知道了如何使用一个无缓冲通道来构建一个 goroutine 池。相比于下面文章中的实现我更喜欢这种方式，话虽如此，下面文章中的描述依然有价值。

*https://github.com/goinggo/work*

**Introduction**

过去在 Microsoft stack 开发服务器软件时，线程池一直是构建鲁棒性比较强的代码的关键。微软在 .Net 中失败了，.Net
给每个进程一个包含数千个线程的线程池，并认为它们可以在运行时管理并发。很早我就认为这不会生效，至少不在我开发的服务器上。

当我在 C/C++ 使用 Win32 API 构建服务时，我抽象了一个 IOCP 的类以便给我提供一个可以工作的线程池。这一直很有效，因为我可以自定义线程池中线程的数量和并发的级别（在任何时间处于激活的线程数）。我为我所有的 C#
开发移植了这段代码。如果你想了解更多关于这方面的信息，我多年前写过一篇文章（http://www.theukwebdesigncompany.com/articles/iocp-thread-pooling.php）。IOCP
为我提供了所需的性能和灵活性。顺便说一句，.NET 线程池在底层使用 IOCP。

线程池的想法非常简单。进入服务器中的任务需要被处理，大部分任务本质上是异步的但并不全是。很多时候任务来自于套接字或是内部程序。线程池将这些任务排队，然后从池中拿出一个线程来处理任务。任务按照被接受的顺序来执行。线程池是高效执行任务的一个很好的方案，因为频繁生成一个新线程会给操作系统带来沉重的负担并导致严重的性能问题。

线程池是如何进行性能调优的呢？您需要确定每个池中应包含的线程数以最快地完成任务。当所有线程都在处理任务时，新任务将会堵塞。需要知道这个是因为有时候更多的线程数反而会拖慢处理速度。这是由多种原因导致的，机器的内核数量、数据库的处理速度等。你应该在测试中来找到这个快乐小数字。

我比较关心的是机器的内核数量、任务类型、这个任务是否会堵塞以及平均堵塞时长等。在 Microsoft stack 上，我发现每个核心三个线程对大多数任务来说是最好的，我还不知道在 Go 中这个数字会是多少。

您也可以为不同类型的任务创建不同的线程池。这样可以单独配置线程池，花时间对服务器进行性能调优以获得最大吞吐量。拥有这种类型的命令和控制来最大化性能至关重要。

在 Go 中，我们不创建线程而是创建 goroutine 。这些 goroutine 的功能类似于线程，Go 来实际管理线程的使用。要了解有关 Go
并发的更多信息，请查看此文档：http://golang.org/doc/effective_go.html#concurrency。

workpool 和 jobpool 是我创建的包，使用 channel 和 goroutine 来实现线程池。

**Workpool**

这个包创建了一个 goroutine 池，专门用于处理发布到池中的任务。一个单独的 goroutine 来对这些任务排队，这个 goroutine 将追踪任务的数量并且在队列满的时候报告错误。

将任务发布到队列中是一个阻塞调用。这样调用者就可以验证任务是否在队列中，维护处于工作态 goroutine 的数量。

下面是一些关于如何使用 workpool 的示例代码：

```
package main

import (
    "bufio"
    "fmt"
    "os"
    "runtime"
    "strconv"
    "time"

    "github.com/goinggo/workpool"
)

type MyWork struct {
    Name string
    BirthYear int
    WP *workpool.WorkPool
}

func (mw *MyWork) DoWork(workRoutine int) {
    fmt.Printf("%s : %d\n", mw.Name, mw.BirthYear)
    fmt.Printf("Q:%d R:%d\n", mw.WP.QueuedWork(), mw.WP.ActiveRoutines())

    // Simulate some delay
    time.Sleep(100 * time.Millisecond)
}

func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())

    workPool := workpool.New(runtime.NumCPU(), 800)

    shutdown := false // Race Condition, Sorry

    go func() {
        for i := 0; i < 1000; i++ {
            work := MyWork {
                Name: "A" + strconv.Itoa(i),
                BirthYear: i,
                WP: workPool,
            }

            if err := workPool.PostWork("routine", &work); err != nil {
                fmt.Printf("ERROR: %s\n", err)
                time.Sleep(100 * time.Millisecond)
            }

            if shutdown == true {
                return
            }
        }
    }()

    fmt.Println("Hit any key to exit")
    reader := bufio.NewReader(os.Stdin)
    reader.ReadString(’\n’)

    shutdown = true

    fmt.Println("Shutting Down")
    workPool.Shutdown("routine")
}
```

查看 main 函数，我们基于机器核心数来创建了一个包含对应 goroutine 的线程池。这意味着每个核心都有一个 goroutine
，如果每个核心都处于工作态的话你就处理不了更多的任务。同样，性能测试将确定这个数字应该是多少。第二个参数是队列的大小。在这种情况下，我让队列足够大以处理所有传入的请求。

MyWork 结构体定义了我执行任务需要的相关状态。DoWork 函数是必须的，需要它来实现 PostWork 回调所需的接口。要传递任务到线程池中都需要对应结构体实现这个方法。

DoWork 方法做了两件事。首先显示了对象的状态，其次报告了队列中的项目数量和处于活跃态的 goroutine 数量。这些数字可用于确定线程池的健康状况和性能测试。

最后，我有一个 goroutine 循环投递任务到线程池中。与此同时，线程池在为每个队列中的任务执行 DoWork。最终 goroutine 完成，线程池继续做它的工作。如果我们在任何时候按回车键，程序就会正常关闭。

PostWork 方法可能会在此示例程序中返回错误。这是因为 PostWork 方法将保证工作被放置在队列中，否则它将失败。失败的唯一原因是队列已满。设置队列长度是一个重要的考虑因素。

**Jobpool**

除了一个实现细节外，jobpool 与 workpool 类似。这个包维护了两个队列，一个用于正常处理，一个用于优先处理。优先级队列中的待处理作业总是在普通队列中的待处理作业之前得到处理。

两个队列的使用使得 jobpool 比 workpool 复杂一些。如果您不需要优先处理，那么使用 workpool 会更快、更高效。

以下是有关如何使用 jobpool 的一些示例代码：

```
package main

import (
    "fmt"
    "time"

    "github.com/goinggo/jobpool"
)

type WorkProvider1 struct {
    Name string
}

func (wp *WorkProvider1) RunJob(jobRoutine int) {
    fmt.Printf("Perform Job : Provider 1 : Started: %s\n", wp.Name)
    time.Sleep(2 * time.Second)
    fmt.Printf("Perform Job : Provider 1 : DONE: %s\n", wp.Name)
}

type WorkProvider2 struct {
    Name string
}

func (wp *WorkProvider2) RunJob(jobRoutine int) {
    fmt.Printf("Perform Job : Provider 2 : Started: %s\n", wp.Name)
    time.Sleep(5 * time.Second)
    fmt.Printf("Perform Job : Provider 2 : DONE: %s\n", wp.Name)
}

func main() {
    jobPool := jobpool.New(2, 1000)

    jobPool.QueueJob("main", &WorkProvider1{"Normal Priority : 1"}, false)

    fmt.Printf("*******> QW: %d AR: %d\n",
        jobPool.QueuedJobs(),
        jobPool.ActiveRoutines())

    time.Sleep(1 * time.Second)

    jobPool.QueueJob("main", &WorkProvider1{"Normal Priority : 2"}, false)
    jobPool.QueueJob("main", &WorkProvider1{"Normal Priority : 3"}, false)

    jobPool.QueueJob("main", &WorkProvider2{"High Priority : 4"}, true)
    fmt.Printf("*******> QW: %d AR: %d\n",
        jobPool.QueuedJobs(),
        jobPool.ActiveRoutines())

    time.Sleep(15 * time.Second)

    jobPool.Shutdown("main")
}
```

在此示例代码中，我们创建了两个 worker 结构体。最好认为每个 worker 是相互独立的。

总的来说，我们创建了一个包含 2 个 g oroutine 并支持 1000 个待处理作业的线程池。首先，我们创建 3 个不同的 WorkProvider1 对象并将它们发布到队列中，将优先级标志设置为 false。接下来我们创建一个
WorkProvider2 对象并将其发布到队列中，将优先级标志设置为 true。

排队的前两个作业将首先处理，因为作业池有 2 个例程。一旦其中一个作业完成，就会从队列中检索下一个作业。WorkProvider2 作业将被下一个处理，因为它被放置在优先级队列中。

要获取工作池和作业池包的副本，请访问 github.com/goinggo

和往常一样，我希望这段代码能在一些小方面帮助你。