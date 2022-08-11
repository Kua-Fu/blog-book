# Lox语言

> What nicer thing can you do for somebody than make them breakfast?
> 
> <p align="right">—— Anthony Bourdain</p>
>
> 还有什么比给别人做早餐更好的事情呢？

We’ll spend the rest of this book illuminating every dark and sundry corner of the Lox language, but it seems cruel to have you immediately start grinding out code for the interpreter without at least a glimpse of what we’re going to end up with.

At the same time, I don’t want to drag you through reams of language lawyering and specification-ese before you get to touch your text editor. So this will be a gentle, friendly introduction to Lox. It will leave out a lot of details and edge cases. We’ve got plenty of time for those later.

我们将在本书的剩余部分，阐明Lox语言的每一个黑暗和杂乱的角落。但是，如果我们无法看到 Lox 语言的最终样子，就开始编写解释器代码，看起来有一些匆忙。

在你使用文本编辑器之前，我不想过多介绍语言的格式和规范，所以，本章将是一个温和友好的Lox介绍。它会遗漏很多细节和边缘部分，我们在后面有足够的时间去学习。

> A tutorial isn’t very fun if you can’t try the code out yourself. Alas, you don’t have a Lox interpreter yet, since you haven’t built one!
>
> Fear not. You can use mine.
>
> 如果一个教程不能自己写代码，那么这个教程就没有吸引力。但是，你还没有一个Lox解释器，因为我们还没有去实现它。
>
> 不用担心，你可以先使用我的😄。

## 一、Hello, Lox

第一个程序

```c

// Your first Lox program!
print "Hello, world!";

```

Here’s your very first taste of Lox:

As that // line comment and the trailing semicolon imply, Lox’s syntax is a member of the C family. (There are no parentheses around the string because print is a built-in statement, and not a library function.)

Now, I won’t claim that C has a great syntax. If we wanted something elegant, we’d probably mimic Pascal or Smalltalk. If we wanted to go full Scandinavian-furniture-minimalism, we’d do a Scheme. Those all have their virtues.

What C-like syntax has instead is something you’ll often find more valuable in a language: familiarity.  I know you are already comfortable with that style because the two languages we’ll be using to implement Lox—Java and C—also inherit it. Using a similar syntax for Lox gives you one less thing to learn.

上面是第一个Lox程序，

正如 // 行注释，行尾；所暗示的，Lox语言，继承了C语言语法。hello, world 字符串周围不需要括号，因为 print是一个内置语句，而不是一个库函数。

现在，我不会说C语言有很好的语法，如果我们想要优雅的东西，可能模仿Pascal、Smalltalk更加合适。如果我们想要完全实现 斯堪的纳维亚家具的极简主义，我们需要先做一个计划。这些都有它们的优势。

相反，我们使用类C语言语法，是因为我们可以从中获得更有价值的东西，熟悉度。我知道，你已经习惯了这种风格，接下来，我们用以实现Lox的两种语言，Java 和C语言，都拥有这种熟悉的风格。Lox语言使用这种风格的语法，可以更加容易入门。

> Your first taste of Lox, the language, that is. I don’t know if you’ve ever had the cured, cold-smoked salmon before. If not, give it a try too.
>
> 你第一次品尝Lox语言，就是这样。我不知道你之前有没有尝试过腌制的冷鲑鱼，如果没有，可以尝试一下。

> I’m surely biased, but I think Lox’s syntax is pretty clean. C’s most egregious grammar problems are around types. Dennis Ritchie had this idea called “declaration reflects use”, where variable declarations mirror the operations you would have to perform on the variable to get to a value of the base type.
>
>Lox doesn’t have static types, so we avoid that.
>
> 我是有偏见的，从我的角度，我认为Lox语言语法非常简洁。C语言令人惊讶的语法问题是，类型。Dennis Ritchie 提出一个“声明反映使用”的想法，其中变量声明反映了你对于变量执行的操作，以获得基类型的值。这个想法非常好，但我认为，在实践中的效果并不好。
>
> Lox不是静态类型语言，所以我们避免了这种情况。


