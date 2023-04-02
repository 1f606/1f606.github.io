---
layout:     post
title:      Shadow Dom
subtitle:   
date:       2023-4-2
author:     sq
header-img: 
catalog: true
tags:
    - Web Component
---
# Shadow Dom
Shadow DOM 让一个组件拥有自己的「影子」DOM 树，这个 DOM 树不能在主文档中被任意访问，可能拥有局部样式规则，还有其他特性。

## 浏览器内置控件
浏览器控件在内部使用 DOM/CSS 来绘制它们。这个 DOM 结构一般来说对我们是隐藏的，但我们可以在开发者工具里面看见它。比如，在 Chrome 里，我们需要
打开「Show user agent shadow DOM」选项。

你在 #shadow-root 下看到的就是被称为「shadow DOM」的东西。

我们**不能使用一般的 JavaScript 调用或者选择器**来获取内建 shadow DOM 元素。它们不是常规的子元素，而是一个强大的封装手段。

部分 shadow dom 的元素有 `pseudo` attribute，可以用它来给元素修改 CSS 样式。

如：pseudo="-webkit-slider-runnable-track"

```css
input::-webkit-slider-runnable-track {
    background: red;
}
```

但 `pesudo` 是非标准属性，接下来，我们将要使用现代 shadow DOM 标准，它在 [DOM spec](https://dom.spec.whatwg.org/#shadow-trees) 和其他相关标准中可以被找到。

## Shadow Tree
一个 DOM 元素可以有以下两类 DOM 子树：
* Light tree（光明树） —— 一个常规 DOM 子树，由 HTML 子元素组成。我们在之前章节看到的所有子树都是「光明的」。
* Shadow tree（影子树） —— 一个隐藏的 DOM 子树，不在 HTML 中反映，无法被察觉。

如果一个元素同时有以上两种子树，那么浏览器只渲染 shadow tree。但是我们同样可以设置两种树的组合。我们将会在后面的 [Shadow DOM 插槽，组成](https://zh.javascript.info/slots-composition) 中
看到更多细节。

影子树可以在自定义元素中被使用，其作用是隐藏组件内部结构和添加只在组件内有效的样式。

例子：
```html
<script>
customElements.define('show-hello', class extends HTMLElement {
  connectedCallback() {
    const shadow = this.attachShadow({mode: 'open'});
    shadow.innerHTML = `<p>
      Hello, ${this.getAttribute('name')}
    </p>`;
  }
});
</script>

<show-hello name="John"></show-hello>
```

首先，调用 `elem.attachShadow({mode: …})` 可以创建一个 shadow tree。

这里有两个限制：
* 在每个元素中，我们只能创建一个 shadow root。
* elem 必须是自定义元素，或者是以下元素的其中一个：「article」、「aside」、「blockquote」、「body」、「div」、「footer」、「h1…h6」、
「header」、「main」、「nav」、「p」、「section」或者「span」。

mode 选项可以设定封装层级。他必须是以下两个值之一：
* 「open」 —— shadow root 可以通过 elem.shadowRoot 访问。

任何代码都可以访问 elem 的 shadow tree。
* 「closed」 —— elem.shadowRoot 永远是 null。

我们只能通过 attachShadow 返回的指针来访问 shadow DOM（并且可能隐藏在一个 class 中）。浏览器原生的 shadow tree，比如 `<input type="range">`，
是封闭的。没有任何方法可以访问它们。

attachShadow 返回的 [shadow root](https://dom.spec.whatwg.org/#shadowroot)，和任何元素一样：我们可以使用 innerHTML 或者 DOM 方法，比如 append 来扩展它。

我们称有 shadow root 的元素叫做「shadow tree host」，可以通过 shadow root 的 host 属性访问：

```javascript
// 假设 {mode: "open"}，否则 elem.shadowRoot 是 null
alert(elem.shadowRoot.host === elem); // true
```

## 封装
Shadow DOM 被非常明显地和主文档分开：
* Shadow DOM 元素对于 light DOM 中的 querySelector 不可见。实际上，Shadow DOM 中的元素可能与 light DOM 中某些元素的 id 冲突。这些元素必须在 shadow tree 中独一无二。
* Shadow DOM 有自己的样式。外部样式规则在 shadow DOM 中不产生作用。

```html
<style>
  /* 文档样式对 #elem 内的 shadow tree 无作用 (1) */
  p { color: red; }
</style>

<div id="elem"></div>

<script>
  elem.attachShadow({mode: 'open'});
  // shadow tree 有自己的样式 (2)
  elem.shadowRoot.innerHTML = `
    <style> p { font-weight: bold; } </style>
    <p>Hello, John!</p>
  `;

  // <p> 只对 shadow tree 里面的查询可见 (3)
  alert(document.querySelectorAll('p').length); // 0
  alert(elem.shadowRoot.querySelectorAll('p').length); // 1
</script>
```

## 总结
Shadow DOM 是创建组件级别 DOM 的一种方法。

* `shadowRoot = elem.attachShadow({mode: open|closed})` —— 为 elem 创建 shadow DOM。如果 mode="open"，那么它通过 elem.shadowRoot 属性被访问。
* 我们可以使用 innerHTML 或者其他 DOM 方法来扩展 shadowRoot。

Shadow DOM 元素：
* 有自己的 id 空间。
* 对主文档的 JavaScript 选择器隐身，比如 querySelector。
* 只使用 shadow tree 内部的样式，不使用主文档的样式。

Shadow DOM，如果存在的话，会被浏览器渲染而不是所谓的 「light DOM」（普通子元素）。

## 资料
https://zh.javascript.info/shadow-dom
DOM：https://dom.spec.whatwg.org/#shadow-trees

兼容性：https://caniuse.com/#feat=shadowdomv1

Shadow DOM 在很多其他标准中被提到，比如：[DOM Parsing](https://zh.javascript.info/shadow-dom) 指定了 shadow root 有 innerHTML。
