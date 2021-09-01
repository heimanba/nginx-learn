#### internal
只能用于内部请求。对于外部请求，返回客户端错误 404（未找到）。内部请求如下：
```
error_page 404 /404.html;

location = /404.html {
    internal;
}
```