## 二、A high-level language

高级语言

While this book ended up bigger than I was hoping, it’s still not big enough to fit a huge language like Java in it. In order to fit two complete implementations of Lox in these pages, Lox itself has to be pretty compact.

When I think of languages that are small but useful, what comes to mind are high-level “scripting” languages like JavaScript, Scheme, and Lua. Of those three, Lox looks most like JavaScript, mainly because most C-syntax languages do. As we’ll learn later, Lox’s approach to scoping hews closely to Scheme. The C flavor of Lox we’ll build in Part III is heavily indebted to Lua’s clean, efficient implementation.

虽然本书内容大大超过了一开始的设想，但是还是无法利用一本书来介绍Java 这样的大型语言。为了在接下来去实现 Lox语言两次，Lox本身需要非常紧凑。

当我们提到小而有用的语言时候，通常会想到的是一些脚本语言，例如：JavaScript, Scheme, Lua 等。在这三种脚本语言中，Lox更像是 JavaScript，因为它们都是类C 语法。正如后面将要介绍的，Lox的代码块范围表示，和Scheme 语言相似。在第三部分，我们将实现的C语言为解释器的Lox语言，将更加接近Lua语言的简洁、高效特征。

Lox shares two other aspects with those three languages:

Lox 和脚本语言还有下面3个相同点：

### 2.1 Dynamic typing

动态类型

Lox is dynamically typed. Variables can store values of any type, and a single variable can even store values of different types at different times. If you try to perform an operation on values of the wrong type—say, dividing a number by a string—then the error is detected and reported at runtime.

There are plenty of reasons to like static types, but they don’t outweigh the pragmatic reasons to pick dynamic types for Lox.  A static type system is a ton of work to learn and implement.Skipping it gives you a simpler language and a shorter book. We’ll get our interpreter up and executing bits of code sooner if we defer our type checking to runtime.

Lox是动态类型语言，变量可以存储任何类型的值，单个变量可以在不同时间存储不同类型的数据，如何尝试对于错误类型的值执行操作，例如：数值除以字符串，运行时候会检测，并且报错。

喜欢静态类型有很多理由，但是为了Lox语言更加实用，我们选择了动态类型。静态类型系统需要学习和实现大量工作。跳过静态类型，会让Lox语言实现更加简单。如果在解释器在运行时候，执行类型检查，我们可以更快的执行代码。

> Now that JavaScript has taken over the world and is used to build ginormous applications, it’s hard to think of it as a “little scripting language”. But Brendan Eich hacked the first JS interpreter into Netscape Navigator in ten days to make buttons animate on web pages. JavaScript has grown up since then, but it was once a cute little language.
> 
> 既然，JavaScript语言已经风靡语言世界，并且用于构建很多的大型项目，我们很难在将它当作一个小众语言。但是，在网景公司，Brendan Eich 仅仅使用了10天时间，就完成了第一个JS编译器，并且实现了网页中的按钮动态展示。JavaScript 从那时开始，不断成长，但是它曾经是一门可爱的小语言。
>
> Because Eich slapped JS together with roughly the same raw materials and time as an episode of MacGyver, it has some weird semantic corners where the duct tape and paper clips show through. Things like variable hoisting, dynamically bound this, holes in arrays, and implicit conversions.
> 
> 因为Eich使用了与《麦基弗》一集大致相同的原材料和时间制作了JS语言，所以，它存在着一些奇怪的语法，会出现一些胶带和回形针。例如：变量提升，动态绑定，数组中的漏洞和隐式转换。
>
> I had the luxury of taking my time on Lox, so it should be a little cleaner. After all, the two languages we’ll be using to implement Lox are both statically typed.
>
> 相比之下，我有更多时间打磨Lox语言，所以，我们会发现，Lox语言会更加简洁。毕竟，我们实现Lox语言的底层语言Java/C都是静态语言。


