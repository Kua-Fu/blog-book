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
	
## 八、 Functions

函数

A function call expression looks the same as it does in C.

```

makeBreakfast(bacon, eggs, toast);

```

You can also call a function without passing anything to it.

```

makeBreakfast();

```

Unlike in, say, Ruby, the parentheses are mandatory in this case. If you leave them off, the name doesn’t call the function, it just refers to it.

A language isn’t very fun if you can’t define your own functions. In Lox, you do that with fun.

```

fun printSum(a, b) {
  print a + b;
}

```


函数调用表达式和C语言中一样。

```

makeBreakfast(bacon, eggs, toast);

```

也可以在不传递任何参数的情况下，调用函数

```

makeBreakfast();

```

与 Ruby语言不一样，在这种场景，括号是必须的，如果不使用括号，不会实际调用函数，而只会引用函数。

如果一门语言无法自定义函数，那么这门语言也不会非常有意义。在Lox语言中，使用 fun 定义一个函数。

```

fun printSum(a, b) {
  print a + b;
}

```


Now’s a good time to clarify some terminology. Some people throw around “parameter” and “argument” like they are interchangeable and, to many, they are. We’re going to spend a lot of time splitting the finest of downy hairs around semantics, so let’s sharpen our words. From here on out:

* An argument is an actual value you pass to a function when you call it. So a function call has an argument list. Sometimes you hear actual parameter used for these.

* A parameter is a variable that holds the value of the argument inside the body of the function. Thus, a function declaration has a parameter list. Others call these formal parameters or simply formals.

The body of a function is always a block. Inside it, you can return a value using a return statement.

```

fun returnSum(a, b) {
  return a + b;
}

```

If execution reaches the end of the block without hitting a return, it implicitly returns nil.

现在是澄清一些术语的好时机。有些人，总会随意的使用 parameter 和 argument 两个术语，很多人会认为这两个术语含义相同，但是其实，它们之间有一些不同之处：

* argument 是调用函数时候，传递给函数的实际值，人们一般称为 实参，函数调用时候，会传递一系列的参数列表，称为实参列表

* parameter 是定义函数时候，保存参数值的变量，人们一般称为 形参，函数定义的时候，会定义一系列的参数列表，称为形参列表。

函数体始终是一个代码块。在函数体中，可以使用return 函数，返回一个值。

```

fun returnSum(a, b) {
  return a + b;
}

```

如果函数执行到末尾，没有return语句，会隐式的返回 nil

> See, I told you nil would sneak in when we weren’t looking.
>
> 看吧，我就说过 nil 总会在我们不注意的时候，偷偷溜进来。

> I’ve seen languages that use fn, fun, func, and function. I’m still hoping to discover a funct, functi, or functio somewhere.
>
> 我已经见过其他语言中，使用fn fun func function 关键字定义函数，我还在期待，有语言使用关键词 funct functi functio

> Speaking of terminology, some statically typed languages like C make a distinction between declaring a function and defining it. A declaration binds the function’s type to its name so that calls can be type-checked but does not provide a body. A definition declares the function and also fills in the body so that the function can be compiled
> 
> 说到术语，一些静态类型语言，例如：C语言，在函数声明和函数定义之间有不同之处。函数声明，函数名称关联着参数类型，以便再调用函数时候，检查参数类型，但是不定义函数体。函数定义，在声明函数的同时，还会定义函数体，以便可以编译函数。

> Since Lox is dynamically typed, this distinction isn’t meaningful. A function declaration fully specifies the function including its body.
>
> 因为Lox是一门动态语言，函数声明和函数定义没有区别，函数声明需要指定函数体。

### 8.1 Closures

闭包

Functions are first class in Lox, which just means they are real values that you can get a reference to, store in variables, pass around, etc. This works:

```

fun addPair(a, b) {
  return a + b;
}

fun identity(a) {
  return a;
}

print identity(addPair)(1, 2); // Prints "3".

```

