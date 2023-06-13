# 类



> One has no right to love or hate anything if one has not acquired a thorough knowledge of its nature. Great love springs from great knowledge of the beloved object, and if you know it but little you will be able to love it only a little or not at all.
> 
> 如果一个人没有深入了解任何事物的本质，他就无权爱或者恨它。伟大的爱来源于对所爱之物的深刻了解，如果你对它知之甚少，你就只能爱它一点点，或者根本不爱它。
> 
> <p align=right> Leonardo da Vinci (达芬奇) </p>

We’re eleven chapters in, and the interpreter sitting on your machine is nearly a complete scripting language. It could use a couple of built-in data structures like lists and maps, and it certainly needs a core library for file I/O, user input, etc. But the language itself is sufficient. We’ve got a little procedural language in the same vein as BASIC, Tcl, Scheme (minus macros), and early versions of Python and Lua.

我们总共有11章，当前我们实现的解释器，几乎是一种完整的脚本语言。它可以使用一些内置的数据结构，例如: list map, 它当然需要一个用于文件I/O， 用户输入等核心库。但是语言本身就足够了，我们有一种与BASIC、Tcl、Scheme（除去宏）以及早期的Python和Lua 类似的语言了。

If this were the ’80s, we’d stop here. But today, many popular languages support “object-oriented programming”. Adding that to Lox will give users a familiar set of tools for writing larger programs. Even if you personally don’t like OOP, this chapter and the next will help you understand how others design and build object systems.

如果这是80年代，我们就可以完成该语言，但是今天，许多流行语言都支持面向对象编程。将其添加到Lox中，用户可以使用熟悉的工具来编写更强大的程序。即使你可能并不喜欢面向对象编程，本章和下一章，我们将介绍其他人是如何设计和构建对象系统的


> If you really hate classes, though, you can skip these two chapters. They are fairly isolated from the rest of the book. Personally, I find it’s good to learn more about the things I dislike. Things look simple at a distance, but as I get closer, details emerge and I gain a more nuanced perspective.
> 
> 如果你真的讨厌面向对象，那么可以跳过这两章。它们和本书中的其他部分是完全隔离的。就我个人而言，我觉得了解更多我不喜欢的事情也是好的。一件事情，可能从远处看，非常简单，但是，当我靠近时，它的细节将呈现出来，我们将获得更加微妙的视角

## 一、OOP and Classes

面向对象和类

There are three broad paths to object-oriented programming: classes, prototypes, and multimethods. Classes came first and are the most popular style. With the rise of JavaScript (and to a lesser extent Lua), prototypes are more widely known than they used to be. I’ll talk more about those later. For Lox, we’re taking the, ahem, classic approach.

