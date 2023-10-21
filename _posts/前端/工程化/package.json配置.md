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

## files
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

## main
main 属性指定当前项目的入口，当作为依赖被引入时，main 属性指定的文件导出的对象将被引入。

## browser
当前 npm 包如果是用在浏览器，而不是nodejs，应该使用 browser 属性而不是main。

## bin
bin 属性设置当前包提供的可执行命令。是一个命令名对可执行文件路径的映射对象。在安装依赖时，npm 会用 symlink 全局命令的可执行文件到 prefix/bin, 非全局命令则是 ./node_modules/.bin/.

例如：
```
{ "bin" : { "myapp" : "./cli.js" } }
```

when you install myapp, it'll create a symlink from the cli.js script to /usr/local/bin/myapp.

另外是可执行文件的顶部必须是 `#!/usr/bin/env node`，否则文件里 node 执行不了。

## 资料
[1] https://docs.npmjs.com/cli/v6/configuring-npm/package-json/#files
