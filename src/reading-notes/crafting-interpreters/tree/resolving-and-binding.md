# 解析与绑定

Resolving and Binding

> Once in a while you find yourself in an odd situation. You get into it by degrees and in the most natural way but, when you are right in the midst of it, you are suddenly astonished and ask yourself how in the world it all came about.
> 
> 偶尔你会发现自己处于一种奇怪的境地，你以一种最自然的方式逐渐进入其中，但是当你置身其中时候，你突然感到惊讶，并问自己这一切到底是怎么发生的
> 
> <p align="right"> Thor Heyerdahl, Kon-Tiki (挪威作家托尔·海尔达尔,  Kon-Tiki探险：穿越南海的筏)</p>

Oh, no! Our language implementation is taking on water! Way back when we added variables and blocks, we had scoping nice and tight. But when we later added closures, a hole opened in our formerly waterproof interpreter. Most real programs are unlikely to slip through this hole, but as language implementers, we take a sacred vow to care about correctness even in the deepest, dampest corners of the semantics.

哦，不，我们实现的语言，现在正在水涨船高，很久以前，当我们添加变量和代码块语法时候，我们的作用域非常严密，但是，当我们后面添加闭包时候，我们以前的解释器，出现了泄漏漏洞。大多数的真正运行的程序，都不太可能遇到这个漏洞，但是作为语言的实现者，我们有一个设计标准，即使在语义的最底层、最薄弱的角落，我们也要关心解释的正确性。


We will spend this entire chapter exploring that leak, and then carefully patching it up. In the process, we will gain a more rigorous understanding of lexical scoping as used by Lox and other languages in the C tradition. We’ll also get a chance to learn about semantic analysis—a powerful technique for extracting meaning from the user’s source code without having to run it.

我们将用一整章来探讨这个漏洞，然后修复它。在这个过程中，我们将更加深入理解Lox和其他类C语言中，所使用的词汇作用域。我们还将有机会学习语义分析，这是一种从用户源代码中提取有用信息的强大技术，而这并不需要代码运行。

## 一、Static Scope

静态作用域

A quick refresher: Lox, like most modern languages, uses lexical scoping. This means that you can figure out which declaration a variable name refers to just by reading the text of the program. For example:

快速复习：与大多数现代语言一样，Lox使用词汇作用域。这意味着，只需要读取程序的文本，就可以找出变量名所值的声明。例如：

```java

var a = "outer";
{
  var a = "inner";
  print a;
}


```

Here, we know that the a being printed is the variable declared on the previous line, and not the global one. Running the program doesn’t—can’t—affect this. The scope rules are part of the static semantics of the language, which is why they’re also called static scope.

上面的代码，我们知道正在打印的a 是上一行声明的变量，而不是全局变量，运行程序不会影响这一点。作用域规则是语言静态语义的一部分，这就是为什么它们被称为静态作用域。

I haven’t spelled out those scope rules, but now is the time for precision:

我们还没有列出这些作用域规则，但是现在，是时候列出规则了

> This is still nowhere near as precise as a real language specification. Those docs must be so explicit that even a Martian or an outright malicious programmer would be forced to implement the correct semantics provided they followed the letter of the spec.
> 
> 这仍然没有真正的语言规范那么精确。这些文档必须非常精确，以至于即使你是火星人或者充满恶意的程序员，也必须实现正确的语义，前提是他们都必须遵循规范
> 
> That exactitude is important when a language may be implemented by competing companies who want their product to be incompatible with the others to lock customers onto their platform. For this book, we can thankfully ignore those kinds of shady shenanigans.
> 
> 当一种语言可能由竞争对手实施时候，这种精确性非常重要，因为他们希望自己的产品和其他产品不兼容，从而将客户锁定在自己的平台上，在这本书中，我们不需要考虑这么多

A variable usage refers to the preceding declaration with the same name in the innermost scope that encloses the expression where the variable is used.

变量的使用规则，变量是指，包含该变量的表达式的作用域中，最内部的具有相同名称的变量。

There’s a lot to unpack in that:


* I say “variable usage” instead of “variable expression” to cover both variable expressions and assignments. Likewise with “expression where the variable is used”.

* “Preceding” means appearing before in the program text.

	```java
	
		var a = "outer";
		{
			print a;
			var a = "inner";
		}
		
	```

	Here, the a being printed is the outer one since it appears before the print statement that uses it. In most cases, in straight line code, the declaration preceding in text will also precede the usage in time. But that’s not always true. As we’ll see, functions may defer a chunk of code such that its dynamic temporal execution no longer mirrors the static textual ordering.

* “Innermost” is there because of our good friend shadowing. There may be more than one variable with the given name in enclosing scopes, as in:

	```java
	
	var a = "outer";
	{
		var a = "inner";
		print a;
	}
	
	```
	
	Our rule disambiguates this case by saying the innermost scope wins.
	
> In JavaScript, variables declared using var are implicitly “hoisted” to the beginning of the block. Any use of that name in the block will refer to that variable, even if the use appears before the declaration. When you write this in JavaScript:
> 
> 在JavaScript中，使用var 声明的变量，隐式的被提升到代码块的开始，块中对该名称的任何引用，都是引用该变量，即使该引用出现在声明之前，例如: 
> 
> ```
> 
> {
>   console.log(a);
>   var a = "value";
> }
> ```
> 
> It behaves like:
> 
> {
>   var a; // Hoist.
>   console.log(a);
>   a = "value";
> }
> 
> That means that in some cases you can read a variable before its initializer has run—an annoying source of bugs. The alternate let syntax for declaring variables was added later to address this problem.
> 
> 这意味着某些情况下，我们可以读取到某个变量，即使它还没有初始化。稍后，我们添加了用于声明变量的let 语法，将解决该问题


还有一些需要东西需要解释

* 我使用名词，变量用法，而不是变量表达式，是为了包含变量表达式和赋值，与“使用变量的表达式”类似

* 前置，意味着声明出现在变量表达式的代码前面


	 ```java


	 var a = "outer";
	 {
		 print a;
		 var a = "inner";
	 }

	 ```

	 上面的代码中，打印的a，表示外部的变量a，因为它出现在使用它的print语句之前。在大多数情况下，在直接式程序中，文本前面的声明也会比变量表达式，更早的运行。但是，这也不总是对的。正如我们将要看到的，函数可能会延迟执行一些代码，这样它的执行就是动态的，不能反映静态文件的代码顺序。

* 最内部是因为我们已经接触的追踪。内部封闭代码中，可能会出现多个给定名称的变量，例如：

	```java
	
	var a = "outer";
	{
		var a = "inner";
		print a;
	}
	
	```

	我们规则，定义了最内层的范围优先级更高，用于消除歧义。

Since this rule makes no mention of any runtime behavior, it implies that a variable expression always refers to the same declaration through the entire execution of the program. Our interpreter so far mostly implements the rule correctly. But when we added closures, an error snuck in.

由于这个规则没有提及任何运行时的行为，因此它意味着变量表达式在程序的整个执行过程中，始终引用相同的声明。到目前为止，我们的解释器基本上是正确执行上面的规则。但是当我们添加闭包时，一个错误悄悄的出现了。


```java

var a = "global";
{
  fun showA() {
    print a;
  }

  showA();
  var a = "block";
  showA();
}

```

Before you type this in and run it, decide what you think it should print.

在你键入代码并且运行之前，你认为会输出什么结果呢？

> I know, it’s a totally pathological, contrived program. It’s just weird. No reasonable person would ever write code like this. Alas, more of your life than you’d expect will be spent dealing with bizarro snippets of code like this if you stay in the programming language game for long.
> 
> 我知道，这个完全是人为构造的代码。这段代码很奇怪，没有一个理智的人会写出这样的代码，哎，如果你在编程语言游戏中呆久了，你的生活中，会出现很多这样的古怪代码需要考虑 

OK . . . got it? If you’re familiar with closures in other languages, you’ll expect it to print “global” twice. The first call to showA() should definitely print “global” since we haven’t even reached the declaration of the inner a yet. And by our rule that a variable expression always resolves to the same variable, that implies the second call to showA() should print the same thing.

Alas, it prints:

```java

global
block

```

