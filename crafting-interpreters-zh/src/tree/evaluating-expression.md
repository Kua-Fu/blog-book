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

Unlike the comparison operators which require numbers, the equality operators support operands of any type, even mixed ones. You can’t ask Lox if 3 is less than "three", but you can ask if it’s equal to it.

与需要数字的比较运算符不同，相等运算符支持所有类型的操作数，甚至是混合操作数，你不能问Lox语言，3是否小于 "three", 但是你可以问Lox语言，3 是否等于 "three"

> Spoiler alert: it’s not.
> 
> 剧透提醒，不是

Like truthiness, the equality logic is hoisted out into a separate method.

和 isTruthy() 函数一样，isEqual() 函数也是一个单独的方法

```

// lox/Interpreter.java, add after isTruthy()

 private boolean isEqual(Object a, Object b) {
    if (a == null && b == null) return true;
    if (a == null) return false;

    return a.equals(b);
  }
  
```

This is one of those corners where the details of how we represent Lox objects in terms of Java matter. We need to correctly implement Lox’s notion of equality, which may be different from Java’s.

Fortunately, the two are pretty similar. Lox doesn’t do implicit conversions in equality and Java does not either. We do have to handle nil/null specially so that we don’t throw a NullPointerException if we try to call equals() on null. Otherwise, we’re fine. Java’s equals() method on Boolean, Double, and String have the behavior we want for Lox.

这是我们如何用Java实现Lox 对象的细节所在的角落之一，我们需要正确的实现Lox 语言的相等概念，这可能和Java 不一样。

幸运的是，两者非常相似，Lox不会对等式中的隐式转换，Java也不做转换，我们必须考虑到nil/null， 以便在尝试对 null调用 isEqual()  函数时候，不会引发空指针报错，其他的，我们都很好，Java对Boolean，Double，String类型具有相同的equals() 函数。

> What do you expect this to evaluate to:
>
> (0 / 0) == (0 / 0)
> 
> According to IEEE 754, which specifies the behavior of double-precision numbers, dividing a zero by zero gives you the special NaN (“not a number”) value. Strangely enough, NaN is not equal to itself.
> 
> In Java, the == operator on primitive doubles preserves that behavior, but the equals() method on the Double class does not. Lox uses the latter, so doesn’t follow IEEE. These kinds of subtle incompatibilities occupy a dismaying fraction of language implementers’ lives.
> 
> 你希望的评估结果是什么呢？
>
> 根据 IEEE 754, 它规定了双精度数字的行为，将 0/0 可以得到特殊的 NaN（非数字值），奇怪的是，NaN并不等于自己。
> 
> 在Java中，基于Double类型的 == 运算符保留着这种行为，但是 Double类上的equals() 方法则不保留这种行为，Lox使用后者，因此不遵守IEEE，这些微妙的不兼容性，占据了语言实现者生活中令人沮丧的一小部分。

And that’s it! That’s all the code we need to correctly interpret a valid Lox expression. But what about an invalid one? In particular, what happens when a subexpression evaluates to an object of the wrong type for the operation being performed?

就这样！这就是正确解释有效Lox表达式所需的全部代码，但是无效的呢？特别是，当子表达式的计算结果为所执行操作的错误类型的对象时候，会发生什么？

## 三、Runtime Errors


I was cavalier about jamming casts in whenever a subexpression produces an Object and the operator requires it to be a number or a string. Those casts can fail. Even though the user’s code is erroneous, if we want to make a usable language, we are responsible for handling that error gracefully.

每当子表达式产生一个Object，但是运算符需要的是数字或者字符串时候，我们会随意的使用强制类型转换。这些类型转换可能会失败，即使用户的代码是错误的，但是，如果我们想要创建一门可用的语言，我们有责任优雅的处理这种错误。

