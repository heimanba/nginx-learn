## WORKDIR 指定工作目录
格式为 `WORKDIR <工作目录路径>`
`WORKDIR`指令可以来指定工作目录, 以后各层的当前目录被改为指定的目录，如果该目录不存在，`WORKDIR`会帮你建立目录。
比如下面的错误
```
RUN cd /app
RUN echo "hello" > world.txt
```
构建镜像后，会发现找不到/app/world.txt文件。在Shell中，连续两行是共同一个进程执行环境，前一个命令修改的内存状态，会直接影响后一个命令；但是在`Dockerfile`中，这两行`RUN`命令的执行环境根本不同，是两个完全不同的容器。这就是对 `Dockerfile` 构建分层存储的概念不了解所导致的错误。


因此如果需要改变以后各层的工作目录的位置，那么应该使用 WORKDIR 指令。
```
WORKDIR /app

RUN echo "hello" > world.txt
```