好了，现在我们知道了？如果你熟悉其他语言中的闭包，我们会期望两次输出都是 global, 因为我们还没有到达内部的变量a的声明语句，根据上面的规则，变量表达式始终解析为相同的变量，这意味着对于 showA() 的第二次调用应该打印相同的内容

但是，实际输出的，并不是我们期望的

Let me stress that this program never reassigns any variable and contains only a single print statement. Yet, somehow, that print statement for a never-assigned variable prints two different values at different points in time. We definitely broke something somewhere.

需要强调的是，这个程序从不重新分配任何变量，只包含一个print语句。然而，不知道什么原因，对于从未赋值变量的print语句，在不同的时间点打印出两个不同的值。我们肯定是在什么地方出现了异常。

### 1.1 Scopes and mutable environments

作用域和可变环境

In our interpreter, environments are the dynamic manifestation of static scopes. The two mostly stay in sync with each other—we create a new environment when we enter a new scope, and discard it when we leave the scope. There is one other operation we perform on environments: binding a variable in one. This is where our bug lies.

在我们的解释器中，环境是静态的作用域的动态表现，这两者基本上保持同步——我们在进入新的作用域时候，创建新的环境变量，当我们离开作用域后，我们将丢弃对应的环境。在环境中，我们会执行的操作有：将变量绑定到环境中。这就是我们的bug所在

Let’s walk through that problematic example and see what the environments look like at each step. First, we declare a in the global scope.

让我们通过这个有问题的示例，看看每一步中的环境是什么样子，首先，我们在全局范围内声明变量a

![env-1](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environment-1.png?raw=true)

That gives us a single environment with a single variable in it. Then we enter the block and execute the declaration of showA().

我们将创建一个新的环境，其中只有一个变量，然后，我们进入代码块中，执行showA()函数的声明

![env-2](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environment-2.png?raw=true)

We get a new environment for the block. In that, we declare one name, showA, which is bound to the LoxFunction object we create to represent the function. That object has a closure field that captures the environment where the function was declared, so it has a reference back to the environment for the block.

我们为代码块，创建了一个新的环境，在环境中，我们声明了一个名称，showA, 它绑定了到我们创建的LoxFunction 对象，表示一个函数，该对象有一个闭包字段，用于获取声明函数的环境，因此，它有一个对块环境的引用。

Now we call showA().

现在我们开始调用showA() 函数

![env-3](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environment-3.png?raw=true)

The interpreter dynamically creates a new environment for the function body of showA(). It’s empty since that function doesn’t declare any variables. The parent of that environment is the function’s closure—the outer block environment.


The interpreter dynamically creates a new environment for the function body of showA(). It’s empty since that function doesn’t declare any variables. The parent of that environment is the function’s closure—the outer block environment.

解释器为showA() 函数的函数体，动态的创建了一个新环境，该环境是空的，因为该函数没有声明任何变量。该环境的父级环境，是函数的闭包，即外部块环境。

Inside the body of showA(), we print the value of a. The interpreter looks up this value by walking the chain of environments. It gets all the way to the global environment before finding it there and printing "global". Great.

Next, we declare the second a, this time inside the block. 

在showA()的函数体中，我们输出变量a的值。解释器通过遍历环境链来查找该值。它会一直找到global 环境，并且输出字符串"global", 接下来，我们在块内声明了第二个变量a

![env-4](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environment-4.png?raw=true)

It’s in the same block—the same scope—as showA(), so it goes into the same environment, which is also the same environment showA()’s closure refers to. This is where it gets interesting. We call showA() again. 

第二个变量a 和showA() 在同一个块中，作用域相同，因此该变量将进入相同的环境，这也是showA() 的闭包所指的相同环境，这就是它有趣的地方，我们再次调用showA()

![env-5](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environment-5.png?raw=true)

We create a new empty environment for the body of showA() again, wire it up to that closure, and run the body. When the interpreter walks the chain of environments to find a, it now discovers the new a in the block environment. Boo.

我们再次为showA() 的主体创建一个新的空环境，将其连接到该闭包，然后运行主体，当解释器遍历环境链，查找变量a的时候，它会在块环境中发现新的a

I chose to implement environments in a way that I hoped would agree with your informal intuition around scopes. We tend to consider all of the code within a block as being within the same scope, so our interpreter uses a single environment to represent that. Each environment is a mutable hash table. When a new local variable is declared, it gets added to the existing environment for that scope.

我选择了一种实现环境的方式，我希望这种方式能够符合我们对作用域的非正式直觉。我们倾向于认为一个块中的所有代码都在同一个作用域内，因此我们的解释器，使用一个相同的环境，来表示这个作用域。每个环境都是一个可变的哈希表，当声明一个新的局部变量时，这个新的变量将被添加到环境中

That intuition, like many in life, isn’t quite right. A block is not necessarily all the same scope. Consider:

这是生活中，大多数人的直觉，但是这并不完全正确。块不一定都是相同的范围，例如：

```java

{
  var a;
  // 1.
  var b;
  // 2.
}

```

At the first marked line, only a is in scope. At the second line, both a and b are. If you define a “scope” to be a set of declarations, then those are clearly not the same scope—they don’t contain the same declarations. It’s like each var statement splits the block into two separate scopes, the scope before the variable is declared and the one after, which includes the new variable.

在第一行代码，作用域中只有变量a，第二行，a和b 有相同的作用域，如果我们把作用域看作一组声明，那么这些声明语句显然不是同一个作用域——它们不包含相同的声明。就像每个var语句，将块分割为两个独立的作用域，一个是声明变量之前的作用域，另一个是包含新变量的作用域。

But in our implementation, environments do act like the entire block is one scope, just a scope that changes over time. Closures do not like that. When a function is declared, it captures a reference to the current environment. The function should capture a frozen snapshot of the environment as it existed at the moment the function was declared. But instead, in the Java code, it has a reference to the actual mutable environment object. When a variable is later declared in the scope that environment corresponds to, the closure sees the new variable, even though the declaration does not precede the function.

但是在我们的实现中，环境确实像整个块是一个作用域，只是一个随着时间变化的作用域。闭包并不是这样，当声明函数时候，它捕获到一个对当前环境的引用，该函数应该捕获一个当前环境的快照，因为这个环境在声明函数时候，已经存在了。但是，在Java代码中，它引用了实际的可变环境对象，当稍后在环境对应的作用域中，声明新的变量后，闭包可以发现新的变量，即使变量声明在函数声明之前。

### 1.2 Persistent environments

持久化环境

There is a style of programming that uses what are called persistent data structures. Unlike the squishy data structures you’re familiar with in imperative programming, a persistent data structure can never be directly modified. Instead, any “modification” to an existing structure produces a brand new object that contains all of the original data and the new modification. The original is left unchanged.

有一种编程风格，使用所谓的持久数据结构，与命令式编程中熟悉的数据结构不同，持久数据结构永远不能直接修改。相反，对现有结构的任何修改都会产生一个全新的对象，该对象包含所有原始数据和新的更新，原始的数据结构保持不变

> This sounds like it might waste tons of memory and time copying the structure for each operation. In practice, persistent data structures share most of their data between the different “copies”.
> 
> 这听起来可能会浪费大量的内存和时间来复制每个操作的结构，实际上，持久化的数据结构在不同的副本之间共享了大部分的数据

If we were to apply that technique to Environment, then every time you declared a variable it would return a new environment that contained all of the previously declared variables along with the one new name. Declaring a variable would do the implicit “split” where you have an environment before the variable is declared and one after:

如果，我们将上面的技术应用于环境，那么每次我们声明一个新的变量时候，都会返回一个新的环境，其中包含所有先前声明的变量以及一个新名称，声明变量将执行隐式拆分，即在声明变量之前和之后，分别有一个环境

![split](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/split.png?raw=true)

A closure retains a reference to the Environment instance in play when the function was declared. Since any later declarations in that block would produce new Environment objects, the closure wouldn’t see the new variables and our bug would be fixed.

闭包在声明函数时候，保留对环境实例的引用，由于该块中的任何后续声明都会生成新的环境对象，因此，闭包将无法发现新的变量，我们的错误将被修复

This is a legit way to solve the problem, and it’s the classic way to implement environments in Scheme interpreters. We could do that for Lox, but it would mean going back and changing a pile of existing code.

