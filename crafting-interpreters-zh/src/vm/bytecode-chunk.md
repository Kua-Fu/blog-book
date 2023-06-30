# 字节码块

> If you find that you’re spending almost all your time on theory, start turning some attention to practical things; it will improve your theories. If you find that you’re spending almost all your time on practice, start turning some attention to theoretical things; it will improve your practice.
>
> 如何你发现自己绝大部分时间，都花在理论上，那么开始把一些注意力放在实践上，这将有助于提高你的理论水平；
> 
> 如果你发现自己几乎把所有时间都用在实践中，那就开始关注一些理论性的东西，这会提高你的实践能力。


> <p align=right> —— Donald Knuth </p>

We already have ourselves a complete implementation of Lox with jlox, so why isn’t the book over yet? Part of this is because jlox relies on the JVM to do lots of things for us. If we want to understand how an interpreter works all the way down to the metal, we need to build those bits and pieces ourselves.

我们已经使用jlox 实现了完整的lox，那么为什么本书还没有结束呢？部分原因是因为jlox依赖于 JVM来完成很多工作，如果我们想要完全理解解释器的工作原理，我们需要自己构建这些部分。也就是说，我们需要从底层开始一步步构建解释器，以深入了解其工作原理。


> Of course, our second interpreter relies on the C standard library for basics like memory allocation, and the C compiler frees us from details of the underlying machine code we’re running it on. Heck, that machine code is probably implemented in terms of microcode on the chip. And the C runtime relies on the operating system to hand out pages of memory. But we have to stop somewhere if this book is going to fit on your bookshelf.
> 
> 当然，我们的第二个解释器使用C 标准库来进行基本的内存分配操作，而C编译器则为我们避免了运行机器码时候的内部执行细节，实际上，这些机器码可能是基于芯片上的微码实现，而C 运行时候，依赖于操作系统来分配内存页。但是，如果我们希望这本书适合放入你的书架，必须在某些地方停下来。

An even more fundamental reason that jlox isn’t sufficient is that it’s too damn slow. A tree-walk interpreter is fine for some kinds of high-level, declarative languages. But for a general-purpose, imperative language—even a “scripting” language like Lox—it won’t fly. Take this little script:

还有一个更加实际的原因，是jlox 执行太慢了。对于某些高级、声明式的语言，基于语法树遍历的解释器是可以接受的。但是，对于通用的、命令式语言——例如：lox，这种方式实现的解释器，就不够用了，执行下面的这个程序

```python


fun fib(n) {
  if (n < 2) return n;
  return fib(n - 1) + fib(n - 2); 
}

var before = clock();
print fib(40);
var after = clock();
print after - before;

```
On my laptop, that takes jlox about 72 seconds to execute. An equivalent C program finishes in half a second. Our dynamically typed scripting language is never going to be as fast as a statically typed language with manual memory management, but we don’t need to settle for more than two orders of magnitude slower.

We could take jlox and run it in a profiler and start tuning and tweaking hotspots, but that will only get us so far. The execution model—walking the AST—is fundamentally the wrong design. We can’t micro-optimize that to the performance we want any more than you can polish an AMC Gremlin into an SR-71 Blackbird.

We need to rethink the core model. This chapter introduces that model, bytecode, and begins our new interpreter, clox.

在我的笔记本上，执行上面的lox 程序耗时 72秒，而一个等效的C 程序只需要半秒钟就可以执行同样功能。我们的动态类型脚本语言，可能永远无法像可以自己管理内存的静态类型语言那样高效，但是，我们也不接受两个数量级的差距。

我们可以使用分析器分析lox程序，找到、调整热点代码，但是，这只能很有限的提高性能。jlox的执行模型——遍历AST，对我们来说，是一个错误的设计方向，我们无法通过微观优化来达到我们想要的性能，就像我们无法把 AMC Gremlin （汽车）变成高性能的 SR-71（飞机)

我们需要重新思考核心模型，本章介绍了这个模型——字节码，接下来，我们将开始新的解释器 clox

> This is a comically inefficient way to actually calculate Fibonacci numbers. Our goal is to see how fast the interpreter runs, not to see how fast of a program we can write. A slow program that does a lot of work—pointless or not—is a good test case for that.
> 
> 这实际上是一种非常低效的计算fibonacci数列的方式，我们的目标是查看解释器的运行速度，而不是考察我们能编写多快的程序。一个慢，但是能进行大量工作（无论释放有意义）的程序，是一个好的测试案例。

## 一、Bytecode?

In engineering, few choices are without trade-offs. To best understand why we’re going with bytecode, let’s stack it up against a couple of alternatives.

在工程领域，很少有选择，可以不经过权衡，为了更好的理解我们为什么选择字节码，让我们把它和一些替代方案进行比较。



### 1.1 Why not walk the AST?

Our existing interpreter has a couple of things going for it:

* Well, first, we already wrote it. It’s done. And the main reason it’s done is because this style of interpreter is really simple to implement. The runtime representation of the code directly maps to the syntax. It’s virtually effortless to get from the parser to the data structures we need at runtime.

* It’s portable. Our current interpreter is written in Java and runs on any platform Java supports. We could write a new implementation in C using the same approach and compile and run our language on basically every platform under the sun.

我们现有的解释器有一些优点：

* 首先，我们编写了它，它已经能使用了。这个解释器之所以能完成，主要是因为这种解释器的实现非常简单。代码的运行时表示直接映射到语法，从解释器到我们在运行时所需的数据结构，几乎没有什么难度。

* 它是可移植的，我们当前的解释器是用Java编写的，可以在Java支持的任何平台上运行，我们可以使用相同的方法在C中编写一个新的实现，并且在几乎所有平台上编译和运行我们的语言。

Those are real advantages. But, on the other hand, it’s not memory-efficient. Each piece of syntax becomes an AST node. A tiny Lox expression like 1 + 2 turns into a slew of objects with lots of pointers between them, something like:

这些的确是真正的优点，但是，另一方面，它并不是内存效率高的实现方式，每个语法片段都变成了一个AST 节点，一个小小的Lox表达式，例如: 1+2 就会变为一堆对象，它们之间有很多指针，大概是这样的，

![ast](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/ast1.png?raw=true)

> The “(header)” parts are the bookkeeping information the Java virtual machine uses to support memory management and store the object’s type. Those take up space too!
> 
> header 部分是Java 虚拟机用于支持内存管理和存储对象类型的记录信息，这些也会占用空间！



Each of those pointers adds an extra 32 or 64 bits of overhead to the object. Worse, sprinkling our data across the heap in a loosely connected web of objects does bad things for spatial locality.

每个指针都会给对象增加额外的32位 或者 64位的开销，更糟糕的是，将我们的数据散布到堆中，形成松散链接的对象网络，对空间局部性产生不良影响。

> I wrote an entire chapter about this exact problem in my first book, Game Programming Patterns, if you want to really dig in.
> 
> 如果你想要深入了解这个问题，可以在我写的第一本书《游戏编程模式》中发现更多的解释。

Modern CPUs process data way faster than they can pull it from RAM. To compensate for that, chips have multiple layers of caching. If a piece of memory it needs is already in the cache, it can be loaded more quickly. We’re talking upwards of 100 times faster.

How does data get into that cache? The machine speculatively stuffs things in there for you. Its heuristic is pretty simple. Whenever the CPU reads a bit of data from RAM, it pulls in a whole little bundle of adjacent bytes and stuffs them in the cache.

If our program next requests some data close enough to be inside that cache line, our CPU runs like a well-oiled conveyor belt in a factory. We really want to take advantage of this. To use the cache effectively, the way we represent code in memory should be dense and ordered like it’s read.

Now look up at that tree. Those sub-objects could be anywhere. Every step the tree-walker takes where it follows a reference to a child node may step outside the bounds of the cache and force the CPU to stall until a new lump of data can be slurped in from RAM. Just the overhead of those tree nodes with all of their pointer fields and object headers tends to push objects away from each other and out of the cache.

Our AST walker has other overhead too around interface dispatch and the Visitor pattern, but the locality issues alone are enough to justify a better code representation.

现代CPU 处理数据的速度，远快于从RAM 中读取数据的速度，为了弥补这一点，芯片中有多级缓存，如果它需要的一块内存已经存在于多级缓存中，那么它可以更快的加载数据，这个速度可以快了100倍。

数据是如何进入缓存中的呢？机器会自动将数据放入缓存，它的算法非常简单，每当CPU 读取RAM的一些数据时候，它会将相邻的一组字节一起拉入到缓存中。

