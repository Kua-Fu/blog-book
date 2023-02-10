# set proxy for github

## 一、References

> [macOS 给 Git(Github) 设置代理（HTTP/SSH）](https://gist.github.com/chuyik/02d0d37a49edc162546441092efae6a1)

## 二、Set proxy

At first, you should select one protocol which you use github

such like:

`http/https` when you clone by `git clone https://github.com/xxx`

`ssh` when you clone by `git clone git@github.com:xxx`

### 2.1 set http proxy

if we select http protocol

```shell

git config --global http.proxy "http://127.0.0.1:8080"
git config --global https.proxy "http://127.0.0.1:8080"

```

or if we use socks5, should set 

```shell

git config --global http.proxy "socks5://127.0.0.1:1080"
git config --global https.proxy "socks5://127.0.0.1:1080"

```

### 2.2 set ssh proxy

we should modify ssh config file (`~/.ssh/config`)

```shell

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github
    # ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=8080 # if use http
    ProxyCommand nc -v -x 127.0.0.1:1080 %h %p 
	
```

### 2.3 unset proxy

```shell

git config --global --unset http.proxy
git config --global --unset https.proxy

```

