通过 [Serverless Framework Express 组件](https://github.com/serverless-components/tencent-express)，可以快速实现 Express 传统应用从本地到 Serverless 函数平台的迁移。

## 迁移前提

- 已经 [安装 Serverless Framework 1.67.2](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85.md) 以上版本。
- 已经[注册腾讯云账号](https://cloud.tencent.com/document/product/378/17985)并完成[实名认证](https://cloud.tencent.com/document/product/378/10495)。

  > 如果您的账户为**腾讯云子账号**，请首先联系主账号，参考 [账号和权限配置](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E.md#%E5%AD%90%E8%B4%A6%E5%8F%B7%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE) 进行授权。

## 架构说明

Express 组件将在腾讯云账号中使用到如下 Serverless 服务：

- **API 网关**：API 网关将会接收外部请求并且转发到 SCF 云函数中。
- **SCF 云函数**：云函数将承载 Express 应用。
- **CAM 访问控制**：该组件会创建默认 CAM 角色用于授权访问关联资源。
- **COS 对象存储**：为确保上传速度和质量，云函数压缩并上传代码时，会默认将代码包存储在特定命名的 COS 桶中
  
## 操作步骤

> 以下步骤主要针对命令行部署操作，控制台部署请参考[控制台部署指南](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E6%A1%86%E6%9E%B6%E8%BF%81%E7%A7%BB/%E6%8E%A7%E5%88%B6%E5%8F%B0%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97.md)。
### 1. （可选）初始化 Express 模版项目
如果您本地并没有 Express 项目，可通过以下指令快速新建一个 Express 项目模版（本地已有项目可跳过该步骤）
```
serverless init express-starter --name example
cd example
```

### 2. 修改项目代码
打开 Express 项目的入口文件 sls.js（或 app.js），注释掉本地的监听端口，并导出默认的 Express app:

```javascript
// sls.js

const express = require('express');
const app = express();

// *****

// 注释掉本地监听端口
// app.listen(3000);

// 导出 Express app
module.exports = app;
```

### 2. 快速生成 yml 文件并进行部署
完成代码修改后，通过执行 `sls deploy` 指令，Serverless Framework 会自动帮您生成基本的 `serverless.yml` 文件，并完成部署，实现 Express 框架应用的快速迁移。

生成的默认配置文件如下：
```yml
component: express
name: expressDemo
app: appDemo

inputs:
  entryFile: sls.js #以您实际入口文件名为准
  src: ./
  region: ap-guangzhou
  runtime: Nodejs10.15
  apigatewayConf:
    protocols:
      - http
      - https
    environment: release
```

部署完成后，通过访问输出的 API 网关链接，完成对应用的访问。

### 3. 修改 yml 文件

基于您实际部署需要，您可以在 `serverless.yml` 中完成更多配置，并执行 `sls deploy`重新部署。

yml 文件的配置信息请参考[ Express 组件全量配置](https://github.com/serverless-components/tencent-express/blob/master/docs/configure.md)

### 4. 监控运维
部署完成后，您可以通过访问 [Serverless 应用控制台](https://console.cloud.tencent.com/ssr)，查看应用的基本信息，监控日志。

