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



## 四、Expressions

表达式

If built-in data types and their literals are atoms, then expressions must be the molecules. Most of these will be familiar.

如果内置的基础数据类型和它们的文字，是原子，那么表达式就是分子，下面将介绍各种表达式

### 4.1 Arithmetic

算术表达式

Lox features the basic arithmetic operators you know and love from C and other languages:

Lox语言具有C语言或者其他语言中，存在的基础算术表达式

```

add + me;
subtract - me;
multiply * me;
divide / me;

```

The subexpressions on either side of the operator are operands. Because there are two of them, these are called binary operators.(It has nothing to do with the ones-and-zeroes use of “binary”.) 

Because the operator is fixed in the middle of the operands, these are also called infix operators (as opposed to prefix operators where the operator comes before the operands, and postfix where it comes after).

One arithmetic operator is actually both an infix and a prefix one. The - operator can also be used to negate a number.

```
-negateMe;
```

All of these operators work on numbers, and it’s an error to pass any other types to them. The exception is the + operator—you can also pass it two strings to concatenate them.

运算符号，两边的子表达式，称为操作数。因为运算符号，有两个操作数，所以称为二元运算符。⚠️这里的二元，和二进制中的0或者1，没有关系。

因为运算符，固定在操作数的中间，所以，我们称之为中缀运算符。（与前缀运算符不同，前缀运算符位于表达式最前面，后缀运算符位于表达式最后。）

一元运算符，实际上可以是中缀运算符，也可以是前缀运算符。例如：- 可以表示负数

```
-negateMe;
```

上面的算术运算符的操作数，只能是数值类型，如果操作数是其他数据类型，表达式计算会报错。但是，有个例外，+运算符可以作用于字符串数据类型，两个字符串的 + 运算，表示连接这两个字符串。

> There are some operators that have more than two operands and the operators are interleaved between them. The only one in wide usage is the “conditional” or “ternary” operator of C and friends:
>
> condition ? thenArm : elseArm;
>
> Some call these mixfix operators. A few languages let you define your own operators and control how they are positioned—their “fixity”.
>
> 有些运算符，可以具有两个以上的操作数，运算符在这些操作数之间。唯一广泛使用的多元运算符是 C语言中的条件（三元）运算符。
>
> condition ? thenArm : elseArm;

### 4.2 Comparison and equality

比较运算符，相等

Moving along, we have a few more operators that always return a Boolean result. We can compare numbers (and only numbers), using Ye Olde Comparison Operators.

```
less < than;
lessThan <= orEqual;
greater > than;
greaterThan >= orEqual;
```

We can test two values of any kind for equality or inequality. Even different types. Values of different types are never equivalent.

```
1 == 2;         // false.
"cat" != "dog"; // true.
314 == "pi"; // false.
123 == "123"; // false.
```

I’m generally against implicit conversions.



接下来，我们将介绍几个返回布尔值的运算符。

我们可以使用 比较运算符，比较并且仅仅比较数值

我们也可以比较两个任意类型的数据，是否相等, 甚至这两个数据，不是相同的类型。不同数据类型的两个数据，肯定是不想等的🤔。

通常，我们不建议使用隐式转换。


### 4.3 Logical operators


1. The not operator, a prefix !, returns false if its operand is true, and vice versa.

   ```
   
   !true;  // false.
   !false; // true.
   
   ```

1. The other two logical operators really are control flow constructs in the guise of expressions. An and expression determines if two values are both true. It returns the left operand if it’s false, or the right operand otherwise.

	```
	
	true and false; // false.
	true and true;  // true.
	
	```
	
	
1.  And an or expression determines if either of two values (or both) are true. It returns the left operand if it is true and the right operand otherwise.

The reason and and or are like control flow structures is that they short-circuit. Not only does and return the left operand if it is false, it doesn’t even evaluate the right one in that case. Conversely (contrapositively?), if the left operand of an or is true, the right is skipped.

逻辑运算
	

1. 逻辑非运算符，使用 ！表示，如果操作数是true，则添加逻辑非的表达式，值为false，反之亦然。

	```

	!true;  // false.
	!false; // true.

	```

1. 另外两个逻辑运算符，与其说是表达式，更加准确的说法是，控制流。

	and 逻辑运算符，将确定两个操作数，是否都是true，如果左操作数是false， 则表达式返回左操作数的值；如果左操作数为true，则表达式返回右操作数的值。

	```

	true and false; // false.
	true and true;  // true.

	```

