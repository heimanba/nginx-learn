## nginx执行命令分为11个阶段，
```
typedef enum {
    NGX_HTTP_POST_READ_PHASE = 0,
    NGX_HTTP_SERVER_REWRITE_PHASE,
    NGX_HTTP_FIND_CONFIG_PHASE,
    NGX_HTTP_REWRITE_PHASE,           //rewrite阶段在这里
    NGX_HTTP_POST_REWRITE_PHASE,
    NGX_HTTP_PREACCESS_PHASE,
    NGX_HTTP_ACCESS_PHASE,
    NGX_HTTP_POST_ACCESS_PHASE,
    NGX_HTTP_TRY_FILES_PHASE,
    NGX_HTTP_CONTENT_PHASE,
    NGX_HTTP_LOG_PHASE
} ngx_http_phases;
```
> `server`和`location`下的return之类有什么区别? server块下的return在第二阶段`SERVER_REWRITE`类中，而location下的return在第四阶段的`REWRITE`下。所以按照阶段来说是server下的return先发挥作用。


## break和last区别
```
location /break/ {
        rewrite ^/break/(.*) /test/$1 break;
        return 200 "break page";
    }

    location /last/ {
        rewrite ^/last/(.*) /test/$1 last;
        return 200  "last page";
    }

    location /test/ {
       return 200 "test page";
    }

}
```
1. break:
```
$ curl 127.0.0.1:8080/break/xxx
# 返回404，由于return 也发生在rewrite阶段
```
2. last
```
$ curl 127.0.0.1:8080/last/xxx
# 返回test page
```

> 原因: break和last都会停止处理后续的`REWRITE`阶段指令，但是last回从新发起请求，break不会。当请求break时，如匹配内容成功，直接返回，不存在则返回404。请求last时候，会重写新的URL重新发起请求。


## try_file
#### 语法
```
try_files file ... uri;
try_files file ... =code;
```
其中`file`遵循`root/alias`为根目录，寻找相对路径的结果。
```
location / {
    try_files $uri /$uri /index.html;
}
```
#### 挑战到指定配置
```
location  /  {
    try_files /app/cache/ $uri @fallback;
    index index.php index.html;
}

location  @fallback {
    rewrite ^/(.*)$ http://www.baidu.com        # 跳转到百度
}

```
#### 注意
try_files按照顺序检查文件是否存在，返回第一个找到的文件，至少需要两个参数，单最后一个是内部重定向和`rewrite`效果一致。如果不注意会有死循环造成500错误。


## rewrite和proxy_pass,try_file区别
- `rewrite`会改变浏览器URL的链接，同时把原来的URL转发都新的URL上
- `proxy_pass`会转发URL到新的upstream对应的服务器上，一般是外部的服务，可以在upstream里解决端口和负载的问题。

