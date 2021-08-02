## autoindex
当URL以`/`结尾时，尝试以`html/xml/json/jsonp`等格式返回`root/alias`中指向的目录结构

## index
默认值为`index index.html`，如果没有给出index默认初始值为`index.html`

> `index`指令并不是查到文件后，直接拿过来使用，实际工作方式为
1.如果文件存在，则使用文件作为路径，发起重定向。看上去就像再一次从客户端发起请求，NGINX再一次搜索`location`一样
2. 既然是`内部重定向`,域名+端口不会发生变化，只会在同一个`server`下搜索
3. 如果`内部重定向`发生在`proxy_pass`反向代理后，重定向只会发生在代理配置的同一个`server`

```
server {
    listen      80;
    server_name example.org www.example.org;    
    
    location / {
        root    /data/www;
        index   index.html index.php;
    }
    
    location ~ \.php$ {
        root    /data/www/test;
    }
}
```
访问`example.org`,会先访问`/`的`location`，如果`/data/www/index.html`不存在，直接使用`/data/www/index.php`