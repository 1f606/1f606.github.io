---
layout:     post
title:      package.json配置
subtitle:   
date:       2023-06-10
author:     sq
header-img: 
catalog: true
tags:
    - 工程化
---
# package.json 配置

## files [1]
决定项目被作为依赖时，哪些文件会被安装。

默认为\["*"\]，表示所有文件

格式与 .gitignore类似，文件，目录或 \*, \*\*/\*

注意：有些文件必定会被安装或忽略

必定被安装的文件：
* package.json
* README
* CHANGES / CHANGELOG / HISTORY
* LICENSE / LICENCE
* NOTICE
* The file in the "main" field

README, CHANGES, LICENSE 和 NOTICE 忽略大小写和扩展名

以下文件始终被忽略
* .git
* CVS
* .svn
* .hg
* .lock-wscript
* .wafpickle-N
* .DS_Store
* npm-debug.log
* .npmrc
* node_modules
* config.gypi
* package-lock.json (use shrinkwrap instead)
* All files containing a * character (incompatible with Windows)



## 资料
[1] https://docs.npmjs.com/cli/v6/configuring-npm/package-json/#files
