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

![ast](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/ast.png?raw=true)

下一步是解析，这就是我们获得语法的地方，语法可以将较小的部分组合成较大的表达式和语句。你在英语课堂上画过句子图吗？如果是这样的话，你已经完成了解析器的工作。除了英语有成千上万个关键词和更多的歧义。相较而言，编程语言就简单太多了。

解析器接收token 序列，然后构建出反应语法嵌套性质的树结构。这些树有一些不同的名称，例如：名称解析树，抽象语法树，命名取决于这些树和源语言的简单语法结构的接近程度。在实践中，语言高手经常称它们为语法树，AST或者通常就称为树。

Parsing has a long, rich history in computer science that is closely tied to the artificial intelligence community. Many of the techniques used today to parse programming languages were originally conceived to parse human languages by AI researchers who were trying to get computers to talk to us.

It turns out human languages were too messy for the rigid grammars those parsers could handle, but they were a perfect fit for the simpler artificial grammars of programming languages. Alas, we flawed humans still manage to use those simple grammars incorrectly, so the parser’s job also includes letting us know when we do by reporting syntax errors.

解析在计算机科学中拥有悠久丰富的历史，与人工智能领域密切相关。今天用于解析编程语言的许多技术最初是由人工智能研究人员构思的，他们最初设想是让计算机与人对话交流。

事实证明，对于解析器所能解析的语法而言，人类的语言太复杂了，但是这些解析器却非常适合编程语言中的人类定义的语法规则。哎，我们这些普通的人，在使用这些简单语法时候，仍然会犯错误，所以，解析器还会报告语法错误，让我们知道。

### 1.3 Static analysis

静态分析

The first two stages are pretty similar across all implementations. Now, the individual characteristics of each language start coming into play. At this point, we know the syntactic structure of the code—things like which expressions are nested in which—but we don’t know much more than that.

In an expression like a + b, we know we are adding a and b, but we don’t know what those names refer to. Are they local variables? Global? Where are they defined?

前面两个阶段（扫描、解析）在所有的实现中都是相似的，现在，每种语言的特性开始显现出来了。解析过后，我们知道了代码的语法结构，例如：我们知道了包含了哪些表达式，但是我们了解的还不够多。

在像表达式 a + b中，我们知道表达式是 a与b求和，但是我们并不知道a，b具体表示什么，它们是局部变量吗，是全局变量吗，它们是在哪里定义的呢？

The first bit of analysis that most languages do is called binding or resolution. For each identifier, we find out where that name is defined and wire the two together. This is where scope comes into play—the region of source code where a certain name can be used to refer to a certain declaration.

If the language is statically typed, this is when we type check. Once we know where a and b are declared, we can also figure out their types. Then if those types don’t support being added to each other, we report a type error.

大多数语言的第一点分析叫做，绑定或解析。对于每个标识符，我们需要找到该标识符名称的定义位置，并且将两者连接在一起，这就是作用域发挥作用的地方——源代码的某个区域中，可以使用一个具体名称来引用某个声明。

如果语言是静态语言，这时候，我们还要进行类型判断，一旦我们找到了 a，b的声明位置，我们也可以获取到它们的数据类型。然后，如果这些数据类型不支持加法规则，我们将报告一个类型错误。

> The language we’ll build in this book is dynamically typed, so it will do its type checking later, at runtime.

> 本书中构建的语言是动态语言，所以，类型检查将发生在运行时，而不是当前阶段。

Take a deep breath. We have attained the summit of the mountain and a sweeping view of the user’s program. All this semantic insight that is visible to us from analysis needs to be stored somewhere. There are a few places we can squirrel it away:

* Often, it gets stored right back as attributes on the syntax tree itself—extra fields in the nodes that aren’t initialized during parsing but get filled in later.

* Other times, we may store data in a lookup table off to the side. Typically, the keys to this table are identifiers—names of variables and declarations. In that case, we call it a symbol table and the values it associates with each key tell us what that identifier refers to.

* The most powerful bookkeeping tool is to transform the tree into an entirely new data structure that more directly expresses the semantics of the code. That’s the next section.

深呼吸，我们已经登上了山顶，用户程序一览无余。从分析阶段后，获取到的语义附加信息，需要保存在某个地方。有几个地方可以保存这些信息。

* 通常，它作为属性存储在语法树的其他字段中，这些字段在解析阶段没有初始化，但是在分析阶段会被填充

* 其他时候，我们可以将数据保存在旁边的查找表中。通常情况下，该查找表的key是标识符（变量名称和声明）。这这种情况下，我们称其为符号表，表中key对应的value，表示该标识符对应的实际值是什么

* 更加强大的记录方式是，将语法树转换为一个全新的数据结构，更加直接的表示代码的含义。这是下一节的内容。


Everything up to this point is considered the front end of the implementation. You might guess everything after this is the back end, but no. Back in the days of yore when “front end” and “back end” were coined, compilers were much simpler. Later researchers invented new phases to stuff between the two halves. Rather than discard the old terms, William Wulf and company lumped those new phases into the charming but spatially paradoxical name middle end.

