# 计算表达式

> You are my creator, but I am your master; Obey!
> 
> 你是我的创造者，但我是你的主人，服从吧！
> 
>  <p align="right">—— Mary Shelley, Frankenstein 玛丽.雪莱的《科学怪人》 </p>

If you want to properly set the mood for this chapter, try to conjure up a thunderstorm, one of those swirling tempests that likes to yank open shutters at the climax of the story. Maybe toss in a few bolts of lightning. In this chapter, our interpreter will take breath, open its eyes, and execute some code.

如果你想要为这一章找到一个合适的设定氛围，试着想象一场雷雨，一场喜欢在故事高潮时候，吹开百叶窗的旋转风暴，也许还会扔出几个闪电，哈哈。在本章中，我们的解释器将屏住呼吸，睁开眼睛，执行一些代码。

![lightning](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/lightning.png?raw=true)

> A decrepit Victorian mansion is optional, but adds to the ambiance.
> 
> 一座破旧的维多利亚式豪宅是可选的，但是增加了氛围。

There are all manner of ways that language implementations make a computer do what the user’s source code commands. They can compile it to machine code, translate it to another high-level language, or reduce it to some bytecode format for a virtual machine to run. For our first interpreter, though, we are going to take the simplest, shortest path and execute the syntax tree itself.

让计算机执行用户源代码的，语言实现方式多种多样。我们可以将其编译为机器码，将其翻译为另外一门高级语言，或者将其简化为某种字节码格式，以供虚拟机运行，然而，对于我们的第一个解释器，我们将采用最简单，最短的路径，执行语法树本身。

Right now, our parser only supports expressions. So, to “execute” code, we will evaluate an expression and produce a value. For each kind of expression syntax we can parse—literal, operator, etc.—we need a corresponding chunk of code that knows how to evaluate that tree and produce a result. That raises two questions:

1. What kinds of values do we produce?

1. How do we organize those chunks of code?

Taking them on one at a time . . . 

当前，我们的解析器只支持解析表达式，所以，执行代码，表示我们将计算表达式，并且生成值。对于每一种表达式语法，我们都可以解析文字、运算符等等，我们需要一段相应的代码，知道如何计算语法树，并且返回计算结果。这引发了两个问题：

1. 我们将计算出什么值

1. 我们如何组织代码

让我们一个个解决问题。

## 一、Representing Values

表示计算值

In Lox, values are created by literals, computed by expressions, and stored in variables. The user sees these as Lox objects, but they are implemented in the underlying language our interpreter is written in. That means bridging the lands of Lox’s dynamic typing and Java’s static types. A variable in Lox can store a value of any (Lox) type, and can even store values of different types at different points in time. What Java type might we use to represent that?

在Lox中，值由文字创建，表达式计算，保存在变量中。用户将计算值当作Lox中的对象，但是它们是由解释器底层编写语言实现的。这意味着，我们在Lox动态语言和Java静态语言之间，搭建了一座桥梁。Lox中的变量可以保存任何Lox类型的值，甚至可以在不同的时间点存储不同的类型的数据，我们可以用Java中的什么类型表示呢？

> Here, I’m using “value” and “object” pretty much interchangeably.
> 
> Later in the C interpreter we’ll make a slight distinction between them, but that’s mostly to have unique terms for two different corners of the implementation—in-place versus heap-allocated data. From the user’s perspective, the terms are synonymous.
> 
> 在这里，我们可以把 value 和 object当作相同的东西
> 
> 稍后，在C解释器中，我们将对它们进行细微的区分，但是，这只是为了使得两种不同的实现方式（栈分配、堆分配），有不同的唯一术语，从用户的角度，它们是同一含义。

Given a Java variable with that static type, we must also be able to determine which kind of value it holds at runtime. When the interpreter executes a + operator, it needs to tell if it is adding two numbers or concatenating two strings. Is there a Java type that can hold numbers, strings, Booleans, and more? Is there one that can tell us what its runtime type is? There is! Good old java.lang.Object.

给定一个具有静态类型的Java变量，我们还必须能够确定它在运行时候，持有哪种类型的值。当解释器执行+ 运算时候，它需要判断是两个数字的加法，还是两个字符串的连接。是否有一种Java类型，可以保存数字、字符串、布尔值等，有没有一个可以告知我们运行时候，是什么类型的Java类型，当然有，它就是Java中的 Object类型。

