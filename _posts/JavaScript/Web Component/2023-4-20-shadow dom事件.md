---
layout:     post
title:      shadow dom 事件
subtitle:   
date:       2023-4-20
author:     sq
header-img: 
catalog: true
tags:
    - Shadow Dom
---
# shadow dom 事件
## shadow dom 事件的 retarget
当事件在 shadow dom 外部捕获时，shadow DOM 中发生的事件将会以 host 元素作为目标。

如果事件发生在 slotted 元素上，实际存在于 light DOM 上，则不会发生重定向。

## 冒泡 和 event.composedPath
使用 `event.composedPath()` 获得原始事件目标的完整路径以及所有 shadow 元素。

例如：
```html
<user-card id="userCard">
  #shadow-root
    <div>
      <b>Name:</b>
      <slot name="username">
        <span slot="username">John Smith</span>
      </slot>
    </div>
</user-card>
```

因此，对于 `<span slot="username">` 上的点击事件，会调用 `event.composedPath()` 并返回一个数组：`[span, slot, div, shadow-root, user-card, body, html, document, window]`。

注意：Shadow 树详细信息仅提供给 `{mode:'open'}` 树，如果 shadow 树是用 `{mode: 'closed'}` 创建的，那么组合路径就从 host 开始：`user-card` 及其更上层。

## event.composed
大多数事件能成功冒泡到 shadow DOM 边界。很少有事件不能冒泡到 shadow DOM 边界。

这由 `composed` 事件对象属性控制。如果 `composed` 是 true，那么事件就能穿过边界。否则它仅能在 shadow DOM 内部捕获。

如果你浏览一下 [UI 事件规范](https://www.w3.org/TR/uievents) 就知道，大部分事件都是 composed: true。

但也有些事件是 composed: false 的：
* mouseenter，mouseleave（它们根本不会冒泡），
* load，unload，abort，error，
* select，
* slotchange。

这些事件仅能在事件目标所在的同一 DOM 中的元素上捕获，

## 自定义事件（Custom events）
当我们发送（dispatch）自定义事件，我们需要设置 `bubbles` 和 `composed` 属性都为 true 以使其冒泡并从组件中冒泡出来。

例子：
```html
<div id="outer"></div>

<script>
outer.attachShadow({mode: 'open'});

let inner = document.createElement('div');
outer.shadowRoot.append(inner);

/*
div(id=outer)
  #shadow-dom
    div(id=inner)
*/

document.addEventListener('test', event => alert(event.detail));

inner.dispatchEvent(new CustomEvent('test', {
  bubbles: true,
  composed: true,
  detail: "composed"
}));

inner.dispatchEvent(new CustomEvent('test', {
  bubbles: true,
  composed: false,
  detail: "not composed"
}));
</script>
```

## 总结
如果我们发送一个 CustomEvent，那么我们应该显式地设置 composed: true。

请注意，如果是嵌套组件，一个 shadow DOM 可能嵌套到另外一个 shadow DOM 中。在这种情况下合成事件冒泡到所有 shadow DOM 边界。因此，如果
一个事件仅用于直接封闭组件，我们也可以在 shadow host 上发送它并设置 composed: false。这样它就不在组件 shadow DOM 中，也不会冒泡到更高级别的 DOM。
