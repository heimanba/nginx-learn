## 实例(一层nginx proxy代理)
一层proxy代理
```
39.99.155.240:7001 -> 39.103.196.78:8090
```

1. 反向代理服务器(服务IP: 39.99.155.240)
```
upstream test.com {
    server 39.103.196.78:8090; # 远程的server服务器
}

server {
    listen 7100;
    server_name 39.99.155.240;
    location / {
        proxy_pass http://test.com;
    }
}
```
2. 真正的nodejs 业务server服务器(服务IP: 39.103.196.78)
```
var http = require('http');

var server = http.createServer(function (req, res) {
        res.writeHead(200,{'Content-Type':'text/html'});
    res.write(JSON.stringify(req.headers));
    res.end();
});

server.listen(8090);

console.log('Node.js web server at port 8090 is running..')
```

3. 请求数据
```
$ curl 39.99.155.240:7100
{"host":"test.com","connection":"close","user-agent":"curl/7.54.0","accept":"*/*"}
```

## 使用proxy_set_header

1. 上述`步骤2`中换成
```
server {
    listen 7100;
    server_name 39.99.155.240;
    location / {
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $http_host;
        proxy_set_header   X-NginX-Proxy    true;
        proxy_set_header   Connection "";
        proxy_http_version 1.1;
        add_header Access-Control-Allow-Origin *;

        proxy_pass http://test.com;
    }
}
```

2. 请求数据
```
$ curl 39.99.155.240:7100
# 返回数据 {"x-real-ip":"42.120.75.253","x-forwarded-for":"42.120.75.253","host":"39.99.155.240:7100","x-nginx-proxy":"true","user-agent":"curl/7.54.0","accept":"*/*"}
```
3. 查看返回头
```
$ curl -i 39.99.155.240:7100

HTTP/1.1 200 OK
Server: nginx/1.14.1
Date: Wed, 14 Jul 2021 09:23:45 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: keep-alive
Access-Control-Allow-Origin: *
```
可以看到添加了`Access-Control-Allow-Origin: *`的header头，由`add_header`指令作用

## 实例(两台nginx proxy代理)
一层proxy代理
```
39.99.155.240:7001 -> 39.103.216.26:8080 -> 39.103.196.78:8090
```

返回的数据如下
```
curl 39.99.155.240:7100
{"x-real-ip":"39.99.155.240","x-forwarded-for":"42.120.75.243, 39.99.155.240","host":"39.99.155.240:7100","x-nginx-proxy":"true","user-agent":"curl/7.54.0","accept":"*/*"}
```


## 总结

#### 在第一台nginx中
使用proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

现在的$proxy_add_x_forwarded_for变量的"X-Forwarded-For"部分是空的，所以只有$remote_addr，而$remote_addr的值是用户的ip，于是赋值以后，X-Forwarded-For变量的值就是用户的真实的ip地址了。


#### 第二台nginx
也使用 `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for`;

现在的$proxy_add_x_forwarded_for变量，
X-Forwarded-For部分包含的是用户的真实ip，
$remote_addr部分的值是上一台nginx的ip地址，
于是通过这个赋值以后现在的X-Forwarded-For的值就变成了`用户的真实ip，第一台nginx的ip`。

## refer
`unkown`来源 https://z.itpub.net/article/detail/CF6703F2C520289EEDE38ECA14A0A0DC