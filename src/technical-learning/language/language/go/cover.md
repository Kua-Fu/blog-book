# The cover story

Rob Pike 2 December 2013


## Introduction

From the beginning of the project, Go was designed with tools in mind. Those tools include some of the most iconic pieces of Go technology such as the documentation presentation tool godoc, the code formatting tool gofmt, and the API rewriter gofix. Perhaps most important of all is the go command, the program that automatically installs, builds, and tests Go programs using nothing more than the source code as the build specification.

The release of Go 1.2 introduces a new tool for test coverage that takes an unusual approach to the way it generates coverage statistics, an approach that builds on the technology laid down by godoc and friends.



从项目开始，Go 就是以工具为设计目标的。这些工具包括一些最具标志性的 Go 技术，例如文档演示工 godoc、代码格式化工具 gofmt 和 API 重写工具 gofix。也许最重要的是 go 命令，这个程序可以使用源作为构建规范，自动安装、构建和测试 Go 程序。

Go 1.2 的发布引入了一种新的测试覆盖具，它采用了一种不同寻常的方法来生成覆盖率统计数据，这种方法建立在 godoc 和其他工具所奠定的技术基础之上。

## Support for tools

First, some background: What does it mean for a language to support good tooling? It means that the language makes it easy to write good tools and that its ecosystem supports the construction of tools of all flavors.

There are a number of properties of Go that make it suitable for tooling. For starters, Go has a regular syntax that is easy to parse. The grammar aims to be free of special cases that require complex machinery to analyze.

Where possible, Go uses lexical and syntactic constructs to make semantic properties easy to understand. Examples include the use of upper-case letters to define exported names and the radically simplified scoping rules compared to other languages in the C tradition.

Finally, the standard library comes with production-quality packages to lex and parse Go source code. They also include, more unusually, a production-quality package to pretty-print Go syntax trees.

These packages in combination form the core of the gofmt tool, but the pretty-printer is worth singling out. Because it can take an arbitrary Go syntax tree and output standard-format, human-readable, correct code, it creates the possibility to build tools that transform the parse tree and output modified but correct and easy-to-read code.

One example is the gofix tool, which automates the rewriting of code to use new language features or updated libraries. Gofix let us make fundamental changes to the language and libraries in the run-up to Go 1.0, with the confidence that users could just run the tool to update their source to the newest version.

Inside Google, we have used gofix to make sweeping changes in a huge code repository that would be almost unthinkable in the other languages we use. There’s no need any more to support multiple versions of some API; we can use gofix to update the entire company in one operation.

It’s not just these big tools that these packages enable, of course. They also make it easy to write more modest programs such as IDE plugins, for instance. All these items build on each other, making the Go environment more productive by automating many tasks.



首先，一些背景知识：什么是一种语言支持良好的工具？这意味着这种语言易于编写优秀的工具，而且其生态系统支持各种口味的工具的构建。

Go 语言有许多特性使其适用于工具开发。首先，Go 语言具有规则的语法，易于解析。语法旨在避免需要复杂机制来分析的特殊情况。

在可能的情况下，Go 使用词汇和句法结构使语义属性易于理解。例如，使用大写字母定义导出名称，以及与 C 语言传统中的其他语言相比，大大简化的作用域规则。

最后，标准库附带了用于词法分析和解析 Go 源代码的生产质量包。更不寻常的是，它们还包括一个用于美化打印 Go 语法树的生产质量包。

这些包结合在一起构成了 gofmt 工具的核心，但美化打印机值得一提。因为它可以接受任意 Go 语法树并输出标准格式、易读、正确的代码，所以它为构建可以转换解析树并输出修改后但正确且易于阅读的代码的工具创造了可能性。

一个例子是 gofix 工具，它可以自动重写代码以使用新的语言特性或更新的库。在 Go 1.0 发布之前，Gofix 让我们能够对语言和库进行根本性的更改，并确信用户只需运行该工具即可将其源代码更新为最新版本。

在谷歌内部，我们使用 gofix 对庞大的代码库进行大规模更改，这在我们使用的其他语言中几乎是不可想象的。不再需要支持某个 API 的多个版本；我们可以使用 gofix 一次性更新整个公司。

当然，这些包不仅仅是为了支持这些大型工具。它们还使编写更为简单的程序变得容易，例如集成开发环境插件。所有这些项目相互依赖，通过自动执行许多任务使 Go 环境更具生产力。

## Test coverage


Test coverage is a term that describes how much of a package’s code is exercised by running the package’s tests. If executing the test suite causes 80% of the package’s source statements to be run, we say that the test coverage is 80%.