In places in the interpreter where we need to store a Lox value, we can use Object as the type. Java has boxed versions of its primitive types that all subclass Object, so we can use those for Lox’s built-in types:

在解释器中，需要保存Lox值的地方，我们可以使用Object当作类型，Java有其所有原始类型，对应的Object子类，因此，我们可以将它们当作Lox语言的内置类型。


| Lox Type | Java representation |
| --- | ---|
| Any Lox value | Object|
| nil | null|
| Boolean | Boolean|
| number | double|
|string | String|

Given a value of static type Object, we can determine if the runtime value is a number or a string or whatever using Java’s built-in instanceof operator. In other words, the JVM’s own object representation conveniently gives us everything we need to implement Lox’s built-in types. We’ll have to do a little more work later when we add Lox’s notions of functions, classes, and instances, but Object and the boxed primitive classes are sufficient for the types we need right now.


给定一个静态类型的对象，我们可以使用Java内置的 instanceof 操作符，来确定运行时候，该对象的值是数字、字符串还是其他什么。也就是说，JVM自己的对象表示，可以方便的为我们提供实现Lox 内置类型的一切。稍后，当我们添加Lox的函数、类、实例时候，我们将需要做更多的工作，但是，Object 和 封装的原始类，对于我们需要的类型已经足够了。

> Another thing we need to do with values is manage their memory, and Java does that too. A handy object representation and a really nice garbage collector are the main reasons we’re writing our first interpreter in Java.
> 
> 对于值，我们需要做的一件事情是，管理它们的内存，Java也这样做，方便的对象表示和非常好的垃圾回收，是我们用Java编写第一个解释器的主要原因


## 二、Evaluating Expressions

计算表达式

Next, we need blobs of code to implement the evaluation logic for each kind of expression we can parse. We could stuff that code into the syntax tree classes in something like an interpret() method. In effect, we could tell each syntax tree node, “Interpret thyself”. This is the Gang of Four’s Interpreter design pattern. It’s a neat pattern, but like I mentioned earlier, it gets messy if we jam all sorts of logic into the tree classes.

Instead, we’re going to reuse our groovy Visitor pattern. In the previous chapter, we created an AstPrinter class. It took in a syntax tree and recursively traversed it, building up a string which it ultimately returned. That’s almost exactly what a real interpreter does, except instead of concatenating strings, it computes values.

