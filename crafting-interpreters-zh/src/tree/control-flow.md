# 控制流程

> Logic, like whiskey, loses its beneficial effect when taken in too large quantities
>
> 逻辑，就像是威士忌，如果摄入过量，就会失去有益作用。
> 
> <p align="right"> —— Edward John Moreton Drax Plunkett, Lord Dunsany</p>

Compared to last chapter’s grueling marathon, today is a lighthearted frolic through a daisy meadow. But while the work is easy, the reward is surprisingly large.

和上一章的马拉松比赛相比，今天是一场轻松愉快的在雏菊草地上的嬉戏，尽管这项工作很容易，但是回报非常大。

Right now, our interpreter is little more than a calculator. A Lox program can only do a fixed amount of work before completing. To make it run twice as long you have to make the source code twice as lengthy. We’re about to fix that. In this chapter, our interpreter takes a big step towards the programming language major leagues: Turing-completeness.

现在，我们的解释器，只是一个强一点的计算器，一个Lox 程序只能完成固定数量的工作，如果想要程序运行多次，需要源代码变为同样倍数，我们即将解决这个问题，在本章中，我们的解释器将向编程语言大联盟迈出一大步：图灵完备性

## 一、Turing Machines (Briefly)

图灵机（简要）

In the early part of last century, mathematicians stumbled into a series of confusing paradoxes that led them to doubt the stability of the foundation they had built their work upon. To address that crisis, they went back to square one. Starting from a handful of axioms, logic, and set theory, they hoped to rebuild mathematics on top of an impervious foundation.

在上世纪初，数学家偶然发现了一系列令人困惑的悖论，这些悖论导致数学家们开始怀疑数学基础的稳定性，为了解决这个危机，他们回到了原点。从少数的公理、逻辑、集合论出发，他们希望在不漏水的基础上重构数学。

They wanted to rigorously answer questions like, “Can all true statements be proven?”, “Can we compute all functions that we can define?”, or even the more general question, “What do we mean when we claim a function is ‘computable’?”

他们想严格回答诸如，“所有真陈述都可以被证明吗？” “我们可以计算我们能够定义的所有函数吗”， 甚至更加一般的问题，“当我们声称一个函数是可计算时候“，这意味着什么？

They presumed the answer to the first two questions would be “yes”. All that remained was to prove it. It turns out that the answer to both is “no”, and astonishingly, the two questions are deeply intertwined. This is a fascinating corner of mathematics that touches fundamental questions about what brains are able to do and how the universe works. I can’t do it justice here.

数学家们假设前面两个问题答案是确定的，剩下的一切都是为了证明这一点，但最后事实证明，这两个问题的答案都是否定的，令人惊讶的是，这两种问题深深交织在一起，这是数学的一个迷人角落，涉及到大脑可以做什么，以及宇宙如何运作的基本问题，我在这里做不到公平。

What I do want to note is that in the process of proving that the answer to the first two questions is “no”, Alan Turing and Alonzo Church devised a precise answer to the last question—a definition of what kinds of functions are computable. They each crafted a tiny system with a minimum set of machinery that is still powerful enough to compute any of a (very) large class of functions.

我的确想要指出的是，在证明前面两个问题的答案是否定的过程中，图灵和丘奇设计了最后一个问题的精确答案——什么样的函数是可计算的。他们每个人都用最少的一组机器构建了一个小系统，这些机器仍然足够强大，可以计算任何一个（非常）大的函数类。

These are now considered the “computable functions”. Turing’s system is called a Turing machine. Church’s is the lambda calculus. Both are still widely used as the basis for models of computation and, in fact, many modern functional programming languages use the lambda calculus at their core.

这些现在被认为是可计算函数，图灵设计的系统被称为图灵机，丘奇设计的是lambda 演算，两者仍然被广泛用于计算模型的基础，事实上，许多现代的函数式编程语言都是以lambda 演算为核心。

![图灵机](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/turing-machine.png?raw=true)

Turing machines have better name recognition—there’s no Hollywood film about Alonzo Church yet—but the two formalisms are equivalent in power. In fact, any programming language with some minimal level of expressiveness is powerful enough to compute any computable function.

图灵机有更好的辨识度（还没有关于图灵和丘奇的好莱坞电影），但是这两种形式主义都非常流行。事实上，任何具有最低表达水平的编程语言，都足以计算任何可计算函数。

You can prove that by writing a simulator for a Turing machine in your language. Since Turing proved his machine can compute any computable function, by extension, that means your language can too. All you need to do is translate the function into a Turing machine, and then run that on your simulator.

你可以用你的语言为图灵机编写一个模拟器来证明这一点，由于图灵证明了他的机器可以计算任何可计算函数，这意味着你的语言也可以。你需要做的是，将函数转换为图灵机，然后在模拟器上运行。

If your language is expressive enough to do that, it’s considered Turing-complete. Turing machines are pretty dang simple, so it doesn’t take much power to do this. You basically need arithmetic, a little control flow, and the ability to allocate and use (theoretically) arbitrary amounts of memory. We’ve got the first. By the end of this chapter, we’ll have the second.

