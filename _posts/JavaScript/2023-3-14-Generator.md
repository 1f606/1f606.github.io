---
layout:     post
title:      
subtitle:   
date:       
author:     
header-img: 
catalog: true
tags:
    - 
---
# Generator
generator 是可迭代的，可以使用 iterator 的所有相关功能，例如：spread 语法 和 for of。

generator 会忽略return，next方法迭代不会忽略return。

## Iterable object（可迭代对象）
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