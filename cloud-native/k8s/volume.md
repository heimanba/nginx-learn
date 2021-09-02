## 存储抽象
![](https://img.alicdn.com/imgextra/i1/O1CN019ZIsiW1nAwpvy4IrV_!!6000000005050-2-tps-2140-1230.png)

#### 示例
- 设置hostPath的volume卷
可以将主机的文件节点系统上的文件或者目录挂载到你的Pod中
- volumeMounts
挂载到容器内的文件系统的路径
```
spec:
    containers:
    - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
            value: petclinic
        - name: MYSQL_DATABASE
            value: petclinic
        volumeMounts:
        - name: mysql-persistent-volume
            mountPath: /var/lib/mysql
    volumes:
    - name: mysql-persistent-volume
        hostPath:
        path: /tmp/data01
        type: DirectoryOrCreate
```
宿主机测试为:
![](https://img.alicdn.com/imgextra/i2/O1CN01FhSFfq1malXz6Yck0_!!6000000004971-2-tps-1752-908.png)

#### PV和PVC
![](https://img.alicdn.com/imgextra/i3/O1CN01aES2Pr1gpz0jEgb9W_!!6000000004192-2-tps-1942-996.png)
1. 描述
- PV: 持久化存储卷，主要定义是一个持久化存储在宿主机的目录，比如NFS挂载目录
- PVC: 描述Pod所希望使用的持久化存储的属性，比如`Volume`存储大小，可读写权限等。
2. 示例
PVC和PV进行关联, 通过 storageClassName进行关联
- mysql pvc
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  storageClassName: standard
  resources:
    requests:
      storage: 250Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce

```
- mysql pv
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 250Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: standard
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```
容器中的volume和pvc使用`name`名称进行关联
```
containers:
    - name: mysql
        image: mysql:5.7
        volumeMounts:
        - name: mysql-persistent-volume
          mountPath: /var/lib/mysql
    volumes:
    - name: mysql-persistent-volume
      PersistentVolumeClaim:
        ClaimName: mysql-pvc
```