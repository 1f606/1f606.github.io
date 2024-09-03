---
layout:     post
title:      四个CSS通用值
subtitle:   inheherit, initial, unset, revert和all
date:       2024-9-3
author:     sq
header-img: 
catalog: true
tags:
    - CSS
---

## revert
revert让当前元素的样式还原成浏览器内置的样式。

## initial
Initial 是还原成CSS语法中规定的初始值，而不是 revert 的效果

## unset
如果当前使用的CSS属性是具有继承特性的，如color属性，则等同于使用inherit关键字；

如果当前使用的CSS属性是没有继承特性的，如background-color，则等同于使用initial关键字。

目前只有搭配all属性使用才有意义，否则直接使用更清晰的关键字就好。
例如用all:unset重置某些内置元素的样式，再设置需要的CSS属性。

## all
all属性可以重置除unicode-bidi、direction以及CSS自定义属性以外的所有CSS属性。

all 支持以上四个关键字，其中inherit和initial没有实用价值。unset可以让元素表现和span标签一样。revert可以让元素恢复成浏览器默认样式。

例如，<progress>进度条效果在iOS端很好看，很有质感，那么无须对其自定义样式，我们就可以使用all:revert将进度条一键还原成系统默认的样式：
```css
/* 仅iOS Safari有效 */
@supports (-webkit-overflow-scrolling: touch) {
    progress {
        all: revert;
    }
}
```

