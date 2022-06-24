# mdbook

## 参考

>[mdBook source code](https://github.com/rust-lang/mdBook)

>[mdBook docs](https://rust-lang.github.io/mdBook/index.html)


## 一、概览

`mdBook` 是一个开源命令行工具, 由 `rust`官方维护, 可以简单快捷的创建`markdown`格式的电子书

`rust`语言的[官方教程](https://doc.rust-lang.org/book/)就是由 `mdBook`构建的

主要具有下列优点:


* `Markdown` 格式可以让书写更简洁

* 集成了简单的搜索功能

* 代码高亮

* 自定义一些主题

* `Preprocessors` 各种预处理器可以预先处理文章内容

* `Backends` 各种后端模块，可以渲染文章

* 原生支持`rust`的代码测试

## 二、基本使用


### 2.1 安装

```

	cargo install mdbook

```

### 2.2 创建一本书

```shell

	mdbook init crafting-interpreters-zh

```

```

➜  blog-book git:(dev) ✗ mdbook init crafting-interpreters-zh

Do you want a .gitignore to be created? (y/n)
y
What title would you like to give the book?
crafting-interpreters-zh
2022-06-24 12:31:30 [INFO] (mdbook::book::init): Creating a new book with stub content

All done, no errors...

```

### 2.3 本地运行


```

	mdbook serve
	
```

```

➜  blog-book git:(dev) ✗ cd crafting-interpreters-zh
➜  crafting-interpreters-zh git:(dev) ✗ mdbook serve
2022-06-24 12:35:30 [INFO] (mdbook::book): Book building has started
2022-06-24 12:35:30 [INFO] (mdbook::book): Running the html backend
2022-06-24 12:35:30 [INFO] (mdbook::cmd::serve): Serving on: http://localhost:3000
2022-06-24 12:35:30 [ERROR] (mdbook::cmd::serve): Unable to serve: panicked at 'error binding to 127.0.0.1:3000: error creating server listener: Address already in use (os error 48)', /Users/yz/.cargo/registry/src/mirrors.tuna.tsinghua.edu.cn-df7c3c540f42cdbd/warp-0.3.2/src/server.rs:213:27
➜  crafting-interpreters-zh git:(dev) ✗ mdbook serve -p 3001
2022-06-24 12:36:18 [INFO] (mdbook::book): Book building has started
2022-06-24 12:36:18 [INFO] (mdbook::book): Running the html backend
2022-06-24 12:36:18 [INFO] (mdbook::cmd::serve): Serving on: http://localhost:3001
2022-06-24 12:36:18 [INFO] (warp::server): Server::run; addr=127.0.0.1:3001
2022-06-24 12:36:18 [INFO] (warp::server): listening on http://127.0.0.1:3001
2022-06-24 12:36:18 [INFO] (mdbook::cmd::watch): Listening for changes...


```

### 2.4 生成静态文件

```

	mdbook build
	
```

执行后，项目根目录下会新增一个 `book`目录，保存`html/css/js`等静态文件

```

➜  crafting-interpreters-zh git:(dev) ✗ mdbook build
2022-06-24 13:01:19 [INFO] (mdbook::book): Book building has started
2022-06-24 13:01:19 [INFO] (mdbook::book): Running the html backend
➜  crafting-interpreters-zh git:(dev) ✗ ls
book      book.toml src
➜  crafting-interpreters-zh git:(dev) ✗ tree book
book
├── 404.html
├── FontAwesome
│   ├── css
│   │   └── font-awesome.css
│   └── fonts
│       ├── FontAwesome.ttf
│       ├── fontawesome-webfont.eot
	
```
