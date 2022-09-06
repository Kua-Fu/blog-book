# 扫描

> Take big bites. Anything worth doing is worth overdoing.
> 
>  <p align="right"> —— Robert A. Heinlein, Time Enough for Love  </p>
>
> 大口吃，任何值得做的事情，都值得做的过头。


The first step in any compiler or interpreter is scanning. The scanner takes in raw source code as a series of characters and groups it into a series of chunks we call tokens. These are the meaningful “words” and “punctuation” that make up the language’s grammar.

Scanning is a good starting point for us too because the code isn’t very hard—pretty much a switch statement with delusions of grandeur. It will help us warm up before we tackle some of the more interesting material later. By the end of this chapter, we’ll have a full-featured, fast scanner that can take any string of Lox source code and produce the tokens that we’ll feed into the parser in the next chapter.

任何编译器或者解释器的，第一个阶段都是扫描。扫描器将源代码当作一系列的字符序列，经过扫描后，会分组为一系列的token，这些token是，构成语言语法的基本单位，例如：英语中的单词和标点符号。

扫描，对于我们来说，是一个好的起点，因为扫描部分代码，并不复杂——是一个有很多 switch语句的代码。扫描部分，是我们后面处理更加有趣阶段的热身。在本章结束时候，我们将拥有一个功能齐全的快速扫描器，它可以根据输入的任意Lox语言源代码，生成我们后面阶段会使用的 token序列。

## 一、The Interpreter Framework

解释器架构

Since this is our first real chapter, before we get to actually scanning some code we need to sketch out the basic shape of our interpreter, jlox. Everything starts with a class in Java.

```java

// lox/Lox.java
package com.craftinginterpreters.lox;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;

public class Lox {
  public static void main(String[] args) throws IOException {
    if (args.length > 1) {
      System.out.println("Usage: jlox [script]");
      System.exit(64); 
    } else if (args.length == 1) {
      runFile(args[0]);
    } else {
      runPrompt();
    }
  }
}

```

Stick that in a text file, and go get your IDE or Makefile or whatever set up. I’ll be right here when you’re ready. Good? OK!

Lox is a scripting language, which means it executes directly from source. Our interpreter supports two ways of running code. If you start jlox from the command line and give it a path to a file, it reads the file and executes it.

```java

// lox/Lox.java

public class Lox {
    public static void main(String[] args) throws IOException {
		...
    }

    private static void runFile(String path) throws IOException {
        byte[] bytes = Files.readAllBytes(Paths.get(path));
        run(new String(bytes, Charset.defaultCharset()));
    }
}

```

If you want a more intimate conversation with your interpreter, you can also run it interactively. Fire up jlox without any arguments, and it drops you into a prompt where you can enter and execute code one line at a time.

```java

// lox/Lox.java
public class Lox {
    public static void main(String[] args) throws IOException {
		...
    }

    private static void runFile(String path) throws IOException {
		...
    }

    private static void runPrompt() throws IOException {
        InputStreamReader input = new InputStreamReader(System.in);
        BufferedReader reader = new BufferedReader(input);

        for (;;) {
            System.out.print("> ");
            String line = reader.readLine();
            if (line == null) {
                break;
            }
            run(line);
        }
    }
}
```

由于这是我们真正的开始章节，在我们实际扫描Lox代码之前，我们需要先描述 jlox 解释器的基本组成部分，一切从一个Java类开始。

将上面代码保存在文件中，可以选择你自己喜欢的IDE。

Lox语言是一种脚本语言，我的jlox解释器，支持两种运行方式，如果指定一个文件路径，jlox将读取并且运行该文件。

如果你想要与解释器，交互运行，可以运行交互模式。不携带任何参数运行jlox，将会进入交互模式，一次可以运行一行lox代码。

> An interactive prompt is also called a “REPL” (pronounced like “rebel” but with a “p”). The name comes from Lisp where implementing one is as simple as wrapping a loop around a few built-in functions: 
> 
> (print (eval (read)))
> 
> Working outwards from the most nested call, you Read a line of input, Evaluate it, Print the result, then Loop and do it all over again.
> 
> 交互式提示，也称为 REPL, (Read, Evaluate, Print, Loop) , 这个名称来源于 lisp语言，在lisp中，实现一个交互式，就好像使用内置多个函数和一个循环包装一样简单。
>
> (print (eval (read)))
> 
> 从最里面的代码开始，先读取一段代码，然后运行，最后打印结果，接下来，开始下一轮循环。

