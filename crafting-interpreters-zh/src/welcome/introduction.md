# 介绍

  > fairy tales are more than true: not because they tell us that dragons exist, but because they tell us that dragons can be beaten. 
  >
  > <p align="right"> —— G.K. Chesterton by way of Neil Gaiman, Coraline </p>
  >
  > 童话绝不仅是真实的: 不仅仅在于它告诉我们龙的存在，更在于它告诉我们勇士可以战胜恶龙。
  >
  > <p align="right"> —— G.K. Chesterton by way of Neil Gaiman, Coraline </p>
  
  
  I'm really excited we're going on this journey together. This is a book on implementing interpreters for programming languages. It's also a book on how to design a language worth implementing. It's the book I wish I'd had when I first started getting into languages, and it's the book I've been writing in my head for nearly a decade.
  
  非常高兴我们可以一起开启新的旅程，
  
  这是一本介绍编程语言解释器的书， 这本书还会介绍如何自己实现一门语言。
  
  这本书在我脑海中已经反复编辑了十年了，多么希望在我刚接触编程语言时候能遇到这种书籍。🐶
  
  > to my friends and family, sorry I've been so absentminded!
  >
  > 对于我的朋友和家人，很抱歉我一直心不在焉。
  
  In these pages, we will walk step-by-step through two complete interpreters for a full-featured language. I assume this is your first foray into languages, so I'll cover each concept and line of code you need to build a complete, usable, fast language implementation.
  
  In order to cram two full implementations inside one book without it turning into a doorstop, this text is lighter on theory than others. As we build each piece of the system, I will introduce the history and concepts behind it. I'll try to get your familiar with the lingo so that if you ever find yourself at a cocktail party full of PL(parogramming language) researchers, you'll fit in.
  
接下来，我们将通过实现两个完整的编译器，去实现一个功能齐全的语言。我会假设你第一次接触语言编程，所以我会详细介绍每一个概念和列出详细的代码，而这一切将会构建完整、可用、快速的语言。

为了在一本书中，实现两个编译器，相对于其他的编译语言书籍，我们会更少介绍编译原理，我们不想成为介绍编译理论的拦路虎。在构建编译系统的每一个部分，我将介绍其背后的历史和概念。我将尽可能使用行话，这样即使将来你出现在一个编程语言的鸡尾酒会，也可以快速融入其中。
  
  > Strangely enough, a situation I have found myself in multiple times. You wouldn't believe how much some of them can drink.
  >
  > 奇怪的是，我发现自己多次陷入这种场景。你不应该假想他们都很能喝。
  
  But we're mostly going to spend our brain juice getting the language up and running. This is not to say theory isn't important. Being able to reason precisely and formally about syntax and semantics is a vital skill when working on a language. But, presonally, I learn best by doing. It's hard for me to wade through paragraphs full of abstract concepts and really absorb them. But if I've coded something, run it, and debugged it, then I get it.

我们将花费精力去开发运行语言，而这并不代表理论不重要。在学习语言时候，掌握语法和语义规则非常重要。但是，个人经验，我总是从实践中获取更多的东西，我通常很难看懂或者真的理解充满抽象概念的段落。但是，但我代码中编码、运行、调试过某个概念，我将能真正掌握它。
  
  > Static type systems in particular require rigorous formal reasoning. Hacking on a type system has the same feel as proving a theorem in mathematics. 
  >
  > 静态类型系统，尤其需要严格的形式推理。在类型系统上进行编程，非常像是证明一个数学定理。
  
  > It turns out this is no coincidence. In the early half of last century, Haskell Curry and William Alvin Howard showed that they are two sides of the same coin: [the Curry-Howard isomorphism](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)
  >
  > 事实证明，这个感觉并非是巧合。在20世纪上半叶，Haskell Curry 和 William Alvin Howard严格证明了编程语言和数学证明之间的关系，称为[柯里-霍华德同构](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)
  
  That's my goal for you. I want you to come away with a solid intuition of how a real language lives and breathes. My hope is that when you read other, more theoretical books later, the concepts there will firmly stick in your mind, adhered to this tangible substrate.
  
 这个本书的一个目标，可以让你更加真切的认识一门语言，可以凭直觉感知一门语言如何编写运行。希望当以后学习更多的编译原理时候，通过现在培养的直觉，可以牢牢记住书中的概念。
  
  ## 一、Why learn this stuff?
  
