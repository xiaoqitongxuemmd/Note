# Pinia

Pinia 是 Vue 3 官方推荐的状态管理库，由 Vue 核心团队开发，旨在替代 Vuex，为 Vue 应用程序提供更简洁、高效的状态管理方案

## 核心概念

- Store 状态容器：Pinia 的核心是 Store，通过 defineStore 函数定义的状态容器，用于集中管理应用中跨组件共享的数据，每个 Store 都有一个唯一的 ID，应用可以有多个 Store 按业务模块拆分
- State 状态数据：Store 中存储原始数据的地方，相当于组件中的 data 选项，是响应式的，当数据变化时，使用该数据的组件会自动更新
- Getters 派生状态：类似 Vue 组件中的计算属性，用于根据 Store 中的 State 派生出新的状态
- Actions 行为：用于修改 State 或处理异步逻辑，可以直接修改状态

```typescript
import { defineStore } from 'pinia';

interface Product {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [] as Product[],
    total: 0,
  }),
  getters: {
    itemCount(state) {
      return state.items.length;
    },
  },
  actions: {
    addItem(product: Product) {
      const existing = this.items.find((item) => item.id === product.id);
      if (existing) {
        existing.quantity += product.quantity;
      } else {
        this.items.push({ ...product });
      }
      this.calculateTotal();
    },
    removeItem(productId: number) {
      this.items = this.items.filter((item) => item.id !== productId);
      this.calculateTotal();
    },
    calculateTotal() {
      this.total = this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    },
  },
});
```

```html
<template>
  <div>
    <h2>购物车</h2>
    <p>商品数量: {{ cart.itemCount }}</p>
    <p>总价: ￥{{ cart.total }}</p>
    <ul>
      <li v-for="item in cart.items" :key="item.id">
        {{ item.name }} - ￥{{ item.price }} x {{ item.quantity }}
        <button @click="cart.removeItem(item.id)">删除</button>
      </li>
    </ul>
    <button @click="addTestItem">添加测试商品</button>
  </div>
</template>

<script setup>
import { useCartStore } from '@/stores/cart';

const cart = useCartStore();

function addTestItem() {
  cart.addItem({ id: 1, name: '商品1', price: 100, quantity: 1 });
}
</script>
```