如果我们的程序接下来请求的数据接近缓存行，那么我们的CPU就会像工厂里面的传送带一样平滑的运行。我们真的想要利用这一点，为了有效的利用缓存，我们在内存中表示代码的方式应该是密集有序的。

现在看看上面的那棵树，这些子对象可能在任何地方，每一次遍历语法树执行操作时候，都有可能超过缓存的边界，迫使CPU停顿，一直到可以从RAM中读取新的数据。这些语法树节点的开销，包括它们所有的指针字段和对象头，往往会将对象推远并将它们移出缓存。

我们的AST遍历还有其他开销，比如: 接口调用和访问者模式，但是，仅仅局部性问题，就足以需要我们更好的代码表达方式。

> Even if the objects happened to be allocated in sequential memory when the parser first produced them, after a couple of rounds of garbage collection—which may move objects around in memory—there’s no telling where they’ll be.
> 
> 即使对象在解释器第一次生成时候，是按照顺序分配的，经过几轮垃圾回收后（可能会移动内存中的对象），我们无法确定它会被移动到哪里。

### 1.2 Why not compile to native code?

If you want to go real fast, you want to get all of those layers of indirection out of the way. Right down to the metal. Machine code. It even sounds fast. Machine code.

Compiling directly to the native instruction set the chip supports is what the fastest languages do. Targeting native code has been the most efficient option since way back in the early days when engineers actually handwrote programs in machine code.

如果你想要真正快速，你需要摆脱所有的间接层级。直接到机器层面，机器码。它甚至听起来就很快 😄

直接编译为芯片支持的本地指令集是最快的语言所做的，自从早期工程师手写机器码以来，针对本地代码一直是最有效的选择。

If you’ve never written any machine code, or its slightly more human-palatable cousin assembly code before, I’ll give you the gentlest of introductions. Native code is a dense series of operations, encoded directly in binary. Each instruction is between one and a few bytes long, and is almost mind-numbingly low level. “Move a value from this address to this register.” “Add the integers in these two registers.” Stuff like that.

The CPU cranks through the instructions, decoding and executing each one in order. There is no tree structure like our AST, and control flow is handled by jumping from one point in the code directly to another. No indirection, no overhead, no unnecessary skipping around or chasing pointers.

Lightning fast, but that performance comes at a cost. First of all, compiling to native code ain’t easy. Most chips in wide use today have sprawling Byzantine architectures with heaps of instructions that accreted over decades. They require sophisticated register allocation, pipelining, and instruction scheduling.

And, of course, you’ve thrown portability out. Spend a few years mastering some architecture and that still only gets you onto one of the several popular instruction sets out there. To get your language on all of them, you need to learn all of their instruction sets and write a separate back end for each one.

如果你以前没有写过任何机器码或者稍微容易理解的汇编代码，我会给你一个最温和的介绍，本地代码是一系列直接编码为二进制的操作，每条指令的长度是1个或者几个字节之间，几乎让人感动无聊的低级。比如：将一个值从这个地址移动到这个寄存器。将两个寄存器中的值相加，类似这样的操纵。

CPU运行这些指令，按照顺序解码和执行它们，没有像之前提到的语法树结构，控制流直接从代码中的一个点跳转到另一个点来处理。没有间接寻址，没有开销，没有不必要的跳跃，或者指针追踪。

运行速度非常快，但是这种性能是有代价的。首先，编译为机器码并不容易，今天广泛使用的大多数芯片，都有庞大的拜占庭式结构，有数十年的指令集积累。它们需要复杂的寄存器分配、流水线和指令调度。

当然，你也放弃了可移植性。花费几年时间掌握某个体系结构，仍然只能让你使用几种流行的指令集中的一种。要在所有的指令集上运行你的语言，你需要学习所有的指令集并且为每个指令集编写一个单独的后端。

> Yes, they actually wrote machine code by hand. On punched cards. Which, presumably, they punched with their fists.
> 
> 是的，他们实际上是手写机器码，在打孔卡上，我想他们可能是用拳头打孔的吧 😠

