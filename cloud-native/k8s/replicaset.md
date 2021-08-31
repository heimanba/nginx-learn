## 作用
pod如果销毁，无法自动拉起，我们需要实现高可用，`ReplicaSet`可以帮助我们实现高可用，实现保持固定的节点数

#### 实例
- 定义一个replicaset
petclinic-replicaset.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: petclinic
spec:
  replicas: 3
  selector:
    matchLabels:
      app: petclinic
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
- 定义一个可以访问的service
petclinic-service.yaml
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