# tmux


## 1. tmux 进入没有名称的session

```

tmux list-sessions

# 4: 1 windows (created Sun Jul 31 12:46:12 2022) [107x24]

tmux attach-session -t 4

```

## 2. tmux 如何翻页


### 2.1 进入tmux翻屏模式

先按 ctrl ＋ｂ，松开，然后再按 [

### 2.2 实现上下翻页

进入翻屏模式后，PgUp PgDn 实现上下翻页

### 2.3 退出翻屏模式

q


