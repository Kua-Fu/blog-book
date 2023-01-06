# Representing Code

代码表示

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
|实现者|扫描器|解析器|

形式文法的目标是指定哪些字符串有效，哪些字符串无效。例如：我们想要给一个英语语句，定义语法规则。"eggs are tasty for breakfast" 是一个合法的英语语句，但是，“tasty breakfast for are eggs”就不是一个合法的英语语句。

### 1.1 Rules for grammars

语法规则

How do we write down a grammar that contains an infinite number of valid strings? We obviously can’t list them all out. Instead, we create a finite set of rules. You can think of them as a game that you can “play” in one of two directions.

If you start with the rules, you can use them to generate strings that are in the grammar. Strings created this way are called derivations because each is derived from the rules of the grammar. In each step of the game, you pick a rule and follow what it tells you to do. Most of the lingo around formal grammars comes from playing them in this direction. Rules are called productions because they produce strings in the grammar.

我们如何写出包含无限个字符串的语法？显然，我们不可能罗列出它们。相反，我们可以定义一组有限的规则，我们可以把语法规则想象为一个游戏，可以从两个方向中的一个玩。

如果你从语法规则开始，我们可以使用语法规则生成合法的语句，这些生成的语句，称为派生语句，因为它们都是从语法规则中派生出。在游戏的每一步，你都需要选择一条语法规则，然后，按照规则继续运行。形式文法的，大部分术语都是从这个方向中产生的，语法规则被称为产生式，因为，它们生产字符串。

Each production in a context-free grammar has a head—its name—and a body, which describes what it generates. In its pure form, the body is simply a list of symbols. Symbols come in two delectable flavors:

