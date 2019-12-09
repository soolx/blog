---
title: 前端面试题整理
date: 2019-03-31 18:40:02
tags:
---

# 前端面试题整理

## JS

### call, apply, bind区别

当函数需要传递多个变量时, apply 可以接受一个数组作为参数输入, call 则是接受一系列的单独变量。

Bind和call很相似，第一个参数是this的指向，从第二个参数开始是接收的参数列表。

bind返回对应函数, 便于稍后调用； apply, call则是立即调用。

### var是否可以省略

一般情况下，是可以省略var的，但有两点值得注意：

1. `var a=1`与`a=1`，这两条语句一般情况下作用是一样的。但是前者不能用delete删除。不过，绝大多数情况下，这种差异是可以忽略的。
2. 在函数内部，如果没有用`var`进行申明，则创建的变量是全局变量，而不是局部变量了。

所以，建议变量申明加上`var`关键字。

### 变量提升

JavaScript引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。

示例：

```
console.log(a);
var a =1;
```

以上语句并不会报错，只是提示`undefined`。实际运行过程：

```
var a;
console.log(a);
a =1;
```

表示变量a已声明，但还未赋值。但是变量提升只对var命令声明的变量有效，如果一个变量不是用var命令声明的，就不会发生变量提升。

```
console.log(aa);
aa =1;
```

以上代码将会报错:`ReferenceError: aa is not defined`。

与普通变量一样，js里的function也可看做变量，也存在变量提升情况：

```
a();

function a(){
    console.log(1);
};
```

表面上，上面代码好像在声明之前就调用了函数a。但是实际上，由于“变量提升”，函数a定义部分被提升到了代码头部，也就是在调用之前已经声明了。但是，如果采用赋值语句定义函数，JavaScript就会报错：

```
a();

var a = function(){
    console.log(1);
};

// TypeError: a is not a function
```

因为，实际运行过程：

```
var a;
a();

a = function(){
    console.log(1);
};
```

这时候a是个变量，并非function。

### this的作用域

#### 普通函数和"this"

this 关键字的价值完全取决于它的函数（或方法）是如何被调用的。this 可以是以下任何内容：

1. 新的对象

如果函数使用`new`被调用：

```
const mySundae = new Sundae('Chocolate', ['Sprinkles', 'Hot Fudge']);
```

在上述代码中，Sundae 这个构造函数内的 this 的值是新的对象，因为它使用 new 被调用。

2. 指定的对象

如果函数使用 call/apply 被调用：

```
const result = obj1.printName.call(obj2);
```

在上述代码中，printName() 中的 this 的值将指的是 obj2，因为 call() 的第一个参数明确设定 this 指代的是什么。

3.  上下文对象

如果函数是对象方法：

```
data.teleport();
```

在上述代码中，teleport() 中的 this 的值将指代 data。

4. 全局对象或 undefined

如果函数被调用时没有上下文：

```
teleport();
```

在上述代码中，teleport() 中的 this 的值是全局对象，如果在严格模式下，是 undefined。

#### 箭头函数和"this"

对于普通函数，this 的值基于函数如何被调用。对于箭头函数，this 的值基于函数周围的上下文。换句话说，箭头函数内的，this 的值与函数外面的 this 的值一样。

我们先看一个普通函数中的 this 示例，再看一个箭头函数是如何使用 this 的。

```
// constructor
function IceCream() {
  this.scoops = 0;
}

// adds scoop to ice cream
IceCream.prototype.addScoop = function() {
  setTimeout(function() {
    this.scoops++;
    console.log('scoop added!');
  }, 500);
};

const dessert = new IceCream();
dessert.addScoop();
```

> Prints: scoop added!

运行上述代码后，你会认为半毫秒之后，dessert.scoops 会是1。但并非这样：

```
console.log(dessert.scoops);
```

> Prints: 0

能说说原因吗？

传递给 setTimeout() 的函数被调用时没用到 new、call() 或 apply()，也没用到上下文对象。意味着函数内的 this 的值是全局对象，不是 dessert 对象。实际上发生的情况是，创建了新的 scoops 变量（默认值为 undefined），然后递增（undefined + 1 结果为 NaN）：

```
console.log(scoops);
```

> Prints: NaN

解决这个问题的方法之一是使用闭包：

```
// constructor
function IceCream() {
  this.scoops = 0;
}

// adds scoop to ice cream
IceCream.prototype.addScoop = function() {
  const cone = this; // sets `this` to the `cone` variable
  setTimeout(function() {
    cone.scoops++; // references the `cone` variable
    console.log('scoop added!');
  }, 0.5);
};

const dessert = new IceCream();
dessert.addScoop();
```