这是解决问题的合法方式，也是在Scheme 解释器实现环境的经典方法，我们可以为Lox这样做，但是这意味着我们要修改一堆现有代码

I won’t drag you through that. We’ll keep the way we represent environments the same. Instead of making the data more statically structured, we’ll bake the static resolution into the access operation itself.

我不会把你拖过去的，我们将保持表示环境的方式不变，我们将把静态解析，迁移到访问操作本身，而不是使数据更加静态结构化。

## 二、Semantic Analysis

语义分析

Our interpreter resolves a variable—tracks down which declaration it refers to—each and every time the variable expression is evaluated. If that variable is swaddled inside a loop that runs a thousand times, that variable gets re-resolved a thousand times.

每次计算变量表达式时候，我们的解释器解析一个变量，跟踪它引用的声明。如果该变量被封装在运行一千次的一个循环中，则该变量将被重新解析一千次。

We know static scope means that a variable usage always resolves to the same declaration, which can be determined just by looking at the text. Given that, why are we doing it dynamically every time? Doing so doesn’t just open the hole that leads to our annoying bug, it’s also needlessly slow.

我们知道静态作用域，意味着变量总是被解析为同一个声明，者可以通过查看名称来确定。既然如此，为什么我们每次还需要动态的解析呢？这样做不仅会引发上面的漏洞，而且速度也会变慢。

A better solution is to resolve each variable use once. Write a chunk of code that inspects the user’s program, finds every variable mentioned, and figures out which declaration each refers to. This process is an example of a semantic analysis. Where a parser tells only if a program is grammatically correct (a syntactic analysis), semantic analysis goes farther and starts to figure out what pieces of the program actually mean. In this case, our analysis will resolve variable bindings. We’ll know not just that an expression is a variable, but which variable it is.

更好的解决方案是解决每个变量使用一次。编写一段代码，检查用户的程序，找到提到的每一个变量，并且找出每个变量引用的声明。这个过程是语义分析的一个例子。当解析器解析一个程序是否语法正确（语法分析）时候，语义分析会更加深入，开始找到程序的实际含义。在这种情况下，我们的分析将解析变量绑定，我们不仅知道表达式是一个变量，还知道它是哪个变量。

There are a lot of ways we could store the binding between a variable and its declaration. When we get to the C interpreter for Lox, we’ll have a much more efficient way of storing and accessing local variables. But for jlox, I want to minimize the collateral damage we inflict on our existing codebase. I’d hate to throw out a bunch of mostly fine code.

有很多方法可以存储变量和变量声明之前的绑定，当我们使用Lox的C解释器时候，我们将有一种更加有效的方法来存储和访问本地变量。但是对于jlox，我想尽量减少我们对现有代码的负面影响，我不想抛弃一堆基本上很好的代码。

Instead, we’ll store the resolution in a way that makes the most out of our existing Environment class. Recall how the accesses of a are interpreted in the problematic example.

相反，我们将以充分利用现有环境类的方式，存储的解决方案。回想一下，在上面的代码中，变量a的解析过程是什么？

![env-3](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environment-3.png?raw=true)

In the first (correct) evaluation, we look at three environments in the chain before finding the global declaration of a. Then, when the inner a is later declared in a block scope, it shadows the global one.

在第一次执行中，我们在找到a的 全局声明之前，先查看环境链中的三个环境，然后，当内部a，在稍后的块作用域中声明后，它会隐藏全局的变量a

![env-5](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environment-5.png?raw=true)

The next lookup walks the chain, finds a in the second environment and stops there. Each environment corresponds to a single lexical scope where variables are declared. If we could ensure a variable lookup always walked the same number of links in the environment chain, that would ensure that it found the same variable in the same scope every time.

下一次的调用，查找遍历环境链，在第二个环境中发现了a，停止继续寻找。每个环境对应于声明的变量的词法作用域。如果我们能够确保变量查找，总是在环境链中遍历相同数量的链接，那么就可以确保每次都在相同的范围内找到相同的变量。


To “resolve” a variable usage, we only need to calculate how many “hops” away the declared variable will be in the environment chain. The interesting question is when to do this calculation—or, put differently, where in our interpreter’s implementation do we stuff the code for it?

为了解决变量的使用，我们只需要计算环境链中声明的变量，将有多少跳，有趣的问题是什么时候进行这个计算，或者换句话说，在我们解释器的实现中，我们在哪里去填充对应的代码？

Since we’re calculating a static property based on the structure of the source code, the obvious answer is in the parser. That is the traditional home, and is where we’ll put it later in clox. It would work here too, but I want an excuse to show you another technique. We’ll write our resolver as a separate pass.

因为我们是基于源代码的结构计算静态属性，所以很明显的答案是解析器。这是传统的方式，我们稍后也会把这个逻辑放入clox，在jlox中同样的工作方式，但是，我将使用另一种技术，我们将把解析器写成一个单独的过程

### 2.1 A variable resolution pass

变量解决管道

After the parser produces the syntax tree, but before the interpreter starts executing it, we’ll do a single walk over the tree to resolve all of the variables it contains. Additional passes between parsing and execution are common. If Lox had static types, we could slide a type checker in there. Optimizations are often implemented in separate passes like this too. Basically, any work that doesn’t rely on state that’s only available at runtime can be done in this way.

在解析器生成语法树之后，但是在解释器开始执行之前，我们将对语法树进行一次遍历，解析语法树包含的所有变量。解析和执行之间添加新的处理管道是正常的。如果Lox中存在静态类型，我们可以在管道中执行类型检测。优化，也经常发生在这样的单独管道中。基本上，任何不依赖于，仅在运行时可用的状态的工作，都可以通过这种方式（添加管道）来完成

Our variable resolution pass works like a sort of mini-interpreter. It walks the tree, visiting each node, but a static analysis is different from a dynamic execution:


* There are no side effects. When the static analysis visits a print statement, it doesn’t actually print anything. Calls to native functions or other operations that reach out to the outside world are stubbed out and have no effect.


* There is no control flow. Loops are visited only once. Both branches are visited in if statements. Logic operators are not short-circuited.

我们的变量处理管道，就像是一个小型的解释器，它会遍历语法树，访问每一个节点，但是静态分析与动态的运行是不同的

* 没有副作用，当静态分析访问一个print语句时候，它不会实际执行该语句，对本地函数或者其他外部函数（调用会对外界产生操作），不会产生任何影响。

* 没有控制流，循环只会访问一次，在if语句中将会访问两个分支，逻辑运算符也没有短路场景

> Variable resolution touches each node once, so its performance is O(n) where n is the number of syntax tree nodes. More sophisticated analyses may have greater complexity, but most are carefully designed to be linear or not far from it. It’s an embarrassing faux pas if your compiler gets exponentially slower as the user’s program grows.
>
> 变量解析器，会遍历每一个节点，所以它的复杂度是 \\( O(n) \\), 其中n 是语法树即诶单的数量。更复杂的分析可能具有更大的复杂度，但是大多数的分析都是精心设计的线性复杂度或者几乎是线性的，如果编译器随着用户程序的增长，分析程序呈现指数级复杂度变化，则分析过程会变慢，这是一个不好的分析设计。


## 三、A Resolver Class

解析类

Like everything in Java, our variable resolution pass is embodied in a class.

像Java中的所有东西一样，我们的变量解析管道，也会封装为一个类

```java

// lox/Resolver.java, create new file

package com.craftinginterpreters.lox;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Stack;

class Resolver implements Expr.Visitor<Void>, Stmt.Visitor<Void> {
  private final Interpreter interpreter;

  Resolver(Interpreter interpreter) {
    this.interpreter = interpreter;
  }
}


```

Since the resolver needs to visit every node in the syntax tree, it implements the visitor abstraction we already have in place. Only a few kinds of nodes are interesting when it comes to resolving variables:

* A block statement introduces a new scope for the statements it contains.

* A function declaration introduces a new scope for its body and binds its parameters in that scope.

* A variable declaration adds a new variable to the current scope

* Variable and assignment expressions need to have their variables resolved.

因为解析器，需要访问语法树中的每一个节点，所以它实现了我们已经之前定义的接口，在解析变量时候，只需对几种类型的节点进行处理

* 块语句，为它包含的语句，创建一个你的作用域

* 函数声明，为它的主体，引入一个新的作用域，并在该作用域中绑定参数

