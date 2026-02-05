# meilisearch

## 一、参考

>[meilisearch](https://www.meilisearch.com/docs/learn/what_is_meilisearch/overview)

>[CentOS7 升级 Glibc 2.17 到2.28](https://roy.wang/centos7-upgrade-glibc/)

>[Install specific version of Glibc](https://iq.opengenus.org/install-specific-version-of-glibc/)

>[centos7 升级 glibc && gcc](https://garlicspace.com/2020/07/18/centos7-%E5%8D%87%E7%BA%A7-glibc-gcc/#nss_test2)

>[How to install gcc8 using devtoolset-8-gcc](https://stackoverflow.com/questions/53310625/how-to-install-gcc8-using-devtoolset-8-gcc)

## 二、安装

### 2.1 环境 

腾讯云服务器 

```shell

uname -a
Linux VM-12-11-centos 3.10.0-1160.45.1.el7.x86_64 #1 SMP Wed Oct 13 17:20:51 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux

```

### 2.2 下载

[releases](https://github.com/meilisearch/meilisearch/releases/tag/v1.2.0)

### 2.3 安装glibc-2.37

(1) 安装gcc-8.2.0所依赖的环境

```shell

yum install bison -y
yum -y install wget bzip2 gcc gcc-c++ glibc-headers

```

(2) 升级GNU Make 到4.2.1

```shell

wget http://ftp.gnu.org/gnu/make/make-4.2.1.tar.gz
tar -zxvf make-4.2.1.tar.gz
cd make-4.2.1
mkdir build
cd build
../configure --prefix=/usr/local/make && make && make install
export PATH=/usr/local/make/bin:$PATH
ln -s /usr/local/make/bin/make /usr/local/make/bin/gmake

```

```shell

make --version
GNU Make 4.2.1
Built for x86_64-redhat-linux-gnu
Copyright (C) 1988-2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

```

(3) 升级GCC

```shell

yum install centos-release-scl
yum install devtoolset-8
echo "source /opt/rh/devtoolset-8/enable" >> /etc/profile
source /etc/profile

```


```shell

gcc --version
gcc (GCC) 8.3.1 20190311 (Red Hat 8.3.1-3)
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

(4) 安装 Glibc

```shell

wget https://ftp.gnu.org/gnu/glibc/glibc-2.37.tar.gz
tar -xzvf glibc-2.37.tar.gz
cd glibc-2.37
mkdir build

../configure --disable-sanity-checks
make -j4
make install
```

(5) 查询支持的 Glibc

```
strings /lib64/libc.so.6 | grep GLIBC

```

## 2.4 运行 meilisearch

```shell

./meilisearch-linux-amd64 --master-key="test_key"


888b     d888          d8b 888 d8b                                            888
8888b   d8888          Y8P 888 Y8P                                            888
88888b.d88888              888                                                888
888Y88888P888  .d88b.  888 888 888 .d8888b   .d88b.   8888b.  888d888 .d8888b 88888b.
888 Y888P 888 d8P  Y8b 888 888 888 88K      d8P  Y8b     "88b 888P"  d88P"    888 "88b
888  Y8P  888 88888888 888 888 888 "Y8888b. 88888888 .d888888 888    888      888  888
888   "   888 Y8b.     888 888 888      X88 Y8b.     888  888 888    Y88b.    888  888
888       888  "Y8888  888 888 888  88888P'  "Y8888  "Y888888 888     "Y8888P 888  888

Config file path:	"none"
Database path:		"./data.ms"
Server listening on:	"http://localhost:7700"
Environment:		"development"
Commit SHA:		"unknown"
Commit date:		"unknown"
Package version:	"1.2.0"

Thank you for using Meilisearch!

```
