# 彻底删除企业版360

## 一、参考

> [mac完全卸载360](https://blog.csdn.net/qq_32371827/article/details/129877093)

## 二、退出正在运行的程序

```

sudo ps aux | grep 360EPP | awk '{print $2}' | xargs  kill -9


```


## 三、删除程序目录


```
sudo rm -rf /Applications/360EPP.app

sudo rm -rf /Library/Application\ Support/Qihoo

```