* 变量声明，在当前的作用域中添加了一个新变量

* 变量和赋值表达式，需要解析对应的变量

The rest of the nodes don’t do anything special, but we still need to implement visit methods for them that traverse into their subtrees. Even though a + expression doesn’t itself have any variables to resolve, either of its operands might.

其余类型节点，没有任何特殊，但是我们仍然需要为遍历的每个节点实现visit方法，即使一个 + 表达式，本身没有任何需要解析的变量，它的任何一个操作数需要解析变量

### 3.1 Resolving blocks

解析语法块


We start with blocks since they create the local scopes where all the magic happens.

我们从语法块开始，因为它们创建了本地作用域，而在其中，将发生一些神奇的变化

```java

// lox/Resolver.java, add after Resolver()

  @Override
  public Void visitBlockStmt(Stmt.Block stmt) {
    beginScope();
    resolve(stmt.statements);
    endScope();
    return null;
  }


```


This begins a new scope, traverses into the statements inside the block, and then discards the scope. The fun stuff lives in those helper methods. We start with the simple one.

这将开始一个新的作用域，遍历块内的语句，然后抛弃作用域，有趣的东西存在于解析方法中，我们从简单开始

```java

// lox/Resolver.java, add after Resolver()

  void resolve(List<Stmt> statements) {
    for (Stmt statement : statements) {
      resolve(statement);
    }
  }


```

This walks a list of statements and resolves each one. It in turn calls:

我们将遍历块中的语句，解析每个语句

```java

// lox/Resolver.java, add after visitBlockStmt()

  private void resolve(Stmt stmt) {
    stmt.accept(this);
  }


```


While we’re at it, let’s add another overload that we’ll need later for resolving an expression.

现在，我们将添加另外一个重载，稍后，我们将需要它来解析表达式

```java

// lox/Resolver.java, add after resolve(Stmt stmt)

  private void resolve(Expr expr) {
    expr.accept(this);
  }
	
```

These methods are similar to the evaluate() and execute() methods in Interpreter—they turn around and apply the Visitor pattern to the given syntax tree node.

这些方法类似于解释器中的evaluate() 和 execute() 方法，它们会将访问者模式应用于具体的语法树节点

The real interesting behavior is around scopes. A new block scope is created like so:

真正有趣的是作用域，创建一个新的作用域如下

```java

// lox/Resolver.java, add after resolve()

  private void beginScope() {
    scopes.push(new HashMap<String, Boolean>());
  }


```

Lexical scopes nest in both the interpreter and the resolver. They behave like a stack. The interpreter implements that stack using a linked list—the chain of Environment objects. In the resolver, we use an actual Java Stack.

词法作用域，嵌套在解释器和变量解析器中。它们表现的像是一个堆栈，解释器使用环境对象链表，实现该堆栈。在解析器中，我们使用实际的Java堆栈。

```java

// lox/Resolver.java, in class Resolver

  private final Interpreter interpreter;
  private final Stack<Map<String, Boolean>> scopes = new Stack<>();

  Resolver(Interpreter interpreter) {
	
```

This field keeps track of the stack of scopes currently, uh, in scope. Each element in the stack is a Map representing a single block scope. Keys, as in Environment, are variable names. The values are Booleans, for a reason I’ll explain soon.


该字段，跟踪当前作用域内的堆栈，堆栈中的每一个元素都是，表示单个块作用域的map。与环境一样，map 的key是变量名称，map的value是布尔值，原因马上揭晓

The scope stack is only used for local block scopes. Variables declared at the top level in the global scope are not tracked by the resolver since they are more dynamic in Lox. When resolving a variable, if we can’t find it in the stack of local scopes, we assume it must be global.

作用域堆栈，仅用于本地块作用域。在全局范围的顶层，声明的变量不会被解析器跟踪，因为它们在Lox中是动态的，当解析一个变量时候，如果我们在局部作用域堆栈中，找不到它，我们假设它是全局的

Since scopes are stored in an explicit stack, exiting one is straightforward.

由于作用域存储在显示的堆栈中，因此，退出一个作用域很简单

```java

// lox/Resolver.java, add after beginScope()

  private void endScope() {
    scopes.pop();
  }
	
```

Now we can push and pop a stack of empty scopes. Let’s put some things in them.

现在，我们可以添加、删除一个空的作用域，接下来，我们将把一些东西放入堆栈

### 3.2 Resolving variable declarations

解析变量声明

Resolving a variable declaration adds a new entry to the current innermost scope’s map. That seems simple, but there’s a little dance we need to do.

解析一个变量声明，将向当前最内部的作用域中添加一个新记录，这看起来非常简单，但是我们需要去实现

```java

// lox/Resolver.java, add after visitBlockStmt()

  @Override
  public Void visitVarStmt(Stmt.Var stmt) {
    declare(stmt.name);
    if (stmt.initializer != null) {
      resolve(stmt.initializer);
    }
    define(stmt.name);
    return null;
  }
	
```

We split binding into two steps, declaring then defining, in order to handle funny edge cases like this:

我们将绑定分为两个步骤，声明然后定义，以便处理向下面这样的边界场景

```java

var a = "outer";
{
  var a = a;
}


```

What happens when the initializer for a local variable refers to a variable with the same name as the variable being declared? We have a few options:

1. Run the initializer, then put the new variable in scope. Here, the new local a would be initialized with “outer”, the value of the global one. In other words, the previous declaration would desugar to:

1. Put the new variable in scope, then run the initializer. This means you could observe a variable before it’s initialized, so we would need to figure out what value it would have then. Probably nil. That means the new local a would be re-initialized to its own implicitly initialized value, nil. Now the desugaring would look like:

1. Make it an error to reference a variable in its initializer. Have the interpreter fail either at compile time or runtime if an initializer mentions the variable being initialized.


当局部变量的初始值，设定项引用的变量 和 所声明的变量同名，会发生什么？

1. 运行初始值设定项，然后将新变量放入到作用域中，在这里，新的局部变量a 将被初始化为 outer, 换言之，之前的声明将减少

		```java

		var temp = a; // Run the initializer.
		var a;        // Declare the variable.
		a = temp;     // Initialize it.

		```

1. 将新的变量放入作用域中，然后运行初始值设定项，这意味着我们可以在变量初始化之前观察它，因此我们需要弄清楚它的值，可能是nil，这意味着新的局部变量a，将被初始化为其隐式的初始化值nil，现在，看起来将变为

		```java

		var a; // Define the variable.
		a = a; // Run the initializer.

		```

1. 在其初始化设定项中引用变量时候报错，如果初始值设定项引用了正在初始化的变量，则解释器在编译或者运行时候报错

Do either of those first two options look like something a user actually wants? Shadowing is rare and often an error, so initializing a shadowing variable based on the value of the shadowed one seems unlikely to be deliberate.

前面两个选项中的哪一个看起来更像是客户真正想要的呢？隐式很少见，而且常常是一个错误，因此基于隐式变量的值，初始化为隐式值，似乎不太可能是真实的

The second option is even less useful. The new variable will always have the value nil. There is never any point in mentioning it by name. You could use an explicit nil instead.

第二个选项甚至不太有用，新变量的值始终是nil，提到它的名字从来没有任何意义，我们可以使用显示的nil

Since the first two options are likely to mask user errors, we’ll take the third. Further, we’ll make it a compile error instead of a runtime one. That way, the user is alerted to the problem before any code is run.

由于前面的两个选项，可能会掩盖用户的错误，我们将采用第三个选项，此外，我们将使其变为一个编译错误，而不是一个运行时错误，这样，在运行任何代码之前，我们可以提醒用户该问题

In order to do that, as we visit expressions, we need to know if we’re inside the initializer for some variable. We do that by splitting binding into two steps. The first is declaring it.

为了做到这一点，当我们访问表达式时候，我们需要知道是否在某个变量的初始值设定项内。我们通过将绑定分为两个步骤来实现这一点，首先是声明

```java

// lox/Resolver.java, add after endScope()

  private void declare(Token name) {
    if (scopes.isEmpty()) return;

    Map<String, Boolean> scope = scopes.peek();
    scope.put(name.lexeme, false);
  }
	
```