> We could simply not detect or report a type error at all. This is what C does if you cast a pointer to some type that doesn’t match the data that is actually being pointed to. C gains flexibility and speed by allowing that, but is also famously dangerous. Once you misinterpret bits in memory, all bets are off.
> 
> Few modern languages accept unsafe operations like that. Instead, most are memory safe and ensure—through a combination of static and runtime checks—that a program can never incorrectly interpret the value stored in a piece of memory.
> 
> 我们根本无法检测或者报告类型错误，在C语言中，如果将指针转换为实际数据不相符的某个类型，C语言通常会，允许用户这样转换，以获取速度和灵活性，但是这也是总所周知的危险，一旦我们错误的转换了指针类型，将可能产生严重的错误。
> 
> 很少有现代语言接受这样的不安全操作，相反，大多数都是内存安全的，并且通过静态和运行时检查的组合，确保程序不会错误的解释存储在内存中的值。

It’s time for us to talk about runtime errors. I spilled a lot of ink in the previous chapters talking about error handling, but those were all syntax or static errors. Those are detected and reported before any code is executed. Runtime errors are failures that the language semantics demand we detect and report while the program is running (hence the name).

Right now, if an operand is the wrong type for the operation being performed, the Java cast will fail and the JVM will throw a ClassCastException. That unwinds the whole stack and exits the application, vomiting a Java stack trace onto the user. That’s probably not what we want. The fact that Lox is implemented in Java should be a detail hidden from the user. Instead, we want them to understand that a Lox runtime error occurred, and give them an error message relevant to our language and their program.

现在是我们讨论运行时错误的时候了，我们已经在前面的章节中讨论了错误处理，但是它们都是语法和静态错误，在执行任何代码之前，这些错误都会被检测并且报告，运行时错误是语言语义要求我们在程序运行时，检测并且报告故障（因此得名）

现在，如果操作数的类型和正在执行的操作运算不符合，Java的强制转换将失败，JVM会抛出类型转换报错，这将解开整个堆栈，并且退出应用程序，从而向用户抛出Java堆栈跟踪，但是，这可能不是我们想要的，Lox语言是在Java中实现的这一个事实，应该是对用户隐藏的细节。相反，我们希望他们了解发生了运行时错误，并且向用户，提供和我们语言和程序相关的错误消息。


The Java behavior does have one thing going for it, though. It correctly stops executing any code when the error occurs. Let’s say the user enters some expression like:

```
2 * (3 / -"muffin")
```

You can’t negate a muffin, so we need to report a runtime error at that inner - expression. That in turn means we can’t evaluate the / expression since it has no meaningful right operand. Likewise for the *. So when a runtime error occurs deep in some expression, we need to escape all the way out.

不过，Java的行为确实有一个原因，当发生错误时候，它会停止执行任何代码，假设用户输入下面的表达式

我们无法计算 muffin 的负值，因此我们需要在子表达式计算时候，报告错误，这样，我们也无法计算 / ，因此该操作符的右操作数没有意义，同样， * 运算符也一样。因此，当运行时错误发生在深层的子表达式时候，我们会一直往上忽略。


> I don’t know, man, can you negate a muffin?
> 
> ![muffin](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/muffin.png?raw=true)

We could print a runtime error and then abort the process and exit the application entirely. That has a certain melodramatic flair. Sort of the programming language interpreter equivalent of a mic drop.

Tempting as that is, we should probably do something a little less cataclysmic. While a runtime error needs to stop evaluating the expression, it shouldn’t kill the interpreter. If a user is running the REPL and has a typo in a line of code, they should still be able to keep the session going and enter more code after that.

我们可以打印一个运行时错误，然后终止进程并且退出应用程序，这有一定的戏剧性，某种语言的解释器，相当于降下麦克风。

尽管如此，我们还是应该做一些事情，不会引发灾难，虽然，运行时错误需要停止计算表达式，但是，它不应该停止解释器，如果用户正在运行一个 REPL 类型的程序，一行代码中有一个错误，那么解释器不应该退出，应该可以继续会话，在终端输入更多的代码。



### 3.1 Detecting runtime errors

检测运行时错误

Our tree-walk interpreter evaluates nested expressions using recursive method calls, and we need to unwind out of all of those. Throwing an exception in Java is a fine way to accomplish that. However, instead of using Java’s own cast failure, we’ll define a Lox-specific one so that we can handle it how we want.

