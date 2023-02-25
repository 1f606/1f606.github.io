---
layout:     post
title:      WebSocket
subtitle:   
date:       2023-2-25
author:     
header-img: 
catalog: true
tags:
    - WebSocket
---
# WebSocket
在 [RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455) 规范中描述的 WebSocket 协议，提供了一种在浏览器和服务器之间建立持久连
接来交换数据的方法。数据可以作为“数据包”在两个方向上传递，而无需中断连接也无需额外的 HTTP 请求。

## 基础
要打开一个 WebSocket 连接，我们需要在 url 中使用特殊的协议 ws 创建 new WebSocket：
```javascript
let socket = new WebSocket("ws://javascript.info");

let socket = new WebSocket("wss://javascript.info");

// 发送数据
socket.send(data);
```

wss 和 ws 的关系就和 HTTPS 跟 HTTP 的关系一样，`wss://`是基于 TLS 的 WebSocket，类似于 HTTPS 是基于 TLS 的 HTTP。

### 事件
一共有 4 个事件：
- open —— 连接已建立，
- message —— 接收到数据，
- error —— WebSocket 错误，
- close —— 连接已关闭。

示例：
```javascript
let socket = new WebSocket("wss://javascript.info/article/websocket/demo/hello");

socket.onopen = function(e) {
  alert("[open] Connection established");
  alert("Sending to server");
  socket.send("My name is John");
};

socket.onmessage = function(event) {
  alert(`[message] Data received from server: ${event.data}`);
};

socket.onclose = function(event) {
  if (event.wasClean) {
    alert(`[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`);
  } else {
    // 例如服务器进程被杀死或网络中断
    // 在这种情况下，event.code 通常为 1006
    alert('[close] Connection died');
  }
};

socket.onerror = function(error) {
  alert(`[error] ${error.message}`);
};
```

## 深入
### 建立连接
当 new WebSocket(url) 被创建后，它将立即开始连接。

这是由 new WebSocket("wss://javascript.info/chat") 发出的请求的浏览器 header 示例。
```
GET /chat
Host: javascript.info
Origin: https://javascript.info
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Key: Iv8io/9s+lYFgZWcXczP8Q==
Sec-WebSocket-Version: 13
```

- Origin —— 客户端页面的源，例如 https://javascript.info。WebSocket 对象是原生支持跨源的。没有特殊的 header 或其他限制。旧的服务器无法处理 WebSocket，因此不存在兼容性问题。但 Origin header 很重要，因为它允许服务器决定是否使用 WebSocket 与该网站通信。
- Connection: Upgrade —— 表示客户端想要更改协议。
- Upgrade: websocket —— 请求的协议是 “websocket”。
- Sec-WebSocket-Key —— 浏览器随机生成的安全密钥。
- Sec-WebSocket-Version —— WebSocket 协议版本，当前为 13。

如果服务器同意切换为 WebSocket 协议，服务器应该返回响应码 101：
```
101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: hsBlbuDTkk24srzEOTBUlZAlC2g=
```

这里 Sec-WebSocket-Accept 是 Sec-WebSocket-Key，是使用特殊的算法重新编码的。浏览器使用它来确保响应与请求相对应。

### 扩展和子协议
WebSocket 可能还有其他 header，Sec-WebSocket-Extensions 和 Sec-WebSocket-Protocol，它们描述了扩展和子协议。

例如：
- Sec-WebSocket-Extensions: deflate-frame 表示浏览器支持数据压缩。扩展与传输数据有关，扩展了 WebSocket 协议的功能。Sec-WebSocket-Extensions header 由浏览器自动发送，其中包含其支持的所有扩展的列表。
- Sec-WebSocket-Protocol: soap, wamp 表示我们不仅要传输任何数据，还要传输 SOAP 或 WAMP（“The WebSocket Application Messaging Protocol”）协议中的数据。WebSocket 子协议已经在 IANA catalogue 中注册。因此，此 header 描述了我们将要使用的数据格式。

这个可选的 header 是使用 new WebSocket 的第二个参数设置的。它是子协议数组，例如，如果我们想使用 SOAP 或 WAMP：
```javascript
let socket = new WebSocket("wss://javascript.info/chat", ["soap", "wamp"]);
```

服务器应该使用同意使用的协议和扩展的列表进行响应。

例如，这个请求：
```
GET /chat
Host: javascript.info
Upgrade: websocket
Connection: Upgrade
Origin: https://javascript.info
Sec-WebSocket-Key: Iv8io/9s+lYFgZWcXczP8Q==
Sec-WebSocket-Version: 13
Sec-WebSocket-Extensions: deflate-frame
Sec-WebSocket-Protocol: soap, wamp
```

响应：
```
101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: hsBlbuDTkk24srzEOTBUlZAlC2g=
Sec-WebSocket-Extensions: deflate-frame
Sec-WebSocket-Protocol: soap
```

在这里服务器响应 —— 它支持扩展 “deflate-frame”，并且仅支持所请求的子协议中的 SOAP。

### 数据传输
WebSocket 通信由 “frames”（即数据片段）组成，可以从任何一方发送，并且有以下几种类型：
- “text frames” —— 包含各方发送给彼此的文本数据。
- “binary data frames” —— 包含各方发送给彼此的二进制数据。
- “ping/pong frames” 被用于检查从服务器发送的连接，浏览器会自动响应它们。
- 还有 “connection close frame” 以及其他服务 frames。

在浏览器里，我们仅直接使用文本或二进制 frames。

`WebSocket.send()` 方法可以发送文本或二进制数据。

`socket.send(body)` 调用允许 body 是字符串或二进制格式，包括 Blob，ArrayBuffer 等。不需要额外的设置：直接发送它们就可以了。

当我们收到数据时，文本总是以字符串形式呈现。而对于二进制数据，我们可以在 Blob 和 ArrayBuffer 格式之间进行选择。

它是由 socket.binaryType 属性设置的，默认为 "blob"，因此二进制数据通常以 Blob 对象呈现。
对于二进制处理，要访问单个数据字节，我们可以将其改为 "arraybuffer"：

```javascript
socket.binaryType = "arraybuffer";
socket.onmessage = (event) => {
  // event.data 可以是文本（如果是文本），也可以是 arraybuffer（如果是二进制数据）
};
```
