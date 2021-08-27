## 动机

用户在使用函数计算的时候最多的场景还是在应用开发上，也就是我们说的 HTTP 触发器。这里在阿里云函数计算中一般有几种方式

- Nodejs Runtime
  冷启动的时间最快，但是限制比较多，不支持传统 web 框架支持，不支持文件写入等。
- [Custom Runtime](https://help.aliyun.com/document_detail/132044.html)
  官方提供的自定义的镜像，可以定制个性化语言以及各种语言的小版本执行环境，支持一键迁移现有 web 应用或者基于传统开发的 web 项目到函数计算平台
  在更目录需要有个`bootstrap`文件

```
#!/usr/bin/env bash
export PORT=9000
npx nuxt start --hostname 0.0.0.0 --port $PORT
```

- [Custom Container](https://help.aliyun.com/document_detail/179367.html)
  开发者将容器镜像作为函数的交付物，通过 HTTP 协议和函数计算系统交互,使得开发环境和线上环境保持一致。

### 上面的几种函数的交付方式优劣

- 冷启动速度

Nodejs Runtime `>` Custom Runtime `>` Custom Container

首先因为`Custom Container`属于用户自定义镜像，函数启动前需要先有 pull 镜像操作，所以启动速度最慢。
`Custom Runtime`使用用户自定义脚本，中间有个 proxy 代理，性能也有所损耗。

- 灵活性

Custom Container `>` Custom Runtime `>` Nodejs Runtime
由于`Custom Container`是用户自定义的镜像，可以自由读写文件等操作。

## Nodejs Runtime 支持传统框架

开源做的比较好的[serverless-http](https://github.com/dougmoscrop/serverless-http)，但是这个库支持官方的 aws。
阿里云函数计算的 HTTP 触发器和 aws 等厂商非常不一样，主要的原因是 aws 触发器都是标准的`event, context, callback`形式。紧密结合 api gateway。但是阿里云的 api gateway 产品和其他云厂商的不太一样，函数计算的 HTTP 触发器中自带 `gateway`,形式为`request, response, callback`。
整体思路是在入口层面将函数计算的请求转化为`serverless-http`能够识别的请求。

#### 核心代码

```
const serverless = require('serverless-http');

module.exports = (app, opts) => {
  return async (res, req, context) => {
    const serverlessHandler = serverless(app, opts);
    const data = await serverlessHandler(event, req);
    const { statusCode, headers, body, isBase64Encoded } = data;

    req.setStatusCode(statusCode);
    req.send(body);
    for (const key in headers) {
      if (headers.hasOwnProperty(key)) {
        const value = headers[key];
        response.setHeader(key, value);
      }
    }
  }
}
```
详细代码参考[fc-http](https://github.com/Serverless-Devs/fc-http/blob/main/src/index.js)