为什么要学习这些东西？
  
  Every introduction to every compiler book seems to have this section. I don't know what it is about programming languages that causes such existential doubt. I don't think ornithology books worry about justifying their existence. They assume the reader loves birds and start teaching.
  
  But programming languages are a little different. I suppose it is true that the odds of any of us creating a broadly successful, general-purpose programming language are slim. The designers of the world's widely used languages could fit in a Volkswagen bus, even without putting the pop-top camper up. If joining that elite group was the only reason to learn languages, it would be hard to justify. Fortunately, it isn't.
  
似乎每本编译器介绍书籍，似乎都包含这个讨论。我不知道，为什么编程语言会出现这样的疑惑。鸟类学研究书籍从来不认为人们会怀疑它是否该存在，他们总是假设读者喜欢鸟，然后开始教学。

但是编程语言总是有点不一样。我认为，我们大部分人，都无法创建一个广泛成功的通用编程语言。世界上最广泛使用的语言设计者可以适应驾驶大众汽车，即使车上没有安装流行的露营设备。如果学习编译器，仅仅是为了加入这个编程精英群体，那么大可不必。幸运的是，事实也并非如此。

### 1.1 Little language are everywhere

For every successful general-purpose language, there are a thousand successful niche ones. We used to call them “little languages”, but inflation in the jargon economy led to the name “domain-specific languages”.These are pidgins tailor-built to a specific task. Think application scripting languages, template engines, markup formats, and configuration files.

Almost every large software project needs a handful of these. When you can, it’s good to reuse an existing one instead of rolling your own.

A random selection of some little languages you might run into.

![A random selection of some little languages you might run into.](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/little-languages.png?raw=true)

But there’s still a good chance you’ll find yourself needing to whip up a parser or other tool when there isn’t an existing library that fits your needs. Even when you are reusing some existing implementation, you’ll inevitably end up needing to debug and maintain it and poke around in its guts.
  

对于每一种通用的流行语言，都对应着成千小众语言。我们可以称为这些语言为小众语言，但是在行业术语中，通常人们会使用领域特定语言(DSL)来描述。它们都是为了执行特定任务设计，想象一下，脚本语言、模版引擎、标记格式、配置文件。

几乎每个大型项目中，我们总会使用上图中的部分语言。通常我们会复用已经出现的语言，而不是自己造轮子。但是，当你考虑到文档、调试、编辑器支持、语法高亮和其他类似功能，就需要自己动手了。😄

当没有现有库匹配新需求时候，可能需要开发新的解析器或者一些小工具。即使对于某些正在使用的第三方库，也会需要不断进行调试和维护，并且需要对其深入研究。

### 1.2 Languages are great exercise

语言是好的锻炼

Long distance runners sometimes train with weights strapped to their ankles or at high altitudes where the atmosphere is thin. When they later unburden themselves, the new relative ease of light limbs and oxygen-rich air enables them to run farther and faster.

Implementing a language is a real test of programming skill. The code is complex and performance critical. You must master recursion, dynamic arrays, trees, graphs, and hash tables. You probably use hash tables at least in your day-to-day programming, but do you really understand them?

While I intend to show you that an interpreter isn’t as daunting as you might believe, implementing one well is still a challenge.

长跑运动员会在训练时候，在脚踝上绑上重物，或者在高海拔、空气稀薄地区训练。当他们卸下负重时候，相对轻松的四肢和正常的氧气，使得他们，可以跑得更快更远。

实现一门语言是对编程技能的真正测试。代码会非常复杂，而且性能也很重要。你必须掌握递归算法、动态数组、树、图和哈希表。可能你每日的编程代码中都会使用哈希表，但是你真的理解它吗？好吧，让我们从头开始，我保证你一定可以学会的。

虽然我想向你们展示，编译器并不像我们认为的那么可怕，但是实现一个好的编译器仍然是一个有挑战的任务。当你做到了这一点，你会变得更加强大，并且在日常开发工作中，使用数据结构和算法会更加熟练。

### 1.3 One more reason

另一个原因

This last reason is hard for me to admit, because it’s so close to my heart. Ever since I learned to program as a kid, I felt there was something magical about languages. When I first tapped out BASIC programs one key at a time I couldn’t conceive how BASIC itself was made.

