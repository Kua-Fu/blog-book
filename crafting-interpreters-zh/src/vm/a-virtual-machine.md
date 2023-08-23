# 一个虚拟机


> Magicians protect their secrets not because the secrets are large and important, but because they are so small and trivial. The wonderful effects created on stage are often the result of a secret so absurd that the magician would be embarrassed to admit that that was how it was done.
>
> <p align="right"> ——Christopher Priest, The Prestige </p>
>
> 魔术师不会公开他们的秘密，不是因为这些密码很大、很重要，而是因为它们非常微小和琐碎，在舞台上创造出的奇妙效果通常是由一个非常荒谬的秘密产生的，以致于魔术师会感到尴尬，不愿意承认事情是这样的。
>
> <p align="right"> ——《致命魔术》</p>


We’ve spent a lot of time talking about how to represent a program as a sequence of bytecode instructions, but it feels like learning biology using only stuffed, dead animals. We know what instructions are in theory, but we’ve never seen them in action, so it’s hard to really understand what they do. It would be hard to write a compiler that outputs bytecode when we don’t have a good understanding of how that bytecode behaves.

So, before we go and build the front end of our new interpreter, we will begin with the back end—the virtual machine that executes instructions. It breathes life into the bytecode. Watching the instructions prance around gives us a clearer picture of how a compiler might translate the user’s source code into a series of them.


我们花了很多时间讨论如何将程序表示为一系列的字节码指令，这是这感觉就像是用死去的动物来学习生物学一样，理论上，我们知道指令是什么，但是，我们从来都没有见过它们的工作方式，所以，很难理解它们真正的作用。如果，我们不了解字节码的行为方式，编写一个输出字节码的编译器将是困难的

因此，在构建新的解释器的前端之前，我们将从后端开始——执行指令的虚拟机，它赋予字节码生命，观察指令在虚拟机伤的运行，可以让我们更清楚的了解编译器如何将用户的源代码转化为一系列的指令。


## 一、An Instruction Execution Machine

一个指令执行机

The virtual machine is one part of our interpreter’s internal architecture. You hand it a chunk of code—literally a Chunk—and it runs it. The code and data structures for the VM reside in a new module.

虚拟机是我们解释器的内部架构的一部分，你将一些代码（实际上是一个chunk）交给虚拟机，它将会运行代码，虚拟机的代码和数据结构存储在一个新模块中

```c

// vm.h, create new file

#ifndef clox_vm_h
#define clox_vm_h

#include "chunk.h"

typedef struct {
  Chunk* chunk;
} VM;

void initVM();
void freeVM();

#endif
```

As usual, we start simple. The VM will gradually acquire a whole pile of state it needs to keep track of, so we define a struct now to stuff that all in. Currently, all we store is the chunk that it executes.

通常情况下，我们从简单的开始，虚拟机将逐渐获取需要跟踪的整个状态，因此，我们现在定义一个结构体，将其全部装入其中，目前，我们只存储它要执行的代码块。

Like we do with most of the data structures we create, we also define functions to create and tear down a VM. Here’s the implementation:

像我们创建的大多数数据结构一样，我们还定义了用于创建和释放VM的函数，下面是实现的代码

```c

// vm.c, create new file

#include "common.h"
#include "vm.h"

VM vm; 

void initVM() {
}

void freeVM() {
}

```


OK, calling those functions “implementations” is a stretch. We don’t have any interesting state to initialize or free yet, so the functions are empty. Trust me, we’ll get there.

The slightly more interesting line here is that declaration of vm. This module is eventually going to have a slew of functions and it would be a chore to pass around a pointer to the VM to all of them. Instead, we declare a single global VM object. We need only one anyway, and this keeps the code in the book a little lighter on the page.

好吧，这里使用实现来描述上面的函数，有些牵强，我们现在还没有一些有趣的状态需要初始化或者释放，所以，这些函数是空的，相信我，我们会实现它们的。

