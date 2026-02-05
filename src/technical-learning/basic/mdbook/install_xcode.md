# After MacOS upgrade, can not use git

## 一、Refercence

> [MacBook Pro 升级到 macOS Monterey12.1 后 git 失效问题](https://juejin.cn/post/7055179978098343972)

## 二、Solving problem

### 2.1 problem description

after Upgrade MacOS, I find git not working, error such like

![git version error](https://github.com/Kua-Fu/blog-book-images/blob/main/basic/mac/git-version.png?raw=true)

### 2.2 find reason

this error report just because of terminal can not run xcode, during upgrade MacOS, program will delete xcode

so, we should install xcode

### 2.3 install xcode

[mac develop website](https://developer.apple.com/)


![download](https://github.com/Kua-Fu/blog-book-images/blob/main/basic/mac/mac-download.png?raw=true)

![download-more](https://github.com/Kua-Fu/blog-book-images/blob/main/basic/mac/mac-download-more.png?raw=true)

![download-xcode](https://github.com/Kua-Fu/blog-book-images/blob/main/basic/mac/mac-download-xcode.png?raw=true)

![install-xcode](https://github.com/Kua-Fu/blog-book-images/blob/main/basic/mac/xcode-install.png?raw=true)

![xcode-success](https://github.com/Kua-Fu/blog-book-images/blob/main/basic/mac/mac-download-success.png?raw=true)

### 2.4 git version

![git version](https://github.com/Kua-Fu/blog-book-images/blob/main/basic/mac/git-version-success.png?raw=true)
