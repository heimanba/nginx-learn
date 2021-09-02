## 原理和作用
![](https://img.alicdn.com/imgextra/i4/O1CN01TgZ2KC1qHlZs3Mr4Y_!!6000000005471-2-tps-1306-712.png)


## 例子
- 构建secrect.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: petclinic-secret
type: Opaque
# data:
#   password: <Password>
stringData:
  DATASOURCE_USERNAME: root
  DATASOURCE_PASSWORD: petclinic
```
- 引用
```
envFrom:
    - secretRef:
        name: petclinic-secret
```

获取secret详细值`kubectl get secret petclinic-secret -o yaml`