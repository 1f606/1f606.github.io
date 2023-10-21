---
layout:     post
title:      clip-path
subtitle:   
date:       2023-7-5
author:     sq
header-img: 
catalog: true
tags:
    - CSS
---
# clip-path
坐标系从左上角开始，向右是X坐标方向，向下是Y坐标方向。

clip-path 的 polygon 语法，每个逗号隔开的是一个点，连接成多边形。


## 单标签实现丝带角标
```html
<style>
    .box {
        display: inline-block;
        width: 250px;
        aspect-ratio: 1;
        margin: 20px;
        background: #ccc;
        position: relative;
        font-size: 25px;
        font-family: sans-serif;
    }
    .ribbon {
        --f: 15px; /* control the folded part */

        position: absolute;
        top: 0;
        color: #fff;
        padding: .1em 1.8em;
        background: var(--c,#45ADA8);
        border-bottom :var(--f) solid #0007;
        clip-path: polygon(
                100% calc(100% - var(--f)),100% 100%,calc(100% - var(--f)) calc(100% - var(--f)),var(--f) calc(100% - var(--f)), 0 100%,0 calc(100% - var(--f)),999px calc(100% - 999px),calc(100% - 999px) calc(100% - 999px))
    }
    .right {
        right: 0;
        transform: translate(calc((1 - cos(45deg))*100%), -100%) rotate(45deg);
        transform-origin: 0% 100%;
    }
    .left {
        left: 0;
        transform: translate(calc((cos(45deg) - 1)*100%), -100%) rotate(-45deg);
        transform-origin: 100% 100%;
    }

    /* a fix for firefox that show some strange lines*/
    @supports (-moz-appearance:none) {
        .ribbon {
            background:
                    linear-gradient(to top,#0000 1px,#0005 0 var(--f),#0000 0) border-box,
                    linear-gradient(var(--c,#45ADA8) 0 0) 50%/
       calc(100% - 2px) calc(100% - 2px) no-repeat border-box;
            border-bottom-color: #0000;
        }
    }

    body {
        text-align: center;
    }
</style>
<div class="box">
  <div class="ribbon left">I am a ribbon</div>
</div>
<div class="box">
  <div class="ribbon right" style="--c: #CC333F;--f: 10px;">I can<br> go multiline</div>
</div>

<div class="box">
  <div class="ribbon left" style="--c: #F8CA00;color: #000;--f:20px">Wow !</div>
</div>

<div class="box">
  <div class="ribbon right" style="--f: 0px">50%</div>
</div>
```
