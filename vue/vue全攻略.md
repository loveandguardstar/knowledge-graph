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
> 最佳使用方式：创建时将变量名加上 Ref，例如 const ageRef = ref(20)

###### toRef 

> 针对一个响应式对象（reactive 封装）的 prop；创建一个 ref，具有响应式；两者保持引用关系

###### roRefs

> 将响应式对象（reactive封装）转换为普通对象；对象的每个prop都是对应的 ref；两者保持引用关系

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

