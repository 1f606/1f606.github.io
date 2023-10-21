---
layout:     post
title:      overflow
subtitle:   
date:       2023-6-17
author:     sq
header-img: 
catalog: true
tags:
    - CSS
---
# overflow

## overflow：hidden
设置 overflow-x: hidden 或者 overflow-y: hidden，表现形式都和 overflow: hidden 一致，是x，y全方位的裁剪。

## overflow: clip
使用 overflow: clip 可以轻松地对溢出方向进行控制。

overflow: clip: 与 overflow: hidden 的表现形式极为类似，也是对元素的 padding-box 进行裁剪。

它们有两点不同：
- overflow: clip 内部完全禁止任何形式的滚动。
- overflow: clip 可以从 x，y 轴方向上对裁剪进行控制，而 overflow: hidden 不行。

overflow-x: clip/ overflow-y: clip 配合另一个方向的 overflow-x: visible，能够实现一个方向允许溢出，一个方向实现裁剪。

## 不用 overflow: hidden 实现 overflow: hidden
https://mp.weixin.qq.com/s/rvFFh4zPAMJF0C14wSqUFg

## 资料
[有意思的方向裁切 overflow: clip](https://mp.weixin.qq.com/s/4CbWmZ5mXjr7oUcL7wvWGQ)
