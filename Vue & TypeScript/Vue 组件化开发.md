# Vue 组件化开发

全局组件是指只需注册一次，整个应用中都可以直接使用的组件（不需要每次 import）。

Vue 注册全局组件，在 main.ts 中
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import MyButton from './components/MyButton.vue'

const app = createApp(App)

app.component('MyButton', MyButton)

app.mount('#app')
```

项目中 public 文件夹和 assets 文件夹的区别

| 对比项              | `src/assets`                                         | `public`                         |
| ---------------- | ---------------------------------------------------- | -------------------------------- |
| 📦 **是否参与打包**    | ✅ 会被构建工具（Vite / Webpack）处理、压缩、指纹哈希                   | ❌ 不会打包，只会原样复制到 `dist/`           |
| 🧭 **引用方式**      | 通过 `import` 或 `@/assets/...` 引入                      | 用**绝对路径** `/` 引用                 |
| 🔁 **路径处理**      | 构建时路径会被解析成哈希文件名                                      | 文件路径保持不变                         |
| ⚙️ **用途**        | 组件内部资源（图片、字体、样式）                                     | 外部静态资源（favicon、robots.txt、第三方脚本） |
| 💥 **能否用变量动态引用** | ✅ 可以通过 `import`、`new URL(..., import.meta.url)` 动态加载 | ❌ 只能通过固定路径访问                     |
| 🧩 **适合场景**      | 项目代码依赖的资源                                            | 不参与打包的独立静态文件                     |
| 🧾 **打包后位置**     | `/dist/assets/xxxxx.[hash].png`                      | `/dist/` 根目录下原样文件                |

## 工程化的组件样式

`<style scoped>` 是样式作用域标记，表明定义的样式仅在当前组件中生效。组件支持包含多个 style 标签

scoped 语法本质上是会被 PostCSS 工具转换为类似如下内容，其中 `[]` 是 CSS 的属性选择器语法，而 Vue 会保证其在各个组件中不会冲突
```html
<template>
    <div class="example" data-v-f3f3eg9>test</div>
</template>

<style scoped>
.example[data-v-f3f3eg9] {
    background-color: red;
}
</style>
```

通常情况下，当使用 scoped 时，父组件的样式将不会泄露到子组件中，不过子组件的根节点会同时受到父组件作用域样式和子组件的作用域样式的影响。开发中可以采用如下方式避免：
- 减少使用标签选择器，多使用 class
- 在每个组件的根元素中添加唯一的 class 选择器

如果需要在父组件的局部样式中修改子组件中某个元素的样式，可以使用 `:deep()` 深度选择器

Child:
```html
<template>
  <div class="child-box">
    <p class="text">我是子组件</p>
  </div>
</template>

<style scoped>
.child-box {
  background-color: lightblue;
}
.text {
  color: black;
}
</style>
```

Parent:
```html
<template>
  <div class="parent">
    <Child />
  </div>
</template>

<script setup>
import Child from './Child.vue'
</script>

<style scoped>
::v-deep(.child-box) {
  background-color: pink;
}

::v-deep(.text) {
  color: red;
}
</style>
```

CSS Module 语法会在 style 标签带有 module 属性时 `<style module>` 将标签编译为 CSS Modules，并可以类似 `<div :class="$style.red">` 这样使用。这个语法使用不多。

CSS 中也可以使用 v-bind 语法将 script 中的一些计算属性或者响应式变量绑定到 CSS 样式中。

## 父子组件的相互通信

父子通信的核心思想为
- 父传子采用 props
- 子传父采用 emit

父组件传递数据给子组件

Child：
```html
<template>
    <div>{{ message }}</div>
</template>

<script setup lang="ts">
interface IProps {
    message: string
}
defineProps<IProps>()
</script>
```

Parent:
```html
<template>
  <Counter message="test"></Counter>
</template>