我们的树遍历解释器，使用递归调用实现了计算嵌套表达式，我们需要解开所有这些。在Java中抛出异常，是实现这一点的好方法。然而，我们将定义一个特定于Lox的失败，而不是使用Java中默认的类型转换错误，这样，我们可以按照自己意愿处理报错。

Before we do the cast, we check the object’s type ourselves. So, for unary -, we add:

在进行强制类型之前，我们需要自己检查对象的类型，因此，对于一元运算符

```java

// lox/Interpreter.java, in visitUnaryExpr()

      case MINUS:
        checkNumberOperand(expr.operator, right);
        return -(double)right;
		
```

The code to check the operand is:

检查运算数的代码如下，

```java

// lox/Interpreter.java, add after visitUnaryExpr()

  private void checkNumberOperand(Token operator, Object operand) {
    if (operand instanceof Double) return;
    throw new RuntimeError(operator, "Operand must be a number.");
  }
  
```

When the check fails, it throws one of these:

当检测到失败，我们会抛出报错

```java

// lox/RuntimeError.java, create new file

package com.craftinginterpreters.lox;

class RuntimeError extends RuntimeException {
  final Token token;

  RuntimeError(Token token, String message) {
    super(message);
    this.token = token;
  }
}

```

Unlike the Java cast exception, our class tracks the token that identifies where in the user’s code the runtime error came from. As with static errors, this helps the user know where to fix their code.

和Java中的强制类型转换异常不一样，我们自己实现的类，追踪到用户代码运行时报错的来源token，与静态错误一样，这样有助于用户知道如何修复代码。


> I admit the name “RuntimeError” is confusing since Java defines a RuntimeException class. An annoying thing about building interpreters is your names often collide with ones already taken by the implementation language. Just wait until we support Lox classes.
> 
> 我承认，我们定义的类，类名是RuntimeError，非常令人困惑，因为Java中定义了RuntimeException 类，构建解释器的一个令人讨厌的问题是，我们的名字经常会和实现语言的已经存在的名字相同。需要我们耐心等待，一直到Lox实现类。

We need similar checking for the binary operators. Since I promised you every single line of code needed to implement the interpreters, I’ll run through them all.

我们还需要对二元运算符进行相似的类型检查，既然，我已经保证了实现解释器的每一行代码都会出现，我会把它们写在下面

```java


  @Override
    public Object visitBinaryExpr(Expr.Binary expr) {
        Object left = evaluate(expr.left);
        Object right = evaluate(expr.right);

        switch (expr.operator.type) {
            case GREATER:
                checkNumberOperands(expr.operator, left, right);
                return (double) left > (double) right;
            case GREATER_EQUAL:
                checkNumberOperands(expr.operator, left, right);
                return (double) left >= (double) right;
            case LESS:
                checkNumberOperands(expr.operator, left, right);
                return (double) left < (double) right;
            case LESS_EQUAL:
                checkNumberOperands(expr.operator, left, right);
                return (double) left <= (double) right;
            case MINUS:
                checkNumberOperands(expr.operator, left, right);
                return (double) left - (double) right;
            case PLUS:
                if (left instanceof Double && right instanceof Double) {
                    return (double) left + (double) right;
                }
                if (left instanceof String && right instanceof String) {
                    return (String) left + (String) right;
                }
            case SLASH:
                checkNumberOperands(expr.operator, left, right);
                return (double) left / (double) right;
            case STAR:
                checkNumberOperands(expr.operator, left, right);
                return (double) left * (double) right;
            case BANG_EQUAL:
                return !isEqual(left, right);
            case EQUAL_EQUAL:
                return isEqual(left, right);
        }
        return null;
    }


```


```java

// lox/Interpreter.java, add after checkNumberOperand()

  private void checkNumberOperands(Token operator,
                                   Object left, Object right) {
    if (left instanceof Double && right instanceof Double) return;
    
    throw new RuntimeError(operator, "Operands must be numbers.");
  }


```


