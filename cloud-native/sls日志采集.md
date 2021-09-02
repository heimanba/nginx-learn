## 日志采集分类
- 日志文件类型
- 标准输出流`stdout`类型
![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/2330559951/p2950.png)
#### 通过daemonSet方式采集标准输出(stdout)
1. 首先安装`logtail-ds`
在阿里云ack集群，`运维管理`>`组件管理`>`日志与监控`
2. 在负载配置中日志配置
![](https://img.alicdn.com/imgextra/i3/O1CN01ujHV5Z1NN9NVWWbfX_!!6000000001557-2-tps-1500-602.png)
3. 收集的日志 默认在content字段中，支持统一的处理配置


## 处理数据
### 采集文本日志
1. [极简模式](https://help.aliyun.com/document_detail/137903.html)
- 单行文本日志
默认一行日志内容为一条日志，在日志文件中，以换行符分隔两条日志。该模式下，只需指定文件目录和文件名称即可，Logtail会按照每行一条日志进行采集
- 多行文本日志
默认一条日志有多行内容，改模式下除了指定文件目录和文件名称外，还需配置日志样例和行首正则表达式，Logtail通过行首正则表达式匹配一条日志的行首，未匹配部分为该条日志一部分。

需要设置行首正则表达式:可以通过日志样例进行自动识别，如果是`单行文本日志`，无需配置此参数。

2. 使用[完整正则模式](https://help.aliyun.com/document_detail/137902.html)采集
3. [Nginx模式](https://help.aliyun.com/document_detail/28988.html)采集日志
配置`log_format`
```
log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
                 '$request_time $request_length '
                 '$status $body_bytes_sent "$http_referer" '
                 '"$http_user_agent"';
```
4. [分隔符模式](https://help.aliyun.com/document_detail/31724.html)采集日志
比如日志样例为`127.0.0.1|2020 09:44|404|36500`
则日志的分隔符为`|`
5. 使用[JSON模式](https://help.aliyun.com/document_detail/31720.html)采集日志


### Logtail插件处理日志

当日志太复杂或者不固定，使用固定的解析模式无法满足日志解析需求时候，可以使用Logtail插件解析日志。具体使用参考[文档](https://help.aliyun.com/document_detail/64957.html)

#### 使用限制:
- 性能限制
Logtail插件会消耗更多的`CPU资源`,需要更具实际情况调整Logtail参数配置。
当原始数据量生成速度超过`5MB/s`时候，不建议使用过于复杂插件组合处理数据。可以使用Logtail插件简单处理后，在通过[数据加工](https://help.aliyun.com/document_detail/125384.htm)完成进一步处理

- 文本日志限制
部分高级功能失效，包括过滤器配置，上传原始日志，机器时区，丢弃解析失败日志等。但这些能力可以通过相关插件实现

#### 配置说明
```
"processors" : [
    {
        "type": "processor_split_char",
        "detail": {
            "SourceKey": "content",
            "SplitSep": "|",
            "SplitKeys": [
                "method",
                "type",
                "ip",
                "time",
                "req_id",
                "size",
                "detail"
            ]
        }
    }
]
```

## 数据加工(能力和kafka接近)
### 解决问题
- 数据规整(一对一)
源logstore数据采集(不做加工，为了快速采集)-> 数据加工服务-> 目标logstore
![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/3924341061/p51410.png)
- 数据分派(一对多)
源logstore -> 数据加工服务 -> 不同的logstore
比如:nginx日志，包含error的需要在一个logstore做告警，响应慢的需要存放到另外一个logstore去做分析
![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/3924341061/p51411.png)
- 多源汇集(多对一)
多源logstore -> 数据加工服务 -> 目标logstore
![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5943749951/p51412.png)