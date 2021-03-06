## 为什么需要动态模块
很多时候我们安装完Nginx后，随着业务的发展，往往需要给安装好的nginx添加其他的功能模块。在为Nginx添加其他功能模块的时候，要求Nginx不停机。这就涉及到如何为已安装的Nginx动态的添加模块的问题。

## 源码编译安装(centos)
1. 安装依赖包
```
sudo yum -y install gcc gcc-c++ # nginx 编译时依赖 gcc 环境
sudo yum -y install pcre pcre-devel # 让 nginx 支持重写功能
sudo yum -y install zlib zlib-devel # zlib 库提供了很多压缩和解压缩的方式，nginx 使用 zlib 对 http 包内容进行 gzip 压缩
sudo yum -y install openssl openssl-devel # 安全套接字层密码库，用于通信加密
```
2. 源码包下载 https://nginx.org/download/nginx-1.21.1.tar.gz
```
sudo tar -zxvf  nginx-1.21.1.tar.gz # 解压缩
```
解压完成后进入 nginx-1.21.1目录进行源码编译安装
```
$ cd nginx-1.21.1
$ ./configure --prefix=/usr/local/nginx # 检查平台安装环境
  # --prefix=/usr/local/nginx 是 nginx 编译安装的目录（推荐|默认），安装完后会在此目录下生成相关文件
```
默认配置：
```
nginx path prefix: "/usr/local/nginx"
nginx binary file: "/usr/local/nginx/sbin/nginx"
nginx modules path: "/usr/local/nginx/modules"
nginx configuration prefix: "/usr/local/nginx/conf"
nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
nginx pid file: "/usr/local/nginx/logs/nginx.pid"
nginx error log file: "/usr/local/nginx/logs/error.log"
nginx http access log file: "/usr/local/nginx/logs/access.log"
nginx http client request body temporary files: "client_body_temp"
nginx http proxy temporary files: "proxy_temp"
nginx http fastcgi temporary files: "fastcgi_temp"
nginx http uwsgi temporary files: "uwsgi_temp"
nginx http scgi temporary files: "scgi_temp"
```
3. 启动服务
```
sudo /usr/local/nginx/sbin/nginx
```
4. 重新加载服务
```
sudo  /usr/local/nginx/sbin/nginx -s reload
```

#### 第一步：确定您的nginx版本
1. nginx开源版本需要大于1.19
```
$ nginx -v
nginx version: nginx/1.19.10
```
2. 下载nginx源码包[nginx](https://nginx.org/en/download.html)
```
$ wget https://nginx.org/download/nginx-1.21.1.tar.gz
$ tar -xzvf nginx-1.21.1.tar.gz
```
#### 第二步：下载三方模块源文件
```
git clone https://github.com/perusio/nginx-hello-world-module.git
```
#### 第三步：编译动态模块
```
$ cd nginx-1.21.1/
$ ./configure --with-compat --add-dynamic-module=../nginx-hello-world-module
$ make modules # 只编译模块，注意不要只想make install
```
同时拷贝模块文件(`.so`文件) 到模块目录 /usr/local/nginx/modules
```
sudo cp objs/ngx_http_hello_world_module.so /usr/local/nginx/modules
```
#### 第四步：加载三方模块
在`nginx.conf`文件中加上配置
```
load_module modules/ngx_http_hello_world_module.so;
```
在http server中配置：
```
server {
    listen 80;

    location / {
         hello_world;
    }
}
```
#### 第五步: 验证
```
$ /usr/local/nginx/sbin/nginx -s reload
$ curl 127.0.0.1
hello world
```