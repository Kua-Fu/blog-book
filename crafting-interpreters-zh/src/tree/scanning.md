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

```java

// lox/Scanner.java, add after scanTokens()

  private void scanToken() {
    char c = advance();
    switch (c) {
      case '(': addToken(LEFT_PAREN); break;
      case ')': addToken(RIGHT_PAREN); break;
      case '{': addToken(LEFT_BRACE); break;
      case '}': addToken(RIGHT_BRACE); break;
      case ',': addToken(COMMA); break;
      case '.': addToken(DOT); break;
      case '-': addToken(MINUS); break;
      case '+': addToken(PLUS); break;
      case ';': addToken(SEMICOLON); break;
      case '*': addToken(STAR); break; 
    }
  }
  
```



> Wondering why / isn’t in here? Don’t worry, we’ll get to it.
> 
> 为什么 / 符号不在上面的判断中，我们将会马上告诉你。

Again, we need a couple of helper methods.

The advance() method consumes the next character in the source file and returns it. Where advance() is for input, addToken() is for output. It grabs the text of the current lexeme and creates a new token for it. We’ll use the other overload to handle tokens with literal values soon.

同样的，我们还需要定义几个方法。

advance() 方法获取源文件中的下一个字符，并且返回。 advance() 方法主要用于输入，addToken() 方法用于输出，它会获取当前词素文本，并且创建新的token，下面将很快应用另外一个 addToken() 重载方法


```java

// lox/Scanner.java, add after isAtEnd()

  private char advance() {
    return source.charAt(current++);
  }

  private void addToken(TokenType type) {
    addToken(type, null);
  }

  private void addToken(TokenType type, Object literal) {
    String text = source.substring(start, current);
    tokens.add(new Token(type, text, literal, line));
  }
  
```

### 5.1 Lexical errors

词素错误

Before we get too far in, let’s take a moment to think about errors at the lexical level.  What happens if a user throws a source file containing some characters Lox doesn’t use, like @#^, at our interpreter? Right now, those characters get silently discarded. They aren’t used by the Lox language, but that doesn’t mean the interpreter can pretend they aren’t there. Instead, we report an error.

在我们深入探讨之前，需要花一点时间考虑一下词汇层面的错误。如果客户源文件中包含了，Lox语言中未定义的字符，例如 @#^，我们的jlox解释器将会如何处理？现在，我们的处理方式是，悄悄丢弃这些字符。但是，Lox语言中不使用它们，并不表示它们不存在，我们将报告一个错误。

```java

// lox/Scanner.java, in scanToken()
            case '*':
                addToken(STAR);
                break;
            default:
                Lox.error(line, "Unexpected character.");
                break;
        }
		
```

Note that the erroneous character is still consumed by the earlier call to advance(). That’s important so that we don’t get stuck in an infinite loop.

Note also that we keep scanning. There may be other errors later in the program.  It gives our users a better experience if we detect as many of those as possible in one go. Otherwise, they see one tiny error and fix it, only to have the next error appear, and so on. Syntax error Whac-A-Mole is no fun.

(Don’t worry. Since hadError gets set, we’ll never try to execute any of the code, even though we keep going and scan the rest of it.)

需要⚠️，advance()方法，仍然会消费错误的字符，这一点非常重要，否则，我们将陷入到无限循环中。

还需要注意，我们一直在扫描源文件，扫描过程中，可能会有其他报错。如果我们一次性检测到尽可能多的报错，会给使用者更好的体验。否则，使用者看到一个小错误，并且修复，然后，又会出现下一个小错误，一直这样进行着。

别担心，即使遇到报错，我们继续扫描源文件，也不会真的执行程序，因为 hadError 的存在。

> The code reports each invalid character separately, so this shotguns the user with a blast of errors if they accidentally paste a big blob of weird text. Coalescing a run of invalid characters into a single error would give a nicer user experience.
>
> 代码会分别报告每一个非法字符，因此，如果客户意外粘贴了一大块非法的文本，使用者将会收到非常多的报错，将一系列的错误合并为一个错误，将提供更好的用户体验。

### 5.2 Operators

操作符

