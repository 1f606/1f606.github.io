---
layout:     post
title:      CORS
subtitle:   
date:       2023-4-1
author:     SQ
header-img: 
catalog: true
tags:
    - Web安全
---
# CORS
跨域资源共享(CORS)是一种机制，是W3C标准。它允许浏览器向跨源服务器，发出XMLHttpRequest或Fetch请求。并且整个CORS通信过程都是浏览器自动完成的，不需要用户参与。

实现CORS的关键是服务器需要配置对应的响应头。

当我们发起跨域请求时，如果是简单请求，则不会触发预检，直接发出正常请求。如果是非简单请求，浏览器会自动触发预检，确认目标资源是否支持跨域。

浏览器会根据服务端响应的 header 自动处理剩余的请求，如果响应支持跨域，则继续发出正常请求，如果不支持，浏览器拦截并在控制台显示错误。

## 同源策略
同源策略，是指只有在地址的协议名、域名和端口名均一样的情况下，才允许访问相同的cookie、localStorage、indexDB，以及访问页面的DOM或是发送Ajax
请求。若在不同源的情况下访问，被称为跨域。

同源策略使得不同源的页面不能获取cookie且不能发起Ajax请求，不能获取DOM。

解决方法有同域服务器代理，JSONP，WebSocket（不受同源策略影响）。

## 简单请求和非简单请求
所有浏览器都支持CORS，IE浏览器需要不低于IE10。

浏览器将CORS请求分成两类：简单请求和需发起预检请求的非简单请求。

若请求满足所有下述条件，则该请求可视为“简单请求”：

- 使用下列方法之一：
  - [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET "HTTP GET 方法请求指定的资源。使用 GET 的请求应该只用于获取数据。")
  - [`HEAD`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD "HTTP HEAD 方法 请求资源的头部信息, 并且这些头部与 HTTP GET 方法请求时返回的一致. 该请求方法的一个使用场景是在下载一个大文件前先获取其大小再决定是否要下载, 以此可以节约带宽资源.")
  - [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST "HTTP POST 方法 发送数据给服务器. 请求主体的类型由 Content-Type 首部指定.")