Later, the mixture of awe and terror on my college friends’ faces when talking about their compilers class was enough to convince me language hackers were a different breed of human—some sort of wizards granted privileged access to arcane arts.

It’s a charming image, but it has a darker side. I didn’t feel like a wizard, so I was left thinking I lacked some inborn quality necessary to join the cabal. Though I’ve been fascinated by languages ever since I doodled made-up keywords in my school notebook, it took me decades to muster the courage to try to really learn them. 

When I did finally start cobbling together my own little interpreters, I quickly learned that, of course, there is no magic at all.

There are a few techniques you don’t often encounter outside of languages, and some parts are a little difficult.  But not more difficult than other obstacles you’ve overcome. My hope is that if you’ve felt intimidated by languages and this book helps you overcome that fear, maybe I’ll leave you just a tiny bit braver than you were before.

And, who knows, maybe you will make the next great language. Someone has to.

最后一个原因我本人很难承认，因为它一直深藏我心底。当我小时候学会接触编程，我就觉得编程语言非常神奇。当我第一次一个按键一个按键敲出 BASIC 程序时候，我无法想象 BASIC 内部是如何运行的。

后来，当我的朋友们谈论起编译器课程时候，他们脸上充满了敬畏和恐惧，这加深了我的想法：编译器黑客是另外一种人类，一些巫师才能拥有的天赋。

这是一个迷人的形象，但是它也有黑暗的一面，我不觉得自己是一个巫师，所以我不认为自己拥有巫师的天赋。虽然从我在学校笔记本上，乱写关键词时候，就对于编程语言十分着迷，但是，我花了几十年时间才真正开始学习编译器。编译器的那种神奇之处、不简单的感觉，让我一直徘徊在门外。

当我开始编写自己的编译器时候，很快就明白了，这个领域根本没有魔法。仅仅只是代码，识别代码的也都是普通人。

有一些语言之外的技巧，我们不会经常遇到，但是这一部分有些难度。但是，也不会比你所克服的其他障碍更加棘手。我的希望是，如果你和我一样，对编译器感到恐惧，那么这本书将帮助你克服这个恐惧，也许这之后，你将变得更加勇敢。

最后，你还可能成为下一个伟大语言的创始人，必须有人去做。谁知道呢？

  
  ## 二、How the book is organized
  
  本书的组织方式
  
  This book is broken into three parts. You're reading the first one now. It's a couple of chapters to get you oriented, teach you some of the lingo that language hackers use, and introduce you to Lox, the language we'll be implementing.
  
  Each of the other two parts builds one complete Lox interpreter. Within those parts, each chapter is structured the same way. The chapter takes a single language feature, tachers you the concepts bebind it, and walks you through an implementation.
  
  It took a good bit of trial and error on my part, but I managed to carve up the two interpreters into chapter-sized chunks that build on the previous chapters but require nothing from later ones. From the very first chapter, you'll have a working program you can run and play with. With each passing chapter, it grows increasing full-featured until you eventually have a complete language.
  
  Aside from copious, scintillating English prose, chapters have a few other delightful facets.
  
  这本书将分为三个部分，现在正在阅读的是第一部分。这几章会让你有一些方向感，教你使用黑客常用的术语。然后，会介绍Lox语言，我们将要实现的语言。
  
  其他两个部分，会分别实现一个完整的编译器。在这两个部分，每一章的结构都是相同的，本章节采用单一语言功能，向您介绍背后的概念，并且引导你去实现解析器。
  
对我来说，需要一些尝试和试错，但我还是把两个编译器分为章节大小的部分，这些章节基于前面几章的基础知识，不需要理解后面几章的内容。从第一章开始，你就拥有了一个可以运行和使用的工作程序。随着更多章节的学习，这个工作程序会越来越全面，直到最终，你会拥有一个完整的编程语言。

除了丰富、华丽的正文外，每个章节还有一些令人愉快的部分。
  
### 2.1 The code
  
We're about crafting interpreters, so this book contains real code. Every single line of code needed is included, and each snippet tells you where to insert it in your ever-growing implementation.
  
