# 解析表达式

> Grammar, which knows how to control even kings.
>
> 语法，知道如何控制国王。
>
> <p align="right">—— Molière </p>


This chapter marks the first major milestone of the book. Many of us have cobbled together a mishmash of regular expressions and substring operations to extract some sense out of a pile of text. The code was probably riddled with bugs and a beast to maintain. Writing a real parser—one with decent error handling, a coherent internal structure, and the ability to robustly chew through a sophisticated syntax—is considered a rare, impressive skill. In this chapter, you will attain it.

这一章是本书的一个重要里程碑，我们中的许多人都拼凑了一堆正则表达式和子字符串处理方法，以便在一堆文本中获取意义。代码可能存在漏洞，需要不断维护。编写一个真正的解析器——具有良好的错误处理能力，连贯的内部结构，对复杂语法的良好处理能力，被认为是一种罕见的、令人印象深刻的能力。在本章，我们将实现这个目标。

> “Parse” comes to English from the Old French “pars” for “part of speech”. It means to take a text and map each word to the grammar of the language. We use it here in the same sense, except that our language is a little more modern than Old French.
> 
> Like many rites of passage, you’ll probably find it looks a little smaller, a little less daunting when it’s behind you than when it loomed ahead.
>
> parse 单词是从古法语中来的，它表示我们取一段文字，将每一个单词映射到语言的语法中，我们在这里使用的是，相同的含义，但是，我们要实现的语言更加现代一些，相比于古法语。
>
> 像是很多成人仪式一样，你可能会发现，这个看起来有点小，比它隐约出现在你的面前时候，令人更少一些畏惧。

It’s easier than you think, partially because we front-loaded a lot of the hard work in the last chapter. You already know your way around a formal grammar. You’re familiar with syntax trees, and we have some Java classes to represent them. The only remaining piece is parsing—transmogrifying a sequence of tokens into one of those syntax trees.

Some CS textbooks make a big deal out of parsers. In the ’60s, computer scientists—understandably tired of programming in assembly language—started designing more sophisticated, human-friendly languages like Fortran and ALGOL. Alas, they weren’t very machine-friendly for the primitive computers of the time.



这个解析过程，比你想要的要简单一些，因为我们在上一章中，提前完成了很多艰苦的工作。你已经熟悉了正则语法，而且我们使用了很多的Java类来表示这些表达式。剩下的解析部分——将一系列的token解析为语法树。

有一些编程教科书，对解析器部分做了很大的改动。上世纪60年代初，计算机科学家对于汇编编程感到厌倦，这是可以理解的，于是，他们开始设计更加高级、人性化的语言，例如：Fortran ,Algol等等，虽然对于当时的原始计算机，这些语言不是很友好。

> Imagine how harrowing assembly programming on those old machines must have been that they considered Fortran to be an improvement.
>
> 想象一下，在那些老机器上，进行汇编语言编程是一件多么痛苦的事情，所以，计算机科学家们，认为使用Fortran语言编程，是一种改进。

These pioneers designed languages that they honestly weren’t even sure how to write compilers for, and then did groundbreaking work inventing parsing and compiling techniques that could handle these new, big languages on those old, tiny machines.

Classic compiler books read like fawning hagiographies of these heroes and their tools. The cover of Compilers: Principles, Techniques, and Tools literally has a dragon labeled “complexity of compiler design” being slain by a knight bearing a sword and shield branded “LALR parser generator” and “syntax directed translation”. They laid it on thick.

A little self-congratulation is well-deserved, but the truth is you don’t need to know most of that stuff to bang out a high quality parser for a modern machine. As always, I encourage you to broaden your education and take it in later, but this book omits the trophy case.

这些计算机领域的先驱者们，设计了他们甚至还不知道如何编写编译器的语言，他们接着发明了解析、编译技术，在老旧的机器上处理新的语言。

经典的编译器书籍，读起来像是，这些先驱及其发明工具的传记。《编译器：原理、技术和工具》的封面是一条龙，龙身上写着编译器设计的复杂性，被一个手持盾牌和宝剑的骑士杀死。骑上盾牌上写着：LALR解析器生成器，语法向导翻译，这个非常重要。

有一些自鸣得意是理所当然的，但事实是，我们不需要了解其中的大部分内容，就可以为现代机器，设计出高质量的解析器。向之前一样，我鼓励你扩大学习范围，然后深入研究，本书省略奖杯案例。

![compiler-dragon](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/compiler-dragon.png?raw=true)

## 一、Ambiguity and the Parsing Game

歧义和解析博弈

In the last chapter, I said you can “play” a context-free grammar like a game in order to generate strings. Parsers play that game in reverse. Given a string—a series of tokens—we map those tokens to terminals in the grammar to figure out which rules could have generated that string.

The “could have” part is interesting. It’s entirely possible to create a grammar that is ambiguous, where different choices of productions can lead to the same string. When you’re using the grammar to generate strings, that doesn’t matter much. Once you have the string, who cares how you got to it?

在上一章中，我们可以像玩游戏一样，通过上下文无关语法，生成一系列字符串，本章中，解析器将反其道而行之。给定一个字符串（一系列token）我们将这些token 映射到语法规则中，确定哪些规则，可以生成这样的字符串。

”可能有“ 部分（找出合适语法规则的过程）非常有意思，我们完全可以创建一个存在歧义的语法规则，不同的选择，却可以生成相同的字符串，当我们确定了语法规则后，这个过程将不再重要，一旦我们有了字符串，没有人会在乎我们是如何生成它的。

When parsing, ambiguity means the parser may misunderstand the user’s code. As we parse, we aren’t just determining if the string is valid Lox code, we’re also tracking which rules match which parts of it so that we know what part of the language each token belongs to. Here’s the Lox expression grammar we put together in the last chapter:

当解析时候，歧义意味着解析器可能会误解用户的代码。在解析时候，我们不仅仅要确定，字符串是否是，有效的lox语言代码，还需要跟踪哪些语法规则和字符串的哪些部分匹配，便于我们获取token属于具体的哪个部分，下面是我们之前总结的lox语法规则

```

expression     → literal
               | unary
               | binary
               | grouping ;

literal        → NUMBER | STRING | "true" | "false" | "nil" ;
grouping       → "(" expression ")" ;
unary          → ( "-" | "!" ) expression ;
binary         → expression operator expression ;
operator       → "==" | "!=" | "<" | "<=" | ">" | ">="
               | "+"  | "-"  | "*" | "/" ;
			   
```

This is a valid string in that grammar:

![tokens](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/tokens.png?raw=true)

But there are two ways we could have generated it. One way is:

1. Starting at expression, pick binary.

1. For the left-hand expression, pick NUMBER, and use 6.

1. For the operator, pick "/".

1. For the right-hand expression, pick binary again.

1. In that nested binary expression, pick 3 - 1.

Another is:

1. Starting at expression, pick binary.

1. For the left-hand expression, pick binary again.

1. In that nested binary expression, pick 6 / 3.

1. Back at the outer binary, for the operator, pick "-".

1. For the right-hand expression, pick NUMBER, and use 1.

Those produce the same strings, but not the same syntax trees:



下面一个合法的lox代码，但是我们可以使用两种方式生成, 生成的语法树如下: 

![syntax-trees](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/syntax-trees.png?raw=true)


In other words, the grammar allows seeing the expression as (6 / 3) - 1 or 6 / (3 - 1). The binary rule lets operands nest any which way you want. That in turn affects the result of evaluating the parsed tree. The way mathematicians have addressed this ambiguity since blackboards were first invented is by defining rules for precedence and associativity.

* Precedence 

	determines which operator is evaluated first in an expression containing a mixture of different operators. Precedence rules tell us that we evaluate the / before the - in the above example. Operators with higher precedence are evaluated before operators with lower precedence. Equivalently, higher precedence operators are said to “bind tighter”.

* Associativity 
  
     determines which operator is evaluated first in a series of the same operator. When an operator is left-associative (think “left-to-right”), operators on the left evaluate before those on the right. Since - is left-associative, this expression:
	 
	 Assignment, on the other hand, is right-associative. This:


换句话说，语法规则，可以将 

` 6/3-1 ` 视为 `(6/3)-1`  或者 `6/(3-1)`

二叉树允许我们按照任意的方式嵌套，而这又会反过来影响，我们的解析过程，自从黑板被发明出来，数学家解决这个歧义的方式是：定义优先级，关联性规则。

* 优先级

	优先级确定了包含不同运算符的表达式中，我们首先计算哪个运算符。优先级规则告知我们，/ 优先于 -， 优先级高的运算符先计算；等价的，优先级较高的运算符，可以称为绑定更紧。
	
* 关联性

	关联性决定了在一些相同优先级的运算符中，如何计算。当一个操作符是左关联的（即从左往右计算），左边的运算符的会先被计算，如果是左关联，下面的表达式一致
	
```

5-3-1

``` 

```

(5-3)-1

```


赋值操作是右关联的，即下面的表达式一致
	
	
```
a = b = c

```

```
a = (b = c)

```

> While not common these days, some languages specify that certain pairs of operators have no relative precedence. That makes it a syntax error to mix those operators in an expression without using explicit grouping.
>
>Likewise, some operators are non-associative. That means it’s an error to use that operator more than once in a sequence. For example, Perl’s range operator isn’t associative, so a .. b is OK, but a .. b .. c is an error.
>
> 虽然现在不常见，但是有一些语言，没有指定运算符的相对优先级，如果在一个表达式中包含不同的运算符，并且没有使用显示的分组，解析器会报错，语法错误。
>
> 同样，有一些运算符是非关联的。这意味着，在一个序列中，多次使用该运算符是错误的，举例: Perl语言中范围运算符是非关联的，即 
> a...b 是合法的，但是 a...b...c 是非法的

Without well-defined precedence and associativity, an expression that uses multiple operators is ambiguous—it can be parsed into different syntax trees, which could in turn evaluate to different results. We’ll fix that in Lox by applying the same precedence rules as C, going from lowest to highest.

如果没有定义运算符的优先级和关联性，那么使用多个运算符的表达式可能会引起歧义。这样的表达式，会生成不同的语法树，执行这些不同的语法树，会产生不同的结果。我们将在Lox语言中，定义和C语言，相同的运算符优先级。下面的优先级，从低到高

|Name| Operators| Associates|
|---|---|---|
|Equality| == != | Left|
|Comparison| > >= < <= | Left|
|Term| + - | Left|
| Factor | * / | Left|
|Unary| ! - | Right|

Right now, the grammar stuffs all expression types into a single expression rule. That same rule is used as the non-terminal for operands, which lets the grammar accept any kind of expression as a subexpression, regardless of whether the precedence rules allow it.


现在，语法规则将所有的表达式类型，填充到单个表达式规则中。相同的规则，可以被当作操作数的非终止符，这使得语法可以接受任何类型的表达式当作子表达式，而不管优先级是否允许。

We fix that by stratifying the grammar. We define a separate rule for each precedence level.

我们通过语法分层来解决这个问题，我们为每个优先级定义了专门的规则。

```

expression     → ...
equality       → ...
comparison     → ...
term           → ...
factor         → ...
unary          → ...
primary        → ...

```

> Instead of baking precedence right into the grammar rules, some parser generators let you keep the same ambiguous-but-simple grammar and then add in a little explicit operator precedence metadata on the side in order to disambiguate.
>
> 有一些解析生成器，没有将优先级正确的添加到语法规则中，而是让你保持相同的模糊但是简单的语法，然后在旁边添加一些显式的运算符优先级元数据，用于消除歧义。


Each rule here only matches expressions at its precedence level or higher. For example, unary matches a unary expression like !negated or a primary expression like 1234. And term can match 1 + 2 but also 3 * 4 / 5. The final primary rule covers the highest-precedence forms—literals and parenthesized expressions.

We just need to fill in the productions for each of those rules. We’ll do the easy ones first. The top expression rule matches any expression at any precedence level. Since equality has the lowest precedence, if we match that, then it covers everything.



上面的规则，只会匹配相同或更高优先级的表达式，例如：

一元 类型表达式可以匹配一个相同优先级的表达式 !unary 或者 像 1234 这样的主表达式

术语表达式可以匹配 1+2  或者 3*4/5 这样的表达式

最后的规则，即定义了主规则拥有最高的优先级，主规则匹配的类型为文字表达式和括号表达式


我们这需要填充每一个规则，让我们先从最简单的开始。顶级表达式可以匹配任何优先级的表达式，因为相等是最低优先级的表达式，如果我们匹配到它，我们将覆盖所有

```

expression     → equality

```

> We could eliminate expression and simply use equality in the other rules that contain expressions, but using expression makes those other rules read a little better.
> 
> Also, in later chapters when we expand the grammar to include assignment and logical operators, we’ll only need to change the production for expression instead of touching every rule that contains an expression.
> 
> 我们可以取消相等表达式，在其他语法规则中，包含等式，但是，使用等式表达式，可以让其他语法规则更加容易理解。
>
> 此外，在后面的章节中，我们会扩展语法，包含赋值、逻辑运算符，我们只需要修改等式表达式的定义，不需要修改每一条其他语法规则。

Over at the other end of the precedence table, a primary expression contains all the literals and grouping expressions.

在优先级表格的另外一边，主表达式包含文本和括号表达式

```

primary        → NUMBER | STRING | "true" | "false" | "nil"
               | "(" expression ")" ;

```

A unary expression starts with a unary operator followed by the operand. Since unary operators can nest—!!true is a valid if weird expression—the operand can itself be a unary operator. A recursive rule handles that nicely.

一元表达式是以一元运算符开始，后面跟随操作数。因为一元运算符可以嵌套 - ！

例如: !true 是一个奇怪但是有效的表达式，

我们使用递归规则可以很好的处理

```

unary          → ( "!" | "-" ) unary ;

```

But this rule has a problem. It never terminates.

Remember, each rule needs to match expressions at that precedence level or higher, so we also need to let this match a primary expression.

但是一元表达式规则，有个问题，它可能永远不会终止。请记住，每一个规则匹配优先级相同或者更高的表达式，因此，我们还需要匹配主表达式

```C

unary          → ( "!" | "-" ) unary
               | primary ;

```

That works.

上面的规则生效了。

The remaining rules are all binary operators. We’ll start with the rule for multiplication and division. Here’s a first try:

其余规则都是二元运算符，我们首先从乘法和除法开始，下面是第一次尝试

```c

factor         → factor ( "/" | "*" ) unary
               | unary ;
			   
```

The rule recurses to match the left operand. That enables the rule to match a series of multiplication and division expressions like 1 * 2 / 3. Putting the recursive production on the left side and unary on the right makes the rule left-associative and unambiguous.

All of this is correct, but the fact that the first symbol in the body of the rule is the same as the head of the rule means this production is left-recursive. Some parsing techniques, including the one we’re going to use, have trouble with left recursion. (Recursion elsewhere, like we have in unary and the indirect recursion for grouping in primary are not a problem.)

该规则会以左关联，递归调用，这样，这条语法规则，将会产生一系列的乘法除法表达式，例如：1*2/3

我们将递归放在左侧，一元运算规则放在右侧，是为了使得该规则左关联，以确保规则是明确没有歧义的

所有这一切都是正确的，当规则正文中的第一个符号和规则标题相同的时候，意味着这个规则是左递归的，有一些解析技术，包括我们将要使用的技术，在解析左递归方面存在一些问题。其他地方的递归，例如：一元中的递归，以及在主表达式中用于分组的间接递归都不是问题

> In principle, it doesn’t matter whether you treat multiplication as left- or right-associative—you get the same result either way. Alas, in the real world with limited precision, roundoff and overflow mean that associativity can affect the result of a sequence of multiplications. Consider:
>
> `print 0.1 * (0.2 * 0.3);`
>
> `print (0.1 * 0.2) * 0.3;`
>
> In languages like Lox that use [IEEE 754](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) double-precision floating-point numbers, the first evaluates to 0.006, while the second yields `0.006000000000000001`.
>
> Sometimes that tiny difference matters. This is a good place to learn more.
> 
> 原则上，无论我们将乘法，设置为左关联/右关联，结果都是相同的，但实际上，在现实世界中，数值型计算会有溢出和舍入，例如:
>
> 在Lox语言中，我们遵守 IEEE 754双精度规则，第一个表达式计算结果是 0.006, 第二个表达式计算结果是 0.006000000000000001
>
> 有时候，这个微小的差别非常重要，需要注意

