# 解析表达式

> Grammar, which knows how to control even kings.
>
> 语法，知道如何控制国王。
>
> <p align="right">—— Molière </p>


This chapter marks the first major milestone of the book. Many of us have cobbled together a mishmash of regular expressions and substring operations to extract some sense out of a pile of text. The code was probably riddled with bugs and a beast to maintain. Writing a real parser—one with decent error handling, a coherent internal structure, and the ability to robustly chew through a sophisticated syntax—is considered a rare, impressive skill. In this chapter, you will attain it.

这一章是本书的一个重要里程碑，我们中的许多人都拼凑了一堆正则表达式和子字符串处理方法，以便在一堆文本中获取意义。代码可能存在漏洞，需要不断维护。编写一个真正的解析器——具有良好的错误处理能力，连贯的内部结构，对复杂语法的良好处理能力，被认为是一种罕见的、令人印象深刻的能力。在本章，我们将实现这个解析器。

> “Parse” comes to English from the Old French “pars” for “part of speech”. It means to take a text and map each word to the grammar of the language. We use it here in the same sense, except that our language is a little more modern than Old French.
> 
> Like many rites of passage, you’ll probably find it looks a little smaller, a little less daunting when it’s behind you than when it loomed ahead.
>
> parse 单词是从古法语中来的，它表示我们从一个文本中，把每个单词映射为语言语法中。

It’s easier than you think, partially because we front-loaded a lot of the hard work in the last chapter. You already know your way around a formal grammar. You’re familiar with syntax trees, and we have some Java classes to represent them. The only remaining piece is parsing—transmogrifying a sequence of tokens into one of those syntax trees.

Some CS textbooks make a big deal out of parsers. In the ’60s, computer scientists—understandably tired of programming in assembly language—started designing more sophisticated, human-friendly languages like Fortran and ALGOL. Alas, they weren’t very machine-friendly for the primitive computers of the time.



这个解析过程，比你想要的要简单一些，因为我们在上一章中，提前完成了很多艰苦的工作。你已经熟悉了正则语法，而且我们使用了很多的Java类来表示这些表达式。剩下的解析部分——将一系列的token解析为语法树。

有一些编程教科书，对解析器部分做了很大的改动。上世纪60年代初，计算机科学家对于汇编编程感到厌倦，这是可以理解的，于是，他们开始设计更加高级、人性化的语言，例如：Fortran ,Algol等等，对于当时的原始计算机，这些语言对于机器不是很友好。

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

在上一章中，我们可以像玩游戏一样，通过上下文无法语法，生成一系列字符串，解析器反过来玩这个游戏。给出一个字符串（一系列token）我们将这些token 映射到语法规则中，确定哪些规则，可以生成这样的字符串。

”可能有“ 部分非常有意思，我们完全可以创建一个不明确、存在歧义的语法，不同的选择导致生成相同的字符串，当我们使用语法生成字符串后，这不再重要，一旦我们有了字符串，没有人会在乎我们是如何生成它的

When parsing, ambiguity means the parser may misunderstand the user’s code. As we parse, we aren’t just determining if the string is valid Lox code, we’re also tracking which rules match which parts of it so that we know what part of the language each token belongs to. Here’s the Lox expression grammar we put together in the last chapter:

当解析时候，歧义意味着解析器可能会误解用户的代码。在解析时候，我们不仅仅要确定字符串是有效的lox语言代码，还需要跟踪哪些语法规则和字符串的哪些部分匹配，便于我们获取token属于具体的哪个部分，下面是我们之前总结的lox语法规则

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

``` 6/3-1 ``` 视为 ```(6/3)-1```  或者 ```6/(3-1)```

二叉树允许我们按照任意的方式嵌套，而这又会反过来影响，我们的解析过程，自从黑板被发明出来，数学家解决这个歧义的方式是：定义优先级，关联性规划。

* 优先级

	优先级确定了包含不同运算符的表达式中，我们首先计算哪个运算符。优先级规则告知我们，/ 优先于 -， 优先级高的运算符先计算，同样的，优先级较高的运算符，可以称为绑定更紧。
	
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
> 有一些解析器或者解释器，没有将优先级正确的添加到语法规则中，而是让你保持相同的模糊但是简单的语法，然后在旁边添加一些显式的运算符优先级元数据，用于消除歧义。


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

There are many grammars you can define that match the same language. The choice for how to model a particular language is partially a matter of taste and partially a pragmatic one. This rule is correct, but not optimal for how we intend to parse it. Instead of a left recursive rule, we’ll use a different one.

我们可以定义许多语法，和同一个语言相匹配。如果为一门语言建模，一部分取决于编程品味，一部分取决于应用。这个规则是正确的，但是，对于我们如何解析它来说不是最佳的，我们将使用新的规则，而不是使用了左递归规则

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


isAtEnd() checks if we’ve run out of tokens to parse. peek() returns the current token we have yet to consume, and previous() returns the most recently consumed token. The latter makes it easier to use match() and then access the just-matched token.

isAtEnd() 方法，检查token是否已经使用完，peek(） 返回current位置的token，previous() 返回最近使用的token，previous() 方法，让 match() 函数更加容易，访问匹配的token

That’s most of the parsing infrastructure we need. Where were we? Right, so if we are inside the while loop in equality(), then we know we have found a != or == operator and must be parsing an equality expression.

We grab the matched operator token so we can track which kind of equality expression we have. Then we call comparison() again to parse the right-hand operand. We combine the operator and its two operands into a new Expr.Binary syntax tree node, and then loop around. For each iteration, we store the resulting expression back in the same expr local variable. As we zip through a sequence of equality expressions, that creates a left-associative nested tree of binary operator nodes.

这就是我们需要使用的大部分的，解析基础结构，我们在哪里？如果我们在 equality() 方法的while循环中，那么我们需要找到 != 或者 == 运算符，并且必须解析等式表达式。

我们将获取匹配到的运算符token（ == 或者 != ), 以便跟踪我们拥有哪种等式表达式，然后，我们将再次调用 comparison() 方法，来解析右边的操作数，我们将两个操作数和运算符，组合为一个新的二元表达式语法树，然后，循环，对于每一次迭代，我们都将结果保存在相同的expr 局部变量中，

当我们创建一系列的等式表达式时候，我们将创建一个二元运算符节点的左关联嵌套树。

![sequence](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/sequence.png?raw=true)

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


语法规则实际等同于等式，相应的代码也是如此，唯一的区别是我们匹配的运算符token的类型，以及，我们为了操作数调用的方法是 term() 而不是 comparison() , 另外的两个二元运算符规则，遵循相同的模式

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
