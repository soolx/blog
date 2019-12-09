---
title: react-hooks
date: 2019-02-28 20:48:01
tags:
---

# 2019-02-soolx-React Hooks笔记

<a name="320ddf96"></a>

# React Hooks
React-Hooks于React 16.8版本正式发布。它为不编写类的情况下，为组件实现state管理和其他React功能提供了可能。

<a name="31e27f59"></a>
## Hook的优势
* 可以方便的在组件中复用逻辑
* 可以更加灵活的拆分组件

<a name="998067ef"></a>
## Hook 规则
* 只在顶层调用Hooks，不要在循环，条件或者嵌套函数中调用
* 仅从React组件或者自定义Hook中调用Hook

<a name="c161b609"></a>
## Hooks类型
<a name="77cb6d35"></a>
### 基本Hooks
* useState
* useEffect
* useContent
  <a name="38974c9e"></a>
### 进阶Hooks
* useReducer
* useCallback
* useMemo
* useRef
* useImperativeHandle
* useLayoutEffect
* useDebugValue

<a name="ecff77a8"></a>
## 使用
<a name="useState"></a>
### useState
<a name="704f29e0"></a>
#### 基本用法
```javascript
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
通过`useState`声明一个状态变量，array的第一项是我们要声明的状态变量，第二项是可以用来改变该变量的方法。

使用多个状态变量时：
```javascript
const [age, setAge] = useState(42);
const [fruit, setFruit] = useState('banana');
const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
```

<a name="e5389bcb"></a>
#### Lazy initial state
如果state初始值的计算成本比较高，可以采用function的形式，这样初始值只会在初始化阶段计算一次

```javascript
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

<a name="useEffect"></a>
### useEffect
<a name="704f29e0-1"></a>
#### 基本用法

```javascript
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Effect Hook用来实现类似于react生命周期的方法。<br />useEffect Hook的作用就是componentDidMount, componentDidUpdate和componentWillUnmount的结合，会在首次渲染和组件更新之后执行。

<a name="11254f34"></a>
#### 清理阶段
Effect Hook还可以return一个方法，实现清理阶段的逻辑：

```javascript
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```
return的该方法会在组件卸载和每次运行这个effect之前执行，不同于componentWillUnmount。

useEffect可以根据实际的使用情况，多次调用，拆分逻辑。

```javascript
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
}
```

useEffect还可以使用第二个参数来跳过效果，优化性能

```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```
该例子只有在count发生变更的时候才会执行，效果类似于

```javascript
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

跳过效果也适用于清理阶段：

```javascript
useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Only re-subscribe if props.friend.id changes
```

注意，第二个参数数组中的值，请使用会随时间变化，且可以被effect使用的值。

<a name="576e7847"></a>
#### 基于前一次的状态更新

```javascript
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
    </>
  );
}
```

<a name="useLayoutEffect"></a>
### useLayoutEffect
和 `useEffect` 相同，但在所有 DOM 修改结束后同步执行。

<a name="useContent"></a>
### useContent

```javascript
const context = useContext(Context);
```
context 变化时，hook 会触发一次 rerender。

<a name="useReducer"></a>
### useReducer

```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init);

// 传初始值
const [state, dispatch] = useReducer(reducer, { count: initialCount });

// lazy 初始值
function init(initialArg) {
  return { count: initialArg };
}
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

reducer 即 redux 里的 reducer，`(sate, action) => newState`。

有复杂的 state 逻辑时使用，

* 多个子数据
* 返回的 state 依赖之前的 state

把 dispatch 通过 context 往下传，可解决深层组件的更新问题。（而不是把 callback 传下去）<br />useReducer 不会像 Redux 一样，把 reducer 执行一遍返回初始值，但可通过 `useReducer(reducer, undefined, reducer)` 实现。<br />reducer 返回相同的值时，不会触发更新，值匹配基于 [Object.is 比较算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description)。

<a name="useRef"></a>
### useRef

```javascript
const refContainer = useRef(initialValue);
```

<a name="af75a73a"></a>
#### 常见用法
```javascript
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

<a name="453f05b5"></a>
#### 特殊用法
useRef不只是用作DOM绑定。ref对象是一个通用容器，其`current`属性是可变的，可以保存任何值.

```javascript
function Timer() {
  const intervalRef = useRef();

  useEffect(() => {
    const id = setInterval(() => {
      // ...
    });
    intervalRef.current = id;
    return () => {
      clearInterval(intervalRef.current);
    };
  });

  // ...
}
```
<a name="d41d8cd9"></a>
### 
<a name="useDebugValue"></a>
### useDebugValue

```javascript
useDebugValue(value)

// 延迟format，直到查看时执行
useDebugValue(date, date => date.toDateString());
```
用于在React DevTools中显示自定义Hook的label<br /><br />
<a name="d17a0f0b"></a>
## 参考
* [https://reactjs.org/docs/hooks-intro.html](https://reactjs.org/docs/hooks-intro.html)
* [https://www.yuque.com/f2e-growth/notes/xfg0tk](https://www.yuque.com/f2e-growth/notes/xfg0tk)