We have single-character lexemes working, but that doesn’t cover all of Lox’s operators. What about !? It’s a single character, right? Sometimes, yes, but if the very next character is an equals sign, then we should instead create a != lexeme. Note that the ! and = are not two independent operators. You can’t write ! = in Lox and have it behave like an inequality operator. That’s why we need to scan != as a single lexeme. Likewise, <, >, and = can all be followed by = to create the other equality and comparison operators.



我们现在已经考虑到单字符的操作符，但是，这并没有包含lox中的所有操作符。例如：！字符，是一个单字符操作符吗？但是，有时候，紧随着 ！后面的是一个 = ， 此时，我们应该创建一个 != 类型的token，而不是简单的 ！token. ⚠️，! 和 = 不是两个独立的操作符，在lox中，我们不会写 ! = 这种运算符， 一般写法为 != 。这样，我们需要把 !=当作一个单独的词素，相似的，>= <= 也应该视为单独的词素

For all of these, we need to look at the second character.

所以对于这些字符，我们需要继续扫描第二个字符

```java

// lox/Scanner.java, in scanToken()

      case '*': addToken(STAR); break; 
      case '!':
        addToken(match('=') ? BANG_EQUAL : BANG);
        break;
      case '=':
        addToken(match('=') ? EQUAL_EQUAL : EQUAL);
        break;
      case '<':
        addToken(match('=') ? LESS_EQUAL : LESS);
        break;
      case '>':
        addToken(match('=') ? GREATER_EQUAL : GREATER);
        break;

      default:
	  
```

Those cases use this new method:

上面场景，需要添加新的方法。

```java

// lox/Scanner.java, add after scanToken()

  private boolean match(char expected) {
    if (isAtEnd()) return false;
    if (source.charAt(current) != expected) return false;

    current++;
    return true;
  }
```

It’s like a conditional advance(). We only consume the current character if it’s what we’re looking for.

Using match(), we recognize these lexemes in two stages. When we reach, for example, !, we jump to its switch case. That means we know the lexeme starts with !. Then we look at the next character to determine if we’re on a != or merely a !.

match() 方法好像是一个有条件的 advance(), 只有当现在的字符是我们期望的，才会消费。

使用match()，我们可以分为两个阶段识别这些词素，当扫描到 ！时候，我们进入判断逻辑，判断下一个字符是不是 =，如果是=，则我们的词素是 !=

## 六、Longer Lexemes

长词素

We’re still missing one operator: / for division. That character needs a little special handling because comments begin with a slash too.

我们仍然没有考虑到 / 运算符，一般用于表示除法，但是，该字符需要一些特殊处理，因为lox语言中，注释也是 / 字符开始的。

```java

// lox/Scanner.java, in scanToken()

        break;
      case '/':
        if (match('/')) {
          // A comment goes until the end of the line.
          while (peek() != '\n' && !isAtEnd()) advance();
        } else {
          addToken(SLASH);
        }
        break;

      default:
	  
```

This is similar to the other two-character operators, except that when we find a second /, we don’t end the token yet. Instead, we keep consuming characters until we reach the end of the line.

This is our general strategy for handling longer lexemes. After we detect the beginning of one, we shunt over to some lexeme-specific code that keeps eating characters until it sees the end.

这类似于其他的两个字符组成的词素，只有当我们找到第二个 /， 才能确定是注释。当我们发现下一个字符依然是 /，我们将不断消耗字符，一直到行尾。

这是我们处理较长词素的一般策略。在我们检测到一个特殊字符后，我们转向一些特定于该词素的代码，而这些代码一直会消费字符，一直到结束。

We’ve got another helper:

这个逻辑，可以变为新函数 peek()

```java

// lox/Scanner.java, add after match()

  private char peek() {
    if (isAtEnd()) return '\0';
    return source.charAt(current);
  }


```

It’s sort of like advance(), but doesn’t consume the character. This is called lookahead. Since it only looks at the current unconsumed character, we have one character of lookahead. The smaller this number is, generally, the faster the scanner runs. The rules of the lexical grammar dictate how much lookahead we need. Fortunately, most languages in wide use peek only one or two characters ahead.

peek() 方法和advance() 方法非常相似，但是peek()方法不会消费字符，我们称为前瞻。由于，它只会查看当前没有使用的字符，因此我们称之为前瞻。通常，这个peek() 调用次数越少，我们的扫描器会越快，词法规则，定义了我们可能需要peek() 的次数。幸运的是，大多数语言的只会有1个或者2个字符需要peek()

