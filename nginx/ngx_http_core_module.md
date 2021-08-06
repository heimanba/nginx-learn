#### $request_body
`POST`请求体，当请求正文被读取到内存缓冲区时，该变量的值在`proxy_paas`,`fastcgi_pass`,`uwsgi_pass` 和 `scgi_pass` 指令处理的位置中可用。
#### 在proxy_paas中
```
http {
    log_format  dm  ' "$request_body" ';
 
    server {
        location /post/ {
            access_log /home/shuhao/openresty-test/logs/post.log dm;
            return 202;
        }
    }
}
```
测试: `curl -i -X POST  -d "arg1=1&arg2=2" "http://127.0.0.1/post/"`

这里的`access.log`中的日志为：
```
"-"
```
#### 不在proxy_paas中
```
http {
    log_format  dm  ' "$request_body" ';
 
    upstream bk_servers_2 {
        server 127.0.0.1:6699;
    }
 
    server {
        listen 6699;
        location /post/ {
            proxy_pass http://bk_servers_2/api/log/letv/env;
            access_log /home/shuhao/openresty-test/logs/post.log dm;
        }
        location /api/log/letv/env {
            return 202;
        }
    }
}
```
里面返回值`"arg1=1&arg2=2"`