The readLine() function, as the name so helpfully implies, reads a line of input from the user on the command line and returns the result. To kill an interactive command-line app, you usually type Control-D. Doing so signals an “end-of-file” condition to the program. When that happens readLine() returns null, so we check for that to exit the loop.

Both the prompt and the file runner are thin wrappers around this core function:

```java
// lox/Lox.java

public class Lox {
    public static void main(String[] args) throws IOException {
		...
    }

    private static void runFile(String path) throws IOException {
		...
    }

    private static void runPrompt() throws IOException {
		...
    }

    private static void run(String source) {
        Scanner scanner = new Scanner(source);
        List<Token> tokens = scanner.scanTokens();

        for (Token token : tokens) {
            System.out.println(token);
        }
    }
}
```

readline() 函数，顾名思义，在命令行，读取用户输入的一行，然后返回运行结果。如果想要终止交互式程序，通常键入 Ctrl-D, 这样做，会向程序发出 文件已经结束信号，当这种情况发生后，readline() 函数，会返回null，我们程序中判断返回值，如果是null，退出循环，即退出交互式程序。

交互式或者从文件中读取，然后运行jlox 的核心函数是 run

It’s not super useful yet since we haven’t written the interpreter, but baby steps, you know? Right now, it prints out the tokens our forthcoming scanner will emit so that we can see if we’re making progress.

因为我们还没有开始编写解释器，所以jlox 还不是非常有用，但是你知道的，接下来，我们将实现扫描仪，run函数将返回扫描仪生成的token 列表，我们可以通过token序列，查看扫描器的运行效果。

### 1.1 Error handling

错误处理

While we’re setting things up, another key piece of infrastructure is error handling. Textbooks sometimes gloss over this because it’s more a practical matter than a formal computer science-y problem. But if you care about making a language that’s actually usable, then handling errors gracefully is vital.

The tools our language provides for dealing with errors make up a large portion of its user interface. When the user’s code is working, they aren’t thinking about our language at all—their headspace is all about their program. It’s usually only when things go wrong that they notice our implementation.

When that happens, it’s up to us to give the user all the information they need to understand what went wrong and guide them gently back to where they are trying to go. Doing that well means thinking about error handling all through the implementation of our interpreter, starting now.



当我们开始运行解释器时候，另外一个需要考虑的基础功能是错误处理。在教科书中，通常不太涉及这个部分，因为错误处理更像是一个实际问题，而不是一个计算机科学的正式问题。但是，如果我们要真正使用一门语言，如果优雅的处理报错，是一个非常重要的部分。

我们语言实现的用于错误处理的工具，构成了用户界面的很大一部分，当用户代码正常运行时候，他们不会关心我们的语言，用户的顶部空间，将展示他们程序运行信息。

当发生错误时候，我们应该提供给用户足够的信息，用于定位哪里出现了问题，并引导用户回到报错的位置。做好这一点意味着从现在开始，在解释器的整个实现过程中，我们都需要考虑报错处理。

> Having said all that, for this interpreter, what we’ll build is pretty bare bones. I’d love to talk about interactive debuggers, static analyzers, and other fun stuff, but there’s only so much ink in the pen.
> 
> 尽管如此，对于这个解释器来说，我们将构建的错误处理，是非常简单的。我很想谈谈交互式调试器、静态分析器和其他有意思的东西，但是笔里面现在只有这些墨水。


```java

// lox/Lox.java, add after run()
static void error(int line, String message) {
	report(line, "", message);
}

private static void report(int line, String where, String message) {
	System.err.println("[line " + line + "] Error" + where + ": " + message);
	hadError = true;
}
	
```

This error() function and its report() helper tells the user some syntax error occurred on a given line. That is really the bare minimum to be able to claim you even have error reporting. Imagine if you accidentally left a dangling comma in some function call and the interpreter printed out:

```

Error: Unexpected "," somewhere in your code. Good luck finding it!

```

That’s not very helpful. We need to at least point them to the right line. Even better would be the beginning and end column so they know where in the line. Even better than that is to show the user the offending line, like:

