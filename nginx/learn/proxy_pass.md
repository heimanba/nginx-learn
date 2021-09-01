## 根路径`/`的问题
```
daemon off;
events {
}
http {
    server {
        listen 8080;
        location /webapp/ {
            proxy_pass http://127.0.0.1:5000/api/;
        }
    }
}
```

#### 加上`/`的区别
| location | proxy_paas | Request | Received by upstream |
| ------ | ------ | ------ | ------ |
| /webapp/ | http://localhost:5000/api/ | /webapp/foo?bar=baz | /api/foo?bar=baz |
| /webapp/ | http://localhost:5000/api | /webapp/foo?bar=baz | /apifoo?bar=baz |
| /webapp | http://localhost:5000/api/ | /webapp/foo?bar=baz | /api//foo?bar=baz |
| /webapp | http://localhost:5000/api | /webapp/foo?bar=baz | /api/foo?bar=baz |
| /webapp | http://localhost:5000/api | /webappfoo?bar=baz | /apifoo?bar=baz |
总结一句话: 通常总是需要在尾部加上一个斜杠`/`,永远不要混合使用或者不使用尾部的斜杠。

#### $uri和$request_uri
比如请求路径为: http://127.0.0.1:8080/proxy/api?bar=baz

`$uri` 代表path路径`/proxy/api`
`$request_uri` 代表请求`/proxy/api?bar=baz`

#### 抓取正则数据
```
location ~ ^/api/cart/([a-z]*)/(.*)$ {
   proxy_pass http://127.0.0.1:5000/cart_api?$1=$2;
}
```
#### 

## 参考
https://dev.to/danielkun/nginx-everything-about-proxypass-2ona