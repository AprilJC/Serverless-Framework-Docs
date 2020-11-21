通过 [Serverless Framework Django 组件](https://github.com/serverless-components/tencent-django)，可以快速实现 Django 应用从本地到 Serverless 函数平台的迁移。


## 迁移前提

- 已经 [安装 Serverless Framework 1.67.2](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85.md) 以上版本。
- 已经[注册腾讯云账号](https://cloud.tencent.com/document/product/378/17985)并完成[实名认证](https://cloud.tencent.com/document/product/378/10495)。

  > 如果您的账户为**腾讯云子账号**，请首先联系主账号，参考 [账号和权限配置](https://github.com/AprilJC/Serverless-Framework-Docs/blob/main/docs/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E.md#%E5%AD%90%E8%B4%A6%E5%8F%B7%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE) 进行授权。

## 操作步骤

### 1. （可选）初始化 Django 模版项目
如果您本地并没有 Django 项目，可通过以下指令完成 Django 项目初始化（本地已有项目可跳过该步骤）
```
serverless init django-starter --name example
cd example
```

### 2. 安装项目依赖
如果您自己创建项目，请将 Python 所需要的依赖安装到项目目录，例如本实例需要Django，所以可以通过pip进行安装：
```
pip install Django -t ./
```

### 3. 配置 yml 文件
在项目根目录下，新建 `serverless.yml` 文件，并将下列配置模版粘贴到文件中，实现基本的项目配置。
>基于您实际部署需要，您可以在 `serverless.yml` 中完成更多配置，yml 文件的配置信息请参考[ Django 组件全量配置](https://github.com/serverless-components/tencent-django/blob/master/docs/configure.md)

```sh
touch serverless.yml
```

```yml
#serverless.yml
component: django
name: djangoDemo
org: orgDemo
app: appDemo
stage: dev

inputs:
  region: ap-guangzhou
  djangoProjectName: mydjangocomponent
  src: ./src
  functionConf:
    timeout: 10
    memorySize: 256
    environment:
      variables:
        TEST: vale
  apigatewayConf:
    protocols:
      - https
    environment: release
```


### 4. 应用部署
通过 `sls deploy` 命令进行部署，并可以添加 --debug 参数查看部署过程中的信息。

```
sls deploy --debug
```
部署完成后，通过访问输出的 API 网关链接，完成对应用的访问。


