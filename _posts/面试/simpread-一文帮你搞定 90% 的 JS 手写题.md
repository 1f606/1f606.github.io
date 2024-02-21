### 23. 写一个通用的事件侦听器函数。

```
const EventUtils = {
  // 视能力分别使用dom0||dom2||IE方式 来绑定事件
  // 添加事件
  addEvent: function(element, type, handler) {
    if (element.addEventListener) {
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent) {
      element.attachEvent("on" + type, handler);
    } else {
      element["on" + type] = handler;
    }
  },
  // 移除事件
  removeEvent: function(element, type, handler) {
    if (element.removeEventListener) {
      element.removeEventListener(type, handler, false);
    } else if (element.detachEvent) {
      element.detachEvent("on" + type, handler);
    } else {
      element["on" + type] = null;
    }
  },
 // 获取事件目标
  getTarget: function(event) {
    return event.target || event.srcElement;
  },
  // 获取 event 对象的引用，取到事件的所有信息，确保随时能使用 event
  getEvent: function(event) {
    return event || window.event;
  },
 // 阻止事件（主要是事件冒泡，因为 IE 不支持事件捕获）
  stopPropagation: function(event) {
    if (event.stopPropagation) {
      event.stopPropagation();
    } else {
      event.cancelBubble = true;
    }
  },
  // 取消事件的默认行为
  preventDefault: function(event) {
    if (event.preventDefault) {
      event.preventDefault();
    } else {
      event.returnValue = false;
    }
  }
};


```

### 24. 使用迭代的方式实现 flatten 函数。

```
var arr = [1, 2, 3, [4, 5], [6, [7, [8]]]]
/** * 使用递归的方式处理 * wrap 内保
存结果 ret * 返回一个递归函数 **/
function wrap() {
    var ret = [];
    return function flat(a) {
        for (var item of
            a) {
                if (item.constructor === Array) {
                    ret.concat(flat(item))
                } else {
                    ret.push(item)
                }
        }
        return ret
    }
} 
console.log(wrap()(arr));


```

### 25. [怎么实现一个 sleep](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fqq_36711388%2Farticle%2Fdetails%2F89787637 "https://blog.csdn.net/qq_36711388/article/details/89787637")

*   sleep 函数作用是让线程休眠，等到指定时间在重新唤起。

```
function sleep(delay) {
  var start = (new Date()).getTime();
  while ((new Date()).getTime() - start < delay) {
    continue;
  }
}

function test() {
  console.log('111');
  sleep(2000);
  console.log('222');
}

test()


```

### 26. [实现正则切分千分位（10000 => 10,000）](https://link.juejin.cn?target=https%3A%2F%2Fmy.oschina.net%2Fu%2F3496297%2Fblog%2F1631309 "https://my.oschina.net/u/3496297/blog/1631309")

```
//无小数点
let num1 = '1321434322222'
num1.replace(/(\d)(?=(\d{3})+$)/g,'$1,')
//有小数点
let num2 = '342243242322.3432423'
num2.replace(/(\d)(?=(\d{3})+\.)/g,'$1,')


```

### 27. [对象数组去重](https://juejin.cn/post/6844904165638963214 "https://juejin.cn/post/6844904165638963214")

```
输入:
[{a:1,b:2,c:3},{b:2,c:3,a:1},{d:2,c:2}]
输出:
[{a:1,b:2,c:3},{d:2,c:2}]


```

*   首先写一个函数把对象中的 key 排序，然后再转成字符串
*   遍历数组利用 Set 将转为字符串后的对象去重

```
function objSort(obj){
    let newObj = {}
    //遍历对象，并将key进行排序
    Object.keys(obj).sort().map(key => {
        newObj[key] = obj[key]
    })
    //将排序好的数组转成字符串
    return JSON.stringify(newObj)
}

function unique(arr){
    let set = new Set();
    for(let i=0;i<arr.length;i++){
        let str = objSort(arr[i])
        set.add(str)
    }
    //将数组中的字符串转回对象
    arr = [...set].map(item => {
        return JSON.parse(item)
    })
    return arr
}


```

### 28. 解析 URL Params 为对象

