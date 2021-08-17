## QLExpress
本质上是一个脚本引擎，提供额外自定义关键字能力，使得它的表达能力大大增强，甚至能够使用中文描述业务逻辑
- 简洁而强大
和java语法接近，而且可以通过自定义操作符号和别名来实现强大的规则判断。
- 轻量
本体只需要引入一个依赖包

## QlExpress vs groovy
qlExpress本身只是一个脚本语言，可以被封装成规则引擎，而drools本身是个规则引擎，专注于规则的条件匹配和执行，不具备可比性。

qlExpress和groovy同属脚本语言，比较如下：
1、groovy比qlExpress更兼容java语法
相对功能复杂处理过程的大段java脚本，groovy可以直接拷贝过来运行，
而qlexpress的语法很轻量，对原始的java代码有一定的兼容性问题，一般需要把数据的类型声明全部去掉，同时不支持异常处理、for循环的集合写法 等

2、qlExpress比groovy更强调功能扩展
因为qlExpress就是诞生于阿里的电商系统，定制了很多特别的常用功能需求（宏定义，语法解析，公式计算，布尔逻辑处理，操作符函数的内置替换），可读性和功能更贴合业务需要，详细看qlExpress的扩展能力部分

3、qlExpress和groovy性能相当
qlExpress和groovy同属弱类型语法，比如a+b，可以在运行时支持字符串，数字等多种计算模式，相比fel，simpleExpress 等强类型语言性能会差一个数量级。
他们都支持编译期做了缓存功能，qlExpress转化为InstructSet，groovy转化为一个特殊的groovyclass子类

## 资料
来自alibaba的流程引擎https://github.com/alibaba/QLExpress