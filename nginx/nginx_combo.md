## 起源

在前端监控的场景中，有些请求需要合并资源上报。但是在 nginx 的`access.log`日志中，又需要拆分成多条日志。这时候，我们需要将多条资源的请求放在`body`中。这时候就衍生了几个问题

#### NGINX 如何处理 post 的请求

我们知道在 NGINX 的 `log_format`中有下面格式：`$request_body`。这时候我们可以定义

```
log_format  postformat  "$remote_addr||$request_time_usec||$http_x_readtime||$time_local||$request_method||http://$host$request_uri$request_body||$status||$body_bytes_sent";
```

使用配置(只有通过 proxy_paas 才支持 HTTP POST 请求)

```
location / {
    if ($request_method = "POST"){
        # 如果不经过proxy_pass转发，客户端拿不到状态码
        proxy_pass $scheme://$server_name:$server_port/post;
        access_log  logs/access.log  postformat;
    }
    # proxyformat;
    access_log  logs/access.log  proxyformat;
}
location /post {
    return 200;
}
```

但是上面的配置存在下面几个问题

1. `$request_body`内容为空
   在 POST 参数数据比较少时候，`$request_body`可以正常记录，当数据量比较大时，`$request_body`为空
   因为内容已经缓存存储在临时文件中了，无法记录`POST`的请求参数，需要修改`nginx`配置

```
client_body_buffer_size 1024k;  #缓冲区代理缓冲用户端请求的最大字节数
```

2. 不支持 https 请求
   `proxy_pass $scheme://$server_name:$server_port/post`在解析 https 时候报错。

这里的方案为使用 lua 获取`$request_body`

```
lua_need_request_body  on;
location / {
    if ($request_method = "POST"){
        access_log  /home/admin/logs/access.log  postformat;
        content_by_lua 'res = ngx.var.request_body ngx.say("")';
    }
    add_header Access-Control-Allow-Origin *; # 跨域iframe请求
    add_header timing-allow-origin *; # 跨域iframe请求
}
```

#### 主请求不落 access.log,子请求落 access.log

请求路径中，使用`_combo=1`标识使用子请求的模式，比如 url 路径为`http://127.0.0.1/r.png?_combo=1&t=recource`

```
log_format  subpostformat  "$remote_addr||$request_time_usec||$http_x_readtime||$time_local||$request_method||http://$host$uri$args$request_body||$status||$body_bytes_sent";

# 默认读取 body
lua_need_request_body on; (可以使用局部行为 ngx.req.read_body();)
# 主请求
location / {
    if ($request_method = "POST"){
        access_log  logs/access.log  postformat;
    }
    if ($arg__combo = 1){
        access_log off; # 如果有参数`_combo=1`,覆盖上面的`access_log`配置
    }
    content_by_lua_file lua/post.lua;
    add_header Access-Control-Allow-Origin *; # 跨域iframe请求
    add_header timing-allow-origin *; # 跨域iframe请求
}
# 子请求
location /subrequest {
    internal; # 只允许内部请求
    if ($request_method = "POST"){
        access_log  logs/access.log  subpostformat;  # 默认access.log中`request_uri`会记录主请求，这里使用 `$uri$args`才会打印子请求
        content_by_lua 'ngx.say("")';
    }
}
```

#### lua 处理请求

```
local cjson = require "cjson"
local get_arg = ngx.req.get_uri_args()
local combo = ngx.var.arg__combo
ngx.req.read_body(); # 开启获取post数据

if (combo == "1") then
    local postData = ngx.req.get_body_data();
    ngx.location.capture("/subrequest", {
        args=string.gsub(ngx.var.request_uri,"&_combo=1","",1),
        --body=postData
    })
else
    ngx.say("")
end
```
