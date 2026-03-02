# Vue Router - 完整教程

## 课程信息
- **课程名称**: Vue Router完整教程
- **难度级别**: 中级
- **预计学时**: 5小时
- **核心内容**: 路由配置、导航守卫、动态路由、路由懒加载、路由元信息
- **@author**: erik.zhou

---

## 目录
1. [路由基础](#1-路由基础)
2. [路由配置](#2-路由配置)
3. [动态路由](#3-动态路由)
4. [嵌套路由](#4-嵌套路由)
5. [导航守卫](#5-导航守卫)
6. [路由懒加载](#6-路由懒加载)
7. [路由元信息](#7-路由元信息)
8. [编程式导航](#8-编程式导航)
9. [路由传参](#9-路由传参)
10. [高级特性](#10-高级特性)

---

## 1. 路由基础

### 1.1 安装和基本配置

```bash
# 安装Vue Router
npm install vue-router@4
```

```javascript
// router/index.js
/**
 * Vue Router基本配置
 * @author erik.zhou
 */
import { createRouter, createWebHistory } from 'vue-router';
import Home from '@/views/Home.vue';
import About from '@/views/About.vue';

const routes = [
    {
        path: '/',
        name: 'Home',
        component: Home
    },
    {
        path: '/about',
        name: 'About',
        component: About
    }
];

const router = createRouter({
    history: createWebHistory(import.meta.env.BASE_URL),
    routes
});

export default router;
```

```javascript
// main.js
/**
 * 应用入口配置
 * @author erik.zhou
 */
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';

const app = createApp(App);

app.use(router);
app.mount('#app');
```

### 1.2 基本使用

```vue
<!-- App.vue -->
<!--
  路由基本使用
  @author erik.zhou
-->
<template>
    <div id="app">
        <nav>
            <!-- 使用router-link进行导航 -->
            <router-link to="/">首页</router-link>
            <router-link to="/about">关于</router-link>
            
            <!-- 使用name导航 -->
            <router-link :to="{ name: 'Home' }">首页</router-link>
            
            <!-- 自定义激活类名 -->
            <router-link
                to="/about"
                active-class="active"
                exact-active-class="exact-active"
            >
                关于
            </router-link>
        </nav>
        
        <!-- 路由出口 -->
        <router-view />
        
        <!-- 命名视图 -->
        <router-view name="sidebar" />
    </div>
</template>

<style>
.active {
    color: #42b983;
}

.exact-active {
    font-weight: bold;
}
</style>
```

### 1.3 History模式

```javascript
// router/index.js
/**
 * 不同的History模式
 * @author erik.zhou
 */
import {
    createRouter,
    createWebHistory,      // HTML5 History模式
    createWebHashHistory,  // Hash模式
    createMemoryHistory    // Memory模式（SSR）
} from 'vue-router';

// 1. HTML5 History模式（推荐）
const router = createRouter({
    history: createWebHistory(import.meta.env.BASE_URL),
    routes
});

// 2. Hash模式
const routerHash = createRouter({
    history: createWebHashHistory(),
    routes
});

// 3. Memory模式（用于SSR）
const routerMemory = createRouter({
    history: createMemoryHistory(),
    routes
});

export default router;
```

---

## 2. 路由配置

### 2.1 完整路由配置

```javascript
// router/index.js
/**
 * 完整路由配置示例
 * @author erik.zhou
 */
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
    {
        path: '/',
        name: 'Home',
        component: () => import('@/views/Home.vue'),
        meta: {
            title: '首页',
            requiresAuth: false
        }
    },
    {
        path: '/about',
        name: 'About',
        component: () => import('@/views/About.vue'),
        meta: {
            title: '关于我们'
        }
    },
    {
        path: '/user/:id',
        name: 'User',
        component: () => import('@/views/User.vue'),
        props: true,
        meta: {
            requiresAuth: true
        }
    },
    {
        path: '/dashboard',
        component: () => import('@/layouts/DashboardLayout.vue'),
        children: [
            {
                path: '',
                name: 'Dashboard',
                component: () => import('@/views/Dashboard.vue')
            },
            {
                path: 'profile',
                name: 'Profile',
                component: () => import('@/views/Profile.vue')
            },
            {
                path: 'settings',
                name: 'Settings',
                component: () => import('@/views/Settings.vue')
            }
        ]
    },
    {
        path: '/login',
        name: 'Login',
        component: () => import('@/views/Login.vue'),
        meta: {
            guest: true
        }
    },
    {
        // 404页面
        path: '/:pathMatch(.*)*',
        name: 'NotFound',
        component: () => import('@/views/NotFound.vue')
    }
];

const router = createRouter({
    history: createWebHistory(import.meta.env.BASE_URL),
    routes,
    
    // 滚动行为
    scrollBehavior(to, from, savedPosition) {
        if (savedPosition) {
            return savedPosition;
        } else if (to.hash) {
            return {
                el: to.hash,
                behavior: 'smooth'
            };
        } else {
            return { top: 0 };
        }
    }
});

export default router;
```

### 2.2 命名视图

```javascript
// router/index.js
/**
 * 命名视图配置
 * @author erik.zhou
 */
const routes = [
    {
        path: '/',
        components: {
            default: Home,
            sidebar: Sidebar,
            footer: Footer
        }
    },
    {
        path: '/settings',
        components: {
            default: Settings,
            sidebar: SettingsSidebar
        }
    }
];
```

```vue
<!-- App.vue -->
<template>
    <div>
        <router-view />
        <router-view name="sidebar" />
        <router-view name="footer" />
    </div>
</template>
```

### 2.3 路由别名和重定向

```javascript
// router/index.js
/**
 * 路由别名和重定向
 * @author erik.zhou
 */
const routes = [
    {
        path: '/home',
        redirect: '/'
    },
    {
        path: '/home',
        redirect: { name: 'Home' }
    },
    {
        path: '/home',
        redirect: (to) => {
            // 动态重定向
            return { path: '/', query: { q: to.params.searchText } };
        }
    },
    {
        path: '/users/:id',
        component: User,
        alias: ['/u/:id', '/profile/:id']
    }
];
```

---

## 3. 动态路由

### 3.1 动态路由匹配

```javascript
// router/index.js
/**
 * 动态路由配置
 * @author erik.zhou
 */
const routes = [
    {
        // 基本动态路由
        path: '/user/:id',
        component: User
    },
    {
        // 多个动态参数
        path: '/post/:category/:id',
        component: Post
    },
    {
        // 可选参数
        path: '/user/:id?',
        component: User
    },
    {
        // 正则表达式
        path: '/user/:id(\\d+)',
        component: User
    },
    {
        // 可重复参数
        path: '/chapters/:chapters+',
        component: Chapters
    },
    {
        // 可选的可重复参数
        path: '/chapters/:chapters*',
        component: Chapters
    }
];
```

### 3.2 获取路由参数

```vue
<!-- views/User.vue -->
<!--
  获取路由参数
  @author erik.zhou
-->
<template>
    <div>
        <h2>用户详情</h2>
        <p>用户ID: {{ userId }}</p>
        <p>查询参数: {{ $route.query }}</p>
    </div>
</template>

<script setup>
import { computed } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();

// 方式1: 使用computed
const userId = computed(() => route.params.id);

// 方式2: 使用props
defineProps({
    id: {
        type: String,
        required: true
    }
});

// 监听路由参数变化
watch(
    () => route.params.id,
    (newId, oldId) => {
        console.log(`用户ID从 ${oldId} 变为 ${newId}`);
        // 重新获取数据
        fetchUser(newId);
    }
);
</script>
```

### 3.3 动态添加路由

```javascript
// router/dynamicRoutes.js
/**
 * 动态添加路由
 * @author erik.zhou
 */
import { useRouter } from 'vue-router';

export function useDynamicRoutes() {
    const router = useRouter();
    
    // 添加单个路由
    const addRoute = (route) => {
        router.addRoute(route);
    };
    
    // 添加嵌套路由
    const addNestedRoute = (parentName, route) => {
        router.addRoute(parentName, route);
    };
    
    // 删除路由
    const removeRoute = (name) => {
        router.removeRoute(name);
    };
    
    // 检查路由是否存在
    const hasRoute = (name) => {
        return router.hasRoute(name);
    };
    
    // 获取所有路由
    const getRoutes = () => {
        return router.getRoutes();
    };
    
    return {
        addRoute,
        addNestedRoute,
        removeRoute,
        hasRoute,
        getRoutes
    };
}

// 使用示例
const loadUserRoutes = async (userRole) => {
    const { addRoute } = useDynamicRoutes();
    
    if (userRole === 'admin') {
        addRoute({
            path: '/admin',
            name: 'Admin',
            component: () => import('@/views/Admin.vue'),
            meta: {
                requiresAuth: true,
                role: 'admin'
            }
        });
    }
};
```

---

## 4. 嵌套路由

### 4.1 嵌套路由配置

```javascript
// router/index.js
/**
 * 嵌套路由配置
 * @author erik.zhou
 */
const routes = [
    {
        path: '/dashboard',
        component: DashboardLayout,
        children: [
            {
                // 默认子路由
                path: '',
                name: 'Dashboard',
                component: Dashboard
            },
            {
                path: 'profile',
                name: 'Profile',
                component: Profile
            },
            {
                path: 'settings',
                component: Settings,
                children: [
                    {
                        path: 'account',
                        name: 'AccountSettings',
                        component: AccountSettings
                    },
                    {
                        path: 'security',
                        name: 'SecuritySettings',
                        component: SecuritySettings
                    }
                ]
            }
        ]
    }
];
```

### 4.2 嵌套路由组件

```vue
<!-- layouts/DashboardLayout.vue -->
<!--
  嵌套路由布局组件
  @author erik.zhou
-->
<template>
    <div class="dashboard-layout">
        <aside class="sidebar">
            <nav>
                <router-link to="/dashboard">概览</router-link>
                <router-link to="/dashboard/profile">个人资料</router-link>
                <router-link to="/dashboard/settings">设置</router-link>
            </nav>
        </aside>
        
        <main class="content">
            <!-- 嵌套路由出口 -->
            <router-view />
        </main>
    </div>
</template>

<style scoped>
.dashboard-layout {
    display: flex;
    min-height: 100vh;
}

.sidebar {
    width: 250px;
    background: #f5f5f5;
    padding: 20px;
}

.content {
    flex: 1;
    padding: 20px;
}
</style>
```

```vue
<!-- views/Settings.vue -->
<!--
  多级嵌套路由
  @author erik.zhou
-->
<template>
    <div class="settings">
        <h2>设置</h2>
        
        <nav class="settings-nav">
            <router-link to="/dashboard/settings/account">
                账户设置
            </router-link>
            <router-link to="/dashboard/settings/security">
                安全设置
            </router-link>
        </nav>
        
        <!-- 二级嵌套路由出口 -->
        <div class="settings-content">
            <router-view />
        </div>
    </div>
</template>

<style scoped>
.settings-nav {
    display: flex;
    gap: 20px;
    margin: 20px 0;
    border-bottom: 1px solid #ddd;
}
</style>
```



---

## 5. 导航守卫

### 5.1 全局前置守卫

```javascript
// router/index.js
/**
 * 全局前置守卫
 * @author erik.zhou
 */
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
    history: createWebHistory(),
    routes
});

// 全局前置守卫
router.beforeEach((to, from, next) => {
    console.log('导航到:', to.path);
    console.log('来自:', from.path);
    
    // 检查是否需要认证
    if (to.meta.requiresAuth) {
        const isAuthenticated = checkAuth();
        
        if (!isAuthenticated) {
            // 重定向到登录页
            next({
                name: 'Login',
                query: { redirect: to.fullPath }
            });
        } else {
            next();
        }
    } else {
        next();
    }
});

// 检查认证状态
function checkAuth() {
    return localStorage.getItem('token') !== null;
}

export default router;
```

### 5.2 全局解析守卫和后置钩子

```javascript
// router/guards.js
/**
 * 全局守卫配置
 * @author erik.zhou
 */

// 全局解析守卫
router.beforeResolve(async (to, from, next) => {
    // 在导航被确认之前，所有组件内守卫和异步路由组件被解析之后调用
    if (to.meta.requiresCamera) {
        try {
            await askForCameraPermission();
            next();
        } catch (error) {
            next(false);
        }
    } else {
        next();
    }
});

// 全局后置钩子
router.afterEach((to, from, failure) => {
    if (!failure) {
        // 更新页面标题
        document.title = to.meta.title || '默认标题';
        
        // 发送页面浏览统计
        sendToAnalytics(to.path);
        
        // 关闭加载指示器
        hideLoadingIndicator();
    }
});

// 导航失败处理
router.onError((error) => {
    console.error('路由错误:', error);
});
```

### 5.3 路由独享守卫

```javascript
// router/index.js
/**
 * 路由独享守卫
 * @author erik.zhou
 */
const routes = [
    {
        path: '/admin',
        component: Admin,
        beforeEnter: (to, from, next) => {
            // 只对这个路由生效
            const user = getCurrentUser();
            
            if (user && user.role === 'admin') {
                next();
            } else {
                next({ name: 'Forbidden' });
            }
        }
    },
    {
        path: '/settings',
        component: Settings,
        beforeEnter: [
            checkAuth,
            checkPermission,
            logAccess
        ]
    }
];

// 守卫函数
function checkAuth(to, from, next) {
    if (isAuthenticated()) {
        next();
    } else {
        next('/login');
    }
}

function checkPermission(to, from, next) {
    if (hasPermission(to.meta.permission)) {
        next();
    } else {
        next('/forbidden');
    }
}

function logAccess(to, from, next) {
    console.log(`访问 ${to.path}`);
    next();
}
```

### 5.4 组件内守卫

```vue
<!-- views/User.vue -->
<!--
  组件内守卫
  @author erik.zhou
-->
<template>
    <div>
        <h2>用户详情</h2>
        <p>{{ user.name }}</p>
    </div>
</template>

<script>
export default {
    name: 'User',
    
    // 进入组件前
    beforeRouteEnter(to, from, next) {
        // 此时组件实例还未创建，不能访问this
        fetchUser(to.params.id).then(user => {
            next(vm => {
                // 通过vm访问组件实例
                vm.user = user;
            });
        });
    },
    
    // 路由参数变化时
    beforeRouteUpdate(to, from, next) {
        // 可以访问this
        this.fetchUser(to.params.id);
        next();
    },
    
    // 离开组件时
    beforeRouteLeave(to, from, next) {
        // 可以访问this
        if (this.hasUnsavedChanges) {
            const answer = window.confirm('有未保存的更改，确定要离开吗？');
            if (answer) {
                next();
            } else {
                next(false);
            }
        } else {
            next();
        }
    },
    
    data() {
        return {
            user: null,
            hasUnsavedChanges: false
        };
    },
    
    methods: {
        async fetchUser(id) {
            const response = await fetch(`/api/users/${id}`);
            this.user = await response.json();
        }
    }
};
</script>
```

```vue
<!-- 组合式API中的守卫 -->
<script setup>
import { ref, onBeforeRouteUpdate, onBeforeRouteLeave } from 'vue-router';

const user = ref(null);
const hasUnsavedChanges = ref(false);

// 路由更新守卫
onBeforeRouteUpdate(async (to, from) => {
    if (to.params.id !== from.params.id) {
        user.value = await fetchUser(to.params.id);
    }
});

// 路由离开守卫
onBeforeRouteLeave((to, from) => {
    if (hasUnsavedChanges.value) {
        const answer = window.confirm('有未保存的更改，确定要离开吗？');
        if (!answer) {
            return false;
        }
    }
});
</script>
```

### 5.5 完整的导航解析流程

```javascript
/**
 * 完整的导航解析流程
 * @author erik.zhou
 */

/*
1. 导航被触发
2. 在失活的组件里调用 beforeRouteLeave 守卫
3. 调用全局的 beforeEach 守卫
4. 在重用的组件里调用 beforeRouteUpdate 守卫
5. 在路由配置里调用 beforeEnter
6. 解析异步路由组件
7. 在被激活的组件里调用 beforeRouteEnter
8. 调用全局的 beforeResolve 守卫
9. 导航被确认
10. 调用全局的 afterEach 钩子
11. 触发 DOM 更新
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数
*/

// 实际应用示例
router.beforeEach((to, from, next) => {
    console.log('1. 全局前置守卫');
    
    // 显示加载指示器
    showLoadingIndicator();
    
    // 权限检查
    if (to.meta.requiresAuth && !isAuthenticated()) {
        next('/login');
    } else {
        next();
    }
});

router.beforeResolve((to, from, next) => {
    console.log('2. 全局解析守卫');
    next();
});

router.afterEach((to, from) => {
    console.log('3. 全局后置钩子');
    
    // 隐藏加载指示器
    hideLoadingIndicator();
    
    // 更新页面标题
    document.title = to.meta.title || '默认标题';
});
```

---

## 6. 路由懒加载

### 6.1 基本懒加载

```javascript
// router/index.js
/**
 * 路由懒加载配置
 * @author erik.zhou
 */
const routes = [
    {
        path: '/',
        name: 'Home',
        // 懒加载
        component: () => import('@/views/Home.vue')
    },
    {
        path: '/about',
        name: 'About',
        // 使用webpack魔法注释命名chunk
        component: () => import(/* webpackChunkName: "about" */ '@/views/About.vue')
    }
];
```

### 6.2 路由分组

```javascript
// router/index.js
/**
 * 路由分组懒加载
 * @author erik.zhou
 */
const routes = [
    {
        path: '/user',
        component: () => import(/* webpackChunkName: "user" */ '@/views/User.vue')
    },
    {
        path: '/user/profile',
        component: () => import(/* webpackChunkName: "user" */ '@/views/UserProfile.vue')
    },
    {
        path: '/user/settings',
        component: () => import(/* webpackChunkName: "user" */ '@/views/UserSettings.vue')
    },
    {
        path: '/admin',
        component: () => import(/* webpackChunkName: "admin" */ '@/views/Admin.vue')
    },
    {
        path: '/admin/users',
        component: () => import(/* webpackChunkName: "admin" */ '@/views/AdminUsers.vue')
    }
];
```

### 6.3 预加载和预获取

```javascript
// router/index.js
/**
 * 预加载和预获取配置
 * @author erik.zhou
 */
const routes = [
    {
        path: '/dashboard',
        component: () => import(
            /* webpackChunkName: "dashboard" */
            /* webpackPrefetch: true */
            '@/views/Dashboard.vue'
        )
    },
    {
        path: '/heavy',
        component: () => import(
            /* webpackChunkName: "heavy" */
            /* webpackPreload: true */
            '@/views/HeavyComponent.vue'
        )
    }
];
```

---

## 7. 路由元信息

### 7.1 定义元信息

```javascript
// router/index.js
/**
 * 路由元信息配置
 * @author erik.zhou
 */
const routes = [
    {
        path: '/dashboard',
        component: Dashboard,
        meta: {
            title: '控制台',
            requiresAuth: true,
            roles: ['admin', 'user'],
            icon: 'dashboard',
            breadcrumb: '控制台',
            keepAlive: true,
            transition: 'slide-left'
        }
    },
    {
        path: '/admin',
        component: Admin,
        meta: {
            title: '管理后台',
            requiresAuth: true,
            roles: ['admin'],
            permission: 'admin:view'
        },
        children: [
            {
                path: 'users',
                component: AdminUsers,
                meta: {
                    title: '用户管理',
                    breadcrumb: '用户管理',
                    permission: 'admin:users:view'
                }
            }
        ]
    }
];
```

### 7.2 使用元信息

```javascript
// router/guards.js
/**
 * 使用路由元信息
 * @author erik.zhou
 */

// 权限检查
router.beforeEach((to, from, next) => {
    // 检查认证
    if (to.meta.requiresAuth) {
        if (!isAuthenticated()) {
            next('/login');
            return;
        }
    }
    
    // 检查角色
    if (to.meta.roles) {
        const userRole = getUserRole();
        if (!to.meta.roles.includes(userRole)) {
            next('/forbidden');
            return;
        }
    }
    
    // 检查权限
    if (to.meta.permission) {
        if (!hasPermission(to.meta.permission)) {
            next('/forbidden');
            return;
        }
    }
    
    next();
});

// 更新页面标题
router.afterEach((to) => {
    if (to.meta.title) {
        document.title = `${to.meta.title} - 我的应用`;
    }
});
```

```vue
<!-- components/Breadcrumb.vue -->
<!--
  面包屑导航
  @author erik.zhou
-->
<template>
    <nav class="breadcrumb">
        <router-link
            v-for="(item, index) in breadcrumbs"
            :key="index"
            :to="item.path"
        >
            {{ item.meta.breadcrumb || item.meta.title }}
        </router-link>
    </nav>
</template>

<script setup>
import { computed } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();

const breadcrumbs = computed(() => {
    return route.matched.filter(r => r.meta.breadcrumb);
});
</script>
```

---

## 8. 编程式导航

### 8.1 基本导航方法

```vue
<!-- components/Navigation.vue -->
<!--
  编程式导航示例
  @author erik.zhou
-->
<template>
    <div>
        <button @click="goToHome">返回首页</button>
        <button @click="goToUser">用户详情</button>
        <button @click="goBack">后退</button>
        <button @click="goForward">前进</button>
    </div>
</template>

<script setup>
import { useRouter } from 'vue-router';

const router = useRouter();

// 导航到指定路径
const goToHome = () => {
    router.push('/');
};

// 使用命名路由
const goToUser = () => {
    router.push({
        name: 'User',
        params: { id: 123 }
    });
};

// 后退
const goBack = () => {
    router.back();
    // 或者
    router.go(-1);
};

// 前进
const goForward = () => {
    router.forward();
    // 或者
    router.go(1);
};

// 替换当前历史记录
const replaceRoute = () => {
    router.replace('/new-path');
};
</script>
```

### 8.2 导航选项

```javascript
/**
 * 导航选项详解
 * @author erik.zhou
 */
import { useRouter } from 'vue-router';

const router = useRouter();

// 1. 基本导航
router.push('/users');

// 2. 带参数的导航
router.push({
    path: '/user',
    query: { id: 123 }
});

// 3. 命名路由导航
router.push({
    name: 'User',
    params: { id: 123 }
});

// 4. 带查询参数和hash
router.push({
    path: '/user',
    query: { plan: 'private' },
    hash: '#team'
});

// 5. 替换历史记录
router.push({
    path: '/user',
    replace: true
});

// 6. 保留当前查询参数
router.push({
    path: '/new-path',
    query: {
        ...router.currentRoute.value.query,
        newParam: 'value'
    }
});
```

### 8.3 导航失败处理

```javascript
/**
 * 导航失败处理
 * @author erik.zhou
 */
import { useRouter, NavigationFailureType, isNavigationFailure } from 'vue-router';

const router = useRouter();

// 检测导航失败
router.push('/admin').catch(failure => {
    if (isNavigationFailure(failure, NavigationFailureType.aborted)) {
        console.log('导航被中止');
    } else if (isNavigationFailure(failure, NavigationFailureType.cancelled)) {
        console.log('导航被取消');
    } else if (isNavigationFailure(failure, NavigationFailureType.duplicated)) {
        console.log('导航重复');
    }
});

// 使用async/await
async function navigateToUser(id) {
    try {
        await router.push({ name: 'User', params: { id } });
        console.log('导航成功');
    } catch (failure) {
        if (isNavigationFailure(failure)) {
            console.log('导航失败:', failure);
        }
    }
}
```

---

## 9. 路由传参

### 9.1 Params传参

```javascript
// router/index.js
/**
 * Params传参配置
 * @author erik.zhou
 */
const routes = [
    {
        path: '/user/:id',
        name: 'User',
        component: User,
        props: true  // 将params作为props传递
    },
    {
        path: '/post/:category/:id',
        name: 'Post',
        component: Post,
        props: true
    }
];
```

```vue
<!-- 传递参数 -->
<template>
    <div>
        <!-- 方式1: 字符串路径 -->
        <router-link to="/user/123">用户123</router-link>
        
        <!-- 方式2: 对象形式 -->
        <router-link :to="{ name: 'User', params: { id: 123 } }">
            用户123
        </router-link>
        
        <!-- 方式3: 编程式导航 -->
        <button @click="goToUser(123)">用户123</button>
    </div>
</template>

<script setup>
import { useRouter } from 'vue-router';

const router = useRouter();

const goToUser = (id) => {
    router.push({
        name: 'User',
        params: { id }
    });
};
</script>
```

```vue
<!-- 接收参数 -->
<template>
    <div>
        <h2>用户ID: {{ id }}</h2>
    </div>
</template>

<script setup>
// 方式1: 通过props接收
defineProps({
    id: {
        type: String,
        required: true
    }
});

// 方式2: 通过route接收
import { useRoute } from 'vue-router';
const route = useRoute();
console.log(route.params.id);
</script>
```

### 9.2 Query传参

```vue
<!-- 传递Query参数 -->
<template>
    <div>
        <!-- 方式1: 字符串路径 -->
        <router-link to="/search?q=vue&page=1">搜索</router-link>
        
        <!-- 方式2: 对象形式 -->
        <router-link :to="{ path: '/search', query: { q: 'vue', page: 1 } }">
            搜索
        </router-link>
        
        <!-- 方式3: 编程式导航 -->
        <button @click="search('vue', 1)">搜索</button>
    </div>
</template>

<script setup>
import { useRouter } from 'vue-router';

const router = useRouter();

const search = (keyword, page) => {
    router.push({
        path: '/search',
        query: {
            q: keyword,
            page
        }
    });
};
</script>
```

```vue
<!-- 接收Query参数 -->
<template>
    <div>
        <h2>搜索: {{ searchQuery }}</h2>
        <p>页码: {{ currentPage }}</p>
    </div>
</template>

<script setup>
import { computed } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();

const searchQuery = computed(() => route.query.q || '');
const currentPage = computed(() => Number(route.query.page) || 1);

// 监听query变化
watch(
    () => route.query,
    (newQuery) => {
        console.log('Query参数变化:', newQuery);
        // 重新获取数据
        fetchData(newQuery);
    }
);
</script>
```

### 9.3 Props传参配置

```javascript
// router/index.js
/**
 * Props传参的不同模式
 * @author erik.zhou
 */
const routes = [
    {
        path: '/user/:id',
        component: User,
        // 布尔模式: params作为props
        props: true
    },
    {
        path: '/search',
        component: Search,
        // 对象模式: 静态props
        props: { default: true }
    },
    {
        path: '/post/:id',
        component: Post,
        // 函数模式: 自定义props
        props: (route) => ({
            id: Number(route.params.id),
            query: route.query.q,
            page: Number(route.query.page) || 1
        })
    },
    {
        path: '/dashboard',
        components: {
            default: Dashboard,
            sidebar: Sidebar
        },
        // 命名视图的props
        props: {
            default: true,
            sidebar: false
        }
    }
];
```



---

## 10. 高级特性

### 10.1 路由过渡动画

```vue
<!-- App.vue -->
<!--
  路由过渡动画
  @author erik.zhou
-->
<template>
    <div>
        <!-- 基本过渡 -->
        <router-view v-slot="{ Component }">
            <transition name="fade" mode="out-in">
                <component :is="Component" />
            </transition>
        </router-view>
        
        <!-- 基于路由的动态过渡 -->
        <router-view v-slot="{ Component, route }">
            <transition :name="route.meta.transition || 'fade'" mode="out-in">
                <component :is="Component" :key="route.path" />
            </transition>
        </router-view>
        
        <!-- 滑动过渡 -->
        <router-view v-slot="{ Component }">
            <transition
                :name="transitionName"
                :mode="transitionMode"
                @before-enter="beforeEnter"
                @after-enter="afterEnter"
            >
                <component :is="Component" />
            </transition>
        </router-view>
    </div>
</template>

<script setup>
import { ref, watch } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const transitionName = ref('slide-left');
const transitionMode = ref('out-in');

// 根据路由深度决定过渡方向
watch(
    () => route.path,
    (to, from) => {
        const toDepth = to.split('/').length;
        const fromDepth = from.split('/').length;
        transitionName.value = toDepth < fromDepth ? 'slide-right' : 'slide-left';
    }
);

const beforeEnter = (el) => {
    console.log('过渡开始');
};

const afterEnter = (el) => {
    console.log('过渡结束');
};
</script>

<style>
/* 淡入淡出 */
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

/* 滑动 */
.slide-left-enter-active,
.slide-left-leave-active,
.slide-right-enter-active,
.slide-right-leave-active {
    transition: all 0.3s;
}

.slide-left-enter-from {
    transform: translateX(100%);
}

.slide-left-leave-to {
    transform: translateX(-100%);
}

.slide-right-enter-from {
    transform: translateX(-100%);
}

.slide-right-leave-to {
    transform: translateX(100%);
}
</style>
```

### 10.2 滚动行为

```javascript
// router/index.js
/**
 * 滚动行为配置
 * @author erik.zhou
 */
const router = createRouter({
    history: createWebHistory(),
    routes,
    
    scrollBehavior(to, from, savedPosition) {
        // 1. 如果有保存的位置（浏览器前进/后退）
        if (savedPosition) {
            return savedPosition;
        }
        
        // 2. 如果有hash锚点
        if (to.hash) {
            return {
                el: to.hash,
                behavior: 'smooth',
                top: 80  // 偏移量（考虑固定头部）
            };
        }
        
        // 3. 滚动到指定位置
        if (to.meta.scrollToTop) {
            return { top: 0 };
        }
        
        // 4. 延迟滚动（等待页面渲染）
        return new Promise((resolve) => {
            setTimeout(() => {
                resolve({ top: 0, behavior: 'smooth' });
            }, 300);
        });
    }
});
```

### 10.3 路由懒加载错误处理

```javascript
// router/index.js
/**
 * 路由懒加载错误处理
 * @author erik.zhou
 */

// 创建带错误处理的懒加载函数
function lazyLoadView(AsyncView) {
    const AsyncHandler = () => ({
        component: AsyncView,
        
        // 加载中显示的组件
        loading: LoadingComponent,
        
        // 加载失败显示的组件
        error: ErrorComponent,
        
        // 延迟显示loading组件的时间
        delay: 200,
        
        // 超时时间
        timeout: 10000
    });
    
    return Promise.resolve({
        functional: true,
        render(h, { data, children }) {
            return h(AsyncHandler, data, children);
        }
    });
}

const routes = [
    {
        path: '/dashboard',
        component: () => lazyLoadView(import('@/views/Dashboard.vue'))
    }
];

// 全局错误处理
router.onError((error) => {
    if (/ChunkLoadError/.test(error.message)) {
        // 处理chunk加载失败
        console.error('路由chunk加载失败:', error);
        
        // 提示用户刷新页面
        if (confirm('页面加载失败，是否刷新页面？')) {
            window.location.reload();
        }
    }
});
```

### 10.4 路由权限控制

```javascript
// router/permission.js
/**
 * 路由权限控制
 * @author erik.zhou
 */
import router from './index';
import { useUserStore } from '@/stores/user';

// 白名单路由
const whiteList = ['/login', '/register', '/404'];

router.beforeEach(async (to, from, next) => {
    const userStore = useUserStore();
    
    // 获取token
    const hasToken = userStore.token;
    
    if (hasToken) {
        if (to.path === '/login') {
            // 已登录，重定向到首页
            next({ path: '/' });
        } else {
            // 检查是否已获取用户信息
            const hasRoles = userStore.roles && userStore.roles.length > 0;
            
            if (hasRoles) {
                next();
            } else {
                try {
                    // 获取用户信息
                    const { roles } = await userStore.getUserInfo();
                    
                    // 根据角色生成可访问路由
                    const accessRoutes = await generateRoutes(roles);
                    
                    // 动态添加路由
                    accessRoutes.forEach(route => {
                        router.addRoute(route);
                    });
                    
                    // 确保添加路由已完成
                    next({ ...to, replace: true });
                } catch (error) {
                    // 获取用户信息失败，重定向到登录页
                    await userStore.logout();
                    next(`/login?redirect=${to.path}`);
                }
            }
        }
    } else {
        // 未登录
        if (whiteList.includes(to.path)) {
            next();
        } else {
            next(`/login?redirect=${to.path}`);
        }
    }
});

// 根据角色生成路由
async function generateRoutes(roles) {
    const asyncRoutes = [
        {
            path: '/admin',
            component: () => import('@/views/Admin.vue'),
            meta: { roles: ['admin'] }
        },
        {
            path: '/editor',
            component: () => import('@/views/Editor.vue'),
            meta: { roles: ['admin', 'editor'] }
        }
    ];
    
    // 过滤有权限的路由
    return asyncRoutes.filter(route => {
        if (route.meta && route.meta.roles) {
            return roles.some(role => route.meta.roles.includes(role));
        }
        return true;
    });
}
```

### 10.5 路由缓存策略

```vue
<!-- App.vue -->
<!--
  路由缓存策略
  @author erik.zhou
-->
<template>
    <div>
        <router-view v-slot="{ Component, route }">
            <keep-alive :include="cachedViews" :max="10">
                <component :is="Component" :key="route.fullPath" />
            </keep-alive>
        </router-view>
    </div>
</template>

<script setup>
import { computed } from 'vue';
import { useRoute } from 'vue-router';
import { useAppStore } from '@/stores/app';

const route = useRoute();
const appStore = useAppStore();

// 需要缓存的组件名称列表
const cachedViews = computed(() => appStore.cachedViews);

// 根据路由meta决定是否缓存
watch(
    () => route.name,
    (newName) => {
        if (route.meta.keepAlive) {
            appStore.addCachedView(newName);
        }
    }
);
</script>
```

```javascript
// stores/app.js
/**
 * 应用状态管理
 * @author erik.zhou
 */
import { defineStore } from 'pinia';

export const useAppStore = defineStore('app', {
    state: () => ({
        cachedViews: []
    }),
    
    actions: {
        addCachedView(view) {
            if (this.cachedViews.includes(view)) {
                return;
            }
            this.cachedViews.push(view);
        },
        
        removeCachedView(view) {
            const index = this.cachedViews.indexOf(view);
            if (index > -1) {
                this.cachedViews.splice(index, 1);
            }
        },
        
        clearCachedViews() {
            this.cachedViews = [];
        }
    }
});
```

### 10.6 路由监听和调试

```javascript
// utils/routerDebug.js
/**
 * 路由调试工具
 * @author erik.zhou
 */
import { watch } from 'vue';
import { useRouter, useRoute } from 'vue-router';

export function useRouterDebug() {
    const router = useRouter();
    const route = useRoute();
    
    // 监听路由变化
    watch(
        () => route.fullPath,
        (to, from) => {
            console.group('路由变化');
            console.log('从:', from);
            console.log('到:', to);
            console.log('参数:', route.params);
            console.log('查询:', route.query);
            console.log('元信息:', route.meta);
            console.groupEnd();
        }
    );
    
    // 监听导航失败
    router.afterEach((to, from, failure) => {
        if (failure) {
            console.error('导航失败:', failure);
        }
    });
    
    // 获取所有路由
    const getAllRoutes = () => {
        return router.getRoutes();
    };
    
    // 检查路由是否存在
    const hasRoute = (name) => {
        return router.hasRoute(name);
    };
    
    return {
        getAllRoutes,
        hasRoute
    };
}
```

---

## 总结

### 核心要点

Vue Router 4是Vue 3的官方路由管理器，主要特点包括：

1. **路由配置**
   - 支持多种History模式
   - 灵活的路由配置选项
   - 命名视图和嵌套路由
   - 路由别名和重定向

2. **动态路由**
   - 动态路由匹配
   - 路由参数和正则表达式
   - 动态添加和删除路由
   - 路由优先级控制

3. **导航守卫**
   - 全局守卫（beforeEach、beforeResolve、afterEach）
   - 路由独享守卫（beforeEnter）
   - 组件内守卫（beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave）
   - 完整的导航解析流程

4. **路由懒加载**
   - 按需加载组件
   - 路由分组优化
   - 预加载和预获取
   - 错误处理机制

5. **路由传参**
   - Params参数传递
   - Query参数传递
   - Props配置模式
   - 参数验证和转换

6. **高级特性**
   - 路由过渡动画
   - 滚动行为控制
   - 权限控制系统
   - 路由缓存策略

### 最佳实践

1. **路由设计**
   - 合理的路由层级结构
   - 清晰的命名规范
   - 使用路由元信息
   - 避免过深的嵌套

2. **性能优化**
   - 使用路由懒加载
   - 合理的路由分组
   - 组件缓存策略
   - 预加载关键路由

3. **权限控制**
   - 统一的权限检查
   - 动态路由生成
   - 白名单机制
   - 错误处理和降级

4. **用户体验**
   - 平滑的路由过渡
   - 合理的滚动行为
   - 加载状态提示
   - 错误页面处理

### 学习路径

1. **基础阶段**
   - 掌握路由基本配置
   - 理解路由匹配规则
   - 学会使用导航守卫
   - 掌握路由传参方式

2. **进阶阶段**
   - 动态路由管理
   - 权限控制实现
   - 路由懒加载优化
   - 路由过渡动画

3. **高级阶段**
   - 复杂权限系统设计
   - 路由性能优化
   - 路由缓存策略
   - 大型应用路由架构

### 推荐资源

**官方文档**
- [Vue Router官方文档](https://router.vuejs.org/zh/)
- [Vue Router GitHub](https://github.com/vuejs/router)
- [Vue Router迁移指南](https://router.vuejs.org/zh/guide/migration/)

**学习资源**
- [Vue Router实战教程](https://www.vuemastery.com/courses/vue-router-4/)
- [Vue Router最佳实践](https://vueschool.io/courses/vue-router-4-for-everyone)

**相关工具**
- [unplugin-vue-router](https://github.com/posva/unplugin-vue-router) - 基于文件的路由
- [vite-plugin-pages](https://github.com/hannoeru/vite-plugin-pages) - 自动路由生成

---

## 附录

### A. 常用API速查

```javascript
/**
 * Vue Router常用API
 * @author erik.zhou
 */

// 创建路由
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
    history: createWebHistory(),
    routes: []
});

// 组合式API
import { useRouter, useRoute } from 'vue-router';

const router = useRouter();  // 路由实例
const route = useRoute();    // 当前路由

// 导航方法
router.push(location);       // 导航到新路由
router.replace(location);    // 替换当前路由
router.go(n);               // 前进/后退
router.back();              // 后退
router.forward();           // 前进

// 路由管理
router.addRoute(route);     // 添加路由
router.removeRoute(name);   // 删除路由
router.hasRoute(name);      // 检查路由
router.getRoutes();         // 获取所有路由

// 导航守卫
router.beforeEach(guard);   // 全局前置守卫
router.beforeResolve(guard);// 全局解析守卫
router.afterEach(hook);     // 全局后置钩子
```

### B. 常见问题FAQ

**Q1: 如何在setup中访问路由？**

A: 使用useRouter和useRoute组合式API
```javascript
import { useRouter, useRoute } from 'vue-router';

const router = useRouter();
const route = useRoute();
```

**Q2: params和query的区别？**

A:
- params: 路径参数，如/user/:id，不会显示在URL查询字符串中
- query: 查询参数，如/user?id=123，会显示在URL查询字符串中
- params需要在路由配置中定义，query不需要

**Q3: 如何实现路由权限控制？**

A: 使用导航守卫结合路由元信息
```javascript
router.beforeEach((to, from, next) => {
    if (to.meta.requiresAuth && !isAuthenticated()) {
        next('/login');
    } else {
        next();
    }
});
```

**Q4: 如何处理404页面？**

A: 使用通配符路由
```javascript
{
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: NotFound
}
```

**Q5: 如何实现路由懒加载？**

A: 使用动态import
```javascript
{
    path: '/about',
    component: () => import('@/views/About.vue')
}
```

---

**课程总结**: 本教程全面讲解了Vue Router 4的核心功能和高级特性，从基础配置到权限控制，从路由懒加载到性能优化，提供了大量实战代码和最佳实践。掌握这些知识，能够帮助你构建功能完善的Vue 3单页应用。

**@author erik.zhou**  
**最后更新**: 2026-03-02
