---
layout:     post
title:      XMLHttpRequest
subtitle:   
date:       2023-2-21
author:     
header-img: 
catalog: true
tags:
    - XMLHttpRequest
---
# XMLHttpRequest
XMLHttpRequest 能够发送 HTTP 请求。现如今，我们有一个更为现代的方法叫做 fetch，它的出现使得 XMLHttpRequest 在某种程度上被弃用。

在现代 Web 开发中，出于以下三种原因，我们还在使用 XMLHttpRequest：
1. 历史原因：我们需要支持现有的使用了 XMLHttpRequest 的脚本。
2. 我们需要兼容旧浏览器，并且不想用 polyfill（例如为了使脚本更小）。
3. 我们需要做一些 fetch 目前无法做到的事情，例如跟踪上传进度。

## 基础
XMLHttpRequest 有两种执行模式：同步（synchronous）和异步（asynchronous）。取决于初始化时 async 的值。

要发送请求，需要按如下步骤：

1. 创建 XMLHttpRequest

```javascript
let xhr = new XMLHttpRequest();
```

此构造器没有参数。

2. 初始化

`xhr.open(method, URL, [async, user, password])`

此方法指定请求的主要参数：
- method —— HTTP 方法。通常是 "GET" 或 "POST"。
- URL —— 要请求的 URL，通常是一个字符串，也可以是 URL 对象。
- async —— 如果显式地设置为 false，那么请求将会以同步的方式处理，我们稍后会讲到它。
- user，password —— HTTP 基本身份验证（如果需要的话）的登录名和密码。

3. 发送

`xhr.send([body])`

4. 监听事件获取响应

这三个事件是最常用的：
- load —— 当请求完成（即使 HTTP 状态为 400 或 500 等），并且响应已完全下载。
- error —— 当无法发出请求，例如网络中断或者无效的 URL。
- progress —— 在下载响应期间定期触发，报告已经下载了多少。

下面是一个完整的示例。它从服务器加载 /article/xmlhttprequest/example/load，并打印加载进度：
```javascript
// 1. 创建一个 new XMLHttpRequest 对象
let xhr = new XMLHttpRequest();

// 2. 配置它：从 URL /article/.../load GET-request
xhr.open('GET', '/article/xmlhttprequest/example/load');

// 设置超时时间，超时后取消请求，触发 timeout 事件。
xhr.timeout = 10000; // timeout 单位是 ms，此处即 10 秒

// 3. 通过网络发送请求
xhr.send();

// 4. 当接收到响应后，将调用此函数
xhr.onload = function() {
  if (xhr.status != 200) { // 分析响应的 HTTP 状态
    alert(`Error ${xhr.status}: ${xhr.statusText}`); // 例如 404: Not Found
  } else { // 显示结果
    alert(`Done, got ${xhr.response.length} bytes`); // response 是服务器响应
  }
};

xhr.onprogress = function(event) {
  if (event.lengthComputable) {
    alert(`Received ${event.loaded} of ${event.total} bytes`);
  } else {
    alert(`Received ${event.loaded} bytes`); // 没有 Content-Length
  }

};

xhr.onerror = function() {
  alert("Request failed");
};
```

一旦服务器有了响应，我们可以在以下 xhr 属性中接收结果：

status。HTTP 状态码（一个数字）：200，404，403 等，如果出现非 HTTP 错误，则为 0。

statusText。HTTP 状态消息（一个字符串）：状态码为 200 对应于 OK，404 对应于 Not Found，403 对应于 Forbidden。

response（旧脚本可能用的是 responseText）。服务器 response body。

URL 搜索参数（URL search parameters）

为了向 URL 添加像 ?name=value 这样的参数，并确保正确的编码，我们可以使用 URL 对象：
```javascript
let url = new URL('https://google.com/search');
url.searchParams.set('q', 'test me!');

// 参数 'q' 被编码
xhr.open('GET', url); // https://google.com/search?q=test+me%21
```