面向对象编程有3种实现方式，类、[原型](http://gameprogrammingpatterns.com/prototype.html)、[多态](https://en.wikipedia.org/wiki/Multiple_dispatch). 类是第一位的，是最流行的方式，随着JavaScript 的兴起（有一部分原因是Lua），原型比以前更广为人知。稍后，我们将详细讨论。对于Lox语言，我们将采用最经典的类方式

Since you’ve written about a thousand lines of Java code with me already, I’m assuming you don’t need a detailed introduction to object orientation. The main goal is to bundle data with the code that acts on it. Users do that by declaring a class that:

既然，我们已经差不多写了1000行Java 代码，我假设大家都不需要介绍基本的面向对象语言了。面向对象的主要目标是，将数据和作用于数据的操作，绑定在一起，用户通过声明一个类实现

1. Exposes a constructor to create and initialize new instances of the class

1. Provides a way to store and access fields on instances

1. Defines a set of methods shared by all instances of the class that operate on each instances’ state.

&nbsp;

1. 构造函数用于创建、初始化一个类的实例

1. 提供存储和访问实例上的字段的方法

1. 定义一组由类的所有实例共享的方法，这些方法根据每个实例的状态，进行不同的操作


That’s about as minimal as it gets. Most object-oriented languages, all the way back to Simula, also do inheritance to reuse behavior across classes. We’ll add that in the next chapter. Even kicking that out, we still have a lot to get through. This is a big chapter and everything doesn’t quite come together until we have all of the above pieces, so gather your stamina.

上面几乎是，最小的面向对象实现。大多数的面向对象语言（一直到Simula），也会支持继承，用于复用祖先类的行为。我们将在下一章中介绍继承。即使本章不包含继承，我们仍然有很多事情需要做。这是一个很大的章节，在我们完成上面的所有内容之前，一切都无法完全融合在一起，所以，我们需要集中精力


> Multimethods are the approach you’re least likely to be familiar with. I’d love to talk more about them—I designed a hobby language around them once and they are super rad—but there are only so many pages I can fit in. If you’d like to learn more, take a look at CLOS (the object system in Common Lisp), Dylan, Julia, or Raku.
> 
> 多态方法，可能是你不熟悉的一种方式，我很想多谈谈它们——我曾经围绕它们设计过一种语言。它们非常棒，但是限于篇幅，我只能介绍这么多。如果你想要了解更多，请看看 [CLOS](https://en.wikipedia.org/wiki/Common_Lisp_Object_System), [opendylan](https://opendylan.org/), [julialang](https://julialang.org/), [raku](https://docs.raku.org/language/functions#Multi-dispatch)



> ![circle](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/circle.png?raw=true)
> 
> It’s like the circle of life, sans Sir Elton John


## 二、Class Declarations

类声明

Like we do, we’re gonna start with syntax. A class statement introduces a new name, so it lives in the declaration grammar rule.

和之前一样，我们将从语法开始，类语句引入了新名称，因此它存在于声明语法规则中

```java

declaration    → classDecl
               | funDecl
               | varDecl
               | statement ;

classDecl      → "class" IDENTIFIER "{" function* "}" ;

```

The new classDecl rule relies on the function rule we defined earlier. To refresh your memory:

新的 classDecl 语法规则，依赖于我们前面定义的函数规则，

```java

function       → IDENTIFIER "(" parameters? ")" block ;
parameters     → IDENTIFIER ( "," IDENTIFIER )* ;

```

In plain English, a class declaration is the class keyword, followed by the class’s name, then a curly-braced body. Inside that body is a list of method declarations. Unlike function declarations, methods don’t have a leading fun keyword. Each method is a name, parameter list, and body. Here’s an example:

在英语中，class 单词声明了类关键字，后面跟随者类名称，然后是类的主体。在该主体中，是方法声明列表。和函数声明不同的是，方法没有前缀的fun 关键字，每个方法都有一个方法名称、方法参数列表和方法主体。下面是一个示例：

```java

class Breakfast {
  cook() {
    print "Eggs a-fryin'!";
  }

  serve(who) {
    print "Enjoy your breakfast, " + who + ".";
  }
}

```

Like most dynamically typed languages, fields are not explicitly listed in the class declaration. Instances are loose bags of data and you can freely add fields to them as you see fit using normal imperative code.

和大多数的动态类型语言一样，类声明中没有列出类字段声明，实例是松散的数据，我们可以使用普通的命令式代码，在合适的情况下，添加实例字段。

Over in our AST generator, the classDecl grammar rule gets its own statement node.

在AST 生成器中，classDecl 语法规则获得自己的语句节点

```java

// tool/GenerateAst.java, in main()

      "Block      : List<Stmt> statements",
      "Class      : Token name, List<Stmt.Function> methods",
      "Expression : Expr expression",



```

It stores the class’s name and the methods inside its body. Methods are represented by the existing Stmt.Function class that we use for function declaration AST nodes. That gives us all the bits of state that we need for a method: name, parameter list, and body.

上面的生成器，将类的名称和方法，存储在主体中，方法由我们用于函数声明AST 节点的现有Stmt.Function 类表示，这为我们提供了方法所需的所有状态信息: 名称，参数，主体

A class can appear anywhere a named declaration is allowed, triggered by the leading class keyword.

一个类，可以出现在任何允许命名声明的地方，由关键字 class 开始

```java

// lox/Parser.java, in declaration()

    try {
      if (match(CLASS)) return classDeclaration();
      if (match(FUN)) return function("function");


```

That calls out to:

```java

// lox/Parser.java, add after declaration()

  private Stmt classDeclaration() {
    Token name = consume(IDENTIFIER, "Expect class name.");
    consume(LEFT_BRACE, "Expect '{' before class body.");

    List<Stmt.Function> methods = new ArrayList<>();
    while (!check(RIGHT_BRACE) && !isAtEnd()) {
      methods.add(function("method"));
    }

    consume(RIGHT_BRACE, "Expect '}' after class body.");

    return new Stmt.Class(name, methods);
  }
	
```

There’s more meat to this than most of the other parsing methods, but it roughly follows the grammar. We’ve already consumed the class keyword, so we look for the expected class name next, followed by the opening curly brace. Once inside the body, we keep parsing method declarations until we hit the closing brace. Each method declaration is parsed by a call to function(), which we defined back in the chapter where functions were introduced.

和大多数的其他解析方法相比，该解析方法有更多的内容，但是它大致遵循语法规则。我们已经先消费了class 关键字，接下来，我们期望获取到类名。然后是类主体的开始大括号，一旦进入类主体，我们开始继续解析每个类中的方法，直到到达右大括号。每个方法的声明都是通过调用 function() 来解析，我们在介绍函数的章节中定义了该函数。

Like we do in any open-ended loop in the parser, we also check for hitting the end of the file. That won’t happen in correct code since a class should have a closing brace at the end, but it ensures the parser doesn’t get stuck in an infinite loop if the user has a syntax error and forgets to correctly end the class body.

就像我们在解析器中，执行的循环那样，我们还会检查是否已经到达文件结尾。在正常的类声明中，结尾会有一个右括号，但是如果用户出现语法错误，该检查可以防止我们的解析器，无限循环。

We wrap the name and list of methods into a Stmt.Class node and we’re done. Previously, we would jump straight into the interpreter, but now we need to plumb the node through the resolver first.

我们将一系列的方法和类名称，包装到Stmt.Class 节点中。如果是以前，接下来，我们将会直接跳转到解释器中，但是，现在，我们需要首先通过变量解析器来检查节点。

```java

// lox/Resolver.java, add after visitBlockStmt()

  @Override
  public Void visitClassStmt(Stmt.Class stmt) {
    declare(stmt.name);
    define(stmt.name);
    return null;
  }
	
```


We aren’t going to worry about resolving the methods themselves yet, so for now all we need to do is declare the class using its name. It’s not common to declare a class as a local variable, but Lox permits it, so we need to handle it correctly.


我们不需要担心方法本身的解析问题，现在，我们需要做的是使用类的名称来声明类。将类声明为一个局部变量并不常见，但是Lox允许我们这样做，因此我们需要正确的处理它。

Now we interpret the class declaration.

现在，我们将开始解释类声明

```java

  @Override
  public Void visitClassStmt(Stmt.Class stmt) {
    environment.define(stmt.name.lexeme, null);
    LoxClass klass = new LoxClass(stmt.name.lexeme);
    environment.assign(stmt.name, klass);
    return null;
  }


```

This looks similar to how we execute function declarations. We declare the class’s name in the current environment. Then we turn the class syntax node into a LoxClass, the runtime representation of a class. We circle back and store the class object in the variable we previously declared. That two-stage variable binding process allows references to the class inside its own methods

这看起来类似于执行函数声明的方式，我们在当前的环境中声明类的名称，然后，我们将类语法节点，转换为LoxClass，即类的运行时表示。我们返回并且将类对象存储在之前声明的变量中。两阶段变量绑定，允许在类的自身方法中引用类本身。

We will refine it throughout the chapter, but the first draft of LoxClass looks like this:

我们将在整个章节中，对其进行改进，但是，最初的LoxClass 如下所示

```java

// lox/LoxClass.java, create new file

package com.craftinginterpreters.lox;

import java.util.List;
import java.util.Map;

class LoxClass {
  final String name;

  LoxClass(String name) {
    this.name = name;
  }

  @Override
  public String toString() {
    return name;
  }
}

```


Literally a wrapper around a name. We don’t even store the methods yet. Not super useful, but it does have a toString() method so we can write a trivial script and test that class objects are actually being parsed and executed.

LoxClass中，只是包装了名称字段，我们甚至还没有开始存储这些方法，虽然不是很有用，但是它确实有一个 toString() 方法，因此，我们可以编写一个简单的脚本，并且测试类对象是否真正被解析和执行。

```java


class DevonshireCream {
  serveOn() {
    return "Scones";
  }
}

print DevonshireCream; // Prints "DevonshireCream".

```


## 三、Creating Instances


创建实例

We have classes, but they don’t do anything yet. Lox doesn’t have “static” methods that you can call right on the class itself, so without actual instances, classes are useless. Thus instances are the next step.

我们现在已经有 类了，但是它们还什么都没有做。Lox没有可以直接在类本身调用的静态方法，因此没有类的实例，类是无用的，因此，实例是我们的下一步工作。

While some syntax and semantics are fairly standard across OOP languages, the way you create new instances isn’t. Ruby, following Smalltalk, creates instances by calling a method on the class object itself, a recursively graceful approach. Some, like C++ and Java, have a new keyword dedicated to birthing a new object. Python has you “call” the class itself like a function. (JavaScript, ever weird, sort of does both.)

虽然一些语法和语义在OOP 语言中是相当标准的，但是我们创建新实例的方式却不是。Ruby 遵循着Smalltalk，通过调用类对象本身的方法来创建实例，这是一种优雅递归的方法，一些，例如，C++ 和 Java，有一个新的关键字专门用于生成一个新对象，Python 让您像调用函数一样，调用类本身。（JavaScript很奇怪，两者都有）

I took a minimal approach with Lox. We already have class objects, and we already have function calls, so we’ll use call expressions on class objects to create new instances. It’s as if a class is a factory function that generates instances of itself. This feels elegant to me, and also spares us the need to introduce syntax like new. Therefore, we can skip past the front end straight into the runtime.

我对Lox采用了最小的方法，我们已经有了类对象，并且，我们也有了函数调用，所以，我们将对类对象使用调用表达式的方式来创建新的实例。这就好像是，一个类是一个生成自身实例的工厂函数。这让我觉得很优雅，也让我们不需要引入像 new 这样的语法，因此，我们可以跳过前端直接进入运行时

Right now, if you try this:

```java


class Bagel {}
Bagel();

```


You get a runtime error. visitCallExpr() checks to see if the called object implements LoxCallable and reports an error since LoxClass doesn’t. Not yet, that is.

你将会得到一个运行时错误，visitCallExpr() 将会检查被调用对象是否实现了 LoxCallable, 并且报告错误，因为LoxClass 还没有实现，现在还没有

```java

// lox/LoxClass.java, replace 1 line

import java.util.Map;

class LoxClass implements LoxCallable {
  final String name;


```

Implementing that interface requires two methods.

实现该接口需要两种方法

```java

// lox/LoxClass.java, add after toString()

  @Override
  public Object call(Interpreter interpreter,
                     List<Object> arguments) {
    LoxInstance instance = new LoxInstance(this);
    return instance;
  }

  @Override
  public int arity() {
    return 0;
  }
	
```


The interesting one is call(). When you “call” a class, it instantiates a new LoxInstance for the called class and returns it. The arity() method is how the interpreter validates that you passed the right number of arguments to a callable. For now, we’ll say you can’t pass any. When we get to user-defined constructors, we’ll revisit this.

有趣的是call() ，当我们调用一个类时候，它会为被调用的类，实例化一个新的LoxInstance 并且返回它，arity() 方法用于，解释程序如何验证你是否向可调用对象传递了正确数量的参数，现在，我们的arity() 表示，该调用不需要传递任何参数。当我们使用用户自定义的构造函数时候，我们将重新讨论这个问题

That leads us to LoxInstance, the runtime representation of an instance of a Lox class. Again, our first implementation starts small.

这将我们引入LoxInstance， 即Lox类实例的运行时表示，同样，我们的第一个实现是简单的


```java

// lox/LoxInstance.java, create new file

package com.craftinginterpreters.lox;

import java.util.HashMap;
import java.util.Map;

class LoxInstance {
  private LoxClass klass;

  LoxInstance(LoxClass klass) {
    this.klass = klass;
  }

  @Override
  public String toString() {
    return klass.name + " instance";
  }
}

```

Like LoxClass, it’s pretty bare bones, but we’re only getting started. If you want to give it a try, here’s a script to run:

像是LoxClass， 它是非常简单的，但我们才刚刚开始，如果你想要尝试一下，下面的脚本可以运行

```java

class Bagel {}
var bagel = Bagel();
print bagel; // Prints "Bagel instance".

```

This program doesn’t do much, but it’s starting to do something.

这个程序做的不多，但是它，已经开始做一些事情

## 四、Properties on Instances

实例上的属性

We have instances, so we should make them useful. We’re at a fork in the road. We could add behavior first—methods—or we could start with state—properties. We’re going to take the latter because, as we’ll see, the two get entangled in an interesting way and it will be easier to make sense of them if we get properties working first.

我们有实例，接下来我们将使用它。当前我们走在岔路口，我们可以添加行为优先方法，也可以从状态属性开始，我们将采用后者，因为正如我们将要看到的一样，两者以一种有趣的方式纠缠在一起，如果我们先让属性工作，更容易理解它们。

Lox follows JavaScript and Python in how it handles state. Every instance is an open collection of named values. Methods on the instance’s class can access and modify properties, but so can outside code. Properties are accessed using a . syntax.

Lox在处理状态方面，遵循着JavaScript 和 Python的方式，每个实例都是命名值的开放集合。实例类上的方法可以访问和修改属性，但是外部代码也可以。实例属性可以通过 点语法访问

```java

someObject.someProperty

```

An expression followed by . and an identifier reads the property with that name from the object the expression evaluates to. That dot has the same precedence as the parentheses in a function call expression, so we slot it into the grammar by replacing the existing call rule with:

上面的表达式，将从对象中读取具有该名称的属性。这个点，和函数的调用表达式中的括号具有相同的优先级，因此，我们通过将现有的调用规则修改为

```java

call           → primary ( "(" arguments? ")" | "." IDENTIFIER )* ;

```

After a primary expression, we allow a series of any mixture of parenthesized calls and dotted property accesses. “Property access” is a mouthful, so from here on out, we’ll call these “get expressions”.

在主表达式之后，我们允许一系列带有括号的调用和带点属性访问的混合，属性访问不是很容易理解，所以从现在开始，我们将这些称为 get表达式


### 4.1 Get expressions

get 表达式

The syntax tree node is:

语法树节点如下

Following the grammar, the new parsing code goes in our existing call() method.

按照最新的语法，新的解析代码将变更为

```java

// lox/Parser.java, in call()

    while (true) { 
      if (match(LEFT_PAREN)) {
        expr = finishCall(expr);
      } else if (match(DOT)) {
        Token name = consume(IDENTIFIER,
            "Expect property name after '.'.");
        expr = new Expr.Get(expr, name);
      } else {
        break;
      }
    }


```

The outer while loop there corresponds to the * in the grammar rule. We zip along the tokens building up a chain of calls and gets as we find parentheses and dots, like so:

外部的while 循环对应于语法规则中的 *， 我们将沿着 token快速移动，建立一个调用链，并且在找到括号和点时候，获取到调用，如下图所示

![zip](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/zip.png?raw=true)

Instances of the new Expr.Get node feed into the resolver.

Expr.Get 的实例，同样，在变量解析器中也要被解析

```java

// lox/Resolver.java, add after visitCallExpr()

  @Override
  public Void visitGetExpr(Expr.Get expr) {
    resolve(expr.object);
    return null;
  }
	
```

OK, not much to that. Since properties are looked up dynamically, they don’t get resolved. During resolution, we recurse only into the expression to the left of the dot. The actual property access happens in the interpreter.

好吧，不多，由于属性是动态查找的，因此无法解析。在解析过程中，我们只会递归到左侧的表达式，实际的属性访问发生在解释器中

```java

// lox/Interpreter.java, add after visitCallExpr()

  @Override
  public Object visitGetExpr(Expr.Get expr) {
    Object object = evaluate(expr.object);
    if (object instanceof LoxInstance) {
      return ((LoxInstance) object).get(expr.name);
    }

    throw new RuntimeError(expr.name,
        "Only instances have properties.");
  }


```

First, we evaluate the expression whose property is being accessed. In Lox, only instances of classes have properties. If the object is some other type like a number, invoking a getter on it is a runtime error.

首先，我们计算要访问其属性的表达式，在Lox中，只有类的实例具有属性，如果对象是其他类型（例如：数字），则对其调用属性，是一个运行时错误

If the object is a LoxInstance, then we ask it to look up the property. It must be time to give LoxInstance some actual state. A map will do fine.

如果对象是一个LoxInstance ，那么我们要找到它的属性，现在必须给LoxInstance 一些实际的状态，一个map将会很好的定义

```java

// lox/LoxInstance.java, in class LoxInstance

  private LoxClass klass;
  private final Map<String, Object> fields = new HashMap<>();

  LoxInstance(LoxClass klass) {
	
```

Each key in the map is a property name and the corresponding value is the property’s value. To look up a property on an instance:

map中的每一个键都是属性名称，对应的值是属性的值，要查找实例的属性，需要执行下面的操作

```java

// lox/LoxInstance.java, add after LoxInstance()

  Object get(Token name) {
    if (fields.containsKey(name.lexeme)) {
      return fields.get(name.lexeme);
    }

    throw new RuntimeError(name, 
        "Undefined property '" + name.lexeme + "'.");
  }
	
```

An interesting edge case we need to handle is what happens if the instance doesn’t have a property with the given name. We could silently return some dummy value like nil, but my experience with languages like JavaScript is that this behavior masks bugs more often than it does anything useful. Instead, we’ll make it a runtime error.


我们需要处理的一个有趣的边缘情况是，如果实例，没有具有给定名称的属性，会发生什么？我们可以默认返回一些空值，例如：nil，但是据我使用JavaScript等语言的经验，这种行为可以会掩盖更多的错误，所以，我们将在该场景下，抛出一个运行时错误

So the first thing we do is see if the instance actually has a field with the given name. Only then do we return it. Otherwise, we raise an error.

因此，我们要做的第一件事情是查看实例中，是否有一个具有给定名称的字段，只有这样，我们才能返回属性值。否则，我们将抛出运行时错误

Note how I switched from talking about “properties” to “fields”. There is a subtle difference between the two. Fields are named bits of state stored directly in an instance. Properties are the named, uh, things, that a get expression may return. Every field is a property, but as we’ll see later, not every property is a field.

请注意，我们如何在属性和 fields之间切换的，两者之间存在着细微的差别。fields 是直接存储在实例中的状态信息，属性则是get 表达式可能返回的命名内容，每个field 都是一个属性，但是正如我们将要看到的，并不是每个属性都是一个field


In theory, we can now read properties on objects. But since there’s no way to actually stuff any state into an instance, there are no fields to access. Before we can test out reading, we must support writing.

理论上，我们现在可以读取对象的属性，但是由于无法将任何状态实际填充到实例中，因此没有可以访问的字段，在我们测试读取之前，我们必须支持写入

### 4.2 Set expressions

Set 表达式


Setters use the same syntax as getters, except they appear on the left side of an assignment.

setters 使用与getter 相同的语法，只是它们出现在赋值语句的左侧

```java

someObject.someProperty = value;

```

In grammar land, we extend the rule for assignment to allow dotted identifiers on the left-hand side.

在语法领域，我们扩展了赋值规则，允许在左侧使用点符号


```java


assignment     → ( call "." )? IDENTIFIER "=" assignment
               | logic_or ;
							 

```


Unlike getters, setters don’t chain. However, the reference to call allows any high-precedence expression before the last dot, including any number of getters, as in:

和 getters 不同，setters 不会及联。但是，对调用的引用允许在最后一个点符号之前使用任何高优先级的表达式，包括任意数量的getter ，例如:

![setters](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/setter.png?raw=true)

Note here that only the last part, the .meat is the setter. The .omelette and .filling parts are both get expressions.

请注意，只有最后的一部分， .meat 是一个setter, 其他之前的部分都是 getter

Just as we have two separate AST nodes for variable access and variable assignment, we need a second setter node to complement our getter node.

正如我们有两个单独的AST 节点用于变量访问和变量分配一样，我们需要第二个setter节点，来补充getter 节点

```java

// tool/GenerateAst.java, in main()

      "Logical  : Expr left, Token operator, Expr right",
      "Set      : Expr object, Token name, Expr value",
      "Unary    : Token operator, Expr right",
			
```


In case you don’t remember, the way we handle assignment in the parser is a little funny. We can’t easily tell that a series of tokens is the left-hand side of an assignment until we reach the =. Now that our assignment grammar rule has call on the left side, which can expand to arbitrarily large expressions, that final = may be many tokens away from the point where we need to know we’re parsing an assignment.

如果你不记得，我们在解析器中处理赋值的方式有些技巧，在到达 = 之前，我们无法轻易判断一系列token 是在赋值的左侧，现在我们的赋值语法规则在左侧调用了，它可以扩展到任意大的表达式， 最终的= 可能离我们正在解析的点非常远

Instead, the trick we do is parse the left-hand side as a normal expression. Then, when we stumble onto the equal sign after it, we take the expression we already parsed and transform it into the correct syntax tree node for the assignment.

相反，我们所做的技巧是将左侧当作一个简单的表达式解析，然后，当我们偶然发现它的后面是一个等号时候，我们使用已经解析的表达式，并将其转换为用于赋值的正确语法树节点

We add another clause to that transformation to handle turning an Expr.Get expression on the left into the corresponding Expr.Set.

我们将该转换添加另一个子句，以处理将左侧的Expr.Get 表达式转换为相应的 Expr.Set


```java

// lox/Parser.java, in assignment()

        return new Expr.Assign(name, value);
      } else if (expr instanceof Expr.Get) {
        Expr.Get get = (Expr.Get)expr;
        return new Expr.Set(get.object, get.name, value);
      }
			
```


That’s parsing our syntax. We push that node through into the resolver.

这是在解析我们的语法，我们将该节点加入到变量解析器

```java

// lox/Resolver.java, add after visitLogicalExpr()

  @Override
  public Void visitSetExpr(Expr.Set expr) {
    resolve(expr.value);
    resolve(expr.object);
    return null;
  }


```

Again, like Expr.Get, the property itself is dynamically evaluated, so there’s nothing to resolve there. All we need to do is recurse into the two subexpressions of Expr.Set, the object whose property is being set, and the value it’s being set to.

同样，像 Expr.Get 一样，属性本身是动态求值的，因此没有什么要解决的，我们所需要做的就是递归到 Expr.Set 的两个子表达式中，即其属性被设置的对象和其设置的值

That leads us to the interpreter.

接下来，我们将开始解释器的更新

```java

// lox/Interpreter.java, add after visitLogicalExpr()

  @Override
  public Object visitSetExpr(Expr.Set expr) {
    Object object = evaluate(expr.object);

    if (!(object instanceof LoxInstance)) { 
      throw new RuntimeError(expr.name,
                             "Only instances have fields.");
    }

    Object value = evaluate(expr.value);
    ((LoxInstance)object).set(expr.name, value);
    return value;
  }
	
```

We evaluate the object whose property is being set and check to see if it’s a LoxInstance. If not, that’s a runtime error. Otherwise, we evaluate the value being set and store it on the instance. That relies on a new method in LoxInstance.

我们评估其属性被设置的对象，检查其是否是 LoxInstance, 如果不是，抛出一个运行时错误，否则，我们将计算要设置的值，并将其保存在实例上，这依赖 LoxInstance中的一个新方法

```java

// lox/LoxInstance.java, add after get()

  void set(Token name, Object value) {
    fields.put(name.lexeme, value);
  }
	
```

No real magic here. We stuff the values straight into the Java map where fields live. Since Lox allows freely creating new fields on instances, there’s no need to see if the key is already present.

这里没有真正的魔法，我们将值直接填充到字段所在的Java 映射中，由于Lox允许在实例上自由的创建新字段，因此无需查看该键是否已经存在

## 五、Methods on Classes

类上的方法

You can create instances of classes and stuff data into them, but the class itself doesn’t really do anything. Instances are just maps and all instances are more or less the same. To make them feel like instances of classes, we need behavior—methods.

你可以创建类的实例，并将数据填充到实例中，但是类本身并没有真正做任何事情，实例只是映射，所有实例或多或少都是相同的，为了让它们看起来像类的实例，我们需要行为方法

Our helpful parser already parses method declarations, so we’re good there. We also don’t need to add any new parser support for method calls. We already have . (getters) and () (function calls). A “method call” simply chains those together

我们的解析器，已经解析了方法声明，我们不需要添加新的解析器，去解析方法调用。一个方法调用只是将 getters 和 () 结合在一起

![method](https://github.com/Kua-Fu/blog-book-images/blob/main/method.png?raw=true)

That raises an interesting question. What happens when those two expressions are pulled apart? Assuming that method in this example is a method on the class of object and not a field on the instance, what should the following piece of code do?

上面场景，提出了一个有趣的问题。当这两个表达式分开时，会发生什么？假设本例中的方法是对象上的方法，而不是实例上的字段，那么下面的代码应该做什么？


```java

var m = object.method;
m(argument);


```

This program “looks up” the method and stores the result—whatever that is—in a variable and then calls that object later. Is this allowed? Can you treat a method like it’s a function on the instance?

上面的程序查找该方法，并将结果存储在变量中，然后调用该对象。这是被允许的吗？我们可以把一个方法当作实例上的函数吗？

What about the other direction?

```java


class Box {}

fun notMethod(argument) {
  print "called function with " + argument;
}

var box = Box();
box.function = notMethod;
box.function("argument");

```

This program creates an instance and then stores a function in a field on it. Then it calls that function using the same syntax as a method call. Does that work?

上面的程序创建了一个实例，然后将一个函数存储在实例的一个字段上，然后使用与方法调用相同的语法，调用该函数，这是可行的吗？


Different languages have different answers to these questions. One could write a treatise on it. For Lox, we’ll say the answer to both of these is yes, it does work. We have a couple of reasons to justify that. For the second example—calling a function stored in a field—we want to support that because first-class functions are useful and storing them in fields is a perfectly normal thing to do.

不同的语言对于这些问题，有不同的答案，一个人可以写一篇关于它的论文。对于Lox，我们会说这两个问题的答案是肯定的，我们允许这样的语法。下面是几个理由：

对于调用存储在字段中的函数的第二个示例，我们希望支持这一点，因为第一类函数非常有用。将它们存储在字段是非常正常的事情

The first example is more obscure. One motivation is that users generally expect to be able to hoist a subexpression out into a local variable without changing the meaning of the program. You can take this:

第一个例子，更加晦涩难懂。一个动机是，用户通常希望能够在不改变程序含义的情况下，将子表达式提升为局部变量。我们可以这样做

```java

breakfast(omelette.filledWith(cheese), sausage);

```

And turn it into this:

```java

var eggs = omelette.filledWith(cheese);
breakfast(eggs, sausage);

```


And it does the same thing. Likewise, since the . and the () in a method call are two separate expressions, it seems you should be able to hoist the lookup part into a variable and then call it later. We need to think carefully about what the thing you get when you look up a method is, and how it behaves, even in weird cases like:

它也会做同样的事情，因为 . 和 () 在一个方法调用中，是不同的表达式，似乎我们应该能够将要查找的部分提升到一个变量中，然后调用它，我们需要仔细考虑一下，当查找一个方法时候，得到的结果是什么？以及方法的行为，即使在像这样的奇怪情况下

```java

class Person {
  sayName() {
    print this.name;
  }
}

var jane = Person();
jane.name = "Jane";

var method = jane.sayName;
method(); // ?

```

If you grab a handle to a method on some instance and call it later, does it “remember” the instance it was pulled off from? Does this inside the method still refer to that original object?

如果你在某个实例上获取一个方法，然后在调用它，它是否记得是从哪个实例中获取的呢？方法中的引用是否是原始的对象吗？

Here’s a more pathological example to bend your brain:

下面有一个更加奇怪的示例

```java

class Person {
  sayName() {
    print this.name;
  }
}

var jane = Person();
jane.name = "Jane";

var bill = Person();
bill.name = "Bill";

bill.sayName = jane.sayName;
bill.sayName(); // ?

```

Does that last line print “Bill” because that’s the instance that we called the method through, or “Jane” because it’s the instance where we first grabbed the method?

最后一行输出的是 "Bill" 因为我们调用的是实例方法；或者，最后一行输出是 "Jane", 因为它是我们第一次获取的方法，对应的实例呢？


Equivalent code in Lua and JavaScript would print “Bill”. Those languages don’t really have a notion of “methods”. Everything is sort of functions-in-fields, so it’s not clear that jane “owns” sayName any more than bill does.

Lua 和JavaScript 中的相同代码，将会输出 "Bill", 这些语言实际上没有方法的概念，一切都是字段中的函数，所以不清楚jane 是否比 bill，更加拥有sayName

Lox, though, has real class syntax so we do know which callable things are methods and which are functions. Thus, like Python, C#, and others, we will have methods “bind” this to the original instance when the method is first grabbed. Python calls these bound methods.

不过，Lox有真正的类语法，所以我们知道哪些是方法，哪些是函数，因此，与Python 、C#和其他类似语言，我们将有对应的方式，在首次抓取方法时候，将其绑定在原始实例上。Python中称呼这些为，绑定方法

In practice, that’s usually what you want. If you take a reference to a method on some object so you can use it as a callback later, you want to remember the instance it belonged to, even if that callback happens to be stored in a field on some other object.

实际上，这通常是我们想要的结果。如果我们引用某个对象上的某个方法，以便以后可以将其作为回调，则需要记住它所属的实例。即使该回调恰好存储在某个其他对象的字段中

OK, that’s a lot of semantics to load into your head. Forget about the edge cases for a bit. We’ll get back to those. For now, let’s get basic method calls working. We’re already parsing the method declarations inside the class body, so the next step is to resolve them.

好的，这是很多需要记录在大脑中的新的语义。暂时忘掉上面的边缘案例，我们会回到一开始，现在，让我们从基本的调用开始，我们已经解析类主体中的方法声明，因此下一步，我们将解析它们

```java

// lox/Resolver.java, in visitClassStmt()

    define(stmt.name);

    for (Stmt.Function method : stmt.methods) {
      FunctionType declaration = FunctionType.METHOD;
      resolveFunction(method, declaration); 
    }

    return null;
		
```

We iterate through the methods in the class body and call the resolveFunction() method we wrote for handling function declarations already. The only difference is that we pass in a new FunctionType enum value.

我们遍历主体中的方法，并且调用我们为处理函数声明，编写的resolveFunction() 方法。唯一的区别是我们传入了一个新的 FunctionType 枚举值



```java

// lox/Resolver.java, in enum FunctionType, add “,” to previous line

    NONE,
    FUNCTION,
    METHOD
  }
	
```

That’s going to be important when we resolve this expressions. For now, don’t worry about it. The interesting stuff is in the interpreter.

当我们解析这个表达式时候，这将非常重要，现在，还不用担心。有趣的东西在解释器中

```java

// lox/Interpreter.java, in visitClassStmt(), replace 1 line

    environment.define(stmt.name.lexeme, null);

    Map<String, LoxFunction> methods = new HashMap<>();
    for (Stmt.Function method : stmt.methods) {
      LoxFunction function = new LoxFunction(method, environment);
      methods.put(method.name.lexeme, function);
    }

    LoxClass klass = new LoxClass(stmt.name.lexeme, methods);
    environment.assign(stmt.name, klass);
		
```

When we interpret a class declaration statement, we turn the syntactic representation of the class—its AST node—into its runtime representation. Now, we need to do that for the methods contained in the class as well. Each method declaration blossoms into a LoxFunction object

当我们的解释器，解释一个类声明语句时候，我们将类的语法表示（AST 节点）转换为其运行时的表示，现在，我们也需要对类中的方法执行该操作，每个方法声明都会扩展到一个LoxFunction 对象中

We take all of those and wrap them up into a map, keyed by the method names. That gets stored in LoxClass.

我们将所有的这些都打包到一个映射中，映射中的keys 是方法的名称，它存储在LoxClass 中

```java

// lox/LoxClass.java, in class LoxClass, replace 4 lines

  final String name;
  private final Map<String, LoxFunction> methods;

  LoxClass(String name, Map<String, LoxFunction> methods) {
    this.name = name;
    this.methods = methods;
  }

  @Override
  public String toString() {
	
```


Where an instance stores state, the class stores behavior. LoxInstance has its map of fields, and LoxClass gets a map of methods. Even though methods are owned by the class, they are still accessed through instances of that class.

一个实例中会保存实例状态信息，一个类同时会保存类行为，LoxInstance 中有字段映射，LoxClass 中有方法映射，即使方法属于该类，它们仍然可以通过类的实例访问

```java

// lox/LoxInstance.java, in get()

  Object get(Token name) {
    if (fields.containsKey(name.lexeme)) {
      return fields.get(name.lexeme);
    }

    LoxFunction method = klass.findMethod(name.lexeme);
    if (method != null) return method;

    throw new RuntimeError(name, 
        "Undefined property '" + name.lexeme + "'.");
				
```


When looking up a property on an instance, if we don’t find a matching field, we look for a method with that name on the instance’s class. If found, we return that. This is where the distinction between “field” and “property” becomes meaningful. When accessing a property, you might get a field—a bit of state stored on the instance—or you could hit a method defined on the instance’s class.

在实例上查找属性时候，如果找不到匹配的字段，则在实例的类中查找具有该名称的方法，如果找到，我们会返回该方法，这就是字段和属性之间的区别，变得有意义的地方，当访问属性时候，可能会得到一个字段（存储在实例上的状态），也可能会碰到在实例所属的类上定义的方法

The method is looked up using this:

查找类定义的方法如下

```java

// lox/LoxClass.java, add after LoxClass()

  LoxFunction findMethod(String name) {
    if (methods.containsKey(name)) {
      return methods.get(name);
    }

    return null;
  }


```

You can probably guess this method is going to get more interesting later. For now, a simple map lookup on the class’s method table is enough to get us started. Give it a try:


你可能会猜到，这种方法后面会变得更有意思，现在，对类的方法表进行简单的映射查找就足以让我们开始，试试

```java

class Bacon {
  eat() {
    print "Crunch crunch crunch!";
  }
}

Bacon().eat(); // Prints "Crunch crunch crunch!".

```

## 六、This

We can define both behavior and state on objects, but they aren’t tied together yet. Inside a method, we have no way to access the fields of the “current” object—the instance that the method was called on—nor can we call other methods on that same object.

我们可以定义对象的行为和状态，但它们还没有绑定在一起，在方法内部，我们无法访问当前的对象（该方法调用的实例）中的字段，也无法在同一个对象上调用其他方法

To get at that instance, it needs a name. Smalltalk, Ruby, and Swift use “self”. Simula, C++, Java, and others use “this”. Python uses “self” by convention, but you can technically call it whatever you like.

要获取该实例，它需要一个名称，Smalltalk,Ruby, Swift 使用 self, Simula、C++、Java等使用 this，Python按照惯例使用self, 但是从技术上讲，我们可以随意的调用它

For Lox, since we generally hew to Java-ish style, we’ll go with “this”. Inside a method body, a this expression evaluates to the instance that the method was called on. Or, more specifically, since methods are accessed and then invoked as two steps, it will refer to the object that the method was accessed from.

对于Lox，由于我们通常遵循着Java风格，我们将使用this 。在方法内部，this 表达式的计算结果是调用该方法的实例，或者，更具体的说，由于方法是通过两个步骤访问和调用的，因此，它将引用从中访问该方法的对象

That makes our job harder. Peep at:

这将让我们的解释器工作更加困难

```java

class Egotist {
  speak() {
    print this;
  }
}

var method = Egotist().speak;
method();

```

On the second-to-last line, we grab a reference to the speak() method off an instance of the class. That returns a function, and that function needs to remember the instance it was pulled off of so that later, on the last line, it can still find it when the function is called.


在倒数第二行，我们从类的实例中获取对 speak() 方法的引用，这将返回一个函数，该函数需要记住它所属的实例，以便于稍后在最后一行，调用该函数时候，我们仍然可以找到实例

We need to take this at the point that the method is accessed and attach it to the function somehow so that it stays around as long as we need it to. Hmm . . . a way to store some extra data that hangs around a function, eh? That sounds an awful lot like a closure, doesn’t it?

我们需要在方法被访问的时候，将它添加到函数上，这样它就可以在我们需要的时候一直存在。一种存储函数周围附加数据的方法，对吧？这听起来很像是闭包，不是吗？

If we defined this as a sort of hidden variable in an environment that surrounds the function returned when looking up a method, then uses of this in the body would be able to find it later. LoxFunction already has the ability to hold on to a surrounding environment, so we have the machinery we need.

如果我们将this 定义为在查找方法时候，返回的函数周围的环境中的一种隐藏变量，那么稍后，在正文中使用它们，将能够找到它，LoxFunction已经具备保持周围环境的能力，因此我们拥有所需的功能

Let’s walk through an example to see how it works:

让我们通过一个示例来了解它的工作原理

```java


class Cake {
  taste() {
    var adjective = "delicious";
    print "The " + this.flavor + " cake is " + adjective + "!";
  }
}

var cake = Cake();
cake.flavor = "German chocolate";
cake.taste(); // Prints "The German chocolate cake is delicious!".

```

When we first evaluate the class definition, we create a LoxFunction for taste(). Its closure is the environment surrounding the class, in this case the global one. So the LoxFunction we store in the class’s method map looks like so:

当我们第一次执行类定义时候，我们为taste() 方法创建了一个LoxFunction, 它的闭包是围绕类的环境，在本例中是全局环境，因此，我们存储在类的方法映射中的LoxFunction 如下所示

![closure](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/closure.png?raw=true)


When we evaluate the cake.taste get expression, we create a new environment that binds this to the object the method is accessed from (here, cake). Then we make a new LoxFunction with the same code as the original one but using that new environment as its closure.

当我们计算 cake.taste 的get表达式时候，我们创建了一个新的环境，将其绑定到方法访问的对象（这里是cake），然后，我们使用与原始LoxFunction 相同的代码创建了一个新的LoxFunction ，但使用该新环境作为其闭包

![bound-method](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/bound-method.png?raw=true)


This is the LoxFunction that gets returned when evaluating the get expression for the method name. When that function is later called by a () expression, we create an environment for the method body as usual.

这是在计算方法名的get 表达式时候，返回的LoxFunction，当该函数稍后被（）表达式调用时候，我们像往常一样为方法创建一个环境

![call](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/call.png?raw=true)

The parent of the body environment is the environment we created earlier to bind this to the current object. Thus any use of this inside the body successfully resolves to that instance.

当前的环境的父级是我们先前创建的将this 绑定到当前对象的环境，因此，任何this 内部使用的环境，都可以成功被解析为该实例

Reusing our environment code for implementing this also takes care of interesting cases where methods and functions interact, like:

复用我们的环境代码来实现一点，还可以处理方法和函数交互的有趣情况，例如：

```java


class Thing {
  getCallback() {
    fun localFunction() {
      print this;
    }

    return localFunction;
  }
}

var callback = Thing().getCallback();
callback();

```


In, say, JavaScript, it’s common to return a callback from inside a method. That callback may want to hang on to and retain access to the original object—the this value—that the method was associated with. Our existing support for closures and environment chains should do all this correctly.

在JavaScript 中，通常从方法内部返回回调，该回调可能希望挂起并保留对与该方法关联的原始对象 this 值的访问，我们对闭包和环境链的现有支持，应该能够正确的完成所有这些。

Let’s code it up. The first step is adding new syntax for this.

让我们把上面的逻辑编码实现，第一步是为此添加新语法

```java

// tool/GenerateAst.java, in main()

    "Set      : Expr object, Token name, Expr value",
    "This     : Token keyword",
    "Unary    : Token operator, Expr right",


```

Parsing is simple since it’s a single token which our lexer already recognizes as a reserved word.

解析很简单，因为它是一个单独的token，我们的词法解析器，已经将其识别为保留字

```java

// lox/Parser.java, in primary()

      return new Expr.Literal(previous().literal);
    }

    if (match(THIS)) return new Expr.This(previous());

    if (match(IDENTIFIER)) {
		
```


You can start to see how this works like a variable when we get to the resolver.

当我们开始变量解析器时候，可以开始查看它是如何像变量一样工作的

```java

// lox/Resolver.java, add after visitSetExpr()

  @Override
  public Void visitThisExpr(Expr.This expr) {
    resolveLocal(expr, expr.keyword);
    return null;
  }



```



We resolve it exactly like any other local variable using “this” as the name for the “variable”. Of course, that’s not going to work right now, because “this” isn’t declared in any scope. Let’s fix that over in visitClassStmt().

我们的解决方式是，像是其他本地变量一样，使用`this` 当成一个变量的名称，当然，现在直接引用 `this` 不会生效，因为我们还没有声明 `this` 的作用域，让我们在 visitCLassStmt() 函数中进行。

```java

// lox/Resolver.java, in visitClassStmt()

    define(stmt.name);

    beginScope();
    scopes.peek().put("this", true);

    for (Stmt.Function method : stmt.methods) {
	
```

Before we step in and start resolving the method bodies, we push a new scope and define “this” in it as if it were a variable. Then, when we’re done, we discard that surrounding scope.

在我们开始处理方法体之前，我们先创建一个新的作用域，并且在其中定义this (this 就好像是一个变量一样），当我们完成函数解析后，我们将丢弃该作用域

```java

// lox/Resolver.java, in visitClassStmt()

    }

    endScope();

    return null;
	
```

Now, whenever a this expression is encountered (at least inside a method) it will resolve to a “local variable” defined in an implicit scope just outside of the block for the method body.

现在，每当遇到一个 this表达式（至少在方法内部），它将解析为在方法块之外的一个隐式作用域中定义的局部变量

The resolver has a new scope for this, so the interpreter needs to create a corresponding environment for it. Remember, we always have to keep the resolver’s scope chains and the interpreter’s linked environments in sync with each other. At runtime, we create the environment after we find the method on the instance. We replace the previous line of code that simply returned the method’s LoxFunction with this:

解析器为this 创建了一个新的作用域，因此解释器需要为this 创建相应的环境，请记住，我们始终必须保持解析器的作用域链和解释器的环境链，彼此同步。在运行时候，当我们找到实例后的方法后，会创建环境。我们使用this 替换之前的代码行（该代码行返回LoxFunction）

```java

// lox/LoxInstance.java, in get(), replace 1 line

    LoxFunction method = klass.findMethod(name.lexeme);
    if (method != null) return method.bind(this);

    throw new RuntimeError(name, 
        "Undefined property '" + name.lexeme + "'.");
		
```

Note the new call to bind(). That looks like so:

请注意新的bind() 函数，该函数看起来如下

```java

// lox/LoxFunction.java, add after LoxFunction()

 LoxFunction bind(LoxInstance instance) {
    Environment environment = new Environment(closure);
    environment.define("this", instance);
    return new LoxFunction(declaration, environment);
  }
  
```

There isn’t much to it. We create a new environment nestled inside the method’s original closure. Sort of a closure-within-a-closure. When the method is called, that will become the parent of the method body’s environment.

这并不复杂，我们在方法的原始闭包中，创建了一个新的环境，有点像是闭包中的闭包。当调用方法时候，这将会成为函数的方法体的环境的父级环境。

We declare “this” as a variable in that environment and bind it to the given instance, the instance that the method is being accessed from. Et voilà, the returned LoxFunction now carries around its own little persistent world where “this” is bound to the object.

The remaining task is interpreting those this expressions. Similar to the resolver, it is the same as interpreting a variable expression.

我们在该环境中，将this 声明为一个本地变量，并且将其绑定到给定的实例上，即访问该方法的实例。然后，返回的LoxFunction 将携带持久化的环境，其中this变量绑定为对象实例。

剩下的任务是解释这些this表达式，与解析器相似，它和解释变量表达式相同。

```java

// lox/Interpreter.java, add after visitSetExpr()

 @Override
  public Object visitThisExpr(Expr.This expr) {
    return lookUpVariable(expr.keyword, expr);
  }
  
```

Go ahead and give it a try using that cake example from earlier. With less than twenty lines of code, our interpreter handles this inside methods even in all of the weird ways it can interact with nested classes, functions inside methods, handles to methods, etc.

请尝试使用之前的cake 示例，使用不到20行代码，我们的解释器将可以处理方法内部的this，即使在所有与嵌套类、方法内部的函数、句柄等奇怪的交互方式中，也可以处理。


### 6.1 Invalid uses of this

this的非法用法

Wait a minute. What happens if you try to use this outside of a method? What about:

请稍等，如果尝试在方法外部调用this，会发生什么呢？

```java 

print this;

```

OR

```

fun notAMethod() {
  print this;
}

```

There is no instance for this to point to if you’re not in a method. We could give it some default value like nil or make it a runtime error, but the user has clearly made a mistake. The sooner they find and fix that mistake, the happier they’ll be.

如果不在方法中，this就没有实例可以指向。我们可以给它一些默认值，比如: nil, 或者将其作为运行时报错，但是用户显然是犯了错误。他们越早发现和修复这个错误，将会越开心。

Our resolution pass is a fine place to detect this error statically. It already detects return statements outside of functions. We’ll do something similar for this. In the vein of our existing FunctionType enum, we define a new ClassType one.

我们的解析器，将是检测这种错误的好地方。它已经可以检测到在函数外部的return 语句。我们将为this 执行类似的操作。在我们现有的FunctionType 枚举类型中，我们将定义一个新的 ClassType

```java

// lox/Resolver.java, add after enum FunctionType

 }

  private enum ClassType {
    NONE,
    CLASS
  }

  private ClassType currentClass = ClassType.NONE;

  void resolve(List<Stmt> statements) {
  
```


Yes, it could be a Boolean. When we get to inheritance, it will get a third value, hence the enum right now. We also add a corresponding field, currentClass. Its value tells us if we are currently inside a class declaration while traversing the syntax tree. It starts out NONE which means we aren’t in one.

When we begin to resolve a class declaration, we change that.

是的，它可以是一个布尔值。当我们达到继承时候，我们将获得第三个值，因此现在是枚举。我们还添加了一个相应的字段 currentClass 。当我们在遍历语法树的时候，它的值将告知我们，是否在类声明的内部。它的初始值是None，这意味着我们不在其中。

当我们开始解析类声明时候，我们将会改变该值

```java

// lox/Resolver.java, in visitClassStmt()

  public Void visitClassStmt(Stmt.Class stmt) {
    ClassType enclosingClass = currentClass;
    currentClass = ClassType.CLASS;

    declare(stmt.name);


```

As with currentFunction, we store the previous value of the field in a local variable. This lets us piggyback onto the JVM to keep a stack of currentClass values. That way we don’t lose track of the previous value if one class nests inside another.

Once the methods have been resolved, we “pop” that stack by restoring the old value.

和 currentFunction 一样，我们使用一个局部变量保存该字段的先前值。这使得我们可以利用JVM来保持当前类值的堆栈。这样，如果一个类嵌套在另一个类中，我们也不会失去先前的值。

一旦方法已经解析完成，我们可以通过恢复旧值来弹出该堆栈。

```java

// lox/Resolver.java, in visitClassStmt()

    endScope();

    currentClass = enclosingClass;
    return null;
	
```

When we resolve a this expression, the currentClass field gives us the bit of data we need to report an error if the expression doesn’t occur nestled inside a method body.

当我们解析表达式this 时候，currentClass 字段，为我们提供了所需的数据，以便在表达式未嵌套在方法体时候，抛出错误。


```java

// lox/Resolver.java, in visitThisExpr()

  public Void visitThisExpr(Expr.This expr) {
    if (currentClass == ClassType.NONE) {
      Lox.error(expr.keyword,
          "Can't use 'this' outside of a class.");
      return null;
    }

    resolveLocal(expr, expr.keyword);
	
```

That should help users use this correctly, and it saves us from having to handle misuse at runtime in the interpreter.

这应该有助于用户正确使用this，并且它可以避免我们在解释器运行时报错。

## 七、Constructors and Initializers

构造器和初始化器

We can do almost everything with classes now, and as we near the end of the chapter we find ourselves strangely focused on a beginning. Methods and fields let us encapsulate state and behavior together so that an object always stays in a valid configuration. But how do we ensure a brand new object starts in a good state?

For that, we need constructors. I find them one of the trickiest parts of a language to design, and if you peer closely at most other languages, you’ll see cracks around object construction where the seams of the design don’t quite fit together perfectly. Maybe there’s something intrinsically messy about the moment of birth.

现在，我们几乎可以使用类，来完成所有的事情。在本章接近完成时候，我们发现自己奇怪的关注于起始。方法和字段，让我们将状态和行为封装在一起，以便对象始终有效。但是，我们如何确保一个新的对象，处于一个良好的状态？

我们，我们需要构造函数，我发现它是语言设计最棘手的一部分。如果你仔细观察大多数语言，你会发现在构造函数周围存在着裂缝，设计的裂缝，不完全完美的契合在一起。也许在创造的开始就存在着一些混乱。

> A few examples: In Java, even though final fields must be initialized, it is still possible to read one before it has been. Exceptions—a huge, complex feature—were added to C++ mainly as a way to emit errors from constructors.
> 
> 一些例子，在Java中，即使 final 字段也需要初始化，但是我们仍然可以在初始化之前读取到。
> 
> 异常——一个巨大切复杂的特性，主要是作为从构造函数中发出错误的一种方式，被添加到C++ 中

“Constructing” an object is actually a pair of operations:

1. The runtime allocates the memory required for a fresh instance. In most languages, this operation is at a fundamental level beneath what user code is able to access.


1. Then, a user-provided chunk of code is called which initializes the unformed object.

构造一个对象，实际上一组操作

1. 运行时候，为新实例分配所需的内存，在大多数的编程语言中，这个操作位于用户代码可以访问的基本层次之下。

1. 然后，调用用户提供的一段代码来初始化未成型的对象。

> C++’s [“placement new”](https://en.wikipedia.org/wiki/Placement_syntax) is a rare example where the bowels of allocation are laid bare for the programmer to prod.
>
> C++的 placement new是一个罕见的例子，其中分配的内部细节被暴露

The latter is what we tend to think of when we hear “constructor”, but the language itself has usually done some groundwork for us before we get to that point. In fact, our Lox interpreter already has that covered when it creates a new LoxInstance object.

We’ll do the remaining part—user-defined initialization—now. Languages have a variety of notations for the chunk of code that sets up a new object for a class. C++, Java, and C# use a method whose name matches the class name. Ruby and Python call it init(). The latter is nice and short, so we’ll do that.

In LoxClass’s implementation of LoxCallable, we add a few more lines.

后者是我们在听到构造函数时候，通常会想到的。但是，在我们执行这些之前，编程语言已经为我们完成了一些基本工作。实际上，我们的Lox解释器，创建一个新的LoxInstance 时候，已经涵盖了这一部分。

现在我们将完成剩下的部分——用户定义的初始化，编程语言对于设置类的新对象的代码块有各种表示法。C++ Java 和 C# 使用与类名相匹配的方法名。Ruby 和 Python将其称为 init函数，后者简单明了。所以，我们将采用后一种方式

在LoxClass 实现 LoxCallable的过程中，我们需要添加一些额外的代码行

```java

// lox/LoxClass.java, in call()

                     List<Object> arguments) {
    LoxInstance instance = new LoxInstance(this);
    LoxFunction initializer = findMethod("init");
    if (initializer != null) {
      initializer.bind(instance).call(interpreter, arguments);
    }

    return instance;
	
```

When a class is called, after the LoxInstance is created, we look for an “init” method. If we find one, we immediately bind and invoke it just like a normal method call. The argument list is forwarded along.

That argument list means we also need to tweak how a class declares its arity.

当一个类被调用时，在创建了LoxInstance 之后，我们会寻找一个 init 方法，如果我们找到了一个，我们会立即绑定并且调用它，就像是一个普通的方法调用一样。参数列表也会被传递。

这个参数列表，意味着我们还需要调整类声明其参数个数（arity)的方式。

```java

// lox/LoxClass.java, in arity(), replace 1 line

  public int arity() {
    LoxFunction initializer = findMethod("init");
    if (initializer == null) return 0;
    return initializer.arity();
  }
  
```

If there is an initializer, that method’s arity determines how many arguments you must pass when you call the class itself. We don’t require a class to define an initializer, though, as a convenience. If you don’t have an initializer, the arity is still zero.

That’s basically it. Since we bind the init() method before we call it, it has access to this inside its body. That, along with the arguments passed to the class, are all you need to be able to set up the new instance however you desire.

如果有一个初始化器，那么该方法的参数个数 arity 决定了在调用类本身时，必须传递多少个参数，然而，我们不要求类一定有一个初始化器，这是为了方便，如果没有初始化器，参数个数仍然是0

基本上就是这样，由于我们在调用init() 方法之前，就已经绑定了它，所以，它就可以在其内部访问 this，然后，在加上我们传递给类的参数，这就是我们需要的全部内容，便于根据我们的需求设置新实例

### 7.1 Invoking init() directly

As usual, exploring this new semantic territory rustles up a few weird creatures. Consider:

像往常一样，探索新的语义领域将引出一些奇怪的现象，请考虑：

```java

class Foo {
  init() {
    print this;
  }
}

var foo = Foo();
print foo.init();


```

Can you “re-initialize” an object by directly calling its init() method? If you do, what does it return? A reasonable answer would be nil since that’s what it appears the body returns.

However—and I generally dislike compromising to satisfy the implementation—it will make clox’s implementation of constructors much easier if we say that init() methods always return this, even when directly called. In order to keep jlox compatible with that, we add a little special case code in LoxFunction.

可以通过直接调用类的init方法，重新初始化一个对象吗？如果可以调用，它将返回什么呢？一个合理的答案是nil，因为从表面上看，返回体的返回值就是 nil

然而，尽管我通常不喜欢为了满足实现而妥协。但是，如果我们设计，init() 方法总是返回this，即使直接调用，那么在clox 中实现构造函数会变得容易的多。为了保持jlox 与之兼容，我们将在LoxFunciton中添加一个特殊情况的代码

> Maybe “dislike” is too strong a claim. It’s reasonable to have the constraints and resources of your implementation affect the design of the language. There are only so many hours in the day, and if a cut corner here or there lets you get more features to users in less time, it may very well be a net win for their happiness and productivity. The trick is figuring out which corners to cut that won’t cause your users and future self to curse your shortsightedness.
> 
> 也许不喜欢，这个说法有点强烈。让实现的约束和资源，影响语言设计是合理的。一天中只有这么多的小时，如果在某个地方稍微简化一下，可以让你在更短的时间内为用户提供更多的功能，那么这可能对于他们的幸福和生产力来说，是一个巨大的优势。关键是找出哪些地方可以简化，而不会让你的用户和未来的自己吐槽。

```java

// lox/LoxFunction.java, in call()

      return returnValue.value;
    }

    if (isInitializer) return closure.getAt(0, "this");
    return null;
	
```


If the function is an initializer, we override the actual return value and forcibly return this. That relies on a new isInitializer field.

如果函数是一个初始化器，我们会覆盖实际的返回值，并且强制返回this，这依赖于一个新的 isInitializer 字段

```java

// lox/LoxFunction.java, in class LoxFunction, replace 1 line

 private final Environment closure;

  private final boolean isInitializer;

  LoxFunction(Stmt.Function declaration, Environment closure,
              boolean isInitializer) {
    this.isInitializer = isInitializer;
    this.closure = closure;
    this.declaration = declaration;
	
```

We can’t simply see if the name of the LoxFunction is “init” because the user could have defined a function with that name. In that case, there is no this to return. To avoid that weird edge case, we’ll directly store whether the LoxFunction represents an initializer method. That means we need to go back and fix the few places where we create LoxFunctions.


我们不能仅仅通过函数的名称是否是 init，来判断。因为用户可能已经定义了一个具有该名称的函数，在这种情况下，没有this 可以返回，为了避免这种奇怪的边界场景，我们将直接存储LoxFunction 是否表示一个初始化器。这意味着，我们将修复之前的创建LoxFunction 的几个地方。

```java

// lox/Interpreter.java, in visitFunctionStmt(), replace 1 line

 public Void visitFunctionStmt(Stmt.Function stmt) {
    LoxFunction function = new LoxFunction(stmt, environment,
                                           false);
    environment.define(stmt.name.lexeme, function);
	
```

For actual function declarations, isInitializer is always false. For methods, we check the name.

对于实际的函数声明，isInitializer始终为false, 对于方法，我们将检查方法名称

```java

// lox/Interpreter.java, in visitClassStmt(), replace 1 line

  for (Stmt.Function method : stmt.methods) {
      LoxFunction function = new LoxFunction(method, environment,
          method.name.lexeme.equals("init"));
      methods.put(method.name.lexeme, function);

```

And then in bind() where we create the closure that binds this to a method, we pass along the original method’s value.

然后，在bind() 函数中，我们创建一个闭包，来将this 绑定到方法，我们将原始方法的值传递给它

```java

// lox/LoxFunction.java, in bind(), replace 1 line

   environment.define("this", instance);
    return new LoxFunction(declaration, environment,
                           isInitializer);
  }


```


### 7.2 Returning from init()

We aren’t out of the woods yet. We’ve been assuming that a user-written initializer doesn’t explicitly return a value because most constructors don’t. What should happen if a user tries:

我们还没有完全解决问题，我们一直假设用户编写的初始化器不会显示的返回值，因为大多数的构造函数不会这样做。如果用户尝试以下操作，将会发生什么呢？

```java

class Foo {
  init() {
    return "something else";
  }
}


```

It’s definitely not going to do what they want, so we may as well make it a static error. Back in the resolver, we add another case to FunctionType.

它肯定不会做用户想的事情，所以，我们可以将这种场景当作一个静态错误检查，回到解析器，我们将添加一种新的FunctionType

```java

// lox/Resolver.java, in enum FunctionType

   FUNCTION,
    INITIALIZER,
    METHOD


```

We use the visited method’s name to determine if we’re resolving an initializer or not.

我们使用visited 方法的名称来确定我们是否正在解析初始化器

```java

// lox/Resolver.java, in visitClassStmt()

    FunctionType declaration = FunctionType.METHOD;
      if (method.name.lexeme.equals("init")) {
        declaration = FunctionType.INITIALIZER;
      }

      resolveFunction(method, declaration); 
	  
```

When we later traverse into a return statement, we check that field and make it an error to return a value from inside an init() method.

当我们稍后遍历，返回语句时候，我们检查该字段，并判断从init函数内部返回值是错误方式

```java

// lox/Resolver.java, in visitReturnStmt()

 if (stmt.value != null) {
      if (currentFunction == FunctionType.INITIALIZER) {
        Lox.error(stmt.keyword,
            "Can't return a value from an initializer.");
      }

      resolve(stmt.value);
	  
```

We’re still not done. We statically disallow returning a value from an initializer, but you can still use an empty early return.

我们还没有完成解析，我们静态的禁止从初始化器中返回值，但是，还可以返回空

```java

class Foo {
  init() {
    return;
  }
}

```

That is actually kind of useful sometimes, so we don’t want to disallow it entirely. Instead, it should return this instead of nil. That’s an easy fix over in LoxFunction.

有时候，这样是有作用的，因此，我们并不想完全禁用这种语法，相反，它应该返回this，而不是nil，我们在LoxFunction中修复

```java

// lox/LoxFunction.java, in call()

  } catch (Return returnValue) {
      if (isInitializer) return closure.getAt(0, "this");

      return returnValue.value;


```

If we’re in an initializer and execute a return statement, instead of returning the value (which will always be nil), we again return this.

Phew! That was a whole list of tasks but our reward is that our little interpreter has grown an entire programming paradigm. Classes, methods, fields, this, and constructors. Our baby language is looking awfully grown-up.

如果我们在初始化器中，调用了return语句，我们将再次返回 this，而不是nil

哇，这是一整个任务列表，但我们的奖励是我们的解释器，已经成长为一个完整的编程范式，类、方法、字段、this、构造器，我们的语言看起来更加成熟了。

