# Pinia状态管理 - 完整教程

## 课程简介

Pinia是Vue 3的官方状态管理库，它提供了更简洁的API、更好的TypeScript支持和更强大的DevTools集成。本教程将深入讲解Pinia的核心概念、使用方法和最佳实践。

## 学习目标

- 理解Pinia的核心概念和设计理念
- 掌握Store的定义和使用方法
- 熟练使用State、Getters、Actions
- 掌握组合式Store的编写
- 学会使用插件系统扩展功能
- 掌握状态持久化方案
- 理解Pinia的最佳实践

## 目录

1. [Pinia基础](#第1章-pinia基础)
2. [Store定义](#第2章-store定义)
3. [State状态](#第3章-state状态)
4. [Getters计算属性](#第4章-getters计算属性)
5. [Actions动作](#第5章-actions动作)
6. [组合式Store](#第6章-组合式store)
7. [插件系统](#第7章-插件系统)
8. [状态持久化](#第8章-状态持久化)
9. [TypeScript支持](#第9章-typescript支持)
10. [最佳实践](#第10章-最佳实践)

---

## 第1章 Pinia基础

### 1.1 Pinia简介

Pinia是Vue的官方状态管理库，相比Vuex有以下优势：

```typescript
/**
 * Pinia vs Vuex对比
 * @author erik.zhou
 */

// Vuex 4的写法
const store = createStore({
    state: () => ({ count: 0 }),
    mutations: {
        increment(state) {
            state.count++;
        }
    },
    actions: {
        incrementAsync({ commit }) {
            setTimeout(() => {
                commit('increment');
            }, 1000);
        }
    }
});

// Pinia的写法（更简洁）
const useCounterStore = defineStore('counter', {
    state: () => ({ count: 0 }),
    actions: {
        increment() {
            this.count++;
        },
        async incrementAsync() {
            await new Promise(resolve => setTimeout(resolve, 1000));
            this.count++;
        }
    }
});
```


### 1.2 安装与配置

```bash
# 使用npm安装
npm install pinia

# 使用yarn安装
yarn add pinia

# 使用pnpm安装
pnpm add pinia
```

```typescript
/**
 * Pinia基础配置
 * @author erik.zhou
 */
import { createApp } from 'vue';
import { createPinia } from 'pinia';
import App from './App.vue';

const app = createApp(App);
const pinia = createPinia();

// 挂载Pinia
app.use(pinia);
app.mount('#app');
```

### 1.3 核心概念

```typescript
/**
 * Pinia核心概念示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

// 1. Store：状态容器
export const useUserStore = defineStore('user', {
    // 2. State：状态数据
    state: () => ({
        name: 'John',
        age: 25
    }),
    
    // 3. Getters：计算属性
    getters: {
        doubleAge: (state) => state.age * 2
    },
    
    // 4. Actions：方法
    actions: {
        updateName(newName: string) {
            this.name = newName;
        }
    }
});
```

### 1.4 基本使用

```vue
<!--
  Pinia基本使用示例
  @author erik.zhou
-->
<template>
    <div>
        <h1>{{ userStore.name }}</h1>
        <p>Age: {{ userStore.age }}</p>
        <p>Double Age: {{ userStore.doubleAge }}</p>
        <button @click="updateUser">Update Name</button>
    </div>
</template>

<script setup lang="ts">
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

const updateUser = () => {
    userStore.updateName('Jane');
};
</script>
```

---

## 第2章 Store定义

### 2.1 选项式Store

```typescript
/**
 * 选项式Store定义
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', {
    state: () => ({
        count: 0,
        name: 'Counter'
    }),
    
    getters: {
        doubleCount: (state) => state.count * 2,
        
        // 访问其他getter
        doubleCountPlusOne(): number {
            return this.doubleCount + 1;
        }
    },
    
    actions: {
        increment() {
            this.count++;
        },
        
        decrement() {
            this.count--;
        },
        
        incrementBy(amount: number) {
            this.count += amount;
        }
    }
});
```

### 2.2 Setup Store（组合式）

```typescript
/**
 * Setup Store定义
 * @author erik.zhou
 */
import { ref, computed } from 'vue';
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', () => {
    // State
    const count = ref(0);
    const name = ref('Counter');
    
    // Getters
    const doubleCount = computed(() => count.value * 2);
    const doubleCountPlusOne = computed(() => doubleCount.value + 1);
    
    // Actions
    function increment() {
        count.value++;
    }
    
    function decrement() {
        count.value--;
    }
    
    function incrementBy(amount: number) {
        count.value += amount;
    }
    
    return {
        count,
        name,
        doubleCount,
        doubleCountPlusOne,
        increment,
        decrement,
        incrementBy
    };
});
```

### 2.3 Store命名规范

```typescript
/**
 * Store命名规范示例
 * @author erik.zhou
 */

// ✅ 推荐：使用use前缀 + 业务名称 + Store后缀
export const useUserStore = defineStore('user', {});
export const useCartStore = defineStore('cart', {});
export const useProductStore = defineStore('product', {});

// ✅ 推荐：Store ID使用小写加连字符
export const useUserProfileStore = defineStore('user-profile', {});

// ❌ 不推荐：不使用use前缀
export const UserStore = defineStore('user', {});

// ❌ 不推荐：Store ID使用驼峰
export const useUserStore = defineStore('userStore', {});
```

### 2.4 Store模块化

```typescript
/**
 * Store模块化组织
 * @author erik.zhou
 */

// stores/modules/user.ts
export const useUserStore = defineStore('user', {
    state: () => ({
        id: null as number | null,
        name: '',
        email: ''
    }),
    actions: {
        setUser(user: any) {
            this.id = user.id;
            this.name = user.name;
            this.email = user.email;
        }
    }
});

// stores/modules/cart.ts
export const useCartStore = defineStore('cart', {
    state: () => ({
        items: [] as any[]
    }),
    getters: {
        totalPrice: (state) => {
            return state.items.reduce((sum, item) => sum + item.price, 0);
        }
    }
});

// stores/index.ts
export { useUserStore } from './modules/user';
export { useCartStore } from './modules/cart';
```

---

## 第3章 State状态

### 3.1 定义State

```typescript
/**
 * State定义示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

interface UserState {
    id: number | null;
    name: string;
    email: string;
    roles: string[];
    settings: {
        theme: 'light' | 'dark';
        language: string;
    };
}

export const useUserStore = defineStore('user', {
    state: (): UserState => ({
        id: null,
        name: '',
        email: '',
        roles: [],
        settings: {
            theme: 'light',
            language: 'zh-CN'
        }
    })
});
```

### 3.2 访问State

```vue
<!--
  访问State示例
  @author erik.zhou
-->
<template>
    <div>
        <!-- 方式1：直接访问 -->
        <p>Name: {{ userStore.name }}</p>
        
        <!-- 方式2：使用storeToRefs -->
        <p>Email: {{ email }}</p>
        
        <!-- 方式3：使用computed -->
        <p>Theme: {{ theme }}</p>
    </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { storeToRefs } from 'pinia';
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

// 方式2：使用storeToRefs保持响应性
const { email, settings } = storeToRefs(userStore);

// 方式3：使用computed
const theme = computed(() => userStore.settings.theme);
</script>
```

### 3.3 修改State

```typescript
/**
 * 修改State的多种方式
 * @author erik.zhou
 */
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

// 方式1：直接修改
userStore.name = 'John';

// 方式2：使用$patch（对象形式）
userStore.$patch({
    name: 'John',
    email: 'john@example.com'
});

// 方式3：使用$patch（函数形式）
userStore.$patch((state) => {
    state.name = 'John';
    state.roles.push('admin');
});

// 方式4：使用actions
userStore.updateUser({
    name: 'John',
    email: 'john@example.com'
});

// 方式5：替换整个state
userStore.$state = {
    id: 1,
    name: 'John',
    email: 'john@example.com',
    roles: ['admin'],
    settings: {
        theme: 'dark',
        language: 'en-US'
    }
};
```

### 3.4 重置State

```typescript
/**
 * 重置State示例
 * @author erik.zhou
 */
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

// 重置到初始状态
userStore.$reset();

// 自定义重置逻辑（Setup Store需要手动实现）
export const useCounterStore = defineStore('counter', () => {
    const count = ref(0);
    const name = ref('Counter');
    
    function $reset() {
        count.value = 0;
        name.value = 'Counter';
    }
    
    return { count, name, $reset };
});
```


---

## 第4章 Getters计算属性

### 4.1 基础Getters

```typescript
/**
 * 基础Getters示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useProductStore = defineStore('product', {
    state: () => ({
        products: [
            { id: 1, name: 'Product 1', price: 100, stock: 10 },
            { id: 2, name: 'Product 2', price: 200, stock: 5 },
            { id: 3, name: 'Product 3', price: 300, stock: 0 }
        ]
    }),
    
    getters: {
        // 简单getter
        totalProducts: (state) => state.products.length,
        
        // 计算总价值
        totalValue: (state) => {
            return state.products.reduce((sum, product) => {
                return sum + product.price * product.stock;
            }, 0);
        },
        
        // 过滤有库存的产品
        availableProducts: (state) => {
            return state.products.filter(product => product.stock > 0);
        },
        
        // 获取最贵的产品
        mostExpensiveProduct: (state) => {
            return state.products.reduce((max, product) => {
                return product.price > max.price ? product : max;
            }, state.products[0]);
        }
    }
});
```

### 4.2 访问其他Getters

```typescript
/**
 * 访问其他Getters示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useCartStore = defineStore('cart', {
    state: () => ({
        items: [
            { id: 1, name: 'Item 1', price: 100, quantity: 2 },
            { id: 2, name: 'Item 2', price: 200, quantity: 1 }
        ],
        discount: 0.1
    }),
    
    getters: {
        // 计算小计
        subtotal: (state) => {
            return state.items.reduce((sum, item) => {
                return sum + item.price * item.quantity;
            }, 0);
        },
        
        // 访问其他getter
        discountAmount(): number {
            return this.subtotal * this.discount;
        },
        
        // 计算总价
        total(): number {
            return this.subtotal - this.discountAmount;
        },
        
        // 格式化总价
        formattedTotal(): string {
            return `¥${this.total.toFixed(2)}`;
        }
    }
});
```

### 4.3 带参数的Getters

```typescript
/**
 * 带参数的Getters示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useProductStore = defineStore('product', {
    state: () => ({
        products: [
            { id: 1, name: 'Product 1', price: 100, category: 'electronics' },
            { id: 2, name: 'Product 2', price: 200, category: 'clothing' },
            { id: 3, name: 'Product 3', price: 300, category: 'electronics' }
        ]
    }),
    
    getters: {
        // 返回函数以接受参数
        getProductById: (state) => {
            return (id: number) => {
                return state.products.find(product => product.id === id);
            };
        },
        
        // 按分类过滤
        getProductsByCategory: (state) => {
            return (category: string) => {
                return state.products.filter(product => product.category === category);
            };
        },
        
        // 价格范围过滤
        getProductsByPriceRange: (state) => {
            return (min: number, max: number) => {
                return state.products.filter(product => {
                    return product.price >= min && product.price <= max;
                });
            };
        }
    }
});

// 使用示例
const productStore = useProductStore();
const product = productStore.getProductById(1);
const electronics = productStore.getProductsByCategory('electronics');
const affordable = productStore.getProductsByPriceRange(0, 150);
```

### 4.4 访问其他Store的Getters

```typescript
/**
 * 访问其他Store的Getters
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

// 用户Store
export const useUserStore = defineStore('user', {
    state: () => ({
        vipLevel: 2
    }),
    getters: {
        discountRate: (state) => {
            const rates = { 1: 0.05, 2: 0.1, 3: 0.15 };
            return rates[state.vipLevel as keyof typeof rates] || 0;
        }
    }
});

// 购物车Store
export const useCartStore = defineStore('cart', {
    state: () => ({
        items: [
            { id: 1, price: 100, quantity: 2 }
        ]
    }),
    getters: {
        subtotal: (state) => {
            return state.items.reduce((sum, item) => {
                return sum + item.price * item.quantity;
            }, 0);
        },
        
        // 访问其他Store的getter
        total(): number {
            const userStore = useUserStore();
            const discount = this.subtotal * userStore.discountRate;
            return this.subtotal - discount;
        }
    }
});
```

---

## 第5章 Actions动作

### 5.1 同步Actions

```typescript
/**
 * 同步Actions示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', {
    state: () => ({
        count: 0,
        history: [] as number[]
    }),
    
    actions: {
        // 简单的同步action
        increment() {
            this.count++;
            this.history.push(this.count);
        },
        
        decrement() {
            this.count--;
            this.history.push(this.count);
        },
        
        // 带参数的action
        incrementBy(amount: number) {
            this.count += amount;
            this.history.push(this.count);
        },
        
        // 重置
        reset() {
            this.count = 0;
            this.history = [];
        },
        
        // 调用其他action
        doubleIncrement() {
            this.increment();
            this.increment();
        }
    }
});
```

### 5.2 异步Actions

```typescript
/**
 * 异步Actions示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

interface User {
    id: number;
    name: string;
    email: string;
}

export const useUserStore = defineStore('user', {
    state: () => ({
        user: null as User | null,
        loading: false,
        error: null as string | null
    }),
    
    actions: {
        // 异步获取用户信息
        async fetchUser(id: number) {
            this.loading = true;
            this.error = null;
            
            try {
                const response = await fetch(`/api/users/${id}`);
                if (!response.ok) {
                    throw new Error('Failed to fetch user');
                }
                this.user = await response.json();
            } catch (error) {
                this.error = error instanceof Error ? error.message : 'Unknown error';
            } finally {
                this.loading = false;
            }
        },
        
        // 异步更新用户信息
        async updateUser(userData: Partial<User>) {
            this.loading = true;
            this.error = null;
            
            try {
                const response = await fetch(`/api/users/${this.user?.id}`, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(userData)
                });
                
                if (!response.ok) {
                    throw new Error('Failed to update user');
                }
                
                this.user = await response.json();
            } catch (error) {
                this.error = error instanceof Error ? error.message : 'Unknown error';
            } finally {
                this.loading = false;
            }
        },
        
        // 异步删除用户
        async deleteUser() {
            if (!this.user) return;
            
            this.loading = true;
            this.error = null;
            
            try {
                const response = await fetch(`/api/users/${this.user.id}`, {
                    method: 'DELETE'
                });
                
                if (!response.ok) {
                    throw new Error('Failed to delete user');
                }
                
                this.user = null;
            } catch (error) {
                this.error = error instanceof Error ? error.message : 'Unknown error';
            } finally {
                this.loading = false;
            }
        }
    }
});
```

### 5.3 Actions中访问其他Store

```typescript
/**
 * Actions中访问其他Store
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

// 认证Store
export const useAuthStore = defineStore('auth', {
    state: () => ({
        token: null as string | null,
        isAuthenticated: false
    }),
    actions: {
        setToken(token: string) {
            this.token = token;
            this.isAuthenticated = true;
        },
        clearToken() {
            this.token = null;
            this.isAuthenticated = false;
        }
    }
});

// 用户Store
export const useUserStore = defineStore('user', {
    state: () => ({
        user: null as any
    }),
    actions: {
        async login(username: string, password: string) {
            const authStore = useAuthStore();
            
            try {
                const response = await fetch('/api/login', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ username, password })
                });
                
                const data = await response.json();
                
                // 设置token到认证Store
                authStore.setToken(data.token);
                
                // 设置用户信息
                this.user = data.user;
            } catch (error) {
                console.error('Login failed:', error);
                throw error;
            }
        },
        
        logout() {
            const authStore = useAuthStore();
            
            // 清除认证信息
            authStore.clearToken();
            
            // 清除用户信息
            this.user = null;
        }
    }
});
```

### 5.4 订阅Actions

```typescript
/**
 * 订阅Actions示例
 * @author erik.zhou
 */
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

// 订阅所有actions
userStore.$onAction(({
    name,        // action名称
    store,       // store实例
    args,        // action参数
    after,       // action成功后的钩子
    onError      // action失败后的钩子
}) => {
    console.log(`Action ${name} called with args:`, args);
    
    // action成功后执行
    after((result) => {
        console.log(`Action ${name} succeeded with result:`, result);
    });
    
    // action失败后执行
    onError((error) => {
        console.error(`Action ${name} failed with error:`, error);
    });
});

// 使用示例
await userStore.fetchUser(1);
```


---

## 第6章 组合式Store

### 6.1 基础组合式Store

```typescript
/**
 * 基础组合式Store示例
 * @author erik.zhou
 */
import { ref, computed } from 'vue';
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', () => {
    // State
    const count = ref(0);
    const name = ref('Counter');
    
    // Getters
    const doubleCount = computed(() => count.value * 2);
    
    // Actions
    function increment() {
        count.value++;
    }
    
    function decrement() {
        count.value--;
    }
    
    function incrementBy(amount: number) {
        count.value += amount;
    }
    
    // 必须返回所有需要暴露的内容
    return {
        count,
        name,
        doubleCount,
        increment,
        decrement,
        incrementBy
    };
});
```

### 6.2 组合式Store的优势

```typescript
/**
 * 组合式Store的优势示例
 * @author erik.zhou
 */
import { ref, computed, watch } from 'vue';
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', () => {
    // 1. 可以使用所有组合式API
    const user = ref<any>(null);
    const loading = ref(false);
    
    // 2. 可以使用watch
    watch(user, (newUser) => {
        console.log('User changed:', newUser);
        // 自动保存到localStorage
        if (newUser) {
            localStorage.setItem('user', JSON.stringify(newUser));
        }
    });
    
    // 3. 可以使用computed
    const isAdmin = computed(() => {
        return user.value?.role === 'admin';
    });
    
    // 4. 可以使用生命周期钩子（在setup中）
    const initUser = () => {
        const savedUser = localStorage.getItem('user');
        if (savedUser) {
            user.value = JSON.parse(savedUser);
        }
    };
    
    // 5. 异步操作更自然
    async function fetchUser(id: number) {
        loading.value = true;
        try {
            const response = await fetch(`/api/users/${id}`);
            user.value = await response.json();
        } finally {
            loading.value = false;
        }
    }
    
    return {
        user,
        loading,
        isAdmin,
        initUser,
        fetchUser
    };
});
```

### 6.3 组合式Store的复用

```typescript
/**
 * 组合式Store的复用示例
 * @author erik.zhou
 */
import { ref, computed } from 'vue';
import { defineStore } from 'pinia';

// 可复用的加载状态逻辑
function useLoading() {
    const loading = ref(false);
    const error = ref<string | null>(null);
    
    async function withLoading<T>(fn: () => Promise<T>): Promise<T> {
        loading.value = true;
        error.value = null;
        try {
            return await fn();
        } catch (e) {
            error.value = e instanceof Error ? e.message : 'Unknown error';
            throw e;
        } finally {
            loading.value = false;
        }
    }
    
    return { loading, error, withLoading };
}

// 可复用的分页逻辑
function usePagination() {
    const page = ref(1);
    const pageSize = ref(10);
    const total = ref(0);
    
    const totalPages = computed(() => Math.ceil(total.value / pageSize.value));
    const hasNext = computed(() => page.value < totalPages.value);
    const hasPrev = computed(() => page.value > 1);
    
    function nextPage() {
        if (hasNext.value) page.value++;
    }
    
    function prevPage() {
        if (hasPrev.value) page.value--;
    }
    
    function goToPage(p: number) {
        if (p >= 1 && p <= totalPages.value) {
            page.value = p;
        }
    }
    
    return {
        page,
        pageSize,
        total,
        totalPages,
        hasNext,
        hasPrev,
        nextPage,
        prevPage,
        goToPage
    };
}

// 使用可复用逻辑的Store
export const useProductStore = defineStore('product', () => {
    const products = ref<any[]>([]);
    
    // 复用加载状态
    const { loading, error, withLoading } = useLoading();
    
    // 复用分页逻辑
    const pagination = usePagination();
    
    async function fetchProducts() {
        await withLoading(async () => {
            const response = await fetch(
                `/api/products?page=${pagination.page.value}&size=${pagination.pageSize.value}`
            );
            const data = await response.json();
            products.value = data.items;
            pagination.total.value = data.total;
        });
    }
    
    return {
        products,
        loading,
        error,
        ...pagination,
        fetchProducts
    };
});
```

### 6.4 组合式Store的最佳实践

```typescript
/**
 * 组合式Store最佳实践
 * @author erik.zhou
 */
import { ref, computed, watch } from 'vue';
import { defineStore } from 'pinia';

export const useTaskStore = defineStore('task', () => {
    // 1. 按类型组织state
    const tasks = ref<any[]>([]);
    const selectedTaskId = ref<number | null>(null);
    const filter = ref<'all' | 'active' | 'completed'>('all');
    
    // 2. 使用computed定义getters
    const selectedTask = computed(() => {
        return tasks.value.find(task => task.id === selectedTaskId.value);
    });
    
    const filteredTasks = computed(() => {
        switch (filter.value) {
            case 'active':
                return tasks.value.filter(task => !task.completed);
            case 'completed':
                return tasks.value.filter(task => task.completed);
            default:
                return tasks.value;
        }
    });
    
    const stats = computed(() => ({
        total: tasks.value.length,
        active: tasks.value.filter(t => !t.completed).length,
        completed: tasks.value.filter(t => t.completed).length
    }));
    
    // 3. 使用watch处理副作用
    watch(tasks, (newTasks) => {
        localStorage.setItem('tasks', JSON.stringify(newTasks));
    }, { deep: true });
    
    // 4. 定义actions
    function addTask(title: string) {
        const task = {
            id: Date.now(),
            title,
            completed: false,
            createdAt: new Date()
        };
        tasks.value.push(task);
    }
    
    function removeTask(id: number) {
        const index = tasks.value.findIndex(task => task.id === id);
        if (index !== -1) {
            tasks.value.splice(index, 1);
        }
    }
    
    function toggleTask(id: number) {
        const task = tasks.value.find(task => task.id === id);
        if (task) {
            task.completed = !task.completed;
        }
    }
    
    function selectTask(id: number | null) {
        selectedTaskId.value = id;
    }
    
    function setFilter(newFilter: typeof filter.value) {
        filter.value = newFilter;
    }
    
    // 5. 初始化逻辑
    function init() {
        const saved = localStorage.getItem('tasks');
        if (saved) {
            tasks.value = JSON.parse(saved);
        }
    }
    
    // 6. 清晰的返回结构
    return {
        // State
        tasks,
        selectedTaskId,
        filter,
        
        // Getters
        selectedTask,
        filteredTasks,
        stats,
        
        // Actions
        addTask,
        removeTask,
        toggleTask,
        selectTask,
        setFilter,
        init
    };
});
```

---

## 第7章 插件系统

### 7.1 创建基础插件

```typescript
/**
 * 创建基础插件示例
 * @author erik.zhou
 */
import { PiniaPluginContext } from 'pinia';

// 简单的日志插件
function loggerPlugin({ store }: PiniaPluginContext) {
    // 订阅state变化
    store.$subscribe((mutation, state) => {
        console.log(`[${store.$id}] State changed:`, mutation.type);
        console.log('New state:', state);
    });
    
    // 订阅actions
    store.$onAction(({ name, args }) => {
        console.log(`[${store.$id}] Action ${name} called with:`, args);
    });
}

// 使用插件
import { createPinia } from 'pinia';

const pinia = createPinia();
pinia.use(loggerPlugin);
```

### 7.2 持久化插件

```typescript
/**
 * 持久化插件示例
 * @author erik.zhou
 */
import { PiniaPluginContext } from 'pinia';

interface PersistOptions {
    key?: string;
    storage?: Storage;
    paths?: string[];
}

function persistPlugin(options: PersistOptions = {}) {
    return ({ store }: PiniaPluginContext) => {
        const {
            key = store.$id,
            storage = localStorage,
            paths
        } = options;
        
        // 从storage恢复state
        const savedState = storage.getItem(key);
        if (savedState) {
            try {
                const parsed = JSON.parse(savedState);
                store.$patch(parsed);
            } catch (error) {
                console.error('Failed to parse saved state:', error);
            }
        }
        
        // 订阅state变化并保存
        store.$subscribe((mutation, state) => {
            let dataToSave = state;
            
            // 如果指定了paths，只保存指定的字段
            if (paths && paths.length > 0) {
                dataToSave = paths.reduce((acc, path) => {
                    acc[path] = state[path];
                    return acc;
                }, {} as any);
            }
            
            storage.setItem(key, JSON.stringify(dataToSave));
        });
    };
}

// 使用持久化插件
const pinia = createPinia();
pinia.use(persistPlugin({
    storage: sessionStorage,
    paths: ['user', 'token']
}));
```

### 7.3 扩展Store功能

```typescript
/**
 * 扩展Store功能的插件
 * @author erik.zhou
 */
import { PiniaPluginContext } from 'pinia';

// 添加全局方法的插件
function globalMethodsPlugin({ store }: PiniaPluginContext) {
    // 添加$reset方法（Setup Store需要）
    if (!store.$reset) {
        const initialState = JSON.parse(JSON.stringify(store.$state));
        store.$reset = () => {
            store.$patch(initialState);
        };
    }
    
    // 添加$hydrate方法
    store.$hydrate = (data: any) => {
        store.$patch(data);
    };
    
    // 添加$serialize方法
    store.$serialize = () => {
        return JSON.stringify(store.$state);
    };
}

// 使用插件
const pinia = createPinia();
pinia.use(globalMethodsPlugin);

// TypeScript类型扩展
declare module 'pinia' {
    export interface PiniaCustomProperties {
        $hydrate: (data: any) => void;
        $serialize: () => string;
    }
}
```

### 7.4 性能监控插件

```typescript
/**
 * 性能监控插件
 * @author erik.zhou
 */
import { PiniaPluginContext } from 'pinia';

interface PerformanceMetrics {
    actionCalls: Map<string, number>;
    actionDurations: Map<string, number[]>;
    stateChanges: number;
}

function performancePlugin() {
    const metrics: PerformanceMetrics = {
        actionCalls: new Map(),
        actionDurations: new Map(),
        stateChanges: 0
    };
    
    return ({ store }: PiniaPluginContext) => {
        // 监控actions性能
        store.$onAction(({ name, after, onError }) => {
            const startTime = performance.now();
            
            // 记录调用次数
            const calls = metrics.actionCalls.get(name) || 0;
            metrics.actionCalls.set(name, calls + 1);
            
            after(() => {
                const duration = performance.now() - startTime;
                const durations = metrics.actionDurations.get(name) || [];
                durations.push(duration);
                metrics.actionDurations.set(name, durations);
                
                // 警告慢操作
                if (duration > 1000) {
                    console.warn(`Slow action detected: ${name} took ${duration}ms`);
                }
            });
            
            onError((error) => {
                console.error(`Action ${name} failed:`, error);
            });
        });
        
        // 监控state变化
        store.$subscribe(() => {
            metrics.stateChanges++;
        });
        
        // 添加获取指标的方法
        store.$getMetrics = () => {
            const report: any = {
                storeId: store.$id,
                stateChanges: metrics.stateChanges,
                actions: {}
            };
            
            metrics.actionCalls.forEach((calls, name) => {
                const durations = metrics.actionDurations.get(name) || [];
                const avgDuration = durations.reduce((a, b) => a + b, 0) / durations.length;
                
                report.actions[name] = {
                    calls,
                    avgDuration: avgDuration.toFixed(2),
                    totalDuration: durations.reduce((a, b) => a + b, 0).toFixed(2)
                };
            });
            
            return report;
        };
    };
}

// 使用插件
const pinia = createPinia();
pinia.use(performancePlugin());

// TypeScript类型扩展
declare module 'pinia' {
    export interface PiniaCustomProperties {
        $getMetrics: () => any;
    }
}
```


---

## 第8章 状态持久化

### 8.1 使用pinia-plugin-persistedstate

```bash
# 安装持久化插件
npm install pinia-plugin-persistedstate
```

```typescript
/**
 * 使用pinia-plugin-persistedstate
 * @author erik.zhou
 */
import { createPinia } from 'pinia';
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate';

const pinia = createPinia();
pinia.use(piniaPluginPersistedstate);

// 在Store中启用持久化
export const useUserStore = defineStore('user', {
    state: () => ({
        name: '',
        token: '',
        preferences: {}
    }),
    
    // 启用持久化
    persist: true
});
```

### 8.2 自定义持久化配置

```typescript
/**
 * 自定义持久化配置
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
    state: () => ({
        name: '',
        email: '',
        token: '',
        preferences: {
            theme: 'light',
            language: 'zh-CN'
        }
    }),
    
    persist: {
        // 自定义key
        key: 'my-user-store',
        
        // 使用sessionStorage
        storage: sessionStorage,
        
        // 只持久化部分字段
        paths: ['name', 'token', 'preferences.theme'],
        
        // 自定义序列化
        serializer: {
            serialize: (state) => JSON.stringify(state),
            deserialize: (value) => JSON.parse(value)
        }
    }
});
```

### 8.3 多存储策略

```typescript
/**
 * 多存储策略示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useAuthStore = defineStore('auth', {
    state: () => ({
        token: '',
        refreshToken: '',
        user: null as any,
        rememberMe: false
    }),
    
    persist: [
        {
            // token存储到sessionStorage
            key: 'auth-session',
            storage: sessionStorage,
            paths: ['token', 'user']
        },
        {
            // refreshToken存储到localStorage（如果勾选记住我）
            key: 'auth-persistent',
            storage: localStorage,
            paths: ['refreshToken', 'rememberMe'],
            beforeRestore: (ctx) => {
                // 只在勾选记住我时恢复
                const saved = localStorage.getItem('auth-persistent');
                if (saved) {
                    const data = JSON.parse(saved);
                    return data.rememberMe;
                }
                return false;
            }
        }
    ]
});
```

### 8.4 加密持久化

```typescript
/**
 * 加密持久化示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';
import CryptoJS from 'crypto-js';

const SECRET_KEY = 'your-secret-key';

// 加密工具
const encryptStorage = {
    getItem(key: string): string | null {
        const encrypted = localStorage.getItem(key);
        if (!encrypted) return null;
        
        try {
            const decrypted = CryptoJS.AES.decrypt(encrypted, SECRET_KEY);
            return decrypted.toString(CryptoJS.enc.Utf8);
        } catch (error) {
            console.error('Decryption failed:', error);
            return null;
        }
    },
    
    setItem(key: string, value: string): void {
        const encrypted = CryptoJS.AES.encrypt(value, SECRET_KEY).toString();
        localStorage.setItem(key, encrypted);
    },
    
    removeItem(key: string): void {
        localStorage.removeItem(key);
    }
};

export const useSecureStore = defineStore('secure', {
    state: () => ({
        sensitiveData: '',
        privateKey: ''
    }),
    
    persist: {
        storage: encryptStorage as any
    }
});
```

### 8.5 条件持久化

```typescript
/**
 * 条件持久化示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useCartStore = defineStore('cart', {
    state: () => ({
        items: [] as any[],
        guestMode: false
    }),
    
    persist: {
        // 只在非访客模式下持久化
        enabled: (state) => !state.guestMode,
        
        // 或使用beforeRestore钩子
        beforeRestore: (ctx) => {
            // 检查用户是否登录
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            return isLoggedIn;
        },
        
        // afterRestore钩子
        afterRestore: (ctx) => {
            console.log('Cart restored:', ctx.store.$state);
            
            // 验证恢复的数据
            if (ctx.store.items.length > 100) {
                console.warn('Too many items, clearing cart');
                ctx.store.items = [];
            }
        }
    }
});
```

---

## 第9章 TypeScript支持

### 9.1 类型化State

```typescript
/**
 * 类型化State示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

// 定义State接口
interface UserState {
    id: number | null;
    name: string;
    email: string;
    roles: string[];
    profile: {
        avatar: string;
        bio: string;
    };
    settings: {
        theme: 'light' | 'dark';
        language: 'zh-CN' | 'en-US';
        notifications: boolean;
    };
}

export const useUserStore = defineStore('user', {
    state: (): UserState => ({
        id: null,
        name: '',
        email: '',
        roles: [],
        profile: {
            avatar: '',
            bio: ''
        },
        settings: {
            theme: 'light',
            language: 'zh-CN',
            notifications: true
        }
    }),
    
    getters: {
        // TypeScript会自动推断返回类型
        isAdmin: (state): boolean => {
            return state.roles.includes('admin');
        },
        
        // 显式指定返回类型
        fullProfile(): string {
            return `${this.name} (${this.email})`;
        }
    },
    
    actions: {
        // 参数类型和返回类型
        updateProfile(profile: Partial<UserState['profile']>): void {
            this.profile = { ...this.profile, ...profile };
        },
        
        // 异步action的类型
        async fetchUser(id: number): Promise<UserState> {
            const response = await fetch(`/api/users/${id}`);
            const user = await response.json();
            this.$patch(user);
            return user;
        }
    }
});
```

### 9.2 类型化Getters

```typescript
/**
 * 类型化Getters示例
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

interface Product {
    id: number;
    name: string;
    price: number;
    category: string;
}

export const useProductStore = defineStore('product', {
    state: () => ({
        products: [] as Product[]
    }),
    
    getters: {
        // 简单getter，自动推断类型
        productCount: (state) => state.products.length,
        
        // 返回函数的getter需要显式类型
        getProductById: (state) => {
            return (id: number): Product | undefined => {
                return state.products.find(p => p.id === id);
            };
        },
        
        // 复杂类型的getter
        productsByCategory: (state) => {
            return (category: string): Product[] => {
                return state.products.filter(p => p.category === category);
            };
        },
        
        // 使用泛型的getter
        filterProducts<T extends keyof Product>(
            this: any,
            key: T,
            value: Product[T]
        ): Product[] {
            return this.products.filter((p: Product) => p[key] === value);
        }
    }
});
```

### 9.3 组合式Store的类型

```typescript
/**
 * 组合式Store的类型示例
 * @author erik.zhou
 */
import { ref, computed, Ref, ComputedRef } from 'vue';
import { defineStore } from 'pinia';

interface Task {
    id: number;
    title: string;
    completed: boolean;
}

interface TaskStore {
    tasks: Ref<Task[]>;
    filter: Ref<'all' | 'active' | 'completed'>;
    filteredTasks: ComputedRef<Task[]>;
    addTask: (title: string) => void;
    removeTask: (id: number) => void;
    toggleTask: (id: number) => void;
}

export const useTaskStore = defineStore('task', (): TaskStore => {
    const tasks = ref<Task[]>([]);
    const filter = ref<'all' | 'active' | 'completed'>('all');
    
    const filteredTasks = computed(() => {
        switch (filter.value) {
            case 'active':
                return tasks.value.filter(t => !t.completed);
            case 'completed':
                return tasks.value.filter(t => t.completed);
            default:
                return tasks.value;
        }
    });
    
    function addTask(title: string): void {
        tasks.value.push({
            id: Date.now(),
            title,
            completed: false
        });
    }
    
    function removeTask(id: number): void {
        const index = tasks.value.findIndex(t => t.id === id);
        if (index !== -1) {
            tasks.value.splice(index, 1);
        }
    }
    
    function toggleTask(id: number): void {
        const task = tasks.value.find(t => t.id === id);
        if (task) {
            task.completed = !task.completed;
        }
    }
    
    return {
        tasks,
        filter,
        filteredTasks,
        addTask,
        removeTask,
        toggleTask
    };
});
```

### 9.4 扩展Pinia类型

```typescript
/**
 * 扩展Pinia类型
 * @author erik.zhou
 */
import 'pinia';

declare module 'pinia' {
    export interface PiniaCustomProperties {
        // 添加自定义属性
        $router: Router;
        $api: ApiClient;
        
        // 添加自定义方法
        $reset: () => void;
        $hydrate: (data: any) => void;
        $serialize: () => string;
    }
    
    export interface PiniaCustomStateProperties<S> {
        // 添加到每个state的属性
        $loading: boolean;
        $error: string | null;
    }
}

// 使用扩展的类型
export const useUserStore = defineStore('user', {
    state: () => ({
        name: ''
    }),
    actions: {
        async fetchUser() {
            // 可以访问自定义属性
            const data = await this.$api.get('/user');
            this.name = data.name;
        },
        
        navigateToProfile() {
            // 可以访问router
            this.$router.push('/profile');
        }
    }
});
```


---

## 第10章 最佳实践

### 10.1 Store组织结构

```typescript
/**
 * Store组织结构最佳实践
 * @author erik.zhou
 */

// stores/
// ├── index.ts              # 导出所有stores
// ├── modules/
// │   ├── user.ts          # 用户相关
// │   ├── auth.ts          # 认证相关
// │   ├── cart.ts          # 购物车
// │   └── product.ts       # 产品
// └── plugins/
//     ├── persist.ts       # 持久化插件
//     └── logger.ts        # 日志插件

// stores/index.ts
export { useUserStore } from './modules/user';
export { useAuthStore } from './modules/auth';
export { useCartStore } from './modules/cart';
export { useProductStore } from './modules/product';

// 在组件中使用
import { useUserStore, useCartStore } from '@/stores';
```

### 10.2 命名规范

```typescript
/**
 * 命名规范最佳实践
 * @author erik.zhou
 */

// ✅ 推荐：清晰的命名
export const useUserStore = defineStore('user', {
    state: () => ({
        currentUser: null,      // 明确表示当前用户
        isAuthenticated: false, // 布尔值用is前缀
        userList: []           // 列表用List后缀
    }),
    
    getters: {
        // getter用名词或形容词
        fullName: (state) => `${state.firstName} ${state.lastName}`,
        isAdmin: (state) => state.role === 'admin',
        activeUsers: (state) => state.userList.filter(u => u.active)
    },
    
    actions: {
        // action用动词开头
        fetchUser() {},
        updateUser() {},
        deleteUser() {},
        setAuthenticated() {}
    }
});

// ❌ 不推荐：模糊的命名
export const useUserStore = defineStore('user', {
    state: () => ({
        data: null,        // 太模糊
        flag: false,       // 不知道是什么标志
        list: []          // 什么的列表？
    }),
    
    getters: {
        get: (state) => state.data,  // 太简单
        check: (state) => true       // 检查什么？
    },
    
    actions: {
        do() {},          // 做什么？
        handle() {}       // 处理什么？
    }
});
```

### 10.3 状态设计原则

```typescript
/**
 * 状态设计原则
 * @author erik.zhou
 */

// ✅ 推荐：扁平化状态
export const useUserStore = defineStore('user', {
    state: () => ({
        userId: null,
        userName: '',
        userEmail: '',
        userRole: 'guest'
    })
});

// ❌ 不推荐：过度嵌套
export const useUserStore = defineStore('user', {
    state: () => ({
        user: {
            info: {
                basic: {
                    id: null,
                    name: ''
                }
            }
        }
    })
});

// ✅ 推荐：合理分组
export const useUserStore = defineStore('user', {
    state: () => ({
        // 基本信息
        id: null,
        name: '',
        email: '',
        
        // 配置信息
        settings: {
            theme: 'light',
            language: 'zh-CN'
        },
        
        // UI状态
        loading: false,
        error: null
    })
});
```

### 10.4 Actions最佳实践

```typescript
/**
 * Actions最佳实践
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
    state: () => ({
        user: null as any,
        loading: false,
        error: null as string | null
    }),
    
    actions: {
        // ✅ 推荐：统一的错误处理
        async fetchUser(id: number) {
            this.loading = true;
            this.error = null;
            
            try {
                const response = await fetch(`/api/users/${id}`);
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                this.user = await response.json();
            } catch (error) {
                this.error = error instanceof Error ? error.message : 'Unknown error';
                console.error('Failed to fetch user:', error);
                throw error; // 重新抛出以便调用者处理
            } finally {
                this.loading = false;
            }
        },
        
        // ✅ 推荐：职责单一
        async updateUserName(name: string) {
            if (!this.user) {
                throw new Error('No user loaded');
            }
            
            const response = await fetch(`/api/users/${this.user.id}`, {
                method: 'PATCH',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ name })
            });
            
            if (response.ok) {
                this.user.name = name;
            }
        },
        
        // ✅ 推荐：可复用的辅助方法
        async request<T>(url: string, options?: RequestInit): Promise<T> {
            this.loading = true;
            this.error = null;
            
            try {
                const response = await fetch(url, options);
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                return await response.json();
            } catch (error) {
                this.error = error instanceof Error ? error.message : 'Unknown error';
                throw error;
            } finally {
                this.loading = false;
            }
        }
    }
});
```

### 10.5 性能优化

```typescript
/**
 * 性能优化最佳实践
 * @author erik.zhou
 */
import { defineStore } from 'pinia';
import { computed } from 'vue';

export const useProductStore = defineStore('product', () => {
    const products = ref<any[]>([]);
    
    // ✅ 推荐：使用computed缓存计算结果
    const expensiveComputation = computed(() => {
        return products.value
            .filter(p => p.active)
            .map(p => ({
                ...p,
                discountPrice: p.price * 0.9
            }))
            .sort((a, b) => b.price - a.price);
    });
    
    // ✅ 推荐：批量更新使用$patch
    function updateMultipleProducts(updates: any[]) {
        this.$patch((state) => {
            updates.forEach(update => {
                const index = state.products.findIndex(p => p.id === update.id);
                if (index !== -1) {
                    state.products[index] = { ...state.products[index], ...update };
                }
            });
        });
    }
    
    // ✅ 推荐：避免在getter中进行复杂计算
    const productMap = computed(() => {
        return new Map(products.value.map(p => [p.id, p]));
    });
    
    const getProductById = (id: number) => {
        return productMap.value.get(id);
    };
    
    return {
        products,
        expensiveComputation,
        productMap,
        getProductById,
        updateMultipleProducts
    };
});
```

### 10.6 测试Store

```typescript
/**
 * 测试Store示例
 * @author erik.zhou
 */
import { setActivePinia, createPinia } from 'pinia';
import { describe, it, expect, beforeEach } from 'vitest';
import { useCounterStore } from '@/stores/counter';

describe('Counter Store', () => {
    beforeEach(() => {
        // 为每个测试创建新的pinia实例
        setActivePinia(createPinia());
    });
    
    it('increments counter', () => {
        const counter = useCounterStore();
        expect(counter.count).toBe(0);
        
        counter.increment();
        expect(counter.count).toBe(1);
    });
    
    it('increments by amount', () => {
        const counter = useCounterStore();
        counter.incrementBy(5);
        expect(counter.count).toBe(5);
    });
    
    it('computes double count', () => {
        const counter = useCounterStore();
        counter.count = 5;
        expect(counter.doubleCount).toBe(10);
    });
    
    it('resets counter', () => {
        const counter = useCounterStore();
        counter.count = 10;
        counter.$reset();
        expect(counter.count).toBe(0);
    });
});

// 测试异步actions
describe('User Store', () => {
    beforeEach(() => {
        setActivePinia(createPinia());
    });
    
    it('fetches user successfully', async () => {
        const userStore = useUserStore();
        
        // Mock fetch
        global.fetch = vi.fn(() =>
            Promise.resolve({
                ok: true,
                json: () => Promise.resolve({ id: 1, name: 'John' })
            } as Response)
        );
        
        await userStore.fetchUser(1);
        
        expect(userStore.user).toEqual({ id: 1, name: 'John' });
        expect(userStore.loading).toBe(false);
        expect(userStore.error).toBeNull();
    });
    
    it('handles fetch error', async () => {
        const userStore = useUserStore();
        
        global.fetch = vi.fn(() =>
            Promise.reject(new Error('Network error'))
        );
        
        await expect(userStore.fetchUser(1)).rejects.toThrow('Network error');
        expect(userStore.error).toBe('Network error');
        expect(userStore.loading).toBe(false);
    });
});
```

### 10.7 常见陷阱

```typescript
/**
 * 常见陷阱和解决方案
 * @author erik.zhou
 */

// ❌ 陷阱1：解构丢失响应性
const userStore = useUserStore();
const { name, email } = userStore; // 丢失响应性

// ✅ 解决方案：使用storeToRefs
import { storeToRefs } from 'pinia';
const { name, email } = storeToRefs(userStore);

// ❌ 陷阱2：在setup外使用store
const userStore = useUserStore(); // 错误：在setup外调用

export default {
    setup() {
        const userStore = useUserStore(); // 正确
    }
};

// ❌ 陷阱3：直接修改getter
const userStore = useUserStore();
userStore.fullName = 'New Name'; // 错误：getter是只读的

// ✅ 解决方案：通过action修改state
userStore.updateName('New Name');

// ❌ 陷阱4：在getter中修改state
getters: {
    badGetter: (state) => {
        state.count++; // 错误：getter不应该修改state
        return state.count;
    }
}

// ✅ 解决方案：getter只用于计算
getters: {
    goodGetter: (state) => {
        return state.count * 2;
    }
}

// ❌ 陷阱5：忘记处理异步错误
actions: {
    async fetchData() {
        const data = await fetch('/api/data'); // 可能抛出错误
        this.data = data;
    }
}

// ✅ 解决方案：添加错误处理
actions: {
    async fetchData() {
        try {
            const response = await fetch('/api/data');
            if (!response.ok) throw new Error('Fetch failed');
            this.data = await response.json();
        } catch (error) {
            this.error = error.message;
            throw error;
        }
    }
}
```

---

## 总结

### 核心要点

1. **Store定义**
   - 选项式Store适合简单场景
   - 组合式Store提供更大灵活性
   - 合理组织Store结构

2. **State管理**
   - 保持状态扁平化
   - 使用TypeScript类型
   - 避免过度嵌套

3. **Getters使用**
   - 用于派生状态
   - 可以访问其他getters
   - 返回函数实现参数化

4. **Actions编写**
   - 统一错误处理
   - 职责单一原则
   - 合理使用异步

5. **插件系统**
   - 扩展Store功能
   - 实现持久化
   - 添加性能监控

6. **性能优化**
   - 使用computed缓存
   - 批量更新用$patch
   - 避免不必要的计算

### 学习路径

1. **基础阶段**
   - 理解Pinia核心概念
   - 掌握Store定义方法
   - 学会State、Getters、Actions

2. **进阶阶段**
   - 组合式Store编写
   - 插件系统使用
   - TypeScript集成

3. **高级阶段**
   - 性能优化技巧
   - 测试Store
   - 大型应用架构

### 推荐资源

- [Pinia官方文档](https://pinia.vuejs.org/)
- [Pinia GitHub](https://github.com/vuejs/pinia)
- [Vue Mastery - Pinia课程](https://www.vuemastery.com/)

---

## 附录

### A. 常用API速查

```typescript
/**
 * Pinia常用API速查
 * @author erik.zhou
 */

// 创建Pinia实例
import { createPinia } from 'pinia';
const pinia = createPinia();

// 定义Store
import { defineStore } from 'pinia';
const useStore = defineStore('id', { /* options */ });

// 使用Store
const store = useStore();

// 访问state
store.count;
store.$state.count;

// 修改state
store.count = 1;
store.$patch({ count: 1 });
store.$patch((state) => { state.count = 1; });

// 重置state
store.$reset();

// 订阅state变化
store.$subscribe((mutation, state) => {});

// 订阅actions
store.$onAction(({ name, args, after, onError }) => {});

// 使用storeToRefs
import { storeToRefs } from 'pinia';
const { count } = storeToRefs(store);
```

### B. TypeScript类型定义

```typescript
/**
 * TypeScript类型定义参考
 * @author erik.zhou
 */
import { Store, StateTree, _GettersTree } from 'pinia';

// Store类型
type MyStore = Store<
    'myStore',
    StateTree,
    _GettersTree<StateTree>,
    {}
>;

// 扩展PiniaCustomProperties
declare module 'pinia' {
    export interface PiniaCustomProperties {
        $myPlugin: MyPlugin;
    }
}
```

### C. 常见问题FAQ

**Q: Pinia和Vuex有什么区别？**
A: Pinia更简洁，没有mutations，更好的TypeScript支持，更小的包体积。

**Q: 如何在组件外使用Store？**
A: 确保在createApp之后使用，或者在函数内部调用useStore。

**Q: 如何实现Store之间的通信？**
A: 直接在一个Store的action中调用另一个Store的方法。

**Q: 如何持久化Store？**
A: 使用pinia-plugin-persistedstate插件或自定义插件。

**Q: 如何测试Store？**
A: 使用setActivePinia创建测试环境，然后正常测试Store的方法。

---

**@author erik.zhou**  
**最后更新**: 2026-03-02  
**版本**: 1.0.0