<script setup lang="ts">
import Counter from './components/Counter.vue';
</script>
```

子组件传递数据给父组件

Child:
```html
<template>
  <div>
    <button @click="sendMessage">点我发送消息给父组件</button>
  </div>
</template>

<script setup lang="ts">
// 声明当前组件会触发哪些事件
const emit = defineEmits<{
  (event: 'send', msg: string): void
}>()

// 点击时触发事件
const sendMessage = () => {
  emit('send', '你好，我是子组件 👋')
}
</script>
```

Parent:
```html
<template>
  <div>
    <h3>父组件</h3>
    <p>子组件的消息：{{ childMessage }}</p>

    <!-- 监听子组件事件 -->
    <Child @send="onChildSend" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Child from './Child.vue'

// 保存子组件传来的消息
const childMessage = ref('（暂无消息）')

// 子组件触发 send 时执行这个函数
const onChildSend = (msg: string) => {
  childMessage.value = msg
  console.log('父组件收到消息：', msg)
}
</script>
```

## 非父子之间的通信
- Provide/Inject：用于非父子组件之间共享数据，这个是单项通信，只能祖先传递给子孙，并且不适合频繁通信
- Mitt：全局总线事件
- Pinia/Vuex：全局状态共享，推荐使用 Pinia

**Provide/Inject 例子**

Parent:
```html
<!-- Parent.vue -->
<script setup lang="ts">
import { ref, provide } from 'vue'

const themeColor = ref('red')
provide('theme', themeColor)
</script>

<template>
  <div>
    <p>父组件提供 theme: {{ themeColor }}</p>
    <Child />
  </div>
</template>
```

Grandchild:
```html
<!-- Grandchild.vue -->
<script setup lang="ts">
import { inject } from 'vue'

const theme = inject('theme', 'black') // 第二个参数是默认值
</script>

<template>
  <div :style="{ color: theme }">
    我是孙组件，主题颜色是 {{ theme }}
  </div>
</template>
```

**全局事件总线 mitt** 为第三方库，通过 `npm install mitt` 安装

在 src/utils/bus.ts 中：
```typescript
import mitt from 'mitt'
const bus = mitt()
export default bus
```

发送事件：
```html
<!-- ChildA.vue -->
<script setup lang="ts">
import bus from '@/utils/bus'

const send = () => {
  bus.emit('sayHello', '你好，我是 ChildA')
}
</script>

<template>
  <button @click="send">发送消息</button>
</template>
```

接收事件：
```html
<!-- ChildB.vue -->
<script setup lang="ts">
import bus from '@/utils/bus'

bus.on('sayHello', (msg) => {
  console.log('ChildB 收到：', msg)
})
</script>

<template>
  <div>ChildB 等待接收消息...</div>
</template>
```

**全局状态管理 Pinia** 也是第三方库，通过 `npm install pinia` 安装

在 main.ts 中注册：
```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
app.use(createPinia()) // 全局注册
app.mount('#app')
```

定义一个 Store（例如：stores/counter.ts）
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

// 定义一个 store（名字叫 counter）
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const double = computed(() => count.value * 2)

  const increment = () => count.value++
  const reset = () => count.value = 0

  return { count, double, increment, reset }
})
```

组件中使用：
```html
<script setup lang="ts">
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()
</script>

<template>
  <div>
    <p>Count: {{ counter.count }}</p>
    <p>Double: {{ counter.double }}</p>
    <button @click="counter.increment">+1</button>
    <button @click="counter.reset">重置</button>
  </div>
</template>
```

## 插槽
使用示例：

Child：
```html
<template>
  <div class="dialog">
    <slot></slot>
  </div>
</template>
```

Parent:
```html
<template>
  <Dialog>
    <h2>提示</h2>
    <p>确认要删除吗？</p>
  </Dialog>
</template>
```

**具名插槽**

Child:
```html
<template>
  <div class="dialog">
    <header><slot name="header"></slot></header>
    <main><slot></slot></main> <!-- 默认插槽 -->
    <footer><slot name="footer"></slot></footer>
  </div>
</template>
```

