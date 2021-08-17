## 规则引擎
一个规则通常是指定义了条件和操作的一个实体。当满足了`一些条件`触发`一系列的动作`
一个典型的规则引擎平台的设计
![images](https://img.alicdn.com/imgextra/i1/O1CN01gChcYB1LsdMrSssYU_!!6000000001355-2-tps-1312-1020.png)

#### 规则
灵活强大的规则语法，具备高扩展性，和通用性。使用脚本语言作为底层

#### 持久化
- OSS持久化
OSS存放实际的不同版本的规则文件

- mysql持久化
mysql存放历史所有的版本信息

- 配置推送
配置推送平台将最新的版本推送给客户端使用

#### 业务平台
- rpc/http
监听配置推送平台`push`的最新版本，通过rpc或者http方式，`pull`最新的配置信息

## 规则引擎对比
1. 编译型 VS 解释型
- 编译型
能够产生独立的class文件，如:`groovy`
- 解释型
通过编译成自定义的内存指令，如:`QLExpress`
2. java语法 VS 表达式语法 VS 脚本
- java语法
语法和Java保持一致，不做任何扩展
- 表达式语法(EL expression language)
语法大量简化，只支持简单的数学公式，对象方法成员调用等
- script
介于上面两者之间，既提供很好的语法糖，又支持大部分java语法(for循环,if判断，函数定义等)，groovy，QLExpress

## QlExpress vs groovy

1、groovy比qlExpress更兼容java语法

相对功能复杂处理过程的大段java脚本，groovy可以直接拷贝过来运行，
而qlexpress的语法很轻量，对原始的java代码有一定的兼容性问题，一般需要把数据的类型声明全部去掉，同时不支持异常处理、for循环的集合写法 等

2、qlExpress比groovy更强调功能扩展

因为qlExpress就是诞生于阿里的电商系统，定制了很多特别的常用功能需求（宏定义，语法解析，公式计算，布尔逻辑处理，操作符函数的内置替换），可读性和功能更贴合业务需要，详细看qlExpress的扩展能力部分

3、qlExpress比groovy实现更简洁，操作系统以及应用领域更广

目前我们的qlExpress因为所有的语法解析、运算执行都自己编码实现，依赖实现极其简单，在阿里的新零售领域，在大量的低版本android系统的pos机上都有大规模使用，稳定性非常好。


4、qlExpress和groovy性能相当

qlExpress和groovy同属弱类型语法，比如a+b，可以在运行时支持字符串，数字等多种计算模式，相比fel，simpleExpress 等强类型语言性能会差一个数量级。
他们都支持编译期做了缓存功能，qlExpress转化为InstructSet，groovy转化为一个特殊的groovyclass子类