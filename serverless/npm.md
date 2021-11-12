#### 删除某个分支
```
npm unpublish @serverless-devs/s@2.0.87-7 --force
```

#### 灰度版本
1. 发布灰度版本
```
npm publish --tag=beta
```
2. 按照灰度版本
```
npm i @serverless-devs/s@beta -g 
```