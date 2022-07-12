# 检查文档是否存在

如果只想检查一个文档是否存在--根本不想关心内容—​那么用 HEAD 方法来代替 GET 方法。 HEAD 请求没有返回体，只返回一个 HTTP 请求报头：

```

curl -i -XHEAD http://localhost:9200/website/blog/123

```

如果文档存在， Elasticsearch 将返回一个 200 ok 的状态码：

```

HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0

```

若文档不存在， Elasticsearch 将返回一个 404 Not Found 的状态码：

```

curl -i -XHEAD http://localhost:9200/website/blog/124

HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0

```

当然，一个文档仅仅是在检查的时候不存在，并不意味着一毫秒之后它也不存在：也许同时正好另一个进程就创建了该文档。


