## 指令
```

    location [=|~*|^~] uri {...}
    location @name{...}
Context: server,location
```
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

## location 匹配规则
![顺序](https://img.alicdn.com/imgextra/i2/O1CN01fCwZxB1hiwWqq69IV_!!6000000004312-0-tps-2400-1080.jpg)

1. `=`    绝对相等
2. `^~`   头部匹配，相当于标准的匹配，匹配成功不会再次匹配了
3. `~&,~*` 正则匹配
4. `无关键字`  头部匹配
