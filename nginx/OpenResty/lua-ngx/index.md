## 前言
nginx-lua-module是nginx的`HTTP`子系统插件，只能对HTTP环境下游进行对话(http 0.9/1.0/1.1/2.0, WebSockets等)，如果想扩展`TCP`的客户端能力，使用 [ngx_stream_lua](https://github.com/openresty/stream-lua-nginx-module#readme) 模块
其中 lua state会被共享给单个nginx worker内所有的请求，达到最小化内存消耗

#### 环境变量输出
如果想在lua API使用`os.getenv`访问系统变量，则需要在`nginx.conf`中，通过`env`指令，将此环境变量列举出来
```
env foo
```
#### http 1.0支持
http1.0不支持分块输出，响应体不会看时候，需要在响应头明确指定`Content-Length`，以支持 HTTP 1.0 长连接。[lua_http10_buffering](https://github.com/iresty/nginx-lua-module-zh-wiki#lua_http10_buffering)设置为`on`时，ngx_lua将缓存的`ngx.say`和`ngx.print`输出，并且构建正确的`content_length`响应头给HTTP1.0客户端

#### 静态链接纯lua模块
首先将`.lua`将lua模块编译成`.o`目标文件(包含导出的字节码数据)，然后链接这些`.o`文件到NGINX构造环境。
```
/path/to/luajit/bin/luajit -bg foo.lua foo.o
```
在构建nginx时，传递给`./configure`
```
./configure --with-ld-opt="/path/to/foo.o" ...
```
#### nginx worker数据共享
同一个`ngx worker`进程处理共享数据，将数据封装一个lua模块，在通过require加载，就可以操作了
```
-- mydata.lua
local _M = {}

local data = {
    dog = 3,
    cat = 4,
    pig = 5,
}

function _M.get_age(name)
    return data[name]
end

return _M
```
然后通过`nginx.conf`访问
```
location /lua {
    content_by_lua_block {
        local mydata = require "mydata"
        ngx.say(mydata.get_age("dog"))
    }
}
```
请求`/lua`时候被加载运行，之后的同一个nginx worker进程所有请求，都加载这个模块实例，共享实例数据。直到NGINX主进程收到`HUP`信号强制重新加载。
> 注意：这种方式不能跨多个`worker`之间进行共享。如果是整个服务之间共享数据，可以使用 [ngx.shared.DICT](https://github.com/iresty/nginx-lua-module-zh-wiki#ngxshareddict) API

#### lua值边界
在代码中导入模块，推荐使用
```
local xxx = require('xxx')
```
而不是
```
xxx = require('xxx')
```
#### 特殊符号
因为反斜杠`\`会被Lua语言和NGINX配置文件解析器在执行时候同时处理，所以下面无法按照预期执行
```
# nginx.conf
location /test {
    content_by_lua_block {
    local regex = "\d+"  -- 这里是错的!!
    local m = ngx.re.match("hello, 1234", regex)
    if m then ngx.say(m[0]) else ngx.say("not matched!") end
    }
}
# 结果为 "not matched!"
```
正确的正则为`local regex = "\\\\d+"`
如果通过`*_by_lua_file`指令执行，反斜杠只需要被Lua语言执行处理。
```
-- test.lua
local regex = "\\d+"
local m = ngx.re.match("hello, 1234", regex)
if m then ngx.say(m[0]) else ngx.say("not matched!") end
-- 结果为 "1234"
```
