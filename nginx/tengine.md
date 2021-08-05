## 安装
```
$ wget http://tengine.taobao.org/download/tengine-2.3.3.tar.gz
$ tar -zxvf tengine-2.3.3.tar.gz
$ ./configure
$ make
$ make install
```

可能会在`make`命令时候报错`make: *** No rule to make target build', needed bydefault’. Stop`
需要先安装依赖
```
yum -y install gcc-c++  pcre pcre-devel zlib zlib-devel openssl openssl-devel
```


#### tengine + lua 
1. LuaJIT安装
```
wget -c http://luajit.org/download/LuaJIT-2.0.4.tar.gz
tar xzvf LuaJIT-2.0.4.tar.gz
cd LuaJIT-2.0.4
make install PREFIX=/usr/local/luajit
#注意环境变量!
export LUAJIT_LIB=/usr/local/luajit/lib
export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
```
2. Tengine 安装
```
$ wget http://tengine.taobao.org/download/tengine-2.3.3.tar.gz
$ tar -zxvf tengine-2.3.3.tar.gz
$ cd tengine-2.3.3.tar.gz && ./configure --with-http_lua_module
$ make
$ make install
```

3. 问题
可能出现问题,找不到`luajit`
```
error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory
```
解决:
找到本地的libluajit-5.1.so.2共享库文件
```
$ find / -name libluajit-5.1.so.2
/usr/local/luajit/lib/libluajit-5.1.so.2
```
如果找到这个文件:
```
cp /usr/local/luajit/lib/libluajit-5.1.so.2 /usr/local/lib/
echo "/usr/local/lib"  >>/etc/ld.so.conf
/sbin/ldconfig
```

4. 测试
- 修改nginx配置
```
location /hello_lua {
    content_by_lua '
        ngx.say("Lua: hello world!")
    ';
}
```
- 运行tengine
```
/opt/tengine/sbin/nginx start
```
- 访问hello_lua
```
$ curl http://localhost/hello_lua
Lua: hello world!
```
