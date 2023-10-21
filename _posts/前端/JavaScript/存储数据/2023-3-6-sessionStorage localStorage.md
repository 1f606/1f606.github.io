---
layout:     post
title:      sessionStorage, localStorage
subtitle:   
date:       2023-3-6
author:     
header-img:
catalog: true
tags:
- storage
---
# sessionStorage, localStorage
两者存储的键和值都必须是字符串。

存储大小限制为 5MB+，具体取决于浏览器。

两个存储对象都提供相同的方法和属性：
- setItem(key, value) —— 存储键/值对。
- getItem(key) —— 按照键获取值。
- removeItem(key) —— 删除键及其对应的值。
- clear() —— 删除所有数据。
- key(index) —— 获取该索引下的键名。
- length —— 存储的内容的长度。

## sessionStorage
特点：
- 数据只存在于当前浏览器标签页。但是，它在同一标签页下的 iframe 之间是共享的（假如它们来自相同的源）。
- 数据在页面刷新后仍然保留，但在关闭/重新打开浏览器标签页后不会被保留。

## localStorage
特点：
- 在同源（域/端口/协议）的所有标签页和窗口之间共享数据。
- 数据不会过期。它在浏览器重启甚至系统重启后仍然存在。
- 可以以类对象形式访问，但不推荐，key和内置方法同名会导致失效，而且这种方式更改数据不触发storage事件。

## 遍历方式
length + key，for in + hasOwnProperty，Object.keys

## storage 事件
sessionStorage 和 localStorage 中数据更新后，storage 事件会触发。它具有以下属性：
- key —— 发生更改的数据的 key（如果调用的是 .clear() 方法，则为 null）。
- oldValue —— 旧值（如果是新增数据，则为 null）。
- newValue —— 新值（如果是删除数据，则为 null）。
- url —— 发生数据更新的文档的 url。
- storageArea —— 发生数据更新的 localStorage 或 sessionStorage 对象。

重要的是：该事件会在所有可访问到相同存储对象的 window 对象上触发，导致当前数据改变的 window 对象除外。

**这允许同源的不同窗口交换消息。**

现代浏览器还支持 Broadcast Channel API 实现同源窗口交换信息。

## 资料
https://zh.javascript.info/localstorage