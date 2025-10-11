# Vue 模板语法与内置指令
- v-once：元素或组件只渲染一次
- v-text：指定元素内容，和 `{{}}` 插值语法效果一样
- v-html：将内容按照 html 解析
  ```html
  <template>
    <div v-html="message"></div>
  </template>
  <script setup>
  import { ref } from 'vue';
  const message = ref('<span style="color: red">test</span>')
  </script>
  ```
- v-pre：跳过元素的编译过程
- v-cloak：隐藏未编译的 mustache 语法标签，直到编译完成
- v-bind：v-bind 用于属性值中，而 `{{}}` 插值语法用于标签中
  - 可以绑定动态属性名
  ```html
  <script setup>
  import { ref } from 'vue'

  const name = ref('class')
  const value = ref('active')
  </script>

  <template>
    <div :[name]="value">test</div>
  </template>
  ```
  - 可以绑定一个对象
  ```html
  <template>
    <div :="info">test</div>
  </template>
  <script setup>
  const info = {
      style: { 'background-color' : 'yellow' }
  }
  </script>
  ```
- v-on：实现对事件的监听，指令简写为 @
  - 指令修饰符如下：
    - .stop：调用 event.stopPropagation
    - .prevent：调用 event.preventDefault
    - .capture：添加事件监听器时使用 capture 模式
    - .self：只有事件从监听器绑定的元素本身触发时才触发回调
    - .{keyAlias}：仅当事件从特定键触发时才触发回调
    - .once：只触发一次回调
      - .left：单击鼠标左键时触发，
      - .right：单机鼠标右键时触发
      - .middle：单机鼠标中键时触发
    - .passive：使用 {passive: true} 模式添加监听器
  - v-on 会自动传入 event 对象，也可以在函数定义时手动传入 event 对象，手动传入写法
  ```html
  <template>
    <button @click="onClick($event, 1)">test</button>
  </template>

  <script setup>
  const onClick = (event, timeRef) => {
    console.log(event)
    console.log(timeRef)
  }
  </script>
  ```
  - 阻止冒泡是指不向父元素传递，如下例中，单击 test 只会打印 22，如果删除 .stop 则在单击 test 时会先打印 22 后打印 11
  ```html
  <template>
    <div @click="onClick1" :style="{ height: '100px', width:  '100px', backgroundColor: 'red' }">
        <button @click.stop="onClick2">test</button>
    </div>
  </template>

  <script setup>
  const onClick1 = () => {
      console.log(11)
  }
  const onClick2 = () => {
      console.log(22)
  }
  </script>
  ```
  - .enter 修饰符: 为 input 元素添加监听 keyup 键盘抬起事件时，每输入一个字符，就会触发一次 enterKeyup 函数，如果用户按 Enter 时再触发，则需要加上 enter 修饰符
- v-if & v-else：条件渲染
  - v-show 与 v-if 的区别：
    - v-show 不支持在 template 标签上使用
    - v-show 不可与 v-else 一起使用
    - v-show 无论是否显示，其 DOM 都会被渲染，本质是通过 CSS 的 display属性控制显示和隐藏，而 v-if 为 false 时不会渲染
    - 开发中如果需要显示和隐藏之间频繁切换，则应使用 v-show
- v-for：v-for 的语法为 `v-for="item in []` 或者 `v-for="(item, index) in []"`
  - v-for 除了支持数组类型的遍历，还支持遍历对象和数字
  - 对于数组类型的响应式变量，调用 filter，concat，slice 方法时，不会触发视图更新，而数组的变更方法例如 push，pop，shift，unshift，splice，sort 和 reverse 则会触发视图更新，或者用新数组替换原有数组也会触发视图更新
- key & diff 算法：给元素 key 可以提高更新时的效率，动态组件中必须要有 key
  ```html
  <template>
    <div style="padding: 20px">
      <h2>用户列表</h2>

      <button @click="addUser">添加用户</button>
      <button @click="shuffleUsers">打乱顺序</button>

      <ul>
        <li
          v-for="user in users"
          :key="user.id"
          style="border: 1px solid #ccc; padding: 8px; margin: 5px 0"
        >
          {{ user.name }} (id: {{ user.id }})
        </li>
      </ul>
    </div>
  </template>

  <script setup>
  import { ref } from 'vue'

  const users = ref([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
    { id: 3, name: 'Charlie' }
  ])

  let nextId = 4

  const addUser = () => {
    users.value.push({ id: nextId++, name: 'User ' + nextId })
  }

  const shuffleUsers = () => {
    users.value.sort(() => Math.random() - 0.5)
  }
  </script>

  <style scoped>
  button {
    margin-right: 10px;
  }
  </style>
  ```