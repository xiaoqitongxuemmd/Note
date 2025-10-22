# Vue Composition API

## setup

setup 是 Vue3 组合式 API 的入口函数，组件 **“一生”中只会被调用一次**，负责：

- 创建响应式数据
- 注册生命周期/监听器
- 把模板需要的数据、函数 return 出去（或返回渲染函数）

`<script setup>` 语法糖（最常用）
```html
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
function inc() { count.value++ }
// 无需 return，编译器自动暴露给模板
</script>

<template>
  <button @click="inc">{{ count }}</button>
</template>
```

普通 `setup()` 函数
```html
<script lang="ts">
import { ref } from 'vue'

export default {
  setup(props, context) {
    const count = ref(0)
    const inc = () => count.value++

    // 必须手动 return
    return { count, inc }
  }
}
</script>
```

## watchEffect
watchEffect 是 Vue3 组合式 API 里的一个自动收集依赖的响应式副作用函数——
一句话：你不用手动声明要监听哪个数据，只要在函数里用到响应式数据，它就会立即执行，并在数据变化时重新执行。
```typescript
import { ref, watchEffect } from 'vue'

const count = ref(0)

// 立即打印 0，以后 count 变化就再打印
watchEffect(() => {
  console.log('count =', count.value)
})
```

停止监听 & 清理副作用
```typescript
const stop = watchEffect((onInvalidate) => {
  onInvalidate(() => {
    // 重新执行前 或 组件卸载时 会调用，做清理
    console.log('cleanup')
  })
})

// 手动停止
stop()
```

## defineProps & defineEmits

defineProps 与 defineEmits 是 `<script setup>` 专属宏（macro），
编译期被 Vue 的编译器直接展开成真正的 props / emits 选项，零运行时开销、自带类型推导。
```typescript
<script setup lang="ts">
interface Props {
  title: string
  count?: number
}
const props = defineProps<Props>()   // 返回响应式对象
// 使用：props.title
</script>
```

defineEmits 类型写法
```typescript
<script setup lang="ts">
const emit = defineEmits<{
  update: [value: number]      // 数组语法：参数列表
  delete: [id: number, name: string]
}>()
// 使用：emit('update', 10)
</script>
```

defineEmits 函数重载写法
```typescript
const emit = defineEmits<{
  (e: 'update', value: number): void
  (e: 'delete', id: number, name: string): void
}>()
```

## defineExpose

defineExpose 是 `<script setup>` 的编译期宏，用来把组件内部的状态或方法主动暴露给父组件（通过模板 ref 调用）。
默认情况下 `<script setup>` 是“全关闭”的——父组件拿到 ref 也访问不到任何东西，必须显式 defineExpose 才放行。

Child
```html
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
function inc() { count.value++ }
function reset() { count.value = 0 }

// 把需要暴露的字段/方法列出来
defineExpose({ count, inc, reset })
</script>

<template>
  <button @click="inc">子组件：{{ count }}</button>
</template>
```

Parent
```html
<script setup lang="ts">
import { ref } from 'vue'
import Counter from './Counter.vue'

const counterRef = ref<InstanceType<typeof Counter>>()
</script>

<template>
  <Counter ref="counterRef" />
  <button @click="counterRef?.inc()">父组件调用 inc</button>
  <button @click="counterRef?.reset()">reset</button>
  <p>父组件读到 count：{{ counterRef?.count }}</p>
</template>
```

## useSlots & useAttrs

useSlots 与 useAttrs 是 `<script setup>` 中的两个编译期宏，
用来获取父组件传进来的“非 props”内容：

useSlots

Parent
```html
<MyCard>
  <template #header>标题</template>
  <template #default>正文</template>
</MyCard>
```

Child
```html
<script setup lang="ts">
import { useSlots } from 'vue'

const slots = useSlots()          // 返回 Slots 对象
// 等价于选项式 this.$slots
</script>

<template>
  <div class="card">
    <div class="header" v-if="slots.header">
      <slot name="header" />
    </div>
    <div class="body">
      <slot />
    </div>
  </div>
</template>
```

useAttrs

Parent
```html
<el-button type="primary" size="small" @click="handle" class="ml-2">保存</el-button>
```

Child
```html
<script setup lang="ts">
import { useAttrs } from 'vue'

const attrs = useAttrs()
// attrs = { type: 'primary', size: 'small', class: 'ml-2', onClick: ... }
</script>
```

与 props 的关系
- 被 props 声明过的属性不会出现在 attrs
- 事件（@click / @update:modelValue 等）也会出现在 attrs，键名是 onXxx（驼峰）