# Vue 3核心原理 - 完整教程

## 课程信息
- **课程名称**: Vue 3核心原理完整教程
- **难度级别**: 中高级
- **预计学时**: 8小时
- **核心内容**: 响应式系统、组合式API、虚拟DOM、编译器、渲染机制
- **@author**: erik.zhou

---

## 目录
1. [响应式系统原理](#1-响应式系统原理)
2. [组合式API详解](#2-组合式api详解)
3. [虚拟DOM与Diff算法](#3-虚拟dom与diff算法)
4. [编译器原理](#4-编译器原理)
5. [渲染机制](#5-渲染机制)
6. [生命周期](#6-生命周期)
7. [组件通信](#7-组件通信)
8. [内置组件](#8-内置组件)
9. [高级特性](#9-高级特性)
10. [最佳实践](#10-最佳实践)

---

## 1. 响应式系统原理

### 1.1 Proxy基础

```javascript
/**
 * Proxy基础示例
 * @author erik.zhou
 */

// 基本Proxy用法
const target = {
    name: 'Vue',
    version: 3
};

const handler = {
    get(target, key, receiver) {
        console.log(`读取属性: ${key}`);
        return Reflect.get(target, key, receiver);
    },
    
    set(target, key, value, receiver) {
        console.log(`设置属性: ${key} = ${value}`);
        return Reflect.set(target, key, value, receiver);
    }
};

const proxy = new Proxy(target, handler);

// 触发get
console.log(proxy.name); // 输出: 读取属性: name, Vue

// 触发set
proxy.version = 4; // 输出: 设置属性: version = 4
```

### 1.2 reactive实现原理

```javascript
/**
 * 简化版reactive实现
 * @author erik.zhou
 */

// 存储副作用函数
let activeEffect = null;
const targetMap = new WeakMap();

// 依赖收集
function track(target, key) {
    if (!activeEffect) {
        return;
    }
    
    let depsMap = targetMap.get(target);
    if (!depsMap) {
        targetMap.set(target, (depsMap = new Map()));
    }
    
    let dep = depsMap.get(key);
    if (!dep) {
        depsMap.set(key, (dep = new Set()));
    }
    
    dep.add(activeEffect);
}

// 触发更新
function trigger(target, key) {
    const depsMap = targetMap.get(target);
    if (!depsMap) {
        return;
    }
    
    const dep = depsMap.get(key);
    if (dep) {
        dep.forEach(effect => effect());
    }
}

// 创建响应式对象
function reactive(target) {
    return new Proxy(target, {
        get(target, key, receiver) {
            const result = Reflect.get(target, key, receiver);
            track(target, key);
            return result;
        },
        
        set(target, key, value, receiver) {
            const result = Reflect.set(target, key, value, receiver);
            trigger(target, key);
            return result;
        }
    });
}

// 副作用函数
function effect(fn) {
    activeEffect = fn;
    fn();
    activeEffect = null;
}

// 使用示例
const state = reactive({
    count: 0,
    message: 'Hello'
});

effect(() => {
    console.log('count变化:', state.count);
});

state.count++; // 输出: count变化: 1
state.count++; // 输出: count变化: 2
```

### 1.3 ref实现原理

```javascript
/**
 * 简化版ref实现
 * @author erik.zhou
 */

class RefImpl {
    constructor(value) {
        this._value = value;
    }
    
    get value() {
        track(this, 'value');
        return this._value;
    }
    
    set value(newValue) {
        this._value = newValue;
        trigger(this, 'value');
    }
}

function ref(value) {
    return new RefImpl(value);
}

// 使用示例
const count = ref(0);

effect(() => {
    console.log('ref count:', count.value);
});

count.value++; // 输出: ref count: 1
count.value++; // 输出: ref count: 2
```

### 1.4 computed实现原理

```javascript
/**
 * 简化版computed实现
 * @author erik.zhou
 */

class ComputedRefImpl {
    constructor(getter) {
        this._getter = getter;
        this._dirty = true;
        this._value = undefined;
        
        this._effect = effect(() => {
            if (!this._dirty) {
                this._dirty = true;
                trigger(this, 'value');
            }
        });
    }
    
    get value() {
        if (this._dirty) {
            this._value = this._getter();
            this._dirty = false;
        }
        track(this, 'value');
        return this._value;
    }
}

function computed(getter) {
    return new ComputedRefImpl(getter);
}

// 使用示例
const count = ref(1);
const double = computed(() => count.value * 2);

console.log(double.value); // 2
count.value = 2;
console.log(double.value); // 4
```


### 1.5 watch实现原理

```javascript
/**
 * 简化版watch实现
 * @author erik.zhou
 */

function watch(source, callback, options = {}) {
    let getter;
    
    // 处理不同类型的source
    if (typeof source === 'function') {
        getter = source;
    } else {
        getter = () => traverse(source);
    }
    
    let oldValue, newValue;
    
    const job = () => {
        newValue = effectFn();
        callback(newValue, oldValue);
        oldValue = newValue;
    };
    
    const effectFn = effect(
        () => getter(),
        {
            lazy: true,
            scheduler: () => {
                if (options.flush === 'post') {
                    Promise.resolve().then(job);
                } else {
                    job();
                }
            }
        }
    );
    
    if (options.immediate) {
        job();
    } else {
        oldValue = effectFn();
    }
}

// 递归遍历对象
function traverse(value, seen = new Set()) {
    if (typeof value !== 'object' || value === null || seen.has(value)) {
        return value;
    }
    
    seen.add(value);
    
    for (const key in value) {
        traverse(value[key], seen);
    }
    
    return value;
}

// 使用示例
const state = reactive({
    count: 0,
    nested: {
        value: 1
    }
});

watch(
    () => state.count,
    (newValue, oldValue) => {
        console.log(`count从 ${oldValue} 变为 ${newValue}`);
    }
);

state.count++; // 输出: count从 0 变为 1
```

### 1.6 响应式系统完整示例

```javascript
/**
 * 响应式系统综合示例
 * @author erik.zhou
 */
import { reactive, ref, computed, watch, watchEffect } from 'vue';

// 创建响应式状态
const state = reactive({
    user: {
        name: 'Vue',
        age: 3
    },
    todos: [
        { id: 1, text: '学习Vue', done: false },
        { id: 2, text: '学习React', done: true }
    ]
});

// ref响应式
const count = ref(0);
const message = ref('Hello Vue 3');

// computed计算属性
const completedTodos = computed(() => {
    return state.todos.filter(todo => todo.done);
});

const completedCount = computed(() => {
    return completedTodos.value.length;
});

// watch监听
watch(
    () => state.user.age,
    (newAge, oldAge) => {
        console.log(`年龄从 ${oldAge} 变为 ${newAge}`);
    }
);

// watchEffect自动追踪依赖
watchEffect(() => {
    console.log(`完成的任务数: ${completedCount.value}`);
});

// 修改状态
state.user.age = 4; // 触发watch
state.todos[0].done = true; // 触发watchEffect
count.value++; // ref更新
```

---

## 2. 组合式API详解

### 2.1 setup函数

```vue
<!-- components/UserProfile.vue -->
<!--
  setup函数详解
  @author erik.zhou
-->
<template>
    <div class="user-profile">
        <h2>{{ user.name }}</h2>
        <p>年龄: {{ user.age }}</p>
        <button @click="incrementAge">增加年龄</button>
        <p>访问次数: {{ visitCount }}</p>
    </div>
</template>

<script>
import { reactive, ref, computed, onMounted } from 'vue';

export default {
    name: 'UserProfile',
    
    props: {
        userId: {
            type: Number,
            required: true
        }
    },
    
    setup(props, context) {
        // context包含: attrs, slots, emit, expose
        
        // 响应式状态
        const user = reactive({
            name: 'Vue User',
            age: 25
        });
        
        const visitCount = ref(0);
        
        // 计算属性
        const userInfo = computed(() => {
            return `${user.name} (${user.age}岁)`;
        });
        
        // 方法
        const incrementAge = () => {
            user.age++;
        };
        
        const loadUserData = async () => {
            try {
                const response = await fetch(`/api/users/${props.userId}`);
                const data = await response.json();
                Object.assign(user, data);
            } catch (error) {
                console.error('加载用户数据失败:', error);
            }
        };
        
        // 生命周期
        onMounted(() => {
            loadUserData();
            visitCount.value++;
        });
        
        // 暴露给模板
        return {
            user,
            visitCount,
            userInfo,
            incrementAge
        };
    }
};
</script>
```

### 2.2 组合式函数（Composables）

```javascript
// composables/useCounter.js
/**
 * 计数器组合式函数
 * @author erik.zhou
 */
import { ref, computed } from 'vue';

export function useCounter(initialValue = 0) {
    const count = ref(initialValue);
    
    const double = computed(() => count.value * 2);
    
    const increment = () => {
        count.value++;
    };
    
    const decrement = () => {
        count.value--;
    };
    
    const reset = () => {
        count.value = initialValue;
    };
    
    return {
        count,
        double,
        increment,
        decrement,
        reset
    };
}
```

```javascript
// composables/useFetch.js
/**
 * 数据获取组合式函数
 * @author erik.zhou
 */
import { ref, watchEffect, toValue } from 'vue';

export function useFetch(url) {
    const data = ref(null);
    const error = ref(null);
    const loading = ref(false);
    
    const fetchData = async () => {
        loading.value = true;
        error.value = null;
        
        try {
            const response = await fetch(toValue(url));
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            data.value = await response.json();
        } catch (err) {
            error.value = err.message;
        } finally {
            loading.value = false;
        }
    };
    
    watchEffect(() => {
        fetchData();
    });
    
    return {
        data,
        error,
        loading,
        refetch: fetchData
    };
}
```

```javascript
// composables/useLocalStorage.js
/**
 * LocalStorage组合式函数
 * @author erik.zhou
 */
import { ref, watch } from 'vue';

export function useLocalStorage(key, defaultValue) {
    const storedValue = localStorage.getItem(key);
    const value = ref(storedValue ? JSON.parse(storedValue) : defaultValue);
    
    watch(
        value,
        (newValue) => {
            localStorage.setItem(key, JSON.stringify(newValue));
        },
        { deep: true }
    );
    
    return value;
}
```

### 2.3 生命周期钩子

```vue
<!-- components/LifecycleDemo.vue -->
<!--
  生命周期钩子示例
  @author erik.zhou
-->
<template>
    <div>
        <h2>生命周期演示</h2>
        <p>{{ message }}</p>
    </div>
</template>

<script setup>
import {
    ref,
    onBeforeMount,
    onMounted,
    onBeforeUpdate,
    onUpdated,
    onBeforeUnmount,
    onUnmounted,
    onErrorCaptured
} from 'vue';

const message = ref('Hello');

// 组件挂载前
onBeforeMount(() => {
    console.log('onBeforeMount: 组件即将挂载');
});

// 组件挂载后
onMounted(() => {
    console.log('onMounted: 组件已挂载');
    // 可以访问DOM
    // 适合发起API请求
});

// 组件更新前
onBeforeUpdate(() => {
    console.log('onBeforeUpdate: 组件即将更新');
});

// 组件更新后
onUpdated(() => {
    console.log('onUpdated: 组件已更新');
});

// 组件卸载前
onBeforeUnmount(() => {
    console.log('onBeforeUnmount: 组件即将卸载');
    // 清理定时器、事件监听器等
});

// 组件卸载后
onUnmounted(() => {
    console.log('onUnmounted: 组件已卸载');
});

// 捕获子组件错误
onErrorCaptured((err, instance, info) => {
    console.error('捕获到错误:', err, info);
    return false; // 阻止错误继续传播
});
</script>
```

### 2.4 依赖注入

```vue
<!-- ParentComponent.vue -->
<!--
  依赖注入 - 父组件
  @author erik.zhou
-->
<template>
    <div>
        <h2>父组件</h2>
        <ChildComponent />
    </div>
</template>

<script setup>
import { provide, ref, readonly } from 'vue';
import ChildComponent from './ChildComponent.vue';

// 提供响应式数据
const theme = ref('dark');
const updateTheme = (newTheme) => {
    theme.value = newTheme;
};

// 提供数据和方法
provide('theme', readonly(theme));
provide('updateTheme', updateTheme);

// 提供对象
provide('userSettings', {
    language: 'zh-CN',
    timezone: 'Asia/Shanghai'
});
</script>
```

```vue
<!-- ChildComponent.vue -->
<!--
  依赖注入 - 子组件
  @author erik.zhou
-->
<template>
    <div>
        <h3>子组件</h3>
        <p>当前主题: {{ theme }}</p>
        <button @click="changeTheme">切换主题</button>
        <p>语言: {{ userSettings.language }}</p>
    </div>
</template>

<script setup>
import { inject } from 'vue';

// 注入数据
const theme = inject('theme');
const updateTheme = inject('updateTheme');
const userSettings = inject('userSettings', {
    language: 'en-US',
    timezone: 'UTC'
});

const changeTheme = () => {
    updateTheme(theme.value === 'dark' ? 'light' : 'dark');
};
</script>
```

---

## 3. 虚拟DOM与Diff算法

### 3.1 虚拟DOM结构

```javascript
/**
 * 虚拟DOM节点结构
 * @author erik.zhou
 */

// VNode类型
const VNodeTypes = {
    ELEMENT: 'element',
    TEXT: 'text',
    COMPONENT: 'component',
    FRAGMENT: 'fragment'
};

// 创建VNode
function createVNode(type, props, children) {
    return {
        type,
        props,
        children,
        el: null,        // 对应的真实DOM
        key: props?.key,
        shapeFlag: getShapeFlag(type)
    };
}

// 获取节点类型标志
function getShapeFlag(type) {
    return typeof type === 'string'
        ? ShapeFlags.ELEMENT
        : ShapeFlags.COMPONENT;
}

// 示例VNode
const vnode = {
    type: 'div',
    props: {
        class: 'container',
        onClick: () => console.log('clicked')
    },
    children: [
        {
            type: 'h1',
            props: null,
            children: 'Hello Vue'
        },
        {
            type: 'p',
            props: { class: 'text' },
            children: 'This is a paragraph'
        }
    ]
};
```

### 3.2 Diff算法实现

```javascript
/**
 * 简化版Diff算法
 * @author erik.zhou
 */

function patchChildren(n1, n2, container) {
    const c1 = n1.children;
    const c2 = n2.children;
    
    // 新节点是文本
    if (typeof c2 === 'string') {
        if (Array.isArray(c1)) {
            c1.forEach(child => unmount(child));
        }
        setElementText(container, c2);
        return;
    }
    
    // 新节点是数组
    if (Array.isArray(c2)) {
        if (Array.isArray(c1)) {
            // 双端Diff算法
            patchKeyedChildren(c1, c2, container);
        } else {
            setElementText(container, '');
            c2.forEach(child => patch(null, child, container));
        }
    }
}

// 双端Diff算法
function patchKeyedChildren(c1, c2, container) {
    let i = 0;
    const l2 = c2.length;
    let e1 = c1.length - 1;
    let e2 = l2 - 1;
    
    // 1. 从头部开始比较
    while (i <= e1 && i <= e2) {
        const n1 = c1[i];
        const n2 = c2[i];
        
        if (isSameVNodeType(n1, n2)) {
            patch(n1, n2, container);
        } else {
            break;
        }
        i++;
    }
    
    // 2. 从尾部开始比较
    while (i <= e1 && i <= e2) {
        const n1 = c1[e1];
        const n2 = c2[e2];
        
        if (isSameVNodeType(n1, n2)) {
            patch(n1, n2, container);
        } else {
            break;
        }
        e1--;
        e2--;
    }
    
    // 3. 新增节点
    if (i > e1) {
        if (i <= e2) {
            const nextPos = e2 + 1;
            const anchor = nextPos < l2 ? c2[nextPos].el : null;
            while (i <= e2) {
                patch(null, c2[i], container, anchor);
                i++;
            }
        }
    }
    // 4. 删除节点
    else if (i > e2) {
        while (i <= e1) {
            unmount(c1[i]);
            i++;
        }
    }
    // 5. 处理中间部分
    else {
        patchMiddleChildren(c1, c2, i, e1, e2, container);
    }
}

// 判断是否相同类型的VNode
function isSameVNodeType(n1, n2) {
    return n1.type === n2.type && n1.key === n2.key;
}
```

### 3.3 最长递增子序列

```javascript
/**
 * 最长递增子序列算法（用于Diff优化）
 * @author erik.zhou
 */

function getSequence(arr) {
    const len = arr.length;
    const result = [0];
    const p = arr.slice();
    
    let i, j, u, v, c;
    
    for (i = 0; i < len; i++) {
        const arrI = arr[i];
        
        if (arrI !== 0) {
            j = result[result.length - 1];
            
            if (arr[j] < arrI) {
                p[i] = j;
                result.push(i);
                continue;
            }
            
            u = 0;
            v = result.length - 1;
            
            while (u < v) {
                c = (u + v) >> 1;
                if (arr[result[c]] < arrI) {
                    u = c + 1;
                } else {
                    v = c;
                }
            }
            
            if (arrI < arr[result[u]]) {
                if (u > 0) {
                    p[i] = result[u - 1];
                }
                result[u] = i;
            }
        }
    }
    
    u = result.length;
    v = result[u - 1];
    
    while (u-- > 0) {
        result[u] = v;
        v = p[v];
    }
    
    return result;
}

// 使用示例
const arr = [2, 3, 1, 5, 6, 4, 8];
console.log(getSequence(arr)); // [2, 3, 5, 6, 8]的索引
```



---

## 4. 编译器原理

### 4.1 模板编译流程

```javascript
/**
 * Vue模板编译流程
 * @author erik.zhou
 */

// 编译流程: 模板 -> AST -> 转换 -> 代码生成

// 1. 解析模板生成AST
function parse(template) {
    // 词法分析和语法分析
    return {
        type: 'Root',
        children: [
            {
                type: 'Element',
                tag: 'div',
                props: [
                    { name: 'class', value: 'container' }
                ],
                children: [
                    {
                        type: 'Text',
                        content: 'Hello Vue'
                    }
                ]
            }
        ]
    };
}

// 2. 转换AST
function transform(ast) {
    // 静态提升
    // 事件缓存
    // 补丁标记
    return ast;
}

// 3. 生成渲染函数代码
function generate(ast) {
    return `
        function render(_ctx) {
            return _createVNode("div", {
                class: "container"
            }, "Hello Vue")
        }
    `;
}

// 完整编译过程
function compile(template) {
    const ast = parse(template);
    const transformedAST = transform(ast);
    const code = generate(transformedAST);
    return code;
}
```

### 4.2 静态提升

```javascript
/**
 * 静态提升优化
 * @author erik.zhou
 */

// 未优化的渲染函数
function renderWithoutHoist() {
    return createVNode('div', null, [
        createVNode('p', { class: 'static' }, 'Static text'),
        createVNode('p', null, dynamicText.value)
    ]);
}

// 静态提升后的渲染函数
const _hoisted_1 = createVNode('p', { class: 'static' }, 'Static text');

function renderWithHoist() {
    return createVNode('div', null, [
        _hoisted_1,
        createVNode('p', null, dynamicText.value)
    ]);
}

// 静态提升的好处:
// 1. 减少每次渲染时的VNode创建
// 2. 减少内存分配
// 3. 提升渲染性能
```

### 4.3 补丁标记（PatchFlags）

```javascript
/**
 * 补丁标记优化
 * @author erik.zhou
 */

// PatchFlags枚举
const PatchFlags = {
    TEXT: 1,              // 动态文本
    CLASS: 2,             // 动态class
    STYLE: 4,             // 动态style
    PROPS: 8,             // 动态属性
    FULL_PROPS: 16,       // 具有动态key的属性
    HYDRATE_EVENTS: 32,   // 具有事件监听器
    STABLE_FRAGMENT: 64,  // 稳定的fragment
    KEYED_FRAGMENT: 128,  // 带key的fragment
    UNKEYED_FRAGMENT: 256,// 不带key的fragment
    NEED_PATCH: 512,      // 需要patch
    DYNAMIC_SLOTS: 1024,  // 动态插槽
    HOISTED: -1,          // 静态节点
    BAIL: -2              // diff算法应该退出优化模式
};

// 带补丁标记的VNode
function renderWithPatchFlag() {
    return createVNode('div', {
        class: dynamicClass.value
    }, [
        createVNode('p', null, dynamicText.value, PatchFlags.TEXT)
    ], PatchFlags.CLASS);
}

// Diff时只需要检查标记的部分
function patchElement(n1, n2) {
    const patchFlag = n2.patchFlag;
    
    if (patchFlag & PatchFlags.TEXT) {
        // 只更新文本
        patchText(n1, n2);
    }
    
    if (patchFlag & PatchFlags.CLASS) {
        // 只更新class
        patchClass(n1, n2);
    }
    
    // 其他标记...
}
```

### 4.4 事件缓存

```vue
<!-- 未优化 -->
<template>
    <button @click="() => handleClick(item)">
        点击
    </button>
</template>

<!-- 优化后 -->
<template>
    <button @click="handleClick">
        点击
    </button>
</template>

<script setup>
/**
 * 事件缓存优化
 * @author erik.zhou
 */

// 编译器会自动缓存事件处理函数
const handleClick = (event) => {
    console.log('clicked', event);
};
</script>
```

---

## 5. 渲染机制

### 5.1 渲染器实现

```javascript
/**
 * 简化版渲染器实现
 * @author erik.zhou
 */

function createRenderer(options) {
    const {
        createElement,
        setElementText,
        insert,
        patchProps
    } = options;
    
    // 挂载元素
    function mountElement(vnode, container) {
        const el = vnode.el = createElement(vnode.type);
        
        // 处理子节点
        if (typeof vnode.children === 'string') {
            setElementText(el, vnode.children);
        } else if (Array.isArray(vnode.children)) {
            vnode.children.forEach(child => {
                patch(null, child, el);
            });
        }
        
        // 处理props
        if (vnode.props) {
            for (const key in vnode.props) {
                patchProps(el, key, null, vnode.props[key]);
            }
        }
        
        insert(el, container);
    }
    
    // 更新元素
    function patchElement(n1, n2) {
        const el = n2.el = n1.el;
        const oldProps = n1.props;
        const newProps = n2.props;
        
        // 更新props
        for (const key in newProps) {
            if (newProps[key] !== oldProps[key]) {
                patchProps(el, key, oldProps[key], newProps[key]);
            }
        }
        
        for (const key in oldProps) {
            if (!(key in newProps)) {
                patchProps(el, key, oldProps[key], null);
            }
        }
        
        // 更新children
        patchChildren(n1, n2, el);
    }
    
    // patch函数
    function patch(n1, n2, container, anchor) {
        if (n1 && n1.type !== n2.type) {
            unmount(n1);
            n1 = null;
        }
        
        const { type } = n2;
        
        if (typeof type === 'string') {
            if (!n1) {
                mountElement(n2, container, anchor);
            } else {
                patchElement(n1, n2);
            }
        } else if (typeof type === 'object') {
            // 组件
            if (!n1) {
                mountComponent(n2, container, anchor);
            } else {
                patchComponent(n1, n2);
            }
        }
    }
    
    // 渲染函数
    function render(vnode, container) {
        if (vnode) {
            patch(container._vnode, vnode, container);
        } else {
            if (container._vnode) {
                unmount(container._vnode);
            }
        }
        container._vnode = vnode;
    }
    
    return {
        render
    };
}

// 浏览器平台的渲染器
const renderer = createRenderer({
    createElement(tag) {
        return document.createElement(tag);
    },
    
    setElementText(el, text) {
        el.textContent = text;
    },
    
    insert(el, parent, anchor = null) {
        parent.insertBefore(el, anchor);
    },
    
    patchProps(el, key, prevValue, nextValue) {
        if (/^on/.test(key)) {
            // 事件处理
            const name = key.slice(2).toLowerCase();
            if (prevValue) {
                el.removeEventListener(name, prevValue);
            }
            if (nextValue) {
                el.addEventListener(name, nextValue);
            }
        } else if (key === 'class') {
            el.className = nextValue || '';
        } else if (key === 'style') {
            // 样式处理
            if (nextValue) {
                for (const styleName in nextValue) {
                    el.style[styleName] = nextValue[styleName];
                }
            }
            if (prevValue) {
                for (const styleName in prevValue) {
                    if (!(styleName in nextValue)) {
                        el.style[styleName] = '';
                    }
                }
            }
        } else {
            // 普通属性
            if (nextValue === null || nextValue === false) {
                el.removeAttribute(key);
            } else {
                el.setAttribute(key, nextValue);
            }
        }
    }
});
```

### 5.2 组件渲染

```javascript
/**
 * 组件渲染实现
 * @author erik.zhou
 */

function mountComponent(vnode, container, anchor) {
    const componentOptions = vnode.type;
    const { render, data, setup, props: propsOption } = componentOptions;
    
    // 创建组件实例
    const instance = {
        state: null,
        props: null,
        isMounted: false,
        subTree: null
    };
    
    // 处理props
    const [props, attrs] = resolveProps(propsOption, vnode.props);
    instance.props = reactive(props);
    
    // 处理setup
    let setupState = null;
    if (setup) {
        const setupContext = { attrs, emit, slots };
        const setupResult = setup(instance.props, setupContext);
        
        if (typeof setupResult === 'function') {
            render = setupResult;
        } else {
            setupState = setupResult;
        }
    }
    
    // 创建渲染上下文
    const renderContext = new Proxy(instance, {
        get(target, key) {
            const { state, props } = target;
            
            if (state && key in state) {
                return state[key];
            } else if (key in props) {
                return props[key];
            } else if (setupState && key in setupState) {
                return setupState[key];
            }
        },
        
        set(target, key, value) {
            const { state, props } = target;
            
            if (state && key in state) {
                state[key] = value;
            } else if (key in props) {
                console.warn('不能修改props');
            } else if (setupState && key in setupState) {
                setupState[key] = value;
            }
        }
    });
    
    // 创建响应式副作用
    effect(() => {
        const subTree = render.call(renderContext, renderContext);
        
        if (!instance.isMounted) {
            patch(null, subTree, container, anchor);
            instance.isMounted = true;
        } else {
            patch(instance.subTree, subTree, container, anchor);
        }
        
        instance.subTree = subTree;
    });
    
    vnode.component = instance;
}
```

### 5.3 异步组件

```vue
<!-- components/AsyncComponent.vue -->
<!--
  异步组件示例
  @author erik.zhou
-->
<script>
import { defineAsyncComponent } from 'vue';

// 基本用法
const AsyncComp = defineAsyncComponent(() =>
    import('./HeavyComponent.vue')
);

// 高级配置
const AsyncCompWithOptions = defineAsyncComponent({
    loader: () => import('./HeavyComponent.vue'),
    
    // 加载中显示的组件
    loadingComponent: LoadingComponent,
    
    // 加载失败显示的组件
    errorComponent: ErrorComponent,
    
    // 延迟显示加载组件的时间（ms）
    delay: 200,
    
    // 超时时间（ms）
    timeout: 3000,
    
    // 是否挂起
    suspensible: false,
    
    // 错误处理
    onError(error, retry, fail, attempts) {
        if (error.message.match(/fetch/) && attempts <= 3) {
            retry();
        } else {
            fail();
        }
    }
});

export default {
    components: {
        AsyncComp,
        AsyncCompWithOptions
    }
};
</script>

<template>
    <div>
        <Suspense>
            <template #default>
                <AsyncComp />
            </template>
            
            <template #fallback>
                <div>加载中...</div>
            </template>
        </Suspense>
    </div>
</template>
```

---

## 6. 生命周期

### 6.1 生命周期流程图

```javascript
/**
 * Vue 3生命周期流程
 * @author erik.zhou
 */

/*
创建阶段:
1. setup() - 组合式API入口
2. beforeCreate - 实例初始化后
3. created - 实例创建完成

挂载阶段:
4. beforeMount - 挂载开始前
5. mounted - 挂载完成

更新阶段:
6. beforeUpdate - 数据更新前
7. updated - 数据更新后

卸载阶段:
8. beforeUnmount - 卸载开始前
9. unmounted - 卸载完成
*/
```

### 6.2 生命周期完整示例

```vue
<!-- components/LifecycleComplete.vue -->
<!--
  完整生命周期示例
  @author erik.zhou
-->
<template>
    <div ref="rootRef">
        <h2>{{ title }}</h2>
        <p>计数: {{ count }}</p>
        <button @click="count++">增加</button>
    </div>
</template>

<script setup>
import {
    ref,
    onBeforeMount,
    onMounted,
    onBeforeUpdate,
    onUpdated,
    onBeforeUnmount,
    onUnmounted,
    onActivated,
    onDeactivated,
    onErrorCaptured,
    onRenderTracked,
    onRenderTriggered
} from 'vue';

const title = ref('生命周期演示');
const count = ref(0);
const rootRef = ref(null);

console.log('setup执行');

// 挂载前
onBeforeMount(() => {
    console.log('onBeforeMount: 组件即将挂载');
    console.log('DOM可访问:', rootRef.value); // null
});

// 挂载后
onMounted(() => {
    console.log('onMounted: 组件已挂载');
    console.log('DOM可访问:', rootRef.value); // HTMLElement
    
    // 适合的操作:
    // - 访问DOM
    // - 发起API请求
    // - 设置定时器
    // - 添加事件监听器
});

// 更新前
onBeforeUpdate(() => {
    console.log('onBeforeUpdate: 组件即将更新');
    console.log('更新前的count:', count.value);
});

// 更新后
onUpdated(() => {
    console.log('onUpdated: 组件已更新');
    console.log('更新后的count:', count.value);
});

// 卸载前
onBeforeUnmount(() => {
    console.log('onBeforeUnmount: 组件即将卸载');
    
    // 适合的操作:
    // - 清理定时器
    // - 移除事件监听器
    // - 取消API请求
});

// 卸载后
onUnmounted(() => {
    console.log('onUnmounted: 组件已卸载');
});

// KeepAlive激活
onActivated(() => {
    console.log('onActivated: 组件被激活');
});

// KeepAlive停用
onDeactivated(() => {
    console.log('onDeactivated: 组件被停用');
});

// 错误捕获
onErrorCaptured((err, instance, info) => {
    console.error('onErrorCaptured:', err, info);
    return false; // 阻止错误继续传播
});

// 调试钩子 - 追踪响应式依赖
onRenderTracked((event) => {
    console.log('onRenderTracked:', event);
});

// 调试钩子 - 响应式依赖触发
onRenderTriggered((event) => {
    console.log('onRenderTriggered:', event);
});
</script>
```

### 6.3 生命周期最佳实践

```vue
<!-- components/LifecycleBestPractices.vue -->
<!--
  生命周期最佳实践
  @author erik.zhou
-->
<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

// 1. 数据获取
const data = ref(null);
const loading = ref(false);

onMounted(async () => {
    loading.value = true;
    try {
        const response = await fetch('/api/data');
        data.value = await response.json();
    } catch (error) {
        console.error('数据获取失败:', error);
    } finally {
        loading.value = false;
    }
});

// 2. 定时器管理
let timer = null;

onMounted(() => {
    timer = setInterval(() => {
        console.log('定时任务');
    }, 1000);
});

onBeforeUnmount(() => {
    if (timer) {
        clearInterval(timer);
        timer = null;
    }
});

// 3. 事件监听器管理
const handleResize = () => {
    console.log('窗口大小改变');
};

onMounted(() => {
    window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
    window.removeEventListener('resize', handleResize);
});

// 4. 第三方库初始化
let chart = null;

onMounted(() => {
    // 初始化图表库
    chart = new Chart(document.getElementById('chart'), {
        // 配置...
    });
});

onBeforeUnmount(() => {
    // 销毁图表实例
    if (chart) {
        chart.destroy();
        chart = null;
    }
});
</script>
```



---

## 7. 组件通信

### 7.1 Props和Emits

```vue
<!-- ParentComponent.vue -->
<!--
  Props和Emits通信
  @author erik.zhou
-->
<template>
    <div>
        <h2>父组件</h2>
        <p>父组件计数: {{ parentCount }}</p>
        
        <ChildComponent
            :count="parentCount"
            :user="user"
            @update:count="handleUpdateCount"
            @custom-event="handleCustomEvent"
        />
    </div>
</template>

<script setup>
import { ref, reactive } from 'vue';
import ChildComponent from './ChildComponent.vue';

const parentCount = ref(0);
const user = reactive({
    name: 'Vue User',
    age: 25
});

const handleUpdateCount = (newCount) => {
    parentCount.value = newCount;
};

const handleCustomEvent = (data) => {
    console.log('接收到自定义事件:', data);
};
</script>
```

```vue
<!-- ChildComponent.vue -->
<!--
  子组件接收Props和触发Emits
  @author erik.zhou
-->
<template>
    <div>
        <h3>子组件</h3>
        <p>接收的计数: {{ count }}</p>
        <p>用户: {{ user.name }} ({{ user.age }}岁)</p>
        
        <button @click="increment">增加</button>
        <button @click="sendCustomEvent">发送自定义事件</button>
    </div>
</template>

<script setup>
import { computed } from 'vue';

// 定义Props
const props = defineProps({
    count: {
        type: Number,
        required: true,
        default: 0
    },
    user: {
        type: Object,
        required: true,
        validator: (value) => {
            return value.name && value.age;
        }
    }
});

// 定义Emits
const emit = defineEmits(['update:count', 'custom-event']);

const increment = () => {
    emit('update:count', props.count + 1);
};

const sendCustomEvent = () => {
    emit('custom-event', {
        message: 'Hello from child',
        timestamp: Date.now()
    });
};
</script>
```

### 7.2 v-model双向绑定

```vue
<!-- CustomInput.vue -->
<!--
  自定义v-model组件
  @author erik.zhou
-->
<template>
    <input
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        :placeholder="placeholder"
    />
</template>

<script setup>
defineProps({
    modelValue: {
        type: String,
        required: true
    },
    placeholder: {
        type: String,
        default: ''
    }
});

defineEmits(['update:modelValue']);
</script>
```

```vue
<!-- 使用自定义v-model -->
<template>
    <div>
        <CustomInput v-model="text" placeholder="请输入..." />
        <p>输入的内容: {{ text }}</p>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import CustomInput from './CustomInput.vue';

const text = ref('');
</script>
```

```vue
<!-- MultipleVModel.vue -->
<!--
  多个v-model
  @author erik.zhou
-->
<template>
    <div>
        <input
            :value="firstName"
            @input="$emit('update:firstName', $event.target.value)"
            placeholder="名"
        />
        
        <input
            :value="lastName"
            @input="$emit('update:lastName', $event.target.value)"
            placeholder="姓"
        />
    </div>
</template>

<script setup>
defineProps({
    firstName: String,
    lastName: String
});

defineEmits(['update:firstName', 'update:lastName']);
</script>

<!-- 使用 -->
<template>
    <MultipleVModel
        v-model:first-name="first"
        v-model:last-name="last"
    />
</template>
```

### 7.3 Provide/Inject

```vue
<!-- GrandParent.vue -->
<!--
  祖先组件提供数据
  @author erik.zhou
-->
<template>
    <div>
        <h2>祖先组件</h2>
        <p>主题: {{ theme }}</p>
        <button @click="toggleTheme">切换主题</button>
        
        <Parent />
    </div>
</template>

<script setup>
import { ref, provide, readonly } from 'vue';
import Parent from './Parent.vue';

const theme = ref('light');

const toggleTheme = () => {
    theme.value = theme.value === 'light' ? 'dark' : 'light';
};

// 提供响应式数据（只读）
provide('theme', readonly(theme));

// 提供方法
provide('toggleTheme', toggleTheme);

// 提供Symbol key（避免命名冲突）
const ThemeSymbol = Symbol('theme');
provide(ThemeSymbol, theme);
</script>
```

```vue
<!-- GrandChild.vue -->
<!--
  后代组件注入数据
  @author erik.zhou
-->
<template>
    <div :class="`theme-${theme}`">
        <h4>孙子组件</h4>
        <p>当前主题: {{ theme }}</p>
        <button @click="toggleTheme">切换主题</button>
    </div>
</template>

<script setup>
import { inject } from 'vue';

// 注入数据
const theme = inject('theme');
const toggleTheme = inject('toggleTheme');

// 提供默认值
const config = inject('config', {
    language: 'zh-CN',
    timezone: 'Asia/Shanghai'
});
</script>
```

### 7.4 EventBus（事件总线）

```javascript
// utils/eventBus.js
/**
 * 事件总线实现
 * @author erik.zhou
 */
import { ref } from 'vue';

class EventBus {
    constructor() {
        this.events = new Map();
    }
    
    on(event, callback) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        this.events.get(event).push(callback);
    }
    
    off(event, callback) {
        if (!this.events.has(event)) {
            return;
        }
        
        const callbacks = this.events.get(event);
        const index = callbacks.indexOf(callback);
        
        if (index > -1) {
            callbacks.splice(index, 1);
        }
    }
    
    emit(event, ...args) {
        if (!this.events.has(event)) {
            return;
        }
        
        this.events.get(event).forEach(callback => {
            callback(...args);
        });
    }
    
    once(event, callback) {
        const wrapper = (...args) => {
            callback(...args);
            this.off(event, wrapper);
        };
        this.on(event, wrapper);
    }
}

export const eventBus = new EventBus();
```

```vue
<!-- ComponentA.vue -->
<template>
    <button @click="sendMessage">发送消息</button>
</template>

<script setup>
import { eventBus } from '@/utils/eventBus';

const sendMessage = () => {
    eventBus.emit('message', {
        text: 'Hello from Component A',
        timestamp: Date.now()
    });
};
</script>
```

```vue
<!-- ComponentB.vue -->
<template>
    <div>
        <p>接收到的消息: {{ message }}</p>
    </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { eventBus } from '@/utils/eventBus';

const message = ref('');

const handleMessage = (data) => {
    message.value = data.text;
};

onMounted(() => {
    eventBus.on('message', handleMessage);
});

onBeforeUnmount(() => {
    eventBus.off('message', handleMessage);
});
</script>
```

---

## 8. 内置组件

### 8.1 Teleport传送门

```vue
<!-- components/Modal.vue -->
<!--
  Teleport传送门示例
  @author erik.zhou
-->
<template>
    <Teleport to="body">
        <div v-if="show" class="modal-overlay" @click="close">
            <div class="modal-content" @click.stop>
                <div class="modal-header">
                    <h3>{{ title }}</h3>
                    <button @click="close">×</button>
                </div>
                
                <div class="modal-body">
                    <slot></slot>
                </div>
                
                <div class="modal-footer">
                    <button @click="close">取消</button>
                    <button @click="confirm">确定</button>
                </div>
            </div>
        </div>
    </Teleport>
</template>

<script setup>
defineProps({
    show: {
        type: Boolean,
        required: true
    },
    title: {
        type: String,
        default: '提示'
    }
});

const emit = defineEmits(['close', 'confirm']);

const close = () => {
    emit('close');
};

const confirm = () => {
    emit('confirm');
};
</script>

<style scoped>
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
}

.modal-content {
    background: white;
    border-radius: 8px;
    padding: 20px;
    min-width: 400px;
    max-width: 90%;
}
</style>
```

### 8.2 Suspense异步组件

```vue
<!-- components/AsyncDataComponent.vue -->
<!--
  Suspense异步组件示例
  @author erik.zhou
-->
<template>
    <div>
        <h3>异步数据</h3>
        <ul>
            <li v-for="item in data" :key="item.id">
                {{ item.name }}
            </li>
        </ul>
    </div>
</template>

<script setup>
import { ref } from 'vue';

// 模拟异步数据获取
const data = ref([]);

const fetchData = async () => {
    const response = await fetch('/api/data');
    return response.json();
};

// 使用顶层await
data.value = await fetchData();
</script>
```

```vue
<!-- ParentWithSuspense.vue -->
<template>
    <div>
        <h2>Suspense示例</h2>
        
        <Suspense>
            <!-- 默认插槽：异步组件 -->
            <template #default>
                <AsyncDataComponent />
            </template>
            
            <!-- 后备插槽：加载状态 -->
            <template #fallback>
                <div class="loading">
                    <div class="spinner"></div>
                    <p>加载中...</p>
                </div>
            </template>
        </Suspense>
        
        <!-- 错误边界 -->
        <Suspense @error="handleError">
            <template #default>
                <AsyncDataComponent />
            </template>
            
            <template #fallback>
                <div>加载中...</div>
            </template>
        </Suspense>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import AsyncDataComponent from './AsyncDataComponent.vue';

const handleError = (error) => {
    console.error('异步组件加载失败:', error);
};
</script>
```

### 8.3 KeepAlive缓存组件

```vue
<!-- components/TabsWithKeepAlive.vue -->
<!--
  KeepAlive缓存组件示例
  @author erik.zhou
-->
<template>
    <div>
        <div class="tabs">
            <button
                v-for="tab in tabs"
                :key="tab.name"
                :class="{ active: currentTab === tab.name }"
                @click="currentTab = tab.name"
            >
                {{ tab.label }}
            </button>
        </div>
        
        <!-- 缓存组件状态 -->
        <KeepAlive :include="['TabA', 'TabB']" :exclude="['TabC']" :max="3">
            <component :is="currentComponent"></component>
        </KeepAlive>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import TabA from './TabA.vue';
import TabB from './TabB.vue';
import TabC from './TabC.vue';

const tabs = [
    { name: 'a', label: '标签A', component: TabA },
    { name: 'b', label: '标签B', component: TabB },
    { name: 'c', label: '标签C', component: TabC }
];

const currentTab = ref('a');

const currentComponent = computed(() => {
    return tabs.find(tab => tab.name === currentTab.value)?.component;
});
</script>
```

```vue
<!-- TabA.vue -->
<template>
    <div>
        <h3>标签A</h3>
        <input v-model="text" placeholder="输入内容..." />
        <p>输入的内容: {{ text }}</p>
    </div>
</template>

<script setup>
import { ref, onActivated, onDeactivated } from 'vue';

const text = ref('');

// KeepAlive激活时调用
onActivated(() => {
    console.log('TabA被激活');
});

// KeepAlive停用时调用
onDeactivated(() => {
    console.log('TabA被停用');
});
</script>
```

### 8.4 Transition过渡动画

```vue
<!-- components/TransitionDemo.vue -->
<!--
  Transition过渡动画示例
  @author erik.zhou
-->
<template>
    <div>
        <button @click="show = !show">切换</button>
        
        <!-- 基本过渡 -->
        <Transition name="fade">
            <p v-if="show">Hello Vue 3</p>
        </Transition>
        
        <!-- 自定义过渡类名 -->
        <Transition
            enter-active-class="animate__animated animate__fadeIn"
            leave-active-class="animate__animated animate__fadeOut"
        >
            <p v-if="show">使用Animate.css</p>
        </Transition>
        
        <!-- JavaScript钩子 -->
        <Transition
            @before-enter="onBeforeEnter"
            @enter="onEnter"
            @after-enter="onAfterEnter"
            @before-leave="onBeforeLeave"
            @leave="onLeave"
            @after-leave="onAfterLeave"
        >
            <p v-if="show">JavaScript动画</p>
        </Transition>
    </div>
</template>

<script setup>
import { ref } from 'vue';

const show = ref(true);

// JavaScript钩子函数
const onBeforeEnter = (el) => {
    el.style.opacity = 0;
};

const onEnter = (el, done) => {
    el.offsetHeight; // 触发重排
    el.style.transition = 'opacity 0.3s';
    el.style.opacity = 1;
    done();
};

const onAfterEnter = (el) => {
    console.log('进入动画完成');
};

const onBeforeLeave = (el) => {
    el.style.opacity = 1;
};

const onLeave = (el, done) => {
    el.style.transition = 'opacity 0.3s';
    el.style.opacity = 0;
    setTimeout(done, 300);
};

const onAfterLeave = (el) => {
    console.log('离开动画完成');
};
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}
</style>
```

```vue
<!-- TransitionGroup列表过渡 -->
<template>
    <div>
        <button @click="addItem">添加</button>
        <button @click="removeItem">删除</button>
        
        <TransitionGroup name="list" tag="ul">
            <li v-for="item in items" :key="item.id">
                {{ item.text }}
            </li>
        </TransitionGroup>
    </div>
</template>

<script setup>
import { ref } from 'vue';

const items = ref([
    { id: 1, text: '项目1' },
    { id: 2, text: '项目2' },
    { id: 3, text: '项目3' }
]);

let nextId = 4;

const addItem = () => {
    items.value.push({
        id: nextId++,
        text: `项目${nextId - 1}`
    });
};

const removeItem = () => {
    items.value.pop();
};
</script>

<style scoped>
.list-enter-active,
.list-leave-active {
    transition: all 0.3s;
}

.list-enter-from {
    opacity: 0;
    transform: translateX(30px);
}

.list-leave-to {
    opacity: 0;
    transform: translateX(-30px);
}

.list-move {
    transition: transform 0.3s;
}
</style>
```

---

## 9. 高级特性

### 9.1 自定义指令

```javascript
// directives/vFocus.js
/**
 * 自动聚焦指令
 * @author erik.zhou
 */
export const vFocus = {
    mounted(el) {
        el.focus();
    }
};
```

```javascript
// directives/vClickOutside.js
/**
 * 点击外部指令
 * @author erik.zhou
 */
export const vClickOutside = {
    mounted(el, binding) {
        el.clickOutsideEvent = (event) => {
            if (!(el === event.target || el.contains(event.target))) {
                binding.value(event);
            }
        };
        document.addEventListener('click', el.clickOutsideEvent);
    },
    
    unmounted(el) {
        document.removeEventListener('click', el.clickOutsideEvent);
    }
};
```

```javascript
// directives/vLazyLoad.js
/**
 * 图片懒加载指令
 * @author erik.zhou
 */
export const vLazyLoad = {
    mounted(el, binding) {
        const observer = new IntersectionObserver(
            (entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        el.src = binding.value;
                        observer.unobserve(el);
                    }
                });
            },
            { threshold: 0.1 }
        );
        
        observer.observe(el);
        el._lazyLoadObserver = observer;
    },
    
    unmounted(el) {
        if (el._lazyLoadObserver) {
            el._lazyLoadObserver.disconnect();
        }
    }
};
```

```vue
<!-- 使用自定义指令 -->
<template>
    <div>
        <!-- 自动聚焦 -->
        <input v-focus />
        
        <!-- 点击外部 -->
        <div v-click-outside="handleClickOutside">
            点击外部关闭
        </div>
        
        <!-- 图片懒加载 -->
        <img v-lazy-load="imageUrl" alt="懒加载图片" />
    </div>
</template>

<script setup>
import { vFocus } from '@/directives/vFocus';
import { vClickOutside } from '@/directives/vClickOutside';
import { vLazyLoad } from '@/directives/vLazyLoad';

const handleClickOutside = () => {
    console.log('点击了外部');
};

const imageUrl = 'https://example.com/image.jpg';
</script>
```

### 9.2 插件开发

```javascript
// plugins/myPlugin.js
/**
 * Vue插件开发示例
 * @author erik.zhou
 */
export default {
    install(app, options) {
        // 1. 添加全局方法或属性
        app.config.globalProperties.$myMethod = (message) => {
            console.log('全局方法:', message);
        };
        
        // 2. 添加全局指令
        app.directive('my-directive', {
            mounted(el, binding) {
                el.textContent = binding.value;
            }
        });
        
        // 3. 添加全局组件
        app.component('MyComponent', {
            template: '<div>全局组件</div>'
        });
        
        // 4. 提供注入
        app.provide('myPlugin', {
            message: options.message || 'Hello Plugin'
        });
        
        // 5. 添加全局混入
        app.mixin({
            created() {
                console.log('全局混入的created钩子');
            }
        });
    }
};
```

```javascript
// main.js
import { createApp } from 'vue';
import App from './App.vue';
import myPlugin from './plugins/myPlugin';

const app = createApp(App);

// 使用插件
app.use(myPlugin, {
    message: 'Custom Message'
});

app.mount('#app');
```

### 9.3 渲染函数

```javascript
// components/RenderFunctionComponent.js
/**
 * 渲染函数组件
 * @author erik.zhou
 */
import { h, ref } from 'vue';

export default {
    name: 'RenderFunctionComponent',
    
    props: {
        level: {
            type: Number,
            required: true
        }
    },
    
    setup(props, { slots }) {
        const count = ref(0);
        
        return () => {
            // 动态标签名
            const tag = `h${props.level}`;
            
            return h(
                tag,
                {
                    class: 'heading',
                    onClick: () => count.value++
                },
                [
                    slots.default ? slots.default() : '默认标题',
                    h('span', ` (点击次数: ${count.value})`)
                ]
            );
        };
    }
};
```

```vue
<!-- 使用渲染函数组件 -->
<template>
    <RenderFunctionComponent :level="2">
        自定义标题
    </RenderFunctionComponent>
</template>

<script setup>
import RenderFunctionComponent from './RenderFunctionComponent';
</script>
```

### 9.4 JSX支持

```jsx
// components/JSXComponent.jsx
/**
 * JSX组件示例
 * @author erik.zhou
 */
import { ref, computed } from 'vue';

export default {
    name: 'JSXComponent',
    
    props: {
        title: String
    },
    
    setup(props) {
        const count = ref(0);
        const double = computed(() => count.value * 2);
        
        const increment = () => {
            count.value++;
        };
        
        return () => (
            <div class="jsx-component">
                <h2>{props.title}</h2>
                <p>计数: {count.value}</p>
                <p>双倍: {double.value}</p>
                <button onClick={increment}>增加</button>
            </div>
        );
    }
};
```



---

## 10. 最佳实践

### 10.1 组件设计原则

```vue
<!-- components/BestPracticeComponent.vue -->
<!--
  组件设计最佳实践
  @author erik.zhou
-->
<template>
    <div class="best-practice-component">
        <!-- 1. 单一职责原则 -->
        <UserInfo :user="user" />
        <UserActions @edit="handleEdit" @delete="handleDelete" />
        
        <!-- 2. 合理的Props设计 -->
        <DataTable
            :data="tableData"
            :columns="columns"
            :loading="loading"
            @row-click="handleRowClick"
        />
        
        <!-- 3. 插槽的合理使用 -->
        <Card>
            <template #header>
                <h3>卡片标题</h3>
            </template>
            
            <template #default>
                <p>卡片内容</p>
            </template>
            
            <template #footer>
                <button>确定</button>
            </template>
        </Card>
    </div>
</template>

<script setup>
import { ref, reactive } from 'vue';

// 1. 使用组合式函数抽离逻辑
import { useUser } from '@/composables/useUser';
import { useTable } from '@/composables/useTable';

// 2. 清晰的变量命名
const { user, loading: userLoading, fetchUser } = useUser();
const { tableData, columns, loading: tableLoading, fetchData } = useTable();

// 3. 事件处理函数命名规范
const handleEdit = (userId) => {
    console.log('编辑用户:', userId);
};

const handleDelete = (userId) => {
    console.log('删除用户:', userId);
};

const handleRowClick = (row) => {
    console.log('点击行:', row);
};
</script>

<style scoped>
/* 4. 使用scoped样式避免污染 */
.best-practice-component {
    padding: 20px;
}
</style>
```

### 10.2 性能优化技巧

```vue
<!-- components/PerformanceOptimization.vue -->
<!--
  性能优化最佳实践
  @author erik.zhou
-->
<template>
    <div>
        <!-- 1. 使用v-show代替v-if（频繁切换） -->
        <div v-show="isVisible">频繁切换的内容</div>
        
        <!-- 2. 使用v-once渲染静态内容 -->
        <div v-once>
            <h2>{{ staticTitle }}</h2>
            <p>这是静态内容，只渲染一次</p>
        </div>
        
        <!-- 3. 使用v-memo缓存子树 -->
        <div v-memo="[user.id, user.name]">
            <UserCard :user="user" />
        </div>
        
        <!-- 4. 列表使用key -->
        <ul>
            <li v-for="item in items" :key="item.id">
                {{ item.name }}
            </li>
        </ul>
        
        <!-- 5. 异步组件 -->
        <Suspense>
            <template #default>
                <AsyncHeavyComponent />
            </template>
            <template #fallback>
                <div>加载中...</div>
            </template>
        </Suspense>
    </div>
</template>

<script setup>
import { ref, shallowRef, shallowReactive, defineAsyncComponent } from 'vue';

// 6. 使用shallowRef/shallowReactive（大型数据）
const largeData = shallowRef({
    // 大量数据...
});

const largeList = shallowReactive([
    // 大量列表项...
]);

// 7. 异步组件
const AsyncHeavyComponent = defineAsyncComponent(() =>
    import('./HeavyComponent.vue')
);

// 8. 计算属性缓存
const expensiveComputed = computed(() => {
    // 昂贵的计算
    return items.value.reduce((sum, item) => sum + item.value, 0);
});
</script>
```

### 10.3 代码组织规范

```
src/
├── components/          # 组件目录
│   ├── common/         # 通用组件
│   │   ├── Button.vue
│   │   └── Input.vue
│   ├── layout/         # 布局组件
│   │   ├── Header.vue
│   │   └── Footer.vue
│   └── business/       # 业务组件
│       └── UserCard.vue
├── composables/        # 组合式函数
│   ├── useUser.js
│   └── useFetch.js
├── directives/         # 自定义指令
│   ├── vFocus.js
│   └── vLazyLoad.js
├── utils/             # 工具函数
│   ├── format.js
│   └── validate.js
├── stores/            # 状态管理
│   └── user.js
├── router/            # 路由配置
│   └── index.js
├── views/             # 页面组件
│   ├── Home.vue
│   └── About.vue
└── App.vue
```

### 10.4 TypeScript最佳实践

```vue
<!-- components/TypeScriptComponent.vue -->
<!--
  TypeScript最佳实践
  @author erik.zhou
-->
<template>
    <div>
        <h2>{{ user.name }}</h2>
        <p>年龄: {{ user.age }}</p>
        <button @click="updateUser">更新用户</button>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, type Ref, type ComputedRef } from 'vue';

// 1. 定义接口
interface User {
    id: number;
    name: string;
    age: number;
    email?: string;
}

interface UserResponse {
    code: number;
    data: User;
    message: string;
}

// 2. Props类型定义
interface Props {
    userId: number;
    title?: string;
}

const props = withDefaults(defineProps<Props>(), {
    title: '默认标题'
});

// 3. Emits类型定义
interface Emits {
    (e: 'update', user: User): void;
    (e: 'delete', id: number): void;
}

const emit = defineEmits<Emits>();

// 4. 响应式数据类型
const user: Ref<User> = ref({
    id: 1,
    name: 'Vue User',
    age: 25
});

const users: Ref<User[]> = ref([]);

// 5. 计算属性类型
const userInfo: ComputedRef<string> = computed(() => {
    return `${user.value.name} (${user.value.age}岁)`;
});

// 6. 函数类型定义
const fetchUser = async (id: number): Promise<User> => {
    const response = await fetch(`/api/users/${id}`);
    const data: UserResponse = await response.json();
    return data.data;
};

const updateUser = (): void => {
    user.value.age++;
    emit('update', user.value);
};

// 7. 组合式函数类型
interface UseUserReturn {
    user: Ref<User | null>;
    loading: Ref<boolean>;
    error: Ref<string | null>;
    fetchUser: (id: number) => Promise<void>;
}

function useUser(): UseUserReturn {
    const user = ref<User | null>(null);
    const loading = ref(false);
    const error = ref<string | null>(null);
    
    const fetchUser = async (id: number): Promise<void> => {
        loading.value = true;
        error.value = null;
        
        try {
            const response = await fetch(`/api/users/${id}`);
            const data: UserResponse = await response.json();
            user.value = data.data;
        } catch (err) {
            error.value = err instanceof Error ? err.message : '未知错误';
        } finally {
            loading.value = false;
        }
    };
    
    return {
        user,
        loading,
        error,
        fetchUser
    };
}
</script>
```

### 10.5 错误处理

```vue
<!-- components/ErrorHandling.vue -->
<!--
  错误处理最佳实践
  @author erik.zhou
-->
<template>
    <div>
        <div v-if="error" class="error-message">
            {{ error }}
        </div>
        
        <div v-else-if="loading">
            加载中...
        </div>
        
        <div v-else>
            <UserList :users="users" />
        </div>
    </div>
</template>

<script setup>
import { ref, onErrorCaptured } from 'vue';

const users = ref([]);
const loading = ref(false);
const error = ref(null);

// 1. 异步错误处理
const fetchUsers = async () => {
    loading.value = true;
    error.value = null;
    
    try {
        const response = await fetch('/api/users');
        
        if (!response.ok) {
            throw new Error(`HTTP错误: ${response.status}`);
        }
        
        users.value = await response.json();
    } catch (err) {
        error.value = err.message || '加载失败';
        console.error('获取用户列表失败:', err);
    } finally {
        loading.value = false;
    }
};

// 2. 捕获子组件错误
onErrorCaptured((err, instance, info) => {
    console.error('捕获到子组件错误:', err);
    console.error('错误来源组件:', instance);
    console.error('错误信息:', info);
    
    error.value = '组件渲染失败';
    
    // 返回false阻止错误继续传播
    return false;
});

// 3. 全局错误处理（main.js）
// app.config.errorHandler = (err, instance, info) => {
//     console.error('全局错误:', err, info);
// };
</script>
```

### 10.6 测试最佳实践

```javascript
// __tests__/Counter.spec.js
/**
 * 组件测试示例
 * @author erik.zhou
 */
import { mount } from '@vue/test-utils';
import { describe, it, expect } from 'vitest';
import Counter from '@/components/Counter.vue';

describe('Counter组件', () => {
    it('应该正确渲染初始计数', () => {
        const wrapper = mount(Counter, {
            props: {
                initialCount: 5
            }
        });
        
        expect(wrapper.text()).toContain('5');
    });
    
    it('点击按钮应该增加计数', async () => {
        const wrapper = mount(Counter);
        const button = wrapper.find('button');
        
        await button.trigger('click');
        
        expect(wrapper.text()).toContain('1');
    });
    
    it('应该触发update事件', async () => {
        const wrapper = mount(Counter);
        const button = wrapper.find('button');
        
        await button.trigger('click');
        
        expect(wrapper.emitted()).toHaveProperty('update');
        expect(wrapper.emitted('update')[0]).toEqual([1]);
    });
});
```

```javascript
// __tests__/useCounter.spec.js
/**
 * 组合式函数测试
 * @author erik.zhou
 */
import { describe, it, expect } from 'vitest';
import { useCounter } from '@/composables/useCounter';

describe('useCounter', () => {
    it('应该返回初始值', () => {
        const { count } = useCounter(10);
        expect(count.value).toBe(10);
    });
    
    it('increment应该增加计数', () => {
        const { count, increment } = useCounter(0);
        
        increment();
        expect(count.value).toBe(1);
        
        increment();
        expect(count.value).toBe(2);
    });
    
    it('reset应该重置计数', () => {
        const { count, increment, reset } = useCounter(0);
        
        increment();
        increment();
        expect(count.value).toBe(2);
        
        reset();
        expect(count.value).toBe(0);
    });
});
```

---

## 总结

### 核心要点

Vue 3带来了全新的响应式系统和组合式API，主要特点包括：

1. **响应式系统**
   - 基于Proxy实现，性能更好
   - 支持深层响应式和浅层响应式
   - ref和reactive的合理使用
   - computed和watch的优化

2. **组合式API**
   - setup函数是入口
   - 逻辑复用更灵活
   - TypeScript支持更好
   - 代码组织更清晰

3. **虚拟DOM优化**
   - 静态提升减少创建开销
   - 补丁标记优化Diff算法
   - 事件缓存减少更新
   - 最长递增子序列算法

4. **编译器优化**
   - 模板编译为优化的渲染函数
   - 静态节点提升
   - 动态节点标记
   - 树结构打平

5. **组件通信**
   - Props和Emits
   - Provide/Inject
   - v-model双向绑定
   - EventBus事件总线

6. **内置组件**
   - Teleport传送门
   - Suspense异步组件
   - KeepAlive缓存
   - Transition动画

### 学习路径

1. **基础阶段**
   - 掌握响应式系统基本原理
   - 熟练使用组合式API
   - 理解生命周期钩子
   - 掌握组件通信方式

2. **进阶阶段**
   - 深入理解虚拟DOM和Diff算法
   - 掌握编译器优化原理
   - 学习自定义指令和插件
   - 掌握渲染函数和JSX

3. **高级阶段**
   - 性能优化技巧
   - 大型项目架构设计
   - 源码阅读和分析
   - 最佳实践总结

### 推荐资源

**官方文档**
- [Vue 3官方文档](https://cn.vuejs.org/)
- [Vue 3 RFC](https://github.com/vuejs/rfcs)
- [Vue 3源码](https://github.com/vuejs/core)

**学习资源**
- [Vue Mastery](https://www.vuemastery.com/)
- [Vue School](https://vueschool.io/)
- [Vue.js挑战](https://cn-vuejs-challenges.netlify.app/)

**工具库**
- [VueUse](https://vueuse.org/) - 组合式工具集
- [Pinia](https://pinia.vuejs.org/) - 状态管理
- [Vue Router](https://router.vuejs.org/) - 路由管理

---

## 附录

### A. 常用API速查

```javascript
/**
 * Vue 3常用API速查
 * @author erik.zhou
 */

// 响应式API
import {
    ref,              // 创建响应式引用
    reactive,         // 创建响应式对象
    computed,         // 计算属性
    watch,            // 侦听器
    watchEffect,      // 自动追踪依赖的侦听器
    readonly,         // 只读代理
    shallowRef,       // 浅层ref
    shallowReactive,  // 浅层reactive
    toRef,            // 创建ref
    toRefs,           // 转换为refs
    isRef,            // 判断是否为ref
    unref             // 解包ref
} from 'vue';

// 生命周期钩子
import {
    onBeforeMount,
    onMounted,
    onBeforeUpdate,
    onUpdated,
    onBeforeUnmount,
    onUnmounted,
    onActivated,
    onDeactivated,
    onErrorCaptured
} from 'vue';

// 依赖注入
import {
    provide,
    inject
} from 'vue';

// 组件API
import {
    defineComponent,
    defineAsyncComponent,
    h,
    createVNode
} from 'vue';
```

### B. 性能优化清单

```javascript
/**
 * Vue 3性能优化清单
 * @author erik.zhou
 */

const performanceChecklist = {
    响应式优化: [
        '大型数据使用shallowRef/shallowReactive',
        '避免不必要的深层响应式',
        '合理使用computed缓存计算结果',
        '使用readonly防止意外修改'
    ],
    
    组件优化: [
        '使用异步组件进行代码分割',
        '使用KeepAlive缓存组件状态',
        '合理使用v-show和v-if',
        '使用v-once渲染静态内容',
        '使用v-memo缓存子树'
    ],
    
    列表优化: [
        '使用稳定的key值',
        '虚拟列表处理大量数据',
        '避免在v-for中使用v-if',
        '使用shallowRef存储大型列表'
    ],
    
    编译优化: [
        '利用静态提升',
        '利用补丁标记',
        '事件监听器缓存',
        '使用生产环境构建'
    ]
};
```

### C. 常见问题FAQ

**Q1: ref和reactive的区别？**

A: 
- ref可以包装任何类型，reactive只能包装对象
- ref需要通过.value访问，reactive直接访问
- ref在模板中自动解包，reactive不需要
- ref更适合基本类型，reactive更适合对象

**Q2: 什么时候使用shallowRef/shallowReactive？**

A: 
- 处理大型数据结构时
- 只需要响应式根级别属性时
- 性能敏感的场景
- 与第三方库集成时

**Q3: 组合式API相比选项式API的优势？**

A: 
- 更好的逻辑复用
- 更好的TypeScript支持
- 更灵活的代码组织
- 更小的打包体积

**Q4: 如何选择watch和watchEffect？**

A: 
- watch: 需要访问旧值、惰性执行、精确控制依赖
- watchEffect: 自动追踪依赖、立即执行、简洁的代码

**Q5: Teleport的使用场景？**

A: 
- 模态框、对话框
- 通知、提示信息
- 全屏组件
- 需要脱离父组件DOM层级的场景

---

**课程总结**: 本教程深入讲解了Vue 3的核心原理，从响应式系统到虚拟DOM，从组合式API到编译器优化，提供了大量实战代码和最佳实践。掌握这些知识，能够帮助你构建高性能的Vue 3应用。

**@author erik.zhou**  
**最后更新**: 2026-03-02