The program that provides test coverage in Go 1.2 is the latest to exploit the tooling support in the Go ecosystem.

The usual way to compute test coverage is to instrument the binary. For instance, the GNU gcov program sets breakpoints at branches executed by the binary. As each branch executes, the breakpoint is cleared and the target statements of the branch are marked as ‘covered’.

This approach is successful and widely used. An early test coverage tool for Go even worked the same way. But it has problems. It is difficult to implement, as analysis of the execution of binaries is challenging. It also requires a reliable way of tying the execution trace back to the source code, which can also be difficult, as any user of a source-level debugger can attest. Problems there include inaccurate debugging information and issues such as in-lined functions complicating the analysis. Most important, this approach is very non-portable. It needs to be done afresh for every architecture, and to some extent for every operating system since debugging support varies greatly from system to system.

It does work, though, and for instance if you are a user of gccgo, the gcov tool can give you test coverage information. However If you’re a user of gc, the more commonly used Go compiler suite, until Go 1.2 you were out of luck.

测试覆盖率是一个术语，用来描述在运行软件包测试时，有多少软件包代码被执行。如果执行测试用例使得软件包的 80% 源代码语句被运行，我们就说测试覆盖率是 80%。

在 Go 1.2 中提供测试覆盖率的程序是最新的，利用了 Go 生态系统中的工具支持。

计算测试覆盖率的常用方法是对二进制文件进行插桩。例如，GNU gcov 程序在二进制文件执行的分支处设置断点。当每个分支执行时，断点被清除，分支的目标语句被标记为“已覆盖”。

这种方法是成功的并且被广泛使用。Go 的早期测试覆盖率工具甚至也是以相同的方式工作。但它存在问题。实现起来困难，因为分析二进制文件的执行具有挑战性。它还需要一种可靠的方法将执行跟踪回溯到源代码，这也可能很困难，任何源级调试器的用户都可以证实这一点。其中的问题包括不准确的调试信息以及内联函数等问题使分析变得复杂。最重要的是，这种方法非常不便携。它需要为每个架构重新完成，而且在某种程度上需要为每个操作系统重新完成，因为调试支持在不同系统之间差异很大。

尽管如此，它确实有效，例如，如果您是 gccgo 的用户，gcov 工具可以为您提供测试覆盖率信息。然而，如果您是 gc 的用户，这是更常用的 Go 编译器套件，在 Go 1.2 之前，您就没有这种便利。

## Test coverage for Go


For the new test coverage tool for Go, we took a different approach that avoids dynamic debugging. The idea is simple: Rewrite the package’s source code before compilation to add instrumentation, compile and run the modified source, and dump the statistics. The rewriting is easy to arrange because the go command controls the flow from source to test to execution.

Here’s an example. Say we have a simple, one-file package like this:

```go

package size

func Size(a int) string {
    switch {
    case a < 0:
        return "negative"
    case a == 0:
        return "zero"
    case a < 10:
        return "small"
    case a < 100:
        return "big"
    case a < 1000:
        return "huge"
    }
    return "enormous"
}

```

and this test:

```go

package size

import "testing"

type Test struct {
    in  int
    out string
}

var tests = []Test{
    {-1, "negative"},
    {5, "small"},
}

func TestSize(t *testing.T) {
    for i, test := range tests {
        size := Size(test.in)
        if size != test.out {
            t.Errorf("#%d: Size(%d)=%s; want %s", i, test.in, size, test.out)
        }
    }
}

```


To get the test coverage for the package, we run the test with coverage enabled by providing the -cover flag to go test:

```shell

% go test -cover
PASS
coverage: 42.9% of statements
ok      size    0.026s
%
```


Notice that the coverage is 42.9%, which isn’t very good. Before we ask how to raise that number, let’s see how that was computed.

When test coverage is enabled, go test runs the “cover” tool, a separate program included with the distribution, to rewrite the source code before compilation. Here’s what the rewritten Size function looks like:


```go

func Size(a int) string {
    GoCover.Count[0] = 1
    switch {
    case a < 0:
        GoCover.Count[2] = 1
        return "negative"
    case a == 0:
        GoCover.Count[3] = 1
        return "zero"
    case a < 10:
        GoCover.Count[4] = 1
        return "small"
    case a < 100:
        GoCover.Count[5] = 1
        return "big"
    case a < 1000:
        GoCover.Count[6] = 1
        return "huge"
    }
    GoCover.Count[1] = 1
    return "enormous"
}

```

