# 解析与绑定

Resolving and Binding

> Once in a while you find yourself in an odd situation. You get into it by degrees and in the most natural way but, when you are right in the midst of it, you are suddenly astonished and ask yourself how in the world it all came about.
> 
> 偶尔你会发现自己处于一种奇怪的境地，你以一种最自然的方式逐渐进入其中，但是当你置身其中时候，你突然感到惊讶，并问自己这一切到底是怎么发生的
> 
> <p align="right"> Thor Heyerdahl, Kon-Tiki (挪威作家托尔·海尔达尔,  Kon-Tiki探险：穿越南海的筏)</p>

Oh, no! Our language implementation is taking on water! Way back when we added variables and blocks, we had scoping nice and tight. But when we later added closures, a hole opened in our formerly waterproof interpreter. Most real programs are unlikely to slip through this hole, but as language implementers, we take a sacred vow to care about correctness even in the deepest, dampest corners of the semantics.

哦，不，我们实现的语言，现在正在水涨船高，很久以前，当我们添加变量和代码块语法时候，我们的作用域非常严密，但是，当我们后面添加闭包时候，我们以前的解释器，出现了泄漏漏洞。大多数的真正运行的程序，都不太可能遇到这个漏洞，但是作为语言的实现者，我们有一个设计标准，即使在语义的最底层、最薄弱的角落，我们也要关心解释的正确性。


We will spend this entire chapter exploring that leak, and then carefully patching it up. In the process, we will gain a more rigorous understanding of lexical scoping as used by Lox and other languages in the C tradition. We’ll also get a chance to learn about semantic analysis—a powerful technique for extracting meaning from the user’s source code without having to run it.

我们将用一整章来探讨这个漏洞，然后修复它。在这个过程中，我们将更加深入理解Lox和其他类C语言中，所使用的词汇作用域。我们还将有机会学习语义分析，这是一种从用户源代码中提取有用信息的强大技术，而这并不需要代码运行。

## 一、Static Scope

静态作用域

A quick refresher: Lox, like most modern languages, uses lexical scoping. This means that you can figure out which declaration a variable name refers to just by reading the text of the program. For example:

快速复习：与大多数现代语言一样，Lox使用词汇作用域。这意味着，只需要读取程序的文本，就可以找出变量名所值的声明。例如：

```java

var a = "outer";
{
  var a = "inner";
  print a;
}


```

Here, we know that the a being printed is the variable declared on the previous line, and not the global one. Running the program doesn’t—can’t—affect this. The scope rules are part of the static semantics of the language, which is why they’re also called static scope.

上面的代码，我们知道正在打印的a 是上一行声明的变量，而不是全局变量，运行程序不会影响这一点。作用域规则是语言静态语义的一部分，这就是为什么它们被称为静态作用域。

I haven’t spelled out those scope rules, but now is the time for precision:

我们还没有列出这些作用域规则，但是现在，是时候列出规则了

A variable usage refers to the preceding declaration with the same name in the innermost scope that encloses the expression where the variable is used.

变量的使用规则，变量是指，包含该变量的表达式的作用域中，最内部的具有相同名称的变量。

There’s a lot to unpack in that:


* I say “variable usage” instead of “variable expression” to cover both variable expressions and assignments. Likewise with “expression where the variable is used”.

* “Preceding” means appearing before in the program text.

还有一些需要东西需要解释

* 我使用名词，变量用法，而不是变量表达式，是为了包含变量表达式和赋值，与“使用变量的表达式”类似

* 前置，意味着声明出现在变量表达式的代码前面


```java


var a = "outer";
{
  print a;
  var a = "inner";
}

```

Here, the a being printed is the outer one since it appears before the print statement that uses it. In most cases, in straight line code, the declaration preceding in text will also precede the usage in time. But that’s not always true. As we’ll see, functions may defer a chunk of code such that its dynamic temporal execution no longer mirrors the static textual ordering.

上面的代码中，打印的a，表示外部的变量a，因为它出现在使用它的print语句之前。在大多数情况下，在直接式程序中，文本前面的声明也会比变量表达式，更早的运行。
