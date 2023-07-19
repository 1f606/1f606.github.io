---
layout:     post
title:      Generator
subtitle:   
date:       2023-3-14
author:     sq
header-img: 
catalog: true
tags:
    - Generator
---
# Generator
Generator 函数是一个状态机，封装了多个内部状态。

Generator 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

Generator 函数的语法：一是在function关键字和函数名之间有一个星号；二是，函数体内可以使用yield定义状态。

Generator 函数调用后的返回值是指向内部状态的指针对象。需要调用指针对象的next方法执行函数内部代码，从函数头部或上一次停下来的地方执行，执行遇到yield后停止。

指针对象还有 value 属性，值是 yield 后跟着的表达式返回的值，如果函数已经执行完就是 undefined；以及 done 属性表示函数是否执行完成。

Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。

yield表达式如果用在另一个表达式之中，必须放在圆括号里面。`console.log('Hello' + (yield));`

generator 是可迭代的，可以使用 iterator 的所有相关功能，例如：spread 语法 和 for of。

generator 会忽略return，next方法迭代不会忽略return。

```javascript
function * foo () {
    console.log('start');
    const bar = yield 'foo'; // 接收到了next方法传递的参数或throw方法抛出的异常
}
// 此时不会调用
const generator = foo();
const result = generator.next(); // 打印start
console.log(result);
<!--{
    value: 'foo',
    done: false
}-->
generator.next('bar');
```

```javascript
function* demo() {
  console.log('Hello' + (yield));
  console.log('Hello' + (yield 123)); 
}

let iterator = demo();

iterator.next();
//  {value: undefined, done: false}

iterator.next();
//  打印Helloundefined
// {value: 123, done: false}

iterator.next();
//  打印Helloundefined
//  {value: undefined, done: true}
```

## next方法运行逻辑

（1）遇到yield表达式，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。

（2）下一次调用next方法时，再继续往下执行，直到遇到下一个yield表达式。

（3）如果没有再遇到新的yield表达式，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。

（4）如果该函数没有return语句，则返回的对象的value属性值为undefined。

需要注意的是，yield表达式后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。

## Iterable object（可迭代对象）
任意一个对象的Symbol.iterator方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。

由于 Generator 函数就是遍历器生成函数，因此可以把 Generator 赋值给对象的Symbol.iterator属性，从而使得该对象具有 Iterator 接口。

Generator 函数执行后，返回一个遍历器对象。该对象本身也具有Symbol.iterator属性，执行后返回自身。

```javascript
const range = {
  from: 1,
  to: 5,
  // for of range 会调用一次这个方法
  [Symbol.iterator] () {
    // 函数返回的 iterator object 将用于 for of 循环，并使用 next 向它请求下一个值
    return {
      current: this.from,
      last: this.to,
      // for of 每次迭代都会调用 next
      next () {
        if (this.current <= this.last) {
          return {done: false, value: this.current++};
        } else {
          return {done: true};
        }
      }
    }
  }
}
```

上面的代码可以用 generator 函数优化：

```javascript
const range = {
  from: 1,
  to: 5,
  *[Symbol.iterator] () {
    for (let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
}

// 1,2,3,4,5
console.log([...range]);
```

## generator 组合
就是使用 yield 后接其他 generator 函数，等于将执行委托给它，在该函数里进行迭代，并将产出yield转发到外部。

例如一个生成0-9, a-z, A-Z的例子：
```javascript
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generateAlphaNum() {

  // yield* generateSequence(48, 57);
  for (let i = 48; i <= 57; i++) yield i;

  // yield* generateSequence(65, 90);
  for (let i = 65; i <= 90; i++) yield i;

  // yield* generateSequence(97, 122);
  for (let i = 97; i <= 122; i++) yield i;

}

let str = '';

for(let code of generateAlphaNum()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```

组合的 generator 的例子：
```javascript
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generatePasswordCodes() {

  // 0..9
  yield* generateSequence(48, 57);

  // A..Z
  yield* generateSequence(65, 90);

  // a..z
  yield* generateSequence(97, 122);

}

let str = '';

for(let code of generatePasswordCodes()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```

## yield是双向的
generator 和可迭代对象类似，都具有用来生成值的特殊语法。但实际上，generator 更加强大且灵活。

这是因为 yield 是一条双向路（two-way street）：它不仅可以向外返回结果，而且还可以将外部的值传递到 generator 内。

调用 generator.next(arg)，我们就能将参数 arg 传递到 generator 内部。这个 arg 参数会变成 yield 的结果。

一个例子：
```javascript
function* gen() {
  // 向外部代码传递一个问题并等待答案
  let result = yield "2 + 2 = ?"; // (*)

  alert(result);// 4
}

let generator = gen();

let question = generator.next().value; // question = '2 + 2 = ?'

generator.next(4); // --> 将结果传递到 generator 中
```

next方法（除了第一个，传了也会被忽略）可以接收参数，参数会被yield接收并作为返回值。

next 方法的调用可以是异步的。

另一个例子：
```javascript
function* gen() {
  let ask1 = yield "2 + 2 = ?";

  alert(ask1); // 4

  let ask2 = yield "3 * 3 = ?"

  alert(ask2); // 9
}

let generator = gen();

alert( generator.next().value ); // "2 + 2 = ?"

alert( generator.next(4).value ); // "3 * 3 = ?"

alert( generator.next(9).done ); // true
```

## Generator异步方案
```javascript
function * main () {
    try {
        const users = yield axios('/api/user');
        console.log(users);
        
        const posts = yield axios('/api/post');
        console.log(posts);
        
        // 访问个不存的接口，报错
        const urls = yield axios('/error');
        console.log(urls);
    }
}

function co (generator) {
    const g = generator();
    
    function handleResult (result) {
        if (result.done) return;
        result.value.then(data => {
            handleResult(g.next(data));
        }, error => {
            g.throw(error);
        })
    }
    
    handleResult(g.next());
}

co(main);
```

## generator.throw
generator.throw，接收一个错误对象，用于抛出错误到generator函数内对应的yield，如果generator函数内部没有捕获该错误（try catch），错误会被
抛出到外层，也就是throw的调用处，可以在此处捕获（try catch）。

## generator.return
generator.return(value) 完成 generator 的执行忽略剩下的yield并返回给定的 value。如果generator已经迭代完，调用该方法也会返回给定的value。

## 伪随机函数
将“种子（seed）”作为第一个值，然后使用公式生成下一个值。以便相同的种子（seed）可以产出（yield）相同的序列，因此整个数据流很容易复现。

```javascript
function* pseudoRandom(seed) {
  let value = seed;

  while(true) {
    value = value * 16807 % 2147483647
    yield value;
  }

};

let generator = pseudoRandom(1);

alert(generator.next().value); // 16807
alert(generator.next().value); // 282475249
alert(generator.next().value); // 1622650073
```
