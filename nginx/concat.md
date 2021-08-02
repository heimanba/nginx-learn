## ngx_http_concat_module
当页面需要访问多个小文件时，把他们的内容合并到一次的http响应中返回，提升性能。来自阿里巴巴提供的模块https://github.com/alibaba/nginx-http-concat

#### 使用
在URL后加上`??`后通过多个`,`逗号进行分隔文件。如果还有参数，则在最后的通过`?`添加参数。
`http://example.com/??style1.css,style2.css,foo/style3.css?v=102234`

#### 规则
- concat: on|off
- concat_delimiter: string
多个文件的分隔符
- concat_types: MIME types
默认 text/css application/x-javascript
- concat_unique on | off
只对一种类型或者多种类型文件进行合并
- concat_ignore_file_error
有些文件出现错误，忽略，返回其他文件内容
- concat_max_file
最多合并多少个文件,默认是`10`个


#### 测试
```
server {
    concat on;
    location /concat {
        concat_unique on;
        concat_ignore_file_error on;
    }
}
```

`curl http://127.0.0.1/concat/??1.txt,2.txt`