# Vue 组件高级化

## render 函数

render 函数就是 **“用 JavaScript 代替模板写法”** 来生成虚拟 DOM（VNode）的函数。
Vue 在 编译阶段 会把 `<template>` 自动编译成 render 函数；你也可以手写** render，完全抛弃模板，获得极致灵活性。

```html
<template>
  <render />
</template>

<script setup lang="ts">
import { ref, h } from 'vue'

const counter = ref(0)

const render = () =>
  h('div', null, [
    h('button', { onClick: () => counter.value++ }, 'Add'),
    h('div', null, counter.value)
  ])
</script>
```

## 组件的 jsx 语法

```html
<template>
    <render />
</template>

<script lang="tsx" setup>
import { ref } from 'vue'

/* 1. props 类型声明 */
interface Props {
  title: string
  modelValue: number
}
const props = defineProps<Props>()

/* 2. emit 类型声明 */
const emit = defineEmits<{
  (e: 'update:modelValue', v: number): void
}>()

/* 3. 局部状态 */
const local = ref(props.modelValue)

/* 4. 更新函数：双向绑定 */
function update(delta: number) {
  local.value += delta
  emit('update:modelValue', local.value)
}

/* 5. 直接返回 TSX（无 <template>） */
const render = () => (
  <div class="tsx-counter">
    <h2>{props.title}</h2>
    <div class="body">
      <button onClick={() => update(-1)}>-</button>
      <span class="val">{local.value}</span>
      <button onClick={() => update(1)}>+</button>
    </div>
  </div>
)
</script>

<style scoped>
.tsx-counter {
  display: inline-block;
  padding: 12px 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}
.body {
  display: flex;
  align-items: center;
  gap: 12px;
}
.val {
  font-weight: bold;
  font-size: 20px;
  min-width: 40px;
  text-align: center;
}
button {
  width: 32px;
  height: 32px;
}
</style>
```

需要安装 vue jsx 插件 `@vitejs/plugin-vue-jsx@5.1.1`，并在 vite.config.ts 中配置
```typescript
// vite.config.ts
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'

export default {
  plugins: [vue(), vueJsx()] // 必须加
}
```

## 自定义指令

自定义局部指令
```html
<!-- LongPressLocal.vue -->
<template>
  <button v-press="onLongPress">长按我 0.5s</button>
</template>

<script setup lang="ts">
// 1. 定义指令
const vPress = {
  mounted(el: HTMLElement, binding) {
    let timer: number | null = null
    const start = (e: Event) => {
      timer = window.setTimeout(() => binding.value(e), 500)
    }
    const cancel = () => {
      if (timer) clearTimeout(timer)
      timer = null
    }

    el.addEventListener('mousedown', start)
    el.addEventListener('touchstart', start)
    el.addEventListener('mouseup', cancel)
    el.addEventListener('touchend', cancel)
    el.addEventListener('mouseleave', cancel)

    // 保存清理函数，卸载时用
    ;(el as any)._longPressCleanup = cancel
  },
  unmounted(el: HTMLElement) {
    const cancel = (el as any)._longPressCleanup
    if (cancel) cancel()
  }
}

// 2. 回调
function onLongPress(e: Event) {
  console.log('长按触发！', e)
}
</script>
```

自定义全局指令
```typescript
// src/directives/press.ts
import type { App } from 'vue'

export const vPress = {
  mounted(el: HTMLElement, binding) {
    let timer: number | null = null
    const start = (e: Event) => {
      timer = window.setTimeout(() => binding.value(e), 500)
    }
    const cancel = () => {
      if (timer) clearTimeout(timer)
      timer = null
    }

    el.addEventListener('mousedown', start)
    el.addEventListener('touchstart', start)
    el.addEventListener('mouseup', cancel)
    el.addEventListener('touchend', cancel)
    el.addEventListener('mouseleave', cancel)

    ;(el as any)._longPressCleanup = cancel
  },
  unmounted(el: HTMLElement) {
    const cancel = (el as any)._longPressCleanup
    if (cancel) cancel()
  }
}

export function setupPressDirective(app: App) {
  app.directive('press', vPress)
}
```

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import { setupPressDirective } from '@/directives/press'

const app = createApp(App)
setupPressDirective(app)   // 注册全局指令
app.mount('#app')
```

自定义指令具有修饰符
```typescript
interface DirectiveBinding {
  value: any        // 等号后面的值
  oldValue: any     // 更新前的旧值
  arg?: string      // 冒号后面的参数
  modifiers: Record<string, boolean> // 点后面的修饰符
  dir: Object       // 指令定义对象本身
  instance: ComponentPublicInstance | null
}
```

```vue
<span v-tooltip:top.delayed="提示文本">悬停我</span>
```

```typescript
const vTooltip = {
  mounted(el, binding) {
    console.log(binding.value)      // → "提示文本"
    console.log(binding.arg)        // → "top"  （位置参数）
    console.log(binding.modifiers)  // → { delayed: true }
  }
}
```

## teleport
Teleport 是 Vue 3 内置组件，作用一句话：
“把子组件的 DOM 搬到任意指定位置，同时保持 Vue 组件树、响应式、事件系统完全不变。”

为什么需要 Teleport？（痛点场景）
- 模态框 / 抽屉

    深层嵌套 → 受父级 overflow:hidden / transform 影响，position:fixed 失效。
- 全局通知 / 悬浮菜单
    
    需要挂载到 body 下，避免 z-index 层级战争。
- 微前端 / 跨应用

    把子应用弹窗插到宿主指定容器，DOM 脱离嵌套，逻辑仍在原组件。