Declaration adds the variable to the innermost scope so that it shadows any outer one and so that we know the variable exists. We mark it as “not ready yet” by binding its name to false in the scope map. The value associated with a key in the scope map represents whether or not we have finished resolving that variable’s initializer.

声明，将变量添加到最内部的作用域中，使其隐藏任何外部作用域，从而我们知道变量存在。我们通过在作用域中，添加映射关系，将该变量名称绑定为false，将其标记为 “尚未就绪”。作用域映射map表中的值，表示该变量的初始值是否设置好

After declaring the variable, we resolve its initializer expression in that same scope where the new variable now exists but is unavailable. Once the initializer expression is done, the variable is ready for prime time. We do that by defining it.

声明变量后，我们在相同的作用域中，解析变量初始值设置表达式，当前，该变量已经存在但是还没有初始化。当变量初始化后，变量可以被访问到，我们可以通过define() 修改变量的状态

```java

// lox/Resolver.java, add after declare()

  private void define(Token name) {
    if (scopes.isEmpty()) return;
    scopes.peek().put(name.lexeme, true);
  }
	
```

We set the variable’s value in the scope map to true to mark it as fully initialized and available for use. It’s alive!

我们将作用域映射表中，该声明变量的值设为true，表示该变量初始化完成，可以被调用，现在它已经存活

### 3.3 Resolving variable expressions

解析变量表达式

Variable declarations—and function declarations, which we’ll get to—write to the scope maps. Those maps are read when we resolve variable expressions.

变量声明和函数声明，我们将把它们写入到作用域映射中，这些映射在我们解析变量表达式时候被读取

```java

// lox/Resolver.java, add after visitVarStmt()

  @Override
  public Void visitVariableExpr(Expr.Variable expr) {
    if (!scopes.isEmpty() &&
        scopes.peek().get(expr.name.lexeme) == Boolean.FALSE) {
      Lox.error(expr.name,
          "Can't read local variable in its own initializer.");
    }

    resolveLocal(expr, expr.name);
    return null;
  }


```

First, we check to see if the variable is being accessed inside its own initializer. This is where the values in the scope map come into play. If the variable exists in the current scope but its value is false, that means we have declared it but not yet defined it. We report that error.

首先，我们检查变量，是否在它的初始值设定项中引用自身，这就是作用域映射中的值发挥作用的地方，如果变量存在于当前作用域中，但是它的值是false，则意味着该变量已经声明，但是还没有初始化，我们抛出错误

After that check, we actually resolve the variable itself using this helper:

当检查之后，我们实际上使用下面的方法，解析变量本身

```java

// lox/Resolver.java, add after define()

  private void resolveLocal(Expr expr, Token name) {
    for (int i = scopes.size() - 1; i >= 0; i--) {
      if (scopes.get(i).containsKey(name.lexeme)) {
        interpreter.resolve(expr, scopes.size() - 1 - i);
        return;
      }
    }
  }
	
```

This looks, for good reason, a lot like the code in Environment for evaluating a variable. We start at the innermost scope and work outwards, looking in each map for a matching name. If we find the variable, we resolve it, passing in the number of scopes between the current innermost scope and the scope where the variable was found. So, if the variable was found in the current scope, we pass in 0. If it’s in the immediately enclosing scope, 1. You get the idea.

这看起来很像环境中用于计算变量的代码，这是有道理的。我们从内部的作用域开始，向外寻找，如果我们找到了变量，我们就解析它，在当前最内部的作用域和找到该变量的作用域之间传递作用域的数量，因此，如果在当前的作用域中找到了变量，我们将传入0，如果它在紧邻的作用域内，我们将传入1。

If we walk through all of the block scopes and never find the variable, we leave it unresolved and assume it’s global. We’ll get to the implementation of that resolve() method a little later. For now, let’s keep on cranking through the other syntax nodes.

如果我们遍历了所有的作用域，但是没有找到该变量，我们将其保存为未解析状态，并假设它是全局的，稍后我们将讨论 resolve() 方法，现在，我们将继续解析其他的语法节点

### 3.4 Resolving assignment expressions

解析赋值表达式

The other expression that references a variable is assignment. Resolving one looks like this:

引用变量的另外一个表达式是赋值表达式，解析如下

```java

// lox/Resolver.java, add after visitVarStmt()

  @Override
  public Void visitAssignExpr(Expr.Assign expr) {
    resolve(expr.value);
    resolveLocal(expr, expr.name);
    return null;
  }

```

First, we resolve the expression for the assigned value in case it also contains references to other variables. Then we use our existing resolveLocal() method to resolve the variable that’s being assigned to.

首先，我们解析表达式的值，因为表达式值可能包含有其他的变量，接下来，我们使用resolveLocal() 方法，解析表达式分配的变量

### 3.5 Resolving function declarations

解析函数声明

Finally, functions. Functions both bind names and introduce a scope. The name of the function itself is bound in the surrounding scope where the function is declared. When we step into the function’s body, we also bind its parameters into that inner function scope.

最后，是函数，函数既绑定了函数名称，又引入了作用域。函数本身的名称绑定在声明函数的周围作用域中，当我们进入函数体时，我们还将其参数绑定到内部函数作用域中


```java

// lox/Resolver.java, add after visitBlockStmt()

  @Override
  public Void visitFunctionStmt(Stmt.Function stmt) {
    declare(stmt.name);
    define(stmt.name);

    resolveFunction(stmt);
    return null;
  }


```

Similar to visitVariableStmt(), we declare and define the name of the function in the current scope. Unlike variables, though, we define the name eagerly, before resolving the function’s body. This lets a function recursively refer to itself inside its own body.

和 visitvariableStmt() 相似，我们声明并且定义了当前作用域内的函数名称，然而，与变量不同，我们在解析函数体之前，定义了函数名称。这允许函数在函数体内递归引用自己

Then we resolve the function’s body using this:

然后，我们将使用下面的方法解析

```java

// lox/Resolver.java, add after resolve()

  private void resolveFunction(Stmt.Function function) {
    beginScope();
    for (Token param : function.params) {
      declare(param);
      define(param);
    }
    resolve(function.body);
    endScope();
  }
	
```

It’s a separate method since we will also use it for resolving Lox methods when we add classes later. It creates a new scope for the body and then binds variables for each of the function’s parameters.

这是一个单独的方法，因为我们稍后添加类时候，也将使用它来解析Lox方法，它为主体创建了一个新的范围，然后为函数的每一个参数绑定了变量。

Once that’s ready, it resolves the function body in that scope. This is different from how the interpreter handles function declarations. At runtime, declaring a function doesn’t do anything with the function’s body. The body doesn’t get touched until later when the function is called. In a static analysis, we immediately traverse into the body right then and there.

一旦准备就绪，它将解析该作用域内的函数体，这与解释器处理函数声明的方式不同。在运行时，声明函数对函数的主体没有任何作用，直到稍后调用函数时候，才会接触到主体，在一个静态分析中，我们将立刻进入函数体

### 3.6 Resolving the other syntax tree nodes


解析其他语法树节点


That covers the interesting corners of the grammars. We handle every place where a variable is declared, read, or written, and every place where a scope is created or destroyed. Even though they aren’t affected by variable resolution, we also need visit methods for all of the other syntax tree nodes in order to recurse into their subtrees. Sorry this bit is boring, but bear with me. We’ll go kind of “top down” and start with statements.

这涵盖了语法中的有趣角落，我们会处理

* 变量声明

* 变量读取

* 变量写入

* 创建作用域

* 销毁作用域

即使它们不受变量解析影响，我们也需要为其他语法树节点的定义visit() 方法， 便于递归解析子树

很抱歉，这一点有些无聊，但是请耐心分析，我们将用 自上而下的方式，从语句开始

> I did say the book would have every single line of code for these interpreters. I didn’t say they’d all be exciting.
>
> 我确实说过，这本书将为这些解释器提供每一行代码，我没有说，这些代码都很有趣。😄


An expression statement contains a single expression to traverse.

一个表达式语句，包含了要遍历的每一个表达式


```java

// lox/Resolver.java, add after visitBlockStmt()

  @Override
  public Void visitExpressionStmt(Stmt.Expression stmt) {
    resolve(stmt.expression);
    return null;
  }

```


An if statement has an expression for its condition and one or two statements for the branches.

一个if语句，有一个条件表达式和一个或者两个分支语句