> Another subtle semantic choice: We evaluate both operands before checking the type of either. Imagine we have a function say() that prints its argument then returns it. Using that, we write:
>
> say("left") - say("right");
>
> Our interpreter prints “left” and “right” before reporting the runtime error. We could have instead specified that the left operand is checked before even evaluating the right.
> 
> 另外一个微妙的语义选择，我们在检查两个操作数的类型之前，对它们进行求值，假设我们有一个say() 函数，它打印其参数，然后返回它，使用这个函数，我们实现
>
> 我们的解释器在报告运算时报错之前，打印出左、右，相反，我们可以指定在计算右操作数之前，检查左操作数。


The last remaining operator, again the odd one out, is addition. Since + is overloaded for numbers and strings, it already has code to check the types. All we need to do is fail if neither of the two success cases match.

剩下最后一个操作符，是加法，由于 + 对于数字和字符串是重载的，因此，它已经有代码进行类型检查了，如果两个重载类型都不符合，需要报错

```java

// lox/Interpreter.java, in visitBinaryExpr(), replace 1 line

          return (String)left + (String)right;
        }

        throw new RuntimeError(expr.operator,
            "Operands must be two numbers or two strings.");
      case SLASH:
	  
```

That gets us detecting runtime errors deep in the innards of the evaluator. The errors are getting thrown. The next step is to write the code that catches them. For that, we need to wire up the Interpreter class into the main Lox class that drives it.

这使得我们可以检测到计算内部的运行时报错，下一步需要编写捕获错误的代码，为此，我们需要将 Interperter类添加到主Lox类上。

## 四、Hooking Up the Interpreter

连接到解释器

The visit methods are sort of the guts of the Interpreter class, where the real work happens. We need to wrap a skin around them to interface with the rest of the program. The Interpreter’s public API is simply one method.

访问方法是解释器类的核心，因为真正的工作都发生在这里，我们需要将它包裹起来，以便和程序的其他部分交互，解释器的公共API只是一种方法。

```java

// lox/Interpreter.java, in class Interpreter

  void interpret(Expr expression) { 
    try {
      Object value = evaluate(expression);
      System.out.println(stringify(value));
    } catch (RuntimeError error) {
      Lox.runtimeError(error);
    }
  }
  
```

This takes in a syntax tree for an expression and evaluates it. If that succeeds, evaluate() returns an object for the result value. interpret() converts that to a string and shows it to the user. To convert a Lox value to a string, we rely on:

上面方法，将会获取表达式的解析树，并且对其求值，如果求值成功，evaluate()方法，将会返回结果值的对象，interpret() 方法，会将结果转换为字符串，并且返回给用户，要将结果转为字符串，我们需要:

```java

// lox/Interpreter.java, add after isEqual()

  private String stringify(Object object) {
    if (object == null) return "nil";

    if (object instanceof Double) {
      String text = object.toString();
      if (text.endsWith(".0")) {
        text = text.substring(0, text.length() - 2);
      }
      return text;
    }

    return object.toString();
  }

```


This is another of those pieces of code like isTruthy() that crosses the membrane between the user’s view of Lox objects and their internal representation in Java.

It’s pretty straightforward. Since Lox was designed to be familiar to someone coming from Java, things like Booleans look the same in both languages. The two edge cases are nil, which we represent using Java’s null, and numbers.

Lox uses double-precision numbers even for integer values. In that case, they should print without a decimal point. Since Java has both floating point and integer types, it wants you to know which one you’re using. It tells you by adding an explicit .0 to integer-valued doubles. We don’t care about that, so we hack it off the end.


这是另外一段，像是 isTruthy() 方法的代码，它跨越了用户对Lox对象的视图和它们在Java中的内部表示之间的隔阂。

这很简单，因为Lox的设计为了让Java的使用者熟悉，所以，布尔类型在两种语言中几乎相同，这两种边缘情况都是nil，我们使用Java的null 和数字来表示

