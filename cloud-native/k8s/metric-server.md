## 简介
Metrics Server 是k8s集群核心监控数据的聚合器，从Kubeley收集资源指标，并通过Metrics API在k8s ApiServer中提供给缩放资源对象HPA使用。可以基于k8s集群的CPU，内存水平自动缩放。

#### 使用
1. 获取当前node的监控 `kubectl top nodes`
```
NAME                          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
cn-zhangjiakou.192.168.0.87   131m         3%     3315Mi          51%
cn-zhangjiakou.192.168.0.88   114m         2%     2308Mi          35%
```
2. 获取当前pod的监控 `kubectl top pods`
```
NAME                    CPU(cores)   MEMORY(bytes)
nginx-fd584594c-fx6tj   0m           3Mi
nginx-fd584594c-rgv8k   0m           3Mi
```

#### java/Spring在K8s内存配置
![](https://img.alicdn.com/imgextra/i4/O1CN01gLHs2z1cTlKeokD9U_!!6000000003602-2-tps-2228-1200.png)
1. Java内存结构
- Heap + Non Heap + 其他
- 从初始分配开始，随实际使用增加向OS申请，直到允许的最大请求
2. 查看容器的JVM参数
```
java -XX:+PrintFlagsFinal -version|grep -E"UseContainerSupport|InitialRAMPercentage|MaxRAMPercentage|MinRAMPercentage"
```
3. 从配置中传递参数
```
 containers:
    - name: petclinic
        image: spring2go/spring-petclinic:1.0.0.RELEASE
        resources:
            requests:
                memory: "20Mi"
            limits:
                memory: "20Mi"
            env:
                - name: JAVA_OPTS
                  value: "-XX:MaxRAMPercentage=80.0"
                - name: SERVER_PORT
                  value: "8080"
```