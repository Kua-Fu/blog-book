# Emacs

## 1. 多行行首添加字符

> 参考
>
> [How to add a prefix to every line?](https://emacs.stackexchange.com/questions/11/how-to-add-a-prefix-to-every-line)

使用命令 `M-x string-rectangle`, 绑定的快捷键是 `C-x r t`

流程: 

* 使用 `set-mark-command`, 绑定快捷键是 `M-SPC` 选定要修改的行

* 使用 `M-x string-rectangle` ，添加指定字符串

* `Enter` 后，所有选定行变化


## 2. diff/ediff 比较文本不同

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
 
 

