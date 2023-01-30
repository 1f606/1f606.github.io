---
layout:     post
title:      Mutation observer
subtitle:   
date:       2023-1-29
author:     sq
header-img: 
catalog: true
tags:
    - Mutation observer
---
# Mutation observer
MutationObserver 是一个内建对象，它可以观察 DOM 元素任何更改，并在检测到更改时触发回调。

## 语法
创建一个带有回调函数的观察器：

`let observer = new MutationObserver(callback);`

然后将其附加到一个 DOM 节点：

`observer.observe(node, config);`

config 是一个具有布尔选项的对象，该布尔选项表示“将对哪些更改做出反应”：
- childList —— node 的直接子节点的更改，
- subtree —— node 的所有后代的更改，
- attributes —— node 的特性（attribute），
- attributeFilter —— 特性名称数组，只观察选定的特性。
- characterData —— 是否观察 node.data（文本内容），
- attributeOldValue —— 如果为 true，则将特性的旧值和新值都传递给回调（参见下文），否则只传新值（需要 attributes 选项），
- characterDataOldValue —— 如果为 true，则将 node.data 的旧值和新值都传递给回调（参见下文），否则只传新值（需要 characterData 选项）。

然后，在发生任何更改后，将执行回调：更改被作为一个 [MutationRecord](https://dom.spec.whatwg.org/#mutationrecord) 对象列表传入第一个参数，而观察器自身作为第二个参数。

MutationRecord 对象具有以下属性：
- type —— 变动类型，以下类型之一：
  - "attributes"：特性被修改了，
  - "characterData"：数据被修改了，用于文本节点，
  - "childList"：添加/删除了子元素。
- target —— 更改发生在何处："attributes" 所在的元素，或 "characterData" 所在的文本节点，或 "childList" 变动所在的元素，
- addedNodes/removedNodes —— 添加/删除的节点，
- previousSibling/nextSibling —— 添加/删除的节点的上一个/下一个兄弟节点，
- attributeName/attributeNamespace —— 被更改的特性的名称/命名空间（用于 XML），
- oldValue —— 之前的值，仅适用于特性或文本更改，如果设置了相应选项 attributeOldValue/characterDataOldValue。

## 其他方法
`observer.disconnect()` —— 停止观察。

当我们停止观察时，观察器可能尚未处理某些更改。在种情况下，我们使用：

`observer.takeRecords()` —— 获取尚未处理的变动记录列表，表中记录的是已经发生，但回调暂未处理的变动。

这些方法可以一起使用，如下所示：
```javascript
// 如果你关心可能未处理的近期的变动
// 那么，应该在 disconnect 前调用获取未处理的变动列表
let mutationRecords = observer.takeRecords();

// 停止跟踪变动
observer.disconnect();
```

> observer.takeRecords() 返回的记录被从处理队列中移除。回调函数不会被 observer.takeRecords() 返回的记录调用。

> 观察器在内部对节点使用弱引用。也就是说，如果一个节点被从 DOM 中移除了，并且该节点变得不可访问，那么它就可以被垃圾回收。
> 观察到 DOM 节点这一事实并不能阻止垃圾回收。

