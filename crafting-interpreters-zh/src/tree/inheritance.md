# 继承


> Once we were blobs in the sea, and then fishes, and then lizards and rats and then monkeys, and hundreds of things in between. This hand was once a fin, this hand once had claws! In my human mouth I have the pointy teeth of a wolf and the chisel teeth of a rabbit and the grinding teeth of a cow! Our blood is as salty as the sea we used to live in! When we’re frightened, the hair on our skin stands up, just like it did when we had fur. We are history! Everything we’ve ever been on the way to becoming us, we still are.
> 
> <p align="right"> —— Terry Pratchett, A Hat Full of Sky</p>
>
> 曾经我们是海洋中的泥块，然后是鱼，然后是蜥蜴、老鼠，接着是猴子，还有数百种中间形态，这双手曾经是鱼鳍，曾经是爪子，在我们人类的嘴中，有狼的尖牙、兔子的凿齿、牛的磨牙，我们的血液和曾经生活过的海洋一样咸。当我们害怕时候，身上的毛发会树立起来，就像我们还有皮毛一样，我们是历史，曾经经历的一切，都是成为现在的我们的过程中的一部分，我们仍然是那些经历的总和。
> 
> <p align="right"> —— Terry Pratchett, A Hat Full of Sky</p>



Can you believe it? We’ve reached the last chapter of Part II. We’re almost done with our first Lox interpreter. The previous chapter was a big ball of intertwined object-orientation features. I couldn’t separate those from each other, but I did manage to untangle one piece. In this chapter, we’ll finish off Lox’s class support by adding inheritance.

Inheritance appears in object-oriented languages all the way back to the first one, Simula. Early on, Kristen Nygaard and Ole-Johan Dahl noticed commonalities across classes in the simulation programs they wrote. Inheritance gave them a way to reuse the code for those similar parts.

你能相信吗？我们已经到达第二部分的最后一章，我们即将完成第一个解释器，上一章是一个交织着对象导向特性的大球，我无法将其分开，但是，我设法解开了其中的一部分，在本章，我们将添加继承特性来完成Lox语言的类的支持。

继承出现在面向对象的语言中，我们可以追溯到第一个面向对象的语言，Simula, 早期，Kristen Nygaard和Ole-Johan Dahl在编写模拟程序时候，注意到类之间的共性。继承为他们提供了一种复用这些相似部分代码的方式

> You could say all those other languages inherited it from Simula. Hey-ooo! I’ll, uh, see myself out.
> 
> 你可以说，其他语言的继承都参考了 Simula

## 一、Superclasses and Subclasses

超类和子类

Given that the concept is “inheritance”, you would hope they would pick a consistent metaphor and call them “parent” and “child” classes, but that would be too easy. Way back when, C. A. R. Hoare coined the term “subclass” to refer to a record type that refines another type. Simula borrowed that term to refer to a class that inherits from another. I don’t think it was until Smalltalk came along that someone flipped the Latin prefix to get “superclass” to refer to the other side of the relationship. From C++, you also hear “base” and “derived” classes. I’ll mostly stick with “superclass” and “subclass”.

鉴于这个概念是继承，你希望他们会选择一个一致的隐喻，称之为父类和子类，这样太容易了。很久以前，C.A.R.Hoare 创造了术语"子类" 来指代一种类型——改进了一种类型。Simula 引入了这个概念，表示从另一个类继承的类。我认为一直到Smalltalk出现，才有人翻转了拉丁前缀，得到超类来指代关系的另一侧，从C++中，你可能还会得到 基类和派生类，多数场景，我会使用超类和子类描述

Our first step towards supporting inheritance in Lox is a way to specify a superclass when declaring a class. There’s a lot of variety in syntax for this. C++ and C# place a : after the subclass’s name, followed by the superclass name. Java uses extends instead of the colon. Python puts the superclass(es) in parentheses after the class name. Simula puts the superclass’s name before the class keyword.

This late in the game, I’d rather not add a new reserved word or token to the lexer. We don’t have extends or even :, so we’ll follow Ruby and use a less-than sign (<).

我们在Lox中支持继承的第一步是，在声明类时候，指定超类的方式。这方面的语法有很多种，

* C++ 和 C#，在子类名称后面添加一个冒号，然后是超类名称

	```C++
	class SubclassName : inheritance-access-specifier SuperclassName {
	    ......
    };
	```
	
* Java使用 extends 代替冒号

	```java
	
	public class ArmoredCar extends Car {

    }
	```

* Python 在类名后面用括号放置超类的名称

	```python
	
	class HourlyEmployee(Employee):
	
	```

* Simula 在class 关键字之前放置超类名称

在游戏的最后，我们并不想在词法分析器中，添加新的保留字或者新的token，我们不会使用 extends 或者冒号，我们将参考Ruby的实现方式，使用< 表示继承。

