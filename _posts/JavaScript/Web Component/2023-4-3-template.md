---
layout:     post
title:      template
subtitle:   
date:       2023-4-3
author:     sq
header-img: 
catalog: true
tags:
    - Web Component
---
# template
内建的 `<template>` 元素用来存储 HTML 模板。浏览器将忽略它的内容，仅检查语法的有效性，但是我们可以在 JavaScript 中访问和使用它来创建其他元素。

`template` 内容可以是任何有效的HTML。

例如，我们可以在其中放置一行表格` <tr> `。

通常如果我们在 `<tr>` 内放置类似 `<div>` 的元素，浏览器会检测到无效的 DOM 结构并对其进行“修复”，然后用 `<table>` 封闭 `<tr>` ，那不是我们想要的。而 `<template>` 则完全保留我们储存的内容。

我们也可以将样式和脚本放入 `<template>` 元素中，浏览器认为 `<template>` 的内容“不在文档中”：样式不会被应用，脚本也不会被执行， `<video autoplay>` 也不会运行，等。

当我们将内容插入文档时，该内容将变为活动状态（应用样式，运行脚本等）。

## 插入模板
模板的 `content` 属性可看作DocumentFragment，可以被插入到文档中。

例子：
```html
<template id="tmpl">
  <style> p { font-weight: bold; } </style>
  <p id="message"></p>
</template>

<div id="elem">Click me</div>

<script>
  elem.onclick = function() {
    elem.attachShadow({mode: 'open'});

    elem.shadowRoot.append(tmpl.content.cloneNode(true)); // (*)

    elem.shadowRoot.getElementById('message').innerHTML = "Hello from the shadows!";
  };
</script>
```

## 总结
* `<template>` 的内容可以是任何语法正确的 HTML。
* `<template>` 内容被视为“超出文档范围”，因此它不会产生任何影响。
* 我们可以在JavaScript 中访问 template.content ，将其克隆以在新组件中重复使用。

`<template>` 标签非常独特，因为：
* 浏览器将检查其中的HTML语法（与在脚本中使用模板字符串不同）。
* 但允许使用任何顶级 HTML 标签，即使没有适当包装元素的无意义的元素（例如 `<tr>` ）。
* 其内容是交互式的：插入其文档后，脚本会运行， `<video autoplay>` 会自动播放。
* `<template>` 元素不具有任何迭代机制，数据绑定或变量替换的功能，但我们可以在其基础上实现这些功能。

## 资料
https://zh.javascript.info/template-element