```
let url = 'http://www.domain.com/?user=anonymous&id=123&id=456&city=%E5%8C%97%E4%BA%AC&enabled';
parseParam(url)
/* 结果
{ user: 'anonymous',
  id: [ 123, 456 ], // 重复出现的 key 要组装成数组，能被转成数字的就转成数字类型
  city: '北京', // 中文需解码
  enabled: true, // 未指定值得 key 约定为 true
}
*/



```

```
function parseParam(url) {
  const paramsStr = /.+\?(.+)$/.exec(url)[1]; // 将 ? 后面的字符串取出来
  const paramsArr = paramsStr.split('&'); // 将字符串以 & 分割后存到数组中
  let paramsObj = {};
  // 将 params 存到对象中
  paramsArr.forEach(param => {
    if (/=/.test(param)) { // 处理有 value 的参数
      let [key, val] = param.split('='); // 分割 key 和 value
      val = decodeURIComponent(val); // 解码
      val = /^\d+$/.test(val) ? parseFloat(val) : val; // 判断是否转为数字

      if (paramsObj.hasOwnProperty(key)) { // 如果对象有 key，则添加一个值
        paramsObj[key] = [].concat(paramsObj[key], val);
      } else { // 如果对象没有这个 key，创建 key 并设置值
        paramsObj[key] = val;
      }
    } else { // 处理没有 value 的参数
      paramsObj[param] = true;
    }
  })

  return paramsObj;
}


```

### 29. 模板引擎实现

```
let template = '我是{{name}}，年龄{{age}}，性别{{sex}}';
let data = {
  name: '姓名',
  age: 18
}
render(template, data); // 我是姓名，年龄18，性别undefined


```

```
function render(template, data) {
  const reg = /\{\{(\w+)\}\}/; // 模板字符串正则
  if (reg.test(template)) { // 判断模板里是否有模板字符串
    const name = reg.exec(template)[1]; // 查找当前模板里第一个模板字符串的字段
    template = template.replace(reg, data[name]); // 将第一个模板字符串渲染
    return render(template, data); // 递归的渲染并返回渲染后的结构
  }
  return template; // 如果模板没有模板字符串直接返回
}


```

### 30. 转化为驼峰命名

```
var s1 = "get-element-by-id"
// 转化为 getElementById


```

```
var f = function(s) {
    return s.replace(/-\w/g, function(x) {
        return x.slice(1).toUpperCase();
    })
}


```

### 31. 查找字符串中出现最多的字符和个数

*   例: abbcccddddd -> 字符最多的是 d，出现了 5 次

```
let str = "abcabcabcbbccccc";
let num = 0;
let char = '';

 // 使其按照一定的次序排列
str = str.split('').sort().join('');
// "aaabbbbbcccccccc"

// 定义正则表达式
let re = /(\w)\1+/g;
str.replace(re,($0,$1) => {
    if(num < $0.length){
        num = $0.length;
        char = $1;        
    }
});
console.log(`字符最多的是${char}，出现了${num}次`);


```

### 32. 图片懒加载

```
let imgList = [...document.querySelectorAll('img')]
let length = imgList.length

const imgLazyLoad = function() {
    let count = 0
    return (function() {
        let deleteIndexList = []
        imgList.forEach((img, index) => {
            let rect = img.getBoundingClientRect()
            if (rect.top < window.innerHeight) {
                img.src = img.dataset.src
                deleteIndexList.push(index)
                count++
                if (count === length) {
                    document.removeEventListener('scroll', imgLazyLoad)
                }
            }
        })
        imgList = imgList.filter((img, index) => !deleteIndexList.includes(index))
    })()
}

// 这里最好加上防抖处理
document.addEventListener('scroll', imgLazyLoad)


```

参考资料
----

*   [高频 JavaScript 手写](https://juejin.cn/post/6895967637580070925 "https://juejin.cn/post/6895967637580070925")
*   [初、中级前端应该要掌握的手写代码实现](https://juejin.cn/post/6844904052237713422 "https://juejin.cn/post/6844904052237713422")
*   [22 道高频 JavaScript 手写](https://juejin.cn/post/6844903911686406158 "https://juejin.cn/post/6844903911686406158")
*   [死磕 36 个 JS 手写题（搞懂后，提升真的大）](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F7KwM6fNM5MICHiIwoRDm-w "https://mp.weixin.qq.com/s/7KwM6fNM5MICHiIwoRDm-w")
