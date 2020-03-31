---
title: WebSocket实现设计
date: 2020-03-31 14:19:13
tags:
---

# WebSocket实现设计

WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。

有时会遇到服务器需要主动推动消息的需求，或者有频繁的少量消息需要交换。因为HTTP请求的头部可能很长，这种场景下，会带来很多的带宽损耗。

WebSocket 协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。

## 使用

### 最简单的例子

```javascript
const ws = new WebSocket("ws://localhost:8080");
ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log("Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};      
```

### ws和wss

ws表示纯文本通信，而wss表示使用加密信道通信(TCP+TLS)。

- **ws协议**：普通请求，占用与HTTP相同的80端口
- **wss协议**：基于SSL的安全传输，占用与TLS相同的443端口。

## 从WebSocket到Socket.IO

简单来说Socket.IO就是对WebSocket的封装，并且实现了WebSocket的服务端代码。Socket.IO将WebSocket和轮询（Polling）机制以及其它的实时通信方式封装成了通用的接口，并且在服务端实现了这些实时机制的相应代码。也就是说，WebSocket仅仅是Socket.IO实现实时通信的一个子集。

Socket.IO简化了WebSocket API，统一了返回传输的API，具有更好的兼容性。

### Socket.IO传输种类

- WebSocket
- Flash Socket
- AJAX long-polling
- AJAX multipart streaming
- IFrame
- JSONP polling

### 例子

```javascript
const instance = io(url);
// 连接成功
instance.on('connect', () => {
  console.log('Socket connect success');
});
// error
instance.on('error', (error) => {
  console.log('Socket error', error);
});
// 断开连接
instance.on('disconnect', (info) => {
  console.log('Socket disconnect', info);
});
// 重联
instance.on('reconnect', (attemptNumber) => {
  console.log('Socket reconnect', attemptNumber);
});
// 监听channel
instance.on(channel, (message) => {
  // handle message
});
```

## WebSocket封装实践

有时我们可能在业务中会有大量的ws使用需求，如果各自处理的话会很混乱，这时候可以采用类似于封装request工具的方式来进行处理。

### 常见场景

* 发送一条消息，需要在返回对应结果时进行响应处理，且一次性返回结果。实际的例子有导出报表，生成文件后返回url等。
* 发送一条消息，需要在返回对应结果时进行响应处理，且多次返回结果。实际的例子有行情推送，运行状态监控等。
* 直接监听channel，等待数据推送持续处理。实际的例子有消息通知等。
* 直接发送消息，不需要返回或不需要处理返回结果。

### 要解决的问题

* 统一使用方式，避免业务层处理socket的连接，断开，重联等复杂情况
* 提供丰富的消息处理机制，满足常见场景
* 对连接异常提供重连机制，保证连接稳定
* 重连后可以保持之前的消息处理继续生效
* 对复杂场景提供更灵活的使用方式，暴露实例以满足业务需求

### 设计思想

通过单例模式实现websocket的实例，随页面加载初始化。

封装API覆盖常见场景，简化操作流程。通过参数调整是否持续处理，提供处理机制等。

因为ws的消息跟ajax请求不同，不是一一对应的。所以对于前端主动请求的结果，可以前端生成唯一标识作为id，随消息发送，后台返回的时候带上对应的id。

断开后主动重连，恢复断开前状态，重新监听之前的channel，收到消息继续之前的处理机制。

自动管理channel的监听，消息使用新的channel和重联时，自动开启。

### 框架设计

