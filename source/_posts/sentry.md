---
title: Sentry使用
date: 2020-06-30 17:32:21
tags:

---

# Sentry使用

## 介绍
Sentry 是一个开源的实时错误追踪系统，可以帮助开发者实时监控并修复异常问题。它主要专注于持续集成、提高效率并且提升用户体验。Sentry 分为服务端和客户端 SDK，前者可以直接使用它家提供的在线服务，也可以本地自行搭建；后者提供了对多种主流语言和框架的支持。目前公司的项目也都在逐步应用上 Sentry 进行错误日志管理。

## 私有化部署
Sentry提供了一套私有化部署的方式,基于Docker来实现快速部署。
> Official bootstrap for running your own Sentry with Docker.
>
> https://github.com/getsentry/onpremise


### 环境需求
* Docker 17.05.0+
* Compose 1.23.0+

### 最低硬件需求
* 至少需要2400M RAM

### 安装
如果执行默认安装的话，直接clone repo并运行`./install.sh`。

通常需要对配置进行修改，包括如下：
* config.yml
* sentry.conf.py
* .env

## 对接React项目

### 使用

首先添加SDK
```
# Using yarn
$ yarn add @sentry/browser

# Using npm
$ npm install @sentry/browser
```

然后需要在React应用启动的时候初始化Sentry。
Example:
```
import React from 'react';
import * as Sentry from '@sentry/browser';
import App from 'src/App';

Sentry.init({dsn: "https://7758d66d228740dfb4a0de39dc69f255@dev-sentry.mokahr.com/56"});

ReactDOM.render(<App />, document.getElementById('root'));
```
`dev-sentry.yourhost.com`替换为自己的sentry url，`7758d66d228740dfb4a0de39dc69f255`替换为自己的publickey

然后就可以进行报错的测试了.
Example:
```
return <button onClick={methodDoesNotExist}>Break the world</button>;
```
### 对接SourceMap
为了安全以及加载性能考虑，一般不会把sourcemap发布到生产环境，所以可以通过把sourcemap上传到sentry的方式来实现错误定位。

Sentry使用Source Map的方式有三种：
* 通过Sentry CLI上传
* 通过Webpack插件上传
* 从生产环境获取

#### Webpack插件方式

##### 安装
Using npm:
```
$ npm install @sentry/webpack-plugin --only=dev
```
Using yarn:
```
$ yarn add @sentry/webpack-plugin --dev
```

##### 配置CLI 配置
如果需要对CLI的参数进行修改，可以通过修改`.sentryclirc`或者env。文档可以查看docs.sentry.io/learn/cli/configuration

通常我们还需要配置
* url sentry 地址
* org 组织名称
* project 项目名称
* token

Example：
```
[defaults]
url=https://dev-sentry.mokahr.com/
org=moka
project=react-fk-test

[auth]
token=e42edc9473294dasdsadsa9fad43b9b899dsdsadsadsadadsada76843e25d
```


##### 使用
```
const SentryCliPlugin = require('@sentry/webpack-plugin');

const config = {
  plugins: [
    new SentryCliPlugin({
      include: '.',
      ignoreFile: '.sentrycliignore',
      ignore: ['node_modules', 'webpack.config.js'],
      configFile: 'sentry.properties',
    }),
  ],
};
```

通常我们还需要配置
* release 版本名称
* urlPrefix 资源前缀

##### 发布前删除map文件
使用CleanWebpackPlugin
```
const config = {
  plugins: [
    new CleanWebpackPlugin({
      verbose: true,
      cleanAfterEveryBuildPatterns: ['./*.map'],
    }),
  ],
};
```