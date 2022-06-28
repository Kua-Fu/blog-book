# 介绍

  > fairy tales are more than true: not because they tell us that dragons exist, but because they tell us that dragons can be beaten. 
  
  > —— G.K. Chesterton by way of Neil Gaiman, Coraline 
  
  I'm really excited we're going on this journey together. This is a book on implementing interpreters for programming languages. It's also a book on how to design a language worth implementing. It's the book I wish I'd had when I first started getting into languages, and it's the book I've been writing in my head for nearly a decade.
  
  > to my friends and family, sorry I've been so absentminded!
  
  In these pages, we will walk step-by-step through two complete interpreters for a full-featured language. I assume this is your first foray into languages, so I'll cover each concept and line of code you need to build a complete, usable, fast language implementation.
  
  In order to cram two full implementations inside one book without it turning into a doorstop, this text is lighter on theory than others. As we build each piece of the system, I will introduce the history and concepts behind it. I'll try to get your familiar with the lingo so that if you ever find yourself at a cocktail party full of PL(parogramming language) researchers, you'll fit in.
  
  > Strangely enough, a situation I have found myself in multiple times. You wouldn't believe how much some of them can drink.
  
  But we're mostly going to spend our brain juice getting the language up and running. This is not to say theory isn't important. Being able to reason precisely and formally about syntax and semantics is a vital skill when working on a language. But, presonally, I learn best by doing. It's hard for me to wade through paragraphs full of abstract concepts and really absorb them. But if I've coded something, run it, and debugged it, then I get it.
  
  > Static type systems in particular require rigorous formal reasoning. Hacking on a type system has the same feel as proving a theorem in mathematics. 
  
  > It turns out this is no coincidence. In the early half of last century, Haskell Curry and William Alvin Howard showed that they are two sides of the same coin: [the Curry-Howard isomorphism](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)
  
  That's my goal for you. I want you to come away with a solid intuition of how a real language lives and breathes. My hope is that when you read other, more theoretical books later, the concepts there will firmly stick in your mind, adhered to this tangible substrate.
  
  ## 一、Why learn this stuff?
  
  Every introduction to every compiler book seems to have this section. I don't know what it is about programming languages that causes such existential doubt. I don't think ornithology books worry about justifying their existence. They assume the reader loves birds and start teaching.
  
  But programming languages are a little different. I suppose it is true that the odds of any of us creating a broadly successful, general-purpose programming language are slim. The designers of the world's widely used languages could fit in a Volkswagen bus, even without putting the pop-top camper up. If joining that elite group was the only reason to learn languages, it would be hard to justify. Fortunately, it isn't.
  
  
  ## 二、How the book is organized
  
  This book is broken into three parts. You're reading the first one now. It's a couple of chapters to get you oriented, teach you some of the lingo that language hackers use, and introduce you to Lox, the language we'll be implementing.
  
  Each of the other two parts builds one complete Lox interpreter. Within those parts, each chapter is structured the same way. The chapter takes a single language feature, tachers you the concepts bebind it, and walks you through an implementation.
  
  It took a good bit of trial and error on my part, but I managed to carve up the two interpreters into chapter-sized chunks that build on the previous chapters but require nothing from later ones. From the very first chapter, you'll have a working program you can run and play with. With each passing chapter, it grows increasing full-featured until you eventually have a complete language.
  
  Aside from copious, scintillating English prose, chapters have a few other delightful facets.
  
  ### 2.1 The code
  
  We're about crafting interpreters, so this book contains real code. Every single line of code needed is included, and each snippet tells you where to insert it in your ever-growing implementation.
  
  Many other language books and language implementations use tools like [Lex](https://en.wikipedia.org/wiki/Lex_(software)) and [Yacc](https://en.wikipedia.org/wiki/Yacc), so-called **compiler-compilers**, that automatically generate some of the source files for an implementation from some higher-level description. There are pros and cons to tool like those, and strong opinions——some might say religious convictions —— on both sides.
  
  ## 五、Challenges
  
  1. There are at least siz domain-specific languages used in the [little system I cobbled together](https://github.com/munificent/craftinginterpreters) to write and publish this book. What are they?
  
  
  2. Get a "Hello, world!" program written and running in Java. Set up whatever makefiles or IDE projects you need to get it working. If you have a debugger, get comfortable with it and step through your program as it runs.
  
  3. Do the same thing for C. To get some practice with pointers, define a [doubly linked list](https://en.wikipedia.org/wiki/Doubly_linked_list) of heap-allocated strings. Write functions to insert, find, and delete items from it. Test them.
  
  
  
  ##  六、Design Note: What's in a name?
  
  One of the hardest challenges in writing book was coming up with a name for the language it implements. I went through pages of candidates before I found one that worked. As you'll discover on the first day you start building your own language, naming is deviously hard. A good name satisfies a few criteria:
  
  1. It isn't in use.
  
  2. It's easy to pronounce.
  
  3. It's distinct enough to search for.
  
  4. It doesn't have negative connotations across a number of cultures.
  
  
  