Since function declarations are statements, you can declare local functions inside another function.

```

fun outerFunction() {
  fun localFunction() {
    print "I'm local!";
  }

  localFunction();
}


```

函数是Lox语言中的第一类变量，这意味着它们是拥有实际值的变量。我们可以引用/存储/传递这些函数。

```

fun addPair(a, b) {
  return a + b;
}

fun identity(a) {
  return a;
}

print identity(addPair)(1, 2); // Prints "3".

```

因为函数声明是语句，所以，我们可以在函数中，定义内部函数

```

fun outerFunction() {
  fun localFunction() {
    print "I'm local!";
  }

  localFunction();
}


```

If you combine local functions, first-class functions, and block scope, you run into this interesting situation:

```

fun returnFunction() {
  var outside = "outside";

  fun inner() {
    print outside;
  }

  return inner;
}

var fn = returnFunction();
fn();

```

Here, inner() accesses a local variable declared outside of its body in the surrounding function. Is this kosher? Now that lots of languages have borrowed this feature from Lisp, you probably know the answer is yes.

For that to work, inner() has to “hold on” to references to any surrounding variables that it uses so that they stay around even after the outer function has returned. We call functions that do this closures. These days, the term is often used for any first-class function, though it’s sort of a misnomer if the function doesn’t happen to close over any variables

As you can imagine, implementing these adds some complexity because we can no longer assume variable scope works strictly like a stack where local variables evaporate the moment the function returns. We’re going to have a fun time learning how to make these work correctly and efficiently.


如果将局部函数、一级函数、代码块作用域组合在一起，我们可以发现一些有意思的地方：

```

fun returnFunction() {
  var outside = "outside";

  fun inner() {
    print outside;
  }

  return inner;
}

var fn = returnFunction();
fn();

```

可以看到，inner函数，调用了外层函数中的局部变量 outside, 这样会报错吗？很多语言都借鉴了 lisp 语言的这种特性，答案是，我们可以使用外层函数中的局部变量。

为此，inner函数，必须包含对于周围变量的引用，这样即使外部函数已经返回，这些局部变量仍然会保留。我们将实现了这个功能的函数闭包。现如今，这个术语，闭包，经常用于第一类函数，尽管如何函数没有在任何变量上闭合，这个用词有些不太合适。

可以想象，实现这个闭包功能，会增加一些复杂性，因为我们不能假设变量，严格的像堆栈一样工作，当函数返回时候，局部变量就消失了。我们将有一段愉快的时光，学习如何正确有效的工作。

