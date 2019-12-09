---
title: Web Worker
date: 2019-02-27 17:01:05
tags:
---

# Web Worker笔记

## 用法

### 主线程

主线程采用new命令，调用Worker()构造函数，新建一个 Worker 线程。<br />`const myWorker = new Worker(jsUrl, options);`<br />Worker()构造函数的第一个参数是一个脚本文件，该文件就是 Worker 线程所要执行的任务。Worker 不能读取本地文件，脚本必须来自网络。如果下载没有成功（比如404错误），Worker 就会默默地失败。<br />第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。

```
// 主线程
var myWorker = new Worker(‘worker.js’, { name : ‘myWorker’ });
// Worker 线程
self.name // myWorker
```

`Worker()` 构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下：

* Worker.onerror：指定 error 事件的监听函数。

```
worker.onerror(function (event) {
 console.log([
   'ERROR: Line ', e.lineno, ' in ', e.filename, ': ', e.message
 ].join(''));
});
```

* Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在Event.data属性中。

```
worker.onmessage = function (event) {
 console.log(‘Received message ‘ + event.data);
 doSomething();
}
```

* Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
* Worker.postMessage()：向 Worker 线程发送消息。

```
worker.postMessage(‘Hello World’);
worker.postMessage({method: ‘echo’, args: [‘Work’]});
```

* Worker.terminate()：立即终止 Worker 线程。

### Work线程

Web Worker 有自己的全局对象，不是主线程的window，而是一个专门为 Worker 定制的全局对象。因此定义在window上面的对象和方法不是全部都可以使用。<br />Worker 线程有一些自己的全局属性和方法：

* self.name： Worker 的名字。该属性只读，由构造函数指定。
* self.onmessage：指定message事件的监听函数。

```
self.onmessage = function (/e/) {
 self.postMessage(‘You said: ‘ + e.data);
};
```

* self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
* self.close()：关闭 Worker 线程自身。
* self.postMessage()：向产生这个 Worker 线程发送消息。
* self.importScripts()：加载 JS 脚本。

## 优缺点

### 优点

Web Worker的作用就是为javascript提供多线程环境，允许主进程将一些任务交给Worker在后台进行。这样的话，在一些计算密集型或者高延迟任务执行的时候就不会影响到主进程的执行，提高流畅度。

### 缺点

Worker一旦执行成功，就会一直运行，比较耗费资源。所以一旦只要使用完毕，就应该关闭。

## 数据通信

主线程与 Worker 之间的通信内容，可以是文本或者对象，也可以交换二进制数据，比如 File、Blob、ArrayBuffer 等类型。
### 大数据量的传递方法
需要注意的是，这种通信是拷贝关系，即是传值而不是传址，Worker 对通信内容的修改，不会影响到主线程。但是，拷贝方式发送二进制数据，会造成性能问题。为了解决这个问题，JavaScript 允许主线程把二进制数据直接转移给子线程，但是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据的方法，叫做 [Transferable Objects](http://www.w3.org/html/wg/drafts/html/master/infrastructure.html#transferable-objects) 。这使得主线程可以快速把数据交给 Worker，对于影像处理、声音处理、3D 运算等就非常方便了，不会产生性能负担。<br />下面是一个例子。

```
// Transferable Objects 格式
worker.postMessage(arrayBuffer, [arrayBuffer]);
// 例子
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```

参考文献<br />[Web Worker 使用教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)<br />[使用 Web Workers - Web API 接口参考 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers)