## 架构
![架构](https://img.draveness.me/2018-11-25-kubernetes-architecture.png)
每个Kubernates集群由一组Master节点和一系列的Worker节点组成，其中Master节点主要负责存储集群状态并为k8s对象分配和调度资源。

#### Master
集群状态管理的`Master`节点,主要负责接收客户端的请求，安排容器执行并控制循环，将集群的状态想目标进行迁移，`Master`节点内部由三个组件构成
![Master](https://img.draveness.me/2018-11-25-kubernetes-master-node.png)
- `etcd` 集群状态的集中存储
- `Api Server` 负责处理来自客户的请求，主要作用就是对外提供`RESTful`的接口，包括用于查看集群状态的读取,也是唯一和`etcd`集群通信的组件。（集群接口和通信总线）
- `Controller Manager`管理器运行了一系列的控制器进程，这些进程会按照用户期望状态在后台不断调节集群中国的对象，当服务状态发生了改变，控制器就会发现这个改变并开始向目标状态迁移。（协调发布状态最终一致性组件）
- `Scheduler`调度器，其实为Kubernetes中国运行的Pod选择部署的worker节点，会根据用户的需要选择最能满足请求的节点来运行`Pod`,它会在每次需要调度`Pod`时执行。（调度决策组件）

#### worker
`Worker`主要由`kubelet`和`kube-proxy`两部分组成
![](https://img.draveness.me/2018-11-25-kubernetes-worker-node.png)
`kubelet`是一个节点上的主要服务，它周期性的从`API Server`接受新的或者修改的Pod规范并且保证节点上的`Pod`和其中的容器正常运行，还会保证节点会向目标状态迁移，改节点仍然会向`Master`节点发生宿主机的健康状况

`kube-proxy`负责宿主机的子网管理，同时也能将服务暴露给外部，其原理就是在多个隔离的网络中把请求转发给正确的Pod或者容器。（实现Service抽象的组件，屏蔽pod Ip 变化以及负载均衡）

![](https://img.alicdn.com/imgextra/i4/O1CN01HZRWgt1l27yXYZU4Y_!!6000000004760-2-tps-1416-800.png)
## pod
Pod 是k8s中最小的调度单元，pod里面住的是container(docker)容器应用。
Pod是容器的进一层抽象，可以兼容多容器场景，比如sideCar。
#### yml配置
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    name: nginx-pod
spec:
  containers:  # 镜像配置
  - name: nginx-pod
    image: nginx:latest
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
```
- 执行`kubectl apply -f nginx-pod.yml`
- 如果需要查看pod的详细信息，可以执行`kubectl describe pod nginx-pod`
- 本机测试`kubectl port-forward nginx-pod 8090:80`,使用端口转发将nginx 80端口绑定到本地的`8090`端口。

## service
通过`selector`和`labels`进行路由分发匹配 
![](https://img.alicdn.com/imgextra/i4/O1CN01L6ZPBq1YS9TXVw2BP_!!6000000003057-2-tps-1410-784.png)
#### 服务类型
- NodePort
通过每个节点的API和静态端口(NodePort)暴露服务，请求通过`<节点IP>:<节点端口>`，可以从集群的外部访问一个`NodePort`服务
- LoadBalancer
适用于使用外部负载均衡器或者云厂商提供的服务
- ClusterIP
通过集群的内部IP暴露服务，选择该值时服务职能在集群内部访问，这也是默认的服务(Service)类型

#### 示例
- nginx.pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod
spec:
  containers:
  - name: nginx-pod
    image: nginx:latest
    ports:
      - containerPort: 80
```
- nginx-service-pod.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-pod
spec:
  selector:
    app: nginx-pod
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 31080
```
验证: 
1. 在本地验证 `curl 127.0.0.1:31080`
2. 如果跑在阿里云ACK集群，应该找到对应的宿主机，执行`curl 127.0.0.1:31080`

#### ports
- port
service的端口，主要作用是集群内其他pod访问本pod时候，需要一个`port`，如`nginx`的pod访问`mysql`的pod,那么mysql的定义如下:
```
apiVersion: v1
 kind: Service
 metadata:
  name: mysql-service
 spec:
  ports:
  - port: 33306 # nginx访问的service端口
    targetPort: 3306
  selector: 
    name: mysql-pod
```
- targetport
targetport是pod暴露出来的port端口,当nginx请求到`33306`时候，service根据这个请求的`selector`,将请求转发到`mysql`。也就是这个pod的`3306`上。

- nodeport
集群内部访问的是`port`,集群外部的可以访问就是nodeport