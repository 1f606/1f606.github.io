---
layout:     post
title:      TextDecoder 和 TextEncoder
subtitle:   
date:       2023-3-9
author:     https://javascript.info/
header-img:
catalog: true
tags:
- 二进制
---
# TextDecoder 和 TextEncoder
## TextDecoder
如果二进制数据实际上是一个字符串怎么办？例如，我们收到了一个包含文本数据的文件。内建的 TextDecoder 对象在给定缓冲区（buffer）和编码
格式（encoding）的情况下，允许将值读取为实际的 JavaScript 字符串。

语法：

`let decoder = new TextDecoder([label], [options]);`

- label —— 编码格式，默认为 utf-8，但同时也支持 big5，windows-1251 等许多其他编码格式。
- options —— 可选对象：fatal —— 布尔值，如果为 true 则为无效（不可解码）字符抛出异常，否则（默认）用字符 \uFFFD 替换无效字符。
- ignoreBOM —— 布尔值，如果为 true 则忽略 BOM（可选的字节顺序 Unicode 标记），很少需要使用。

> 当从某语言向Unicode转化时，如果在某语言中没有该字符，得到的将是Unicode的代码\uFFFD

解码：
`let str = decoder.decode([input], [options]);`

- input —— 要被解码的 BufferSource。
- options —— 可选对象
  - stream —— 对于解码流，为 true，则将传入的数据块（chunk）作为参数重复调用 decoder。在这种情况下，多字节的字符可能偶尔会在块与块之间
    被分割。这个选项告诉 TextDecoder 记住“未完成”的字符，并在下一个数据块来的时候进行解码。

例如：
```javascript
let uint8Array = new Uint8Array([72, 101, 108, 108, 111]);

alert( new TextDecoder().decode(uint8Array) ); // Hello
```

我们可以通过为其创建子数组视图来解码部分缓冲区：

```javascript
let uint8Array = new Uint8Array([0, 72, 101, 108, 108, 111, 0]); // 该字符串位于中间 
// 在不复制任何内容的前提下，创建一个新的视图
let binaryString = uint8Array.subarray(1, -1);

alert( new TextDecoder().decode(binaryString) ); // Hello
```

## TextEncoder
TextEncoder 做相反的事情 —— 将字符串转换为字节。

语法为：
`let encoder = new TextEncoder();`

只支持 utf-8 编码。

它有两种方法：
- encode(str) —— 从字符串返回 Uint8Array。
- encodeInto(str, destination) —— 将 str 编码到 destination 中，该 destination 必须为 Uint8Array。
