## 滚动发布
按批次依次替换老版本，逐步升级到新版本。发布过程中，应用不中断，用户体验平滑
#### 蓝绿发布 VS滚动发布
- 蓝绿发布需要两倍机器资源，滚动发布无需额外机器资源
- 蓝绿发布支持版本不兼容升级，滚动发布仅支持兼容升级(新老版本会并存一段时间)

## Deployment
Deployment 是基于ReplicaSet的包装
![](https://img.alicdn.com/imgextra/i1/O1CN01paqO2D1Wsb78z3Qs3_!!6000000002844-2-tps-2014-1010.png)

#### 示例
- deployment-pod.yaml
其中`selector`中的`matchLabels`和`labels`中要一致。表示这个`Deployment`管理的`pod`资源
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: petclinic
spec:
  selector:
    matchLabels:
      app: petclinic
  minReadySeconds: 10
  replicas: 3
  template:
    metadata:
      labels:
        app: petclinic
    spec:
      containers:
      - name: petclinic
        image: spring2go/spring-petclinic:1.0.0.RELEASE
        ports:
        - containerPort: 8080
```
- petclinic-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: petclinic
spec:
  selector:
    app: petclinic
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    name: http
    nodePort: 31080
```