# remote files


## 一、参考文档

>[emacs tramp](https://www.gnu.org/software/tramp/)

>[emacs wiki: tramp mode](https://www.emacswiki.org/emacs/TrampMode)

## 二、MacOS 编辑远程文件

### 2.1 ssh 模式

```

C-x C-f /ssh:user@host#port:/tmp/

```

但是会报错，connection failed

### 2.2 sshx 模式



```

C-x C-f /sshx:user@host#port:/tmp/

```

可以正常连接