> Restricting heads to a single symbol is a defining feature of context-free grammars. More powerful formalisms like unrestricted grammars allow a sequence of symbols in the head as well as in the body.
> 
> 将头限制为单个符号，是上下文无关语法的一个定义特性，更加强大的形式主义，例如: [无限制文法](https://en.wikipedia.org/wiki/Unrestricted_grammar), 允许头部和body中定义一系列符号

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


> Yes, we need to define a syntax to use for the rules that define our syntax. Should we specify that metasyntax too? What notation do we use for it? It’s languages all the way down!
> 
> 我们需要定义一个语法，使用这些语法规则，我们将定义语言的语法。我们是否也应该指定元语法？我们用什么符号表示它，一直以来都是语言。

Using that, here’s a grammar for breakfast menus:

> Yes, I really am going to be using breakfast examples throughout this entire book. Sorry.
> 
> 很抱歉，我一直在书中，使用早餐举例。

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

> Imagine that we’ve recursively expanded the breakfast rule here several times, like “bacon with bacon with bacon with . . . ” In order to complete the string correctly, we need to add an equal number of “on the side” bits to the end. Tracking the number of required trailing parts is beyond the capabilities of a regular grammar. Regular grammars can express repetition, but they can’t keep count of how many repetitions there are, which is necessary to ensure that the string has the same number of with and on the side parts.
> 
> 想象一下，我们在这里，递归的扩展了早餐规则好几次，为了正确的产生字符串，我们需要在末尾添加相同数量的位，跟踪所需的尾部部分的数量，超过了常规语法的能力。常规语法，可以表示递归，但是无法计算重复的次数。这对于确保字符串的with 和 on the side 部分，维持相同数量是非常重要的。

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

> If you’re so inclined, try using this grammar to generate a few expressions like we did with the breakfast grammar before. Do the resulting expressions look right to you? Can you make it generate anything wrong like 1 + / 3?
> 
> 如果你这么想，试着使用这些语法规则，生成一些表达式，就像我们之前早餐语法那样。这些生成的表达式是正确的吗？它是否会生成像是 1 + / 3 这样的错误？



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

> In particular, we’re defining an abstract syntax tree (AST). In a parse tree, every single grammar production becomes a node in the tree. An AST elides productions that aren’t needed by later phases.
> 
> 特别的，我们实现了一个抽象语法树，在解析树中，每个语法生成都是一个节点，AST树省略了后面阶段不需要的部分。

>Tokens aren’t entirely homogeneous either. Tokens for literals store the value, but other kinds of lexemes don’t need that state. I have seen scanners that use different classes for literals and other kinds of lexemes, but I figured I’d keep things simpler.
> 
> token 也不是完全相同的，文字的token，会保存文字的值，其他类型的token可能不需要这个信息。我见过扫描器对文字和其他类型的词汇，使用不同的类，但是我将让事情更加简单。

> I avoid abbreviations in my code because they trip up a reader who doesn’t know what they stand for. But in compilers I’ve looked at, “Expr” and “Stmt” are so ubiquitous that I may as well start getting you used to them now.
>
> 我在代码中将尽量不使用缩写，因为，缩写会让不明白含义的读者感到困惑。但是，我见过的编译器中，使用 Expr 和 Stmt 是如此的普遍，所以我们将从现在开始习惯使用它们。

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

> Picture me doing an awkward robot dance when you read that. “AU-TO-MATE.”
>
> 当你读到这篇文章时候，想象一下，我在跳舞，跳一个笨拙的机器人舞。

Instead of tediously handwriting each class definition, field declaration, constructor, and initializer, we’ll hack together a script that does it for us. It has a description of each tree type—its name and fields—and it prints out the Java code needed to define a class with that name and state.

我们将编写一个脚本，生成类，而不是，繁琐的编写每一个类定义，字段声明，构造函数，初始化变量。它将描述每一个定义的语法树类型，包含类名称和类字段列表，打印出类名称和相应的Java定义代码

This script is a tiny Java command-line app that generates a file named “Expr.java”:

这个脚本是一个简单的java代码，生成一个Expr.java 文件

```java

// tool/GenerateAst.java, create new file

package com.craftinginterpreters.tool;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Arrays;
import java.util.List;

public class GenerateAst {
  public static void main(String[] args) throws IOException {
    if (args.length != 1) {
      System.err.println("Usage: generate_ast <output directory>");
      System.exit(64);
    }
    String outputDir = args[0];
  }
}

```

>I got the idea of scripting the syntax tree classes from Jim Hugunin, creator of Jython and IronPython.
>
>An actual scripting language would be a better fit for this than Java, but I’m trying not to throw too many languages at you.
>
> 我从Jython 和 IronPython的发明者 Jim Hugunin那里获取到灵感，有关编写语法树类脚本的想法。
> 
> 实际上一门真的脚本语言更加适合做这件事，但是，我不想一下子抛出这么多的语言。

Note that this file is in a different package, .tool instead of .lox. This script isn’t part of the interpreter itself. It’s a tool we, the people hacking on the interpreter, run ourselves to generate the syntax tree classes. When it’s done, we treat “Expr.java” like any other file in the implementation. We are merely automating how that file gets authored.

要注意，这个新的文件位于新的包 tool，而不是之前的lox中。这个脚本不属于解释器的一部分, 它只是一个工具，编程高手用它来，自动生成语法树。该脚本将生成一个 Expr.java文件

To generate the classes, it needs to have some description of each type and its fields.

For brevity’s sake, I jammed the descriptions of the expression types into strings. Each is the name of the class followed by : and the list of fields, separated by commas. Each field has a type and a name.

The first thing defineAst() needs to do is output the base Expr class.

为了生成一个类，对于每个类的类型和字段，我们需要进行一些描述

```java

// tool/GenerateAst.java, in main()

    String outputDir = args[0];
    defineAst(outputDir, "Expr", Arrays.asList(
      "Binary   : Expr left, Token operator, Expr right",
      "Grouping : Expr expression",
      "Literal  : Object value",
      "Unary    : Token operator, Expr right"
    ));
  }


```


为了简洁，我们把表达式类型的描述写入到字符串中，每个类中，后面跟上一个: 符号和一些类字段，字段中间使用逗号分隔。每个字段都有一个类型 和 名称。

defineAst() 需要输出Expr类

```java

// tool/GenerateAst.java add after main()

  private static void defineAst(
      String outputDir, String baseName, List<String> types)
      throws IOException {
    String path = outputDir + "/" + baseName + ".java";
    PrintWriter writer = new PrintWriter(path, "UTF-8");

    writer.println("package com.craftinginterpreters.lox;");
    writer.println();
    writer.println("import java.util.List;");
    writer.println();
    writer.println("abstract class " + baseName + " {");

    writer.println("}");
    writer.close();
  }
  
```

When we call this, baseName is “Expr”, which is both the name of the class and the name of the file it outputs. We pass this as an argument instead of hardcoding the name because we’ll add a separate family of classes later for statements.

当我们调用这个函数，baseName 是 Expr, 它即是类的名称，也是输出的文件的名称。我们将expr当作一个参数，而不是硬编码，因为稍后，我们将添加一个类家族。

Inside the base class, we define each subclass.

在基类中，我们将定义每一个子类

```java

// tool/GenerateAst.java in defineAst()

 writer.println("abstract class " + baseName + " {");

    // The AST classes.
    for (String type : types) {
      String className = type.split(":")[0].trim();
      String fields = type.split(":")[1].trim(); 
      defineType(writer, baseName, className, fields);
    }
    writer.println("}");
	
```

```java

// tool/GenerateAst.java add after defineAst()

  private static void defineType(
      PrintWriter writer, String baseName,
      String className, String fieldList) {
    writer.println("  static class " + className + " extends " +
        baseName + " {");

    // Constructor.
    writer.println("    " + className + "(" + fieldList + ") {");

    // Store parameters in fields.
    String[] fields = fieldList.split(", ");
    for (String field : fields) {
      String name = field.split(" ")[1];
      writer.println("      this." + name + " = " + name + ";");
    }

    writer.println("    }");

    // Fields.
    writer.println();
    for (String field : fields) {
      writer.println("    final " + field + ";");
    }

    writer.println("  }");
  }
  
```

There we go. All of that glorious Java boilerplate is done. It declares each field in the class body. It defines a constructor for the class with parameters for each field and initializes them in the body.

Compile and run this Java program now and it blasts out a new “.java” file containing a few dozen lines of code. That file’s about to get even longer.

我们已经开始构造了，所有java类已经完成了，它声明了类中的每一个字段，它为类定义了一个构造函数，构造函数中包含有每个字段，并且在函数中进行字段初始化。

编译运行这个程序，将会创建一个新的java文件，其中包含几十行代码，java文件将会变得更长。

>This isn’t the world’s most elegant string manipulation code, but that’s fine. It only runs on the exact set of class definitions we give it. Robustness ain’t a priority.
>
> 这不是最优雅的字符串操作代码，但是它可以在我们定义的精确的类集合中，生成我们想要的类定义代码，健壮性不是首要任务。

## 三、Working with Trees

使用树

Put on your imagination hat for a moment. Even though we aren’t there yet, consider what the interpreter will do with the syntax trees. Each kind of expression in Lox behaves differently at runtime. That means the interpreter needs to select a different chunk of code to handle each expression type. With tokens, we can simply switch on the TokenType. But we don’t have a “type” enum for the syntax trees, just a separate Java class for each one.

带上想象的帽子一会儿，尽管我们还没有达到，想象一下，解释器如何出来语法树。Lox的每一种表达式在运行时候，行为都不相同。这意味着，解释器需要选择不同的代码块，来处理不同类型的表达式。使用token，我们可以通过tokenType转换。但是，我们没有语法树的枚举，只是对于每一个语法树，有一个独立的java类。

We could write a long chain of type tests:

我们可以编写一长串类型测试

```java

if (expr instanceof Expr.Binary) {
  // ...
} else if (expr instanceof Expr.Grouping) {
  // ...
} else // ...

```

But all of those sequential type tests are slow. Expression types whose names are alphabetically later would take longer to execute because they’d fall through more if cases before finding the right type. That’s not my idea of an elegant solution.

We have a family of classes and we need to associate a chunk of behavior with each one. The natural solution in an object-oriented language like Java is to put those behaviors into methods on the classes themselves. We could add an abstract interpret() method on Expr which each subclass would then implement to interpret itself.

但是，所有这些顺序式测试都很慢，如果按照表达式类型名称，按照字母顺序排序，然后执行，可能需要更多时间，因为在找到合适的类型之前，可能包含更多的判断，这不是我们认为的优雅的解决方式。

我们有一个类家族，需要将其中每一个类的行为都关联在一起。在Java这样的面向对象语言中，很自然的解决方式是，将这些类行为放入类方法中。我们可以在每个类中都添加一个抽象的方法，interpret() , Expr基类的每一个子类都实现自身的interpret() 方法。

> This exact thing is literally called the “Interpreter pattern” in Design Patterns: Elements of Reusable Object-Oriented Software, by Erich Gamma, et al.
> 
> 这个编程模式被称为[解释器模式](https://en.wikipedia.org/wiki/Interpreter_pattern), 在书籍[《设计模式：可复用面向对象软件的基础》](https://learning.oreilly.com/library/view/design-patterns-elements/0201633612/) 中 


This works alright for tiny projects, but it scales poorly. Like I noted before, these tree classes span a few domains. At the very least, both the parser and interpreter will mess with them. As you’ll see later, we need to do name resolution on them. If our language was statically typed, we’d have a type checking pass.

If we added instance methods to the expression classes for every one of those operations, that would smush a bunch of different domains together. That violates separation of concerns and leads to hard-to-maintain code.

这样解决，对于小项目来说很好，它们的规模很小。正如我之前提到的，这些语法树跨域了几个域，至少，解析器和解释器都会使用到语法树。稍后你将看到，我们将进行名称解析。如果我们的语言是静态类型的，我们将有一个类型检查步骤。

如果我们对于每一个操作，都在表达式实例中添加对应的方法，那么将会把一系列的域，混杂在一起。这违反了[关注点分离](https://en.wikipedia.org/wiki/Separation_of_concerns), 会让后面的代码更难维护。

### 3.1 The expression problem

表达式问题

This problem is more fundamental than it may seem at first. We have a handful of types, and a handful of high-level operations like “interpret”. For each pair of type and operation, we need a specific implementation. Picture a table:

这个问题，比最初看起来更加根本，我们有一些类型，还有一些操作，例如: interpret, 对于每一个类型和操作的组合，我们需要一个特定的实现。表格看起来是这样：

![table](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/table.png?raw=true)

Rows are types, and columns are operations. Each cell represents the unique piece of code to implement that operation on that type.

An object-oriented language like Java assumes that all of the code in one row naturally hangs together. It figures all the things you do with a type are likely related to each other, and the language makes it easy to define them together as methods inside the same class.

This makes it easy to extend the table by adding new rows. Simply define a new class. No existing code has to be touched. But imagine if you want to add a new operation—a new column. In Java, that means cracking open each of those existing classes and adding a method to it.

表格中，行是类型，列是操作，每个单元格表示某种类型的某个操作，具体的实现代码。

在Java这样的面向对象语言中，会将一行中的所有操作，自然的放到一起。它认为我们对于每个类型的所有操作都是相互关联的。Java中可以很简单的将这些方法定义到同一个类中

![rows](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/rows.png?raw=true)

这样，我们可以很容易的扩展新的一行，即添加一个新类。无需修改现有代码。但是，想象一下，如果你添加了一个新的操作，即添加了一列。在Java中，这意味着，我们需要在每个类中，添加一个新的方法。

Functional paradigm languages in the ML family flip that around. There, you don’t have classes with methods. Types and functions are totally distinct. To implement an operation for a number of different types, you define a single function. In the body of that function, you use pattern matching—sort of a type-based switch on steroids—to implement the operation for each type all in one place.

This makes it trivial to add new operations—simply define another function that pattern matches on all of the types.

[ML语言](https://craftinginterpreters.com/representing-code.html)家族中的，函数范型语言正好相反，在那里，你没有类和类方法的概念，类型和函数完全不相同。为了定义某个操作（不同的类型），我们可以定义一个函数，在每个函数中，我们使用类型匹配，根据不同的类型，实现不同的具体操作代码。

这样，我们可以很容易添加一个新的操作，在新操作对应的函数中，我们根据不同类型，定义不同的实现。


![columns](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/columns.png?raw=true)

But, conversely, adding a new type is hard. You have to go back and add a new case to all of the pattern matches in all of the existing functions.

但是，如果我们想要添加一个新的类型，会非常麻烦，我们必须在所有的函数中，添加一个模式，匹配新的类型，并且添加对应的代码。

> ML, short for “metalanguage” was created by Robin Milner and friends and forms one of the main branches in the great programming language family tree. Its children include SML, Caml, OCaml, Haskell, and F#. Even Scala, Rust, and Swift bear a strong resemblance.
> 
> Much like Lisp, it is one of those languages that is so full of good ideas that language designers today are still rediscovering them over forty years later.
> 
> ML, 是元语言的缩写，是由[Robin Milner](https://en.wikipedia.org/wiki/Robin_Milner) 和他的朋友们创建，是伟大的编程语言的一个重要分支，受到它影响的语法包括，SML Caml OCaml Haskell F#等等，即使是 Scala, Rust 和 Swift也和它有很多的相似性。
>
> 与lisp语言相似，ML是一门充满了想法的语言，以致于语言设计者，在40年后的今天，依然可以发现ML中的新想法。

Each style has a certain “grain” to it. That’s what the paradigm name literally says—an object-oriented language wants you to orient your code along the rows of types. A functional language instead encourages you to lump each column’s worth of code together into a function.

A bunch of smart language nerds noticed that neither style made it easy to add both rows and columns to the table. They called this difficulty the “expression problem” because—like we are now—they first ran into it when they were trying to figure out the best way to model expression syntax tree nodes in a compiler.

People have thrown all sorts of language features, design patterns, and programming tricks to try to knock that problem down but no perfect language has finished it off yet. In the meantime, the best we can do is try to pick a language whose orientation matches the natural architectural seams in the program we’re writing.

Object-orientation works fine for many parts of our interpreter, but these tree classes rub against the grain of Java. Fortunately, there’s a design pattern we can bring to bear on it.

每一种编程方式，都有自己的风格，这也体现在编程范式名称上，面向对象编程语言，希望我们沿着表格中行方向，函数式编程，鼓励我们沿着列方向编程。

一群聪明的书呆子注意到，上面的两种编程风格，都无法简单的添加一行或者一列。他们将这个困难，称为表达式问题，因为和我们现在一样，他们也是在试图找到一种最好的方法，对表达式语法树进行建模时候，遇到这个问题。

人们已经抛弃了各种语言特性、设计模式、编程技巧，试图去解决这个问题，但是，还没有一种语言可以完美的解决这个问题。所以，当前我们最好的方法是，选择一种语言，它的编程方向，和我们要实现的语法树更加契合。

面向对象语言，对于解释器部分一般更加合适，但是这些语法树，和Java的编程风格有冲突，幸运的是，我们可以引入新的设计模式，利用它，更好的编写语法树的实现。

>Languages with multimethods, like Common Lisp’s CLOS, Dylan, and Julia do support adding both new types and operations easily. What they typically sacrifice is either static type checking, or separate compilation.
>
> 具有多方法的语言，例如: Common Lisp, 确实支持轻松添加新的类型和操作，但是，它们通常会缺少静态类型的检查或者单独编译。

### 3.2 The Visitor pattern

[访问者模式](https://en.wikipedia.org/wiki/Visitor_pattern)

The Visitor pattern is the most widely misunderstood pattern in all of Design Patterns, which is really saying something when you look at the software architecture excesses of the past couple of decades.

The trouble starts with terminology. The pattern isn’t about “visiting”, and the “accept” method in it doesn’t conjure up any helpful imagery either. Many think the pattern has to do with traversing trees, which isn’t the case at all. We are going to use it on a set of classes that are tree-like, but that’s a coincidence. As you’ll see, the pattern works as well on a single object.

The Visitor pattern is really about approximating the functional style within an OOP language. It lets us add new columns to that table easily. We can define all of the behavior for a new operation on a set of types in one place, without having to touch the types themselves. It does this the same way we solve almost every problem in computer science: by adding a layer of indirection.

当你查看过去几十年的软件架构历史，访问者模式是最容易被误解的模式

问题从术语开始，该模式与访问无关，其中的接受方法，也不会产生任何有用的图像。许多人认为这种模式和遍历语法树有关，但事实并不是这样。我们接下来，将在一组树类，使用访问者模式，但是这只是巧合。正如，你将看到，该模式也适用于单个对象。

访问者模式，在面向对象语言中，类似于函数式。它，可以让我们轻松的添加一列。我们可以在一个地方，定义一组类型上的新操作的所有行为，而不需要接触具体的类型。这与我们解决计算中的几乎所有问题的思想一样——添加一层抽象，间接寻址。

> A beignet (pronounced “ben-yay”, with equal emphasis on both syllables) is a deep-fried pastry in the same family as doughnuts. When the French colonized North America in the 1700s, they brought beignets with them. Today, in the US, they are most strongly associated with the cuisine of New Orleans.
> 
> My preferred way to consume them is fresh out of the fryer at Café du Monde, piled high in powdered sugar, and washed down with a cup of café au lait while I watch tourists staggering around trying to shake off their hangover from the previous night’s revelry
> 
> beignet是一种油炸糕点，和甜甜圈是同一类型糕点。

Before we apply it to our auto-generated Expr classes, let’s walk through a simpler example. Say we have two kinds of pastries: beignets and crullers.

在我们应用该模式于我们的Expr类之前，我们先进行一个简单示例。假设我们有两类糕点：甜饼和煎饼

```java


  abstract class Pastry {
  }

  class Beignet extends Pastry {
  }

  class Cruller extends Pastry {
  }
  
  
```

We want to be able to define new pastry operations—cooking them, eating them, decorating them, etc.—without having to add a new method to each class every time. Here’s how we do it. First, we define a separate interface.

我们想要定义新的糕点操作，烹饪、食用、装饰它们，而不需要向每个类中添加方法，下面是我们的做法。首先，我们定义一个单独的接口

```java 



  interface PastryVisitor {
    void visitBeignet(Beignet beignet); 
    void visitCruller(Cruller cruller);
  }
  
  
```

> In Design Patterns, both of these methods are confusingly named visit(), and they rely on overloading to distinguish them. This leads some readers to think that the correct visit method is chosen at runtime based on its parameter type. That isn’t the case. Unlike overriding, overloading is statically dispatched at compile time.
> 
> Using distinct names for each method makes the dispatch more obvious, and also shows you how to apply this pattern in languages that don’t support overloading.
>
> 在设计模式中，所有方法名称都相同，visit(), 它们非常容易混淆，我们需要使用重载来区分它们。这导致一些读者认为，visit()方法的区分是在运行时候根据参数类型不同，执行不同的方法，但是，实际上，与重写不同，重载是在静态编译阶段区分的。
>
> 对于每个方法使用不同的名称 visitBeignet/visitCruller 可以让分派更加明显。并且，还可以展示在不支持重载的语言中，如何实现访问者模式。

Each operation that can be performed on pastries is a new class that implements that interface. It has a concrete method for each type of pastry. That keeps the code for the operation on both types all nestled snugly together in one class.

Given some pastry, how do we route it to the correct method on the visitor based on its type? Polymorphism to the rescue! We add this method to Pastry:


可以在糕点上执行的每个操作都是一个实现该接口的新类。每个糕点，都有具体的制作方法，这样，两个类型的相同操作代码将写入一个类中

给定一些糕点，我们如何根据其类型将它们发送到访问者的正确方法？我们将使用多态，每个子类都将实现

```java


 abstract class Pastry {
    abstract void accept(PastryVisitor visitor);
  }
  
```

```java

 class Beignet extends Pastry {
    @Override
    void accept(PastryVisitor visitor) {
      visitor.visitBeignet(this);
    }
  }
  
```

```java


  class Cruller extends Pastry {
    @Override
    void accept(PastryVisitor visitor) {
      visitor.visitCruller(this);
    }
  }
  
```

To perform an operation on a pastry, we call its accept() method and pass in the visitor for the operation we want to execute. The pastry—the specific subclass’s overriding implementation of accept()—turns around and calls the appropriate visit method on the visitor and passes itself to it.

That’s the heart of the trick right there. It lets us use polymorphic dispatch on the pastry classes to select the appropriate method on the visitor class. In the table, each pastry class is a row, but if you look at all of the methods for a single visitor, they form a column.

We added one accept() method to each class, and we can use it for as many visitors as we want without ever having to touch the pastry classes again. It’s a clever pattern.

要在糕点上执行操作，我们调用它的accept() 方法，并且传参是 要执行操作的访问者。糕点的子类，重写 accept() 方法，它会对应不同的访问者，调用不同的访问方法，并将自身当作传参。

这就是关键所在，它允许我们在糕点类上使用多态，选择访问者类上的特定方法，在下面的表格中，每个具体的糕点类都是一行，但是，当你查看单个访问者的所有方法，它们将会是一列

我们为每一个糕点子类，添加了一个accept方法，我们可以根据需要，为任意多的访问者使用不同的accept方法，而无须修改糕点类，这是一个好的模式。

![visitor](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/visitor.png?raw=true)

### 3.3 Visitors for expressions

表达式访问者模式

OK, let’s weave it into our expression classes. We’ll also refine the pattern a little. In the pastry example, the visit and accept() methods don’t return anything. In practice, visitors often want to define operations that produce values. But what return type should accept() have? We can’t assume every visitor class wants to produce the same type, so we’ll use generics to let each implementation fill in a return type.

好的，接下来，让我们在表达式类中，使用访问者模式，我们还将对访问者模式进行一些改进，在糕点示例中，vistor() 和 accept() 方法，没有具体的返回值。实际上，vistor期望定义可以产生值的操作，但是,accept() 方法，应该有什么类型的返回值呢？我们不能假设，每一个visitor都返回相同的类型，所以，我们将使用泛型，让每一个具体实现，去填充类型

>Another common refinement is an additional “context” parameter that is passed to the visit methods and then sent back through as a parameter to accept(). That lets operations take an additional parameter. The visitors we’ll define in the book don’t need that, so I omitted it.
> 
> 另一个常见的细化是通过一个参数context, 该参数传递给visit(), 然后，在作为一个参数，发送回accept()， 这允许操作使用附加参数，我们在本书中定义的访问者不需要该参数，所以，我们省略了context的介绍。

First, we define the visitor interface. Again, we nest it inside the base class so that we can keep everything in one file.

首先，我们定义访问者接口，同样的，我们将其嵌套在基类中，以便我们将所有内容保存在一个文件中。

```java

// tool/GenerateAst.java, in defineAst()

    writer.println("abstract class " + baseName + " {");

    defineVisitor(writer, baseName, types);

    // The AST classes.


```

```java

// tool/GenerateAst.java, add after defineAst()
  private static void defineVisitor(
      PrintWriter writer, String baseName, List<String> types) {
    writer.println("  interface Visitor<R> {");

    for (String type : types) {
      String typeName = type.split(":")[0].trim();
      writer.println("    R visit" + typeName + baseName + "(" +
          typeName + " " + baseName.toLowerCase() + ");");
    }

    writer.println("  }");
  }
  
```
Here, we iterate through all of the subclasses and declare a visit method for each one. When we define new expression types later, this will automatically include them.

Inside the base class, we define the abstract accept() method.

在这里，我们遍历每一个子类，并且为每一个子类，声明了一个访问方法，当我们定义新的表达式类时候，这将自动包括它们

在基类中，我们将定义抽象类 accept() 方法

```java

// tool/GenerateAst.java, in defineAst()

      defineType(writer, baseName, className, fields);
    }

    // The base accept() method.
    writer.println();
    writer.println("  abstract <R> R accept(Visitor<R> visitor);");

    writer.println("}");


```

Finally, each subclass implements that and calls the right visit method for its own type.

最后，每一个子类中都实现了accept() 方法，每个具体的accept() 方法将会调用正确的visit方法

```java

// tool/GenerateAst.java, in defineType()

    writer.println("    }");

    // Visitor pattern.
    writer.println();
    writer.println("    @Override");
    writer.println("    <R> R accept(Visitor<R> visitor) {");
    writer.println("      return visitor.visit" +
        className + baseName + "(this);");
    writer.println("    }");

    // Fields.


```

There we go. Now we can define operations on expressions without having to muck with the classes or our generator script. Compile and run this generator script to output an updated “Expr.java” file. It contains a generated Visitor interface and a set of expression node classes that support the Visitor pattern using it.

Before we end this rambling chapter, let’s implement that Visitor interface and see the pattern in action.

我们继续开始，现在，我们可以在表达式上定义操作，而不需要处理类和生成类脚本， 编译运行这个脚本，更新Expr.java 文件，它包含一个生成Visitor接口，和一系列表达式节点类，支持Visitor模式。

在我们结束这漫无边际的一章之前，让我们实现Visitor接口，并且实际使用访问者模式。


## 四、A (Not Very) Pretty Printer

When we debug our parser and interpreter, it’s often useful to look at a parsed syntax tree and make sure it has the structure we expect. We could inspect it in the debugger, but that can be a chore.

Instead, we’d like some code that, given a syntax tree, produces an unambiguous string representation of it. Converting a tree to a string is sort of the opposite of a parser, and is often called “pretty printing” when the goal is to produce a string of text that is valid syntax in the source language.

That’s not our goal here. We want the string to very explicitly show the nesting structure of the tree. A printer that returned 1 + 2 * 3 isn’t super helpful if what we’re trying to debug is whether operator precedence is handled correctly. We want to know if the + or * is at the top of the tree.

当我们调试解析器和解释器时候，查看已经解析的语法树，并且确保它们拥有我们期望的结构，通常非常重要，我们可以在调试阶段检查。但是，这也是一项繁重的任务

相反，我们更喜欢一些输出，在给定语法树场景，输出明确的字符串，表示这个语法树。将树返回为一个字符串，过程和解析器正好相反，这通常称为完美输出，如果我们输出的字符串是一个合法的原始语言的字符串

但是，这不是我们的目标，我们希望字符串，可以非常明确的显示语法树结构，如果我们想要调试运算符优先级是否正确被处理，那么我们直接输出 1 +  2*3 , 不会有很大的作用。我们想要知道，+ 还是 * 在语法树的顶部。

To that end, the string representation we produce isn’t going to be Lox syntax. Instead, it will look a lot like, well, Lisp. Each expression is explicitly parenthesized, and all of its subexpressions and tokens are contained in that.

为此，我们输出的字符串不是符合lox语法的字符串，相反，它看起来更像是 lisp语言，每个表达式都显示的用括号括起来，其中包含所有的子表达式和token

Given a syntax tree like:

给定下面的语法树，

![expression](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/expression.png?raw=true)

It produces:

它对应的输出字符串是

```(* (- 123) (group 45.67))```


Not exactly “pretty”, but it does show the nesting and grouping explicitly. To implement this, we define a new class.

虽然，不是非常完美，但是它，的确显示了嵌套和分组结构，为了实现这个输出，我们定义一个新的类


```java

// lox/AstPrinter.java, create new file

package com.craftinginterpreters.lox;

class AstPrinter implements Expr.Visitor<String> {
  String print(Expr expr) {
    return expr.accept(this);
  }
}


```

As you can see, it implements the visitor interface. That means we need visit methods for each of the expression types we have so far.

如我们所见，它实现了访问者模式的接口，这意味着我们需要对每个表达式类型实现访问方法，

```java

// lox/AstPrinter.java, add after print()

    return expr.accept(this);
  }

  @Override
  public String visitBinaryExpr(Expr.Binary expr) {
    return parenthesize(expr.operator.lexeme,
                        expr.left, expr.right);
  }

  @Override
  public String visitGroupingExpr(Expr.Grouping expr) {
    return parenthesize("group", expr.expression);
  }

  @Override
  public String visitLiteralExpr(Expr.Literal expr) {
    if (expr.value == null) return "nil";
    return expr.value.toString();
  }

  @Override
  public String visitUnaryExpr(Expr.Unary expr) {
    return parenthesize(expr.operator.lexeme, expr.right);
  }
}

```


Literal expressions are easy—they convert the value to a string with a little check to handle Java’s null standing in for Lox’s nil. The other expressions have subexpressions, so they use this parenthesize() helper method:

文字表达式的访问者接口很容易实现——它们将值转换为字符串，只需要稍微检查一下，是否是Java 中的null值，如果是，需要转为lox语言中的nil；其他的表达式，含有子表达式，因此，我们还需要借助这个括号处理帮助函数

```java

// lox/AstPrinter.java, add after visitUnaryExpr()

  private String parenthesize(String name, Expr... exprs) {
    StringBuilder builder = new StringBuilder();

    builder.append("(").append(name);
    for (Expr expr : exprs) {
      builder.append(" ");
      builder.append(expr.accept(this));
    }
    builder.append(")");

    return builder.toString();
  }


```

It takes a name and a list of subexpressions and wraps them all up in parentheses, yielding a string like:

它接受一个名称和一串子表达式，并且将它们都是用括号括起来，生成如下的字符串

```

(+ 1 2)

```

Note that it calls accept() on each subexpression and passes in itself. This is the recursive step that lets us print an entire tree.

需要注意的是，每个子表达式都会调用accept() 方法，并且将自身传参，这样，我们可以递归调用，打印出语法树。

> This recursion is also why people think the Visitor pattern itself has to do with trees.
> 
> 这种递归，也让人们更加认为，访问者模式和语法树是有关的。

We don’t have a parser yet, so it’s hard to see this in action. For now, we’ll hack together a little main() method that manually instantiates a tree and prints it.

我们还没有解析器，所以很难看到具体的实际应用。但是，我们将先实现一个demo main函数，手动实例化语法树，并且打印


```java
// lox/AstPrinter.java, add after parenthesize()

  public static void main(String[] args) {
    Expr expression = new Expr.Binary(
        new Expr.Unary(
            new Token(TokenType.MINUS, "-", null, 1),
            new Expr.Literal(123)),
        new Token(TokenType.STAR, "*", null, 1),
        new Expr.Grouping(
            new Expr.Literal(45.67)));

    System.out.println(new AstPrinter().print(expression));
  }


```

If we did everything right, it prints:

接下来，我们运行，将会输出

```(* (- 123) (group 45.67))```

You can go ahead and delete this method. We won’t need it. Also, as we add new syntax tree types, I won’t bother showing the necessary visit methods for them in AstPrinter. If you want to (and you want the Java compiler to not yell at you), go ahead and add them yourself. It will come in handy in the next chapter when we start parsing Lox code into syntax trees. Or, if you don’t care to maintain AstPrinter, feel free to delete it. We won’t need it again.

我们可以继续下去，删除当前demo，我们不需要它。此外，当我们添加新的表达式类型时候，我不会费心在AstPrinter中实现它们的访问方法，如果你想要添加（并且希望Java编译器不会报错），那么你可以自己添加对应的vistor方法，当我们将Lox代码解析为语法树时候，它将在接下来发挥作用。或者，如果你不想维护AstPrinter, 可以随时删除它，我们不再需要它。



## 五、CHALLENGES

习题

1. Earlier, I said that the |, *, and + forms we added to our grammar metasyntax were just syntactic sugar. Take this grammar:

   早先，我讲了我们的元语法中，只是将 | * + 当作一些语法糖，
   
   ```
   
   expr → expr ( "(" ( expr ( "," expr )* )? ")" | "." IDENTIFIER )+
     | IDENTIFIER
     | NUMBER

   
   ```
   
   请生成同样的语法规则，不使用语法糖，上面这段语法规则，是描述了什么表达式？
   
1. The Visitor pattern lets you emulate the functional style in an object-oriented language. Devise a complementary pattern for a functional language. It should let you bundle all of the operations on one type together and let you define new types easily.

   (SML or Haskell would be ideal for this exercise, but Scheme or another Lisp works as well.)

	访问者模式允许我们在面向对象语言中使用模拟函数式编程风格。请，为函数式语言设计一个模式，它允许我们将所有的操作写入到一起，并且允许我们轻松的定义新的类型
	
	SML 和 Haskell 是这个练习的理想语言，但是 Scheme 和 Lisp 也可以
	
1. In reverse Polish notation (RPN), the operands to an arithmetic operator are both placed before the operator, so 1 + 2 becomes 1 2 +. Evaluation proceeds from left to right. Numbers are pushed onto an implicit stack. An arithmetic operator pops the top two numbers, performs the operation, and pushes the result. Thus, this:

	(1 + 2) * (4 - 3) in RPN becomes: 1 2 + 4 3 - *

	Define a visitor class for our syntax tree classes that takes an expression, converts it to RPN, and returns the resulting string.

	在逆波兰表示法中RPN，算术运算符的操作数放到操作符之前，例如：1 + 2将变为 1 2 + , 执行时候，从左到右。将操作数放入堆栈中，遇到操作符后，将弹出对应的操作数，执行运算，将结果重新放入堆栈
	
	例如: 
	
	(1 + 2) * (4 - 3)  使用逆波兰表示法 1 2 + 4 3 - *
	
	
	定义一个访问者类，针对不同的表达式类型，输出其逆波兰表示法。
