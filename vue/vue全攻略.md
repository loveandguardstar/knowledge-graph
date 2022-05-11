## vue2

### 基础

#### 基本使用

> 指令、插值，computed和watch，class和style，条件渲染，循环列表，事件，表单

#### 组件

> 生命周期，props，$emit和v-on，自定义事件

#### 高级特性

> 自定义 v-model，$nextTick，slot，动态组件，异步组件，keep-alive，mixin

#### 周边工具

> vuex，vue-router，

#### 原理

> 组件化和MVVM，响应式原理，vdom和diff算法，模板编译，组件渲染过程，前端路由

##### 响应式原理

```javascript
// 监听数组
const oldArrayProperty = Array.prototype;
// 创建新对象，再扩展不影响原型
const arrProto = Object.create(oldArrayProperty);
['push', 'pop', 'shift', 'unshift', 'splice'].forEach(methodName => {
    arrProto[methodName] = function() {
        console.log('触发视图更新')
        oldArrayProperty[methodName].call(this, ...arguments)
    }
});


// 实现响应式？更改值，触发视图更新，对数组呢？对值呢？
function useObserve(target) {
    if (typeof target !== 'object' || target === null) {
        return target
    }
    if (Array.isArray(target)) {
        target.__proto__ = arrProto
    }
    // 重新定义各个属性，for...in 也可以遍历数组
    for (let key in target) {
        defineReactive(target, key, target[key])
    }
}

// 重新定义属性，监听起来
function defineReactive(target, key, value) {
    // 深度监听，避免值为对象
    useObserve(value)
    // 核心API
    Object.defineProperty(target, key, {
        get() {
            return value
        },
        set(newValue) {
            if (newValue !== value) {
                value = newValue
                // 触发视图更新
                console.log('触发了')
            }
        }
    })
}

export default useObserve
```

##### vdom和diff算法

> 背景：DOM 操作非常耗费性能；Vue 和 React 是数据驱动视图，如何有效控制 DOM 操作，有了一定的复杂度，想减少计算次数比较难；

```javascript
// JS 模拟 DOM 结构
<div id="div" class="container">
    <p style="font-size: 20px"> vdom </p>
</div>

{
  tag: 'div',
  props: {
    className: 'container',
    id: 'div'
  },
  children: [
    {
      tag: 'p',
      props: {
        style: 'font-size: 20px'
      },
      children: 'vdom'
    }
  ]
}

```

###### 通过 snabbdom 学习 vdom（vdom，数据操作视图，控制DOM）

> Vue 参考它实现的 vdom 和 diff；
>
> 利用JS 模拟 DOM结构；新旧 Vnode 对比，得出最小更新范围，最后更新 DOM 
>
> vdom核心：h、vnode、patch、diff、key等

###### diff 算法

> diff 算法是 vdom 中最核心、最关键的部分，新旧 tree vdom 对比

1. 如何将O(n^3) 优化到 O(n)   （学习 snabbdom 源码)

   1. 只比较同一层级，不跨级比较
   2. tag 不相同，则直接删掉重建，不再深度比较
   3. tag 和 key，两者都相同，则认为是相同节点，不再深度比较

   > patchVnode；addVnodes removeVnodes；updateChildren（key 的重要性）

   ```javascript
   function sameVnode(vnode1: VNode, vnode2: VNode): boolean {
     const isSameKey = vnode1.key === vnode2.key;
     const isSameIs = vnode1.data?.is === vnode2.data?.is;
     const isSameSel = vnode1.sel === vnode2.sel;
   
     return isSameSel && isSameKey && isSameIs;
   }
   
   // patchVnode 核心逻辑
   function patchVnode(....) {
     ....
     if (isUndef(vnode.text)) {
         if (isDef(oldCh) && isDef(ch)) {
           if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue);
             // updateChildren 核心.. while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) 
         } else if (isDef(ch)) {
           if (isDef(oldVnode.text)) api.setTextContent(elm, "");
           addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);
         } else if (isDef(oldCh)) {
           removeVnodes(elm, oldCh, 0, oldCh.length - 1);
         } else if (isDef(oldVnode.text)) {
           api.setTextContent(elm, "");
         }
       } else if (oldVnode.text !== vnode.text) {
         if (isDef(oldCh)) {
           removeVnodes(elm, oldCh, 0, oldCh.length - 1);
         }
         api.setTextContent(elm, vnode.text!);
       }
   ```

   

##### 模板编译

> 通过组件渲染和更新过程
>
> 核心： vue template complier 将模板编译为 render 函数；执行 render 函数生成 vnode；基于 vnode 再执行 patch 和 diff

###### 模板编译 （vue-compiler)

