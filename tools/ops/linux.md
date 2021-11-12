#### Linux 中的文件数量
```
sudo find . -type f -print | wc -l
```

#### 查看证书过期
```
$ openssl x509 -in arms.aliyuncs.com_SHA256withRSA_RSA.crt -noout -dates -subject
notBefore=Jan 11 08:36:02 2021 GMT
notAfter=Feb 12 08:36:02 2022 GMT
subject= /C=CN/ST=ZheJiang/L=HangZhou/O=Alibaba (China) Technology Co., Ltd./CN=*.arms.aliyuncs.com
```
证书有效期为 `2021/1/11`到`2022/2/12`

#### ps
1. ps aux VS ps -aux
`ps -aux`会解释为用户名为`x`的用户所有进程，如果用户名`x`不存在，则解释为`ps aux`

2. ps aux VS ps -ef
输出差别不大，只是展示风格不一致，aux是BSD风格，`-ef`是System V风格，另外是`aux`会截断command列，但是`-ef`不会，直接影响`grep`时候结果


3. 查看挂载点信息
```
mount | grep nfs
```