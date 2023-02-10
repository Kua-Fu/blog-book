# 函数

> And that is also the way the human mind works—by the compounding of old ideas into new structures that become new ideas that can themselves be used in compounds, and round and round endlessly, growing ever more remote from the basic earthbound imagery that is each language’s soil.
> 
> 这也是人类思维的方式，通过将旧思想组合成新的结构，从而形成新思想，这些新思想本身可以用在复合词中，并且不断循环，越来越远离每种语言的土壤中的基本地球意象。
> 
><p align="right">  Douglas R. Hofstadter, I Am a Strange Loop </p>

[I Am a Strange Loop 我是一个奇怪的循环](https://en.wikipedia.org/wiki/I_Am_a_Strange_Loop) </p>

This chapter marks the culmination of a lot of hard work. The previous chapters add useful functionality in their own right, but each also supplies a piece of a puzzle. We’ll take those pieces—expressions, statements, variables, control flow, and lexical scope—add a couple more, and assemble them all into support for real user-defined functions and function calls.

这一章标志着许多艰苦工作的高潮，前面的章节中各自添加了有用的功能，但是每一个章节都提供了一块拼图。我们将把这些表达式、语句、变量、控制流和词法作用域，组合在一起，从而支持用户自定义函数和函数调用

## 一、Function Calls

函数调用

You’re certainly familiar with C-style function call syntax, but the grammar is more subtle than you may realize. Calls are typically to named functions like:

我们当然熟悉了C样式的函数调用语法，但是语法比我们认识的，可能更加微妙。通常调用函数的语法如下

```C
average(1, 2);
```

> The name is part of the call syntax in Pascal. You can call only named functions or functions stored directly in variables.
> 
> 函数名称也是调用语法的一部分，在Pascal中。只能调用命名函数或者直接存储在变量中的函数。
>
> ![lambda](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/lambda.png?raw=true)

But the name of the function being called isn’t actually part of the call syntax. The thing being called—the callee—can be any expression that evaluates to a function. (Well, it does have to be a pretty high precedence expression, but parentheses take care of that.) For example:

但是被调用函数的名称实际上不是调用语法的一部分，被调用者调用的对象可以是任何表达式，计算结果是一个函数。（好吧，它必须是一个非常高优先级的表达式，但是括号可以解决这个问题）例如：

```C

getCallback()();

```

There are two call expressions here. The first pair of parentheses has getCallback as its callee. But the second call has the entire getCallback() expression as its callee. It is the parentheses following an expression that indicate a function call. You can think of a call as sort of like a postfix operator that starts with (.

这里有两个调用表达式，第一对括号将 getCallback作为其调用者，但是第二个调用将整个 getCallback() 表达式作为其被调用者。表达式后面的括号表示函数调用，我们可以将一个调用看成是一个以 （ 开始的后缀运算符


This “operator” has higher precedence than any other operator, even the unary ones. So we slot it into the grammar by having the unary rule bubble up to a new call rule.

这个运算符的优先级高于任何其他运算符，即使是一元运算符。因此，我们通过将一元规则冒泡为新的调用规则，将其插入语法中

```java

unary          → ( "!" | "-" ) unary | call ;
call           → primary ( "(" arguments? ")" )* ;

```

> The rule uses * to allow matching a series of calls like fn(1)(2)(3). Code like that isn’t common in C-style languages, but it is in the family of languages derived from ML. There, the normal way of defining a function that takes multiple arguments is as a series of nested functions. Each function takes one argument and returns a new function. That function consumes the next argument, returns yet another function, and so on. Eventually, once all of the arguments are consumed, the last function completes the operation.
> 
> 这个规则使用* 匹配一系列的调用，例如: fn(1)(2)(3), 这样的代码在C风格语言中并不常见，但是它在ML派生的语言家族中很常见，在那里，定义一个包含多个参数的函数的正常方式是一系列嵌套函数。每个函数接受一个参数，并且返回一个新函数，该函数使用下一个参数，返回另一个函数，以此类推。最终，一旦所有参数都被使用，最后一个函数完成操作。
> 
> This style, called currying, after Haskell Curry (the same guy whose first name graces that other well-known functional language), is baked directly into the language syntax so it’s not as weird looking as it would be here.
> 
> 这种风格被称为 [currying](https://en.wikipedia.org/wiki/Haskell_Curry), 

This rule matches a primary expression followed by zero or more function calls. If there are no parentheses, this parses a bare primary expression. Otherwise, each call is recognized by a pair of parentheses with an optional list of arguments inside. The argument list grammar is:

此规则匹配一个主表达式，后面跟随零个或者多个函数调用，如果没有括号，这将解析一个空的主表达式，否则，每个调用都由一对括号识别，括号内有可选的参数列表，参数列表语法为

```java

arguments      → expression ( "," expression )* ;

```

This rule requires at least one argument expression, followed by zero or more other expressions, each preceded by a comma. To handle zero-argument calls, the call rule itself considers the entire arguments production to be optional.

此规则要求至少有一个参数表达式，后面跟随零个或者多个其他表达式，每个表达式前面都有逗号，为了处理零参数调用，调用规则本身认为整个参数生成是可选的，

I admit, this seems more grammatically awkward than you’d expect for the incredibly common “zero or more comma-separated things” pattern. There are some sophisticated metasyntaxes that handle this better, but in our BNF and in many language specs I’ve seen, it is this cumbersome.

我承认，这在语法上似乎比你想象中的“零或者更多逗号分隔的东西”模式更尴尬，有一些复杂的元语法可以更好的处理这个问题，但在我们的巴科斯范式 和我见过的其他语言中，它都是如此麻烦。

Over in our syntax tree generator, we add a new node.

在语法树生成器中，我们添加了一个新节点。


```java

// tool/GenerateAst.java in main()

"Binary   : Expr left, Token operator, Expr right",
"Call     : Expr callee, Token paren, List<Expr> arguments",
"Grouping : Expr expression",

```

It stores the callee expression and a list of expressions for the arguments. It also stores the token for the closing parenthesis. We’ll use that token’s location when we report a runtime error caused by a function call.

它存储被调用者表达式，和参数的表达式列表，它还存储右括号token，当我们报告由函数调用导致的运行时错误时候，我们将使用该token的位置

Crack open the parser. Where unary() used to jump straight to primary(), change it to call, well, call().

打开解析器，其中unary() 用于直接跳转到primary(), 将其修改为call()

```java

// lox/Parser.java, in unary(), replace 1 line

      return new Expr.Unary(operator, right);
    }

    return call();
  }
  
```

call() 函数定义如下

```java

// lox/Parser.java, add after unary()

  private Expr call() {
    Expr expr = primary();

    while (true) { 
      if (match(LEFT_PAREN)) {
        expr = finishCall(expr);
      } else {
        break;
      }
    }

    return expr;
  }


```

The code here doesn’t quite line up with the grammar rules. I moved a few things around to make the code cleaner—one of the luxuries we have with a handwritten parser. But it’s roughly similar to how we parse infix operators. First, we parse a primary expression, the “left operand” to the call. Then, each time we see a (, we call finishCall() to parse the call expression using the previously parsed expression as the callee. The returned expression becomes the new expr and we loop to see if the result is itself called.

这里的代码不太符合语法规则，我移动了一些东西，让代码更干净，这是我们使用手写解析器的一种优点，但是它大致类似于我们解析中缀运算符的方式，首先，我们解析一个主表达式，即调用的“左操作数”，然后，每次看到（）时候，我们调用finishCall() 以使用先前解析的表达式作为被调用者，解析调用表达式。返回的表达式成为新的expr， 我们循环查看结果本身是否被调用。

> This code would be simpler as while (match(LEFT_PAREN)) instead of the silly while (true) and break. Don’t worry, it will make sense when we expand the parser later to handle properties on objects.
>
> 如果使用 while(match(LEFT_PAREN)) 而不是使用 while(true) ，不用担心，当我们稍后展开解析器来处理对象上属性时候，这是有意义的

The code to parse the argument list is in this helper:

解析参数列表的代码如下

```java

// lox/Parser.java, add after unary()

  private Expr finishCall(Expr callee) {
    List<Expr> arguments = new ArrayList<>();
    if (!check(RIGHT_PAREN)) {
      do {
        arguments.add(expression());
      } while (match(COMMA));
    }

    Token paren = consume(RIGHT_PAREN,
                          "Expect ')' after arguments.");

    return new Expr.Call(callee, paren, arguments);
  }
  
```

This is more or less the arguments grammar rule translated to code, except that we also handle the zero-argument case. We check for that case first by seeing if the next token is ). If it is, we don’t try to parse any arguments.

这或多或少是转换为代码的参数语法规则，除了我们还处理零参数情况，我们首先检查下一个token是否为 ），如果是，我们不会尝试解析任何参数

Otherwise, we parse an expression, then look for a comma indicating that there is another argument after that. We keep doing that as long as we find commas after each expression. When we don’t find a comma, then the argument list must be done and we consume the expected closing parenthesis. Finally, we wrap the callee and those arguments up into a call AST node.

否则，我们解析一个表达式，然后查找一个逗号，逗号表示后面还有另外一个参数，只要我们在每个表达式后面找到逗号，我们将一直这样做。如果没有逗号，表示我们已经完成参数列表，并使用预期的右括号。最后，我们将被调用者和这些参数包装为一个 Call节点中


### 1.1 Maximum argument counts

最大的参数数量

Right now, the loop where we parse arguments has no bound. If you want to call a function and pass a million arguments to it, the parser would have no problem with it. Do we want to limit that?

现在，我们解析参数的循环没有边界，如果你想调用一个函数并传递一百万个参数给它，解析器也不会有问题，我们要限制参数的数量吗？

Other languages have various approaches. The C standard says a conforming implementation has to support at least 127 arguments to a function, but doesn’t say there’s any upper limit. The Java specification says a method can accept no more than 255 arguments.

其他语言有各种方法，C标准里面，一个符合标准的实现必须支持至少127个函数参数，但是没有要求参数上限。Java规范中，一个方法最多只能接受255个参数。

> The limit is 254 arguments if the method is an instance method. That’s because this—the receiver of the method—works like an argument that is implicitly passed to the method, so it claims one of the slots.
> 
> 如果方法是实例方法，则限制为254个参数，这是因为该方法的接收方的工作方式类似于隐式传递给该方法的参数，因此会占用一个参数

Our Java interpreter for Lox doesn’t really need a limit, but having a maximum number of arguments will simplify our bytecode interpreter in Part III. We want our two interpreters to be compatible with each other, even in weird corner cases like this, so we’ll add the same limit to jlox.

我们实现Lox的Java解释器，实际上不需要限制，但是如果我们限制函数的参数最大数量，将简化第三部分中的字节码解释器，我们希望两个解释器彼此兼容，即使在像这样的奇怪的角落中。因此我们对jlox 添加相同的限制。

```java

// lox/Parser.java, in finishCall()
      do {
        if (arguments.size() >= 255) {
          error(peek(), "Can't have more than 255 arguments.");
        }
        arguments.add(expression());
		
```


Note that the code here reports an error if it encounters too many arguments, but it doesn’t throw the error. Throwing is how we kick into panic mode which is what we want if the parser is in a confused state and doesn’t know where it is in the grammar anymore. But here, the parser is still in a perfectly valid state—it just found too many arguments. So it reports the error and keeps on keepin’ on.

注意，如果代码遇到了太多的参数，它会报错，但不会抛出错误，抛出是我们进入恐慌模式的方式，如果解析器处于混乱状态并且不知道它在语法中的位置，这就是我们想要的。但是在这里，解析器仍然处于完成有效的状态，它只是发现了太多的参数，所以它会报错，并将继续运行。

### 1.2 Interpreting function calls

解释调用函数

We don’t have any functions we can call, so it seems weird to start implementing calls first, but we’ll worry about that when we get there. First, our interpreter needs a new import.

我们没有任何可以调用的函数，所以，首先开始实现调用似乎有些奇怪，但是当我们到达那里时候，我们会担心这一点。首先，我们的解释器需要新的导入

```java
// lox/Interpreter.java

import java.util.ArrayList;
import java.util.List;

```

As always, interpretation starts with a new visit method for our new call expression node.

一如既往，解释器从新的调用表达式节点的新访问方法开始

```java

// lox/Interpreter.java, add after visitBinaryExpr()

  @Override
  public Object visitCallExpr(Expr.Call expr) {
    Object callee = evaluate(expr.callee);

    List<Object> arguments = new ArrayList<>();
    for (Expr argument : expr.arguments) { 
      arguments.add(evaluate(argument));
    }

    LoxCallable function = (LoxCallable)callee;
    return function.call(this, arguments);
  }
```

First, we evaluate the expression for the callee. Typically, this expression is just an identifier that looks up the function by its name, but it could be anything. Then we evaluate each of the argument expressions in order and store the resulting values in a list.

首先，我们计算被调用者的表达式，通常，这个表达式只是一个标识符，它通过名称查找函数，但是它可以是任何东西。然后我们按照顺序计算每一个参数表达式，并将结果存储在列表中

> This is another one of those subtle semantic choices. Since argument expressions may have side effects, the order they are evaluated could be user visible. Even so, some languages like Scheme and C don’t specify an order. This gives compilers freedom to reorder them for efficiency, but means users may be unpleasantly surprised if arguments aren’t evaluated in the order they expect.
> 
> 这是另一种微妙的语义选择，由于参数表达式可能有副作用，因此它们的求值顺序可能是用户可见的，即使如此，一些语言，例如：Scheme和C都不指定顺序，这让编译器可以自由的对它们进行重新排序以提高效率，但是这意味着，如果参数没有按照预期的顺序进行求值，用户可能会感到不快。

Once we’ve got the callee and the arguments ready, all that remains is to perform the call. We do that by casting the callee to a LoxCallable and then invoking a call() method on it. The Java representation of any Lox object that can be called like a function will implement this interface. That includes user-defined functions, naturally, but also class objects since classes are “called” to construct new instances. We’ll also use it for one more purpose shortly.

一旦我们准备好了被调用者和参数，剩下的就是执行调用函数，我们通过将被调用者强制转换为LoxCallable, 然后对其调用call() 方法来实现这一点。可以像函数一样调用的任何Lox对象，都需要实现这个接口。这当然包含用户定义的函数，但是也包含类对象。因为类也可以被调用构造新的实例。不久，我们还会将其用于另外一个目的

> I stuck “Lox” before the name to distinguish it from the Java standard library’s own Callable interface. Alas, all the good simple names are already taken.
> 
> 我在名称之前添加上"Lox", 以区别Java标准库自己的Callable接口，哎，所有简单的好名字，已经被占用了。


There isn’t too much to this new interface.

这个新接口没有太多代码

```java

// lox/LoxCallable.java, create new file

package com.craftinginterpreters.lox;

import java.util.List;

interface LoxCallable {
  Object call(Interpreter interpreter, List<Object> arguments);
}

```

We pass in the interpreter in case the class implementing call() needs it. We also give it the list of evaluated argument values. The implementer’s job is then to return the value that the call expression produces.

我们传入解释器，以防实现call() 的类需要它，我们还为它提供了已经求值的参数列表，然后，实现者的任务是返回调用表达式生成的值。


### 1.3 Call type errors

调用类型错误

Before we get to implementing LoxCallable, we need to make the visit method a little more robust. It currently ignores a couple of failure modes that we can’t pretend won’t occur. First, what happens if the callee isn’t actually something you can call? What if you try to do this:

在实现LoxCallable接口之前，我们需要使调用方法更加健壮。它目前忽略了一些我们无法假装不会发生的错误。首先，如果被调用者实际上不是我们可以调用的对象，会发生什么？如果我们尝试这样做

```java

"totally not a function"();

```

Strings aren’t callable in Lox. The runtime representation of a Lox string is a Java string, so when we cast that to LoxCallable, the JVM will throw a ClassCastException. We don’t want our interpreter to vomit out some nasty Java stack trace and die. Instead, we need to check the type ourselves first.

在Lox中不能调用一个字符串，Lox字符串运行时表示一个Java字符串，因此当我们将其转换为LoxCallable时候，JVM将抛出ClassCastException类型错误，我们不希望解释器吐出Java相关的堆栈报错。所以，我们将检查类型

```java

// lox/Interpreter.java, in visitCallExpr()

    }

    if (!(callee instanceof LoxCallable)) {
      throw new RuntimeError(expr.paren,
          "Can only call functions and classes.");
    }

    LoxCallable function = (LoxCallable)callee;
	
```

We still throw an exception, but now we’re throwing our own exception type, one that the interpreter knows to catch and report gracefully.

我们将抛出一个异常，但是现在我们抛出的是自己的异常类型，一个解释器应该知道如何捕获并优雅的报告异常


### 1.4 Checking arity

The other problem relates to the function’s arity. Arity is the fancy term for the number of arguments a function or operation expects. Unary operators have arity one, binary operators two, etc. With functions, the arity is determined by the number of parameters it declares.

另外一个问题与函数的arity 有关，arity是函数或者操作所期望的参数数量的一个花哨术语。一元运算符的 arity是1， 二元运算符的arity 是2，等等。对于函数，arity由函数的声明中的参数数量决定。


```java

fun add(a, b, c) {
  print a + b + c;
}

```

This function defines three parameters, a, b, and c, so its arity is three and it expects three arguments. So what if you try to call it like this:

上面的函数，定义了三个参数，a/b/c, 因此它的arity 是3，需要三个参数，所以，如果我们试着这样调用它

```java

add(1, 2, 3, 4); // Too many.
add(1, 2);       // Too few.

```

不同语言对于这个问题采取了不同的方法。当然，大多数的静态类型语言在编译时候，都会检查这一点。如果参数计数和函数的arity不匹配，则拒绝编译代码。JavaScript 会丢弃传递的任何额外参数，如果传递的参数不够，它会用类似于null但不是真正的值undefined， 填充缺失的参数。Python更加严格，如果参数列表太短或者太长，则会引发运行时错误。

I think the latter is a better approach. Passing the wrong number of arguments is almost always a bug, and it’s a mistake I do make in practice. Given that, the sooner the implementation draws my attention to it, the better. So for Lox, we’ll take Python’s approach. Before invoking the callable, we check to see if the argument list’s length matches the callable’s arity.

我认为后者是更好的方式，传递错误数量的参数，几乎总是一个错误，这是我在实践犯的错误。考虑到这一点，数量检查工作越早越好。所以对于Lox，我们将采用Python 的方式，在调用可调用对象之前，我们检查参数列表的长度是否和可调用对象的arity 匹配。

```java

// lox/Interpreter.java, in visitCallExpr()

    LoxCallable function = (LoxCallable)callee;
    if (arguments.size() != function.arity()) {
      throw new RuntimeError(expr.paren, "Expected " +
          function.arity() + " arguments but got " +
          arguments.size() + ".");
    }

    return function.call(this, arguments);


```

That requires a new method on the LoxCallable interface to ask it its arity.

这需要LoxCallable 接口上的一个新方法来获取函数的arity

We could push the arity checking into the concrete implementation of call(). But, since we’ll have multiple classes implementing LoxCallable, that would end up with redundant validation spread across a few classes. Hoisting it up into the visit method lets us do it in one place.

我们可以将arity 检查推到call() 的具体实现时候，但是，因为，我们将有多个类实现的LoxCallable接口，这将导致冗余验证方法，分散在每一个类中。将其放到visit() 方法中，我们只需要统一检查即可。

## 二、Native Functions

基本函数

We can theoretically call functions, but we have no functions to call yet. Before we get to user-defined functions, now is a good time to introduce a vital but often overlooked facet of language implementations—native functions. These are functions that the interpreter exposes to user code but that are implemented in the host language (in our case Java), not the language being implemented (Lox).

理论上，我们可以调用函数，但是我们还没有函数可以调用。在我们讨论用户自定义函数之前，现在是介绍语言实现本机函数的一个重要但是经常被忽略的方面的好时机。这些是解释器向用户代码公开的函数，但是这些函数是用宿主语言（在我们例子中是Java）实现的，而不是正在实现的语言（Lox）

Sometimes these are called primitives, external functions, or foreign functions. Since these functions can be called while the user’s program is running, they form part of the implementation’s runtime. A lot of programming language books gloss over these because they aren’t conceptually interesting. They’re mostly grunt work.

有时候，这些函数被称为原语、外部函数或者扩展函数，由于这些函数可以在用户程序运行时候调用，因此它们构成了实现运行时候的一部分，很多编程语言书籍都对这些进行了说明，因为它们在概念上并不有趣，它们大多数是粗活。

> Curiously, two names for these functions—“native” and “foreign”—are antonyms. Maybe it depends on the perspective of the person choosing the term. If you think of yourself as “living” within the runtime’s implementation (in our case, Java) then functions written in that are “native”. But if you have the mindset of a user of your language, then the runtime is implemented in some other “foreign” language.
> 
> 奇怪的是，这些函数的两个名称——本地、外来，是反义词，也许这取决于选择术语的人的观点。如果你认为自己生活在运行时候的实现中，在我们的例子是Java，那么用它编写的函数就是原生的，但是，如果你把自己的语言当作原有，那么运行时的其他语言实现的，称为外来
>
> Or it may be that “native” refers to the machine code language of the underlying hardware. In Java, “native” methods are ones implemented in C or C++ and compiled to native machine code.
>
> 或者，本机是指底层硬件的机器代码语言。在Java中，本机方法是用C或者C++实现，并且编译成本机机器代码的方法
>
> ![foreign](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/foreign.png?raw=true)

But when it comes to making your language actually good at doing useful stuff, the native functions your implementation provides are key. They provide access to the fundamental services that all programs are defined in terms of. If you don’t provide native functions to access the file system, a user’s going to have a hell of a time writing a program that reads and displays a file.

但是，当谈到让你的语言真正擅长做有用的事情时候，你的实现提供的本机函数是关键。它们提供对所有程序定义的基本服务的访问。如果你不提供本机函数来访问文件系统，那么用户将很难编写读取和显示文件的程序。

> A classic native function almost every language provides is one to print text to stdout. In Lox, I made print a built-in statement so that we could get stuff on screen in the chapters before this one.
>
> 几乎每种语言都提供了一个经典的本机函数，即将文本输出到标准输出，在Lox中，我制作了一个内置语句，这样，我们就可以在本章之前的章节中看到屏幕上的内容了
>
> Once we have functions, we could simplify the language by tearing out the old print syntax and replacing it with a native function. But that would mean that examples early in the book wouldn’t run on the interpreter from later chapters and vice versa. So, for the book, I’ll leave it alone.
> 
> 一旦我们有了函数，我们就可以通过删除旧的print语句语法，并用本地函数替换它来简化语言，但是这意味着书中早期的例子不能在后面的解释器上运行，所以，为了本书的一致性，我就不这样做了。
> 
> If you’re building an interpreter for your own language, though, you may want to consider it.
> 
> 不过，如果你正在编写自己的解释器，可以考虑如此做。

Many languages also allow users to provide their own native functions. The mechanism for doing so is called a foreign function interface (FFI), native extension, native interface, or something along those lines. These are nice because they free the language implementer from providing access to every single capability the underlying platform supports. We won’t define an FFI for jlox, but we will add one native function to give you an idea of what it looks like.

很多语言还允许用户提供自己的本地函数，这样做的机制被称为外部函数接口（FFI）、本机扩展、本机接口或者类似的东西，这些很好，因为它们使得语言实现者，不必提供对底层平台支持的每一种功能的访问，我们不会为jlox 定义 FFI, 但是我们将添加一个本机函数，让你了解它的外观。

### 2.1 Telling time

关于时间

When we get to Part III and start working on a much more efficient implementation of Lox, we’re going to care deeply about performance. Performance work requires measurement, and that in turn means benchmarks. These are programs that measure the time it takes to exercise some corner of the interpreter.

当我们进入第三部分并且开始致力于更高效的实现Lox时候，我们将非常关注性能。性能需要指标衡量，这意味着基准测试，这些程序是用来测试解释器某个纬度所需要的时间。

We could measure the time it takes to start up the interpreter, run the benchmark, and exit, but that adds a lot of overhead—JVM startup time, OS shenanigans, etc. That stuff does matter, of course, but if you’re just trying to validate an optimization to some piece of the interpreter, you don’t want that overhead obscuring your results.

我们可以测试启动解释器、运行基准测试、退出解释器所需要的时间，但这会增加大量的JVM启动时间，操作系统等开销。当然，这些东西确实很重要，但如果我们只是试图验证解释器的某个部分的优化，我们不希望这些开销影响结果

A nicer solution is to have the benchmark script itself measure the time elapsed between two points in the code. To do that, a Lox program needs to be able to tell time. There’s no way to do that now—you can’t implement a useful clock “from scratch” without access to the underlying clock on the computer.

一个更好的解决方案是让基准测试脚本测量代码两点之间运行的时间，要做到这一点，Lox程序需要能够分辨时间，现在无法做到这一点，如果不访问计算机上的基础时钟，就无法从头开始实现有用的时钟。

So we’ll add clock(), a native function that returns the number of seconds that have passed since some fixed point in time. The difference between two successive invocations tells you how much time elapsed between the two calls. This function is defined in the global scope, so let’s ensure the interpreter has access to that.

因此，我们添加一个clock() 函数，这是一个返回自某个固定时间点以来所经过的秒数的本地函数，两次连续调用之间的差异，可以获取到程序运行时间，这个函数是在全局范围内定义的，所以让我们确保解释器可以访问它。

```java

// lox/Interpreter.java, in class Interpreter, replace 1 line


class Interpreter implements Expr.Visitor<Object>,
                             Stmt.Visitor<Void> {
  final Environment globals = new Environment();
  private Environment environment = globals;

  void interpret(List<Stmt> statements) {
  
```

The environment field in the interpreter changes as we enter and exit local scopes. It tracks the current environment. This new globals field holds a fixed reference to the outermost global environment.

解释器中的环境字段随着我们进入和退出本地作用域而改变，它跟踪当前的环境，这个新的全局字段对最外层的全局环境有一个固定的引用。


When we instantiate an Interpreter, we stuff the native function in that global scope.

当我们实例化解释器时候，我们将本地函数填充到全局范围中。


```java

// lox/Interpreter.java, in class Interpreter

  private Environment environment = globals;

  Interpreter() {
    globals.define("clock", new LoxCallable() {
      @Override
      public int arity() { return 0; }

      @Override
      public Object call(Interpreter interpreter,
                         List<Object> arguments) {
        return (double)System.currentTimeMillis() / 1000.0;
      }

      @Override
      public String toString() { return "<native fn>"; }
    });
  }

  void interpret(List<Stmt> statements) {
  
```

This defines a variable named “clock”. Its value is a Java anonymous class that implements LoxCallable. The clock() function takes no arguments, so its arity is zero. The implementation of call() calls the corresponding Java function and converts the result to a double value in seconds.

这定义了一个名为"clock"的变量，它的值是一个实现了LoxCallable接口的Java匿名类，clock()函数不带参数，因此它的arity 为零，call() 的实现调用相应的Java函数，并在结果转换为单位为秒的double 值

If we wanted to add other native functions—reading input from the user, working with files, etc.—we could add them each as their own anonymous class that implements LoxCallable. But for the book, this one is really all we need.

如果我们想要添加其他读取用户输入、处理文件等的本地函数，我们可以将它们各自添加为实现LoxCallable的匿名类，但是对于本书来说，clock() 函数是我们目前需要的本地函数

Let’s get ourselves out of the function-defining business and let our users take over . . . 

让我们脱离功能定义业务，让我们的用户接管

>In Lox, functions and variables occupy the same namespace. In Common Lisp, the two live in their own worlds. A function and variable with the same name don’t collide. If you call the name, it looks up the function. If you refer to it, it looks up the variable. This does require jumping through some hoops when you do want to refer to a function as a first-class value.
> 
> 在Lox中，函数和变量占据相同的命名空间，在commonLisp 中，函数和变量在不同的命名空间中，同名的函数和变量不会冲突，如果调用某个名称，它将查找函数，如果引用名称，它将查找变量。当你确实希望将函数当作一级对象时候，这确实需要跳过一些限制。
> 
> Richard P. Gabriel and Kent Pitman coined the terms “Lisp-1” to refer to languages like Scheme that put functions and variables in the same namespace, and “Lisp-2” for languages like Common Lisp that partition them. Despite being totally opaque, those names have since stuck. Lox is a Lisp-1.
> 
> [Richard P. Gabriel ](https://en.wikipedia.org/wiki/Richard_P._Gabriel) 和 [Kent Pitman](https://en.wikipedia.org/wiki/Kent_Pitman) 创造了术语lisp-1 , 表示Scheme等将函数和变量放在同一命名空间的语言，而lisp-2 则指代Common lisp 等对它们进行分区的语言，尽管这些名字是完全不透明的，但是从那时候开始，它们就一直存在着。Lox属于lisp-1语言。

## 三、Function Declarations

函数声明

We finally get to add a new production to the declaration rule we introduced back when we added variables. Function declarations, like variables, bind a new name. That means they are allowed only in places where a declaration is permitted.

我们终于可以在添加变量时候，引入的声明规则中添加一个新的产品，函数声明和变量声明一样，绑定一个新名称。这意味着只有在允许声明的地方才可以声明函数

> A named function declaration isn’t really a single primitive operation. It’s syntactic sugar for two distinct steps: (1) creating a new function object, and (2) binding a new variable to it. If Lox had syntax for anonymous functions, we wouldn’t need function declaration statements. You could just do:
> 
> 命名函数声明实际上不是单个基本操作。这是两个不同步骤的语法糖: (1) 创建一个新的函数对象；（2）绑定一个新变量，如果Lox有匿名函数的语法，我们就不需要函数声明语句，你可以这样做
>
> ```
> var add = fun (a, b) {
>   print a + b;
> };
> ```
> 
> However, since named functions are the common case, I went ahead and gave Lox nice syntax for them.
> 
> 然而，由于命名函数是常见的情况，我继续为它们提供Lox的对应语法。

However, since named functions are the common case, I went ahead and gave Lox nice syntax for them.

然而，由于命名函数是常见的情况，我们需要继续为它们提供Lox对应的语法

```java


declaration    → funDecl
               | varDecl
               | statement ;
			   
```

The updated declaration rule references this new rule:

更新的声明规则中，引用了新规则

```java

funDecl        → "fun" function ;
function       → IDENTIFIER "(" parameters? ")" block ;


```

> Methods are too classy to have fun.


The main funDecl rule uses a separate helper rule function. A function declaration statement is the fun keyword followed by the actual function-y stuff. When we get to classes, we’ll reuse that function rule for declaring methods. Those look similar to function declarations, but aren’t preceded by fun.

funDecl 规则使用一个单独的helper规则函数，函数声明语句以fun 关键字开始，后面紧随着实际的内容。当我们到达类时候，我们将重用该函数规则来声明方法，这些看起来类似于函数声明，但是前面没有有趣的地方

The function itself is a name followed by the parenthesized parameter list and the body. The body is always a braced block, using the same grammar rule that block statements use. The parameter list uses this rule:

函数本身是一个名称，后面跟随带括号的参数列表和函数正文，主体始终是一个代码块，使用块语句相同的语法规则。参数列表使用下面的规则

```java

parameters     → IDENTIFIER ( "," IDENTIFIER )* ;

```

It’s like the earlier arguments rule, except that each parameter is an identifier, not an expression. That’s a lot of new syntax for the parser to chew through, but the resulting AST node isn’t too bad.

这与前面的arguments 规则类似，只是每个参数都是标识符，而不是表达式，这是解析器需要仔细研究的新语法，但是生成的AST节点挺好的


```java

// tool/GenerateAst.java, in main()

      "Expression : Expr expression",
      "Function   : Token name, List<Token> params," +
                  " List<Stmt> body",
      "If         : Expr condition, Stmt thenBranch," +
	  
```

A function node has a name, a list of parameters (their names), and then the body. We store the body as the list of statements contained inside the curly braces.

函数节点有名称、参数列表（参数名称）、主体，我们将主体存储为包含在大括号内的语句列表

Over in the parser, we weave in the new declaration.

在解析器中，我们解析新的函数声明语句

```java

// lox/Parser.java, in declaration()

    try {
      if (match(FUN)) return function("function");
      if (match(VAR)) return varDeclaration();
	  
```

Like other statements, a function is recognized by the leading keyword. When we encounter fun, we call function. That corresponds to the function grammar rule since we already matched and consumed the fun keyword. We’ll build the method up a piece at a time, starting with this:

和其他语句一样，函数的开始关键字是 fun, 当我们遇到fun, 我们调用function()，这对应着函数语法规则，因为我们已经匹配并且使用了fun 关键字，我们将一次构建方法，从下面的函数开始

```java

// lox/Parser.java, add after expressionStatement()

  private Stmt.Function function(String kind) {
    Token name = consume(IDENTIFIER, "Expect " + kind + " name.");
  }

```

Right now, it only consumes the identifier token for the function’s name. You might be wondering about that funny little kind parameter. Just like we reuse the grammar rule, we’ll reuse the function() method later to parse methods inside classes. When we do that, we’ll pass in “method” for kind so that the error messages are specific to the kind of declaration being parsed.

现在，它只使用函数名称的标识符token，你可能想知道这个有趣的小参数，就像我们重用语法规则一样，我们稍后将重用function() 方法来解析类内的方法。当我们这样做时候，我们将传递method 来表示种类，以便错误消息可以添加要解析的声明函数信息。

Next, we parse the parameter list and the pair of parentheses wrapped around it.

接下来，我们解析参数列表以及参数列表的一对括号

```java

// lox/Parser.java, in function()

    Token name = consume(IDENTIFIER, "Expect " + kind + " name.");
    consume(LEFT_PAREN, "Expect '(' after " + kind + " name.");
    List<Token> parameters = new ArrayList<>();
    if (!check(RIGHT_PAREN)) {
      do {
        if (parameters.size() >= 255) {
          error(peek(), "Can't have more than 255 parameters.");
        }

        parameters.add(
            consume(IDENTIFIER, "Expect parameter name."));
      } while (match(COMMA));
    }
    consume(RIGHT_PAREN, "Expect ')' after parameters.");
  }

```

This is like the code for handling arguments in a call, except not split out into a helper method. The outer if statement handles the zero parameter case, and the inner while loop parses parameters as long as we find commas to separate them. The result is the list of tokens for each parameter’s name.

这类似于处理调用中的参数的代码，只是没有拆分成一个helper方法，外部if语句处理零参数情况，内部的while 循环解析参数，只要我们找到逗号来分隔它们。结果是每个参数名称的token列表

Just like we do with arguments to function calls, we validate at parse time that you don’t exceed the maximum number of parameters a function is allowed to have.

就像我们处理函数调用的参数一样，我们在解析时候验证函数的参数不会超过最大参数限制。

Finally, we parse the body and wrap it all up in a function node.

最后，我们解析函数主体，并将解析结果包装在一个函数节点中

```java

// lox/Parser.java, in function()

    consume(RIGHT_PAREN, "Expect ')' after parameters.");

    consume(LEFT_BRACE, "Expect '{' before " + kind + " body.");
    List<Stmt> body = block();
    return new Stmt.Function(name, parameters, body);
  }
  
```

Note that we consume the { at the beginning of the body here before calling block(). That’s because block() assumes the brace token has already been matched. Consuming it here lets us report a more precise error message if the { isn’t found since we know it’s in the context of a function declaration.

请注意，在调用block() 之前，我们在这里将先消耗开始的{ , 这是因为block() 假设大括号token，已经被匹配。如果我们知道在函数声明的上下文中没有找到 {, 那么在这里使用函数名称，报告更加准确的错误信息。

## 四、Function Objects

函数对象

We’ve got some syntax parsed so usually we’re ready to interpret, but first we need to think about how to represent a Lox function in Java. We need to keep track of the parameters so that we can bind them to argument values when the function is called. And, of course, we need to keep the code for the body of the function so that we can execute it.

我们已经解析了一些语法，所以通常我们也已经准备好解释了，但是首先，我们需要考虑如何在Java中表示Lox函数，我们需要跟踪函数的参数，以便在调用函数时候，将它们绑定到具体的参数值，当然，我们还需要保留函数主体代码，以便执行函数。

That’s basically what the Stmt.Function class is. Could we just use that? Almost, but not quite. We also need a class that implements LoxCallable so that we can call it. We don’t want the runtime phase of the interpreter to bleed into the front end’s syntax classes so we don’t want Stmt.Function itself to implement that. Instead, we wrap it in a new class.

这就是Stmt.Function 类的基本含义，我们可以使用它吗？差不多，但是不完全是。我们还需要一个实现了LoxCallable的类，以便我们可以调用函数，我们不希望解释器的运行阶段渗入到前端的语法类，因此我们不希望使用Stmt.Function 本身实现这一点。相反，我们将其包装在一个新的类中


```java

// lox/LoxFunction.java, create new file


package com.craftinginterpreters.lox;

import java.util.List;

class LoxFunction implements LoxCallable {
  private final Stmt.Function declaration;
  LoxFunction(Stmt.Function declaration) {
    this.declaration = declaration;
  }
}

```

We implement the call() of LoxCallable like so:

我们实现的LoxCallable 的call() ，如下所示

```java

// lox/LoxFunction.java, add after LoxFunction()

  @Override
  public Object call(Interpreter interpreter,
                     List<Object> arguments) {
    Environment environment = new Environment(interpreter.globals);
    for (int i = 0; i < declaration.params.size(); i++) {
      environment.define(declaration.params.get(i).lexeme,
          arguments.get(i));
    }

    interpreter.executeBlock(declaration.body, environment);
    return null;
  }
  
```

> We’ll dig even deeper into environments in the next chapter.
> 
> 我们将在下一章，深入探讨环境

This handful of lines of code is one of the most fundamental, powerful pieces of our interpreter. As we saw in the chapter on statements and state, managing name environments is a core part of a language implementation. Functions are deeply tied to that.

这几行代码是我们解释器中的最基本、最强大的部分之一，正如我们在关于语句和状态的章节中所看到的，管理名称环境是语言实现的核心部分，函数和环境密切相关。

Parameters are core to functions, especially the fact that a function encapsulates its parameters—no other code outside of the function can see them. This means each function gets its own environment where it stores those variables.

参数是函数的核心，尤其是一个函数封装了它的参数，函数之外的其他代码都无法查看这些参数，这意味着每个函数都有自己的环境来存储这些变量。

Further, this environment must be created dynamically. Each function call gets its own environment. Otherwise, recursion would break. If there are multiple calls to the same function in play at the same time, each needs its own environment, even though they are all calls to the same function.

此外，这个环境必须是动态创建的，每个函数调用需要获取独立的环境，否则，递归将被中断。如果同时有多个对同一函数的调用，则每个调用都需要自己的环境，即使它们都是对同一个函数的调用。

For example, here’s a convoluted way to count to three:

例如，这里有一个复杂的数到3的方法

```java

fun count(n) {
  if (n > 1) count(n - 1);
  print n;
}

count(3);

```


Imagine we pause the interpreter right at the point where it’s about to print 1 in the innermost nested call. The outer calls to print 2 and 3 haven’t printed their values yet, so there must be environments somewhere in memory that still store the fact that n is bound to 3 in one context, 2 in another, and 1 in the innermost, like:

假设我们在解释器，即将在最内部的嵌套调用中打印1时候暂停它。打印2 和3 的外部调用，还没有打印它们的值。所以，内存中一定有一些环境仍然存储着，n的一个上下文绑定到3， 在另一个上下文中将绑定到2， 在最内部将绑定到1，例如：

![recursion](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/recursion.png?raw=true)

That’s why we create a new environment at each call, not at the function declaration. The call() method we saw earlier does that. At the beginning of the call, it creates a new environment. Then it walks the parameter and argument lists in lockstep. For each pair, it creates a new variable with the parameter’s name and binds it to the argument’s value.

这就是为什么我们在每次调用时候，都需要创建一个新的环境。而不是在函数声明时候。我们前面看到的call() 方法就是这样做的。在call()开始，它会创建一个新的环境。然后，它会同步遍历参数和参数列表。对于每一对，它都使用参数的名称创建一个新变量，并将其绑定到参数的值。

So, for a program like this:

因此，对于这样的程序


```java

fun add(a, b, c) {
  print a + b + c;
}

add(1, 2, 3);

```

At the point of the call to add(), the interpreter creates something like this:

在调用add() 时候，解释器将创建如下内容

![binding](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/binding.png?raw=true)

Then call() tells the interpreter to execute the body of the function in this new function-local environment. Up until now, the current environment was the environment where the function was being called. Now, we teleport from there inside the new parameter space we’ve created for the function.

然后，call()告诉解释器在这个新的函数局部环境中，执行函数体。到目前为止，当前环境是调用函数的环境。现在，我们从那里传送到我们为函数创建的新的参数作用域

This is all that’s required to pass data into the function. By using different environments when we execute the body, calls to the same function with the same code can produce different results.

这就是将数据传递到函数所需要的全部内容，通过在执行主体时候，使用不同的环境，用相同的代码调用相同的函数，可以产生不同的结果。

Once the body of the function has finished executing, executeBlock() discards that function-local environment and restores the previous one that was active back at the callsite. Finally, call() returns null, which returns nil to the caller. (We’ll add return values later.)

一旦函数的主体完成执行，executeBlock() 将丢弃该函数的本地环境，并恢复在调用站点上激活的前一个环境，最后，call() 返回null,, 这将向调用者返回nil( 稍后我们将添加返回值）

Mechanically, the code is pretty simple. Walk a couple of lists. Bind some new variables. Call a method. But this is where the crystalline code of the function declaration becomes a living, breathing invocation. This is one of my favorite snippets in this entire book. Feel free to take a moment to meditate on it if you’re so inclined.

从机械上来说，代码非常简单，列出几个列表，绑定一些新变量，调用一个方法，但是这正是从函数声明，到函数调用运行的过程。这是整本书中我最喜欢的代码片段。如果你有兴趣的话，不妨花点时间沉思一下。

Done? OK. Note when we bind the parameters, we assume the parameter and argument lists have the same length. This is safe because visitCallExpr() checks the arity before calling call(). It relies on the function reporting its arity to do that.

已经完成了吗？注意，当我们绑定参数时候，我们假设参数和参数列表具有相同的长度，只是安全的，因为visitCallExpr() 在调用call() 之前检查arity，它依赖于arity() 来做到这一点。

```java

// lox/LoxFunction.java, add after LoxFunction()

  @Override
  public int arity() {
    return declaration.params.size();
  }


```

That’s most of our object representation. While we’re in here, we may as well implement toString().

这是我们的大多数对象表示，在这里，我们不妨实现toString() 

```java

// lox/LoxFunction.java, add after LoxFunction()

  @Override
  public String toString() {
    return "<fn " + declaration.name.lexeme + ">";
  }
  
```

This gives nicer output if a user decides to print a function value.

如果用户决定打印函数值，这将提供更好的输出

```java

fun add(a, b) {
  print a + b;
}

print add; // "<fn add>".

```

### 4.1 Interpreting function declarations

解释函数声明

We’ll come back and refine LoxFunction soon, but that’s enough to get started. Now we can visit a function declaration.

我们很快会回来完善 LoxFunction, 但是这已经足够开始了。现在我们可以访问函数声明

```java

// lox/Interpreter.java, add after visitExpressionStmt()

  @Override
  public Void visitFunctionStmt(Stmt.Function stmt) {
    LoxFunction function = new LoxFunction(stmt);
    environment.define(stmt.name.lexeme, function);
    return null;
  }


```

This is similar to how we interpret other literal expressions. We take a function syntax node—a compile-time representation of the function—and convert it to its runtime representation. Here, that’s a LoxFunction that wraps the syntax node.

这与我们解释其他文字表达式的方法类似，我们获取函数语法节点（函数编译时候的表示），并且将其转换为其运行时候的表示，这里，这是一个包装语法节点的LoxFunction

Function declarations are different from other literal nodes in that the declaration also binds the resulting object to a new variable. So, after creating the LoxFunction, we create a new binding in the current environment and store a reference to it there.

函数声明与其他文本节点的不同之处在于，声明还将生成的对象绑定到新变量，因此，在创建LoxFunction之后，我们在当前环境中创建一个新的绑定，并在那里存储对它的引用。

With that, we can define and call our own functions all within Lox. Give it a try:

这样，我们就可以在Lox定义和调用自己的函数，试试看

```java

fun sayHi(first, last) {
  print "Hi, " + first + " " + last + "!";
}

sayHi("Dear", "Reader");

```

I don’t know about you, but that looks like an honest-to-God programming language to me.

我不知道你如何觉得，对于我，我认为这已经是一门真正的语言了。


## 五、Return Statements

返回语句

We can get data into functions by passing parameters, but we’ve got no way to get results back out. If Lox were an expression-oriented language like Ruby or Scheme, the body would be an expression whose value is implicitly the function’s result. But in Lox, the body of a function is a list of statements which don’t produce values, so we need dedicated syntax for emitting a result. In other words, return statements. I’m sure you can guess the grammar already.

我们可以通过传递参数将数据传递到函数中，但是我们无法将函数结果返回，如果Lox是像Ruby或者 Scheme这样的面向表达式的语言，那么主体将是一个表达式，其值隐式地表示函数的结果，但是在Lox中，函数的主体是一系列不产生值的语句，因此，我们需要专门的语法来发出结果。换句话来说，返回语句，我相信你已经猜测到语法了。

```java

statement      → exprStmt
               | forStmt
               | ifStmt
               | printStmt
               | returnStmt
               | whileStmt
               | block ;

returnStmt     → "return" expression? ";" ;

```

> The Hotel California of data.

We’ve got one more—the final, in fact—production under the venerable statement rule. A return statement is the return keyword followed by an optional expression and terminated with a semicolon.

我们还有一个最终的，事实上是根据古老的声明语法制作的。return 语句是一个以return 关键字开始，跟随着可选的表达式，最终以分号结尾的语句。

The return value is optional to support exiting early from a function that doesn’t return a useful value. In statically typed languages, “void” functions don’t return a value and non-void ones do. Since Lox is dynamically typed, there are no true void functions. The compiler has no way of preventing you from taking the result value of a call to a function that doesn’t contain a return statement.

如果函数不需要返回有用值，return 语句是可选的，return语句可以提前退出函数。在静态类型语言中，void 函数不返回值，而非void 的函数，需要返回一个值。由于Lox是动态类型语言，因此没有真正的void函数，编译器无法阻止您获取对不包含return 语句的函数的调用的结果值

```java

fun procedure() {
  print "don't return anything";
}

var result = procedure();
print result; // ?

```

This means every Lox function must return something, even if it contains no return statements at all. We use nil for this, which is why LoxFunction’s implementation of call() returns null at the end. In that same vein, if you omit the value in a return statement, we simply treat it as equivalent to:

这意味着每个Lox函数都必须返回一些东西，即使它根本不包含返回语句。我们对此使用nil, 这就是为什么LoxFunction的call() 实现在结尾返回null，同样，如果忽略return语句中的值，我们只需要将其等效为 

```java

return nil;

```

Over in our AST generator, we add a new node.

在AST生成器中，我们添加一个新的节点

```java

// tool/GenerateAst.java, in main()

      "Print      : Expr expression",
      "Return     : Token keyword, Expr value",
      "Var        : Token name, Expr initializer",

```

It keeps the return keyword token so we can use its location for error reporting, and the value being returned, if any. We parse it like other statements, first by recognizing the initial keyword.

它保留return关键字token, 以便我们可以使用它的位置进行错误报告，以及返回值（如果有返回值），我们像其他语句一样解析它，首先识别关键字 return

```java

// lox/Parser.java, in statement()

    if (match(PRINT)) return printStatement();
    if (match(RETURN)) return returnStatement();
    if (match(WHILE)) return whileStatement();
	
```

That branches out to:

```java

// lox/Parser.java, add after printStatement()

  private Stmt returnStatement() {
    Token keyword = previous();
    Expr value = null;
    if (!check(SEMICOLON)) {
      value = expression();
    }

    consume(SEMICOLON, "Expect ';' after return value.");
    return new Stmt.Return(keyword, value);
  }
  
```

After snagging the previously consumed return keyword, we look for a value expression. Since many different tokens can potentially start an expression, it’s hard to tell if a return value is present. Instead, we check if it’s absent. Since a semicolon can’t begin an expression, if the next token is that, we know there must not be a value.

在获取之前使用的return 关键字之后，我们寻找一个值表达式，由于许多不同的token可能会启动一个表达式，因此很难判断是否存在返回值。相反，我们检查它是否缺失，由于分号不能开始表达式，因此如果下一个token是分号，我们知道肯定不存在表达式

### 5.1 Returning from calls

调用函数返回

Interpreting a return statement is tricky. You can return from anywhere within the body of a function, even deeply nested inside other statements. When the return is executed, the interpreter needs to jump all the way out of whatever context it’s currently in and cause the function call to complete, like some kind of jacked up control flow construct.

解释返回语句很棘手，你可以从函数体的任何位置返回，甚至可以在深度嵌套的语句中添加返回语句。当执行返回时候，解释器需要一直跳转到它当前所在的任何上下文之外，并使函数调用完成。就像某种自升式控制流构造一样。

For example, say we’re running this program and we’re about to execute the return statement:

例如，假设我们正在运行这个程序，我们将要执行return语句

```java

fun count(n) {
  while (n < 100) {
    if (n == 3) return n; // <--
    print n;
    n = n + 1;
  }
}

count(1);

```

The Java call stack currently looks roughly like this:

Java的调用堆栈大概看起来，像是

```java

Interpreter.visitReturnStmt()
Interpreter.visitIfStmt()
Interpreter.executeBlock()
Interpreter.visitBlockStmt()
Interpreter.visitWhileStmt()
Interpreter.executeBlock()
LoxFunction.call()
Interpreter.visitCallExpr()

```


We need to get from the top of the stack all the way back to call(). I don’t know about you, but to me that sounds like exceptions. When we execute a return statement, we’ll use an exception to unwind the interpreter past the visit methods of all of the containing statements back to the code that began executing the body.

我们需要从堆栈顶部一直返回到call(), 我不知道你的情况，但对我来说，这看起来像是错误处理，当我们执行return语句时候，我们将使用一个异常来将解释器从所有包含语句的访问方法，返回到开始执行主体的代码。

The visit method for our new AST node looks like this:

新的AST节点的访问方法如下

```java

// lox/Interpreter.java, add after visitPrintStmt()

  @Override
  public Void visitReturnStmt(Stmt.Return stmt) {
    Object value = null;
    if (stmt.value != null) value = evaluate(stmt.value);

    throw new Return(value);
  }


```

If we have a return value, we evaluate it, otherwise, we use nil. Then we take that value and wrap it in a custom exception class and throw it.

如果我们有一个返回值，我们就对它求值，否则，我们使用nil, 然后，我们获取该值，并将其包装在自定义的异常类中，然后抛出异常

```java

// lox/Return.java, create new file


package com.craftinginterpreters.lox;

class Return extends RuntimeException {
  final Object value;

  Return(Object value) {
    super(null, null, false, false);
    this.value = value;
  }
}

```

This class wraps the return value with the accoutrements Java requires for a runtime exception class. The weird super constructor call with those null and false arguments disables some JVM machinery that we don’t need. Since we’re using our exception class for control flow and not actual error handling, we don’t need overhead like stack traces.

此类使用Java 为运行时异常类所需的配置，包装返回值。带有这些null 和false 参数的奇怪的父构造函数调用，禁用了我们不需要的JVM机制。由于我们将异常类用于控制流，而不是实际的错误处理，所以我们不需要像堆栈跟踪那样的开销。

> For the record, I’m not generally a fan of using exceptions for control flow. But inside a heavily recursive tree-walk interpreter, it’s the way to go. Since our own syntax tree evaluation is so heavily tied to the Java call stack, we’re pressed to do some heavyweight call stack manipulation occasionally, and exceptions are a handy tool for that.
> 
> 事实上，我不喜欢使用异常作为控制流，但是在一个递归性很强的语法树遍历解释器中，这是一个很好的方式，由于我们自己的语法树运行和Java调用堆栈紧密相连，我们不得不偶尔进行一些重量级的调用堆栈操作，而异常是一个方便的工具

We want this to unwind all the way to where the function call began, the call() method in LoxFunction

我们希望它一直展开到函数调用开始的位置，即LoxFunction中的call() 方法

```java

// lox/LoxFunction.java, in call(), replace 1 line

          arguments.get(i));
    }

    try {
      interpreter.executeBlock(declaration.body, environment);
    } catch (Return returnValue) {
      return returnValue.value;
    }
    return null;
	
```

We wrap the call to executeBlock() in a try-catch block. When it catches a return exception, it pulls out the value and makes that the return value from call(). If it never catches one of these exceptions, it means the function reached the end of its body without hitting a return statement. In that case, it implicitly returns nil.

我们把executeBlock() 的调用包装在try-catch 块中，当它捕捉到一个返回异常时候，它会获取异常的值，并使其成为 call() 的返回值，如果它从未捕捉到这些异常中的任何一个，则意味着函数到达了其主体的末尾，而没有命中返回语句。在这种情况下，它隐式返回nil

Let’s try it out. We finally have enough power to support this classic example—a recursive function to calculate Fibonacci numbers:

让我们试试看，我们终于有足够的能力支持经典的例子——一个计算斐波那契的递归函数

```java

fun fib(n) {
  if (n <= 1) return n;
  return fib(n - 2) + fib(n - 1);
}

for (var i = 0; i < 20; i = i + 1) {
  print fib(i);
}

```

This tiny program exercises almost every language feature we have spent the past several chapters implementing—expressions, arithmetic, branching, looping, variables, functions, function calls, parameter binding, and returns.

这个经典例子，练习了我们在过去几章中的几乎所有语言功能，实现了表达式、算术、分支、循环、变量、函数、函数调用、参数绑定和返回

![run-fibonacci](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/run_fibonacci.png?raw=true)

> You might notice this is pretty slow. Obviously, recursion isn’t the most efficient way to calculate Fibonacci numbers, but as a microbenchmark, it does a good job of stress testing how fast our interpreter implements function calls.
> 
> 你可能会注意到这很慢，显然，递归不是计算斐波那契数列的最有效的方法，但是作为一个基准脚本，它在压力测试我们的解释器实现的函数调用的速度方面，是个好的方法。
> 
> As you can see, the answer is “not very fast”. That’s OK. Our C interpreter will be faster.
> 
> 正如你所看到的，答案是不快，先这样，我们的C解释器将会更快。

## 六、Local Functions and Closures

局部函数和闭包

Our functions are pretty full featured, but there is one hole to patch. In fact, it’s a big enough gap that we’ll spend most of the next chapter sealing it up, but we can get started here.

我们的功能非常齐全，但是有一个漏洞需要修补。事实上，这是一个足够大的差距，我们将在下一章的大部分时间里把它封闭起来，但是我们可以先从这里开始

LoxFunction’s implementation of call() creates a new environment where it binds the function’s parameters. When I showed you that code, I glossed over one important point: What is the parent of that environment?

LoxFunction对象的call() 实现了创建一个新的环境，在这里它绑定了函数的参数，当我向您展示代码时候，我忽略了一个重要的问题，环境的父级是什么？

Right now, it is always globals, the top-level global environment. That way, if an identifier isn’t defined inside the function body itself, the interpreter can look outside the function in the global scope to find it. In the Fibonacci example, that’s how the interpreter is able to look up the recursive call to fib inside the function’s own body—fib is a global variable.

现在，它始终是全局的，顶层的全局环境。这样，如果没有在函数体内部定义标识符，则解释器可以在全局范围内查找函数外部的标识符，在斐波那契示例中，这就是解释器如何在函数自身内部查找对 fib 的递归调用 fib 的方式，fib是一个全局变量

But recall that in Lox, function declarations are allowed anywhere a name can be bound. That includes the top level of a Lox script, but also the inside of blocks or other functions. Lox supports local functions that are defined inside another function, or nested inside a block.

但是请记住，在Lox中，函数声明在任何可以绑定名称的地方都是允许的，这包括Lox脚本的顶层，也包括块或者其他函数的内部，Lox支持在另外一个函数内定义或者嵌套在块内的局部函数

Consider this classic example:

查看下面这个经典示例


```java

fun makeCounter() {
  var i = 0;
  fun count() {
    i = i + 1;
    print i;
  }

  return count;
}

var counter = makeCounter();
counter(); // "1".
counter(); // "2".

```

Here, count() uses i, which is declared outside of itself in the containing function makeCounter(). makeCounter() returns a reference to the count() function and then its own body finishes executing completely.

在这里，count()用到i， 变量i声明在makeCounter() 外层函数中，makeCounter() 返回对count() 函数的引用，然后它自己的主体完全执行完毕

Meanwhile, the top-level code invokes the returned count() function. That executes the body of count(), which assigns to and reads i, even though the function where i was defined has already exited.

同时，顶层代码调用返回的count() 函数，执行count() 函数的主体，该主体赋值并且读取i，即使定义i的函数已经退出

If you’ve never encountered a language with nested functions before, this might seem crazy, but users do expect it to work. Alas, if you run it now, you get an undefined variable error in the call to counter() when the body of count() tries to look up i. That’s because the environment chain in effect looks like this:

如果你之前从来没有遇到过具有嵌套函数的语言，这可能看起来非常疯狂，但是用户的确希望，它能工作。哎，如果你现在运行它，当counter()的主体代码试图查找i时候，你会在counter() 运行时得到一个未定义变量的报错。这是因为环境链实际上是这样的

![global](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/global.png?raw=true)

When we call count() (through the reference to it stored in counter), we create a new empty environment for the function body. The parent of that is the global environment. We lost the environment for makeCounter() where i is bound.

当我们调用count()（通过存储在counter() 中的引用）时候，我们为函数体创建了一个新的空的环境，它的父级环境就是全局环境，我们失去了makeCounter() 中的环境


Let’s go back in time a bit. Here’s what the environment chain looked like right when we declared count() inside the body of makeCounter():

让我们回到过去一点，下面是我们在makeCounter() 的主体内声明count() 时候，环境链的样子


![global](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/body.png?raw=true)

So at the point where the function is declared, we can see i. But when we return from makeCounter() and exit its body, the interpreter discards that environment. Since the interpreter doesn’t keep the environment surrounding count() around, it’s up to the function object itself to hang on to it.

因此，在声明函数时候，我们可以看到i，但是当我们从makeCounter()返回并且退出其主体时候，解释器会丢弃该环境。由于解释器不保持count() 周围的环境，所以由函数对象自身来保持这个环境

This data structure is called a closure because it “closes over” and holds on to the surrounding variables where the function is declared. Closures have been around since the early Lisp days, and language hackers have come up with all manner of ways to implement them. For jlox, we’ll do the simplest thing that works. In LoxFunction, we add a field to store an environment.

这个数据结构被称为闭包，因为它“关闭”并保持在声明函数的周围变量上。自从Lisp早期以来，闭包就一直存在，语言高手想出来各种方法来实现它们。对于jlox, 我们将做最简单的工作。在LoxFunction中，我们添加一个新字段来存储周围环境


```java

// lox/LoxFunction.java, in class LoxFunction

  private final Stmt.Function declaration;
  private final Environment closure;

  LoxFunction(Stmt.Function declaration) {
  
```

> “Closure” is yet another term coined by [Peter J. Landin](https://en.wikipedia.org/wiki/Peter_Landin). I assume before he came along that computer scientists communicated with each other using only primitive grunts and pawing hand gestures.
> 
> 闭包是Peter J. Landin创造的另一个术语，我猜想，在他出现之前，计算机科学家之间的交流只使用原始方式。

We initialize that in the constructor.

我们在初始化时候，构造它

```java

// lox/LoxFunction.java, constructor LoxFunction(), replace 1 line

  LoxFunction(Stmt.Function declaration, Environment closure) {
    this.closure = closure;
    this.declaration = declaration;
	
```

When we create a LoxFunction, we capture the current environment.

当我们创建一个LoxFunction对象时候，我们将捕获当前的环境

```java

// lox/Interpreter.java, in visitFunctionStmt(), replace 1 line

  public Void visitFunctionStmt(Stmt.Function stmt) {
    LoxFunction function = new LoxFunction(stmt, environment);
    environment.define(stmt.name.lexeme, function);
	
```

This is the environment that is active when the function is declared not when it’s called, which is what we want. It represents the lexical scope surrounding the function declaration. Finally, when we call the function, we use that environment as the call’s parent instead of going straight to globals.

这是在声明函数时候激活的环境，而不是在调用函数时候激活。这是我们想要的，它表示函数声明周围的词法范围。最后，当我们调用函数时候，我们使用该环境作为调用的父级环境，而不是直接使用全局环境

```java

// lox/LoxFunction.java, in call(), replace 1 line

                     List<Object> arguments) {
    Environment environment = new Environment(closure);
    for (int i = 0; i < declaration.params.size(); i++) {
	
```

This creates an environment chain that goes from the function’s body out through the environments where the function is declared, all the way out to the global scope. The runtime environment chain matches the textual nesting of the source code like we want. The end result when we call that function looks like this:

这将创建一个环境链，从函数的主体到声明函数的环境，一直到全局范围，运行时，环境链与源代码的文本嵌套相匹配。调用该函数时候的最终结果如下

![global](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/closure.png?raw=true)

Now, as you can see, the interpreter can still find i when it needs to because it’s in the middle of the environment chain. Try running that makeCounter() example now. It works!

现在，如你所见，解释器仍然可以在需要时候，找到变量i，因为它位于环境链的中间，现在尝试运行makeCounter() 示例，它可以正常运行。

Functions let us abstract over, reuse, and compose code. Lox is much more powerful than the rudimentary arithmetic calculator it used to be. Alas, in our rush to cram closures in, we have let a tiny bit of dynamic scoping leak into the interpreter. In the next chapter, we will explore deeper into lexical scope and close that hole.

函数是我们能够抽象、重用和编写代码，Lox比它之前的基本算术计算器功能，强大多了。哎，在我们匆忙将闭包塞入的过程中，我们添加了一些动态范围泄露到解释器中，在下一章，我们将更加深入的探讨词汇作用域，并且填补现在的漏洞

## 七、CHALLENGES

习题

1. Our interpreter carefully checks that the number of arguments passed to a function matches the number of parameters it expects. Since this check is done at runtime on every call, it has a performance cost. Smalltalk implementations don’t have that problem. Why not?

1. Lox’s function declaration syntax performs two independent operations. It creates a function and also binds it to a name. This improves usability for the common case where you do want to associate a name with the function. But in functional-styled code, you often want to create a function to immediately pass it to some other function or return it. In that case, it doesn’t need a name.

	Languages that encourage a functional style usually support anonymous functions or lambdas—an expression syntax that creates a function without binding it to a name. Add anonymous function syntax to Lox so that this works:

	```java

	fun thrice(fn) {
	  for (var i = 1; i <= 3; i = i + 1) {
		fn(i);
	  }
	}

	thrice(fun (a) {
	  print a;
	});
	// "1".
	// "2".
	// "3".

	```

	How do you handle the tricky case of an anonymous function expression occurring in an expression statement:
	
	```java
	
	fun () {};
	
	```


1. Is this program valid?

	```java

	fun scope(a) {
	  var a = "local";
	}
	```

	In other words, are a function’s parameters in the same scope as its local variables, or in an outer scope? What does Lox do? What about other languages you are familiar with? What do you think a language should do?

&nbsp;


1. 我们的解释器仔细检查传递给函数的参数数量，是否与它期望的参数数量相匹配，由于每次调用都在运行时候执行此检查，因此会产生性能成本。Smalltalk 的实现没有这个问题，为什么呢？


1. Lox函数声明语法，执行两个独立的操作。它创建一个函数，并且将其绑定到一个名称，这提高了在我们希望将名称和函数关联的常见情况下的可用性。但是函数式代码中，你通常希望创建一个函数，以便立即将其传递给其他函数或者返回它。在这种情况下，它不需要名称

	鼓励函数式的语言，通常支持匿名函数或者lambda表达式语法，该语法创建函数，而不将其绑定到名称，将匿名函数语法添加到Lox中
	
	如何处理表达式语句中，出现的匿名函数表达式这种棘手情况
	
1. 下面的代码是合法的吗？

	换句话说，函数的参数是在与其局部变量相同的范围内，还是在外部范围内？Lox是如何做的呢，你熟悉的其他语言呢？你认为语言应该怎么做呢？
	
	