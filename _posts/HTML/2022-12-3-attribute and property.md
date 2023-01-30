---
layout:     post
title:      attribute and property
subtitle:   
date:       2022-12-3
author:     sq
header-img: 
catalog: true
tags:
    - HTML
---
DOM 元素的属性和方法的行为和 JavaScript 对象一样。可以在 DOM 元素上增加属性或方法。

## attribute 特性
当浏览器解析 HTML 文本，并根据标签创建 DOM 对象时，浏览器会辨别标准的特性并以此创建 DOM 属性。
但是非标准的特性则不会。每个元素所支持的特性也不一定相同。

HTML 特性有以下几个特征：
- 它们的名字是大小写不敏感的（id 与 ID 相同）。
- 它们的值总是字符串类型的。

所有标准或非标准特性都可以通过使用以下方法进行访问：
- elem.hasAttribute(name) —— 检查特性是否存在。
- elem.getAttribute(name) —— 获取这个特性值。
- elem.setAttribute(name, value) —— 设置这个特性值。
- elem.removeAttribute(name) —— 移除这个特性。

## 属性（property）—特性（attribute）同步
当一个标准的特性被改变，对应的属性也会自动更新，（除了几个特例，例如input的value，改变特性值 value 会更新属性，属性更改不更新特性）反之亦然。

特性都是字符串类型的，属性是不一定

有一种非常少见的情况，即使一个 DOM 属性是字符串类型的，但它可能和 HTML 特性也是不同的。
例如，href DOM 属性一直是一个 完整的 URL，即使该特性包含一个相对路径或者包含一个 #hash。

为了避免自定义特性和原生冲突，存在 data-* 特性。
所有以 “data-” 开头的特性均被保留供程序员使用。它们可在 dataset 属性中使用。
例如，如果一个 elem 有一个名为 "data-order-state" 的特性，那么可以通过 elem.dataset.orderState 取到它。