## 前言
使用nodes通过云效cicd平台部署到阿里云ack上

#### docker镜像制作

- 通过[express生成器](https://www.expressjs.com.cn/starter/generator.html)生成express代码
- dockerfile制作
```
FROM registry.cn-beijing.aliyuncs.com/hub-mirrors/node:10

COPY . .
RUN npm install -g cnpm --registry=https://registry.npm.taobao.org && cnpm install

CMD ["node", "./bin/www"]
```

#### 构建流水线
1. 阿里云镜像构建
可以直接通过[容器镜像服务](https://help.aliyun.com/product/60716.html)构建镜像。
![](https://img.alicdn.com/imgextra/i4/O1CN01AFdXsu1Pss2gRqJss_!!6000000001897-2-tps-2488-1284.png)
2. 部署到ACK
我们的k8s配置如下:
- 应用的`Deployment`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: node-expressjs-sample
  name: node-expressjs-sample
spec:
  replicas: 2
  selector:
    matchLabels:
      run: node-expressjs-sample
  template:
    metadata:
      labels:
        run: node-expressjs-sample
    spec:
      containers:
      - image: ${IMAGE}
        name: app
```
- service
```
apiVersion: v1
kind: Service
metadata:
  name: node-expressjs-sample
spec:
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 3000
  selector:
    run: node-expressjs-sample
  sessionAffinity: None
  type: ClusterIP
```
- ingress
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: node-expressjs-sample
spec:
  rules:
    - host: express.c1be8ea9689364d08be53979a7913b422.cn-wulanchabu.alicontainer.com
      http:
        paths:
          - backend:
              serviceName: node-expressjs-sample
              servicePort: 80
            path: /
```
![](https://img.alicdn.com/imgextra/i4/O1CN01V8YQaG1grMCTByBHt_!!6000000004195-2-tps-2002-1288.png)

3. 通过域名访问
```
curl express.c1be8ea9689364d08be53979a7913b422.cn-wulanchabu.alicontainer.com
```