1. or 逻辑运算符，将确定两个操作数，是否包含true，如果左操作数为true，则表达式结果，返回左操作数；如果左操作数为false，表达式将会返回右操作数的值。

	```

	false or false; // false.
	true or false;  // true.

	```

and 和 or 逻辑运算符，本质上是控制流结构的原因是，它们是短路。当逻辑运算符是 and，左操作数值为 false，我们甚至不会去计算右操作数，直接返回左操作数的值。如果逻辑运算符是 or，左操作数值为 true，同样的，我们不会去计算右操作数，直接返回左操作数的值。

### 4.4 Precedence and grouping

优先级和分组

All of these operators have the same precedence and associativity that you’d expect coming from C. (When we get to parsing, we’ll get way more precise about that.) In cases where the precedence isn’t what you want, you can use () to group stuff.

```

var average = (min + max) / 2;

```

Since they aren’t very technically interesting, I’ve cut the remainder of the typical operator menagerie out of our little language. 

所有这些运算符的优先级和关联性，和C语言中的相同。当我们进入解析过程时候，将会更加理解这一点。如果，优先级不是我们想要的，可以使用 （）对内容进行分组。

因为在技术上不太有趣，在Lox语言中，我删除了一些典型的运算符，例如：位运算，位移，取模，条件运算符等。我不会让你们评分，但是如果你在Lox语言实现中，添加了这些运算符，在我心中，你将会大大加分。

Those are the expression forms (except for a couple related to specific features that we’ll get to later), so let’s move up a level.

上面介绍了表达式形式，除了一些我们下面将要介绍的、与特定功能相关的表达式，让我们继续学习。

## 五、Statements

语句

Now we’re at statements. Where an expression’s main job is to produce a value, a statement’s job is to produce an effect. Since, by definition, statements don’t evaluate to a value, to be useful they have to otherwise change the world in some way—usually modifying some state, reading input, or producing output.

You’ve seen a couple of kinds of statements already. The first one was:

```

print "Hello, world!";

```

A print statement evaluates a single expression and displays the result to the user. You’ve also seen some statements like:

```

"some expression";

```

An expression followed by a semicolon (;) promotes the expression to statement-hood. This is called (imaginatively enough), an expression statement.

If you want to pack a series of statements where a single one is expected, you can wrap them up in a block.

```

{
  print "One statement.";
  print "Two statements.";
}

```

Blocks also affect scoping, which leads us to the next section . . . 

现在，我们开始学习语句。表达式的主要作用是生成值，语句的主要任务是产生效果。因为，语句的结果不是一个具体的值，所以，为了能够有影响，语句的结果必须可以以某种方式改变编程世界，例如：修改某些状态，读取用户输入，产生输出。

你已经看到了几种不同类型的语句，例如:

```

print "Hello, world!";

```

一个print语句，计算出单个表达式，并且向用户展示计算结果。

还有一些其他类型的语句，例如：

```

"some expression";

```

表达式后面加上 ; 该表达式会变为表达式语句。

如果想将一系列语句，组合为一条语句，可以使用{}, 将这些语句打包为一个块

```

{
  print "One statement.";
  print "Two statements.";
}

```

语法块，还影响生命周期，马上我们将会看到。

> Baking print into the language instead of just making it a core library function is a hack. But it’s a useful hack for us: it means our in-progress interpreter can start producing output before we’ve implemented all of the machinery required to define functions, look them up by name, and call them.
>
> Lox语言中，将print表示为语句，而不是核心函数库中的一个print函数，是一种黑客行为。这对我们来说是一个有用的技巧：这意味着，解释器运行时，我们可以在定义函数之前，实现输出功能。按照名称查找并且调用它们。

## 六、Variables

变量

You declare variables using var statements. If you omit the initializer, the variable’s value defaults to nil.

```

var imAVariable = "here is my value";
var iAmNil;

```

Once declared, you can, naturally, access and assign a variable using its name.

```

var breakfast = "bagels";
print breakfast; // "bagels".
breakfast = "beignets";
print breakfast; // "beignets".

```

I won’t get into the rules for variable scope here, because we’re going to spend a surprising amount of time in later chapters mapping every square inch of the rules. In most cases, it works like you would expect coming from C or Java.

可以使用 var语句，声明变量。如果变量省略初始化，该变量的默认值是nil。

一旦声明，我们可以使用变量名，访问和分配变量。

