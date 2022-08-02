# Lox语言

> What nicer thing can you do for somebody than make them breakfast?
> 
> <p align="right">—— Anthony Bourdain</p>
>
> 还有什么比给别人做早餐更好的事情呢？

We’ll spend the rest of this book illuminating every dark and sundry corner of the Lox language, but it seems cruel to have you immediately start grinding out code for the interpreter without at least a glimpse of what we’re going to end up with.

At the same time, I don’t want to drag you through reams of language lawyering and specification-ese before you get to touch your text editor. So this will be a gentle, friendly introduction to Lox. It will leave out a lot of details and edge cases. We’ve got plenty of time for those later.

我们将在本书的剩余部分，阐明Lox语言的每一个黑暗和杂乱的角落。但是，如果我们无法看到 Lox 语言的最终样子，就开始编写解释器代码，看起来有一些匆忙。

在你使用文本编辑器之前，我不想过多介绍语言的格式和规范，所以，本章将是一个温和友好的Lox介绍。它会遗漏很多细节和边缘部分，我们在后面有足够的时间去学习。

> A tutorial isn’t very fun if you can’t try the code out yourself. Alas, you don’t have a Lox interpreter yet, since you haven’t built one!
>
> Fear not. You can use mine.
>
> 如果一个教程不能自己写代码，那么这个教程就没有吸引力。但是，你还没有一个Lox解释器，因为我们还没有去实现它。
>
> 不用担心，你可以先使用我的😄。

## 一、Hello, Lox

第一个程序

```c

// Your first Lox program!
print "Hello, world!";

```

Here’s your very first taste of Lox:

As that // line comment and the trailing semicolon imply, Lox’s syntax is a member of the C family. (There are no parentheses around the string because print is a built-in statement, and not a library function.)

Now, I won’t claim that C has a great syntax. If we wanted something elegant, we’d probably mimic Pascal or Smalltalk. If we wanted to go full Scandinavian-furniture-minimalism, we’d do a Scheme. Those all have their virtues.

What C-like syntax has instead is something you’ll often find more valuable in a language: familiarity.  I know you are already comfortable with that style because the two languages we’ll be using to implement Lox—Java and C—also inherit it. Using a similar syntax for Lox gives you one less thing to learn.

上面是第一个Lox程序，

正如 // 行注释，行尾；所暗示的，Lox语言，继承了C语言语法。hello, world 字符串周围不需要括号，因为 print是一个内置语句，而不是一个库函数。

现在，我不会说C语言有很好的语法，如果我们想要优雅的东西，可能模仿Pascal、Smalltalk更加合适。如果我们想要完全实现 斯堪的纳维亚家具的极简主义，我们需要先做一个计划。这些都有它们的优势。

相反，我们使用类C语言语法，是因为我们可以从中获得更有价值的东西，熟悉度。我知道，你已经习惯了这种风格，接下来，我们用以实现Lox的两种语言，Java 和C语言，都拥有这种熟悉的风格。Lox语言使用这种风格的语法，可以更加容易入门。

> Your first taste of Lox, the language, that is. I don’t know if you’ve ever had the cured, cold-smoked salmon before. If not, give it a try too.
>
> 你第一次品尝Lox语言，就是这样。我不知道你之前有没有尝试过腌制的冷鲑鱼，如果没有，可以尝试一下。

> I’m surely biased, but I think Lox’s syntax is pretty clean. C’s most egregious grammar problems are around types. Dennis Ritchie had this idea called “declaration reflects use”, where variable declarations mirror the operations you would have to perform on the variable to get to a value of the base type.
>
>Lox doesn’t have static types, so we avoid that.
>
> 我是有偏见的，从我的角度，我认为Lox语言语法非常简洁。C语言令人惊讶的语法问题是，类型。Dennis Ritchie 提出一个“声明反映使用”的想法，其中变量声明反映了你对于变量执行的操作，以获得基类型的值。这个想法非常好，但我认为，在实践中的效果并不好。
>
> Lox不是静态类型语言，所以我们避免了这种情况。