Many other language books and language implementations use tools like [Lex](https://en.wikipedia.org/wiki/Lex_(software)) and [Yacc](https://en.wikipedia.org/wiki/Yacc), so-called **compiler-compilers**, that automatically generate some of the source files for an implementation from some higher-level description. There are pros and cons to tool like those, and strong opinions——some might say religious convictions —— on both sides.
  
We will abstain from using them here. I want to ensure there are no dark corners where magic and confusion can hide, so we’ll write everything by hand. As you’ll see, it’s not as bad as it sounds, and it means you really will understand each line of code and how both interpreters work.
  
A book has different constraints from the “real world” and so the coding style here might not always reflect the best way to write maintainable production software. If I seem a little cavalier about, say, omitting private or declaring a global variable, understand I do so to keep the code easier on your eyes. The pages here aren’t as wide as your IDE and every character counts.

Also, the code doesn’t have many comments. That’s because each handful of lines is surrounded by several paragraphs of honest-to-God prose explaining it. When you write a book to accompany your program, you are welcome to omit comments too. Otherwise, you should probably use // a little more than I do.

While the book contains every line of code and teaches what each means, it does not describe the machinery needed to compile and run the interpreter. I assume you can slap together a makefile or a project in your IDE of choice in order to get the code to run. 
  
  我们是介绍编译器的，所以本书会包含真是可用的代码。每行代码，每个代码段，都会告诉你它们的作用和在实现的不断完善的编译器中的位置。
  
  许多其他语言书籍和语言实现书籍中，通常会使用 Lex，Yacc等称为编译编译器的编译语言，这些语言，可以从更高级的描述中自动生成源文件。 直接使用这些语言工具，有好处也有弊端，而对于这两个观点，都有很多宗教信仰般的拥趸。
  
  本书中将避免使用编译器语言，我想要确保没有黑暗的角落，隐藏着一些魔法和未知，所以我们将手写所有内容。正如你看到的，这并不像听起来那么糟糕，这意味着你将真正理解每一行代码，并且真的理解这两个编译器是如何工作的。
  
  一本书不同于真实世界的约束，因此本书的编码风格可能并不是编写可维护性生产级别软件的最佳实践。如果我省略了 private 或者 忽略了全局变量声明，请理解我这么做是为了让代码更加容易理解，而不是来源于傲慢。这里的页面没有 IDE 那么宽，但是它们都非常重要。
  
此外，代码没有太多的注释，这是因为每段代码上下文，都是对其的大段正文说明。当你自己编写一本代码实现的书籍时候，也欢迎你省略其中的注释。当然，你也可以使用更多的 //

虽然这本书包含了编译器的每一行代码，介绍了每一行代码的含义，但是我并没有描述编译和运行代码的具体机器信息。我预想你可以使用自己熟悉的 IDE 创建一个文件或是一个项目。这些具体机器说明很快就会过时，我希望这本书可以像 XO白兰地一样历久弥新，而不是像 backyard hooch（一直保质期不长的蜂蜜酒）一样很快过时。

![yak](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/yak.png?raw=true)

> Yacc is a tool that takes in a grammar file and produces a source file for a compiler, so it’s sort of like a “compiler” that outputs a compiler, which is where we get the term “compiler-compiler”.
>
> Yacc wasn’t the first of its ilk, which is why it’s named “Yacc”—Yet Another Compiler-Compiler. A later similar tool is Bison, named as a pun on the pronunciation of Yacc like “yak”.
>
> If you find all of these little self-references and puns charming and fun, you’ll fit right in here. If not, well, maybe the language nerd sense of humor is an acquired taste.
>
> Yacc 是一个工具，可以接收语法文件，然后为编译器生成源文件。所以，它好像是一个可以输出"编译器"的编译器，一般使用术语"编译器编译程序"形容 Yacc
>
> Yacc并不是第一个"编译器编译程序", 这就是这个程序缩写的来源：另一个编译器编译程序。还有一个相似的工具，称为 Bison（野牛), 命名来源于 Yacc的双关语 yak （牦牛)。
>
> 如果你发现，这些小小的自我暗示和双关语，很有魅力和乐趣，你将很快适应这里。如果你对此并不感冒，可能书呆子的幽默感是后天养成的。

## 2.2 Snippets

代码段

Since the book contains literally every line of code needed for the implementations, the snippets are quite precise. Also, because I try to keep the program in a runnable state even when major features are missing, sometimes we add temporary code that gets replaced in later snippets.

A snippet with all the bells and whistles looks like this:

In the center, you have the new code to add. It may have a few faded out lines above or below to show where it goes in the existing surrounding code. There is also a little blurb telling you in which file and where to place the snippet. If that blurb says “replace _ lines”, there is some existing code between the faded lines that you need to remove and replace with the new snippet.

```java

	// lox/Scanner.java in scanToken() replace 1 line
	
	default:
        if (isDigit(c)) {
          number();
        } else {
          Lox.error(line, "Unexpected character.");
        }
        break;
		
```

因为本书中包含有所有编译器的实现代码，所以书中涉及的代码段非常准确。此外，因为我试图让程序即使在缺少主要功能的情况下，也可以保持可运行状态。所以，有时候我会添加临时代码，在之后章节会替换这些临时代码。

代码段示例如下:

中间代码是实际要添加的代码，上面和下面的代码表示要添加代码的位置。还有一个小提示，告诉你这段代码添加到哪个文件的哪个函数中。如何小提示上面写了 "替换该行", 表示新增的代码需要替换之前的代码。

## 2.3 Asides

Asides contain biographical sketches, historical background, references to related topics, and suggestions of other areas to explore. There’s nothing that you need to know in them to understand later parts of the book, so you can skip them if you want. I won’t judge you, but I might be a little sad.

Well, some asides do, at least. Most of them are just dumb jokes and amateurish drawings.

旁白包含了传记历史、历史背景、相关主题的参考文献以及推荐的延伸阅读。如果是为了阅读后面章节的内容，你不需要阅读旁白，也就是说你可以直接跳过旁白。我不会批评你，但是我会感到难过 😫。

嗯，还有些旁白不尽如人意，这些旁白大部分都是愚蠢的笑话和业余水平的绘画。

## 2.4 Challenges

Each chapter ends with a few exercises. Unlike textbook problem sets, which tend to review material you already covered, these are to help you learn more than what’s in the chapter. They force you to step off the guided path and explore on your own. They will make you research other languages, figure out how to implement features, or otherwise get you out of your comfort zone.

Vanquish the challenges and you’ll come away with a broader understanding and possibly a few bumps and scrapes. Or skip them if you want to stay inside the comfy confines of the tour bus. It’s your book.

A word of warning: the challenges often ask you to make changes to the interpreter you’re building. You’ll want to implement those in a copy of your code. The later chapters assume your interpreter is in a pristine (“unchallenged”?) state.

每个章节都以一些练习题结束。但是不同于教科书的习题集，它们通常让你复习已经学过的知识，本书的习题更加倾向于让你学到更多的东西。这些习题会让你离开章节的已有路径，独自探索更多的未知领域。它们会让你去研究其他语言，去寻找如何实现功能，或者让你走出舒适区。

接受习题集的挑战，并且战胜它们，你将会有更加广阔的视野，虽然你可能会遇到一些坎坷和挫折。如果你想要呆在舒服的游览车内，你也可以跳过这些挑战习题，毕竟这是你的书。

一句警告⚠️，这些挑战通常会要求你修改已构建的编译器，建议你在副本项目中实现这些挑战，我们后面的章节基于没有修改的编译器。

## 2.5 Design notes

设计思路

Most “programming language” books are strictly programming language implementation books. They rarely discuss how one might happen to design the language being implemented. Implementation is fun because it is so precisely defined. We programmers seem to have an affinity for things that are black and white, ones and zeroes.

Personally, I think the world needs only so many implementations of FORTRAN 77. At some point, you find yourself designing a new language. Once you start playing that game, then the softer, human side of the equation becomes paramount.  Things like which features are easy to learn, how to balance innovation and familiarity, what syntax is more readable and to whom.

All of that stuff profoundly affects the success of your new language. I want your language to succeed, so in some chapters I end with a “design note”, a little essay on some corner of the human aspect of programming languages. I’m no expert on this—I don’t know if anyone really is—so take these with a large pinch of salt. That should make them tastier food for thought, which is my main aim.

I know a lot of language hackers whose careers are based on this. You slide a language spec under their door, wait a few months, and code and benchmark results come out. Hopefully your new language doesn’t hardcode assumptions about the width of a punched card into its grammar.

大多数的编程语言书籍，都是严格意义上的编写代码实现，他们很少讨论如何设计一门语言、设计一段实现。实现非常有趣，因为它是确定的，而且被精确定义。我们程序员似乎对于确定性的东西，例如：黑与白，1与0，天生有好感。

