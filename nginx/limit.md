## 前言
connection: 就是常说的tcp连接，通过三次握手建立的完整状态机，建立一个连接，需要三次握手。
request: 指的是请求，即http请求。
在一个连接的生命周期中，会存在一个或者多个请求，为了加快效率，避免每次请求都要三次握手建立连接，这就是`HTTP/1.1`协议特性，称为`heepalive`。

## ngx_http_limit_connn_module
#### 问题： 如何限制每个客户端的并发连接数
- 生效阶段: NGX_HTTP_PREACCESS_PHASE阶段
- 模块： http_limit_connn_module
- 生效范围:
    - 全部worker进程（基于共享内存）
    - 进入preprocess阶段前不生效
    - 限制的有效性取决于key的设计，依赖postread阶段的realip阶段的真实ip

#### 使用
1. 定义共享内存(包括大小),以及key关键字
```
Syntax: limit_conn_zone key zone=name:size; 
Context: http
```
2. 限制并发连接数
```
Syntax: limit_conn zone numer;
Context: http,server,location
```
3. 限制并发时日志级别
```
Syntax: limit_conn_log_level info|notice|warn|error;
default: limit_conn_log_level error;
Context: http,server,location
```
3. 限制发生时向客户端返回的错误码
```
Syntax: limit_conn_status code;
default: limit_conn_status 503;
Context: http,server,location
```

#### 示例
```
limit_conn_zone $binary_remote_addr zone_addr:10m;
server {
    root html/
    location / {
        limit_conn_status 500;
        limit_conn_log_level warn;
        limit_rate 50;
        limit_conn addr 1; # 演示效果
    }
}
```

## ngx_http_limit_req_module
作用: 限制每个客户端每秒处理的请求数
- 生效阶段: NGX_HTTP_PREACCESS_PHASE阶段
- 生效范围:
    - 全部worker进程(基于共享内存)
    - 进入preprocess阶段前不生效
#### 指令
1. 定义共享内存，以及Key关键字和限速速率
```
Syntax: limit_req_zone key zone=name:size rate=rate;
Context: http
```
2. 限制并发连接数
```
Syntax: limit_req zone=name [burst=number, nodelay];
Context: http,server,location
    - burst默认为0
    - nodelay 对burst中请求不在采用延时处理，而是立即处理
```
