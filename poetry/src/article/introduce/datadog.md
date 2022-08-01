# datadog基本使用


## 一、参考

> [datadog agent](https://docs.datadoghq.com/agent/)

## 二、安装

### 2.1 安装agent

```
➜  ~ cat /etc/system-release
CentOS Linux release 7.6.1810 (Core)
➜  ~
➜  ~
➜  ~ DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=xxx DD_SITE="datadoghq.com" bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"

```

