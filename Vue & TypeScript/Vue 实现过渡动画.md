# Vue 实现过渡动画

## transition

为了实现单个元素或组件的过渡动画效果，在 Vue 中可以使用内置组件`<transition>`，该组件是对 CSS 中 transition 属性的封装，在 `v-if`、`v-show`、动态组件以及组件根节点情况下都可以为任何元素和组件添加进入或离开的过渡动画效果

```html
<template>
  <button @click="show = !show">Toggle</button>

  <transition name="fade">
    <p v-show="show">Hello transition!</p>
  </transition>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const show = ref(true)
</script>

<style>
/* 进入的起点 */
.fade-enter-from { opacity: 0; }
/* 进入的终点 */
.fade-enter-to   { opacity: 1; }
/* 进入的过程 */
.fade-enter-active { transition: opacity 0.5s ease; }

/* 离开的起点 */
.fade-leave-from { opacity: 1; }
/* 离开的终点 */
.fade-leave-to   { opacity: 0; }
/* 离开的过程 */
.fade-leave-active { transition: opacity 0.5s ease; }
</style>
```

| 阶段  | 类名                                 | 说明                 |
| --- | ---------------------------------- | ------------------ |
| 进入前 | `v-enter-from` / `name-enter-from` | 插入前 1 帧，下一帧立即删除    |
| 进入中 | `v-enter-active`                   | 整个进入阶段一直存在，可写时长/缓动 |
| 进入后 | `v-enter-to`                       | 插入后 1 帧，动画结束即删除    |
| 离开前 | `v-leave-from`                     | 开始离开 1 帧           |
| 离开中 | `v-leave-active`                   | 整个离开阶段存在           |
| 离开后 | `v-leave-to`                       | 离开动画最后 1 帧         |


CSS 的 animation 属性也可以实现过渡动画。

```html
<template>
  <button @click="show = !show">Toggle</button>

  <!-- 关键点 1：type="animation" 让 Vue 用 animation 时长 -->
  <transition
    name="bounce"
    type="animation"
    mode="out-in"
  >
    <p v-if="show" key="hi">Hello animation!</p>
  </transition>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const show = ref(true)
</script>

<style>
/* 进入动画：从 0 放大到 1，同时上下微移 */
@keyframes bounceIn {
  0%   { transform: scale(0) translateY(-20px); opacity: 0; }
  60%  { transform: scale(1.1) translateY(4px); }
  100% { transform: scale(1) translateY(0); opacity: 1; }
}

/* 离开动画：缩小并下移 */
@keyframes bounceOut {
  0%   { transform: scale(1) translateY(0); opacity: 1; }
  60%  { transform: scale(1.1) translateY(-4px); }
  100% { transform: scale(0) translateY(20px); opacity: 0; }
}

/* 把关键帧挂到 Vue 的过渡类上 */
.bounce-enter-active {
  animation: bounceIn 0.5s ease;
}
.bounce-leave-active {
  animation: bounceOut 0.4s ease;
}
</style>
```

transition 组件的常见属性
- name：指定过渡类名的基础名称，默认为 v
- type：指定过渡类型，可选值为 transition（默认值）或 animation
- mode：指定过渡模式，可选值为 in-out、out-in、默认为空
- appear：指定是否在初次渲染时执行过渡动画、默认为 false

实际开发中通常会使用第三方动画库例如 animation GSAP 等

## transition-group

`transition-group` 是 Vue 内置的列表过渡组件，用来给 v-for 渲染的一组元素 添加

- 插入（新增项滑入）
- 删除（旧项滑出）
- 排序（拖拽/数据顺序变化时的平滑移位）
- 普通更新（可选）

核心规则
- 默认渲染为真实 DOM 元素（默认 <span>，可改 tag="ul"）。
- 子元素必须带 唯一 key（v-for="item in list" :key="item.id"）。
- 提供 6 个类名前缀（同 <transition>）：
    - v-enter-from v-enter-active v-enter-to
    - v-leave-from v-leave-active v-leave-to
- 新增：v-move 在排序时自动加到所有变动项，用来写“位移”过渡。
- CSS 必须显式给 leave-active 设 position: absolute; 才能让旧项“脱标”滑出，不撑开高度。
- 可配 name duration move-class 等，也有 JS 钩子。

```html
<template>
  <input v-model="text" @keyup.enter="add" placeholder="回车新增" />
  <button @click="sort">随机排序</button>

  <!-- 关键 1：transition-group 包裹 v-for -->
  <transition-group name="flip" tag="ul">
    <li v-for="(item,idx) in list" :key="item.id" class="item">
      {{ item.text }}
      <button @click="remove(idx)">×</button>
    </li>
  </transition-group>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface Todo { id: number; text: string }
const list = ref<Todo[]>([
  { id: 1, text: 'Vue' },
  { id: 2, text: 'React' },
  { id: 3, text: 'Svelte' }
])
let nextId = 4
const text = ref('')

function add() {
  if (!text.value.trim()) return
  list.value.push({ id: nextId++, text: text.value })
  text.value = ''
}
function remove(idx: number) {
  list.value.splice(idx, 1)
}
function sort() {
  list.value.sort(() => Math.random() - 0.5)
}
</script>

<style>
/* 进入 */
.flip-enter-from {
  opacity: 0;
  transform: translateX(30px);
}
.flip-enter-active {
  transition: all 0.4s ease;
}

/* 离开：先脱标，再滑走 */
.flip-leave-active {
  position: absolute;   /* 关键 2：脱标不占高度 */
  transition: all 0.4s ease;
}
.flip-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

/* 排序平滑移位 */
.flip-move {
  transition: transform 0.4s ease;
}

/* 简单样式 */
ul {
  position: relative;   /* 给绝对定位子项当参考 */
  padding: 0;
}
.item {
  list-style: none;
  background: #ffe5d3;
  margin: 4px 0;
  padding: 8px 12px;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
```