> 不是html，有指令，插值，js表达式，能实现判断、循环
>
> html 是标签语言，只有 JS 才能实现判断、循环（图灵完备）

```javascript
const compiler = require('vue-template-compiler')
// 插值
// const template = `<p>{{message}}</p>`
// with(this){return createElement('p',[createTextVNode(toString(message))])}
// h -> vnode
// createElement -> vnode

// 编译
const res = compiler.compile(template)
console.log(res.render)

// ---------------分割线--------------

// 从 vue 源码中找到缩写函数的含义
function installRenderHelpers (target) {
    target._o = markOnce;
    target._n = toNumber;
    target._s = toString;
    target._l = renderList;
    target._t = renderSlot;
    target._q = looseEqual;
    target._i = looseIndexOf;
    target._m = renderStatic;
    target._f = resolveFilter;
    target._k = checkKeyCodes;
    target._b = bindObjectProps;
    target._v = createTextVNode;
    target._e = createEmptyVNode;
    target._u = resolveScopedSlots;
    target._g = bindObjectListeners;
    target._d = bindDynamicKeys;
    target._p = prependModifier;
}
```

###### vue 组件中使用 render 代替 template

##### 组件 渲染/更新过程

> 一个组件渲染到页面，修改 data 触发更新（数据驱动视图）；背后原理是什么；
>
> 初次渲染过程；更新过程；异步渲染

###### 初次渲染过程

1. 解析模板为 render函数（或在开发环境已完成，vue-loader）
2. 触发响应式，监听 data 属性 getter setter
3. 执行 render 函数，生成 vnode，patch(elem, vnode)

###### 更新过程

1. 修改 data，触发 setter（此前在 getter 中已经被监听）
2. 重新执行 render 函数，生成 newVnode
3. patch(vnode，newVnode)

###### 异步渲染

1. vue 会汇总 data 修改，一次性更新视图（$nextTick）
2. 减少DOM操作，提升性能

##### 前端路由原理

网页 url 组成部分

```javascript
hash: "#/"
host: "localhost:3000"
hostname: "localhost"
href: "http://localhost:3000/#/"
origin: "http://localhost:3000"
pathname: "/"
port: "3000"
protocol: "http:"
```

1. hash 的特点
   1. hash 变化会触发网页跳转，即浏览器的前进、后退
   2. hash 变化不会刷新页面，SPA必需的特点
   3. hash 永远不会提交到 server 端

```js
 // hash 变化，包括：
 // a. JS 修改 url
 // b. 手动修改 url 的 hash
 // c. 浏览器前进、后退
 window.onhashchange = (event) => {
	console.log('old url', event.oldURL)
 	console.log('new url', event.newURL)
	console.log('hash:', location.hash)
}
```

1. H5 history 的特点

```javascript
// 打开一个新的路由
// 【注意】用 pushState 方式，浏览器不会刷新页面
document.getElementById('btn1').addEventListener('click', () => {
    const state = { name: 'page1' }
    console.log('切换路由到', 'page1')
    history.pushState(state, '', 'page1') // 重要！！
})

// 监听浏览器前进、后退
window.onpopstate = (event) => { // 重要！！
    console.log('onpopstate', event.state, location.pathname)
}
```



#### vue2 真题演示

> v-for 为什么用key，组件data为什么是函数，何时使用keep-alive，何时需要使用beforeDestory，diff 算法时间复杂度，vue常见性能优化

## vue3

#### vue3新功能

> createApp，emits属性，多事件处理，Fragment，移除.sync 改为 v-model 参数，异步组件的引入方式，移除filter，Teleport，Suspense，Composition API

#### vue3真题演示

> Vue3 比 Vue2 有什么优势，描述 Vue3生命周期，如何看待 Composition API 和 Options API，如何理解 ref toRef 和 toRefs，Vue3 升级了哪些重要的功能，Composition API 如何实现代码逻辑复用，Vue3如何实现响应式，watch 和 watchEffect 的区别是什么，setup 中如何获取组件实例，Vue3 为何比 Vue2快，Vite 是什么，Composition API 和 React Hooks 的对比

##### Vue3 比 Vue2有什么优势

> 性能更好、体积更小、更好的ts 支持、更好的代码组织、更好的逻辑抽离

##### Vue3生命周期

> beforeCreate，created，beforeMount，mounted，beforeUpdate，updated，beforeUnMount，unMounted
>
>  Composition API 所有前面加了一个on 没有 created类，// setup等于 beforeCreate，created

##### Composition API 对比 Options API

> Composition API带来了什么、Composition API 和 Options API 如何选择

###### Composition API 带来了什么

> 更好的代码组织、更好的逻辑复用、更好的类型推导

