---
layout:     post
title:      async和await
subtitle:   
date:       2023-8-13
author:     sq
header-img: 
catalog: true
tags:
    - async
    - await
---
# async和await
## async
async函数返回一个 Promise 对象。函数体内一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。

### Promise 对象
async函数返回的 Promise 对象，必须等到内部所有await命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到return语句或者抛出错误。

async函数内部return语句返回的值，会成为then方法回调函数的参数。

async函数内部抛出错误，会导致返回的 Promise 对象变为reject状态。

## await
await命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。

```javascript
async function f() {
  // 等同于
  // return 123;
  return await 123;
}

```

另一种情况是，await命令后面是一个thenable对象（即定义了then方法的对象），那么await会将其视为Promise处理。

例如：
```javascript
class Sleep {
  constructor(timeout) {
    this.timeout = timeout;
  }
  then(resolve, reject) {
    const startTime = Date.now();
    setTimeout(
      () => resolve(Date.now() - startTime),
      this.timeout
    );
  }
}

(async () => {
  const sleepTime = await new Sleep(1000);
  console.log(sleepTime);
})();
// 1000
```

## 错误处理
任何一个await语句后面的 Promise 对象变为reject状态，那么整个async函数都会中断执行。

如果即使reject也要继续执行，把可能报错的代码放在try catch中。或者在 await 后面的 Promise 对象再写一个catch方法，处理前面可能出现的错误。

## 执行顺序
### 顺序执行
for await of 和 for 循环里面用 await 都可以实现按顺序执行，还有下面两种

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}

async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 最后一个迭代函数返回了promise，所以这里需要await
  await docs.reduce(async (_, doc) => {
    // 迭代函数是async，所以这里是前一步返回的promise
    await _;
    await db.post(doc);
  }, undefined);
}
```

并发调接口，继发输出
```javascript
async function logInOrder(urls) {
  // 并发读取远程URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序输出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
```

如果await后面的异步操作出错，那么等同于async函数返回的 Promise 对象被reject。

### 同时执行
没有执行顺序要求的，应该同时执行

```javascript
// 顺序执行
let foo = await getFoo();
let bar = await getBar();

// 改成同时执行
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
// TODO 不知道原理
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的写法
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```

## async 函数可以保留运行堆栈

```javascript
const a = () => {
  // b 或 c报错，错误堆栈不包括a（存疑，在谷歌上有的，可能旧版本或其他浏览器没有吧）
  b().then(() => c());
};
```

这样错误堆栈会包括a
```javascript
const a = async () => {
  await b();
  c();
};
```

## 顶层 await
es2022开始，允许顶层await。之前的解决方法是模块导出一个promise，通过promise判断模块的异步操作是否完成。

```javascript
// awaiting.js
const dynamic = import(someMission);
const data = fetch(url);
export const output = someProcess((await dynamic).default, await data);
```

上面代码中，两个异步操作在输出的时候，都加上了await命令。只有等到异步操作完成，这个模块才会输出值。

加载这个模块的写法如下。
```javascript
// usage.js
import { output } from "./awaiting.js";
function outputPlusValue(value) { return output + value }

console.log(outputPlusValue(100));
setTimeout(() => console.log(outputPlusValue(100)), 1000);
```

模块的加载会等待依赖模块（上例是awaiting.js）的异步操作完成，才执行后面的代码。

顶层await只能用在 ES6 模块，不能用在 CommonJS 模块。这是因为 CommonJS 模块的require()是同步加载。

如果加载多个包含顶层await命令的模块，加载命令是同步执行的。