> Technically, match() is doing lookahead too. advance() and peek() are the fundamental operators and match() combines them.
> 
> 从技术角度，match() 方法也是一种前瞻。advance() 和 peek() 方法是基础运算单元，match() 方法，可以由它们组合。

Comments are lexemes, but they aren’t meaningful, and the parser doesn’t want to deal with them. So when we reach the end of the comment, we don’t call addToken(). When we loop back around to start the next lexeme, start gets reset and the comment’s lexeme disappears in a puff of smoke.

While we’re at it, now’s a good time to skip over those other meaningless characters: newlines and whitespace.

注释也是词素，但是它们没有实际意义，解析器也不会处理它们。因此，当我们到达行尾时候，我们不会添加新的token，当我们到达写一个词素时，将更新start，即，我们不会考虑注释部分词素。

同样的，也需要跳过其他没有实际意义的字符，例如：换行符 \n 和 其他空白符

```java

// lox/Scanner.java, in scanToken()

        break;

      case ' ':
      case '\r':
      case '\t':
        // Ignore whitespace.
        break;

      case '\n':
        line++;
        break;

      default:
        Lox.error(line, "Unexpected character.");

```

When encountering whitespace, we simply go back to the beginning of the scan loop. That starts a new lexeme after the whitespace character. For newlines, we do the same thing, but we also increment the line counter. (This is why we used peek() to find the newline ending a comment instead of match(). We want that newline to get us here so we can update line.)


当遇到空白字符，我们将回到扫描循环开始。这将在空白字符后，开始扫描下一个词素。对于换行符，我们同样这样，但是还需要增加行计数器。这就是，我们使用peek() 来查找注释行，行尾的换行符，而不是使用match() 方法，我们希望换行符，走到这个case分支，这样我们就可以增加行计数器，所以，我们可以更新行。

Our scanner is getting smarter. It can handle fairly free-form code like:

我们的扫描器越来越智能了，它可以处理这种类型的代码

```java

// this is a comment
(( )){} // grouping stuff
!*+-/=<> <= == // operators

```

### 6.1 String literals

字符串常量


Now that we’re comfortable with longer lexemes, we’re ready to tackle literals. We’ll do strings first, since they always begin with a specific character, ".

既然现在，我们已经习惯了较长的词素，我们可以继续处理文字了。我们将首先处理，字符串，它们总是以 " 字符开始。

```java

// lox/Scanner.java, in scanToken()
        break;

      case '"': string(); break;

      default:
```

```java

// lox/Scanner.java, add after scanToken()

  private void string() {
    while (peek() != '"' && !isAtEnd()) {
      if (peek() == '\n') line++;
      advance();
    }

    if (isAtEnd()) {
      Lox.error(line, "Unterminated string.");
      return;
    }

    // The closing ".
    advance();

    // Trim the surrounding quotes.
    String value = source.substring(start + 1, current - 1);
    addToken(STRING, value);
  }

```

Like with comments, we consume characters until we hit the " that ends the string. We also gracefully handle running out of input before the string is closed and report an error for that.

For no particular reason, Lox supports multi-line strings. There are pros and cons to that, but prohibiting them was a little more complex than allowing them, so I left them in. That does mean we also need to update line when we hit a newline inside a string.

Finally, the last interesting bit is that when we create the token, we also produce the actual string value that will be used later by the interpreter. Here, that conversion only requires a substring() to strip off the surrounding quotes. If Lox supported escape sequences like \n, we’d unescape those here.

和注释一样，我们不断消费字符，直到遇到结尾的 " 字符，如果源文件已经消费完了，但是仍然没有发现字符串结尾 ", 我们将会报告一个错误。

没有什么特殊原因，lox语言支持多行字符串。这样做，各有利弊，但是禁止使用多行字符串，比支持使用更加复杂，所以，我将运行lox使用多行字符串。这意味着，如果字符串中遇到换行符，我们需要更新行计数器。

最后，还有一个有意思的地方，当我们创建token时候，还会包含字符串的实际值，稍后，解释器将会使用字符串的实际值。在这里，还需要一个子字符串函数，去除字符串的 " 字符。如果lox支持转义，例如：\n, 我们在这里将取消转义。

