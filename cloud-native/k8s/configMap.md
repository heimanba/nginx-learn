## 作用
![](https://img.alicdn.com/imgextra/i1/O1CN01vDNgZm29gw2qW3SCX_!!6000000008098-2-tps-1994-1140.png)

#### 例子
- 创建configMap.yml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: petclinic-config
data:
  SPRING_PROFILES_ACTIVE: mysql
  DATASOURCE_URL: jdbc:mysql://mysql/petclinic
  DATASOURCE_USERNAME: root
  DATASOURCE_PASSWORD: petclinic
  DATASOURCE_INIT_MODE: always
  TEST_CONFIG: test_config_v1
```
- 更改之前的 env配置为
```
envFrom:
    - configMapRef:
        name: petclinic-config
```
- 验证
使用`kubectl exec petclinic-f77f5c8f-fw8j7 printenv`可以打印出环境变量

#### configMap更改生效
需要重启Pod,并且更新`ConfigMap`的`name和引用`。

比如生成一个新的configMap比如命名为`petclinic-config-v2`,在`Deployment`上引用这个新的`configMap`就能被更新使用。

#### 绑定持久卷Volume
这样就支持配置热更新