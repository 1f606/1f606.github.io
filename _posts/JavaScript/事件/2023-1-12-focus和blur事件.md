---
layout:     post
title:      focus和blur事件
subtitle:   
date:       sq
author:     2023-1-12
header-img: 
catalog: true
tags:
    - JavaScript
    - 事件
---
# focus/blur
当用户点击某个元素或使用键盘上的 Tab 键选中时，该元素将会获得聚焦（focus）。当然还有其它途径可以获得焦点。
可以通过 document.activeElement 来获取当前所聚焦的元素。

当元素聚焦时，会触发 focus 事件，当元素失去焦点时，会触发 blur 事件。

elem.focus() 和 elem.blur() 方法可以设置和移除元素上的焦点。

请注意，我们无法通过在 onblur 事件处理程序中调用 event.preventDefault() 来“阻止失去焦点”，因为 onblur 事件处理程序是在元素失去焦点 之后 运行的。

## JavaScript 导致的焦点丢失
很多种原因可以导致焦点丢失。例如：
- 一个 alert 会将焦点移至自身，因此会导致元素失去焦点（触发 blur 事件），而当 alert 对话框被取消时，焦点又会重新回到原元素上（触发 focus 事件）。
- 如果一个元素被从 DOM 中移除，那么也会导致焦点丢失。如果稍后它被重新插入到 DOM，焦点也不会回到它身上。

如果我们想要跟踪用户导致的焦点丢失，则应该避免自己造成的焦点丢失。

## tabindex
默认情况下，很多元素不支持聚焦。 focus/blur 保证支持那些用户可以交互的元素：<button>，<input>，<select>，<a> 等。

另一方面，为了格式化某些东西而存在的元素像 <div>，<span> 和 <table> —— 默认是不能被聚焦的。elem.focus() 方法不适用于它们，并且 focus/blur 事件也绝不会被触发。

使用 HTML-特性（attribute）tabindex 可以改变这种情况。也可以使用 elem.tabIndex 通过 JavaScript 来添加 tabindex。效果是一样的。

任何具有 tabindex 特性的元素，都会变成可聚焦的。该特性的 value 是当使用 Tab（或类似的东西）在元素之间进行切换时，元素的顺序号。

切换顺序为：从 1 开始的具有 tabindex 的元素排在前面（按 tabindex 顺序），然后是不具有 tabindex 的元素（例如常规的 <input>）。
不具有 tabindex 的元素按文档源顺序（默认顺序）切换。

这里有两个特殊的值：
- tabindex="0" 会使该元素被与那些不具有 tabindex 的元素放在一起。也就是说，当我们切换元素时，具有 tabindex="0" 的元素将排在那些具有 tabindex ≥ 1 的元素的后面。
通常，它用于使元素具有焦点，但是保留默认的切换顺序。使元素成为与 <input> 一样的表单的一部分。
- tabindex="-1" 只允许以编程的方式聚焦于元素。Tab 键会忽略这样的元素，但是 elem.focus() 有效。

## focus/blur 委托
focus 和 blur 事件不会向上冒泡。

这里有两个解决方案。

方案一，有一个遗留下来的有趣的特性（feature）：focus/blur 不会向上冒泡，但会在捕获阶段向下传播。
```javascript
form.addEventListener("focus", () => form.classList.add('focused'), true);
form.addEventListener("blur", () => form.classList.remove('focused'), true);
```

方案二，可以使用 focusin 和 focusout 事件 —— 与 focus/blur 事件完全一样，只是它们会冒泡。
值得注意的是，必须使用 elem.addEventListener 来分配它们，而不是 on<event>。