## 6.2 Number literals

数值

All numbers in Lox are floating point at runtime, but both integer and decimal literals are supported. A number literal is a series of digits optionally followed by a . and one or more trailing digits.

lox运行时候，所有的数字都是浮点数，但是，我们支持使用者使用整数 和十进制的数字。数值是一个数字序列，其中可能是整数，也可能是浮点数。

```

1234
12.34

```

We don’t allow a leading or trailing decimal point, so these are both invalid:

但是我们不允许，小数点之前或者之后，没有实际数字

```

.1234
1234.

```

> Since we look only for a digit to start a number, that means -123 is not a number literal. Instead, -123, is an expression that applies - to the number literal 123. In practice, the result is the same, though it has one interesting edge case if we were to add method calls on numbers. Consider:
> 
> ```print -123.abs();```
> 
> This prints -123 because negation has lower precedence than method calls. We could fix that by making - part of the number literal. But then consider:
> 
> 
> ```
>  var n = 123;
>  print -n.abs();
> 
> ```
> 
> 
> This still produces -123, so now the language seems inconsistent. No matter what you do, some case ends up weird.
> 
> 因为我们将数字当作，数值词素的一个开始，这意味着 -123 不是一个数字。对应的，-123 是一个表达式。实际上，表达式的计算结果 和 -123 相同，有一个有趣的边缘场景，如果对于数字进行方法调用，例如:
> 
> ```print -123.abs();```
> 
> 这将打印出 -123, 因为负数优先级 低于 方法调用，即先执行 abs() 方法，然后才会执行负数运算。我们可以将负数符号，添加到数值中，来解决这个问题。但是，还需要考虑下面场景:
> 
> ```
> var n = 123;
> print -n.abs();
> ```
> 
> 上面的代码，将仍然打印出 -123，所以现在，lox语言出现了不一致，不管我们怎样修改，都会非常奇怪。

We could easily support the former, but I left it out to keep things simple. The latter gets weird if we ever want to allow methods on numbers like 123.sqrt()

To recognize the beginning of a number lexeme, we look for any digit. It’s kind of tedious to add cases for every decimal digit, so we’ll stuff it in the default case instead.

对于 `.1234`这种形式的数值，我们可以很容易兼容它，但是，为了简单，lox将不支持这种写法。但是，如果我们允许 `1234.` 这种写法，`1234.sqrt()` 将会变得很令人疑惑。

为了识别数字词素的开头，我们将先发现一个数字，为每一个十进制数字添加大小写非常乏味，所以，我们将它们填充为默认的大小写。

```java

// lox/Scanner.java, in scanToken(), replace 1 line
      default:
        if (isDigit(c)) {
          number();
        } else {
          Lox.error(line, "Unexpected character.");
        }
        break;
```

This relies on this little utility:

这依赖于新的方法 isDigit()

```java

// lox/Scanner.java, add after peek()

  private boolean isDigit(char c) {
    return c >= '0' && c <= '9';
  } 

```


> The Java standard library provides Character.isDigit(), which seems like a good fit. Alas, that method allows things like Devanagari digits, full-width numbers, and other funny stuff we don’t want.
> 
> java 标准库提供了 Character.isDigit() 函数，用于判断数字，看起俩很符合我们的要求。但是，这个函数可以兼容梵文数字，全宽数字，这样子虽然非常有意思，但不是我们想要的数字。

Once we know we are in a number, we branch to a separate method to consume the rest of the literal, like we do with strings.

一旦我们遇到了数字，我们将不断消费下面的字符，这时候，我们将进入新的独立方法中，就像是我们处理字符串一样。

```java

// lox/Scanner.java, add after scanToken()
  private void number() {
    while (isDigit(peek())) advance();

    // Look for a fractional part.
    if (peek() == '.' && isDigit(peekNext())) {
      // Consume the "."
      advance();

      while (isDigit(peek())) advance();
    }

    addToken(NUMBER,
        Double.parseDouble(source.substring(start, current)));
  }
  
```

We consume as many digits as we find for the integer part of the literal. Then we look for a fractional part, which is a decimal point (.) followed by at least one digit. If we do have a fractional part, again, we consume as many digits as we can find.

