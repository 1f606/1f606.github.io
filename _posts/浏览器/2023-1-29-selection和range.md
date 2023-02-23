---
layout:     post
title:      selection和range
subtitle:   
date:       2023-1-29
author:     
header-img: 
catalog: true
tags:
    - selection
    - range
---
# selection和range
JavaScript 可以访问现有的选择，选择/取消全部或部分 DOM 节点的选择，从文档中删除所选部分，将其包装到一个标签（tag）中，等。

## range
Range：本质上是一对“边界点”：范围起点和范围终点。

在没有任何参数的情况下，创建一个 Range 对象：
```javascript
let range = new Range();
```
然后，我们可以使用 range.setStart(node, offset) 和 range.setEnd(node, offset) 来设置选择边界。

offset 最小为0，最大为文本长度。setStart 设置的 offset会被选中，setEnd 设置的 offset 不会被选中。

### 选择部分文本
这两种方法中的第一个参数 node 都可以是文本节点或元素节点，而第二个参数的含义依赖于此。

如果 node 是一个文本节点，那么 offset 则必须是其文本中的位置。

### 选择元素节点
如果 node 是一个元素节点，那么 offset 则必须是子元素的编号。

例如，我们有一个更复杂的文档片段：
```html
<p id="p">Example: <i>italic</i> and <b>bold</b></p>
```
这是它的 DOM 结构，包含元素和文本节点：
- P
  - #text Example：
  - I
    - #text italic
  - #text and
  - B
    - #text bold

正如我们所看到的，这个短语正好由 `<p>` 的索引为 0 和 1 的两个子元素组成。
![img.png](/img/selection和range-选择元素节点.png)

- 起点以 <p> 作为父节点 node，0 作为偏移量。

因此，我们可以将其设置为 range.setStart(p, 0)。

- 终点也是以 <p> 作为父节点 node，但以 2 作为偏移量（它指定最大范围，但不包括 offset）。

因此，我们可以将其设置为 range.setEnd(p, 2)。

> 起始和结束的节点可以不同。一个范围可能会跨越很多不相关的节点。唯一要注意的是终点要在起点之后。

### 选择更大的片段
例如我们想要选择上例中，"ample" 开头，"bol" 结尾的范围。
```javascript
let range = new Range();

range.setStart(p.firstChild, 2);
range.setEnd(p.querySelector('b').firstChild, 3);
```

### range属性
我们在上面的示例中创建的 range 对象具有以下属性：
- startContainer，startOffset —— 起始节点和偏移量，
  - 在上例中：分别是 <p> 中的第一个文本节点和 2。
- endContainer，endOffset —— 结束节点和偏移量，
  - 在上例中：分别是 <b> 中的第一个文本节点和 3。
- collapsed —— 布尔值，如果范围在同一点上开始和结束（所以范围内没有内容）则为 true，
  - 在上例中：false
- commonAncestorContainer —— 在范围内的所有节点中最近的共同祖先节点，
  - 在上例中：<p>

### 选择范围的方法
设置范围的起点：
- setStart(node, offset) 将起点设置在：node 中的位置 offset
- setStartBefore(node) 将起点设置在：node 前面
- setStartAfter(node) 将起点设置在：node 后面

设置范围的终点（类似的方法）：
- setEnd(node, offset) 将终点设置为：node 中的位置 offset
- setEndBefore(node) 将终点设置为：node 前面
- setEndAfter(node) 将终点设置为：node 后面

创建范围的方法：
- selectNode(node) 设置范围以选择整个 node
- selectNodeContents(node) 设置范围以选择整个 node 的内容
- collapse(toStart) 如果 toStart=true 则设置 end=start，否则设置 start=end，从而折叠范围
- cloneRange() 创建一个具有相同起点/终点的新范围

编辑范围的方法：
- deleteContents() —— 从文档中删除范围中的内容
- extractContents() —— 从文档中删除范围中的内容，并将删除的内容作为 DocumentFragment 返回
- cloneContents() —— 复制范围中的内容，并将复制的内容作为 DocumentFragment 返回
- insertNode(node) —— 在范围的起始处将 node 插入文档
- surroundContents(node) —— 使用 node 将所选范围中的内容包裹起来。要使此操作有效，则该范围必须包含其中所有元素的开始和结束标签：不能像 <i>abc 这样的部分范围。

