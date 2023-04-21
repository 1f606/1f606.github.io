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

一般来说，局部样式只在 shadow 树内起作用，文档样式在 shadow 树外起作用。但也有少数例外。

我们可以使用 :host-family 系列的选择器来对组件的主元素进行样式设置，具体取决于上下文。这些样式（除 !important 外）可以被文档样式覆盖。

## :host
`:host` 选择器允许选择 shadow 根元素（包含 shadow 树的元素）。

例如，我们正在创建 Shadow Dom 元素，并且想使它居中。为此，我们需要对 Shadow Dom 元素本身设置样式。

```html
<template id="tmpl">
  <style>
    /* 这些样式将从内部应用到 custom-dialog 元素上 */
    :host {
      position: fixed;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      display: inline-block;
      border: 1px solid red;
      padding: 10px;
    }
  </style>
  <slot></slot>
</template>

<script>
customElements.define('custom-dialog', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'}).append(tmpl.content.cloneNode(true));
  }
});
</script>

<custom-dialog>
  Hello!
</custom-dialog>
```

## 级联
shadow Dom 根元素驻留在 light DOM 中，因此它受到文档 CSS 规则的影响。

如果在局部的 `:host` 和文档中都给一个属性设置样式，那么文档样式优先。

我们可以在其 `:host` 规则中设置 “默认” 组件样式，然后在文档中轻松地覆盖它们。

唯一的例外是当局部属性被标记 !important 时，对于这样的属性，局部样式优先。

## :host(selector)
与 `:host` 相同，但仅在 shadow 根元素与 selector 匹配时才应用样式。

例如，匹配具有 `center` 属性的根元素：
```html
<template id="tmpl">
  <style>
    :host([centered]) {
      position: fixed;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      border-color: blue;
    }

    :host {
      display: inline-block;
      border: 1px solid red;
      padding: 10px;
    }
  </style>
  <slot></slot>
</template>

<script>
customElements.define('custom-dialog', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'}).append(tmpl.content.cloneNode(true));
  }
});
</script>


<custom-dialog centered>
  Centered!
</custom-dialog>

<custom-dialog>
  Not centered.
</custom-dialog>
```

## :host-context(selector)
与 :host 相同，但仅当外部文档中的 shadow 根元素或它的任何祖先节点与 selector 匹配时才应用样式。

例如，`:host-context(.dark-theme)` 只有在 `<custom-dialog>` 或者 `<custom-dialog>` 的任何祖先节点上有 `dark-theme` 类时才匹配：
```html
<body class="dark-theme">
  <!--
    :host-context(.dark-theme) 只应用于 .dark-theme 内部的 custom-dialog
  -->
  <custom-dialog>...</custom-dialog>
</body>
```

## 给插槽内容设置样式
插槽内容元素来自 light DOM，所以它们使用文档样式，Shadow Dom 样式不会影响它们。

如果我们想要在我们的组件中设置占槽元素的样式，有两种选择。

首先，我们可以对 `<slot>` 本身进行样式化，并借助 CSS 继承：
```html
<user-card>
  <div slot="username"><span>John Smith</span></div>
</user-card>

<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `
      <style>
      slot[name="username"] { font-weight: bold; }
      </style>
      Name: <slot name="username"></slot>
    `;
  }
});
</script>
```

另一个选项是使用 `::slotted(selector)` 伪类。它根据两个条件来匹配元素：

1. 这是一个占槽元素，来自于 light DOM。只能是插槽元素本身，而不是它的子元素，也就是带有 `slot` 属性的顶层元素。
2. 该元素与 selector 匹配。

## 用自定义 CSS 属性设置 Shadow Dom 子元素样式
像 `:host` 这样的选择器应用规则到的是 Shadow Dom 的根元素，内部子元素无法被设置。

自定义 CSS 属性存在于所有层次，包括 light DOM 和 shadow DOM。

例如，在 shadow DOM 中，我们可以使用 `--user-card-field-color` CSS 变量来设置字段的样式，而外部文档可以设置它的值：

```html
<style>
  user-card {
    --user-card-field-color: green;
  }
</style>

<template id="tmpl">
  <style>
    .field {
      /*默认值为black*/
      color: var(--user-card-field-color, black);
    }
  </style>
  <div class="field">Name: <slot name="username"></slot></div>
  <div class="field">Birthday: <slot name="birthday"></slot></div>
</template>

<script>
  customElements.define('user-card', class extends HTMLElement {
    connectedCallback() {
      this.attachShadow({mode: 'open'});
      this.shadowRoot.append(document.getElementById('tmpl').content.cloneNode(true));
    }
  });
</script>

<user-card>
  <span slot="username">John Smith</span>
  <span slot="birthday">01.01.2001</span>
</user-card>
```

## 总结
shadow DOM 可以引入样式，如 `<style>` 或 `<link rel="stylesheet">`。

局部样式可以影响：
* shadow 树,
* shadow 根元素（通过 :host-family 系列伪类），
* 占槽元素（来自 light DOM），`::slotted(selector)` 允许选择占槽元素本身，但不能选择它们的子元素。

文档样式可以影响：
* shadow 根元素（因为它位于外部文档中）
* 占槽元素及占槽元素的内容（因为它们同样位于外部文档中）

当 CSS 属性冲突时，通常文档样式具有优先级，除非属性被标记为 !important，那么局部样式优先。

CSS 自定义属性穿透 shadow DOM。它们被用作 “勾子” 来设计组件的样式：

组件使用自定义 CSS 属性对关键元素进行样式设置，比如 `var(--component-name-title, <default value>)` 。

