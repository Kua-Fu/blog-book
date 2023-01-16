# comment system: Giscus

## 一、References

>[Giscus](https://giscus.app/)

>[mdbook 和 giscus](https://sakaketsukihana.github.io/Notes/mdBook-and-Giscus.html)

## 一、Giscus

you can find more details about Giscus from office website

We assume that 

* you have one book named test-book

* you have one github project named test-project, and have install Giscus

## 1.1 add custom js

```shell
➜  tree test-book
test-book
├── book
├── book.toml
└── src
    ├── SUMMARY.md
    └── chapter_1.md

2 directories, 3 files

```

modify `book.toml`, add custom js info

```
[book]
authors = ["yz"]
language = "en"
multilingual = false
src = "src"
title = "test-book"

[output.html]
additional-js = ["giscus.js"]

```

add `giscus.js` file at book root dir, 

⚠️ the script info, can get from [giscus](https://giscus.app/)

```js

var giscus = function () {
    const script = document.createElement("script");
    script.type = "text/javascript";
    script.src = "https://giscus.app/client.js";
var giscus = function () {
    const script = document.createElement("script");
    script.type = "text/javascript";
    script.src = "https://giscus.app/client.js";


    script.setAttribute("data-repo", "your github project repo");
    script.setAttribute("data-repo-id", "your github project repo id");
    script.setAttribute("data-category", "Announcements");
    script.setAttribute("data-category-id", "xxx");

    script.setAttribute("data-mapping", "title");
    script.setAttribute("data-term", "0");
    script.setAttribute("data-reactions-enabled", "1");
    script.setAttribute("data-emit-metadata", "0");
    script.setAttribute("data-input-position", "top");
    script.setAttribute("data-theme", "light");
    script.setAttribute("data-lang", "zh-CN");
    script.setAttribute("data-loading", "lazy");

    script.crossOrigin = "anonymous";
    script.async = true;
    document.getElementById("giscus-container").appendChild(script);
  };

  window.addEventListener('load', giscus);

```

## 1.2 add custom index.hbs

we need modify index.html, add one more `div` named `giscus-container`

so you can add a new file named index.hbs in theme dir

```

mkdir theme && cd theme

wget https://raw.githubusercontent.com/rust-lang/mdBook/master/src/theme/index.hbs 
```

modify `index.hbs`, add `div` 

```js
<main>
    {{{ content }}}
    <div id="giscus-container"></div>
</main>
```

## 1.3 rebuild book

test-book tree

```shell

➜  test-book tree .
.
├── book
├── book.toml
├── giscus.js
├── src
│   ├── SUMMARY.md
│   └── chapter_1.md
└── theme
    └── index.hbs

3 directories, 5 files

```

```
mdbook build
```