这里稍微有些有趣的声明 VM 那一行，该模块最终将拥有一堆函数，而将一个指向VM的指针 传递给所有这些函数将会是一项繁琐的任务，因此，我们声明了一个单一全局的VM 对象，我们只需要一个VM对象，这使得代码在书面上更加整洁一些

> The choice to have a static VM instance is a concession for the book, but not necessarily a sound engineering choice for a real language implementation. If you’re building a VM that’s designed to be embedded in other host applications, it gives the host more flexibility if you do explicitly take a VM pointer and pass it around.
>
> That way, the host app can control when and where memory for the VM is allocated, run multiple VMs in parallel, etc.
> 
> What I’m doing here is a global variable, and [everything bad you’ve heard about global variables](http://gameprogrammingpatterns.com/singleton.html) is still true when programming in the large. But when keeping things small for a book . . . 
> 
> 选择使用静态的VM 实例，是本书的需要，但对于真正的语言实现，这并不一定是一个合理的工程选择。如果你正在构建一个VM，该VM被设计为嵌入到其他主机应用程序中，那么，如果你明确的使用VM 指针传递给它，那么主机将拥有更多的灵活性。
>
> 这样，主机应用程序，可以控制VM的内存分配时间 和 位置，并行运行多个VM 等等
> 
> 我在这里所做的是一个全局变量，你听到的关于全局变量的所有负面影响，在大规模编程时候，仍然是正确的，但是当为了一本书，而保持事情简单时...


Before we start pumping fun code into our VM, let’s go ahead and wire it up to the interpreter’s main entrypoint.

在我们开始为VM 添加有趣的代码之前，让我们先将其连接到解释器的主入口点。

```c

// main.c, in main()

int main(int argc, const char* argv[]) {
  initVM();

  Chunk chunk;


```

We spin up the VM when the interpreter first starts. Then when we’re about to exit, we wind it down.

在解释器初次启动时候，我们启动VM，然后在解释器即将退出时候，我们关闭VM

```c
// main.c, in main()

  disassembleChunk(&chunk, "test chunk");
  freeVM();
  freeChunk(&chunk);


```

One last ceremonial obligation:

最后一个仪式性的任务


```c

// main.c

#include "debug.h"
#include "vm.h"

int main(int argc, const char* argv[]) {

```

Now when you run clox, it starts up the VM before it creates that hand-authored chunk from the last chapter. The VM is ready and waiting, so let’s teach it to do something.

现在当你运行clox时候，它会在创建上一章的 chunk之前，启动VM，VM已经准备好并等待命令，那么让我们将它做点什么吧

### 1.1 Executing instructions

The VM springs into action when we command it to interpret a chunk of bytecode.

当我们命令VM解释一段字节码时候，VM就会开始工作


```c

// main.c, in main()

  disassembleChunk(&chunk, "test chunk");
  interpret(&chunk);
  freeVM();


```

This function is the main entrypoint into the VM. It’s declared like so:

这个函数是VM的主入口，它声明如下

```c
//vm.h, add after freeVM()

void freeVM();
InterpretResult interpret(Chunk* chunk);

#endif

```

The VM runs the chunk and then responds with a value from this enum:

VM执行该chunk，然后，通过该枚举类型中的一个值做出回应

```c
// vm.h, add after struct VM

} VM;

typedef enum {
  INTERPRET_OK,
  INTERPRET_COMPILE_ERROR,
  INTERPRET_RUNTIME_ERROR
} InterpretResult;

void initVM();
void freeVM();

```


We aren’t using the result yet, but when we have a compiler that reports static errors and a VM that detects runtime errors, the interpreter will use this to know how to set the exit code of the process.

We’re inching towards some actual implementation.

虽然我们还没有使用该结果，但是，当我们拥有一个报告静态错误的编译器和一个检测运行时错误的VM 时候，解释器将使用这个返回值来设置进程的退出码

我们正朝着一些实际的实现迈进

```c

// vm.c, add after freeVM()

InterpretResult interpret(Chunk* chunk) {
  vm.chunk = chunk;
  vm.ip = vm.chunk->code;
  return run();
}

```

First, we store the chunk being executed in the VM. Then we call run(), an internal helper function that actually runs the bytecode instructions. Between those two parts is an intriguing line. What is this ip business?

As the VM works its way through the bytecode, it keeps track of where it is—the location of the instruction currently being executed. We don’t use a local variable inside run() for this because eventually other functions will need to access it. Instead, we store it as a field in VM.

首先，我们将要执行的chunk 存储在VM中，然后，我们调用run() ，这是一个内部的辅助函数，它实际上运行字节码指令，在这两行代码之间，有一行有趣的代码，这个 ip是什么？

当VM 逐步完成字节码时候，它会跟踪自己的位置——当前正在执行的指令的位置，我们不在run() 函数中使用局部变量来存储这个位置，因为，最终其他函数也需要访问它，相反，我们将其存储为VM的一个字段

> If we were trying to squeeze every ounce of speed out of our bytecode interpreter, we would store ip in a local variable. It gets modified so often during execution that we want the C compiler to keep it in a register.
> 
> 如果我们试图，尽可能的提高字节码解释器的速度，我们会将ip保存在一个局部变量中，由于在执行时候要经常修改ip，我们想让C编译器将其保存在一个寄存器中

```c

// vm.h, in struct VM

typedef struct {
  Chunk* chunk;
  uint8_t* ip;
} VM;


```

Its type is a byte pointer. We use an actual real C pointer pointing right into the middle of the bytecode array instead of something like an integer index because it’s faster to dereference a pointer than look up an element in an array by index.

The name “IP” is traditional, and—unlike many traditional names in CS—actually makes sense: it’s an [instruction pointer](https://en.wikipedia.org/wiki/Program_counter). Almost every instruction set in the world, real and virtual, has a register or variable like this.


ip的类型是一个字节指针，我们使用一个真正的C指针，指向字节码数组的中间，而不是像整数索引，那样查找数组中的元素，因为，通过指针进行解引用，比通过索引查找数组元素要快

IP这个名称是传统的，在CS中，和其他许多传统名称不同的是，它实际上是有意义的，他是指令指针。几乎世界上的每个指令集，无论是实际的，还是虚拟的，都有一个类似的寄存器或者变量

> x86, x64, and the CLR call it “IP”. 68k, PowerPC, ARM, p-code, and the JVM call it “PC”, for program counter.
> 
> x86 x64 CLR架构指令集，将其称为 IP, 68k powerPC ARM p-code JVM 将其称为PC，即程序计数器

We initialize ip by pointing it at the first byte of code in the chunk. We haven’t executed that instruction yet, so ip points to the instruction about to be executed. This will be true during the entire time the VM is running: the IP always points to the next instruction, not the one currently being handled.

我们通过将ip 指向chunk中的第一个代码字节来初始化它，我们还没有执行该指令，因此，ip指向即将执行的指令，在VM运行期间，这将一直保持不变: 

ip指向下一条执行的指令，而不是当前正在处理的指令

The real fun happens in run().

真正有趣的事情发生在 run() 函数中


```c

// vm.c, add after freeVM()

static InterpretResult run() {
#define READ_BYTE() (*vm.ip++)

  for (;;) {
    uint8_t instruction;
    switch (instruction = READ_BYTE()) {
      case OP_RETURN: {
        return INTERPRET_OK;
      }
    }
  }

#undef READ_BYTE
}


```


This is the single most important function in all of clox, by far. When the interpreter executes a user’s program, it will spend something like 90% of its time inside run(). It is the beating heart of the VM.

这是整个clox中，最重要的一个函数，远远超过其他函数，当解释器执行用户的程序时候，它将会在run()函数中花费90% 的时间，这个函数是虚拟机的核心

>Or, at least, it will be in a few chapters when it has enough content to be useful. Right now, it’s not exactly a wonder of software wizardry.
> 
> 或者至少，在几个章节之后，当它拥有足够的内容变得有用时候，它将成为最重要的函数，现在，它还不是一个令人惊叹的软件！

Despite that dramatic intro, it’s conceptually pretty simple. We have an outer loop that goes and goes. Each turn through that loop, we read and execute a single bytecode instruction.

To process an instruction, we first figure out what kind of instruction we’re dealing with. The READ_BYTE macro reads the byte currently pointed at by ip and then advances the instruction pointer. The first byte of any instruction is the opcode. Given a numeric opcode, we need to get to the right C code that implements that instruction’s semantics. This process is called decoding or dispatching the instruction.

尽管有如此夸张的介绍，它在概念上非常简单，我们有一个循环，不断的执行，每次循环，我们会读取并且执行单字节码指令

为了处理指令，我们首先需要确定我们正在处理哪种类型的指令，READ_BYTE() 宏读取指令指针指向的字节，然后将指令指针向前移动，任何指令的第一个字节是操作码，给定一个数字操作码，我们需要找到实现该指令语义的正确的C代码，这个过程被称为解码或则分发指令


> Note that ip advances as soon as we read the opcode, before we’ve actually started executing the instruction. So, again, ip points to the next byte of code to be used.
> 
> 请注意，在我们实际开始执行指令之前，一旦我们读取了操作码，ip就会前进。因此，ip指向下一个要使用的代码字节

We do that process for every single instruction, every single time one is executed, so this is the most performance critical part of the entire virtual machine. Programming language lore is filled with clever techniques to do bytecode dispatch efficiently, going all the way back to the early days of computers.

每一条指令都会经过这个过程，每次执行一条指令都要这么做。因此，这是整个虚拟机中最关键的性能部分，编程语言的传说中充满了技巧，以便有效的执行字节码分发，一直追溯到计算机早期。

> If you want to learn some of these techniques, look up “direct threaded code”, “jump table”, and “computed goto”.
> 
> 如果你想要学习这些技巧，可以查阅 直接线程代码，跳转表，计算跳转，等相关内容


Alas, the fastest solutions require either non-standard extensions to C, or handwritten assembly code. For clox, we’ll keep it simple. Just like our disassembler, we have a single giant switch statement with a case for each opcode. The body of each case implements that opcode’s behavior.

So far, we handle only a single instruction, OP_RETURN, and the only thing it does is exit the loop entirely. Eventually, that instruction will be used to return from the current Lox function, but we don’t have functions yet, so we’ll repurpose it temporarily to end the execution.

可惜的是，最快的解决方案，要么需要非标准的C扩展，要么需要手写汇编代码，对于clox，我们将保持简单，就像是我们的反汇编器一样，我们有一个单独的巨大的switch 语句，每个操作码都有一个case，每个case主体实现了该操作码的行为

到目前为止，我们只处理了一条指令 OP_RETURN,并且它所做的唯一事情是完全退出循环，最终，该指令将用于从当前的lox函数返回，但是，我们还没有函数，因此，我们将暂时使用它来结束执行


Let’s go ahead and support our one other instruction.

让我们继续支持另外一条指令


```c

// vm.c, in run()

    switch (instruction = READ_BYTE()) {
      case OP_CONSTANT: {
        Value constant = READ_CONSTANT();
        printValue(constant);
        printf("\n");
        break;
      }
      case OP_RETURN: {


```


We don’t have enough machinery in place yet to do anything useful with a constant. For now, we’ll just print it out so we interpreter hackers can see what’s going on inside our VM. That call to printf() necessitates an include.

我们还没有足够的机制，用于处理常量，因此，现在我们只是简答的打印出常量，以便于我们的解释器程序员可以看到VM内部发生了什么，这个printf函数调用需要引入新的头文件

```c

// vm.c, add to top of file

#include <stdio.h>

#include "common.h"

```

We also have a new macro to define.

我们还需要定义一个新的宏

```c

// vm.c, in run()

#define READ_BYTE() (*vm.ip++)
#define READ_CONSTANT() (vm.chunk->constants.values[READ_BYTE()])

  for (;;) {
  
```

READ_CONSTANT() reads the next byte from the bytecode, treats the resulting number as an index, and looks up the corresponding Value in the chunk’s constant table. In later chapters, we’ll add a few more instructions with operands that refer to constants, so we’re setting up this helper macro now.

Like the previous READ_BYTE macro, READ_CONSTANT is only used inside run(). To make that scoping more explicit, the macro definitions themselves are confined to that function. We define them at the beginning and—because we care—undefine them at the end.

READ_CONSTANT() 从字节码中读取下一个字节，将得到的数字视为索引，并且在chunk的常量表中查找相应的值，在后面的章节中，我们将添加一些引用常量的操作数，因此，现在设置这个辅助宏

与先前的READ_BYTE()宏一样，READ_CONSTANT()仅在run()内部使用，为了使作用域更加明确，宏定义本身限制在该run() 函数内部，我们在开头定义它们，并且在结尾取消


```c

// vm.c, in run()

#undef READ_BYTE
#undef READ_CONSTANT
}


```

>Undefining these macros explicitly might seem needlessly fastidious, but C tends to punish sloppy users, and the C preprocessor doubly so.
> 
> 显式取消定义这些宏，可能看起来过于细节化，但是，C语言往往会惩罚那些粗心的使用者，C预处理器则更是如此

### 1.2 Execution tracing

执行追踪

If you run clox now, it executes the chunk we hand-authored in the last chapter and spits out 1.2 to your terminal. We can see that it’s working, but that’s only because our implementation of OP_CONSTANT has temporary code to log the value. Once that instruction is doing what it’s supposed to do and plumbing that constant along to other operations that want to consume it, the VM will become a black box. That makes our lives as VM implementers harder.

To help ourselves out, now is a good time to add some diagnostic logging to the VM like we did with chunks themselves. In fact, we’ll even reuse the same code. We don’t want this logging enabled all the time—it’s just for us VM hackers, not Lox users—so first we create a flag to hide it behind.

如果现在运行clox，它将执行我们在上一章手动编写的chunk，并且将1.2输出到终端，我们可以看到它正在工作，但是这只是因为我们的OP_CONSTANT 实现，用临时的代码来记录该值，一旦该指令执行了它应该执行的操作并且将该常量，传递给其他要使用它的操作，虚拟机将变为一个黑盒子，这使得我们作为VM的编写者生活更加困难

为了帮助自己，现在是添加一些诊断日志到VM的好时机，将像是我们上一章中，对chunk做的一样，实际上，我们甚至会重用相同的代码，我们不希望始终启用此日志记录——它只是为我们的VM程序员，而不是Lox用户设计的——因此，首先我们创建一个标志来隐藏它

```c

// common.h

#include <stdint.h>

#define DEBUG_TRACE_EXECUTION

#endif

```

When this flag is defined, the VM disassembles and prints each instruction right before executing it. Where our previous disassembler walked an entire chunk once, statically, this disassembles instructions dynamically, on the fly.

当定义了该标志时，VM将在执行每一个指令之前对其进行反汇编，并且打印。我们之前的反汇编器在静态状态下，遍历整个chunk，而这个反汇编器，则动态的即时反汇编指令。

```c

// vm.c, in run()

  for (;;) {
#ifdef DEBUG_TRACE_EXECUTION
    disassembleInstruction(vm.chunk,
                           (int)(vm.ip - vm.chunk->code));
#endif

    uint8_t instruction;
	
```

Since disassembleInstruction() takes an integer byte offset and we store the current instruction reference as a direct pointer, we first do a little pointer math to convert ip back to a relative offset from the beginning of the bytecode. Then we disassemble the instruction that begins at that byte.

As ever, we need to bring in the declaration of the function before we can call it.

由于disassembleInstruction() 接受一个整数字节偏移量，而我们将当前指令引用存储为直接指针，因此，我们首先进行一些指针数学运算，将ip转换回从字节码开头开始的相对偏移量，然后，我们反汇编从该字节开始的指令

像往常一样，在调用它之前，我们需要引入该函数的声明

```c
// vm.c

#include "common.h"
#include "debug.h"
#include "vm.h"

```

I know this code isn’t super impressive so far—it’s literally a switch statement wrapped in a for loop but, believe it or not, this is one of the two major components of our VM. With this, we can imperatively execute instructions. Its simplicity is a virtue—the less work it does, the faster it can do it. Contrast this with all of the complexity and overhead we had in jlox with the Visitor pattern for walking the AST.

我知道到目前为止，这段代码并不是非常令人印象深刻——它只是一个包装在for循环中的switch语句，但是不管你信不信，这就是我们的VM的两个最主要组成部分之一，有了这个，我们可以命令式的执行指令，他的简单性是一种美德——所做的事情越少，就可以越快的完成，与jlox中利用Visitor模式遍历AST所具有的所有复杂性与开销形成对比

## 二、A Value Stack Manipulator

值栈操作器

In addition to imperative side effects, Lox has expressions that produce, modify, and consume values. Thus, our compiled bytecode needs a way to shuttle values around between the different instructions that need them. For example:

除了命令式的副作用，Lox还有产生、修改、消耗值的表达式，因此，我们编写的字节码需要一种方式在需要它们的不同指令之间传递值，例如:

```c

print 3 - 2;

```

We obviously need instructions for the constants 3 and 2, the print statement, and the subtraction. But how does the subtraction instruction know that 3 is the minuend and 2 is the subtrahend? How does the print instruction know to print the result of that?

显然，我们需要常量3 和2，打印语句和减法指令，但是，减法指令怎么知道3是被减数，2是减数呢？打印指令，又怎么知道要打印这个结果呢？


> Yes, I did have to look up “subtrahend” and “minuend” in a dictionary. But aren’t they delightful words? “Minuend” sounds like a kind of Elizabethan dance and “subtrahend” might be some sort of underground Paleolithic monument.
> 
> 是的，我不得不在字典中寻找 subtrahend, minued, 但它们不是令人愉快的单词？“Minuend”听起来像是一种伊丽莎白时代的舞蹈，“subtrahend”可能是某种地下史前纪念碑。

To put a finer point on it, look at this thing right here:

更准确的说，看看这个东西


```c

fun echo(n) {
  print n;
  return n;
}

print echo(echo(1) + echo(2)) + echo(echo(4) + echo(5));

```

I wrapped each subexpression in a call to echo() that prints and returns its argument. That side effect means we can see the exact order of operations.

Don’t worry about the VM for a minute. Think about just the semantics of Lox itself. The operands to an arithmetic operator obviously need to be evaluated before we can perform the operation itself. (It’s pretty hard to add a + b if you don’t know what a and b are.) Also, when we implemented expressions in jlox, we decided that the left operand must be evaluated before the right.

我在每一个子表达式中，包装了一个调用echo()的函数，它打印并且返回参数，这个副作用，意味着我们可以看到操作的确切顺序

暂时不要担心VM，只考虑Lox本身的语义，明显，算术运算符的操作数，在执行操作本身之前，需要先进行计算，（如果，你不知道a、b是什么，将很难将它们相加），此外，我们在jlox中实现表达式时候，我们还决定了左操作数必须在右操作数之前计算

> We could have left evaluation order unspecified and let each implementation decide. That leaves the door open for optimizing compilers to reorder arithmetic expressions for efficiency, even in cases where the operands have visible side effects. C and Scheme leave evaluation order unspecified. Java specifies left-to-right evaluation like we do for Lox.
>
> I think nailing down stuff like this is generally better for users. When expressions are not evaluated in the order users intuit—possibly in different orders across different implementations!—it can be a burning hellscape of pain to figure out what’s going on.
>
> 我们也可以不指定计算顺序，让每个实现决定。这为优化编译器在操作数，具有可见副作用的情况下，重新排序算术运算表达式，以提高效率留下了空间，C和Scheme 没有指定计算顺序，Java指定了与我们在实现jlox中的从左到右计算，相同的计算顺序
>
> 我认为明确这些，对于用户更加友好，当表达式未按照用户的直觉顺序计算时候，可能在不同的实现中，以不同的顺序执行，这可能是一个痛苦的场景，需要费尽心思才能弄清楚发生了什么

Here is the syntax tree for the print statement:

这是print语句的语法树

![ast](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/18_ast.png?raw=true)

Given left-to-right evaluation, and the way the expressions are nested, any correct Lox implementation must print these numbers in this order:

给定从左到右的计算顺序，已经表达式嵌套的方式，任何正确的Lox实现都必须按照下面的顺序打印这些数字


```

1  // from echo(1)
2  // from echo(2)
3  // from echo(1 + 2)
4  // from echo(4)
5  // from echo(5)
9  // from echo(4 + 5)
12 // from print 3 + 9

```

Our old jlox interpreter accomplishes this by recursively traversing the AST. It does a postorder traversal. First it recurses down the left operand branch, then the right operand, then finally it evaluates the node itself.

After evaluating the left operand, jlox needs to store that result somewhere temporarily while it’s busy traversing down through the right operand tree. We use a local variable in Java for that. Our recursive tree-walk interpreter creates a unique Java call frame for each node being evaluated, so we could have as many of these local variables as we needed.

In clox, our run() function is not recursive—the nested expression tree is flattened out into a linear series of instructions. We don’t have the luxury of using C local variables, so how and where should we store these temporary values? You can probably guess already, but I want to really drill into this because it’s an aspect of programming that we take for granted, but we rarely learn why computers are architected this way.

之前实现的jlox解释器，通过递归遍历AST 来完成这一点，它执行后序遍历，首先，它递归遍历左操作数分支，然后是右操作数，最后是计算节点本身

在计算左操作数之后，jlox需要将结果暂时存储在某个地方，而同时又需要遍历，右操作数，我们在Java中使用一个本地变量，来实现这一点，我们的递归遍历解析树，为每一个被计算的节点，创建一个唯一的Java调用帧，因此，我们可以根据需要，拥有尽可能多的本地变量

在clox中，我们的run()函数，不是递归的——嵌套的表达式被展开为一系列线性指令，我们没有使用C本地变量的奢侈，因此，我们应该在哪里存储这些临时值？你可能已经猜到了，但我想深入探讨这一点，因为这是编程的一个方面，我们认为是理所当然的，但是，我们很少学习，为什么计算机被设计成这样

>Hint: it’s in the name of this section, and it’s how Java and C manage recursive calls to functions.
> 
> 提示，它在这个部分的名字中，也是Java和C 如何管理对函数集的递归调用的方式

Let’s do a weird exercise. We’ll walk through the execution of the above program a step at a time:

让我们做一个奇怪的练习，我们将逐步执行上面的程序

![bars](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/18_bars.png?raw=true)


On the left are the steps of code. On the right are the values we’re tracking. Each bar represents a number. It starts when the value is first produced—either a constant or the result of an addition. The length of the bar tracks when a previously produced value needs to be kept around, and it ends when that value finally gets consumed by an operation.

As you step through, you see values appear and then later get eaten. The longest-lived ones are the values produced from the left-hand side of an addition. Those stick around while we work through the right-hand operand expression.

In the above diagram, I gave each unique number its own visual column. Let’s be a little more parsimonious. Once a number is consumed, we allow its column to be reused for another later value. In other words, we take all of those gaps up there and fill them in, pushing in numbers from the right:


左边是代码的步骤，右边是我们正在跟踪的值，每个条形图表示一个数字，它从该值首次生成开始——无论是常量，还是加法运算的结果，条形图的长度，表示，跟踪的值何时需要保留，并在最终被操作消耗时，结束

当我们逐步执行时，会看到值出现，然后值被使用，保留时间最长的值，是从加法的左侧产生的值，当我们处理右侧操作数表达式时候，这些值会保留下来

在上面的图表中，我为每一个唯一的数字，都生成了自己的可视列。让我们在节约一点，一旦数字被消耗，我们允许其列被重用于稍后的另一个值，换句话说，我们会将上面的所有间隙填满，从右侧插入数字

![bars-stacked](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/18_bars-stacked.png?raw=true)

There’s some interesting stuff going on here. When we shift everything over, each number still manages to stay in a single column for its entire life. Also, there are no gaps left. In other words, whenever a number appears earlier than another, then it will live at least as long as that second one. The first number to appear is the last to be consumed. Hmm . . . last-in, first-out . . . why, that’s a stack!

这里有一些有趣的事情，当我们将所有内容向左移动时候，每个数字，仍然在其整个生命周期内保持在单个列中。而且，没有留下任何间隙，换一句话说，每当一个数字出现在另一个数字之前时候，它将至少与第二个数字一样长，第一个出现的数字是最后被消耗的，嗯，后进先出，为什么？这个一个栈！

> This is also a stack:
> 
> ![pancakes](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/18_pancakes.png?raw=true)


In the second diagram, each time we introduce a number, we push it onto the stack from the right. When numbers are consumed, they are always popped off from rightmost to left.

Since the temporary values we need to track naturally have stack-like behavior, our VM will use a stack to manage them. When an instruction “produces” a value, it pushes it onto the stack. When it needs to consume one or more values, it gets them by popping them off the stack.


在第二个图表中，每次我们引入一个数字时候，我们从右侧将其推入栈中，当数字被消耗时，它们总是从最右端到最左端弹出

由于我们需要跟踪的临时值，具有堆栈式的行为，因此，我们的虚拟机将使用堆栈管理它们，当一条指令产生一个值时候，VM会将其推入栈中，当VM需要消耗一个或者多个值时候，从栈中弹出它们，并且获取。

### 2.1 The VM’s Stack

Maybe this doesn’t seem like a revelation, but I love stack-based VMs. When you first see a magic trick, it feels like something actually magical. But then you learn how it works—usually some mechanical gimmick or misdirection—and the sense of wonder evaporates. There are a couple of ideas in computer science where even after I pulled them apart and learned all the ins and outs, some of the initial sparkle remained. Stack-based VMs are one of those.

也许这并不算一个惊人的发现，但是我喜欢基于堆栈的虚拟机，当你第一次看到一个魔术表演时候，它感觉像是真正的魔法，但是，当你学会课它是如何工作后——通常是一些机械装置或者误导——那种惊奇感就消失了，在计算机科学中，有一些想法，即使我将它们分解并且，学会了所有的细节，它们最初的魅力仍然存在，基于堆栈的虚拟机就是其中之一。


> Heaps—[the data structure](https://en.wikipedia.org/wiki/Heap_(data_structure)), not [the memory management](https://en.wikipedia.org/wiki/Memory_management#HEAP) thing—are another. And Vaughan Pratt’s top-down operator precedence parsing scheme, which we’ll learn about in due time.
> 
> 堆——数据结构，不是内存管理的东西——是另一个。还有 Vaughan Pratt的自顶向下的运算符优先级解析方案，我们会适时的了解它

As you’ll see in this chapter, executing instructions in a stack-based VM is dead simple. In later chapters, you’ll also discover that compiling a source language to a stack-based instruction set is a piece of cake. And yet, this architecture is fast enough to be used by production language implementations. It almost feels like cheating at the programming language game.