Looking past the decimal point requires a second character of lookahead since we don’t want to consume the . until we’re sure there is a digit after it. So we add:


我们将尽可能多的获取数值的整数部分，接下来，我们将寻找小数点部分，它由一个 . 字符开始，后面至少加上一个数字。如果我们有一个分数部分，我们同样将消费尽可能多的数字。

查看小数部分，需要我们前瞻后面的第二个字符，因为，如果小数点后面没有紧随着至少一个数字，我们将不会认为是一个小数部分。

```java

// lox/Scanner.java, add after peek()

  private char peekNext() {
    if (current + 1 >= source.length()) return '\0';
    return source.charAt(current + 1);
  } 
```

> I could have made peek() take a parameter for the number of characters ahead to look instead of defining two functions, but that would allow arbitrarily far lookahead. Providing these two functions makes it clearer to a reader of the code that our scanner looks ahead at most two characters.
> 
> 我本来可以定义一个 peek() 方法，接受一个参数，表示要前瞻的字符数，而不是定义两个函数，但是，这样，我们将允许任意长度的前瞻。提供了peek() 和 peeknext() 两个方法，可以更加明确，我们的扫描器，将最多允许2个字符的前瞻。

Finally, we convert the lexeme to its numeric value. Our interpreter uses Java’s Double type to represent numbers, so we produce a value of that type. We’re using Java’s own parsing method to convert the lexeme to a real Java double. We could implement that ourselves, but, honestly, unless you’re trying to cram for an upcoming programming interview, it’s not worth your time.

The remaining literals are Booleans and nil, but we handle those as keywords, which gets us to . . . 

最后，我们将词素转变为它的数值。我们解释器，使用java double类型表示数字，所以，扫描器这里，我们也将生成一个double数字。我们使用java 的parse方法，将数值变为双精度数字。其实，我们也可以自己实现双精度数字的转换，但是，除非你在准备编程面试，没有必要浪费时间。

剩下来，是布尔类型和 nil，但是我们将这些当作关键字处理，下面我们将探讨这些。

## 七、Reserved Words and Identifiers

保留字和标识符

Our scanner is almost done. The only remaining pieces of the lexical grammar to implement are identifiers and their close cousins, the reserved words. You might think we could match keywords like or in the same way we handle multiple-character operators like <=.

我们的扫描仪几乎快完成了。词法中还没有实现的是，标识符和保留字。你可能认为，我们应该像 <= 一样，一个个字符去匹配关键字，例如：我们要匹配 or ，代码如下

```java
case 'o':
  if (match('r')) {
    addToken(OR);
  }
  break;
```


Consider what would happen if a user named a variable orchid.  The scanner would see the first two letters, or, and immediately emit an or keyword token. This gets us to an important principle called maximal munch. When two lexical grammar rules can both match a chunk of code that the scanner is looking at, whichever one matches the most characters wins.

That rule states that if we can match orchid as an identifier and or as a keyword, then the former wins. This is also why we tacitly assumed, previously, that <= should be scanned as a single <= token and not < followed by =.

