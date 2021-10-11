## 介绍
Lua 中使用的 API 以两个标准模块的形式封装：ngx 和 ndk。这两个模块在 ngx_lua 默认的全局作用域中，在 ngx_lua 指令中总是可用。
### ngx.arg
当被用在 `set_by_lua*` 指令环境中时，本表是一个只读表，包含输入参数供配置命令使用：

### ngx.var.VARIABLE
读写 Nginx 变量值。
```
location /foo {
    set $my_var ''; # 需要在设置时创建 $my_var 变量
    content_by_lua_block {
        ngx.say(ngx.var.my_var)
        ...
    }
}
```
### ngx.ctx
这个 Lua 表可以用来存储基于请求的 Lua 环境数据，其生存周期与当前请求相同 (类似 Nginx 变量)。

#### 各个处理阶段`ngx.ctx`保持一致
```
location /test {
    rewrite_by_lua_block {
        ngx.ctx.foo = 76
    }
    access_by_lua_block {
        ngx.ctx.foo = ngx.ctx.foo + 3
    }
    content_by_lua_block {
        ngx.say(ngx.ctx.foo)
    }
}
```
使用`curl /test`返回`79`,`ngx.ctx.foo` 条目跨越一个请求的 `rewrite (重写)`，`access (访问)`，和 `content (内容)` 各处理阶段保持一致。

#### 每个请求，包括子请求，都有一份自己的`ngx.ctx`
```
location /sub {
    content_by_lua_block {
        ngx.say("sub pre: ", ngx.ctx.blah)
        ngx.ctx.blah = 32
        ngx.say("sub post: ", ngx.ctx.blah)
    }
}

location /main {
    content_by_lua_block {
        ngx.ctx.blah = 73
        ngx.say("main pre: ", ngx.ctx.blah)
        local res = ngx.location.capture("/sub")
        ngx.print(res.body)
        ngx.say("main post: ", ngx.ctx.blah)
    }
}
```

### ngx.location.capture

#### 基本使用
发起一个同步非阻塞 Nginx 子请求。需要注意的是，子请求只是模拟 HTTP 接口的形式， 没有 额外的 HTTP/TCP 流量，也 没有 IPC (进程间通信) 调用。所有工作在内部高效地在 C 语言级别完成。

因为 Nginx 内核限制，子请求不允许类似 `@fo`o 命名 location。请使用标准 `location`，并设置 internal 指令，仅服务内部请求。

假设我们发起一个POST请求，如下
```
res = ngx.location.capture(
    '/foo/bar',
    { method = ngx.HTTP_POST, body = 'hello, world' }
)
```

#### 变量传递
- share_all_vars
选项控制是否让当前请求与其子请求共享 nginx 变量。如果设为 true，则当前请求与所有子请求共享所有 nginx 变量作用域。在子请求中修改 nginx 变量将影响当前请求
- copy_all_vars
在子请求被创建时，提供给它一份父请求的 Nginx 变量拷贝。在子请求中对这些变量的修改将不会影响父请求和其他任何共享父请求变量的子请求
- 通过传递`vars`选项来设置子请求变量值。
```
location /other {
    content_by_lua_block {
        ngx.say("dog = ", ngx.var.dog)
        ngx.say("cat = ", ngx.var.cat)
    }
}

location /lua {
    set $dog '';
    set $cat '';
    content_by_lua_block {
        res = ngx.location.capture("/other",
            { vars = { dog = "hello", cat = 32 }});

        ngx.print(res.body)
    }
}
```
### ngx.location.capture_multi

#### 基本使用
并允许多个子请求并行访问,此函数根据输入列表并行发起多个子请求，并按原顺序返回结果
```
res1, res2, res3 = ngx.location.capture_multi{
    { "/foo", { args = "a=3&b=4" } },
    { "/bar" },
    { "/baz", { method = ngx.HTTP_POST, body = "hello" } },
}

if res1.status == ngx.HTTP_OK then
    ...
end

if res2.body == "BLAH" then
    ...
end
```
#### 当子请求未知
通过`Lua`表来存储所有的请求和返回
```
-- 创建请求表
local reqs = {}
table.insert(reqs, { "/mysql" })
table.insert(reqs, { "/postgres" })
table.insert(reqs, { "/redis" })
table.insert(reqs, { "/memcached" })

-- 同时执行所有请求，并等待所有返回
local resps = { ngx.location.capture_multi(reqs) }

-- 循环返回表
for i, resp in ipairs(resps) do
    -- 处理返回表 "resp"
end
```

### ngx.status
读写当前请求的响应状态码。这个方法需要在发送响应头前调用。
```
ngx.status = ngx.HTTP_CREATED
status = ngx.status
```
> 要在响应发送前设置` ngx.status`,否在不生效，并在ngix错误日志生成`ERRPR`记录

### ngx.header.HEADER
修改，添加，或者清除当前请求待发送的`HEADER`响应头信息
```

-- 与 ngx.header["Content-Type"] = 'text/plain' 相同
ngx.header.content_type = 'text/plain';

ngx.header["X-My-Header"] = 'blah blah';

```
这个 API 在 `header_filter_by_lua*` 环境中非常有用，例如：
```

location /test {
    set $footer '';

    proxy_pass http://some-backend;

    header_filter_by_lua '
        if ngx.header["X-My-Header"] == "blah" then
            ngx.var.footer = "some value"
        end
    ';

    echo_after_body $footer;
}

```
### ngx.resp.get_headers
返回一个 Lua 表，包含当前请求的所有响应头信息。
```
local h = ngx.resp.get_headers()
for k, v in pairs(h) do
    ...
end
```
### ngx.req
#### ngx.req.is_internal
返回一个布尔值，说明当前请求是否是一个“内部请求”，既：一个请求的初始化是在当前 nginx 服务端完成初始化，不是在客户端。

