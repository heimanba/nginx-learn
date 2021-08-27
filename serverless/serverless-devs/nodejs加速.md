## 背景
在平时开发Node.js应用过程中，很少关注进程启动耗时，但是以下几个场景需要考虑
- serverless: Node.js serverless-runtime已经作为前端新研发模式的基石。Nodejs Faas的平台致力于缩短应用的冷启动时间
- cli: 作为一个开箱即用的工具，需要提供快速启动能力
- Electron: 桌面应用
## 优化
影响我们启动速度的无非是两个点，文件IO和代码编译，我们来看下如何优化。

#### 文件IO
1. 查找模块
nodejs模块查找是一个嗅探文件是否存在过程，其中会因为判断文件是否存在，产生大量Open操作，在依赖复杂场景，开销较大
2. 读取文件内容
找到模块后，就需要读取文件内容，之后进入编译过程，如果文件内容过多，这个过程也会比较慢
既然模块依赖会产生很多I/O操作，那么将模块扁平化，变成一个文件，应该可以加速。`vercel`开发`ncc`工具可以来做这方面工作

#### 代码编译
另外一个耗时的操作就是把javascript代码编译成V8字节码执行。但是很多模块是公用的并非动态变化，不需要每次编译。可以编译好直接使用。通过`v8-compile-cache`库可以解决这个问题


## 组件库
#### ncc
1. 安装
```
$ npm i @vercel/ncc
```

#### v8-compile-cache
```
$ npm install --save v8-compile-cache
```
- 缓存目录为定义的环境变量`V8_COMPILE_CACHE_CACHE_DIR`,或者默认为`<os.tmpdir()>/v8-compile-cache-<V8_VERSION>`
- 