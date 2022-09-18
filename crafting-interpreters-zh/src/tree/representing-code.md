# Representing Code


> To dwellers in a wood, almost every species of tree has its voice as well as its feature.
> 
> <p align="right"> Thomas Hardy, Under the Greenwood Tree</p>
> 
> 对于森林中的居民来说，几乎每一棵树木都有自己的声音和特征。

In the last chapter, we took the raw source code as a string and transformed it into a slightly higher-level representation: a series of tokens. The parser we’ll write in the next chapter takes those tokens and transforms them yet again, into an even richer, more complex representation.

Before we can produce that representation, we need to define it.That’s the subject of this chapter. Along the way, we’ll cover some theory around formal grammars, feel the difference between functional and object-oriented programming, go over a couple of design patterns, and do some metaprogramming.

在上一章中，我们通过扫描器，将源代码转换为稍微高级一些的表示：一系列的token。下一章，将要介绍的解释器，会把tokens，转换为更加丰富、复杂的表示。

在我们使用解释器之前，我们需要先定义token，这是本章节的主题。在本章，我们将介绍一些形式语法理论，感受函数式编程和面向对象编程的区别。我们还将讨论一些设计模式，进行一些元编程。

> I was so worried about this being one of the most boring chapters in the book that I kept stuffing more fun ideas into it until I ran out of room.
> 
> 我一直担心，本章节是此书中最无聊的一章，所以，我一直添加有意思的东西，直到塞满了本章。

Before we do all that, let’s focus on the main goal—a representation for code. It should be simple for the parser to produce and easy for the interpreter to consume. If you haven’t written a parser or interpreter yet, those requirements aren’t exactly illuminating. Maybe your intuition can help. What is your brain doing when you play the part of a human interpreter? How do you mentally evaluate an arithmetic expression like this:

```
1 + 2 * 3 - 4
```

Because you understand the order of operations—the old “Please Excuse My Dear Aunt Sally” stuff—you know that the multiplication is evaluated before the addition or subtraction. One way to visualize that precedence is using a tree. Leaf nodes are numbers, and interior nodes are operators with branches for each of their operands.

In order to evaluate an arithmetic node, you need to know the numeric values of its subtrees, so you have to evaluate those first. That means working your way from the leaves up to the root—a post-order traversal:

A. Starting with the full tree, evaluate the bottom-most operation, 2 * 3.

B. Now we can evaluate the +.

C. Next, the -.

D. The final answer.


在我们开始做这些之前，让我们先关注主要目标-代码表示。它应该容易被解释器生成，并且解释器更加容易使用它们。但是，如果你还没有编写过解释器或者编译器，那么这些需求你可能不太熟悉。也许，你的直觉将会起作用。当你在扮演人类语言翻译角色时候，你的大脑会如何思考？让我们来看一下下面的示例，你会如何计算下面的算术表达式：

```
1 + 2 * 3 - 4
```

因为我们知道算术运算符的优先级，所以，我们知道乘法会优先加减法，先计算。可视化运算优先级的一种方法是，使用树结构。其中，叶节点是具体的数字，中间层节点是运算符。

In order to evaluate an arithmetic node, you need to know the numeric values of its subtrees, so you have to evaluate those first. That means working your way from the leaves up to the root—a post-order traversal:

为了计算算术节点，我们需要知道它的子树的结果，所以，我们需要先计算这些子树的结果，这意味着，我们需要从叶节点到根节点计算，也就是一个后序遍历。

![tree-travel](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/tree-evaluate.png?raw=true)

由上图，可以看到计算步骤

A. 计算最底层的表达式，2 * 3

B. 计算加法表达式

C. 计算减法表达式

D. 得到最终结果

If I gave you an arithmetic expression, you could draw one of these trees pretty easily. Given a tree, you can evaluate it without breaking a sweat. So it intuitively seems like a workable representation of our code is a tree that matches the grammatical structure—the operator nesting—of the language.

如果我们提供了一个算术表达式，那么可以很容易的转变为一棵树，同样的，如果有一棵树，我们也可以容易变为一个表达式。因此，从直觉上，我们可以将代码表示为一棵树，这棵树与我们使用的编程语言的语法结构、运算符，嵌套匹配。

