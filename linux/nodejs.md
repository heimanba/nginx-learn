## EPEL源
EPEL 是`Fedora`的一个项目，适用于CentOS等操作系统。
我们在CentOS下使用yum安装时候找不到rpm包的情况，官方的rpm repository提供的rpm包也不够丰富。这时候需要自己编译非常痛苦。`EPEL` 恰恰可以解决这两方面问题。安装了`EPEL`后，相当于添加了一个三方源。

#### 安装EPEL
1. 先确认系统是否安装过`epel-release`包
```
yum info epel-release
```
2. 如果有相关安装信息，则说明已经安装，如果提示没有安装则执行安装命令
```
yum install epel-release
```

## 安装nodejs
一般会默认安装`6.x`版本的,npm作为依赖一起安装
```
sudo yum install nodejs
```
安装完成后，`node -v`输出下面版本信息说明安装成功
```
v6.13.3
```

## 升级nodejs
1. 安装`n`,`n`是nodejs管理工具
```
npm install -g n
```
2. 安装nodejs最新版本
```
n latest
# 安装指定版本: n 10.1.13
```