```java


class Doughnut {
  // General doughnut stuff...
}

class BostonCream < Doughnut {
  // Boston Cream-specific stuff...
}

```

> “Super-” and “sub-” mean “above” and “below” in Latin, respectively. Picture an inheritance tree like a family tree with the root at the top—subclasses are below their superclasses on the diagram. More generally, “sub-” refers to things that refine or are contained by some more general concept. In zoology, a subclass is a finer categorization of a larger class of living things.
> 
> In set theory, a subset is contained by a larger superset which has all of the elements of the subset and possibly more. Set theory and programming languages meet each other in type theory. There, you have “supertypes” and “subtypes”.
>
> In statically typed object-oriented languages, a subclass is also often a subtype of its superclass. Say we have a Doughnut superclass and a BostonCream subclass. Every BostonCream is also an instance of Doughnut, but there may be doughnut objects that are not BostonCreams (like Crullers).
> 
> Think of a type as the set of all values of that type. The set of all Doughnut instances contains the set of all BostonCream instances since every BostonCream is also a Doughnut. So BostonCream is a subclass, and a subtype, and its instances are a subset. It all lines up.
> 
> super- 和 sub- 在拉丁语中可以表示 上方和下方。想象一个继承树就是一个家谱，根在顶部，子类在根的下方。更一般的，sub-是指对某个一般概念进行细化或者包含的事物。在动物学中，子类是对更大类别的生物的细分。
> 
> 在集合论中，一个子集包含于一个更大的超集中，该超集具有子集的所有元素，可能还有更多元素。集合论和编程语言在类型理论中相遇。在那里，你有“超类型”和“子类型”。
> 
> 在静态类型的面向对象语言中，子类通常也是其超类的子类型。假设我们有一个 Doughnut 超类和一个 BostonCream 子类。每个 BostonCream 也是 Doughnut 的一个实例，但可能有不是 BostonCream 的 doughnut对象（如 Crullers）。
>
> 把一个类型看作所有该类型值的集合。所有 Doughnut 实例的集合包含了所有 BostonCream 实例的集合，因为每个 BostonCream 也是 Doughnut。因此，BostonCream 是一个子类和子类型，它的实例是一个子集。这一切都是相互对应的。

![Doughnut](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/doughnuts.png?raw=true)

To work this into the grammar, we add a new optional clause in our existing classDecl rule.

为了将其纳入到语法规则中，我们在classDecl 中添加一个新的分支

```java

classDecl      → "class" IDENTIFIER ( "<" IDENTIFIER )?
                 "{" function* "}" ;
				 
```

After the class name, you can have a < followed by the superclass’s name. The superclass clause is optional because you don’t have to have a superclass. Unlike some other object-oriented languages like Java, Lox has no root “Object” class that everything inherits from, so when you omit the superclass clause, the class has no superclass, not even an implicit one.

We want to capture this new syntax in the class declaration’s AST node.


```java

// tool/GenerateAst.java, in main(), replace 1 line

      "Block      : List<Stmt> statements",
      "Class      : Token name, Expr.Variable superclass," +
                  " List<Stmt.Function> methods",
      "Expression : Expr expression",


```