> That’s not to say a tree is the only possible representation of our code. In Part III, we’ll generate bytecode, another representation that isn’t as human friendly but is closer to the machine.
> 
> 这并不是说，语法树是我们代码的唯一表示方式。在本书的第三部分，我们还将介绍字节码，这是一种更加接近机器的表示，但是对于人类来说，不是那么人性化。

We need to get more precise about what that grammar is then. Like lexical grammars in the last chapter, there is a long ton of theory around syntactic grammars. We’re going into that theory a little more than we did when scanning because it turns out to be a useful tool throughout much of the interpreter. We start by moving one level up the [Chomsky hierarchy](https://en.wikipedia.org/wiki/Chomsky_hierarchy). 

我们需要更加精确去了解，什么是语法。就像上一章的词法规则一样，关于语法规则也有很多理论。我们对这些理论的探讨将更加深入一些，因为这些理论被证明，对于解释器是一个有用的工具。我们首先要学习 [Chomsky hierarchy](https://en.wikipedia.org/wiki/Chomsky_hierarchy). 

## 一、Context-Free Grammars

上下文无关语法

In the last chapter, the formalism we used for defining the lexical grammar—the rules for how characters get grouped into tokens—was called a regular language. That was fine for our scanner, which emits a flat sequence of tokens. But regular languages aren’t powerful enough to handle expressions which can nest arbitrarily deeply.

We need a bigger hammer, and that hammer is a context-free grammar (CFG). It’s the next heaviest tool in the toolbox of formal grammars. A formal grammar takes a set of atomic pieces it calls its “alphabet”. Then it defines a (usually infinite) set of “strings” that are “in” the grammar. Each string is a sequence of “letters” in the alphabet.

I’m using all those quotes because the terms get a little confusing as you move from lexical to syntactic grammars. In our scanner’s grammar, the alphabet consists of individual characters and the strings are the valid lexemes—roughly “words”. In the syntactic grammar we’re talking about now, we’re at a different level of granularity. Now each “letter” in the alphabet is an entire token and a “string” is a sequence of tokens—an entire expression.

Oof. Maybe a table will help:

|Terminology	|	Lexical grammar	|Syntactic grammar|
|---|---|---|
|The “alphabet” is . . .|Characters|Tokens|
|A “string” is . . .|Lexeme or token|Expression|
|It’s implemented by the . . .|Scanner|Parser|

A formal grammar’s job is to specify which strings are valid and which aren’t. If we were defining a grammar for English sentences, “eggs are tasty for breakfast” would be in the grammar, but “tasty breakfast for are eggs” would probably not.

在上一章中，我们定义词汇的规则——将字符分组为token的规则，被称为正则语言。这对于我们实现的扫描器是非常好的，它将输出一个token序列。但是，正则语言的功能不足以处理任何深度嵌套的表达式。

我们需要更强大的工具，这个工具就是CFG 上下文无关语法。CFG上下文无关语法，是[形式文法](https://en.wikipedia.org/wiki/Formal_grammar)中第二重要的工具. 形式文法定义了一组原子化的字符表，称为字母表，然后，我们可以定义一组字符串（通常是无限的），每一个字符串都由字母表中的字母组合。

我使用了一些引号标注，是因为当我们从词法规则变更到语法规则时候，其中的术语会变得令人困惑。在词法规则中，字母表是单个字符组成的，字符串是有效的词素（即单词）。但是，在我们当前的语法讨论中，现在的字母表中每一个元素都是一个token，我们处于不同的粒度级别。同样的，这时候，一个字符串表示一组token，即实际上是一个表达式。

它们的区别，详见下表：

|术语|词法规则|语法规则|
|---|---|---|
|字母表|字符列表|token列表|
|字符串|词素（单词）或者token|表达式|
|实现者|扫描器|解释器|

形式文法的目标是指定哪些字符串有效，哪些字符串无效。例如：我们想要给一个英语语句，定义语法规则。"eggs are tasty for breakfast" 是一个合法的英语语句，但是，“tasty breakfast for are eggs”就不是一个合法的英语语句。

### 1.1 Rules for grammars

语法规则

How do we write down a grammar that contains an infinite number of valid strings? We obviously can’t list them all out. Instead, we create a finite set of rules. You can think of them as a game that you can “play” in one of two directions.

If you start with the rules, you can use them to generate strings that are in the grammar. Strings created this way are called derivations because each is derived from the rules of the grammar. In each step of the game, you pick a rule and follow what it tells you to do. Most of the lingo around formal grammars comes from playing them in this direction. Rules are called productions because they produce strings in the grammar.

我们如何写出包含无限个字符串的语法？显然，我们不可能罗列出它们。相反，我们可以定义一组有限的规则，我们可以把语法规则想象为一个游戏，可以从两个方向中的一个玩。

如果你从语法规则开始，我们可以使用语法规则生成合法的语句，这些生成的语句，称为派生语句，因为它们都是从语法规则中派生出。在游戏的每一步，你都需要选择一条语法规则，然后，按照规则继续运行。形式文法的，大部分术语都是从这个方向中产生的，语法规则被称为产生式，因为，它们生产字符串。

Each production in a context-free grammar has a head—its name—and a body, which describes what it generates. In its pure form, the body is simply a list of symbols. Symbols come in two delectable flavors:

* A terminal is a letter from the grammar’s alphabet. You can think of it like a literal value. In the syntactic grammar we’re defining, the terminals are individual lexemes—tokens coming from the scanner like if or 1234.

	These are called “terminals”, in the sense of an “end point” because they don’t lead to any further “moves” in the game. You simply produce that one symbol.

* A nonterminal is a named reference to another rule in the grammar. It means “play that rule and insert whatever it produces here”. In this way, the grammar composes.

There is one last refinement: you may have multiple rules with the same name. When you reach a nonterminal with that name, you are allowed to pick any of the rules for it, whichever floats your boat.


上下文无关语法中的每一个输出，都有一个头（名称），一个主体（描述生成的内容），在纯形式中，主体是一个符号列表。符号主要分为两类：

* 终止符，是语法字母表中的字母。可以将它想象为文字值。在我们定义的语法规则中，终止符是来自扫描器生成的单个词素或者token，例如：if， 1234，这些被称为终止符，因为它们不会导致规则的进一步前进，它们就是语法规则的终点。只需要生成一个符号

* 非终止符，是对语法规则中的另外一个规则的引用。它的意思是，继续这个规则，将产生的任何东西，插入当前位置，语法就是这样生成的。

还有最后一个改进，你可能会有多个同名规则。当我们遇到该名称的非终止符时候，我们可以选择任意一个规则，无论那个规则使我们继续进行下去。

To make this concrete, we need a way to write down these production rules.People have been trying to crystallize grammar all the way back to Pāṇini’s Ashtadhyayi, which codified Sanskrit grammar a mere couple thousand years ago. Not much progress happened until John Backus and company needed a notation for specifying ALGOL 58 and came up with Backus-Naur form (BNF). Since then, nearly everyone uses some flavor of BNF, tweaked to their own tastes.

I tried to come up with something clean. Each rule is a name, followed by an arrow (→), followed by a sequence of symbols, and finally ending with a semicolon (;). Terminals are quoted strings, and nonterminals are lowercase words.



为了使其具体化，我们需要一种方法来写下这些规则。人们一直尝试着将语法具体化，例如：两千年前的Pāṇini ，编写了梵语语法书  [Aṣṭādhyāyī](https://en.wikipedia.org/wiki/P%C4%81%E1%B9%87ini) 。但是，一直到 [John Backus](https://en.wikipedia.org/wiki/John_Backus)和他的公司，需要为ALGOL 58 语言指定一种符号，最后被命名为[巴科斯范式](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form), 缩写为BNF，从此以后，每一个语言发明者都使用变化的BNF，根据自己的偏好修改。

我尝试着设计出一个简洁的方式，每个规则都是一个名称，后面紧跟着一个箭头 —>，然后在跟随者一串符号，最终用分号结束; 

终止符是带引号的字符串，非终止符是小写字符

Using that, here’s a grammar for breakfast menus:

```

breakfast  → protein "with" breakfast "on the side" ;
breakfast  → protein ;
breakfast  → bread ;

protein    → crispiness "crispy" "bacon" ;
protein    → "sausage" ;
protein    → cooked "eggs" ;

crispiness → "really" ;
crispiness → "really" crispiness ;

cooked     → "scrambled" ;
cooked     → "poached" ;
cooked     → "fried" ;

bread      → "toast" ;
bread      → "biscuits" ;
bread      → "English muffin" ;

```

使用它，我们可以形成早餐的语法规则。

We can use this grammar to generate random breakfasts. Let’s play a round and see how it works. By age-old convention, the game starts with the first rule in the grammar, here breakfast. There are three productions for that, and we randomly pick the first one. Our resulting string looks like:

```protein "with" breakfast "on the side"```

We need to expand that first nonterminal, protein, so we pick a production for that. Let’s pick:

```protein → cooked "eggs" ;```

Next, we need a production for cooked, and so we pick "poached". That’s a terminal, so we add that. Now our string looks like:

```"poached" "eggs" "with" breakfast "on the side"```

The next non-terminal is breakfast again. The first breakfast production we chose recursively refers back to the breakfast rule. Recursion in the grammar is a good sign that the language being defined is context-free instead of regular. In particular, recursion where the recursive nonterminal has productions on both sides implies that the language is not regular.

We could keep picking the first production for breakfast over and over again yielding all manner of breakfasts like “bacon with sausage with scrambled eggs with bacon . . . ” We won’t though. This time we’ll pick bread. There are three rules for that, each of which contains only a terminal. We’ll pick “English muffin”.

With that, every nonterminal in the string has been expanded until it finally contains only terminals and we’re left with:

![breakfast](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/breakfast.png?raw=true)

Throw in some ham and Hollandaise, and you’ve got eggs Benedict.


我们可以使用这个规则，生成随机的早餐，让我们实际生成一个早餐，看看它是如何生效的。按照古老的惯例，游戏从语法中的第一条规则开始，这里是 breakfast, 对于 breakfast 名称的规则，我们有3个不同的主体，让我们随机选择一个规则，

```breakfast  → protein "with" breakfast "on the side" ;```

接下来，我们将扩展第一个非终止符 protein, 同样的，我们将随机选择 protein 名称的规则，我们选择第3个规则

```protein → cooked "eggs" ;```

接着，我们将继续扩展非终止符 cooked, 这里我们选择 poached, 而这是一个终止符，所以我们的语法规则到此结束，完整的字符串看起来是

```"poached" "eggs" "with" breakfast "on the side"```

下面的一个非终止符是 breakfast , 如果我们选择第一个规则，那么我们将递归的调用 breakfast, 语法中的递归是一个标志，表示语法规则是上下文无关的，而不是正则的。特别情况下，非终止符在规则两边都存在，意味着递归，而不是正则的。

我们可以一直选择breakfast 非终止符的第一个规则，然后，我们将生成源源不断的早餐，但是我们不会这样做的，我们将选择breakfast的第3条规则，bread，下面是我们的早餐

![breakfast](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/breakfast.png?raw=true)


Any time we hit a rule that had multiple productions, we just picked one arbitrarily. It is this flexibility that allows a short number of grammar rules to encode a combinatorially larger set of strings. The fact that a rule can refer to itself—directly or indirectly—kicks it up even more, letting us pack an infinite number of strings into a finite grammar.

任何时候，我们遇到了一个名称，对应着多个规则，我们将选择一个。正是这种灵活性，使得我们可以用有限的语法规则，组合成更多的字符串集合。一个规则中可以递归的调用自己，这样子，我们可以将无限的字符串，表示为有限的语法规则。

### 1.2 Enhancing our notation

增强我们的符号

Stuffing an infinite set of strings in a handful of rules is pretty fantastic, but let’s take it further. Our notation works, but it’s tedious. So, like any good language designer, we’ll sprinkle a little syntactic sugar on top—some extra convenience notation. In addition to terminals and nonterminals, we’ll allow a few other kinds of expressions in the body of a rule:

* Instead of repeating the rule name each time we want to add another production for it, we’ll allow a series of productions separated by a pipe (|).

	```bread → "toast" | "biscuits" | "English muffin" ;```
	
* Further, we’ll allow parentheses for grouping and then allow | within that to select one from a series of options within the middle of a production.

	```protein → ( "scrambled" | "poached" | "fried" ) "eggs" ;```

* Using recursion to support repeated sequences of symbols has a certain appealing purity, but it’s kind of a chore to make a separate named sub-rule each time we want to loop. So, we also use a postfix * to allow the previous symbol or group to be repeated zero or more times.

	```crispiness → "really" "really"* ;```
	
* A postfix + is similar, but requires the preceding production to appear at least once.

	```crispiness → "really"+ ;```
	
* A postfix ? is for an optional production. The thing before it can appear zero or one time, but not more.

	```breakfast → protein ( "with" breakfast "on the side" )? ;```
	
With all of those syntactic niceties, our breakfast grammar condenses down to:

```

breakfast → protein ( "with" breakfast "on the side" )?
          | bread ;

protein   → "really"+ "crispy" "bacon"
          | "sausage"
          | ( "scrambled" | "poached" | "fried" ) "eggs" ;

bread     → "toast" | "biscuits" | "English muffin" ;

```

在一些规则中，我们填充一组无限的字符串是非常奇妙的，但是，让我们更加深入一些。我们的语法符号生效了，但是十分乏味。所以，像很多有限的语言设计者一样，我们将添加一些语法糖（即添加一些其他符号，更加方便的编写语法规则），除了终止符和非终止符，我们还将在语法规则中，使用下面的符号

* 我们将添加管道符| 用于分隔规则名称，而不是每一次都重复写入多个相同规则名称

	```bread → "toast" | "biscuits" | "English muffin" ;```

* 进一步，我们将允许使用括号进行分组，每一个组中，可以使用管道符 | ，选择其中的一个

	```protein → ( "scrambled" | "poached" | "fried" ) "eggs" ;```

* 使用递归来支持某个规则的重复使用，是一种好的方式，但是，需要我们每次都定义一个单独的子规则，这有一些麻烦。所以，我们可以通过一个后缀*，表示前面的规则或者组，重复零次或者多次。

	```crispiness → "really" "really"* ;```

* 相似的后缀，还有+ 表示它之前的规则或者组，至少存在一次

	```crispiness → "really"+ ;```
	
* 和 + 相似的后缀为 ? 表示前面的规则或者组可以出现零次或者1次

	```breakfast → protein ( "with" breakfast "on the side" )? ;```
	

当我们使用了上面的语法糖后，可以将早餐规则变更为

```
breakfast → protein ( "with" breakfast "on the side" )?
          | bread ;

protein   → "really"+ "crispy" "bacon"
          | "sausage"
          | ( "scrambled" | "poached" | "fried" ) "eggs" ;

bread     → "toast" | "biscuits" | "English muffin" ;
```

Not too bad, I hope. If you’re used to grep or using regular expressions in your text editor, most of the punctuation should be familiar. The main difference is that symbols here represent entire tokens, not single characters.

We’ll use this notation throughout the rest of the book to precisely describe Lox’s grammar. As you work on programming languages, you’ll find that context-free grammars (using this or EBNF or some other notation) help you crystallize your informal syntax design ideas. They are also a handy medium for communicating with other language hackers about syntax.

The rules and productions we define for Lox are also our guide to the tree data structure we’re going to implement to represent code in memory. Before we can do that, we need an actual grammar for Lox, or at least enough of one for us to get started.

我希望我们的语法糖不会太糟糕，如果你之前使用过 grep 命令或者[正则表达式](https://en.wikipedia.org/wiki/Regular_expression#Standards), 那么其中大部分符号是相同的含义。主要区别是，这里的符号对象是整个token，而不是单个字符

在本书接下来部分，我们将使用这种语法，描述我们的lox语言。当我们使用编程语言时候，会发现上下文无关语法（通常具有BNF，[EBNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form)范式）, 有助于我们具体化语法设计思想。它们也是我们同编程高手交流语法的便捷媒介。

我们定义的lox语言语法和产品，是我们将要实现的语法树结构的指南，表示内存中的代码。在我们可以这样做之前，我们需要一个实际的lox语言语法，或者至少有一些语法开始我们的解析。

### 1.3 A Grammar for Lox expressions

lox语言表达式语法

In the previous chapter, we did Lox’s entire lexical grammar in one fell swoop. Every keyword and bit of punctuation is there. The syntactic grammar is larger, and it would be a real bore to grind through the entire thing before we actually get our interpreter up and running.

Instead, we’ll crank through a subset of the language in the next couple of chapters. Once we have that mini-language represented, parsed, and interpreted, then later chapters will progressively add new features to it, including the new syntax. For now, we are going to worry about only a handful of expressions:

* Literals. Numbers, strings, Booleans, and nil.

* Unary expressions. A prefix ! to perform a logical not, and - to negate a number.

* Binary expressions. The infix arithmetic (+, -, *, /) and logic operators (==, !=, <, <=, >, >=) we know and love.

* Parentheses. A pair of ( and ) wrapped around an expression.

That gives us enough syntax for expressions like:


```C

1 - (2 * 3) < 4 == false

```



上一章中，我们一下子就完成了整个词法规则的介绍，每个关键字和标点符号都在那里。但是，语法规则内容更多，在我们真正启动解释器并且运行之前，对于整个规则的介绍，将是一个真正的麻烦。

相反，我们将在接下来的几章中，快速浏览lox语言的一个子集。一旦我们有了代码表示、解析、编译的迷你语言，随后的章节，将添加新的功能与语法。让我们先从几个表达式入门：


* 文字，数值、字符串、布尔类型和nil

* 一元表达式，前缀! 表示逻辑非， 前缀-表示负数

* 二元表达式，中缀运算符 + - * / 和 逻辑运算符 == != < <= > >=

* 圆括号，包含一个表达式

下面的表达式，包含上面的语法规则


```C

1 - (2 * 3) < 4 == false

```

Using our handy dandy new notation, here’s a grammar for those:

There’s one bit of extra metasyntax here. In addition to quoted strings for terminals that match exact lexemes, we CAPITALIZE terminals that are a single lexeme whose text representation may vary. NUMBER is any number literal, and STRING is any string literal. Later, we’ll do the same for IDENTIFIER.

This grammar is actually ambiguous, which we’ll see when we get to parsing it. But it’s good enough for now.

使用我们之前定义的语法糖，这个语法可以写成：

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

上面还有一些额外的说明，除了匹配准确的词素，我们当作终止符之外，我们还把大写的字符串当作终止符。NUMBER表示任何的数字，STRING表示任何的字符串。稍后，我们还将介绍 IDENTIFIER

这个语法规则实际上不够明确，但是我们将在遇到它们时候，在探讨，现在我们将继续下面介绍。

## 二、Implementing Syntax Trees

实现语法树

Finally, we get to write some code. That little expression grammar is our skeleton. Since the grammar is recursive—note how grouping, unary, and binary all refer back to expression—our data structure will form a tree. Since this structure represents the syntax of our language, it’s called a syntax tree.

Our scanner used a single Token class to represent all kinds of lexemes. To distinguish the different kinds—think the number 123 versus the string "123"—we included a simple TokenType enum. Syntax trees are not so homogeneous. Unary expressions have a single operand, binary expressions have two, and literals have none.

We could mush that all together into a single Expression class with an arbitrary list of children. Some compilers do. But I like getting the most out of Java’s type system. So we’ll define a base class for expressions. Then, for each kind of expression—each production under expression—we create a subclass that has fields for the nonterminals specific to that rule. This way, we get a compile error if we, say, try to access the second operand of a unary expression.

最后，我们将编写代码实现语法规则。上面的表达式语法，是我们要实现的骨架。由于语法是递归的，需要注意分组、一元、二元对于表达式是如何实现的，我们的数据结构将形成一棵树，因为这是对语法的描述，我们称之为语法树。

我们的扫描器，使用单个token类型，表示所有类型的词素。为了区分不同的类型，例如：数字123 和 字符串"123", 我们token类型，包含有一个TokenType属性。语法树不是那么同质化，一元表达式有一个操作数，二元表达式有两个操作数，文字没有操作数。

我们可以将这些都组合为一个表达式类中，其中包含任意的子类列表。有些编译器是这样做的，但是我喜欢充分利用Java类型系统的分类。因此，我们将首先定义表达式基类。然后，对于每种类型的表达式，和表达式下的每一个子类，我们都创建一个子类。这个子类具有特定该规则的非终端符。通过这样做，如果我们尝试访问一元表达式类中的第二个操作数，我们将接收到编译报错。

Something like this:

```java

package com.craftinginterpreters.lox;

abstract class Expr { 
  static class Binary extends Expr {
    Binary(Expr left, Token operator, Expr right) {
      this.left = left;
      this.operator = operator;
      this.right = right;
    }

    final Expr left;
    final Token operator;
    final Expr right;
  }

  // Other expressions...
}

```

Expr is the base class that all expression classes inherit from. As you can see from Binary, the subclasses are nested inside of it. There’s no technical need for this, but it lets us cram all of the classes into a single Java file.

实现代码如上，Expr是所有表达式可以继承的基类。正如我们看到的Binary 子类嵌套在Expr中。这在技术上没有必要，但是如此做，我们可以将所有类塞到一个java源文件中。

### 2.1 Disoriented objects

面向对象的困惑

You’ll note that, much like the Token class, there aren’t any methods here. It’s a dumb structure. Nicely typed, but merely a bag of data. This feels strange in an object-oriented language like Java. Shouldn’t the class do stuff?

The problem is that these tree classes aren’t owned by any single domain. Should they have methods for parsing since that’s where the trees are created? Or interpreting since that’s where they are consumed? Trees span the border between those territories, which means they are really owned by neither.

In fact, these types exist to enable the parser and interpreter to communicate. That lends itself to types that are simply data with no associated behavior. This style is very natural in functional languages like Lisp and ML where all data is separate from behavior, but it feels odd in Java.

Functional programming aficionados right now are jumping up to exclaim “See! Object-oriented languages are a bad fit for an interpreter!” I won’t go that far. You’ll recall that the scanner itself was admirably suited to object-orientation. It had all of the mutable state to keep track of where it was in the source code, a well-defined set of public methods, and a handful of private helpers.

My feeling is that each phase or part of the interpreter works fine in an object-oriented style. It is the data structures that flow between them that are stripped of behavior.

我们注意到，和Token类一样，Expr中没有类方法，这是一个不好的结构，只有一堆数据，在Java这样的面向对象语言中，我们还有补充一些东西。

问题是，这些语法树类不属于单个域。它们是否应该有对应的解析方法，当语法树被创建出来，或者是否应该有编译方法，当它们被编译时候。这些语法树，横跨这些边界，这意味着它们不属于任何一方。

实际上，这些类型的存在是为了让解析器和解释器可以通信。这样的需求，使得这些类只有简单的数据类型，而没有关联行为。这种风格在Lisp，ML这些函数语言中，非常自然，但是，所有数据与行为分离，在Java中非常奇怪。

此时，函数式编程拥护者跳起来大喊：看，面向对象的语言不适合编写解释器。我不会走那么远，你应该记得，面向对象语言非常适合编写扫描器。它拥有可变状态来跟踪源码位置，一组定义良好的公共方法，和一些helper函数。

我的感觉是，解释器的每一个阶段或者每一个部分，都以面向对象的方式运行的非常好。它们直接的数据结构的通信，使得不需要行为。


### 2.2 Metaprogramming the trees

语法树中的元编程

Java can express behavior-less classes, but I wouldn’t say that it’s particularly great at it. Eleven lines of code to stuff three fields in an object is pretty tedious, and when we’re all done, we’re going to have 21 of these classes.

I don’t want to waste your time or my ink writing all that down. Really, what is the essence of each subclass? A name, and a list of typed fields. That’s it. We’re smart language hackers, right? Let’s automate.

Java可以定义一个没有方法的类，但是我想说Java不太擅长处理这种类。在一个对象中，使用11行代码，填充3个字段，相当乏味，当我们全部完成后，将发现这样的类有21个

我不想浪费你的时间，或者我的墨水将这些类列举出来。真的，每一个类的本质是什么呢？类名称，类中的类型字段列表。就这样，那么我们可以自动化创建了

Instead of tediously handwriting each class definition, field declaration, constructor, and initializer, we’ll hack together a script that does it for us. It has a description of each tree type—its name and fields—and it prints out the Java code needed to define a class with that name and state.

我们将编写一个脚本，生成类，而不是，繁琐的编写每一个类定义，字段声明，构造函数，初始化变量。它将描述每一个定义的语法树类型，包含类名称和类字段列表，打印出类名称和相应的Java定义代码

This script is a tiny Java command-line app that generates a file named “Expr.java”:

这个脚本是一个简单的java代码，生成一个Expr.java 文件










