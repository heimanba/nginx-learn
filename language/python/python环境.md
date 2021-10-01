## 前言

由于 Python2 和 Python3 差异极大(三方库也不兼容)，Python 开发时候，需要同时运行多个不同版本 Python，这时候需要环境管理工具

## virtualenv

pycharm 中默认管理工具，或者直接使用命令安装

```
pip install virtualenv
```

![](https://img.alicdn.com/imgextra/i3/O1CN01xylwuz1oGjSMVkoNK_!!6000000005198-2-tps-1600-1244.png)

#### 脚本使用形式

1. python2

```
$ pip install virtualenv
$ virtualenv --no-site-packages venv
$ source venv/bin/activate
```

2. python3

```
$ python3 -m venv my_venv
$ source my_venv/bin/activate
```

这时候 命令行的前缀变成有个`(venv)`的前缀，表示当前环境是一个`venv`的 python 环境。可以在这个环境下安装各种第三方包

## requirements.txt

python 项目必须包含一个`requirements.txt`文件类型`nodejs`中的`package.json`文件，用来记录所有依赖包以及精确的版本号，以便新环境部署。安装好的文件如下:

```
asgiref==3.4.1
Django==3.2.7
pytz==2021.1
sqlparse==0.4.2
typing-extensions==3.10.0.2
```

#### 生成文件

```
$ pip freeze >requirements.txt
```

#### 安装文件

```
$ pip install -r requirements.txt
```

## python -m

### 例子

```
├── try_demo
│   ├── package1
│   │   ├── __init__.py
│   │   ├── module1.py
│   ├── package2
│   │   ├── __init__.py
│   │   ├── module2.py
```

#### module2.py

模块中测试为：

```
import sys
print("module2 is running!")
print("module2 __name__ is:", __name__)
print("now sys.path is:")
print('\n'.join(sys.path))
```

1.  python package2/module2.py

```
module2 is running!
module2 __name__ is: __main__
now sys.path is:
/Users/yk/heimanba/aliFE/python/pythonProject/package2
/Library/Frameworks/Python.framework/Versions/3.7/lib/python37.zip
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload
/Users/yk/heimanba/aliFE/python/pythonProject/venv/lib/python3.7/site-packages
```

2.  python -m package2/module2

```
module2 is running!
module2 __name__ is: __main__
now sys.path is:
/Users/yk/heimanba/aliFE/python/pythonProject
/Library/Frameworks/Python.framework/Versions/3.7/lib/python37.zip
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload
/Users/yk/heimanba/aliFE/python/pythonProject/venv/lib/python3.7/site-packages

```
