## 背景
Serverless-devs cli工具本质也是一个用户的交互界面，但是现在无论在输入输出还是错误处理等都非常不规范，输出一大堆的用户看不懂，或者不需要用户理解的信息，恰好Serverless-Framework发布了他的新版的cli设计，我们来参考下

## 设计原则
设计目标是让serverless产品，提供更简单输出和一致性设计。来提升用户体验
- 优先考虑最重要的信息
比如部署是让更改生效，最重要的信息不是部署日志，而是应用程序URL。
另外一方面，失败部署的最重要的信息是错误本身
- 最小化噪音
默认情况下只显示必要的信息，我们可提供详细信息作为一个选项，但它不应是默认行为

## 颜色和样式
应该和产品本身保持一致，比如：
白色(#fff)：默认颜色，几乎用于所有的信息
红色(#fd5750)：用于引起注意
灰色(#8c8d91)：用户辅助信息和辅助颜色
其他的颜色使用，应该属于例外情况。
#### 使用红色
使用红色来突出一切，包括成功等，用来突出品牌色
但是并不意味着成功和错误应该以相同方式显示，
![images](https://img.alicdn.com/imgextra/i1/O1CN01s60gFz1GTNRqt3dpS_!!6000000000623-2-tps-1456-412.png)

## 设计样例
#### 部署过程


#### 部署结果
![deploy](https://img.alicdn.com/imgextra/i1/O1CN01jMJlMM1yuFmnyPoEg_!!6000000006638-2-tps-1322-576.png)

#### 错误
![](https://img.alicdn.com/imgextra/i2/O1CN01aKsYNA1mmDRkS9DXB_!!6000000004996-2-tps-1440-976.png)


## 参考
[sls](https://sls-redesign.netlify.app/)