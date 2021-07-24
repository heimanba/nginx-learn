## CMD 容器启动命令
格式为:
- `shell` 格式： CMD <命令>
- `exec` 格式：`CMD` ["可执行文件", "参数1", "参数2"]

Docker不是虚拟机，容器就是进程，所以在启动容器的时候，需要指定所运行的程序及参数。`CMD`指令就是用于指定默认的容器主进程的启动命令的。
在运行时可以指定新的命令来替代这个默认命令，比如`ubuntu`镜像默认的`CMD`是`/bin/bash`,我们运行`docker run -it ubuntu`的话，会直接进入`bash`。也可以在运行时候指定运行别的命令`docker run -it ubuntu cat /etc/os-release`。这里使用`cat /etc/os-release`命令替换了默认的`/bin/bash`命令。

#### 容器应用在前台执行和后台执行
因为Docker是进程，容器中的应用只能以前台方式执行，而不能像虚拟机，使用`systemd`启动后台程序，容器中没有后台服务概念。
```
CMD service nginx start
```
如果使用上述方式启动，容器会执行后立即退出。对于容器而言，启动程序就是容器应用进程，容器就是为了`主进程`而存在的。`主进程`退出，容器就失去了存在意义。
`CMD service nginx start`会被理解为`CMD [ "sh", "-c", "service nginx start"]`,主进程实际为`sh`。那么当`service nginx start`命令结束后，`sh`也就结束了，`sh`作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行`nginx`可执行问津，并且以前台形式运行。
```
CMD ["nginx", "-g", "daemon off;"]
```


## ENTRYPOINT
`ENTRYPOINT`的目的和`CMD`一样，都是在指定容器启动程序及参数。`ENTRYPOINT`在运行时也可以替代，不过比`CMD`要略显繁琐，需要通过`docker run`的参数`--entrypoint`来指定。

一旦指定了`ENTRYPOINT`后，`CMD`的含义就会发生改变，不在直接运行其命令，而是将`CMD`的内容作为参数传递给`ENTRYPOINT`指令，实际执行变成
```
<ENTRYPOINT> "<CMD>"
```
为啥有了`CMD`还需要`ENTRYPOINT`?`<ENTRYPOINT> "<CMD>"`有什么好处呢?

#### 场景一：让镜像变成命令一样使用
```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://myip.ipip.net" ]
```
使用`docker build -it myip`，来构建镜像时，如果需要查询当前公网IP，只需要执行
```
docker run myip
```
但是如果需要传递参数的时候，如果我们希望现实HTTP头信息，需要加上`-i`参数。
```
docker run myip -i
```
可以看到可执行文件找不到报错,`executable file not found`。

使用`ENTRYPOINT`可以很好的解决此类问题。
```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

#### 应用运行前的准备工作
启动容器就是启动主进程，但有时候，启动主进程前，需要做一些准备工作。
比如`mysql`类数据库，可能需要一些数据库配置，初始化工作，这些工作要在最终`mysql`服务器运行之前解决。

这些准备工作和`CMD`无关，无关`CMD`Wie什么，都需要事先执行一个预处理工作。可以写一个脚本，放在`ENTRYPOINT`中执行，这个脚本将会接到的参数(也就是`<CMD>`)作为命令，在脚本最后执行。官方的`redis`就是这样执行的。
```
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```
可以看到，在最后指定了`ENTRYPOINT`为`docker-entrypoint.sh`脚本
```
#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    find . \! -user redis -exec chown redis '{}' +
    exec gosu redis "$0" "$@"
fi

exec "$@"
```
根据该脚本的内容就是根据`CMD`来判断，如果是`redis-server`，则切换为到`redis`用户身份启动，否则使用`root`身份执行。
```
$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)
```