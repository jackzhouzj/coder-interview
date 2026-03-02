# Zustand - 完整教程

## 课程信息
- **课程名称**: Zustand完整教程
- **难度级别**: 中级
- **预计学时**: 4小时
- **核心内容**: 状态管理、中间件、持久化、TypeScript集成
- **@author**: erik.zhou

---

## 目录
1. [Zustand概述](#1-zustand概述)
2. [基础使用](#2-基础使用)
3. [状态更新](#3-状态更新)
4. [异步操作](#4-异步操作)
5. [中间件系统](#5-中间件系统)
6. [状态持久化](#6-状态持久化)
7. [TypeScript集成](#7-typescript集成)
8. [性能优化](#8-性能优化)
9. [最佳实践](#9-最佳实践)
10. [实战案例](#10-实战案例)

---

## 1. Zustand概述

### 1.1 什么是Zustand

Zustand是一个轻量级的React状态管理库，提供简单直观的API和优秀的性能。

**核心特点**:
- 极简API，学习成本低
- 无需Provider包裹
- 支持TypeScript
- 体积小（约1KB）
- 性能优异
- 支持中间件

### 1.2 为什么选择Zustand

```javascript
// 对比其他状态管理方案
const comparison = {
    redux: {
        优点: ['生态成熟', '工具完善', '社区活跃'],
        缺点: ['样板代码多', '学习曲线陡', '配置复杂']
    },
    mobx: {
        优点: ['响应式', '代码简洁', '易于理解'],
        缺点: ['魔法较多', '调试困难', '体积较大']
    },
    zustand: {
        优点: ['API简单', '无样板代码', '体积小', '性能好'],
        缺点: ['生态较小', '高级功能需自行实现']
    }
};
```

### 1.3 安装配置

```bash
# 安装Zustand
npm install zustand

# 或使用yarn
yarn add zustand

# 或使用pnpm
pnpm add zustand
```

---

## 2. 基础使用

### 2.1 创建Store

```javascript
// store/counterStore.js
/**
 * 计数器Store
 * @author erik.zhou
 */
import { create } from 'zustand';

const useCounterStore = create((set) => ({
    // 状态
    count: 0,
    
    // 操作方法
    increment: () => set((state) => ({ count: state.count + 1 })),
    decrement: () => set((state) => ({ count: state.count - 1 })),
    reset: () => set({ count: 0 })
}));

export default useCounterStore;
```

### 2.2 在组件中使用

```javascript
// Counter.jsx
/**
 * 计数器组件
 * @author erik.zhou
 */
import React from 'react';
import useCounterStore from './store/counterStore';

function Counter() {
    // 订阅整个store
    const { count, increment, decrement, reset } = useCounterStore();
    
    return (
        <div>
            <h2>计数: {count}</h2>
            <button onClick={increment}>+1</button>
            <button onClick={decrement}>-1</button>
            <button onClick={reset}>重置</button>
        </div>
    );
}

export default Counter;
```

### 2.3 选择性订阅

```javascript
// OptimizedCounter.jsx
/**
 * 优化的计数器组件
 * @author erik.zhou
 */
import React from 'react';
import useCounterStore from './store/counterStore';

function OptimizedCounter() {
    // 只订阅需要的状态
    const count = useCounterStore((state) => state.count);
    const increment = useCounterStore((state) => state.increment);
    
    console.log('组件重渲染');
    
    return (
        <div>
            <h2>计数: {count}</h2>
            <button onClick={increment}>+1</button>
        </div>
    );
}

// 显示组件
function Display() {
    // 只订阅count，不会因为其他状态变化而重渲染
    const count = useCounterStore((state) => state.count);
    
    return <div>当前计数: {count}</div>;
}

export default OptimizedCounter;
```

### 2.4 多个Store

```javascript
// store/userStore.js
/**
 * 用户Store
 * @author erik.zhou
 */
import { create } from 'zustand';

const useUserStore = create((set) => ({
    user: null,
    isLoading: false,
    error: null,
    
    setUser: (user) => set({ user }),
    clearUser: () => set({ user: null }),
    setLoading: (isLoading) => set({ isLoading }),
    setError: (error) => set({ error })
}));

export default useUserStore;
```

```javascript
// store/todoStore.js
/**
 * Todo Store
 * @author erik.zhou
 */
import { create } from 'zustand';

const useTodoStore = create((set) => ({
    todos: [],
    filter: 'all',
    
    addTodo: (text) => set((state) => ({
        todos: [...state.todos, {
            id: Date.now(),
            text,
            completed: false
        }]
    })),
    
    toggleTodo: (id) => set((state) => ({
        todos: state.todos.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        )
    })),
    
    deleteTodo: (id) => set((state) => ({
        todos: state.todos.filter(todo => todo.id !== id)
    })),
    
    setFilter: (filter) => set({ filter })
}));

export default useTodoStore;
```

---

## 3. 状态更新

### 3.1 直接更新

```javascript
// store/directUpdateStore.js
/**
 * 直接更新示例
 * @author erik.zhou
 */
import { create } from 'zustand';

const useStore = create((set) => ({
    name: 'John',
    age: 25,
    
    // 直接设置新值
    setName: (name) => set({ name }),
    setAge: (age) => set({ age }),
    
    // 同时更新多个值
    updateUser: (name, age) => set({ name, age })
}));

export default useStore;
```

### 3.2 基于前一个状态更新

```javascript
// store/stateBasedUpdateStore.js
/**
 * 基于前一个状态更新
 * @author erik.zhou
 */
import { create } from 'zustand';

const useStore = create((set) => ({
    count: 0,
    items: [],
    
    // 基于前一个状态
    increment: () => set((state) => ({ 
        count: state.count + 1 
    })),
    
    incrementBy: (amount) => set((state) => ({ 
        count: state.count + amount 
    })),
    
    addItem: (item) => set((state) => ({
        items: [...state.items, item]
    })),
    
    removeItem: (id) => set((state) => ({
        items: state.items.filter(item => item.id !== id)
    })),
    
    updateItem: (id, updates) => set((state) => ({
        items: state.items.map(item =>
            item.id === id ? { ...item, ...updates } : item
        )
    }))
}));

export default useStore;
```

### 3.3 使用get访问当前状态

```javascript
// store/getStateStore.js
/**
 * 使用get访问状态
 * @author erik.zhou
 */
import { create } from 'zustand';

const useStore = create((set, get) => ({
    count: 0,
    multiplier: 2,
    
    increment: () => set((state) => ({ 
        count: state.count + 1 
    })),
    
    // 使用get获取当前状态
    getTotal: () => {
        const { count, multiplier } = get();
        return count * multiplier;
    },
    
    // 在action中使用get
    doubleCount: () => {
        const currentCount = get().count;
        set({ count: currentCount * 2 });
    },
    
    // 复杂计算
    calculateResult: () => {
        const state = get();
        const result = state.count * state.multiplier;
        set({ result });
    }
}));

export default useStore;
```

---

## 4. 异步操作

### 4.1 基础异步操作

```javascript
// store/asyncStore.js
/**
 * 异步操作Store
 * @author erik.zhou
 */
import { create } from 'zustand';

const useAsyncStore = create((set) => ({
    data: null,
    isLoading: false,
    error: null,
    
    // 异步获取数据
    fetchData: async () => {
        set({ isLoading: true, error: null });
        
        try {
            const response = await fetch('https://api.example.com/data');
            const data = await response.json();
            set({ data, isLoading: false });
        } catch (error) {
            set({ error: error.message, isLoading: false });
        }
    },
    
    // 异步创建数据
    createData: async (payload) => {
        set({ isLoading: true, error: null });
        
        try {
            const response = await fetch('https://api.example.com/data', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const newData = await response.json();
            set((state) => ({
                data: [...(state.data || []), newData],
                isLoading: false
            }));
        } catch (error) {
            set({ error: error.message, isLoading: false });
        }
    }
}));

export default useAsyncStore;
```

### 4.2 使用组件

```javascript
// DataList.jsx
/**
 * 数据列表组件
 * @author erik.zhou
 */
import React, { useEffect } from 'react';
import useAsyncStore from './store/asyncStore';

function DataList() {
    const { data, isLoading, error, fetchData } = useAsyncStore();
    
    useEffect(() => {
        fetchData();
    }, [fetchData]);
    
    if (isLoading) {
        return <div>加载中...</div>;
    }
    
    if (error) {
        return <div>错误: {error}</div>;
    }
    
    return (
        <div>
            <h2>数据列表</h2>
            {data && data.map(item => (
                <div key={item.id}>{item.name}</div>
            ))}
        </div>
    );
}

export default DataList;
```

### 4.3 Promise处理

```javascript
// store/promiseStore.js
/**
 * Promise处理Store
 * @author erik.zhou
 */
import { create } from 'zustand';

const usePromiseStore = create((set, get) => ({
    users: [],
    isLoading: false,
    error: null,
    
    // 返回Promise的action
    fetchUsers: async () => {
        set({ isLoading: true, error: null });
        
        try {
            const response = await fetch('/api/users');
            if (!response.ok) {
                throw new Error('获取用户失败');
            }
            const users = await response.json();
            set({ users, isLoading: false });
            return users;
        } catch (error) {
            set({ error: error.message, isLoading: false });
            throw error;
        }
    },
    
    // 链式调用
    fetchAndProcessUsers: async () => {
        const users = await get().fetchUsers();
        const processedUsers = users.map(user => ({
            ...user,
            fullName: `${user.firstName} ${user.lastName}`
        }));
        set({ users: processedUsers });
        return processedUsers;
    }
}));

export default usePromiseStore;
```

---

## 5. 中间件系统

### 5.1 persist中间件（持久化）

```javascript
// store/persistStore.js
/**
 * 持久化Store
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const usePersistStore = create(
    persist(
        (set) => ({
            theme: 'light',
            language: 'zh-CN',
            
            setTheme: (theme) => set({ theme }),
            setLanguage: (language) => set({ language })
        }),
        {
            name: 'app-settings', // localStorage key
            // 可选配置
            partialize: (state) => ({
                theme: state.theme,
                language: state.language
            })
        }
    )
);

export default usePersistStore;
```

### 5.2 immer中间件（不可变更新）

```javascript
// store/immerStore.js
/**
 * Immer中间件Store
 * @author erik.zhou
 */
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

const useImmerStore = create(
    immer((set) => ({
        user: {
            name: 'John',
            age: 25,
            address: {
                city: 'Beijing',
                street: 'Main St'
            }
        },
        
        // 使用immer可以直接修改
        updateUserName: (name) => set((state) => {
            state.user.name = name;
        }),
        
        updateCity: (city) => set((state) => {
            state.user.address.city = city;
        }),
        
        incrementAge: () => set((state) => {
            state.user.age += 1;
        })
    }))
);

export default useImmerStore;
```

### 5.3 devtools中间件（开发工具）

```javascript
// store/devtoolsStore.js
/**
 * DevTools中间件Store
 * @author erik.zhou
 */
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

const useDevtoolsStore = create(
    devtools(
        (set) => ({
            count: 0,
            
            increment: () => set(
                (state) => ({ count: state.count + 1 }),
                false,
                'increment' // action名称
            ),
            
            decrement: () => set(
                (state) => ({ count: state.count - 1 }),
                false,
                'decrement'
            )
        }),
        {
            name: 'CounterStore', // DevTools中显示的名称
            enabled: process.env.NODE_ENV === 'development'
        }
    )
);

export default useDevtoolsStore;
```

### 5.4 组合多个中间件

```javascript
// store/combinedMiddlewareStore.js
/**
 * 组合多个中间件
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

const useCombinedStore = create(
    devtools(
        persist(
            immer((set) => ({
                todos: [],
                filter: 'all',
                
                addTodo: (text) => set((state) => {
                    state.todos.push({
                        id: Date.now(),
                        text,
                        completed: false
                    });
                }),
                
                toggleTodo: (id) => set((state) => {
                    const todo = state.todos.find(t => t.id === id);
                    if (todo) {
                        todo.completed = !todo.completed;
                    }
                }),
                
                deleteTodo: (id) => set((state) => {
                    state.todos = state.todos.filter(t => t.id !== id);
                }),
                
                setFilter: (filter) => set((state) => {
                    state.filter = filter;
                })
            })),
            {
                name: 'todo-storage'
            }
        ),
        {
            name: 'TodoStore'
        }
    )
);

export default useCombinedStore;
```

### 5.5 自定义中间件

```javascript
// middleware/logger.js
/**
 * 日志中间件
 * @author erik.zhou
 */
const logger = (config) => (set, get, api) =>
    config(
        (...args) => {
            console.log('  applying', args);
            set(...args);
            console.log('  new state', get());
        },
        get,
        api
    );

export default logger;
```

```javascript
// store/customMiddlewareStore.js
/**
 * 使用自定义中间件
 * @author erik.zhou
 */
import { create } from 'zustand';
import logger from '../middleware/logger';

const useCustomStore = create(
    logger((set) => ({
        count: 0,
        increment: () => set((state) => ({ count: state.count + 1 }))
    }))
);

export default useCustomStore;
```

---

## 6. 状态持久化

### 6.1 localStorage持久化

```javascript
// store/localStorageStore.js
/**
 * localStorage持久化
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const useLocalStorageStore = create(
    persist(
        (set) => ({
            user: null,
            token: null,
            preferences: {
                theme: 'light',
                language: 'zh-CN'
            },
            
            setUser: (user) => set({ user }),
            setToken: (token) => set({ token }),
            updatePreferences: (preferences) => set((state) => ({
                preferences: { ...state.preferences, ...preferences }
            })),
            logout: () => set({ user: null, token: null })
        }),
        {
            name: 'user-storage',
            storage: createJSONStorage(() => localStorage),
            // 部分持久化
            partialize: (state) => ({
                user: state.user,
                token: state.token,
                preferences: state.preferences
            })
        }
    )
);

export default useLocalStorageStore;
```

### 6.2 sessionStorage持久化

```javascript
// store/sessionStorageStore.js
/**
 * sessionStorage持久化
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const useSessionStorageStore = create(
    persist(
        (set) => ({
            formData: {},
            currentStep: 1,
            
            updateFormData: (data) => set((state) => ({
                formData: { ...state.formData, ...data }
            })),
            
            setStep: (step) => set({ currentStep: step }),
            
            resetForm: () => set({
                formData: {},
                currentStep: 1
            })
        }),
        {
            name: 'form-session',
            storage: createJSONStorage(() => sessionStorage)
        }
    )
);

export default useSessionStorageStore;
```

### 6.3 自定义存储

```javascript
// storage/customStorage.js
/**
 * 自定义存储实现
 * @author erik.zhou
 */
const customStorage = {
    getItem: async (name) => {
        try {
            const value = await AsyncStorage.getItem(name);
            return value ? JSON.parse(value) : null;
        } catch (error) {
            console.error('获取存储失败:', error);
            return null;
        }
    },
    
    setItem: async (name, value) => {
        try {
            await AsyncStorage.setItem(name, JSON.stringify(value));
        } catch (error) {
            console.error('设置存储失败:', error);
        }
    },
    
    removeItem: async (name) => {
        try {
            await AsyncStorage.removeItem(name);
        } catch (error) {
            console.error('删除存储失败:', error);
        }
    }
};

export default customStorage;
```

```javascript
// store/customStorageStore.js
/**
 * 使用自定义存储
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import customStorage from '../storage/customStorage';

const useCustomStorageStore = create(
    persist(
        (set) => ({
            settings: {},
            updateSettings: (settings) => set({ settings })
        }),
        {
            name: 'app-settings',
            storage: customStorage
        }
    )
);

export default useCustomStorageStore;
```

---

## 7. TypeScript集成

### 7.1 基础类型定义

```typescript
// store/typedStore.ts
/**
 * TypeScript类型定义
 * @author erik.zhou
 */
import { create } from 'zustand';

interface User {
    id: number;
    name: string;
    email: string;
}

interface UserState {
    user: User | null;
    isLoading: boolean;
    error: string | null;
    setUser: (user: User) => void;
    clearUser: () => void;
    setLoading: (isLoading: boolean) => void;
    setError: (error: string | null) => void;
}

const useUserStore = create<UserState>((set) => ({
    user: null,
    isLoading: false,
    error: null,
    
    setUser: (user) => set({ user }),
    clearUser: () => set({ user: null }),
    setLoading: (isLoading) => set({ isLoading }),
    setError: (error) => set({ error })
}));

export default useUserStore;
```

### 7.2 复杂类型定义

```typescript
// store/complexTypedStore.ts
/**
 * 复杂类型定义
 * @author erik.zhou
 */
import { create } from 'zustand';

interface Todo {
    id: number;
    text: string;
    completed: boolean;
    createdAt: Date;
}

type Filter = 'all' | 'active' | 'completed';

interface TodoState {
    todos: Todo[];
    filter: Filter;
    addTodo: (text: string) => void;
    toggleTodo: (id: number) => void;
    deleteTodo: (id: number) => void;
    setFilter: (filter: Filter) => void;
    getFilteredTodos: () => Todo[];
}

const useTodoStore = create<TodoState>((set, get) => ({
    todos: [],
    filter: 'all',
    
    addTodo: (text) => set((state) => ({
        todos: [...state.todos, {
            id: Date.now(),
            text,
            completed: false,
            createdAt: new Date()
        }]
    })),
    
    toggleTodo: (id) => set((state) => ({
        todos: state.todos.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        )
    })),
    
    deleteTodo: (id) => set((state) => ({
        todos: state.todos.filter(todo => todo.id !== id)
    })),
    
    setFilter: (filter) => set({ filter }),
    
    getFilteredTodos: () => {
        const { todos, filter } = get();
        switch (filter) {
            case 'active':
                return todos.filter(todo => !todo.completed);
            case 'completed':
                return todos.filter(todo => todo.completed);
            default:
                return todos;
        }
    }
}));

export default useTodoStore;
```

### 7.3 中间件类型

```typescript
// store/middlewareTypedStore.ts
/**
 * 中间件类型定义
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface CartItem {
    id: number;
    name: string;
    price: number;
    quantity: number;
}

interface CartState {
    items: CartItem[];
    addItem: (item: Omit<CartItem, 'quantity'>) => void;
    removeItem: (id: number) => void;
    updateQuantity: (id: number, quantity: number) => void;
    clearCart: () => void;
    getTotal: () => number;
}

const useCartStore = create<CartState>()(
    devtools(
        persist(
            immer((set, get) => ({
                items: [],
                
                addItem: (item) => set((state) => {
                    const existingItem = state.items.find(i => i.id === item.id);
                    if (existingItem) {
                        existingItem.quantity += 1;
                    } else {
                        state.items.push({ ...item, quantity: 1 });
                    }
                }),
                
                removeItem: (id) => set((state) => {
                    state.items = state.items.filter(item => item.id !== id);
                }),
                
                updateQuantity: (id, quantity) => set((state) => {
                    const item = state.items.find(i => i.id === id);
                    if (item) {
                        item.quantity = quantity;
                    }
                }),
                
                clearCart: () => set({ items: [] }),
                
                getTotal: () => {
                    return get().items.reduce(
                        (total, item) => total + item.price * item.quantity,
                        0
                    );
                }
            })),
            {
                name: 'cart-storage'
            }
        ),
        {
            name: 'CartStore'
        }
    )
);

export default useCartStore;
```

---

## 8. 性能优化

### 8.1 选择器优化

```javascript
// hooks/useOptimizedSelector.js
/**
 * 优化的选择器Hook
 * @author erik.zhou
 */
import { useStore } from 'zustand';
import { shallow } from 'zustand/shallow';

// 使用shallow比较
function useOptimizedUser() {
    return useStore(
        (state) => ({
            name: state.user.name,
            email: state.user.email
        }),
        shallow
    );
}

// 自定义比较函数
function useCustomSelector(selector, equalityFn) {
    return useStore(selector, equalityFn);
}

export { useOptimizedUser, useCustomSelector };
```

### 8.2 避免不必要的重渲染

```javascript
// OptimizedComponent.jsx
/**
 * 优化的组件
 * @author erik.zhou
 */
import React from 'react';
import useStore from './store';
import { shallow } from 'zustand/shallow';

// 不好的做法 - 订阅整个store
function BadComponent() {
    const store = useStore();
    return <div>{store.user.name}</div>;
}

// 好的做法 - 只订阅需要的数据
function GoodComponent() {
    const userName = useStore((state) => state.user.name);
    return <div>{userName}</div>;
}

// 更好的做法 - 使用shallow比较
function BetterComponent() {
    const { name, email } = useStore(
        (state) => ({
            name: state.user.name,
            email: state.user.email
        }),
        shallow
    );
    
    return (
        <div>
            <div>{name}</div>
            <div>{email}</div>
        </div>
    );
}

export { BadComponent, GoodComponent, BetterComponent };
```

### 8.3 计算属性优化

```javascript
// store/computedStore.js
/**
 * 计算属性优化
 * @author erik.zhou
 */
import { create } from 'zustand';

const useComputedStore = create((set, get) => ({
    items: [],
    filter: 'all',
    
    // 不好的做法 - 每次都重新计算
    getFilteredItems: () => {
        const { items, filter } = get();
        return items.filter(item => {
            if (filter === 'active') {
                return !item.completed;
            }
            if (filter === 'completed') {
                return item.completed;
            }
            return true;
        });
    }
}));

// 好的做法 - 使用useMemo在组件中缓存
function OptimizedComponent() {
    const items = useComputedStore((state) => state.items);
    const filter = useComputedStore((state) => state.filter);
    
    const filteredItems = React.useMemo(() => {
        return items.filter(item => {
            if (filter === 'active') {
                return !item.completed;
            }
            if (filter === 'completed') {
                return item.completed;
            }
            return true;
        });
    }, [items, filter]);
    
    return (
        <div>
            {filteredItems.map(item => (
                <div key={item.id}>{item.text}</div>
            ))}
        </div>
    );
}

export default useComputedStore;
```

---

## 9. 最佳实践

### 9.1 Store组织结构

```javascript
// store/index.js
/**
 * Store统一导出
 * @author erik.zhou
 */
export { default as useUserStore } from './userStore';
export { default as useTodoStore } from './todoStore';
export { default as useCartStore } from './cartStore';
export { default as useSettingsStore } from './settingsStore';
```

```javascript
// store/userStore.js
/**
 * 用户Store - 单一职责
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

const useUserStore = create(
    devtools(
        persist(
            (set) => ({
                // 状态
                user: null,
                isAuthenticated: false,
                
                // Actions
                login: (user) => set({
                    user,
                    isAuthenticated: true
                }),
                
                logout: () => set({
                    user: null,
                    isAuthenticated: false
                }),
                
                updateProfile: (updates) => set((state) => ({
                    user: { ...state.user, ...updates }
                }))
            }),
            {
                name: 'user-storage',
                partialize: (state) => ({
                    user: state.user,
                    isAuthenticated: state.isAuthenticated
                })
            }
        ),
        { name: 'UserStore' }
    )
);

export default useUserStore;
```

### 9.2 Actions命名规范

```javascript
// store/namingConventionStore.js
/**
 * Actions命名规范
 * @author erik.zhou
 */
import { create } from 'zustand';

const useStore = create((set) => ({
    // 状态
    data: [],
    isLoading: false,
    error: null,
    
    // 设置类 - set前缀
    setData: (data) => set({ data }),
    setLoading: (isLoading) => set({ isLoading }),
    setError: (error) => set({ error }),
    
    // 更新类 - update前缀
    updateItem: (id, updates) => set((state) => ({
        data: state.data.map(item =>
            item.id === id ? { ...item, ...updates } : item
        )
    })),
    
    // 添加类 - add前缀
    addItem: (item) => set((state) => ({
        data: [...state.data, item]
    })),
    
    // 删除类 - remove/delete前缀
    removeItem: (id) => set((state) => ({
        data: state.data.filter(item => item.id !== id)
    })),
    
    // 清空类 - clear前缀
    clearData: () => set({ data: [] }),
    
    // 重置类 - reset前缀
    resetStore: () => set({
        data: [],
        isLoading: false,
        error: null
    }),
    
    // 异步操作 - fetch/load前缀
    fetchData: async () => {
        set({ isLoading: true, error: null });
        try {
            const response = await fetch('/api/data');
            const data = await response.json();
            set({ data, isLoading: false });
        } catch (error) {
            set({ error: error.message, isLoading: false });
        }
    }
}));

export default useStore;
```

### 9.3 状态分片

```javascript
// store/slices/userSlice.js
/**
 * 用户状态切片
 * @author erik.zhou
 */
export const createUserSlice = (set) => ({
    user: null,
    isAuthenticated: false,
    
    login: (user) => set({
        user,
        isAuthenticated: true
    }),
    
    logout: () => set({
        user: null,
        isAuthenticated: false
    })
});
```

```javascript
// store/slices/todoSlice.js
/**
 * Todo状态切片
 * @author erik.zhou
 */
export const createTodoSlice = (set) => ({
    todos: [],
    
    addTodo: (text) => set((state) => ({
        todos: [...state.todos, {
            id: Date.now(),
            text,
            completed: false
        }]
    })),
    
    toggleTodo: (id) => set((state) => ({
        todos: state.todos.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        )
    }))
});
```

```javascript
// store/combinedStore.js
/**
 * 组合多个切片
 * @author erik.zhou
 */
import { create } from 'zustand';
import { createUserSlice } from './slices/userSlice';
import { createTodoSlice } from './slices/todoSlice';

const useCombinedStore = create((...args) => ({
    ...createUserSlice(...args),
    ...createTodoSlice(...args)
}));

export default useCombinedStore;
```

### 9.4 错误处理

```javascript
// store/errorHandlingStore.js
/**
 * 错误处理最佳实践
 * @author erik.zhou
 */
import { create } from 'zustand';

const useErrorHandlingStore = create((set) => ({
    data: null,
    isLoading: false,
    error: null,
    
    fetchData: async () => {
        set({ isLoading: true, error: null });
        
        try {
            const response = await fetch('/api/data');
            
            if (!response.ok) {
                throw new Error(`HTTP错误: ${response.status}`);
            }
            
            const data = await response.json();
            set({ data, isLoading: false });
            return { success: true, data };
        } catch (error) {
            const errorMessage = error.message || '未知错误';
            set({ 
                error: errorMessage, 
                isLoading: false 
            });
            
            // 记录错误日志
            console.error('获取数据失败:', error);
            
            return { success: false, error: errorMessage };
        }
    },
    
    clearError: () => set({ error: null })
}));

export default useErrorHandlingStore;
```

---

## 10. 实战案例

### 10.1 购物车系统

```javascript
// store/shoppingCartStore.js
/**
 * 购物车Store
 * @author erik.zhou
 */
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

const useShoppingCartStore = create(
    devtools(
        persist(
            immer((set, get) => ({
                items: [],
                
                // 添加商品
                addItem: (product) => set((state) => {
                    const existingItem = state.items.find(
                        item => item.id === product.id
                    );
                    
                    if (existingItem) {
                        existingItem.quantity += 1;
                    } else {
                        state.items.push({
                            ...product,
                            quantity: 1
                        });
                    }
                }),
                
                // 移除商品
                removeItem: (productId) => set((state) => {
                    state.items = state.items.filter(
                        item => item.id !== productId
                    );
                }),
                
                // 更新数量
                updateQuantity: (productId, quantity) => set((state) => {
                    const item = state.items.find(i => i.id === productId);
                    if (item) {
                        if (quantity <= 0) {
                            state.items = state.items.filter(
                                i => i.id !== productId
                            );
                        } else {
                            item.quantity = quantity;
                        }
                    }
                }),
                
                // 清空购物车
                clearCart: () => set({ items: [] }),
                
                // 计算总价
                getTotal: () => {
                    return get().items.reduce(
                        (total, item) => total + item.price * item.quantity,
                        0
                    );
                },
                
                // 获取商品数量
                getItemCount: () => {
                    return get().items.reduce(
                        (count, item) => count + item.quantity,
                        0
                    );
                },
                
                // 检查商品是否在购物车中
                hasItem: (productId) => {
                    return get().items.some(item => item.id === productId);
                }
            })),
            {
                name: 'shopping-cart',
                partialize: (state) => ({ items: state.items })
            }
        ),
        { name: 'ShoppingCartStore' }
    )
);

export default useShoppingCartStore;
```

```javascript
// components/ShoppingCart.jsx
/**
 * 购物车组件
 * @author erik.zhou
 */
import React from 'react';
import useShoppingCartStore from '../store/shoppingCartStore';

function ShoppingCart() {
    const items = useShoppingCartStore((state) => state.items);
    const removeItem = useShoppingCartStore((state) => state.removeItem);
    const updateQuantity = useShoppingCartStore((state) => state.updateQuantity);
    const clearCart = useShoppingCartStore((state) => state.clearCart);
    const getTotal = useShoppingCartStore((state) => state.getTotal);
    
    if (items.length === 0) {
        return <div>购物车为空</div>;
    }
    
    return (
        <div>
            <h2>购物车</h2>
            {items.map(item => (
                <div key={item.id}>
                    <h3>{item.name}</h3>
                    <p>价格: ¥{item.price}</p>
                    <input
                        type="number"
                        value={item.quantity}
                        onChange={(e) => updateQuantity(
                            item.id,
                            parseInt(e.target.value)
                        )}
                        min="0"
                    />
                    <button onClick={() => removeItem(item.id)}>
                        删除
                    </button>
                </div>
            ))}
            <div>
                <h3>总计: ¥{getTotal().toFixed(2)}</h3>
                <button onClick={clearCart}>清空购物车</button>
            </div>
        </div>
    );
}

export default ShoppingCart;
```

### 10.2 表单管理系统

```javascript
// store/formStore.js
/**
 * 表单管理Store
 * @author erik.zhou
 */
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

const useFormStore = create(
    immer((set, get) => ({
        formData: {},
        errors: {},
        touched: {},
        isSubmitting: false,
        
        // 设置字段值
        setFieldValue: (name, value) => set((state) => {
            state.formData[name] = value;
            // 如果字段已被触摸，立即验证
            if (state.touched[name]) {
                const error = get().validateField(name, value);
                state.errors[name] = error;
            }
        }),
        
        // 设置字段触摸状态
        setFieldTouched: (name, isTouched = true) => set((state) => {
            state.touched[name] = isTouched;
            if (isTouched) {
                const value = state.formData[name];
                const error = get().validateField(name, value);
                state.errors[name] = error;
            }
        }),
        
        // 验证单个字段
        validateField: (name, value) => {
            const validators = {
                email: (val) => {
                    if (!val) {
                        return '邮箱不能为空';
                    }
                    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(val)) {
                        return '邮箱格式不正确';
                    }
                    return '';
                },
                password: (val) => {
                    if (!val) {
                        return '密码不能为空';
                    }
                    if (val.length < 6) {
                        return '密码至少6个字符';
                    }
                    return '';
                },
                username: (val) => {
                    if (!val) {
                        return '用户名不能为空';
                    }
                    if (val.length < 3) {
                        return '用户名至少3个字符';
                    }
                    return '';
                }
            };
            
            const validator = validators[name];
            return validator ? validator(value) : '';
        },
        
        // 验证所有字段
        validateForm: () => {
            const { formData, validateField } = get();
            const newErrors = {};
            
            Object.keys(formData).forEach(name => {
                const error = validateField(name, formData[name]);
                if (error) {
                    newErrors[name] = error;
                }
            });
            
            set({ errors: newErrors });
            return Object.keys(newErrors).length === 0;
        },
        
        // 提交表单
        submitForm: async (onSubmit) => {
            set({ isSubmitting: true });
            
            // 标记所有字段为已触摸
            const { formData } = get();
            const touched = {};
            Object.keys(formData).forEach(key => {
                touched[key] = true;
            });
            set({ touched });
            
            // 验证表单
            const isValid = get().validateForm();
            
            if (!isValid) {
                set({ isSubmitting: false });
                return { success: false, errors: get().errors };
            }
            
            try {
                await onSubmit(formData);
                set({ isSubmitting: false });
                return { success: true };
            } catch (error) {
                set({ 
                    isSubmitting: false,
                    errors: { submit: error.message }
                });
                return { success: false, error: error.message };
            }
        },
        
        // 重置表单
        resetForm: () => set({
            formData: {},
            errors: {},
            touched: {},
            isSubmitting: false
        })
    }))
);

export default useFormStore;
```

```javascript
// components/RegistrationForm.jsx
/**
 * 注册表单组件
 * @author erik.zhou
 */
import React from 'react';
import useFormStore from '../store/formStore';

function RegistrationForm() {
    const {
        formData,
        errors,
        touched,
        isSubmitting,
        setFieldValue,
        setFieldTouched,
        submitForm,
        resetForm
    } = useFormStore();
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        
        const result = await submitForm(async (data) => {
            // 模拟API调用
            await new Promise(resolve => setTimeout(resolve, 1000));
            console.log('提交数据:', data);
        });
        
        if (result.success) {
            alert('注册成功！');
            resetForm();
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <div>
                <label>用户名</label>
                <input
                    type="text"
                    value={formData.username || ''}
                    onChange={(e) => setFieldValue('username', e.target.value)}
                    onBlur={() => setFieldTouched('username')}
                />
                {touched.username && errors.username && (
                    <div style={{ color: 'red' }}>{errors.username}</div>
                )}
            </div>
            
            <div>
                <label>邮箱</label>
                <input
                    type="email"
                    value={formData.email || ''}
                    onChange={(e) => setFieldValue('email', e.target.value)}
                    onBlur={() => setFieldTouched('email')}
                />
                {touched.email && errors.email && (
                    <div style={{ color: 'red' }}>{errors.email}</div>
                )}
            </div>
            
            <div>
                <label>密码</label>
                <input
                    type="password"
                    value={formData.password || ''}
                    onChange={(e) => setFieldValue('password', e.target.value)}
                    onBlur={() => setFieldTouched('password')}
                />
                {touched.password && errors.password && (
                    <div style={{ color: 'red' }}>{errors.password}</div>
                )}
            </div>
            
            {errors.submit && (
                <div style={{ color: 'red' }}>{errors.submit}</div>
            )}
            
            <button type="submit" disabled={isSubmitting}>
                {isSubmitting ? '提交中...' : '注册'}
            </button>
        </form>
    );
}

export default RegistrationForm;
```

### 10.3 通知系统

```javascript
// store/notificationStore.js
/**
 * 通知系统Store
 * @author erik.zhou
 */
import { create } from 'zustand';

const useNotificationStore = create((set, get) => ({
    notifications: [],
    
    // 添加通知
    addNotification: (notification) => {
        const id = Date.now();
        const newNotification = {
            id,
            type: 'info',
            duration: 3000,
            ...notification
        };
        
        set((state) => ({
            notifications: [...state.notifications, newNotification]
        }));
        
        // 自动移除
        if (newNotification.duration > 0) {
            setTimeout(() => {
                get().removeNotification(id);
            }, newNotification.duration);
        }
        
        return id;
    },
    
    // 移除通知
    removeNotification: (id) => set((state) => ({
        notifications: state.notifications.filter(n => n.id !== id)
    })),
    
    // 清空所有通知
    clearNotifications: () => set({ notifications: [] }),
    
    // 便捷方法
    success: (message, options = {}) => {
        return get().addNotification({
            type: 'success',
            message,
            ...options
        });
    },
    
    error: (message, options = {}) => {
        return get().addNotification({
            type: 'error',
            message,
            duration: 5000,
            ...options
        });
    },
    
    warning: (message, options = {}) => {
        return get().addNotification({
            type: 'warning',
            message,
            ...options
        });
    },
    
    info: (message, options = {}) => {
        return get().addNotification({
            type: 'info',
            message,
            ...options
        });
    }
}));

export default useNotificationStore;
```

```javascript
// components/NotificationContainer.jsx
/**
 * 通知容器组件
 * @author erik.zhou
 */
import React from 'react';
import useNotificationStore from '../store/notificationStore';
import './NotificationContainer.css';

function NotificationContainer() {
    const notifications = useNotificationStore((state) => state.notifications);
    const removeNotification = useNotificationStore(
        (state) => state.removeNotification
    );
    
    return (
        <div className="notification-container">
            {notifications.map(notification => (
                <div
                    key={notification.id}
                    className={`notification notification-${notification.type}`}
                >
                    <div className="notification-content">
                        {notification.message}
                    </div>
                    <button
                        className="notification-close"
                        onClick={() => removeNotification(notification.id)}
                    >
                        ×
                    </button>
                </div>
            ))}
        </div>
    );
}

export default NotificationContainer;
```

---

## 总结

### 核心要点

1. **Zustand优势**
   - API简单直观
   - 无需Provider包裹
   - 体积小性能好
   - TypeScript支持完善

2. **状态管理最佳实践**
   - 选择性订阅避免不必要重渲染
   - 使用中间件增强功能
   - 合理组织Store结构
   - 遵循命名规范

3. **性能优化关键**
   - 使用shallow比较
   - 避免订阅整个store
   - 合理使用useMemo
   - 状态分片管理

4. **TypeScript集成**
   - 完整的类型定义
   - 类型安全的actions
   - 中间件类型支持

### 学习路径

1. **基础阶段**（1周）
   - 掌握create API
   - 理解状态更新机制
   - 学习选择性订阅

2. **进阶阶段**（1-2周）
   - 掌握中间件使用
   - 学习状态持久化
   - TypeScript集成

3. **高级阶段**（2-3周）
   - 性能优化技巧
   - 复杂状态管理
   - 实战项目应用

### 推荐资源

1. **官方文档**
   - [Zustand官方文档](https://github.com/pmndrs/zustand)
   - [API参考](https://docs.pmnd.rs/zustand)

2. **学习资源**
   - Zustand最佳实践
   - 性能优化指南
   - 实战案例分析

3. **工具推荐**
   - Redux DevTools
   - React DevTools
   - TypeScript

---

## 附录

### A. 常用API速查

```javascript
// 创建store
import { create } from 'zustand';

// 中间件
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// 工具函数
import { shallow } from 'zustand/shallow';
```

### B. 常见问题

**问题1：如何避免重渲染？**
- 使用选择器只订阅需要的状态
- 使用shallow比较对象
- 合理拆分store

**问题2：如何处理异步操作？**
- 直接在action中使用async/await
- 管理loading和error状态
- 使用try-catch处理错误

**问题3：如何持久化状态？**
- 使用persist中间件
- 配置storage选项
- 使用partialize选择持久化字段

---

**课程结束**

通过本教程，你已经掌握了Zustand的核心概念、中间件系统、性能优化和实战应用。继续实践，不断提升！

**@author**: erik.zhou
