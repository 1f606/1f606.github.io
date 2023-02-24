---
layout:     post
title:      ContentType 和 Content-Disposition
subtitle:   
date:       2023-2-24
author:     
header-img: 
catalog: true
tags:
    - 
---
Content-Type: application/octet-stream，告知浏览器这是一个字节流，浏览器处理字节流的默认方式就是下载。

一般会配合另一个响应头Content-Disposition，该响应头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。

```
Content-Disposition: inline
Content-Disposition: attachment
Content-Disposition: attachment; filename="filename.jpg"
```