```

Error: Unexpected "," in argument list.

    15 | function(first, second,);
                               ^-- Here.
							   
```

error() 函数 和 report() 函数，告诉用户某一行发生了语法错误，这实际上是报错处理的最低要求。想象一下，如果你在代码中添加了一个多余的逗号， 解释器运行结果如下:

```

Error: Unexpected "," somewhere in your code. Good luck finding it!

```

这不是非常有帮助，我们至少应该指向正确的报错位置。更好的方式，是设置报错代码的起始位置和结束位置，以便于展示报错位置，更加好的做法是展示报错的原始代码行，例如：

```

Error: Unexpected "," in argument list.

    15 | function(first, second,);
                               ^-- Here.
							   
```
  
I’d love to implement something like that in this book but the honest truth is that it’s a lot of grungy string manipulation code. Very useful for users, but not super fun to read in a book and not very technically interesting. So we’ll stick with just a line number. In your own interpreters, please do as I say and not as I do.

The primary reason we’re sticking this error reporting function in the main Lox class is because of that hadError field. It’s defined here:

```java

// lox/Lox.java, in class Lox

public class Lox {
  static boolean hadError = false;
  ...
}
```

We’ll use this to ensure we don’t try to execute code that has a known error. Also, it lets us exit with a non-zero exit code like a good command line citizen should.

```java

// lox/Lox.java, in runFile()

private static void runFile(String path) throws IOException {
	byte[] bytes = Files.readAllBytes(Paths.get(path));
	run(new String(bytes, Charset.defaultCharset()));

	// Indicate an error in the exit code.
	if (hadError) {
		System.exit(65);
	}
}
	

```

We need to reset this flag in the interactive loop. If the user makes a mistake, it shouldn’t kill their entire session.

```java

// lox/Lox.java, in runPrompt()
    private static void runPrompt() throws IOException {
        InputStreamReader input = new InputStreamReader(System.in);
        BufferedReader reader = new BufferedReader(input);

        for (;;) {
            System.out.print("> ");
            String line = reader.readLine();
            if (line == null) {
                break;
            }
            run(line);
			hadError = false;
        }
    }



```

我很想在本书中，实现上面说的报错处理功能，老实说，这是一个非常糟糕的字符串处理代码。对于用户来说，错误处理非常重要，但是，对于我们来说，在技术上实现不是非常有趣。所以，我们将只使用一个行号，当你实现自己的解释器时候，尽量按照我说的去做，而不是按照我做的去做。

我们在Lox主类中，坚持使用report函数的原因是，我们定义了一个字段 hadError

```java

// lox/Lox.java, in class Lox

public class Lox {
  static boolean hadError = false;
  ...
}
```

通过 hadError 字段，我们将确保在出现报错时候，不再执行代码。此外，利用该字段，我们可以退出程序，就像是一个好的命令行工具那样。


```java

// lox/Lox.java, in runFile()

private static void runFile(String path) throws IOException {
	byte[] bytes = Files.readAllBytes(Paths.get(path));
	run(new String(bytes, Charset.defaultCharset()));

	// Indicate an error in the exit code.
	if (hadError) {
		System.exit(65);
	}
}
	

```

我们需要在交互式执行时候，重置 hadError信息，如果用户写错了命令，不应该把整个会话断开。

```java

// lox/Lox.java, in runPrompt()
    private static void runPrompt() throws IOException {
        InputStreamReader input = new InputStreamReader(System.in);
        BufferedReader reader = new BufferedReader(input);

        for (;;) {
            System.out.print("> ");
            String line = reader.readLine();
            if (line == null) {
                break;
            }
            run(line);
			hadError = false;
        }
    }

```

The other reason I pulled the error reporting out here instead of stuffing it into the scanner and other phases where the error might occur is to remind you that it’s good engineering practice to separate the code that generates the errors from the code that reports them.

Various phases of the front end will detect errors, but it’s not really their job to know how to present that to a user. In a full-featured language implementation, you will likely have multiple ways errors get displayed: on stderr, in an IDE’s error window, logged to a file, etc. You don’t want that code smeared all over your scanner and parser.

Ideally, we would have an actual abstraction, some kind of “ErrorReporter” interface that gets passed to the scanner and parser so that we can swap out different reporting strategies. For our simple interpreter here, I didn’t do that, but I did at least move the code for error reporting into a different class.


