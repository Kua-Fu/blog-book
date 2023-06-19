# 虚拟机


A BYTECODE VIRTUAL MACHINE

Our Java interpreter, jlox, taught us many of the fundamentals of programming languages, but we still have much to learn. First, if you run any interesting Lox programs in jlox, you’ll discover it’s achingly slow. The style of interpretation it uses—walking the AST directly—is good enough for some real-world uses, but leaves a lot to be desired for a general-purpose scripting language.

我们通过Java实现的解释器jlox，在这个过程中，我们学会了很多基础的编程知识，但是，我们仍然有很多需要继续学习的地方。首先，如果我们使用jlox运行一些程序时候，可能会发现运行非常慢；它所使用的解释器风格——直接遍历语法树——对于一些实际应用已经足够了，但是，对于通用的脚本语言，还需要改进。

Also, we implicitly rely on runtime features of the JVM itself. We take for granted that things like instanceof in Java work somehow. And we never for a second worry about memory management because the JVM’s garbage collector takes care of it for us.

此外，jlox 依赖于JVM本身运行时特性，我们本能的认为，例如：Java的 instanceof 等操作是可行的。而且，我们从来不需要自己考虑内存管理问题，因为，JVM的垃圾采集器会自动处理。

When we were focused on high-level concepts, it was fine to gloss over those. But now that we know our way around an interpreter, it’s time to dig down to those lower layers and build our own virtual machine from scratch using nothing more than the C standard library . . . 

当我们专注于高级概念时候，可以忽略这些细节。但是，现在我们已经熟悉了解释器的内部工作原理，是时候深入到底层，使用C标准库实现一个自己的虚拟机了。

