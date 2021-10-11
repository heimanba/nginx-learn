## top命令
#### 第一行(总览，负载相关)
```
top - 22:39:19 up 114 days,  5:40,  1 user,  load average: 0.06, 0.09, 0.06
```
- top - 22:39:19 
当前系统时间
- up 114 days,  5:40
 系统已经运行时间
- 1 user
当前登录用户数
- load average: 0.06, 0.09, 0.06
系统负载，任务队列的平均长度，分别为 1分钟，5分钟，15分钟到现在的平均值

#### load average相关
1. 经验
当系统负荷持续大于0.7，你必须开始调查了，问题出在哪里，防止情况恶化。
当系统负荷持续大于1.0，你必须动手寻找解决办法，把这个值降下来。
当系统负荷达到5.0，就表明你的系统有很严重的问题，长时间没有响应，或者接近死机了。你不应该让系统达到这个值
2. 多核处理器
N个CPU的电脑，比如4核CPU，可接受的系统负荷最大值为`4`，
> 可以通过`cat /proc/cpuinfo`查看CPU核数。

### 第二行(进程相关)
```
Tasks: 157 total,   1 running, 156 sleeping,   0 stopped,   0 zombie
```
- 157 total
进程总数
- 1 running
当前运行进程数
- 156 sleeping
休眠进程数
- 0 stopped
停止的进程数
- 0 zombie
僵尸进程数

### 第三行(CPU相关)
```
%Cpu(s): 20.2 us,  1.9 sy,  0.0 ni, 77.0 id,  0.0 wa,  0.0 hi,  0.8 si,  0.0 st
```
- 20.2 us
用户控件占用CPU百分比
- 1.9 sy
内核空间占用CPU百分比
- 0.0 ni
用户进程空间内改变过优先级的进程占用CPU百分比
- 77.0 id
空闲CPU百分比
- 0.0 wa
等待输入输出的CPU时间百分比
- 0.0 hi
硬中断（Hardware IRQ）占用CPU的百分比,CPU服务于硬中断所耗费的时间总
- 0.8 si,  0.0 st
CPU服务于软中断所耗费的时间总额、Steal Time

### 第四行(内存相关)
```
KiB Mem :  7735020 total,   140772 free,  4092884 used,  3501364 buff/cache
```
- 7735020 total
内存总量(8G)
- 140772 free
内存空闲(1.4G)
- 4092884 used
内存使用量
- 3501364 buff/cache
缓存的内存量

#### 第五行(交换区使用情况)
```
KiB Swap:        0 total,        0 free,        0 used.  1554188 avail Mem 
```
- 0 total
使用的交换区总量
- 0 free
空闲交换区总量
- 0 used
交换区使用量

#### 第六行(进程相关)
- PID	进程号
- USER	进程创建者
- PR	进程优先级
- NI	nice值。越小优先级越高，最小-20，最大20（用户设置最大19）
- VIRT	进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
- RES	进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
- SHR	共享内存大小，单位kb
- S	进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
- %CPU	进程占用cpu百分比
- %MEM	进程占用内存百分比
- TIME+	进程运行时间
- COMMAND	进程名称
