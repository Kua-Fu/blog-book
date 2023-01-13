# 解析树

A TREE-WALK INTERPRETER

With this part, we begin jlox, the first of our two interpreters. Programming languages are a huge topic with piles of concepts and terminology to cram into your brain all at once. Programming language theory requires a level of mental rigor that you probably haven’t had to summon since your last calculus final. (Fortunately there isn’t too much theory in this book.)

Implementing an interpreter uses a few architectural tricks and design patterns uncommon in other kinds of applications, so we’ll be getting used to the engineering side of things too. Given all of that, we’ll keep the code we have to write as simple and plain as possible.

In less than two thousand lines of clean Java code, we’ll build a complete interpreter for Lox that implements every single feature of the language, exactly as we’ve specified. The first few chapters work front-to-back through the phases of the interpreter—scanning, parsing, and evaluating code. After that, we add language features one at a time, growing a simple calculator into a full-fledged scripting language.

第二部分，我们将从jLox解释器开始，jLox解释器，是我们将要介绍的两个解释器之一。编程语言是一个非常大的话题，有成堆的概念和术语，将一下子塞进你的大脑。编程语言理论，需要一定程度上的严谨性，这可能是你上一次微积分考试后，就没有再经历过的。但幸运的是，本书中，我们将很少涉及严谨的理论研究。

实现一个解释器，使用了其他类型应用程序中，不太使用的，架构技巧和设计模式。因此，我们将熟悉一些软件工程方面的事情，我们将编写尽可能简明的代码。

在不到两千行Java代码中，我们将实现一个Lox语言的完整的解释器，这个解释器，实现了语言的所有特性，正如我们期望的那样。接下来的几章，我们将介绍解释器的扫描、解析、优化阶段，之后，我们将一次直接添加一个特性，把一个简单的计算器，扩展为一个成熟的脚本语言。


