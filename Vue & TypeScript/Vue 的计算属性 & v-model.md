# Vue.js 3 的计算属性 & v-model

## 计算属性
对于任何包含响应式数据的复杂逻辑，都应该使用计算属性
```html
<template>
    <div>{{ myName }}</div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
const firstName = ref<string>("Ma")
const lastName = ref<string>("XinNan")

const myName = computed(() => {
    return firstName.value + lastName.value
})
</script>
```
计算属性有缓存，当多次使用计算属性时，只会执行一次，当依赖的响应式变量改变时才会重新计算。

大多数情况下，计算属性只需要一个 getter 方法，即可以将计算属性直接写成一个函数，但可以通过给出 setter 方法来设置计算属性的值。
```javascript
const myName = computed({
    get: () => {
        return firstName.value + lastName.value
    },
    set: (name: string) => {
        firstName.value = name
        lastName.value = name
    }
})
```

## 监听器 watch

监听器用于监听某个响应式数据的变化
```javascript
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})
```

watchEffect 是 Vue 3 引入的自动依赖追踪监听器。
- 自动收集在回调中用到的响应式变量
- 当这些变量变化时，自动重新执行回调函数
- 无需你手动指定监听的目标
```javascript
watchEffect(() => {
  console.log(`count 改变时会重新执行: ${count.value}`)
})
```

watch 的配置选项：
- handler：监听的回调函数，当监听属性发生变化时会调用该函数
- deep：是否深度监听对象或数组中每个属性的变化，默认值为 false
- immediate 是否立即执行回调函数，默认值是 false（watchEffect 创建时就会执行一次，相当于 watch 加 immediate）

## v-model

v-model 指令可以创建双向绑定，本质上是语法糖，底层实现是使用 v-bind 为 value 属性绑定变量，并使用 v-on 绑定 input 事件，并在事件回调中重新为 value 属性绑定的变量赋值。

v-model 用于 input，textarea，select 元素上
```html
<template>
<input type="text" v-model="name">
<div>Name: {{ name }}</div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const name = ref<string>('')
</script>
```

v-model 的修饰符
- .lazy 修饰符：将输入内容的更新延迟到 change 事件触发时再进行，而不是每次输入内容都进行更新
- .number 修饰符：自动将输入内容转换为数字类型
- .trim 修饰符：去除输入内容的首尾空格