上述代码将可行，因为它没有在函数内使用 this，而是将 cone 变量设为 this，然后当函数被调用时查找 cone 变量。这样可行，因为使用了函数外面的 this 值。如果现在查看甜点中的勺子数量，正确值将为 1：

```
console.log(dessert.scoops);
```

> Prints: 1

这正是箭头函数的作用，我们将传递给 setTimeout() 的函数替换为箭头函数：

```
// constructor
function IceCream() {
  this.scoops = 0;
}

// adds scoop to ice cream
IceCream.prototype.addScoop = function() {
  setTimeout(() => { // an arrow function is passed to setTimeout
    this.scoops++;
    console.log('scoop added!');
  }, 0.5);
};

const dessert = new IceCream();
dessert.addScoop();
```

因为箭头函数从周围上下文继承了 this 值，所以这段代码可行！

```
console.log(dessert.scoops);
```

> Prints: 1

当 addScoop() 被调用时，addScoop() 中的 this 的值指的是 dessert。因为箭头函数被传递给 setTimeout()，它使用周围上下文判断它里面的 this 指的是什么。因为箭头函数外面的 this 指的是 dessert，所以箭头函数里面的 this 的值也将是 dessert。

如果我们将 addScoop() 方法改为箭头函数，你认为会发生什么？

```
// constructor
function IceCream() {
    this.scoops = 0;
}

// adds scoop to ice cream
IceCream.prototype.addScoop = () => { // addScoop is now an arrow function
  setTimeout(() => {
    this.scoops++;
    console.log('scoop added!');
  }, 0.5);
};

const dessert = new IceCream();
dessert.addScoop();
```

这段代码因为同一原因而不起作用，即箭头函数从周围上下文中继承了 this 值。在 addScoop() 方法外面，this 的值是全局对象。因此如果 addScoop() 是箭头函数，addScoop() 中的 this 的值是全局对象。这样的话，传递给 setTimeout() 的函数中的 this 的值也设为了该全局对象！

### ES6新特性

- Let and Const
- 模板字面量
- 结构
- 对象字面量简写法
- 迭代 For...of循环
- 展开...运算符
- ...剩余参数
- 箭头函数
- 默认函数参数
- 默认值和解构
- 类
- super and extends
- Symbol
- Set
- WeakSet
- Map
- WeakMap
- Promise
- Proxy
- 生成器和迭代器
- Polyfill
- 转译 Babel



### 对象的拷贝

- 浅拷贝: 以赋值的形式拷贝引用对象，仍指向同一个地址，**修改时原对象也会受到影响**

  - `Object.assign`
  - 展开运算符(...)

- 深拷贝: 完全拷贝一个新对象，**修改时原对象不再受到任何影响**

  - ```
    JSON.parse(JSON.stringify(obj))
    ```

    : 性能最快 

    - 具有循环引用的对象时，报错
    - 当值为函数、`undefined`、或`symbol`时，无法拷贝

  - 递归进行逐一赋值



### 代码的复用

当你发现任何代码开始写第二遍时，就要开始考虑如何复用。一般有以下的方式:

- 函数封装
- 继承
- 复制`extend`
- 混入`mixin`
- 借用`apply/call`



### 原型链

**原型链是由原型对象组成**，每个对象都有 `__proto__` 属性，指向了创建该对象的构造函数的原型，`__proto__` 将对象连接起来组成了原型链。是一个用来**实现继承和共享属性**的有限的对象链。

- **属性查找机制**: 当查找对象的属性时，如果实例对象自身不存在该属性，则沿着原型链往上一级查找，找到时则输出，不存在时，则继续沿着原型链往上一级查找，直至最顶级的原型对象`Object.prototype`，如还是没找到，则输出`undefined`；
- **属性修改机制**: 只会修改实例对象本身的属性，如果不存在，则进行添加该属性，如果需要修改原型的属性时，则可以用: `b.prototype.x = 2`；但是这样会造成所有继承于该对象的实例的属性发生改变。

### 闭包

闭包属于一种特殊的作用域，称为 **静态作用域**。它的定义可以理解为: **父函数被销毁** 的情况下，返回出的子函数的`[[scope]]`中仍然保留着父级的单变量对象和作用域链，因此可以继续访问到父级的变量对象，这样的函数称为闭包。

- 闭包会产生一个很经典的问题:
  - 多个子函数的`[[scope]]`都是同时指向父级，是完全共享的。因此当父级的变量对象被修改时，所有子函数都受到影响。
