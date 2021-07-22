## Registry 介绍
Docker Registry分为共有仓库和私有仓库，私有仓库只有有权限才能访问私有仓库。
对于官方镜像，没有仓库镜像加速，存步难行。设置`Registry Mirrors`,当拉取`image`时，Docker Daemon会先去`Registry Mirrors`拉取镜像，没有找到的话，`Registry Mirrors`会去找官方的`Registry`拉取镜像，再返回本地。


## 镜像加速配置
`daemon.json`配置docker的启动参数，可以通过`docker info`命令查看。
在`~/.docker/daemon.json`配置如下：
```
{
    "insecure-registries":["192.168.9.8:80"], #这个私库的服务地址
    "registry-mirrors": ["https://ower.site.com"] #镜像加速的地址，增加后在 docker info中可查看。
}
```

#### 阿里云镜像地址
可以使用阿里云加速地址：`https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors`。