### 2.2 Automatic memory management

自动内存管理

High-level languages exist to eliminate error-prone, low-level drudgery, and what could be more tedious than manually managing the allocation and freeing of storage? No one rises and greets the morning sun with, “I can’t wait to figure out the correct place to call free() for every byte of memory I allocate today!”

高级语言的出现是为了，消除更加容易出错、低级别、乏味的工作，还有什么比手动管理内存分配与释放，更加繁琐呢？没有人会站起来迎接太阳，我忍不住要为今天分配的每一个字节的内存，找到调用 free() 函数的正确位置。

There are two main techniques for managing memory: reference counting and tracing garbage collection (usually just called garbage collection or GC). Ref counters are much simpler to implement—I think that’s why Perl, PHP, and Python all started out using them. But, over time, the limitations of ref counting become too troublesome. All of those languages eventually ended up adding a full tracing GC, or at least enough of one to clean up object cycles.

Tracing garbage collection has a fearsome reputation. It is a little harrowing working at the level of raw memory. Debugging a GC can sometimes leave you seeing hex dumps in your dreams. But, remember, this book is about dispelling magic and slaying those monsters, so we are going to write our own garbage collector. I think you’ll find the algorithm is quite simple and a lot of fun to implement.

内存管理主要有两种技术：引用计数 和 追踪垃圾回收，通常也称为，垃圾回收，缩写为 GC

引用计数的实现要简单一些，我认为这也是Perl/PHP/Python 语言一开始使用这个技术，实现内存管理的原因。但是，随着时间的变更，引用计数的局限性越来越多。所以，上面的语言，最终都添加了完整的追踪GC 实现，或者有足够的 GC逻辑 周期性清理对象。

追踪垃圾回收具有可怕的名声。在原生内存级别工作，非常痛苦。调试 GC，会让你在梦里都还在想着16进制转储问题。但是，请记住，本书会带着我们一起驱散魔法，杀死怪兽，所以，我们也会编写自己的垃圾回收程序。我猜想，你一定会发现算法非常简单，并且整个程序非常有趣。