- 更好的类型推导

  Options APi 中 this.属性和this.方法混用，很难类型推导（看场景，业务复杂度低用Options API）

###### Composition API 和 Options API 如何选择

- 不建议共用，会引起混乱；小型项目，业务逻辑简单用 Options API；
- 中大型项目，业务逻辑复杂用 Composition API（属于高阶使用）

##### 如何理解 ref toRef 和 toRefs

> 是什么，最佳使用方式，进阶，深入理解

##### ref toRef 和 toRefs

###### ref

> 生成值类型的响应式数据，可用于模板 和 reactive，用过.value 修改值
>

###### toRef 

> 针对一个响应式对象（reactive 封装）的 prop；创建一个 ref，具有响应式；两者保持引用关系

###### roRefs

> 将响应式对象（reactive封装）转换为普通对象；对象的每个prop都是对应的 ref；两者保持引用关系

###### 最佳使用方式

> - 用 reactive 做对象的响应式，用 ref 做值类型响应式
> - 合成函数返回响应式对象时，使用 toRefs
> - ref 创建时将变量名加上 xxxRef，例如 const ageRef = ref(20)

###### 进阶，深入理解

> 为何需要 ref；为何需要 .value；为何需要 toRef toRefs；

1. 为何需要 ref
   1. 返回值类型，会丢失响应式
   2. 在 setup、computed、合成函数、会有可能返回值类型
   3. Vue 如果不定义 ref，用户将自造 ref，反而混乱；
2. 为何需要 .value
   1. vue 响应式用的是 proxy，只能监听对象类型，ref是一个对象，value存储值
   2. 用过.value 属性的 get 和 set 实现响应式
   3. 用于模板、reactive 时，不需要 .value，其他都要
3. 为何需要 toRef toRefs
   1. 初衷：返回值类型，不丢失响应式
   2. 前提：针对的是响应式对象（reactive 封装的） 非普通对象
   3. 注意：不创造响应式，而是延续响应式

```javascript
<template>
  <div>    
    ref demo {{ageRef}} {{state.name}}
	<p ref="elemRef">验证templateRef</p>
  </div>
</template>

<script setup>
import { ref, toRef, toRefs, reactive, onMounted } from 'vue'

const ageRef = ref(20) // 值类型
const nameRef = ref('刷')

const state = reactive({
  name: nameRef
})
const test = reactive({
    name: '888',
    age: 5
})

//reactive对普通对象实现响应式， toRef对reactive某个对象进行响应式监听，对普通对象不具有响应式
const nameToRef = toRef(state, 'name')

const testToRefs = toRefs(test)
const { name: nameToRefs, age: ageToage } = testToRefs // 可命名也可以不命名，名字便于看到

onMounted(() => {
    console.log('ref template', elemRef.value.innerHTML, elemRef.value)
})

setTimeout(() => {
  console.log(ageRef.value);
  ageRef.value = 25
  nameRef.value = '666'
}, 2000)
</script>
```

##### Vue3 升级了哪些重要功能

> createAPP、emits属性、生命周期、多事件、Fragment、移除 .sync、异步组件写法、
>
> 移除 filter、Teleport、Suspense、Composition API（reactive,ref 相关,readonly,watch,watchEffect,setup,生命周期钩子)

##### Composition API 实现逻辑复用

> 抽离逻辑代码到一个函数；函数命名约定为 useXxx 函数（React Hooks）；在 setup 中引入 useXxx 函数

```javascript
import { ref, onMounted, onUnmounted } from 'vue'

function useMousePosition() {
    let x = ref(0)
    let y = ref(0)

    function update(e) {
        x.value = e.pageX
        y.value = e.pageY
    }

    onMounted(() => {
        window.addEventListener('mousemove', update)
    })

    onUnmounted(() => {
        window.removeEventListener('mousemove')
    })

    return {
        x,
        y
    }
}

export default useMousePosition

/// 使用：
import useMousePosition from '....'
const {x, y} = useMousePosition()
```

##### Vue3 如何实现响应式

> Vue2.x 的 Object.defineProperty；学习 Proxy 语法；Vue3 如何用 Proxy 实现响应式

###### Proxy

> 和 Proxy 能力一一对应； 规范化、标准化、函数式；替代掉 Object 上的工具函数

###### Vue3 如何用 Proxy 实现响应式

> Proxy 能规避 Object.definePropery 的问题；Proxy 无法兼容所有浏览器，无法 polyfill

###### Vue2.x 的 Object.defineProperty

> 深度监听需要一次性递归； 无法监听新增属性 / 删除属性（Vue.set Vue.delete)；
>
> 无法原生监听数组，需要特殊处理

###### Proxy 实现响应式

> 基本使用；Reflect；实现响应式
