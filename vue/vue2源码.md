#### 准备工作

##### 认识 flow

> 为什么用 flow，flow 带来了什么

##### Vue.js 源码目录设计

> compiler 编译相关，core 核心代码，platforms 不同平台支持，server 服务端渲染，sfc .vue文件解析，shared 共享代码

1. compiler 包括把模板解析为AST语法树，ast 语法树优化，代码生成等，（推荐用 webpack vue-loader 在编译期做）
2. core 包括内置组件、全局API封装、Vue实例化、观察者、虚拟DOM、工具函数等
3. platforms 跨平台打包....

##### 源码构建

> 构建脚本，构建过程，Runtime Only VS Runtime + Compiler

##### 从入口开始

> 入口 src/platforms/web/entry-runtime-with-compiler.js；导入import Vue from 'core/index'进行真正的初始化；
>
> 然后这里有2 处关键的代码，`import Vue from './instance/index'` 和 `initGlobalAPI(Vue)`

​	import Vue from './instance/index' 中是对 Vue的各种定义（按功能将扩展分散到多个模块中去，xxxMixin 它们的功能都是给 Vue 的 prototype 上扩展一些方法），实际上是一个Function实现的类，只能通过new Vue() 实例化它

###### initGlobalAPI

​	除了给它的原型 prototype 上扩展方法，还会给 `Vue` 这个对象本身扩展全局的静态方法（Vue 官网中关于全局 API 都可以在这里找到）

#### 数据驱动

> introduction；new Vue 发生了什么；Vue实例挂载的实现；render；Virtual DOM；createElement；update

##### introduction

​	Vue的核心思想是数据驱动，视图由数据利用指令数据驱动生成。对视图修改，不操作DOM，而是修改数据

##### new Vue 发生了什么

​	new Vue 初始化调用 _init方法（在instance/init.js）中定义，主要干了合并配置，初始化生命周期，初始化事件中心，初始化渲染，调用beforeCreate钩子函数，初始化 data,props,computed,watcher，调用 created 钩子函数

​	初始化的最后，检测到如果有 `el` 属性，则调用 `vm.$mount` 方法挂载 `vm`，挂载的目标就是把模板渲染成最终的 DOM。

##### Vue 实例挂载的实现

​	Vue 中我们是通过 `$mount` 实例方法去挂载 `vm` 的，compiler` 版本的 `$mount 首先缓存了$mount方法再重定义，对el做了限制，不能挂载在html、body这种根节点，如果没有定义 render 方法，便将 el 或者 temlate 转成 render方法，Vue所有的组件渲染都必须要转成 render方法。

​	`$mount` 方法实际上会去调用 `mountComponent` 方法。`mountComponent` 核心就是先实例化一个渲染`Watcher`，在它的回调函数中会调用 `updateComponent` 方法，在此方法中调用 `vm._render` 方法先生成虚拟 Node，最终调用 `vm._update` 更新 DOM。

​	`Watcher` 在这里起到两个作用，一个是初始化的时候会执行回调函数，另一个是当 vm 实例中的监测的数据发生变化的时候执行回调函数，最后判断vm._isMounted 为 true，表示实例已经挂载调用 mounted 钩子

##### render

​	src/core/instance/render.js 文件中的  `vm._render` 最终是通过执行 `createElement` 方法并返回的是 `vnode`，它是一个虚拟 Node

##### Virtual DOM

​	VNode 是对真实 DOM 的一种抽象描述，它的核心定义无非就几个关键属性，标签名、数据、子节点、键值等。Virtual DOM 是借鉴了一个开源库 [snabbdom](https://github.com/snabbdom/snabbdom)

##### createElement

​	Vue.js 利用 createElement 方法创建 VNode，它定义在 `src/core/vdom/create-element.js` 中

###### 	children 的规范化

​		src/core/vdom/helpers/normalzie-children.js中 对于编译 `slot`、`v-for` 的时候会产生嵌套数组，经过对 `children` 的规范化（即数组拍平），`children` 变成了一个类型为 VNode 的 Array

###### 	Vnode 的创建

##### update

​	`_update` 是实例的一个私有方法，它被调用的时机有 2 个，一个是首次渲染，一个是数据更新的时候。`_update` 的核心就是调用 `vm.__patch__` 方法（用到了一个函数柯里化的技巧）

​	首次渲染我们调用了 `createElm` 方法，这里传入的 `parentElm` 是 `oldVnode.elm` 的父元素，在我们的例子是 id 为 `#app` div 的父元素，也就是 Body；实际上整个过程就是递归创建了一个完整的 DOM 树并插入到 Body 上。

#### 组件化

#### 深入响应式原理

> 响应式对象，依赖收集，派发更新，nextTick，监测变化的注意事项，计算属性 VS 侦听属性

##### 响应式对象

> Object.defineProperty，initState，Proxy，observe，Observer，defineReactive

###### initState

​	initState` 方法主要是对 `props`、`methods`、`data`、`computed` 和 `wathcer` 等属性做了初始化操作。这里我们重点分析 `props` 和 `data

- initProps、initData

  `props` 的初始化主要过程，就是遍历定义的 `props` 配置。遍历的过程主要做两件事情：一个是调用 `defineReactive` 方法把每个 `prop` 对应的值变成响应式，可以通过 `vm._props.xxx` 访问到定义 `props` 中对应的属性。

  `data` 的初始化主要过程也是做两件事，一个是对定义 `data` 函数返回对象的遍历，通过 `proxy` 把每一个值 `vm._data.xxx` 都代理到 `vm.xxx` 上；另一个是调用 `observe` 方法观测整个 `data` 的变化，把 `data` 也变成响应式，可以通过 `vm._data.xxx` 访问到定义 `data` 返回函数中对应的属性

###### proxy

​	利用Object.defineProperty

###### observe

​	observe` 方法的作用就是给非 VNode 的对象类型数据添加一个 `Observer

Observer

​	`Observer` 的构造函数逻辑很简单，首先实例化 `Dep` 对象。它的作用是给对象的属性添加 getter 和 setter，用于依赖收集和派发更新

###### defineReactive

​	`defineReactive` 的功能就是定义一个响应式对象，给对象动态添加 getter 和 setter

##### 依赖收集

> Dep 和 Watcher





