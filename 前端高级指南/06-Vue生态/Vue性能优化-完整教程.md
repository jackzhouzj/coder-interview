# Vue性能优化 - 完整教程

## 课程简介

Vue性能优化是构建高性能Web应用的关键。本教程将深入讲解Vue 3应用的性能优化技巧，包括组件优化、响应式优化、渲染优化、打包优化等方面的最佳实践。

## 学习目标

- 理解Vue性能优化的核心原理
- 掌握组件级别的优化技巧
- 学会响应式系统的优化方法
- 掌握虚拟列表和懒加载技术
- 理解打包和构建优化
- 学会性能监控和分析

## 目录

1. [性能优化概述](#第1章-性能优化概述)
2. [组件优化](#第2章-组件优化)
3. [响应式优化](#第3章-响应式优化)
4. [渲染优化](#第4章-渲染优化)
5. [列表优化](#第5章-列表优化)
6. [路由优化](#第6章-路由优化)
7. [状态管理优化](#第7章-状态管理优化)
8. [打包优化](#第8章-打包优化)
9. [性能监控](#第9章-性能监控)
10. [最佳实践](#第10章-最佳实践)

---

## 第1章 性能优化概述

### 1.1 性能指标

```typescript
/**
 * 性能指标定义
 * @author erik.zhou
 */

// 核心Web指标
interface WebVitals {
    // 最大内容绘制（LCP）- 应小于2.5秒
    lcp: number;
    
    // 首次输入延迟（FID）- 应小于100毫秒
    fid: number;
    
    // 累积布局偏移（CLS）- 应小于0.1
    cls: number;
    
    // 首次内容绘制（FCP）- 应小于1.8秒
    fcp: number;
    
    // 可交互时间（TTI）- 应小于3.8秒
    tti: number;
}

// Vue特定指标
interface VueMetrics {
    // 组件挂载时间
    mountTime: number;
    
    // 组件更新时间
    updateTime: number;
    
    // 响应式依赖数量
    dependencyCount: number;
    
    // 虚拟DOM节点数量
    vnodeCount: number;
}
```

### 1.2 性能分析工具

```typescript
/**
 * 性能分析工具使用
 * @author erik.zhou
 */

// 1. Vue DevTools
// - 组件性能分析
// - 响应式依赖追踪
// - 时间线记录

// 2. Chrome DevTools
// - Performance面板
// - Memory面板
// - Network面板

// 3. Lighthouse
// - 性能评分
// - 优化建议
// - 最佳实践检查

// 4. Web Vitals
import { onLCP, onFID, onCLS } from 'web-vitals';

onLCP(console.log);
onFID(console.log);
onCLS(console.log);
```

### 1.3 性能优化策略

```typescript
/**
 * 性能优化策略概览
 * @author erik.zhou
 */

// 1. 减少不必要的渲染
// - 使用v-once
// - 使用v-memo
// - 合理使用computed

// 2. 优化响应式系统
// - 使用shallowRef/shallowReactive
// - 避免深层响应式
// - 合理使用readonly

// 3. 代码分割
// - 路由懒加载
// - 组件异步加载
// - 动态导入

// 4. 资源优化
// - 图片懒加载
// - 按需加载
// - Tree Shaking

// 5. 打包优化
// - 代码压缩
// - 依赖优化
// - 构建缓存
```

---

## 第2章 组件优化

### 2.1 异步组件

```vue
<!--
  异步组件示例
  @author erik.zhou
-->
<template>
    <div>
        <button @click="showHeavyComponent = true">
            加载重型组件
        </button>
        
        <!-- 异步加载组件 -->
        <Suspense v-if="showHeavyComponent">
            <template #default>
                <HeavyComponent />
            </template>
            <template #fallback>
                <div>加载中...</div>
            </template>
        </Suspense>
    </div>
</template>

<script setup lang="ts">
import { ref, defineAsyncComponent } from 'vue';

const showHeavyComponent = ref(false);

// 异步组件定义
const HeavyComponent = defineAsyncComponent({
    loader: () => import('./HeavyComponent.vue'),
    loadingComponent: () => import('./LoadingSpinner.vue'),
    errorComponent: () => import('./ErrorDisplay.vue'),
    delay: 200,
    timeout: 3000
});
</script>
```

### 2.2 KeepAlive缓存

```vue
<!--
  KeepAlive组件缓存
  @author erik.zhou
-->
<template>
    <div>
        <button @click="currentTab = 'tab1'">Tab 1</button>
        <button @click="currentTab = 'tab2'">Tab 2</button>
        <button @click="currentTab = 'tab3'">Tab 3</button>
        
        <!-- 缓存组件状态 -->
        <KeepAlive :include="['Tab1', 'Tab2']" :max="3">
            <component :is="currentComponent" />
        </KeepAlive>
    </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import Tab1 from './Tab1.vue';
import Tab2 from './Tab2.vue';
import Tab3 from './Tab3.vue';

const currentTab = ref('tab1');

const currentComponent = computed(() => {
    const components = { tab1: Tab1, tab2: Tab2, tab3: Tab3 };
    return components[currentTab.value as keyof typeof components];
});
</script>
```

### 2.3 v-once优化

```vue
<!--
  v-once静态内容优化
  @author erik.zhou
-->
<template>
    <div>
        <!-- 静态内容只渲染一次 -->
        <div v-once>
            <h1>{{ staticTitle }}</h1>
            <p>{{ staticDescription }}</p>
        </div>
        
        <!-- 动态内容正常更新 -->
        <div>
            <p>计数器: {{ count }}</p>
            <button @click="count++">增加</button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const staticTitle = '静态标题';
const staticDescription = '这是静态描述，不会更新';
const count = ref(0);
</script>
```

### 2.4 v-memo优化

```vue
<!--
  v-memo条件缓存
  @author erik.zhou
-->
<template>
    <div>
        <!-- 只有当item.id或selected变化时才重新渲染 -->
        <div
            v-for="item in list"
            :key="item.id"
            v-memo="[item.id, selected === item.id]"
            @click="selected = item.id"
        >
            <h3>{{ item.title }}</h3>
            <p>{{ item.description }}</p>
            <span v-if="selected === item.id">✓ 已选中</span>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const selected = ref<number | null>(null);
const list = ref([
    { id: 1, title: 'Item 1', description: 'Description 1' },
    { id: 2, title: 'Item 2', description: 'Description 2' },
    { id: 3, title: 'Item 3', description: 'Description 3' }
]);
</script>
```

### 2.5 函数式组件

```vue
<!--
  函数式组件优化
  @author erik.zhou
-->
<script setup lang="ts">
/**
 * 函数式组件 - 无状态、无实例
 */
interface Props {
    title: string;
    description: string;
}

defineProps<Props>();
</script>

<template>
    <div class="card">
        <h3>{{ title }}</h3>
        <p>{{ description }}</p>
    </div>
</template>

<style scoped>
.card {
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
}
</style>
```

---

## 第3章 响应式优化

### 3.1 shallowRef和shallowReactive

```typescript
/**
 * 浅层响应式优化
 * @author erik.zhou
 */
import { ref, shallowRef, reactive, shallowReactive } from 'vue';

// 深层响应式（默认）
const deepState = ref({
    user: {
        profile: {
            name: 'John',
            age: 25
        }
    }
});

// 浅层响应式 - 只有根级别是响应式的
const shallowState = shallowRef({
    user: {
        profile: {
            name: 'John',
            age: 25
        }
    }
});

// 修改深层属性不会触发更新
shallowState.value.user.profile.name = 'Jane'; // 不会触发更新

// 替换整个对象会触发更新
shallowState.value = {
    user: {
        profile: {
            name: 'Jane',
            age: 26
        }
    }
}; // 会触发更新

// shallowReactive示例
const shallowObj = shallowReactive({
    count: 0,
    nested: {
        value: 1
    }
});

shallowObj.count++; // 触发更新
shallowObj.nested.value++; // 不触发更新
```

### 3.2 readonly优化

```typescript
/**
 * readonly优化
 * @author erik.zhou
 */
import { ref, readonly, shallowReadonly } from 'vue';

const state = ref({
    count: 0,
    user: {
        name: 'John'
    }
});

// 创建只读副本
const readonlyState = readonly(state.value);

// 尝试修改会警告
// readonlyState.count = 1; // 警告：无法修改只读属性

// 浅层只读
const shallowReadonlyState = shallowReadonly(state.value);

// 根级别只读
// shallowReadonlyState.count = 1; // 警告

// 深层可以修改（但不推荐）
shallowReadonlyState.user.name = 'Jane'; // 可以修改
```

### 3.3 computed缓存

```vue
<!--
  computed缓存优化
  @author erik.zhou
-->
<template>
    <div>
        <p>过滤后的列表: {{ filteredList.length }}</p>
        <p>总价: {{ totalPrice }}</p>
    </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

const items = ref([
    { id: 1, name: 'Item 1', price: 100, active: true },
    { id: 2, name: 'Item 2', price: 200, active: false },
    { id: 3, name: 'Item 3', price: 300, active: true }
]);

// ✅ 使用computed缓存计算结果
const filteredList = computed(() => {
    console.log('计算filteredList');
    return items.value.filter(item => item.active);
});

const totalPrice = computed(() => {
    console.log('计算totalPrice');
    return filteredList.value.reduce((sum, item) => sum + item.price, 0);
});

// ❌ 不推荐：使用方法，每次都会重新计算
function getFilteredList() {
    console.log('计算getFilteredList');
    return items.value.filter(item => item.active);
}
</script>
```

### 3.4 watchEffect优化

```vue
<!--
  watchEffect优化
  @author erik.zhou
-->
<script setup lang="ts">
import { ref, watch, watchEffect } from 'vue';

const count = ref(0);
const doubled = ref(0);

// ✅ 使用watch，明确依赖
watch(count, (newValue) => {
    doubled.value = newValue * 2;
});

// ✅ 使用watchEffect，自动追踪依赖
watchEffect(() => {
    doubled.value = count.value * 2;
});

// ✅ 停止监听
const stop = watchEffect(() => {
    console.log(count.value);
});

// 不再需要时停止
setTimeout(() => {
    stop();
}, 5000);

// ✅ 清理副作用
watchEffect((onCleanup) => {
    const timer = setTimeout(() => {
        console.log(count.value);
    }, 1000);
    
    onCleanup(() => {
        clearTimeout(timer);
    });
});
</script>
```


---

## 第4章 渲染优化

### 4.1 v-show vs v-if

```vue
<!--
  v-show vs v-if选择
  @author erik.zhou
-->
<template>
    <div>
        <!-- ✅ 频繁切换使用v-show -->
        <div v-show="isVisible">
            频繁切换的内容
        </div>
        
        <!-- ✅ 条件很少改变使用v-if -->
        <div v-if="isLoggedIn">
            用户已登录
        </div>
        
        <!-- ✅ 初始条件为false且很少变为true使用v-if -->
        <HeavyComponent v-if="showHeavyComponent" />
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const isVisible = ref(true);
const isLoggedIn = ref(false);
const showHeavyComponent = ref(false);
</script>
```

### 4.2 key优化

```vue
<!--
  key优化示例
  @author erik.zhou
-->
<template>
    <div>
        <!-- ✅ 使用唯一且稳定的key -->
        <div v-for="item in items" :key="item.id">
            {{ item.name }}
        </div>
        
        <!-- ❌ 不推荐：使用index作为key -->
        <div v-for="(item, index) in items" :key="index">
            {{ item.name }}
        </div>
        
        <!-- ✅ 强制替换元素 -->
        <transition>
            <span :key="text">{{ text }}</span>
        </transition>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const items = ref([
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
]);

const text = ref('Hello');
</script>
```

### 4.3 事件处理优化

```vue
<!--
  事件处理优化
  @author erik.zhou
-->
<template>
    <div>
        <!-- ✅ 使用事件修饰符 -->
        <button @click.stop="handleClick">点击</button>
        <form @submit.prevent="handleSubmit">提交</form>
        
        <!-- ✅ 事件委托 -->
        <ul @click="handleItemClick">
            <li v-for="item in items" :key="item.id" :data-id="item.id">
                {{ item.name }}
            </li>
        </ul>
        
        <!-- ✅ 防抖和节流 -->
        <input @input="debouncedSearch" placeholder="搜索" />
        <div @scroll="throttledScroll">滚动内容</div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useDebounceFn, useThrottleFn } from '@vueuse/core';

const items = ref([
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' }
]);

function handleClick() {
    console.log('Clicked');
}

function handleSubmit() {
    console.log('Submitted');
}

// 事件委托
function handleItemClick(event: Event) {
    const target = event.target as HTMLElement;
    if (target.tagName === 'LI') {
        const id = target.dataset.id;
        console.log('Item clicked:', id);
    }
}

// 防抖
const debouncedSearch = useDebounceFn((event: Event) => {
    const value = (event.target as HTMLInputElement).value;
    console.log('Search:', value);
}, 300);

// 节流
const throttledScroll = useThrottleFn(() => {
    console.log('Scrolling');
}, 100);
</script>
```

### 4.4 Teleport优化

```vue
<!--
  Teleport优化
  @author erik.zhou
-->
<template>
    <div>
        <button @click="showModal = true">打开模态框</button>
        
        <!-- 将模态框渲染到body下 -->
        <Teleport to="body">
            <div v-if="showModal" class="modal">
                <div class="modal-content">
                    <h2>模态框</h2>
                    <button @click="showModal = false">关闭</button>
                </div>
            </div>
        </Teleport>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const showModal = ref(false);
</script>

<style scoped>
.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-content {
    background: white;
    padding: 2rem;
    border-radius: 8px;
}
</style>
```

---

## 第5章 列表优化

### 5.1 虚拟列表

```vue
<!--
  虚拟列表实现
  @author erik.zhou
-->
<template>
    <div class="virtual-list" @scroll="handleScroll" ref="containerRef">
        <div class="virtual-list-phantom" :style="{ height: totalHeight + 'px' }"></div>
        <div class="virtual-list-content" :style="{ transform: `translateY(${offset}px)` }">
            <div
                v-for="item in visibleData"
                :key="item.id"
                class="virtual-list-item"
                :style="{ height: itemHeight + 'px' }"
            >
                {{ item.text }}
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

interface Item {
    id: number;
    text: string;
}

const props = defineProps<{
    items: Item[];
    itemHeight: number;
    containerHeight: number;
}>();

const containerRef = ref<HTMLElement>();
const scrollTop = ref(0);

// 计算总高度
const totalHeight = computed(() => {
    return props.items.length * props.itemHeight;
});

// 计算可见数量
const visibleCount = computed(() => {
    return Math.ceil(props.containerHeight / props.itemHeight);
});

// 计算起始索引
const startIndex = computed(() => {
    return Math.floor(scrollTop.value / props.itemHeight);
});

// 计算结束索引
const endIndex = computed(() => {
    return startIndex.value + visibleCount.value;
});

// 计算偏移量
const offset = computed(() => {
    return startIndex.value * props.itemHeight;
});

// 计算可见数据
const visibleData = computed(() => {
    return props.items.slice(startIndex.value, endIndex.value);
});

// 处理滚动
function handleScroll(event: Event) {
    const target = event.target as HTMLElement;
    scrollTop.value = target.scrollTop;
}
</script>

<style scoped>
.virtual-list {
    height: 400px;
    overflow-y: auto;
    position: relative;
}

.virtual-list-phantom {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    z-index: -1;
}

.virtual-list-content {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
}

.virtual-list-item {
    padding: 10px;
    border-bottom: 1px solid #ddd;
}
</style>
```

### 5.2 使用vue-virtual-scroller

```vue
<!--
  使用vue-virtual-scroller库
  @author erik.zhou
-->
<template>
    <RecycleScroller
        class="scroller"
        :items="items"
        :item-size="50"
        key-field="id"
        v-slot="{ item }"
    >
        <div class="item">
            {{ item.text }}
        </div>
    </RecycleScroller>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { RecycleScroller } from 'vue-virtual-scroller';
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css';

const items = ref(
    Array.from({ length: 10000 }, (_, i) => ({
        id: i,
        text: `Item ${i}`
    }))
);
</script>

<style scoped>
.scroller {
    height: 400px;
}

.item {
    padding: 10px;
    border-bottom: 1px solid #ddd;
}
</style>
```

### 5.3 分页加载

```vue
<!--
  分页加载示例
  @author erik.zhou
-->
<template>
    <div>
        <div v-for="item in items" :key="item.id" class="item">
            {{ item.text }}
        </div>
        
        <div v-if="loading" class="loading">加载中...</div>
        
        <button v-if="hasMore && !loading" @click="loadMore">
            加载更多
        </button>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const items = ref<any[]>([]);
const loading = ref(false);
const page = ref(1);
const pageSize = 20;
const hasMore = ref(true);

async function loadMore() {
    if (loading.value || !hasMore.value) {
        return;
    }
    
    loading.value = true;
    
    try {
        const response = await fetch(`/api/items?page=${page.value}&size=${pageSize}`);
        const data = await response.json();
        
        items.value.push(...data.items);
        page.value++;
        hasMore.value = data.hasMore;
    } catch (error) {
        console.error('Failed to load items:', error);
    } finally {
        loading.value = false;
    }
}

// 初始加载
loadMore();
</script>

<style scoped>
.item {
    padding: 1rem;
    border-bottom: 1px solid #ddd;
}

.loading {
    text-align: center;
    padding: 1rem;
}
</style>
```

### 5.4 无限滚动

```vue
<!--
  无限滚动示例
  @author erik.zhou
-->
<template>
    <div ref="containerRef" class="container">
        <div v-for="item in items" :key="item.id" class="item">
            {{ item.text }}
        </div>
        
        <div v-if="loading" class="loading">加载中...</div>
        
        <div ref="sentinelRef" class="sentinel"></div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

const containerRef = ref<HTMLElement>();
const sentinelRef = ref<HTMLElement>();
const items = ref<any[]>([]);
const loading = ref(false);
const page = ref(1);
const hasMore = ref(true);

let observer: IntersectionObserver | null = null;

async function loadMore() {
    if (loading.value || !hasMore.value) {
        return;
    }
    
    loading.value = true;
    
    try {
        const response = await fetch(`/api/items?page=${page.value}&size=20`);
        const data = await response.json();
        
        items.value.push(...data.items);
        page.value++;
        hasMore.value = data.hasMore;
    } catch (error) {
        console.error('Failed to load items:', error);
    } finally {
        loading.value = false;
    }
}

onMounted(() => {
    // 使用IntersectionObserver监听哨兵元素
    observer = new IntersectionObserver(
        (entries) => {
            if (entries[0].isIntersecting) {
                loadMore();
            }
        },
        {
            root: containerRef.value,
            threshold: 0.1
        }
    );
    
    if (sentinelRef.value) {
        observer.observe(sentinelRef.value);
    }
    
    // 初始加载
    loadMore();
});

onUnmounted(() => {
    if (observer) {
        observer.disconnect();
    }
});
</script>

<style scoped>
.container {
    height: 600px;
    overflow-y: auto;
}

.item {
    padding: 1rem;
    border-bottom: 1px solid #ddd;
}

.loading {
    text-align: center;
    padding: 1rem;
}

.sentinel {
    height: 1px;
}
</style>
```

---

## 第6章 路由优化

### 6.1 路由懒加载

```typescript
/**
 * 路由懒加载配置
 * @author erik.zhou
 */
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
    history: createWebHistory(),
    routes: [
        {
            path: '/',
            name: 'Home',
            component: () => import('./views/Home.vue')
        },
        {
            path: '/about',
            name: 'About',
            component: () => import('./views/About.vue')
        },
        {
            path: '/dashboard',
            name: 'Dashboard',
            component: () => import('./views/Dashboard.vue'),
            // 路由元信息
            meta: {
                requiresAuth: true
            }
        }
    ]
});

export default router;
```

### 6.2 路由预加载

```typescript
/**
 * 路由预加载策略
 * @author erik.zhou
 */
import { createRouter } from 'vue-router';

const router = createRouter({
    // ... 路由配置
});

// 预加载策略
router.beforeEach((to, from, next) => {
    // 预加载下一个可能访问的路由
    if (to.name === 'Home') {
        import('./views/About.vue');
    }
    
    next();
});

// 鼠标悬停时预加载
function prefetchRoute(routeName: string) {
    const route = router.resolve({ name: routeName });
    if (route.matched.length > 0) {
        const component = route.matched[0].components?.default;
        if (typeof component === 'function') {
            component();
        }
    }
}
```

### 6.3 路由缓存

```vue
<!--
  路由缓存示例
  @author erik.zhou
-->
<template>
    <div>
        <router-link to="/list">列表</router-link>
        <router-link to="/detail">详情</router-link>
        
        <!-- 缓存路由组件 -->
        <router-view v-slot="{ Component }">
            <KeepAlive :include="['ListView', 'DetailView']">
                <component :is="Component" />
            </KeepAlive>
        </router-view>
    </div>
</template>

<script setup lang="ts">
// 在路由组件中使用生命周期钩子
// activated() - 组件被激活时调用
// deactivated() - 组件被停用时调用
</script>
```

### 6.4 路由过渡动画

```vue
<!--
  路由过渡动画
  @author erik.zhou
-->
<template>
    <router-view v-slot="{ Component, route }">
        <Transition :name="route.meta.transition || 'fade'" mode="out-in">
            <component :is="Component" :key="route.path" />
        </Transition>
    </router-view>
</template>

<style>
/* 淡入淡出 */
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

/* 滑动 */
.slide-enter-active,
.slide-leave-active {
    transition: transform 0.3s ease;
}

.slide-enter-from {
    transform: translateX(100%);
}

.slide-leave-to {
    transform: translateX(-100%);
}
</style>
```


---

## 第7章 状态管理优化

### 7.1 Pinia性能优化

```typescript
/**
 * Pinia性能优化
 * @author erik.zhou
 */
import { defineStore } from 'pinia';
import { computed, ref } from 'vue';

export const useOptimizedStore = defineStore('optimized', () => {
    // 使用shallowRef减少深层响应式开销
    const largeData = shallowRef<any[]>([]);
    
    // 使用computed缓存计算结果
    const filteredData = computed(() => {
        return largeData.value.filter(item => item.active);
    });
    
    // 批量更新使用$patch
    function updateMultiple(updates: any[]) {
        largeData.value = [...largeData.value, ...updates];
    }
    
    return {
        largeData,
        filteredData,
        updateMultiple
    };
});
```

### 7.2 状态分割

```typescript
/**
 * 状态分割优化
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

// ❌ 不推荐：所有状态放在一个store
export const useBigStore = defineStore('big', {
    state: () => ({
        user: {},
        posts: [],
        comments: [],
        settings: {}
    })
});

// ✅ 推荐：按功能分割store
export const useUserStore = defineStore('user', {
    state: () => ({
        user: null,
        profile: null
    })
});

export const usePostStore = defineStore('post', {
    state: () => ({
        posts: [],
        currentPost: null
    })
});

export const useCommentStore = defineStore('comment', {
    state: () => ({
        comments: []
    })
});
```

### 7.3 选择性订阅

```vue
<!--
  选择性订阅状态
  @author erik.zhou
-->
<template>
    <div>
        <p>用户名: {{ userName }}</p>
        <p>邮箱: {{ userEmail }}</p>
    </div>
</template>

<script setup lang="ts">
import { storeToRefs } from 'pinia';
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

// ✅ 只订阅需要的状态
const { userName, userEmail } = storeToRefs(userStore);

// ❌ 不推荐：订阅整个store
// const { user } = storeToRefs(userStore);
</script>
```

### 7.4 状态持久化优化

```typescript
/**
 * 状态持久化优化
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
    state: () => ({
        token: '',
        user: null,
        preferences: {},
        cache: {} // 不需要持久化
    }),
    
    persist: {
        // 只持久化必要的字段
        paths: ['token', 'user', 'preferences'],
        
        // 使用sessionStorage减少存储开销
        storage: sessionStorage,
        
        // 自定义序列化
        serializer: {
            serialize: (state) => {
                // 压缩数据
                return JSON.stringify(state);
            },
            deserialize: (value) => {
                return JSON.parse(value);
            }
        }
    }
});
```

---

## 第8章 打包优化

### 8.1 代码分割

```typescript
/**
 * Vite代码分割配置
 * @author erik.zhou
 */
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [vue()],
    build: {
        rollupOptions: {
            output: {
                // 手动分割代码
                manualChunks: {
                    // Vue核心
                    'vue-vendor': ['vue', 'vue-router', 'pinia'],
                    
                    // UI库
                    'ui-vendor': ['element-plus'],
                    
                    // 工具库
                    'utils': ['lodash-es', 'dayjs', 'axios']
                },
                
                // 分割策略
                chunkFileNames: 'js/[name]-[hash].js',
                entryFileNames: 'js/[name]-[hash].js',
                assetFileNames: '[ext]/[name]-[hash].[ext]'
            }
        },
        
        // 压缩配置
        minify: 'terser',
        terserOptions: {
            compress: {
                drop_console: true,
                drop_debugger: true
            }
        }
    }
});
```

### 8.2 Tree Shaking

```typescript
/**
 * Tree Shaking优化
 * @author erik.zhou
 */

// ✅ 推荐：使用ES模块导入
import { debounce } from 'lodash-es';

// ❌ 不推荐：导入整个库
import _ from 'lodash';

// ✅ 推荐：按需导入组件
import { ElButton, ElInput } from 'element-plus';

// ❌ 不推荐：导入整个组件库
import ElementPlus from 'element-plus';

// package.json配置
{
    "sideEffects": [
        "*.css",
        "*.scss",
        "*.vue"
    ]
}
```

### 8.3 依赖优化

```typescript
/**
 * 依赖优化配置
 * @author erik.zhou
 */
// vite.config.ts
export default defineConfig({
    optimizeDeps: {
        // 预构建依赖
        include: [
            'vue',
            'vue-router',
            'pinia',
            'axios'
        ],
        
        // 排除预构建
        exclude: ['your-local-package']
    },
    
    build: {
        // 启用CSS代码分割
        cssCodeSplit: true,
        
        // 设置chunk大小警告限制
        chunkSizeWarningLimit: 500,
        
        // 启用gzip压缩
        reportCompressedSize: true
    }
});
```

### 8.4 资源优化

```typescript
/**
 * 资源优化配置
 * @author erik.zhou
 */
// vite.config.ts
export default defineConfig({
    build: {
        // 小于10kb的资源内联为base64
        assetsInlineLimit: 10240,
        
        rollupOptions: {
            output: {
                // 静态资源分类
                assetFileNames: (assetInfo) => {
                    const info = assetInfo.name?.split('.');
                    let extType = info?.[info.length - 1];
                    
                    if (/\.(png|jpe?g|gif|svg|webp)$/i.test(assetInfo.name || '')) {
                        extType = 'images';
                    } else if (/\.(woff2?|eot|ttf|otf)$/i.test(assetInfo.name || '')) {
                        extType = 'fonts';
                    }
                    
                    return `${extType}/[name]-[hash][extname]`;
                }
            }
        }
    }
});
```

---

## 第9章 性能监控

### 9.1 性能指标收集

```typescript
/**
 * 性能指标收集
 * @author erik.zhou
 */
import { onLCP, onFID, onCLS, onFCP, onTTFB } from 'web-vitals';

interface PerformanceMetric {
    name: string;
    value: number;
    rating: 'good' | 'needs-improvement' | 'poor';
}

function sendToAnalytics(metric: PerformanceMetric) {
    // 发送到分析服务
    console.log('Performance metric:', metric);
    
    // 示例：发送到Google Analytics
    // gtag('event', metric.name, {
    //     value: Math.round(metric.value),
    //     metric_rating: metric.rating
    // });
}

// 收集核心Web指标
onLCP(sendToAnalytics);
onFID(sendToAnalytics);
onCLS(sendToAnalytics);
onFCP(sendToAnalytics);
onTTFB(sendToAnalytics);

// 自定义性能监控
export function measureComponentPerformance(componentName: string) {
    const startTime = performance.now();
    
    return () => {
        const endTime = performance.now();
        const duration = endTime - startTime;
        
        console.log(`${componentName} render time: ${duration}ms`);
        
        // 发送到分析服务
        sendToAnalytics({
            name: `component_${componentName}`,
            value: duration,
            rating: duration < 100 ? 'good' : duration < 300 ? 'needs-improvement' : 'poor'
        });
    };
}
```

### 9.2 组件性能监控

```vue
<!--
  组件性能监控
  @author erik.zhou
-->
<template>
    <div>
        <h1>{{ title }}</h1>
        <p>{{ content }}</p>
    </div>
</template>

<script setup lang="ts">
import { onMounted, onUpdated } from 'vue';

const props = defineProps<{
    title: string;
    content: string;
}>();

let mountTime = 0;
let updateTime = 0;

onMounted(() => {
    mountTime = performance.now();
    console.log('Component mounted at:', mountTime);
});

onUpdated(() => {
    updateTime = performance.now();
    console.log('Component updated at:', updateTime);
    console.log('Update duration:', updateTime - mountTime);
});
</script>
```

### 9.3 内存监控

```typescript
/**
 * 内存监控
 * @author erik.zhou
 */
export function monitorMemory() {
    if ('memory' in performance) {
        const memory = (performance as any).memory;
        
        const metrics = {
            // 已使用的JS堆大小
            usedJSHeapSize: memory.usedJSHeapSize,
            
            // JS堆大小限制
            jsHeapSizeLimit: memory.jsHeapSizeLimit,
            
            // 总的JS堆大小
            totalJSHeapSize: memory.totalJSHeapSize,
            
            // 使用率
            usagePercent: (memory.usedJSHeapSize / memory.jsHeapSizeLimit * 100).toFixed(2)
        };
        
        console.log('Memory metrics:', metrics);
        
        // 警告：内存使用率过高
        if (parseFloat(metrics.usagePercent) > 90) {
            console.warn('High memory usage detected!');
        }
        
        return metrics;
    }
    
    return null;
}

// 定期监控内存
setInterval(() => {
    monitorMemory();
}, 30000); // 每30秒检查一次
```

### 9.4 性能分析工具

```typescript
/**
 * 性能分析工具
 * @author erik.zhou
 */
export class PerformanceAnalyzer {
    private marks: Map<string, number> = new Map();
    private measures: Map<string, number> = new Map();
    
    // 标记时间点
    mark(name: string) {
        this.marks.set(name, performance.now());
        performance.mark(name);
    }
    
    // 测量时间段
    measure(name: string, startMark: string, endMark?: string) {
        const start = this.marks.get(startMark);
        const end = endMark ? this.marks.get(endMark) : performance.now();
        
        if (start && end) {
            const duration = end - start;
            this.measures.set(name, duration);
            
            console.log(`${name}: ${duration.toFixed(2)}ms`);
            
            return duration;
        }
        
        return 0;
    }
    
    // 获取所有测量结果
    getMetrics() {
        return {
            marks: Object.fromEntries(this.marks),
            measures: Object.fromEntries(this.measures)
        };
    }
    
    // 清除所有标记
    clear() {
        this.marks.clear();
        this.measures.clear();
        performance.clearMarks();
        performance.clearMeasures();
    }
}

// 使用示例
const analyzer = new PerformanceAnalyzer();

analyzer.mark('start');
// ... 执行操作
analyzer.mark('end');
analyzer.measure('operation', 'start', 'end');
```

---

## 第10章 最佳实践

### 10.1 性能优化清单

```typescript
/**
 * 性能优化清单
 * @author erik.zhou
 */

// 1. 组件优化
// ✅ 使用异步组件
// ✅ 使用KeepAlive缓存组件
// ✅ 使用v-once和v-memo
// ✅ 合理使用函数式组件

// 2. 响应式优化
// ✅ 使用shallowRef/shallowReactive
// ✅ 使用readonly避免不必要的响应式
// ✅ 使用computed缓存计算结果
// ✅ 及时清理watch和watchEffect

// 3. 渲染优化
// ✅ 合理使用v-show和v-if
// ✅ 使用稳定的key
// ✅ 事件委托和防抖节流
// ✅ 使用Teleport优化DOM结构

// 4. 列表优化
// ✅ 使用虚拟列表
// ✅ 分页或无限滚动
// ✅ 避免在v-for中使用v-if

// 5. 路由优化
// ✅ 路由懒加载
// ✅ 路由预加载
// ✅ 使用KeepAlive缓存路由

// 6. 打包优化
// ✅ 代码分割
// ✅ Tree Shaking
// ✅ 压缩和混淆
// ✅ 资源优化
```

### 10.2 常见性能陷阱

```vue
<!--
  常见性能陷阱
  @author erik.zhou
-->
<template>
    <div>
        <!-- ❌ 陷阱1：在模板中使用复杂计算 -->
        <div>{{ items.filter(i => i.active).length }}</div>
        
        <!-- ✅ 解决方案：使用computed -->
        <div>{{ activeItemsCount }}</div>
        
        <!-- ❌ 陷阱2：在v-for中使用v-if -->
        <div v-for="item in items" :key="item.id" v-if="item.active">
            {{ item.name }}
        </div>
        
        <!-- ✅ 解决方案：先过滤再渲染 -->
        <div v-for="item in activeItems" :key="item.id">
            {{ item.name }}
        </div>
        
        <!-- ❌ 陷阱3：使用index作为key -->
        <div v-for="(item, index) in items" :key="index">
            {{ item.name }}
        </div>
        
        <!-- ✅ 解决方案：使用唯一ID -->
        <div v-for="item in items" :key="item.id">
            {{ item.name }}
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

const items = ref([
    { id: 1, name: 'Item 1', active: true },
    { id: 2, name: 'Item 2', active: false },
    { id: 3, name: 'Item 3', active: true }
]);

// ✅ 使用computed缓存计算结果
const activeItemsCount = computed(() => {
    return items.value.filter(i => i.active).length;
});

const activeItems = computed(() => {
    return items.value.filter(i => i.active);
});
</script>
```

### 10.3 性能优化实战案例

```vue
<!--
  性能优化实战案例：大型表格
  @author erik.zhou
-->
<template>
    <div class="table-container">
        <!-- 虚拟滚动表格 -->
        <RecycleScroller
            class="scroller"
            :items="tableData"
            :item-size="50"
            key-field="id"
            v-slot="{ item }"
        >
            <div class="table-row">
                <div class="table-cell">{{ item.id }}</div>
                <div class="table-cell">{{ item.name }}</div>
                <div class="table-cell">{{ item.email }}</div>
                <div class="table-cell">
                    <button @click="handleEdit(item)">编辑</button>
                    <button @click="handleDelete(item.id)">删除</button>
                </div>
            </div>
        </RecycleScroller>
    </div>
</template>

<script setup lang="ts">
import { ref, shallowRef } from 'vue';
import { RecycleScroller } from 'vue-virtual-scroller';

// 使用shallowRef减少响应式开销
const tableData = shallowRef(
    Array.from({ length: 10000 }, (_, i) => ({
        id: i + 1,
        name: `User ${i + 1}`,
        email: `user${i + 1}@example.com`
    }))
);

// 事件处理使用防抖
import { useDebounceFn } from '@vueuse/core';

const handleEdit = useDebounceFn((item: any) => {
    console.log('Edit:', item);
}, 300);

const handleDelete = useDebounceFn((id: number) => {
    // 使用新数组替换，触发更新
    tableData.value = tableData.value.filter(item => item.id !== id);
}, 300);
</script>

<style scoped>
.table-container {
    height: 600px;
    border: 1px solid #ddd;
}

.scroller {
    height: 100%;
}

.table-row {
    display: flex;
    padding: 0.5rem;
    border-bottom: 1px solid #eee;
}

.table-cell {
    flex: 1;
    padding: 0 0.5rem;
}
</style>
```

### 10.4 性能优化工作流

```typescript
/**
 * 性能优化工作流
 * @author erik.zhou
 */

// 1. 性能分析
// - 使用Chrome DevTools Performance面板
// - 使用Vue DevTools分析组件性能
// - 使用Lighthouse评估整体性能

// 2. 识别瓶颈
// - 找出渲染时间长的组件
// - 识别频繁更新的响应式数据
// - 检查大型列表和复杂计算

// 3. 应用优化
// - 组件级优化（异步组件、KeepAlive）
// - 响应式优化（shallowRef、computed）
// - 渲染优化（虚拟列表、懒加载）

// 4. 验证效果
// - 重新测量性能指标
// - 对比优化前后的数据
// - 确保没有引入新问题

// 5. 持续监控
// - 设置性能监控
// - 定期检查性能指标
// - 及时发现和解决问题
```

---

## 总结

### 核心要点

1. **组件优化**
   - 使用异步组件减少初始加载
   - KeepAlive缓存组件状态
   - v-once和v-memo减少渲染

2. **响应式优化**
   - shallowRef/shallowReactive减少开销
   - computed缓存计算结果
   - 及时清理副作用

3. **渲染优化**
   - 合理使用v-show和v-if
   - 稳定的key提升Diff效率
   - 事件委托和防抖节流

4. **列表优化**
   - 虚拟列表处理大数据
   - 分页或无限滚动
   - 避免不必要的渲染

5. **打包优化**
   - 代码分割和懒加载
   - Tree Shaking移除无用代码
   - 资源压缩和优化

### 学习路径

1. **基础阶段**
   - 理解Vue性能优化原理
   - 掌握基本优化技巧
   - 学会使用性能分析工具

2. **进阶阶段**
   - 深入响应式系统优化
   - 掌握虚拟列表技术
   - 学会打包优化配置

3. **高级阶段**
   - 性能监控和分析
   - 大型应用优化实战
   - 自定义性能优化方案

### 推荐资源

- [Vue 3性能优化指南](https://vuejs.org/guide/best-practices/performance.html)
- [Web Vitals](https://web.dev/vitals/)
- [Chrome DevTools](https://developer.chrome.com/docs/devtools/)
- [Vue DevTools](https://devtools.vuejs.org/)

---

## 附录

### A. 性能优化工具清单

```typescript
/**
 * 性能优化工具清单
 * @author erik.zhou
 */

// 1. 分析工具
// - Chrome DevTools
// - Vue DevTools
// - Lighthouse
// - WebPageTest

// 2. 监控工具
// - web-vitals
// - performance.now()
// - PerformanceObserver

// 3. 优化库
// - vue-virtual-scroller
// - @vueuse/core
// - lodash-es

// 4. 构建工具
// - Vite
// - Rollup
// - esbuild
```

### B. 性能指标参考值

```typescript
/**
 * 性能指标参考值
 * @author erik.zhou
 */
interface PerformanceTargets {
    // 最大内容绘制
    lcp: {
        good: 2500;      // < 2.5秒
        needsWork: 4000; // 2.5-4秒
        poor: 4000;      // > 4秒
    };
    
    // 首次输入延迟
    fid: {
        good: 100;       // < 100毫秒
        needsWork: 300;  // 100-300毫秒
        poor: 300;       // > 300毫秒
    };
    
    // 累积布局偏移
    cls: {
        good: 0.1;       // < 0.1
        needsWork: 0.25; // 0.1-0.25
        poor: 0.25;      // > 0.25
    };
}
```

### C. 常见问题FAQ

**Q: 什么时候使用shallowRef？**
A: 当数据结构复杂但只需要响应根级别变化时使用。

**Q: 虚拟列表适用于什么场景？**
A: 适用于需要渲染大量数据（1000+条）的列表场景。

**Q: 如何选择v-show和v-if？**
A: 频繁切换用v-show，条件很少改变用v-if。

**Q: computed和method有什么区别？**
A: computed有缓存，依赖不变时不会重新计算；method每次都会执行。

**Q: 如何优化首屏加载时间？**
A: 使用代码分割、路由懒加载、资源压缩、CDN加速等。

---

**@author erik.zhou**  
**最后更新**: 2026-03-02  
**版本**: 1.0.0

