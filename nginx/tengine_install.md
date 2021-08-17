#### 编译按照luajit
```
$ wget -c http://luajit.org/download/LuaJIT-2.0.4.tar.gz
$ tar xzvf LuaJIT-2.0.4.tar.gz
$ cd LuaJIT-2.0.4
$ make install PREFIX=/usr/local/luajit
$ ln -s /usr/local/luajit/bin/luajit /usr/local/bin/luajit # 全局使用luajit命令
#注意环境变量!
$ export LUAJIT_LIB=/usr/local/luajit/lib
$ export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
```
可能出现的问题,找不到 `luajit`
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

#### 安装tengine
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

#### luajit 和lua-cjson恩怨情仇
1. luajit环境下安装lua-cjson
```
lua用于Nginx环境的lua开发时候，使用lua-cjson需要将其编译进luajit
需要改动lua-cjson默认的Makefile文件，编译相关的默认目录指定的是系统lua

修改为:
# PREFIX =            /usr/local
PREFIX =            /usr/local/luajit
# LUA_INCLUDE_DIR =   $(PREFIX)/include
LUA_INCLUDE_DIR =   $(PREFIX)/include/luajit-2.0
# LUA_MODULE_DIR =    $(PREFIX)/share/lua/$(LUA_VERSION)
LUA_MODULE_DIR =    $(PREFIX)/share/luajit-2.0.4
```
修改后执行编译安装
```
make
sudo make install
```

2. lua 环境的 lua-cjson 下载安装
```
wget https://www.kyne.com.au/~mark/software/download/lua-cjson-2.1.0.tar.gz
cd lua-cjson-2.1.0/
make
make install
```

## refer 
mac机器安装cjson版本
https://github.com/gedennis/blog/issues/6