```java

// lox/Resolver.java, add after visitFunctionStmt()

  @Override
  public Void visitIfStmt(Stmt.If stmt) {
    resolve(stmt.condition);
    resolve(stmt.thenBranch);
    if (stmt.elseBranch != null) resolve(stmt.elseBranch);
    return null;
  }


```

Here, we see how resolution is different from interpretation. When we resolve an if statement, there is no control flow. We resolve the condition and both branches. Where a dynamic execution steps only into the branch that is run, a static analysis is conservative—it analyzes any branch that could be run. Since either one could be reached at runtime, we resolve both.

在这里，我们将看到分析和解释的区别，当我们分析一个if语句，没有控制流。我们将解析，条件表达式和两个判断分支，当动态的执行时候，我们只会进入其中的某一个分支，静态分析是保守的——它会分析任何可能运行的分支，因为运行时，有可能进入每一个分支，所有我们将会同时，解析这两个分支

Like expression statements, a print statement contains a single subexpression.

和表达式语句一样，print语句包含单个子表达式

```java

// lox/Resolver.java, add after visitIfStmt()

  @Override
  public Void visitPrintStmt(Stmt.Print stmt) {
    resolve(stmt.expression);
    return null;
  }
	
```

Same deal for return.

对于return语句，同样方式解析

```java

// lox/Resolver.java, add after visitPrintStmt()

  @Override
  public Void visitReturnStmt(Stmt.Return stmt) {
    if (stmt.value != null) {
      resolve(stmt.value);
    }

    return null;
  }
	
```

As in if statements, with a while statement, we resolve its condition and resolve the body exactly once.


和if语句一样，当我们解析while语句时候，我们将解析它的条件判断表达式，然后，解析循环主体，但是只会解析一次

```java

// lox/Resolver.java, add after visitVarStmt()

  @Override
  public Void visitWhileStmt(Stmt.While stmt) {
    resolve(stmt.condition);
    resolve(stmt.body);
    return null;
  }


```

That covers all the statements. On to expressions . . 

上面已经包含了所有的语句类型，接下来我们将进入表达式

Our old friend the binary expression. We traverse into and resolve both operands.


先从我们熟悉的二元表达式开始，我们将遍历二元表达式的两个操作数


```java

// lox/Resolver.java, add after visitAssignExpr()

  @Override
  public Void visitBinaryExpr(Expr.Binary expr) {
    resolve(expr.left);
    resolve(expr.right);
    return null;
  }


```


Calls are similar—we walk the argument list and resolve them all. The thing being called is also an expression (usually a variable expression), so that gets resolved too.

调用是类似的，我们遍历参数列表并且解析每一个参数，被调用对象也是一个表达式（通常是一个变量表达式），因此也会得到解析

```java

// lox/Resolver.java, add after visitBinaryExpr()

  @Override
  public Void visitCallExpr(Expr.Call expr) {
    resolve(expr.callee);

    for (Expr argument : expr.arguments) {
      resolve(argument);
    }

    return null;
  }

```

Parentheses are easy.

括号表达式很简单

```java

// lox/Resolver.java, add after visitCallExpr()

  @Override
  public Void visitGroupingExpr(Expr.Grouping expr) {
    resolve(expr.expression);
    return null;
  }


```

Literals are easiest of all.

文本表达式是最简单的

```java

// lox/Resolver.java, add after visitGroupingExpr()

  @Override
  public Void visitLiteralExpr(Expr.Literal expr) {
    return null;
  }
	
```

A literal expression doesn’t mention any variables and doesn’t contain any subexpressions so there is no work to do.

文本表达式不涉及任何的变量，也不包含任何子表达式，因此没有解析工作要做

Since a static analysis does no control flow or short-circuiting, logical expressions are exactly the same as other binary operators.

由于静态分析不控制流或者短路，因此逻辑表达式与其他二元运算符完全相同

```java

// lox/Resolver.java, add after visitLiteralExpr()

  @Override
  public Void visitLogicalExpr(Expr.Logical expr) {
    resolve(expr.left);
    resolve(expr.right);
    return null;
  }
	
```

And, finally, the last node. We resolve its one operand.

最后一个语法树节点是一元运算符，我们解析它的操作数

```

// lox/Resolver.java, add after visitLogicalExpr()

  @Override
  public Void visitUnaryExpr(Expr.Unary expr) {
    resolve(expr.right);
    return null;
  }

```


With all of these visit methods, the Java compiler should be satisfied that Resolver fully implements Stmt.Visitor and Expr.Visitor. Now is a good time to take a break, have a snack, maybe a little nap.

如上，我们实现了所有语法树节点对应的visit() 方法，Java编译器应该会，对于每个语法树，运行对应的Stmt.Visitor 或者 Expr.Visitor, 现在我们可以中场休息一下😄

## 四、Interpreting Resolved Variables

解释解析出的变量

Let’s see what our resolver is good for. Each time it visits a variable, it tells the interpreter how many scopes there are between the current scope and the scope where the variable is defined. At runtime, this corresponds exactly to the number of environments between the current one and the enclosing one where the interpreter can find the variable’s value. The resolver hands that number to the interpreter by calling this:

让我们看看我们的解析器的作用。每次访问一个变量时候，它会告诉解释器，当前的作用域和定义变量的作用域之间的范围，在运行时候，这正好对应于当前环境和封闭环境之间的环境差距数量。解析器通过下面的方式，将该差距数量传递给解释器

```java

// lox/Interpreter.java, add after execute()

  void resolve(Expr expr, int depth) {
    locals.put(expr, depth);
  }
	
```

We want to store the resolution information somewhere so we can use it when the variable or assignment expression is later executed, but where? One obvious place is right in the syntax tree node itself. That’s a fine approach, and that’s where many compilers store the results of analyses like this.

我们希望将解析信息保存到某个地方，以便在稍后执行变量或者赋值表达式时候，使用它。但是保存在哪里呢？一个明显的地方就是语法树节点本身，只是一个很好的方法，许多编译器都在这里存储这样的分析结果。

We could do that, but it would require mucking around with our syntax tree generator. Instead, we’ll take another common approach and store it off to the side in a map that associates each syntax tree node with its resolved data.

我们可以做到这一点，但是需要修改语法生成器，相反，我们将采用另外一种常见的方法，我们将吧语法树每一个节点，和与其对应的解析数据，存储到一个独立的map映射中

> I think I’ve heard this map called a “side table” since it’s a tabular data structure that stores data separately from the objects it relates to. But whenever I try to Google for that term, I get pages about furniture.
> 
> 我听到别人称呼这个map为 符号表，因为它是这样的一张表格，将数据和相关对象分开存储。但是每当我搜索这个术语时候，谷歌总是显示一堆家具的广告

Interactive tools like IDEs often incrementally reparse and re-resolve parts of the user’s program. It may be hard to find all of the bits of state that need recalculating when they’re hiding in the foliage of the syntax tree. A benefit of storing this data outside of the nodes is that it makes it easy to discard it—simply clear the map.

像是IDE这样的交互工具，经常会增量分析和重新分析用户程序的更新部分，当隐藏在语法树的叶节点时候，可能很难找到需要重新计算状态信息的节点，如果将这些数据存储在节点之外，的其中一个好处是，只需要更新map，就可以修改这些数据

```java

// lox/Interpreter.java, in class Interpreter

  private Environment environment = globals;
  private final Map<Expr, Integer> locals = new HashMap<>();

  Interpreter() {


```

You might think we’d need some sort of nested tree structure to avoid getting confused when there are multiple expressions that reference the same variable, but each expression node is its own Java object with its own unique identity. A single monolithic map doesn’t have any trouble keeping them separated.

你可能会认为，当有多个表达式引用同一个变量时候，我们需要某种嵌套的树结构来避免混淆，但是每个表达式节点都是自己的Java对象，具有自己的唯一标识，一个单一的map，没有任何问题，可以将节点区别开

As usual, using a collection requires us to import a couple of names.

```java

// lox/Interpreter.java

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

```

```java

// lox/Interpreter.java

import java.util.List;
import java.util.Map;

class Interpreter implements Expr.Visitor<Object>,

```

### 4.1 Accessing a resolved variable

访问一个已经解析的变量

