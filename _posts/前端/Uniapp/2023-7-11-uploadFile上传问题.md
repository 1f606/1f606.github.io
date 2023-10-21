---
layout:     post
title:      uploadFile上传问题
subtitle:   
date:       2023-7-11
author:     sq
header-img: 
catalog: true
tags:
    - Uniapp
---
# uploadFile上传问题
`uni.chooseImage` 方法，会将选择的文件作为 value，文件在本地的路径 objectUrl 作为 key 保存起来。

当调用 `uni.uploadFile` 的时候，参数传入`action` 和 `path`，`file` 参数默认会从内部维护的数据去取 file 对象，找不到的话会调用
XmlHttpRequest 去拿，但是 responseType 设置为 blob，所以拿到 Blob 对象只有 size 和 type 两个属性，而 uploadFile 上传时会固定取这个对
象的 name 属性，不存在就以 file- 加时间戳组成名字，但是没有后缀名（断点调试时，保存拿到的 Blob 对象，转为有名字的 File 对象，上传成功）。上传会失败。