There are many grammars you can define that match the same language. The choice for how to model a particular language is partially a matter of taste and partially a pragmatic one. This rule is correct, but not optimal for how we intend to parse it. Instead of a left recursive rule, we’ll use a different one.

我们可以定义许多语法，和同一个语言相匹配。如果为一门语言建模，一部分取决于编程品味，一部分取决于应用。这个规则是正确的，但是，对于我们如何解析它来说不是最佳的，我们将使用新的规则，而不是使用左递归规则

```c

factor         → unary ( ( "/" | "*" ) unary )* ;

```

We define a factor expression as a flat sequence of multiplications and divisions. This matches the same syntax as the previous rule, but better mirrors the code we’ll write to parse Lox. We use the same structure for all of the other binary operator precedence levels, giving us this complete expression grammar:

我们将因子表达式定义为乘法和除法混合的平面的序列。这个规则和前面的规则，匹配相同的语法，但是，更好的反映了我们将要编写的解析Lox的代码。我们对所有的其他二元运算符优先级，使用相同结构，从而，我们将获得完整的表达式语法

```C

expression -> equality;

equality -> comparison (( "!=" | "==" ) comparison )* ;

comparison -> term ( ( ">" | ">=" | "<" | "<=" ) term)* ;

term           → factor ( ( "-" | "+" ) factor )* ;

factor         → unary ( ( "/" | "*" ) unary )* ;

unary          → ( "!" | "-" ) unary
	           | primary ;
			   
primary        → NUMBER | STRING | "true" | "false" | "nil"
	           | "(" expression ")" ;
			   
```


This grammar is more complex than the one we had before, but in return we have eliminated the previous one’s ambiguity. It’s just what we need to make a parser.

上面的语法规则，比我们之前的语法规则更加复杂，但是作为回报，我们消除了之前的歧义，这就是我们想要的解析器。

## 二、Recursive Descent Parsing

递归下降分析

There is a whole pack of parsing techniques whose names are mostly combinations of “L” and “R”—LL(k), LR(1), LALR—along with more exotic beasts like parser combinators, Earley parsers, the shunting yard algorithm, and packrat parsing. For our first interpreter, one technique is more than sufficient: recursive descent.