在当前章节，我不会讨论变量的有效使用范围，因为，我们将在后面的章节中，花费很多篇幅讲解变量的使用范围。在多数场景下，Lox语言中，变量的使用范围和 C/Java语言中的规则相同。

> This is one of those cases where not having nil and forcing every variable to be initialized to some value would be more annoying than dealing with nil itself.
>
> 这是nil存在的一种好处，如果我们强制要求任意的变量，都必须初始化为具体值，那么，这个强制初始化的过程比我们定一个一个nil类型，更加麻烦。
>
>Can you tell that I tend to work on this book in the morning before I’ve had anything to eat?
>
> 你可以看出来我倾向于，在吃早餐之前，开始写一会儿书吗？😂

## 七、Control Flow

控制流

It’s hard to write useful programs if you can’t skip some code or execute some more than once. That means control flow. In addition to the logical operators we already covered, Lox lifts three statements straight from C.

1. **if**

   An if statement executes one of two statements based on some condition.
   
   ```
   
   if (condition) {
	 print "yes";
   } else {
	 print "no";
   }

   ```
   
1. **while**

	A while loop executes the body repeatedly as long as the condition expression evaluates to true.
	
	```
	
	var a = 1;
	while (a < 10) {
	  print a;
	  a = a + 1;
	}

	```
	
1. **for**

	Finally, we have for loops. This loop does the same thing as the previous while loop. Most modern languages also have some sort of for-in or foreach loop for explicitly iterating over various sequence types. In a real language, that’s nicer than the crude C-style for loop we got here. Lox keeps it basic.
	
	```
	
	for (var a = 1; a < 10; a = a + 1) {
	  print a;
	}

	```
	
> We already have and and or for branching, and we could use recursion to repeat code, so that’s theoretically sufficient. It would be pretty awkward to program that way in an imperative-styled language, though.
>
> 我们已经有了 and 和 or 逻辑运算符，如果在加上递归调用函数，理论上可以实现重复执行代码。但是，这种函数式编程方式，非常难使用。

> Scheme, on the other hand, has no built-in looping constructs. It does rely on recursion for repetition. Smalltalk has no built-in branching constructs, and relies on dynamic dispatch for selectively executing code.
>
> lisp的方言，scheme, 没有内置的循环语句。它依赖递归执行，实现复用代码。Smalltalk语言，没有内置的分支语句，它依赖动态调度，来选择性的执行代码。

>I left do while loops out of Lox because they aren’t that common and wouldn’t teach you anything that you won’t already learn from while. Go ahead and add it to your implementation if it makes you happy. It’s your party.
>
> 我没有在Lox语言中，引入 do while循环语句，因为，该循环语句，和while语句，效果一致。如果你想在Lox语言中，实现该语句，那么非常欢迎，因为这是你的语言。

> This is a concession I made because of how the implementation is split across chapters. A for-in loop needs some sort of dynamic dispatch in the iterator protocol to handle different kinds of sequences, but we don’t get that until after we’re done with control flow. We could circle back and add for-in loops later, but I didn’t think doing so would teach you anything super interesting.
>
> 这是我做出的让步，我将在后面的章节中，添加 for-in 循环语句。我们需要在迭代器中，根据不同的数据类型，动态调度该数据序列。虽然，我们添加了 for-in语句，但是我并不认为，这个语句非常有趣。

如果语言不支持，跳过执行某些代码或者重复执行某些代码，那么，我们很难写出有用的程序。而这些，表明我们需要引入控制流。除了上面介绍的逻辑运算符之外，我们还从C语言中直接借用了3种控制流。

1. if

	if语句，根据条件，选择执行其中的一个语句。
	
    ```
   
   if (condition) {
	 print "yes";
   } else {
	 print "no";
   }

   ```

1. while

	只要while 语句中，表达式执行结果为true，循环语句会一直执行。
	
	```

	var a = 1;
	while (a < 10) {
	  print a;
	  a = a + 1;
	}

	```
	
1. for

	最后，我们介绍for 循环语句。
	
	```
	
	for (var a = 1; a < 10; a = a + 1) {
	  print a;
	}

	```
	
	for循环语句，和 while循环语句，效果相同。大多数现代语句，还支持for-in，foreach语句，用于迭代各种序列类型数据。在实际编程中，这种新的语句，比C语言的for语句，更加简洁。但是，我们Lox语言，将保持最原始的for语句。




	





