## selection
Range 是用于管理选择范围的通用对象。但它们并不会在视觉上选择任何内容。

文档选择是由 Selection 对象表示的，可通过 window.getSelection() 或 document.getSelection() 来获取。一个选择可以包括零个或多个范围。
至少，[Selection API 规范](https://www.w3.org/TR/selection-api/) 是这么说的。不过实际上，
只有 Firefox 允许使用 Ctrl+click (Mac 上用 Cmd+click) 在文档中选择多个范围。其他浏览器最多支持 1 个范围。

### 选择属性
我们可以使用下面这个方法获取这些范围对象：
- getRangeAt(i) —— 获取第 i 个范围，i 从 0 开始。在除 Firefox 之外的所有浏览器中，仅使用 0。
- anchorNode —— 选择的起始节点，
- anchorOffset —— 选择开始的 anchorNode 中的偏移量，
- focusNode —— 选择的结束节点，
- focusOffset —— 选择开始处 focusNode 的偏移量，
- isCollapsed —— 如果未选择任何内容（空范围）或不存在，则为 true 。
- rangeCount —— 选择中的范围数，除 Firefox 外，其他浏览器最多为 1。

选择的起点被称为“锚点（anchor）”，终点被称为“焦点（focus）”。

### 选择和范围的起点和终点对比
正如我们所知道的，Range 对象的起点必须在其终点之前。

但对于选择，并不总是这样的。

我们可以在两个方向上使用鼠标进行选择：“从左到右”或“从右到左”。

### 选择事件
- elem.onselectstart —— 当在元素 elem 上（或在其内部）开始选择时。例如，当用户在元素 elem 上按下鼠标按键并开始移动指针时。
  - 阻止默认行为取消了选择的开始。因此，从该元素开始选择变得不可能，但该元素仍然是可选择的。用户只需要从其他地方开始选择。
- document.onselectionchange —— 当选择发生变化或开始时。
  - 请注意：此处理程序只能在 document 上设置。它跟踪的是 document 中的所有选择。

### 选择复制
复制所选内容有两种方式：
- 我们可以使用 document.getSelection().toString() 来获取其文本形式。
- 此外，想要复制整个 DOM 节点，例如，如果我们需要保持其格式不变，我们可以使用 getRangeAt(...) 获取底层的（underlying）范围。Range 对象还具有 cloneContents() 方法，该方法会拷贝范围中的内容并以 DocumentFragment 的形式返回，我们可以将这个返回值插入到其他位置。

例如：
```html
<p id="p">Select me: <i>italic</i> and <b>bold</b></p>

Cloned: <span id="cloned"></span>
<br>
As text: <span id="astext"></span>

<script>
  document.onselectionchange = function() {
    let selection = document.getSelection();

    cloned.innerHTML = astext.innerHTML = "";

    // 从范围复制 DOM 节点（这里我们支持多选）
    for (let i = 0; i < selection.rangeCount; i++) {
      cloned.append(selection.getRangeAt(i).cloneContents());
    }

    // 获取为文本形式
    astext.innerHTML += selection;
  };
</script>
```

### 选择方法
我们可以通过添加/移除范围来处理选择：
- getRangeAt(i) —— 获取从 0 开始的第 i 个范围。在除 Firefox 之外的所有浏览器中，仅使用 0。
- addRange(range) —— 将 range 添加到选择中。如果选择已有关联的范围，则除 Firefox 外的所有浏览器都将忽略该调用。
- removeRange(range) —— 从选择中删除 range。
- removeAllRanges() —— 删除所有范围。
- empty() —— removeAllRanges 的别名。

还有一些方便的方法可以直接操作选择范围，而无需中间的 Range 调用：
- collapse(node, offset) —— 用一个新的范围替换选定的范围，该新范围从给定的 node 处开始，到偏移 offset 处结束。
- setPosition(node, offset) —— collapse 的别名。
- collapseToStart() —— 折叠（替换为空范围）到选择起点，
- collapseToEnd() —— 折叠到选择终点，
- extend(node, offset) —— 将选择的焦点（focus）移到给定的 node，位置偏移 oofset，
- setBaseAndExtent(anchorNode, anchorOffset, focusNode, focusOffset) —— 用给定的起点 anchorNode/anchorOffset 和终点 focusNode/focusOffset 来替换选择范围。选中它们之间的所有内容。
- selectAllChildren(node) —— 选择 node 的所有子节点。
- deleteFromDocument() —— 从文档中删除所选择的内容。
- containsNode(node, allowPartialContainment = false) —— 检查选择中是否包含 node（若第二个参数是 true，则只需包含 node 的部分内容即可）

例如，选择段落 <p> 的全部内容：
```html
<p id="p">Select me: <i>italic</i> and <b>bold</b></p>

<script>
  // 从 <p> 的第 0 个子节点选择到最后一个子节点
  document.getSelection().setBaseAndExtent(p, 0, p, p.childNodes.length);
</script>
```
使用范围来完成同一个操作：
```html
<p id="p">Select me: <i>italic</i> and <b>bold</b></p>

<script>
  let range = new Range();
  range.selectNodeContents(p); // 或者也可以使用 selectNode(p) 来选择 <p> 标签

  document.getSelection().removeAllRanges(); // 清除现有选择（如果有的话）
  document.getSelection().addRange(range);
</script>
```

#### 如要选择一些内容，请先移除现有的选择。
如果在文档中已存在选择，则首先使用 removeAllRanges() 将其清空。然后添加范围。否则，除 Firefox 外的所有浏览器都将忽略新范围。

某些选择方法例外，它们会替换现有的选择，例如 setBaseAndExtent。

### 表单控件中的选择
诸如 input 和 textarea 等表单元素提供了 专用的选择 API，没有 Selection 或 Range 对象。由于输入值是纯文本而不是 HTML，因此不需要此类对象，一切都变得更加简单。

属性：
- `input.selectionStart` —— 选择的起始位置（可写），
- `input.selectionEnd` —— 选择的结束位置（可写），
- `input.selectionDirection` —— 选择方向，其中之一：“forward”，“backward” 或 “none”（例如使用鼠标双击进行的选择），

事件：
- `input.onselect` —— 当某个东西被选择时触发。

方法：
- `input.select()` —— 选择文本控件中的所有内容（可以是 textarea 而不是 input），
- `input.setSelectionRange(start, end, [direction])` —— 在给定方向上（可选），从 start 一直选择到 end。
- `input.setRangeText(replacement, [start], [end], [selectionMode])` —— 用新文本替换范围中的文本。
  - 可选参数 start 和 end，如果提供的话，则设置范围的起点和终点，否则使用用户的选择。
    最后一个参数 selectionMode 决定替换文本后如何设置选择。可能的值为：
    - "select" —— 将选择新插入的文本。
    - "start" —— 选择范围将在插入的文本之前折叠（光标将在其之前）。
    - "end" —— 选择范围将在插入的文本之后折叠（光标将紧随其后）。
    - "preserve" —— 尝试保留选择。这是默认值。

#### 示例：跟踪选择
```html
<textarea id="area" style="width:80%;height:60px">
Selecting in this text updates values below.
</textarea>
<br>
From <input id="from" disabled> – To <input id="to" disabled>

<script>
  area.onselect = function() {
    from.value = area.selectionStart;
    to.value = area.selectionEnd;
  };
</script>
```
请注意：
- onselect 是在某项被选择时触发，而在选择被删除时不触发。
- 根据 规范，表单控件内的选择不应该触发 document.onselectionchange 事件，因为它与 document 选择和范围不相关。一些浏览器会生成它，但我们不应该依赖它。

#### 示例：移动光标
我们可以更改 selectionStart 和 selectionEnd，二者设定了选择。

一个重要的边界情况是 selectionStart 和 selectionEnd 彼此相等。那正是光标位置。或者，换句话说，当未选择任何内容时，选择会折叠在光标位置。

因此，通过将 selectionStart 和 selectionEnd 设置为相同的值，我们可以移动光标。
```html
<textarea id="area" style="width:80%;height:60px">
Focus on me, the cursor will be at position 10.
</textarea>

<script>
  area.onfocus = () => {
    // 设置零延迟 setTimeout 以在浏览器 "focus" 行为完成后运行
    setTimeout(() => {
      // 我们可以设置任何选择
      // 如果 start=end，则光标就会在该位置
      area.selectionStart = area.selectionEnd = 10;
    });
  };
</script>
```

#### 示例：修改选择
如要修改选择的内容，我们可以使用 input.setRangeText() 方法。当然，我们可以读取 selectionStart/End，
并在了解选择的情况下更改 value 的相应子字符串，但是 setRangeText 功能更强大，它可以替换用户选择的范围并删除该选择。

例如：这里的用户的选择将被包装在星号中：
```html
<input id="input" style="width:200px" value="Select here and click the button">
<button id="button">Wrap selection in stars *...*</button>

<script>
button.onclick = () => {
  if (input.selectionStart == input.selectionEnd) {
    return; // 什么都没选
  }

  let selected = input.value.slice(input.selectionStart, input.selectionEnd);
  input.setRangeText(`*${selected}*`);
};
</script>
```
使用更多参数，我们可以设置范围 start 和 end。

在下面这个示例中，我们在输入文本中找到 "THIS"，将其替换，并保持替换文本的选中状态：
```html
<input id="input" style="width:200px" value="Replace THIS in text">
<button id="button">Replace THIS</button>

<script>
button.onclick = () => {
  let pos = input.value.indexOf("THIS");
  if (pos >= 0) {
    input.setRangeText("*THIS*", pos, pos + 4, "select");
    input.focus(); // 聚焦（focus），以使选择可见
  }
};
</script>
```

#### 示例：在光标处插入
如果未选择任何内容，或者我们在 setRangeText 中使用了相同的 start 和 end，则仅插入新文本，不会删除任何内容。

我们也可以使用 setRangeText 在“光标处”插入一些东西。

这是一个按钮，按下后会在光标位置插入 "HELLO"，然后光标紧随其后。如果选择不为空，则将其替换（我们可以通过比较 selectionStart!=selectionEnd 来进行检查，为空则执行其他操作）：
```html
<input id="input" style="width:200px" value="Text Text Text Text Text">
<button id="button">Insert "HELLO" at cursor</button>

<script>
  button.onclick = () => {
    input.setRangeText("HELLO", input.selectionStart, input.selectionEnd, "end");
    input.focus();
  };
</script>
```

#### 使不可选
要使某些内容不可选，有三种方式：
- 使用 CSS 属性 user-select: none。这样不允许选择从 elem 开始。但是用户可以在其他地方开始选择，并将 elem 包含在内。然后 elem 将成为 document.getSelection() 的一部分，因此选择实际发生了，但是在复制粘贴中，其内容通常会被忽略。
- 禁止 onselectstart 或 mousedown 事件中的默认行为。这样可以防止在 elem 上开始选择，但是访问者可以在另一个元素上开始选择，然后扩展到 elem。 当同一行为上有另一个事件处理程序触发选择时（例如 mousedown），这会很方便。因此我们禁用选择以避免冲突，仍然允许复制 elem 内容。
- 还可以使用 document.getSelection().empty() 来在选择发生后清除选择。很少使用这种方法，因为这会在选择项消失时导致不必要的闪烁。


## 总结
常用方法：

- 获取选择。`document.getSelection()`
- 设置选择。

```javascript
let selection = document.getSelection();

// 直接：
selection.setBaseAndExtent(...from...to...);

// 或者我们可以创建一个范围并：
selection.removeAllRanges();
selection.addRange(range);
```

关于光标。在诸如 `<textarea>` 之类的可编辑元素中，光标的位置始终位于选择的起点或终点。我们可以通过设置 elem.selectionStart 和 elem.selectionEnd 来获取光标位置或移动光标。

## 资料
[在Web中实现表情符号的输入](https://zhuanlan.zhihu.com/p/355532173)
