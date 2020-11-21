通过 [Serverless Framework Next.js 组件](https://github.com/serverless-components/tencent-nextjs)，可以快速实现 Next.js 传统应用从本地到 Serverless 函数平台的迁移。

## 迁移前提

- 已经 [安装 Serverless Framework 1.67.2](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85.md) 以上版本。
- 已经[注册腾讯云账号](https://cloud.tencent.com/document/product/378/17985)并完成[实名认证](https://cloud.tencent.com/document/product/378/10495)。

  > 如果您的账户为**腾讯云子账号**，请首先联系主账号，参考 [账号和权限配置](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E.md#%E5%AD%90%E8%B4%A6%E5%8F%B7%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE) 进行授权。

## 架构说明

Next.js 组件将在腾讯云账号中使用到如下 Serverless 服务：

- **API 网关**：API 网关将会接收外部请求并且转发到 SCF 云函数中。
- **SCF 云函数**：云函数将承载 Next.js 应用。
- **CAM 访问控制**：该组件会创建默认 CAM 角色用于授权访问关联资源。
- **COS 对象存储**：为确保上传速度和质量，云函数压缩并上传代码时，会默认将代码包存储在特定命名的 COS 桶中

## 操作步骤

> 以下步骤主要针对命令行部署操作，控制台部署请参考[控制台部署指南](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E6%A1%86%E6%9E%B6%E8%BF%81%E7%A7%BB/%E6%8E%A7%E5%88%B6%E5%8F%B0%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97.md)。

### 1. （可选）初始化 Next.js 模版项目
如果您本地并没有 Next.js 项目，可通过以下指令快速新建一个 Next.js 项目模版（本地已有项目可跳过该步骤）
```
serverless init nextjs-starter --name example
cd example
```

### 2. （可选）修改入口函数代码
如果项目使用了自定义 Node.js 服务，比如 express 或者 koa 框架，你需要对入口文件 sls.js（或 app.js）做出改造，导出对应框架 app（未使用可跳过该步骤）, 点此查看[改造模版](#1)。


### 3. 项目构建
```
npm run build
```

### 4. 快速生成 yml 文件并进行部署

完成代码修改后，通过执行 `sls deploy` 指令，Serverless Framework 会自动帮您生成基本的 `serverless.yml` 文件，并完成部署，实现 Next.js 框架应用的快速迁移。

生成的默认配置文件如下：
```yml
component: nextjs
name: nextjsDemo
app: appDemo

inputs:
  src: ./
  exclude:
      - .env
  region: ap-guangzhou
  runtime: Nodejs10.15
  apigatewayConf:
    protocols:
      - http
      - https
    environment: release
```
部署完成后，通过访问输出的 API 网关链接，完成对应用的访问。

### 5. 查看部署状态

在`serverless.yml`文件所在的目录下，通过如下命令查看部署状态：

```
$ serverless info
```

## 更多操作

<span id="1"></span>
### 项目迁移改造模版
- Express 模版

```js
const express = require('express')
const next = require('next')

// not report route for custom monitor
const noReportRoutes = ['/_next', '/static', '/favicon.ico']

async function createServer() {
  const app = next({ dev: false })
  const handle = app.getRequestHandler()

  await app.prepare()

  const server = express()
  server.all('*', (req, res) => {
    noReportRoutes.forEach((route) => {
      if (req.path.indexOf(route) !== -1) {
        req.__SLS_NO_REPORT__ = true
      }
    })
    return handle(req, res)
  })

  // define binary type for response
  // if includes, will return base64 encoded, very useful for images
  server.binaryTypes = ['*/*']

  return server
}

module.exports = createServer
```

- Koa 模版
```js
const Koa = require('koa')
const next = require('next')

async function createServer() {
  const app = next({ dev: false })
  const handle = app.getRequestHandler()

  const server = new Koa()
  server.use((ctx) => {
    ctx.status = 200
    ctx.respond = false
    ctx.req.ctx = ctx

    return handle(ctx.req, ctx.res)
  })

  // define binary type for response
  // if includes, will return base64 encoded, very useful for images
  server.binaryTypes = ['*/*']

  return server
}

module.exports = createServer
```

### 自定义监控

当您在部署 Next.js 应用时，如果 `serverless.yml` 中未指定 `role`，默认会尝试绑定 `QCS_SCFExcuteRole`，并且开启自定义监控，帮助您收集应用监控指标。对于为自定义入口文件的项目，会默认上报除含有 `/_next` 和 `/static` 的路由。

如果您想自定义上报自己的路由性能，可以自定义 `sls.js` 入口文件。
对于无需上报的路由，在 express 服务的 `req` 对象上添加 `__SLS_NO_REPORT__` 属性值为 `true` 即可。例如：
```js
server.get('/no-report', (req, res) => {
  req.__SLS_NO_REPORT__ = true
  return handle(req, res)
})
```
配置后，用户在访问 `GET /no-report` 路由时，就不会上报自定义监控指标。


