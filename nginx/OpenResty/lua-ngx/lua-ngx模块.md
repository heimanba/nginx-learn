### 输出
HTTP输出分为三部分
- 响应行
- 响应头
- 响应体
#### `ngx.say`和`ngx.print`
两者均为异步输出，而且输出的都是响应体，区别是`ngx.say`会输出响应体多输出一个`\n`。


## ngx.exit VS ngx.eof
#### ngx.exit(status)
- 当`status`>=200时(ngx.HTTP_OK以上)
本函数中断当前请求执行并返回状态给`nginx`。注意是`所有的请求`
- 当`status ===0`(ngx.OK)时
本函数退出当前”处理阶段句柄“，继续执行下一阶段请求。

下面代码
```
ngx.status = ngx.HTTP_GONE
ngx.say("hello request")
-- 退出整个请求而不是当前处理阶段
ngx.exit(ngx.HTTP_NOT_FOUND)
```
返回效果
```
< HTTP/2 410
HTTP/2 410
< server: nginx
server: nginx
< date: Sat, 09 Oct 2021 11:50:41 GMT
date: Sat, 09 Oct 2021 11:50:41 GMT
< content-type: image/png
content-type: image/png

<
hello1111
```
因为调用此方法会终端当前请求处理，所以建议代码风格和`return`语句一起调用此方法。`return ngx.exit(...)`来强调处理过程已经中断。因为在未来版本中`ngx.exit`可能是异步操作，所以建议使用`return`语句来做中断。

#### ngx.eof
ngx.eof 显式指定了响应流输出的结束，后面的逻辑继续在服务端执行

 `ngx.exit()` 和 `ngx.eof（）` 本质区别在于`ngx.exit()`作用在于中断当前操作，不管是ngx-lua模块请求处理的当前阶段还是整个请求，而 `ngx.eof（）` 只是结束响应流的输出，中断HTTP连接，后面的代码逻辑还会继续在服务端执行，而且 `ngx.eof（）`支持运行的上下文比 `ngx.exit（）`少太多， `ngx.eof（）` 有返回值， `ngx.exit（）`则没有，因为请求已经结束

 在响应头发出去后更改`ngx.status`会在错误日志中记录一条`[error]`，这个错误对本次请求响应没有很大影响，但是可能是一个未知的坑。
 