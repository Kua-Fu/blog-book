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

如果我们提供了一个算术表达式，那么可以很容易的转变为一棵树，同样的，如果有一棵树，我们也可以容易变为一个表达式。因此，从直觉上，我们可以将代码表示为一棵树，这棵树与我们使用的编程语言的语法结构、

