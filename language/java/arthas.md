## 入门
### 1. 安装
```
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar
```
### 2. 工具
可以通过下在ideal插件，方便使用arthas指令
```
https://github.com/WangJi92/arthas-plugin-demo?spm=ata.21736010.0.0.7269368aIdDOaN
```
#### 全局命令说明
- `-x`是展示结果属性遍历深度，默认为1
- `-n`是执行的次数，q 退出
- `-c` classloader 的hash值
- 退出q，关闭stop
### 3. 最常用的trace,watch等功能
#### trace
> 作用：方法内部调用路径，并输出方法路径上的每个节点耗时
```
trace com.aliware.middleware.MainController root  -n 5 --skipJDKMethod false
```
- trace多个类或者函数
trace命令只会trce匹配到函数中的子函数，不会向下trace多层，因为trace的代价比较昂贵，多层trace可能导致最终的trace类或者函数非多
```
trace -E demo.MathGame primeFactors|run -n 5  --skipJDKMethod false '1==1'
```


#### watch
> 观察值的信息，可以查看入参，返回值，异常。可执行表达式获取静态变量等。
说明：
- watch定义了4个观察点事件: `-b`方法调用前，`-e`方法异常后，`-s`方法返回后，`-f`方法结束后
- 4个观察点事件`-b`，`-e`，`-s`默认关闭，`-f`默认打开，当指定观察点被打开后，相应的事件点会对观察表达式求值并输出
- 当使用 `-b` 时，由于观察事件点是在方法调用前，此时返回值或异常均不存在

1. 观察方法出参和返回值
```
watch demo.MathGame primeFactors "{params,returnObj}" -x 2
```
2. 观察当前对象的属性
如果想查看方法运行前后，当前对象的属性，可以使用`target`关键字，代表当前的对象
```
watch demo.MathGame primeFactors 'target'
```
或这使用`target.field_name`访问当前对象的某个属性
```
watch demo.MathGame primeFactors 'target.illegalArgumentCount'
```
### arthas表达式核心变量
```
public class Advice {
    private final ClassLoader loader;
    private final Class<?> clazz;
    private final ArthasMethod method;
    private final Object target;
    private final Object[] params;
    private final Object returnObj;
    private final Throwable throwExp;
    private final boolean isBefore;
    private final boolean isThrow;
    private final boolean isReturn;
}
```

#### ognl
ognl获取静态变量
1. 获取classLoader的hash值
```
sc -d demo.MathGame // 返回 7a81197d
```
2. 通过ognl语法获取静态变量值
```
ognl -x 3 '@demo.MathGame@random' -c 7a81197d
```