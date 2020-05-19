---
title: Deno
date: 2020-05-19 14:37:40
tags:
---

# Deno入门

## 简介
终于等到Deno发布1.0了，很开心。A secure runtime for JavaScript and TypeScript.

## 安装
```bash
brew install deno
```
运行一个例子：
```bash
deno run https://deno.land/std/examples/welcome.ts
# Welcome to Deno 🦕
```
## 特点
Deno 内置了开发者需要的各种功能，不再需要外部工具。打包、格式清理、测试、安装、文档生成、linting、脚本编译成可执行文件等，都有专门命令。

Deno是一个采用Rust开发，使用V8的简单，现代且安全的Jav [app.ts](../../../../../deno-color/app.ts) aScript和TypeScript运行时。

1. 默认为安全模式。除非明确启用，否则没有文件，网络或环境的访问权限。
2. 开箱即用的支持TypeScript。
3. 只需要一个可执行文件。
4. 具有内置的工具，例如依赖项检查器（deno info）和代码格式化工具（deno fmt）。
5. 拥有一组保证能与[Deno](https://deno.land/std)一起使用的经过审查（审核）的标准模块：[deno.land/std](https://deno.land/std)

执行`deno -h`或`deno help`，就可以显示 Deno 支持的子命令。

## 一个简单的例子

一个可以从项目里遍历所有颜色的简单示例。

```typescript
const startTime = new Date();
const result = new Set();
const fileList: Array<string> = [];
async function printFilesNames(path: string) {
  for await (const entry of Deno.readDir(path)) {
    const newPath = `${path}/${entry.name}`;
    fileList.push(newPath);
    if (entry.isFile && entry.name.match(/\.less/)) {
      // console.log(entry);
      const decoder = new TextDecoder("utf-8");
      const data = await Deno.readFile(newPath);
      const content = decoder.decode(data);
      const list = content.match(/#[A-Fa-f0-9]{1,6}|rgba?\([0-9,]+\)/) || [];
      list.forEach((color) => {
        result.add(color);
      });
    }
    if (entry.isDirectory) {
      await printFilesNames(newPath);
    }
  }
}
printFilesNames(Deno.args[0]).then(() => console.log(Array.from(result), `读取文件数${fileList.length}`, `消耗时间${new Date().getTime() - startTime.getTime()}ms`));

```

执行方式

```
deno run --allow-read app.ts /Users/abc/project/demo
```



## 常见问题

1、使用std的时候遇到报错

```bash
error: TS2339 [ERROR]: Property 'utime' does not exist on type 'typeof Deno'.
    await Deno.utime(dest, statInfo.atime, statInfo.mtime);
               ~~~~~
    at https://deno.land/std/fs/copy.ts:90:16
```

这种问题一般是因为std的版本和deno的版本不一致的问题，目前std没有最新的1.0，可以通过添加`--unstable`标志解决。

```bash
deno run --unstable example.ts
```

2、没有文件系统的访问权限

通过添加--allow-read标志解决。