#### ngx.req.start_time
返回当前请求创建时的时间戳，格式为浮点数，其中小数部分代表毫秒值。
以下用 Lua 代码模拟计算了 $request_time 变量值 (由 ngx_http_log_module 模块生成)
```
local request_time = ngx.now() - ngx.req.start_time()
```
#### ngx.req.http_version
返回一个 Lua 数字代表当前请求的 HTTP 版本号。

当前的可能结果值为 2.0, 1.0, 1.1 和 0.9。无法识别时值时返回 nil。


#### ngx.req.raw_header

返回 Nginx 服务器接收到的原始 HTTP 协议头。

默认时，请求行和末尾的 CR LF 结束符也被包括在内。例如，
```
ngx.print(ngx.req.raw_header())
```
可以通过指定可选的 no_request_line 参数为 true 来去除结果中的请求行。例如，
```
ngx.print(ngx.req.raw_header(true))
```
#### ngx.req.get_method
获取当前请求的`HTTP`请求方案名称，结果类似`GET`和`POST`字符串
#### ngx.req.set_method
改写当前请求的`HTTP`请求方法，仅支持 [https://github.com/iresty/nginx-lua-module-zh-wiki#http-method-constants](HTTP 请求方法) 中定义的数值常量，例如 ngx.HTTP_POST 和 ngx.HTTP_GET。

#### ngx.req.set_uri
用 uri 参数重写当前请求 (已解析过的) URI。该 uri 参数必须是 Lua 字符串，并且长度不能是 0，否则将抛出 Lua 异常。
```
rewrite ^ /foo last;
// => 等同于
ngx.req.set_uri("/foo", false)
```
稍微复杂些例子
```
location /test {
    rewrite ^/test/(.*) /$1 break;
    proxy_pass http://my_backend;
}
// => 等同于
location /test {
    rewrite_by_lua '
        local uri = ngx.re.sub(ngx.var.uri, "^/test/(.*)", "/$1", "o")
        ngx.req.set_uri(uri)
    ';
    proxy_pass http://my_backend;
}
```
#### ngx.req.set_uri_args
用 args 参数重写当前请求的 URI 请求参数。args 参数可以是一个 Lua 字符串，比如
```
ngx.req.set_uri_args("a=3&b=hello%20world")
```
#### ngx.req.get_uri_args
返回一个 Lua table，包含当前请求的所有 URL 查询参数。
```
location = /test {
    content_by_lua_block {
        local args = ngx.req.get_uri_args()
        for key, val in pairs(args) do
            if type(val) == "table" then
                ngx.say(key, ": ", table.concat(val, ", "))
            else
                ngx.say(key, ": ", val)
            end
        end
    }
}
```
#### ngx.req.get_post_args
返回一个 Lua table，包含当前请求的所有 POST 查询参数 (MIME type 是 application/x-www-form-urlencoded)。使用前需要调 用 ngx.req.read_body 读取完整请求体，或通过设置 lua_need_request_body 指令为 on 以避免报错。
```
location = /test {
    content_by_lua_block {
        ngx.req.read_body()
        local args, err = ngx.req.get_post_args()
        if not args then
            ngx.say("failed to get post args: ", err)
            return
        end
        for key, val in pairs(args) do
            if type(val) == "table" then
                ngx.say(key, ": ", table.concat(val, ", "))
            else
                ngx.say(key, ": ", val)
            end
        end
    }
}
```
#### ngx.req.get_headers
返回一个 Lua table，包含当前请求的所有请求头信息。
```
local h = ngx.req.get_headers()
for k, v in pairs(h) do
    ...
end
```
#### ngx.req.set_header
将当前请求的名为`header_name`的头信息值设置为`header_value`,如果此头信息名称已经存在，修改其值。
默认时，之后通过 [https://github.com/iresty/nginx-lua-module-zh-wiki#ngxlocationcapture](ngx.location.capture) 和 [https://github.com/iresty/nginx-lua-module-zh-wiki#ngxlocationcapture_multi](ngx.location.capture_multi) 发起的所有子请求都将继承新的头信息。

#### ngx.req.clear_header
清除当前请求的名为 `header_name` 的请求头信息。已经存在的子请求不受影响，此命令之后发起的子请求将默认继承修改后的头信息。

#### ngx.req.read_body
同步读取客户端请求体，不阻塞 Nginx 事件循环。
```
ngx.req.read_body()
local args = ngx.req.get_post_args()
```
#### ngx.req.get_body_data
取回内存中的请求体数据。本函数返回 Lua 字符串而不是包含解析过参数的 Lua table。如果想要返回 Lua table，请使用 ngx.req.get_post_args 函数。


### ngx.exec
使用 uri、args 参数执行一个内部跳转
```
ngx.exec('/some-location?a=3&b=5', 'c=6');
```
下面例子`GET /foo/file.php?a=hello`
```

location /foo {
    content_by_lua_block {
        ngx.exec("@bar", "a=goodbye");
    }
}

location @bar {
    content_by_lua_block {
        local args = ngx.req.get_uri_args()
        for key, val in pairs(args) do
            if key == "a" then
                ngx.say(val)
            end
        end
    }
}
```
### gx.redirect
发出一个 HTTP 301 或 302 重定向到 uri
```
return ngx.redirect("/foo")
// 等同于
// return ngx.redirect("/foo", ngx.HTTP_MOVED_TEMPORARILY)
rewrite ^ /foo? redirect;
// 等同于
//  return ngx.redirect('/foo');
rewrite ^ /foo? permanent;
// 等同于
//  return ngx.redirect('/foo', ngx.HTTP_MOVED_PERMANENTLY) 
```