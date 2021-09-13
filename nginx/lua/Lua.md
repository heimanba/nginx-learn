## Lua和LuaJIT区别
Lua非常高效，他的运行比其他很多脚本(Perl,Python,Ruby)都快，但是还不够快，LuaJIT是为了压榨速度的尝试。它利用即时编译(Just-in Time)技术把Lua代码编译成本地机器码后交给CPU直接执行。
LuaJIT 是采用 C 和汇编语言编写的 Lua 解释器与即时编译器。LuaJIT 被设计成全兼容标准的 `Lua 5.1` 语言。可以认为`LuaJIT`是一个高效的`Lua`实现。另一个区别是`LuaJIT`支持比`Lua5.1`语言更多的基本原语和特性，功能更加强大

## 基础数据类型
### string
```
print(type("hello world"))
```
#### 表现形式有
- 一对单引号`'hello'`
- 一对双引号`"hello"`
- 长括号方式定义(`[[ ]]`), `[["add\name",'hello']]`

### nil(空)
```
print(type(nil)) 
```
表示"无效值"，一个变量在第一次赋值前的默认值是`nil`,将`nil`赋予一个全局变量等同于删除它

### boolean(布尔)
可选值为true/false,Lua总的`nil`和`false`为`假`,其他所有值均为`真`。比如0和字符串为`真`

### table (表)
Table类型实现了一种抽象的`关联数组`。通常是`string`或者`number`类型，也可以是除`nil`外任意类型

## 表达式
### 算术表达式
- `+` 加法
- `-` 减法
- `*` 乘法
- `/` 除法
- `/` 除法
- `^` 指数
- `%` 取模

### 关系运算法
- `<` 小于
- `>` 大于
- `<=` 小于等于
- `>=` 大于等于
- `==` 等于
- `~=` 不等于
> 注意: 使用`==`做判断时候，要注意对于`table`和函数，lua是作引用比较的，只有当两个变量引用等于同一个对象时，才认为相等。

### 逻辑运算符
- `and` 逻辑与
- `or` 逻辑或
- `not` 逻辑非

#### 字符串拼接
使用`..`(两个点)拼接字符串，也可以使用`string.format`连接字符串
```
str1 = string.format("%s world", "hello")
pring(str1)
str2 = "hello".."world"
print(str2)
```
由于lua字符串是只读的，每次运算都会产生新字符串，推荐使用`table.concat()`进行拼接。
## 控制语句
### if-else控制结构
```
x=10
if x > 0 then
    print('x is positive')
else
    print('x  is nagtive')
end
```
### while型控制结构
```
x = 1
sum = 0

while x <= 5 do
    sum = sum + x
    x = x + 1
end
print(sum)
```
### for 控制结构
#### 格式为:
```
for var = begin, finish, step do
    --body
end
```
#### for泛型
通过一个迭代器(iterator)函数来遍历所有值
```
local a = {"a", "b", "c", "d"}
for i, v in ipairs(a) do
  print("index:", i, " value:", v)
end
```

### break, return, goto
1. break 
break 可以用来终止`while`,`for`三种循环的执行，并跳出当前循环体。继续执行当前循环后的语句
2. return
主要从函数中返回结果
3. goto
goto 可以用来简化错误处理的流程
```
local function process(input)
    print("the input is", input)
    if input < 2 then
        goto failed
    end
    -- 更多处理流程和 goto err

    print("processing...")
    do return end
    ::failed::
    print("handle error with input", input)
end

process(1)
```
## 函数
函数是一种对语句和表达式进行抽象的主要机制
### 定义
```
function function_name(arc)
    -- body
end
```
由于全局变量一般会污染全局命名空间，也有性能的损耗，因此应该尽量使用`局部函数`，在开头加上`local`修饰符
### 参数
函数参数是按值传递的
#### 可变参数
```
local function func(...)
    local temp = {...}
    local ans = table.concat(temp, "|")
    print(ans)
end
```
### 函数返回值
允许函数返回多个值，Luau的库函数中，有一些就是返回多个值
```
local s, e = string.find("hello world", "llo")
print(s, e)
```
## 模块
`lua`提供了`require`函数用来加载模块，只需要调用`require`"file"就可以的。
- `my.lua`代码
```
local _M = {}

local function get_name()
    return "Lucy"
end

function _M.greeting()
    print("hello " .. get_name())
end

return _M
```
- `main.lua`代码
```
local my_module = require("my")
my_module.greeting()
```
## string库
#### string.byte(s [, i [, j ]])
返回字符串对应`ASCII`码,`i`的默认值为`1`
```
print(string.byte("abc", 1, 2)) # 97      98
```
#### string.char (...)
返回这些整数对应`ASCII`码组成字符串。
```
print(string.char(65, 66)) # AB
```
#### string.upper(s)
把所有小写字母变成大写字符串
```
print(string.upper("Hello Lua"))  # HELLO LUA
```
#### string.lower(s)
把所有大写字母变成小写字母的字符串。
```
print(string.upper("HELLO LUA"))  # hello lua
```
#### string.len(s)
接收一个字符串，返回他的长度
```
print(string.len("hello lua")) # 9
```
不推荐使用，应总是使用`#`运算符获取长度。
```
print(#"hello lua") # 9
```
#### string.format(formatstring, ...)
格式化参数,`%d`十进制，`%f`浮点数，`%s`字符串
```
print(string.format("%.4f", 3.1415926))     -- 保留4位小数
```
#### string.match(s, p [, init])
第三个参数 init 默认为 1，并且可以为负整数，当 init 为负数时，表示从 s 字符串的 string.len(s) + init + 1 索引处开始向后匹配字符串 p。
```
print(string.match("hello lua", "lua")) # lua
print(string.match("today is 27/7/2015", "%d+/%d+/%d+")) # 27/7/2015
```
#### string.gmatch(s, p)
返回一个迭代器函数，可以编译到字符串`s`中出现模式`p`所有的地方
```
s = "hello world from Lua"
for w in string.gmatch(s, "%a+") do  --匹配最长连续且只含字母的字符串
    print(w)
end
```
#### string.sub(s, i [, j])
返回字符串 s 中，索引 i 到索引 j 之间的子字符串。
```
print(string.sub("Hello Lua", 4, 7)) # lo L
```
#### string.gsub(s, p, r [, n])
将目标字符串 s 中所有的子串 p 替换成字符串 r。可选参数 n，表示限制替换次数。返回值有两个，第一个是被替换后的字符串，第二个是替换了多少次。
```
print(string.gsub("Lua Lua Lua", "Lua", "hello", 2)) --指明第四个参数
```
#### string.reverse (s)
字符串反转
```
print(string.reverse("Hello Lua"))  --> output: auL olleH
```