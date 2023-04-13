---
layout:     post
title:      Shadow Dom 样式
subtitle:   
date:       2023-4-7
author:     sq
header-img: 
catalog: true
tags:
    - Web Component
---
# Shadow Dom 样式
shadow DOM 可以包含 `<style>` 和 `<link rel="stylesheet" href="…">` 标签。在后一种情况下，样式表是 HTTP 缓存的，因此不会为使用同一模
板的多个组件重新下载样式表。


