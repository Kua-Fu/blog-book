# Introduction

![header](https://github.com/Kua-Fu/blog-book-images/blob/main/crafting-interpreters/header.png?raw=true)


## 1. jlox 

implement lox by Java

### 1.1 run interpreter

```
// 1. download source code
git clone https://github.com/Kua-Fu/jlox.git

// 2. build source code
gradle build

// 3. run 
java -cp app/build/classes/java/main com.craftinginterpreters.lox.App 

```

### 1.2 run tool

```
// 1. download source code
git clone https://github.com/Kua-Fu/jlox.git

// 2. build source code
gradle build

// 3. run
java -cp app/build/classes/java/main com.craftinginterpreters.tool.GenerateAst someDir

```

## 2. glox 

implement lox by Go 

### 2.1 run interpreter

```
// 1. download source code

git@github.com:Kua-Fu/glox.git

// 2. run parse
go run cmd/parser/main.go 
```


