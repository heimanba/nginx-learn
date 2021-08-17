#### maven helper
如何查看自己项目中哪些依赖的jar包冲突？
![images](https://img.alicdn.com/imgextra/i1/O1CN01tQOFZV1WjtaSBoPqT_!!6000000002825-2-tps-2236-1208.png)
![images](https://img.alicdn.com/imgextra/i2/O1CN01eHhUtJ1annQISO0R9_!!6000000003375-2-tps-2224-1160.png)

#### Alibaba Java Coding Guidelines
打开alibaba规约检查
![images](https://img.alicdn.com/imgextra/i2/O1CN01rm2kUh27eWjB5L0br_!!6000000007822-2-tps-746-1030.png)

如果返回: `No suspicious code found`代表你的代码写的非常棒

否则不符合规约的代码`Blocker/Critical/Major` 三个等级显示在下方，会下ideal的面板中出现
![](https://img.alicdn.com/imgextra/i1/O1CN01oKhIJv1WXzHny7bty_!!6000000002799-2-tps-2932-406.png)

#### arthas idea
Arthas idea命令帮助工具
![arthas](https://img.alicdn.com/imgextra/i4/O1CN016aEj3i1xwiIKgoLAJ_!!6000000006508-2-tps-1476-1302.png)

#### HTTP Client
IDEA自带的HTTP工具
在Tools->HTTP Client->Test RESTful Web Service
![http client](https://img.alicdn.com/imgextra/i1/O1CN01cIkxln1gu6ZVGjuKF_!!6000000004201-2-tps-512-1036.png)

- 普通`GET`请求
```
GET http://localhost:80/api/item?id=99
Accept: application/json
```
- 带JSON的`POST`请求
```
POST http://localhost:80/api/item
Content-Type: application/json

{
  "id": 99,
  "value":"content"
}
```
- form表单提交形式
```
POST https://httpbin.org/post
Content-Type: application/x-www-form-urlencoded

id=999&value=content
```

#### RestfulTool
一套Restfull服务开发辅助工具集，由于原来的`RESTFulToolkit`在IDEA.201以上版本不再适配。

![](https://img.alicdn.com/imgextra/i2/O1CN01Vlgjpl1y4V1kl3OsF_!!6000000006525-2-tps-856-1030.png)
