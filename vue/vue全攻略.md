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