我把error report函数独立出来，而不是把它们写入scanner，或者其他可能发生错误的阶段，是提醒你，将产生错误的代码，和报告错误的代码分开，是一个很好的工程实践。

前端的各个阶段都会检测错误，但是如何将错误呈现给用户，实际上不是它们的工作。在一个全功能的语言实现中，我们可能会实现多种报错展示，例如：在标准错误输出中（stderr），在IDE的报错展示界面，将报错信息写入日志，我们肯定不希望在前端的每个阶段，都实现一次报错处理。

理想状态下，我们将定义一个抽象接口，ErrorReporter ，这样，在不同的编译阶段，例如：在扫描、解析阶段，我们可以使用不同的报错策略。对于我们当前实现的简单的jlox解释器，我将不打算实现，但是至少，我将把报错处理函数，移动到另外一个类中。

> I had exactly that when I first implemented jlox. I ended up tearing it out because it felt over-engineered for the minimal interpreter in this book.
> 
> 当我一开始实现jlox时候，我的确实现了，一个错误处理接口，但是，接下来，我又把这些代码删除了，因为，对于一个简洁的解释器，我们不应该过度设计错误处理。

With some rudimentary error handling in place, our application shell is ready. Once we have a Scanner class with a scanTokens() method, we can start running it. Before we get to that, let’s get more precise about what tokens are.

有了基本的报错处理，我们可以进一步开始扫描器，一旦我们定义了一个带有 scanTokens方法的Scanner 类，我们可以考虑运行了。但是，在真正运行扫描器之前，我们还是需要先了解一下 token。

## 二、 Lexemes and Tokens

词素 和 token

Here’s a line of Lox code:

```java

var language = "lox";

```

Here, var is the keyword for declaring a variable. That three-character sequence “v-a-r” means something. But if we yank three letters out of the middle of language, like “g-u-a”, those don’t mean anything on their own.

That’s what lexical analysis is about. Our job is to scan through the list of characters and group them together into the smallest sequences that still represent something. Each of these blobs of characters is called a lexeme. In that example line of code, the lexemes are:

![lexeme](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/lexemes.png?raw=true)

The lexemes are only the raw substrings of the source code. However, in the process of grouping character sequences into lexemes, we also stumble upon some other useful information. When we take the lexeme and bundle it together with that other data, the result is a token. It includes useful stuff like:

下面是一个Lox代码

```java

var language = "lox";

```



在这里，var 用于声明一个变量，是一个关键字，3个字符 var 意味着什么。但是，如果我们从 language中取出3个字符，gua，这些字母没有实际含义。

这就是词法分析的意义，我们的工作是，扫描字符列表，然后，把它们组合成最小的序列，这些序列仍然代表了某些东西，这些字符序列，称为一个个词素，例如：

![lexeme](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/lexemes.png?raw=true)

词素只是源代码的字串，然而，在将字符列表组合成词素列表的过程中，我们还发现了一些其他有用信息。当我们将词素和其他有用信息绑定在一起时候，我们将获取到一个token，有用信息如下。

### 2.1 Token type

tokne类型

Keywords are part of the shape of the language’s grammar, so the parser often has code like, “If the next token is while then do . . . ” That means the parser wants to know not just that it has a lexeme for some identifier, but that it has a reserved word, and which keyword it is.

The parser could categorize tokens from the raw lexeme by comparing the strings, but that’s slow and kind of ugly. Instead, at the point that we recognize a lexeme, we also remember which kind of lexeme it represents. We have a different type for each keyword, operator, bit of punctuation, and literal type.

```java

// lox/TokenType.java, create new file
package com.craftinginterpreters.lox;

enum TokenType {
  // Single-character tokens.
  LEFT_PAREN, RIGHT_PAREN, LEFT_BRACE, RIGHT_BRACE,
  COMMA, DOT, MINUS, PLUS, SEMICOLON, SLASH, STAR,

  // One or two character tokens.
  BANG, BANG_EQUAL,
  EQUAL, EQUAL_EQUAL,
  GREATER, GREATER_EQUAL,
  LESS, LESS_EQUAL,

  // Literals.
  IDENTIFIER, STRING, NUMBER,

  // Keywords.
  AND, CLASS, ELSE, FALSE, FUN, FOR, IF, NIL, OR,
  PRINT, RETURN, SUPER, THIS, TRUE, VAR, WHILE,

  EOF
}

```