就我个人而言，我认为世界上只需要 Fortran77 （1976年，美国标准化协会重新对Fortran（x3.9-1966）进行了评估，公布了新的Fortran标准，也就是Fortran 77。Fortran 77是具有结构化特性的编程语言。Fortran77在短时间内获取了巨大的成功，广泛地应用于科学和工程计算，几乎统治了数值计算领域。）中实现的功能。但是在某些时候，你发现自己在设计一门新语言。一旦你开始设计语言这个游戏，那么人性化的一面变得非常重要。需要考虑更加具体的内容，例如：哪些功能更易于学习，如何平衡创新和保留，哪些语法更易于阅读，学习语言的人群是谁？

所有这些都深刻的影响着一门新语言的成功与否。我希望你的新语言可以成功，所以我会在某些章节的结尾，发表一些设计方面的思考，这是一些编程语言中人们设计的最佳实践。我不是设计方面的专家，不知道是否有人真的这么想—在一些编程语言的正文中，添加一些设计方面的内容。这应该会让设计内容称为更加美味的思想食粮，这也是我的主要目标。

我认识很多编程老手，他们的职业就在于此——你将一门语言的规范告知他们，等待几个月，你将会得到这门新语言的代码和基准测试结果。希望你的新语言，不会将穿孔纸的宽度信息，硬编码到语言语法中。

# 三、The First Interpreter

第一个编译器

We’ll write our first interpreter, jlox, in Java. The focus is on concepts. We’ll write the simplest, cleanest code we can to correctly implement the semantics of the language. This will get us comfortable with the basic techniques and also hone our understanding of exactly how the language is supposed to behave.

The book uses Java and C, but readers have ported the code to many other languages. If the languages I picked aren’t your bag, take a look at those.

Java is a great language for this. It’s high level enough that we don’t get overwhelmed by fiddly implementation details, but it’s still pretty explicit. Unlike in scripting languages, there tends to be less complex machinery hiding under the hood, and you’ve got static types to see what data structures you’re working with.

我们将使用JAVA语言实现第一个编译器，jlox，第一个编译器的重点是基本概念，我们将编写最简洁、最基础的代码，实现编译器的语义。这将让我们熟悉基本的技术，让我们能更加准确理解语言行为。

本书将使用 JAVA, C语言，但是读者可能更加熟悉其他的编程语言，如果我使用的语言不是你的菜，可以尝试使用你最熟悉的语言。

JAVA是一门很好的语言，我们不需要关注底层的大量实现细节，因为JAVA是一门高级语言，拥有更多的确定性。和脚本语言不同的是，JAVA与不同机器的关联性并没有那么复杂，通常，你可以使用静态类型去获取查看当前正在使用的结构体。

I also chose Java specifically because it is an object-oriented language.  That paradigm swept the programming world in the ’90s and is now the dominant way of thinking for millions of programmers. Odds are good you’re already used to organizing code into classes and methods, so we’ll keep you in that comfort zone.

While academic language folks sometimes look down on object-oriented languages, the reality is that they are widely used even for language work. GCC and LLVM are written in C++, as are most JavaScript virtual machines. Object-oriented languages are ubiquitous, and the tools and compilers for a language are often written in the same language.

And, finally, Java is hugely popular. That means there’s a good chance you already know it, so there’s less for you to learn to get going in the book. If you aren’t that familiar with Java, don’t freak out. I try to stick to a fairly minimal subset of it.  I use the diamond operator from Java 7 to make things a little more terse,  but that’s about it as far as “advanced” features go. If you know another object-oriented language, like C# or C++, you can muddle through.

By the end of part II, we’ll have a simple, readable implementation. It’s not very fast, but it’s correct. However, we are only able to accomplish that by building on the Java virtual machine’s own runtime facilities. We want to learn how Java itself implements those things.

我们选择Java，还因为它是一门面向对象的语言。这种编程范式在90年代席卷了整个世界，现在也是数百万程序员的主流思维方式。很可能你已经习惯了将代码组织成类和方法，接下来我们也会让你处于舒适区中。

