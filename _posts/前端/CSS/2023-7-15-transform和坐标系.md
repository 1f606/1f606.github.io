---
layout:     post
title:      transform、rotate和坐标系
subtitle:   
date:       2023-7-16
author:     sq
header-img: 
catalog: true
tags:
    - CSS
---
# transform
transform: translate，原点默认在元素正中间(transform-origin: 50%, 50%)。

其中Y轴垂直，X轴水平，Z轴面向屏幕。X轴从右往左看，顺时针正向。Y轴从上往下看，逆时针正向。Z轴正对着看，顺时针正向。

![img.png](/img/transform坐标系.png)

## 旋转
在动画中旋转的方向可能有所不同。

先设置transform: rotateX(180deg)，再设置transform: rotateZ(180deg)，会从rotateX180deg的状态，翻转变成rotateZ180deg的状态。就像rotateY180deg旋转的效果一样。

rotate3d(0, 1, 0, -360deg);角度增加是逆时针旋转，不知道为什么mdn上说正值是顺时针旋转

## perspective 和 translateZ
translateZ值越大，离屏幕越近。

perspective: 800px，代表3D物体距离屏幕是800px。假如元素translateZ为799px时，元素撑满屏幕，800px时元素消失，“出现”在眼睛后面了。

## 实现全景效果

计算机的3D世界就是现实3D世界的模拟。所以构建3D空间应该考虑以下三个要素。

第一个：镜头，第二个：拍摄的环境的空间，第三个：要拍摄的物件。

而在CSS的3D世界，我们也需要去模仿这三要素。我们用三层div来表示，第一层是摄像镜头、第二层是立体空间或也可叫舞台，第三层是立体空间内的元素。

[我用纯 CSS 写了一个720°全景效果](https://mp.weixin.qq.com/s/YlcYFOqdkII_TUEamRUYNg)