如果你的语言有足够的表达能力，那么它就被认为是图灵完备的，图灵机非常简单，不需要太多的功能。你基本上只需要算术、一点控制流、以及内存的分配与使用（理论上是任何大小的内存），我们将去实现一个图灵完备的语言。


## 二、Conditional Execution

条件执行

Enough history, let’s jazz up our language. We can divide control flow roughly into two kinds:

历史回溯已经够多了，现在让我们继续解释器。我们可以将控制流分为两类：

* Conditional or branching control flow is used to not execute some piece of code. Imperatively, you can think of it as jumping ahead over a region of code.

	条件或者分支控制流，用于不执行某一段代码。强制性的，你可以看成是跳过一段代码。
	
* Looping control flow executes a chunk of code more than once. It jumps back so that you can do something again. Since you don’t usually want infinite loops, it typically has some conditional logic to know when to stop looping as well.

	循环控制流多次执行代码。它可以跳回某个地方，这样我们可以继续循环执行代码。由于，我们通常不会需要无限循环，因为循环控制一般和条件控制一起，用于控制何时停止循环。
	

Branching is simpler, so we’ll start there. C-derived languages have two main conditional execution features, the if statement and the perspicaciously named “conditional” operator (?:). An if statement lets you conditionally execute statements and the conditional operator lets you conditionally execute expressions.

分支比较简单，所以，我们从分支开始。C派生语言，有两个主要的条件控制特性，即if 语言 和命名明确的条件运算符 ?: 

if 语句允许有条件的执行语句，条件运算符，允许有条件的执行表达式。

For simplicity’s sake, Lox doesn’t have a conditional operator, so let’s get our if statement on. Our statement grammar gets a new production.

为了简单起见，Lox没有条件运算符，所以让我们从if语句开始，我们的语句语法需要更新

```

statement      → exprStmt
               | ifStmt
               | printStmt
               | block ;
			   
ifStmt         → "if" "(" expression ")" statement
               ( "else" statement )? ;

```

An if statement has an expression for the condition, then a statement to execute if the condition is truthy. Optionally, it may also have an else keyword and a statement to execute if the condition is falsey. The syntax tree node has fields for each of those three pieces.

if语句有一个条件表达式，如果条件是真，则执行一个语句，如果条件为假，则执行另外的语句。语法树中，这三个部分都有对应的字段

```java

// tool/GenerateAst.java, in main()

      "Expression : Expr expression",
      "If         : Expr condition, Stmt thenBranch," +
                  " Stmt elseBranch",
      "Print      : Expr expression",
	  
```

Like other statements, the parser recognizes an if statement by the leading if keyword.

和其他语句一样，解析器通过前导 if关键字，识别if语句。

```Java

// lox/Parser.java, in statement()

  private Stmt statement() {
    if (match(IF)) return ifStatement();
    if (match(PRINT)) return printStatement();
	
```

When it finds one, it calls this new method to parse the rest:

当发现一个if 关键字后，我们会调用新方法，解析剩下的代码

```Java

// lox/Parser.java, add after statement()

  private Stmt ifStatement() {
    consume(LEFT_PAREN, "Expect '(' after 'if'.");
    Expr condition = expression();
    consume(RIGHT_PAREN, "Expect ')' after if condition."); 

    Stmt thenBranch = statement();
    Stmt elseBranch = null;
    if (match(ELSE)) {
      elseBranch = statement();
    }

    return new Stmt.If(condition, thenBranch, elseBranch);
  }


```

As usual, the parsing code hews closely to the grammar. It detects an else clause by looking for the preceding else keyword. If there isn’t one, the elseBranch field in the syntax tree is null.

与往常一样，解析代码和语法非常接近，我们通过找到else 关键字，检测 else子语句，如果没有，则语法树对应的else为空。

That seemingly innocuous optional else has, in fact, opened up an ambiguity in our grammar. Consider:

事实上，这个看似无害的可选项，在我们的语法中打开了一个歧义。考虑：

```
if (first) if (second) whenTrue(); else whenFalse();

```

Here’s the riddle: Which if statement does that else clause belong to? This isn’t just a theoretical question about how we notate our grammar. It actually affects how the code executes:

这是一个迷，if语句和哪个else子句对应，这不仅仅是我们如何记语法的理论问题，它实际上影响代码的执行方式。

* If we attach the else to the first if statement, then whenFalse() is called if first is falsey, regardless of what value second has.

	如果我们将else子句，附加到第一个if子句上，那么当 first为false时候，我们将调用 whenFalse(), 而不需要考虑 second
	
* If we attach it to the second if statement, then whenFalse() is only called if first is truthy and second is falsey.

	如果我们将else子句，附加到第二个if子句上，那么只有当first = true，并且second=false, 时候，我们才会调用 whenFalse()
	
Since else clauses are optional, and there is no explicit delimiter marking the end of the if statement, the grammar is ambiguous when you nest ifs in this way. This classic pitfall of syntax is called the dangling else problem.

