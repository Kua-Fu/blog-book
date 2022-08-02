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
