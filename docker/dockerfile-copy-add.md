## 前言
`ADD`和`COPY`指令在Dockerfile中具有相同的功能。都是将构建的上下文的文件复制到镜像中去，预发规则如下。
```
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]

COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```
其中`src`(源文件)和`dest`(目标文件)，参数可以是文件或者目录。

## COPY
修改Dockerfile，将`cmd`文件复制到`app`中
```
FROM alpine:latest
COPY cmd app/
CMD ["sh"]
```
## ADD
1. `ADD`指令处理`COPY`指令的简单复制功能外，还支持从网络地址下载。
```
FROM alpine:latest
ADD https://github.com/vuejs/vue/archive/v2.6.10.tar.gz app/
CMD ["sh"]
```
查看镜像构建的结果，默认情况下载文件的权限为`600`, 如果不是想要的权限可以加一层`RUN`指令进行修改。

2. ADDN 源路径为打包的压缩文件
- 使用`wget`命令下载`vue.tar.gz`文件。
- 修改文件的`Dockerfile`为:
```
FROM alpine:latest
ADD v2.6.10.tar.gz app/
CMD ["sh"]
```
可以看到文件内容，Docker已经解压了这个文件。但是网络下载的形式不支持自动解压的。

## 总结
看起来`ADD`指令要比`COPY`指令功能更多，但是根据`Docker`最佳实践说明，除非需要解压缩功能，否则尽可能使用`COPY`指令，因为`COPY`语义很明确，就是复制文件而已。