由于else语句是可选的，并且没有明确的分隔符标记if语句的结尾，因此当你以这种方式嵌套if语句时候，语法是不明确的。这种经典的语法缺陷被称为 [悬挂的else语句](https://en.wikipedia.org/wiki/Dangling_else)

![dangling-else](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/dangling-else.png?raw=true)

It is possible to define a context-free grammar that avoids the ambiguity directly, but it requires splitting most of the statement rules into pairs, one that allows an if with an else and one that doesn’t. It’s annoying.

可以定义一个上下文无关的语法来直接避免歧义，但是它需要将大多数的语句规则分为两队，一对允许 if 和else，另外一对，不允许。这很烦人。

Instead, most languages and parsers avoid the problem in an ad hoc way. No matter what hack they use to get themselves out of the trouble, they always choose the same interpretation—the else is bound to the nearest if that precedes it.

相反，大多数的语言和解析器都以特殊的方式避免了这个问题。无论它们用什么方法来摆脱困境，他们总是会选择相同的解析路径—— else语句总是匹配最近的if语句。

Our parser conveniently does that already. Since ifStatement() eagerly looks for an else before returning, the innermost call to a nested series will claim the else clause for itself before returning to the outer if statements.

我们的解析器已经方便的做到了这一点，由于ifStatement() 在返回之前，会寻找对应的else语句。所以，内部的嵌套if语句，将在返回之前，找到对应的else语句。

Syntax in hand, we are ready to interpret.

语法已经写好，下面我们将开始完善解释器。

```java

// lox/Interpreter.java, add after visitExpressionStmt()

  @Override
  public Void visitIfStmt(Stmt.If stmt) {
    if (isTruthy(evaluate(stmt.condition))) {
      execute(stmt.thenBranch);
    } else if (stmt.elseBranch != null) {
      execute(stmt.elseBranch);
    }
    return null;
  }
  
```


The interpreter implementation is a thin wrapper around the self-same Java code. It evaluates the condition. If truthy, it executes the then branch. Otherwise, if there is an else branch, it executes that.

解释器的实现是围绕相同的Java代码的一个封装，它计算条件表达式，如果是true，执行 thenBranch ，否则，执行 elseBranch.

If you compare this code to how the interpreter handles other syntax we’ve implemented, the part that makes control flow special is that Java if statement. Most other syntax trees always evaluate their subtrees. Here, we may not evaluate the then or else statement. If either of those has a side effect, the choice not to evaluate it becomes user visible.

如果将这段代码，和解释器实现其他语法的代码比较，那么使得控制流变得特殊的代码是Java 的if语句。大多数其他的语法树，总是计算它的子树，需要注意，我们可能不会真的计算 then 或者 else 语句，如果这些语句计算有bug，我们将会很难发现。

## 三、Logical Operators

Since we don’t have the conditional operator, you might think we’re done with branching, but no. Even without the ternary operator, there are two other operators that are technically control flow constructs—the logical operators and and or.

由于我们没有条件运算符，你可能认为我们已经完成了分支，但是并没有。即使没有三元运算符，从技术上讲，我们有其他两个运算符，逻辑与和逻辑或，可以被认为是控制流结构。

These aren’t like other binary operators because they short-circuit. If, after evaluating the left operand, we know what the result of the logical expression must be, we don’t evaluate the right operand. For example:

逻辑与或，不像是其他二元运算符，因为它们有短路。如果在计算了左操作数后，我们知道了逻辑运算后的结果是什么，则我们可以不计算右操作数。

```
false and sideEffect();
```

For an and expression to evaluate to something truthy, both operands must be truthy. We can see as soon as we evaluate the left false operand that that isn’t going to be the case, so there’s no need to evaluate sideEffect() and it gets skipped.

表达式1 and 表达式1 ,如果想要结果为真，则表达式1，表达式2都必须为真，所以，上面的例子中，左操作数结果是false，则整个逻辑表达式结果是false，我们不需要计算右操作数。

This is why we didn’t implement the logical operators with the other binary operators. Now we’re ready. The two new operators are low in the precedence table. Similar to || and && in C, they each have their own precedence with or lower than and. We slot them right between assignment and equality.

这就是为什么，我们不用其他的二元运算符来实现逻辑运算符，但是，现在我们准备好了，这两个运算符优先级比较低，和C语言中的 || && 差不多

```

expression     → assignment ;
assignment     → IDENTIFIER "=" assignment
               | logic_or ;
logic_or       → logic_and ( "or" logic_and )* ;
logic_and      → equality ( "and" equality )* ;

```


Instead of falling back to equality, assignment now cascades to logic_or. The two new rules, logic_or and logic_and, are similar to other binary operators. Then logic_and calls out to equality for its operands, and we chain back to the rest of the expression rules.

赋值不会返回到 equality, 而是级联到logic_or, 这两个新的规则，logic_or 和 logic_and, 和其他的二元运算符相似，然后，logic_and 会调用 equality，从而进入到其他的表达式中

We could reuse the existing Expr.Binary class for these two new expressions since they have the same fields. But then visitBinaryExpr() would have to check to see if the operator is one of the logical operators and use a different code path to handle the short circuiting. I think it’s cleaner to define a new class for these operators so that they get their own visit method.

我们可以复用现有的
