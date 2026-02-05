# datadog

## 一、Reference

> [datadog agent](https://docs.datadoghq.com/agent/)

> [How to monitor NGINX with Datadog](https://www.datadoghq.com/blog/how-to-monitor-nginx-with-datadog/)

## 二、Install Datadog Agent

### 2.1 install agent

```
➜  ~ cat /etc/system-release
CentOS Linux release 7.6.1810 (Core)
➜  ~
➜  ~
➜  ~ DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=xxx DD_SITE="datadoghq.com" bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"

```

## 三、Set agent configuration

### 3.1 Collect logs


Custom log collection
Datadog Agent v6 can collect logs and forward them to Datadog from files, the network (TCP or UDP), journald, and Windows channels:

1. Create a new <CUSTOM_LOG_SOURCE>.d/ folder in the conf.d/ directory at the root of your Agent’s configuration directory.

1. Create a new conf.yaml file in this new folder.

1. Add a custom log collection configuration group with the parameters below.

1. Restart your Agent to take into account this new configuration.

1. Run the Agent’s status subcommand and look for <CUSTOM_LOG_SOURCE> under the Checks section.

采集文件中的日志步骤:

1. 修改 agent 日志采集模块, 默认不采集日志

   修改文件 `/etc/datadog-agent/datadog.yaml` 中 配置 `logs_enabled: true`

1. 在 `/etc/datadog-agent/conf.d/` 目录下，创建一个新的目录 `python.d`

1. 在 `python.d` 目录下创建配置文件 `conf.yaml` ，内容如下: 

   ```

   logs:
	  - type: file
		path: /var/log/yzapp1.log
		service: yzapp1
		source: python

   ```

1. 重启agent， `systemctl restart datadog-agent`

![采集文件日志](https://github.com/Kua-Fu/blog-book-images/blob/main/datadog/datadog-custom-file-log.png?raw=true)

### 3.2 collect nginx logs

1. Add config yaml file, `/etc/datadog-agent/conf.d/nginx.d` add config file `conf.yaml`

   ```
   
   logs:
   - type: file
     path: /var/log/nginx/access.log
     source: nginx
   - type: file
     path: /var/log/nginx/error.log
     source: nginx
	 
   ```
   
1. restart agent，  `systemctl restart datadog-agent`

![nginx 日志](https://github.com/Kua-Fu/blog-book-images/blob/main/datadog/datadog-nginx-log.png?raw=true)

## 四、Create Massive Testing logs

create testing script, run it, will create massive logs to file `nginx access.log` 

```golang

package main

import (
	"fmt"
	"math/rand"
	"strconv"
	"time"

	"gopkg.in/natefinch/lumberjack.v2"
)

// 220.196.160.96 - - [31/Jul/2022:13:58:47 +0800] "GET /__livereload HTTP/1.1" 404 153 "-" "Linux Gnu (cow)" "-"
// 220.196.160.45 - - [31/Jul/2022:13:58:47 +0800] "GET /searchindex.json HTTP/1.1" 200 673061 "https://www.poetries.cn/" "Linux Gnu (cow)" "-"
// 198.199.95.33 - - [31/Jul/2022:14:01:09 +0800] "GET / HTTP/1.1" 400 255 "-" "Mozilla/5.0 zgrab/0.x" "-"
// 40.77.167.63 - - [31/Jul/2022:14:03:30 +0800] "GET / HTTP/1.1" 302 145 "-" "Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)" "-"
// 38.55.194.230 - - [31/Jul/2022:14:03:48 +0800] "GET /Public/admin/webuploader/server/preview.php HTTP/1.1" 404 153 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:48.0) Gecko/20100101 Firefox/48.0" "-"
// 51.222.253.4 - - [31/Jul/2022:14:04:53 +0800] "GET / HTTP/1.1" 200 28722 "-" "Mozilla/5.0 (compatible; AhrefsBot/7.0; +http://ahrefs.com/robot/)" "-"
// 185.7.214.104 - - [31/Jul/2022:14:07:50 +0800] "POST /Autodiscover/Autodiscover.xml HTTP/1.1" 302 145 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-"
// 185.7.214.104 - - [31/Jul/2022:14:07:58 +0800] "GET /Autodiscover/Autodiscover.xml HTTP/1.1" 404 555 "http://124.222.47.111:80/Autodiscover/Autodiscover.xml" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-"
// 185.7.214.104 - - [31/Jul/2022:14:21:52 +0800] "GET /_ignition/execute-solution HTTP/1.1" 302 145 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-"

var (
	Methods = []string{
		"GET",
		"POST",
	}

	URLs = []string{
		"/",
		"/work_reading/often/often",
		"/learning/basic/basic",
	}

	StatusCodes = []int{
		200,
		201,
		203,
		301,
		400,
		500,
		404,
	}

	Serives = []string{
		"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36",
		"Mozilla/5.0 (Windows NT 10.0; WOW64; rv:48.0) Gecko/20100101 Firefox/48.0",
		"Mozilla/5.0 (compatible; AhrefsBot/7.0; +http://ahrefs.com/robot/)",
	}
)

func writeLog(l *lumberjack.Logger) {
	outerNumber := 1000
	innerNumber := 10000
	for i := 0; i < outerNumber; i++ {
		for j := 0; j < innerNumber; j++ {
			ip := strconv.Itoa(rand.Intn(256)) + "." + strconv.Itoa(rand.Intn(256)) + "." + strconv.Itoa(rand.Intn(256)) + "." + strconv.Itoa(rand.Intn(256))
			dataStr := fmt.Sprintf(
                                `%s - - [%s] "%s %s HTTP/1.1" %d %d "-" "%s" "-"`,
				ip,
				time.Now().Format("02/Jan/2006:15:04:05 -0700"),
				Methods[rand.Intn(len(Methods))],
				URLs[rand.Intn(len(URLs))]+strconv.Itoa(j)+".html",
				StatusCodes[rand.Intn(len(StatusCodes))],
				rand.Intn(10000),
				Serives[rand.Intn(len(Serives))],
			) + "\n"
			_, err2 := l.Write([]byte(dataStr))
			if err2 != nil {
				return
			}
		}
		fmt.Printf("--write %d lines, total %d lines--\n", innerNumber, (i+1)*innerNumber)
	}
}

func main() {
	l := &lumberjack.Logger{
		Filename:   "/var/log/nginx/access.log",
		MaxSize:    500, // megabytes
		MaxBackups: 3,
		MaxAge:     0,     //days
		Compress:   false, // disabled by default
	}
	for {
		writeLog(l)
		time.Sleep(10)
	}

}

```