- Fetch 规范定义了[对 CORS 安全的首部字段集合](https://fetch.spec.whatwg.org/#cors-safelisted-request-header)，不得人为设置该集合之外的其他首部字段。该集合为：
  - [`Accept`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept "Accept 请求头用来告知（服务器）客户端可以处理的内容类型，这种内容类型用MIME类型来表示。借助内容协商机制, 服务器可以从诸多备选项中选择一项进行应用，并使用 Content-Type 应答头通知客户端它的选择。浏览器会基于请求的上下文来为这个请求头设置合适的值，比如获取一个CSS层叠样式表时值与获取图片、视频或脚本文件时的值是不同的。")
  - [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language "Accept-Language 请求头允许客户端声明它可以理解的自然语言，以及优先选择的区域方言。借助内容协商机制，服务器可以从诸多备选项中选择一项进行应用， 并使用 Content-Language 应答头通知客户端它的选择。浏览器会基于其用户界面语言为这个请求头设置合适的值，即便是用户可以进行修改，但是这种情况极少发生（因为可增加指纹独特性，通常也不被鼓励）（译者注：通常只在测试网站的多语言支持时手动修改它；或为进一步减少指纹独特性，改为最常见的英文）。")
  - [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language "Content-Language 是一个 entity header （实体消息首部），用来说明访问者希望采用的语言或语言组合，这样的话用户就可以根据自己偏好的语言来定制不同的内容。")
  - [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type "Content-Type 实体头部用于指示资源的MIME类型 media type 。") （需要注意额外的限制）
  - [DPR](http://httpwg.org/http-extensions/client-hints.html#dpr)
  - [Downlink](http://httpwg.org/http-extensions/client-hints.html#downlink)
  - [Save-Data](http://httpwg.org/http-extensions/client-hints.html#save-data)
  - [Viewport-Width](http://httpwg.org/http-extensions/client-hints.html#viewport-width)
  - [Width](http://httpwg.org/http-extensions/client-hints.html#width)
- [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type "Content-Type 实体头部用于指示资源的MIME类型 media type 。") 的值仅限于下列三者之一：
  - `text/plain`
  - `multipart/form-data`
  - `application/x-www-form-urlencoded`
- 请求中的任意`XMLHttpRequestUpload` 对象均没有注册任何事件监听器；`XMLHttpRequestUpload` 对象可以使用 [`XMLHttpRequest.upload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/upload "XMLHttpRequest.upload方法返回一个 XMLHttpRequestUpload对象，用来表示上传的进度。这个对象是不透明的，但是作为一个XMLHttpRequestEventTarget，可以通过对其绑定事件来追踪它的进度。") 属性访问。
- 请求中没有使用 [`ReadableStream`](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream "下面的例子，针对浏览器创建了一个智能的Response来流式化从别的资源处取得的HTML片段。") 对象。

**不满足上面条件的就是非简单请求，需要发送预检请求options，以获知服务器是否允许该实际请求。**

### 预检请求
预检请求要求必须使用 OPTIONS 方法发起。

OPTIONS 预检请求中同时携带了两个头部字段：

`Access-Control-Request-Method`来声明实际请求方法

`Access-Control-Request-Headers` 告知服务器实际请求携带的自定义请求

预检请求的响应

Access-Control-Allow-Origin: 服务器允许的来源，应当为 * 或者包含由 Origin 首部字段所指明的域名。

Access-Control-Allow-Methods: POST, GET, OPTIONS

Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
    
Access-Control-Max-Age: 86400 该预检请求可供缓存的时间长短，单位为秒，默认值是 5 秒。该响应的有效时间为 86400 秒，在有效时间内，浏览器无
须为同一请求再次发起预检请求。请注意，浏览器自身维护了一个最大有效时间，如果该首部字段的值超过了最大有效时间，将不会生效。

预检请求、简单请求和非简单请求都会携带 [`Origin`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin "请求首部字段 Origin 指示了请求来自于哪个站点。该字段仅指示服务器名称，并不包含任何路径信息。该首部用于 CORS 请求。除了不包含路径信息，该字段与 Referer 首部字段相似。") 表明该请求来源

## 预检请求与重定向

大多数浏览器不支持针对于预检请求的重定向。如果一个预检请求发生了重定向，浏览器将报告错误：

在浏览器的实现跟上规范之前，有两种方式规避上述报错行为：

- 在服务端去掉对预检请求的重定向；
- 将实际请求变成一个简单请求。

如果上面两种方式难以做到，我们仍有其他办法：

- 发出一个简单请求（使用  [Response.url](https://developer.mozilla.org/en-US/docs/Web/API/Response/url) 或 [XHR.responseURL](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseURL)）以判断真正的预检请求会返回什么地址。
- 发出另一个请求（真正的请求），使用在上一步通过[Response.url](https://developer.mozilla.org/en-US/docs/Web/API/Response/url) 或 [XMLHttpRequest.responseURL](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseURL)获得的URL。

不过，如果请求是由于存在 Authorization 字段而引发了预检请求，则这一方法将无法使用。这种情况只能由服务端进行更改。

## 附带身份凭证的请求
一般而言，对于跨域 [`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest
或 [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 请求，浏览器**不会**发送身份凭证信息。如果要发送凭证信息，
需要设置 `XMLHttpRequest` 的 `withCredentials` 和 `fetch` 的 `credentials`。

对于附带身份凭证的请求，服务器不得设置 `Access-Control-Allow-Origin 的值为“*”。` 必须指定为明确的请求对应的域名，否则 `Cookie` 不会被携带。

## HTTP 响应首部字段

本节列出了规范所定义的响应首部字段。上一小节中，我们已经看到了这些首部字段在实际场景中是如何工作的。

1. Access-Control-Allow-Origin

2. Access-Control-Expose-Headers
   
   在跨域访问时，XMLHttpRequest对象的getResponseHeader()方法只能拿到一些最基本的响应头，Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma，如果要访问其他头，则需要服务器设置本响应头。
   
   [`Access-Control-Expose-Headers`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Expose-Headers "响应首部 Access-Control-Expose-Headers 列出了哪些首部可以作为响应的一部分暴露给外部。") 头让服务器把允许浏览器访问的头放入白名单，例如：
   
   > Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
   
   这样浏览器就能够通过getResponseHeader访问`X-My-Custom-Header`和 `X-Another-Custom-Header` 响应头了。

3. Access-Control-Max-Age, 表示preflight请求的结果在多少秒内有效。

4. Access-Control-Allow-Credentials
   
   [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials "Access-Control-Allow-Credentials 响应头表示是否可以将对请求的响应暴露给页面。返回true则可以，其他值均不可以。") 头指定了当浏览器的`credentials`设置为true时是否允许浏览器读取response的内容。当用在对preflight预检测请求的响应中时，它指定了实际的请求是否可以使用`credentials`。请注意：简单 GET 请求不会被预检；如果对此类请求的响应中不包含该字段，这个响应将被忽略掉，并且浏览器也不会将相应内容返回给网页。

5. Access-Control-Allow-Methods
   
   [`Access-Control-Allow-Methods`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Methods "响应首部 Access-Control-Allow-Methods 在对 preflight request.（预检请求）的应答中明确了客户端所要访问的资源允许使用的方法或方法列表。") 首部字段用于预检请求的响应。其指明了实际请求所允许使用的 HTTP 方法。

6. Access-Control-Allow-Headers 首部字段用于预检请求的响应。其指明了实际请求中允许携带的首部字段。

## HTTP 请求首部字段

本节列出了可用于发起跨域请求的首部字段。请注意，这些首部字段无须手动设置。 当开发者使用 XMLHttpRequest 对象发起跨域请求时，它们已经被设置就绪。

1. [`Origin`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin "请求首部字段 Origin 指示了请求来自于哪个站点。该字段仅指示服务器名称，并不包含任何路径信息。该首部用于 CORS 请求或者 POST 请求。除了不包含路径信息，该字段与 Referer 首部字段相似。") 首部字段表明预检请求或实际请求的源站。注意，不管是否为跨域请求，ORIGIN 字段总是被发送。

2. [`Access-Control-Request-Method`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Request-Method "The compatibility table in this page is generated from structured data. If you'd like to contribute to the data, please check out https://github.com/mdn/browser-compat-data and send us a pull request.") 首部字段用于预检请求。其作用是，将实际请求所使用的 HTTP 方法告诉服务器。

3. [`Access-Control-Request-Headers`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Request-Headers "请求头  Access-Control-Request-Headers 出现于 preflight request （预检请求）中，用于通知服务器在真正的请求中会采用哪些请求头。") 首部字段用于预检请求。其作用是，将实际请求所携带的首部字段告诉服务器。
