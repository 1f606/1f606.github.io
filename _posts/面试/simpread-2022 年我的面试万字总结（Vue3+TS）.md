> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7160962909332307981?share_token=8f293652-22bd-443a-b9e7-0e973113b0fc)

往期文章
====

（1） [2022 年我的面试万字总结（浏览器网络篇）](https://juejin.cn/post/7149438206419664927 "https://juejin.cn/post/7149438206419664927")

（2） [2022 年我的面试万字总结（CSS 篇）](https://juejin.cn/post/7149716216167268366 "https://juejin.cn/post/7149716216167268366")

（3） [2022 年我的面试万字总结（HTML 篇）](https://juejin.cn/post/7150109570609152014 "https://juejin.cn/post/7150109570609152014")

（4） [2022 年我的面试万字总结（JS 篇上）](https://juejin.cn/post/7150462512817782815 "https://juejin.cn/post/7150462512817782815")

（5） [2022 年我的面试万字总结（JS 篇下）](https://juejin.cn/post/7150861842888261668 "https://juejin.cn/post/7150861842888261668")

（6） [2022 年我的面试万字总结（代码篇）](https://juejin.cn/post/7151221875224346637 "https://juejin.cn/post/7151221875224346637")

（7） [2022 年我的面试万字总结（Vue 上）](https://juejin.cn/post/7151597651719356446 "https://juejin.cn/post/7151597651719356446")

（8） [2022 年我的面试万字总结（Vue 下）](https://juejin.cn/post/7151604799077613599 "https://juejin.cn/post/7151604799077613599")

（10） [2022 年我的面试万字总结（Node、webpack、性能优化）](https://juejin.cn/post/7161292246526984228 "https://juejin.cn/post/7161292246526984228")

（11） [2022 年我的面试万字总结（小程序、git）](https://juejin.cn/post/7161584249898795045 "https://juejin.cn/post/7161584249898795045")

一、Vue3
======

1. vue2 和 vue3 的区别
==================

1.  监测机制的改变
    
    *   vue3 中使用了 ES6 的 `Proxy`API 对数据代理，监测的是整个对象，而不再是某个属性。
        
    *   消除了 Vue 2 当中基于 Object.defineProperty 的实现所存在的很多限制
        
    *   vue3 可以监测到对象属性的添加和删除，可以监听数组的变化；
        
    *   vue3 支持 Map、Set、WeakMap 和 WeakSet。
        
2.  `Vue3支持碎片(Fragments)`
    
    *   Vue2 在组件中只有一个根节点。
    *   Vue3 在组件可以拥有多个根节点。
3.  API 模式不同
    
    *   Vue2 与 Vue3 `最大的`区别：Vue2 使用`选项式`API（Options API）对比 Vue3`组合式`API（Composition API）
4.  建立数据的方式不同
    
    *   Vue2：这里把数据放入 data 属性中
        
    *   Vue3：需要使用一个新的 setup() 方法，此方法在组件初始化构造的时候触发。
        
    *   使用以下三步来建立响应式数据:
        
        *   从 vue`引入ref或reactive`
        *   简单数据类型使用`ref()`方法处理，复杂类型数据用`reactive()`处理
        *   使用`setup()`方法来`返回`我们的响应性数据，从而我们的`template`可以`获取`这些响应性数据
5.  生命周期钩子不同 — `Lifecyle Hooks`
    
    *   setup() : 开始创建组件之前，在 beforeCreate 和 created 之前执行。创建的是 data 和 method
    *   onBeforeMount() : 组件挂载到节点上之前执行的函数。
    *   onMounted() : 组件挂载完成后执行的函数。
    *   onBeforeUpdate(): 组件更新之前执行的函数。
    *   onUpdated(): 组件更新完成之后执行的函数。
    *   onBeforeUnmount(): 组件卸载之前执行的函数。
    *   onUnmounted(): 组件卸载完成后执行的函数
    
    若组件被`<keep-alive>`包含，则多出下面两个钩子函
    
    *   onActivated(): 被包含在中的组件，会多出两个生命周期钩子函数。被激活时执行 。
    *   onDeactivated(): 比如从 A 组件，切换到 B 组件，A 组件消失时执行。
6.  `父子传参不同`，子组件通过`defineProps()`进行接收，并且接收这个函数的返回值进行数据操作。
    

**总结： vue3 性能更高, 体积更小, 更利于复用, 代码维护更方便**

2. defineProperty 和 proxy 的区别
=============================

Vue 在实例初始化时遍历 data 中的所有属性，并使用 `Object.defineProperty` 把这些属性全部转为 getter/setter。并 劫持各个属性 getter 和 setter，在数据变化时发布消息给订阅者，触发相应的监听回调，而这之间存在几个问题

*   初始化时需要遍历对象所有 key，如果对象层次较深，性能不好
*   通知更新过程需要维护大量 dep 实例和 watcher 实例，额外占用内存较多
*   Object.defineProperty 无法监听到数组元素的变化，只能通过劫持重写数方法
*   动态新增，删除对象属性无法拦截，只能用特定 set/delete API 代替
*   不支持 Map、Set 等数据结构

Vue3 使用 Proxy 来监控数据的变化。Proxy 是 ES6 中提供的功能，其作用为：用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）。相对于`Object.defineProperty()`，其有以下特点：

1.  **Proxy 直接代理整个对象而非对象属性**，这样只需做一层代理就可以监听同级结构下的所有属性变化，包括新增属性和删除属性。
2.  它的处理方式是在 getter 中去递归响应式，这样的好处是真正访问到的内部属性才会变成响应式，简单的可以说是按需实现响应式，减少性能消耗。
3.  Proxy 可以监听数组的变化。

3. Vue3 Diff 算法和 Vue2 的区别
=========================

我们知道在数据变更触发页面重新渲染，会生成虚拟 DOM 并进行 patch 过程，这一过程在 Vue3 中的优化有如下

**编译阶段的优化：**

*   事件缓存：将事件缓存 (如: @click)，可以理解为变成静态的了
*   静态提升：第一次创建静态节点时保存，后续直接复用
*   添加静态标记：给节点添加静态标记，以优化 Diff 过程

由于编译阶段的优化，除了能更快的生成虚拟 DOM 以外，还使得 Diff 时可以跳过 "永远不会变化的节点"，

**Diff 优化如下**

*   Vue2 是全量 Diff，Vue3 是静态标记 + 非全量 Diff
*   使用最长递增子序列优化了对比流程

根据尤大公布的数据就是 Vue3 `update` 性能提升了 `1.3~2 倍`

4. composition API 与 options API 的区别
====================================

1.  vue2 采用的就是 `optionsAPI`
    
    (1) 优点:**`易于学习和使用`**, 每个代码有着明确的位置 (例如: 数据放 data 中, 方法放 methods 中)
    
    (2) 缺点: 相似的逻辑, 不容易复用, 在大项目中尤为明显
    
    (3) 虽然 optionsAPI 可以通过 mixins 提取相同的逻辑, 但是也并不是特别好维护
    
2.  vue3 新增的就是 `compositionAPI`
    
    (1) compositionAPI 是基于 **逻辑功能** 组织代码的, 一个功能 api 相关放到一起
    
    (2) 即使项目大了, 功能多了, 也能快速定位功能相关的 api
    
    (3) 大大的提升了 `代码可读性` 和 `可维护性`
    
3.  vue3 推荐使用 composition API, 也保留了 options API
    
    即就算不用 composition API, 用 vue2 的写法也完全兼容!!
    

5. Composition API 与 React Hook 很像，区别是什么
========================================

从 React Hook 的实现角度看，React Hook 是根据 useState 调用的顺序来确定下一次重渲染时的 state 是来源于哪个 useState，所以出现了以下限制

*   不能在循环、条件、嵌套函数中调用 Hook
*   必须确保总是在你的 React 函数的顶层调用 Hook
*   useEffect、useMemo 等函数必须手动确定依赖关系

而 Composition API 是基于 Vue 的响应式系统实现的，与 React Hook 的相比

*   声明在 setup 函数内，一次组件实例化只调用一次 setup，而 React Hook 每次重渲染都需要调用 Hook，使得 React 的 GC 比 Vue 更有压力，性能也相对于 Vue 来说也较慢
*   Compositon API 的调用不需要顾虑调用顺序，也可以在循环、条件、嵌套函数中使用
*   响应式系统自动实现了依赖收集，进而组件的部分的性能优化由 Vue 内部自己完成，而 React Hook 需要手动传入依赖，而且必须必须保证依赖的顺序，让 useEffect、useMemo 等函数正确的捕获依赖变量，否则会由于依赖不正确使得组件性能下降。

虽然 Compositon API 看起来比 React Hook 好用，但是其设计思想也是借鉴 React Hook 的。

6. setup 函数
===========

`setup()` 函数是 vue3 中，专门为组件提供的新属性。它为我们使用 vue3 的 `Composition API` 新特性提供了统一的入口, `setup` 函数会在 `beforeCreate` 、`created` 之前执行, vue3 也是取消了这两个钩子，统一用`setup`代替, 该函数相当于一个生命周期函数，vue 中过去的`data`，`methods`，`watch`等全部都用对应的新增`api`写在`setup()`函数中

`setup()` 接收两个参数 `props` 和 `context`。它里面不能使用 `this`，而是通过 context 对象来代替当前执行上下文绑定的对象，context 对象有四个属性：`attrs`、`slots`、`emit`、`expose`

里面通过 `ref` 和 `reactive` 代替以前的 data 语法，`return` 出去的内容，可以在模板直接使用，包括变量和方法

```
<template>
  <div class="container">
    <h1 @click="say()">{{msg}}</h1>
  </div>
</template>
​
<script>
export default {
  setup (props,context) {
    console.log('setup执行了')
    console.log(this)  // undefined
    // 定义数据和函数
    const msg = 'hi vue3'
    const say = () => {
      console.log(msg)
    }
    // Attribute (非响应式对象，等同于 $attrs)
    context.attrs
    // 插槽 (非响应式对象，等同于 $slots)
    context.slots
    // 触发事件 (方法，等同于 $emit)
    context.emit
    // 暴露公共 property (函数)
    context.expose
​
    return { msg , say}
  },
  beforeCreate() {
    console.log('beforeCreate执行了')
    console.log(this)  
  }
}
</script>


```

7. setup 语法糖 （script setup 语法）
==============================

script setup 是在单文件组件 (SFC) 中使用组合式 API 的编译时语法糖。相比于普通的 script 语法更加简洁

要使用这个语法，需要将 `setup` attribute 添加到 `<script>` 代码块上：

格式：

```
<script setup>
console.log('hello script setup')
</script>


```

顶层的绑定会自动暴露给模板，所以定义的变量，函数和 import 导入的内容都可以直接在模板中直接使用

```
<template>
  <div>
    <h3>根组件</h3>
    <div>点击次数：{{ count }}</div>
    <button @click="add">点击修改</button>
  </div>
</template>
​
<script setup>
import { ref } from 'vue'
​
const count = ref(0)
const add = () => {
  count.value++
}
</script>


```

使用 `setup` 语法糖时，不用写 `export default {}`，子组件只需要 `import` 就直接使用，不需要像以前一样在 components 里注册，属性和方法也不用 return。

并且里面不需要用 `async` 就可以直接使用 `await`，因为这样默认会把组件的 `setup` 变为 `async setup`

用语法糖时，props、attrs、slots、emit、expose 的获取方式也不一样了

3.0~3.2 版本变成了通过 import 引入的 API：`defineProps`、`defineEmit`、`useContext`(在 3.2 版本已废弃)，useContext 的属性 `{ emit, attrs, slots, expose }`

3.2 + 版本不需要引入，而直接调用：`defineProps`、`defineEmits`、`defineExpose`、`useSlots`、`useAttrs`

8. reactive、 shallowReactive 函数
===============================

**reactive**

`reactive()` 函数接收一个普通对象，返回一个响应式的数据对象, 相当于 `Vue 2.x` 中的 `Vue.observable()` API，响应式转换是 “深层” 的——它影响所有嵌套属性。基于 proxy 来实现，想要使用创建的响应式数据也很简单，创建出来之后，在`setup`中`return`出去，直接在`template`中调用即可

**shallowReactive**

创建一个响应式代理，它跟踪其自身属性的响应性`shallowReactive`生成非递归响应数据，只监听第一层数据的变化，但不执行嵌套对象的深层响应式转换 (暴露原始值)。

9. ref、 shallowRef 、isRef、toRefs 函数
===================================

**ref**

`ref()` 函数用来根据给定的值创建一个响应式的数据对象，`ref()` 函数调用的返回值是一个对象，这个对象上只包含一个 `value` 属性, 只在 setup 函数内部访问`ref`函数需要加`.value`，其用途创建独立的原始值

`reactive` 将解包所有深层的 `refs`，同时维持 ref 的响应性。当将 `ref`分配给 `reactive` property 时，ref 将被自动解包

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9cc17c3983d423e833219ba6a7f1b2f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**shallowRef**

`ref()` 的浅层作用形式。`shallowRef()` 常常用于对大型数据结构的性能优化或是与外部的状态管理系统集成

**isRef**

`isRef()` 用来判断某个值是否为 `ref()` 创建出来的对象

**toRefs**

**使用场景: 如果对一个响应数据, 进行解构 或者 展开, 会丢失他的响应式特性!**

原因: vue3 底层是对 对象 进行监听劫持

作用: 对一个响应式对象的所有内部属性, 都做响应式处理

1.  reactive/ref 的响应式功能是赋值给对象的, 如果给对象解构或者展开, 会让数据丢失响应式的能力
2.  **使用 toRefs 可以保证该对象展开的每个属性都是响应式的**

10. readonly、isReadonly、shallowReadonly 函数
==========================================

**readonly**

传入`ref`或 `reactive`对象, 并返回一个原始对象的只读代理, 对象内部任何嵌套的属性也都是只读的、 并且是递归只读。

**isReadonly**

检查对象是否是由 `readonly` 创建的只读对象

**shallowReadonly**

`shallowReadonly` 作用只处理对象最外层属性的响应式（浅响应式）的只读，但不执行嵌套对象的深度只读转换 (暴露原始值)

`readonly`和`const`有什么区别？
------------------------

*   `const` 定义的变量不能重新赋值，对应的那个地址不能改变，如果是对象则属性可以被修改。
*   `readonly` 不能给属性重新赋值

11.computed、watch 函数
====================

**computed**

该函数用来创造计算属性，它返回的值是一个 ref 对象。里面可以传方法，或者一个对象，对象中包含`set()`、`get()`方法

**watch**

`watch` 函数用来侦听特定的数据源，并在回调函数中执行副作用。默认情况是懒执行的，也就是说仅在侦听的源数据变更时才执行回调。

```
// 监听单个ref
const money = ref(100)
watch(money, (value, oldValue) => {
  console.log(value)
})
​
// 监听多个ref
const money = ref(100)
const count = ref(0)
watch([money, count], (value) => {
  console.log(value)
})
​
// 监听ref复杂数据
const user = ref({
  name: 'zs',
  age: 18,
})
watch(
  user,
  (value) => {
    console.log('user变化了', value)
  },
  {
    // 深度监听，，，当ref的值是一个复杂数据类型，需要深度监听
    deep: true,
    immediate: true
  }
)
​
// 监听对象的某个属性的变化
const user = ref({
  name: 'zs',
  age: 18,
})
watch(
  () => user.value.name,
  (value) => {
    console.log(value)
  }
)


```

11. watch 和 watchEffect 的区别
===========================

`watch` 作用是对传入的某个或多个值的变化进行监听；触发时会返回新值和老值；第一次不会执行，只有变化时才会重新执行

`watchEffect` 是传入一个函数, 默认会执行一次；不需要传入监听内容，会**自动收集函数内的数据源作为依赖**，在依赖变化的时候又会重新执行
该函数，不会返回新值和老值

共同点是 `watch` 和 `watchEffect` 都有以下四种行为：

*   `停止监听`：组件卸载时都会自动停止监听
*   `清除副作用`：onInvalidate 会作为回调的第三个参数传入
*   `副作用刷新时机`：响应式系统会缓存副作用函数，并异步刷新，避免同一个 tick 中多个状态改变导致的重复调用
*   `监听器调试`：开发模式下可以用 onTrack 和 onTrigger 进行调试

12. Vue3 的生命周期
==============

基本上就是在 Vue2 生命周期钩子函数名基础上加了 `on`；beforeDestory 和 destoryed 更名为 onBeforeUnmount 和 onUnmounted；然后用 setup 代替了两个钩子函数 beforeCreate 和 created；新增了两个开发环境用于调试的钩子

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8832a11723a4c9e9d495fcfa336a6c9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

13. setup 语法下怎么设置 name 属性
=========================

1.  安装插件
    
    yarn add vite-plugin-vue-setup-extend -D
    
2.  配置 vite.config.ts
    

```
import vueSetupExtend from 'vite-plugin-vue-setup-extend'
​
export default defineConfig({
  plugins: [vue(), vueSetupExtend()],
})


```

3.  在标签中使用

```
<script setup >
    // 必须在script标签里面写一点类容，这个插件才会生效,哪怕是注释
</script>


```

14. Vue3 怎么让全局组件有提示
===================

vue3 中如果注册的是局部组件，那么 props 是有类型提示的, 但是如果注册的是全局组件，props 就没有类型提示了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37e98dc55fe7433d890be0a651120cd7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**解决办法**

```
// 在src目录下新建一个文件 global.d.ts
import XtxSkeleton from '@/components/XtxSkeleton/XtxSkeleton.vue'
// 参考：
declare module 'vue' {
  export interface GlobalComponents {
    XtxSkeleton: typeof XtxSkeleton
  }
}
export {}


```

15. Vue3 怎么注册全局自定义指令
====================

```
app.directive('lazy'，{  // app.directive('指令名‘，配置对象)
    mounted(el){
        .......
    }
})


```

16. Vite 和 Webpack 的区别
======================

*   启动方式不一样。vite 在启动的时候不需要打包，所以不用分析模块与模块之间的依赖关系，不用进行编译。这种方式类似按需加载。
同样的，vite 也是这种机制，当浏览器请求某个模块时，再根据需要对模块内容进行编译。按需动态编译可以缩减编译时间，当项目越复杂，模块越多的情况下，vite 明显优于 webpack.
*   热更新方面，效率更高。当改动了某个模块的时候，也只用让浏览器重新请求该模块，不需要像 webpack 那样将模块以及模块依赖的模块全部编译一次。

**缺点**

*   vite 相关生态没有 webpack 完善

17. pinia 的好处
=============

pinia 相比 vuex4，具备完善的 **类型推荐** => 对 TS 支持很友好
    
vuex 只能有一个根级别的状态, pinia 直接就可以定义多个根级别状态
    

18. Vue3 的 v-model 语法
=====================

0.  父组件给子组件传入一个 modelValue 的属性
1.  子组件通知父组件的 update:modelValue 事件，将修改后的值传给父组件
2.  父组件监听 update:modelValue，修改对应的值

```
// 父组件
// 原生写法
<son :model-value="money" @update:modelValue="val=>money = val" />
// v-mode语法糖写法
<son v-model="money" v-mode:house="house" />
    
    
 // 子组件
<button @click="$emit('update:modelValue',modelValue+100)">点我加钱 </button>


```

**好处是什么**

为了整合 .sync 和 v-model

在 Vue2 中，v-mode 只能绑定一个属性，如果需要绑定多个属性则需要借助. sync 修饰符

.sync 修饰符在 Vue3 中已被移除，直接被 v-model 取代。

二、TS
====

**Tuple**

**数组一般由同种类型的值组成，但有时我们需要在单个变量中存储不同类型的值，这时候我们就可以使用元组**。在 JavaScript 中是没有元组的，元组是 TypeScript 中特有的类型，其工作方式类似于数组。

元组可用于定义具有有限数量的未命名属性的类型。每个属性都有一个关联的类型。使用元组时，必须提供每个属性的值。

```
let position: [number, number] = [39.5427, 116.2317]


```

### 1.3 undefined 、 null

默认情况下 `null` 和 `undefined` 是所有类型的子类型。 就是说你可以把 null 和 undefined 赋值给 number 类型的变量。

```
let age: number = null
let realName: string = undefined


```

### 1.4 any、unknown 、never

**any**

在 TypeScript 中，任何类型都可以被归为 `any` 类型。这让`any`类型成为了类型系统的顶级类型（也被称作全局超级类型）。但是不建议使用 any，不然就丧失了 TS 提供的保护机制，失去了使用 TS 的意义。

**unknown**

所有类型也都可以赋值给 `unknown`。这使得 `unknown` 成为 TypeScript 类型系统的另一种顶级类型（另一种是 `any`）。它的定义和 `any` 定义很像，但是它是一个安全类型，使用 `unknown` 做任何事情都是不合法的。

**never**

`never`类型表示的是那些永不存在的值的类型。

有些情况下值会永不存在，比如，

*   如果一个函数执行时抛出了异常，那么这个函数永远不存在返回值，因为抛出异常会直接中断程序运行。
    
*   函数中执行无限循环的代码，使得程序永远无法运行到函数返回值那一步。
    
    never 类型是任何类型的子类型，也可以赋值给任何类型。
    
    **没有类型是 never 的子类型**，没有类型可以赋值给 never 类型（除了 never 本身之外）。 即使 `any`也不可以赋值给 never 。
    

②、 函数类型
-------

*   函数的类型实际上指的是：`函数参数`和`返回值`的类型
    
*   为函数指定类型的两种方式：
    
    1.  单独指定参数、返回值的类型
    2.  同时指定参数、返回值的类型

**单独指定参数、返回值的类型**：

```
// 函数声明
function add(num1: number, num2: number): number {
  return num1 + num2
}
​
// 箭头函数
const add = (num1: number, num2: number): number => {
  return num1 + num2
}


```

**同时指定参数、返回值的类型:**

```
type AddFn = (num1: number, num2: number) => number
​
const add: AddFn = (num1, num2) => {
  return num1 + num2
}


```

### 2.2 void 类型

如果函数没有返回值，那么，函数返回值类型为：`void`

```
function greet(name: string): void {
  console.log('Hello', name)
}


```

注意：

*   如果一个函数没有返回值，此时，在 TS 的类型中，应该使用 `void` 类型

```
// 如果什么都不写，此时，add 函数的返回值类型为： void
const add = () => {}
// 这种写法是明确指定函数返回值类型为 void，与上面不指定返回值类型相同
const add = (): void => {}
​
// 但，如果指定 返回值类型为 undefined，此时，函数体中必须显示的 return undefined 才可以
const add = (): undefined => {
  // 此处，返回的 undefined 是 JS 中的一个值
  return undefined
}


```

### **2.3 可选参数**

*   使用函数实现某个功能时，参数可以传也可以不传。这种情况下，在给函数参数指定类型时，就用到**可选参数**了
*   比如，数组的 slice 方法，可以 `slice()` 也可以 `slice(1)` 还可以 `slice(1, 3)`

```
function mySlice(start?: number, end?: number): void {
  console.log('起始索引：', start, '结束索引：', end)
}


```

*   可选参数：在可传可不传的参数名称后面添加 `?`（问号）
*   注意：**可选参数只能出现在参数列表的最后**，也就是说可选参数后面不能再出现必选参数

### 2.4 默认参数

跟 JS 的写法一样，在入参里定义初始值。

和可选参数不同的是，默认参数可以不放在函数入参的最后面

### 2.5 函数重载

函数重载或方法重载是使用相同名称和不同参数数量或类型创建多个方法的一种能力。

不必太纠结函数重载，知道有这个概念即可，平时一般用泛型来解决类似问题。

③、 对象类型
-------

JS 中的对象是由属性和方法构成的，而 **TS 对象的类型就是在描述对象的结构**（有什么类型的属性和方法）

对象类型的写法:

```
// 空对象
let person: {} = {}
​
// 有属性的对象
let person: { name: string } = {
  name: '同学'
}
​
// 既有属性又有方法的对象
// 在一行代码中指定对象的多个属性类型时，使用 `;`（分号）来分隔
let person: { name: string; sayHi(): void } = {
  name: 'jack',
  sayHi() {}
}
​
// 对象中如果有多个类型，可以换行写：
// 通过换行来分隔多个属性类型，可以去掉 `;`
let person: {
  name: string
  sayHi(): void
} = {
  name: 'jack',
  sayHi() {}
}
​
// 方法的类型也可以使用箭头函数形式
{
    greet(name: string):string,
    greet: (name: string) => string
}


```

### 3.2 对象可选属性

*   对象的属性或方法，也可以是可选的，此时就用到**可选属性**了
*   比如，我们在使用 `axios({ ... })` 时，如果发送 GET 请求，method 属性就可以省略
*   可选属性的语法与函数可选参数的语法一致，都使用 `?` 来表示

```
type Config = {
  url: string
  method?: string
}
​
function myAxios(config: Config) {
  console.log(config)
}


```

④、 interface 接口类型
-----------------

当一个对象类型被多次使用时，一般会使用接口（`interface`）来描述对象的类型，达到复用的目的

*   解释：
    
    0.  使用 `interface` 关键字来声明接口
    1.  接口名称 (比如，此处的 IPerson)，可以是任意合法的变量名称，推荐以 `I` 开头
    2.  声明接口后，直接使用接口名称作为变量的类型
    3.  因为每一行只有一个属性类型，因此，属性类型后没有 ;(分号)

```
interface IPerson {
  name: string
  age: number
  sayHi(): void
}
​
​
let person: IPerson = {
  name: 'jack',
  age: 19,
  sayHi() {}
}


```

### 4.2 接口继承

*   如果两个接口之间有相同的属性或方法，可以将**公共的属性或方法抽离出来，通过继承来实现复用**
*   比如，这两个接口都有 x、y 两个属性，重复写两次，可以，但很繁琐

```
interface Point2D { x: number; y: number }
// 继承 Point2D
interface Point3D extends Point2D {
  z: number
}


```

### 4.3 interface 和 type 的区别

*   interface（接口）和 type（类型别名）的对比：
    
*   相同点：都可以给对象指定类型
    
*   不同点:
    
    *   **interface** ：
        
        *   只能为对象指定类型
        *   可以使用 extends 继承
        *   多个同名的 interface 会合并
    *   **type**：
        
        *   不仅可以为对象指定类型，实际上可以为任意类型指定别名
        *   可以使用 & 运算符实现继承效果
        *   多个同名的 type 会报错

⑤、 联合类型
-------

```
let arr: (number | string)[] = [1, 'a', 3, 'b']


```

*   解释：`|`（竖线）在 TS 中叫做**联合类型**，即：由两个或多个其他类型组成的类型，表示可以是这些类型中的任意一种
*   注意：这是 TS 中联合类型的语法，只有一根竖线，不要与 JS 中的或（|| 或）混淆了

⑥、字面量类型、枚举 (enum) 类型
--------------------

**字面量类型**

```
const str = 'Hello TS'


```

str 是一个常量 (const)，它的值不能变化只能是'Hello TS'，所以，它的类型为:'Hello TS'

*   注意：此处的'Hello TS'，就是一个**字面量类型**，也就是说某个特定的字符串也可以作为 TS 中的类型
    
*   任意的 JS 字面量（比如，对象、数字等）都可以作为类型使用
    
    *   字面量：`{ name: 'jack' }` `[]` `18` `20` `'abc'` `false` `function() {}`

**枚举类型**

在任何项目开发中，我们都会遇到定义常量的情况，常量就是指不会被改变的值。

TS 中我们使用 `const` 来声明常量，但是有些取值是在一定范围内的一系列常量，比如一周有七天，比如方向分为上下左右四个方向。

这时就可以使用枚举（Enum）来定义。

```
// 创建枚举
enum Direction { Up, Down, Left, Right }
​
// 使用枚举类型
function changeDirection(direction: Direction) {
  console.log(direction)
}
​
// 调用函数时，需要应该传入：枚举 Direction 成员的任意一个
// 类似于 JS 中的对象，直接通过 点（.）语法 访问枚举的成员
changeDirection(Direction.Up)


```

**枚举实现原理**

*   枚举是 TS 为数不多的非 JavaScript 类型级扩展 (不仅仅是类型) 的特性之一
*   因为：其他类型仅仅被当做类型，而枚举不仅用作类型，还提供值 (枚举成员都是有值的)
*   也就是说，其他的类型会在编译为 JS 代码时自动移除。但是，**枚举类型会被编译为 JS 代码**

```
enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT'
}
​
// 会被编译为以下 JS 代码：
var Direction;
​
(function (Direction) {
  Direction['Up'] = 'UP'
  Direction['Down'] = 'DOWN'
  Direction['Left'] = 'LEFT'
  Direction['Right'] = 'RIGHT'
})(Direction || Direction = {})


```

*   说明：枚举与前面讲到的字面量类型 + 联合类型组合的功能类似，都用来表示一组明确的可选值列表
*   一般情况下，**推荐使用字面量类型 + 联合类型组合的方式**，因为相比枚举，这种方式更加直观、简洁、高效

4. TS 中的 class 类的关键字
====================

**extends**

在 TypeScript 中，我们可以通过 `extends` 关键字来实现继承

**super**

子类没有定义自己的属性，可以不写 super ，但是如果子类有自己的属性，就要用到 super 关键字来把父类的属性继承过来。

**public**

`public`，公有的，一个类里默认所有的方法和属性都是 public。

**private**

`private`，私有的，只属于这个类自己，它的实例和继承它的子类都访问不到。

**protected**

`protected` 受保护的，继承它的子类可以访问，实例不能访问。

**static**

`static` 是静态属性，可以理解为是类上的一些常量，实例不能访问。

**abstract**

`abstract` 关键字来定义抽象类和抽象方法

抽象类，是指**只能被继承，但不能被实例化的类**，就这么简单。

抽象类有两个特点：

*   抽象类不允许被实例化
*   抽象类中的抽象方法必须被子类实现

**# （私有字段）**

私有字段与常规属性（甚至使用 `private` 修饰符声明的属性）不同，私有字段要牢记以下规则：

*   私有字段以 `#` 字符开头，有时我们称之为私有名称；
*   每个私有字段名称都唯一地限定于其包含的类；
*   不能在私有字段上使用 TypeScript 可访问性修饰符（如 public 或 private）；
*   私有字段不能在包含的类之外访问，甚至不能被检测到。

5. 类型推断、类型断言、非空断言
=================

5.1 **类型推断**
------------

在 TS 中，某些没有明确指出类型的地方，**TS 的类型推论机制会帮助提供类型** 换句话说：由于类型推论的存在，有些场合下的类型注解可以省略不写

发生类型推论的 2 种常见场景:

0.  声明变量并初始化时
1.  决定函数返回值时

```
// 变量 age 的类型被自动推断为：number
let age = 18
​
// 函数返回值的类型被自动推断为：number
function add(num1: number, num2: number) {
  return num1 + num2
}


```

5.2 **类型断言**
------------

有时候你会比 TS 更加明确一个值的类型，此时，可以使用类型断言来指定**更具体**的类型。

类型断言好比其他语言里的类型转换，但是不进行特殊的数据检查和解构。它没有运行时的影响，只是在编译阶段起作用。

```
const aLink = document.getElementById('link') as HTMLAnchorElement


```

另一种语法，使用 `<>` 语法，这种语法形式不常用，知道即可:

```
// 尖括号语法，知道即可：
const aLink = <HTMLAnchorElement>document.getElementById('link')


```

5.3 **非空断言**
------------

在上下文中当类型检查器无法断定类型时，一个新的后缀表达式操作符 `!` 可以用于断言操作对象是非 null 和非 undefined 类型。**具体而言，x! 将从 x 值域中排除 null 和 undefined 。**

```
const aLink = document.getElementById('link')! 
 //如果没有非空断言，使用aLink时会报错，因为页面可能没有link这个标签，得到的就是undefined


```

6. 泛型
=====

6.1 泛型 - 基本介绍
-------------

*   **泛型是可以在保证类型安全前提下，让函数等与多种类型一起工作，从而实现复用**，常用于：函数、接口、class 中
*   需求：创建一个 id 函数，传入什么数据就返回该数据本身 (也就是说，参数和返回值类型相同)

```
function id(value: number): number { return value }


```

*   比如，id(10) 调用以上函数就会直接返回 10 本身。但是，该函数只接收数值类型，无法用于其他类型
*   为了能让函数能够接受任意类型，可以将参数类型修改为 any。但是，这样就失去了 TS 的类型保护，类型不安全

```
function id(value: any): any { return value }


```

*   **泛型在保证类型安全 (不丢失类型信息) 的同时，可以让函数等与多种不同的类型一起工作，灵活可复用**
*   实际上，在 C# 和 Java 等编程语言中，泛型都是用来实现可复用组件功能的主要工具之一

6.2 泛型函数
--------

定义泛型函数

```
function id<Type>(value: Type): Type { return value }
​
function id<T>(value: T): T { return value }


```

*   解释:
    
    1.  语法：在函数名称的后面添加 `<>`(尖括号)，**尖括号中添加类型变量**，比如此处的 Type
    2.  **类型变量 Type，是一种特殊类型的变量，它处理类型而不是值**
    3.  **该类型变量相当于一个类型容器**，能够捕获用户提供的类型 (具体是什么类型由用户调用该函数时指定)
    4.  因为 Type 是类型，因此可以将其作为函数参数和返回值的类型，表示参数和返回值具有相同的类型
    5.  类型变量 Type，可以是任意合法的变量名称

调用泛型函数

```
const num = id<number>(10)
const str = id<string>('a')


```

*   解释：
    
    1.  语法：在函数名称的后面添加 `<>`(尖括号)，**尖括号中指定具体的类型**，比如，此处的 number
    2.  当传入类型 number 后，这个类型就会被函数声明时指定的类型变量 Type 捕获到
    3.  此时，Type 的类型就是 number，所以，函数 id 参数和返回值的类型也都是 number
*   同样，如果传入类型 string，函数 id 参数和返回值的类型就都是 string
    
*   这样，通过泛型就做到了让 id 函数与多种不同的类型一起工作，**实现了复用的同时保证了类型安全**
    

**简化泛型函数调用**

```
// 省略 <number> 调用函数
let num = id(10)
let str = id('a')


```

*   解释:
    
    1.  在调用泛型函数时，**可以省略 `<类型>` 来简化泛型函数的调用**
    2.  此时，TS 内部会采用一种叫做**类型参数推断**的机制，来根据传入的实参自动推断出类型变量 Type 的类型
    3.  比如，传入实参 10，TS 会自动推断出变量 num 的类型 number，并作为 Type 的类型
*   推荐：使用这种简化的方式调用泛型函数，使代码更短，更易于阅读
    
*   说明：**当编译器无法推断类型或者推断的类型不准确时，就需要显式地传入类型参数**
    

6.3 泛型约束
--------

*   默认情况下，泛型函数的类型变量 Type 可以代表多个类型，这导致无法访问任何属性
*   比如，id('a') 调用函数时获取参数的长度：

```
function id<Type>(value: Type): Type {
  console.log(value.length)
  return value
}
​
id('a')


```

*   解释：Type 可以代表任意类型，无法保证一定存在 length 属性，比如 number 类型就没有 length
*   此时，就需要**为泛型添加约束来`收缩类型`(缩窄类型取值范围)**
*   添加泛型约束收缩类型，主要有以下两种方式：1 指定更加具体的类型 2 添加约束

**指定更加具体的类型**

比如，将类型修改为 `Type[]`(Type 类型的数组)，因为只要是数组就一定存在 length 属性，因此就可以访问了

```
function id<Type>(value: Type[]): Type[] {
  console.log(value.length)
  return value
}


```

**添加约束**

```
// 创建一个接口
interface ILength { length: number }
​
// Type extends ILength 添加泛型约束
// 解释：表示传入的 类型 必须满足 ILength 接口的要求才行，也就是得有一个 number 类型的 length 属性
function id<Type extends ILength>(value: Type): Type {
  console.log(value.length)
  return value
}


```

*   解释:
    
    1.  创建描述约束的接口 ILength，该接口要求提供 length 属性
    2.  通过 `extends` 关键字使用该接口，为泛型 (类型变量) 添加约束
    3.  该约束表示：**传入的类型必须具有 length 属性**
*   注意: 传入的实参 (比如，数组) 只要有 length 属性即可（类型兼容性)
    

6.4 多个类型变量
----------

泛型的类型变量可以有多个，并且**类型变量之间还可以约束** (比如，第二个类型变量受第一个类型变量约束) 比如，创建一个函数来获取对象中属性的值：

```
function getProp<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key]
}
let person = { name: 'jack', age: 18 }
getProp(person, 'name')


```

*   解释:
    
    1.  添加了第二个类型变量 Key，两个类型变量之间使用 `,` 逗号分隔。
    2.  **keyof 关键字接收一个对象类型，生成其键名称 (可能是字符串或数字) 的联合类型**。
    3.  本示例中 `keyof Type` 实际上获取的是 person 对象所有键的联合类型，也就是：`'name' | 'age'`
    4.  类型变量 Key 受 Type 约束，可以理解为：Key 只能是 Type 所有键中的任意一个，或者说只能访问对象中存在的属性

```
// Type extends object 表示： Type 应该是一个对象类型，如果不是 对象 类型，就会报错
// 如果要用到 对象 类型，应该用 object ，而不是 Object
function getProperty<Type extends object, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key]
}


```

6.5 泛型接口
--------

泛型接口：接口也可以配合泛型来使用，以增加其灵活性，增强其复用性

```
interface IdFunc<Type> {
  id: (value: Type) => Type
  ids: () => Type[]
}
​
let obj: IdFunc<number> = {
  id(value) { return value },
  ids() { return [1, 3, 5] }
}


```

*   解释:
    
    1.  在接口名称的后面添加 `<类型变量>`，那么，这个接口就变成了泛型接口。
    2.  接口的类型变量，对接口中所有其他成员可见，也就是**接口中所有成员都可以使用类型变量**。
    3.  使用泛型接口时，**需要显式指定具体的类型** (比如，此处的 IdFunc)。
    4.  此时，id 方法的参数和返回值类型都是 number;ids 方法的返回值类型是 number[]。

7. TS 内置的常用工具类型
===============

7.1 typeof
----------

在 TypeScript 中，`typeof` 操作符可以用来获取一个变量声明或对象的类型。

```
interface Person {
  name: string;
  age: number;
}
​
const sem: Person = { name: 'semlinker', age: 33 };
type Sem= typeof sem; // -> Person
​
function toArray(x: number): Array<number> {
  return [x];
}
​
type Func = typeof toArray; // -> (x: number) => number[]
​


```

7.2 keyof
---------

`keyof` 操作符是在 TypeScript 2.1 版本引入的，该操作符可以用于获取某种类型的所有键，其返回类型是联合类型。

```
interface Person {
  name: string;
  age: number;
}
​
type K1 = keyof Person; // "name" | "age"
type K2 = keyof Person[]; // "length" | "toString" | "pop" | "push" | "concat" | "join" 
type K3 = keyof { [x: string]: Person };  // string | number
​


```

7.3 in
------

`in` 用来遍历枚举类型：

```
type Keys = "a" | "b" | "c"
​
type Obj =  {
  [p in Keys]: any
} // -> { a: any, b: any, c: any }


```

7.4 infer
---------

在条件类型语句中，可以用 `infer` 声明一个类型变量并且对它进行使用。

```
type ReturnType<T> = T extends (
  ...args: any[]
) => infer R ? R : any;


```

以上代码中 `infer R` 就是声明一个变量来承载传入函数签名的返回值类型，简单说就是用它取到函数返回值的类型方便之后使用。

7.5 extends
-----------

有时候我们定义的泛型不想过于灵活或者说想继承某些类等，可以通过 extends 关键字添加泛型约束。

```
interface Lengthwise {
  length: number;
}
​
function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}
​


```

7.6 Partial、Readonly、Required
-----------------------------

`Partial<T>` 的作用就是将某个类型里的属性全部变为可选项 `?`。

```
type Partial<T> = {
  [P in keyof T]?: T[P];
};


```

在以上代码中，首先通过 `keyof T` 拿到 `T` 的所有属性名，然后使用 `in` 进行遍历，将值赋给 `P`，最后通过 `T[P]` 取得相应的属性值。中间的 `?` 号，用于将所有属性变为可选。

`Readonly<T>`

将 T 中的所有属性设置为只读

`Required<T>`

将 T 中的所有属性设置为必须

7.7 Omit
--------

`Omit<T, U>`从类型 `T` 中剔除 `U` 中的所有属性

```
interface IPerson {
    name: string
    age: number
}
​
type IOmit = Omit<IPerson, 'age'>
// 这样就剔除了 IPerson 上的 age 属性。


```

8. Vue3 中父子传值 , 用 TS 怎么写，怎么设置默认值
================================

```
// 用泛型来约束收到的数据
// TS的defineProps写法 , defineProps<....>()
const {msg='123'}defineProps<{  //设置默认值需要解构，并且添加全局配置
  msg？: string,
  arr: { name: string }[]
}>()
// 用TS来子传父  defineEmits<(...):void>()
const emit = defineEmits<{
  (e: 'changeMsg', val: string): void
  (e: 'addMsg'): void
}>()


```

**默认值的全局配置**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49e51ed60ae145ee916b2ca490a293ef~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

9. TS 怎么给引入的第三方库设置类型声明文件
========================

*   目前，几乎所有常用的第三方库都有相应的类型声明文件
    
*   第三方库的类型声明文件有两种存在形式：
    
    *   1 、库自带类型声明文件
    *   2、 由 TS 官方给它写的（DefinitelyTyped 提供）

**库自带类型声明文件**：比如，axios

这种情况下，正常导入该库，**TS 就会自动加载库自己的类型声明文件**，以提供该库的类型声明。

**由 DefinitelyTyped 提供**：

*   DefinitelyTyped 是一个 github 仓库，用来提供高质量 TypeScript 类型声明
*   可以通过 npm/yarn 来下载该仓库提供的 TS 类型声明包，这些包的名称格式为:`@types/*`比如，@types/react、@types/lodash 等
*   在实际项目开发时，如果你使用的第三方库没有自带的声明文件，VSCode 会给出明确的提示

10. 说说你对 TypeScript 装饰器的理解？
===========================

装饰器是一种特殊类型的声明，它能够被附加到类声明，方法， 访问符，属性或参数上

是一种在不改变原类和使用继承的情况下，动态地扩展对象功能

同样的，本质也不是什么高大上的结构，就是一个普通的函数，`@expression` 的形式其实是`Object.defineProperty`的语法糖

`expression`求值后必须也是一个函数，它会在运行时被调用，被装饰的声明信息做为参数传入

11. 说说对 TypeScript 中命名空间与模块的理解
==============================

**模块**

`TypeScript` 与`ECMAScript` 2015 一样，任何包含顶级 `import` 或者 `export` 的文件都被当成一个模块

相反地，如果一个文件不带有顶级的`import`或者`export`声明，那么它的内容被视为全局可见的

**命名空间**

命名空间一个最明确的目的就是解决重名问题

命名空间定义了标识符的可见范围，一个标识符在不同命名空间中是互不相干的

12. TS 怎么自定义类型声明文件
==================

**如下两种场景需要提供类型声明文件**

0.  项目内共享类型
1.  为已有 JS 文件提供类型声明

12.1 项目内共享类型
------------

将公共的类型定义提取出来，写在 index.d.ts 文件中 , 并导出

```
export interface Token {
  token: string
  refreshToken: string
}


```

导入接口并使用

```
<script setup lang='ts'>
import {Token} from '.' 
function fn(token:Token){
  
}
</script>


```

12.2 为已有 JS 文件提供类型声明
--------------------

**编写同名的. d.ts 文件**

```
demo.ts
utils/index.js
utils/index.d.ts // 这里是重点


```

**定义类型声明文件**

1.  它的作用是提供声明，不需要提供逻辑代码；
2.  declare 关键字: 用于类型声明，为其他地方 (比如，.js 文件) 已存在的变量声明类型，而不是创建一个新的变量。
3.  *   对于 type、interface 等这些明确就是 TS 类型的 (只能在 TS 中使用的)，可以省略 declare 关键字。
    *   对于 let、function 等具有双重含义 (在 JS、TS 中都能用)，应该使用 declare 关键字，明确指定此处用于类型声明。
4.  ```
    export declare let count = number
    export declare let songName = string
    export declare let position = {
      x: number,
      y: number
    }
    export declare function add(x: number, y: number): number {
    ​
    }
    enum Direction {
      'top',
      'right',
      'bottom',
      'left'
    }
    export declare function changeDirection(direction: Direction): void
    type FomatPoint = (point: number) => void
    export declare const fomatPoint: FomatPoint
    
    
    ```