Lox甚至对整数，使用双精度类型表示，在这种情况下，它们应该没有小数点，由于Java既有浮点型，也有整数型，所以它希望你知道使用的哪一种类型，它通过显式的将 .0 添加到整数，用双精度表示的方法，来告知。我们不在乎这一点，所以，我们从头开始。


> Yet again, we take care of this edge case with numbers to ensure that jlox and clox work the same. Handling weird corners of the language like this will drive you crazy but is an important part of the job.
> 
> Users rely on these details—either deliberately or inadvertently—and if the implementations aren’t consistent, their program will break when they run it on different interpreters.
> 
> 再一次，我们用数字来处理边缘状况，以确保jlox 和 clox的处理方式相同，像这样的处理，语言中奇怪角落会让你发疯，但这是工作的重要组成部分
> 
> 用户有意或者无意的依赖这些细节，如果实现不一致，他们的程序将在不同的解释器上，出现中断。


### 4.1 Reporting runtime errors

报告运行时报错

If a runtime error is thrown while evaluating the expression, interpret() catches it. This lets us report the error to the user and then gracefully continue. All of our existing error reporting code lives in the Lox class, so we put this method there too:

如果在计算表达式时候，引发运行时报错，interpert() 方法，将会捕获到该类型的错误，这样，我们可以向用户报告错误，然后，优雅的继续运行，我们现在，所有的报错代码都在Lox类中，所以，我们将此方法写在那里：

```java

// lox/Lox.java, add after error()

  static void runtimeError(RuntimeError error) {
    System.err.println(error.getMessage() +
        "\n[line " + error.token.line + "]");
    hadRuntimeError = true;
  }


```


We use the token associated with the RuntimeError to tell the user what line of code was executing when the error occurred. Even better would be to give the user an entire call stack to show how they got to be executing that code. But we don’t have function calls yet, so I guess we don’t have to worry about it.

我们使用与 RuntimeError 关联的token，来告诉用户，错误发生时候正在运行的代码行数，更好的方法是给用户一个完整的调用堆栈，以显示他们是如何执行代码的，但是我们还没有函数调用，所以，我想还不需要担心。

After showing the error, runtimeError() sets this field:

显示错误后，将把变量 hadRuntimeError 设置为true

```java

// lox/Lox.java, in class Lox

  static boolean hadError = false;
  static boolean hadRuntimeError = false;

  public static void main(String[] args) throws IOException {
  
```


That field plays a small but important role.

这个字段 hadRuntimeError有重要的作用

```java

// lox/Lox.java, in runFile()

    run(new String(bytes, Charset.defaultCharset()));

    // Indicate an error in the exit code.
    if (hadError) System.exit(65);
    if (hadRuntimeError) System.exit(70);
  }
  
```

If the user is running a Lox script from a file and a runtime error occurs, we set an exit code when the process quits to let the calling process know. Not everyone cares about shell etiquette, but we do.

如果用户正在从文件运行Lox 脚本，并且发生了运行时报错，我们将在进程退出时候，设置退出代码，让调用进程知道，不是每一个人都关心shell 退出信号，但是我们确实关心。

> If the user is running the REPL, we don’t care about tracking runtime errors. After they are reported, we simply loop around and let them input new code and keep going.
>
> 如果用户正在运行 REPL, 我们不关心跟踪运行时报错，在运行时报错报告后，我们只需要循环，让用户继续输入新代码并且继续解释运行。

### 4.2 Running the interpreter

运行解释器

Now that we have an interpreter, the Lox class can start using it.

现在我们已经有了一个解释器，Lox类可以使用它了

```java

// lox/Lox.java, in class Lox

public class Lox {
  private static final Interpreter interpreter = new Interpreter();
  static boolean hadError = false;

```

We make the field static so that successive calls to run() inside a REPL session reuse the same interpreter. That doesn’t make a difference now, but it will later when the interpreter stores global variables. Those variables should persist throughout the REPL session.

我们将字段 interpreter 设置为静态，以便于对 REPL会话中，对于run()方法的连续调用，使用相同的解释器，现在，这样并没有什么不同，但是稍后，当解释器存储了全局变量后，情况就会发生变化，这些变量在 REPL 会话中会始终存在。