Our interpreter now has access to each variable’s resolved location. Finally, we get to make use of that. We replace the visit method for variable expressions with this:

我们的解释器现在可以访问每个变量的解析位置，最后，我们要利用这一点，我们将变量表达式的visit() 方法替换为

```java

// lox/Interpreter.java, in visitVariableExpr(), replace 1 line

  public Object visitVariableExpr(Expr.Variable expr) {
    return lookUpVariable(expr.name, expr);
  }


```

That delegates to:

```java

// lox/Interpreter.java, add after visitVariableExpr()

  private Object lookUpVariable(Token name, Expr expr) {
    Integer distance = locals.get(expr);
    if (distance != null) {
      return environment.getAt(distance, name.lexeme);
    } else {
      return globals.get(name);
    }
  }

```

There are a couple of things going on here. First, we look up the resolved distance in the map. Remember that we resolved only local variables. Globals are treated specially and don’t end up in the map (hence the name locals). So, if we don’t find a distance in the map, it must be global. In that case, we look it up, dynamically, directly in the global environment. That throws a runtime error if the variable isn’t defined.

这里发生了一些事情，首先，我们在map中查找节点对应的解析距离， 请记住，我们只是解析了局部变量，全局变量被特殊对待，不会出现在map中（因此，本地变量被称为 locals) 所以，如果我们在map中找不到 距离，它一定是全局变量，在这种情况下，我们直接在 全局环境变量中，查找对应的变量，如果未定义变量，则会引发运行时错误

If we do get a distance, we have a local variable, and we get to take advantage of the results of our static analysis. Instead of calling get(), we call this new method on Environment:

如果我们确实得到了一个距离，我们有一个局部变量，我们可以利用静态分析的结果。在当前环境中调用这个新方法，而不是使用get

```java

// lox/Environment.java, add after define()

  Object getAt(int distance, String name) {
    return ancestor(distance).values.get(name);
  }


```

The old get() method dynamically walks the chain of enclosing environments, scouring each one to see if the variable might be hiding in there somewhere. But now we know exactly which environment in the chain will have the variable. We reach it using this helper method:

以前的get() 方法，将动态的遍历环境链，搜索每一个环境，看看变量是否隐藏在其中，但是现在，我们确切的知道了环境链中哪个环境将包含该变量，我们使用下面的方法来获取

```java

// lox/Environment.java, add after define()

  Environment ancestor(int distance) {
    Environment environment = this;
    for (int i = 0; i < distance; i++) {
      environment = environment.enclosing; 
    }

    return environment;
  }


```

This walks a fixed number of hops up the parent chain and returns the environment there. Once we have that, getAt() simply returns the value of the variable in that environment’s map. It doesn’t even have to check to see if the variable is there—we know it will be because the resolver already found it before.

这将沿着父链向上移动固定数量，并且返回那里的环境，一旦我们完成了查找环境，我们将会返回该环境中的变量值，甚至不需要检查变量是否存在，我们知道它一定是存在的，因为解析器之前已经静态分析过

> The way the interpreter assumes the variable is in that map feels like flying blind. The interpreter code trusts that the resolver did its job and resolved the variable correctly. This implies a deep coupling between these two classes. In the resolver, each line of code that touches a scope must have its exact match in the interpreter for modifying an environment.
> 
> 解释器假设变量一定在map中，看起来像是盲目自信。解释器代码，假设变量解析器，会正确完成它的任务。这意味着两个类之前是深度耦合的，在变量解析器中，涉及作用域的每行代码，必须在解释器中具有完全匹配的意义，以便修改环境。
>
> I felt that coupling firsthand because as I wrote the code for the book, I ran into a couple of subtle bugs where the resolver and interpreter code were slightly out of sync. Tracking those down was difficult. One tool to make that easier is to have the interpreter explicitly assert—using Java’s assert statements or some other validation tool—the contract it expects the resolver to have already upheld.
> 
> 我亲身感受过这种深度耦合，因为当我编写本书代码时候，我遇到过一些微小的错误，其中的变量解析器和解释器代码不同步，追踪这些错误非常困难，一种更加简单的工具是让解释器使用Java 的assert 语句或者其他验证工具，显式的断言，它期望变量解析器的解析结果

### 4.2 Assigning to a resolved variable

分配给已经解析的变量

We can also use a variable by assigning to it. The changes to visiting an assignment expression are similar.

我们也可以通过赋值来使用变量，更改赋值表达式类似

```java

// lox/Interpreter.java, in visitAssignExpr(), replace 1 line

  public Object visitAssignExpr(Expr.Assign expr) {
    Object value = evaluate(expr.value);

    Integer distance = locals.get(expr);
    if (distance != null) {
      environment.assignAt(distance, expr.name, value);
    } else {
      globals.assign(expr.name, value);
    }

    return value;
		
```


Again, we look up the variable’s scope distance. If not found, we assume it’s global and handle it the same way as before. Otherwise, we call this new method:

接下来，我们查找变量的距离，如果没有找到，我们假设它是全局变量，否则，我们将调用新方法

```java

// lox/Environment.java, add after getAt()

  void assignAt(int distance, Token name, Object value) {
    ancestor(distance).values.put(name.lexeme, value);
  }
	
```

As getAt() is to get(), assignAt() is to assign(). It walks a fixed number of environments, and then stuffs the new value in that map.

正如，getAt() 对于 get(), assignAt() 是 assign() 的替换方法，它会遍历固定数量，找到那里的环境，然后重新填充到map中


Those are the only changes to Interpreter. This is why I chose a representation for our resolved data that was minimally invasive. All of the rest of the nodes continue working as they did before. Even the code for modifying environments is unchanged.

这些是解释器的唯一变化，这就是为什么我选择了最小改变的解析数据表示方法，其余节点都正常工作，与之前一样。甚至修改环境的代码也没有改变


### 4.3 Running the resolver

运行解析

We do need to actually run the resolver, though. We insert the new pass after the parser does its magic.

不过，我们需要实际运行解析器，我们在解析器完成其魔术后，插入新的管道

```java

// lox/Lox.java, in run()

    // Stop if there was a syntax error.
    if (hadError) return;

    Resolver resolver = new Resolver(interpreter);
    resolver.resolve(statements);

    interpreter.interpret(statements);
		
```


We don’t run the resolver if there are any parse errors. If the code has a syntax error, it’s never going to run, so there’s little value in resolving it. If the syntax is clean, we tell the resolver to do its thing. The resolver has a reference to the interpreter and pokes the resolution data directly into it as it walks over variables. When the interpreter runs next, it has everything it needs.


如果解析语法树过程出现任何错误，我们不会运行变量解析器。如果代码有语法错误，将永远不会运行该代码，因此解决它没有什么价值。如果语法是正确的，我们的变量解析器将运行，变量解析器，有一个对于解释器的引用，并在遍历语法树时候，将解析数据添加到解释器中，当解释器运行时候，可以利用这里的变量解析数据

At least, that’s true if the resolver succeeds. But what about errors during resolution?

至少，如果解析器是成功的，但是如果我们分析过程中出现错误，将发生什么？


## 五、Resolution Errors

解析错误

Since we are doing a semantic analysis pass, we have an opportunity to make Lox’s semantics more precise, and to help users catch bugs early before running their code. Take a look at this bad boy:

由于，我们正在进行语义分析，我们有机会使Lox的语义更加精确，并帮助用户在运行代码之前，尽早的发现错误，看看下面这个错误示例

```java

fun bad() {
  var a = "first";
  var a = "second";
}

```

We do allow declaring multiple variables with the same name in the global scope, but doing so in a local scope is probably a mistake. If they knew the variable already existed, they would have assigned to it instead of using var. And if they didn’t know it existed, they probably didn’t intend to overwrite the previous one.

我们确实允许在全局范围内，声明多个同名变量，但是在局部作用域中，这样做可能是错误的。如果它们知道该变量已经存在，用户需要将其修改为赋值，而不是使用声明，如果用户不知道该变量已经存在，他们可能不打算覆盖前一个变量

We can detect this mistake statically while resolving.

我们可以在变量解析器运行时候，静态检测此错误

```java

// lox/Resolver.java, in declare()

    Map<String, Boolean> scope = scopes.peek();
    if (scope.containsKey(name.lexeme)) {
      Lox.error(name,
          "Already a variable with this name in this scope.");
    }

    scope.put(name.lexeme, false);
		
```