> Peter J. Landin coined the term “closure”. Yes, he invented damn near half the terms in programming languages. Most of them came out of one incredible paper, “The Next 700 Programming Languages”.
> 
> [Peter J. Landin](https://zh.wikipedia.org/zh-tw/%E5%BD%BC%E5%BE%97%C2%B7%E5%85%B0%E4%B8%81) 创造了闭包概念，是的，他几乎发明了一半的编程语言术语，其中大部分都出自那边重要论文 [The Next 700 Programming Languages](https://github.com/Kua-Fu/blog-book-images/blob/main/paper/The-Next-700-Programming-Languages.pdf)
>
> In order to implement these kind of functions, you need to create a data structure that bundles together the function’s code and the surrounding variables it needs. He called this a “closure” because it closes over and holds on to the variables it needs.
> 
> 为了实现这些函数，我们需要创建一个数据结构，将函数代码和函数所需要变量绑定在一起。他称这种数据结构为闭包，因为它包围着函数，并且包含了相关变量。

## 九、Classes

类

Since Lox has dynamic typing, lexical (roughly, “block”) scope, and closures, it’s about halfway to being a functional language. But as you’ll see, it’s also about halfway to being an object-oriented language. Both paradigms have a lot going for them, so I thought it was worth covering some of each.

Since classes have come under fire for not living up to their hype, let me first explain why I put them into Lox and this book. There are really two questions:

因为Lox语言具有动态类型，词法范围、闭包，因此，它已经一部分是函数式语言了，但是，正如我们将看到的，它也同样具有面向语言对象的一部分特性。这两种语言范式分别有很多优点，我们将简单介绍一些：

下面我将解释一下，为什么Lox语言具有面向对象特性：

### 9.1 Why might any language want to be object oriented?

为什么所有语言都想要面向对象特性？

Now that object-oriented languages like Java have sold out and only play arena shows, it’s not cool to like them anymore. Why would anyone make a new language with objects? Isn’t that like releasing music on 8-track?


It is true that the “all inheritance all the time” binge of the ’90s produced some monstrous class hierarchies, but object-oriented programming (OOP) is still pretty rad. Billions of lines of successful code have been written in OOP languages, shipping millions of apps to happy users. Likely a majority of working programmers today are using an object-oriented language. They can’t all be that wrong.

In particular, for a dynamically typed language, objects are pretty handy. We need some way of defining compound data types to bundle blobs of stuff together.

If we can also hang methods off of those, then we avoid the need to prefix all of our functions with the name of the data type they operate on to avoid colliding with similar functions for different types. In, say, Racket, you end up having to name your functions like hash-copy (to copy a hash table) and vector-copy (to copy a vector) so that they don’t step on each other. Methods are scoped to the object, so that problem goes away.

现在，面向对象语言，例如Java，已经普遍被使用，而且已经被接收到主流编程世界了，现在在喜欢它们，也不是很酷的事情了。为什么还有人用对象创造一门新语言，这好像是在8音节上，写出新的歌曲。

诚然，90年代的“一直继承”热潮产生了一些可怕的类层次结构，但是面向对象语言（OOP）仍然非常难实现。现在，面向对象语言，已经应用于数十亿行代码，分布于数百万的应用程序中，今天大多数的程序员都在使用面向对象语言。

特别的，对于动态类型，对象非常方便。我们需要一些方法，将一些复杂的数据类型绑定在一起。

如果我们可以挂起这些方法，那么，我们不需要在所有函数前面加上它们所操作的数据类型的名称，以避免和不同类型的类似函数发生冲突。例如：在Rocket语言中，你需要根据不同的数据类型，分别命名 hash-copy 和 vector-copy 函数，这样它们不会相互重叠。方法的作用域是对象，这样问题，就解决了。

### 9.2 Why is Lox object oriented?

为什么Lox语言是面向对象的？

I could claim objects are groovy but still out of scope for the book. Most programming language books, especially ones that try to implement a whole language, leave objects out. To me, that means the topic isn’t well covered. With such a widespread paradigm, that omission makes me sad.

Given how many of us spend all day using OOP languages, it seems like the world could use a little documentation on how to make one. As you’ll see, it turns out to be pretty interesting. Not as hard as you might fear, but not as simple as you might presume, either.

我可以说对象是 groovy语言，但是仍然超出了本书的范围。大多数的编程语言书籍，尤其是那些想要实现一门完整语言的书籍，都忽略了面向对象介绍。对我而言，这意味着面向对象这个话题，没有很好的被覆盖。在如此广泛的示例中，这种遗漏让我感到悲伤。

考虑到我们大部分人，每天都在使用面向对象语言，似乎全世界都需要一些关于面向对象语言的介绍。正如你看到的，结果非常有意思，既没有你想象的那么困难，但是也没你想象的那么简单。


### 9.3 Classes or prototypes

类和原型

When it comes to objects, there are actually two approaches to them, classes and prototypes. Classes came first, and are more common thanks to C++, Java, C#, and friends. Prototypes were a virtually forgotten offshoot until JavaScript accidentally took over the world.

In class-based languages, there are two core concepts: instances and classes. Instances store the state for each object and have a reference to the instance’s class. Classes contain the methods and inheritance chain. To call a method on an instance, there is always a level of indirection. You look up the instance’s class and then you find the method there:

Prototype-based languages merge these two concepts. There are only objects—no classes—and each individual object may contain state and methods. Objects can directly inherit from each other (or “delegate to” in prototypal lingo):

对于对象，实际上有两种方法实现，类和原型。类，更加通用，因为C++， Java，C#，firends等语言。原型几乎是一个被遗忘的分支，直到JavaScript 使用原型实现了面向对象。

在基于类的面向对象语言中，有两个核心概念：实例和类，实例中保存每个对象的状态，而且具有对实例类的引用。类，包含实现方法和继承链。如果通过实例，调用方法，总是存在一定程度上的间接。我们需要先找到实例对应的类，然后找到类的方法。

![class look-up](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/class-lookup.png?raw=true)


基于原型的面向对象语言中，融合了实例和类，这两个概念。它们只有对象，没有类，每个单独的对象包含状态和方法。对象可以直接相互继承（或者用原型中的术语，委托给）


![class look-up](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/prototype-lookup.png?raw=true)

This means that in some ways prototypal languages are more fundamental than classes. They are really neat to implement because they’re so simple. Also, they can express lots of unusual patterns that classes steer you away from.

But I’ve looked at a lot of code written in prototypal languages—including  [some of my own devising](http://finch.stuffwithstuff.com/index.html). Do you know what people generally do with all of the power and flexibility of prototypes?  . . . They use them to reinvent classes.

I don’t know why that is, but people naturally seem to prefer a class-based (Classic? Classy?) style. Prototypes are simpler in the language, but they seem to accomplish that only by pushing the complexity onto the user. So, for Lox, we’ll save our users the trouble and bake classes right in.

这意味着某些方面，基于原型的语言，比基于类的语言，更加基础，它们实现起来非常简洁。此外，我们也可以实现很多不寻常的模式，而这些模式，通常基于类的语言都会尽量远离。

但是我看到很多，使用原型语言编写的代码，包括我自己的一些设计。你知道，人们通常如何利用原型的强大功能和灵活性吗？他们使用原型来改造类。

我不知道为什么会这样，人们似乎更加喜欢基于类的风格，尽管基于原型的语言更加容易理解。但是，基于原型的语言，似乎将更多的复杂性转移到用户身上了，因此，对于Lox语言，我们将省去这些麻烦，一开始就使用基于类的面向对象。

> In a statically typed language like C++, method lookup typically happens at compile time based on the static type of the instance, giving you static dispatch. In contrast, dynamic dispatch looks up the class of the actual instance object at runtime. This is how virtual methods in statically typed languages and all methods in a dynamically typed language like Lox work.
> 
> 在C++ 这样的静态语言中，查找实例方法，一般是在编译阶段进行的，在编译时候，根据实例的静态类型，实现静态调度。相反的，动态调度，是在运行时候，才会去查看实例对应的对象类。 这就是，静态语言中的虚拟方法，和动态语言中的所有方法的工作方式。

> In practice the line between class-based and prototype-based languages blurs. JavaScript’s “constructor function” notion pushes you pretty hard towards defining class-like objects. Meanwhile, class-based Ruby is perfectly happy to let you attach methods to individual instances.
>
> 在实践中，基于类和基于原型的语言之间的界限已经模糊了，JavaScript 的构造函数，让你很难定义一个基于类的对象；同样的，基于类的Ruby，非常乐意让你把方法添加到具体的实例上。

### 9.4 Classes in Lox

Lox语言中的类

Enough rationale, let’s see what we actually have. Classes encompass a constellation of features in most languages. For Lox, I’ve selected what I think are the brightest stars. You declare a class and its methods like so:

```

class Breakfast {
  cook() {
    print "Eggs a-fryin'!";
  }

  serve(who) {
    print "Enjoy your breakfast, " + who + ".";
  }
}

```

The body of a class contains its methods. They look like function declarations but without the fun keyword. When the class declaration is executed, Lox creates a class object and stores that in a variable named after the class. Just like functions, classes are first class in Lox.

```

// Store it in variables.
var someVariable = Breakfast;

// Pass it to functions.
someFunction(Breakfast);

```

Next, we need a way to create instances. We could add some sort of new keyword, but to keep things simple, in Lox the class itself is a factory function for instances. Call a class like a function, and it produces a new instance of itself.

```

var breakfast = Breakfast();
print breakfast; // "Breakfast instance".

```

有了足够的理由，让我们看看Lox语言是如何实现的。在大多数的语言中，类包含了一系列特征，在Lox中，我选择了我认为的最闪亮的特性，我们可以这样定义一个类和它的方法：


```

class Breakfast {
  cook() {
    print "Eggs a-fryin'!";
  }

  serve(who) {
    print "Enjoy your breakfast, " + who + ".";
  }
}

```

类的主体，包含了它的方法，它们看起来像是函数声明，但是没有fun关键字。当执行类声明时候，Lox创建了一个类对象，并且将它保存在以类命名的变量中。像函数一样，类也是第一类对象。


```

// Store it in variables.
var someVariable = Breakfast;

// Pass it to functions.
someFunction(Breakfast);

```

接下来，我们需要一种创建实例的方法。我们可以添加一些关键字，但是为了简单起见，在Lox语言中，类本身作为实例的工厂函数。像函数一样调用类，就会产生一个实例。

```

var breakfast = Breakfast();
print breakfast; // "Breakfast instance".

```

> Larry Wall, Perl’s inventor/prophet calls this the [“waterbed theory”](http://wiki.c2.com/?WaterbedTheory). Some complexity is essential and cannot be eliminated. If you push it down in one place, it swells up in another.
>
> Larry Wall, Perl语言创始人，称之为水床理论——有些复杂性是无法消除，必不可少的。如果你在一个地方推它，它会在另一个地方膨胀。
>
> Prototypal languages don’t so much eliminate the complexity of classes as they do make the user take that complexity by building their own class-like metaprogramming libraries.
>
> 原型语言并没有消除类的复杂性。而是，让用户自己构建元类编程库，来承担复杂性。

### 9.5 Instantiation and initialization

实例化和初始化

Classes that only have behavior aren’t super useful. The idea behind object-oriented programming is encapsulating behavior and state together. To do that, you need fields. Lox, like other dynamically typed languages, lets you freely add properties onto objects.


```

breakfast.meat = "sausage";
breakfast.bread = "sourdough";

```

Assigning to a field creates it if it doesn’t already exist.

If you want to access a field or method on the current object from within a method, you use good old this.


```

class Breakfast {
  serve(who) {
    print "Enjoy your " + this.meat + " and " +
        this.bread + ", " + who + ".";
  }

  // ...
}

```

只有方法的类，不是非常有用，面向对象背后的思想是，将行为和状态封装在一起，为此，需要字段，Lox和其他动态语言一样，允许你在对象上自由添加对象属性。

```

breakfast.meat = "sausage";
breakfast.bread = "sourdough";

```

如果某个字段不存在，则分配给该字段将创建该字段，如果想要从方法中访问，当前对象的字段或者方法，需要使用this关键字

```

class Breakfast {
  serve(who) {
    print "Enjoy your " + this.meat + " and " +
        this.bread + ", " + who + ".";
  }

  // ...
}

```

Part of encapsulating data within an object is ensuring the object is in a valid state when it’s created. To do that, you can define an initializer. If your class has a method named init(), it is called automatically when the object is constructed. Any parameters passed to the class are forwarded to its initializer.


```

class Breakfast {
  init(meat, bread) {
    this.meat = meat;
    this.bread = bread;
  }

  // ...
}

var baconAndToast = Breakfast("bacon", "toast");
baconAndToast.serve("Dear Reader");
// "Enjoy your bacon and toast, Dear Reader."

```


在对象中封装的部分数据，是为了确保对象在创建时候处于有效状态。为此，我们可以创建一个初始化函数，如果类有一个名为init()的方法，那么我们在创建实例时候，自动调用该方法。传递给类的任何参数，都将变为类的初始值。

```

class Breakfast {
  init(meat, bread) {
    this.meat = meat;
    this.bread = bread;
  }

  // ...
}

var baconAndToast = Breakfast("bacon", "toast");
baconAndToast.serve("Dear Reader");
// "Enjoy your bacon and toast, Dear Reader."

```

### 9.6 Inheritance

继承

Every object-oriented language lets you not only define methods, but reuse them across multiple classes or objects. For that, Lox supports single inheritance. When you declare a class, you can specify a class that it inherits from using a less-than (<) operator.

```

class Brunch < Breakfast {
  drink() {
    print "How about a Bloody Mary?";
  }
}

```

Here, Brunch is the derived class or subclass, and Breakfast is the base class or superclass.

每个面向对象语言，都允许你定义方法，并且在其他类或者对象中复用这些方法。Lox语言，同样支持单继承，声明类时候，可以使用运算符 < 表示要继承的类。

```

class Brunch < Breakfast {
  drink() {
    print "How about a Bloody Mary?";
  }
}

```

Every method defined in the superclass is also available to its subclasses.

```

var benedict = Brunch("ham", "English muffin");
benedict.serve("Noble Reader");

```

Even the init() method gets inherited. In practice, the subclass usually wants to define its own init() method too. But the original one also needs to be called so that the superclass can maintain its state. We need some way to call a method on our own instance without hitting our own methods.

As in Java, you use super for that.

```

class Brunch < Breakfast {
  init(meat, bread, drink) {
    super.init(meat, bread);
    this.drink = drink;
  }
}

```

由上面的类声明，我们可以称 Brunch 为派生类或者子类，称 Breakfast 为基类或者超类。

每个基类中定义的方法，在它的派生类中，也可以调用

```

var benedict = Brunch("ham", "English muffin");
benedict.serve("Noble Reader");

```


甚至init() 方法也可以被继承，实际上，子类通常也想要定义自己的init() 方法，但是也需要调用基类，以便于基类更新它的状态信息。我们需要一个方式，只调用超类中的某个方法，但是不调用自身的同名方法。在Java中，我们可以使用super实现。

```

class Brunch < Breakfast {
  init(meat, bread, drink) {
    super.init(meat, bread);
    this.drink = drink;
  }
}

```


That’s about it for object orientation. I tried to keep the feature set minimal. The structure of the book did force one compromise. Lox is not a pure object-oriented language. In a true OOP language every object is an instance of a class, even primitive values like numbers and Booleans.

Because we don’t implement classes until well after we start working with the built-in types, that would have been hard. So values of primitive types aren’t real objects in the sense of being instances of classes. They don’t have methods or properties. If I were trying to make Lox a real language for real users, I would fix that.

这就是面向对象的方法，我试图保持功能集最小化，本书结构迫使我做出一定的妥协。Lox不是一种纯粹的面向对象语言，在真正的面向对象语言中，每个对象都是一个类的实例，即使是数值和布尔类型这样的原始值，都是一个类的实例。

因为我们在开始使用内置类后，才实现类，所以，实现纯粹的面向对象，比较困难。因此，从类实例的意义上，基本类型的值不是实例。它们没有方法，也没有属性。如果，我想要让Lox变为一门用户可用的真正的语言，我会尝试解决这个问题。

> Why the < operator? I didn’t feel like introducing a new keyword like extends. Lox doesn’t use : for anything else so I didn’t want to reserve that either. Instead, I took a page from Ruby and used <.
> 
> 为什么要使用 < 表示继承，而不是使用 extends 这样的关键字。同样的，Lox语言也不使用 : 用于任何用途，我从Ruby语言中得到灵感，使用< 表示继承。
> 
> If you know any type theory, you’ll notice it’s not a totally arbitrary choice. Every instance of a subclass is an instance of its superclass too, but there may be instances of the superclass that are not instances of the subclass. That means, in the universe of objects, the set of subclass objects is smaller than the superclass’s set, though type nerds usually use <: for that relation.
> 
> 如果你知道一些类型理论，你会注意到，这不是一个完全武断的决定。子类的每个实例，同时，也是父类的一个实例。可能存在一个实例，是父类的实例，但不是其子类的实例。在对象宇宙中，子类对象集合小于父类对象集合，类型理论的书呆子，会使用 <: 表示这种关系。
> 
> Lox is different from C++, Java, and C#, which do not inherit constructors, but similar to Smalltalk and Ruby, which do.
> 
> 不同于 C++ Java C#, 它们不会继承父类的构造函数，Lox语言，更加类似 Smalltalk Ruby，会继承构造函数。

## 十、The Standard Library

标准库

We’re almost done. That’s the whole language, so all that’s left is the “core” or “standard” library—the set of functionality that is implemented directly in the interpreter and that all user-defined behavior is built on top of.

This is the saddest part of Lox. Its standard library goes beyond minimalism and veers close to outright nihilism. For the sample code in the book, we only need to demonstrate that code is running and doing what it’s supposed to do. For that, we already have the built-in print statement.

Later, when we start optimizing, we’ll write some benchmarks and see how long it takes to execute code. That means we need to track time, so we’ll define one built-in function, clock(), that returns the number of seconds since the program started.

我们差不多完成了Lox语言，这就是整个语言。剩下来的部分是核心库或者称为标准库——直接在解释器中实现的功能，所有用户定义的方法都建立在标准库上面。

这是Lox语言中最悲伤的地方，它的标准库超越了极简主义，转向了完全的虚无主义。对于书中的示例代码，我们只需要证明代码正在运行或者执行它应该执行的操作。为此，我们已经有了内置的print语句。

稍后，当我们优化时候，我们将编写一些基准测试代码，查看执行代码的耗时。这意味着我们需要跟踪时间，因此，我们定义一个内置函数 clock() , 它返回程序启动后的秒数。

And . . . that’s it. I know, right? It’s embarrassing.


If you wanted to turn Lox into an actual useful language, the very first thing you should do is flesh this out. String manipulation, trigonometric functions, file I/O, networking, heck, even reading input from the user would help. But we don’t need any of that for this book, and adding it wouldn’t teach you anything interesting, so I’ve left it out.

Don’t worry, we’ll have plenty of exciting stuff in the language itself to keep us busy.

如果你想把Lox语言变为一门真正有用的语言，第一件事情，就是充实Lox语言。字符串操作，三角函数，文件I/O，网络，检查，甚至读取用户输入都会有所帮助，但是本书将不会涉及。因为加上它们，不会增加任何有意义的地方，所以，我把它们删除了。

别担心，语言本身有很多令人兴奋的东西，让我们忙个不停。

## 十一、CHALLENGES

习题

1. Write some sample Lox programs and run them (you can use the implementations of Lox in my repository). Try to come up with edge case behavior I didn’t specify here. Does it do what you expect? Why or why not?

1. This informal introduction leaves a lot unspecified. List several open questions you have about the language’s syntax and semantics. What do you think the answers should be?

1. Lox is a pretty tiny language. What features do you think it is missing that would make it annoying to use for real programs? (Aside from the standard library, of course.)

1. 编写一些Lox程序并且运行它们， 尝试给出一些书中没有提及的边缘示例，它是否符合你的期望，并给出原因？

1. 本章的简短介绍留下了很多没有明确的地方，列出关于语言的语法和语义的几个开放性问题，你认为答案是什么？

1. Lox是一门很小的语言，你觉得还应该添加哪些新功能，这些功能的缺少，让你在实际应用时候感到恼火（除了标准库之外）


## 十二、DESIGN NOTE: EXPRESSIONS AND STATEMENTS

设计思想：表达式和语句

Lox has both expressions and statements. Some languages omit the latter. Instead, they treat declarations and control flow constructs as expressions too. These “everything is an expression” languages tend to have functional pedigrees and include most Lisps, SML, Haskell, Ruby, and CoffeeScript.

To do that, for each “statement-like” construct in the language, you need to decide what value it evaluates to. Some of those are easy:

* An if expression evaluates to the result of whichever branch is chosen. Likewise, a switch or other multi-way branch evaluates to whichever case is picked.

* A variable declaration evaluates to the value of the variable.

* A block evaluates to the result of the last expression in the sequence.

Some get a little stranger. What should a loop evaluate to? A while loop in CoffeeScript evaluates to an array containing each element that the body evaluated to. That can be handy, or a waste of memory if you don’t need the array.

You also have to decide how these statement-like expressions compose with other expressions—you have to fit them into the grammar’s precedence table. For example, Ruby allows:

puts 1 + if true then 2 else 3 end + 4

Lox语言既有表达式，也有语句。有些语言省略了语句，它们会将声明和控制流也当作表达式，这些具有“一切都是表达式“特性的语言，往往具有函数式语言特征，例如：LISP，SML，Haskell，Ruby 和 CoffeeScript

要做到这一点，对于语言中的每一个语句结构，我们需要确定语句的最终值。其中，有些语句很简单：

* if 语句的计算结果是所选分支的结果。同样的，switch语句或者其他多路分支，计算结果为根据情况选择的分支的计算结果

* 变量声明语句的结果，为变量的值

* 代码块的计算结果为序列中最后一个表达式的结果


还有一些语句，变得有些奇怪，循环语句的计算结果应该是什么？CoffeeScript 语言的循环语句的计算结果是一个数组，该数组包含了主体计算到的每个元素。这很方便，如果不使用这个数组，会浪费内存。

我们还需要确定一些类似语句的表达式，如何和其他表达式组合使用。我们需要将这些表达式放入语法的优先级表中。例如：Ruby语言中，

puts 1 + if true then 2 else 3 end + 4

这就是你所期待的吗？这就是用户期待的吗？这对于你自己设计语句有什么影响？需要注意，Ruby语言有个显示的关键词 end，表示表达式什么时候完成，如果没有end，+4部分可能被解析为else的一部分。

Turning every statement into an expression forces you to answer a few hairy questions like that. In return, you eliminate some redundancy. C has both blocks for sequencing statements, and the comma operator for sequencing expressions. It has both the if statement and the ?: conditional operator. If everything was an expression in C, you could unify each of those.

Languages that do away with statements usually also feature implicit returns—a function automatically returns whatever value its body evaluates to without need for some explicit return syntax. For small functions and methods, this is really handy. In fact, many languages that do have statements have added syntax like => to be able to define functions whose body is the result of evaluating a single expression.


将每个语句变为表达式，迫使我们回答上面的棘手问题，作为回报，这样做，消除了一些冗余。C语言中，既有语句，也包含表达式，例如：它既有if语句，也有三元运算符 ?: ，如果C语言中一切都是表达式，那么这两种写法将统一为一种。

不使用语句的语言，通常具有隐式返回特性——函数会自动返回计算结果，而不需要显示返回语句。对于小函数和方法，这样非常方便。事实上，很多存在语句的语言，都实现了=> 这样的语法，定义一个函数，函数体是一个表达式的计算结果


But making all functions work that way can be a little strange. If you aren’t careful, your function will leak a return value even if you only intend it to produce a side effect. In practice, though, users of these languages don’t find it to be a problem.


For Lox, I gave it statements for prosaic reasons. I picked a C-like syntax for familiarity’s sake, and trying to take the existing C statement syntax and interpret it like expressions gets weird pretty fast.









