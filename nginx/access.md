## ngx_http_access_module
作用: 限制某些IP地方的访问,内网限制非常有效
- 生效: NGX_HTTP_ACCESS_PHASE阶段
- 模块： http_access_module
#### 指令
```
allow address  CIDR|all
deny  address  CIDR|all
```
## auth_basic
作用： 提供一些简单的权限认证


## auth_request
功能: 向上有服务转发请求，如果上游服务返回响应码为2xx,则继续执行，如果返回401或者403则将响应返回给客户端
原因: 收到请求后，生成子请求，通过反向代理把请求传递给上游服务

#### 指令
```
auth_request url|pff;
auth_request_set $variable value;
```
#### 示例
```
location / {
    auth_request /test_auth;
}
location /test_auth {
    proxy_pass http://127.0.0.1:8090/auth_upstream;
    proxy_set_header Content-length "";
    proxy_set_header X-Originl-URI $request_uri;
}
```
如果server http://127.0.0.1:8090/auth_upstream 返回200,则返回NGINX配置的默认首页，否则返回对应的错误码


## satisfy指令
限制所有access阶段的模块
```
Syntax: satisfy all| any;
Context: http,server,locations
```

- satisfy all 所有模块都必须放行，请求才能向下执行
- satisfy any 任意一个模块同意放行，即使之前模块拒绝，或者之后模块拒绝，仍然可以继续执行请求。