关键字是语言语法的一部分，因此，解析器通常会有下面的逻辑：如果下一个token是 while，那么..., 这意味着，解析器不仅仅想要知道，它具有某个标识符，还想要知道它包含有一个保留字，用于关键字声明。

解析器可以通过比较字符串，对原始词素进行分类，但是这样既慢，并且非常难实现。相反，在我们识别一个词素的时候，每一个词素都有特定的类型，例如：关键字、运算符、标点符号、文字类型等等。


```java

// lox/TokenType.java, create new file
package com.craftinginterpreters.lox;

enum TokenType {
  // Single-character tokens.
  LEFT_PAREN, RIGHT_PAREN, LEFT_BRACE, RIGHT_BRACE,
  COMMA, DOT, MINUS, PLUS, SEMICOLON, SLASH, STAR,

  // One or two character tokens.
  BANG, BANG_EQUAL,
  EQUAL, EQUAL_EQUAL,
  GREATER, GREATER_EQUAL,
  LESS, LESS_EQUAL,

  // Literals.
  IDENTIFIER, STRING, NUMBER,

  // Keywords.
  AND, CLASS, ELSE, FALSE, FUN, FOR, IF, NIL, OR,
  PRINT, RETURN, SUPER, THIS, TRUE, VAR, WHILE,

  EOF
}

```

> After all, string comparison ends up looking at individual characters, and isn’t that the scanner’s job?
> 
> 毕竟，字符串比较会最终比较每一个字符，这个不是扫描器的功能吗？

### 2.2 Literal value

字符值

There are lexemes for literal values—numbers and strings and the like. Since the scanner has to walk each character in the literal to correctly identify it, it can also convert that textual representation of a value to the living runtime object that will be used by the interpreter later

词素具有实际值，可能是数字，字符串等。由于扫描器需要遍历文本中的每一个字符，因此，扫描器扫描过程中，会将具体的文本值，转化为一个个运行时对象，在接下来的解释阶段，将使用这些对象。

### 2.3 Location information

位置信息

Back when I was preaching the gospel about error handling, we saw that we need to tell users where errors occurred. Tracking that starts here. In our simple interpreter, we note only which line the token appears on, but more sophisticated implementations include the column and length too.

回到上一章，我宣传错误处理的时候，我们必须告诉用户错误发生的具体位置，而记录位置信息，是从这里开始的。在我们实现的简单的解释器中，我们只会记录行信息，但是，对于更加复杂的解释器，需要记录具体的列和长度信息。

> Some token implementations store the location as two numbers: the offset from the beginning of the source file to the beginning of the lexeme, and the length of the lexeme. The scanner needs to know these anyway, so there’s no overhead to calculate them.
> 
> 有些token 实现中使用两个数字保存位置信息：从源文件开始到词素开始的偏移量，词素的长度，扫描器需要知道这些位置信息，计算这个位置，不是过度计算。
>
> An offset can be converted to line and column positions later by looking back at the source file and counting the preceding newlines. That sounds slow, and it is. However, you need to do it only when you need to actually display a line and column to the user. Most tokens never appear in an error message. For those, the less time you spend calculating position information ahead of time, the better.
> 
> 通过偏移量信息（即源文件到词素的偏移量），我们可以通过查看源文件，换算为具体的行、列信息。 注意，换行符的存在。这个过程看起来很慢，实际的确是，但是，我们只是在需要展示实际行、列信息时候，才需要如此计算。但是，大多数 token不会出现在错误信息中，所以，对于这些token，计算它们的位置信息不会消耗很多时间。

We take all of this data and wrap it in a class.

```java

// lox/Token.java, create new file
package com.craftinginterpreters.lox;

class Token {
  final TokenType type;
  final String lexeme;
  final Object literal;
  final int line; 

  Token(TokenType type, String lexeme, Object literal, int line) {
    this.type = type;
    this.lexeme = lexeme;
    this.literal = literal;
    this.line = line;
  }

  public String toString() {
    return type + " " + lexeme + " " + literal;
  }
}

```

我们将token 打包为一个类。

