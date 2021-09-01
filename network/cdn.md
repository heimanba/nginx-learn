## 介绍
CDN技术最大的用处，就是加速了网站的访问-用户内容之间的物理距离缩短，用户的等待时间得以缩短

#### 原理
![cdn](https://img.alicdn.com/imgextra/i4/O1CN01MpfHRA1tecBJpY6Ck_!!6000000005927-2-tps-1602-1048.png)

#### 安全控制
CDN是对外公网开放，如果有人恶意盗刷，比如通过脚本下载大的图片或者文件，会造成很大的损失。
1. Referer防盗链
通过配置`Referer`黑名单以及是否允许空`Referer`,限制资源的访问来源，防止网站资源被非法盗用
2. URL鉴权
通过Referer黑白名单的方式，可以解决一部分盗链的问题，因为`Referer`内容可以伪造，所以无法彻底保证站点的资源。使用URL鉴权方式更为安全有效，参考[阿里云文档](https://help.aliyun.com/document_detail/85113.htm)

> 缺点：如果盗链者不及时更新URL，就无法访问。但是如果盗链者定期来更新URL，这种方式也会失效

#### 应用场景
1. 网站站点、应用加速
![](https://pic3.zhimg.com/80/v2-69de8db0b2538c59753bc6d39d66a4fd_1440w.jpg?source=1940ef5c)