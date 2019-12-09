---
title: 创建一个组件库
date: 2019-04-24 19:32:21
tags:

---

# 创建一个组件库

组件库一般分为两个主要部分，组件实现和文档示例。

目前考虑基于[umi-library](https://github.com/umijs/umi/tree/master/packages/umi-library)实现一个组件库。



## umi-library

### 介绍

umi-library是一个基于rollup和docz实现的一个组件库工具，同时它也支持babel打包。

### 特性

- ✔︎ 基于 [docz](https://www.docz.site/) 的文档功能
- ✔︎ 基于 [rollup](http://rollupjs.org/) 和 babel 的组件打包功能
- ✔︎ 支持 TypeScript
- ✔︎ 支持 cjs、esm 和 umd 三种格式的打包
- ✔︎ esm 支持生成 mjs，直接为浏览器使用
- ✔︎ 支持用 babel 或 rollup 打包 cjs 和 esm
- ✔︎ 支持多 entry
- ✔︎ 支持 lerna
- ✔︎ 支持 css 和 less，支持开启 css modules

### 安装

Install `umi-library` via yarn or npm.

```bash
$ yarn add umi-library
```

### 使用

```bash
# Bundle library
$ umi-library build

# umi-lib is the alias for umi-library
$ umi-lib build

# dev with doc
$ umi-lib doc dev

# build doc
$ umi-lib doc build

# deploy doc to github.io
$ umi-lib doc deploy
```



## docz

### 介绍

docz是一个非常简单的组件文档工具，基于MDX(Markdown + JSX)，支持typescript，但是只能用于react。

### 安装

```bash
$ yarn add docz docz-theme-default --dev
```

### 使用

```bash
$ yarn docz:dev
```

### MDX例子

```markdown
---
name: Hello world
---

import { Button } from './Button'

# Hello world

Hello, I'm still a mdx file, but now I have a button component!

<Button>Click</Button>
```



## 创建组件库

这里使用create-umi创建

```bash
$ mkdir demo
$ cd demo
$ yarn create umi
```

选择library，并按照提示填写相关内容，然后会得到一个这样的目录结构

```
.
├── .gitignore
├── .prettierignore
├── .prettierrc
├── .umirc.library.js
├── LICENSE
├── README.md
├── package.json
└── src
    ├── index.css
    ├── index.js
    └── index.mdx
```

然后添加typescript的支持

```bash
$ yarn add --dev tslint tslint-react typescript tslint-eslint-rules
```

创建`tsconfig.json`文件

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "importHelpers": true,
    "jsx": "react",
    "esModuleInterop": true,
    "sourceMap": true,
    "baseUrl": ".",
    "strict": true,
    "paths": {
      "@/*": ["src/*"]
    },
    "allowSyntheticDefaultImports": true
  }
}
```

创建`tslint.yml`文件，进行typescript的语法检测，类似于eslint.

```yaml
defaultSeverity: error
extends:
  - tslint-react
  - tslint-eslint-rules
jsRules:
rules:
  eofline: true
  no-console: true
  no-construct: true
  no-debugger: true
  no-reference: true
  jsx-no-multiline-js: false

```

创建typings.d.ts文件，提供样式和图片的引入支持。

```typescript
declare module '*.css';
declare module "*.png";
declare module '*.less';
```

在`.umirc.library.js`文件中添加docz的ts支持

```js
export default {
  ......
  doc: {
    typescript: true,
  },
}
```

如果使用antd的话，添加按需加载的支持

```bash
$ yarn add --dev babel-plugin-import
```

在`.umirc.library.js`文件中添加

```js
export default {
  ......
  extraBabelPlugins: [
    ['babel-plugin-import', {
      libraryName: 'antd',
      libraryDirectory: 'es',
      style: true,
    }]
  ],
}
```

剩下的就可以按照团队喜好添加各种eslint等的配置

虽然umi-library支持lerna，但是暂时只介绍单包方式

在src下创建各种组件的实现，然后在`src/index.tsx`中添加类似于

```tsx
export { default as TestIcon } from './TestIcon';

export { default as TestInput } from './TestInput';

export { default as TestModal } from './TestModal';
```

然后就可以通过`yarn build`构建了

验证通过就可以进入愉快的发布流程。