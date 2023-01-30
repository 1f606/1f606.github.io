---
layout:     post
title:      async和defer脚本
subtitle:   
date:       2023-1-22
author:     sq
header-img: 
catalog: true
tags:
    - async
    - defer
---
# async，defer
script 标签会阻塞浏览器加载 HTML，并且脚本无法访问在该 script 标签下的 DOM 元素。

一般会将脚本放在页面底部，这样不会阻塞加载并且可以访问上面的元素，但会导致在最后才加载该脚本，造成明显延迟。更好的解决方法时 async 和 defer特性。
## defer
defer 特性告诉浏览器不要等待脚本。浏览器将继续处理 HTML，构建 DOM。脚本会“在后台”下载，然后等 DOM 构建完成后，脚本才会执行。

特点：
- 具有 defer 特性的脚本不会阻塞页面。
- 具有 defer 特性的脚本总是要等到 DOM 解析完毕，但在 DOMContentLoaded 事件之前执行。
- 具有 defer 特性的脚本执行会保持其相对顺序，就像常规脚本一样。
- defer 特性仅适用于外部脚本。如果 script 没有 src，defer 将被忽略。

## async
async 脚本会在后台加载，并在加载就绪时运行。

- 浏览器不会因 async 脚本而阻塞。
- 其他脚本不会等待 async 脚本加载完成，同样，async 脚本也不会等待其他脚本。
- DOMContentLoaded 和异步脚本不会彼此等待
- async 特性仅适用于外部脚本。如果 script 没有 src，async 将被忽略。

## 动态脚本
使用 JavaScript 动态地创建一个脚本，并将其附加（append）到文档（document）中。

当脚本被附加到文档 (*) 时，脚本就会立即开始加载。

默认情况下，动态脚本的行为是和 async 一致的。除非设置了 `script.async=false`。然后脚本将按照脚本在文档中的顺序执行，就像 defer 那样。
