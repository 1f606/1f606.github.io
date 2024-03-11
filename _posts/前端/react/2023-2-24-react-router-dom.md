---
layout:     post
title:      react-router-dom
subtitle:   
date:       2023-2-24
author:     sq
header-img: 
catalog: true
tags:
    - React
---
## v5
### 安装

```shell
npm i react-router-dom
npm i --save-dev @types/react-router-dom
```

### 基础使用
使用 `BrowserRouter` 组件，子组件用 `Route` 组件来定义路由地址。

### BrowserRouter
`BrowserRouter` 下的 `Route` 组件的 `path` 只要能被匹配，默认都会显示。

解决：`Route` 的 `exact` 或 `Switch` 组件。 

### Route
`Route` 组件接收 `path` 来设置地址，`component` 接收组件，`render` 接收返回元素的渲染函数。

根路径的 `path` 设置 `/`。路由参数设置格式 `/detail/:id`。

#### 路由参数
`component` 属性对应的组件的 props 会接收到 `history`, `location` 和 `match` 三个 props。可以从 `match.params` 中获取路由参数。

对应的 typescript 类型声明导出名字：`RouteComponentProps`，是一个泛型，接收 props 类型。

`history` 对象有路由跳转的方法。

### Switch
`Switch` 组件渲染子组件中的一个匹配路径的优先级最高的 `Route` 组件。最后一个没有 `path` 的 `Route` 是备选项。

### 路由切换

- `withRouter` 接收一个组件，渲染时用返回的组件。此时组件内部可以接收 `history`, `location` 和 `match`。方便跨组件传递路由参数。
- `useHistory`，`useLocation`, `useParams` 和 `useRouteMatch` 可以在函数组件中使用拿到对应组件路由相关对象。
- `Link` 组件，`to` 属性设置路径字符串。
