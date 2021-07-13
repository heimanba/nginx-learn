## 使用场景
1. 转发动态请求到后端服务器
```
location / {
    proxy_pass http://tomcat:8080/
}
```
2. 处理静态文件请求
```
location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}
```

## location 加载顺序
1. `=`    绝对相等
2. `^~`   头部匹配，相当于标准的匹配，但是权重更高
3. `~&,~*` 正则匹配
4. `无关键字`  头部匹配
