# Emacs

## 1. 查看文档

在 Emacs 中，可以使用 C-h b 快捷键来查看当前缓冲区可用的所有快捷键。这个命令会列出当前缓冲区中定义的所有快捷键和它们的描述。

另外，你也可以使用 C-h k 快捷键来查看任何一个快捷键的功能。只需按下 C-h k，然后输入要查看的快捷键即可。Emacs 会显示该快捷键的功能和用法。

如果想要全面了解 Emacs 的使用方法，推荐阅读 Emacs 的官方文档，可以通过 C-h r 快捷键打开 Emacs 自带的 Info 文档阅读器，并查看 Emacs 的完整使用手册。

## 2. 多行行首添加字符

> 参考
>
> [How to add a prefix to every line?](https://emacs.stackexchange.com/questions/11/how-to-add-a-prefix-to-every-line)

使用命令 `M-x string-rectangle`, 绑定的快捷键是 `C-x r t`

流程: 

* 使用 `set-mark-command`, 绑定快捷键是 `M-SPC` 选定要修改的行

* 使用 `M-x string-rectangle` ，添加指定字符串

* `Enter` 后，所有选定行变化


## 3. 如何使用emacs，在多行代码的每一行后面，都添加一个空行

在 Emacs 中 \n 表示一个字符，而不是一个换行符。要在 Emacs 中表示换行符，需要使用 C-q C-j。因此，如果您想在每一行后面添加一个换行符，请按照以下步骤进行操作：

* 打开要编辑的文件。

	移动光标到要添加空行的第一行。

	按下 C-SPC (Ctrl + Space) 开始标记。

	移动光标到要添加空行的最后一行。

	按下 M-x。

* 输入 replace-regexp 并按下 Enter 键。

   输入 C-q C-j 并按下 Enter 键。

   输入 C-q C-j C-q C-j 并按下 Enter 键。

* 在确认提示中按下 y 键。

  这将在每一行后面添加一个换行符。
  
⚠️ 输入 C-q C-j 是指按下 Ctrl-q Ctrl-j



## 4. diff/ediff 比较文本不同

> 参考
> 
> [Comparing Files](https://www.gnu.org/software/emacs/manual/html_node/emacs/Comparing-Files.html)

 ⚠️ ediff 不能一次性展示所有不同处的高亮，
 
 可以使用session中的
 
|命令|说明|
|---|---|
|p |(表示上一个差异) |
| n|(表示下一个差异） |
| j|(前面可以添加数字，例如：3j 表示跳转到第三个差异之处)|
| q| 退出session|
 
 

## 5. emacs 如何查看当前打开的窗口，使用了哪些package

在 Emacs 中，您可以使用以下命令来查看当前打开的窗口以及使用了哪些包（packages）：

打开 Emacs，并进入命令行模式（“M-x”）。

输入“describe-mode”并按下回车键，这将打开一个新的窗口，其中包含当前正在使用的 major mode 和 minor modes 的详细信息。

在这个窗口中，您可以看到当前打开的所有窗口以及它们所使用的 mode。此外，您还可以看到当前 mode 所使用的所有 minor modes 和加载的所有包。

要查看特定窗口中使用的包，请先将光标移动到该窗口中，然后使用上述步骤查看它所使用的 mode 和加载的包。

请注意，如果您使用了包管理器（如 ELPA 或 MELPA），则可以使用该管理器来列出和管理已安装的包，并查看当前正在使用的包。例如，使用“M-x package-list-packages”命令可以打开包管理器并列出所有可用的包，并且使用“M-x package-menu-quick-describe”命令可以查看特定包的描述和详细信息。


## 6. lsp 模式，如何重新加载？

在 Emacs 中，使用 LSP (Language Server Protocol) 模式的时候，如果需要重启 LSP Server，你可以按照以下步骤操作：

1. 打开你正在使用 LSP 的文件

2. 执行 `M-x lsp-restart-workspace` 命令。其中，`M-x` 代表调用 Emacs 的命令，`lsp-restart-workspace` 是重启 LSP 的命令。一般情况下，M 键（或者 Esc 键）和 x 键同时按下即可调用 `M-x`。

这样就可以重启 LSP Server。如果你希望关闭 LSP Server，并且不再为当前的工作区启动，你可以使用 `M-x lsp-disconnect` 命令。

