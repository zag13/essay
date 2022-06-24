# [Frequently Asked Questions (FAQ)](https://go.dev/doc/faq)

- [Origins](#origins)
    - [What is the purpose of the project?](#what-is-the-purpose-of-the-project)
    - [What is the history of the project?](#what-is-the-history-of-the-project)
    - [What's the origin of the gopher mascot?](#whats-the-origin-of-the-gopher-mascot)
    - [Is the language called Go or Golang?](#is-the-language-called-go-or-golang)
    - [Why did you create a new language?](#why-did-you-create-a-new-language)
    - [What are Go's ancestors?](#what-are-gos-ancestors)
    - [What are the guiding principles in the design?](#what-are-the-guiding-principles-in-the-design)
- [Usage](#usage)
    - [Is Google using Go internally?](#is-google-using-go-internally)
    - [What other companies use Go?](#what-other-companies-use-go)
    - [Do Go programs link with C/C++ programs?](#do-go-programs-link-with-cc-programs)
    - [What IDEs does Go support?](#what-ides-does-go-support)
    - [Does Go support Google's protocol buffers?](#does-go-support-googles-protocol-buffers)
    - [Can I translate the Go home page into another language?](#can-i-translate-the-go-home-page-into-another-language)
- [Design](#design)
    - [Does Go have a runtime?](#does-go-have-a-runtime)
    - [What's up with Unicode identifiers?](#whats-up-with-unicode-identifiers)
    - [Why does Go not have feature X?](#why-does-go-not-have-feature-x)
    - [Why does Go not have generic types?](#why-does-go-not-have-generic-types)
    - [Why does Go not have exceptions?](#why-does-go-not-have-exceptions)
    - [Why does Go not have assertions?](#why-does-go-not-have-assertions)
    - [Why build concurrency on the ideas of CSP?](#why-build-concurrency-on-the-ideas-of-csp)
    - [Why goroutines instead of threads?](#why-goroutines-instead-of-threads)
    - [Why are map operations not defined to be atomic?](#why-are-map-operations-not-defined-to-be-atomic)
    - [Will you accept my language change?](#will-you-accept-my-language-change)
- [Types](#types)
    - [Is Go an object-oriented language?](#is-go-an-object-oriented-language)
    - [How do I get dynamic dispatch of methods?](#how-do-i-get-dynamic-dispatch-of-methods)
    - [Why is there no type inheritance?](#why-is-there-no-type-inheritance)
    - [Why is len a function and not a method?](#why-is-len-a-function-and-not-a-method)
    - [Why does Go not support overloading of methods and operators?](#why-does-go-not-support-overloading-of-methods-and-operators)
    - [Why doesn't Go have "implements" declarations?](#why-doesnt-go-have-implements-declarations)
    - [How can I guarantee my type satisfies an interface?](#how-can-i-guarantee-my-type-satisfies-an-interface)
    - [Why doesn't type T satisfy the Equal interface?](#why-doesnt-type-t-satisfy-the-equal-interface)
    - [Can I convert a []T to an []interface{}?](#can-i-convert-a-t-to-an-interface)
    - [Can I convert []T1 to []T2 if T1 and T2 have the same underlying type?](#can-i-convert-t1-to-t2-if-t1-and-t2-have-the-same-underlying-type)
    - [Why is my nil error value not equal to nil?](#why-is-my-nil-error-value-not-equal-to-nil)
    - [Why are there no untagged unions, as in C?](#why-are-there-no-untagged-unions-as-in-c)
    - [Why does Go not have variant types?](#why-does-go-not-have-variant-types)
    - [Why does Go not have covariant result types?](#why-does-go-not-have-covariant-result-types)
- [Values](#values)
    - [Why does Go not provide implicit numeric conversions?](#why-does-go-not-provide-implicit-numeric-conversions)
    - [How do constants work in Go?](#how-do-constants-work-in-go)
    - [Why are maps built in?](#why-are-maps-built-in)
    - [Why don't maps allow slices as keys?](#why-dont-maps-allow-slices-as-keys)
    - [Why are maps, slices, and channels references while arrays are values?](#why-are-maps-slices-and-channels-references-while-arrays-are-values)
- [Writing Code](#writing-code)
    - [How are libraries documented?](#how-are-libraries-documented)
    - [Is there a Go programming style guide?](#is-there-a-go-programming-style-guide)
    - [How do I submit patches to the Go libraries?](#how-do-i-submit-patches-to-the-go-libraries)
    - [Why does "go get" use HTTPS when cloning a repository?](#why-does-go-get-use-https-when-cloning-a-repository)
    - [How should I manage package versions using "go get"?](#how-should-i-manage-package-versions-using-go-get)
- [Pointers and Allocation](#pointers-and-allocation)
    - [When are function parameters passed by value?](#when-are-function-parameters-passed-by-value)
    - [When should I use a pointer to an interface?](#when-should-i-use-a-pointer-to-an-interface)
    - [Should I define methods on values or pointers?](#should-i-define-methods-on-values-or-pointers)
    - [What's the difference between new and make?](#whats-the-difference-between-new-and-make)
    - [What is the size of an int on a 64 bit machine?](#what-is-the-size-of-an-int-on-a-64-bit-machine)
    - [How do I know whether a variable is allocated on the heap or the stack?](#how-do-i-know-whether-a-variable-is-allocated-on-the-heap-or-the-stack)
    - [Why does my Go process use so much virtual memory?](#why-does-my-go-process-use-so-much-virtual-memory)
- [Concurrency](#concurrency)
    - [What operations are atomic? What about mutexes?](#what-operations-are-atomic-what-about-mutexes)
    - [Why doesn't my program run faster with more CPUs?](#why-doesnt-my-program-run-faster-with-more-cpus)
    - [How can I control the number of CPUs?](#how-can-i-control-the-number-of-cpus)
    - [Why is there no goroutine ID?](#why-is-there-no-goroutine-id)
- [Functions and Methods](#functions-and-methods)
    - [Why do T and *T have different method sets?](#why-do-t-and-t-have-different-method-sets)
    - [What happens with closures running as goroutines?](#what-happens-with-closures-running-as-goroutines)
- [Control flow](#control-flow)
    - [Why does Go not have the ?: operator?](#why-does-go-not-have-the--operator)
- [Packages and Testing](#packages-and-testing)
    - [How do I create a multifile package?](#how-do-i-create-a-multifile-package)
    - [How do I write a unit test?](#how-do-i-write-a-unit-test)
    - [Where is my favorite helper function for testing?](#where-is-my-favorite-helper-function-for-testing)
    - [Why isn't X in the standard library?](#why-isnt-x-in-the-standard-library)
- [Implementation](#implementation)
    - [What compiler technology is used to build the compilers?](#what-compiler-technology-is-used-to-build-the-compilers)
    - [How is the run-time support implemented?](#how-is-the-run-time-support-implemented)
    - [Why is my trivial program such a large binary?](#why-is-my-trivial-program-such-a-large-binary)
    - [Can I stop these complaints about my unused variable/import?](#can-i-stop-these-complaints-about-my-unused-variableimport)
    - [Why does my virus-scanning software think my Go distribution or compiled binary is infected?](#why-does-my-virus-scanning-software-think-my-go-distribution-or-compiled-binary-is-infected)
- [Performance](#performance)
    - [Why does Go perform badly on benchmark X?](#why-does-go-perform-badly-on-benchmark-x)
- [Changes from C](#changes-from-c)
    - [Why is the syntax so different from C?](#why-is-the-syntax-so-different-from-c)
    - [Why are declarations backwards?](#why-are-declarations-backwards)
    - [Why is there no pointer arithmetic?](#why-is-there-no-pointer-arithmetic)
    - [Why are ++ and -- statements and not expressions? And why postfix, not prefix?](#why-are--and----statements-and-not-expressions-and-why-postfix-not-prefix)
    - [Why are there braces but no semicolons? And why can't I put the opening brace on the next line?](#why-are-there-braces-but-no-semicolons-and-why-cant-i-put-the-opening-brace-on-the-next-line)
    - [Why do garbage collection? Won't it be too expensive?](#why-do-garbage-collection-wont-it-be-too-expensive)

## Origins

### What is the purpose of the project?

在 Go 诞生之时，也就是十年前，编程世界与今天不同。生产软件通常是用 C++ 或 Java 编写的，GitHub 还不存在，大多数计算机还不是多处理器，除了 Visual Studio 和 Eclipse 之外，几乎没有可用的 IDE
或其他高级工具，更不用说在 Internet 上免费提供.

同时，我们对开发服务器软件所需语言的过度复杂性感到沮丧。自从 C、C++ 和 Java 等语言首次开发以来，计算机已经变得非常快，但编程本身并没有那么先进。此外，多处理器正在变得普遍，但大多数语言对高效和安全地编程几乎没有帮助。

我们决定退后一步，思考随着技术的发展，未来几年软件工程将面临哪些主要问题，以及新语言如何帮助解决这些问题。例如，多核 CPU
的兴起认为一种语言应该为某种并发性或并行性提供一流的支持。为了使资源管理在大型并发程序中易于处理，垃圾收集或至少是某种安全的自动内存管理是必须的。

这些考虑引发了[一系列讨论](https://commandcenter.blogspot.com/2017/09/go-ten-years-and-climbing.html)，Go
由此产生，首先是作为一组想法和需求，然后是作为一种语言。一个总体目标是 Go 通过启用工具、自动化诸如代码格式化、消除在大型代码库上工作的障碍来更好的帮助程序员。

[Go at Google: Language Design in the Service of Software Engineering](https://go.dev/talks/2012/splash.article) 对 Go
的目标以及如何实现或至少接近这些目标进行了更广泛的描述。

### What is the history of the project?

Robert Griesemer、Rob Pike 和 Ken Thompson 于 2007 年 9 月 21 日在白板上草拟了创建一种新语言的目标。
几天之内，这些目标就形成了一项计划，而且对它会是什么有了一个清晰的想法。设计工作总是在闲暇之余。到 2008 年 1 月，Ken 开始着手开发一个用于探索想法的编译器。它生成 C
代码作为输出。到年中，该语言已经成为一个全职项目，并且已经稳定到尝试生产编译器。2008 年 5 月，Ian Taylor 使用草案规范独立开始了 Go 的 GCC 前端。Russ Cox 于 2008
年底加入并帮助将语言和库从原型变为现实。

Go 于 2009 年 11 月 10 日成为公共开源项目。来自社区的无数人贡献了想法、讨论和代码。

现在全世界有数以百万计的 Go 程序员——gophers——而且每天都在增加。Go 的成功远远超出了我们的预期。

### What's the origin of the gopher mascot?

吉祥物和标志由 [Renée French](https://reneefrench.blogspot.com/) 设计，他还设计了 Plan 9
bunty [Glenda](https://9p.io/plan9/glenda.html)。一篇关于 gopher 的 [博客](https://blog.golang.org/gopher)
阐述了它是如何从几年前她用于[WFMU](https://wfmu.org/)T恤的设计推导出来的。徽标和吉祥物受 [Creative Commons Attribution 3.0](https://creativecommons.org/licenses/by/3.0/)
许可保护。

Gopher 有一张 [model sheet](https://go.dev/doc/gopher/modelsheet.jpg)，说明了它的特征以及如何正确表示它们。2016 年 Renée 在 Gophercon
的一次[演讲](https://www.youtube.com/watch?v=4rw_B4yY69k)中首次展示了该模型表。它具有独特的特性；它是 _Go gopher_，而不仅仅是任何老的 gopher。

### Is the language called Go or Golang?

该语言称为 Go。"golang" 这个名字的出现是因为该网站最初是 _golang.org_。（当时没有 .dev 域名）不过，许多人使用 golang 名称，它作为标签很方便。例如，该语言的 Twitter 标签是 "#golang"
。不管怎样，该语言的名称只是简单的 Go。

备注：虽然 [官方标志](https://blog.golang.org/go-brand) 是两个大写字母，但语言名称是 Go，而不是 GO。

### Why did you create a new language?

Go 的诞生源于我们在 Google 工作时对所使用的现有语言和环境的失望。编程变得太难了，语言的选择是部分原因。必须选择高效编译、高效执行或易于编程的语言；但没有主流语言能同时提供满足这三点。
那些选择易于编程胜过其它两点的程序员转向了动态类型语言，如 Python 和 JavaScript，而不是 C++ 或 Java（在较小程度上）。

我们并不是唯一关注这些的人。经过多年时间后，编程语言环境平静下来，Go 成为了几种新语言（Rust、Elixir、Swift 等）中的第一个，这些新语言使编程语言开发再次成为一个活跃的、几乎是主流的领域。

Go 通过尝试将解释型动态类型语言的编程易用性与静态类型编译语言的效率和安全性结合起来来解决这些问题。它还旨在实现现代化，支持网络和多核计算。最后，使用 Go 工作的目的是 _fast_
：在单台计算机上构建大型可执行文件最多需要几秒钟。为了实现这些目标，需要解决一些语言问题：一个富有表现力但轻量的类型系统；并发和垃圾收集；精确的依赖规范；等等。库或工具无法很好地解决这些问题；这需要一种新的语言。

[Go at Google](https://go.dev/talks/2012/splash.article) 讨论了 Go 语言设计背后的背景和动机，并提供了有关此文章中提供的许多答案的详细信息。

### What are Go's ancestors?

Go 主要属于 C 家族（基本语法），从 Pascal/Modula/Oberon 家族（声明、包）也获得了重要输入，再加上受 Tony Hoare 的 CSP 启发的语言的一些想法，例如 Newsqueak 和
Limbo（并发）。但是，它是一种全面的新语言。在各个方面，语言的设计都是通过思考程序员做什么以及如何进行编程（至少是我们所做的编程）更有效，这意味着更多乐趣。

### What are the guiding principles in the design?

在设计 Go 时，Java 和 C++ 是编写服务器最常用的语言，至少在 Google 是这样。我们觉得这些语言需要太多的记录和重复。一些程序员转向了更动态、更流畅的语言，比如
Python，代价是效率和类型安全。我们认为应该可以在一种语言中实现效率、安全性和流畅性。

Go 试图减少单词的歧义性。在整个设计过程中，我们试图减少混乱和复杂性。没有前置声明，也没有头文件；一切都只声明一次。初始化是富有表现力的、自动的且易于使用的。语法干净，关键字清晰明了。使用 := 声明并初始化会减少类似 (
foo.Foo* myFoo = new(foo.Foo)) 的重复。也许最根本的是，没有类型层次结构：类型只是，它们不必宣布它们的关系。这些简化让 Go 在不牺牲复杂性的情况下具有表现力且易于理解。

另一个重要原则是保持概念正交。可以为任何类型实现方法；结构代表数据，而接口代表抽象；等等。正交性使我们更容易理解事物结合时会发生什么。

## Usage

### Is Google using Go internally?

是的。Go 在 Google 内部的生产环境中被广泛使用。一个简单的例子是 [golang.org](https://golang.org/)
背后的服务器。它只是在 [Google App Engine](https://developers.google.com/appengine/)
上的生产配置中运行的 godoc 文档服务器 。

一个更重要的例子是 Google 的下载服务器，dl.google.com，它提供 Chrome 二进制文件和其他大型可安装文件，例如 apt-get 软件包。

Go 不是谷歌唯一使用的语言，远非如此，但它是许多领域的关键语言，包括 [站点可靠性工程 (SRE)](https://go.dev/talks/2013/go-sreops.slide) 和大规模数据处理。

### What other companies use Go?

Go 的使用在全球范围内都在增长，尤其是但绝不仅仅是在云计算领域。用 Go 编写的几个主要云基础设施项目是 Docker 和 Kubernetes，但还有更多。

不过，这不仅仅是云。Go Wiki 包含一个定期更新的[页面](https://github.com/golang/go/wiki/GoUsers)，其中列出了一些使用 Go 的公司。

Wiki 还有一个页面，其中包含指向使用该语言的公司和项目的 [成功故事](https://github.com/golang/go/wiki/SuccessStories) 的链接。

### Do Go programs link with C/C++ programs?

可以在同一个地址空间中同时使用 C 和 Go，但这不是一种天然的配合，可能需要特殊的接口软件。此外，将 C 与 Go 代码链接放弃了 Go 提供的内存安全和堆栈管理属性。有时绝对有必要使用 C 库来解决问题，但这样做总是会引入纯 Go
代码不存在的风险元素，因此请谨慎使用。

如果你确实需要在 Go 中使用 C，如何进行取决于 Go 编译器的实现。Go 团队支持三种 Go 编译器实现。它们是 gc，默认编译器，gccgo，它使用 GCC 后端，以及一个不太成熟的 gollvm，它使用 LLVM 基础设施。

Gc 使用与 C 不同的调用约定和链接器，因此不能直接从 C 程序调用，反之亦然。cgo 程序为“外部函数接口”提供了机制，以允许从 Go 代码安全调用 C 库。SWIG 将此功能扩展到 C++ 库。

您还可以将 cgo 和 SWIG 与 Gccgo 和 gollvm 一起使用。由于它们使用传统的 API，因此也可以非常小心地将来自这些编译器的代码直接与 GCC/LLVM 编译的 C 或 C++
程序链接起来。但是，安全地这样做需要了解所有相关语言的调用约定，以及从 Go 调用 C 或 C++ 时的堆栈限制。

### What IDEs does Go support?

Go 项目不包含自定义 IDE，但语言和库的设计使分析源代码变得容易。因此，大多数知名的编辑器和 IDE 都直接或通过插件很好地支持 Go。

具有良好 Go 支持的知名 IDE 和编辑器列表包括 Emacs、Vim、VSCode、Atom、Eclipse、Sublime、IntelliJ（通过称为 Goland 的自定义变体）等等。很有可能你最喜欢的环境是 Go 编程的高效环境。

### Does Go support Google's protocol buffers?

一个单独的开源项目提供了必要的编译器插件和库。它可以在 [github.com/golang/protobuf/](https://github.com/golang/protobuf) 上找到。

### Can I translate the Go home page into another language?

绝对地。我们鼓励开发人员用他们自己的语言制作 Go 语言网站。但是，如果您选择将 Google 徽标或品牌添加到您的网站（它不会出现在 [golang.org](https://go.dev/)
上），您需要遵守 [www.google.com/permissions/guidelines.html](https://www.google.com/permissions/guidelines.html) 上的指南

## Design

### Does Go have a runtime?

Go 确实有一个扩展的库，称为 runtime，它是每个 Go 程序的一部分。运行时库实现了 Go 语言的垃圾收集、并发、堆栈管理和其他关键特性。尽管它在语言中更为核心，但 Go 的运行时类似于 C 库 libc。

然而，重要的是要理解 Go 的运行时不包括虚拟机，例如 Java 运行时提供的。Go 程序被提前编译为本机机器代码（或 JavaScript 或 WebAssembly，对于某些变体实现）。
因此，尽管该术语通常用于描述程序运行的虚拟环境，但在 Go 中，“runtime” 一词只是提供关键语言服务的库的名称。

### What's up with Unicode identifiers?

在设计 Go 时，我们希望确保它不会过度以 ASCII 为中心，这意味着将标识符的空间从 7 位 ASCII 的范围中扩展出来。Go 的规则——标识符字符必须是 Unicode
定义的字母或数字——易于理解和实现，但有一些限制。例如，组合字符被设计排除在外，并且排除了一些语言，如梵文。

这条规则还有另一个不幸的后果。由于导出的标识符必须以大写字母开头，因此根据定义，从某些语言中的字符创建的标识符不能被导出。目前唯一的解决方案是使用类似X日本语的东西，这显然不能令人满意。

自该语言的最早版本以来，人们一直在思考如何最好地扩展标识符空间以适应使用其他本地语言的程序员。究竟该做什么仍然是一个活跃的讨论话题，并且该语言的未来版本在标识符的定义上可能会更加自由。例如，它可能会采用 Unicode
组织对标识符的建议中的一些想法。无论发生什么，都必须兼容地完成，同时保留（或扩展）字母大小写确定标识符可见性的方式，这仍然是我们最喜欢的 Go 特性之一。

目前，我们有一个简单的规则，以后可以在不破坏程序的情况下进行扩展，它可以避免肯定会因允许标识符不明确的规则而产生的错误。

### Why does Go not have feature X?

每种语言都包含新颖的功能，并省略了某人最喜欢的功能。Go 的设计着眼于编程的便利性、编译的速度、概念的正交性以及支持并发和垃圾收集等特性的需求。
您最喜欢的功能可能会因为不适合而丢失，因为它会影响编译速度或设计的清晰度，或者因为它会使基本系统模型变得过于困难。

如果 Go 缺少功能 X 让您感到困扰，请原谅我们并查阅 Go 已经具有的功能。您可能会发现它们以有趣的方式弥补了 X 的缺失。

### Why does Go not have generic types?

[实现一种泛型类型的语言提案](https://go.dev/issue/43651) 已被接受以包含在该语言中。如果一切顺利，它将在 Go 1.18 版本中可用。

Go 旨在作为一种用于编写随时间推移易于维护的服务器程序的语言。
（有关更多背景信息，请参阅[本文](https://go.dev/talks/2012/splash.article)）设计集中在可伸缩性、可读性和并发性等方面。多态编程在当时似乎对语言的目标并不重要，因此为了简单起见被排除在外。

该语言现在更加成熟，并且有考虑某种形式的通用编程的空间。但是，仍有一些警告。

泛型很方便，但它们的代价是类型系统和运行时的复杂性。尽管我们仍在继续思考，但我们还没有找到一种能够提供与复杂性成比例的价值的设计。同时，Go
的内置映射和切片，以及使用空接口构造容器（通过显式拆箱）的能力，意味着在许多情况下，即使不太顺利，也可以编写出泛型支持的代码。

话题保持开放。要查看之前为 Go 设计良好的泛型解决方案的几次失败尝试，请参阅[此提案](https://go.dev/issue/15292)。

### Why does Go not have exceptions?

我们相信将异常耦合到控制结构中，就像 try-catch-finally 一样，会导致代码复杂。它还倾向于鼓励程序员将太多的普通错误（例如无法打开文件）标记为异常。

Go 采用了不同的方法。对于简单的错误处理，Go
的多值返回可以很容易地报告错误而不会重载返回值。[规范的错误类型与 Go 的其他特性相结合](https://go.dev/doc/articles/error_handling.html)，使错误处理变得愉快，但与其他语言中的错误处理完全不同。

Go 还有一些内置函数可以发出信号并从真正的异常情况中恢复。恢复机制仅作为函数状态的一部分在错误后被拆除，这足以处理灾难，但不需要额外的控制结构，如果使用得当，可以产生干净的错误处理代码。

有关详细信息，请参阅 [Defer、Panic、和 Recover](https://go.dev/doc/articles/defer_panic_recover.html)。
此外，[Errors are values](https://blog.golang.org/errors-are-values)
描述了一种在 Go 中干净地处理错误的方法，通过演示，由于错误只是值，因此可以在错误处理中部署 Go 的全部功能。

### Why does Go not have assertions?

Go 不提供断言。不可否认，它们很方便，但我们的经验是，程序员将它们用作拐杖，以避免考虑正确的错误处理和报告。
正确的错误处理意味着服务器继续运行而不是在发生非致命错误后崩溃。正确的错误报告意味着错误是直截了当的，使程序员免于解释大量的崩溃跟踪。当看到错误的程序员不熟悉代码时，精确的错误尤为重要。

我们知道这是一个争论点。Go 语言和库中有许多与现代实践不同的地方，仅仅是因为我们觉得有时值得尝试不同的方法。

### Why build concurrency on the ideas of CSP?

随着时间的推移，并发和多线程编程以困难而闻名。
我们认为这部分是由于复杂的设计（例如 [pthread](https://en.wikipedia.org/wiki/POSIX_Threads)），部分是由于过分强调低级细节（例如互斥体、条件变量和内存屏障）。
更高级别的接口可以实现更简单的代码，即使在幕后仍然存在互斥锁等。

为并发提供高级语言支持的最成功的模型之一来自 Hoare 的 Communicating Sequential Processes，或 CSP。Occam 和 Erlang 是源自 CSP 的两种众所周知的语言。Go
的并发原语源自家族树的不同部分，其主要贡献是将通道作为第一类对象的强大概念。几种早期语言的经验表明，CSP 模型非常适合过程语言框架。

### Why goroutines instead of threads?

Goroutines 是使并发易于使用的一部分。这个想法已经存在了一段时间，它是将独立执行的函数（协程）多路复用到一组线程上。
当协程阻塞时，例如通过调用阻塞系统调用，运行时会自动将同一操作系统线程上的其他协程移动到不同的可运行线程，这样它们就不会被阻塞。程序员看不到这些，这就是重点。结果，我们称之为
goroutine，可能非常便宜：除了堆栈内存之外，它们几乎没有开销，只有几千字节。

为了使堆栈变小，Go 的运行时使用可调整大小的有界堆栈。一个新创建的 goroutine 有几千字节，这几乎总是足够的。如果不是这样，运行时会自动增长（和缩小）用于存储堆栈的内存，从而允许许多 goroutines 存在于适度的内存中。
CPU 开销平均每个函数调用大约三个廉价指令。在同一个地址空间中创建数十万个 goroutine 是很实用的。如果 goroutines 只是线程，系统资源会以更少的数量耗尽。

### Why are map operations not defined to be atomic?

经过长时间的讨论，我们认为 map 的典型使用不需要多个 goroutine 的安全访问，在需要的情况下，map
可能是一些更大的数据结构或已经同步的计算的一部分。因此，要求所有映射操作都获取互斥锁会减慢大多数程序的速度并增加少数程序的安全性。然而，这不是一个容易的决定，因为这意味着不受控制的地图访问可能会使程序崩溃。

该语言不排除原子映射更新。在需要时，例如在托管不受信任的程序时，实现可以互锁地图访问。

只有在发生更新时，地图访问才是不安全的。只要所有的 goroutine 都只是读取——在 map 中查找元素，包括使用 for range循环遍历它——而不是通过分配给元素或执行删除来更改 map，它们就可以安全地在不同步的情况下同时访问
map。

作为正确使用地图的帮助，该语言的一些实现包含一个特殊的检查，当地图被并发执行不安全地修改时，该检查会在运行时自动报告。

### Will you accept my language change?

人们经常建议改进语言——[邮件列表](https://groups.google.com/group/golang-nuts)包含丰富的此类讨论历史——但这些更改很少被接受。

尽管 Go 是一个开源项目，但语言和库受到 [兼容性承诺](https://go.dev/doc/go1compat.html)
的保护，该承诺防止破坏现有程序的更改，至少在源代码级别（程序可能需要偶尔重新编译以保持最新状态）。如果您的提案违反了 Go 1 规范，我们甚至无法接受这个想法，无论其优点如何。Go 未来的主要版本可能与 Go 1
不兼容，但关于该主题的讨论才刚刚开始，有一点是肯定的：在此过程中引入的此类不兼容问题将非常少。此外，兼容性承诺鼓励我们为旧程序提供一条自动前进的路径，以便在出现这种情况时进行调整。

即使您的提案与 Go 1 规范兼容，它也可能不符合 Go
设计目标的精神。这篇文章：[Go at Google: Language Design in the Service of Software Engineering](https://go.dev/talks/2012/splash.article)
解释 了 Go 的起源及其设计背后的动机。

## Types

### Is Go an object-oriented language?

是也不是。尽管 Go 有类型和方法并允许面向对象的编程风格，但没有类型层次结构。Go 中的 “interface”
概念提供了一种不同的方法，我们认为这种方法易于使用，并且在某些方面更通用。还有一些方法可以将类型嵌入到其他类型中，以提供与子类化类似但不完全相同的东西。此外，Go 中的方法比 C++ 或 Java
中的方法更通用：它们可以为任何类型的数据定义，甚至是内置类型，例如普通的 “unboxed” 整数。它们不限于结构体（类）。

此外，没有类型层次结构使得 Go 中的 “objects” 感觉比 C++ 或 Java 等语言更轻量。

### How do I get dynamic dispatch of methods?

拥有动态分派方法的唯一方法是通过接口。结构体或任何其他具体类型的方法总是静态解析。

### Why is there no type inheritance?

面向对象的编程，至少在最著名的语言中，涉及对类型之间关系的过多讨论，这些关系通常可以自动派生。Go 采用了不同的方法。

不需要程序员提前声明两种类型是相关的，在 Go 中，类型自动满足任何指定其方法子集的接口。除了减少繁琐的书写外，这种方法还具有真正的优势。类型可以同时满足多个接口，没有传统多重继承的复杂性。
接口可以是非常轻量级的——具有一个甚至零个方法的接口可以表达一个有用的概念。如果出现新想法或进行测试，可以在事后添加接口——无需注释原始类型。因为类型和接口之间没有明确的关系，所以没有需要管理或讨论的类型层次结构。

可以使用这些想法来构建类似于类型安全的 Unix 管道的东西。例如，查看 fmt.Fprintf 如何将格式化打印到任何输出，而不仅仅是文件，或者 package bufio 如何与文件 I/O 完全分离，或者图像包如何生成压缩图像文件。
所有这些想法都源于表示单个方法（Write）的单个接口（io.Writer）。而这只是表面问题。Go 的接口对程序的结构有着深远的影响。

这需要一些时间来适应，但这种隐含的类型依赖风格是 Go 最有成效的事情之一。

### Why is len a function and not a method?

我们讨论了这个问题，但决定将 len 和类似函数在语言层面实现，这样不会使关于基本类型的接口（在 Go 类型意义上）的问题复杂化。

### Why does Go not support overloading of methods and operators?

如果方法分派也不需要进行类型匹配，则它会被简化。使用其他语言的经验告诉我们，具有相同名称但不同签名的各种方法有时很有用，但在实践中也可能令人困惑和脆弱。仅按名称匹配并要求类型的一致性是 Go 类型系统中一个主要的简化决定。

关于运算符重载，它似乎比绝对要求更方便。同样，没有它，事情会变得更简单。

### Why doesn't Go have "implements" declarations?

Go 类型通过实现接口的方法来满足接口，仅此而已。此属性允许在无需修改现有代码的情况下定义和使用接口。
它支持一种促进关注点分离和改进代码重用的 [结构类型](https://en.wikipedia.org/wiki/Structural_type_system)，并且更容易在代码开发时出现的模式上进行构建。接口的语义是 Go
灵活、轻量级的主要原因之一。

有关更多详细信息，请参阅[关于类型继承的问题](https://go.dev/doc/faq#inheritance)。

### How can I guarantee my type satisfies an interface?

您可以要求编译器通过尝试使用 T 的零值或指向 T 的指针进行赋值来检查类型 T 是否实现了接口 I，视情况而定：

```
type T struct{}
var _ I = T{}       // Verify that T implements I.
var _ I = (*T)(nil) // Verify that *T implements I.
```

如果 T（或 *T，相应地）没有实现 I，错误将在编译时被发现。

如果您希望接口的用户明确声明他们实现了它，您可以将具有描述性名称的方法添加到接口的方法集中。例如：

```
type Fooer interface {
    Foo()
    ImplementsFooer()
}
```

然后类型必须实现 ImplementsFooer 方法才能成为
Fooer，清楚地记录事实并在 [go doc](https://go.dev/cmd/go/#hdr-Show_documentation_for_package_or_symbol) 的输出中宣布它。

```
type Bar struct{}
func (b Bar) ImplementsFooer() {}
func (b Bar) Foo() {}
```

大多数代码没有使用这些约束，因为它们限制了接口思想的实用性。但有时，它们对于解决相似接口之间的歧义是必要的。

### Why doesn't type T satisfy the Equal interface?

考虑这个简单的接口来表示一个可以将自己与另一个值进行比较的对象：

```
type Equaler interface {
    Equal(Equaler) bool
}
```

和这种类型，T：

```
type T int
func (t T) Equal(u T) bool { return t == u } // does not satisfy Equaler
```

与某些多态类型系统中的类似情况不同，T 没有实现 Equaler。T.Equal 的参数类型是 T，而不是字面上要求的 Equaler 类型。

在 Go 中，类型系统不提倡 Equal 的参数；这是程序员的责任，如 T2 类型所示，它确实实现了 Equaler：

```
type T2 int
func (t T2) Equal(u Equaler) bool { return t == u.(T2) }  // satisfies Equaler
```

但是，即使这与其他类型系统不同，因为在 Go 中，任何满足 Equaler 的类型都可以作为参数传递给 T2.Equal，并且在运行时我们必须检查参数是否属于 T2 类型。一些语言安排在编译时做出这种保证。

一个相关的例子是相反的：

```
type Opener interface {
   Open() Reader
}

func (t T3) Open() *os.File
```

在 Go 中，T3 不满足 Opener，尽管它可能在另一种语言中。

虽然在这种情况下，Go 的类型系统确实为程序员做的更少，但缺少子类型使得关于接口满足的规则很容易陈述：函数的名称和签名是否正是接口的名称和签名？ Go 的规则也很容易高效地实现。我们认为这些好处抵消了自动类型提升的不足。如果有一天 Go
采用某种形式的多态类型，我们期望会有一种方法来表达这些示例的想法，并且还可以对它们进行静态检查。

### Can I convert a []T to an []interface{}?

不能直接这样做。语言规范不允许这样做，因为这两种类型在内存中没有相同的表示。有必要将元素单独复制到目标切片。此示例将 int 的切片转换为 interface{} 的切片：

```
t := []int{1, 2, 3, 4}
s := make([]interface{}, len(t))
for i, v := range t {
    s[i] = v
}
```

### Can I convert []T1 to []T2 if T1 and T2 have the same underlying type?

该代码示例的最后一行无法编译。

```
type T1 int
type T2 int
var t1 T1
var x = T2(t1) // OK
var st1 []T1
var sx = ([]T2)(st1) // NOT OK
```

在 Go 中，类型与方法密切相关，因为每个命名类型都有一个（可能为空的）方法集。一般规则是您可以更改要转换的类型的名称（因此可能更改其方法集），但不能更改复合类型元素的名称（和方法集）。Go 要求你明确类型转换。

### Why is my nil error value not equal to nil?

在幕后，接口被实现为两个元素，一个类型 T 和一个值 V。V 是一个具体的值，例如 int、struct 或 pointer，而不是接口本身，并且具有类型 T。例如，如果我们存储接口中的 int 值 3，结果接口值示意性地具有 (
T=int, V=3)。值 V 也称为接口的动态值，因为给定的接口变量在程序执行期间可能保存不同的值 V（和相应的类型 T）。

只有当 V 和 T 都未设置时，接口值才为 nil，（T=nil，V 未设置），特别是，nil 接口将始终保持 nil 类型。如果我们将 *int 类型的 nil 指针存储在接口值中，则内部类型将是 *int 而与指针的值无关：(T=*
int, V=nil)。因此，即使内部的指针值 V 为 nil，这样的接口值也将非 nil。

这种情况可能会令人困惑，并且当 nil 值存储在接口值中时会出现，例如错误返回：

```
func returnsError() error {
	var p *MyError = nil
	if bad() {
		p = ErrBad
	}
	return p // Will always return a non-nil error.
}
```

如果一切顺利，该函数返回一个 nil p，因此返回值是一个错误接口值持有 (T=*MyError, V=nil)。这意味着如果调用者将返回的错误与 nil 进行比较，即使没有发生任何不好的事情，它也总是看起来好像有错误。
要向调用者返回正确的 nil 错误，函数必须返回显式的 nil：

```
func returnsError() error {
	if bad() {
		return ErrBad
	}
	return nil
}
```

对于返回错误的函数，最好在其签名中始终使用错误类型（正如我们上面所做的那样），而不是像 *MyError 这样的具体类型，以帮助确保正确创建错误。例如，os.Open 返回一个错误，即使不是 nil，它也总是具体类型 *
os.PathError。

每当使用接口时，都会出现与此处描述的类似情况。请记住，如果接口中存储了任何具体值，则接口不会为 nil。
有关更多信息，请参阅[反射定律](https://go.dev/doc/articles/laws_of_reflection.html)。

### Why are there no untagged unions, as in C?

未标记的 unions 会违反 Go 的内存安全保证。

### Why does Go not have variant types?

变体类型，也称为代数类型，提供了一种方法来指定一个值可能采用一组其他类型中的一个，但仅限于这些类型。系统编程中的一个常见示例将指定错误是网络错误、安全错误或应用程序错误，并允许调用者通过检查错误的类型来区分问题的根源。
另一个例子是语法树，其中每个节点可以是不同的类型：声明、语句、赋值等等。

我们考虑在 Go 中添加变体类型，但在讨论后决定将它们排除在外，因为它们以令人困惑的方式与接口重叠。如果变体类型的元素本身就是接口会发生什么？

此外，该语言已经涵盖了一些变体类型地址。错误示例很容易表达，使用接口值来保存错误并使用类型开关来区分情况。语法树示例也是可行的，尽管没有那么优雅。

### Why does Go not have covariant result types?

协变结果类型意味着像

```
type Copyable interface {
	Copy() interface{}
}
```

该方法会满足

```
func (v Value) Copy() Value
```

因为 Value 实现了空接口。在 Go 中方法类型必须完全匹配，所以 Value 没有实现 Copyable。Go 将类型做什么的概念——它的方法——从类型的实现中分离出来。如果两个方法返回不同的类型，它们就不是在做同样的事情。
想要协变结果类型的程序员经常尝试通过接口来表达类型层次结构。在 Go 中，在接口和实现之间有一个清晰的分离更为自然。

## Values

### Why does Go not provide implicit numeric conversions?

C 中数字类型之间自动转换的便利性被它引起的混乱所抵消。什么时候表达式是无符号的？ 价值有多大？ 它溢出了吗？ 结果是否可移植，独立于执行它的机器？ 它也使编译器复杂化；“通常的算术转换”不容易实现，并且跨架构不一致。
出于可移植性的原因，我们决定以代码中的一些显式转换为代价使事情变得清晰明了。不过，Go 中常量的定义——没有符号和大小注释的任意精度值——大大改善了问题。

一个相关的细节是，与 C 不同，int 和 int64 是不同的类型，即使 int 是 64 位类型。int 类型是通用的；如果您关心整数包含多少位，Go 鼓励您明确表示。

### How do constants work in Go?

尽管 Go 对不同数值类型的变量之间的转换有严格的要求，但语言中的常量要灵活得多。诸如 23、3.14159 和 math.Pi 等字面常量占据了一种理想的数字空间，具有任意精度，没有上溢或下溢。例如，math.Pi 的值在源代码中被指定为
63 位，并且涉及该值的常量表达式保持的精度超出了 float64 所能容纳的范围。只有将常量或常量表达式分配给变量（程序中的内存位置）时，它才会成为具有通常浮点属性和精度的“计算机”数字。

此外，由于它们只是数字，而不是类型值，因此 Go 中的常量可以比变量更自由地使用，从而缓解了围绕严格转换规则的一些尴尬。可以写出诸如

```
sqrt2 := math.Sqrt(2)
```

无需编译器抱怨，因为理想的数字 2 可以安全准确地转换为 float64 以调用 math.Sqrt。

一篇名为 [Constants](https://blog.golang.org/constants) 的博客文章更详细地探讨了这个问题。

### Why are maps built in?

与字符串相同的原因是：它们是如此强大和重要的数据结构，它提供了一个具有语法支持的出色实现，使编程更加愉快。我们相信 Go 的 map 实现足够强大，可以满足绝大多数用途。
如果特定应用程序可以从自定义实现中受益，则可以编写一个，但在语法上不会那么方便；这似乎是一个合理的权衡。

### Why don't maps allow slices as keys?

Map 查找需要一个相等运算符，切片没有实现。它们没有实现相等，因为在此类类型上没有很好地定义相等；有多个考虑因素，涉及浅层与深层比较、指针与值比较、如何处理递归类型等等。
我们可能会重新审视这个问题——实现切片相等不会使任何现有程序无效——但如果不清楚切片相等应该意味着什么，现在将其忽略会更简单。

在 Go 1 中，与以前的版本不同，为结构和数组定义了相等性，因此这些类型可以用作映射键。但是，切片仍然没有相等的定义。

### Why are maps, slices, and channels references while arrays are values?

关于这个话题有很多历史。早期，map 和 channel 在语法上是指针，不可能声明或使用非指针实例。此外，我们还为数组应该如何工作而苦苦挣扎。最终我们决定指针和值的严格分离使语言更难使用。
更改这些类型以充当对关联的共享数据结构的引用解决了这些问题。这种变化给语言增加了一些令人遗憾的复杂性，但对可用性产生了很大影响：Go 在引入时成为一种更高效、更舒适的语言。

## Writing Code

### How are libraries documented?

有一个用 Go 编写的程序 godoc，它从源代码中提取包文档并将其作为一个网页提供，其中包含指向声明、文件等的链接。一个实例正在 [golang.org/pkg/](https://go.dev/pkg/) 上运行。事实上，godoc
在 [golang.org/](https://go.dev/) 上实现了完整的站点。

Godoc 实例可以配置为在它显示的程序中提供丰富的、交互式的符号静态分析；详细信息在[此处](https://go.dev/lib/godoc/analysis/help.html)列出。

为了从命令行访问文档，[go](https://go.dev/pkg/cmd/go/)
工具有一个 [doc](https://go.dev/pkg/cmd/go/#hdr-Show_documentation_for_package_or_symbol) 子命令，它为相同的信息提供了一个文本界面。

### Is there a Go programming style guide?

没有明确的风格指南，尽管肯定有一种可识别的 "Go style"。

Go 已经建立了约定来指导围绕命名、布局和文件组织的决策。[Effective Go](https://go.dev/doc/effective_go.html) 包含关于这些主题的一些建议。更直接地说，程序 gofmt
是一个漂亮的打印机，其目的是强制执行布局规则。它取代了允许解释的通常的注意事项纲要。存储库中的所有 Go 代码，以及开源世界中的绝大多数代码，都是通过 gofmt 运行的。

[Go Code Review Comments](https://go.dev/s/comments) 的文档是一组非常简短的文章，这些文章介绍了程序员经常错过的 Go 习惯用法的细节。对于为 Go
项目进行代码审查的人来说，它是一个方便的参考。

### How do I submit patches to the Go libraries?

库源位于存储库的 src 目录中。如果您想做出重大改变，请在开始之前在邮件列表中讨论。

有关如何继续进行的更多信息，请参阅[Contributing to the Go project](https://go.dev/doc/contribute.html)。

### Why does "go get" use HTTPS when cloning a repository?

公司通常只允许标准 TCP 端口 80 (HTTP) 和 443 (HTTPS) 上的传出流量，阻止其他端口上的传出流量，包括 TCP 端口 9418 (git) 和 TCP 端口 22 (SSH)。当使用 HTTPS 而不是 HTTP
时，git 默认强制执行证书验证，提供针对中间人、窃听和篡改攻击的保护。因此 go get 命令使用 HTTPS 以确保安全。

Git 可以配置为通过 HTTPS 进行身份验证或使用 SSH 代替 HTTPS。要通过 HTTPS 进行身份验证，您可以在 git 查询的 $HOME/.netrc 文件中添加一行：

```
machine github.com login USERNAME password APIKEY
```

对于 GitHub 帐户，密码可以是[个人访问令牌](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)。

Git 也可以配置为使用 SSH 代替 HTTPS 来匹配给定前缀的 URL。例如，要对所有 GitHub 访问使用 SSH，请将这些行添加到您的 ~/.gitconfig：

```
[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
```

### How should I manage package versions using "go get"?

Go 工具链有一个内置系统，用于管理版本化的相关包集，称为模块。模块是在 [Go 1.11](https://go.dev/doc/go1.11#modules)
中引入的，并且自 [1.14](https://go.dev/doc/go1.14#introduction) 起已准备好用于生产。

要使用模块创建项目，请运行 go mod init。此命令创建一个跟踪依赖版本的 go.mod 文件。

```
go mod init example/project
```

要添加、升级或降级依赖项，请运行 go get：

```
go get golang.org/x/text@v0.3.5
```

有关入门的更多信息，请参阅 [Tutorial: Create a module](https://go.dev/doc/tutorial/create-module.html)。

有关使用模块管理依赖项的指南，请参阅 [Developing modules](https://go.dev/doc/#developing-modules)。

模块中的包应遵循[导入兼容性规则](https://research.swtch.com/vgo-import)，随着它们的发展保持向后兼容性：

```
    If an old package and a new package have the same import path,
    the new package must be backwards compatible with the old package.
```

[Go 1 兼容性指南](https://go.dev/doc/go1compat.html) 在这里是一个很好的参考：不要删除导出的名称，鼓励标记复合字面量，等等。如果需要不同的功能，请添加新名称而不是更改旧名称。

模块使用[语义版本](https://semver.org/)控制和语义导入版本控制来编码。如果需要中断兼容性，请以新的主要版本发布模块。主版本 2
及更高版本的模块需要 [主版本后缀](https://go.dev/ref/mod#major-version-suffixes)
作为其路径的一部分（如 /v2）。这保留了导入兼容性规则：模块的不同主要版本中的包具有不同的路径。

## Pointers and Allocation

### When are function parameters passed by value?

与 C 家族中的所有语言一样，Go 中的所有内容都是按值传递的。也就是说，一个函数总是得到一个被传递的东西的副本，就好像有一个赋值语句将值分配给参数一样。例如，将 int 值传递给函数会生成 int
的副本，传递指针值会生成指针的副本，但不会复制它指向的数据。（有关这如何影响方法接收器的讨论，请参见[后面的部分](https://go.dev/doc/faq#methods_on_values_or_pointers)）

映射和切片值的行为类似于指针：它们是包含指向底层映射或切片数据的指针的描述符。复制地图或切片值不会复制它指向的数据。复制接口值会复制存储在接口值中的事物。如果接口值包含一个结构，则复制接口值会复制该结构。如果接口值包含一个指针，则复制接口值会复制指针，但不会复制它指向的数据。

请注意，此讨论是关于操作的语义。只要优化不改变语义，实际实现可能会应用优化以避免复制。

### When should I use a pointer to an interface?

几乎从不。指向接口值的指针只出现在罕见的、棘手的情况下，包括为延迟评估而伪装接口值的类型。

将指向接口值的指针传递给期望接口的函数是一个常见的错误。编译器会抱怨这个错误，但情况仍然可能令人困惑，因为有时需要一个指针来满足接口。洞察力是，尽管指向具体类型的指针可以满足接口，但有一个例外，指向接口的指针永远不能满足接口。

考虑变量声明，

```
var w io.Writer
```

打印函数 fmt.Fprintf 将满足 io.Writer 的值作为其第一个参数——它实现了规范的 Write 方法。因此我们可以写

```
fmt.Fprintf(w, "hello, world\n")
```

但是，如果我们传递 w 的地址，程序将无法编译。

```
fmt.Fprintf(&w, "hello, world\n") // Compile-time error.
```

一个例外是，任何值，甚至是指向接口的指针，都可以分配给空接口类型 (interface{}) 的变量。即便如此，如果值是指向接口的指针，那几乎肯定是错误的。结果可能令人困惑。

### Should I define methods on values or pointers?

```
func (s *MyStruct) pointerMethod() { } // method on pointer
func (s MyStruct)  valueMethod()   { } // method on value
```

对于不习惯指针的程序员来说，这两个例子之间的区别可能会让人感到困惑，但情况其实很简单。当在一个类型上定义一个方法时，接收者（在上面的例子中）表现得就像它是方法的一个参数一样。将接收者定义为值还是指针是同一个问题，那么，函数参数应该是值还是指针。有几个考虑。

首先，也是最重要的，该方法是否需要修改接收器？如果是这样，接收者必须是一个指针。（切片和映射充当引用，因此它们的故事有点微妙，但例如要在方法中更改切片的长度，接收者仍然必须是指针。）在上面的示例中，如果 pointerMethod
修改了s，调用者将看到这些更改，但 valueMethod 是使用调用者参数的副本调用的（这是传递值的定义），因此调用者将看不到它所做的更改。

顺便说一句，在 Java 中，方法接收器始终是指针，尽管它们的指针性质有些伪装（并且有人提议在语言中添加值接收器）。Go 中的值接收器是不寻常的。

其次是效率的考虑。如果接收器很大，例如一个大结构体，使用指针接收器会便宜得多。

其次是一致性。如果该类型的某些方法必须具有指针接收器，那么其余的也应该具有，因此无论该类型如何使用，方法集都是一致的。有关详细信息，请参阅[方法集](https://go.dev/doc/faq#different_method_sets)部分。

对于基本类型、切片和小型结构体等类型，值接收器非常便宜，因此除非方法的语义需要指针，否则值接收器是高效且清晰的。

### What's the difference between new and make?

简而言之：new 分配内存，make 同时会初始化 slice、map 和 channel。

参阅 [relevant section of Effective Go](https://go.dev/doc/effective_go.html#allocation_new) 获取更多信息。

### What is the size of an int on a 64 bit machine?

int 和 uint 的大小是特定于实现的，但在给定平台上彼此相同。为了可移植性，依赖于特定大小值的代码应使用显式大小的类型，如 int64。在 32 位机器上，编译器默认使用 32 位整数，而在 64 位机器上，整数有 64 位。
（从历史上看，这并不总是正确的。）

另一方面，浮点标量和复数类型总是有大小的（没有浮点数或复数基本类型），因为程序员在使用浮点数时应该注意精度。用于（无类型）浮点常量的默认类型是 float64。因此 foo := 3.0 声明了一个 float64 类型的变量 foo。
对于由（无类型）常量初始化的 float32 变量，必须在变量声明中明确指定变量类型：

```
var foo float32 = 3.0
```

或者，必须为常量赋予一个转换类型，如 foo := float32(3.0)。

### How do I know whether a variable is allocated on the heap or the stack?

从正确性的角度来看，您不需要知道。只要有对它的引用，Go 中的每个变量就存在。实现选择的存储位置与语言的语义无关。

存储位置确实对编写高效程序有影响。如果可能，Go 编译器将在该函数的堆栈帧中分配该函数的本地变量。
但是，如果编译器在函数返回后无法证明该变量未被引用，那么编译器必须在垃圾回收堆上分配该变量以避免悬空指针错误。此外，如果局部变量非常大，将其存储在堆上而不是堆栈上可能更有意义。

在当前的编译器中，如果一个变量的地址被占用，那么该变量就是在堆上分配的候选者。但是，基本的逃逸分析会识别某些情况，即此类变量不会超过函数的返回值并且可以驻留在堆栈中。

### Why does my Go process use so much virtual memory?

Go 内存分配器保留了一个大的虚拟内存区域作为分配的竞技场。这个虚拟内存对于特定的 Go 进程是本地的；保留不会剥夺其他进程的内存。

要查找分配给 Go 进程的实际内存量，请使用 Unix top 命令并查阅 RES (Linux) 或 RSIZE (macOS) 列。

## Concurrency

### What operations are atomic? What about mutexes?

Go 中操作的原子性描述可以在 [Go Memory Model](https://go.dev/ref/mem) 文档中找到。

低级同步和原子原语在 [sync](https://go.dev/pkg/sync) 和 [sync/atomic](https://go.dev/pkg/sync/atomic) 中可用。
这些包适用于简单的任务，例如增加引用计数或保证小规模互斥。

对于更高级别的操作，例如并发服务器之间的协调，更高级别的技术可以带来更好的程序，Go 通过其 goroutine 和通道支持这种方法。例如，您可以构建您的程序，以便一次只有一个 goroutine 负责特定的数据。
原始的 [Go 谚语](https://www.youtube.com/watch?v=PAAkCSZUG1c) 总结了这种方法，

不要通过共享内存进行通信。相反，通过通信共享内存。

有关此概念的详细讨论，请参阅 [Share Memory By Communicating](https://go.dev/doc/codewalk/sharemem/)
代码及[相关文章](https://blog.golang.org/2010/07/share-memory-by-communicating.html)。

大型并发程序可能会从这两个工具包中借用。

### Why doesn't my program run faster with more CPUs?

一个程序是否在更多 CPU 上运行得更快取决于它要解决的问题。Go 语言提供并发原语，例如 goroutine 和通道，但并发仅在底层问题本质上是并行时才启用并行性。本质上是顺序的问题不能通过添加更多的 CPU
来加速，而那些可以分解成可以并行执行的部分的问题可以加速，有时甚至是显着的。

有时添加更多 CPU 会减慢程序速度。实际上，花费更多时间进行同步或通信而不是进行有用计算的程序在使用多个操作系统线程时可能会遇到性能下降。这是因为在线程之间传递数据涉及切换上下文，这会产生巨大的成本，而且随着 CPU
的增加，成本会增加。例如，[Go 规范](https://go.dev/ref/spec#An_example_package) 中的素筛示例虽然启动了许多
goroutine，但并没有显着的并行性；增加线程（CPU）的数量更有可能减慢速度而不是加快速度。

有关此主题的更多详细信息，请参阅题为 [Concurrency is not Parallelism](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)
的演讲。

### How can I control the number of CPUs?

可同时执行 goroutine 的 CPU 数量由 GOMAXPROCS shell 环境变量控制，其默认值是可用的 CPU 内核数。因此，具有并行执行潜力的程序应该默认在多 CPU 机器上实现它。要更改要使用的并行 CPU
数量，请设置环境变量或使用运行时包的类似名称函数来配置运行时支持以利用不同数量的线程。将其设置为 1 消除了真正并行的可能性，迫使独立的 goroutine 轮流执行。

运行时可以分配比 GOMAXPROCS 的值更多的线程来服务多个未完成的 I/O 请求。GOMAXPROCS 只影响一次实际可以执行多少个 goroutine；在系统调用中可能会阻塞任意更多。

Go 的 goroutine 调度器并没有它需要的那么好，尽管随着时间的推移它已经改进了。未来，它可能会更好地优化其对 OS 线程的使用。目前，如果存在性能问题，基于每个应用程序设置 GOMAXPROCS 可能会有所帮助。

### Why is there no goroutine ID?

Goroutines 没有名字；他们只是匿名的工人。它们不向程序员公开唯一标识符、名称或数据结构。有些人对此感到惊讶，期望 go 语句返回一些可用于稍后访问和控制 goroutine 的项。

goroutines 是匿名的根本原因是在编写并发代码时可以使用完整的 Go 语言。相比之下，在命名线程和 goroutine 时开发的使用模式会限制使用它们的库可以做什么。

这是困难的例证。一旦命名了一个 goroutine 并围绕它构建了一个模型，它就会变得特别，并且人们很想将所有计算与该 goroutine 相关联，而忽略了使用多个可能共享的 goroutine 进行处理的可能性。如果 net/http
包将 per-request 状态与 goroutine 相关联，则客户端在处理请求时将无法使用更多 goroutine。

此外，诸如图形系统库之类的库的经验表明，这些库要求所有处理都发生在“主线程”上，这表明当以并发语言部署时，这种方法是多么尴尬和限制。特殊线程或 goroutine
的存在迫使程序员扭曲程序以避免因无意中在错误线程上操作而导致的崩溃和其他问题。

对于特定 goroutine 真正特殊的情况，该语言提供了诸如通道之类的功能，可以以灵活的方式与之交互。

## Functions and Methods

### Why do T and *T have different method sets?

正如 Go 规范所说，一个类型 T 的方法集由所有接收者类型为 T 的方法组成，而对应指针类型 *T 的方法集由所有接收者为 *T 或 T 的方法组成。这意味着 *T 的方法集 包括 T 的，但不包括相反的。

之所以出现这种区别，是因为如果接口值包含指针 *T，则方法调用可以通过取消引用指针来获取值，但如果接口值包含值 T，则方法调用没有安全的方法来获取指针。（这样做将允许方法修改接口内的值的内容，这是语言规范所不允许的。）

即使在编译器可以将值的地址传递给方法的情况下，如果方法修改了值，这些更改也会在调用者中丢失。例如，如果 bytes.Buffer 的 Write 方法使用值接收器而不是指针，则此代码：

```
var buf bytes.Buffer
io.Copy(buf, os.Stdin)
```

会将标准输入复制到 buf 的副本中，而不是 buf 本身。这几乎不是所期望的行为。

### What happens with closures running as goroutines?

并发使用闭包时可能会出现一些混淆。考虑以下程序：

```
func main() {
    done := make(chan bool)

    values := []string{"a", "b", "c"}
    for _, v := range values {
        go func() {
            fmt.Println(v)
            done <- true
        }()
    }

    // wait for all goroutines to complete before exiting
    for _ = range values {
        <-done
    }
}
```

人们可能会错误地期望看到 a、b、c 作为输出。相反，您可能会看到 c、c、c。这是因为循环的每次迭代都使用变量 v 的相同实例，因此每个闭包共享该单个变量。当闭包运行时，它会在执行 fmt.Println 时打印 v 的值，但 v 可能在
goroutine 启动后已被修改。为了帮助在这些问题和其他问题发生之前发现它们，运行 go vet。

要在启动时将 v 的当前值绑定到每个闭包，必须修改内部循环以在每次迭代时创建一个新变量。一种方法是将变量作为参数传递给闭包：

```
    for _, v := range values {
        go func(u string) {
            fmt.Println(u)
            done <- true
        }(v)
    }
```

在此示例中， v 的值作为参数传递给匿名函数。然后该值可以在函数内部作为变量 u 访问。

更简单的是创建一个新变量，使用一种可能看起来很奇怪但在 Go 中可以正常工作的声明风格：

```
    for _, v := range values {
        v := v // create a new 'v'.
        go func() {
            fmt.Println(v)
            done <- true
        }()
    }
```

语言的这种行为，没有为每次迭代定义一个新变量，回想起来可能是一个错误。它可能会在以后的版本中得到解决，但为了兼容性，在 Go 版本 1 中不能更改。

## Control flow

### Why does Go not have the ?: operator?

Go 中没有三元运算符。您可以使用以下方法来实现相同的结果：

```
if expr {
    n = trueVal
} else {
    n = falseVal
}
```

Go 没有出现 ?: 的原因是该语言的设计者已经看到该操作过于频繁地用于创建难以理解的复杂表达式。If-else 虽然更长，但无疑更清晰。一种语言只需要一个条件控制流结构。

## Packages and Testing

### How do I create a multifile package?

将包的所有源文件单独放在一个目录中。源文件可以随意引用不同文件中的项目；不需要前向声明或头文件。

除了被拆分成多个文件之外，该包将像单个文件包一样编译和测试。

### How do I write a unit test?

在与您的包源相同的目录中创建一个以 _test.go 结尾的新文件。在该文件中，导入 "testing" 并编写表单的函数

```
func TestFoo(t *testing.T) { 
    ... 
}
```

在该目录中运行 go test。该脚本找到测试函数，构建测试二进制文件并运行它。

有关更多详细信息，请参阅[How to Write Go Code](https://go.dev/doc/code.html)、package testing 和 go test 子命令。

### Where is my favorite helper function for testing?

Go 的标准测试包使编写单元测试变得容易，但它缺乏其他语言测试框架提供的功能，例如断言函数。本文档的[前面部分](https://go.dev/doc/faq#assertions)解释了为什么 Go
没有断言，并且相同的论点适用于在测试中使用断言。正确的错误处理意味着在一个测试失败后让其他测试运行，这样调试失败的人就可以全面了解问题所在。报告 isPrime 对 2、3、5 和 7（或 2、4、8 和 16）给出错误答案的测试比报告
isPrime 对 2 给出错误答案因此没有更有用的测试进行了更多测试。触发测试失败的程序员可能不熟悉失败的代码。现在，当测试中断时，花费时间编写一个好的错误消息会得到回报。

与此相关的一点是，测试框架倾向于发展成自己的迷你语言，具有条件、控制和打印机制，但 Go 已经具备所有这些功能；为什么要重新创建它们？我们宁愿用 Go 编写测试；学习的语言少了一种，而且这种方法使测试简单易懂。

如果编写好的错误所需的额外代码量似乎是重复性和压倒性的，那么如果表驱动的测试可能会更好，迭代数据结构中定义的输入和输出列表（Go 对数据结构文字有很好的支持）。编写好的测试和好的错误消息的工作将分摊到许多测试用例中。标准 Go
库充满了说明性示例，例如 [fmt 包的格式测试](https://go.dev/src/fmt/fmt_test.go)。

### Why isn't X in the standard library?

标准库的目的是支持运行时，连接到操作系统，并提供许多 Go 程序所需的关键功能，例如格式化 I/O 和网络。它还包含对 Web 编程很重要的元素，包括加密和对 HTTP、JSON 和 XML 等标准的支持。

没有明确的标准来定义包含的内容，因为长期以来，这是唯一的 Go 库。但是，有一些标准可以定义今天添加的内容。

标准库的新增内容很少见，并且包含的门槛很高。包含在标准库中的代码需要承担大量的持续维护成本（通常由原作者以外的人承担），受 [Go 1 兼容性承诺](https://go.dev/doc/go1compat.html)（阻止对 API
中的任何缺陷的修复）的约束，并且受 Go 版本的约束计划，防止错误修复快速提供给用户。

大多数新代码应该存在于标准库之外，并且可以通过 [go 工具](https://go.dev/cmd/go/)的 go get
命令访问。这样的代码可以有自己的维护者、发布周期和兼容性保证。用户可以在 [godoc.org](https://godoc.org/) 上找到包并阅读他们的文档。

尽管标准库中有一些并不真正属于的部分，例如 log/syslog，但由于 Go 1 兼容性承诺，我们继续维护库中的所有内容。但是我们鼓励大多数新代码存在于其他地方。

## Implementation

### What compiler technology is used to build the compilers?

Go 有几个生产编译器，还有一些正在为各种平台开发的其他编译器。

默认编译器 gc 包含在 Go 发行版中，作为对 go 命令的支持的一部分。Gc 最初是用 C 编写的，因为引导很困难——你需要一个 Go 编译器来设置一个 Go 环境。但是事情已经发展了，自从 Go 1.5 发布以来，编译器一直是一个
Go 程序。编译器使用自动翻译工具从 C 转换为
Go，如[设计文档](https://go.dev/s/go13compiler)和[谈话](https://go.dev/talks/2015/gogo.slide#1)中所述。因此编译器现在是 "self-hosting"
的，这意味着我们需要面对引导问题。解决方案是已经有一个工作的 Go 安装，就像一个正常工作的 C
安装一样。[此处](https://go.dev/s/go15bootstrap)和[此处](https://go.dev/doc/install/source)描述了如何从源代码构建新 Go 环境的故事。

Gc 是用 Go 编写的，带有递归下降解析器，并使用自定义加载器（也是用 Go 编写但基于 Plan 9 加载器）来生成 ELF/Mach-O/PE 二进制文件。

在项目开始时，我们考虑使用 LLVM 进行 gc，但认为它太大且速度太慢，无法满足我们的性能目标。回想起来更重要的是，从 LLVM 开始会使引入一些 ABI 和相关更改（例如堆栈管理）变得更加困难，这是 Go 需要但不是标准 C
设置的一部分。然而，一个新的 [LLVM 实现](https://go.googlesource.com/gollvm/)现在开始融合在一起。

Gccgo 编译器是一个用 C++ 编写的前端，带有一个与标准 GCC 后端耦合的递归下降解析器。

Go 被证明是一种很好的语言，可以用来实现 Go 编译器，尽管这不是它的最初目标。从一开始就不是自托管的，这使得 Go 的设计能够专注于其最初的用例，即联网服务器。如果我们决定 Go
应该尽早自行编译，我们最终可能会得到一种更针对编译器构建的语言，这是一个有价值的目标，但不是我们最初的目标。

尽管 gc 不使用它们（还没有？），但 go 包中提供了本机词法分析器和解析器，并且还有一个本机类型检查器。

### How is the run-time support implemented?

同样由于引导问题，运行时代码最初主要是用 C 编写的（带有一点汇编程序），但后来被翻译成 Go（除了一些汇编程序位）。Gccgo 的运行时支持使用 glibc。gccgo 编译器使用称为分段堆栈的技术实现
goroutine，最近对黄金链接器的修改支持。Gollvm 同样建立在相应的 LLVM 基础设施之上。

### Why is my trivial program such a large binary?

gc 工具链中的链接器默认创建静态链接的二进制文件。因此，所有 Go 二进制文件都包含 Go 运行时，以及支持动态类型检查、反射甚至恐慌时间堆栈跟踪所需的运行时类型信息。

在 Linux 上使用 gcc 静态编译和链接的简单 C“hello, world”程序大约 750 kB，包括 printf 的实现。使用 fmt.Printf 的等效 Go
程序重达几兆字节，但其中包括更强大的运行时支持以及类型和调试信息。

使用 gc 编译的 Go 程序可以与 -ldflags=-w 标志链接以禁用 DWARF 生成，从二进制文件中删除调试信息，但不会丢失其他功能。这可以大大减少二进制大小。

### Can I stop these complaints about my unused variable/import?

未使用的变量的存在可能表明存在错误，而未使用的导入只会减慢编译速度，随着时间的推移，随着程序积累代码和程序员，这种影响会变得很大。由于这些原因，Go 拒绝使用未使用的变量或导入来编译程序，以短期的便利性换取长期的构建速度和程序的清晰度。

尽管如此，在开发代码时，临时创建这些情况是很常见的，并且在程序编译之前必须将它们编辑出来可能会很烦人。

有些人要求提供编译器选项来关闭这些检查或至少将它们减少为警告。但是，没有添加这样的选项，因为编译器选项不应该影响语言的语义，并且因为 Go 编译器不报告警告，只报告阻止编译的错误。

没有警告有两个原因。首先，如果值得抱怨，那就值得在代码中修复。（如果它不值得修复，那就不值得一提。）其次，让编译器生成警告会鼓励实现警告可能使编译嘈杂的弱案例，掩盖应该修复的真正错误。

不过，解决这种情况很容易。在开发过程中，使用空白标识符让未使用的东西保持不变。

```
import "unused"

// This declaration marks the import as used by referencing an
// item from the package.
var _ = unused.Item  // TODO: Delete before committing!

func main() {
    debugData := debug.Profile()
    _ = debugData // Used only during debugging.
    ....
}
```

现在，大多数 Go 程序员使用 [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) 工具，它会自动重写 Go
源文件以具有正确的导入，从而在实践中消除未使用的导入问题。这个程序很容易连接到大多数编辑器，以便在编写 Go 源文件时自动运行。

### Why does my virus-scanning software think my Go distribution or compiled binary is infected?

这是一种常见的情况，尤其是在 Windows 机器上，并且几乎总是误报。商业病毒扫描程序经常被 Go 二进制文件的结构所迷惑，它们不像从其他语言编译的那样经常看到这种结构。

如果您刚刚安装了 Go 发行版并且系统报告它已被感染，那肯定是一个错误。要真正彻底，您可以通过将校验和与[下载页面](https://go.dev/dl/)上的校验和进行比较来验证下载。

在任何情况下，如果您认为报告有误，请向您的病毒扫描程序供应商报告错误。也许病毒扫描程序可以及时学会理解 Go 程序。

## Performance

### Why does Go perform badly on benchmark X?

Go 的设计目标之一是在可比较的程序中接近 C
的性能，但在某些基准测试中它的表现相当差，包括 [golang.org/x/exp/shootout](https://go.googlesource.com/exp/+/master/shootout/) 中的几个。最慢的取决于在 Go
中没有可比性能版本的库。例如，[pidigits.go](https://go.googlesource.com/exp/+/master/shootout/pidigits.go) 依赖于多精度数学包，而 C 版本与 Go 不同，使用
[GMP](https://gmplib.org/)（用优化的汇编程序编写）。依赖于正则表达式的基准（例如 [regex-dna.go](https://go.googlesource.com/exp/+/master/shootout/regex-dna.go)）本质上是将
Go 的本机 [regexp 包](https://go.dev/pkg/regexp) 与成熟的、高度优化的正则表达式库（如 PCRE）进行比较。

基准游戏是通过广泛的调整赢得的，大多数基准的围棋版本需要注意。如果你测量可比较的 C 和 Go
程序（[reverse-complement.go](https://go.googlesource.com/exp/+/master/shootout/reverse-complement.go)
就是一个例子），你会发现这两种语言在原始性能上比这个套件所显示的要接近得多。

尽管如此，仍有改进的余地。编译器很好，但可能会更好，许多库需要主要的性能工作，而垃圾收集器还不够快。（即使是这样，注意不要产生不必要的垃圾也会产生巨大的影响。）

无论如何，Go
通常很有竞争力。随着语言和工具的发展，许多程序的性能有了显着提高。请参阅有关[分析 Go 程序](https://blog.golang.org/2011/06/profiling-go-programs.html)的博客文章以获取信息示例。

## Changes from C

### Why is the syntax so different from C?

除了声明语法之外，差异并不大，并且源于两个期望。首先，语法应该感觉轻松，没有太多强制性关键字、重复或奥秘。其次，该语言被设计成易于分析并且可以在没有符号表的情况下进行解析。这使得构建调试器、依赖分析器、自动文档提取器、IDE
插件等工具变得更加容易。C 及其后代在这方面是出了名的困难。

### Why are declarations backwards?

如果你习惯了 C，它们只会倒退。在 C 中，概念是变量被声明为表示其类型的表达式，这是一个好主意，但类型和表达式语法不能很好地混合，并且 结果可能令人困惑；考虑函数指针。Go 主要将表达式和类型语法分开并且简化了事情（使用前缀 *
作为指针是证明规则的例外）。在 C 中，声明

```
    int* a, b;
```

将 a 声明为指针但不是 b；在 Go 中

```
    var a, b *int
```

声明两者都是指针。这更清晰，更规律。此外， := 短声明形式认为完整的变量声明应该呈现与 := 相同的顺序

```
    var a uint64 = 1
```

具有相同的效果

```
    a := uint64(1)
```

解析也被简化了，因为它有一个独特的类型语法，而不仅仅是表达式语法；诸如 func 和 chan 之类的关键字使事情变得清晰。

有关更多详细信息，请参阅有关 [Go 的声明语法](https://go.dev/doc/articles/gos_declaration_syntax.html)的文章。

### Why is there no pointer arithmetic?

安全。如果没有指针算法术，就有可能创建一种永远不会得出错误成功的非法地址的语言。编译器和硬件技术已经发展到使用数组索引的循环可以与使用指针算法的循环一样高效的程度。此外，缺少指针算法可以简化垃圾收集器的实现。

### Why are ++ and -- statements and not expressions? And why postfix, not prefix?

如果没有指针算术，前置和后置增量运算符的便利值就会下降。通过将它们从表达式层次结构中完全删除，表达式语法得到了简化，并且围绕 ++ 和 --（考虑 f(i++) 和 p[i] = q[++i]）的求值顺序的混乱问题也被消除了 .
简化意义重大。至于后缀与前缀，两者都可以正常工作，但后缀版本更传统；对前缀的坚持源于 STL，这是一个语言库，其名称讽刺地包含后缀增量。

### Why are there braces but no semicolons? And why can't I put the opening brace on the next line?

Go 使用大括号进行语句分组，这是使用过 C 家族中任何语言的程序员所熟悉的语法。然而，分号是用于解析器的，而不是用于人的，我们希望尽可能地消除它们。为了实现这个目标，Go 借鉴了 BCPL
的一个技巧：分隔语句的分号在形式语法中，但在可能是语句结尾的任何行的末尾由词法分析器自动注入，无需前瞻。这在实践中非常有效，但它的效果是强制使用大括号样式。例如，函数的左大括号不能单独出现在一行上。

一些人认为词法分析器应该进行前瞻以允许大括号存在于下一行。我们不同意。由于 Go 代码旨在由 gofmt 自动格式化，因此必须选择某种样式。这种风格可能与您在 C 或 Java 中使用的不同，但 Go 是一种不同的语言，并且 gofmt
的风格与其他任何语言一样好。更重要的是——更重要的是——所有 Go 程序的单一、程序化强制格式的优点远远超过了特定风格的任何可察觉的缺点。还要注意 Go 的风格意味着 Go 的交互式实现可以一次使用一行标准语法，而无需特殊规则。

### Why do garbage collection? Won't it be too expensive?

系统程序中簿记的最大来源之一是管理已分配对象的生命周期。在诸如 C 之类的手动完成的语言中，它会消耗大量的程序员时间，并且通常是有害错误的原因。即使在提供辅助机制的 C++ 或 Rust
等语言中，这些机制也会对软件的设计产生重大影响，通常会增加其自身的编程开销。我们认为消除此类程序员开销至关重要，而过去几年垃圾收集技术的进步使我们相信它可以足够便宜地实现，并且延迟足够低，它可能是网络系统的一种可行方法。

并发编程的大部分困难都源于对象生命周期问题：随着对象在线程之间传递，保证它们被安全释放变得很麻烦。自动垃圾收集使并发代码更容易编写。当然，在并发环境中实现垃圾收集本身就是一个挑战，但是一次而不是在每个程序中满足它对每个人都有帮助。

最后，除了并发之外，垃圾收集使接口更简单，因为它们不需要指定如何跨接口管理内存。

这并不是说最近在诸如 Rust 之类的语言中为资源管理问题带来新思想的工作是错误的；我们鼓励这项工作，并很高兴看到它如何发展。但是 Go 采用更传统的方法，通过垃圾收集和单独的垃圾收集来处理对象生命周期。

当前的实现是一个标记和清除收集器。如果机器是多处理器，则收集器与主程序并行运行在单独的 CPU
内核上。近年来收集器的主要工作已将暂停时间通常减少到亚毫秒范围，即使对于大型堆也是如此，几乎消除了网络服务器中垃圾收集的主要反对意见之一。工作继续改进算法，进一步减少开销和延迟，并探索新的方法。Go 团队的 Rick Hudson 在
2018 年 [ISMM 主题演讲](https://blog.golang.org/ismmkeynote) 中描述了迄今为止的进展，并提出了一些未来的方法。

关于性能的话题，请记住，Go 为程序员提供了对内存布局和分配的相当大的控制权，这比垃圾收集语言中的典型控制要多得多。一个细心的程序员可以通过很好地使用语言来显着减少垃圾收集的开销；请参阅有关分析 Go
程序的文章以获取工作示例，包括 [Go 分析工具](https://blog.golang.org/2011/06/profiling-go-programs.html) 的演示。