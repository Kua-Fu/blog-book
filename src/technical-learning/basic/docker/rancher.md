# rancher


## 一、参考

> [rancher docs](https://www.rancher.cn/quick-start/)


## 二、安装 rancher


```

docker run -d --restart=unless-stopped -p 10080:80 -p 10443:443 rancher/rancher

Unable to find image 'rancher/rancher:latest' locally
latest: Pulling from rancher/rancher
Digest: sha256:5ba20e4e51a484f107f3f270fa52c5e609cad0692dd00a26169cc3541b1f3788
Status: Downloaded newer image for rancher/rancher:latest
a95e640d300d88243dc0608a883ea12add1ff31756e0898f97002438ae5572c5

```


让我们逐个解释每个参数的含义：

docker run: 使用 Docker 运行一个容器。

* -d: 在后台（守护进程）模式下运行容器。

* --restart=unless-stopped: 如果容器意外停止，自动重启容器，除非手动停止容器。

* -p 10080:80: 将主机（宿主机）的 10080 端口映射到容器的 80 端口。这允许从主机上的浏览器访问 Rancher 的 Web 界面。

* -p 10443:443: 将主机的 10443 端口映射到容器的 443 端口。这是用于 Rancher Web 界面的安全连接（HTTPS）端口。

* rancher/rancher: Docker Hub 上 Rancher 镜像的名称。这是要运行的容器的基础镜像。

通过运行以上命令，您将在 Docker 中以 Rancher 镜像为基础创建并运行一个 Rancher 容器。Rancher 是一个用于管理和编排容器化环境的工具。运行该命令后，您可以通过浏览器访问 Rancher Web 界面，并开始管理和配置容器集群。请注意，容器的启动可能需要一些时间，具体取决于您的系统性能和网络连接速度。

```
➜  rancher docker logs a95e640d300d
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes

```

报错如上

修改启动命令 

```

sudo docker run --privileged -d --restart=unless-stopped -p 10080:80 -p 10443:443 rancher/rancher:stable

```