接下来，我们需要写代码，实现我们可以解析的每一种表达式的求值逻辑，我们可以用类似于 interpret()方法的方式，将代码填写到语法树类中。实际上，我们可以告诉每一个语法树节点，解释自己。这是设计模式书籍中的[解析器模式](https://en.wikipedia.org/wiki/Interpreter_pattern), 这是一个整洁的模式，但是，正如我前面提到的，如果我们把各种逻辑都塞到语法树类中，就会变得非常混乱。

相反，我们将使用更加常规的访问者模式，在先前章节中，我们创建了一个 AstPrinter类，它接受一个语法树，然后递归遍历它，构建一个字符串，并且最终返回这个字符串。这几乎是，真正的解释器所做的，只是它不是连接字符串，而是计算值。

We start with a new class.

我们新建一个类

```java

// lox/Interpreter.java, create new file

package com.craftinginterpreters.lox;

class Interpreter implements Expr.Visitor<Object> {
}

```

The class declares that it’s a visitor. The return type of the visit methods will be Object, the root class that we use to refer to a Lox value in our Java code. To satisfy the Visitor interface, we need to define visit methods for each of the four expression tree classes our parser produces. We’ll start with the simplest . . . 

Interpreter类，声明了它是访问者，访问方法的返回值是一个object类型，这是我们Java代码中用来引用Lox值的根类。为了满足Visitor接口，我们需要为解析器生成的4个表达式树类中的每一个定义访问方法，让我们从最简单的开始...

### 2.1 Evaluating literals

计算文本

The leaves of an expression tree—the atomic bits of syntax that all other expressions are composed of—are literals. Literals are almost values already, but the distinction is important. A literal is a bit of syntax that produces a value. A literal always appears somewhere in the user’s source code. Lots of values are produced by computation and don’t exist anywhere in the code itself. Those aren’t literals. A literal comes from the parser’s domain. Values are an interpreter concept, part of the runtime’s world.

表达式语法树的叶子，所有其他表达式组成的语法，都是原子性的文本。文本几乎已经是值了，但是区分也很重要。文本也是一个产生值的语法，文本总是出现在用户源代码中的某个位置，很多值都是通过计算产生的，并且不存在于代码的任何地方，它们不是文本，文本来自解析器的作用域，值是一个解释器概念，是运行时的一部分。

> In the next chapter, when we implement variables, we’ll add identifier expressions, which are also leaf nodes.
> 
> 在下一章中，当我们实现变量时候，我们将添加标识符表达式，它们也是叶节点。


So, much like we converted a literal token into a literal syntax tree node in the parser, now we convert the literal tree node into a runtime value. That turns out to be trivial.

因此，就像我们在解析器中，将文本 token转变为文本语法树节点一样，现在我们将文本语法树节点，转变为运行时值，转变是非常简单的

```java

// lox/Interpreter.java, in class Interpreter

  @Override
  public Object visitLiteralExpr(Expr.Literal expr) {
    return expr.value;
  }
  
```

We eagerly produced the runtime value way back during scanning and stuffed it in the token. The parser took that value and stuck it in the literal tree node, so to evaluate a literal, we simply pull it back out.

我们在扫描阶段生成了运行时候值，并且将它填充到token中，解析器获取到该值，并且将其固定在文本树节点中，因此，接下来想要计算文本，我们只需要将其拉出来就可以了。

### 2.2 Evaluating parentheses

计算括号

The next simplest node to evaluate is grouping—the node you get as a result of using explicit parentheses in an expression.

接下来要处理的节点，最简单，我们只需要显示的用括号，将获取到的节点进行分组

```java

// lox/Interpreter.java, in class Interpreter

 @Override
  public Object visitGroupingExpr(Expr.Grouping expr) {
    return evaluate(expr.expression);
  }
  
```

A grouping node has a reference to an inner node for the expression contained inside the parentheses. To evaluate the grouping expression itself, we recursively evaluate that subexpression and return it.

一个分组节点，包含有对于内部表达式节点的引用，为了计算表达式本身，我们需要递归的计算子表达式，并且返回结果

We rely on this helper method which simply sends the expression back into the interpreter’s visitor implementation:

我们将依赖这个帮助方法，它只需要将表达式发送回解释器的访问者模式实现

```java

// lox/Interpreter.java, in class Interpreter

  private Object evaluate(Expr expr) {
    return expr.accept(this);
  }
  
```

> Some parsers don’t define tree nodes for parentheses. Instead, when parsing a parenthesized expression, they simply return the node for the inner expression. We do create a node for parentheses in Lox because we’ll need it later to correctly handle the left-hand sides of assignment expressions.
>
> 有一些解析器不会为括号定义树节点，相反，当遇到带括号的表达式时候，它们只会返回内部表达式的节点，我们确实为Lox语言中为括号创建了一个节点，因为稍后，我们将用到它，来正确处理赋值表达式的左侧。

## 2.3 Evaluating unary expressions

计算一元表达式

Like grouping, unary expressions have a single subexpression that we must evaluate first. The difference is that the unary expression itself does a little work afterwards.

和分组一样，一元表达式，有一个子表达式，我们需要先进行计算，不同于分组，我们在一元表达式计算结果后，又做了一些工作。


```java

// lox/Interpreter.java, add after visitLiteralExpr()

  @Override
  public Object visitUnaryExpr(Expr.Unary expr) {
    Object right = evaluate(expr.right);

    switch (expr.operator.type) {
      case MINUS:
        return -(double)right;
    }

    // Unreachable.
    return null;
  }
  
```

First, we evaluate the operand expression. Then we apply the unary operator itself to the result of that. There are two different unary expressions, identified by the type of the operator token.

Shown here is -, which negates the result of the subexpression. The subexpression must be a number. Since we don’t statically know that in Java, we cast it before performing the operation. This type cast happens at runtime when the - is evaluated. That’s the core of what makes a language dynamically typed right there.

首先，我们计算内部的操作数表达式，然后，我们将一元运算符本身应用于计算结果，有两个不同的一元运算符，是由运算符token的类型决定的

上面展示的是 -， 它表示子表达式结果的负值，子表达式的结果必须是数值，因为我们在Java中不知道这一点，所以，我们在进行操作之前，强制类型转换，当计算 - 运算时候，此类型转换在运行时候发生，这正是动态类型语言的核心所在。


> You’re probably wondering what happens if the cast fails. Fear not, we’ll get into that soon.
> 
> 你可能想要知道如果类型转换失败，会有什么结果，我们马上就会介绍

You can start to see how evaluation recursively traverses the tree. We can’t evaluate the unary operator itself until after we evaluate its operand subexpression. That means our interpreter is doing a post-order traversal—each node evaluates its children before doing its own work.

你可以开始看到，求值如何递归遍历树，在计算一元表达式的右边子表达式之前，我们无法计算一元表达式的值，这意外着我们的解释器将进行一个后序遍历——执行自己本身之前，先对自己的子节点进行计算

The other unary operator is logical not.

还有一个一元运算符是 布尔非

```java

// lox/Interpreter.java, in visitUnaryExpr()

    switch (expr.operator.type) {
      case BANG:
        return !isTruthy(right);
      case MINUS:
	  
```

The implementation is simple, but what is this “truthy” thing about? We need to make a little side trip to one of the great questions of Western philosophy: What is truth?

实现非常简单，但是"真实“ 是什么东西呢？我们需要对西方哲学的一个重大问题做一个小小的旁观者——什么是真理？

### 2.4 Truthiness and falsiness

真实和虚假

OK, maybe we’re not going to really get into the universal question, but at least inside the world of Lox, we need to decide what happens when you use something other than true or false in a logic operation like ! or any other place where a Boolean is expected.

We could just say it’s an error because we don’t roll with implicit conversions, but most dynamically typed languages aren’t that ascetic. Instead, they take the universe of values of all types and partition them into two sets, one of which they define to be “true”, or “truthful”, or (my favorite) “truthy”, and the rest which are “false” or “falsey”. This partitioning is somewhat arbitrary and gets weird in a few languages.

好吧，也许我们不会去思考这个重大的哲学问题，但是，至少在Lox的世界中，我们需要定义当 我们使用非 true/false这样的逻辑值，和逻辑运算符 ！，计算时候，将会得到什么呢？或者其他需要提供一个逻辑值的地方

我们可以定义，不支持其他类型的逻辑非运算，因为我们不支持隐式转换，但是，大多数的动态语言没有这么严格的限制，相反，我们会将所有类型的值分为两组，其中一组定义为真，另外一组定义为假，这种分区比较随意，在一些语言中会变得有些奇怪。

> In JavaScript, strings are truthy, but empty strings are not. Arrays are truthy but empty arrays are . . . also truthy. The number 0 is falsey, but the string "0" is truthy.
>
> In Python, empty strings are falsey like in JS, but other empty sequences are falsey too.
>
> In PHP, both the number 0 and the string "0" are falsey. Most other non-empty strings are truthy.
>
> Get all that?
>
> 在JavaScript 中，字符串是true，空字符串是false，数组是true，但是空数组也是true，😄，数字0是false，但是字符串"0"是true
>
> 在Python中，空字符串和JS相似，但是其他空序列也是 false
>
> 在PHP中，无论数字0还是字符串 "0" 都是false，大多数其他非空字符串都是true
>
> 明白了吗？

Lox follows Ruby’s simple rule: false and nil are falsey, and everything else is truthy. We implement that like so:

Lox借鉴了Ruby中的简单判断方法，false和 nil 是虚假的，其他都是真实的

```java

// lox/Interpreter.java, add after visitUnaryExpr()

  private boolean isTruthy(Object object) {
    if (object == null) return false;
    if (object instanceof Boolean) return (boolean)object;
    return true;
  }
  
```

### 2.5 Evaluating binary operators

计算二元运算符

On to the last expression tree class, binary operators. There’s a handful of them, and we’ll start with the arithmetic ones.

最后一个表达式计算，二元运算符，我们将先从算术运算符开始

```java

// lox/Interpreter.java, add after evaluate()

  @Override
  public Object visitBinaryExpr(Expr.Binary expr) {
    Object left = evaluate(expr.left);
    Object right = evaluate(expr.right); 

    switch (expr.operator.type) {
      case MINUS:
        return (double)left - (double)right;
      case SLASH:
        return (double)left / (double)right;
      case STAR:
        return (double)left * (double)right;
    }

    // Unreachable.
    return null;
  }
  
```

> Did you notice we pinned down a subtle corner of the language semantics here? In a binary expression, we evaluate the operands in left-to-right order. If those operands have side effects, that choice is user visible, so this isn’t simply an implementation detail.
> 
> If we want our two interpreters to be consistent (hint: we do), we’ll need to make sure clox does the same thing.
>
> 你是否注意到，我们这里了一个语言语义的一个角落，在二元表达式中，我们会从左到右计算操作数，如果这些操作数有副作用，用户可以看到这些选项，因此，这不仅仅是一个实现细节
> 
> 如果你希望我们的两个解释器保持一致，（我们真的做到了），我们需要确保cLox解释器，实现相同的操作

I think you can figure out what’s going on here. The main difference from the unary negation operator is that we have two operands to evaluate.

I left out one arithmetic operator because it’s a little special.

我认为，你应该可以弄清楚这里发生了什么，与一元否定操作符不同的是，我们需要计算左/右两个操作数

我遗漏了一个算术运算符，因为它有些特殊

```java


// lox/Interpreter.java, in visitBinaryExpr()

    switch (expr.operator.type) {
      case MINUS:
        return (double)left - (double)right;
      case PLUS:
        if (left instanceof Double && right instanceof Double) {
          return (double)left + (double)right;
        } 

        if (left instanceof String && right instanceof String) {
          return (String)left + (String)right;
        }

        break;
      case SLASH:
	  
```

The + operator can also be used to concatenate two strings. To handle that, we don’t just assume the operands are a certain type and cast them, we dynamically check the type and choose the appropriate operation. This is why we need our object representation to support instanceof.

+ 运算符，可以用于连接两个字符串，为了处理这个问题，我们不只是假设操作数是某个类型，并且进行类型转换，而是动态检查类型，并且进行适当的操作，这就是我们为什么需要对象支持 instanceof


> We could have defined an operator specifically for string concatenation. That’s what Perl (.), Lua (..), Smalltalk (,), Haskell (++), and others do.
>
> I thought it would make Lox a little more approachable to use the same syntax as Java, JavaScript, Python, and others. This means that the + operator is overloaded to support both adding numbers and concatenating strings. Even in languages that don’t use + for strings, they still often overload it for adding both integers and floating-point numbers.
> 
> 我们也可以定义一个字符串连接的运算符，这也是 Perl(.) Lua(..) Smalltalk(,) Haskell(++) 等语言所做的
>
> 我认为使用和Java/JS/Python一样的语法，会让Lox更加容易接近，这意味着 + 运算符被重载，用于支持数字的加法和字符串的连接，即使在不使用 + 连接字符串的语言中，它们仍然在计算整数和浮点数加法运算时，重载。

Next up are the comparison operators.

接下来是比较运算符

```java

// lox/Interpreter.java, in visitBinaryExpr()

    switch (expr.operator.type) {
      case GREATER:
        return (double)left > (double)right;
      case GREATER_EQUAL:
        return (double)left >= (double)right;
      case LESS:
        return (double)left < (double)right;
      case LESS_EQUAL:
        return (double)left <= (double)right;
      case MINUS:
	  
```


They are basically the same as arithmetic. The only difference is that where the arithmetic operators produce a value whose type is the same as the operands (numbers or strings), the comparison operators always produce a Boolean.

它们和算术运算符一样，唯一的区别是，算术运算符的计算结果总是和操作数类型相同（数值或者string），而比较运算符结果总是布尔值

The last pair of operators are equality.

最后，一个二元运算符是 相等判断

```java

// lox/Interpreter.java, in visitBinaryExpr()

  case BANG_EQUAL: return !isEqual(left, right);
  case EQUAL_EQUAL: return isEqual(left, right);
```
