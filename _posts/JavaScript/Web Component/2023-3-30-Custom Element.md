---
layout:     post
title:      Custom Element
subtitle:   
date:       2023-3-30
author:     sq
header-img:
catalog: true
tags:
     - Web Component
---
# Custom Element
Custom elements 有两种：
- Autonomous custom elements （自主自定义标签） —— “全新的” 元素, 继承自 `HTMLElement` 抽象类.
- Customized built-in elements （自定义内建元素） —— 继承内建的 HTML 元素，比如自定义 `HTMLButtonElement` 等。

## Custom element的创建
需要先创建带有特殊方法的类，然后注册元素。

```javascript
class MyElement extends HTMLElement {
  constructor() {
    super();
    // 元素在这里创建
  }

  connectedCallback() {
    // 在元素被添加到文档之后，浏览器会调用这个方法
    //（如果一个元素被反复添加到文档／移除文档，那么这个方法会被多次调用）
  }

  disconnectedCallback() {
    // 在元素从文档移除的时候，浏览器会调用这个方法
    // （如果一个元素被反复添加到文档／移除文档，那么这个方法会被多次调用）
  }

  static get observedAttributes() {
    return [/* 属性数组，这些属性的变化会被监视 */];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // 当上面数组中的属性发生变化的时候，这个方法会被调用
  }

  adoptedCallback() {
    // 在元素被移动到新的文档的时候，这个方法会被调用
    // （document.adoptNode 会用到, 非常少见）
  }

  // 还可以添加更多的元素方法和属性
}
customElements.define("my-element", MyElement);
```

现在 `<my-element>` 标签和 `document.createElement('my-element')` 的调用是有效的，会生成`MyElement`实例。

> define 传入的名称必须包含短横线-

> 在调用 define 前使用元素会被当作未知元素
> :not(:defined)CSS 选择器可以对这样「未定义」的元素加上样式。
> :defined选择器可以对定义的元素加上样式。

我们可以通过这些方法来获取更多的自定义标签的信息：
- customElements.get(name) —— 返回指定 custom element name 的类。
- customElements.whenDefined(name) – 返回一个 promise，将会在这个具有给定 name 的 custom element 变为已定义状态的时候 resolve（不带值）。

## 监听属性
为了对变化的属性做出响应，我们可以在 observedAttributes() static getter 中提供属性列表。当这些属性发生变化的时候，attributeChangedCallback 
会被调用。出于性能优化的考虑，其他属性变化的时候并不会触发这个回调方法。

一个带格式化功能的 time 标签
```html
<script>
class TimeFormatted extends HTMLElement {

  render() { // (1)
    let date = new Date(this.getAttribute('datetime') || Date.now());

    this.innerHTML = new Intl.DateTimeFormat("default", {
      year: this.getAttribute('year') || undefined,
      month: this.getAttribute('month') || undefined,
      day: this.getAttribute('day') || undefined,
      hour: this.getAttribute('hour') || undefined,
      minute: this.getAttribute('minute') || undefined,
      second: this.getAttribute('second') || undefined,
      timeZoneName: this.getAttribute('time-zone-name') || undefined,
    }).format(date);
  }

  connectedCallback() { // (2)
    if (!this.rendered) {
      this.render();
      this.rendered = true;
    }
  }

  static get observedAttributes() { // (3)
    return ['datetime', 'year', 'month', 'day', 'hour', 'minute', 'second', 'time-zone-name'];
  }

  attributeChangedCallback(name, oldValue, newValue) { // (4)
    this.render();
  }

}

customElements.define("time-formatted", TimeFormatted);
</script>

<time-formatted id="elem" hour="numeric" minute="numeric" second="numeric"></time-formatted>

<script>
setInterval(() => elem.setAttribute('datetime', new Date()), 1000); // (5)
</script>
```

基于 TimeFormatted 实现的实时时间
```javascript
class LiveTimer extends HTMLElement {

  render() {
    this.innerHTML = `
    <time-formatted hour="numeric" minute="numeric" second="numeric">
    </time-formatted>
    `;

    this.timerElem = this.firstElementChild;
  }

  connectedCallback() { // (2)
    if (!this.rendered) {
      this.render();
      this.rendered = true;
    }
    this.timer = setInterval(() => this.update(), 1000);
  }

  update() {
    this.date = new Date();
    this.timerElem.setAttribute('datetime', this.date);
    this.dispatchEvent(new CustomEvent('tick', { detail: this.date }));
  }

  disconnectedCallback() {
    clearInterval(this.timer); // important to let the element be garbage-collected
  }

}

customElements.define("live-timer", LiveTimer);
```

## 渲染顺序
在 HTML 解析器构建 DOM 的时候，会按照先后顺序处理元素，先处理父级元素再处理子元素。

如果一个 custom element 想要在 connectedCallback 内访问 innerHTML，它什么也拿不到。

如果我们要给 custom element 传入信息，我们可以使用元素属性。它们是即时生效的。

或者，如果我们需要子元素，我们可以使用延迟时间为零的 setTimeout 来推迟访问子元素。

但定时器的方法并不完美，如果嵌套的子元素也使用了定时器来初始化自身，那么外层元素还是早于内层元素完成初始化。

并没有任何内建的回调方法可以在嵌套元素渲染好之后通知我们。但我们可以自己实现这样的回调。比如，内层元素可以分派像 initialized 这样的事件，
同时外层的元素监听这样的事件并做出响应。

## Customized built-in elements
我们创建的 `<time-formatted>` 这些新元素，并没有任何相关的语义。搜索引擎并不知晓它们的存在，同时无障碍设备也无法处理它们。

我们可以通过继承内建元素的类来扩展和定制它们，从而拥有和内建类相同的样式和标准特性。

比如，按钮是 HTMLButtonElement 的实例，让我们在这个基础上创建元素。

1. 我们的类继承自 HTMLButtonElement：
```javascript
class HelloButton extends HTMLButtonElement { /* custom element 方法 */ }
```
2. 给 customElements.define 提供定义标签的第三个参数：
```javascript
customElements.define('hello-button', HelloButton, {extends: 'button'});
```
这一步是必要的，因为不同的标签会共享同一个类。

3. 最后，插入一个普通的 <button> 标签，但添加 is="hello-button" 到这个元素，这样就可以使用我们的 custom element：
```javascript
<button is="hello-button">...</button>
```

一个完整的例子：
```javascript
<script>
// 这个按钮在被点击的时候说 "hello"
class HelloButton extends HTMLButtonElement {
  constructor() {
    super();
    this.addEventListener('click', () => alert("Hello!"));
  }
}

customElements.define('hello-button', HelloButton, {extends: 'button'});
</script>

<button is="hello-button">Click me</button>

<button is="hello-button" disabled>Disabled</button>
```

## 总结
有两种 custom element：
1. “Autonomous” —— 全新的标签，继承 HTMLElement。
2. “Customized built-in elements” —— 已有元素的扩展。 需要传多一个 `define` 参数，同时需要添加 `is` attribute。

