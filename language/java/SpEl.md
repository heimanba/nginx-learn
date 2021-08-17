## 概述
SpELl是Spring表达式语言，默认格式为`#{expression}`。


1. 基本使用
```
String expressionStr = "1 + 2";
ExpressionParser parpser = new SpelExpressionParser();
Expression exp = parpser.parseExpression(expressionStr);

// 方式一：直接计算
Object value = exp.getValue();
System.out.println(value);

// 方式二：定义环境变量，在环境内计算拿值
EvaluationContext context = new StandardEvaluationContext();
System.out.println(exp.getValue(context));
```
2. EvaluationContext
在表达式计算中和`EvaluationContext`这个上下文关系莫大，用来解析环境中执行`parseExpression`的解析操作。主要有两个实现
- SimpleEvaluationContext
仅支持部分的SpEl，有意限制表达式类别，不包括java类型引用，构造函数以及bean引用等，并且明确选择对象表达式属性和方法的支持级别
- StandardEvaluationContext
公开支持`所有`SpEL语言功能和配置选项。您可以使用它来指定默认的根对象并配置每个可用的评估相关策略


3. 语法
- 直接量表达式
```
@Value("#{'Hello World'}")
```
- 直接使用java代码
直接使用类名时，此类必须是`java.lang`包中的类，才可以在SpEll中省略包名，否则需要全名
```
Expression exp = parser.parseExpression("new Spring('Hello World')");
```
- 使用T(Type)
使用“T(Type)”来表示java.lang.Class类的实例，即如同java代码中直接写类名。同样，只有java.lang 下的类才可以省略包名
`此方法一般用来引用常量或静态方法`
```
parser.parseExpression("T(Integer).MAX_VALUE"); //等同于java代码中的：Integer.MAX_VALUE
```
- Elvis运算符
是三目运算符的特殊写法，可以避免null报错的情况
```
name?:"other" //等同于java代码 name != null? name : "other"
```
- 运算符表达式
    - 算术表达式 (1+2-3*4/2)
    - 比较表达式 1>2
    - 逻辑表达式 2>1 and (!true or !false)
    - 赋值表达式（#variableName=value）
    - 三目表达式（表达式1?表达式2:表达式3）
    - 正则表达式（123′ matches ‘\\d{3}）