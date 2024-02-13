### 19. webpack 中的 loader 和 plugin 有什么区别

(答案参考童欧巴的一篇`webpack`面试文章哦：[「吐血整理」再来一打 Webpack 面试题 (持续更新)](https://juejin.cn/post/6844904094281236487 "https://juejin.cn/post/6844904094281236487"))

loader 是一个专用于转换文件的转换器，完成压缩、打包、语言编译，**它做的是为了打包**。运行在打包之前。

而 plugin 是一个扩展器，它丰富了 webpack 本身，提供了一些功能的扩展。**它不局限于打包，资源的加载，还有其它的功能**。它是在整个编译周期都起作用。

### 20. HTTP 和 TCP 的不同

在两台计算机相互传递信息时，HTTP 规定了每段数据以什么形式表达才是能够被另外一台计算机理解。

TCP 规定了数据应该怎么传输才能在计算机之间稳定且高效的传递。

### 21. TCP 和 UDP 的区别

1.  TCP 是一个面向连接的、可靠的、基于字节流的传输层协议。
2.  UDP 是一个面向无连接的传输层协议。

TCP 为什么可靠，是因为它有三次握手来保证双方都有接受和发送数据的能力。

字节流服务：将大块数据分割为以报文段为单位的数据包进行管理

### 1. JSONP 的原理以及手写一个实现

基本原理：主要就是利用 `script` 标签的`src`属性没有跨域的限制，通过指向一个需要访问的地址，由服务端返回一个预先定义好的 `Javascript` 函数的调用，并且将服务器数据以该函数参数的形式传递过来，此方法需要前后端配合完成。

执行过程：

*   前端定义一个解析函数 (如: `jsonpCallback = function (res) {}`)
*   通过`params`的形式包装`script`标签的请求参数，并且声明执行函数 (如`cb=jsonpCallback`)
*   后端获取到前端声明的执行函数 (`jsonpCallback`)，并以带上参数且调用执行函数的方式传递给前端
*   前端在`script`标签返回资源的时候就会去执行`jsonpCallback`并通过回调函数的方式拿到数据了。

缺点：

*   只能进行`GET`请求

优点：

*   兼容性好，在一些古老的浏览器中都可以运行

代码实现：

```javascript
<script>
    function JSONP({
        url,
        params = {},
        callbackKey = 'cb',
        callback
    }) {
        // 定义本地的唯一callbackId，若是没有的话则初始化为1
        JSONP.callbackId = JSONP.callbackId || 1;
        let callbackId = JSONP.callbackId;
        // 把要执行的回调加入到JSON对象中，避免污染window
        JSONP.callbacks = JSONP.callbacks || [];
        JSONP.callbacks[callbackId] = callback;
        // 把这个名称加入到参数中: 'cb=JSONP.callbacks[1]'
        params[callbackKey] = `JSONP.callbacks[${callbackId}]`;
        // 得到'id=1&cb=JSONP.callbacks[1]'
        const paramString = Object.keys(params).map(key => {
            return `${key}=${encodeURIComponent(params[key])}`
        }).join('&')
        // 创建 script 标签
        const script = document.createElement('script');
        script.setAttribute('src', `${url}?${paramString}`);
        document.body.appendChild(script);
        // id自增，保证唯一
        JSONP.callbackId++;

    }
    JSONP({
        url: 'http://localhost:8080/api/jsonps',
        params: {
            a: '2&b=3',
            b: '4'
        },
        callbackKey: 'cb',
        callback (res) {
            console.log(res)
        }
    })
    JSONP({
        url: 'http://localhost:8080/api/jsonp',
        params: {
            id: 1
        },
        callbackKey: 'cb',
        callback (res) {
            console.log(res)
        }
    })
</script>


```

### 2. 浏览器为什么要跨域？如果是因为安全的话那小程序或者其他的为什么没有跨域？

跨域的产生来源于现代浏览器所通用的`同源策略`，所谓同源策略，是指只有在地址的：

1.  协议名
2.  域名
3.  端口名

均一样的情况下，才允许访问相同的 cookie、localStorage，以及访问页面的`DOM`或是发送`Ajax`请求。若在不同源的情况下访问，就称为跨域。

注意⚠️：

但是有两种情况：`http`默认的端口号为`80`，`https`默认端口号为`443`。

所以：

```
http://www.example.com:80 === http://www.example.com
https://www.example.com:443 === https://www.example.com


```

**为什么浏览器会禁止跨域？**

**简答**：

首先，跨域只存在于浏览器端。

其次，同源策略主要是为了保证用户信息的安全。

一个是 `Ajax`同源策略使得不同源的页面不能获取`cookie`且不能发起`Ajax`请求，这样在一定程度上防止了`CSRF`攻击。

另一个是`DOM`同源策略，限制了不同源页面不能获取`DOM`，这样可以防止一些恶意网站在自己的网站中利用`iframe`嵌入网站。

**深答**：

*   首先，跨域只存在于浏览器端。浏览器它为`web`提供了访问入口，并且访问的方式很简单，在地址栏输入要访问的地址或者点击某个链接就可以了，正是这种**开放的形态**，所以我们需要对它有所限制。
    
*   所以同源策略它的产生主要是为了保证用户信息的安全，防止恶意的网站窃取数据。分为两种：`Ajax`同源策略与`DOM`同源策略：
    
    *   `Ajax`同源策略它主要做了这两种限制：1. 不同源页面不能获取`cookie`；2. 不同源页面不能发起`Ajax`请求。我认为它是防止`CSRF`攻击的一种方式吧。因为我们知道`cookie`这个东西它主要是为了解决浏览器与服务器会话状态的问题，它本质上是存储在浏览器或本地文件中一个小小的文本文件，那么它里面一般都会存储了用户的一些信息，包括隐私信息。如果没有`Ajax`同源策略，恶意网站只需要一段脚本就可以获取你的`cookie`，从而冒充你的身份去给其它网站发送恶意的请求。
    *   `DOM`同源策略也一样，它限制了不同源页面不能获取`DOM`。例如一个假的网站利用`iframe`嵌套了一个银行网站 [mybank.com](https://link.juejin.cn?target=)，并把宽高或者其它部分调整的和原银行网站一样，仅仅只是地址栏上的域名不同，若是用户没有注意的话就以为这个是个真的网站。如果这时候用户在里面输入了账号密码，如果没有同源策略，那么这个恶意网站就可以获取到银行网站中的`DOM`，也就能拿到用户的输入内容以此来达到窃取用户信息的攻击。
    
    同源策略它算是浏览器安全的第一层屏障吧，因为就像`CSRF`攻击，它只能限制不同源页面`cookie`的获取，但是攻击者还可能通过其它的方式来达到攻击效果。
    
    （注，上面提到的`iframe`限制`DOM`查询，案例如下）
    
    ```
    // HTML
    <iframe ></iframe>
    // JS
    // 由于没有同源策略的限制，钓鱼网站可以直接拿到别的网站的Dom
    const iframe = window.frames['yinhang']
    const node = iframe.document.getElementById('你输入账号密码的Input')
    console.log(`拿到了这个${node}，我还拿不到你刚刚输入的账号密码吗`)
    
    
    ```
    

参考：

*   [segmentfault.com/a/119000001…](https://link.juejin.cn?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000015597029 "https://segmentfault.com/a/1190000015597029")
*   [juejin.cn/post/684490…](https://juejin.cn/post/6844903816060469262 "https://juejin.cn/post/6844903816060469262")

### 3. CORS 跨域的原理

跨域资源共享 (`CORS`) 是一种机制。它允许浏览器向跨源服务器，发出`XMLHttpRequest`或`Fetch`请求。并且整个`CORS`通信过程都是浏览器自动完成的。

而使用这种`跨域资源共享`的前提是，浏览器必须支持这个功能，并且服务器端也必须同意这种`"跨域"`请求。因此实现`CORS`的关键是服务器需要服务器。通常是有以下几个配置：

*   **Access-Control-Allow-Origin**
*   **Access-Control-Allow-Methods**
*   **Access-Control-Allow-Headers**
*   **Access-Control-Allow-Credentials**
*   **Access-Control-Max-Age**

过程分析：

**简单回答**：

*   当我们发起跨域请求时，**如果是非简单请求**，浏览器会帮我们自动触发预检请求，也就是 OPTIONS 请求，用于确认目标资源是否支持跨域。**如果是简单请求，则不会触发预检，直接发出正常请求。**
    
*   浏览器会根据服务端响应的 header 自动处理剩余的请求，如果响应支持跨域，则继续发出正常请求，如果不支持，则在控制台显示错误。
    

**详细回答**：

*   浏览器先根据同源策略对前端页面和后台交互地址做匹配，若同源，则直接发送数据请求；若不同源，则发送跨域请求。
    
*   服务器收到浏览器跨域请求后，根据自身配置返回对应文件头。若未配置过任何允许跨域，则文件头里不包含 `Access-Control-Allow-origin` 字段，若配置过域名，则返回 `Access-Control-Allow-origin + 对应配置规则里的域名的方式`。
    
*   浏览器根据接受到的 响应头里的 `Access-Control-Allow-origin` 字段做匹配，若无该字段，说明不允许跨域，从而抛出一个错误；若有该字段，则对字段内容和当前域名做比对，如果同源，则说明可以跨域，浏览器接受该响应；若不同源，则说明该域名不可跨域，浏览器不接受该响应，并抛出一个错误。
    

在`CORS`中有`简单请求`和`非简单请求`，简单请求是不会触发`CORS`的预检请求的，而非简单请求会。

`“需预检的请求”`要求必须首先使用 [`OPTIONS`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FMethods%2FOPTIONS "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS") 方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。" 预检请求 “的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响。

（关于更多 CORS 的内容可以看我的另一篇文章：[CORS 原理及实现](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FLinDaiDai%2Fniubility-coding-js%2Fblob%2Fmaster%2F%25E8%25AE%25A1%25E7%25AE%2597%25E6%259C%25BA%25E7%25BD%2591%25E7%25BB%259C%2F%25E8%25B7%25A8%25E5%259F%259F%2FCORS%25E5%258E%259F%25E7%2590%2586%25E5%258F%258A%25E5%25AE%259E%25E7%258E%25B0.md "https://github.com/LinDaiDai/niubility-coding-js/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E8%B7%A8%E5%9F%9F/CORS%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0.md")）

### 4. CORS 预请求 OPTIONS 就一定是安全的吗？

### 5. 在深圳的网页上输入百度，是怎么把这个请求发到北京的

这个当时面试官和我说的是，中间会经过很多的站点，比如会经过湖南，或者其它城市，由各个城市的这些站点一层一层分发下去。

### 6. 输入 URL 到页面的呈现

### 7. Vue 的响应式原理

### 8. 那在这个响应式中一个数据改变它是怎么通知要更新的，也就是如何把数据和页面关联起来？

面的最惨的一次... 因为这次面试是当天下午 6 点才去面的，在这之前呆呆已经经过了 3 轮面试的折磨，所以身心疲惫很不在状态。当然最主要的是自己确实准备的还不够充分，其实现在回过头来看看这些题都不太难的...

当天也小小的自闭了一下，整理好状态第二天好好总结吧 😄。

深圳某海外直播公司
---------

4 月 28 日

（当时是电话面，一个小时 20 分钟，问了我大概五六十道题，我能想到的一共是 50 题，还有一些记不起来了）

### 1. CommonJS 和 ES6 模块的区别

*   CommonJS 模块是运行时加载，ES6 Modules 是编译时输出接口
*   CommonJS 输出是值的拷贝；ES6 Modules 输出的是值的引用，被输出模块的内部的改变会影响引用的改变
*   CommonJs 导入的模块路径可以是一个表达式，因为它使用的是`require()`方法；而 ES6 Modules 只能是字符串
*   CommonJS `this`指向当前模块，ES6 Modules `this`指向`undefined`
*   且 ES6 Modules 中没有这些顶层变量：`arguments`、`require`、`module`、`exports`、`__filename`、`__dirname`

关于第一个差异，是因为 CommonJS 加载的是一个对象（即`module.exports`属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

（具体可以看我的这篇文章：[一篇不是标题党的 CommonJS 和 ES6 模块规范讲解](https://juejin.cn/post/6844904145443356680 "https://juejin.cn/post/6844904145443356680"))

### 2. 模块的异步加载

模块的异步加载可以使用`AMD`或者`CMD`规范。

（具体可以看我的这篇文章：[一篇不是标题党的 CommonJS 和 ES6 模块规范讲解](https://juejin.cn/post/6844904145443356680 "https://juejin.cn/post/6844904145443356680")）

### 3. 开发一个模块要考虑哪些问题？

封闭开放式原则、安全性

（应该还有，但是没想到）

### 4. 实现一个一组异步请求按顺序执行你有哪些方法？

1.  利用`reduce`，初始值传入一个`Promise.resolve()`，之后往里面不停的叠加`.then()`。(类似于这里 [juejin.cn/post/684490…](https://juejin.cn/post/6844904077537574919#heading-51 "https://juejin.cn/post/6844904077537574919#heading-51"))
2.  利用`forEach`，本质和`reduce`原理相同。(类似于这里 [juejin.cn/post/684490…](https://juejin.cn/post/6844904077537574919#heading-53 "https://juejin.cn/post/6844904077537574919#heading-53"))
3.  还可以用`ES9`中的`for...await...of`来实现。

### 5. Promise.all() 是并发的还是串行的？

并发的。不过`Promise.all().then()`结果中数组的顺序和`Promise.all()`接收到的数组顺序一致。

### 6. 平时写过哪些正则表达式

*   之前有用过用正则去除输入框的首尾空格，正则表达式为：`var trimReg = /(^\s+)|(\s+$)/g`；不过后来由于`Vue`中有一个修饰符`.trim`，使用起来更方便 (如`v-model.trim="msg"`) 就用这种方式多一些；再或者也可以用`ES10`新出的`trimStart`和`trimEnd`来去除首尾空格。
*   用于校验手机号的正则：`var phoneReg = /^1[3456789]\d{9}$/g`。
*   用正则写一个根据 name 获取 cookie 中的值的方法：

```
function getCookie(name) {
  var match = document.cookie.match(new RegExp('(^| )' + name + '=([^;]*)'));
  if (match) return unescape(match[2]);
}


```

（详细介绍可以看这里：[每日一题 - JS 篇 - 根据 name 获取 cookie 中值的方法](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FLinDaiDai%2Fniubility-coding-js%2Fblob%2Fmaster%2F%25E6%25AF%258F%25E6%2597%25A5%25E4%25B8%2580%25E9%25A2%2598%2F%25E6%25AF%258F%25E6%2597%25A5%25E4%25B8%2580%25E9%25A2%2598-JS%25E7%25AF%2587.md%23%25E7%2594%25A8%25E6%25AD%25A3%25E5%2588%2599%25E5%2586%2599%25E4%25B8%2580%25E4%25B8%25AA%25E6%25A0%25B9%25E6%258D%25AEname%25E8%258E%25B7%25E5%258F%2596cookie%25E4%25B8%25AD%25E7%259A%2584%25E5%2580%25BC%25E7%259A%2584%25E6%2596%25B9%25E6%25B3%2595 "https://github.com/LinDaiDai/niubility-coding-js/blob/master/%E6%AF%8F%E6%97%A5%E4%B8%80%E9%A2%98/%E6%AF%8F%E6%97%A5%E4%B8%80%E9%A2%98-JS%E7%AF%87.md#%E7%94%A8%E6%AD%A3%E5%88%99%E5%86%99%E4%B8%80%E4%B8%AA%E6%A0%B9%E6%8D%AEname%E8%8E%B7%E5%8F%96cookie%E4%B8%AD%E7%9A%84%E5%80%BC%E7%9A%84%E6%96%B9%E6%B3%95")）

### 7. 正则里的非如何实现的

`^`要是放在`[]`里的话就表示`"除了^后面的内容都能匹配"`，也就是非的意思。

例如：

(除了`l`，其它都变成了`"帅"`)

```
var str = 'lindaidai';
console.log(str.replace(/[^l]/g, '帅'));
// l帅帅帅帅帅帅帅帅


```

反之，如果是不在`[]`里的话则表示开头匹配：

(只有`l`变成了`"帅"`)

```
var str = 'lindaidai';
console.log(str.replace(/^l/g, '帅'));


```

### 8. webpack 几种 hash 的实现原理

*   `hash`是跟整个项目的构建相关，只要项目里有文件更改，整个项目构建的`hash`值都会更改，并且全部文件都共用相同的`hash`值。(粒度整个项目)
*   `chunkhash`是根据不同的入口进行依赖文件解析，构建对应的`chunk`(模块)，生成对应的`hash`值。只有被修改的`chunk`(模块) 在重新构建之后才会生成新的`hash`值，不会影响其它的`chunk`。(粒度`entry`的每个入口文件)
*   `contenthash`是跟每个生成的文件有关，每个文件都有一个唯一的`hash`值。当要构建的文件内容发生改变时，就会生成新的`hash`值，且该文件的改变并不会影响和它同一个模块下的其它文件。(粒度每个文件的内容)

（具体可以看我简书上的这篇文章：[www.jianshu.com/p/486453d81…](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F486453d81088 "https://www.jianshu.com/p/486453d81088")）

这里只是说明了三种`hash`的不同... 至于原理暂时没了解。

### 9. webpack 如果使用了 hash 命名，那是每次都会重写生成 hash 吗

这个问题在上一个问题中已经说明了，要看`webpack`的配置。

有三种情况：

*   如果是`hash`的话，是和整个项目有关的，有一处文件发生更改则所有文件的`hash`值都会发生改变且它们共用一个`hash`值；
*   如果是`chunkhash`的话，只和`entry`的每个入口文件有关，也就是同一个`chunk`下的文件有所改动该`chunk`下的文件的`hash`值就会发生改变
*   如果是`contenthash`的话，和每个生成的文件有关，只有当要构建的文件内容发生改变时才会给该文件生成新的`hash`值，并不会影响其它文件。

### 10. webpack 中如何处理图片的？

在`webpack`中有两种处理图片的`loader`：

*   `file-loader`：解决`CSS`等中引入图片的路径问题；(解决通过`url`,`import/require()`等引入图片的问题)
*   `url-loader`：当图片小于设置的`limit`参数值时，`url-loader`将图片进行`base64`编码 (当项目中有很多图片，通过`url-loader`进行`base64`编码后会减少`http`请求数量，提高性能)，大于 limit 参数值，则使用`file-loader`拷贝图片并输出到编译目录中；

（详细使用可以查看这里：[霖呆呆的 webpack 之路 - loader 篇](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FLinDaiDai%2Fniubility-coding-js%2Fblob%2Fmaster%2F%25E5%2589%258D%25E7%25AB%25AF%25E5%25B7%25A5%25E7%25A8%258B%25E5%258C%2596%2Fwebpack%2F%25E9%259C%2596%25E5%2591%2586%25E5%2591%2586%25E7%259A%2584webpack%25E4%25B9%258B%25E8%25B7%25AF-loader%25E7%25AF%2587.md%23file-loader "https://github.com/LinDaiDai/niubility-coding-js/blob/master/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%8C%96/webpack/%E9%9C%96%E5%91%86%E5%91%86%E7%9A%84webpack%E4%B9%8B%E8%B7%AF-loader%E7%AF%87.md#file-loader")）

### 11. 说一下回流和重绘

**回流**：

触发条件：

当我们对 DOM 结构的修改引发 DOM 几何尺寸变化的时候，会发生`回流`的过程。

例如以下操作会触发回流：

1.  一个 DOM 元素的几何属性变化，常见的几何属性有`width`、`height`、`padding`、`margin`、`left`、`top`、`border` 等等, 这个很好理解。
    
2.  使 DOM 节点发生`增减`或者`移动`。
    
3.  读写 `offset`族、`scroll`族和`client`族属性的时候，浏览器为了获取这些值，需要进行回流操作。
    
4.  调用 `window.getComputedStyle` 方法。
    

回流过程：由于 DOM 的结构发生了改变，所以需要从生成 DOM 这一步开始，重新经过`样式计算`、`生成布局树`、`建立图层树`、再到`生成绘制列表`以及之后的显示器显示这整一个渲染过程走一遍，开销是非常大的。

**重绘**：

触发条件：

当 DOM 的修改导致了样式的变化，并且没有影响几何属性的时候，会导致`重绘`(`repaint`)。

重绘过程：由于没有导致 DOM 几何属性的变化，因此元素的位置信息不需要更新，所以当发生重绘的时候，会跳过`生存布局树`和`建立图层树`的阶段，直接到`生成绘制列表`，然后继续进行分块、生成位图等后面一系列操作。

**如何避免触发回流和重绘**：

1.  避免频繁使用 style，而是采用修改`class`的方式。
2.  将动画效果应用到`position`属性为`absolute`或`fixed`的元素上。
3.  也可以先为元素设置`display: none`，操作结束后再把它显示出来。因为在`display`属性为`none`的元素上进行的 DOM 操作不会引发回流和重绘
4.  使用`createDocumentFragment`进行批量的 DOM 操作。
5.  对于 resize、scroll 等进行防抖 / 节流处理。
6.  避免频繁读取会引发回流 / 重绘的属性，如果确实需要多次使用，就用一个变量缓存起来。
7.  利用 CSS3 的`transform`、`opacity`、`filter`这些属性可以实现合成的效果，也就是`GPU`加速。

参考来源：[juejin.cn/post/684490…](https://juejin.cn/post/6844904021308735502#heading-57 "https://juejin.cn/post/6844904021308735502#heading-57")

### 12. 盒模型及如何转换

`box-sizing: content-box`（W3C 盒模型，又名标准盒模型）：元素的宽高大小表现为内容的大小。

`box-sizing: border-box`（IE 盒模型，又名怪异盒模型）：元素的宽高表现为内容 + 内边距 + 边框的大小。背景会延伸到边框的外沿。

### 13. 实现水平垂直居中的几种方式

这里我是按照子弈的总结答的： [juejin.cn/post/684490…](https://juejin.cn/post/6844903928442667015#heading-81 "https://juejin.cn/post/6844903928442667015#heading-81")

*   Flex 布局（子元素是块级元素）

```
.box {
  display: flex;
  width: 100px;
  height: 100px;
  background-color: pink;
}

.box-center{
  margin: auto;
  background-color: greenyellow;
}


```

*   Flex 布局

```
.box {
  display: flex;
  width: 100px;
  height: 100px;
  background-color: pink;
  justify-content: center;
  align-items: center;
}

.box-center{
  background-color: greenyellow;
}


```

*   绝对定位实现 (定位元素定宽定高)

```
.box {
  position: relative;
  height: 100px;
  width: 100px;
  background-color: pink;
}

.box-center{
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
  margin: auto;
  width: 50px;
  height: 50px;
  background-color: greenyellow;
}


```

### 14. flex 的兼容性怎样

**简单回答：**

`IE6~9`不支持，`IE10~11`部分支持`flex的2012版`，但是需要`-ms-`前缀。

其它的主流浏览器包括安卓和`IOS`基本上都支持了。

**详细回答：**

*   IE10 部分支持 2012，需要 - ms - 前缀
*   _Android4.1/4.2-4.3 部分支持 2009_ ，需要 - webkit - 前缀
*   Safari7/7.1/8 部分支持 2012， _需要 - webkit - 前缀_
*   IOS Safari7.0-7.1/8.1-8.3 部分支持 2012，需要 - webkit - 前缀

### 15. 你知道到哪里查看兼容性吗

可以到`Can I use`上去查看，官网地址为：[caniuse.com/](https://link.juejin.cn?target=https%3A%2F%2Fcaniuse.com%2F "https://caniuse.com/")

### 16. 移动端中 css 你是使用什么单位

**比较常用的**：

*   `em`：定义字体大小时以父级的字体大小为基准；定义长度单位时以当前字体大小为基准。例父级`font-size: 14px`，则子级`font-size: 1em;`为`font-size: 14px;`；若定义长度时，子级的字体大小如果为`14px`，则子级`width: 2em;`为`width: 24px`。
*   `rem`：以根元素的字体大小为基准。例如`html`的`font-size: 14px`，则子级`1rem = 14px`。
*   `%`：以父级的宽度为基准。例父级`width: 200px`，则子级`width: 50%;height:50%;`为`width: 100px;height: 100px;`
*   `vw和vh`：基于视口的宽度和高度 (视口不包括浏览器的地址栏工具栏和状态栏)。例如视口宽度为`1000px`，则`60vw = 600px;`
*   `vmin和vmax`：`vmin`为当前`vw` 和`vh`中较小的一个值；`vmax`为较大的一个值。例如视口宽度`375px`，视口高度`812px`，则`100vmin = 375px;`，`100vmax = 812px;`

**不常用的：**

*   `ex和ch`：`ex`以字符`"x"`的高度为基准；例如`1ex`表示和字符`"x"`一样长。`ch`以数字`"0"`的宽度为基准；例如`2ch`表示和 2 个数字`"0"`一样长。

**移动端布局总结**：

1.  移动端布局的方式主要使用 rem 和 flex，可以结合各自的优点，比如 flex 布局很灵活，但是字体的大小不好控制，我们可以使用 rem 和媒体查询控制字体的大小，媒体查询视口的大小，然后不同的上视口大小下设置设置 html 的 font-size。
2.  可单独制作移动端页面也可响应式 pc 端移动端共用一个页面。没有好坏，视情况而定，因势利导

（总结来源：玲珑）

### 17. rem 和 em 的区别

**em:**

定义字体大小时以父级的字体大小为基准；定义长度单位时以当前字体大小为基准。例父级`font-size: 14px`，则子级`font-size: 1em;`为`font-size: 14px;`；若定义长度时，子级的字体大小如果为`14px`，则子级`width: 2em;`为`width: 24px`。

**rem:**

以根元素的字体大小为基准。例如`html`的`font-size: 14px`，则子级`1rem = 14px`。

### 18. 在移动端中怎样初始化根元素的字体大小

一个简易版的初始化根元素字体大小。

页面开头处引入下面这段代码，用于动态计算`font-size`：

(假设你需要的`1rem = 20px`)

```
(function () {
  var html = document.documentElement;
  function onWindowResize() {
    html.style.fontSize = html.getBoundingClientRect().width / 20 + 'px';
  }
  window.addEventListener('resize', onWindowResize);
  onWindowResize();
})();


```

*   `document.documentElement`：获取`document`的根元素
*   `html.getBoundingClientRect().width`：获取`html`的宽度 (窗口的宽度)
*   监听`window`的`resize`事件

一般还需要配合一个`meta`头：

```
<meta  />


```

### 19. 移动端中不同手机 html 默认的字体大小都是一样的吗

如果没有人为取改变根元素字体大小的话，默认是`1rem = 16px`；根元素默认的字体大小是`16px`。

### 20. 你做过哪些动画效果

实话实说没太做过。

### 21. 如果让你实现一个一直旋转的动画你会如何做

_css 代码：_

```
<style>
  .box {
    width: 100px;
    height: 100px;
    background-color: red;
    animation: spin 2s linear infinite;
  }
  @keyframes spin {
    from { transform: rotate(0deg) }
    to { transform: rotate(360deg) }
  }
</style>


```

_html 代码：_

```
<div class="box"></div>


```

### 22. animation 介绍一下

语法：

```
animation: name duration timing-function delay iteration-count direction;


```

<table><thead><tr><th>值</th><th>描述</th></tr></thead><tbody><tr><td><em><a target="_blank" href="https://link.juejin.cn?target=https%3A%2F%2Fwww.w3school.com.cn%2Fcssref%2Fpr_animation-name.asp" title="https://www.w3school.com.cn/cssref/pr_animation-name.asp" ref="nofollow noopener noreferrer">animation-name</a></em></td><td>规定需要绑定到选择器的 keyframe 名称。(mymove)</td></tr><tr><td><em><a target="_blank" href="https://link.juejin.cn?target=https%3A%2F%2Fwww.w3school.com.cn%2Fcssref%2Fpr_animation-duration.asp" title="https://www.w3school.com.cn/cssref/pr_animation-duration.asp" ref="nofollow noopener noreferrer">animation-duration</a></em></td><td>规定完成动画所花费的时间，以秒或毫秒计。(2s)</td></tr><tr><td><em><a target="_blank" href="https://link.juejin.cn?target=https%3A%2F%2Fwww.w3school.com.cn%2Fcssref%2Fpr_animation-timing-function.asp" title="https://www.w3school.com.cn/cssref/pr_animation-timing-function.asp" ref="nofollow noopener noreferrer">animation-timing-function</a></em></td><td>规定动画的速度曲线。(ease|linear|ease-in|cubic-bezier(n,n,n,n))</td></tr><tr><td><em><a target="_blank" href="https://link.juejin.cn?target=https%3A%2F%2Fwww.w3school.com.cn%2Fcssref%2Fpr_animation-delay.asp" title="https://www.w3school.com.cn/cssref/pr_animation-delay.asp" ref="nofollow noopener noreferrer">animation-delay</a></em></td><td>规定在动画开始之前的延迟。(2s)</td></tr><tr><td><em><a target="_blank" href="https://link.juejin.cn?target=https%3A%2F%2Fwww.w3school.com.cn%2Fcssref%2Fpr_animation-iteration-count.asp" title="https://www.w3school.com.cn/cssref/pr_animation-iteration-count.asp" ref="nofollow noopener noreferrer">animation-iteration-count</a></em></td><td>规定动画应该播放的次数。(n | infinite) n 次 / 无限</td></tr><tr><td><em><a target="_blank" href="https://link.juejin.cn?target=https%3A%2F%2Fwww.w3school.com.cn%2Fcssref%2Fpr_animation-direction.asp" title="https://www.w3school.com.cn/cssref/pr_animation-direction.asp" ref="nofollow noopener noreferrer">animation-direction</a></em></td><td>规定是否应该轮流反向播放动画。(normal | alternate) 正常 / 反向</td></tr></tbody></table>

### 23. animation 有一个 steps() 功能符知道吗？

一句话介绍：`steps()`功能符可以让动画不连续。

地位和作用：和贝塞尔曲线 (`cubic-bezier()`修饰符) 一样，都可以作为`animation`的第三个属性值。

和贝塞尔曲线的区别：贝塞尔曲线像是滑梯且有 4 个关键字 (参数)，而`steps`像是楼梯坡道且只有`number`和`position`两个关键字。

语法：

```
steps(number, position)


```

*   number: 数值，表示把动画分成了多少段
*   position: 表示动画是从时间段的开头连续还是末尾连续。支持`start`和`end`两个关键字，含义分别如下：
    *   `start`：表示直接开始。
    *   `end`：表示戛然而止。是默认值。

具体可以看这里：[www.zhangxinxu.com/wordpress/2…](https://link.juejin.cn?target=https%3A%2F%2Fwww.zhangxinxu.com%2Fwordpress%2F2018%2F06%2Fcss3-animation-steps-step-start-end%2F "https://www.zhangxinxu.com/wordpress/2018/06/css3-animation-steps-step-start-end/")

### 24. 用过哪些移动端的调试工具

*   `Chrome`浏览器 -> `more tools` -> `Remote devices` -> `chrome://inspect/#devices`
*   `Mac` + `IOS` + `Safari`

### 25. 说一下原型链

### 26. 详细说一下 instanceof

### 27. V8 的垃圾回收是发生在什么时候？

V8 引擎帮助我们实现了自动的垃圾回收管理，**利用浏览器渲染页面的空闲时间进行垃圾回收。**

### 28. 具体说一下垃圾回收机制

（这里我用的是：[juejin.cn/post/684490…](https://juejin.cn/post/6844904116552990727#heading-20 "https://juejin.cn/post/6844904116552990727#heading-20") 里的总结）

**栈内存的回收：**

栈内存调用栈上下文切换后就被回收，比较简单。

**堆内存的回收：**

V8 的堆内存分为新生代内存和老生代内存，新生代内存是临时分配的内存，存在时间短，老生代内存存在时间长。

新生代内存回收机制：

*   新生代内存容量小，64 位系统下仅有 32M。新生代内存分为 **From、To** 两部分，进行垃圾回收时，先扫描 From，将非存活对象回收，将存活对象顺序复制到 To 中，之后调换 From/To，等待下一次回收

老生代内存回收机制

*   **晋升**：如果新生代的变量经过多次回收依然存在，那么就会被放入老生代内存中
*   **标记清除**：老生代内存会先遍历所有对象并打上标记，然后对正在使用或被强引用的对象取消标记，回收被标记的对象
*   **整理内存碎片**：把对象挪到内存的一端

（当然想要详细了解的话也可以看我的这篇文章：[JavaScript 进阶 - 内存机制 (表情包初探)](https://juejin.cn/post/6844904033317027854 "https://juejin.cn/post/6844904033317027854")）

### 29. 在项目中如何把 http 的请求换成 https

由于我在项目中是会对`axios`做一层封装，所以每次请求的域名也是写在配置文件中，有一个`baseURL`字段专门用于存储它，所以只要改这个字段就可以达到替换所有请求`http`为`https`了。

当然后面我也有了解到：

利用`meta`标签把`http`请求换为`https`:

```
<meta http-equiv ="Content-Security-Policy" content="upgrade-insecure-requests">


```

### 30. 知道 meta 标签有把 http 换成 https 的功能吗？

参考上一题👆。

### 31. http 请求可以怎么拦截

在浏览器和服务器进行传输的时候，可以被`nginx`代理所拦截，也可以被网关拦截。

### 32. https 的加密方式

HTTPS 主要是采用对称密钥加密和非对称密钥加密组合而成的混合加密机制进行传输。

也就是发送密文的一方用 "对方的公钥" 进行加密处理 "对称的密钥"，然后对方在收到之后使用自己的私钥进行解密得到 "对称的密钥"，这在确保双发交换的密钥是安全的前提下使用对称密钥方式进行通信。

### 33. 混合加密的好处

对称密钥加密和非对称密钥加密都有它们各种的优缺点，而混合加密机制就是将两者结合利用它们各自的优点来进行加密传输。

比如既然对称密钥的优点是加解密效率快，那么在客户端与服务端确定了连接之后就可以用它来进行加密传输。不过前提是得解决双方都能安全的拿到这把对称密钥。这时候就可以利用非对称密钥加密来传输这把对称密钥，因为我们知道非对称密钥加密的优点就是能保证传输的内容是安全的。

所以它的好处是即保证了对称密钥能在双方之间安全的传输，又能使用对称加密方式进行通信，这比单纯的使用非对称加密通信快了很多。以此来解决了 HTTP 中内容可能被窃听的问题。

其它 HTTP 相关的问题：

如：

*   HTTPS 的工作流程
    
*   混合加密机制的好处
    
*   数字签名
    
*   ECDHE 握手和 RSA 握手
    
*   向前安全性
    

这些问题都可以看到我的这篇文章：[HTTPS 面试问答](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FLinDaiDai%2Fniubility-coding-js%2Fblob%2Fmaster%2F%25E8%25AE%25A1%25E7%25AE%2597%25E6%259C%25BA%25E7%25BD%2591%25E7%25BB%259C%2FHTTPS%25E9%259D%25A2%25E8%25AF%2595%25E9%2597%25AE%25E7%25AD%2594.md "https://github.com/LinDaiDai/niubility-coding-js/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTPS%E9%9D%A2%E8%AF%95%E9%97%AE%E7%AD%94.md")

### 34. 浏览器如何验证服务器的身份

这道题主要可以从`数字签名`和`数字证书`上来答。

当时我答的时候就把`证书的颁发流程`、`HTTPS`数字证书的验证过程完整的说了一遍就算过了。

具体可以看 [HTTPS 面试问答](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FLinDaiDai%2Fniubility-coding-js%2Fblob%2Fmaster%2F%25E8%25AE%25A1%25E7%25AE%2597%25E6%259C%25BA%25E7%25BD%2591%25E7%25BB%259C%2FHTTPS%25E9%259D%25A2%25E8%25AF%2595%25E9%2597%25AE%25E7%25AD%2594.md "https://github.com/LinDaiDai/niubility-coding-js/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTPS%E9%9D%A2%E8%AF%95%E9%97%AE%E7%AD%94.md")中的第`5、6、7`问。

### 35. ETag 首部字段说一下

这个无非就是配合`If-None-Match`来达到一个`协商缓存`的作用。值为服务器某个资源的唯一标识。

具体可以看我的这篇文章：[霖呆呆你来说说浏览器缓存吧](https://juejin.cn/post/6844904053223358471#heading-6 "https://juejin.cn/post/6844904053223358471#heading-6")

### 36. 你们的 token 一般是存放在哪里的

`Token`其实就是**访问资源的凭证**。

一般是用户通过用户名和密码登录成功之后，服务器将登陆凭证做数字签名，加密之后得到的字符串作为`token`。

它在用户登录成功之后会返回给客户端，客户端主要有这么几种存储方式：

1.  存储在`localStorage`中，每次调用接口的时候都把它当成一个字段传给后台
2.  存储在`cookie`中，让它自动发送，不过缺点就是不能跨域
3.  拿到之后存储在`localStorage`中，每次调用接口的时候放在`HTTP`请求头的`Authorization`字段里

(很明显我答的不够专业，欢迎补充，感谢😊)

### 37. token 会不会被伪造？

(很明显我答的不够专业，欢迎补充，感谢😊)

### 38. redis 中一般用来存什么

用户登录成功之后的一些信息

(很明显我答的不够专业，欢迎补充，感谢😊)

### 39. 前后端如何验证一个用户是否下线了

(很明显我答的不够专业，欢迎补充，感谢😊)

### 40. CSP 白名单知道吗？

(很明显我答的不够专业，欢迎补充，感谢😊)

### 41. nginx 有配置过吗？

只会配置一些跨域方面的问题。

```
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            proxy_pass  http://localhost:8887;
            add_header  Access-Control-Allow-Origin *;
        }
    }
}


```

利用 ngnix 跨域的关键就是在配置文件中设置`server`项，然后设置其中的`location`属性，`proxy_pass`：需要代理的服务器地址，add_header：给响应报文中添加首部字段，例如`Access-Control-Allow-Origin`设置为`*`，即允许所有的请求源请求。

具体可以看：[Yiming 君 - 面试题：nginx 有配置过吗? 反向代理知道吗?](https://juejin.cn/post/6844904148022870023 "https://juejin.cn/post/6844904148022870023")

### 42. 反向代理知道吗？

我们将请求发送到服务器，然后服务器对我们的请求进行转发，我们只需要和代理服务器进行通信就好。所以对于客户端来说，是感知不到服务器的。

### 43. 有用过抓包工具吗？

唔... 没有...

### 44. 你平常用的电脑是 Mac 吗？

(... 默默的点头)

### 45. Fiddler 有用过吗？

唔... 知道...

### 46. Vue 的 diff 算法

这里我是按照:

[mp.weixin.qq.com/s/2xyP4jVev…](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F2xyP4jVevuXrGov_VsWfvA "https://mp.weixin.qq.com/s/2xyP4jVevuXrGov_VsWfvA")

中的第 13 题答的。

### 47. Vue 中 computed 和 methods 的区别

### 48. 例如要获取当前时间你会放到 computed 还是 methods 里？

放在`methods`中，因为`computed`会有惰性，并不能知道`new Date()`的改变。

### 49. 你们的权限功能是怎么做的？

小小的写了一篇文章，可以看这里：[数据权限如何控制](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FLinDaiDai%2Fniubility-coding-js%2Fblob%2Fmaster%2Fother%2F%25E6%2595%25B0%25E6%258D%25AE%25E6%259D%2583%25E9%2599%2590%25E5%25A6%2582%25E4%25BD%2595%25E6%258E%25A7%25E5%2588%25B6.md "https://github.com/LinDaiDai/niubility-coding-js/blob/master/other/%E6%95%B0%E6%8D%AE%E6%9D%83%E9%99%90%E5%A6%82%E4%BD%95%E6%8E%A7%E5%88%B6.md")

### 50. 那你在判断权限的时候是用的字符串匹配还是位运算？

和面试官扯了一堆我数据权限判断的具体过程，其中可能有多个权限：并的情况`000011110001&000011110002`，或的情况`000011110001|000011110002`，以及如何做的权限匹配。最后面试官：

`"所以那还是用的字符串匹配咯？"`

尬... 我比较`low`... 用的字符串匹配...

（哇，真的绝了...1 个小时 20 分钟 50 多道题，答的我口渴😂，不过也可以看出有很多移动端的我都没有答上来，面试官也表示理解，毕竟我主要是以 PC 端为主，所以竟然也算是过了，很感谢这位面试官细心的帮我分析一些问题）

后来有了解这位面试官近期也跳槽去腾讯了，果然面完呆呆之后他就去大厂了，好人一生平安😂。

深圳某国内直播公司
---------

这家公司其实是上家公司的总部，因为面完上家之后，HR 也知道我的顾虑，想要去一个大点的团队，所以就把我推荐去了他们的总部。非常 Nice 的 HR 小姐姐，好感动，就算你看不到我的文章，我也还是要感谢你 😂。

> 5 月 8 日

一面 (前端副总监)

### 1. 输入 URL 到页面呈现

(必问...)

看三元的[《(1.6w 字) 浏览器灵魂之问，请问你能接得住几个？》](https://juejin.cn/post/6844904021308735502 "https://juejin.cn/post/6844904021308735502")

分别从网络，解析，渲染来说

### 2. 为什么说 script 标签建议放在 body 下面？

`JS`代码在加载完之后是立即执行的，且`JS`代码执行时会阻塞页面的渲染。

### 3. 为什么说 script 标签会阻塞页面的渲染呢？渲染线程和 js 引擎线程不是分开的吗？

JS 属于单线程，当我们在加载`script`标签内容的时候，渲染线程会被暂停，因为`script`标签里可能会操作`DOM`的，所以如果你加载`script`标签又同时渲染页面肯定就冲突了，因此说渲染线程 (`GUI`) 和 js 引擎线程互斥。

### 4. 协商缓存说一下

`Last-Modefied`配合`If-Modified-Since`

`ETag`配合`If-None-Match`

也是个常见的问题了，不了解的小伙伴可以看我的这篇文章：[霖呆呆你来说说浏览器缓存吧](https://juejin.cn/post/6844904053223358471 "https://juejin.cn/post/6844904053223358471")

(当时面试官还重复了一下我说的这 4 个头部字段，自己回顾了一下我说的对不对，好可爱～)

### 5. HTTP 中的 Keep-Alive 有了解过吗？

`Keep-Alive`是`HTTP`的一个头部字段`Connection`中的一个值，它是保证我们的`HTTP`请求能建立一个持久连接。也就是说建立一次`TCP`连接即可进行多次请求和响应的交互。它的特点就是只要有一方没有明确的提出断开连接，则保持`TCP`连接状态，减少了`TCP`连接和断开造成的额外开销。

另外，在`HTTP/1.1`中所有的连接默认都是持久连接的，但是`HTTP/1.0`并未标准化。

### 6. 跨域有了解吗？如何解决跨域？

我工作中碰到主要是利用`CORS`来解决跨域问题，说了一下它的原理以及后台需要如何做。

另外说到了`JSONP`的原理，以及它的优点：兼容性好；缺点：只能进行`GET`请求，且有安全问题。

还有说到了`ngnix`反向代理来解决跨域。

*   [CORS 原理及实现](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fb2bdf55e1bf5 "https://www.jianshu.com/p/b2bdf55e1bf5")
*   [JSONP 原理及实现](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F88bb82718517 "https://www.jianshu.com/p/88bb82718517")
*   [面试题：nginx 有配置过吗? 反向代理知道吗?](https://juejin.cn/post/6844904148022870023 "https://juejin.cn/post/6844904148022870023")

其它的，我当时说我有看过一篇文章里面详细的介绍 10 多种跨域解决方案，但是自己没有过多的去了解。

哈哈，其实也就是秋风大大的这篇文章 [10 种跨域解决方案（附终极大招）](https://juejin.cn/post/6844904126246027278 "https://juejin.cn/post/6844904126246027278")

### 7. WebSocket 有了解过吗？它也可以跨域的

这个当时答的没用过。

我知道它是能使得客户端和服务器之间存在持久的连接，而且双方都可以随时开始发送数据，这种方式本质没有使用 HTTP 的响应头，因此也没有跨域的限制。

(多的不会了)

### 8. 前端安全方面？XSS？CSRF？

(必问...)

(以下回答参考子弈小哥哥的[面试分享：两年工作经验成功面试阿里 P6 总结](https://juejin.cn/post/6844903928442667015#heading-60 "https://juejin.cn/post/6844903928442667015#heading-60")

以及蔡徐坤小哥哥的 [2 万字 | 前端基础拾遗 90 问](https://juejin.cn/post/6844904116552990727#heading-48 "https://juejin.cn/post/6844904116552990727#heading-48"))

**XSS**

XSS(Cross Site Script) 跨站脚本攻击。指的是攻击者向网页注入恶意的客户端代码，通过恶意的脚本对客户端网页进行篡改，从而在用户浏览网页时，对用户浏览器进行控制或者获取用户隐私数据的一种攻击方式。

主要是分为三种：

**存储型**：即攻击被存储在服务端，常见的是在评论区插入攻击脚本，如果脚本被储存到服务端，那么所有看见对应评论的用户都会受到攻击。

**反射型**：攻击者将脚本混在 URL 里，服务端接收到 URL 将恶意代码当做参数取出并拼接在 HTML 里返回，浏览器解析此 HTML 后即执行恶意代码

**DOM 型**：将攻击脚本写在 URL 中，诱导用户点击该 URL，如果 URL 被解析，那么攻击脚本就会被运行。和前两者的差别主要在于 DOM 型攻击不经过服务端

如何防御 XSS 攻击

*   **输入检查**：对输入内容中的`script`和`<iframe>`等标签进行转义或者过滤
*   **设置 httpOnly**：很多 XSS 攻击目标都是窃取用户 cookie 伪造身份认证，设置此属性可防止 JS 获取 cookie
*   **开启 CSP**，即开启白名单，可阻止白名单以外的资源加载和运行

**CSRF**

CSRF 攻击 (Cross-site request forgery) 跨站请求伪造。是一种劫持受信任用户向服务器发送非预期请求的攻击方式，通常情况下，它是攻击者借助受害者的 Cookie 骗取服务器的信任，但是它并不能拿到 Cookie，也看不到 Cookie 的内容，它能做的就是给服务器发送请求，然后执行请求中所描述的命令，以此来改变服务器中的数据，也就是并不能窃取服务器中的数据。

防御主要有三种：

验证`Token`：浏览器请求服务器时，服务器返回一个 token，每个请求都需要同时带上 token 和 cookie 才会被认为是合法请求

验证`Referer`：通过验证请求头的 Referer 来验证来源站点，但请求头很容易伪造

设置`SameSite`：设置 cookie 的 SameSite，可以让 cookie 不随跨站请求发出，但浏览器兼容不一

**点击挟持**

*   诱使用户点击看似无害的按钮（实则点击了透明 iframe 中的按钮）.
*   监听鼠标移动事件，让危险按钮始终在鼠标下方.
*   使用 HTML5 拖拽技术执行敏感操作（例如 deploy key）.

预防策略：

1.  服务端添加 X-Frame-Options 响应头, 这个 HTTP 响应头是为了防御用 iframe 嵌套的点击劫持攻击。 这样浏览器就会阻止嵌入网页的渲染。
2.  JS 判断顶层视口的域名是不是和本页面的域名一致，不一致则不允许操作，`top.location.hostname === self.location.hostname`；
3.  敏感操作使用更复杂的步骤（验证码、输入项目名称以删除）。

(这个来源于 LuckyWinty: [www.imooc.com/article/295…](https://link.juejin.cn?target=http%3A%2F%2Fwww.imooc.com%2Farticle%2F295400 "http://www.imooc.com/article/295400"))

### 9. setTimeout 的执行原理 (EventLoop)

(必问...)

(回答参考：[juejin.cn/post/684490…](https://juejin.cn/post/6844904083204079630 "https://juejin.cn/post/6844904083204079630"))

`setTimeout`的运行机制：执行该语句时，是立即把当前定时器代码推入事件队列，当定时器在事件列表中满足设置的时间值时将传入的函数加入任务队列，之后的执行就交给任务队列负责。但是如果此时任务队列不为空，则需等待，所以执行定时器内代码的时间可能会大于设置的时间

说了一下它属于异步任务，然后说了一下还有哪些宏任务以及微任务，最后说了一下`EventLoop`的执行过程。

*   一开始整个脚本作为一个宏任务执行
    
*   执行过程中同步代码直接执行，宏任务进入宏任务队列，微任务进入微任务队列
    
*   当前宏任务执行完出队，检查微任务列表，有则依次执行，直到全部执行完
    
*   执行浏览器 UI 线程的渲染工作
    
*   检查是否有`Web Worker`任务，有则执行
    
*   执行完本轮的宏任务，回到 2，依此循环，直到宏任务和微任务队列都为空
    

（具体可以看这里：[juejin.cn/post/684490…](https://juejin.cn/post/6844904077537574919#heading-1%EF%BC%89 "https://juejin.cn/post/6844904077537574919#heading-1%EF%BC%89")

### 10. requestAnimationFrame 有了解过吗？

(啪啪啪，不长记性，其实之前面试有被问过，但是忘了再去了解了，这就吃亏了，没答上来)

`requestAnimationFrame`是浏览器用于定时循环操作的一个接口，类似于`setTimeout`，主要用途是按帧对网页进行重绘。对于`JS`动画，用`requestAnimationFrame` 会比 `setInterval` 效果更好。

具体可以看：[juejin.cn/post/684490…](https://juejin.cn/post/6844904083204079630 "https://juejin.cn/post/6844904083204079630")

### 11. requestAnimationFrame 和 setTimeout 的区别？

同上...

### 12. 平常工作中 ES6 + 主要用到了哪些？

(下面看着很多，但我肯定不是全答哈，挑了几个来回答)

`ES6`：

1.  `Class`
    
2.  模块`import`和`export`
    
3.  箭头函数
    
4.  函数默认参数
    
5.  `...`扩展运输符允许展开数组
    
6.  解构
    
7.  字符串模版
    
8.  Promise
    
9.  `let const`
    
10.  `Proxy、Map、Set`
    
11.  对象属性同名能简写
    

`ES7`：

1.  `includes`
    
2.  `**`求幂运算符
    

`ES8`：

1.  `async/await`
2.  `Object.values()和Object.entries()`
3.  `padStart()和padEnd()`
4.  `Object.getOwnPropertyDescriptors()`
5.  函数参数允许尾部`,`

`ES9`：

1.  `for...await...of`
2.  `...`展开符合允许展开对象收集剩余参数
3.  `Promise.finally()`
4.  正则中的四个新功能

`ES10`：

1.  `flat()`
2.  `flatMap()`
3.  `fromEntries()`
4.  `trimStart`和`trimEnd`
5.  `matchAll`
6.  `BigInt`
7.  `try/catch`中报错允许没有`err`异常参数
8.  `Symbol.prototype.description`
9.  `Function.toString()`调用时呈现原本源码的样子

（还不了解的小伙伴可以看看浪里哥的这篇：[盘点 ES7、ES8、ES9、ES10 新特性](https://juejin.cn/post/6844904018834096142 "https://juejin.cn/post/6844904018834096142")）

### 13. 如何在前端实现一个图片压缩

实话实话没做过，但是后来面试官告诉我：可以使用`canvas`来实现。具体做法等我写篇文章哈。

(当时我还反问了一句面试官：那批量图片压缩要怎么做呢？把他惊的... 然后他和我说挺复杂的...)

### 14. 你上家公司主要是做什么的？

### 15. 团队多少人呢？

1 个前端 (我)，1 个小程序老哥 (IOS 转行的)，6 个后台。

### 16. 项目中有碰到什么难的问题吗？如何解决的？

例举了我最经典的`bpmn.js`，以此来引出我写了很多关于这方面的教材，以及建立了微信群，为国内的`bpmn.js`社区贡献了一份力量... 怎么高大上怎么来...

当然也有提到我`GitHub`上的`bpmn-chinese-document`项目只有`100`多的`Star`，他说理解，毕竟这东西用的人不是很多。

### 17. 期望薪资多少？

喊了个挺高的数，老哥笑了笑，你这个工作年限我们可能给不到，然后扯了点别的。

### 18. 还有什么想要问我的吗？

*   团队人员分布情况
*   技术栈
*   我进去主要是负责哪块内容
*   年终奖 / 季度奖
*   调薪的频率以及幅度
*   加班多不多

(其实后面这些问题应该是等到 HR 面的时候问的，但是感觉和面试官挺聊的来的我就先打听了)

二面 (CTO)

### 1. JSONP 的实现原理

绝了... 又来

### 2. XSS 攻击以及如何预防？

绝了... 又来 X2

### 3. 不使用框架如何实现组件按需加载以及原理

当时答的是是用`import`来按需引入，以及提到了`Vue.use`。

但后来有去了解，`babel-plugin-import`就可以实现。

### 4. 你们这个是自己写的组件库吗？

估计面试官看错了... 虽然我的项目有个组件库的功能，但是是基于`Ant Design of Vue`二次开发的。

### 5. 还有什么想要问我的吗？？

没了...

三面 (HR)

问的问题有点多，我挑一些记得住的哈

### 1. 第一家公司为什么离职？第二家为什么离职？

### 2. 第一家工资多少？第二家多少？

### 3. 两家公司主要是做什么的？规模是多大？

### 4. 之前都是你一个前端吗？

### 5. 有了解过我们公司吗？感觉怎么样？

### 6. 因为我们现在整个研发团队人也不是太多就 30 多个，前端加上总监可能就 4 个，你会考虑这么一个团队吗？

### 7. 有拿到其它的 offer 吗？

### 8. 拒绝一些 offer 的原因是什么？

### 9. 你的期望薪资我们可能给不到，你怎么想的？

### 10. 平常的兴趣爱好是什么？

### 11. 老家在哪里？...

### 12. 现在住哪里？...

### 13. 还有什么想要问我的吗？？？

面试推荐好文
------

*   [面试分享：两年工作经验成功面试阿里 P6 总结](https://juejin.cn/post/6844903928442667015 "https://juejin.cn/post/6844903928442667015")
*   [2 万字 | 前端基础拾遗 90 问](https://juejin.cn/post/6844904116552990727 "https://juejin.cn/post/6844904116552990727")
*   [一位前端小姐姐的五万字面试宝典](https://juejin.cn/post/6844904121380667399 "https://juejin.cn/post/6844904121380667399")
*   [艺术喵 2 年前端面试心路历程（字节跳动、YY、虎牙、BIGO）| 掘金技术征文](https://juejin.cn/post/6844904113302568973 "https://juejin.cn/post/6844904113302568973")
*   [一年半经验前端社招 7 家大厂 & 独角兽全过经历 | 掘金技术征文](https://juejin.cn/post/6844904137495150599 "https://juejin.cn/post/6844904137495150599")
*   [面试完 50 个人后我写下这篇总结](https://juejin.cn/post/6844904019165446158 "https://juejin.cn/post/6844904019165446158")
*   [「吐血整理」再来一打 Webpack 面试题 (持续更新)](https://juejin.cn/post/6844904094281236487 "https://juejin.cn/post/6844904094281236487")
*   [(1.6w 字) 浏览器灵魂之问，请问你能接得住几个？](https://juejin.cn/post/6844904021308735502 "https://juejin.cn/post/6844904021308735502")

(还有很多大佬的很多好文，不是呆呆不写在这里啊，是因为呆呆暂时只刷了这些，抱歉了😂)

后语
--

你盼世界，我盼望你无`bug`。这篇文章就介绍到了这里。

有总结的不足的地方还喜欢小伙伴能在评论区留言。

我是一只正在努力求生存的呆呆，也在这条路上不断的总结和成长，希望自己能够坚持✊。

`"风浪没平息 我宣告奔跑的意义"`

`"这不是叛逆 我只是淋了一场雨"`

喜欢**霖呆呆**的小伙还希望可以关注霖呆呆的公众号 `LinDaiDai` 或者扫一扫下面的二维码👇👇👇.

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/8/171f4731e341579b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

我会不定时的更新一些前端方面的知识内容以及自己的原创文章🎉

你的鼓励就是我持续创作的主要动力 😊.

相关推荐:

[《全网最详 bpmn.js 教材》](https://juejin.cn/post/6844904017567416328 "https://juejin.cn/post/6844904017567416328")

[《【建议改成】读完这篇你还不懂 Babel 我给你寄口罩》](https://juejin.cn/post/6844904065223098381 "https://juejin.cn/post/6844904065223098381")

[《【建议星星】要就来 45 道 Promise 面试题一次爽到底 (1.1w 字用心整理)》](https://juejin.cn/post/6844904077537574919 "https://juejin.cn/post/6844904077537574919")

[《【建议👍】再来 40 道 this 面试题酸爽继续 (1.2w 字用手整理)》](https://juejin.cn/post/6844904083707396109 "https://juejin.cn/post/6844904083707396109")

[《【何不三连】比继承家业还要简单的 JS 继承题 - 封装篇 (牛刀小试)》](https://juejin.cn/post/6844904094948130824 "https://juejin.cn/post/6844904094948130824")

[《【何不三连】做完这 48 道题彻底弄懂 JS 继承 (1.7w 字含辛整理 - 返璞归真)》](https://juejin.cn/post/6844904098941108232 "https://juejin.cn/post/6844904098941108232")

[《【精】从 206 个 console.log() 完全弄懂数据类型转换的前世今生 (上)》](https://juejin.cn/post/6844903951993667592 "https://juejin.cn/post/6844903951993667592")