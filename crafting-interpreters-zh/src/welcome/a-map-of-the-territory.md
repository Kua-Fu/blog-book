# 总览图

## A Map of the Territory

> you must have a map, no matter how rough. Otherwise you wander all over the place. In The Lord of the Rings I never made anyone go farther than he could on a given day.
>
> <p align="right">—— J.R.R. Tolkien</p>

领土图

>不管有多么粗糙，你都必须拥有一幅领土地图，否则你将到处游荡。在《指环王》中，我从来都没有让任何人一天内走的更远。
>
> <p align="right">—— J.R.R. Tolkien</p>

We don’t want to wander all over the place, so before we set off, let’s scan the territory charted by previous language implementers. It will help us understand where we are going and the alternate routes others have taken.

First, let me establish a shorthand.  Much of this book is about a language’s implementation, which is distinct from the language itself in some sort of Platonic ideal form. Things like “stack”, “bytecode”, and “recursive descent”, are nuts and bolts one particular implementation might use. From the user’s perspective, as long as the resulting contraption faithfully follows the language’s specification, it’s all implementation detail.

We’re going to spend a lot of time on those details, so if I have to write “language implementation” every single time I mention them, I’ll wear my fingers off. Instead, I’ll use “language” to refer to either a language or an implementation of it, or both, unless the distinction matters.

我们不想到处瞎逛，所以在出发之前，我们先浏览一下以前的语言实现者绘制的领土图。这将帮助我们理解我们的目标，了解更多的替代路径。

首先，让我们建立一个概览。本书的大部分内容都是如何实现一门语言，这和一门语言本身柏拉图式的理想概念有所不同。 像是栈、字节码、递归下降等东西，是一个特定实现可能会用到的具体细节。 从用户的角度，只要生成的内容还遵循着语言的规范，它就是所有的实现细节。

我们将在这些细节上花费大量时间，因此，如果每次提到这些细节，我都要加上语言实现说明，那么我会累晕的。所以，我将使用语言来表示一门语言或者这门语言的实现，或者两者，除非两者的区别非常重要。


## 一、The Parts of a Language

语言的组成部分

Engineers have been building programming languages since the Dark Ages of computing. As soon as we could talk to computers, we discovered doing so was too hard, and we enlisted their help. I find it fascinating that even though today’s machines are literally a million times faster and have orders of magnitude more storage, the way we build programming languages is virtually unchanged.

Though the area explored by language designers is vast, the trails they’ve carved through it are few. Not every language takes the exact same path—some take a shortcut or two—but otherwise they are reassuringly similar, from Rear Admiral Grace Hopper’s first COBOL compiler all the way to some hot, new, transpile-to-JavaScript language whose “documentation” consists entirely of a single, poorly edited README in a Git repository somewhere.

自计算的黑暗时代以来，工程师们一直在构建编程语言。当我们可以与电脑交流时候，我们发现这样做太难了，需要电脑的帮助。我发现一个有趣的现象，即使今天的机器运行速度快了数百万倍，存储量也增加了几个数量级，但是我们构建编程语言的方式几乎没有任何改变。

虽然，语言设计者探索的领域非常大，但是他们在其中开辟的道路却非常少。并不是所有的语言都走相同的路径，有些语言的实现，会走一、两条捷径。但是从另一个角度来看，它们都是相似的。从第一个 COBOL编译器到现在最新的可以转换为 JavaScript的语言，在它们 git仓库README文件中的描述都是相似的。

>There are certainly dead ends, sad little cul-de-sacs of CS papers with zero citations and now-forgotten optimizations that only made sense when memory was measured in individual bytes.
>
>毫无疑问，计算机科学论文存在一些死胡同。这些论文现在已经没有人引用，都是在内存需要以一个一个字节来衡量时期的优化使用论文。

I visualize the network of paths an implementation may choose as climbing a mountain. You start off at the bottom with the program as raw source text, literally just a string of characters. Each phase analyzes the program and transforms it to some higher-level representation where the semantics—what the author wants the computer to do—become more apparent.