> The situation isn’t entirely dire. A well-architected compiler lets you share the front end and most of the middle layer optimization passes across the different architectures you support. It’s mainly the code generation and some of the details around instruction selection that you’ll need to write afresh each time.
>
> The LLVM project gives you some of this out of the box. If your compiler outputs LLVM’s own special intermediate language, LLVM in turn compiles that to native code for a plethora of architectures.
> 
> 情况并非完全绝望，一个良好设计的编译器可以让你在，支持的不同体系结构之间共享前端和大多数中间层优化处理。主要是，代码生成和一些关于指令选择的细节需要每次重新编写。
>
> [LLVM 项目](https://llvm.org/)可以为你提供一些这方面的支持，如果你的编译器输出LLVM 的特殊中间语言，LLVM可以将其编译为众多体系结构的机器码。


### 1.3 What is bytecode?

Fix those two points in your mind. On one end, a tree-walk interpreter is simple, portable, and slow. On the other, native code is complex and platform-specific but fast. Bytecode sits in the middle. It retains the portability of a tree-walker—we won’t be getting our hands dirty with assembly code in this book. It sacrifices some simplicity to get a performance boost in return, though not as fast as going fully native.

请记住这两个观点，

* 一方面，语法树遍历方式的解释器简单、可移植，但是运行速度慢

* 另一方面，编译为机器码方式，复杂、与平台有关，但是运行速度快

字节码处于两者之间。

它保留了第一种方式的可移植性——在本书中，我们不会涉及到汇编代码。但是，它也牺牲了一些简单性，换取性能提升，尽管不如完全的机器码那么快。

Structurally, bytecode resembles machine code. It’s a dense, linear sequence of binary instructions. That keeps overhead low and plays nice with the cache. However, it’s a much simpler, higher-level instruction set than any real chip out there. (In many bytecode formats, each instruction is only a single byte long, hence “bytecode”.)

Imagine you’re writing a native compiler from some source language and you’re given carte blanche to define the easiest possible architecture to target. Bytecode is kind of like that. It’s an idealized fantasy instruction set that makes your life as the compiler writer easier.

在结构上，字节码类似于机器码。它是一个密集的、线性的二进制指令序列。这使得开销比较低，能够很好的和缓存配合。然而，它比任何真实芯片上的指令集要简答很多，属于更高层次的指令集。（在很多字节码格式中，每个指令只有一个字节长，因此称为字节码）

想象一下，你正在从源语言编写一个机器码编译器，你被授权定义一个最简单的架构，字节码就有点像是这样，它是一个理想化的假想的指令集，使得，我们编写编译器更加容易。

The problem with a fantasy architecture, of course, is that it doesn’t exist. We solve that by writing an emulator—a simulated chip written in software that interprets the bytecode one instruction at a time. A virtual machine (VM), if you will.

That emulation layer adds overhead, which is a key reason bytecode is slower than native code. But in return, it gives us portability. Write our VM in a language like C that is already supported on all the machines we care about, and we can run our emulator on top of any hardware we like.

当然，假想的架构的问题是它并不存在，我们通过编写模拟器——一种以软件形式编写的模拟芯片，逐条解释字节码来解决这个问题，一个虚拟机 VM，如果我们要称呼它。

这个仿真层增加了一些开销，这也是字节码比机器码慢的一个重要原因，但是，作为回报，它给了我们可移植性。我们使用类似C 的语言编写VM，而这在所有的机器上都是支持的。我们可以在任何我们喜欢的硬件上，运行我们的模拟器。

This is the path we’ll take with our new interpreter, clox. We’ll follow in the footsteps of the main implementations of Python, Ruby, Lua, OCaml, Erlang, and others. In many ways, our VM’s design will parallel the structure of our previous interpreter:

![phrases](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/phases.png?raw=true)

Of course, we won’t implement the phases strictly in order. Like our previous interpreter, we’ll bounce around, building up the implementation one language feature at a time. In this chapter, we’ll get the skeleton of the application in place and create the data structures needed to store and represent a chunk of bytecode.

这就是我们新的解释器 clox，所要使用的路径。我们将跟随着 Python、Ruby、Lua、OCaml、Erlang和其他语言的实现方式。在很多方面，我们的虚拟机设计将与我们之前的解释器结构相对应。

当然，我们不会按照顺序严格实现各个阶段。就像是以前的解释器一样，我们将跳转，最终实现一个解释器。在本章中，我们将获取该解释器的架构，并且创建数据结构，该结构存储着字节码。

> One of the first bytecode formats was [p-code](https://en.wikipedia.org/wiki/P-code_machine), developed for Niklaus Wirth’s Pascal language. You might think a PDP-11 running at 15MHz couldn’t afford the overhead of emulating a virtual machine. But back then, computers were in their Cambrian explosion and new architectures appeared every day. Keeping up with the latest chips was worth more than squeezing the maximum performance from each one. That’s why the “p” in p-code doesn’t stand for “Pascal”, but “portable”.
> 
> 最早的一种字节码是，Niklaus Wirth 为Pascal语言开发的 p-code. 你可能会认为，PDP-11 以15MHz 的运行频率无法承受一个虚拟机的运行。但当时，计算机正处于寒武纪爆发期，每天都会出现新的体系结构。跟上最新芯片的步伐，总比从每种芯片中挤出更大的性能更值得。这也是为什么，p-code中的p 表示为 protable（可移植性），而不是Pascal 的原因。




## 二、Getting Started


Where else to begin, but at main()? Fire up your trusty text editor and start typing.

没什么比从 main() 开始更好的起点了吧，打开编辑器，让我们开始编程吧

```c

// main.c, create new file

#include "common.h"

int main(int argc, const char* argv[]) {
  return 0;
}

```

> Now is a good time to stretch, maybe crack your knuckles. A little montage music wouldn’t hurt either.
> 
> 现在是一个好的时机，编写新的代码了，同时播放一些音乐或许更好！

From this tiny seed, we will grow our entire VM. Since C provides us with so little, we first need to spend some time amending the soil. Some of that goes into this header:

从这棵微小的种子开始，我们将开始整个虚拟机的编写。由于C语言提供的内容很少，首先，我们需要花费一些时间，改善一下土壤。其中，一部分工作涉及这个头文件。

```c

# common.h, create new file

#ifndef clox_common_h
#define clox_common_h

#include <stdbool.h>
#include <stddef.h>
#include <stdint.h>

#endif

```

There are a handful of types and constants we’ll use throughout the interpreter, and this is a convenient place to put them. For now, it’s the venerable NULL, size_t, the nice C99 Boolean bool, and explicit-sized integer types—uint8_t and friends.

 在解释器中，我们将使用一些类型和常量，并且在common.h 中放置它们。目前，这些包含 NULL，size_t ，C99规范中定义的布尔类型 bool，以及显式大小的整数类型 unit8_t 等
 
## 三、 Chunks of Instructions


Next, we need a module to define our code representation. I’ve been using “chunk” to refer to sequences of bytecode, so let’s make that the official name for that module.

接下来，我们需要一个模块来定义我们的代码表示形式，我一直使用 chunk来指代字节码序列，所以，我们将其作为模块的名称。

```c

// chunk.h, create new file

#ifndef clox_chunk_h
#define clox_chunk_h

#include "common.h"

#endif


```

In our bytecode format, each instruction has a one-byte operation code (universally shortened to opcode). That number controls what kind of instruction we’re dealing with—add, subtract, look up variable, etc. We define those here:

在我们的字节码格式中，每一个指令都有一个一字节长度的操作码（通常缩写为opcode), 该数字控制我们正在处理的指令类型——加、减、查找变量等等。我们在这里定义:

```c

// chunk.h

#include "common.h"

typedef enum {
  OP_RETURN,
} OpCode;

#endif

```

For now, we start with a single instruction, OP_RETURN. When we have a full-featured VM, this instruction will mean “return from the current function”. I admit this isn’t exactly useful yet, but we have to start somewhere, and this is a particularly simple instruction, for reasons we’ll get to later.

目前，我们只是用一个指令 OP_RETURN ,当我们拥有一个功能完整的虚拟机时候，这个指令将意味着 从当前函数返回，我承认现在这并不是非常有用，但是我们必须从某个地方开始，而这是一个特别简单的指令，原因稍后我们会讲到。

### 3.1 A dynamic array of instructions

Bytecode is a series of instructions. Eventually, we’ll store some other data along with the instructions, so let’s go ahead and create a struct to hold it all.

字节码是一系列指令，最终，我们将存储一些和指令一起的其他数据，因此我们将创建一个结构体来保存它们。

```c

// chunk.h, add after enum OpCode

} OpCode;

typedef struct {
  uint8_t* code;
} Chunk;

#endif

```


At the moment, this is simply a wrapper around an array of bytes. Since we don’t know how big the array needs to be before we start compiling a chunk, it must be dynamic. Dynamic arrays are one of my favorite data structures. That sounds like claiming vanilla is my favorite ice cream flavor, but hear me out. Dynamic arrays provide:

* Cache-friendly, dense storage

* Constant-time indexed element lookup

* Constant-time appending to the end of the array

> Butter pecan is actually my favorite.
> 
> 牛油山核桃冰淇淋实际上是我的最爱

Those features are exactly why we used dynamic arrays all the time in jlox under the guise of Java’s ArrayList class. Now that we’re in C, we get to roll our own. If you’re rusty on dynamic arrays, the idea is pretty simple. In addition to the array itself, we keep two numbers: the number of elements in the array we have allocated (“capacity”) and how many of those allocated entries are actually in use (“count”).

目前，这只是一个包装在字节数组周围的结构体，由于在编译块之前，我们不知道数组有多大，因此它必须是动态的，动态数组是我最喜欢的结构之一，这听起来像是声称香草是我最喜欢的冰淇淋口味，但是，请听我解释，动态数组提供了: 

* 缓存友好，密集的存储方式

* 常数时间的索引元素查找

* 在列表添加元素，常数时间

由于这些特征，我们在jlox中一直使用Java 的 ArrayList类来伪装动态数组，现在在我们的clox 中，我们可以自己编写。如果你对动态数组不太熟悉，这个想法相当简单，除了数组本身，我们还保留了两个数字:

(1) 我们已经分配的数组元素个数（容量）

(2) 实际使用的分配元素的数量（计数）

```c

// chunk.h, in struct Chunk

typedef struct {
  int count;
  int capacity;
  uint8_t* code;
} Chunk;


```


When we add an element, if the count is less than the capacity, then there is already available space in the array. We store the new element right in there and bump the count.

![insert](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/14_insert.png?raw=true)

当我们添加一个元素时候，如果计数小于容量，那么数组中仍然有可用的空间。我们将新元素保存在那里，增加计数

If we have no spare capacity, then the process is a little more involved.


![grow](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/14_grow.png?raw=true)

1. Allocate a new array with more capacity.

1. Copy the existing elements from the old array to the new one.

1. Store the new capacity.

1. Delete the old array.

1. Update code to point to the new array.

1. Store the element in the new array now that there is room.

1. Update the count.

如果没有多余的空间，那么这个过程会稍微复杂一点。

1. 分配一个有更大容量的新数组

1. 将现有数据从旧数组复制到新数组

1. 存储新的容量

1. 删除旧数组

1. 更新代码指向新的数组

1. 现在有空间了，将新元素保存到新数组中

1. 更新计数

> Copying the existing elements when you grow the array makes it seem like appending an element is O(n), not O(1) like I said above. However, you need to do this copy step only on some of the appends. Most of the time, there is already extra capacity, so you don’t need to copy.
>
> To understand how this works, we need [amortized analysis](https://en.wikipedia.org/wiki/Amortized_analysis). That shows us that as long as we grow the array by a multiple of its current size, when we average out the cost of a sequence of appends, each append is O(1).
> 
> 当我们增加数组大小时候，复制所有的现有元素似乎使得添加一个元素的时间复杂度为 \\( O(n) \\), 而不是之前的 \\( O(1) \\), 但是，你只需要在某些追加操作中进行复制，大多数情况下，已经有了额外的容量，所以，我们不需要复制
>
> 要理解这是如何工作的，我们需要进行分摊分析，这可以告诉我们，只要我们把数组的大小增加到当前大小的倍数，当我们平均一系列追加操作的成本时候，每个追加操作都是 \\( O(1) \\)


We have our struct ready, so let’s implement the functions to work with it. C doesn’t have constructors, so we declare a function to initialize a new chunk.

我们已经准备好了结构体，接下来，让我们实现与之一起工作的函数，C没有构造函数，因此我们声明一个函数，用于初始化一个新的 chunk

```C

// chunk.h, add after struct Chunk

} Chunk;

void initChunk(Chunk* chunk);

#endif


```

And implement it thusly:

然后，这样实现它

```c

// chunk.c, create new file

#include <stdlib.h>

#include "chunk.h"

void initChunk(Chunk* chunk) {
  chunk->count = 0;
  chunk->capacity = 0;
  chunk->code = NULL;
}


```


The dynamic array starts off completely empty. We don’t even allocate a raw array yet. To append a byte to the end of the chunk, we use a new function.

动态数组一开始是完全空的，我们甚至都没有分配原始数组，要将一个字节追加到块的末尾，我们使用一个新的函数

```c

// chunk.h, add after initChunk()

void initChunk(Chunk* chunk);
void writeChunk(Chunk* chunk, uint8_t byte);

#endif

```

This is where the interesting work happens.

接下来将是有趣的地方

```c 

// chunk.c, add after initChunk()

void writeChunk(Chunk* chunk, uint8_t byte) {
  if (chunk->capacity < chunk->count + 1) {
    int oldCapacity = chunk->capacity;
    chunk->capacity = GROW_CAPACITY(oldCapacity);
    chunk->code = GROW_ARRAY(uint8_t, chunk->code,
        oldCapacity, chunk->capacity);
  }

  chunk->code[chunk->count] = byte;
  chunk->count++;
}

```


The first thing we need to do is see if the current array already has capacity for the new byte. If it doesn’t, then we first need to grow the array to make room. (We also hit this case on the very first write when the array is NULL and capacity is 0.)

To grow the array, first we figure out the new capacity and grow the array to that size. Both of those lower-level memory operations are defined in a new module.

我们要做的第一件事是，查看当前数组是否还有新字节的添加空间，

如果没有空间，我们需要扩展数组，腾出空间（当数组为NULL，容量为0，时候，我们也会遇到这种问题）

要扩展数组，我们需要计算新数组的容量，将数组大小扩展到该大小，这两个较低级别的内存操作，我们将在一个新模块中定义。

```c

// chunk.c

#include "chunk.h"
#include "memory.h"

void initChunk(Chunk* chunk) {

```

This is enough to get us started.


```c

// memory.h, create new file

#ifndef clox_memory_h
#define clox_memory_h

#include "common.h"

#define GROW_CAPACITY(capacity) \
    ((capacity) < 8 ? 8 : (capacity) * 2)

#endif


```

This macro calculates a new capacity based on a given current capacity. In order to get the performance we want, the important part is that it scales based on the old size. We grow by a factor of two, which is pretty typical. 1.5× is another common choice.

We also handle when the current capacity is zero. In that case, we jump straight to eight elements instead of starting at one. That avoids a little extra memory churn when the array is very small, at the expense of wasting a few bytes on very small chunks.

这个宏根据当前给定的容量，计算一个新容量。为了得到我们想要的性能，重要的部分是它基于旧容量进行缩放，我们按照两倍增长，这是相当典型的，1.5倍是另外一个常见的选择。

我们还将处理当前容量为0的场景，在这种情况下，我们直接跳到8个元素，而不是从一个元素开始，这避免了在数组非常小的情况下，多余的内存波动，代价是在非常小的块上浪费了一些字节。

> I picked the number eight somewhat arbitrarily for the book. Most dynamic array implementations have a minimum threshold like this. The right way to pick a value for this is to profile against real-world usage and see which constant makes the best performance trade-off between extra grows versus wasted space.
> 
> 我在本书中，选择数组最小长度为8，有些随意，大多数的动态数组实现，都有这样的最小阈值。选择这个值的正确方法是针对实际使用情况进行分析，并查看哪个常量在额外增长和浪费空间之间，取得最佳性能平衡。

Once we know the desired capacity, we create or grow the array to that size using GROW_ARRAY().

一旦我们知道所需容量的大小，我们使用GROW_ARRAY() 创建或者增长数组到该大小

```c

// memory.h

#define GROW_CAPACITY(capacity) \
    ((capacity) < 8 ? 8 : (capacity) * 2)

#define GROW_ARRAY(type, pointer, oldCount, newCount) \
    (type*)reallocate(pointer, sizeof(type) * (oldCount), \
        sizeof(type) * (newCount))

void* reallocate(void* pointer, size_t oldSize, size_t newSize);

#endif

```

This macro pretties up a function call to reallocate() where the real work happens. The macro itself takes care of getting the size of the array’s element type and casting the resulting void* back to a pointer of the right type.

这个宏美化了对 reallocate() 函数的调用，实际的工作发生在这里。宏本身负责获取数组元素类型的大小并将结果 void* 强制转化为正确类型的指针。

This reallocate() function is the single function we’ll use for all dynamic memory management in clox—allocating memory, freeing it, and changing the size of an existing allocation. Routing all of those operations through a single function will be important later when we add a garbage collector that needs to keep track of how much memory is in use.

The two size arguments passed to reallocate() control which operation to perform:

|oldSize|	newSize|	Operation|
|---|---|---|
|0	|Non‑zero	|Allocate new block.|
|Non‑zero|	0	|Free allocation.|
|Non‑zero|	Smaller than oldSize|	Shrink existing allocation.|
|Non‑zero|	Larger than oldSize|	Grow existing allocation.|


这个reallocate 函数是我们在clox 中用于所有动态内存管理的唯一函数——分配内存、释放它、更改现有分配的大小。将所有这些操作路由到一个函数中，在稍后添加需要跟踪使用多少内存的垃圾回收器时，将非常重要。

传递给reallocate() 的两个参数控制将要执行的操作





|旧的size| 新的size| 操作|
|---|---|---|
|0| 非0| 分配新块|
|非0|0| 释放块|
|非0| 新的size < 旧的size|缩小现有的分配|
|非0| 新的size > 旧的size|增加现有分配|


That sounds like a lot of cases to handle, but here’s the implementation:

听起来，好像要处理很多的场景，但这是实现

```c

// memory.c, create new file


#include <stdlib.h>

#include "memory.h"

void* reallocate(void* pointer, size_t oldSize, size_t newSize) {
  if (newSize == 0) {
    free(pointer);
    return NULL;
  }

  void* result = realloc(pointer, newSize);
  return result;
}


```

When newSize is zero, we handle the deallocation case ourselves by calling free(). Otherwise, we rely on the C standard library’s realloc() function. That function conveniently supports the other three aspects of our policy. When oldSize is zero, realloc() is equivalent to calling malloc().

The interesting cases are when both oldSize and newSize are not zero. Those tell realloc() to resize the previously allocated block. If the new size is smaller than the existing block of memory, it simply updates the size of the block and returns the same pointer you gave it. If the new size is larger, it attempts to grow the existing block of memory.

It can do that only if the memory after that block isn’t already in use. If there isn’t room to grow the block, realloc() instead allocates a new block of memory of the desired size, copies over the old bytes, frees the old block, and then returns a pointer to the new block. Remember, that’s exactly the behavior we want for our dynamic array.

Because computers are finite lumps of matter and not the perfect mathematical abstractions computer science theory would have us believe, allocation can fail if there isn’t enough memory and realloc() will return NULL. We should handle that.

当newSize为0时，我们通过调用free() 释放空间。否则，我们将依赖C标准库的 realloc() 函数。该函数很方便的支持我们的另外三种情况。当oldSize 为0时， realloc() 等效于 malloc()

有趣的场景是 newSize 和 oldSize 都不等于0时，这告诉 realloc() 调整先前分配块的大小。

* 如果newSize < oldSize, 则它只会更新块的大小，返回我们提供的相同的指针，

* 如果 newSize > oldSize, 则它会尝试增加现有块的大小

它只能这样做，前提是块后面的内存还没有被使用，如果没有足够的空间增加块，则realloc() 会分配一个所需大小的新内存块，复制新字节，释放旧块，然后返回指向新块的指针，记住，这正是我们想要的动态数组的行为

因为计算机是有限资源，而不是计算机科学理论所希望的完美数学抽象，如果没有足够的内存，分配可能会失败，realloc() 可能会返回NULL, 我们应该处理这种情况

```c

// memory.c, in reallocate()

  void* result = realloc(pointer, newSize);
  if (result == NULL) exit(1);
  return result;


```

There’s not really anything useful that our VM can do if it can’t get the memory it needs, but we at least detect that and abort the process immediately instead of returning a NULL pointer and letting it go off the rails later.

> Since all we passed in was a bare pointer to the first byte of memory, what does it mean to “update” the block’s size? Under the hood, the memory allocator maintains additional bookkeeping information for each block of heap-allocated memory, including its size.
>
> Given a pointer to some previously allocated memory, it can find this bookkeeping information, which is necessary to be able to cleanly free it. It’s this size metadata that realloc() updates.
>
> Many implementations of malloc() store the allocated size in memory right before the returned address.
> 
> 由于我们传递的仅仅是指向内存第一个字节的裸指针，那么更新块的大小意味着什么呢？在底层，内存分配器为每一个堆分配的内存块维护额外的记录信息，包括其大小
> 
> 给定指向先前分配的内存的指针，它可以找到该内存块的记录信息，这是能够干净的释放内存的必要条件，realloc() 就是更新这个大小元数据
>
> 许多malloc() 实现，将已经分配的大小存储在返回地址前面的内存中


如果我们无法获取所需的内存，我们的虚拟机实际上，不能执行任何有用的操作，但我们至少会检测到并且立即终止进程，而不是返回一个NULL指针，并且在这之后，让它偏离正轨。

OK, we can create new chunks and write instructions to them. Are we done? Nope! We’re in C now, remember, we have to manage memory ourselves, like in Ye Olden Times, and that means freeing it too.

好的，我们可以创建新的 chunk，并且将指令写入其中。我们完成了吗？并没有，现在我们使用的是C，必须自己管理内存，就像是以前的时代一样，我们需要释放它。

```c 
// chunk.h, add after initChunk()

void initChunk(Chunk* chunk);
void freeChunk(Chunk* chunk);
void writeChunk(Chunk* chunk, uint8_t byte);

```

The implementation is:

```c

// chunk.c, add after initChunk()

void freeChunk(Chunk* chunk) {
  FREE_ARRAY(uint8_t, chunk->code, chunk->capacity);
  initChunk(chunk);
}


```

We deallocate all of the memory and then call initChunk() to zero out the fields leaving the chunk in a well-defined empty state. To free the memory, we add one more macro.

我们释放所有内存，然后调用 initChunk() 来清零字段，将chunk 定义为一个良好的空状态，为了释放内存，我们添加一个额外的宏

```c

// memory.h

#define GROW_ARRAY(type, pointer, oldCount, newCount) \
    (type*)reallocate(pointer, sizeof(type) * (oldCount), \
        sizeof(type) * (newCount))

#define FREE_ARRAY(type, pointer, oldCount) \
    reallocate(pointer, sizeof(type) * (oldCount), 0)

void* reallocate(void* pointer, size_t oldSize, size_t newSize);

```

Like GROW_ARRAY(), this is a wrapper around a call to reallocate(). This one frees the memory by passing in zero for the new size. I know, this is a lot of boring low-level stuff. Don’t worry, we’ll get a lot of use out of these in later chapters and will get to program at a higher level. Before we can do that, though, we gotta lay our own foundation.

像是 GROW_ARRAY() ,这是对reallocate() 调用的包装，通过将newSize 设置为0来释放空间，我知道，这是很多低级别的无聊工作，但是，不用担心，我们将在后面的章节中，大量使用这些，并将在较高层次上进行编程。但在那之前，我们必须打下自己的基础。

## 四、Disassembling Chunks

Now we have a little module for creating chunks of bytecode. Let’s try it out by hand-building a sample chunk.

现在我们已经有了一个创建字节码块的小模块，让我们手工创建一个示例块

```c
// main.c, in main()

int main(int argc, const char* argv[]) {
  Chunk chunk;
  initChunk(&chunk);
  writeChunk(&chunk, OP_RETURN);
  freeChunk(&chunk);
  return 0;


```

Don’t forget the include.

```c

// main.c

#include "common.h"
#include "chunk.h"

int main(int argc, const char* argv[]) {

```

Run that and give it a try. Did it work? Uh . . . who knows? All we’ve done is push some bytes around in memory. We have no human-friendly way to see what’s actually inside that chunk we made.

To fix this, we’re going to create a disassembler. An assembler is an old-school program that takes a file containing human-readable mnemonic names for CPU instructions like “ADD” and “MULT” and translates them to their binary machine code equivalent. A disassembler goes in the other direction—given a blob of machine code, it spits out a textual listing of the instructions.

We’ll implement something similar. Given a chunk, it will print out all of the instructions in it. A Lox user won’t use this, but we Lox maintainers will certainly benefit since it gives us a window into the interpreter’s internal representation of code.

运行并试试，它是否起作用？嗯，谁知道吗？我们所做的只是在内存中移动一些字节，还没有更加好的方式可以查看，程序中创建的块的内容

为了解决这个问题，我们将创建一个反汇编器，汇编器是一种古老的程序，它接受一个人类可读的CPU 指令助记符名称（例如: ADD MULT等）的文件，并且将它们转换为二进制的机器码形式。反汇编器则相反——给定一块机器码，它可以输出指令的文本列表

我们将实现类似的东西，给定一个块，它将打印出其中的所有指令，Lox用户不会使用它，但是，我们的Lox维护者会用到。因为，它给了我们查看解释器代码的内部表示的窗口

> In jlox, our analogous tool was the AstPrinter class.
> 
> 在jlox中，类似的工具是AstProinter 类。


In main(), after we create the chunk, we pass it to the disassembler.

在main() 函数中，我们创建好chunk后，将其传递给了 反汇编器

```c

// main.c, in main()

  initChunk(&chunk);
  writeChunk(&chunk, OP_RETURN);

  disassembleChunk(&chunk, "test chunk");
  freeChunk(&chunk);
  
```

Again, we whip up yet another module.

我们将再次创建一个新的模块

```c

// main.c

#include "chunk.h"
#include "debug.h"

int main(int argc, const char* argv[]) {

```

Here’s that header:

```c

// debug.h, create new file

#ifndef clox_debug_h
#define clox_debug_h

#include "chunk.h"

void disassembleChunk(Chunk* chunk, const char* name);
int disassembleInstruction(Chunk* chunk, int offset);

#endif


```

> I promise you we won’t be creating this many new files in later chapters.
> 
> 我向你保证，后面的章节中，不会创建这么多的新文件

In main(), we call disassembleChunk() to disassemble all of the instructions in the entire chunk. That’s implemented in terms of the other function, which just disassembles a single instruction. It shows up here in the header because we’ll call it from the VM in later chapters.

Here’s a start at the implementation file:

在main() 函数中，我们调用 disassembleChunk() 来反汇编这个chunk 中的所有指令，这是通过其他函数来实现的， 它只能反汇编单个指令。它在这里出现在头文件中，因为我们在后面的章节中，会从VM 中调用这个函数

下面是实现的开头

```c

// debug.c, create new file

#include <stdio.h>

#include "debug.h"

void disassembleChunk(Chunk* chunk, const char* name) {
  printf("== %s ==\n", name);

  for (int offset = 0; offset < chunk->count;) {
    offset = disassembleInstruction(chunk, offset);
  }
}

```

To disassemble a chunk, we print a little header (so we can tell which chunk we’re looking at) and then crank through the bytecode, disassembling each instruction. The way we iterate through the code is a little odd. Instead of incrementing offset in the loop, we let disassembleInstruction() do it for us. When we call that function, after disassembling the instruction at the given offset, it returns the offset of the next instruction. This is because, as we’ll see later, instructions can have different sizes.

The core of the “debug” module is this function:

为了反汇编一个代码块，我们首先打印一个小标题，（这样我们就可以知道我们在查看哪一个代码块），然后遍历字节码并且反汇编每个指令，我们迭代代码的方式有点奇怪，我们不是在循环中递增偏移量，而是让disassembleInstruction() 函数为我们完成这个任务，当我们调用该函数时候，在反汇编给定偏移量处的指令后，它会返回下一条指令的偏移量，这是因为，正如我们稍后将看到的，指令的大小可能不相同

debug模块的核心是这个函数

```c
// debug.c, add after disassembleChunk()

int disassembleInstruction(Chunk* chunk, int offset) {
  printf("%04d ", offset);

  uint8_t instruction = chunk->code[offset];
  switch (instruction) {
    case OP_RETURN:
      return simpleInstruction("OP_RETURN", offset);
    default:
      printf("Unknown opcode %d\n", instruction);
      return offset + 1;
  }
}

```

First, it prints the byte offset of the given instruction—that tells us where in the chunk this instruction is. This will be a helpful signpost when we start doing control flow and jumping around in the bytecode.

Next, it reads a single byte from the bytecode at the given offset. That’s our opcode. We switch on that. For each kind of instruction, we dispatch to a little utility function for displaying it. On the off chance that the given byte doesn’t look like an instruction at all—a bug in our compiler—we print that too. For the one instruction we do have, OP_RETURN, the display function is:

首先，它会打印给定指令的字节偏移量——这告诉我们这个指令在代码块中的位置，当我们开始进行控制流和字节码跳转时候，这将是一个有用的标志。

接下来，它从给定偏移量处的字节码中读取一个字节，这就是我们的操作码，我们对操作码进行 switch语句，对于每种类型的指令，我们会调用一个小的实用程序来显示。万一给定的字节根本不是一个指令——这是编译器中的一个错误——我们也会打印出来，对于我们唯一拥有的指令OP_RETURN, 显示函数是

```c

// debug.c, add after disassembleChunk()

static int simpleInstruction(const char* name, int offset) {
  printf("%s\n", name);
  return offset + 1;
}

```

> We have only one instruction right now, but this switch will grow throughout the rest of the book.
> 
> 我们现在只有一条指令，但是，在本书的其余部分，这个switch 语句，会不断增长。

There isn’t much to a return instruction, so all it does is print the name of the opcode, then return the next byte offset past this instruction. Other instructions will have more going on.

If we run our nascent interpreter now, it actually prints something:

return指令，并没有太多内容，因此它只是打印操作码的名称，然后，返回跳过return 指令的下一个字节偏移量。其他指令将会有更多的内容。

如果我们现在运行 clox解释器，它会打印出一些东西

```c

== test chunk ==
0000 OP_RETURN

```

It worked! This is sort of the “Hello, world!” of our code representation. We can create a chunk, write an instruction to it, and then extract that instruction back out. Our encoding and decoding of the binary bytecode is working.

它起作用了，这有点像是我们代码表示法 的"hello, world!" 我们可以创建一个代码块，向其中写入一个指令，然后将该指令提取出来，我们的二进制字节码的编码和解码正在工作。

## 五、Constants

Now that we have a rudimentary chunk structure working, let’s start making it more useful. We can store code in chunks, but what about data? Many values the interpreter works with are created at runtime as the result of operations.

现在我们有了一个基本的代码块结构，让我们开始让它更加有用，我们可以在代码块中存储代码，但是数据呢？许多解释器处理的值是在运行时，作为操作结果创建的

```
1 + 2;

```

The value 3 appears nowhere in the code here. However, the literals 1 and 2 do. To compile that statement to bytecode, we need some sort of instruction that means “produce a constant” and those literal values need to get stored in the chunk somewhere. In jlox, the Expr.Literal AST node held the value. We need a different solution now that we don’t have a syntax tree.

代码中没有出现3， 然而，字面量1 和 2 出现了，为了将该语句编译为字节码，我们需要某种指令，表示生成一个常量，并且这些字面值需要被存储到代码中的某个地方，在jlox 中，Expr.Literal 节点，保存了该值。由于现在我们没有语法树，我们需要一个不同的解决方案

### 5.1 Representing values

表示值

We won’t be running any code in this chapter, but since constants have a foot in both the static and dynamic worlds of our interpreter, they force us to start thinking at least a little bit about how our VM should represent values.

For now, we’re going to start as simple as possible—we’ll support only double-precision, floating-point numbers. This will obviously expand over time, so we’ll set up a new module to give ourselves room to grow.

在本章中，我们不会运行任何代码，但是，由于常量同时涉及到解释器的静态和动态世界，它迫使我们开始考虑如何表示值

现在，我们将尽可能简单的开始——我们仅仅支持 double 类型，随着时间的扩展，这显然会扩展，因此，我们将设置一个新的模块，留出发展空间

```c


// value.h, create new file

#ifndef clox_value_h
#define clox_value_h

#include "common.h"

typedef double Value;

#endif


```

This typedef abstracts how Lox values are concretely represented in C. That way, we can change that representation without needing to go back and fix existing code that passes around values.

Back to the question of where to store constants in a chunk. For small fixed-size values like integers, many instruction sets store the value directly in the code stream right after the opcode. These are called immediate instructions because the bits for the value are immediately after the opcode.

That doesn’t work well for large or variable-sized constants like strings. In a native compiler to machine code, those bigger constants get stored in a separate “constant data” region in the binary executable. Then, the instruction to load a constant has an address or offset pointing to where the value is stored in that section.

Most virtual machines do something similar. For example, the Java Virtual Machine associates a constant pool with each compiled class. That sounds good enough for clox to me. Each chunk will carry with it a list of the values that appear as literals in the program. To keep things simpler, we’ll put all constants in there, even simple integers.

这个 typedef 抽象了Lox中的值在C中的具体表示方式，这样，我们可以更改该表达方式，而不需要回到现有代码来修复传递值的代码

回到将常量存储在代码块中的问题，对于像整数这样固定长度的值，许多指令集将该值直接存储到操作码后面的，代码流中。这些被称为立即指令，因为该值的位，紧跟在操作码后面。

但是，这对于像是字符串这样的大的或者可变长度的常量，并不适用。在本机编译到机器码的情况下，这些更大的常量被存储在二进制可执行文件中的单独的——常量数据，区域中。然后，加载常量的指令，具有指向该部分存储值的地址，或者偏移量

大多数的虚拟机都会做相同的事情，例如：Java虚拟机将常量池与每个编译的类关联起来，这对于clox来说足够好，每个代码块将携带在程序中出现的字面量值的列表，为了使事情更加简单，我们将所有的常量放入其中，甚至是简单的整数

> In addition to needing two kinds of constant instructions—one for immediate values and one for constants in the constant table—immediates also force us to worry about alignment, padding, and endianness. Some architectures aren’t happy if you try to say, stuff a 4-byte integer at an odd address.
> 
> 除了需要两种常量指令（一种用于立即数，另一种用于常量表中的常量）之外，立即数还需要我们关注对齐、填充和字节序的问题，如果你试图将一个4字节整数存储在奇数位地址上，在某些架构中，可能会出现问题。

### 5.2 Value arrays


The constant pool is an array of values. The instruction to load a constant looks up the value by index in that array. As with our bytecode array, the compiler doesn’t know how big the array needs to be ahead of time. So, again, we need a dynamic one. Since C doesn’t have generic data structures, we’ll write another dynamic array data structure, this time for Value.


常量池是值的数组，加载常量的指令，在该数组中通过索引查找该值，与字节码数组一样，编译器事先不知道数组的大小，因此，我们再次需要一个动态数组，由于C 没有通用数据结构，我们将为Value编写另一个动态数组数据结构

```c

// value.h

typedef double Value;

typedef struct {
  int capacity;
  int count;
  Value* values;
} ValueArray;

#endif

```

> Defining a new struct and manipulation functions each time we need a dynamic array of a different type is a chore. We could cobble together some preprocessor macros to fake generics, but that’s overkill for clox. We won’t need many more of these.
> 
> 每次需要不同类型的动态数组时候，都定义一个新的结构体和操作函数，是很繁琐的。我们可以拼凑出一些预处理器宏，来模拟泛型。但这对于clox来说过于繁琐，我们不需要更多这样的操作。


As with the bytecode array in Chunk, this struct wraps a pointer to an array along with its allocated capacity and the number of elements in use. We also need the same three functions to work with value arrays.

和chunk中的字节码数组一样，这个结构体包装了指向数组的指针，以及其分配的容量和正在使用的元素数，我们还需要三个函数来处理值数组。

```c

// value.h, add after struct ValueArray

} ValueArray;

void initValueArray(ValueArray* array);
void writeValueArray(ValueArray* array, Value value);
void freeValueArray(ValueArray* array);

#endif

```


The implementations will probably give you déjà vu. First, to create a new one:

这些实现可能会让你有一种似曾相识的感觉，首先，创建一个新的值数组

```c

// value.c, create new file

#include <stdio.h>

#include "memory.h"
#include "value.h"

void initValueArray(ValueArray* array) {
  array->values = NULL;
  array->capacity = 0;
  array->count = 0;
}

```

Once we have an initialized array, we can start adding values to it.

当我们有了一个已经初始化的数组，我们就可以向其中添加值了

```c 

// value.c, add after initValueArray()

void writeValueArray(ValueArray* array, Value value) {
  if (array->capacity < array->count + 1) {
    int oldCapacity = array->capacity;
    array->capacity = GROW_CAPACITY(oldCapacity);
    array->values = GROW_ARRAY(Value, array->values,
                               oldCapacity, array->capacity);
  }

  array->values[array->count] = value;
  array->count++;
}

```

> Fortunately, we don’t need other operations like insertion and removal.
> 
> 幸运的是，我们不需要其他操作，例如: 插入、删除。

The memory-management macros we wrote earlier do let us reuse some of the logic from the code array, so this isn’t too bad. Finally, to release all memory used by the array:

我们之前编写的内存管理，可以让我们复用在代码数组中，因此，情况并不算太差。最后，要释放数组使用的所有内存。

```c

// value.c, add after writeValueArray()

void freeValueArray(ValueArray* array) {
  FREE_ARRAY(Value, array->values, array->capacity);
  initValueArray(array);
}


```

Now that we have growable arrays of values, we can add one to Chunk to store the chunk’s constants.

现在，我们有了可增长的值数组，我们可以将其添加到 chunk中，用于存储代码块的常量。

```c
// chunk.h, in struct Chunk

  uint8_t* code;
  ValueArray constants;
} Chunk;

```


Don’t forget the include.

```c

// chunk.h

#include "common.h"
#include "value.h"

typedef enum {

```

Ah, C, and its Stone Age modularity story. Where were we? Right. When we initialize a new chunk, we initialize its constant list too.

C  和它的古老的模块化故事，我们在哪里？当我们初始化一个新的代码块时候，我们也初始化它的常量列表


```c 

// chunk.c, in initChunk()

  chunk->code = NULL;
  initValueArray(&chunk->constants);
}


```

Likewise, we free the constants when we free the chunk.

 同样的，当我们释放代码块时候，我们也会释放常量
 
 
```c 

// chunk.c, in freeChunk()

  FREE_ARRAY(uint8_t, chunk->code, chunk->capacity);
  freeValueArray(&chunk->constants);
  initChunk(chunk);


```


Next, we define a convenience method to add a new constant to the chunk. Our yet-to-be-written compiler could write to the constant array inside Chunk directly—it’s not like C has private fields or anything—but it’s a little nicer to add an explicit function.

接下来，我们定义一个方便的方法，将一个新的常量添加到chunk中，我们还没有编写的编译器，可以直接写入chunk内部的常量数组，——这不像是C 有私有字段或者其他东西——但是，添加一个显式的函数会让事情更加美好

```c 


// chunk.h, add after writeChunk()

void writeChunk(Chunk* chunk, uint8_t byte);
int addConstant(Chunk* chunk, Value value);

#endif

```

Then we implement it.

```c

// chunk.c, add after writeChunk()

int addConstant(Chunk* chunk, Value value) {
  writeValueArray(&chunk->constants, value);
  return chunk->constants.count - 1;
}

```

After we add the constant, we return the index where the constant was appended so that we can locate that same constant later.

我们添加了常量后，返回该常量被追加的索引，以便于以后可以定位到同一个常量


### 5.3 Constant instructions

常量指令

We can store constants in chunks, but we also need to execute them. In a piece of code like:

我们可以将常量保存在chunk中，但是，我们还需要执行它们，在下面这样的代码片段中

```c

print 1;
print 2;

```

The compiled chunk needs to not only contain the values 1 and 2, but know when to produce them so that they are printed in the right order. Thus, we need an instruction that produces a particular constant.

编译后的块，不仅需要包含值1和2，还需要知道何时生成它们，以便以正确的顺序打印它们，因此，我们需要一条指令来生成特定的常量

```c

// chunk.h, in enum OpCode

typedef enum {
  OP_CONSTANT,
  OP_RETURN,
	
```

When the VM executes a constant instruction, it “loads” the constant for use. This new instruction is a little more complex than OP_RETURN. In the above example, we load two different constants. A single bare opcode isn’t enough to know which constant to load.

当虚拟机执行常量指令时，它会加载该常量，以供使用，这个新指令比OP_RETURN 复杂一些，在上面的例子中，我们加载了两个不同的常量，一个单独的操作码，不足以知道要加载哪个常量

To handle cases like this, our bytecode—like most others—allows instructions to have operands. These are stored as binary data immediately after the opcode in the instruction stream and let us parameterize what the instruction does.

为了处理这种情况，我们的字节码（像是大多数字节码一样），允许指令具有操作数，这些操作数作为二进制数据，立即存储在指令流的操作码之后，让我们对指令的操作进行参数化

![format](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/14_format.png?raw=true)

> I’m being vague about what it means to “load” or “produce” a constant because we haven’t learned how the virtual machine actually executes code at runtime yet. For that, you’ll have to wait until you get to (or skip ahead to, I suppose) the next chapter.
> 
> 

Each opcode determines how many operand bytes it has and what they mean. For example, a simple operation like “return” may have no operands, where an instruction for “load local variable” needs an operand to identify which variable to load. Each time we add a new opcode to clox, we specify what its operands look like—its instruction format.

每种操作码，确定它有多少操作数字节，以及它们的含义，例如：return 这样简单类型的操作码，可能没有操作数；而 加载本地变量 操作码，还需要一个操作数，用于标识要加载哪个变量；每次像clox 添加新的操作码时候，我们都会指定它们的操作数是什么样的，也就是指令格式

In this case, OP_CONSTANT takes a single byte operand that specifies which constant to load from the chunk’s constant array. Since we don’t have a compiler yet, we “hand-compile” an instruction in our test chunk.

在这种情况下，OP_CONSTANT 指令需要一个单字节操作数，用于指定要从块的常量数组中加载哪个常量，由于我们还没有编译器，因此，我们在测试块中手动编译指令。

```c

// main.c, in main()

  initChunk(&chunk);

  int constant = addConstant(&chunk, 1.2);
  writeChunk(&chunk, OP_CONSTANT);
  writeChunk(&chunk, constant);

  writeChunk(&chunk, OP_RETURN);


```


We add the constant value itself to the chunk’s constant pool. That returns the index of the constant in the array. Then we write the constant instruction, starting with its opcode. After that, we write the one-byte constant index operand. Note that writeChunk() can write opcodes or operands. It’s all raw bytes as far as that function is concerned.

If we try to run this now, the disassembler is going to yell at us because it doesn’t know how to decode the new instruction. Let’s fix that.


我们将常量值本身，添加到块的常量池中，这会返回该常量在数组中的索引，然后，我们编写常量指令，从其操作码开始，之后，我们写入一个字节的常量索引操作数，请注意，writeChunk() 可以写入操作数或者操作码。在该函数看来，都是原始字节

如果我们现在尝试运行，反汇编器会因为，不知道如何解码新指令，而出错，接下来，我们将解决这个问题

```c

// debug.c, in disassembleInstruction()

  switch (instruction) {
    case OP_CONSTANT:
      return constantInstruction("OP_CONSTANT", chunk, offset);
    case OP_RETURN:
		
```

This instruction has a different instruction format, so we write a new helper function to disassemble it.

由于这个指令有不同的指令格式，因此，我们编写一个新的辅助函数，用于反汇编它

```c

// debug.c, add after disassembleChunk()

static int constantInstruction(const char* name, Chunk* chunk,
                               int offset) {
  uint8_t constant = chunk->code[offset + 1];
  printf("%-16s %4d '", name, constant);
  printValue(chunk->constants.values[constant]);
  printf("'\n");
}

```

There’s more going on here. As with OP_RETURN, we print out the name of the opcode. Then we pull out the constant index from the subsequent byte in the chunk. We print that index, but that isn’t super useful to us human readers. So we also look up the actual constant value—since constants are known at compile time after all—and display the value itself too.

This requires some way to print a clox Value. That function will live in the “value” module, so we include that.

这里有更多的内容，和OP_RETURN 指令一样，我们打印操作码的名称，然后，我们从chunk的下一个字节中，提取常量的索引，我们打印该索引，但是对于读者来说，常量的索引，并不非常有用，因此，我们还需要查找实际的常量数值，并且显示该值本身。

由于常量在编译时候，已知，因此，这需要某种方式来打印clox的值，这个函数将存在于value 模块，因此，我们需要引入 value.h

```c

// debug.c

#include "debug.h"
#include "value.h"

void disassembleChunk(Chunk* chunk, const char* name) {

```

Over in that header, we declare:

```c

// value.h, add after freeValueArray()

void freeValueArray(ValueArray* array);
void printValue(Value value);

#endif

```

And here’s an implementation:

```c

// value.c, add after freeValueArray()

void printValue(Value value) {
  printf("%g", value);
}

```

Magnificent, right? As you can imagine, this is going to get more complex once we add dynamic typing to Lox and have values of different types.

Back in constantInstruction(), the only remaining piece is the return value.

这很棒，不是吗？正如你想象的那样，一旦我们在clox中，添加了动态类型，并具有不同类型的值，这将变得更加复杂

回到constantInstruction(), 唯一剩下的部分就是返回值了

```c 

// debug.c, in constantInstruction()

  printf("'\n");
  return offset + 2;
}

```

Remember that disassembleInstruction() also returns a number to tell the caller the offset of the beginning of the next instruction. Where OP_RETURN was only a single byte, OP_CONSTANT is two—one for the opcode and one for the operand.

记住，disassembleInstruction() 还会返回一个数字，告诉调用者下一条指令的偏移量，其中 OP_RETURN 指令只有一个字节，而 OP_CONSTANT 指令有两个字节——一个是操作码，另外一个是操作数

```c

== test chunk ==
0000 OP_CONSTANT         0 '1.2'
0002 OP_RETURN

```


## 六、Line Information

行信息

Chunks contain almost all of the information that the runtime needs from the user’s source code. It’s kind of crazy to think that we can reduce all of the different AST classes that we created in jlox down to an array of bytes and an array of constants. There’s only one piece of data we’re missing. We need it, even though the user hopes to never see it.

块包含了运行时候，从用户源代码中需要的几乎所有信息，在jlox中，我们创建的所有不同的AST类，在clox中，我们都可以简化为一系列的字节和常量数组，这似乎有点疯狂，我们只是缺少一个数据片段，即使用户希望永远不会看到它，我们仍然需要它。

When a runtime error occurs, we show the user the line number of the offending source code. In jlox, those numbers live in tokens, which we in turn store in the AST nodes. We need a different solution for clox now that we’ve ditched syntax trees in favor of bytecode. Given any bytecode instruction, we need to be able to determine the line of the user’s source program that it was compiled from.

There are a lot of clever ways we could encode this. I took the absolute simplest approach I could come up with, even though it’s embarrassingly inefficient with memory. In the chunk, we store a separate array of integers that parallels the bytecode. Each number in the array is the line number for the corresponding byte in the bytecode. When a runtime error occurs, we look up the line number at the same index as the current instruction’s offset in the code array.

当运行时，发生了错误，我们会显示错误的代码行号给用户，在jlox中，这些数字存储在token中，而我们将 token保存在AST类节点中，既然我们已经放弃了语法树，而使用字节码。那么，现在我们需要一个不同的解决方案。针对任何的字节码指令，我们需要能够确定它编译自用户原始程序的行数

有很多好的编码方式可供选择，我选择能想到的最简单的方式，尽管，它在内存使用方面，效率非常低。在块中，我们存储一个与字节码对应的整数数组，数组中的每个数字都是对应于字节码中的字节的行号，当运行报错时候，我们查找与代码数组中，当前指令偏移量相同索引处的行号

To implement this, we add another array to Chunk.

为了实现这一点，我们在chunk中添加另一个数组

```c

// chunk.h, in struct Chunk

  uint8_t* code;
  int* lines;
  ValueArray constants;
	
```

Since it exactly parallels the bytecode array, we don’t need a separate count or capacity. Every time we touch the code array, we make a corresponding change to the line number array, starting with initialization.

由于它和字节码数组完全相对应，因此我们不需要单独的计数或者容量字段，每一次我们操作代码数组时候，都会进行相应的更改，更新行号数组，从初始化开始。

```c

// chunk.c, in initChunk()

  chunk->code = NULL;
  chunk->lines = NULL;
  initValueArray(&chunk->constants);
	
```

And likewise deallocation:

同样，在释放内存时候

```c

// chunk.c, in freeChunk()

  FREE_ARRAY(uint8_t, chunk->code, chunk->capacity);
  FREE_ARRAY(int, chunk->lines, chunk->capacity);
  freeValueArray(&chunk->constants);
	
```

When we write a byte of code to the chunk, we need to know what source line it came from, so we add an extra parameter in the declaration of writeChunk().

当我们像chunk中写入一个字节的代码时候，我们需要知道它来自于哪一行源代码，因此我们在，声明writeChunk()时候，添加了一个额外的参数

```c

// chunk.h, function writeChunk(), replace 1 line

void freeChunk(Chunk* chunk);
void writeChunk(Chunk* chunk, uint8_t byte, int line);
int addConstant(Chunk* chunk, Value value);

```

And in the implementation:

```c

// chunk.c, function writeChunk(), replace 1 line

void writeChunk(Chunk* chunk, uint8_t byte, int line) {
  if (chunk->capacity < chunk->count + 1) {


```

When we allocate or grow the code array, we do the same for the line info too.

当我们分配或者扩展代码数组时候，我们也会对行信息，提供相同的操作

```c

// chunk.c, in writeChunk()

    chunk->code = GROW_ARRAY(uint8_t, chunk->code,
        oldCapacity, chunk->capacity);
    chunk->lines = GROW_ARRAY(int, chunk->lines,
        oldCapacity, chunk->capacity);
  }


```

Finally, we store the line number in the array.

```c

// chunk.c, in writeChunk()

  chunk->code[chunk->count] = byte;
  chunk->lines[chunk->count] = line;
  chunk->count++;
	
```


### 6.1 Disassembling line information

反汇编行信息

Alright, let’s try this out with our little, uh, artisanal chunk. First, since we added a new parameter to writeChunk(), we need to fix those calls to pass in some—arbitrary at this point—line number.

好的，让我们实践一下行信息，首先，由于我们需要向wirteChunk() 添加一个新参数，因此我们需要更新一下调用，传递行号信息（此时是任意的）


```c

// main.c, in main(), replace 4 lines

  int constant = addConstant(&chunk, 1.2);
  writeChunk(&chunk, OP_CONSTANT, 123);
  writeChunk(&chunk, constant, 123);

  writeChunk(&chunk, OP_RETURN, 123);

  disassembleChunk(&chunk, "test chunk");


```

Once we have a real front end, of course, the compiler will track the current line as it parses and pass that in.

Now that we have line information for every instruction, let’s put it to good use. In our disassembler, it’s helpful to show which source line each instruction was compiled from. That gives us a way to map back to the original code when we’re trying to figure out what some blob of bytecode is supposed to do. After printing the offset of the instruction—the number of bytes from the beginning of the chunk—we show its source line.

当然，当我们拥有了真正的前端，编译器将会在解析时候，追踪当前行并传递行号。

现在我们的每一个指令都有了行号信息，让我们好好利用它。在我们的反汇编器中，显示每个指令编译源自哪个源代码行，是有用的。这为我们提供了一种，在确定某个字节码应该执行某些操作时候，可以映射回原始代码的方式，在打印指令的偏移量（从块的开头算起的字节数）之后，我们将显示它的源代码行数

```c

// debug.c, in disassembleInstruction()

int disassembleInstruction(Chunk* chunk, int offset) {
  printf("%04d ", offset);
  if (offset > 0 &&
      chunk->lines[offset] == chunk->lines[offset - 1]) {
    printf("   | ");
  } else {
    printf("%4d ", chunk->lines[offset]);
  }

  uint8_t instruction = chunk->code[offset];
	
```

Bytecode instructions tend to be pretty fine-grained. A single line of source code often compiles to a whole sequence of instructions. To make that more visually clear, we show a | for any instruction that comes from the same source line as the preceding one. The resulting output for our handwritten chunk looks like:

字节码指令，通常非常细粒度，一行源代码通常编译为整个指令序列。为了让行信息，更加清晰，我们在任何来自与前一个指令相同源代码行数的指令之间显示一个 | ，下面是，我们运行的结果

```c

== test chunk ==
0000  123 OP_CONSTANT         0 '1.2'
0002   | OP_RETURN


```


We have a three-byte chunk. The first two bytes are a constant instruction that loads 1.2 from the chunk’s constant pool. The first byte is the OP_CONSTANT opcode and the second is the index in the constant pool. The third byte (at offset 2) is a single-byte return instruction.

In the remaining chapters, we will flesh this out with lots more kinds of instructions. But the basic structure is here, and we have everything we need now to completely represent an executable piece of code at runtime in our virtual machine. Remember that whole family of AST classes we defined in jlox? In clox, we’ve reduced that down to three arrays: bytes of code, constant values, and line information for debugging.

This reduction is a key reason why our new interpreter will be faster than jlox. You can think of bytecode as a sort of compact serialization of the AST, highly optimized for how the interpreter will deserialize it in the order it needs as it executes. In the next chapter, we will see how the virtual machine does exactly that.

我们有一个三字节的chunk，前两个字节是一个常量指令，它从chunk的常量池中加载1.2 ， 第一个字节是 OP_CONSTANT 操作码，第二个是常量池中的索引，第三个字节是（在偏移量2处）是一个单字节的返回指令。

在接下来的章节中，我们将使用更多种类的指令来扩展它。但是基本结构已经在这里了，我们现在拥有了在虚拟机中完全表示可执行代码所需的一切，还记得我们在jlox中，定义的整个AST类族吗？在clox中，我们将其简化为三个数组: 代码字节数组、常量值数组、用于调试的行信息数组

这种简化是我们的新解释器，比jlox更快的最重要原因之一，你可以将字节码视为AST的一种紧凑的序列化形式，高度优化，以便于解释器按照需要执行的顺序，反序列化它。在下一章中，我们将看到虚拟机如何完全做到这一点。
