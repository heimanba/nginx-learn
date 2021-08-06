
## lua指令
![image](https://img.alicdn.com/imgextra/i3/O1CN01Ae0y6o1tue2e8eHR5_!!6000000005962-2-tps-1005-910.png)

- set_by_lua: 流程分支处理判断变量初始化
- rewrite_by_lua: 转发重定向，缓存等功能(特定请求代理到外网)
- access_by_lua: IP准入，接口权限等情况集中处理(比如配合iptable完成防火墙)
- content_by_lua: 内容生成
- header_by_lua: 响应HTTP过滤处理(添加头部信息等)
- body_filter_by_lua: 响应BODY过滤处理(完成应答内容统一大写)
- log_by_lua: 会话完成后一步完成日志记录(日志可以记录在本地，也可以同步到其他机器)

#### 指令说明
- `*_by_lua <lua-script-str>`
后加字符串型的lua程序
```
location / {
    default_type text/html;
    content_by_lua '
        ngx.say("<p>hello, world</p>")
    ';
}
```
- `*_by_lua_block {lua_script}`
可以直接跟lua程序段
```
location / {
    content_by_lua_block{
        local test = 'Hello,world'
        ngx.say(test)
    }
}
```
- `*_by_lua_file {path-to-lua-script-file}`
跟lua文件路径
```
location ~ ^/api/([-_a-zA-Z0-9/]+) {
    # 准入阶段完成参数验证
    access_by_lua_file lua/access_check.lua;
    
    #内容生成阶段
    content_by_lua_file lua/$1.lua;
 
}
```

## 获取请求body
在NGINX的应用场景中，几乎都是只读取HTTP头即可，如负载均衡，正反向代理等场景。但是对于`API Server`或者`Web Application`对body可以说比较敏感的。
```
location /test {
    content_by_lua_block {
        local data = ngx.req.get_body_data()
        ngx.say("hello ", data)
    }
}
```
测试:
```
➜  ~  curl 127.0.0.1/test -d jack
hello nil
```
因为NGINX诞生之初，为了解决负载均衡问题，这样的场景下，是不需要读取body就可以决定负载均衡策略的。
```
server {
    listen    80;
    # 默认读取 body
    lua_need_request_body on;
    location /test {
        content_by_lua_block {
            local data = ngx.req.get_body_data()
            ngx.say("hello ", data)
        }
    }
}
```
如果只是某个部分需要读取`body`,而不是全局行为，可以显示调用`ngx.req.read_body() `
```
location /test {
    content_by_lua_block {
        ngx.req.read_body()
        local data = ngx.req.get_body_data()
        ngx.say("hello ", data)
    }
}
```

## 输出响应体
HTTP的响应报文分为三部分
- 响应行
- 响应头
- 响应体
`nginx.say`相比`nginx.print`多了一个换行输出`\n`。都是异步输出的
```
location /test_print {
    content_by_lua_block {
        ngx.say("hello")
        ngx.sleep(3)
        ngx.say("the world")
    }
}
```
可以看到同时输出`hello`和`the world`
#### 优雅处理响应体过大输出
1. 输出内容本身过大，比如超过`2G`的文件下载
使用http1.1的`CHUNKED`编码来完成。
```
location /test {
    content_by_lua_block {
        -- ngx.var.limit_rate = 1024*1024
        local file, err = io.open(ngx.config.prefix() .. "data.db","r")
        if not file then
            ngx.log(ngx.ERR, "open file error:", err)
            ngx.exit(ngx.HTTP_SERVICE_UNAVAILABLE)
        end

        local data
        while true do
            data = file:read(1024)
            if nil == data then
                break
            end
            ngx.print(data)
            ngx.flush(true)
        end
        file:close()
    }
}
```
本地文件内容(每次1KB)，以流失方式进行响应。以`data.db`大小是`4G`。NGINX服务维持内存占用`几MB`范畴
2. 输出内容本身由各种碎片拼凑，碎片数量庞大。
只需要利用`ngx.print`的特性，它的输入参数可以是单个或多个字符串参数，也可以是`table`对象
```
local table = {
    "hello, ",
    {"world: ", true, " or ", false,
        {": ", nil}}
}
ngx.print(table)
```

## 场景
#### 灰度发布的场景
我们有文件`release.lua`
```
clientip=ngx.req.get_headers()["ip"] --- 获取客户端IP真实情况下需要判断X-Real-IP X-Forworded—For这里用ip代替客户端的IP
is_in=0
local ipfile = io.open("iplist.txt","r") ---读取IP地址库这里我们可以使用Redis或者Memcached代替可以后台处理。
local ip_arr = {}
for line in ipfile:lines() do
    table.insert(ip_arr,line);
end

for i=1,#ip_arr do
    if clientip==ip_arr[i] then
        is_in=1
    end
end

if is_in == 1 then
    ngx.exec('/test_env');        --- 如果是灰度的用户跳转到新系统中
else
    ngx.exec('/product_env');    --- 如果是其他的用户还是在原先的生产环境
end
```
我们在`nginx.conf`配置
```
server {
    server_name localhost;
    location  = / {
        content_by_lua_file luafile/release.lua;
    }
    location = /test_env {
        content_by_lua '
            ngx.say("我是灰度用户所以我在新的测试系统上"); --- 实际上我们可以使用proxy_pass到后端服务的端口
        ';
    }
    location = /product_env {
        content_by_lua '
            ngx.say("我是非灰度用户所以我仍然在原先的生产环境上.");
        ';
    }
}
```
