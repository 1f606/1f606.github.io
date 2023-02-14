---
layout:     post
title:      formData
subtitle:   
date:       2023-2-13
author:     
header-img: 
catalog: true
tags:
    - formData
---
# formData
## 实例化
语法：
```javascript
let formData = new FormData([formElem]);
```
如果提供了 HTML form 元素，它会自动捕获 form 元素字段。

## 方法
- formData.append(name, value) —— 添加具有给定 name 和 value 的表单字段，
- formData.append(name, blob, fileName) —— fileName代表文件在系统中的名字，
- formData.delete(name) —— 移除带有给定 name 的字段，
- formData.get(name) —— 获取带有给定 name 的字段值，
- formData.has(name) —— 如果存在带有给定 name 的字段，则返回 true，否则返回 false。
- formData.set(name, value)，
- formData.set(name, blob, fileName)。

注意：一个表单可以包含多个具有相同 name 的字段，因此，多次调用 append 将会添加多个具有相同名称的字段。而.set 移除所有具有给定 name 的字段，然后附加一个新字段。因此，它确保了只有一个具有这种 name 的字段，其他的和 append 一样。

可以使用 for..of 循环迭代 formData 字段

表单始终以 Content-Type: multipart/form-data 来发送数据，这个编码允许发送文件。因此 <input type="file"> 字段也能被发送，类似于普通的表单提交。

## 例子
```html
<form id="formElem">
    <input type="text" name="firstName" value="john"/>
    <input type="file" name="picture" accept="image/*"/>
    <input type="submit"/>
</form>
<script>
    formElem.onsubmit = async e => {
      e.preventDefault();
      const response = await fetch('/article/user-avatar', {
        method: 'post',
          body: new FormData(formElem)
      });
      const result = await response.json();
    };
</script>
```

```html
<html>
<body>
<canvas id="canvasElem"></canvas>
<input type="button" onclick="submit()"/>
<script >
    canvasElem.onmousemove = function (e) {
      const ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };
    async function submit () {
      const imageBlob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));
      const formData = new FormData();
      formData.append('image', imageBlob, 'image.png');
      const response = await fetch('/article/image-form', {
        method: 'post',
        body: formData
      });
      const result = await response.json();
    }
</script>
</body>
</html>
```