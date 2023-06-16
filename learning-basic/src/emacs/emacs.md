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
 
 

