# Vue Router

在前端开发中，经常会提到单页面应用程序（Single-Page Application SPA），这种应用程序只有一个页面，页面的内容是通过 JavaScript 动态渲染的，在切换页面时 SPA 应用并不会重新加载新页面，而是通过 hash 或 history API 实现前端路由的切换

- hash 带 # 不请求服务器，刷新永不 404；
- history 像后端路径，刷新需服务器重定向到 index.html。

```html
<!-- pages/Home.vue -->
<template>
    <div>Home</div>
</template>
```

```html
<!-- pages/About.vue -->
<template>
    <div>About</div>
</template>
```

```typescript
// router/index.ts
import { createRouter, createWebHistory } from "vue-router";

import Home from "@/pages/Home.vue";
import About from "@/pages/About.vue";

const routes = [
    {
        path: '/home',
        component: Home
    },
    {
        path: '/about',
        component: About
    }
]

const router = createRouter({
    routes,
    history: createWebHistory()
})

export default router
```

```typescript
// main.ts
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'
import router from './router/index'
const app = createApp(App)
app.use(router)
app.mount('#app')
```

```html
<!-- App.vue -->
<template>
  <div>
    <RouterLink to="/home">首页</RouterLink>
    <RouterLink to="/about">关于</RouterLink>
    
  </div>
  <RouterView></RouterView>
</template>
```

在路由中可以配置默认路径
```typescript
const routes = [
    {
        path: '/',
        redirect: '/home'
    },
    {
        path: '/home',
        component: Home
    },
    {
        path: '/about',
        component: About
    }
]
```


RouterLink 是 Vue 的内置组件
- to：当组件被单击时，内部会立刻把 to 的值传给 router.push API 实现页面跳转
- replace：这个页面跳转是直接替换，因此 页面不会压入浏览器的历史栈中，因此无法使用返回功能

## 动态路由
```html
<template>
    <div>User: {{ $route.params.username }}</div>
</template>

<script setup lang="ts">
import { useRoute } from 'vue-router';

const route = useRoute();
console.log(route.params.username)
</script>
```

```typescript
const routes = [
    {
        path: '/home',
        component: Home,
        meta: {name: 'aaa'}
    },
    {
        path: '/about',
        component: About,
        meta: {name: 'bbb'}
    },
    {
        path: '/user/:username',
        component: () => import ("@/pages/User.vue")
    }
]
```

## 嵌套路由
通过 children 属性可以注册子路由

```typescript
const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    children: [
      {
        path: 'profile',
        name: 'Profile',
        component: Profile,
      },
      {
        path: 'settings',
        name: 'Settings',
        component: Settings,
      },
    ],
  },
  {
    path: '/about',
    name: 'About',
    component: About,
  },
];
```

## 动态添加路由

可以通过 addRoute 动态添加路由
```typescript
router.addRoute({
  path: '/dynamic',
  name: 'Dynamic',
  component: () => import('./components/Dynamic.vue'),
});
```

## 路由守卫

在某些情况下，我们希望能够拦截路由导航，比如在进入某个路由之前，先判断用户是否已登录等

可以通过 beforeEach 方法注册一个路由的全局前置守卫
```typescript
const router = CreateRouter({ ... })
router.beforeEach((to, from) => {
    if (to.path !== '/login') {
        const token = window.sessionStorage.getItem('token')
        if (!token) {
            return { path: '/login'}
        }
    }
})
```