考虑一下，当我们命名一个变量 orchid ，如果我们扫描到该变量，将会发生什么？扫描器将会根据前面的两个字符 or, 立即提交一个token， 下面我们将介绍扫描器的一个重要的原则，[最大匹配原则](https://en.wikipedia.org/wiki/Maximal_munch), 当扫描器当前扫描的词素，同时满足两个词法规则，匹配最长字符的词法规则被命中。

例如：orchid，能匹配两个词法规则，可以被当作一个标识符orchid，或被当作一个关键词 or, 根据最大匹配原则，我们选择 orchild 为一个标识符。同样的，这也是，为什么我们把 <= 当作一个完整的词素，而不是把 < 当作一个词素。

> Consider this nasty bit of C code:
> 
> ```---a;```
>
> Is it valid? That depends on how the scanner splits the lexemes. What if the scanner sees it like this:
>
> ```- --a```
>
>Then it could be parsed. But that would require the scanner to know about the grammatical structure of the surrounding code, which entangles things more than we want. Instead, the maximal munch rule says that it is always scanned like:
>
> ```-- -a```
>
> It scans it that way even though doing so leads to a syntax error later in the parser.
>
> 考虑一下，这个讨厌的c代码
>
> ```---a;```
> 
> 这个语句是否合法，这取决于扫描器如何分割这个词素，如果扫描器把这个代码看作 
> 
> ```- --a;```
>
> 这样，是可以被解析的，但是，这需要扫描器了解周围代码的语法结构，这会让事情比我们想象的更加复杂。但是，实际上，按照最大匹配原则，它实际上应该是
> 
> ```-- -a;```
>
> 扫描器将会如上面这样，扫描代码，即使这样，会导致解析器中的语法错误。

Maximal munch means we can’t easily detect a reserved word until we’ve reached the end of what might instead be an identifier. After all, a reserved word is an identifier, it’s just one that has been claimed by the language for its own use. That’s where the term reserved word comes from.

最大匹配原则，意味着我们无法简单的判断一个词素是标识符，直到我们扫描到了可以确定是标识符结尾的位置。实际上，保留字也是一个标识符，它只是lox语言为了自己本身使用，而声明的一个标识符。这就是术语，保留字的来源。

So we begin by assuming any lexeme starting with a letter or underscore is an identifier.

所以，我们首先假设任何以字母或者下划线，开始的单词，都是一个标识符

```java

// lox/Scanner.java, in scanToken()

      default:
        if (isDigit(c)) {
          number();
        } else if (isAlpha(c)) {
          identifier();
        } else {
          Lox.error(line, "Unexpected character.");
        }

```

```java

// lox/Scanner.java, add after scanToken()
  private void identifier() {
    while (isAlphaNumeric(peek())) advance();

    addToken(IDENTIFIER);
  }

```

```java

// lox/Scanner.java, add after peekNext()

  private boolean isAlpha(char c) {
    return (c >= 'a' && c <= 'z') ||
           (c >= 'A' && c <= 'Z') ||
            c == '_';
  }

  private boolean isAlphaNumeric(char c) {
    return isAlpha(c) || isDigit(c);
  }
```

That gets identifiers working. To handle keywords, we see if the identifier’s lexeme is one of the reserved words. If so, we use a token type specific to that keyword. We define the set of reserved words in a map.

上面的代码将会识别出标识符，为了处理关键字，我们将查看标识符的词素是否是一个保留字，如果是保留字，我们将会定义为特殊类型，下面是一组预定义的保留字。


```java

// lox/Scanner.java, in class Scanner
	private static final Map<String, TokenType> keywords;


    static {
        keywords = new HashMap<>();
        keywords.put("and", AND);
        keywords.put("class", CLASS);
        keywords.put("else", ELSE);
        keywords.put("false", FALSE);
        keywords.put("for", FOR);
        keywords.put("fun", FUN);
        keywords.put("if", IF);
        keywords.put("nil", NIL);
        keywords.put("or", OR);
        keywords.put("print", PRINT);
        keywords.put("return", RETURN);
        keywords.put("super", SUPER);
        keywords.put("this", THIS);
        keywords.put("true", TRUE);
        keywords.put("var", VAR);
        keywords.put("while", WHILE);
    }
```

Then, after we scan an identifier, we check to see if it matches anything in the map.

然后，我们在确定一个标识符后，可以在判断一下是否是关键字

```java

// lox/Scanner.java, in identifier(), replace 1 line

    private void identifier() {
        while (isAlphaNumberic(peek()))
            advance();
        String text = source.substring(start, current);
        TokenType type = keywords.get(text);
        if (type == null)
            type = IDENTIFIER;
        addToken(type);
    }
  
```

If so, we use that keyword’s token type. Otherwise, it’s a regular user-defined identifier.

And with that, we now have a complete scanner for the entire Lox lexical grammar. Fire up the REPL and type in some valid and invalid code. Does it produce the tokens you expect? Try to come up with some interesting edge cases and see if it handles them as it should.


如果标识符是一个关键字，我们提交的token需要为关键字类型。

有了这些，我们现在有了一个完整的扫描器。可以扫描整个lox程序。启动扫描器，输入一些lox代码，执行看看。它是否会输出你想要的token列表，尽可能尝试一些边缘示例，看看我们的扫描器是否可以执行呢？

## 八、挑战习题

1. The lexical grammars of Python and Haskell are not regular. What does that mean, and why aren’t they?

   python 和 Haskell语言词法规则 不是规则的，这是什么意思？它们为什么不是规则的呢？
   
1. Aside from separating tokens—distinguishing print foo from printfoo—spaces aren’t used for much in most languages. However, in a couple of dark corners, a space does affect how code is parsed in CoffeeScript, Ruby, and the C preprocessor. Where and what effect does it have in each of those languages?

	除了区分 `print foo` 和 `printfoo` , 空格在大多数的语言中没有太多作用。然而，在很多偏僻的角落，空格会影响 CoffeeScript, Ruby, C预处理器，解析代码。在这些语言中，它们分别有什么影响呢？
	
1. Our scanner here, like most, discards comments and whitespace since those aren’t needed by the parser. Why might you want to write a scanner that does not discard those? What would it be useful for?

	我们的扫描器和大多数的扫描器一样，会丢弃注释和空白字符，因为，解释器不需要它。为什么要编写一个保留注释和空白字符的扫描器呢？它有什么作用呢？
	
1. Add support to Lox’s scanner for C-style /* ... */ block comments. Make sure to handle newlines in them. Consider allowing them to nest. Is adding support for nesting more work than you expected? Why?

   为lox扫描器，添加 /* ... */ 形式的注释语法，确保正确处理其中的换行符。还需要考虑嵌套，添加嵌套支持的工作量是否超过你的预期？为什么？
   
   
## 九、DESIGN NOTE: IMPLICIT SEMICOLONS

设计思路：隐式分号

Programmers today are spoiled for choice in languages and have gotten picky about syntax. They want their language to look clean and modern. One bit of syntactic lichen that almost every new language scrapes off (and some ancient ones like BASIC never had) is ; as an explicit statement terminator.

Instead, they treat a newline as a statement terminator where it makes sense to do so. The “where it makes sense” part is the challenging bit. While most statements are on their own line, sometimes you need to spread a single statement across a couple of lines. Those intermingled newlines should not be treated as terminators.


今天的程序员被语言的选择宠坏了，对语法变得非常挑剔，他们希望自己使用的语言看起来干净、现代。几乎每一种现代语言都会删除一些语法苔藓（有些古老语言，例如: BASIC, 从未删除）。例如： ; 作为显示的语句终止符

相反，它们都将换行符当作语句终止符号，在合适的时候。其中，合适的时候，是指一个语句只会在一行中。但是，还有一些语句，会分散到多行中。这些语句分散的行中，换行符不应该被当作终止符号。

Most of the obvious cases where the newline should be ignored are easy to detect, but there are a handful of nasty ones:

* A return value on the next line:

	```
	if (condition) return
	"value"
	```
	
	Is “value” the value being returned, or do we have a return statement with no value followed by an expression statement containing a string literal?
	
* A parenthesized expression on the next line:


   ```
   func
   (parenthesized)

   ```


	Is this a call to func(parenthesized), or two expression statements, one for func and one for a parenthesized expression?
	
* A - on the next line:

	```
	
	first
    -second

	```
	
	Is this first - second—an infix subtraction—or two expression statements, one for first and one to negate second?

大多数应该忽略换行符的场景，都很容易被发现，但是，也有一些令人讨厌的场景：

* 返回着在下一行

	```
	if (condition) return
	"value"
	```
	
	这是一个return 语句，返回 "value"; 还是一个return语句，没有返回值；紧跟着一个字符串表达式
	
* 下一行，是括号表达式

   ```
   func
   (parenthesized)

   ```
   
   这是一个函数调用，还是一个func 表达式，紧跟着一个括号表达式
	
* 负号出现在下一行

	```
	first
	-second
	```
	
	这是一个中缀减法表达式，还是两个表达式呢
	
	
In all of these, either treating the newline as a separator or not would both produce valid code, but possibly not the code the user wants. Across languages, there is an unsettling variety of rules used to decide which newlines are separators. Here are a couple:

* Lua completely ignores newlines, but carefully controls its grammar such that no separator between statements is needed at all in most cases. This is perfectly legit:

  `a = 1 b = 2`
  
  Lua avoids the return problem by requiring a return statement to be the very last statement in a block. If there is a value after return before the keyword end, it must be for the return. For the other two cases, they allow an explicit ; and expect users to use that. In practice, that almost never happens because there’s no point in a parenthesized or unary negation expression statement.
  
* Go handles newlines in the scanner. If a newline appears following one of a handful of token types that are known to potentially end a statement, the newline is treated like a semicolon. Otherwise it is ignored. The Go team provides a canonical code formatter, gofmt, and the ecosystem is fervent about its use, which ensures that idiomatic styled code works well with this simple rule.

* Python treats all newlines as significant unless an explicit backslash is used at the end of a line to continue it to the next line. However, newlines anywhere inside a pair of brackets ((), [], or {}) are ignored. Idiomatic style strongly prefers the latter.

  This rule works well for Python because it is a highly statement-oriented language. In particular, Python’s grammar ensures a statement never appears inside an expression. C does the same, but many other languages which have a “lambda” or function literal syntax do not.
  
  An example in JavaScript:
  
  ```javascript
  console.log(function() {
    statement();
  });
  ```
  
  Here, the console.log() expression contains a function literal which in turn contains the statement statement();.

  
  Python would need a different set of rules for implicitly joining lines if you could get back into a statement where newlines should become meaningful while still nested inside brackets.

* JavaScript’s “automatic semicolon insertion” rule is the real odd one. Where other languages assume most newlines are meaningful and only a few should be ignored in multi-line statements, JS assumes the opposite. It treats all of your newlines as meaningless whitespace unless it encounters a parse error. If it does, it goes back and tries turning the previous newline into a semicolon to get something grammatically valid.

  This design note would turn into a design diatribe if I went into complete detail about how that even works, much less all the various ways that JavaScript’s “solution” is a bad idea. It’s a mess. JavaScript is the only language I know where many style guides demand explicit semicolons after every statement even though the language theoretically lets you elide them.
  

If you’re designing a new language, you almost surely should avoid an explicit statement terminator. Programmers are creatures of fashion like other humans, and semicolons are as passé as ALL CAPS KEYWORDS. Just make sure you pick a set of rules that make sense for your language’s particular grammar and idioms. And don’t do what JavaScript did.


上面的场景中，将换行符当作分隔符，或者不把换行符当作分隔符，都可以产生有效的代码。但是，这样可能产生的不是用户想要的效果。在不同的语言中，有许多特殊规则，指定了哪些换行符是分隔符。

* Lua语言中，完全忽略了换行符，但是，需要小心的使用。在大多数场景，没有换行符的语句，可以正常运行，例如：orchid

  `a = 1 b = 2`
  
  Lua通过将代码块中的最后一条语句定义为return 语句，来避免return问题。
  
  (1) 如果关键词 return 之后，end关键词之前，存在 "value" 字符串，则Lua中表示返回 "value"
  
  ```
  return 
  "value"
  ```
  
  (2) 其他两种情况，几乎不会发生，因为括号或者一元否定表达式语句，几乎没有意义
  
* go语言，会处理扫描仪中的换行符。如果换行符，出现在可能是结束标志后，则换行符被当作隔离符。否则，换行符将被忽略。go开发团队提供了一个代码格式化的程序gofmt，go生态中非常热衷于使用它，这保证了习惯风格的代码和换行符规则可以很好的配合。

* Python语言将所有的换行符，视为有效的。除了，你在一行的末尾，使用显示的 \ ，表示代码将继续到下一行中。但是，在`() [] {} `中的换行符将被忽略。python习惯风格也是这样的

  在Python中，这些规则很好的运行，因为它是一种高度面向语句的语言。特别的，python的语法规则，保证了表达式中，不会出现语句。C语言也是这样做的，但是很多语言，包含了 `lambda`函数或者其他函数，可能会在表达式中包含有语句。
  
  例如，javascript中的代码
  
  ```javascript
  console.log(function() {
    statement();
  });
  ```

  如上，console.log() 函数中，包含了一个匿名函数，函数代码中包含一个语句 statement();
  
  
* javascript语言

如果你正在设计一种语言，你应该避免使用显示分隔符，程序员也会紧跟时尚，分号和所有的大写关键词，都是过时设计。只需要确保，你选择了一套对于新语言特定短语和习语，都有意义的规则。
  