Recursive descent is the simplest way to build a parser, and doesn’t require using complex parser generator tools like Yacc, Bison or ANTLR. All you need is straightforward handwritten code. Don’t be fooled by its simplicity, though. Recursive descent parsers are fast, robust, and can support sophisticated error handling. In fact, GCC, V8 (the JavaScript VM in Chrome), Roslyn (the C# compiler written in C#) and many other heavyweight production language implementations use recursive descent. It rocks.

Recursive descent is considered a top-down parser because it starts from the top or outermost grammar rule (here expression) and works its way down into the nested subexpressions before finally reaching the leaves of the syntax tree. This is in contrast with bottom-up parsers like LR that start with primary expressions and compose them into larger and larger chunks of syntax.



有一整套的解析技术，其名称大多是L R 的组合——[LL(k)](https://en.wikipedia.org/wiki/LL_parser) , [LR(1)](https://en.wikipedia.org/wiki/LR_parser) , [LALR](https://en.wikipedia.org/wiki/LALR_parser), 此外，还有一些更奇特的组合，[语法分析组合子 parser combinators](https://en.wikipedia.org/wiki/Parser_combinator), [Earley parser](https://en.wikipedia.org/wiki/Earley_parser), [调度场算法 the shunting yard algorithm](https://zh.wikipedia.org/wiki/%E8%B0%83%E5%BA%A6%E5%9C%BA%E7%AE%97%E6%B3%95), [解析表达文法 packrat parsing](https://zh.wikipedia.org/wiki/%E8%A7%A3%E6%9E%90%E8%A1%A8%E8%BE%BE%E6%96%87%E6%B3%95)。 但是对于我们的第一个解析器，使用一种技巧就足够了: 递归下降分析

递归下降是构造解析器的最简单方法，而且我们不需要使用 yacc, Bison, ANTLR 这样的生成器工具。我们需要的仅仅是简单的手写代码，但是，不要被它的简单所迷惑。递归下降的解析器，速度快，代码健壮，可以支持复杂的错误处理。事实上，很多重量级的解析器，都是用递归下降分析，例如：GCC， V8（chrome中JavaScript的VM），Roslyn（C#编写的C#编译器）

递归下降被认为是一种自顶向下的解析器，因为它是从最上面或者最外面的语法规则（上文中的 expression规则）开始，一直向下到嵌套的子表达式，最后到达，语法树的叶子节点，这与自底向上的解析器，形成对比。例如：自底向上的解析器LR，会从最底层的 primary开始，然后，一直组合子表达式，语法块越来越大。

> It’s called “recursive descent” because it walks down the grammar. Confusingly, we also use direction metaphorically when talking about “high” and “low” precedence, but the orientation is reversed. In a top-down parser, you reach the lowest-precedence expressions first because they may in turn contain subexpressions of higher precedence.
>
> ![direction](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/direction.png?raw=true)
>
> CS people really need to get together and straighten out their metaphors. Don’t even get me started on which direction a stack grows or why trees have their roots on top.
> 
> 这种解析技术，被称为递归下降，因为它遵循这语法，令人困惑的是，当我们谈论优先级的高低时候，我们也会使用方向来比喻，但是方向相反。在自上而下的解析器中，我们将首先到达优先级最低的表达式，因为优先级低的表达式中，可能包含着优先级较高的子表达式
>
> 计算机科学领域，真的需要统一一下表述。甚至，不要让我一开始在了解语法树方向时候，就困惑，为什么树根在最上面。

A recursive descent parser is a literal translation of the grammar’s rules straight into imperative code. Each rule becomes a function. The body of the rule translates to code roughly like:

|Grammar notation|Code representation|
|---|---|
|Terminal|Code to match and consume a token|
|Nonterminal|Call to that rule’s function|
| || if or switch statement|
|* or +|while or for loop|
|?|if statement|

The descent is described as “recursive” because when a grammar rule refers to itself—directly or indirectly—that translates to a recursive function call.

递归下降解析器是将语法规则直接翻译为命令式代码的直译，每个规则都成为一个函数，规则的转换可以参考：

下降将被描述为“递归”，因为当语法规则直接或者间接的引用自身时候，它会转化为递归函数调用

### 2.1 The parser class

解析类

Each grammar rule becomes a method inside this new class:

每个语法规则都会变为解析类中的一个方法


```java

// lox/Parser.java, create new file


package com.craftinginterpreters.lox;

import java.util.List;

import static com.craftinginterpreters.lox.TokenType.*;

class Parser {
  private final List<Token> tokens;
  private int current = 0;

  Parser(List<Token> tokens) {
    this.tokens = tokens;
  }
}

```


Like the scanner, the parser consumes a flat input sequence, only now we’re reading tokens instead of characters. We store the list of tokens and use current to point to the next token eagerly waiting to be parsed.

和扫描器一样，解析器使用输入的序列，只不过我们从扫描器的字符序列，变为了解析器的token序列，在类中，我们会存储 token序列，并且使用 current, 表示下一个等待解析的token

We’re going to run straight through the expression grammar now and translate each rule to Java code. The first rule, expression, simply expands to the equality rule, so that’s straightforward.

现在，我们将直接运行表达式语法，并将每个语法规则转为Java 代码，第一条规则，表达式，简单的扩展为等式规则，所以这个非常简单

```java 

// lox/Parser.java, add after Parser()

  private Expr expression() {
    return equality();
  }
  
```

Each method for parsing a grammar rule produces a syntax tree for that rule and returns it to the caller. When the body of the rule contains a nonterminal—a reference to another rule—we call that other rule’s method.

The rule for equality is a little more complex.

每一个语法规则解析方法，都为该语法规则生成一个语法树，并且将语法树，返回给调用者，当规则中包含有一个非终止符时候——对另一个规则的引用，我们将调用另外一个规则。

> This is why left recursion is problematic for recursive descent. The function for a left-recursive rule immediately calls itself, which calls itself again, and so on, until the parser hits a stack overflow and dies.
> 
> 这就是为什么左递归，对于递归下降是有问题的原因，左递归语法规则，将会立即调用自身，然后一直无限循环，直到解析器遇到堆栈溢出，程序崩溃

下面等式的规则有些复杂

```

equality       → comparison ( ( "!=" | "==" ) comparison )* ;

```

In Java, that becomes:

使用 Java实现，变为：

```java

// lox/Parser.java, add after expression()

  private Expr equality() {
    Expr expr = comparison();

    while (match(BANG_EQUAL, EQUAL_EQUAL)) {
      Token operator = previous();
      Expr right = comparison();
      expr = new Expr.Binary(expr, operator, right);
    }

    return expr;
  }


```

Let’s step through it. The first comparison nonterminal in the body translates to the first call to comparison() in the method. We take that result and store it in a local variable.

Then, the ( ... )* loop in the rule maps to a while loop. We need to know when to exit that loop. We can see that inside the rule, we must first find either a != or == token. So, if we don’t see one of those, we must be done with the sequence of equality operators. We express that check using a handy match() method.


让我们逐一了解，语法规则中的 comparison 非终止符，将变为解析类中的一个方法—— comparison(), 我们将调用 comparison() ，并且返回结果，存储在局部变量中。

接下来，语法规则中的 ( ... ) * 循环将变为Java代码中的 while 循环，我们需要知道什么时候，退出循环，从语法规则中，可以发现，我们需要先找到一个 != 或者  == 的 token, 所以，如果我们看不到 == 或者 != , 我们必须对等式运算符序列进行处理，我们将简单的使用一个match() 函数，实现该检查

```java 

// lox/Parser.java, add after equality()

  private boolean match(TokenType... types) {
    for (TokenType type : types) {
      if (check(type)) {
        advance();
        return true;
      }
    }

    return false;
  }
  
```


This checks to see if the current token has any of the given types. If so, it consumes the token and returns true. Otherwise, it returns false and leaves the current token alone. The match() method is defined in terms of two more fundamental operations.

The check() method returns true if the current token is of the given type. Unlike match(), it never consumes the token, it only looks at it.

The advance() method consumes the current token and returns it, similar to how our scanner’s corresponding method crawled through characters.

These methods bottom out on the last handful of primitive operations.

这个match()函数，将检查当前的token，是否是参数中的token类型，如果当前token满足给定的类型，将返回 true, 否则，将返回false，并且保留在当前的位置，match() 函数中，将会调用其他两个函数 check() , advance()

如果当前的token符合给定的 token类型，check() 方法将返回true， 与match() 方法不一样，check()方法，不会前移token（及发现满足条件后，current前移一位），只会检查token类型

advance() 方法使用current位置的token，并且返回，类似于我们在扫描器中，对应的字符级别的 advance() 方法

advance() 方法，将会使用到下面的基本方法

```java

// lox/Parser.java, add after match()

  private boolean check(TokenType type) {
    if (isAtEnd()) return false;
    return peek().type == type;
  }


```

```java

// lox/Parser.java, add after check()

  private Token advance() {
    if (!isAtEnd()) current++;
    return previous();
  }


```

```java

// lox/Parser.java, add after advance()

  private boolean isAtEnd() {
    return peek().type == EOF;
  }

  private Token peek() {
    return tokens.get(current);
  }

  private Token previous() {
    return tokens.get(current - 1);
  }


```

![func](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/parse-fun.png?raw=true)


isAtEnd() checks if we’ve run out of tokens to parse. peek() returns the current token we have yet to consume, and previous() returns the most recently consumed token. The latter makes it easier to use match() and then access the just-matched token.

isAtEnd() 方法，检查token是否已经使用完，peek(） 返回current位置的token，previous() 返回最近使用的token，previous() 方法，让 match() 函数更加容易，访问匹配的token

That’s most of the parsing infrastructure we need. Where were we? Right, so if we are inside the while loop in equality(), then we know we have found a != or == operator and must be parsing an equality expression.

We grab the matched operator token so we can track which kind of equality expression we have. Then we call comparison() again to parse the right-hand operand. We combine the operator and its two operands into a new Expr.Binary syntax tree node, and then loop around. For each iteration, we store the resulting expression back in the same expr local variable. As we zip through a sequence of equality expressions, that creates a left-associative nested tree of binary operator nodes.

这就是我们需要使用的大部分的，解析基础结构，我们在哪里？如果我们在 equality() 方法的while循环中，那么我们需要找到 != 或者 == 运算符，并且必须解析等式表达式。

我们将获取匹配到的运算符token（ == 或者 != ), 以便跟踪我们拥有哪种等式表达式，然后，我们将再次调用 comparison() 方法，来解析右边的操作数，我们将两个操作数和运算符，组合为一个新的二元表达式语法树，然后，循环，对于每一次迭代，我们都将结果保存在相同的expr 局部变量中，

当我们创建一系列的等式表达式时候，我们将创建一个二元运算符节点的左关联嵌套树。

![sequence](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/sequence.png?raw=true)

> Parsing a == b == c == d == e. For each iteration, we create a new binary expression using the previous one as the left operand.
> 
> 解析 a == b == c == d == e, 我们每一次都会使用前一个计算结果，当作后面的二元表达式的左值，例如：我们先计算 a == b，将结果当作 x == c 中的 x

The parser falls out of the loop once it hits a token that’s not an equality operator. Finally, it returns the expression. Note that if the parser never encounters an equality operator, then it never enters the loop. In that case, the equality() method effectively calls and returns comparison(). In that way, this method matches an equality operator or anything of higher precedence.

一旦解析器碰到不是等式运算符的token， 它就会退出循环，最后，解析器会返回表达式，需要注意的是，如果解析器没有遇到等式运算符，它永远不会进入循环，在这种情况下，equaility() 函数，调用会返回 comparison(). 通过上面的方式，此方法将会匹配相等运算符，或者任何优先级较高的运算符。

Moving on to the next rule . . . 

```

comparison     → term ( ( ">" | ">=" | "<" | "<=" ) term )* ;

```

Translated to Java:

```java 

// lox/Parser.java add after equality()

  private Expr comparison() {
    Expr expr = term();

    while (match(GREATER, GREATER_EQUAL, LESS, LESS_EQUAL)) {
      Token operator = previous();
      Expr right = term();
      expr = new Expr.Binary(expr, operator, right);
    }

    return expr;
  }
  
```

The grammar rule is virtually identical to equality and so is the corresponding code. The only differences are the token types for the operators we match, and the method we call for the operands—now term() instead of comparison(). The remaining two binary operator rules follow the same pattern.

继续进行下个语法规则


语法规则实际等同于相等表达式，相应的代码也是如此，唯一的区别是我们匹配的运算符token的类型，以及，我们为了操作数调用的方法是 term() 而不是 comparison() , 另外的两个二元运算符规则，遵循相同的模式

> If you wanted to do some clever Java 8, you could create a helper method for parsing a left-associative series of binary operators given a list of token types, and an operand method handle to simplify this redundant code.
> 
> 如果不想一直这样构造函数，可以通过一个helper函数，生成对应的解析方法

In order of precedence, first addition and subtraction:

按照运算优先级，首先考虑加法和减法运算

```java

// lox/Parser.java add after comparison()

  private Expr term() {
    Expr expr = factor();

    while (match(MINUS, PLUS)) {
      Token operator = previous();
      Expr right = factor();
      expr = new Expr.Binary(expr, operator, right);
    }

    return expr;
  }
  
```

And finally, multiplication and division:

最后，考虑乘法和除法

```java

//lox/Parser.java add after term()

 private Expr factor() {
    Expr expr = unary();

    while (match(SLASH, STAR)) {
      Token operator = previous();
      Expr right = unary();
      expr = new Expr.Binary(expr, operator, right);
    }

    return expr;
  }
  
```

That’s all of the binary operators, parsed with the correct precedence and associativity. We’re crawling up the precedence hierarchy and now we’ve reached the unary operators.

这就是所有的二元运算符，以正确的优先级和关联性解析，我们正在往优先级更高的运算符进行解析，下一步，我们将解析一元运算符

```

unary          → ( "!" | "-" ) unary
               | primary ;
			   
```

The code for this is a little different.

代码看起来有点不一样

```java

// lox/Parser.java, add after factor()

  private Expr unary() {
    if (match(BANG, MINUS)) {
      Token operator = previous();
      Expr right = unary();
      return new Expr.Unary(operator, right);
    }

    return primary();
  }
  
```


Again, we look at the current token to see how to parse. If it’s a ! or -, we must have a unary expression. In that case, we grab the token and then recursively call unary() again to parse the operand. Wrap that all up in a unary expression syntax tree and we’re done.

我们查看当前的 token，判断如何解析； 如果当前的 token 是 ！或者 - 等一元运算符，我们将获得一个一元表达式，在这种情况下，我们通过 previous() 获取操作符，还需要通过再次调用 unary() 获取一元运算符的右值，即操作数。然后，我们将获取到一个一元运算符语法树。

> The fact that the parser looks ahead at upcoming tokens to decide how to parse puts recursive descent into the category of predictive parsers.
> 
> 事实上，解析器会提前查看当前位置的token，决定如何接下来的解析流程，递归下降由此被归类为，预测解析器的范畴。

Otherwise, we must have reached the highest level of precedence, primary expressions.

接下来，我们将进入主表达式的解析方法，主表达式是优先级最高的表达式

```

primary        → NUMBER | STRING | "true" | "false" | "nil"
               | "(" expression ")" ;
			   
```

Most of the cases for the rule are single terminals, so parsing is straightforward.

这个表达式的大多数情况，是一个单终结符，因此解析方法很简单。


```java

// lox/Parser.java, add after unary()

 private Expr primary() {
    if (match(FALSE)) return new Expr.Literal(false);
    if (match(TRUE)) return new Expr.Literal(true);
    if (match(NIL)) return new Expr.Literal(null);

    if (match(NUMBER, STRING)) {
      return new Expr.Literal(previous().literal);
    }

    if (match(LEFT_PAREN)) {
      Expr expr = expression();
      consume(RIGHT_PAREN, "Expect ')' after expression.");
      return new Expr.Grouping(expr);
    }
  }
  
```

The interesting branch is the one for handling parentheses. After we match an opening ( and parse the expression inside it, we must find a ) token. If we don’t, that’s an error.

有趣的分支是处理括号的分支，在匹配到一个左括号后，我们必须找到一个对应的括号，如果没有找到对应的括号，解析将报错。


##  三、Syntax Errors

语法错误

A parser really has two jobs:

1. Given a valid sequence of tokens, produce a corresponding syntax tree.

1. Given an invalid sequence of tokens, detect any errors and tell the user about their mistakes.

解析器实际上有两项任务，


1. 给定一个合法有效 token 序列，生成一个语法树

1. 给定一个非法的 token 序列，检测错误，并且通知用户详细错误

Don’t underestimate how important the second job is! In modern IDEs and editors, the parser is constantly reparsing code—often while the user is still editing it—in order to syntax highlight and support things like auto-complete. That means it will encounter code in incomplete, half-wrong states all the time.

When the user doesn’t realize the syntax is wrong, it is up to the parser to help guide them back onto the right path.  The way it reports errors is a large part of your language’s user interface. Good syntax error handling is hard. By definition, the code isn’t in a well-defined state, so there’s no infallible way to know what the user meant to write. The parser can’t read your mind.

不要低估第二个任务的重要性，解析器经常在用户在用户仍在编辑代码时候，同步分析代码，用于支持代码高亮和自动补全功能。这意味着，分析的代码将始终处于不完整、半错误状态的代码。

当用户还没有注意到语法错误时候，解析器将帮助他们返回正确的语法，语法错误的提示，是IDE 用户界面的很大一部分，通常很好的处理报错是一项艰难的任务，根据定义，代码没有处于定义良好的状态，因此，我们没有可靠的方式，获知用户将要输入什么，毕竟，解析器无法获知用户的思想。

> Not yet at least. With the way things are going in machine learning these days, who knows what the future will bring?
>
> 至少现在还没有，随着机器学习的发展，我们并不知道未来会变成什么样？🐶

There are a couple of hard requirements for when the parser runs into a syntax error. A parser must:

* **Detect and report the error.**  

	If it doesn’t detect the error and passes the resulting malformed syntax tree on to the interpreter, all manner of horrors may be summoned.
	
* **Avoid crashing or hanging.**


	Syntax errors are a fact of life, and language tools have to be robust in the face of them. Segfaulting or getting stuck in an infinite loop isn’t allowed. While the source may not be valid code, it’s still a valid input to the parser because users use the parser to learn what syntax is allowed.


当解析器遇到语法错误时候，有几个硬性要求，

* 检测并且报告错误，

	如果没有及时检测到错误，并且报告错误，而是将错误格式的语法树返回给解析器，那么可能引发各种各样的报错。
	
* 避免程序挂起或者崩溃
  
    语法错误是生活中的一个事实，面对这些错误，语言解析器必须是健壮的，不能出现 [存储器段错误](https://zh.wikipedia.org/zh-cn/%E8%A8%98%E6%86%B6%E9%AB%94%E5%8D%80%E6%AE%B5%E9%8C%AF%E8%AA%A4) 或者无限循环。虽然，源代码不是有效的代码，但是它仍然是解析器的有效输入，因为用户通过解析器了解到哪些语法是允许的。
	

Those are the table stakes if you want to get in the parser game at all, but you really want to raise the ante beyond that. A decent parser should:

* **Be fast. **

  Computers are thousands of times faster than they were when parser technology was first invented. The days of needing to optimize your parser so that it could get through an entire source file during a coffee break are over. But programmer expectations have risen as quickly, if not faster. They expect their editors to reparse files in milliseconds after every keystroke.
  
* **Report as many distinct errors as there are. **

	Aborting after the first error is easy to implement, but it’s annoying for users if every time they fix what they think is the one error in a file, a new one appears. They want to see them all.
	
* **Minimize cascaded errors.**

	Once a single error is found, the parser no longer really knows what’s going on. It tries to get itself back on track and keep going, but if it gets confused, it may report a slew of ghost errors that don’t indicate other real problems in the code. When the first error is fixed, those phantoms disappear, because they reflect only the parser’s own confusion. Cascaded errors are annoying because they can scare the user into thinking their code is in a worse state than it is.
	
The last two points are in tension. We want to report as many separate errors as we can, but we don’t want to report ones that are merely side effects of an earlier one.


上面两点是入局解析器游戏的基本条件，但是，如果你想提高自己的筹码，一个更好的解析器应该:

* 编译更快 

	计算机的运行速度，比解析器技术，首次发明的时候，快了数千倍。需要优化解析器编译速度，需要喝完一整杯咖啡才能编译完成的日子，已经过去了。但是，程序员的期望值也在不断变高，他们希望在毫秒内重新编译源码。

* 报告尽可能多的明显错误

	在发现第一个错误时候，马上终止解析器，很容易实现，但是，如果用户每次只能发现代码中的一个错误，一次次的重新修改，会给用户很不好的体验，他们希望一次发现所有的明显错误。

* 最小化级联错误

	一旦发现了一个错误，解析器不再真的知道，发生了什么。解析器试图让解析回到正轨，并且继续解析，但是如果它无法前进，可能会发现大量的关联错误，这些错误并不表示，这些错误处是真正的问题所在。当第一个错误被发现并且修复后，大量的关联错误将会消失，因为它们不是真的错误。级联错误非常烦人，因为，客户通常会认为，自己的代码状态比实际情况差多了。
	
后面的两点要求，有一定的竞争关系。我们既希望，报告尽可能多的明显错误，但是，我们也希望，报告尽可能少的关联错误。


The way a parser responds to an error and keeps going to look for later errors is called error recovery. This was a hot research topic in the ’60s. Back then, you’d hand a stack of punch cards to the secretary and come back the next day to see if the compiler succeeded. With an iteration loop that slow, you really wanted to find every single error in your code in one pass.

Today, when parsers complete before you’ve even finished typing, it’s less of an issue. Simple, fast error recovery is fine.

解析器响应错误，并且继续查找后续错误的方式，称为错误恢复。这是60年代的热门研究课题。在那个年代，你会把一叠穿孔卡纸，交给相关人员，然后，在第二天，去查看编译器是否执行成功。由于迭代循环如何缓慢，我们希望在一次编译中发现尽可能完整的错误。

今天，解析器通常在你还没有编辑完成，就已经完成解析，问题不是很大，简单、快速的错误恢复更好。


### 3.1 Panic mode error recovery

恐慌模式错误恢复

> You know you want to push it.
> 
> ![panic](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/panic.png?raw=true)
>
> 你想要按下它 😄

Of all the recovery techniques devised in yesteryear, the one that best stood the test of time is called—somewhat alarmingly—panic mode. As soon as the parser detects an error, it enters panic mode. It knows at least one token doesn’t make sense given its current state in the middle of some stack of grammar productions.

Before it can get back to parsing, it needs to get its state and the sequence of forthcoming tokens aligned such that the next token does match the rule being parsed. This process is called synchronization.

To do that, we select some rule in the grammar that will mark the synchronization point. The parser fixes its parsing state by jumping out of any nested productions until it gets back to that rule. Then it synchronizes the token stream by discarding tokens until it reaches one that can appear at that point in the rule.

Any additional real syntax errors hiding in those discarded tokens aren’t reported, but it also means that any mistaken cascaded errors that are side effects of the initial error aren’t falsely reported either, which is a decent trade-off.

The traditional place in the grammar to synchronize is between statements. We don’t have those yet, so we won’t actually synchronize in this chapter, but we’ll get the machinery in place for later.


在过去发明的所有错误恢复技术中，最禁得起时间考验的是恐慌模式的错误恢复技术。一旦解析器检测到错误，它将进入到恐慌模式，它知道至少有一个 token，没有实际意义，当我们需要获取语法生成的堆栈中的当前状态。

在返回解析之前，它需要获取其状态，并且和即将解析的 token序列对齐，以便下一个token 和当前的解析规则匹配，这个过程称之为同步。

为此，我们在语法中，选择一些规则，来标记同步点。解析器，肯能会跳出一些嵌套循环，用于恢复其解析状态，直到返回该解析规则。然后，解析器通过丢弃token，来同步token 流，直到它到达可以出现在规则中的那个点。

隐藏在这些被丢弃的token中的，任何其他真实的错误都不会被报告，这也意外着，任何错误的级联错误（初始错误的副作用）也不会被错误报告，这是一个不错的权衡。

语法中的传统同步位置，是语句之间。我们还没有涉及到，所以我们将不会在本章中实现同步，但是后面章节会实现。

### 3.2 Entering panic mode

进入恐慌模式

Back before we went on this side trip around error recovery, we were writing the code to parse a parenthesized expression. After parsing the expression, the parser looks for the closing ) by calling consume(). Here, finally, is that method:

在我们讨论错误恢复之前，我们的解析器正在，编写代码解析带括号的表达式。解析表达式以后，我们通过调用 consume()方法，来查找后面的 ），最后的方法如下：

```java 

// lox/Parser.java, add after match()

  private Token consume(TokenType type, String message) {
    if (check(type)) return advance();

    throw error(peek(), message);
  }
  
```


It’s similar to match() in that it checks to see if the next token is of the expected type. If so, it consumes the token and everything is groovy. If some other token is there, then we’ve hit an error. We report it by calling this:


consume() 方法和match() 方法类似，它们都会检查当前 token 是否是预期的类型，如果是合适的类型，consume() 方法会消耗一个位置，一切都会正常运行。如果不是合适的类型，将会产生一个错误


```java

// lox/Parser.java, add after previous()

  private ParseError error(Token token, String message) {
    Lox.error(token, message);
    return new ParseError();
  }
  
```

First, that shows the error to the user by calling:

首先，我们通过调用下面方法，向用户报告错误

```java

// lox/Lox.java, add after report()

  static void error(Token token, String message) {
    if (token.type == TokenType.EOF) {
      report(token.line, " at end", message);
    } else {
      report(token.line, " at '" + token.lexeme + "'", message);
    }
  }
  
```

This reports an error at a given token. It shows the token’s location and the token itself. This will come in handy later since we use tokens throughout the interpreter to track locations in code.

这个方法，将报告给定token 的具体错误，它将会报告token 的位置和token 的字符值，这将在以后派上用场，因为我们在这个解释器中，使用token来定位追踪代码的位置

After we report the error, the user knows about their mistake, but what does the parser do next? Back in error(), we create and return a ParseError, an instance of this new class:

在我们报告了错误后，用户知道了他们代码中的错误，但是，接下来，解析器将如何运行呢？回到error() 方法，我们将创建并且返回一个新的类实例，ParseError()

```java

// lox/Parser.java, nest inside class Parser


class Parser {
  private static class ParseError extends RuntimeException {}

  private final List<Token> tokens;
  
```

This is a simple sentinel class we use to unwind the parser.  The error() method returns the error instead of throwing it because we want to let the calling method inside the parser decide whether to unwind or not.  Some parse errors occur in places where the parser isn’t likely to get into a weird state and we don’t need to synchronize. In those places, we simply report the error and keep on truckin’.

For example, Lox limits the number of arguments you can pass to a function. If you pass too many, the parser needs to report that error, but it can and should simply keep on parsing the extra arguments instead of freaking out and going into panic mode.

这是一个简单的哨兵类，用于展开解析器。error() 方法将返回错误，而不是抛出错误，因为，我们希望让解析器内部的调用方法决定是否展开。有一些解析错误，发生在解析器不太可能进入奇怪状态，并且也不需要我们同步的地方，在这些地方，我们只需要报告错误，并且继续运行。

举例，如果Lox 限制了函数可以传参的个数，如果你传递了太多的参数，解析器需要报告该错误，但是，解析器，接下来应该解析额外的参数，而不是马上进入恐慌模式


> Another way to handle common syntax errors is with error productions. You augment the grammar with a rule that successfully matches the erroneous syntax. The parser safely parses it but then reports it as an error instead of producing a syntax tree.
>
> For example, some languages have a unary + operator, like +123, but Lox does not. Instead of getting confused when the parser stumbles onto a + at the beginning of an expression, we could extend the unary rule to allow it.
> 
> ```
> unary → ( "!" | "-" | "+" ) unary
>      | primary ;
> ```
> 
> This lets the parser consume + without going into panic mode or leaving the parser in a weird state.
> 
> Error productions work well because you, the parser author, know how the code is wrong and what the user was likely trying to do. That means you can give a more helpful message to get the user back on track, like, “Unary ‘+’ expressions are not supported.” Mature parsers tend to accumulate error productions like barnacles since they help users fix common mistakes.
> 
> 另外一种处理常见语法错误的报错是，我们可以使用成功匹配错误语法的规则来扩充语法，解析器安全的解析它，但是随后，我们将其报告为错误，而不是生成语法树。
> 
> 举个例子，有些语言，支持一个一元运算符 +，例如： +123， 但是Lox语言没有。所以，当解析器在表达式的开始，遇到一个+时候，我们可以扩充语法规则，添加一个新的一元运算符，而不是解析报错。
> 
> 这样，解析器可以正常解析+，不会进入到恐慌模式，或者解析器出现奇怪的状态。
> 
> 错误生成工作的很好，因为作为解析器的作者，我们知道代码是如何产生错误的，以及用户可能想要做什么。这意味着，你可能给出更加有用的信息，让用户更好的修改错误，例如：不支持一元运算符 +， 成熟的解析器，倾向于像藤壶一样积累错误，因为，它可以让用户更好的修复常见错误。


In our case, though, the syntax error is nasty enough that we want to panic and synchronize. Discarding tokens is pretty easy, but how do we synchronize the parser’s own state?

然而，在我们的例子中，语法错误非常严重，以至于我们想要进入恐慌模式，并且同步。丢弃token非常简单，但是我们如何同步解析器自己的状态呢？

### 3.3 Synchronizing a recursive descent parser

同步递归下降解析器

With recursive descent, the parser’s state—which rules it is in the middle of recognizing—is not stored explicitly in fields. Instead, we use Java’s own call stack to track what the parser is doing. Each rule in the middle of being parsed is a call frame on the stack. In order to reset that state, we need to clear out those call frames.

The natural way to do that in Java is exceptions. When we want to synchronize, we throw that ParseError object. Higher up in the method for the grammar rule we are synchronizing to, we’ll catch it. Since we synchronize on statement boundaries, we’ll catch the exception there. After the exception is caught, the parser is in the right state. All that’s left is to synchronize the tokens.

通过递归下降，解析器在识别过程中的状态不会显示存储在字段中，相反，我们将使用Java自己的堆栈来跟踪解析器，正在做什么。正在解析的每一个规则，都是堆栈上的调用帧。为了重置该状态，我们需要清除这些调用帧。

在Java中，这样做的自然是使用 Exception，当我们想要同步时候，我们将抛出 ParseError对象。在我们想要同步的语法规则，对应的方法的上层，我们将会捕捉到 ParseError对象。因为我们在语句的边界处同步，所以，我们将会在那里，捕获到异常。捕获到异常后，解析器将进入正常的状态，剩下的就是同步token。

We want to discard tokens until we’re right at the beginning of the next statement. That boundary is pretty easy to spot—it’s one of the main reasons we picked it. After a semicolon, we’re probably finished with a statement. Most statements start with a keyword—for, if, return, var, etc. When the next token is any of those, we’re probably about to start a statement.

我们希望在解析下一条语句之前，丢弃token。这个边界非常容易被发现，这也是我们使用它的主要原因。分号之后，我们可能完成了一个语句。大多数的语法，以 `for if return var `等token开始，如果下一个解析的token是其中之一，那么，很可能我们将开始一个语句。

> I say “probably” because we could hit a semicolon separating clauses in a for loop. Our synchronization isn’t perfect, but that’s OK. We’ve already reported the first error precisely, so everything after that is kind of “best effort”.
> 
> 我说可能，是因为，我们可以在for 循环中，使用分号，分隔字句，我们的同步并不完美，耽没有关系，我们已经准确的报告了第一个错误。所以，之后的都是尽力而为。

This method encapsulates that logic:

下面的方法封装了该逻辑。


```java

  private void synchronize() {
    advance();

    while (!isAtEnd()) {
      if (previous().type == SEMICOLON) return;

      switch (peek().type) {
        case CLASS:
        case FUN:
        case VAR:
        case FOR:
        case IF:
        case WHILE:
        case PRINT:
        case RETURN:
          return;
      }

      advance();
    }
  }

```


It discards tokens until it thinks it has found a statement boundary. After catching a ParseError, we’ll call this and then we are hopefully back in sync. When it works well, we have discarded tokens that would have likely caused cascaded errors anyway, and now we can parse the rest of the file starting at the next statement.

Alas, we don’t get to see this method in action, since we don’t have statements yet. We’ll get to that in a couple of chapters. For now, if an error occurs, we’ll panic and unwind all the way to the top and stop parsing. Since we can parse only a single expression anyway, that’s no big loss.

一直到我们找到了语句边界，我们才会丢掉token，在捕获到 ParseError对象后，我们将调用此函数，然后，我们有希望能恢复同步。当它可以正常工作的时候，我们丢弃了可能会导致级联错误的token，现在，我们可以从下一条语句，开始解析文件的剩余部分。

但是，我们还是没有看到这种方法的实际应用，因为我们还没有语句。我们将在下面的章节中，接着讨论。现在，如果遇到解析错误，我们将会立即进入恐慌模式，并且一直展开到顶端，停止解析。因为，当前我们最多只会解析一个表达式语句，所以，我们并没有什么大的损失。

### 3.4 Wiring up the Parser


连接解析器

We are mostly done parsing expressions now. There is one other place where we need to add a little error handling. As the parser descends through the parsing methods for each grammar rule, it eventually hits primary(). If none of the cases in there match, it means we are sitting on a token that can’t start an expression. We need to handle that error too.


我们现在基本上，完成了解析表达式，但是，还有一个地方需要添加错误处理。当解析器通过每一个语法规则对应的方法，递归下降时候，它最终将到达 primary() 方法。如果primary() 中所有的情况都无法匹配，这意味着我们处于一个无法解析的token，我们同样需要处理这种错误。


```java

// lox/Parser.java, in primary()

    private Expr primary() {
        if (match(FALSE))
            return new Expr.Literal(false);
        if (match(TRUE))
            return new Expr.Literal(true);
        if (match(NIL))
            return new Expr.Literal(null);

        if (match(NUMBER, STRING)) {
            return new Expr.Literal(previous().literal);
        }
        if (match(LEFT_PAREN)) {
            Expr expr = expression();
            consume(RIGHT_PAREN, "Expect ')' after expression.");
            return new Expr.Grouping(expr);
        }

        throw error(peek(), "Expect expression.");
    }
	
```


With that, all that remains in the parser is to define an initial method to kick it off. That method is called, naturally enough, parse().

现在，解析器中剩下的就是，定义一个启动方法，这个方法自然被命名为 parse().

```java

// lox/Parser.java, add after Parser()

  Expr parse() {
    try {
      return expression();
    } catch (ParseError error) {
      return null;
    }
  }
  
```

We’ll revisit this method later when we add statements to the language. For now, it parses a single expression and returns it. We also have some temporary code to exit out of panic mode. Syntax error recovery is the parser’s job, so we don’t want the ParseError exception to escape into the rest of the interpreter.

When a syntax error does occur, this method returns null. That’s OK. The parser promises not to crash or hang on invalid syntax, but it doesn’t promise to return a usable syntax tree if an error is found. As soon as the parser reports an error, hadError gets set, and subsequent phases are skipped.

Finally, we can hook up our brand new parser to the main Lox class and try it out. We still don’t have an interpreter, so for now, we’ll parse to a syntax tree and then use the AstPrinter class from the last chapter to display it.



当我们在语言中添加了语句后，我们还将重新讨论这个方法。现在，它只是解析表达式并且返回。我们还有一些临时代码，可以退出恐慌模式。语法错误恢复是解析器的工作，因此我们不希望ParseError 对象逃逸到，解释器的其他部分。

当出现语法错误时候，parse() 方法返回 null，没关系，解析器，承诺当遇到语法错误时候，不会引发崩溃或者挂起，但是，如果发现了语法错误，它不会返回一个语法树。一旦解析器报错，就会设置 hadError, 并且会跳过后续阶段。

最后，我们将把全新的解析器，连接到主Lox类上，进行尝试。我们仍然没有完成解释器，现在，我们的解析结果是语法树，然后，我们将使用上一章中的 AstPrinter 类，展示语法树。

Delete the old code to print the scanned tokens and replace it with this:

删除打印token 的旧代码，替换为新的解析器

```java

// lox/Lox.java, in run(), replace 5 lines


    List<Token> tokens = scanner.scanTokens();
    Parser parser = new Parser(tokens);
    Expr expression = parser.parse();

    // Stop if there was a syntax error.
    if (hadError) return;

    System.out.println(new AstPrinter().print(expression));
  }
  
```

Congratulations, you have crossed the threshold! That really is all there is to handwriting a parser. We’ll extend the grammar in later chapters with assignment, statements, and other stuff, but none of that is any more complex than the binary operators we tackled here.

恭喜🎉，你已经跨过了门槛，这就是编写解析器的全部内容。我们将在后面的章节中，用赋值、语句，其他的东西，来扩展语法。但是，这些都没有本章讨论的二元运算符，更加复杂。

> It is possible to define a more complex grammar than Lox’s that’s difficult to parse using recursive descent. Predictive parsing gets tricky when you may need to look ahead a large number of tokens to figure out what you’re sitting on.
>
> In practice, most languages are designed to avoid that. Even in cases where they aren’t, you can usually hack around it without too much pain. If you can parse C++ using recursive descent—which many C++ compilers do—you can parse anything.
> 
> 有可能定一个比Lox更加复杂的语法，使用递归下降技术很难解析，当你可能需要提前查看大量的token，用来确定你的所在位置时候，预测性解析变得很棘手。
> 
> 实际上，大多数的语言都是为了避免这种场景。即使无法避免的时候，我们也可以绕过它，而不会太痛苦。如果你可以用递归下降技术解析C++，像许多C++解析器那样做的，那么，你将可以解析任意的语言。

Fire up the interpreter and type in some expressions. See how it handles precedence and associativity correctly? Not bad for less than 200 lines of code.

启动解释器，并且输出一些表达式，看看我们的解释器，是如何处理优先级和级联的，对一个少于200行的代码的解释器来说，还算是不错了。

## 四、习题

1. In C, a block is a statement form that allows you to pack a series of statements where a single one is expected. The comma operator is an analogous syntax for expressions. A comma-separated series of expressions can be given where a single expression is expected (except inside a function call’s argument list). At runtime, the comma operator evaluates the left operand and discards the result. Then it evaluates and returns the right operand.

	Add support for comma expressions. Give them the same precedence and associativity as in C. Write the grammar, and then implement the necessary parsing code.
	
1. Likewise, add support for the C-style conditional or “ternary” operator ?:. What precedence level is allowed between the ? and :? Is the whole operator left-associative or right-associative?


1. Add error productions to handle each binary operator appearing without a left-hand operand. In other words, detect a binary operator appearing at the beginning of an expression. Report that as an error, but also parse and discard a right-hand operand with the appropriate precedence.

---

1. 在C语言中，代码块也是一种语句形式，它允许我们将多行代码打包到一个只需要一个语句的地方，[逗号运算符](https://en.wikipedia.org/wiki/Comma_operator)是表达式的类似语法，例如: `int a=1, b=2, c=3, i=0;`, 在需要单个表达式的地方（函数调用的参数列表除外），可以给出逗号分隔的表达式系列。在运行时候，逗号运算符，计算左操作数，然后丢弃结果，然后计算并且返回右操作数。

	添加对于逗号表达式的支持，然后赋予它和C 语言中相同的优先级和关联性。编写语法，然后实现，对应的解析方法。
	
1. 同样的，添加对于三元运算符 `?:`的支持， ? 和 : 之间是什么样的关联性呢？整个三元运算符是左关联还是右关联的呢？

1. 添加错误处理，处理每一个二元运算符，没有出现左操作数。换句话，就是，检查每一个出现在表达式开始的二元运算符，将其报告为错误，但是还要继续分析并且丢弃具有适当优先级的右操作数。


## 五、DESIGN NOTE: LOGIC VERSUS HISTORY

设计说明：逻辑和历史

Let’s say we decide to add bitwise & and | operators to Lox. Where should we put them in the precedence hierarchy? C—and most languages that follow in C’s footsteps—place them below ==. This is widely considered a mistake because it means common operations like testing a flag require parentheses.

假设我们决定添加位运算符，& 和 ｜， 我们应该把它们放在优先级中的哪个位置呢？C语言和大部分紧随C脚步的语言中，位运算都在 == 运算符之下，这被广泛认为是一个错误，因为，如果位运算符优先级低于 ==，下面的测试表达式需要添加括号

```

if (flags & FLAG_MASK == SOME_FLAG) { ... } // Wrong.
if ((flags & FLAG_MASK) == SOME_FLAG) { ... } // Right.

```

Should we fix this for Lox and put bitwise operators higher up the precedence table than C does? There are two strategies we can take.

You almost never want to use the result of an == expression as the operand to a bitwise operator. By making bitwise bind tighter, users don’t need to parenthesize as often. So if we do that, and users assume the precedence is chosen logically to minimize parentheses, they’re likely to infer it correctly.

This kind of internal consistency makes the language easier to learn because there are fewer edge cases and exceptions users have to stumble into and then correct. That’s good, because before users can use our language, they have to load all of that syntax and semantics into their heads. A simpler, more rational language makes sense.


But, for many users there is an even faster shortcut to getting our language’s ideas into their wetware—use concepts they already know. Many newcomers to our language will be coming from some other language or languages. If our language uses some of the same syntax or semantics as those, there is much less for the user to learn (and unlearn).

This is particularly helpful with syntax. You may not remember it well today, but way back when you learned your very first programming language, code probably looked alien and unapproachable. Only through painstaking effort did you learn to read and accept it. If you design a novel syntax for your new language, you force users to start that process all over again.


Taking advantage of what users already know is one of the most powerful tools you can use to ease adoption of your language. It’s almost impossible to overestimate how valuable this is. But it faces you with a nasty problem: What happens when the thing the users all know kind of sucks? C’s bitwise operator precedence is a mistake that doesn’t make sense. But it’s a familiar mistake that millions have already gotten used to and learned to live with.

我们是否应该在Lox语言中，修复这个错误，将位运算符优先级设置更高，而不是C语言中的优先级，我们可以采取两种策略：

因为，我们几乎不会想将 == 表达式的结果当作位运算符的操作数，通过设置更高的位运算符优先级，用户不需要经常添加 括号，因此，如果我们这样做，并且用户假设优先顺序是逻辑选择的，以最小化括号，那么他们很可能会正确推断。

这种内部一致性，使得语言更加容易学习，因为用户偶然发现的边缘情况和异常更少。这很好，因为在用户使用我们的语言之前，我们必须将这些语法和语义加载到大脑中。更简单，更理性的语言是有意义的

但是，对于许多用户来说，有一条捷径可以将我们的语言理念，融入到他们已经熟悉的其他语言中，我们语言的使用者，通常熟悉其他语言，如果我们的语言中使用了一些与常见语言相同的语法或者语义，那么他们将需要学习的东西将更少。


这对于语法特别有用，现在，你可能不太记得了，但是早在你学习第一门编程语言的时候，代码可能看起来很陌生，很难接近，只有通过艰苦的努力，你才能学会阅读和接受它，如果你为你的新语言，设计了一个新颖的语法，你实际上，在强迫用户重新开始这个过程。


利用用户已经知道的知识，是你简化语言，可以采用的最强大的工具之一。几乎不可能高估它的价值。但是，我们会面临一个棘手的问题：当用户都知道，某个东西比较糟糕时，会发生什么呢？C的位运算符的优先级，是一个没有意义的错误，数百万人已经习惯了这个错误。

Do you stay true to your language’s own internal logic and ignore history? Do you start from a blank slate and first principles? Or do you weave your language into the rich tapestry of programming history and give your users a leg up by starting from something they already know?

There is no perfect answer here, only trade-offs. You and I are obviously biased towards liking novel languages, so our natural inclination is to burn the history books and start our own story.

In practice, it’s often better to make the most of what users already know. Getting them to come to your language requires a big leap. The smaller you can make that chasm, the more people will be willing to cross it. But you can’t always stick to history, or your language won’t have anything new and compelling to give people a reason to jump over.

你是否要坚持逻辑正确，忽略历史遗留问题。你是从一张白板和第一原则开始的吗？或者你是把自己的语言，融入到编程语言的历史中呢？从用户已经知道的东西开始，给他们一个帮助？

这里没有完美的答案，只有权衡，对我而言，更加喜欢新颖的语言，所以，我们将自然的烧掉历史书，开始我们自己的故事

实际上，最好是充分利用用户已有知识，让他们适应你的语言是一个巨大的飞跃，你越是缩小这个鸿沟，人们越是有可能跨过它。但是，你也不能总是坚持历史，否则你的语言不会有任何创新，让人们想要跳过鸿沟。
	