Each executable section of the program is annotated with an assignment statement that, when executed, records that that section ran. The counter is tied to the original source position of the statements it counts through a second read-only data structure that is also generated by the cover tool. When the test run completes, the counters are collected and the percentage is computed by seeing how many were set.

Although that annotating assignment might look expensive, it compiles to a single “move” instruction. Its run-time overhead is therefore modest, adding only about 3% when running a typical (more realistic) test. That makes it reasonable to include test coverage as part of the standard development pipeline.

对于 Go 的新测试覆盖工具，我们采用了一种避免动态调试的不同方法。这个想法很简单：在编译之前重写软件包的源代码以添加插桩，编译并运行修改后的源代码，然后转储统计信息。由于 go 命令控制了从源代码到测试到执行的流程，所以重写很容易安排。

以下是一个示例。假设我们有一个简单的单文件软件包，如下所示：

```go

package size

func Size(a int) string {
    switch {
    case a < 0:
        return "negative"
    case a == 0:
        return "zero"
    case a < 10:
        return "small"
    case a < 100:
        return "big"
    case a < 1000:
        return "huge"
    }
    return "enormous"
}

```

```go

package size

import "testing"

type Test struct {
    in  int
    out string
}

var tests = []Test{
    {-1, "negative"},
    {5, "small"},
}

func TestSize(t *testing.T) {
    for i, test := range tests {
        size := Size(test.in)
        if size != test.out {
            t.Errorf("#%d: Size(%d)=%s; want %s", i, test.in, size, test.out)
        }
    }
}

```


执行测试的结果

```shell


% go test -cover
PASS
coverage: 42.9% of statements
ok      size    0.026s
%

```

请注意，覆盖率为 42.9%，这并不是很好。在我们询问如何提高这个数字之前，让我们看看是如何计算出来的。

当启用测试覆盖率时，`go test` 会运行 "cover" 工具（与发行版一起提供的单独程序）在编译之前重写源代码。以下是重写后的 Size 函数的样子：

```go


func Size(a int) string {
    GoCover.Count[0] = 1
    switch {
    case a < 0:
        GoCover.Count[2] = 1
        return "negative"
    case a == 0:
        GoCover.Count[3] = 1
        return "zero"
    case a < 10:
        GoCover.Count[4] = 1
        return "small"
    case a < 100:
        GoCover.Count[5] = 1
        return "big"
    case a < 1000:
        GoCover.Count[6] = 1
        return "huge"
    }
    GoCover.Count[1] = 1
    return "enormous"
}

```

程序的每个可执行部分都用一个赋值语句进行注释，当执行时，记录该部分已运行。计数器通过第二个只读结构与其计数的语句的原始源位置相关联，该数据结构也由 cover 工具生成。当测试运行完成时，收集计器并通过查看设置了多少个计数器来计算百分比。
尽管这个注释赋值看起来可能很昂贵，但它编译为一个“move”指令。因此，其运行时开销是适中的，在运行典型（更现实）测试时仅增加约 3%。这使得将测试覆盖率作为标准开发流程的一部分变得合理。


## viewing the results



The test coverage for our example was poor. To discover why, we ask go test to write a “coverage profile” for us, a file that holds the collected statistics so we can study them in more detail. That’s easy to do: use the -coverprofile flag to specify a file for the output:

```shell

% go test -coverprofile=coverage.out
PASS
coverage: 42.9% of statements
ok      size    0.030s
%

```

(The -coverprofile flag automatically sets -cover to enable coverage analysis.) The test runs just as before, but the results are saved in a file. To study them, we run the test coverage tool ourselves, without go test. As a start, we can ask for the coverage to be broken down by function, although that’s not going to illuminate much in this case since there’s only one function:


```shell

% go tool cover -func=coverage.out
size.go:    Size          42.9%
total:      (statements)  42.9%
%


```

A much more interesting way to see the data is to get an HTML presentation of the source code decorated with coverage information. This display is invoked by the -html flag:

```shell

$ go tool cover -html=coverage.out

```

When this command is run, a browser window pops up, showing the covered (green), uncovered (red), and uninstrumented (grey) source. Here’s a screen dump:


![cover](https://github.com/Kua-Fu/blog-book-images/blob/main/learn_language/cover.png?raw=true)

With this presentation, it’s obvious what’s wrong: we neglected to test several of the cases! And we can see exactly which ones they are, which makes it easy to improve our test coverage.



我们示例的测试覆盖率很差。为了发现原因，我们要求 go test 为我们编写一个“覆盖率概要文件”，这是一个保存收集到的统计信息的文件，以便我们可以更详细地研究它们。这很容易做到：使用 -coverprofile 标志指定一个输出文件：

```shell

% go test -coverprofile=coverage.out
PASS
coverage: 42.9% of statements
ok      size    0.030s
%

```

（-coverprofile 标志会自动设置 -cover 以启用覆盖率分析。）测试像以前一样运行，但结果保存在一个文件中。为了研究它们，我们自己运行测试覆盖率工具，而不使用 go test。首先，我们可以要求按功能划分覆盖率，尽管在这种情况下，由于只有一个功能，这不会让我们了解更多信息：


```shell


% go tool cover -func=coverage.out
size.go:    Size          42.9%
total:      (statements)  42.9%
%

```

查看数据的一种更有趣的方法是获取带有覆盖率信息的源代码的 HTML 展示。这个显示由 -html 标志调用：

```shell


$ go tool cover -html=coverage.out

```

当运行此命令时，浏览器窗口会弹出，显示已覆盖（绿色）、未覆盖（红色）和未插桩（灰色）的源代码。这是一个屏幕截图：

![cover](https://github.com/Kua-Fu/blog-book-images/blob/main/learn_language/cover.png?raw=true)

通过这种展示，问题显而易见：我们忽略了测试几个案例！而且我们可以清楚地看到它们是哪些，这使得提高测试覆盖率变得容易。

## Heat maps


A big advantage of this source-level approach to test coverage is that it’s easy to instrument the code in different ways. For instance, we can ask not only whether a statement has been executed, but how many times.

The go test command accepts a -covermode flag to set the coverage mode to one of three settings:


* set: did each statement run?

* count: how many times did each statement run?

* atomic: like count, but counts precisely in parallel programs

The default is ‘set’, which we’ve already seen. The atomic setting is needed only when accurate counts are required when running parallel algorithms. It uses atomic operations from the sync/atomic package, which can be quite expensive. For most purposes, though, the count mode works fine and, like the default set mode, is very cheap.

Let’s try counting statement execution for a standard package, the fmt formatting package. We run the test and write out a coverage profile so we can present the information nicely afterwards.

```shell
% go test -covermode=count -coverprofile=count.out fmt
ok      fmt 0.056s  coverage: 91.7% of statements
%

```

That’s a much better test coverage ratio than for our previous example. (The coverage ratio is not affected by the coverage mode.) We can display the function breakdown:

```shell

% go tool cover -func=count.out
fmt/format.go: init              100.0%
fmt/format.go: clearflags        100.0%
fmt/format.go: init              100.0%
fmt/format.go: computePadding     84.6%
fmt/format.go: writePadding      100.0%
fmt/format.go: pad               100.0%
...
fmt/scan.go:   advance            96.2%
fmt/scan.go:   doScanf            96.8%
total:         (statements)       91.7%

```

The big payoff happens in the HTML output:

```shell

% go tool cover -html=count.out

```

Here’s what the pad function looks like in that presentation:

![fmt-cover](https://github.com/Kua-Fu/blog-book-images/blob/main/learn_language/fmt_cover.png?raw=true)

Notice how the intensity of the green changes. Brighter-green statements have higher execution counts; less saturated greens represent lower execution counts. You can even hover the mouse over the statements to see the actual counts pop up in a tool tip. At the time of writing, the counts come out like this (we’ve moved the counts from the tool tips to beginning-of-line markers to make them easier to show):

```go

2933    if !f.widPresent || f.wid == 0 {
2985        f.buf.Write(b)
2985        return
2985    }
  56    padding, left, right := f.computePadding(len(b))
  56    if left > 0 {
  37        f.writePadding(left, padding)
  37    }
  56    f.buf.Write(b)
  56    if right > 0 {
  13        f.writePadding(right, padding)
  13    }
  
```

That’s a lot of information about the execution of the function, information that might be useful in profiling.


热图


这种基于源代码级别的测试覆盖率方法的一个很大优势是，可以轻松地以不同方式对代码进行插桩。例如，我们不仅可以询问一个语句是否已执行，还可以询问执行了多少次。

go test 命令接受一个 -covermode 标志，将覆盖模式设置为以下三个设置之一：

* set：每个语句是否运行？

* count：每个语句运行了多少次？

* atomic：类似于 count，但在并行程序中精确计算

默认值是 'set'，我们已经看到了。当运行并行算法时，只有在需要准确计数时才需要使用 atomic 设置。它使用来自 sync/atomic 包的操作，这可能相当昂贵。然而，对于大多数目的，count 模式工作得很好，就像默认的 set 模式一样，非常便宜。

让我们尝试为标准包（如 fmt 格式化包）计算语句执行次数。我们运行测试并写出覆盖概要文件，以便我们可以在之后很好地呈现信息。

```shell

% go test -covermode=count -coverprofile=count.out fmt
ok      fmt 0.056s  coverage: 91.7% of statements
%

```

这比我们前的示例的测试覆盖率要好得多。（覆盖率比例不受覆盖模式的影响。）我们可以显示功能分解在 输出中，收益最大

```shell

% go tool cover -func=count.out
fmt/format.go: init              100.0%
fmt/format.go: clearflags        100.0%
fmt/format.go: init              100.0%
fmt/format.go: computePadding     84.6%
fmt/format.go: writePadding      100.0%
fmt/format.go: pad               100.0%
...
fmt/scan.go:   advance            96.2%
fmt/scan.go:   doScanf            96.8%
total:         (statements)       91.7%

```

以下是 pad 函数在该演示中的样子：

![fmt-cover](https://github.com/Kua-Fu/blog-book-images/blob/main/learn_language/fmt_cover.png?raw=true)

注意绿色的强度如何变化。更亮的绿色语句具有更高的执行计数；较不饱和的绿色表示较低的执行计数。您甚至可以将鼠标悬停在语句上，看到实际计数在工具提示中弹出。在撰写本文时，计数结果如下（我们已将工具提示中的计数移至行首标记，以便更容易显示）：

```go

2933    if !f.widPresent || f.wid == 0 {
2985        f.buf.Write(b)
2985        return
2985    }
  56    padding, left, right := f.computePadding(len(b))
  56    if left > 0 {
  37        f.writePadding(left, padding)
  37    }
  56    f.buf.Write(b)
  56    if right > 0 {
  13        f.writePadding(right, padding)
  13    }
  
```


关于函数执行的信息非常多，这些信息可能在性能分析中非常有用。

## Basic blocks

You might have noticed that the counts in the previous example were not what you expected on the lines with closing braces. That’s because, as always, test coverage is an inexact science.

What’s going on here is worth explaining, though. We’d like the coverage annotations to be demarcated by branches in the program, the way they are when the binary is instrumented in the traditional method. It’s hard to do that by rewriting the source, though, since the branches don’t appear explicitly in the source.

What the coverage annotation does is instrument blocks, which are typically bounded by brace brackets. Getting this right in general is very hard. A consequence of the algorithm used is that the closing brace looks like it belongs to the block it closes, while the opening brace looks like it belongs outside the block. A more interesting consequence is that in an expression like

```

f() && g()

```


there is no attempt to separately instrument the calls to f and g, Regardless of the facts it will always look like they both ran the same number of times, the number of times f ran.

To be fair, even gcov has trouble here. That tool gets the instrumentation right but the presentation is line-based and can therefore miss some nuances.

基本块

您可能已经注意到，在具有闭合括号的行上，前面示例中的计数不是您预期的计数。这是因为，像往常一样，测试覆盖率是一门不精确的科学。

但是，这里发生了什么值得解释。我们希望覆盖注释由程序中的分支标记，就像在传统方法中对二进制文件进行插装时一样。但是，通过重写源代码很难做到这一点，因为分支在源代码中并不明确出现。

覆盖注释所做的是对块进行插装，这些块通常由大括号界定。通常很难正确地完成这项工作。使用的算法的一个结果是，闭合括号看起来属于它关闭的块，而开放括号看起来属于块外。更有趣的结果是，在像

```

f() && g()

```

这样的表达式中，没有尝试单独插装对f和g的调用，无论事实如何，它们始终看起来都运行了相同次数的f运行次数。

公平地说，使是gcov在这里也有困难。该工具可以正确地进行插装，但是呈现是基于行的，因此可能会错过一些细微差别。

## The big picture

That’s the story about test coverage in Go 1.2. A new tool with an interesting implementation enables not only test coverage statistics, but easy-to-interpret presentations of them and even the possibility to extract profiling information.

Testing is an important part of software development and test coverage a simple way to add discipline to your testing strategy. Go forth, test, and cover.

这是关于Go 1.2中测试覆盖率的故事。一种具有有趣实现的新工具不仅可以提供测试覆盖率统计信息，还可以提供易于解释的演示文稿甚至可以提取分析信息。

测试是软件开发的重要组成部分，而测试覆盖率是为您的测试策略增加纪律的简单方法。继续前进，测试和覆盖。