![a map of the territory](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/mountain.png?raw=true)

Eventually we reach the peak. We have a bird’s-eye view of the user’s program and can see what their code means. We begin our descent down the other side of the mountain. We transform this highest-level representation down to successively lower-level forms to get closer and closer to something we know how to make the CPU actually execute.

我把编译领域图，想象为一幅包含很多路径的爬山图。从底部开始，一开始只是一个文本，实际上只是一个字符串。经过，每个分析阶段，都会生成更加高级的表示，设计者希望计算机执行的语言都更加明确。

最后，我们爬上了山顶。我们鸟瞰全局，可以得到使用者编写的代码含义。我们从山的另一边开始下山，我们将连续将高级别的表示转换为更低级别的表示，以越来越接近计算机 CPU执行的语言。

Let’s trace through each of those trails and points of interest. Our journey begins on the left with the bare text of the user’s source code:

![string](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/string.png?raw=true)

接下来，我们将追踪每一条路径、每一个停留点，我们的旅途从左边山脚开始（用户源代码）。

### 1.1 Scanning

扫描

The first step is scanning, also known as lexing, or (if you’re trying to impress someone) lexical analysis. They all mean pretty much the same thing. I like “lexing” because it sounds like something an evil supervillain would do, but I’ll use “scanning” because it seems to be marginally more commonplace.

A scanner (or lexer) takes in the linear stream of characters and chunks them together into a series of something more akin to “words”. In programming languages, each of these words is called a token. Some tokens are single characters, like ( and ,. Others may be several characters long, like numbers (123), string literals ("hi!"), and identifiers (min).

Some characters in a source file don’t actually mean anything. Whitespace is often insignificant, and comments, by definition, are ignored by the language. The scanner usually discards these, leaving a clean sequence of meaningful tokens.

![tokens](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/tokens.png?raw=true)

第一步是扫描，也称为词法，如果你想给别人留下深刻印象，还可以称为词法分析。他们的含义都差不多，我更喜欢词法，因为这听起来像是一个恶作剧，但是接下来我会使用扫描表示这个过程，因为这种说法更加常见。

一个扫描器，接收线性的字符串，将它们分块为一个个单词，在编程语言中，分成的单词称为 token, 一些 token 是单字符，例如: `(` `,` 还有一些token长度是多个字符，例如: 数字 `123` ，字符串 `"hi!"` , 标识符 `min`

源文件中的某些字符没有实际意义。空白字符，通常没有实际意义，还有注释，根据语言定义，注释会被语言忽略。扫描器通常会忽略这些内容，最终生成一个干净的有意义的token 序列。

### 1.2 Parsing

解析

The next step is parsing. This is where our syntax gets a grammar—the ability to compose larger expressions and statements out of smaller parts. Did you ever diagram sentences in English class? If so, you’ve done what a parser does, except that English has thousands and thousands of “keywords” and an overflowing cornucopia of ambiguity. Programming languages are much simpler.

A parser takes the flat sequence of tokens and builds a tree structure that mirrors the nested nature of the grammar. These trees have a couple of different names—parse tree or abstract syntax tree—depending on how close to the bare syntactic structure of the source language they are.  In practice, language hackers usually call them syntax trees, ASTs, or often just trees.

下一步是解析，这就是我们获得语法的地方，语法可以将较小的部分组合成较大的表达式和语句。你在英语课堂上画过句子图吗？如果是这样的话，你已经完成了解析器的工作。除了英语有成千上万个关键词和更多的歧义。相较而言，编程语言就简单太多了。

解析器接收token 序列，然后构建出反应语法嵌套性质的树结构。这些树有一些不同的名称，例如：名称解析树，抽象语法树，命名取决于这些树和源语言的简单语法结构的接近程度。在实践中，语言高手经常称它们为语法树，AST或者通常就称为树。

