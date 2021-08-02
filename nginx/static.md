## root/alias
作用：将URL映射为文件路径，以返回静态文件内容
#### root
```
location ^~ /123/abc/ {
    root /data/www;
}
```
当请求`http://127.0.0.1/123/abc/logo.png`时，会返回服务器上的`/data/www/123/abc/logo.png`文件

#### alias
```
server {
    listen 1001;
    location ^~ /123/abc/ {
        alias /data/www/;
    }
}
```
当请求`curl http://127.0.0.1:1001/123/abc/logo.png`时，会返回服务器的`/data/www/logo.png`文件。


#### 总结
- root 请求路径会带上location的前缀进行寻找,映射完成的文件路径
- alias 直接匹配`alias`后面的路径进行寻找，只映射location后的URL文件路径

## static模块中三个变量
- request_filename : 访问文件的完整路径包括扩展名
- document_root : 由URI和root/alias规则生成的文件夹路径 
- realpath_root : 将document_root软连接替换成真实路径
```
location /realpath{
    alias html/realpath;
    return 200 '$request_filename:$document_root:$realpath_root\n';
}
```
访问`http://xxxx/realpath/1.txt`则返回 `/usr/local/nginx/html/realpath/1.txt:/usr/local/nginx/html/realpath:/usr/local/nginx/html/realpath` 输出的路径

## 301重定向
当用户访问资源时没有在末尾加上反斜杠时候，NGINX会返回一个301重定向
- `absolute_redirect: on|off`
`off`可以取消绝对路径的重定向。变成相对路径
- `port_in_redirect on | off`

- `server_name_in_redirect on | off`
以`server_name`中配置，而不是`$Host`的配置域名生效