Parent:
```html
<template>
  <Dialog>
    <template #header>
      <h2>自定义标题</h2>
    </template>

    <p>这里是内容</p> <!-- 默认插槽 -->

    <template #footer>
      <button>确定</button>
      <button>取消</button>
    </template>
  </Dialog>
</template>
```

**作用域插槽**：子组件希望把它内部的数据传给插槽内容使用

Child:
```html
<template>
  <ul>
    <slot v-for="item in items" :item="item"></slot>
  </ul>
</template>

<script setup lang="ts">
const props = defineProps<{
  items: string[]
}>()
</script>
```

Parent:
```html
<template>
  <List :items="['苹果', '香蕉', '西瓜']">
    <template #default="{ item }">
      <li>{{ item }} 很好吃！</li>
    </template>
  </List>
</template>
```

## 动态组件

```html
<script setup>
import { ref } from 'vue'
import ChartView from './ChartView.vue'
import TableView from './TableView.vue'
const currentView = ref('chart')
const views = { chart: ChartView, table: TableView }
</script>

<template>
  <div>
    <button @click="currentView = 'chart'">图表</button>
    <button @click="currentView = 'table'">表格</button>

    <keep-alive>
      <component :is="views[currentView]" />
    </keep-alive>
  </div>
</template>
```

keep-alive 会让 vue 将移除的组件缓存而不是销毁，keep-alive 属性如下
- include：支持 string，RegExp，Array，只有名称匹配的组件才会被缓存
- exclude：支持 string，RegExp，Array，任何名称匹配的组件都不会被缓存
- max：支持 number，string，最多可以缓存多少个组件实例

对于缓存组件来说，再次进入不会执行 created 或 mounted，但是可以通过 activated 和 deactivated 来判断是否显示还是隐藏。

## 异步组件
```typescript
const ChartView = defineAsyncComponent(() =>
  import('./ChartView.vue')
)
```

通过 defineAsyncComponent 函数实现懒加载，
- Vue 只在第一次渲染到它时才会执行 `import('./ChartView.vue')`

- 会自动返回一个 Promise，在加载完成后渲染组件

Vue 允许你配置异步组件的加载中状态和错误重试逻辑：
```typescript
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent({
  loader: () => import('./ChartView.vue'),
  loadingComponent: () => import('./Loading.vue'), // 加载中
  errorComponent: () => import('./Error.vue'),     // 加载失败时显示
  delay: 200, // 延迟 200ms 才显示 loading
  timeout: 3000, // 超时 3 秒则报错
  onError(error, retry, fail, attempts) {
    if (attempts < 3) retry()
    else fail()
  }
})
```

在一个组件树中，如果存在多个异步组件，那么每个异步组件都需要处理自己的加载，报错和完成状态，为了能统一这些组件，Vue 提供了内置的 Suspense 组件。

Suspense 可以让我们在组件树的上层等待下层的多个嵌套异步依赖项解析完成，并可以在等待时渲染一个加载状态，防止在最坏情况下，看到多个 Loading 加载状态。如果异步组件的父组件链中存在一个 Suspense 组件，那么该异步组件将被视为该 Suspense 组件的异步依赖项，这种情况下，异步组件的加载状态由 Suspense 控制，异步组件自身的加载，错误，延迟和超时都会被忽略。

Suspense 组件包含两个插槽
- default：如果 default 插槽可以显示，则显示 default 插槽内容
- fallback：如果 default 插槽无法显示，则会显示 fallback 插槽的内容

```html
<script setup>
import { defineAsyncComponent } from 'vue'

const AsyncChart = defineAsyncComponent(() => import('./ChartView.vue'))
</script>

<template>
  <Suspense>
    <template #default>
      <AsyncChart />
    </template>
    <template #fallback>
      <div>正在加载中...</div>
    </template>
  </Suspense>
</template>
```

