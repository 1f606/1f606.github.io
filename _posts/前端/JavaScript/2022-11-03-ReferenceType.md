---
layout:     post
title:      Reference Type
subtitle:   
date:       2022-11-03
author:     现代JavaScript教程
header-img: 
catalog: true
tags:
    - JavaScript
---
## 前言
> 以下例子都是在严格模式下执行，报错是因为this是undefined，非严格模式下this是window
## this丢失
动态执行的方法调用可能会丢失 this。
```javascript
let user = {
  name: "John",
  hi() { alert(this.name); },
  bye() { alert("Bye"); }
};

user.hi(); // 正常运行

(user.name == "John" ? user.hi : user.bye)(); // Error!
```
## Reference Type解读
user.hi()能正常运行是因为在对属性user.hi的访问中，“.”返回的不是一个函数，而是返回了一个特殊的[Reference Type](https://tc39.github.io/ecma262/#sec-reference-specification-type)值，它被用在 JavaScript 语言内部。
Reference Type 的值是一个三个值的组合 (base, name, strict)，其中：
- base 是对象。
- name 是属性名。
- strict 在 use strict 模式下为 true。
当 () 被在 Reference Type 上调用时，它们会接收到关于对象和对象的方法的完整信息，然后可以设置正确的 this（在此处 =user）。
赋值操作会将Reference Type丢弃，而将值传递，所以后续操作丢失了this。
## 总结
Reference Type 是语言内部的一个类型。
读取一个属性，. 返回的准确来说不是属性的值，而是一个特殊的 “Reference Type” 值，其中储存着属性的值和它的来源对象。这是为了随后的方法调用 () 获取来源对象，然后将 this 设为它。
对于所有其它操作，Reference Type 会自动变成属性的值（在我们这个情况下是一个函数）。
这个机制仅在一些情况下才重要，例如使用表达式从对象动态地获取一个方法时。