## 响应类型
我们可以使用 xhr.responseType 属性来设置响应格式：
- ""（默认）—— 响应格式为字符串，
- "text" —— 响应格式为字符串，
- "arraybuffer" —— 响应格式为 ArrayBuffer（对于二进制数据，请参见 ArrayBuffer，二进制数组），
- "blob" —— 响应格式为 Blob（对于二进制数据，请参见 Blob），
- "document" —— 响应格式为 XML document（可以使用 XPath 和其他 XML 方法）或 HTML document（基于接收数据的 MIME 类型）
- "json" —— 响应格式为 JSON（自动解析）。

## readyState
XMLHttpRequest 的状态（state）会随着它的处理进度变化而变化。可以通过 xhr.readyState 来了解当前状态。

[规范](https://xhr.spec.whatwg.org/#states) 中提到的所有状态如下：
- UNSENT = 0; // 初始状态
- OPENED = 1; // open 被调用
- HEADERS_RECEIVED = 2; // 接收到 response header
- LOADING = 3; // 响应正在被加载（接收到一个数据包）
- DONE = 4; // 请求完成

XMLHttpRequest 对象以 0 → 1 → 2 → 3 → … → 3 → 4 的顺序在它们之间转变。每当通过网络接收到一个数据包，就会重复一次状态 3。

我们可以使用 readystatechange 事件来跟踪它们。

如今，它已被 load/error/progress 事件处理程序所替代。

## 中止请求
`xhr.abort(); // 终止请求`

它会触发 abort 事件，且 xhr.status 变为 0。

## 同步请求
如果在 open 方法中将第三个参数 async 设置为 false，那么请求就会以同步的方式进行。

换句话说，JavaScript 执行在 send() 处暂停，并在收到响应后恢复执行。这有点儿像 alert 或 prompt 命令。

下面是重写的示例，open 的第三个参数为 false：
```javascript
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/hello.txt', false);

try {
  xhr.send();
  if (xhr.status != 200) {
    alert(`Error ${xhr.status}: ${xhr.statusText}`);
  } else {
    alert(xhr.response);
  }
} catch(err) { // 代替 onerror
  alert("Request failed");
}
```

同步调用会阻塞页面内的 JavaScript，直到加载完成。在某些浏览器中，滚动可能无法正常进行。如果一个同步调用执行时间过长，浏览器可能会建议关闭“挂起（hanging）”的网页。

XMLHttpRequest 的很多高级功能在同步请求中都不可用，例如向其他域发起请求或者设置超时。并且，正如你所看到的，没有进度指示。

基于这些原因，同步请求使用的非常少，几乎从不使用。在这我们就不再讨论它了。

## HTTP-header
XMLHttpRequest 允许发送自定义 header，并且可以从响应中读取 header。

HTTP-header 有三种方法：

`setRequestHeader(name, value)`

使用给定的 name 和 value 设置 request header。

例如：

`xhr.setRequestHeader('Content-Type', 'application/json');`

> 一些 header 是由浏览器专门管理的，无法修改，例如 Referer 和 Host。 完整列表请见 [规范](https://xhr.spec.whatwg.org/#the-setrequestheader()-method)。

> XMLHttpRequest 的另一个特点是不能撤销 setRequestHeader。 一旦设置了 header，就无法撤销了。多次设置同名 header，值不会覆盖，会一并发送。

`getResponseHeader(name)`

获取具有给定 name 的 header（Set-Cookie 和 Set-Cookie2 除外）。

`getAllResponseHeaders()`

返回除 Set-Cookie 和 Set-Cookie2 外的所有 response header。

header 以单行形式返回，例如：
```
Cache-Control: max-age=31536000
Content-Length: 4260
Content-Type: image/png
Date: Sat, 08 Sep 2012 16:53:16 GMT
```

header 之间的换行符始终为 "\r\n"（不依赖于操作系统），name 和 value 之间总是以冒号后跟一个空格 ": " 分隔。这是标准格式。

## POST，FormData
要建立一个 POST 请求，我们可以使用内建的 FormData 对象。

1. `xhr.open('POST', ...)` —— 使用 POST 方法。
2. `xhr.send(formData)` 将表单发送到服务器。

## 上传进度
progress 事件仅在下载阶段触发。

这里有另一个对象，它没有方法，它专门用于跟踪上传事件：xhr.upload。

它会生成事件，类似于 xhr，但是 xhr.upload 仅在上传时触发它们：
- loadstart —— 上传开始。
- progress —— 上传期间定期触发。
- abort —— 上传中止。
- error —— 非 HTTP 错误。
- load —— 上传成功完成。
- timeout —— 上传超时（如果设置了 timeout 属性）。
- loadend —— 上传完成，无论成功还是 error。

示例：
```javascript
xhr.upload.onprogress = function(event) {
  alert(`Uploaded ${event.loaded} of ${event.total} bytes`);
};

xhr.upload.onload = function() {
  alert(`Upload finished successfully.`);
};

xhr.upload.onerror = function() {
  alert(`Error during the upload: ${xhr.status}`);
};
```

这是一个真实示例：带有进度指示的文件上传：
```javascript
<input type="file" onchange="upload(this.files[0])">

<script>
function upload(file) {
  let xhr = new XMLHttpRequest();

  // 跟踪上传进度
  xhr.upload.onprogress = function(event) {
    console.log(`Uploaded ${event.loaded} of ${event.total}`);
  };

  // 跟踪完成：无论成功与否
  xhr.onloadend = function() {
    if (xhr.status == 200) {
      console.log("success");
    } else {
      console.log("error " + this.status);
    }
  };

  xhr.open("POST", "/article/xmlhttprequest/post/upload");
  xhr.send(file);
}
</script>
```

## 跨源请求
XMLHttpRequest 可以使用和 fetch 相同的 CORS 策略进行跨源请求。

就像 fetch 一样，默认情况下不会将 cookie 和 HTTP 授权发送到其他域。要启用它们，可以将 xhr.withCredentials 设置为 true：
```javascript
let xhr = new XMLHttpRequest();
xhr.withCredentials = true;

xhr.open('POST', 'http://anywhere.com/request');
...
```

## 可恢复的文件上传
fetch 不允许跟踪上传进度，XMLHttpRequest可以。

### 不太实用的进度事件
要恢复上传，我们需要知道在连接断开前已经上传了多少。

我们有 xhr.upload.onprogress 来跟踪上传进度。不幸的是，它不会帮助我们在此处恢复上传，因为它会在数据 被发送 时触发，但是服务器是否接收到了？浏览器并不知道。

要恢复上传，我们需要 确切地 知道服务器接收的字节数。而且只有服务器能告诉我们，因此，我们将发出一个额外的请求。

具体做法：

1. 首先，创建一个文件 id，以唯一地标识我们要上传的文件：

`let fileId = file.name + '-' + file.size + '-' + file.lastModified;`

在恢复上传时需要用到它，以告诉服务器我们要恢复的内容。如果名称，或大小，或最后一次修改时间发生了更改，则将有另一个 fileId。

2. 向服务器发送一个请求，询问它已经有了多少字节，像这样：

```
let response = await fetch('status', { headers: { 'X-File-Id': fileId } });
// 服务器已有的字节数
let startByte = +await response.text();
```

3. 然后，我们可以使用 Blob 和 slice 方法来发送从 startByte 开始的文件：

```javascript
xhr.open('post', 'upload', true);

// 文件id，以便服务器知道我们要恢复的是哪个文件
xhr.setRequestHeader('X-File-Id', fileId);

// 发送我们要从哪个字节开始恢复，因此服务器知道我们正在恢复
xhr.setRequestHeader('X-Start-Byte', startByte);

xhr.upload.onprogress = e => {
  console.log(`uploaded ${starByte + e.loaded} of ${starByte + e.total}`);
};

xhr.send(file.slice(startByte));
```