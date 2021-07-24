## ENV设置环境变量
格式分别为
- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>`
无论是后面的其他指令，如`RUN`，还是运行时的应用，都可以直接使用这里定义的环境变量。
```
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```
通过环境变量，我们可以让一份`Dockerfile`制作更多的镜像，只需哟使用不同的环境变量即可

## ARG构建参数
格式：`ARG <参数名>[=<默认值>]`
构建参数和`ENV`效果一样，都是设置环境变量。不同的是，`ARG`所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此使用`ARG`保存密码之类的信息，因为`docker history`还是可以看到所有的值。
`ARG`用于传入外部参数，定义在`FROM`指令前，`FROM`后的其他指令无法使用`ARG`定义的环境变量，如果`FROM`指令后的指令要使用`ARG`定义的值，需要在FROM后再次定义。

`Dockerfile`里面ARG指令定义了一个变量，在运行`docker build`命令时使用`--build-arg <varname>= <value>`参数传递给构建器。
```
# Dockerfile
FROM redis:3.2-alpine

LABEL maintainer="GPF <5173180@qq.com>"

ARG REDIS_SET_PASSWORD=developer
ENV REDIS_PASSWORD ${REDIS_SET_PASSWORD}

VOLUME /data

EXPOSE 6379

CMD ["sh", "-c", "exec redis-server --requirepass \"$REDIS_PASSWORD\""]
```
