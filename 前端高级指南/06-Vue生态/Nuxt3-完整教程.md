# Nuxt 3 - 完整教程

## 课程简介

Nuxt 3是基于Vue 3的全栈框架，提供了服务端渲染(SSR)、静态站点生成(SSG)、混合渲染等多种渲染模式。本教程将深入讲解Nuxt 3的核心概念、使用方法和最佳实践。

## 学习目标

- 理解Nuxt 3的核心概念和架构
- 掌握多种渲染模式(SSR/SSG/ISR)
- 熟练使用文件路由系统
- 掌握数据获取方法
- 学会使用中间件和插件
- 掌握SEO优化技巧
- 理解部署和性能优化

## 目录

1. [Nuxt 3基础](#第1章-nuxt-3基础)
2. [文件路由系统](#第2章-文件路由系统)
3. [渲染模式](#第3章-渲染模式)
4. [数据获取](#第4章-数据获取)
5. [组件与布局](#第5章-组件与布局)
6. [中间件](#第6章-中间件)
7. [插件系统](#第7章-插件系统)
8. [状态管理](#第8章-状态管理)
9. [SEO优化](#第9章-seo优化)
10. [部署与优化](#第10章-部署与优化)

---

## 第1章 Nuxt 3基础

### 1.1 Nuxt 3简介

```typescript
/**
 * Nuxt 3核心特性
 * @author erik.zhou
 */

// Nuxt 3的主要特性：
// 1. 基于Vue 3和Vite
// 2. 服务端渲染(SSR)
// 3. 静态站点生成(SSG)
// 4. 混合渲染(Hybrid)
// 5. 自动导入
// 6. 文件路由
// 7. TypeScript支持
// 8. 零配置
```

### 1.2 创建项目

```bash
# 使用npx创建项目
npx nuxi@latest init my-nuxt-app

# 使用pnpm创建项目
pnpm dlx nuxi@latest init my-nuxt-app

# 进入项目目录
cd my-nuxt-app

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev
```

### 1.3 项目结构

```typescript
/**
 * Nuxt 3项目结构
 * @author erik.zhou
 */

// my-nuxt-app/
// ├── .nuxt/              # 构建输出目录
// ├── .output/            # 生产构建输出
// ├── assets/             # 静态资源（需要处理）
// ├── components/         # Vue组件
// ├── composables/        # 组合式函数
// ├── layouts/            # 布局组件
// ├── middleware/         # 路由中间件
// ├── pages/              # 页面组件（自动路由）
// ├── plugins/            # 插件
// ├── public/             # 静态资源（不处理）
// ├── server/             # 服务端代码
// │   ├── api/           # API路由
// │   ├── routes/        # 服务端路由
// │   └── middleware/    # 服务端中间件
// ├── utils/              # 工具函数
// ├── app.vue             # 根组件
// ├── nuxt.config.ts      # Nuxt配置
// └── package.json
```

### 1.4 基础配置

```typescript
/**
 * Nuxt配置文件
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    // 开发工具
    devtools: { enabled: true },
    
    // TypeScript配置
    typescript: {
        strict: true,
        typeCheck: true
    },
    
    // 应用配置
    app: {
        head: {
            title: 'My Nuxt App',
            meta: [
                { charset: 'utf-8' },
                { name: 'viewport', content: 'width=device-width, initial-scale=1' },
                { name: 'description', content: 'My amazing Nuxt 3 app' }
            ],
            link: [
                { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
            ]
        }
    },
    
    // CSS配置
    css: ['~/assets/css/main.css'],
    
    // 模块
    modules: [],
    
    // 运行时配置
    runtimeConfig: {
        // 服务端可用
        apiSecret: process.env.API_SECRET,
        
        // 客户端和服务端都可用
        public: {
            apiBase: process.env.API_BASE_URL || '/api'
        }
    }
});
```


### 1.5 根组件

```vue
<!--
  Nuxt 3根组件
  @author erik.zhou
-->
<template>
    <div>
        <NuxtLayout>
            <NuxtPage />
        </NuxtLayout>
    </div>
</template>

<script setup lang="ts">
// 全局配置
useHead({
    titleTemplate: '%s - My Nuxt App',
    meta: [
        { name: 'description', content: 'My amazing Nuxt 3 app' }
    ]
});

// 全局错误处理
onErrorCaptured((error) => {
    console.error('Global error:', error);
    return false;
});
</script>

<style>
body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
</style>
```

---

## 第2章 文件路由系统

### 2.1 基础路由

```typescript
/**
 * 文件路由系统示例
 * @author erik.zhou
 */

// pages/index.vue -> /
// pages/about.vue -> /about
// pages/posts/index.vue -> /posts
// pages/posts/[id].vue -> /posts/:id
// pages/posts/[...slug].vue -> /posts/:slug(.*)
```

```vue
<!--
  基础页面组件
  @author erik.zhou
-->
<!-- pages/index.vue -->
<template>
    <div>
        <h1>首页</h1>
        <NuxtLink to="/about">关于我们</NuxtLink>
    </div>
</template>

<script setup lang="ts">
definePageMeta({
    title: '首页',
    description: '这是首页'
});
</script>
```

### 2.2 动态路由

```vue
<!--
  动态路由示例
  @author erik.zhou
-->
<!-- pages/posts/[id].vue -->
<template>
    <div>
        <h1>文章详情</h1>
        <p>文章ID: {{ route.params.id }}</p>
        <div v-if="post">
            <h2>{{ post.title }}</h2>
            <p>{{ post.content }}</p>
        </div>
    </div>
</template>

<script setup lang="ts">
const route = useRoute();
const postId = computed(() => route.params.id);

// 获取文章数据
const { data: post } = await useFetch(`/api/posts/${postId.value}`);
</script>
```

### 2.3 嵌套路由

```typescript
/**
 * 嵌套路由结构
 * @author erik.zhou
 */

// pages/
// ├── users/
// │   ├── index.vue          -> /users
// │   ├── [id].vue           -> /users/:id
// │   └── [id]/
// │       ├── profile.vue    -> /users/:id/profile
// │       └── settings.vue   -> /users/:id/settings
```

```vue
<!--
  父级路由组件
  @author erik.zhou
-->
<!-- pages/users/[id].vue -->
<template>
    <div>
        <h1>用户: {{ userId }}</h1>
        <nav>
            <NuxtLink :to="`/users/${userId}/profile`">个人资料</NuxtLink>
            <NuxtLink :to="`/users/${userId}/settings`">设置</NuxtLink>
        </nav>
        <!-- 子路由出口 -->
        <NuxtPage />
    </div>
</template>

<script setup lang="ts">
const route = useRoute();
const userId = computed(() => route.params.id);
</script>
```

### 2.4 路由导航

```vue
<!--
  路由导航示例
  @author erik.zhou
-->
<template>
    <div>
        <!-- 声明式导航 -->
        <NuxtLink to="/about">关于</NuxtLink>
        <NuxtLink :to="{ name: 'posts-id', params: { id: 1 } }">
            文章1
        </NuxtLink>
        
        <!-- 编程式导航 -->
        <button @click="goToAbout">跳转到关于</button>
        <button @click="goBack">返回</button>
    </div>
</template>

<script setup lang="ts">
const router = useRouter();

function goToAbout() {
    router.push('/about');
}

function goBack() {
    router.back();
}

// 路由守卫
onBeforeRouteLeave((to, from) => {
    const answer = window.confirm('确定要离开吗？');
    if (!answer) {
        return false;
    }
});
</script>
```

### 2.5 路由元信息

```vue
<!--
  路由元信息示例
  @author erik.zhou
-->
<template>
    <div>
        <h1>需要认证的页面</h1>
    </div>
</template>

<script setup lang="ts">
definePageMeta({
    // 页面标题
    title: '个人中心',
    
    // 需要认证
    middleware: 'auth',
    
    // 自定义元信息
    requiresAuth: true,
    roles: ['user', 'admin'],
    
    // 布局
    layout: 'dashboard',
    
    // 过渡动画
    pageTransition: {
        name: 'fade',
        mode: 'out-in'
    }
});
</script>
```

---

## 第3章 渲染模式

### 3.1 服务端渲染(SSR)

```typescript
/**
 * SSR配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    // SSR模式（默认）
    ssr: true,
    
    // 路由规则
    routeRules: {
        // 所有页面使用SSR
        '/**': { ssr: true }
    }
});
```

```vue
<!--
  SSR页面示例
  @author erik.zhou
-->
<template>
    <div>
        <h1>服务端渲染页面</h1>
        <p>当前时间: {{ serverTime }}</p>
        <ul>
            <li v-for="post in posts" :key="post.id">
                {{ post.title }}
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
// 服务端获取数据
const { data: posts } = await useFetch('/api/posts');

// 服务端时间
const serverTime = new Date().toISOString();

// SEO优化
useHead({
    title: '文章列表',
    meta: [
        { name: 'description', content: '最新文章列表' }
    ]
});
</script>
```

### 3.2 静态站点生成(SSG)

```typescript
/**
 * SSG配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    // 启用SSG
    ssr: true,
    
    // 路由规则
    routeRules: {
        // 预渲染首页
        '/': { prerender: true },
        
        // 预渲染所有文章页面
        '/posts/**': { prerender: true }
    },
    
    // 生成配置
    nitro: {
        prerender: {
            crawlLinks: true,
            routes: ['/sitemap.xml']
        }
    }
});
```

```typescript
/**
 * 动态路由预渲染
 * @author erik.zhou
 */
// server/routes/sitemap.xml.ts
export default defineEventHandler(async (event) => {
    const posts = await $fetch('/api/posts');
    
    const urls = posts.map((post: any) => ({
        loc: `/posts/${post.id}`,
        lastmod: post.updatedAt
    }));
    
    return {
        urls: [
            { loc: '/', lastmod: new Date().toISOString() },
            { loc: '/about', lastmod: new Date().toISOString() },
            ...urls
        ]
    };
});
```

### 3.3 客户端渲染(CSR)

```typescript
/**
 * CSR配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    // 禁用SSR
    ssr: false,
    
    // 或者针对特定路由
    routeRules: {
        '/dashboard/**': { ssr: false }
    }
});
```

```vue
<!--
  CSR页面示例
  @author erik.zhou
-->
<template>
    <div>
        <h1>客户端渲染页面</h1>
        <ClientOnly>
            <Dashboard />
            <template #fallback>
                <div>加载中...</div>
            </template>
        </ClientOnly>
    </div>
</template>

<script setup lang="ts">
definePageMeta({
    ssr: false
});
</script>
```

### 3.4 混合渲染(Hybrid)

```typescript
/**
 * 混合渲染配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    routeRules: {
        // 首页：SSR
        '/': { ssr: true },
        
        // 文章列表：ISR（增量静态再生）
        '/posts': { swr: 3600 }, // 1小时缓存
        
        // 文章详情：ISR
        '/posts/**': { swr: true },
        
        // 用户中心：CSR
        '/dashboard/**': { ssr: false },
        
        // API路由：缓存
        '/api/**': { cache: { maxAge: 60 } }
    }
});
```

### 3.5 增量静态再生(ISR)

```typescript
/**
 * ISR配置示例
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    routeRules: {
        // 基础ISR：使用stale-while-revalidate
        '/posts/**': {
            swr: 3600 // 1小时后重新验证
        },
        
        // 高级ISR配置
        '/products/**': {
            swr: true,
            cache: {
                maxAge: 3600,
                staleMaxAge: 7200,
                swr: true
            }
        }
    }
});
```

```vue
<!--
  ISR页面示例
  @author erik.zhou
-->
<template>
    <div>
        <h1>{{ post.title }}</h1>
        <p>{{ post.content }}</p>
        <p>生成时间: {{ generatedAt }}</p>
    </div>
</template>

<script setup lang="ts">
const route = useRoute();
const postId = route.params.id;

// 数据会被缓存，定期重新生成
const { data: post } = await useFetch(`/api/posts/${postId}`);

const generatedAt = new Date().toISOString();
</script>
```

---

## 第4章 数据获取

### 4.1 useFetch

```vue
<!--
  useFetch基础用法
  @author erik.zhou
-->
<template>
    <div>
        <div v-if="pending">加载中...</div>
        <div v-else-if="error">错误: {{ error.message }}</div>
        <div v-else>
            <h1>{{ data.title }}</h1>
            <button @click="refresh">刷新</button>
        </div>
    </div>
</template>

<script setup lang="ts">
// 基础用法
const { data, pending, error, refresh } = await useFetch('/api/posts/1');

// 带选项
const { data: posts } = await useFetch('/api/posts', {
    method: 'GET',
    query: { page: 1, limit: 10 },
    headers: {
        'Authorization': 'Bearer token'
    },
    // 转换响应数据
    transform: (data: any) => {
        return data.map((post: any) => ({
            id: post.id,
            title: post.title.toUpperCase()
        }));
    },
    // 选择需要的数据
    pick: ['id', 'title'],
    // 监听依赖变化
    watch: [page]
});
</script>
```

### 4.2 useAsyncData

```vue
<!--
  useAsyncData用法
  @author erik.zhou
-->
<template>
    <div>
        <div v-if="pending">加载中...</div>
        <div v-else>
            <ul>
                <li v-for="post in data" :key="post.id">
                    {{ post.title }}
                </li>
            </ul>
        </div>
    </div>
</template>

<script setup lang="ts">
// 自定义异步函数
const { data, pending, error, refresh } = await useAsyncData(
    'posts', // 唯一key
    async () => {
        const response = await $fetch('/api/posts');
        return response.data;
    },
    {
        // 服务端和客户端都执行
        server: true,
        lazy: false,
        
        // 默认值
        default: () => [],
        
        // 转换数据
        transform: (data) => {
            return data.filter((post: any) => post.published);
        }
    }
);
</script>
```

### 4.3 useLazyFetch和useLazyAsyncData

```vue
<!--
  懒加载数据示例
  @author erik.zhou
-->
<template>
    <div>
        <!-- 不会阻塞导航 -->
        <div v-if="pending">加载中...</div>
        <div v-else-if="data">
            <h1>{{ data.title }}</h1>
        </div>
    </div>
</template>

<script setup lang="ts">
// useLazyFetch：不阻塞导航
const { data, pending } = useLazyFetch('/api/posts/1');

// useLazyAsyncData：不阻塞导航
const { data: posts, pending: postsPending } = useLazyAsyncData(
    'posts',
    () => $fetch('/api/posts')
);
</script>
```

### 4.4 $fetch

```typescript
/**
 * $fetch使用示例
 * @author erik.zhou
 */

// 基础用法
const data = await $fetch('/api/posts');

// POST请求
const newPost = await $fetch('/api/posts', {
    method: 'POST',
    body: {
        title: 'New Post',
        content: 'Content'
    }
});

// 错误处理
try {
    const data = await $fetch('/api/posts/1');
} catch (error: any) {
    if (error.response?.status === 404) {
        console.error('Post not found');
    }
}

// 在组件中使用
async function deletePost(id: number) {
    try {
        await $fetch(`/api/posts/${id}`, {
            method: 'DELETE'
        });
        // 刷新数据
        await refresh();
    } catch (error) {
        console.error('Failed to delete post:', error);
    }
}
```

### 4.5 数据缓存和刷新

```vue
<!--
  数据缓存和刷新示例
  @author erik.zhou
-->
<template>
    <div>
        <button @click="refresh">刷新</button>
        <button @click="clear">清除缓存</button>
        
        <ul>
            <li v-for="post in data" :key="post.id">
                {{ post.title }}
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
const page = ref(1);

// 带缓存的数据获取
const { data, pending, refresh, clear } = await useFetch('/api/posts', {
    key: 'posts',
    query: { page },
    // 缓存配置
    getCachedData(key) {
        return useNuxtData(key).data.value;
    }
});

// 监听page变化自动刷新
watch(page, () => {
    refresh();
});

// 清除特定缓存
function clearCache() {
    clearNuxtData('posts');
}

// 清除所有缓存
function clearAllCache() {
    clearNuxtData();
}
</script>
```


---

## 第5章 组件与布局

### 5.1 自动导入组件

```vue
<!--
  自动导入组件示例
  @author erik.zhou
-->
<!-- components/AppHeader.vue -->
<template>
    <header>
        <h1>My App</h1>
        <nav>
            <NuxtLink to="/">首页</NuxtLink>
            <NuxtLink to="/about">关于</NuxtLink>
        </nav>
    </header>
</template>

<!-- 在页面中使用（无需导入） -->
<template>
    <div>
        <AppHeader />
        <main>
            <h1>页面内容</h1>
        </main>
    </div>
</template>
```

### 5.2 布局系统

```vue
<!--
  默认布局
  @author erik.zhou
-->
<!-- layouts/default.vue -->
<template>
    <div class="layout">
        <AppHeader />
        <main>
            <slot />
        </main>
        <AppFooter />
    </div>
</template>

<style scoped>
.layout {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}

main {
    flex: 1;
    padding: 2rem;
}
</style>
```

```vue
<!--
  自定义布局
  @author erik.zhou
-->
<!-- layouts/dashboard.vue -->
<template>
    <div class="dashboard-layout">
        <aside class="sidebar">
            <DashboardNav />
        </aside>
        <div class="content">
            <slot />
        </div>
    </div>
</template>

<style scoped>
.dashboard-layout {
    display: grid;
    grid-template-columns: 250px 1fr;
    min-height: 100vh;
}

.sidebar {
    background: #f5f5f5;
    padding: 1rem;
}

.content {
    padding: 2rem;
}
</style>
```

```vue
<!--
  使用自定义布局
  @author erik.zhou
-->
<template>
    <div>
        <h1>仪表盘</h1>
    </div>
</template>

<script setup lang="ts">
definePageMeta({
    layout: 'dashboard'
});
</script>
```

### 5.3 动态布局

```vue
<!--
  动态布局示例
  @author erik.zhou
-->
<template>
    <div>
        <h1>动态布局页面</h1>
        <button @click="toggleLayout">切换布局</button>
    </div>
</template>

<script setup lang="ts">
const layout = ref('default');

function toggleLayout() {
    layout.value = layout.value === 'default' ? 'dashboard' : 'default';
}

// 动态设置布局
setPageLayout(layout.value);

watch(layout, (newLayout) => {
    setPageLayout(newLayout);
});
</script>
```

### 5.4 错误页面

```vue
<!--
  错误页面
  @author erik.zhou
-->
<!-- error.vue -->
<template>
    <div class="error-page">
        <h1>{{ error.statusCode }}</h1>
        <p>{{ error.message }}</p>
        <button @click="handleError">返回首页</button>
    </div>
</template>

<script setup lang="ts">
const props = defineProps({
    error: {
        type: Object,
        required: true
    }
});

function handleError() {
    clearError({ redirect: '/' });
}
</script>

<style scoped>
.error-page {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    text-align: center;
}
</style>
```

---

## 第6章 中间件

### 6.1 路由中间件

```typescript
/**
 * 认证中间件
 * @author erik.zhou
 */
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
    const user = useState('user');
    
    // 检查用户是否登录
    if (!user.value) {
        return navigateTo('/login');
    }
});
```

```typescript
/**
 * 权限中间件
 * @author erik.zhou
 */
// middleware/admin.ts
export default defineNuxtRouteMiddleware((to, from) => {
    const user = useState('user');
    
    // 检查是否是管理员
    if (!user.value || user.value.role !== 'admin') {
        return abortNavigation({
            statusCode: 403,
            message: '无权访问'
        });
    }
});
```

### 6.2 全局中间件

```typescript
/**
 * 全局中间件
 * @author erik.zhou
 */
// middleware/analytics.global.ts
export default defineNuxtRouteMiddleware((to, from) => {
    // 页面访问统计
    if (process.client) {
        console.log('Page view:', to.path);
        // 发送统计数据
        // analytics.track('pageview', { path: to.path });
    }
});
```

### 6.3 命名中间件

```vue
<!--
  使用命名中间件
  @author erik.zhou
-->
<template>
    <div>
        <h1>管理员页面</h1>
    </div>
</template>

<script setup lang="ts">
definePageMeta({
    middleware: ['auth', 'admin']
});
</script>
```

### 6.4 内联中间件

```vue
<!--
  内联中间件示例
  @author erik.zhou
-->
<template>
    <div>
        <h1>受保护的页面</h1>
    </div>
</template>

<script setup lang="ts">
definePageMeta({
    middleware: [
        function (to, from) {
            const user = useState('user');
            if (!user.value) {
                return navigateTo('/login');
            }
        }
    ]
});
</script>
```

---

## 第7章 插件系统

### 7.1 创建插件

```typescript
/**
 * API插件
 * @author erik.zhou
 */
// plugins/api.ts
export default defineNuxtPlugin((nuxtApp) => {
    const config = useRuntimeConfig();
    
    const api = $fetch.create({
        baseURL: config.public.apiBase,
        onRequest({ request, options }) {
            // 添加认证token
            const token = useCookie('token');
            if (token.value) {
                options.headers = {
                    ...options.headers,
                    Authorization: `Bearer ${token.value}`
                };
            }
        },
        onResponseError({ response }) {
            // 统一错误处理
            if (response.status === 401) {
                navigateTo('/login');
            }
        }
    });
    
    return {
        provide: {
            api
        }
    };
});
```

### 7.2 使用插件

```vue
<!--
  使用插件示例
  @author erik.zhou
-->
<template>
    <div>
        <ul>
            <li v-for="post in posts" :key="post.id">
                {{ post.title }}
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
const { $api } = useNuxtApp();

// 使用插件提供的API
const posts = ref([]);

onMounted(async () => {
    posts.value = await $api('/posts');
});
</script>
```

### 7.3 Vue插件集成

```typescript
/**
 * 集成Vue插件
 * @author erik.zhou
 */
// plugins/vue-plugins.ts
import VueGtag from 'vue-gtag';

export default defineNuxtPlugin((nuxtApp) => {
    nuxtApp.vueApp.use(VueGtag, {
        config: { id: 'GA_MEASUREMENT_ID' }
    });
});
```

### 7.4 客户端插件

```typescript
/**
 * 客户端插件
 * @author erik.zhou
 */
// plugins/client-only.client.ts
export default defineNuxtPlugin(() => {
    // 只在客户端执行
    if (process.client) {
        console.log('Client-side plugin loaded');
        
        // 初始化客户端库
        // initAnalytics();
    }
});
```

---

## 第8章 状态管理

### 8.1 useState

```typescript
/**
 * useState基础用法
 * @author erik.zhou
 */
// composables/useUser.ts
export const useUser = () => {
    const user = useState('user', () => null);
    
    async function fetchUser() {
        const data = await $fetch('/api/user');
        user.value = data;
    }
    
    function logout() {
        user.value = null;
    }
    
    return {
        user,
        fetchUser,
        logout
    };
};
```

```vue
<!--
  使用useState
  @author erik.zhou
-->
<template>
    <div>
        <div v-if="user">
            <p>欢迎, {{ user.name }}</p>
            <button @click="logout">退出</button>
        </div>
        <div v-else>
            <button @click="fetchUser">登录</button>
        </div>
    </div>
</template>

<script setup lang="ts">
const { user, fetchUser, logout } = useUser();
</script>
```

### 8.2 集成Pinia

```typescript
/**
 * Pinia集成
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    modules: ['@pinia/nuxt']
});

// stores/user.ts
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
    state: () => ({
        user: null as any,
        token: null as string | null
    }),
    
    actions: {
        async login(credentials: any) {
            const data = await $fetch('/api/login', {
                method: 'POST',
                body: credentials
            });
            this.user = data.user;
            this.token = data.token;
        },
        
        logout() {
            this.user = null;
            this.token = null;
        }
    }
});
```

### 8.3 持久化状态

```typescript
/**
 * 状态持久化
 * @author erik.zhou
 */
// composables/usePersistedState.ts
export const usePersistedState = <T>(key: string, initialValue: T) => {
    const state = useState(key, () => initialValue);
    
    // 从localStorage恢复
    if (process.client) {
        const saved = localStorage.getItem(key);
        if (saved) {
            try {
                state.value = JSON.parse(saved);
            } catch (error) {
                console.error('Failed to parse saved state:', error);
            }
        }
    }
    
    // 监听变化并保存
    watch(state, (newValue) => {
        if (process.client) {
            localStorage.setItem(key, JSON.stringify(newValue));
        }
    }, { deep: true });
    
    return state;
};
```

---

## 第9章 SEO优化

### 9.1 Meta标签

```vue
<!--
  Meta标签优化
  @author erik.zhou
-->
<template>
    <div>
        <h1>{{ post.title }}</h1>
        <p>{{ post.content }}</p>
    </div>
</template>

<script setup lang="ts">
const route = useRoute();
const { data: post } = await useFetch(`/api/posts/${route.params.id}`);

// 设置页面meta
useHead({
    title: post.value.title,
    meta: [
        { name: 'description', content: post.value.excerpt },
        { name: 'keywords', content: post.value.tags.join(', ') },
        
        // Open Graph
        { property: 'og:title', content: post.value.title },
        { property: 'og:description', content: post.value.excerpt },
        { property: 'og:image', content: post.value.coverImage },
        { property: 'og:type', content: 'article' },
        
        // Twitter Card
        { name: 'twitter:card', content: 'summary_large_image' },
        { name: 'twitter:title', content: post.value.title },
        { name: 'twitter:description', content: post.value.excerpt },
        { name: 'twitter:image', content: post.value.coverImage }
    ],
    link: [
        { rel: 'canonical', href: `https://example.com/posts/${route.params.id}` }
    ]
});
</script>
```

### 9.2 结构化数据

```vue
<!--
  结构化数据示例
  @author erik.zhou
-->
<template>
    <article>
        <h1>{{ post.title }}</h1>
        <p>{{ post.content }}</p>
    </article>
</template>

<script setup lang="ts">
const route = useRoute();
const { data: post } = await useFetch(`/api/posts/${route.params.id}`);

// 添加结构化数据
useHead({
    script: [
        {
            type: 'application/ld+json',
            children: JSON.stringify({
                '@context': 'https://schema.org',
                '@type': 'Article',
                headline: post.value.title,
                description: post.value.excerpt,
                image: post.value.coverImage,
                author: {
                    '@type': 'Person',
                    name: post.value.author.name
                },
                datePublished: post.value.publishedAt,
                dateModified: post.value.updatedAt
            })
        }
    ]
});
</script>
```

### 9.3 Sitemap生成

```typescript
/**
 * Sitemap生成
 * @author erik.zhou
 */
// server/routes/sitemap.xml.ts
export default defineEventHandler(async (event) => {
    const posts = await $fetch('/api/posts');
    
    const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <url>
        <loc>https://example.com/</loc>
        <lastmod>${new Date().toISOString()}</lastmod>
        <priority>1.0</priority>
    </url>
    ${posts.map((post: any) => `
    <url>
        <loc>https://example.com/posts/${post.id}</loc>
        <lastmod>${post.updatedAt}</lastmod>
        <priority>0.8</priority>
    </url>
    `).join('')}
</urlset>`;
    
    event.node.res.setHeader('Content-Type', 'application/xml');
    return sitemap;
});
```

### 9.4 Robots.txt

```typescript
/**
 * Robots.txt生成
 * @author erik.zhou
 */
// server/routes/robots.txt.ts
export default defineEventHandler((event) => {
    const robots = `User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/

Sitemap: https://example.com/sitemap.xml`;
    
    event.node.res.setHeader('Content-Type', 'text/plain');
    return robots;
});
```


---

## 第10章 部署与优化

### 10.1 构建配置

```typescript
/**
 * 生产构建配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    // 生产环境优化
    nitro: {
        preset: 'node-server', // 或 'vercel', 'netlify', 'cloudflare'
        
        // 压缩
        compressPublicAssets: true,
        
        // 预渲染
        prerender: {
            crawlLinks: true,
            routes: ['/sitemap.xml', '/robots.txt']
        }
    },
    
    // 构建优化
    vite: {
        build: {
            // 代码分割
            rollupOptions: {
                output: {
                    manualChunks: {
                        vendor: ['vue', 'vue-router'],
                        utils: ['lodash-es', 'dayjs']
                    }
                }
            }
        }
    }
});
```

### 10.2 性能优化

```vue
<!--
  性能优化示例
  @author erik.zhou
-->
<template>
    <div>
        <!-- 图片懒加载 -->
        <NuxtImg
            src="/images/hero.jpg"
            loading="lazy"
            width="800"
            height="600"
            alt="Hero image"
        />
        
        <!-- 组件懒加载 -->
        <LazyHeavyComponent v-if="showComponent" />
        
        <!-- 客户端组件 -->
        <ClientOnly>
            <ExpensiveChart :data="chartData" />
        </ClientOnly>
    </div>
</template>

<script setup lang="ts">
// 预加载关键资源
useHead({
    link: [
        { rel: 'preload', as: 'image', href: '/images/hero.jpg' },
        { rel: 'prefetch', href: '/api/posts' }
    ]
});

// 懒加载数据
const showComponent = ref(false);
onMounted(() => {
    setTimeout(() => {
        showComponent.value = true;
    }, 1000);
});
</script>
```

### 10.3 缓存策略

```typescript
/**
 * 缓存策略配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    routeRules: {
        // 静态资源：长期缓存
        '/images/**': {
            headers: {
                'Cache-Control': 'public, max-age=31536000, immutable'
            }
        },
        
        // API：短期缓存
        '/api/**': {
            cache: {
                maxAge: 60,
                staleMaxAge: 120
            }
        },
        
        // 页面：ISR
        '/posts/**': {
            swr: 3600,
            cache: {
                maxAge: 3600,
                staleMaxAge: 7200
            }
        }
    }
});
```

### 10.4 部署到Vercel

```typescript
/**
 * Vercel部署配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    nitro: {
        preset: 'vercel'
    }
});
```

```json
// vercel.json
{
    "buildCommand": "npm run build",
    "devCommand": "npm run dev",
    "installCommand": "npm install",
    "framework": "nuxtjs",
    "outputDirectory": ".output/public"
}
```

### 10.5 部署到Netlify

```typescript
/**
 * Netlify部署配置
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    nitro: {
        preset: 'netlify'
    }
});
```

```toml
# netlify.toml
[build]
    command = "npm run build"
    publish = ".output/public"
    functions = ".output/server"

[[redirects]]
    from = "/*"
    to = "/.netlify/functions/server"
    status = 200
```

### 10.6 Docker部署

```dockerfile
# Dockerfile
# @author erik.zhou
FROM node:18-alpine

WORKDIR /app

# 复制package文件
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# 暴露端口
EXPOSE 3000

# 启动应用
CMD ["node", ".output/server/index.mjs"]
```

```yaml
# docker-compose.yml
# @author erik.zhou
version: '3.8'

services:
    nuxt-app:
        build: .
        ports:
            - "3000:3000"
        environment:
            - NODE_ENV=production
            - API_BASE_URL=https://api.example.com
        restart: unless-stopped
```

### 10.7 性能监控

```typescript
/**
 * 性能监控配置
 * @author erik.zhou
 */
// plugins/performance.client.ts
export default defineNuxtPlugin(() => {
    if (process.client) {
        // Web Vitals监控
        const observer = new PerformanceObserver((list) => {
            for (const entry of list.getEntries()) {
                console.log('Performance entry:', entry);
                
                // 发送到分析服务
                // analytics.track('performance', {
                //     name: entry.name,
                //     value: entry.value
                // });
            }
        });
        
        observer.observe({ entryTypes: ['navigation', 'paint', 'largest-contentful-paint'] });
    }
});
```

---

## 总结

### 核心要点

1. **渲染模式**
   - SSR适合SEO和首屏性能
   - SSG适合静态内容
   - ISR结合两者优势
   - CSR适合交互密集应用

2. **数据获取**
   - useFetch用于API调用
   - useAsyncData用于自定义逻辑
   - 合理使用缓存和懒加载

3. **路由系统**
   - 文件路由自动生成
   - 支持动态和嵌套路由
   - 中间件控制访问权限

4. **SEO优化**
   - 完善的meta标签
   - 结构化数据
   - Sitemap和Robots.txt

5. **性能优化**
   - 代码分割
   - 图片优化
   - 缓存策略
   - 懒加载

### 学习路径

1. **基础阶段**
   - 理解Nuxt 3核心概念
   - 掌握文件路由系统
   - 学会数据获取方法

2. **进阶阶段**
   - 掌握多种渲染模式
   - 使用中间件和插件
   - SEO优化技巧

3. **高级阶段**
   - 性能优化实战
   - 部署和运维
   - 大型应用架构

### 推荐资源

- [Nuxt 3官方文档](https://nuxt.com/)
- [Nuxt GitHub](https://github.com/nuxt/nuxt)
- [Nuxt Examples](https://nuxt.com/docs/examples)
- [Nuxt Modules](https://nuxt.com/modules)

---

## 附录

### A. 常用API速查

```typescript
/**
 * Nuxt 3常用API速查
 * @author erik.zhou
 */

// 数据获取
const { data, pending, error, refresh } = await useFetch('/api/data');
const { data } = await useAsyncData('key', () => $fetch('/api/data'));

// 路由
const route = useRoute();
const router = useRouter();
router.push('/path');

// 状态管理
const state = useState('key', () => initialValue);

// Head管理
useHead({ title: 'Page Title' });
useSeoMeta({ description: 'Page description' });

// 运行时配置
const config = useRuntimeConfig();

// Cookie
const cookie = useCookie('name');

// 错误处理
throw createError({ statusCode: 404, message: 'Not found' });
clearError({ redirect: '/' });

// 导航
await navigateTo('/path');
await navigateTo({ name: 'route-name' });
```

### B. 配置模板

```typescript
/**
 * Nuxt配置模板
 * @author erik.zhou
 */
// nuxt.config.ts
export default defineNuxtConfig({
    // 应用配置
    app: {
        head: {
            title: 'My App',
            meta: [
                { charset: 'utf-8' },
                { name: 'viewport', content: 'width=device-width, initial-scale=1' }
            ]
        }
    },
    
    // 模块
    modules: [
        '@nuxtjs/tailwindcss',
        '@pinia/nuxt'
    ],
    
    // 运行时配置
    runtimeConfig: {
        apiSecret: '',
        public: {
            apiBase: ''
        }
    },
    
    // 路由规则
    routeRules: {
        '/': { prerender: true },
        '/api/**': { cors: true }
    },
    
    // Nitro配置
    nitro: {
        preset: 'node-server'
    }
});
```

### C. 常见问题FAQ

**Q: Nuxt 3和Nuxt 2有什么区别？**
A: Nuxt 3基于Vue 3，使用Vite构建，性能更好，API更简洁，支持TypeScript。

**Q: 如何选择渲染模式？**
A: 根据需求选择：SEO重要用SSR/SSG，交互密集用CSR，混合场景用ISR。

**Q: 如何处理环境变量？**
A: 使用runtimeConfig，服务端变量放在根级别，客户端变量放在public中。

**Q: 如何优化首屏加载？**
A: 使用SSR/SSG、代码分割、图片优化、预加载关键资源。

**Q: 如何调试SSR问题？**
A: 检查服务端日志，使用process.server/client判断环境，避免使用浏览器API。

---

**@author erik.zhou**  
**最后更新**: 2026-03-02  
**版本**: 1.0.0