虽然，学术研究语言的人们，有时候看不起面向对象的编程语言，但是实际情况是，即使在他们日常编程工作中，也会广泛使用面向对象语言。GCC/LLVM 还有大多数的 JavaScript 虚拟机都是使用面向对象的C++语言实现的。面向对象语言无处不在，一门语言的编译器和工具，通常会使用相同的语言实现。

最后，Java非常流行。这意味着，这是一个很好机会去使用熟悉这门语言。如果你不熟悉Java，也不用担心，本书中只会使用一小部分功能，我将使用Java7 标准里面的运算符，相对于使用更多的高级用法，这会让编程变得更加简洁。如果你还熟悉其他的面向对象语言，例如: C#, C++, 你也可以尝试使用它们去实现编译器。

在第二部分结束时候，我们将有一个简单易读的编译器实现，它的运行性能不是很高，但是可以保证准确性。但是，我们实现的编译器，是基于Java 语言和 Java 虚拟机底层。我们想要了解 Java本身是如何实现这些功能的。


A compiler reads files in one language, translates them, and outputs files in another language.
You can implement a compiler in any language, including the same language it compiles, a process called self-hosting.

You can’t compile your compiler using itself yet, but if you have another compiler for your language written in some other language, you use that one to compile your compiler once. Now you can use the compiled version of your own compiler to compile future versions of itself, and you can discard the original one compiled from the other compiler. This is called bootstrapping, from the image of pulling yourself up by your own bootstraps.

![bootstraps](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/bootstrap.png?raw=true)

编译器读取一种语言的原始文件，翻译它们，然后输出另外一门语言的文件。你可以用任何语言实现编译器，甚至你可以使用相同的语言去实现该语言的编译器，这通常被称为自编译。

一开始，你无法使用相同的语言写成一个编译器，但是如果你已经使用过其他语言实现了新语言的编译器，那么你就可以使用已经实现的编译器，去编译一次，然后你可以获取到使用新语言实现的编译器。接下来，你可以愉快的使用新的编译器，去升级新的编译器版本，也可以扔掉之前的其他语言写成的编译器了。专业术语一般称为自举，图中形象的描述了一个人的自举。

# 四、The Second Interpreter

第二个编译器

So in the next part, we start all over again, but this time in C. C is the perfect language for understanding how an implementation really works, all the way down to the bytes in memory and the code flowing through the CPU.

A big reason that we’re using C is so I can show you things C is particularly good at, but that does mean you’ll need to be pretty comfortable with it. You don’t have to be the reincarnation of Dennis Ritchie, but you shouldn’t be spooked by pointers either.

If you aren’t there yet, pick up an introductory book on C and chew through it, then come back here when you’re done. In return, you’ll come away from this book an even stronger C programmer. That’s useful given how many language implementations are written in C: Lua, CPython, and Ruby’s MRI, to name a few.

In our C interpreter, clox, we are forced to implement for ourselves all the things Java gave us for free. We’ll write our own dynamic array and hash table. We’ll decide how objects are represented in memory, and build a garbage collector to reclaim them.

Our Java implementation was focused on being correct. Now that we have that down, we’ll turn to also being fast. Our C interpreter will contain a compiler that translates Lox to an efficient bytecode representation (don’t worry, I’ll get into what that means soon), which it then executes. This is the same technique used by implementations of Lua, Python, Ruby, PHP, and many other successful languages.

We’ll even try our hand at benchmarking and optimization. By the end, we’ll have a robust, accurate, fast interpreter for our language, able to keep up with other professional caliber implementations out there. Not bad for one book and a few thousand lines of code.

在下个部分中，我们将从头开始，但是这一次将使用C语言。C语言可以让我们更好的理解计算机是如何工作的，深入到底层，例如：内存中的字节和 cpu 执行的代码。

我们使用C语言实现编译器，的一个重要原因是，我可以使用C语言一些特别擅长的功能，但是这也意味着你必须非常熟悉C语言。你不需要像创始人 Dennis Ritchie 一样熟悉 C语言，但是你至少不能被指针吓倒。

如果你还不太了解C语言，那么先拿起一本C语言入门书籍仔细阅读后，再回到这里。作为回报，你将变成一个更强大的C语言程序员。你可以先看看有哪些语言是基于C语言实现的：Lua，CPython，Ruby的MRI实现，等等。

在C语言实现的解释器 clox 中，我们将要实现一些 Java中原生存在的结构，例如：我们将实现动态数组和哈希表，我们将设计决定如何在内存中表示对象，并且构建垃圾采集器回收它们。