When we declare a variable in a local scope, we already know the names of every variable previously declared in that same scope. If we see a collision, we report an error.

当我们在局部作用域中已经声明了一个变量时，如果我们在当前作用域中，能找到同名的变量。我们将抛出一个错误

### 5.1 Invalid return errors

非法的返回值

Here’s another nasty little script:

下面是一个错误示例

```java

return "at top level";

```

This executes a return statement, but it’s not even inside a function at all. It’s top-level code. I don’t know what the user thinks is going to happen, but I don’t think we want Lox to allow this.

这会执行一个return 语句，但是它根本不在函数内部，只是一个顶层代码。我不知道用户希望发生什么，但是我们不希望Lox允许这样的语法

We can extend the resolver to detect this statically. Much like we track scopes as we walk the tree, we can track whether or not the code we are currently visiting is inside a function declaration.

我们可以扩展变量解析器，去检查这种场景。就像是我们在遍历语法树，跟踪作用域一样，我们可以跟踪当前访问的代码是否在函数声明中。

```java

// lox/Resolver.java, in class Resolver

  private final Stack<Map<String, Boolean>> scopes = new Stack<>();
  private FunctionType currentFunction = FunctionType.NONE;

  Resolver(Interpreter interpreter) {
	
```


Instead of a bare Boolean, we use this funny enum:

我们将使用下面的枚举，而不是简单的布尔值

```java

// lox/Resolver.java, add after Resolver()

  private enum FunctionType {
    NONE,
    FUNCTION
  }
	
```

It seems kind of dumb now, but we’ll add a couple more cases to it later and then it will make more sense. When we resolve a function declaration, we pass that in.

它现在看起来有点简单，但是稍后，我们将再添加几个类型，这样会更加有意义，当我们解析函数声明时候，将会传递该值

```java

// lox/Resolver.java, in visitFunctionStmt(), replace 1 line

    define(stmt.name);

    resolveFunction(stmt, FunctionType.FUNCTION);
    return null;
		
```

Over in resolveFunction(), we take that parameter and store it in the field before resolving the body.

在 resloveFunction() 函数中，我们获取该参数并在解析函数体之前，将其存储起来

```java

// lox/Resolver.java, method resolveFunction(), replace 1 line

  private void resolveFunction(
      Stmt.Function function, FunctionType type) {
    FunctionType enclosingFunction = currentFunction;
    currentFunction = type;

    beginScope();

```

We stash the previous value of the field in a local variable first. Remember, Lox has local functions, so you can nest function declarations arbitrarily deeply. We need to track not just that we’re in a function, but how many we’re in.

我们首先将字段的前一个值，存储在局部变量中，记住，Lox具有局部函数，因此可以任意嵌套函数声明。我们不仅仅需要跟踪当前的函数，也需要跟踪嵌套函数中的每一层

We could use an explicit stack of FunctionType values for that, but instead we’ll piggyback on the JVM. We store the previous value in a local on the Java stack. When we’re done resolving the function body, we restore the field to that value.

我们可以使用FunctionType 值的一个显式堆栈，但是正相反，我们将依赖JVM，我们将上一个值存储在Java 堆栈的本地作用域中。当我们解析完后，将恢复该字段值

```java

// lox/Resolver.java, in resolveFunction()

    endScope();
    currentFunction = enclosingFunction;
  }


```


Now that we can always tell whether or not we’re inside a function declaration, we check that when resolving a return statement.

现在，我们总是可以判断，我们是否在一个函数声明中，在解析return语句时候，添加判断

```java

// lox/Resolver.java, in visitReturnStmt()

  public Void visitReturnStmt(Stmt.Return stmt) {
    if (currentFunction == FunctionType.NONE) {
      Lox.error(stmt.keyword, "Can't return from top-level code.");
    }

    if (stmt.value != null) {


```

Neat, right?

There’s one more piece. Back in the main Lox class that stitches everything together, we are careful to not run the interpreter if any parse errors are encountered. That check runs before the resolver so that we don’t try to resolve syntactically invalid code.

还有一个问题，回到将所有内容已经缝合在一起的Lox类中，如果遇到任何的解析报错，我们不会运行解释器，该检查也在变量解析器之前运行，所以，我们也不会试图解析语法报错的代码

But we also need to skip the interpreter if there are resolution errors, so we add another check.

但是如果存在解析错误，我们也需要跳过解释器，因此我们还需要添加另外一个检查

```java

// lox/Lox.java, in run()

    resolver.resolve(statements);

    // Stop if there was a resolution error.
    if (hadError) return;

    interpreter.interpret(statements);
		
```


You could imagine doing lots of other analysis in here. For example, if we added break statements to Lox, we would probably want to ensure they are only used inside loops.

你可以想象在这里做更多的分析，例如，如果我们将 break语句添加到 Lox 中，我们可能希望确保它们只在内循环中使用

We could go farther and report warnings for code that isn’t necessarily wrong but probably isn’t useful. For example, many IDEs will warn if you have unreachable code after a return statement, or a local variable whose value is never read. All of that would be pretty easy to add to our static visiting pass, or as separate passes.

我们可以更进一步，报告一些warn 类型的警告，这些代码不一定是错误的，但是可能是无用的。例如：

* 在return语句，后面还有无法访问的代码

* 某个本地变量，从来没有被使用

许多IDE 都会发出警告。所有这些都可以简单的添加到我们的静态分析中，或者我们另加一个解析器

But, for now, we’ll stick with that limited amount of analysis. The important part is that we fixed that one weird annoying edge case bug, though it might be surprising that it took this much work to do it.

但是，现在，我们将继续进行有限的分析。重要的一点是，我们修复了一个奇怪的令人讨厌的bug，尽管这可能令你非常惊讶，修复这个bug，需要添加这么多的工作

> The choice of how many different analyses to lump into a single pass is difficult. Many small isolated passes, each with their own responsibility, are simpler to implement and maintain. However, there is a real runtime cost to traversing the syntax tree itself, so bundling multiple analyses into a single pass is usually faster.
> 
> 把多少个不同的分析逻辑放入一个管道中，是非常难决定的。许多单独的小管道，每一个管道都有自己的责任，这样，实施和维护更加简单。然后，遍历语法树本身会产生一些运行时成本，因此将多个分析管道绑定到一起，通常可以更快的完成分析阶段。

## 六、CHALLENGES


1. Why is it safe to eagerly define the variable bound to a function’s name when other variables must wait until after they are initialized before they can be used?

1. How do other languages you know handle local variables that refer to the same name in their initializer, like:

	```java
	
		var a = "outer";
		{
			var a = a;
		}
	
	```
	
	Is it a runtime error? Compile error? Allowed? Do they treat global variables differently? Do you agree with their choices? Justify your answer.

1. Extend the resolver to report an error if a local variable is never used.

1. Our resolver calculates which environment the variable is found in, but it’s still looked up by name in that map. A more efficient environment representation would store local variables in an array and look them up by index.

	Extend the resolver to associate a unique index for each local variable declared in a scope. When resolving a variable access, look up both the scope the variable is in and its index and store that. In the interpreter, use that to quickly access a variable by its index instead of using a map. 

习题

1. 为什么一个变量，绑定了一个函数名称是安全的，但是，其他类型的变量，必须要等到初始化之后，才能使用？

1. 其他语言中，是如何处理这种场景，本地变量声明时候，引用了同名的本地变量，例如: 

	```java
	
		var a = "outer";
		{
			var a = a;
		}
	
	```
	
	这是一个运行时错误吗？编译错误？是否被允许？对于全局变量是否同样处理？你赞同这样处理吗？证明你的答案是正确的
	
1. 添加一个检查，如果一个局部变量定义后，从未使用，我们抛出一个错误

1. 我们的变量解析器，会计算我们在哪个环境中发现该变量，但是它仍然在map中按照变量名称去查找，一种更加有效的环境，表示方法是将局部变量存储在数组中，并且按照索引查找它们。

	扩展我们的变量解析程序，为每个作用域中的本地变量定义一个唯一索引，当我们解析一个变量时候，我们根据变量所在的环境索引和位置索引，获取到该变量。在解释器中，使用这种方式可以更快的访问变量
	
	
