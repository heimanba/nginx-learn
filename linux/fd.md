## fd
`fd`是`File descriptor`缩写，中文名称为`文件描述符`，文件描述符是非负整数，本质上一个索引值。
什么时候拿到`fd`
当打开一个文件时，内核想进程返回一个文件描述符(`open`系统调用得到)，后续`read`,`write`这个文件时，则只需要这个文件描述符来标识该文件，将其作为参数传入`read`,`write`。
#### fd的值范围是什么？
在`POSIX`语义中，0,1,2这三个fd值被赋予特殊含义，分别是标准输入（ STDIN_FILENO ），标准输出（ STDOUT_FILENO ），标准错误（ STDERR_FILENO ）。
可以通过`ulimit`命令查看当前系统配置
```
$ ulimit -n
4864
```
系统进程默认最多打开`4864`文件。

## inode
inode包含文件的元信息，具体有以下内容
```
- 文件字节数
- 文件拥有者 User ID
- 文件的Group ID
- 文件读写执行权限
- 文件的时间戳，共有三个：ctime指inode上一次变动的时间，mtime指文件内容上一次变动的时间，atime指文件上一次打开的时间。
- 链接数，即有多少文件名指向这个inode
- 文件数据block的位置
```
使用`stat`命令查看文件的`inode`信息
```
$ stat 3.html
  File: 3.html
  Size: 7         	Blocks: 8          IO Block: 4096   regular file
Device: fd01h/64769d	Inode: 789667      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2021-07-12 20:12:38.027826658 +0800
Modify: 2021-07-12 20:08:56.709832295 +0800
Change: 2021-07-12 20:08:56.709832295 +0800
 Birth: -
```
#### inode大小
indode会消耗硬盘空间，在硬盘格式化的时候，操作系统自动将硬盘分为两个区域
- 数据区: 存放文件数据
- inode区，存放inode包含的信息
查看每个硬盘分区的inode总数和已使用的数量，使用`df`命令
```
df -i
```

#### inode号码
Unix/Linux系统内部不使用文件名，而使用inode号码来识别文件。对于系统来说，文件名只是inode号码便于识别的别称或者绰号。
```
$ ls -i 1.txt
789666 1.txt
```

#### 硬链接
文件名和inode号码是`一一对应`关系，每个inode号码对应一个文件名，但是Unix/Linux系统允许，多个文件名指向同一个inode号码。
不同的文件名可以访问同样内容，对于文件内容修改会影响所有的文件名，但是删除一个文件名，不影响另一个文件名访问，称为`硬链接`。
```
$ ln 1.html 1.bak.html
$ ls -i 1.html 1.bak.html
789666 1.bak.html  789666 1.html
```
#### 软链接
`A`文件和`B`文件号码虽然不一样，但是A文件内容是B的路径，读取A时，系统会自动指向文件B。因此无论打开哪个文件，读取的都是文件B，这时候A就称为文件B的`软链接(soft link)`或者符号链接
```
$ ln -s 1.html 1.bak-s.html
$ ls -i 1.html 1.bak-s.html
789663 1.bak-s.html  789666 1.html
```