在类名之后，可以紧跟一个小于号，然后是超类的名称，超类子句是可选的，因为我们不是必须指定一个超类，和Java等面向对象的语言不同，Lox语言中没有根类 Object,(所有类都继承Object）。因此，当省略了超类子句时候，该类没有超类，甚至没有隐含的超类。我们将在类声明的AST节点捕获这个新的语法。

You might be surprised that we store the superclass name as an Expr.Variable, not a Token. The grammar restricts the superclass clause to a single identifier, but at runtime, that identifier is evaluated as a variable access. Wrapping the name in an Expr.Variable early on in the parser gives us an object that the resolver can hang the resolution information off of.

The new parser code follows the grammar directly.

你可能会感到惊讶，我们将超类名称存储为Expr.Variable, 而不是Token，语法将超类子句限制为单个标识符，但是在运行时，该标识符将作为变量访问进行计算，在解释器的早期将超类名称包装在 Expr.Variable中，为我们提供了一个对象，解析器可以将解析信息绑定在上面

新的解析器代码将直接遵循语法

```java

// lox/Parser.java, in classDeclaration()

  Token name = consume(IDENTIFIER, "Expect class name.");

    Expr.Variable superclass = null;
    if (match(LESS)) {
      consume(IDENTIFIER, "Expect superclass name.");
      superclass = new Expr.Variable(previous());
    }

    consume(LEFT_BRACE, "Expect '{' before class body.");
	
```


Once we’ve (possibly) parsed a superclass declaration, we store it in the AST.

一旦我们解析了超类声明语句，我们将保存超类

```java

// lox/Parser.java, in classDeclaration(), replace 1 line

   consume(RIGHT_BRACE, "Expect '}' after class body.");

    return new Stmt.Class(name, superclass, methods);
  }


```

If we didn’t parse a superclass clause, the superclass expression will be null. We’ll have to make sure the later passes check for that. The first of those is the resolver.

如果没有解析到超类声明子句，超类表达式值为null，我们需要确保后续步骤检查了这一点，第一步是解析器。

```java

// lox/Resolver.java, in visitClassStmt()

    define(stmt.name);

    if (stmt.superclass != null) {
      resolve(stmt.superclass);
    }

    beginScope();
	
```

The class declaration AST node has a new subexpression, so we traverse into and resolve that. Since classes are usually declared at the top level, the superclass name will most likely be a global variable, so this doesn’t usually do anything useful. However, Lox allows class declarations even inside blocks, so it’s possible the superclass name refers to a local variable. In that case, we need to make sure it’s resolved.

Because even well-intentioned programmers sometimes write weird code, there’s a silly edge case we need to worry about while we’re in here. Take a look at this:


类声明AST 节点，有一个新的子表达式，因此我们需要遍历并且解析它，由于类通常在顶层声明，因此超类名称很可能是全局变量，因此，这通常不会产生任何有用的结果，但是，Lox允许在块内声明类，因此，超类名称可能引用一个局部变量，在这种情况下，我们需要确保它被解析。

因为即使是有良好意图的程序员，有时候也会写出奇怪的代码，所以，我们需要担心一个愚蠢的边界问题，

```java

class Oops < Oops {}
```

There’s no way this will do anything useful, and if we let the runtime try to run this, it will break the expectation the interpreter has about there not being cycles in the inheritance chain. The safest thing is to detect this case statically and report it as an error.

上面的代码没有实际意义，如果在运行时候，尝试运行上面的代码，它会打破解释器对继承链中不存在的循环的预期，最安全的做法是在静态检测时候，检测到这种情况，并且将其报告为错误。

```java

// lox/Resolver.java, in visitClassStmt()

    define(stmt.name);

    if (stmt.superclass != null &&
        stmt.name.lexeme.equals(stmt.superclass.name.lexeme)) {
      Lox.error(stmt.superclass.name,
          "A class can't inherit from itself.");
    }

    if (stmt.superclass != null) {
	
```

Assuming the code resolves without error, the AST travels to the interpreter.

假设代码在解析时候没有报错，AST将被传递到解释器

```java

// lox/Interpreter.java, in visitClassStmt()

  public Void visitClassStmt(Stmt.Class stmt) {
    Object superclass = null;
    if (stmt.superclass != null) {
      superclass = evaluate(stmt.superclass);
      if (!(superclass instanceof LoxClass)) {
        throw new RuntimeError(stmt.superclass.name,
            "Superclass must be a class.");
      }
    }

    environment.define(stmt.name.lexeme, null);
	
```

If the class has a superclass expression, we evaluate it. Since that could potentially evaluate to some other kind of object, we have to check at runtime that the thing we want to be the superclass is actually a class. Bad things would happen if we allowed code like:

如果类有 superclass 表达式，我们会先计算它，由于它可能会被计算为其他类型的对象，因此，我们必须在运行时候，检查我们想要作为超类的东西是一个类。否则，如果我们允许像下面这样的代码存在，将发生糟糕的事情

```java

var NotAClass = "I am totally not a class";

class Subclass < NotAClass {} // ?!

```


Assuming that check passes, we continue on. Executing a class declaration turns the syntactic representation of a class—its AST node—into its runtime representation, a LoxClass object. We need to plumb the superclass through to that too. We pass the superclass to the constructor.

假设，检查超类通过，我们将继续进行。执行类声明将类的语法表示（即把AST节点——> 运行时表示，即LoxClass对象），我们也需要将超类传入其中，我们将超类传递给构造函数。

```java

// lox/Interpreter.java, in visitClassStmt(), replace 1 line

      methods.put(method.name.lexeme, function);
    }

    LoxClass klass = new LoxClass(stmt.name.lexeme,
        (LoxClass)superclass, methods);

    environment.assign(stmt.name, klass);
	
```


The constructor stores it in a field.

```java

// lox/LoxClass.java, constructor LoxClass(), replace 1 line

  LoxClass(String name, LoxClass superclass,
           Map<String, LoxFunction> methods) {
    this.superclass = superclass;
    this.name = name;


```


Which we declare here:

```java

// lox/LoxClass.java, in class LoxClass

  final String name;
  final LoxClass superclass;
  private final Map<String, LoxFunction> methods;
  
```

With that, we can define classes that are subclasses of other classes. Now, what does having a superclass actually do?

有了这个，我们可以定义一个是其他类的子类，那么，我们拥有一个超类，可以实际做什么呢？

## 二、Inheriting Methods


继承方法

Inheriting from another class means that everything that’s true of the superclass should be true, more or less, of the subclass. In statically typed languages, that carries a lot of implications. The subclass must also be a subtype, and the memory layout is controlled so that you can pass an instance of a subclass to a function expecting a superclass and it can still access the inherited fields correctly.


从另一个类继承意味着超类的所有特性，在子类中应该大体上也是成立的。在静态类型语言中，这带来了许多含义。子类必须也是超类的子类型，并且内存中可以将子类的实例传递给希望超类的函数，并且依然可以正确的访问继承的字段。

> A fancier name for this hand-wavey guideline is the Liskov substitution principle. Barbara Liskov introduced it in a keynote during the formative period of object-oriented programming.
> 
> 更正式的讲，这条含糊不清的指导原则叫做 [Liskov替换原则](https://en.wikipedia.org/wiki/Liskov_substitution_principle), [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov) 在面向对象编程初期的一次主旨演讲中提出了这一原则。

Lox is a dynamically typed language, so our requirements are much simpler. Basically, it means that if you can call some method on an instance of the superclass, you should be able to call that method when given an instance of the subclass. In other words, methods are inherited from the superclass.

This lines up with one of the goals of inheritance—to give users a way to reuse code across classes. Implementing this in our interpreter is astonishingly easy.

Lox是一种动态类型语言，因此我们的要求简单的多，基本上，这意味着，如果你可以在超类的某个实例上调用某个方法，那么当你给定了超类的某个子类的实例时候，我们也应该能够调用该方法。换句话说，方法是从超类继承来的

这样的表现和继承的一个目标吻合——为用户提供了一种在类之间复用代码的方式，在我们的解释器中实现这一点非常容易。


```java

// lox/LoxClass.java, in findMethod()

      return methods.get(name);
    }

    if (superclass != null) {
      return superclass.findMethod(name);
    }

    return null;
	
```


That’s literally all there is to it. When we are looking up a method on an instance, if we don’t find it on the instance’s class, we recurse up through the superclass chain and look there. Give it a try:

确实就是这样，当我们在实例中查找方法时候，如果在当前实例的类中找不到它，我们会递归的查找超类链，一层层查找是否存在该函数

```java


class Doughnut {
  cook() {
    print "Fry until golden brown.";
  }
}

class BostonCream < Doughnut {}

BostonCream().cook();

```

There we go, half of our inheritance features are complete with only three lines of Java code.

就这样，我们的继承功能已经完成了一半，只用了三行Java代码

## 三、Calling Superclass Methods

调用超类方法

In findMethod() we look for a method on the current class before walking up the superclass chain. If a method with the same name exists in both the subclass and the superclass, the subclass one takes precedence or overrides the superclass method. Sort of like how variables in inner scopes shadow outer ones.

That’s great if the subclass wants to replace some superclass behavior completely. But, in practice, subclasses often want to refine the superclass’s behavior. They want to do a little work specific to the subclass, but also execute the original superclass behavior too.

However, since the subclass has overridden the method, there’s no way to refer to the original one. If the subclass method tries to call it by name, it will just recursively hit its own override. We need a way to say “Call this method, but look for it directly on my superclass and ignore my override”. Java uses super for this, and we’ll use that same syntax in Lox. Here is an example:

在 findMethod() 方法中，我们在遍历超类链之前，优先在当前的实例中查找方法，如果在超类和子类中都存在同名的函数，则子类中的方法具有更高的优先级，会覆盖超类方法。这有点像是，内部作用域中的变量覆盖外部同名变量

这对于，如果我们想要完全替换一些超类的行为。但是，在实践中，我们通常想要替换部分的超类行为，我们想要子类执行部分特定的工作，但是也要执行超类的原始行为

然而，子类已经覆盖了同名的方法，没有办法再引用超类的方法了。如果子类方法按照名称调用它，它只会调用子类的方法。我们需要一种方法——调用一个方法，但是只是调用超类中定义的方法，忽略当前子类的同名方法。Java中使用super 完成这种需求，Lox中我们将使用相同的方式。

```java

class Doughnut {
  cook() {
    print "Fry until golden brown.";
  }
}

class BostonCream < Doughnut {
  cook() {
    super.cook();
    print "Pipe full of custard and coat with chocolate.";
  }
}

BostonCream().cook();

```


If you run this, it should print:

```

Fry until golden brown.
Pipe full of custard and coat with chocolate.

```


We have a new expression form. The super keyword, followed by a dot and an identifier, looks for a method with that name. Unlike calls on this, the search starts at the superclass.

我们有一个新的表达式形式，关键字是super, 后面跟一个点和一个标识符，查找具有更名称的方法，与this关键字不同的是，我们的搜索是从超类开始

### 3.1 Syntax

语法

With this, the keyword works sort of like a magic variable, and the expression is that one lone token. But with super, the subsequent . and property name are inseparable parts of the super expression. You can’t have a bare super token all by itself.

```java

print super; // Syntax error.

```

当我们使用this关键字，它的工作方式好像是一个魔法变量，表达式也可以是一个孤立的this关键字。但是super关键字使用方式不一样，super和 . 标识符三个部分，不能分开，一个表达式不能是一个孤立的super 关键字。

So the new clause we add to the primary rule in our grammar includes the property access as well.

所以，我们在语法规则中添加了新的子句，包含了属性访问

```

primary        → "true" | "false" | "nil" | "this"
               | NUMBER | STRING | IDENTIFIER | "(" expression ")"
               | "super" "." IDENTIFIER ;
			   
```


Typically, a super expression is used for a method call, but, as with regular methods, the argument list is not part of the expression. Instead, a super call is a super access followed by a function call. Like other method calls, you can get a handle to a superclass method and invoke it separately.

通常，super表达式用于方法调用，但是与常规方法不同，参数列表不是表达式的一部分。相反，super调用是一个super访问，后面跟着一个函数调用。像其他方法调用一样，可以获得一个超类方法的句柄，并且单独使用它

```java

var method = super.cook;
method();

```

So the super expression itself contains only the token for the super keyword and the name of the method being looked up. The corresponding syntax tree node is thus:

所以，super表达式仅仅包含super 关键字和方法名。相应的语法树节点如下所示

Following the grammar, the new parsing code goes inside our existing primary() method.

遵循语法规则，新的解析代码将会放置在我们现有的primary() 方法中

```java

// lox/Parser.java, in primary()


      return new Expr.Literal(previous().literal);
    }

    if (match(SUPER)) {
      Token keyword = previous();
      consume(DOT, "Expect '.' after 'super'.");
      Token method = consume(IDENTIFIER,
          "Expect superclass method name.");
      return new Expr.Super(keyword, method);
    }

    if (match(THIS)) return new Expr.This(previous());

```

A leading super keyword tells us we’ve hit a super expression. After that we consume the expected . and method name.

super关键字告诉我们遇到了一个super表达式，然后，我们将预期后面有一个 . 和 一个方法名称

### 3.2 Semantics

语义

Earlier, I said a super expression starts the method lookup from “the superclass”, but which superclass? The naïve answer is the superclass of this, the object the surrounding method was called on. That coincidentally produces the right behavior in a lot of cases, but that’s not actually correct. Gaze upon:

先前我们说过，super 表达式从超类开始进行方法的查找，但是是从哪一个超类呢，简单的回答是：从当前类的超类开始查找，即调用周围方法的对象。这在很多场景都会进行正确的操作，但是，实际上，这是不正确的。

```java

class A {
  method() {
    print "A method";
  }
}

class B < A {
  method() {
    print "B method";
  }

  test() {
    super.method();
  }
}

class C < B {}

C().test();


```


Translate this program to Java, C#, or C++ and it will print “A method”, which is what we want Lox to do too. When this program runs, inside the body of test(), this is an instance of C. The superclass of C is B, but that is not where the lookup should start. If it did, we would hit B’s method().

Instead, lookup should start on the superclass of the class containing the super expression. In this case, since test() is defined inside B, the super expression inside it should start the lookup on B’s superclass—A.


将上面程序翻译为 Java C# C++， 我们将得到 "A method", 这也是我们希望Lox做的。当这个程序运行时候，在test() 函数内部，this 是C的一个实例，C的超类是B，但是，这不是应该查找开始的地方，因为如果是从此开始，我们将会得到 B method.

相反，查找应该从包含super表达式的类的超类开始，如此，在B中定义的test() 函数，因此super表达式所表示的超类是A,所以，我们应该从A开始查找。

![classes](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/classes.png?raw=true)


The execution flow looks something like this:

* We call test() on an instance of C.

* That enters the test() method inherited from B. That calls super.method().

* The superclass of B is A, so that chains to method() on A, and the program prints “A method”.

执行流程大概是

* 在C的实例中 调用 test()

* 进入B类中的test() 方法，然后调用 super.method()

* super指代的类是A, 如果调用A的 method() 方法


Thus, in order to evaluate a super expression, we need access to the superclass of the class definition surrounding the call. Alack and alas, at the point in the interpreter where we are executing a super expression, we don’t have that easily available.

We could add a field to LoxFunction to store a reference to the LoxClass that owns that method. The interpreter would keep a reference to the currently executing LoxFunction so that we could look it up later when we hit a super expression. From there, we’d get the LoxClass of the method, then its superclass.

That’s a lot of plumbing. In the last chapter, we had a similar problem when we needed to add support for this. In that case, we used our existing environment and closure mechanism to store a reference to the current object. Could we do something similar for storing the superclass? Well, I probably wouldn’t be talking about it if the answer was no, so . . . yes.

因此，为了获取到super表达式的指代，我们需要访问调用周围的类定义的超类，但是，在解释器执行super 表达式时候，我们很难获取到这个信息

我们可以在LoxFunction中添加一个字段，存储一个引用，引用的LoxClass 拥有该方法。解释器将会保留对当前执行的LoxFunction 的引用，以便在遇到super表达式时候，查找到它。从这个新增的字段，我们将获取该方法的LoxClass，然后是它的超类

这需要更多的工作。在上一章中，我们在需要支持this 时候遇到了同样的问题，在那种情况下，我们使用现有的环境，和闭包机制，存储当前对象的引用。我们是否可以类似的存储超类呢？好吧，这个答案是肯定的，如果答案是否定的，我也不会谈论它，😄

> Does anyone even like rhetorical questions?
> 
> 有人喜欢修辞性问题吗？

One important difference is that we bound this when the method was accessed. The same method can be called on different instances and each needs its own this. With super expressions, the superclass is a fixed property of the class declaration itself. Every time you evaluate some super expression, the superclass is always the same.

That means we can create the environment for the superclass once, when the class definition is executed. Immediately before we define the methods, we make a new environment to bind the class’s superclass to the name super.

一个重要的区别是，我们在访问方法时候，绑定了this，相同的方法可以在不同的实例上调用，所以，每个实例都有自己的this。但是，对于super表达式，超类是一个类声明本身的固定属性，每个获取某个super表达式时候，超类都是相同的

这意味着，我们可以在执行类的定义时候，创建一次类的超类环境，在我们定义方法之前，我们创建了一个新的环境，将类的超类绑定到名称 super

![superclass](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/superclass.png?raw=true)

When we create the LoxFunction runtime representation for each method, that is the environment they will capture in their closure. Later, when a method is invoked and this is bound, the superclass environment becomes the parent for the method’s environment, like so:

当我们为每个方法创建 LoxFunction 的运行时表示时候，我们将在其闭包中捕获环境，稍后，当调用方法并且绑定时候，超类环境成为方法的环境的超环境，如下所示：

![environments](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/environments.png?raw=true)

That’s a lot of machinery, but we’ll get through it a step at a time. Before we can get to creating the environment at runtime, we need to handle the corresponding scope chain in the resolver.

这需要很多机制，我们将逐渐完善，在运行时创建环境之前，我们需要在解释器中处理相应的调用链


```java

// lox/Resolver.java, in visitClassStmt()

      resolve(stmt.superclass);
    }

    if (stmt.superclass != null) {
      beginScope();
      scopes.peek().put("super", true);
    }

    beginScope();


```


If the class declaration has a superclass, then we create a new scope surrounding all of its methods. In that scope, we define the name “super”. Once we’re done resolving the class’s methods, we discard that scope.

如果类声明有一个超类，我们将在所有方法周围创建一个新的作用域，在这个作用域中，我们定义名称 "super", 当完成类的方法解析之后，我们将会丢弃该作用域


```java

// lox/Resolver.java, in visitClassStmt()

    endScope();

    if (stmt.superclass != null) endScope();

    currentClass = enclosingClass;


```

It’s a minor optimization, but we only create the superclass environment if the class actually has a superclass. There’s no point creating it when there isn’t a superclass since there’d be no superclass to store in it anyway.

With “super” defined in a scope chain, we are able to resolve the super expression itself.

这是一个次要的优化，但我们只有在类实际上有一个超类时候，才会创建超类环境，如果没有超类，就没有意义创建它，因为它也没有需要存储的超类

在作用域链中定义了super 后，我们可以解析super 表达式本身

```java

// lox/Resolver.java, add after visitSetExpr()

  @Override
  public Void visitSuperExpr(Expr.Super expr) {
    resolveLocal(expr, expr.keyword);
    return null;
  }


```

We resolve the super token exactly as if it were a variable. The resolution stores the number of hops along the environment chain that the interpreter needs to walk to find the environment where the superclass is stored.

This code is mirrored in the interpreter. When we evaluate a subclass definition, we create a new environment.

我们像解析变量一样，解析super token, 解析会存储沿着环境链需要走多少步，才可以找到存储超类的环境

这段代码在解释器中也有对应，当我们要得到一个子类的定义，我们将会得到一个新的环境


```java

// lox/Interpreter.java, in visitClassStmt()

        throw new RuntimeError(stmt.superclass.name,
            "Superclass must be a class.");
      }
    }

    environment.define(stmt.name.lexeme, null);

    if (stmt.superclass != null) {
      environment = new Environment(environment);
      environment.define("super", superclass);
    }

    Map<String, LoxFunction> methods = new HashMap<>();
	
```

Inside that environment, we store a reference to the superclass—the actual LoxClass object for the superclass which we have now that we are in the runtime. Then we create the LoxFunctions for each method. Those will capture the current environment—the one where we just bound “super”—as their closure, holding on to the superclass like we need. Once that’s done, we pop the environment.

在这个环境中，我们将存储对超类的引用——超类的实际LoxClass 对象，因为我们现在处于运行时。然后，我们为每个方法创建 LoxFunction,这些函数将会捕获当前的环境——刚刚绑定了super 的环境——作为它们的闭包，保留我们需要的超类，完成后，我们将弹出环境

```java

// lox/Interpreter.java, in visitClassStmt()

    LoxClass klass = new LoxClass(stmt.name.lexeme,
        (LoxClass)superclass, methods);

    if (superclass != null) {
      environment = environment.enclosing;
    }

    environment.assign(stmt.name, klass);
	
```


We’re ready to interpret super expressions themselves. There are a few moving parts, so we’ll build this method up in pieces.

我们已经准备好解释super表达式本身了，这里有一些需要处理的细节，因此，我们将逐步构建这个方法。

```java

// lox/Interpreter.java, add after visitSetExpr()

  @Override
  public Object visitSuperExpr(Expr.Super expr) {
    int distance = locals.get(expr);
    LoxClass superclass = (LoxClass)environment.getAt(
        distance, "super");
  }
  
```


First, the work we’ve been leading up to. We look up the surrounding class’s superclass by looking up “super” in the proper environment.

When we access a method, we also need to bind this to the object the method is accessed from. In an expression like doughnut.cook, the object is whatever we get from evaluating doughnut. In a super expression like super.cook, the current object is implicitly the same current object that we’re using. In other words, this. Even though we are looking up the method on the superclass, the instance is still this.

Unfortunately, inside the super expression, we don’t have a convenient node for the resolver to hang the number of hops to this on. Fortunately, we do control the layout of the environment chains. The environment where “this” is bound is always right inside the environment where we store “super”.

首先，我们需要完成前面的工作，通过在正确的环境中查找super，我们查找周围类的超类

当我们访问方法时候，我们还需要将 this 绑定到访问方法的对象上，在表达式，doughnut.cook ，对象是从获取到的 doughnut 的任何内容，在super表达式，例如: super.cook, 当前对象，隐式的表达是当前对象，也就是this， 即使我们在超类中查找方法，实例仍然是this

不幸的是，在super表达式的内部，我们没有一个方便的节点，让解析器挂起到this 的跳数，幸运的是，我们控制了环境链，绑定this 的环境总是我们存储的super环境的直接内部

```java

// lox/Interpreter.java, in visitSuperExpr()

    LoxClass superclass = (LoxClass)environment.getAt(
        distance, "super");

    LoxInstance object = (LoxInstance)environment.getAt(
        distance - 1, "this");
  }
  
```

Offsetting the distance by one looks up “this” in that inner environment. I admit this isn’t the most elegant code, but it works.

通过将距离偏移一次，在内部环境中查找到 this,我承认这不是最优雅的代码，但是它可以正常工作

> Writing a book that includes every single line of code for a program means I can’t hide the hacks by leaving them as an “exercise for the reader”.
> 
> 编写一本书，包含程序中的每一行代码，意味着我不能将这些hack隐藏起来，留给读者当作课后练习

Now we’re ready to look up and bind the method, starting at the superclass.

现在，我们已经准备好从，超类开始查找并且绑定方法了

```java

// lox/Interpreter.java, in visitSuperExpr()

    LoxInstance object = (LoxInstance)environment.getAt(
        distance - 1, "this");

    LoxFunction method = superclass.findMethod(expr.method.lexeme);
    return method.bind(object);
  }


```


This is almost exactly like the code for looking up a method of a get expression, except that we call findMethod() on the superclass instead of on the class of the current object.

That’s basically it. Except, of course, that we might fail to find the method. So we check for that too.

这几乎和查找get表达式的方法的代码完全相同，只是我们在超类上调用findMethod() 而不是在当前类上

基本上就是这样，当然，我们也可能无法找到该方法，因此，我们需要添加检查

```java

// lox/Interpreter.java, in visitSuperExpr()

    LoxFunction method = superclass.findMethod(expr.method.lexeme);

    if (method == null) {
      throw new RuntimeError(expr.method,
          "Undefined property '" + expr.method.lexeme + "'.");
    }

    return method.bind(object);
  }


```

There you have it! Take that BostonCream example earlier and give it a try. Assuming you and I did everything right, it should fry it first, then stuff it with cream.

现在，让我们来体会一下之前的 BostonCream 示例，假设我们都没有出错，我们应该先煎炸，然后再填充奶油

### 3.3 Invalid uses of super

super的非法使用

As with previous language features, our implementation does the right thing when the user writes correct code, but we haven’t bulletproofed the intepreter against bad code. In particular, consider:

与先前的语言特性一样，当用户编写了正确的代码时候，我们的解释器可以正常工作。但是，我们对于一些错误代码，还是不够稳定。特别的，考虑下面的代码

```java

class Eclair {
  cook() {
    super.cook();
    print "Pipe full of crème pâtissière.";
  }
}

```

This class has a super expression, but no superclass. At runtime, the code for evaluating super expressions assumes that “super” was successfully resolved and will be found in the environment. That’s going to fail here because there is no surrounding environment for the superclass since there is no superclass. The JVM will throw an exception and bring our interpreter to its knees.

Heck, there are even simpler broken uses of super:

```java

super.notEvenInAClass();

```

这个类中有一个super表达式，但是没有超类，在运行时候，计算super表达式的代码，假设super 已经成功解析，并且将在环境中找到它。寻找结果将失败，因为周围环境没有超类，Java虚拟机会经常抛出这样的报错，程序停止运行。

更简单的错误使用 super 的用例

We could handle errors like these at runtime by checking to see if the lookup of “super” succeeded. But we can tell statically—just by looking at the source code—that Eclair has no superclass and thus no super expression will work inside it. Likewise, in the second example, we know that the super expression is not even inside a method body.

Even though Lox is dynamically typed, that doesn’t mean we want to defer everything to runtime. If the user made a mistake, we’d like to help them find it sooner rather than later. So we’ll report these errors statically, in the resolver.

我们可以通过检查 super的查找是否成功，在运行时候，处理这些错误。但是，我们也可以静态检查判断——仅仅通过查看源代码（Eclair类没有超类，因此，super表达式不能在Eclair中运行）。同样，在第二个示例中，我们知道，super表达式甚至不在方法体中

即使Lox是动态类型的语言，这也并不代表，我们希望将所有的事情都放到运行时。如果用户犯了一个错误，我们希望帮助他们尽早发现错误。因此，我们将在解析器中，静态报告这些错误。

First, we add a new case to the enum we use to keep track of what kind of class is surrounding the current code being visited.

```java

// lox/Resolver.java, in enum ClassType, add “,” to previous line

    NONE,
    CLASS,
    SUBCLASS
  }
  
```

We’ll use that to distinguish when we’re inside a class that has a superclass versus one that doesn’t. When we resolve a class declaration, we set that if the class is a subclass.

```java

// lox/Resolver.java, in visitClassStmt()

    if (stmt.superclass != null) {
      currentClass = ClassType.SUBCLASS;
      resolve(stmt.superclass);
	  
```

Then, when we resolve a super expression, we check to see that we are currently inside a scope where that’s allowed.

```java

// lox/Resolver.java, in visitSuperExpr()

  public Void visitSuperExpr(Expr.Super expr) {
    if (currentClass == ClassType.NONE) {
      Lox.error(expr.keyword,
          "Can't use 'super' outside of a class.");
    } else if (currentClass != ClassType.SUBCLASS) {
      Lox.error(expr.keyword,
          "Can't use 'super' in a class with no superclass.");
    }

    resolveLocal(expr, expr.keyword);
	
```

If not—oopsie!—the user made a mistake.

首先，我们向 用于跟踪当前正在访问的代码的周围是哪种类型的类的 枚举中添加一个新的case

我们将使用这个值，表示当前是否在一个超类的子类中，当我们解析类的声明时候，如果该类是子类，我们会设置对应的标志

最后，当我们解析super表达式时候，会检查当前是否允许使用super表达式

如果当前不能使用 super表达式——可以确定，用户使用错误。

## 四、Conclusion

We made it! That final bit of error handling is the last chunk of code needed to complete our Java implementation of Lox. This is a real accomplishment and one you should be proud of. In the past dozen chapters and a thousand or so lines of code, we have learned and implemented . . . 

* tokens and lexing,

* abstract syntax trees,

* recursive descent parsing,

* prefix and infix expressions,

* runtime representation of objects,

* interpreting code using the Visitor pattern,

* lexical scope,

* environment chains for storing variables,

* control flow,

* functions with parameters,

* closures,

* static variable resolution and error detection,

* classes,

* constructors,

* fields,

* methods, 

and finally,

* inheritance.


我们做到了，错误处理是我们用Java 实现Lox的最后一部分代码。这是一个成就，我们应该感到自豪。在过去的十几章和一千多行代码中，我们学习和实现了

* Token 和词法分析

* 抽象语法树

* 递归下降解析

* 前缀和中缀表达式

* 对象的运行时表示

* 使用访问者模式解释代码

* 词法作用域

* 用于存储变量的环境链

* 控制流

* 带参数的函数

* 闭包

* 静态变量解析和错误检查

* 类

* 构造函数

* 类的字段

* 类的方法

* 继承

![superhero](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/superhero.png?raw=true)

We did all of that from scratch, with no external dependencies or magic tools. Just you and I, our respective text editors, a couple of collection classes in the Java standard library, and the JVM runtime.

This marks the end of Part II, but not the end of the book. Take a break. Maybe write a few fun Lox programs and run them in your interpreter. (You may want to add a few more native methods for things like reading user input.) When you’re refreshed and ready, we’ll embark on our next adventure.
