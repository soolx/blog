---
title: NPM包发布与管理
date: 2019-03-31 21:07:39
tags:
---

# NPM包发布与管理

## 发布到官方NPM仓库

1. 首先需要在[NPM官网](https://www.npmjs.com/)创建一个账号

2. 然后通过`npm login`登录账号
3. 创建一个项目文件夹，通过`npm init`初始化，修改package.json里的项目信息，添加README.md.(已有项目的跳过)
4. 通过`npm publish`发布(包名不能与现有的包重名或者非常接近)



## 使用nrm进行npm镜像源管理

发布包的时候，可能需要在各种npm库之间切换，通过nrm可以快速的实现这个需求。

安装

```bash
yarn global add nrm
```

使用

```bash
nrm ls #列出nrm管理的registry列表
nrm current #显示当前的registry名称
nrm use taobao #切换registry到taobao源，这个命令对yarn和npm同时生效
nrm add taobao https://registry.npm.taobao.org #添加一个名为taobao的源到列表
nrm del taobao #从列表中删除taobao源
```



## 通过lerna进行包代码管理

### lerna init

初始化

```js
mkdir lerna-repo && cd $_
npx lerna init
```

将会创建一个 `lerna.json` 配置文件和一个`packages` 文件夹，目录结构大概这样：

```
lerna-repo/
  packages/
  package.json
  lerna.json
```

在`packages`里添加module并通过`npm init`初始化

```bash
mkdir packages/lerna-demo-core && cd packages/lerna-demo-core
npm init
```



### lerna bootstrap

通过`lerna bootstrap`处理依赖

`lerna bootstrap`会安装并处理package之间的依赖问题，比如`packages/lerna-demo-module-a`依赖了`packages/lerna-demo-core`，`bootstrap`会在`lerna-demo-module-a`的`node_modules`目录下建立指向`lerna-demo-core`的软链接。
```js
lerna add lerna-demo-core --scope=lerna-demo-module-a
```

此处有个[活生生的例子](https://github.com/soolx/lerna-demo)


### lerna publish

发布当前项目中的包。

如果有package发生变更的话，会让你选择一个新的版本号，然后就可以发布了。

***这里有个不好的地方是，如果发布包失败，也无法重试发布，必须删除这个tag，然后回退版本变更的commit才可以重新发布。***



### 版本管理方式

Lerna 提供两类管理项目的模式: Fixed 和 Independent.

1. Fixed/Locked mode (default)
    Fixed模式下，项目通过单一的版本进行控制。版本号放在项目根目录下的lerna.json文件的version这个字段。当你执行 lerna publish，如果有文件更新，它将发布新的版本。
    babel项目采用的就是这种模式
2. Independent mode (--independent)
    这种模式下，项目里的各个package独立维护自己的version，它将会忽略lerna.json中定义的version