```java

// lox/Token.java, create new file
package com.craftinginterpreters.lox;

class Token {
  final TokenType type;
  final String lexeme;
  final Object literal;
  final int line; 

  Token(TokenType type, String lexeme, Object literal, int line) {
    this.type = type;
    this.lexeme = lexeme;
    this.literal = literal;
    this.line = line;
  }

  public String toString() {
    return type + " " + lexeme + " " + literal;
  }
}

```

Now we have an object with enough structure to be useful for all of the later phases of the interpreter.

现在，我们拥有了一个对象，具有足够的结构，可以用于解释器后续阶段。

## 三、 Regular Languages and Expressions

Now that we know what we’re trying to produce, let’s, well, produce it. The core of the scanner is a loop. Starting at the first character of the source code, the scanner figures out what lexeme the character belongs to, and consumes it and any following characters that are part of that lexeme. When it reaches the end of that lexeme, it emits a token.

Then it loops back and does it again, starting from the very next character in the source code. It keeps doing that, eating characters and occasionally, uh, excreting tokens, until it reaches the end of the input.

现在，我们知道了想要生产什么，那就让我们去生产吧。扫描器的核心是一个循环，从源代码的第一个字符开始，扫描器识别出该字符所属的词素，然后，继续扫描，一直到词素的结束。然后，扫描器会提交一个token。

然后，扫描器再次从前一个词素结尾开始，它会一直这样，扫描字符，偶尔，到达词素结尾，提交token，一直到源代码的结尾。

> Lexical analygator.
> 
> 词法分析器

![lexical](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/lexigator.png?raw=true)

The part of the loop where we look at a handful of characters to figure out which kind of lexeme it “matches” may sound familiar. If you know regular expressions, you might consider defining a regex for each kind of lexeme and using those to match characters. For example, Lox has the same rules as C for identifiers (variable names and the like). This regex matches one:


```

[a-zA-Z_][a-zA-Z_0-9]*

```


在扫描器的循环过程中，我们通过几个字符，判断出词素的类型，这看起来很熟悉。如果你知道正则表达式，你可能会为每种类型的词素，定义一个正则表达式，并且使用它来匹配字符。例如：lox语言中，对于标识符的定义（即变量名称）具有和C语言相同的规则，可以用正则表达式表示

```

[a-zA-Z_][a-zA-Z_0-9]*

```

If you did think of regular expressions, your intuition is a deep one. The rules that determine how a particular language groups characters into lexemes are called its lexical grammar. In Lox, as in most programming languages, the rules of that grammar are simple enough for the language to be classified a regular language. That’s the same “regular” as in regular expressions.

如果你想到了正则表达式，你的直觉是正确深刻的。如何将字符序列分组为词素的规则称为词法规则。在Lox中，和大多数的语言一样，该语法规则非常简单，可以将 lox语言归类为常规语言，这与正则表达式中的正则是一个含义。

