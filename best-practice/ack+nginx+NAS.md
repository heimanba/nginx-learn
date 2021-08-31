## 基本架构图
![架构图](https://ucc.alicdn.com/pic/developer-ecology/46cba27a5cb947a2a6a16808900b3abf.jpg)

#### 基本步骤
1. 创建NAS资源
选择对应的VPC/VSwitch,同时创建对应的挂载点
2. 上传文件到NAS
- 通过一台ECS服务器来做搭建`SFTP`服务，将文件上传到NAS。
- 在[NAS控制台](https://help.aliyun.com/document_detail/285696.html)挂载文件系统，或者通过[命令行挂载](https://help.aliyun.com/document_detail/268208.html)到ECS
3. 创建ACK集群

