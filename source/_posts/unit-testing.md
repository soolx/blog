---
title: 单元测试
date: 2019-03-31 10:23:34
tags:
---

# React单元测试笔记



## 什么是单元测试

单元测试（unit testing），是指对软件中的最小可测试单元进行检查和验证。



## React常用的单元测试工具

Jest+Enzyme



## Jest使用

### 安装

用yarn安装jest:
`yarn add --dev jest`

或者npm:

`npm install --save-dev jest`

使用Babel

通过`yarn`安装依赖:

```bash
yarn add --dev babel-jest @babel/core @babel/preset-env
```

在 `babel.config.js`文件里添加配置:

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: 'current',
        },
      },
    ],
  ],
};
```



### 例子

首先创建一个 `sum.js`文件:

```javascript
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

然后创建 `sum.test.js`:

```javascript
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

添加这部分代码到 `package.json`:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

最后运行`yarn test` 或者 `npm run test` ，jest将会打印如下的输出:

```bash
PASS  ./sum.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```



### 基本方法

- 分组（Test Group）：`describe(name, fn)`

  `describe`创建一个将几个相关测试组合在一起的块，这不是必须的，而且可以嵌套。

  ```javascript
  const myBeverage = {
    delicious: true,
    sour: false,
  };
  
  describe('my beverage', () => {
    test('is delicious', () => {
      expect(myBeverage.delicious).toBeTruthy();
    });
  
    test('is not sour', () => {
      expect(myBeverage.sour).toBeFalsy();
    });
  });
  ```

  

- 测试用例（Test Case）：`test(name, fn, timeout)`

  test是jest.It的别名。

  test用来编写测试方法。

  ```javascript
  test('did not rain', () => {
    expect(inchesOfRain()).toBe(0);
  });
  ```

- 断言（Assert）：`expect(value).toBe(value)`

  对结果进行断言。

  expect判定条件

  toBe匹配器

  not判定相反的结果

  ```js
  expect(2 + 2).toBe(4);
  expect(2 + 2).not.toBe(5);
  ```



### 匹配器(Matchers)

#### 普通匹配器

最简单的测试值的方法是看是否精确匹配。

```javascript
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

#### Truthiness

在测试中，你有时需要区分 `undefined`、 `null`，和 `false`，但有时你又不需要区分。 Jest 让你明确你想要什么。

- `toBeNull` 只匹配 `null`

- `toBeUndefined` 只匹配 `undefined`

- `toBeDefined` 与 `toBeUndefined` 相反

- `toBeTruthy` 匹配任何为真的 `if` 语句

- `toBeFalsy` 匹配任何 为假的`if` 语句

  

```js
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```

  

#### 数字

多种比较数值的匹配器。

```js
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe and toEqual are equivalent for numbers
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```

比较浮点数相等时，使用`toBeCloseTo`而不是`toEqual`，可以避免舍入误差导致的测试错误。

```js
test('两个浮点数字相加', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);           这句会报错，因为浮点数有舍入误差
  expect(value).toBeCloseTo(0.3); // 这句可以运行
});
```



#### 字符串

可以用 `toMatch`通过正则表达式检查字符串︰

```js
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```



#### 数组和迭代器

可以用`toContain`匹配是否包含此元素。

```js
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'beer',
];

test('the shopping list has beer on it', () => {
  expect(shoppingList).toContain('beer');
  expect(new Set(shoppingList)).toContain('beer');
});
```



#### 例外

如果你想要测试的特定函数抛出一个错误，在它调用时，使用 `toThrow`。

```js
function compileAndroidCode() {
  throw new ConfigError('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(compileAndroidCode).toThrow();
  expect(compileAndroidCode).toThrow(ConfigError);

  // You can also use the exact error message or a regexp
  expect(compileAndroidCode).toThrow('you are using the wrong JDK');
  expect(compileAndroidCode).toThrow(/JDK/);
});
```



#### 更多

[参考文档](https://jestjs.io/docs/zh-Hans/expect)



### 测试 React Apps

使用Create React App安装

> Create React App 初始化的项目预置了Jest，只需要添加react-test-renderer`来渲染快照.

不使用Create React App安装

>如果现有项目不是通过Creact React App创建，则需要添加一些依赖来搭配使用

```bash
yarn add --dev jest babel-jest @babel/preset-env @babel/preset-react react-test-renderer
```

> 添加这些配置到相应的文件，`current-version`为当前依赖的最新版本号

```json
// package.json
  "dependencies": {
    "react": "<current-version>",
    "react-dom": "<current-version>"
  },
  "devDependencies": {
    "@babel/preset-env": "<current-version>",
    "@babel/preset-react": "<current-version>",
    "babel-jest": "<current-version>",
    "jest": "<current-version>",
    "react-test-renderer": "<current-version>"
  },
  "scripts": {
    "test": "jest"
  }
// babel.config.js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
};
```



### 搭配Enzyme使用

运行`yarn add --dev enzyme`添加Enzyme。如果您使用的是低于15.5.0的React版本，则还需要安装`react-addons-test-utils`。

例子：

```javascript
import React from 'react';
import {shallow} from 'enzyme';
import CheckboxWithLabel from '../CheckboxWithLabel';

test('CheckboxWithLabel changes the text after click', () => {
  // Render a checkbox with label in the document
  const checkbox = shallow(<CheckboxWithLabel labelOn="On" labelOff="Off" />);
  expect(checkbox.text()).toEqual('Off');
  checkbox.find('input').simulate('change');
  expect(checkbox.text()).toEqual('On');
});
```