第一部分，我们用Java语言实现的编译器 jlox主要专注于准确性，现在我们已经实现了准确性，接下来将专注于性能。我们的clox解释器，将实现一个编译器，将lox 编译为有效的字节码（别担心，很快我将解释它是什么），然后执行字节码。这与其他语言的实现使用相同技术，例如：Lua，Python，Ruby，PHP等等。

我们甚至会尝试基准测试和优化。到最后，我们将实现一个强大、准确、快速的语言解释器，能够和其他专业级别的语言媲美，而这对于一本书和几千行代码而言，并不简单。



# 五、 Challenges
	
习题
  
1. There are at least siz domain-specific languages used in the [little system I cobbled together](https://github.com/munificent/craftinginterpreters) to write and publish this book. What are they?
	
   在我编写本书中，至少使用了6种小众语言，请列举中它们？
 
1. Get a "Hello, world!" program written and running in Java. Set up whatever makefiles or IDE projects you need to get it working. If you have a debugger, get comfortable with it and step through your program as it runs.

	使用 Java语言实现一个 "hello, world" 程序，描述一下你使用的IDE 和配置文件，并且在IDE中习惯调试。
  
1. Do the same thing for C. To get some practice with pointers, define a [doubly linked list](https://en.wikipedia.org/wiki/Doubly_linked_list) of heap-allocated strings. Write functions to insert, find, and delete items from it. Test them.

	使用C语言实现一个 "hello, world" 程序，为了练习指针，定义一个堆分配字符串的双链表，编写函数插入、查找、删除链表元素。
  
  
  
#  六、Design Note: What's in a name?

设计说明：如何命名？
  
  One of the hardest challenges in writing book was coming up with a name for the language it implements. I went through pages of candidates before I found one that worked. As you'll discover on the first day you start building your own language, naming is deviously hard. A good name satisfies a few criteria:
  
1. It isn't in use.

   You can run into all sorts of trouble, legal and social, if you inadvertently step on someone else’s name.
  
1. It's easy to pronounce. If things go well, hordes of people will be saying and writing your language’s name. Anything longer than a couple of syllables or a handful of letters will annoy them to no end.
  
1. It's distinct enough to search for. People will Google your language’s name to learn about it, so you want a word that’s rare enough that most results point to your docs. Though, with the amount of AI search engines are packing today, that’s less of an issue. Still, you won’t be doing your users any favors if you name your language “for”.
  
1. It doesn't have negative connotations across a number of cultures. This is hard to be on guard for, but it’s worth considering. The designer of Nimrod ended up renaming his language to “Nim” because too many people remember that Bugs Bunny used “Nimrod” as an insult. (Bugs was using it ironically.)

If your potential name makes it through that gauntlet, keep it. Don’t get hung up on trying to find an appellation that captures the quintessence of your language.
	
本书编写过程中一个挑战是，如何为实现的语言命名。我需要从众多的候选中，找到最合适的。正如你在构建自己语言时候会遇到的一样，命名非常困难，一个好的命名符合下面的标准：

1. 该名字之前没有被使用。如果你不小心使用了他人的命名，可能会有非常多的麻烦，包含法律和社会问题。

1. 该名字需要朗朗上口，如果一切顺利的话，会有非常多人书写、说出你的语言名称，而任何拗口的名字，都会给人们带来困惑。

1. 该名字需要足够特别，更容易搜索到。人们第一时间会使用搜索引擎了解你的语言，如果你使用一个足够特别的单词命名，那么大家将很容易获取到。尽管，随着搜索引擎拥有更多的AI能力，更容易显示你的语言。但是，如果你把自己的新语言命名为"for", 那么这通常不能带给使用者任何帮助。

1. 该名字在其他文化中没有负面含义。这一点很难防范，但是需要认真考虑。Nimrod 的设计者最终将这个语言命名为 "Nim", Bugs Bunny曾经故意使用有争议的名称 Nimrod 命名新的语言，人们依然记得。

如果你的候选命名，通过了上面的标准，那么留着它吧。不要试图去找到一个能抓住你的语言精髓的名称。如果说世界上其他已经成功的语言教会了我们什么的话，那就是语言名字和语言精髓没有关联。你所需要的只是一个，独特的名称。






  
  
  
