---
layout:     post
title:      File 和 FileReader
subtitle:   
date:       2023-3-9
author:     https://javascript.info/
header-img:
catalog: true
tags:
- 二进制
---
# File 和 FileReader

## File
File 对象继承自 Blob，并扩展了与文件系统相关的功能。附加了两个属性：
- name —— 文件名，
- lastModified —— 最后一次修改的时间戳。

有两种方式可以获取它。

第一种，与 Blob 类似，有一个构造器：

`new File(fileParts, fileName, [options])`

- fileParts —— Blob/BufferSource/String 类型值的数组。
- fileName —— 文件名字符串。
- options —— 可选对象：lastModified —— 最后一次修改的时间戳（整数日期）。

第二种，更常见的是，我们从 `<input type="file" onchange="">` 或拖放或其他浏览器接口来获取文件。在这种情况下，file 将从操作系统（OS）获得 this 信息。

## FileReader
FileReader 是一个对象，其唯一目的是从 Blob（因此也从 File）对象中读取数据。

它使用事件来传递数据，因为从磁盘读取数据可能比较费时间。

构造函数：

`let reader = new FileReader(); // 没有参数`

主要方法:
- `readAsArrayBuffer(blob)` —— 将 blob 读取为二进制格式的 ArrayBuffer。
- `readAsText(blob, [encoding])` —— 将数据读取为给定编码（默认为 utf-8 编码）的文本字符串。
- `readAsDataURL(blob)` —— 读取二进制数据，并将其编码为 base64 的 data url。还有一种用于此的读取文件的替代方案：`URL.createObjectURL(file)`。
- `abort()` —— 取消操作。

读取过程中，有以下事件：
- loadstart —— 开始加载。
- progress —— 在读取过程中出现。
- load —— 读取完成，没有 error。
- abort —— 调用了 abort()。
- error —— 出现 error。
- loadend —— 读取完成，无论成功还是失败。

读取完成后，我们可以通过以下方式访问读取结果：
- reader.result 是结果（如果成功）
- reader.error 是 error（如果失败）。

这是一个读取文件的示例：
```html
<input type="file" onchange="readFile(this)">
<script>
    function readFile(input) {
      let file = input.files[0];
      let reader = new FileReader();
      reader.readAsText(file);
      reader.onload = function() {
        console.log(reader.result);
      };
      reader.onerror = function() {
        console.log(reader.error);
      };
    }
</script>
```

## FileReader 用于 blob
正如我们在 Blob 一章中所提到的，FileReader 不仅可读取文件，还可读取任何 blob。

在 Web Workers 中可以使用同步的 FileReaderSync。它的读取方法 read* 不会生成事件，但是会像常规函数那样返回一个结果。

不过，这仅在 Web Worker 中可用，因为在读取文件的时候，同步调用会有延迟，而在 Web Worker 中，这种延迟并不是很重要。它不会影响页面。

## 总结
File 对象继承自 Blob。除了 Blob 方法和属性外，File 对象还有 name 和 lastModified 属性，以及从文件系统读取的内部功能。我们通常从用户输入如 <input> 或拖放事件来获取 File 对象。

FileReader 对象可以从文件或 blob 中读取数据，可以读取为以下三种格式：
- 字符串（readAsText）。
- ArrayBuffer（readAsArrayBuffer）。
- data url，base-64 编码（readAsDataURL）。