> In practice, ref counting and tracing are more ends of a continuum than opposing sides. Most ref counting systems end up doing some tracing to handle cycles, and the write barriers of a generational collector look a bit like retain calls if you squint.
>
> 在实践中，引用计数和追踪技术会混合使用，而不是相互对立。大多数的引用计数，都会进行一些周期性的追踪，如果你仔细查看，分代采集器的用法看起来像是保留调用。
> 
>For lots more on this, see [“A Unified Theory of Garbage Collection”](https://github.com/Kua-Fu/blog-book-images/blob/main/paper/unified-theory-gc.pdf) (PDF).
>
> 更多的信息，可以查看 [“垃圾收集的统一理论”](https://github.com/Kua-Fu/blog-book-images/blob/main/paper/unified-theory-gc.pdf)

## 三、Data Types

数据类型

In Lox’s little universe, the atoms that make up all matter are the built-in data types. There are only a few:

1. **Booleans**

	You can’t code without logic and you can’t logic without Boolean values. “True” and “false”, the yin and yang of software. Unlike some ancient languages that repurpose an existing type to represent truth and falsehood, Lox has a dedicated Boolean type. 
	
	There are two Boolean values, obviously, and a literal for each one.
	
	```
	
	true;  // Not false.
	false; // Not *not* false.

	```
	
	> Boolean variables are the only data type in Lox named after a person, George Boole, which is why “Boolean” is capitalized. He died in 1864, nearly a century before digital computers turned his algebra into electricity. I wonder what he’d think to see his name all over billions of lines of Java code.
	> 
	> 布尔变量是Lox语言中，唯一使用人名命名的数据类型。Boolean是为了纪念 George Boole, 他于1864年去世，一个世纪后，计算机科学将他发明的布尔代数，转换为计算机表示。我想知道，当他在数十亿Java代码中，看到自己的名字，会有什么感想😄
	
1. **Numbers**

	Lox has only one kind of number: double-precision floating point. Since floating-point numbers can also represent a wide range of integers, that covers a lot of territory, while keeping things simple.
	
	Full-featured languages have lots of syntax for numbers—hexadecimal, scientific notation, octal, all sorts of fun stuff. We’ll settle for basic integer and decimal literals.
	
	```
	
	1234;  // An integer.
	12.34; // A decimal number.

	```
	
1. **Strings**

	We’ve already seen one string literal in the first example. Like most languages, they are enclosed in double quotes.
	
	As we’ll see when we get to implementing them, there is quite a lot of complexity hiding in that innocuous sequence of characters.
	
	```
	
	"I am a string";
	"";    // The empty string.
	"123"; // This is a string, not a number.
	
	```
	
	> Even that word “character” is a trickster. Is it ASCII? Unicode? A code point or a “grapheme cluster”? How are characters encoded? Is each character a fixed size, or can they vary?
	>
	> 即使是单词，字节，也包含了一些隐藏信息，字节是ASCII编码的，还是Unicode编码；一个字节是一个代码点，还是一个图形集簇；字节是如何编码的，是定长编码，还是变长编码？

1. **Nil**

	There’s one last built-in value who’s never invited to the party but always seems to show up. It represents “no value”. It’s called “null” in many other languages. In Lox we spell it nil.  (When we get to implementing it, that will help distinguish when we’re talking about Lox’s nil versus Java or C’s null.)
	
	There are good arguments for not having a null value in a language since null pointer errors are the scourge of our industry. If we were doing a statically typed language, it would be worth trying to ban it. In a dynamically typed one, though, eliminating it is often more annoying than having it.




在Lox的小宇宙中，构成物质的原子是内置的几种数据类型，详见下面的介绍:

1. 布尔类型

   没有逻辑运算，我们无法进行编程，而逻辑运算，需要布尔数据类型。真与假，编程世界中的阴与阳，与一些早先语言复用其他数据类型表示布尔类型不同的是，我们会在Lox语言中定义一个专用的布尔数据类型。
   
   显然，布尔值有两个选择，每个值对应了一个文本
   
   ```
   true;  // Not false.
   false; // Not *not* false.
   ```

1. 数值类型

	Lox语言，只有一种数值类型，双精度浮点数。因为浮点数，可以包含大范围的整数，这样做，不但可以包含很多其他数据类型，而且可以保持简洁。
	
	功能齐全的语言，会有很多的数字语法，例如：十六进制表示、科学计数法、八进制表示，以及其他的有趣的东西。但是在Lox语言中，我们将只会满足基本的整数运算和十进制表示。
	
	```
	
	1234;  // An integer.
	12.34; // A decimal number.

	```
	
1. 字符串

	从上面的第一个示例中，我们可以看到字符串类型 "hello, world!"。 和大多数的语言一样，字符串类型的数据，需要使用双引号括起来。
	
	当我们实现字符串类型时候，将会看到，在这个定义明确的字符序列中，隐藏着许多复杂场景，需要特殊处理。
	
	```
	
	"I am a string";
	"";    // The empty string.
	"123"; // This is a string, not a number.
	
	```
	
1. Nil

	最后，还有一个数据类型，它似乎从来没有被邀请参加编程聚会，但是你总是能看到它的身影。它，代表了不存在，没有数值。在许多语言中，使用 null 表示它，而在Lox语言中，我们将使用 nil 表示它。当我们实现它时，可以更好的与 Java/C语言中的null 区别开。
	
	在语言中，不使用 nil 存在很多好处，空指针报错是编程行业常见的报错。如果，我们要实现一门静态类型语言，那么我们禁止 nil类型是值得的。但是，如果我们要实现一门动态类型语言，通常，保留 nil类型，拥有更多的好处，相比于禁止该类型。



















