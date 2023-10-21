---
layout:     post
title:      input和change事件
subtitle:   
date:       2023-1-12
author:     
header-img: 
catalog: true
tags:
    - JavaScript
    - 事件
---
# input和change事件
文本输入框当其失去焦点且值改变时，就会触发 change 事件。

文本输入框值改变后会立即触发 input 事件，因此无法使用 preventDefault 阻止默认行为，因为值已经发生改变。

