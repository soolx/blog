---
title: 前端监控
date: 2020-04-28 14:46:13
tags:
---

# 前端监控系统踩坑（一）

随着系统的功能不断的增加，慢慢就会有对各种指标、报错以及用户行为进行统计分析的需求产生。

通过分析采集的数据，来了解用户的需求，更有侧重的去完善功能，以便更好的贴近用户。



## 为什么需要前端监控系统

* 不了解哪些功能是用户高频使用的，哪些是无人问津的
* 有些功能的稳定性较差，需要进行异常记录，方便排查bug甚至故障回放
* 记录页面的性能指标，接口调用时间，加载时间等，有针对性的进行优化

## 数据监控

数据监控主要是用户记录用户行为。

主要包括：

- PV/UV:PV(page view)，即页面浏览量或点击量。UV:指访问某个站点的人数

- 用户在每一个页面的停留时间

- 用户通过什么入口来访问该网页

- 用户在相应的页面中触发的行为

  

  通过记录并分析用户的行为，来确定功能迭代的优先级。给后续功能的增删提供有价值的参考指标。

## 异常监控

此外，产品的前端代码在执行过程中也会发生异常，因此需要引入异常监控。及时的上报异常情况，可以避免线上故障的发上。虽然大部分异常可以通过try catch的方式捕获，但是比如内存泄漏以及其他偶现的异常难以捕获。

主要包括：

- 接口异常
- 资源加载异常
- 代码执行异常
- 样式丢失的异常监控

## 性能监控

性能监控指的是监听前端的性能，主要包括监听网页或者说产品在用户端的体验。

主要包括：

* UA记录

- 不同用户，不同机型和不同系统下的首屏加载时间
- 白屏时间
- http等请求的响应时间
- 静态资源整体下载时间
- 页面渲染时间
- 页面交互动画完成时间

## 常用前端埋点方案

### 代码埋点

代码埋点，就是以嵌入代码的形式进行埋点，比如需要监控用户的点击事件，会选择在用户点击时，插入一段代码，保存这个监听行为或者直接将监听行为以某一种数据格式直接传递给server端。此外比如需要统计产品的PV和UV的时候，需要在网页的初始化时，发送用户的访问信息等。

优势：可以精细化的进行埋点

缺点：开发工作量较大，而且对业务代码有很大的侵入性



### 可视化埋点

通过可视化交互的手段，代替代码埋点。将业务代码和埋点代码分离，提供一个可视化交互的页面，输入为业务代码，通过这个可视化系统，可以在业务代码中自定义的增加埋点事件等等，最后输出的代码耦合了业务代码和埋点代码。

可视化埋点听起来比较高大上，实际上跟代码埋点还是区别不大。也就是用一个系统来实现手动插入代码埋点的过程。

优点：相对方便。

缺点：可视化埋点可以埋点的控件有限，不能手动定制，代码变更后可能会失效。

### 无埋点（全埋点）

无埋点并不是说不需要埋点，而是全部埋点，前端的任意一个事件都被绑定一个标识，所有的事件都别记录下来。通过定期上传记录文件，配合文件解析，解析出来我们想要的数据。

从语言层面实现无埋点也很简单，比如从页面的js代码中，找出dom上被绑定的事件，然后进行全埋点。

优点：

- 由于采集的是全量数据，所以产品迭代过程中是不需要关注埋点逻辑的，也不会出现漏埋、误埋等现象

缺点：

- 无埋点采集全量数据，给数据传输和服务器增加压力
- 无法灵活的定制各个事件所需要上传的数据

## 实现

考虑自研一个前端监控系统，先实现数据监控和性能监控，异常暂时使用Sentry实现。

第一个版本先实现接口调用的统计和PV记录，优先跑通流程，暂时不考虑鉴权和性能的问题。

![](http://resource.soolx.top/image/png/Xnip2020-04-28_16-00-24.png)

如图，主要包含三个部分。

数据存储暂时使用postgres，后续根据使用情况可能考虑采用mongodb进行采集数据的存储，采用MQ来解决高并发的写入压力。

因为现在的系统大多是SPA，使用了前端路由进行管理。前端路由主要包含两种方案，hash和history。

hash方案可以通过监听hashchange实现监控

```javascript
window.addEventListener("hashchange",function(event){
  console.log(event.oldURL, event.newURL)
}, false);
```

低版本做兼容的话大概是这样

```javascript
(function(window) {

  // 如果浏览器原生支持该事件,则退出  
if ( "onhashchange" in window.document.body ) { return; }

  var location = window.location,
    oldURL = location.href,
    oldHash = location.hash;

  // 每隔100ms检测一下location.hash是否发生变化
  setInterval(function() {
    var newURL = location.href,
      newHash = location.hash;

    // 如果hash发生了变化,且绑定了处理函数...
    if ( newHash != oldHash && typeof window.onhashchange === "function" ) {
      // execute the handler
      window.onhashchange({
        type: "hashchange",
        oldURL: oldURL,
        newURL: newURL
      });

      oldURL = newURL;
      oldHash = newHash;
    }
  }, 100);

})(window);
```



history可以通过监听popstate实现监控

```javascript
window.addEventListener('popstate', (event) => {
  console.log("location: " + document.location + ", state: " + JSON.stringify(event.state));
});
```

实践过程中有发现跟umi配置使用的时候，遇到监听不到Link跳转的问题，怀疑是react-router使用了history的polyfill的问题，暂时用了很low的方法进行测试

```javascript
history.pushState = function(state, title, url, first){
    if (!first){console.log('record', url);
    history.pushState.apply(history, [state, title, url, true]);}
}
history.replaceState = function(state, title, url, first){
    if (!first){console.log('record', url);
    history.replaceState.apply(history, [state, title, url, true]);}
}
```

未完待续