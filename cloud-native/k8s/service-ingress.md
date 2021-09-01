## service
Pod 运行起来后，面临着两个问题：
- Pod无法从集群外部直接访问
- Pod出现故障自愈合，IP会发生变化
#### 传统负载均衡
既有外部的反向路由，又有内部的负载均衡代理
![](https://img.alicdn.com/imgextra/i1/O1CN01UPnYr61QXdHyaU9nz_!!6000000001986-2-tps-1238-700.png)

#### service几种类型
- clusterIp
集群的内部 IP 暴露服务，选择该值时候只能在集群内部访问，也是默认的`ServiceType`
- NodePort
通过每个节点的IP和静态端口暴露服务，通过请求`<节点 IP>:<节点端口>`可以从集群外部访问一个`NodePort`服务

#### 实例
- 拉起一个mysql`pod`
```
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  containers:
    - name: mysql
      image: mysql:5.7
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: petclinic
        - name: MYSQL_DATABASE
          value: petclinic
```
- 暴露一个mysql的`service`
```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
    - name: tcp
      port: 3306
      targetPort: 3306
  type: ClusterIP
```
- 拉起应用的`Deployment`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: petclinic
spec:
  selector:
    matchLabels:
      app: petclinic
  replicas: 1
  template:
    metadata:
      labels:
        app: petclinic
    spec:
      containers:
        - name: petclinic
          image: spring2go/spring-petclinic:1.0.1.RELEASE
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: mysql
            - name: DATASOURCE_URL
              value: jdbc:mysql://mysql/petclinic
            - name: DATASOURCE_USERNAME
              value: root
            - name: DATASOURCE_PASSWORD
              value: petclinic
            - name: DATASOURCE_INIT_MODE
              value: always
```
- 应用的`service`
```
apiVersion: v1
kind: Service
metadata:
  name: petclinic
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 8080
      nodePort: 31080
  selector:
    app: petclinic
  type: NodePort
```
> 问题: 通过`jdbc:mysql://mysql/petclinic`可以访问到mysql的服务，这里的`mysql`是`service`的名称。如何找到这个服务呢？


#### service的dns解析
1. 查看mysql的`service`的clusterIp
```
    NAME        TYPE         CLUSTER-IP
service/mysql ClusterIP   172.16.163.163
```
2. k8s集群中`namespace`
- 默认的namespace`default`

用户创建的资源如果不手动指定,默认都会创建在这个`namespace`下。
- kube-system
k8s`系统创建对象`所使用的命名空间
- kube-public
所有用户（包括未经过身份验证的用户）都可以读取它。 这个名字空间主要用于集群使用，以防某些资源在整个集群中应该是可见和可读的

3. kube-system
通过`kubectl get all -n kube-system`查看`kube-dns`如下
```
       NAME          TYPE      CLUSTER-IP
service/kube-dns  ClusterIP   172.16.0.10 
```
4. 进入容器查看
- 进入容器 `kubectl exec -it petclinic-cb946d644-l9rkq sh`
- `resolv.conf`
它是DNS客户机配置文件，用户设置的DNS服务器的IP地址及DNS域名，包含
```
nameserver // 定义DNS服务器的IP地址 
domain // 定义本地域名 
search // 定义域名的搜索列表 
sortlist // 对返回的域名进行排序
```
我们查看下容器中的配置:
```
# cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 172.16.0.10
```
- 查看DNS记录
使用`nslookup`命令，查看域名解析是否正常，在网络故障的时候，在网络故障的时候来诊断网络的问题。
```
# nslookup mysql
Name:      mysql
Address 1: 172.16.163.163 mysql.default.svc.cluster.local
```
可以看到查到的`172.16.163.163`正是`mysql`的`service`的clusterIp。