到现在为止，所有内容阶段，都是实现的前端部分。你可能会想象，那么之后的内容是后端了，其实并不是，哈哈😄。当编译器，刚开始有前端、后端概念的时候，那时候的编译器非常简单。后来，研究人员又发明了其他阶段，它们在前端、后端之间。[William Wulf](https://en.wikipedia.org/wiki/William_Wulf) 和他们公司并没有抛弃旧的术语，他们把这些新发明的阶段归为有意思的但是有些矛盾的术语，中间端。

### 1.4 Intermediate representations

中间表示法

You can think of the compiler as a pipeline where each stage’s job is to organize the data representing the user’s code in a way that makes the next stage simpler to implement. The front end of the pipeline is specific to the source language the program is written in. The back end is concerned with the final architecture where the program will run.

In the middle, the code may be stored in some intermediate representation (IR) that isn’t tightly tied to either the source or destination forms (hence “intermediate”). Instead, the IR acts as an interface between these two languages.

我们可以将编译器视为一个管道，每个阶段的工作是用一种更易于实现的方式，组织表示用户代码。管道前端，作用于编写程序的源语言，后端与程序运行的最终架构相关。

在中间端，代码可能存储在一些中间表示中，这些表示，与源语言和目标形式都没有紧密关联。相反，这种中间表示，充当源语言和目标语言之间的接口。

> There are a few well-established styles of IRs out there. Hit your search engine of choice and look for “control flow graph”, “static single-assignment”, “continuation-passing style”, and “three-address code”.

> 有一些成熟的中间表示风格，有兴趣的话，可以去研究一下，控制流图、静态单赋值、连续传递样式、三地址代码等等。

This lets you support multiple source languages and target platforms with less effort. Say you want to implement Pascal, C, and Fortran compilers, and you want to target x86, ARM, and, I dunno, SPARC. Normally, that means you’re signing up to write nine full compilers: Pascal→x86, C→ARM, and every other combination.

A shared intermediate representation reduces that dramatically. You write one front end for each source language that produces the IR. Then one back end for each target architecture. Now you can mix and match those to get every combination.

There’s another big reason we might want to transform the code into a form that makes the semantics more apparent . . . 

这种方式，可以让我们更容易支持多种源语言和目标语言的匹配。假设你想要实现，Pascal、C、Fortran语言的编译器，并且这些编译器，可以运行在X86， arm，SPARC等架构上，如果没有使用中间表示，你需要实现9种编译器，例如: Pascal——> x86，C——> arm等等。

使用一种共享的中间表示，可以大大减少这些组合。对于每一种源语言，编写一个前端，将源语言解析为中间表示，对于每个目标架构，针对中间表示，编写一个后端。所以，现在只需要实现 6 中前端、后端组合。

还有一个重要原因，让我们使用中间表示。我们可以利用中间表示，将代码转为一种形式，使得语义更加明显。

>If you’ve ever wondered how GCC supports so many crazy languages and architectures, like Modula-3 on Motorola 68k, now you know. Language front ends target one of a handful of IRs, mainly GIMPLE and RTL. Target back ends like the one for 68k then take those IRs and produce native code.
>
> 如果你想要知道 [GCC](https://en.wikipedia.org/wiki/GNU_Compiler_Collection)是如何支持这么多语言和架构的，现在你知道原因了。针对前端的少量中间表示，主要是 [GIMPLE](https://gcc.gnu.org/onlinedocs/gccint/GIMPLE.html) 和 [RTL](https://gcc.gnu.org/onlinedocs/gccint/RTL.html), 不同的目标架构，获取中间表示，生成对应的机器代码。

### 1.5 Optimization

优化

Once we understand what the user’s program means, we are free to swap it out with a different program that has the same semantics but implements them more efficiently—we can optimize it.

A simple example is constant folding: if some expression always evaluates to the exact same value, we can do the evaluation at compile time and replace the code for the expression with its result. If the user typed in this:

```

pennyArea = 3.14159 * (0.75 / 2) * (0.75 / 2);

```

we could do all of that arithmetic in the compiler and change the code to:

```
pennyArea = 0.4417860938;
```

一旦我们理解了用户程序的含义，我们就可以自由的把用户程序替换为具有相同语义，但是更加高效的实现的另外一种程序。我们可以进行优化。

一个简单示例是常数计算。如果某个表达式的计算结果总是相同，那么我们可以在编译时候计算该表达式，并且使用表达式计算结果代替该表达式，例如，客户输入

```
pennyArea = 3.14159 * (0.75 / 2) * (0.75 / 2);
```

我们可以在编译时候计算出值，将代码修改为

```
pennyArea = 0.4417860938;
```

Optimization is a huge part of the programming language business.  Many language hackers spend their entire careers here, squeezing every drop of performance they can out of their compilers to get their benchmarks a fraction of a percent faster. It can become a sort of obsession.

We’re mostly going to hop over that rathole in this book. Many successful languages have surprisingly few compile-time optimizations. For example, Lua and CPython generate relatively unoptimized code, and focus most of their performance effort on the runtime.

优化是编程语言的一个重要组成部分，许多语言高手在整个职业生涯都在不断优化，从编译器中榨取每一点性能，最终使得他们的编译器，基准测试结果提高了0.5%，优化是一个不断进行的过程。

本书中，我们会跳过优化这个步骤，有很多成功语言，也很少使用编译时优化。举例，Lua 和 CPython生成相对未优化的代码，将大部分优化放在运行时。

> If you can’t resist poking your foot into that hole, some keywords to get you started are “constant propagation”, “common subexpression elimination”, “loop invariant code motion”, “global value numbering”, “strength reduction”, “scalar replacement of aggregates”, “dead code elimination”, and “loop unrolling”.
>
> 如果你忍不住想要探索优化这个领域，那么你可以从一些术语入手，例如: "恒定传播", "公共子表达式消除", "循环不变代码", "全局值编号", "强度降低", "聚合标量代替", "僵尸代码消除", "循环展开"。

### 1.6 Code generation

代码生成

We have applied all of the optimizations we can think of to the user’s program. The last step is converting it to a form the machine can actually run. In other words, generating code (or code gen), where “code” here usually refers to the kind of primitive assembly-like instructions a CPU runs and not the kind of “source code” a human might want to read.

Finally, we are in the back end, descending the other side of the mountain. From here on out, our representation of the code becomes more and more primitive, like evolution run in reverse, as we get closer to something our simple-minded machine can understand.

We have a decision to make. Do we generate instructions for a real CPU or a virtual one? If we generate real machine code, we get an executable that the OS can load directly onto the chip. Native code is lightning fast, but generating it is a lot of work. Native code is lightning fast, but generating it is a lot of work. 

我们已经将能够想到的所有优化，应用于用户代码中。最后一步是，将代码转换为机器可以实际运行的形式。换句话说，代码生成（或者生成代码），这里的代码是指，CPU直接运行的类似于原始汇编的指令，而不用人们可以直接阅读的源代码。

最后，我们处于后端，从山的另一端往下走。从现在开始，我们对于代码的表示越来越原始，和上山方向相反，我们希望代码变为机器可以直接运行的形式。

我们需要决定，是生成真实的CPU指令，还是生成虚拟的指令。如果我们生成真实的机器代码，我们会得到一个可执行文件，操作系统可以直接加载到芯片中。原生本机代码运行速度非常快，但是，生成真实的 CPU指令需要大量的工作。现在的架构，有成堆的指令集，复杂的管道，和能够塞满747飞机的历史遗留包袱。

> For example, the AAD (“ASCII Adjust AX Before Division”) instruction lets you perform division, which sounds useful. Except that instruction takes, as operands, two binary-coded decimal digits packed into a single 16-bit register. When was the last time you needed BCD on a 16-bit machine
>
> 举例，AAD指令可以执行除法运算，这听起来非常有用。除此之外，ADD指令会将两个二进制编码额十进制数字作为操作数压缩到单个16位寄存器上，上一次，你需要BCD，并且在16位机器，是什么时候呢？

Speaking the chip’s language also means your compiler is tied to a specific architecture. If your compiler targets x86 machine code, it’s not going to run on an ARM device.  All the way back in the ’60s, during the Cambrian explosion of computer architectures, that lack of portability was a real obstacle.

To get around that, hackers like Martin Richards and Niklaus Wirth, of BCPL and Pascal fame, respectively, made their compilers produce virtual machine code. Instead of instructions for some real chip, they produced code for a hypothetical, idealized machine. Wirth called this p-code for portable, but today, we generally call it bytecode because each instruction is often a single byte long.

These synthetic instructions are designed to map a little more closely to the language’s semantics, and not be so tied to the peculiarities of any one computer architecture and its accumulated historical cruft. You can think of it like a dense, binary encoding of the language’s low-level operations.

转换为芯片特定语言，意味着你的编译器和特定架构相关联。如果编译器以x86 机器代码为目标，那么它将无法在arm架构机器上运行。一直追溯到上世纪60年代，在当时的计算机体系结构爆炸时期，缺乏可移植性的编译器是一个真正的缺点。

为了避免这种问题，编程高手，例如：BCPL 语言的发明者[Martin Richards](https://en.wikipedia.org/wiki/Martin_Richards_(computer_scientist)) 和 Pascal语言的主要开发者[Niklaus Wirth](https://en.wikipedia.org/wiki/Niklaus_Wirth) ,不约而同的，让他们实现的编译器最终生成虚拟机代码。他们为一个假想的、理想化的机器生成代码，而不是为了一些真实的芯片生成指令。[Niklaus Wirth](https://en.wikipedia.org/wiki/Niklaus_Wirth)称这些代码为P代码，因为可移植单词的缩写，但是今天，我们通常称为字节码，因为每条指令通常只有一个字节长度。

这些合成指令，是为了更加接近代码的语义，而不是因为更加关联那些架构或是其后的历史。我们可以想象字节码是更加底层的二进制编码。

> The basic principle here is that the farther down the pipeline you push the architecture-specific work, the more of the earlier phases you can share across architectures.
> 
> 基本的原则是，越晚把编译器局限于特定架构上，就可以越多的享受不同架构间代码共用。

>There is a tension, though. Many optimizations, like register allocation and instruction selection, work best when they know the strengths and capabilities of a specific chip. Figuring out which parts of your compiler can be shared and which should be target-specific is an art.
>
> 不过，存在一种均衡，许多优化，例如：寄存器分配和指令选择，当你了解特定芯片的优点和能力时候，使用效果会更好。搞清楚编译器哪些部分可以共享，哪些部分只适用于特定架构，是一门艺术。


### 1.7 Virtual machine

虚拟机

If your compiler produces bytecode, your work isn’t over once that’s done. Since there is no chip that speaks that bytecode, it’s your job to translate. Again, you have two options. You can write a little mini-compiler for each target architecture that converts the bytecode to native code for that machine. You still have to do work for each chip you support, but this last stage is pretty simple and you get to reuse the rest of the compiler pipeline across all of the machines you support. You’re basically using your bytecode as an intermediate representation.

Or you can write a virtual machine (VM), a program that emulates a hypothetical chip supporting your virtual architecture at runtime. Running bytecode in a VM is slower than translating it to native code ahead of time because every instruction must be simulated at runtime each time it executes. In return, you get simplicity and portability. Implement your VM in, say, C, and you can run your language on any platform that has a C compiler. This is how the second interpreter we build in this book works.

如果你的编译器，最终产生字节码，那么，你的工作还没有结束。如果你的工作是翻译的话，那么现在没有芯片可以直接执行字节码。同样，现在你有两个选择。

第一个选择是，你可以为每一种目标架构编写一个小型编译器，把字节码转换为架构中机器使用的机器码。我们可以为支持的每一种芯片编写对应的编译器，但是最后这个阶段非常简单，你也可以重复使用编译器之前的代码。我们使用编译器生成的字节码当作中间表示。

或者，我们可以编写一个虚拟机，一个在运行时候，模拟虚构芯片的程序。在虚拟机中运行字节码比直接在本机运行机器码，慢一些，因为虚拟机每次运行指令，都必须模拟实际指令。作为回报，我们获得了简单性和可移植性。假设用C语言实现虚拟机，那么我们可以在任何有C编译器的机器上运行代码。本书第二部分实现的编译器采用该原理。

> The term “virtual machine” also refers to a different kind of abstraction. A system virtual machine emulates an entire hardware platform and operating system in software. This is how you can play Windows games on your Linux machine, and how cloud providers give customers the user experience of controlling their own “server” without needing to physically allocate separate computers for each user.
>
>The kind of VMs we’ll talk about in this book are language virtual machines or process virtual machines if you want to be unambiguous.
>
> 术语虚拟机是一种抽象。一个系统级别的虚拟机，会模拟整个硬件平台和操作系统。这就是，你可以在Linux服务器上玩Windows游戏的原因，还有，这也是云服务厂商，为用户分配指定的云服务器，而不需要真实提供对应的机器的原因。
>
> 但是，本书中涉及到的虚拟机，只是语言虚拟机或者进程虚拟机，如果你想要一个准确的描述词。

### 1.8 Runtime

运行时

We have finally hammered the user’s program into a form that we can execute. The last step is running it. If we compiled it to machine code, we simply tell the operating system to load the executable and off it goes. If we compiled it to bytecode, we need to start up the VM and load the program into that.

In both cases, for all but the basest of low-level languages, we usually need some services that our language provides while the program is running. For example, if the language automatically manages memory, we need a garbage collector going in order to reclaim unused bits. If our language supports “instance of” tests so you can see what kind of object you have, then we need some representation to keep track of the type of each object during execution.

All of this stuff is going at runtime, so it’s called, appropriately, the runtime. In a fully compiled language, the code implementing the runtime gets inserted directly into the resulting executable.  In, say, Go, each compiled application has its own copy of Go’s runtime directly embedded in it. If the language is run inside an interpreter or VM, then the runtime lives there. This is how most implementations of languages like Java, Python, and JavaScript work.

最终，我们终于把用户程序，转换为一种可以执行的形式。最后一步是运行，如果我们最后编译为机器码，我们只需要加载可执行文件，然后运行。如果将其编译为字节码，我们需要启动虚拟机，把编译的字节码加载到虚拟机中。

在两种场景下，除了最基本的低层语言，当程序运行时，我们还需要提供一些其他服务。例如：如果语言是自动管理内存的，那么我们需要一个垃圾收集器，回收不再使用的内存。如果我们实现的语言，支持实例测试，以便于获取对象的实际数据类型，那么我们需要一些功能，跟踪运行时候的对象。

所有这些都发生在程序运行时候，所以，我们称呼这个阶段为运行时。在一个完全编译的语言中，运行时，代码将直接插入到可执行文件中。举例，在go语言中，每个编译的应用程序都有自己的 go运行时副本，直接嵌入其中。如果语言在解释器或者虚拟机中运行，那么运行时就在其中。这也是 Java/Python/JavaScript等语言的运行工作方式。

## 二、Shortcuts and Alternate Routes

快捷方式和备选路径

That’s the long path covering every possible phase you might implement. Many languages do walk the entire route, but there are a few shortcuts and alternate paths.

这一条漫长的路，可能包含你的实现的每一个阶段。许多语言贯穿了整个过程，但是也有一些语言，会包含捷径和其他备选路径。

### 2.1 Single-pass compilers

单通道编译器

Some simple compilers interleave parsing, analysis, and code generation so that they produce output code directly in the parser, without ever allocating any syntax trees or other IRs. These single-pass compilers restrict the design of the language. You have no intermediate data structures to store global information about the program, and you don’t revisit any previously parsed part of the code. That means as soon as you see some expression, you need to know enough to correctly compile it.

Pascal and C were designed around this limitation. At the time, memory was so precious that a compiler might not even be able to hold an entire source file in memory, much less the whole program. This is why Pascal’s grammar requires type declarations to appear first in a block. It’s why in C you can’t call a function above the code that defines it unless you have an explicit forward declaration that tells the compiler what it needs to know to generate code for a call to the later function.

一些简单的编译器，会把解析阶段、分析阶段、代码生成阶段混杂在一起，它们直接在解析过程生成代码，而不需要生成语法树或者其他中间表示。这些单通道编译器限制了语言的设计。你没有中间数据结构来存储有关程序的全局信息，也无法重新访问任何之前解析过的代码。这意味着，一旦看到某个表达式，我们需要足够的信息来，正确的编译表达式。

Pascal 和 C语言是围绕上面的限制设计的。当时，内存非常宝贵，编译器甚至无法将整个源文件放入到内存中，更不用说在内存中，保存整个程序了。这就是为什么Pascal语言要求类型声明首先出现在代码块中，这也是为什么在C语言中，除非有一个明确的正向声明，告诉编译器生成调用后面的函数所需要的代码，否则无法在函数定义的位置上面，调用该函数。

> Syntax-directed translation is a structured technique for building these all-at-once compilers. You associate an action with each piece of the grammar, usually one that generates output code. Then, whenever the parser matches that chunk of syntax, it executes the action, building up the target code one rule at a time.
> 
> 语法定向翻译是一种结构化技术，用于同时构建这些编译器，将每一个动作和每一个语法片段相关联，通常是输出代码的语法片段。然后，每当解析器匹配该语法块时候，它会执行操作，一次建立一个规则的目标代码。

### 2.2 Tree-walk interpreters

树遍历解释器

Some programming languages begin executing code right after parsing it to an AST (with maybe a bit of static analysis applied). To run the program, the interpreter traverses the syntax tree one branch and leaf at a time, evaluating each node as it goes.

This implementation style is common for student projects and little languages, but is not widely used for general-purpose languages since it tends to be slow. Some people use “interpreter” to mean only these kinds of implementations, but others define that word more generally, so I’ll use the inarguably explicit tree-walk interpreter to refer to these. Our first interpreter rolls this way.

许多的语言，在解析阶段生成了语法树后，就开始执行代码（可能会使用一些静态分析）。为了运行程序，解释器每一次都会遍历语法树的一个分支和叶节点，在每一个节点运行时候，进行评估。

这种实现风格，在学生作业和小的语言中非常常见，但是，由于运行速度比较慢，没有广泛的应用于通用的语言。一些人使用解释器，表示这种类型的实现，但是，另外一些人，使用更加一般的术语描述这种实现方式，本书中我使用树遍历解释器来描述这种实现。我们第一个实现Lox语言采用这种实现风格。

> A notable exception is early versions of Ruby, which were tree walkers. At 1.9, the canonical implementation of Ruby switched from the original MRI (Matz’s Ruby Interpreter) to Koichi Sasada’s YARV (Yet Another Ruby VM). YARV is a bytecode virtual machine.
>
> 一个著名的例子是，Ruby语言的早期版本，使用了树遍历风格的实现方式。在1.9版本，Ruby的实现从早期的MRI变更为YARV，YARV是一个字节码虚拟机。

### 2.3 Transpilers

转换机

Writing a complete back end for a language can be a lot of work. If you have some existing generic IR to target, you could bolt your front end onto that. Otherwise, it seems like you’re stuck. But what if you treated some other source language as if it were an intermediate representation?

Writing a complete back end for a language can be a lot of work. If you have some existing generic IR to target, you could bolt your front end onto that. Otherwise, it seems like you’re stuck. But what if you treated some other source language as if it were an intermediate representation?

You write a front end for your language. Then, in the back end, instead of doing all the work to lower the semantics to some primitive target language, you produce a string of valid source code for some other language that’s about as high level as yours. Then, you use the existing compilation tools for that language as your escape route off the mountain and down to something you can execute.

They used to call this a source-to-source compiler or a transcompiler. After the rise of languages that compile to JavaScript in order to run in the browser, they’ve affected the hipster sobriquet transpiler.

为一种语言编写一个完整的后端，需要很多的工作。如果你有一些现有的通用后端目标，那么你可以把前端设计为通用后端的匹配前端。否则的话，你似乎卡在这里了。但是，如果你把其他的语言视为一种中间表示，那么我们应该如何匹配？

你为自己的语言编写了前端，然后，在后端，你的想法不是，把用户程序的语义变更为底层原始的目标语言，而是把用户语义转换为一个更高级语言的源代码。然后，你可以使用这种高级语言的已经存在的编译器，作为下山的备选路径。

在以前，这种实现方式被称为源代码到源代码编译器或者 转换编译器，当出现了一些语言，为了能在浏览器运行，，最终编译为JavaScript后，大家想到了一个新的名称 转换机 来描述。

While the first transcompiler translated one assembly language to another, today, most transpilers work on higher-level languages. After the viral spread of UNIX to machines various and sundry, there began a long tradition of compilers that produced C as their output language. C compilers were available everywhere UNIX was and produced efficient code, so targeting C was a good way to get your language running on a lot of architectures.

Web browsers are the “machines” of today, and their “machine code” is JavaScript, so these days it seems almost every language out there has a compiler that targets JS since that’s the main way to get your code running in a browser.

The front end—scanner and parser—of a transpiler looks like other compilers. Then, if the source language is only a simple syntactic skin over the target language, it may skip analysis entirely and go straight to outputting the analogous syntax in the destination language.

If the two languages are more semantically different, you’ll see more of the typical phases of a full compiler including analysis and possibly even optimization. Then, when it comes to code generation, instead of outputting some binary language like machine code, you produce a string of grammatically correct source (well, destination) code in the target language.

Either way, you then run that resulting code through the output language’s existing compilation pipeline, and you’re good to go.

虽然第一个转换机，把一种汇编语言转换为另外一种汇编语言，但是现在，我们常常把一门语言转换为更加高级的语言。因为UNIX的风靡，编译器开始了一个传统，那就是把转换机输出语言变为 C语言。C语言编译器，存在于任意的UNIX系统中，支持更多的架构，并且C编译器可以生成更加高效的代码，因此，把C语言作为输出目标，是一个非常好的方向。

现在，web浏览器是一种新型机器，这种新机器的运行代码是JavaScript，所以现在很多语言都有一个针对JS的编译器，最终输出JS代码，这样可以让新的语言运行在浏览器中。[详细清单](https://github.com/jashkenas/coffeescript/wiki/list-of-languages-that-compile-to-js)

转换机的前端部分，扫描阶段、解析阶段，和其他的编译器相似，然而，如果新语言只是目标语言的一个简单皮肤，那么编译器可能会完全跳过分析阶段，直接输出目标语言。

但是，如果这两种语言，在语义上非常不相同，那么，你可能会添加一些传统编译器的其他阶段，例如：分析阶段，优化阶段。但是，最终代码生成阶段，我们不会生成传统的机器码或者字节码，而是生成一个语法正确的目标语言字符串。

无论如何，你都可以在编译后，获取到目标语言的源代码，然后通过目标语言的编译器，开始运行新语言。

> The first transcompiler, XLT86, translated 8080 assembly into 8086 assembly. That might seem straightforward, but keep in mind the 8080 was an 8-bit chip and the 8086 a 16-bit chip that could use each register as a pair of 8-bit ones. XLT86 did data flow analysis to track register usage in the source program and then efficiently map it to the register set of the 8086.
>
> It was written by Gary Kildall, a tragic hero of computer science if there ever was one. One of the first people to recognize the promise of microcomputers, he created PL/M and CP/M, the first high-level language and OS for them.
> 
> He was a sea captain, business owner, licensed pilot, and motorcyclist. A TV host with the Kris Kristofferson-esque look sported by dashing bearded dudes in the ’80s. He took on Bill Gates and, like many, lost, before meeting his end in a biker bar under mysterious circumstances. He died too young, but sure as hell lived before he did.
>
>第一个转换机，XLT86，把8080 汇编转换为8086汇编，这看起来似乎很简单，但是，请注意，8080是一个8位芯片，而8086是一个16位芯片，可以将8086的寄存器，作为8080的一对寄存器使用，XLT86进行了数据流分析，用于跟踪源程序中的寄存器使用，然后，有效的把这些寄存器，映射为8086的寄存器集。
>
> XLT86 是由Gary KIldall实现的，他是计算机科学领域的一个悲剧人物，作为最早认识到微型计算机前景的人之一，他开发了PL/M 和 CP/M，而CP/M是第一种和操作系统交互的高级语言。

>JS used to be the only way to execute code in a browser. Thanks to WebAssembly, compilers now have a second, lower-level language they can target that runs on the web.
>
>JS曾经是浏览器中唯一的运行语言，但是现在我们多了一种选择 [WebAssembly](https://webassembly.org/), 编译器现在拥有了第二种可以直接在浏览器运行的语言。


### 2.4 Just-in-time compilation

即时编译

This last one is less a shortcut and more a dangerous alpine scramble best reserved for experts. The fastest way to execute code is by compiling it to machine code, but you might not know what architecture your end user’s machine supports. What to do?

You can do the same thing that the HotSpot Java Virtual Machine (JVM), Microsoft’s Common Language Runtime (CLR), and most JavaScript interpreters do. On the end user’s machine, when the program is loaded—either from source in the case of JS, or platform-independent bytecode for the JVM and CLR—you compile it to native code for the architecture their computer supports. Naturally enough, this is called just-in-time compilation. Most hackers just say “JIT”, pronounced like it rhymes with “fit”.

The most sophisticated JITs insert profiling hooks into the generated code to see which regions are most performance critical and what kind of data is flowing through them. Then, over time, they will automatically recompile those hot spots with more advanced optimizations.

最后一个不是捷径，而是危险的高山攀岩，最好留给专家。最快的代码执行速度，肯定是翻译为具体的机器码，但是，编译阶段，你可能不知道用户的机器是什么架构，那么我们应该怎么办呢？

你可以借鉴JVM（Java语言虚拟机），CLR（微软的通用语言运行库），还有大多数JS编译器做的，在用户的机器上，当程序加载时候，无论是从源代码，还是在JVM/CLR中加载字节码，你可以将其编译为本机的机器码。很自然的，这个过程称为即时编译。大多数编程高手称这种实现为JIT，发音类似fit。

最复杂的JIT，会在生成代码中插入一些性能分析代码，查看哪些代码块、哪些数据结构对于运行性能影响最大。然后，随着时间累积，JIT将自动使用更高级的优化方式，重新编译热点代码。

> This is, of course, exactly where the HotSpot JVM gets its name.
>
> 当然，这也是JVM的一种实现，HotSpot JVM的名称来源。

## 三、Compilers and Interpreters

编译器和解释器

Now that I’ve stuffed your head with a dictionary’s worth of programming language jargon, we can finally address a question that’s plagued coders since time immemorial: What’s the difference between a compiler and an interpreter?

It turns out this is like asking the difference between a fruit and a vegetable. That seems like a binary either-or choice, but actually “fruit” is a botanical term and “vegetable” is culinary. One does not strictly imply the negation of the other. There are fruits that aren’t vegetables (apples) and vegetables that aren’t fruits (carrots), but also edible plants that are both fruits and vegetables, like tomatoes.

现在，我们大脑中已经塞满了各种编译术语，接下来，我们将解决一个自古以来就困扰着程序员的问题，编译器和解释器有什么区别？

实际上，这个问题可以类比为，水果和蔬菜有什么区别？这个答案看起来是一个二选一问题，但是实际上，水果是一个植物学术语，蔬菜则是一个烹饪用语，一个东西是水果并不代表它不可以是蔬菜，现实生活中，我们可以找到某些水果，不是蔬菜，例如：苹果；也可以找到某些蔬菜，不属于水果，例如：胡萝卜；但是，我们也可以找到，某些东西，即是水果，也是蔬菜，例如：西红柿。

![plants](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/plants.png?raw=true)

So, back to languages:

* Compiling is an implementation technique that involves translating a source language to some other—usually lower-level—form. When you generate bytecode or machine code, you are compiling. When you transpile to another high-level language, you are compiling too.

* When we say a language implementation “is a compiler”, we mean it translates source code to some other form but doesn’t execute it. The user has to take the resulting output and run it themselves.

* Conversely, when we say an implementation “is an interpreter”, we mean it takes in source code and executes it immediately. It runs programs “from source”.

那么，返回到语言部分，

* 编译是一种实现技术，通常是将源语言转换为更加低级别的形式，当你把一门语言编译为字节码或者机器码时候，你使用了编译技术，当你转换为另外一种高级语言时候，你也在使用编译。

* 当我们说实现了一个编译器，我们的意思是，将源语言转换为其他形式，但是并不执行，用户需要获取到编译结果，然后再运行。

* 对应的，当我们说实现了一个解释器，我们的意思是，可以直接执行源代码，看起来，我们好像是直接从源语言运行。

Like apples and oranges, some implementations are clearly compilers and not interpreters. GCC and Clang take your C code and compile it to machine code. An end user runs that executable directly and may never even know which tool was used to compile it. So those are compilers for C.

In older versions of Matz’s canonical implementation of Ruby, the user ran Ruby from source. The implementation parsed it and executed it directly by traversing the syntax tree. No other translation occurred, either internally or in any user-visible form. So this was definitely an interpreter for Ruby.

>Peanuts (which are not even nuts) and cereals like wheat are actually fruit, but I got this drawing wrong. What can I say, I’m a software engineer, not a botanist. I should probably erase the little peanut guy, but he’s so cute that I can’t bear to.
> 
> Now pine nuts, on the other hand, are plant-based foods that are neither fruits nor vegetables. At least as far as I can tell.


像苹果和橘子一样，它们是水果但不是蔬菜，有一些实现，是编译器，而不是解释器。GCC和Clang 接收原始的C语言程序，最终编译为机器码。用户最终运行可执行文件，而不需要知道具体使用了哪个编译器，它们都是C语言编译器。

Ruby的老版本中，用户可以直接从Ruby源码运行。Ruby解释器直接解析源程序，生成语法树，然后，遍历语法树，直接执行，无论处于用户角度，还是实际内部机制，都没有其他的转换过程，我们可以确定这种实现为解释器。


But what of CPython? When you run your Python program using it, the code is parsed and converted to an internal bytecode format, which is then executed inside the VM. From the user’s perspective, this is clearly an interpreter—they run their program from source. But if you look under CPython’s scaly skin, you’ll see that there is definitely some compiling going on.

The answer is that it is both. CPython is an interpreter, and it has a compiler. In practice, most scripting languages work this way, as you can see:

![venn](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/venn.png?raw=true)

但是，CPython是什么呢？当我们运行Python程序时候，CPython编译器将Python代码转换为内部的字节码，在Python虚拟机中运行。从用户角度看，这是一个解释器，因为我们可以直接从原始代码运行，但是如果从内部实现角度，还存在着一些编译器。

准确答案是，CPython既是一个解释器，也是一个编译器，实际上，大部分的脚本语言都是这样的

That overlapping region in the center is where our second interpreter lives too, since it internally compiles to bytecode. So while this book is nominally about interpreters, we’ll cover some compilation too.

如上图，我们第二部分实现的Lox解释器，处于中间的重叠部分，因为我们会生成字节码。因此，虽然本书是关于解释器介绍的，但是，我们还是会涉及到编译器的内容。

>The Go tool is even more of a horticultural curiosity. If you run go build, it compiles your Go source code to machine code and stops. If you type go run, it does that, then immediately executes the generated executable.
>
> So go is a compiler (you can use it as a tool to compile code without running it), is an interpreter (you can invoke it to immediately run a program from source), and also has a compiler (when you use it as an interpreter, it is still compiling internally).
>
> go语言的命令更加能说明问题，例如：执行命令 go build，我们能得到一个可执行文件，如果执行命令，go run，程序会马上运行。
>
> 所以，go是一个编译器，可以将go程序编译为可执行文件，go也是一个解释器，可以直接运行go程序，但是解释器中还包含一个编译器，当直接运行go程序时候，内部仍有编译步骤。

## 四、Our Journey

我们的旅途

That’s a lot to take in all at once. Don’t worry. This isn’t the chapter where you’re expected to understand all of these pieces and parts. I just want you to know that they are out there and roughly how they fit together.

This map should serve you well as you explore the territory beyond the guided path we take in this book. I want to leave you yearning to strike out on your own and wander all over that mountain.

But, for now, it’s time for our own journey to begin. Tighten your bootlaces, cinch up your pack, and come along. From here on out, all you need to focus on is the path in front of you.

本章，我们介绍了很多内容，别担心，你不需要现在就理解所有内容，我们只是，先介绍它们，你需要知道它们是存在的，并且需要知道它们是如何结合在一起的。

本章涉及的地图将很好的陪伴着你，因为它包含有一些内容，本书中不会涉及到。我想要离开你，让你独自去探索、享受爬山的过程。

但是，现在还不是独自探索的时候，让我们一起开启旅程。系紧鞋带，收好背包，跟上来，从现在开始，你需要关注眼前的道路。

> Henceforth, I promise to tone down the whole mountain metaphor thing.
> 
> 从今往后，我会淡化爬山这件事情。

## 五、CHALLENGES

习题集

1. Pick an open source implementation of a language you like. Download the source code and poke around in it. Try to find the code that implements the scanner and parser. Are they handwritten, or generated using tools like Lex and Yacc? (.l or .y files usually imply the latter.)

   选择一种你熟悉、喜欢的开源语言，下载源码然后浏览一下，尝试找出其中的扫描器、解析器部分，判断它们是自己实现的，还是使用Lex/Yacc等编译器工具实现的，可以查看是否存在 .l, .y 后缀的文件，这通常是工具生成文件。
   
   
1. Just-in-time compilation tends to be the fastest way to implement dynamically typed languages, but not all of them use it. What reasons are there to not JIT?

	即时编译通常是动态语言最快的实现方式，但是并非所有的语言都利用这种特性，为什么它们不提供即时编译？
	
1. Most Lisp implementations that compile to C also contain an interpreter that lets them execute Lisp code on the fly as well. Why?

	大多数的Lisp实现，在实现一个编译器，编译为C语言的同时，还提供一个解释器，保证可以动态执行Lisp代码，为什么？



























