## 前言
在构建Docker镜像的电脑查看本地镜像的列表，可能看到有展示为 `<none>:<none`的镜像列表：
这种镜像在Docker文档称为`dangling images`,指的是没使用标签并且没有被容器使用的镜像。

## 怎么来的
1. 第一次构建镜像时生成镜像ID为`079dbd67f9f4`,此镜像会被构建工具加上标签`dankun/rectcode:0.0.1`;
2. 第一次构建镜像时生成镜像ID为`e40a97f764ef`,此镜像会被构建工具加上标签`dankun/rectcode:0.0.1`;
3. `Docker`会移除`079dbd67f9f4`的标签，此时`079dbd67f9f4`变成了dangling images，在镜像列表中展示为`<none>:<none>`

## 清理dangling images
1. 执行命令`docker image prune`即可删除`dangling images`