> It pains me to gloss over the theory so much, especially when it’s as interesting as I think the Chomsky hierarchy and finite-state machines are. But the honest truth is other books cover this better than I could. Compilers: Principles, Techniques, and Tools (universally known as “the dragon book”) is the canonical reference.
> 
> 我很痛苦的掩盖了这个理论，特别是当它像我认为的[乔姆斯基谱系 (Chomsky hierarchy)](https://en.wikipedia.org/wiki/Chomsky_hierarchy)和[有限状态机 (Finite-state machine)](https://en.wikipedia.org/wiki/Finite-state_machine)一样有意思。但老实说，其他书比我写得更好一些，[编译器：原理、技术和工具](https://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools)，通常被称为龙书，是编译器领域的标准。

You very precisely can recognize all of the different lexemes for Lox using regexes if you want to, and there’s a pile of interesting theory underlying why that is and what it means. Tools like Lex or Flex are designed expressly to let you do this—throw a handful of regexes at them, and they give you a complete scanner back.

Since our goal is to understand how a scanner does what it does, we won’t be delegating that task. We’re about handcrafted goods.

如果你想的话，你可以非常精确地使用正则表达式来识别Lox的所有不同词素，并且有一堆理论说明，为什么会这样，以及它的含义。像是 Lex Flex就是这样的工具，你可以向它们提供一些正则表达式，它们会生成一个完整的扫描器。

因为我们的目标是了解扫描器内部工作原理，所以我们不会使用Lex这样的工具，我们希望的是纯手工打造的工艺品。

> Lex was created by Mike Lesk and Eric Schmidt. Yes, the same Eric Schmidt who was executive chairman of Google. I’m not saying programming languages are a surefire path to wealth and fame, but we can count at least one mega billionaire among us.
> 
> lex是由  Mike Lesk 和 Eric Schmidt 开发的工具，是的，就是 google公司创始人  Eric Schmidt. 我并不是说编程语言可以让人变得富有，但是至少我们当中出现了一个亿万富翁。


## 四、The Scanner Class

扫描器类

Without further ado, let’s make ourselves a scanner.

不需要麻烦了，我们将自己实现一个扫描器。

```java

// lox/Scanner.java, create new file

package com.craftinginterpreters.lox;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static com.craftinginterpreters.lox.TokenType.*; 

class Scanner {
  private final String source;
  private final List<Token> tokens = new ArrayList<>();

  Scanner(String source) {
    this.source = source;
  }
}


```




> I know static imports are considered bad style by some, but they save me from having to sprinkle TokenType. all over the scanner and parser. Forgive me, but every character counts in a book.
> 
> 我知道静态导入，被很多人认为是一种糟糕的写法，但是，静态导入，可以让我们的引用代码，不用再添加 TokenType , 扫描器和解释器，会充满 TokenType, 请原谅我，书中的每一个章节都很重要。

We store the raw source code as a simple string, and we have a list ready to fill with tokens we’re going to generate. The aforementioned loop that does that looks like this:

我们将源代码存储为一个简单的字符串，我们有一个列表，扫描器提交的token都会写入这个列表。上面提到的扫描器循环，看起来是下面代码。

```java

// lox/Scanner.java, add after Scanner()

  List<Token> scanTokens() {
    while (!isAtEnd()) {
      // We are at the beginning of the next lexeme.
      start = current;
      scanToken();
    }

    tokens.add(new Token(EOF, "", null, line));
    return tokens;
  }


```

The scanner works its way through the source code, adding tokens until it runs out of characters. Then it appends one final “end of file” token. That isn’t strictly needed, but it makes our parser a little cleaner.

This loop depends on a couple of fields to keep track of where the scanner is in the source code.

扫描器通过扫描源代码，添加token，一直到字符序列遍历完成。然后，在最后添加一个“文件结束”的token，这个文件结束 token不是严格必须的，但是它会让我们的解释器更加简洁。

循环阶段，依据几个字段，来追踪扫描器的具体位置信息。

```java

// lox/Scanner.java, in class Scanner
  private final List<Token> tokens = new ArrayList<>();
  private int start = 0;
  private int current = 0;
  private int line = 1;

  Scanner(String source) {

```

The start and current fields are offsets that index into the string. The start field points to the first character in the lexeme being scanned, and current points at the character currently being considered. The line field tracks what source line current is on so we can produce tokens that know their location.

Then we have one little helper function that tells us if we’ve consumed all the characters.

start 和 current变量表示词素的位置信息，start字段表示词素的第一个字符的位置，current表示当前扫描到的字符位置，line字段表示当前扫描的行，根据这些字段，我们可以构造token中的位置信息。

我们还定义一个函数 isAtEnd，用于判断是否已经扫描完。

```java

// lox/Scanner.java, add after scanTokens()

  private boolean isAtEnd() {
    return current >= source.length();
  }
  
```


## 五、Recognizing Lexemes

识别词素

In each turn of the loop, we scan a single token. This is the real heart of the scanner. We’ll start simple. Imagine if every lexeme were only a single character long. All you would need to do is consume the next character and pick a token type for it. Several lexemes are only a single character in Lox, so let’s start with those.

在循环过程中，我们会扫描token，这是扫描器的核心功能，我们从简单开始，设想一下，每个token都是一个简单的字符，我们需要做的是，继续判断下一个token，在Lox语言中，有几个词素是简单的一个字符，我们从这些单字符词素开始：


> Wondering why / isn’t in here? Don’t worry, we’ll get to it.
> 
> 为什么 / 符号不在上面的判断中，我们将会马上告诉你。

Again, we need a couple of helper methods.

同样的，我们还需要定义几个函数。





