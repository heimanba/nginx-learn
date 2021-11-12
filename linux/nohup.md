## 简介
如果想让某个程序在后台运行，希望使用`&`在程序结尾让程序自动运行。(但是如果终端关闭，那么程序也会关闭)

#### 方法1
使用 `nohup` 和 `&`,两者并行使用，方便后台正常运行
```
$ nohup node hello1.js &
nohup: ignoring input and appending output to 'nohup.out'
```
这里经常会出现，没有写入权限的错误

#### 方法2
```
nohup node /root/pro/serve.js >/dev/null 2>/dev/null & 
```