```javascript
export class SocketFactory {
  constructor() {
    this.instance = null;
    this.handleMQ = {};
    this.handleMQPersist = {};
    this.noIdChannelList = {};
    this.errorCallback = {};
  }
	// 初始化
  init = () => {};
	// 关闭实例
  close = () => {};
	// 暴露实例
  getSocket = () => {
    return this.instance;
  };
	// 错误处理
  showError = (channel, msg, requestId) => {};
	// 消息处理
  handleMessage = (channel, msg, noId) => {};
	// 判定是否开启通道监听
  listenChannel = (channel, noId) => {
    if (condition) { this.openListenChannel(channel, noId) }
    else { ... }
  };
	// 开启通道监听
  openListenChannel = (channel, noId) => {};
	// 发送消息，并绑定后续的处理逻辑
  sendMessage = (channel, message, handleFun, persist, errorCallback) => {};
	// 直接监听某条消息
  listenMessage = (channel, messageId, handleFun, persist) => {};
	// 直接监听通道
  listenMessageNoId = (channel, messagetype, handleFun, persist) => {};
	// 停止通道或者消息的监听
  stopListenMessage = (channel, messageId) => {};
}
// 单例模式
const GlobalSocket = new SocketFactory();
window.GlobalSocket = GlobalSocket;
export default GlobalSocket;

```

### 发送消息的设计

```javascript
sendMessage = (channel, message, handleFun, persist, errorCallback) => {
  const uuid = UUID(); // 生成唯一标识
  this.listenChannel(channel); // 通知需要监听通道
  if (handleFun) { // 绑定处理函数
    this.handleMQ[channel][uuid] = handleFun;
  }
  if (persist) { // 是否需要持续处理
    this.handleMQPersist[channel][uuid] = true;
  }
  if (errorCallback) { // 是否需要特殊的错误处理
    this.errorCallback[uuid] = errorCallback;
  }
  this.instance.emit(channel, { requestId: uuid, content: message }); // 发送消息
}
```

### 处理消息的设计

```javascript
handleMessage = (channel, msg, noId) => {
    if (this.handleMQ[channel]) { // 判定是否处理消息
      if (noId) { // 监听通道的处理，如果消息通知
        for (const type of Object.keys(this.handleMQ[channel])) {
          if (msg.code === 0) {
            this.handleMQ[channel][type](msg.data);
            if (!(this.handleMQPersist[channel] && this.handleMQPersist[channel][type])) {
              delete this.handleMQ[channel][type];
            }
          } else if (!(this.handleMQPersist[channel]
            && this.handleMQPersist[channel][type])) {
            this.showError(channel, msg);
            delete this.handleMQ[channel][type];
          } else {
            this.showError(channel, msg);
          }
        }
      } else if (this.handleMQ[channel][msg.requestId]) { // 业务的处理
        if (msg.code === 0) { // 正常数据
          this.handleMQ[channel][msg.requestId](msg.data);
          if (!(this.handleMQPersist[channel] && this.handleMQPersist[channel][msg.requestId])) {
            delete this.handleMQ[channel][msg.requestId]; // 移除一次处理的任务
          }
        } else if (!(this.handleMQPersist[channel] // 非持续处理任务，发生错误后移除
          && this.handleMQPersist[channel][msg.requestId])) {
          this.showError(channel, msg, msg.requestId);
          delete this.handleMQ[channel][msg.requestId];
        } else {
          this.showError(channel, msg, msg.requestId); // 持续处理任务，发生错误后保持
        }
      }
    }
  }
```

### 重连后的设计

重连之后需要恢复之前的状态，需要在连接后判断需要开启的通道。

```javascript
this.instance.on('connect', () => {
  console.log('Socket connect success');
  for (const channel of Object.keys(this.handleMQ)) {
    if (this.noIdChannelList[channel]) {
      this.openListenChannel(channel, true);
    } else {
      this.openListenChannel(channel);
    }
  }
});
```

因为重连之后某些通道可能会自动开启，为了避免重复监听导致处理两次，需要判断。

```javascript
openListenChannel = (channel, noId) => {
	if (this.instance && !this.instance['_callbacks'][`$${channel}`]) {
    this.instance.on(channel, (message) => {
    	this.handleMessage(channel, message, noId);
    });
    console.log(channel, 'listen');
  }
}
```