Finally, we remove the line of temporary code from the last chapter for printing the syntax tree and replace it with this:

最后，我们删除了上一章，用于打印语法树的一行临时代码，并且替换为

```java

// lox/Lox.java, in run(), replace 1 line

    // Stop if there was a syntax error.
    if (hadError) return;

    interpreter.interpret(expression);
  }

```

We have an entire language pipeline now: scanning, parsing, and execution. Congratulations, you now have your very own arithmetic calculator.

As you can see, the interpreter is pretty bare bones. But the Interpreter class and the Visitor pattern we’ve set up today form the skeleton that later chapters will stuff full of interesting guts—variables, functions, etc. Right now, the interpreter doesn’t do very much, but it’s alive!

我们现在有了一个完整的语言管道，扫描、解析和执行，恭喜你，现在我们有了自己的算术计算器

正如我们都可以看到的，现在的解释器是非常简单的，但是，我们今天构造的解释器类和访问者模式，构成了一个框架，在后面，将会充满有意思的内脏——变量、函数。现在，解释器做的不多，但是的确有作用！

![skeleton](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/skeleton.png?raw=true)

## 五、CHALLENGES

习题

1. Allowing comparisons on types other than numbers could be useful. The operators might have a reasonable interpretation for strings. Even comparisons among mixed types, like 3 < "pancake" could be handy to enable things like ordered collections of heterogeneous types. Or it could simply lead to bugs and confusion.

   Would you extend Lox to support comparing other types? If so, which pairs of types do you allow and how do you define their ordering? Justify your choices and compare them to other languages.


	
1. Many languages define + such that if either operand is a string, the other is converted to a string and the results are then concatenated. For example, "scone" + 4 would yield scone4. Extend the code in visitBinaryExpr() to support that.


1. What happens right now if you divide a number by zero? What do you think should happen? Justify your choice. How do other languages you know handle division by zero, and why do they make the choices they do?

	Change the implementation in visitBinaryExpr() to detect and report a runtime error for this case.




##

1. 	允许对数字之外的类型进行比较，可能会非常有用，运算符可能对字符串有合理的解释，即使是混合类型之间的比较，例如：3 < "pancake", 也可以方便的实现，异构类型的有序集合，或者它可能会导致错误和混乱。

	你会扩展比较运算符，支持不同的数据类型吗？如果扩展，你会支持对于哪些类型，如何定义它们的大小，验证你的想法，并且与其他语法进行比较。

1. 许多语言定义+ 运算符，如果一个操作数是字符串，那么另外一个操作数也会转变为字符串，然后将它们的结果连接，例如: "scone" + 4 将变为 "scone4", 扩展代码，实现这个功能

1. 如果除零，现在Lox会发生什么，你认为应该发生什么？证明你的想法是正确的。其他语言是如何实现除零的，为什么他们会这样做呢？

	修改 visitBinaryExpr()方法，遇到除零情况，报告一个运行时错误
	
	
## 六、DESIGN NOTE: STATIC AND DYNAMIC TYPING

设计说明：静态和动态类型


Some languages, like Java, are statically typed which means type errors are detected and reported at compile time before any code is run. Others, like Lox, are dynamically typed and defer checking for type errors until runtime right before an operation is attempted. We tend to consider this a black-and-white choice, but there is actually a continuum between them.

It turns out even most statically typed languages do some type checks at runtime. The type system checks most type rules statically, but inserts runtime checks in the generated code for other operations.

有些语言，例如Java，是静态类型语言，这意味着，在运行任何代码之前，都会在编译时候，检测并且报告类型错误； 其他的，例如：Lox语言，是动态类型语言，将检查类型错误的时间推迟到运行时候，然后再尝试操作。我们倾向于这是一个非黑即白的选择，但是，实际上，它们之间存在连续场景。

事实证明，即使是，大多数的静态类型语言，也会在运行时候，执行一些类型检测。类型系统会静态检查大多数的规则，但是，生成的代码中还是会插入运行时检查，以用于其他操作。

For example, in Java, the static type system assumes a cast expression will always safely succeed. After you cast some value, you can statically treat it as the destination type and not get any compile errors. But downcasts can fail, obviously. The only reason the static checker can presume that casts always succeed without violating the language’s soundness guarantees, is because the cast is checked at runtime and throws an exception on failure.

例如：在Java中，静态类型系统，总是假设强制转换表达式，一定执行成功。在强制转换某个值后，可以将其静态的视为目标类型，而不会出现任何的编译报错。但是，显然，不是所有的类型转换都能成功。静态检查器，可以假设强制转换总是成功，并且不违反语言的可靠性的唯一原因是，强制转换在运行时候还会被检查，并且在转换失败后，抛出异常。

A more subtle example is covariant arrays in Java and C#. The static subtyping rules for arrays allow operations that are not sound. Consider:

一个更加微妙的例子是，Java 或者 C# 中的协变数组，数组的静态子类型规则允许不正确的操作，例如：

```java

Object[] stuff = new Integer[1];
stuff[0] = "not an int!";


```

This code compiles without any errors. The first line upcasts the Integer array and stores it in a variable of type Object array. The second line stores a string in one of its cells. The Object array type statically allows that—strings are Objects—but the actual Integer array that stuff refers to at runtime should never have a string in it! To avoid that catastrophe, when you store a value in an array, the JVM does a runtime check to make sure it’s an allowed type. If not, it throws an ArrayStoreException.

Java could have avoided the need to check this at runtime by disallowing the cast on the first line. It could make arrays invariant such that an array of Integers is not an array of Objects. That’s statically sound, but it prohibits common and safe patterns of code that only read from arrays. Covariance is safe if you never write to the array. Those patterns were particularly important for usability in Java 1.0 before it supported generics. James Gosling and the other Java designers traded off a little static safety and performance—those array store checks take time—in return for some flexibility.



上面的代码在编译时候，不会报错。第一行代码，转换Integer数组，并且保存在Object数组类型的变量 stuff中，第二行代码，在其中的一个数组单元中，保存字符串。Object数组，在编译时候，允许其中的元素是字符串（字符串也是Object），但是，在实际运行时候，引用的整数数组中不应该有字符串。为了避免这种灾难级错误，当你在数组中存储数值时候，JVM 将会在运行时候进行类型检查，以确保是正确的类型，如果出现不允许的类型，将会抛出 ArrayStoreException 报错。


Java 也可以通过静止第一行代码中的，强制类型转换，来避免运行时候的检查。它可以使数组保持不变，从而整数数组不会变为对象数组。这是静态的，但是它禁止，只从数组中读取的常见安全的代码模式。如果从不写入数组，则协变数组是安全的。在Java1.0支持 泛型之前，这些模式对于可用性非常重要。James Gosling 和其他的Java设计人员，交换了一些静态安全性和性能， 因为运行时候的类型检查需要一些时间，但是提高了代码的灵活性。


There are few modern statically typed languages that don’t make that trade-off somewhere. Even Haskell will let you run code with non-exhaustive matches. If you find yourself designing a statically typed language, keep in mind that you can sometimes give users more flexibility without sacrificing too many of the benefits of static safety by deferring some type checks until runtime.

On the other hand, a key reason users choose statically typed languages is because of the confidence the language gives them that certain kinds of errors can never occur when their program is run. Defer too many type checks until runtime, and you erode that confidence.

很少有现代的静态类型语言，在某些方面不做出这样的权衡。甚至，Haskell也允许我们使用非穷尽匹配来运行代码。如果，你在设计静态语言。请记住，通过将某些类型检查推迟到运行时进行，有时候可以给用户提供更高的灵活性，而且不会牺牲太多的静态安全性能。

另一方面，用户选择静态类型语言的一个关键原因是，这种语言让他们相信，运行程序时候，永远不会发生类型的错误。如果将过多的类型检查推迟到运行时，将会减少用户的信心。
