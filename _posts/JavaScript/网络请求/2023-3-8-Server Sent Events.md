---
layout:     post
title:      Server Sent Events
subtitle:   
date:       2023-3-8
author:     javascript.info
header-img: 
catalog: true
tags:
    - 网络请求
---
# Server Sent Events
Server-Sent Events 规范描述了一个内建的类 `EventSource`，它能保持与服务器的持久连接，并允许从中接收事件。

特性：
- 自动重新连接
- 能保持持久连接
- 仅服务器端能发送文本数据

## 获取消息
要开始接收消息，我们只需要创建 `new EventSource(url, [credentials])` 即可。

浏览器将会连接到 url 并保持连接打开，等待message事件。

服务器响应状态码应该为 200，header 为 `Content-Type: text/event-stream`，然后保持此连接并以一种特殊的格式写入消息，就像这样：
```
data: Message 1

data: Message 2
data: of two lines
```

- `data:`后为消息文本，冒号后面的空格是可选的
- 消息以双换行符`\n\n`分割
- 要发送一个换行`\n`，我们可以在要换行的位置立即再发送一个`data:`

对于每个这样的消息，都会生成 message 事件：
```javascript
const eventSource = new EventSource('/events/subscribe');

eventSource.onmessage = function (e) {
}

// or
eventSource.addEventListener('message', e => {
})
```

## 跨源请求
EventSource 支持跨源请求。

远程服务器将会获取到 Origin header，并且必须以 Access-Control-Allow-Origin 响应来处理。

要传递凭证（credentials），我们应该设置第二个参数options的 withCredentials 为 true。

## 重新连接
创建之后，new EventSource 连接到服务器，如果连接断开 —— 则重新连接。

每次重新连接之间有一点小的延迟，默认为几秒钟。

服务器可以使用 `retry:` 来设置需要的延迟响应时间（以毫秒为单位）。
```
retry: 15000
data: set reconnection delay to 15s
```

`retry: `既可以与某些数据一起出现，也可以作为独立的消息出现。

在重新连接之前，浏览器需要等待那么多毫秒。甚至更长，例如，如果浏览器知道（从操作系统）此时没有网络连接，它会等到连接出现，然后重试。

- 如果服务器想要浏览器停止重新连接，那么它应该使用 HTTP 状态码 204 进行响应。
- 如果浏览器想要关闭连接，则应该调用 `eventSource.close()`。

如果响应具有不正确的 Content-Type 或者其 HTTP 状态码不是 301，307，200 和 204，则不会进行重新连接。在这种情况下，将会发出 "error" 事件，并且浏览器不会重新连接。

注意：当连接最终被关闭时，就无法复用，需要创建一个新的EventSource。

## 消息ID
每条消息都应该有一个 id 字段，方便双方确认哪些信息收到和没有收到

```
data: Message 1
id: 1

data: Message 2
data: 2
```

当收到具有 id 的消息时，浏览器会：
1. 将属性 `eventSource.lastEventId` 设置为其值。
2. 重新连接后，发送带有 id 的 header Last-Event-ID，以便服务器可以重新发送后面的消息。

注意：要把 `id:` 放在 `data:` 后，以确保在收到消息后 `lastEventId` 会被更新。

## 连接状态
EventSource 对象有 `readyState` 属性，该属性具有下列值之一：
- EventSource.CONNECTING = 0; // 连接中或者重连中
- EventSource.OPEN = 1; // 已连接
- EventSource.CLOSED = 2; // 连接已关闭

对象创建完成或者连接断开后，它始终是 EventSource.CONNECTING（等于 0）。

## Event 类型
默认情况下 EventSource 对象生成三个事件：
- message —— 收到消息，可以用 event.data 访问。
- open —— 连接已打开。
- error —— 无法建立连接，例如，服务器返回 HTTP 500 状态码。

服务器可以在事件开始时使用 `event:` 指定另一种类型事件。
```
event: join
data: Bob
```

要处理自定义事件，我们必须使用 `addEventListener` 而非 `onmessage`：
```javascript
eventSource.addEventListener('join', e => {
})
```

## 总结
Server Sent Event提供了：
- 在可调的 retry 超时内自动重新连接。
- 用于恢复事件的消息 id，重新连接后，最后接收到的标识符被在 Last-Event-ID header 中发送出去。
- 当前状态位于 readyState 属性中。

这使得 EventSource 成为 WebSocket 的一个可行的替代方案，在很多实际应用中，EventSource 的功能就已经够用了。

EventSource 在所有现代浏览器（除了 IE）中都得到了支持。

语法：

`let source = new EventSource(url, [credentials]);`

第二个参数只有一个可选项：`{ withCredentials: true }`，它允许发送跨源凭证。

EventSource 对象的属性

readyState当前连接状态为 `EventSource.CONNECTING` (=0)，`EventSource.OPEN` (=1)，`EventSource.CLOSED` (=2) 三者之一。

lastEventId最后接收到的 id。重新连接后，浏览器在 header Last-Event-ID 中发送此 id。

EventSource 对象的方法

`close()`关闭连接。

EventSource 对象的事件

- message：接收到的消息，消息数据在 event.data 中。
- open：连接已建立。
- error： 如果发生错误，包括连接丢失（将会自动重连）以及其他致命错误。我们可以检查 `readyState` 以查看是否正在尝试重新连接。

服务器可以在 `event:` 中设置自定义事件名称。应该使用 `addEventListener` 来处理此类事件，而不是使用 `on<event>`。

服务器响应格式

服务器发送由 `\n\n` 分隔的消息。

一条消息可能有以下字段：
- `data:` —— 消息体（body），一系列多个 data 被解释为单个消息，各个部分之间由 \n 分隔。
- `id:` —— 更新 `lastEventId`，重连时以 Last-Event-ID 发送此 id。
- `retry:` —— 建议重连的延迟，以 ms 为单位。无法通过 JavaScript 进行设置。
- `event:` —— 事件名，必须在 data: 之前。

一条消息可以按任何顺序包含一个或多个字段，但是 `id:` 通常排在最后。