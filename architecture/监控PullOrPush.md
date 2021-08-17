## 总览
市面上监控系统不下上百种，按照不同的类别也有很多的划分方式。例如:
1. 监控对象：
- 通用型： 通用的监控方式，适用大部分监控对象。比如cpu,load等。典型产品如:阿里云的云监控
- 专一型：为某一功能定制，比如Java的JMX系统，NodeJS监控系统等。典型产品如:阿里云arms应用监控
2. 获取数据方式：
- Push: CollectD, Zabbix, influxDB
- Pull: Prometheus, SNMP, JMX
3. 部署方式：
- 单机： 单机实例部署
- 分布式：可以横向扩展
- Saas化：很多商业公司提供SaaS的方式，无需部署
4. 获取数据的方式： 
- 接口：通过API获取
- DSL：有一些计算，比如 PromQL, GraphQL,SQL(标准SQL，类SQL)
5. 商业属性
- 开源免费： Prometheus、InfluxDB单机版
- 开源商业型：InfluxDB集群版、Elastic Search X-Pack
- 闭源商业型：DataDog、ARMS、Splunk、AWS Cloud Watch
## Pull Or Push对比
基于Pull类型的监控系统，就是由监控系统主动去获取指标，需要被监控的对象能够具备被远端访问的能力。
基于Push类型的监控系统不会主动获取数据，而监控的对象主动推送指标
#### 原理和部署
- 配置
    - pull: 云原生中心化配置
    - push: 端上配置，通过配置中心支持中心化
- 监控对象的发现
    - pull: 依赖服务发现的机制，比如Zookeeper,Etcd,Consul等注册中心
    - push: 由Agent自主上报，无需服务发现模块
- 部署方式
    - pull: 
    1. 应用暴露端口，接入服务发现，原生支持pull协议，
    2. 其他系统如：ecs,mysql等中间件依赖适配器去抓取指标再提供Pull端口
    - push: 
    1. Agent统一代理，抓取主机，MySQL等中间件数据推送到监控系统，Agent也作为转发器接收应用推送
    2. 应用主动推送到监控系统
#### 扩展性
    - pull
    依赖Pull端扩展，需要Pull Agent和存储解耦(原生的Prometheus不支持)；Push Agent按照分片划分
    - push
    简单，本身Agent可横向扩展
Push 天然是分布式的，子监控后端能力可以跟上的时候，可以无限横向扩展。相比下Pull扩展比较麻烦
1. pull模块和监控后端解耦，Pull作为Agent单独部署
2. Pull Agent需要做分布式协同。
#### 机器，代价
- 资源消耗
    - pull
    1. 应用暴露端口的方式，资源消耗较低
    2. `Exporter`方式资源消耗较高
    - push
    1. 应用推送方式资源消耗低
    2. Agent方式资源消耗较低
- 安全性
    - pull
    工作量大，需要保证应用暴露端口的安全性和Exporter端口安全性，比较容易受到DDOS攻击出现数据泄漏
    - push
    低，Agent与服务端一般都进行带有加密、鉴权的数据传输
- 运维消耗
    - pull
    1. 服务端稳定性
    2. 服务发现稳定性
    3. Exporter稳定性与扩容
    - push
    1. Push Agent 稳定性
    2. 服务端稳定性
    3. 配置中心稳定性与扩容
> 从部署复杂性看，Pull模型部署过于复杂，维护代价高，Push模型较为便捷，应用提供Metrics端口或主动Push部署代价相差不大


## Pull or Push如何选型
1. Pull模式的代表Prometheus的家族方案（之所以称之为家族，主要是默认单点的Prometheus扩展性受限，社区有非常多Prometheus的分布式方案，比如Thanos、VictoriaMetrics、Cortex等）
2. Push模式的代表InfluxDB的TICK（Telegraf, InfluxDB, Chronograf, Kapacitor）方案。这两种方案都有各自的优缺点，在云原生的大背景下，随着Prometheus在CNCF、Kubernetes带领下的大火，很多开源软件都开始提供Prometheus模式的Pull端口；

如果有很多短声明周期应用，需要使用Push方式，比如移动端只能使用Push方式
1. 主机，进程，中间件监控使用Push Agent
2. K8s等直接暴露Pull端口使用Pull模式
3. 应用更具实际场景选择Pull或者Push