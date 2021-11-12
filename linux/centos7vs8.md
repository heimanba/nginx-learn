## 区别
centos7 已经在`2020-08-06`停止更新维护
centos8 刚刚发布(2019-09-24)
#### python
centos7默认是 python2.7
centos8 默认是 python3.6

#### 查看当前版本
```
cat /etc/redhat-release
```
#### yum源
- 使用工具 `yum-config-manager`
```
yum-config-manager --add-repo repository_url
```
- 查看文件
具体文件存储在 `/etc/yum.repos.d/`下面，以`repo`为后缀