- 解决:
  - 变量可以通过 **函数参数的形式** 传入，避免使用默认的`[[scope]]`向上查找
  - 使用`setTimeout`包裹，通过第三个参数传入
  - 使用 **块级作用域**，让变量成为自己上下文的属性，避免共享



### 防抖与节流

防抖与节流函数是一种最常用的 **高频触发优化方式**，能对性能有较大的帮助。

- **防抖 (debounce)**: 将多次高频操作优化为只在最后一次执行，通常使用的场景是：用户输入，只需再输入完成后做一次输入校验即可。

```js
function debounce(fn, wait, immediate) {
    let timer = null

    return function() {
        let args = arguments
        let context = this

        if (immediate && !timer) {
            fn.apply(context, args)
        }

        if (timer) clearTimeout(timer)
        timer = setTimeout(() => {
            fn.apply(context, args)
        }, wait)
    }
}
```

- **节流(throttle)**: 每隔一段时间后执行一次，也就是降低频率，将高频操作优化成低频操作，通常使用场景: 滚动条事件 或者 resize 事件，通常每隔 100~500 ms执行一次即可。

```js
function throttle(fn, wait, immediate) {
    let timer = null
    let callNow = immediate
    
    return function() {
        let context = this,
            args = arguments

        if (callNow) {
            fn.apply(context, args)
            callNow = false
        }

        if (!timer) {
            timer = setTimeout(() => {
                fn.apply(context, args)
                timer = null
            }, wait)
        }
    }
}
```



## 浏览器

### 跨标签页通讯

不同标签页间的通讯，本质原理就是去运用一些可以 **共享的中间介质**，因此比较常用的有以下方法:

- 通过父页面`window.open()`和子页面`postMessage`
  - 异步下，通过 `window.open('about: blank')` 和 `tab.location.href = '*'`
- 设置同域下共享的`localStorage`与监听`window.onstorage`
  - 重复写入相同的值无法触发
  - 会受到浏览器隐身模式等的限制
- 设置共享`cookie`与不断轮询脏检查(`setInterval`)
- 借助服务端或者中间层实现

### Web Worker

### 内存泄露

- 意外的**全局变量**: 无法被回收
- **定时器**: 未被正确关闭，导致所引用的外部变量无法被释放
- **事件监听**: 没有正确销毁 (低版本浏览器可能出现)
- **闭包**: 会导致父级中的变量无法被释放
- **dom 引用**: dom 元素被删除时，内存中的引用未被正确清空

可用 chrome 中的 timeline 进行内存标记，可视化查看内存的变化情况，找出异常点。



## 服务端

### Web Socket

### 跨域

- JSONP: 利用`<script>`标签不受跨域限制的特点，缺点是只能支持 get 请求

```
function jsonp(url, jsonpCallback, success) {
  const script = document.createElement('script')
  script.src = url
  script.async = true
  script.type = 'text/javascript'
  window[jsonpCallback] = function(data) {
    success && success(data)
  }
  document.body.appendChild(script)
}
复制代码
```

- 设置 CORS: Access-Control-Allow-Origin：*
- postMessage

### 安全

- XSS攻击: 注入恶意代码
  - cookie 设置 httpOnly
  - 转义页面上的输入内容和输出内容
- CSRF: 跨站请求伪造，防护:
  - get 不修改数据
  - 不被第三方网站访问到用户的 cookie
  - 设置白名单，不被第三方网站请求
  - 请求校验



## 算法

### 快速排序

```javascript
function quickSort(arr){
    if (arr.length<=1){
        return arr;
    }
   var pivotIndex = 0,
        pivort = arr.splice(pivortIndex, 1)[0];
    var left = [],
        right = [];
    for (var i = 1, length = arr.length; i < length; i++) {
        if (arr[i] < pivort) {
            left.push(arr[i]);
        }else if (arr[i] > = pivort) {
            right.push(arr[i]);
        }
    }
   return quickSort(left).concat([pivort], quickSort(right));
}
```

### 深克隆

```javascript
function deepClone (test) {
    if (Array.isArray(test)) {
        var arr = [];
       for (var i = 0, length = test.length; i < length; i++) {
               arr.push(clone(arr[i]));
       }
    }else if (typeOf test === "object") {
        var obj = {};
       for (var j in test) {
               obj[j] = clone(test[j]);
       }
       return obj;
    }else {
        return test;
    }
}
```



参考资料

https://juejin.im/post/5c64d15d6fb9a049d37f9c20

