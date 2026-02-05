# 列出镜像

## 一、docker image ls

```

➜  ~ docker image ls
REPOSITORY                                      TAG       IMAGE ID       CREATED         SIZE
rancher/rancher                                 latest    81ee0878ffcd   7 weeks ago     1.73GB
rancher/rancher                                 stable    81ee0878ffcd   7 weeks ago     1.73GB
provectuslabs/kafka-ui                          latest    7270929fd871   7 weeks ago     290MB
ubuntu                                          18.04     97ba4bbc97fc   3 months ago    63.2MB
opensearchproject/opensearch                    latest    5b7cb929b964   3 months ago    1.19GB

```

在上面的例子中，我们可以看到 `rancher/rancher:latest` 和 `rancher/rancher:latest` 拥有相同的 ID，因为它们对应的是同一个镜像

`docker image ls` 显示的是镜像下载到本地后，展开的大小，准确说，是展开后的各层所占空间的总和，因为镜像到本地后，查看空间的时候，更关心的是本地磁盘空间占用的大小

`docker image ls` 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 `Docker` 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 `Docker` 使用 `Union FS`，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多

`docker image ls -f dangling=true` 

## 二、docker system df

通过 `docker system df` 命令来便捷的查看镜像、容器、数据卷所占用的空间。

```

➜  ~ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          12        11        6.938GB   1.259GB (18%)
Containers      22        1         930.8MB   201.6MB (21%)
Local Volumes   15        7         3.463GB   1.525GB (44%)
Build Cache     11        0         146.6MB   146.6MB

```

## 三、docker image prune

一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除

## 四、中间层镜像


默认的 `docker image ls` 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加 `-a` 参数


## 五、列出部分镜像


* 根据仓库名列出镜像 `docker image ls ubuntu`

* 指定仓库名和标签 `docker image ls ubuntu:18.04`

* 过滤器参数 `--filter`，或者简写 `-f`

	`docker image ls -f since=mongo:3.2` 在 `mongo:3.2` 之后建立的镜像
	
	`docker image ls -f before=mongo:3.2` 在 `mongo:3.2` 之前建立的镜像

	`docker image ls -f label=com.example.version=0.1` 通过 `LABEL` 来过滤

## 六、以特定格式显示


默认情况下，`docker image ls` 会输出一个完整的表格，但是我们并非所有时候都会需要这些内容

* `docker image ls -q` 产生 ID 列表

* `docker image ls --format "{{.ID}}: {{.Repository}}"` 直接列出镜像结果，并且只包含镜像ID和仓库名

* `docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"` 以表格等距显示，并且有标题行，和默认一样，不过自己定义列

