# React性能优化 - 完整教程

## 课程信息
- **课程名称**: React性能优化完整教程
- **难度级别**: 中高级
- **预计学时**: 6小时
- **核心内容**: 渲染优化、代码分割、状态管理、性能监控
- **@author**: erik.zhou

---

## 目录
1. [性能优化概述](#1-性能优化概述)
2. [渲染优化](#2-渲染优化)
3. [代码分割与懒加载](#3-代码分割与懒加载)
4. [状态管理优化](#4-状态管理优化)
5. [列表渲染优化](#5-列表渲染优化)
6. [事件处理优化](#6-事件处理优化)
7. [资源加载优化](#7-资源加载优化)
8. [性能监控](#8-性能监控)
9. [最佳实践](#9-最佳实践)
10. [实战案例](#10-实战案例)

---

## 1. 性能优化概述

### 1.1 为什么需要性能优化

```javascript
/**
 * React性能问题的常见表现
 * @author erik.zhou
 */
const performanceIssues = {
    渲染缓慢: '组件更新时页面卡顿',
    内存泄漏: '长时间运行后内存占用过高',
    首屏慢: '初始加载时间过长',
    交互延迟: '用户操作响应不及时',
    资源浪费: '不必要的重新渲染'
};

// 优化目标
const optimizationGoals = {
    减少渲染次数: '避免不必要的组件更新',
    降低渲染成本: '优化单次渲染的性能',
    加快加载速度: '减少首屏加载时间',
    提升交互体验: '确保操作流畅响应',
    合理使用资源: '避免内存泄漏和资源浪费'
};
```

### 1.2 性能优化原则

```javascript
/**
 * React性能优化原则
 * @author erik.zhou
 */

// 1. 测量优先
// 先测量，找到真正的性能瓶颈，再优化

// 2. 避免过早优化
// 不要在没有性能问题时就进行优化

// 3. 权衡取舍
// 性能优化可能增加代码复杂度，需要权衡

// 4. 持续监控
// 建立性能监控体系，及时发现问题
```

### 1.3 性能分析工具

```javascript
/**
 * React性能分析工具
 * @author erik.zhou
 */

// 1. React DevTools Profiler
// 分析组件渲染性能

// 2. Chrome DevTools Performance
// 分析整体页面性能

// 3. Lighthouse
// 综合性能评分

// 4. Web Vitals
// 核心性能指标监控
```

---

## 2. 渲染优化

### 2.1 React.memo

```javascript
// components/ExpensiveComponent.jsx
/**
 * 使用React.memo避免不必要的重渲染
 * @author erik.zhou
 */
import React, { memo } from 'react';

// 未优化版本
function ExpensiveComponentBad({ data, onClick }) {
    console.log('ExpensiveComponent渲染');
    
    return (
        <div onClick={onClick}>
            <h3>{data.title}</h3>
            <p>{data.description}</p>
        </div>
    );
}

// 优化版本 - 使用memo
const ExpensiveComponent = memo(function ExpensiveComponent({ data, onClick }) {
    console.log('ExpensiveComponent渲染');
    
    return (
        <div onClick={onClick}>
            <h3>{data.title}</h3>
            <p>{data.description}</p>
        </div>
    );
});

// 自定义比较函数
const ExpensiveComponentWithCompare = memo(
    function ExpensiveComponent({ data, onClick }) {
        return (
            <div onClick={onClick}>
                <h3>{data.title}</h3>
                <p>{data.description}</p>
            </div>
        );
    },
    (prevProps, nextProps) => {
        // 返回true表示props相同，不需要重新渲染
        return prevProps.data.id === nextProps.data.id;
    }
);

export default ExpensiveComponent;
```

### 2.2 useMemo

```javascript
// hooks/useOptimizedData.js
/**
 * 使用useMemo缓存计算结果
 * @author erik.zhou
 */
import { useMemo } from 'react';

function useOptimizedData(items, filter) {
    // 未优化版本 - 每次渲染都会重新计算
    const filteredItemsBad = items.filter(item => 
        item.name.includes(filter)
    );
    
    // 优化版本 - 只在依赖变化时重新计算
    const filteredItems = useMemo(() => {
        console.log('重新计算过滤结果');
        return items.filter(item => item.name.includes(filter));
    }, [items, filter]);
    
    // 复杂计算示例
    const statistics = useMemo(() => {
        console.log('重新计算统计数据');
        
        return {
            total: items.length,
            average: items.reduce((sum, item) => sum + item.value, 0) / items.length,
            max: Math.max(...items.map(item => item.value)),
            min: Math.min(...items.map(item => item.value))
        };
    }, [items]);
    
    return { filteredItems, statistics };
}

export default useOptimizedData;
```

```javascript
// components/DataList.jsx
/**
 * useMemo实战示例
 * @author erik.zhou
 */
import React, { useState, useMemo } from 'react';

function DataList({ data }) {
    const [sortOrder, setSortOrder] = useState('asc');
    const [filterText, setFilterText] = useState('');
    
    // 过滤数据
    const filteredData = useMemo(() => {
        console.log('执行过滤');
        return data.filter(item => 
            item.name.toLowerCase().includes(filterText.toLowerCase())
        );
    }, [data, filterText]);
    
    // 排序数据
    const sortedData = useMemo(() => {
        console.log('执行排序');
        return [...filteredData].sort((a, b) => {
            if (sortOrder === 'asc') {
                return a.value - b.value;
            }
            return b.value - a.value;
        });
    }, [filteredData, sortOrder]);
    
    return (
        <div>
            <input
                type="text"
                value={filterText}
                onChange={(e) => setFilterText(e.target.value)}
                placeholder="搜索..."
            />
            
            <button onClick={() => setSortOrder(sortOrder === 'asc' ? 'desc' : 'asc')}>
                排序: {sortOrder}
            </button>
            
            <ul>
                {sortedData.map(item => (
                    <li key={item.id}>
                        {item.name}: {item.value}
                    </li>
                ))}
            </ul>
        </div>
    );
}

export default DataList;
```

### 2.3 useCallback

```javascript
// components/ParentComponent.jsx
/**
 * 使用useCallback缓存回调函数
 * @author erik.zhou
 */
import React, { useState, useCallback, memo } from 'react';

// 子组件
const ChildComponent = memo(function ChildComponent({ onClick, data }) {
    console.log('ChildComponent渲染');
    
    return (
        <button onClick={onClick}>
            {data.name}
        </button>
    );
});

function ParentComponent() {
    const [count, setCount] = useState(0);
    const [items, setItems] = useState([
        { id: 1, name: '项目1' },
        { id: 2, name: '项目2' }
    ]);
    
    // 未优化版本 - 每次渲染都创建新函数
    const handleClickBad = (id) => {
        console.log('点击:', id);
    };
    
    // 优化版本 - 使用useCallback
    const handleClick = useCallback((id) => {
        console.log('点击:', id);
        // 如果需要使用state，添加到依赖数组
    }, []);
    
    // 带依赖的useCallback
    const handleDelete = useCallback((id) => {
        setItems(prevItems => prevItems.filter(item => item.id !== id));
    }, []); // 使用函数式更新，不需要依赖items
    
    return (
        <div>
            <p>计数: {count}</p>
            <button onClick={() => setCount(count + 1)}>增加</button>
            
            {items.map(item => (
                <ChildComponent
                    key={item.id}
                    data={item}
                    onClick={() => handleClick(item.id)}
                />
            ))}
        </div>
    );
}

export default ParentComponent;
```

### 2.4 避免内联对象和数组

```javascript
// components/OptimizedComponent.jsx
/**
 * 避免内联对象和数组导致的重渲染
 * @author erik.zhou
 */
import React, { memo, useMemo } from 'react';

const ChildComponent = memo(function ChildComponent({ style, items }) {
    console.log('ChildComponent渲染');
    return <div style={style}>{items.length}</div>;
});

// 不好的做法
function BadParent() {
    return (
        <div>
            {/* 每次渲染都创建新对象，导致子组件重渲染 */}
            <ChildComponent 
                style={{ color: 'red', fontSize: '16px' }}
                items={[1, 2, 3]}
            />
        </div>
    );
}

// 好的做法1：提取到组件外部
const STYLE = { color: 'red', fontSize: '16px' };
const ITEMS = [1, 2, 3];

function GoodParent1() {
    return (
        <div>
            <ChildComponent style={STYLE} items={ITEMS} />
        </div>
    );
}

// 好的做法2：使用useMemo
function GoodParent2() {
    const style = useMemo(() => ({ 
        color: 'red', 
        fontSize: '16px' 
    }), []);
    
    const items = useMemo(() => [1, 2, 3], []);
    
    return (
        <div>
            <ChildComponent style={style} items={items} />
        </div>
    );
}

export { BadParent, GoodParent1, GoodParent2 };
```

---

## 3. 代码分割与懒加载

### 3.1 React.lazy和Suspense

```javascript
// App.jsx
/**
 * 使用React.lazy进行代码分割
 * @author erik.zhou
 */
import React, { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// 懒加载组件
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

// 加载组件
function LoadingSpinner() {
    return (
        <div className="loading-spinner">
            <div className="spinner"></div>
            <p>加载中...</p>
        </div>
    );
}

function App() {
    return (
        <BrowserRouter>
            <Suspense fallback={<LoadingSpinner />}>
                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="/about" element={<About />} />
                    <Route path="/dashboard" element={<Dashboard />} />
                </Routes>
            </Suspense>
        </BrowserRouter>
    );
}

export default App;
```

### 3.2 动态导入

```javascript
// components/DynamicComponent.jsx
/**
 * 动态导入组件
 * @author erik.zhou
 */
import React, { useState } from 'react';

function DynamicComponent() {
    const [Component, setComponent] = useState(null);
    const [loading, setLoading] = useState(false);
    
    const loadComponent = async (componentName) => {
        setLoading(true);
        
        try {
            // 动态导入
            const module = await import(`./components/${componentName}`);
            setComponent(() => module.default);
        } catch (error) {
            console.error('加载组件失败:', error);
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <div>
            <button onClick={() => loadComponent('Chart')}>
                加载图表组件
            </button>
            <button onClick={() => loadComponent('Editor')}>
                加载编辑器组件
            </button>
            
            {loading && <div>加载中...</div>}
            {Component && <Component />}
        </div>
    );
}

export default DynamicComponent;
```

### 3.3 路由级代码分割

```javascript
// router/index.jsx
/**
 * 路由级代码分割
 * @author erik.zhou
 */
import React, { lazy, Suspense } from 'react';
import { createBrowserRouter } from 'react-router-dom';

// 懒加载页面组件
const HomePage = lazy(() => import('@/pages/Home'));
const ProductsPage = lazy(() => import('@/pages/Products'));
const ProductDetailPage = lazy(() => import('@/pages/ProductDetail'));
const CartPage = lazy(() => import('@/pages/Cart'));
const CheckoutPage = lazy(() => import('@/pages/Checkout'));

// 加载组件
const PageLoader = () => (
    <div className="page-loader">
        <div className="spinner"></div>
    </div>
);

// 路由配置
const router = createBrowserRouter([
    {
        path: '/',
        element: (
            <Suspense fallback={<PageLoader />}>
                <HomePage />
            </Suspense>
        )
    },
    {
        path: '/products',
        element: (
            <Suspense fallback={<PageLoader />}>
                <ProductsPage />
            </Suspense>
        )
    },
    {
        path: '/products/:id',
        element: (
            <Suspense fallback={<PageLoader />}>
                <ProductDetailPage />
            </Suspense>
        )
    },
    {
        path: '/cart',
        element: (
            <Suspense fallback={<PageLoader />}>
                <CartPage />
            </Suspense>
        )
    },
    {
        path: '/checkout',
        element: (
            <Suspense fallback={<PageLoader />}>
                <CheckoutPage />
            </Suspense>
        )
    }
]);

export default router;
```

### 3.4 组件级代码分割

```javascript
// components/HeavyComponent.jsx
/**
 * 组件级代码分割
 * @author erik.zhou
 */
import React, { lazy, Suspense, useState } from 'react';

// 懒加载重型组件
const Chart = lazy(() => import('./Chart'));
const Editor = lazy(() => import('./Editor'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));

function HeavyComponent() {
    const [showChart, setShowChart] = useState(false);
    const [showEditor, setShowEditor] = useState(false);
    const [showVideo, setShowVideo] = useState(false);
    
    return (
        <div>
            <h2>重型组件示例</h2>
            
            <button onClick={() => setShowChart(!showChart)}>
                {showChart ? '隐藏' : '显示'}图表
            </button>
            
            {showChart && (
                <Suspense fallback={<div>加载图表中...</div>}>
                    <Chart />
                </Suspense>
            )}
            
            <button onClick={() => setShowEditor(!showEditor)}>
                {showEditor ? '隐藏' : '显示'}编辑器
            </button>
            
            {showEditor && (
                <Suspense fallback={<div>加载编辑器中...</div>}>
                    <Editor />
                </Suspense>
            )}
            
            <button onClick={() => setShowVideo(!showVideo)}>
                {showVideo ? '隐藏' : '显示'}视频
            </button>
            
            {showVideo && (
                <Suspense fallback={<div>加载视频播放器中...</div>}>
                    <VideoPlayer />
                </Suspense>
            )}
        </div>
    );
}

export default HeavyComponent;
```

---

## 4. 状态管理优化

### 4.1 状态拆分

```javascript
// components/UserProfile.jsx
/**
 * 状态拆分优化
 * @author erik.zhou
 */
import React, { useState } from 'react';

// 不好的做法 - 所有状态放在一起
function BadUserProfile() {
    const [state, setState] = useState({
        name: '',
        email: '',
        age: 0,
        address: '',
        phone: '',
        bio: ''
    });
    
    // 任何字段更新都会导致整个组件重渲染
    const updateName = (name) => {
        setState({ ...state, name });
    };
    
    return <div>{/* ... */}</div>;
}

// 好的做法 - 拆分独立状态
function GoodUserProfile() {
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');
    const [age, setAge] = useState(0);
    const [address, setAddress] = useState('');
    const [phone, setPhone] = useState('');
    const [bio, setBio] = useState('');
    
    // 只更新相关状态，减少不必要的重渲染
    return <div>{/* ... */}</div>;
}

export { BadUserProfile, GoodUserProfile };
```

### 4.2 状态下沉

```javascript
// components/StateColocation.jsx
/**
 * 状态下沉（State Colocation）
 * @author erik.zhou
 */
import React, { useState } from 'react';

// 不好的做法 - 状态提升过高
function BadParent() {
    const [inputValue, setInputValue] = useState('');
    
    return (
        <div>
            <ExpensiveComponent1 />
            <ExpensiveComponent2 />
            <input 
                value={inputValue} 
                onChange={(e) => setInputValue(e.target.value)} 
            />
            <ExpensiveComponent3 />
        </div>
    );
}

// 好的做法 - 状态下沉到需要的组件
function GoodParent() {
    return (
        <div>
            <ExpensiveComponent1 />
            <ExpensiveComponent2 />
            <InputComponent />
            <ExpensiveComponent3 />
        </div>
    );
}

function InputComponent() {
    const [inputValue, setInputValue] = useState('');
    
    return (
        <input 
            value={inputValue} 
            onChange={(e) => setInputValue(e.target.value)} 
        />
    );
}

// 模拟昂贵组件
function ExpensiveComponent1() {
    console.log('ExpensiveComponent1渲染');
    return <div>组件1</div>;
}

function ExpensiveComponent2() {
    console.log('ExpensiveComponent2渲染');
    return <div>组件2</div>;
}

function ExpensiveComponent3() {
    console.log('ExpensiveComponent3渲染');
    return <div>组件3</div>;
}

export { BadParent, GoodParent };
```

### 4.3 Context优化

```javascript
// context/OptimizedContext.jsx
/**
 * Context性能优化
 * @author erik.zhou
 */
import React, { createContext, useContext, useState, useMemo } from 'react';

// 不好的做法 - 单一Context包含所有状态
const BadContext = createContext();

function BadProvider({ children }) {
    const [user, setUser] = useState(null);
    const [theme, setTheme] = useState('light');
    const [language, setLanguage] = useState('zh-CN');
    
    // 任何状态变化都会导致所有消费者重渲染
    const value = { user, setUser, theme, setTheme, language, setLanguage };
    
    return (
        <BadContext.Provider value={value}>
            {children}
        </BadContext.Provider>
    );
}

// 好的做法1 - 拆分Context
const UserContext = createContext();
const ThemeContext = createContext();
const LanguageContext = createContext();

function GoodProvider1({ children }) {
    const [user, setUser] = useState(null);
    const [theme, setTheme] = useState('light');
    const [language, setLanguage] = useState('zh-CN');
    
    return (
        <UserContext.Provider value={{ user, setUser }}>
            <ThemeContext.Provider value={{ theme, setTheme }}>
                <LanguageContext.Provider value={{ language, setLanguage }}>
                    {children}
                </LanguageContext.Provider>
            </ThemeContext.Provider>
        </UserContext.Provider>
    );
}

// 好的做法2 - 使用useMemo缓存value
function GoodProvider2({ children }) {
    const [user, setUser] = useState(null);
    const [theme, setTheme] = useState('light');
    
    const userValue = useMemo(() => ({ user, setUser }), [user]);
    const themeValue = useMemo(() => ({ theme, setTheme }), [theme]);
    
    return (
        <UserContext.Provider value={userValue}>
            <ThemeContext.Provider value={themeValue}>
                {children}
            </ThemeContext.Provider>
        </UserContext.Provider>
    );
}

export { BadProvider, GoodProvider1, GoodProvider2, UserContext, ThemeContext };
```

---

## 5. 列表渲染优化

### 5.1 虚拟列表

```javascript
// components/VirtualList.jsx
/**
 * 虚拟列表实现
 * @author erik.zhou
 */
import React, { useState, useRef, useEffect } from 'react';

function VirtualList({ items, itemHeight, containerHeight }) {
    const [scrollTop, setScrollTop] = useState(0);
    const containerRef = useRef(null);
    
    // 计算可见范围
    const visibleCount = Math.ceil(containerHeight / itemHeight);
    const startIndex = Math.floor(scrollTop / itemHeight);
    const endIndex = Math.min(startIndex + visibleCount + 1, items.length);
    
    // 可见项
    const visibleItems = items.slice(startIndex, endIndex);
    
    // 总高度
    const totalHeight = items.length * itemHeight;
    
    // 偏移量
    const offsetY = startIndex * itemHeight;
    
    const handleScroll = (e) => {
        setScrollTop(e.target.scrollTop);
    };
    
    return (
        <div
            ref={containerRef}
            style={{
                height: containerHeight,
                overflow: 'auto',
                position: 'relative'
            }}
            onScroll={handleScroll}
        >
            <div style={{ height: totalHeight }}>
                <div
                    style={{
                        transform: `translateY(${offsetY}px)`,
                        position: 'absolute',
                        width: '100%'
                    }}
                >
                    {visibleItems.map((item, index) => (
                        <div
                            key={startIndex + index}
                            style={{ height: itemHeight }}
                        >
                            {item.content}
                        </div>
                    ))}
                </div>
            </div>
        </div>
    );
}

export default VirtualList;
```

```javascript
// components/VirtualListExample.jsx
/**
 * 虚拟列表使用示例
 * @author erik.zhou
 */
import React from 'react';
import { FixedSizeList } from 'react-window';

function VirtualListExample() {
    // 生成大量数据
    const items = Array.from({ length: 10000 }, (_, index) => ({
        id: index,
        content: `项目 ${index + 1}`
    }));
    
    // 渲染单个项
    const Row = ({ index, style }) => (
        <div style={style}>
            {items[index].content}
        </div>
    );
    
    return (
        <FixedSizeList
            height={600}
            itemCount={items.length}
            itemSize={50}
            width="100%"
        >
            {Row}
        </FixedSizeList>
    );
}

export default VirtualListExample;
```

### 5.2 列表Key优化

```javascript
// components/OptimizedList.jsx
/**
 * 列表Key优化
 * @author erik.zhou
 */
import React, { useState } from 'react';

function OptimizedList() {
    const [items, setItems] = useState([
        { id: 1, name: '项目1' },
        { id: 2, name: '项目2' },
        { id: 3, name: '项目3' }
    ]);
    
    // 不好的做法 - 使用index作为key
    const BadList = () => (
        <ul>
            {items.map((item, index) => (
                <li key={index}>{item.name}</li>
            ))}
        </ul>
    );
    
    // 好的做法 - 使用唯一ID作为key
    const GoodList = () => (
        <ul>
            {items.map(item => (
                <li key={item.id}>{item.name}</li>
            ))}
        </ul>
    );
    
    const addItem = () => {
        const newItem = {
            id: Date.now(),
            name: `项目${items.length + 1}`
        };
        setItems([newItem, ...items]);
    };
    
    return (
        <div>
            <button onClick={addItem}>添加项目</button>
            <GoodList />
        </div>
    );
}

export default OptimizedList;
```

### 5.3 分页和无限滚动

```javascript
// components/InfiniteScroll.jsx
/**
 * 无限滚动优化
 * @author erik.zhou
 */
import React, { useState, useEffect, useRef, useCallback } from 'react';

function InfiniteScroll() {
    const [items, setItems] = useState([]);
    const [page, setPage] = useState(1);
    const [loading, setLoading] = useState(false);
    const [hasMore, setHasMore] = useState(true);
    
    const observerRef = useRef();
    const lastItemRef = useCallback(node => {
        if (loading) {
            return;
        }
        
        if (observerRef.current) {
            observerRef.current.disconnect();
        }
        
        observerRef.current = new IntersectionObserver(entries => {
            if (entries[0].isIntersecting && hasMore) {
                setPage(prevPage => prevPage + 1);
            }
        });
        
        if (node) {
            observerRef.current.observe(node);
        }
    }, [loading, hasMore]);
    
    useEffect(() => {
        const fetchData = async () => {
            setLoading(true);
            
            try {
                const response = await fetch(`/api/items?page=${page}&limit=20`);
                const data = await response.json();
                
                setItems(prevItems => [...prevItems, ...data.items]);
                setHasMore(data.hasMore);
            } catch (error) {
                console.error('加载失败:', error);
            } finally {
                setLoading(false);
            }
        };
        
        fetchData();
    }, [page]);
    
    return (
        <div>
            <ul>
                {items.map((item, index) => {
                    if (items.length === index + 1) {
                        return (
                            <li ref={lastItemRef} key={item.id}>
                                {item.name}
                            </li>
                        );
                    }
                    return <li key={item.id}>{item.name}</li>;
                })}
            </ul>
            
            {loading && <div>加载中...</div>}
            {!hasMore && <div>没有更多了</div>}
        </div>
    );
}

export default InfiniteScroll;
```

---

## 6. 事件处理优化

### 6.1 事件委托

```javascript
// components/EventDelegation.jsx
/**
 * 事件委托优化
 * @author erik.zhou
 */
import React from 'react';

// 不好的做法 - 每个项都绑定事件
function BadList({ items }) {
    const handleClick = (id) => {
        console.log('点击项目:', id);
    };
    
    return (
        <ul>
            {items.map(item => (
                <li key={item.id} onClick={() => handleClick(item.id)}>
                    {item.name}
                </li>
            ))}
        </ul>
    );
}

// 好的做法 - 使用事件委托
function GoodList({ items }) {
    const handleClick = (e) => {
        const id = e.target.dataset.id;
        if (id) {
            console.log('点击项目:', id);
        }
    };
    
    return (
        <ul onClick={handleClick}>
            {items.map(item => (
                <li key={item.id} data-id={item.id}>
                    {item.name}
                </li>
            ))}
        </ul>
    );
}

export { BadList, GoodList };
```

### 6.2 防抖和节流

```javascript
// hooks/useDebounce.js
/**
 * 防抖Hook
 * @author erik.zhou
 */
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
    const [debouncedValue, setDebouncedValue] = useState(value);
    
    useEffect(() => {
        const timer = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);
        
        return () => {
            clearTimeout(timer);
        };
    }, [value, delay]);
    
    return debouncedValue;
}

export default useDebounce;
```

```javascript
// hooks/useThrottle.js
/**
 * 节流Hook
 * @author erik.zhou
 */
import { useRef, useCallback } from 'react';

function useThrottle(callback, delay) {
    const lastRun = useRef(Date.now());
    
    return useCallback((...args) => {
        const now = Date.now();
        
        if (now - lastRun.current >= delay) {
            callback(...args);
            lastRun.current = now;
        }
    }, [callback, delay]);
}

export default useThrottle;
```

```javascript
// components/SearchInput.jsx
/**
 * 防抖搜索示例
 * @author erik.zhou
 */
import React, { useState, useEffect } from 'react';
import useDebounce from '../hooks/useDebounce';

function SearchInput() {
    const [searchTerm, setSearchTerm] = useState('');
    const [results, setResults] = useState([]);
    const [loading, setLoading] = useState(false);
    
    // 防抖处理
    const debouncedSearchTerm = useDebounce(searchTerm, 500);
    
    useEffect(() => {
        if (debouncedSearchTerm) {
            setLoading(true);
            
            // 模拟API调用
            fetch(`/api/search?q=${debouncedSearchTerm}`)
                .then(res => res.json())
                .then(data => {
                    setResults(data);
                    setLoading(false);
                });
        } else {
            setResults([]);
        }
    }, [debouncedSearchTerm]);
    
    return (
        <div>
            <input
                type="text"
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
                placeholder="搜索..."
            />
            
            {loading && <div>搜索中...</div>}
            
            <ul>
                {results.map(result => (
                    <li key={result.id}>{result.name}</li>
                ))}
            </ul>
        </div>
    );
}

export default SearchInput;
```

### 6.3 被动事件监听

```javascript
// components/ScrollHandler.jsx
/**
 * 被动事件监听优化
 * @author erik.zhou
 */
import React, { useEffect, useRef } from 'react';

function ScrollHandler() {
    const containerRef = useRef(null);
    
    useEffect(() => {
        const container = containerRef.current;
        
        // 不好的做法 - 默认事件监听
        const badHandler = (e) => {
            // 可能会阻止默认行为
            console.log('滚动位置:', e.target.scrollTop);
        };
        
        // 好的做法 - 被动事件监听
        const goodHandler = (e) => {
            console.log('滚动位置:', e.target.scrollTop);
        };
        
        // 添加被动监听器
        container.addEventListener('scroll', goodHandler, { passive: true });
        
        return () => {
            container.removeEventListener('scroll', goodHandler);
        };
    }, []);
    
    return (
        <div
            ref={containerRef}
            style={{ height: '400px', overflow: 'auto' }}
        >
            {/* 内容 */}
        </div>
    );
}

export default ScrollHandler;
```

---

## 7. 资源加载优化

### 7.1 图片懒加载

```javascript
// components/LazyImage.jsx
/**
 * 图片懒加载
 * @author erik.zhou
 */
import React, { useState, useEffect, useRef } from 'react';

function LazyImage({ src, alt, placeholder }) {
    const [imageSrc, setImageSrc] = useState(placeholder);
    const [isLoaded, setIsLoaded] = useState(false);
    const imgRef = useRef();
    
    useEffect(() => {
        const observer = new IntersectionObserver(
            (entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        setImageSrc(src);
                        observer.disconnect();
                    }
                });
            },
            { threshold: 0.1 }
        );
        
        if (imgRef.current) {
            observer.observe(imgRef.current);
        }
        
        return () => {
            if (imgRef.current) {
                observer.unobserve(imgRef.current);
            }
        };
    }, [src]);
    
    return (
        <img
            ref={imgRef}
            src={imageSrc}
            alt={alt}
            onLoad={() => setIsLoaded(true)}
            style={{
                opacity: isLoaded ? 1 : 0.5,
                transition: 'opacity 0.3s'
            }}
        />
    );
}

export default LazyImage;
```

### 7.2 预加载和预连接

```javascript
// components/ResourcePreload.jsx
/**
 * 资源预加载
 * @author erik.zhou
 */
import React from 'react';
import { Helmet } from 'react-helmet';

function ResourcePreload() {
    return (
        <Helmet>
            {/* DNS预解析 */}
            <link rel="dns-prefetch" href="https://api.example.com" />
            
            {/* 预连接 */}
            <link rel="preconnect" href="https://cdn.example.com" />
            
            {/* 预加载关键资源 */}
            <link 
                rel="preload" 
                href="/fonts/main.woff2" 
                as="font" 
                type="font/woff2" 
                crossOrigin="anonymous" 
            />
            
            {/* 预加载图片 */}
            <link rel="preload" href="/images/hero.jpg" as="image" />
            
            {/* 预获取下一页资源 */}
            <link rel="prefetch" href="/next-page.js" />
        </Helmet>
    );
}

export default ResourcePreload;
```

### 7.3 Web Workers

```javascript
// workers/dataProcessor.worker.js
/**
 * Web Worker处理数据
 * @author erik.zhou
 */
self.addEventListener('message', (e) => {
    const { type, data } = e.data;
    
    switch (type) {
        case 'PROCESS_DATA':
            const result = processLargeData(data);
            self.postMessage({ type: 'RESULT', data: result });
            break;
            
        case 'CALCULATE':
            const calculated = performHeavyCalculation(data);
            self.postMessage({ type: 'CALCULATED', data: calculated });
            break;
            
        default:
            break;
    }
});

function processLargeData(data) {
    // 处理大量数据
    return data.map(item => ({
        ...item,
        processed: true,
        timestamp: Date.now()
    }));
}

function performHeavyCalculation(numbers) {
    // 执行复杂计算
    return numbers.reduce((sum, num) => sum + num, 0);
}
```

```javascript
// hooks/useWorker.js
/**
 * Web Worker Hook
 * @author erik.zhou
 */
import { useEffect, useRef, useState } from 'react';

function useWorker(workerPath) {
    const workerRef = useRef(null);
    const [result, setResult] = useState(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        workerRef.current = new Worker(workerPath);
        
        workerRef.current.onmessage = (e) => {
            setResult(e.data);
            setLoading(false);
        };
        
        workerRef.current.onerror = (e) => {
            setError(e.message);
            setLoading(false);
        };
        
        return () => {
            workerRef.current?.terminate();
        };
    }, [workerPath]);
    
    const postMessage = (message) => {
        setLoading(true);
        setError(null);
        workerRef.current?.postMessage(message);
    };
    
    return { result, loading, error, postMessage };
}

export default useWorker;
```

```javascript
// components/WorkerExample.jsx
/**
 * Web Worker使用示例
 * @author erik.zhou
 */
import React, { useState } from 'react';
import useWorker from '../hooks/useWorker';

function WorkerExample() {
    const [data, setData] = useState([]);
    const { result, loading, postMessage } = useWorker('/workers/dataProcessor.worker.js');
    
    const handleProcess = () => {
        const largeData = Array.from({ length: 100000 }, (_, i) => ({
            id: i,
            value: Math.random()
        }));
        
        postMessage({
            type: 'PROCESS_DATA',
            data: largeData
        });
    };
    
    return (
        <div>
            <button onClick={handleProcess} disabled={loading}>
                {loading ? '处理中...' : '处理数据'}
            </button>
            
            {result && (
                <div>
                    <p>处理完成！</p>
                    <p>处理了 {result.data.length} 条数据</p>
                </div>
            )}
        </div>
    );
}

export default WorkerExample;
```

---

## 8. 性能监控

### 8.1 React Profiler API

```javascript
// components/ProfilerWrapper.jsx
/**
 * React Profiler性能监控
 * @author erik.zhou
 */
import React, { Profiler } from 'react';

function onRenderCallback(
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime,
    interactions
) {
    console.log('Profiler数据:', {
        id,                 // 组件ID
        phase,              // "mount" 或 "update"
        actualDuration,     // 本次更新花费的时间
        baseDuration,       // 不使用memoization的渲染时间
        startTime,          // 开始渲染的时间
        commitTime,         // 提交更新的时间
        interactions        // 触发更新的交互集合
    });
    
    // 发送到分析服务
    if (actualDuration > 16) { // 超过一帧的时间
        sendToAnalytics({
            component: id,
            duration: actualDuration,
            phase
        });
    }
}

function ProfilerWrapper({ id, children }) {
    return (
        <Profiler id={id} onRender={onRenderCallback}>
            {children}
        </Profiler>
    );
}

function sendToAnalytics(data) {
    // 发送性能数据到分析服务
    console.log('发送分析数据:', data);
}

export default ProfilerWrapper;
```

### 8.2 性能指标监控

```javascript
// utils/performanceMonitor.js
/**
 * 性能指标监控
 * @author erik.zhou
 */

// 监控Core Web Vitals
export function monitorWebVitals() {
    // LCP - Largest Contentful Paint
    const lcpObserver = new PerformanceObserver((list) => {
        const entries = list.getEntries();
        const lastEntry = entries[entries.length - 1];
        
        console.log('LCP:', lastEntry.renderTime || lastEntry.loadTime);
        sendMetric('LCP', lastEntry.renderTime || lastEntry.loadTime);
    });
    
    lcpObserver.observe({ entryTypes: ['largest-contentful-paint'] });
    
    // FID - First Input Delay
    const fidObserver = new PerformanceObserver((list) => {
        const entries = list.getEntries();
        entries.forEach(entry => {
            console.log('FID:', entry.processingStart - entry.startTime);
            sendMetric('FID', entry.processingStart - entry.startTime);
        });
    });
    
    fidObserver.observe({ entryTypes: ['first-input'] });
    
    // CLS - Cumulative Layout Shift
    let clsValue = 0;
    const clsObserver = new PerformanceObserver((list) => {
        const entries = list.getEntries();
        entries.forEach(entry => {
            if (!entry.hadRecentInput) {
                clsValue += entry.value;
                console.log('CLS:', clsValue);
                sendMetric('CLS', clsValue);
            }
        });
    });
    
    clsObserver.observe({ entryTypes: ['layout-shift'] });
}

// 监控长任务
export function monitorLongTasks() {
    const observer = new PerformanceObserver((list) => {
        const entries = list.getEntries();
        entries.forEach(entry => {
            console.log('长任务:', {
                duration: entry.duration,
                startTime: entry.startTime
            });
            
            if (entry.duration > 50) {
                sendMetric('LONG_TASK', entry.duration);
            }
        });
    });
    
    observer.observe({ entryTypes: ['longtask'] });
}

// 监控资源加载
export function monitorResourceTiming() {
    const observer = new PerformanceObserver((list) => {
        const entries = list.getEntries();
        entries.forEach(entry => {
            console.log('资源加载:', {
                name: entry.name,
                duration: entry.duration,
                size: entry.transferSize
            });
            
            if (entry.duration > 1000) {
                sendMetric('SLOW_RESOURCE', {
                    name: entry.name,
                    duration: entry.duration
                });
            }
        });
    });
    
    observer.observe({ entryTypes: ['resource'] });
}

function sendMetric(name, value) {
    // 发送指标到监控服务
    if (navigator.sendBeacon) {
        const data = JSON.stringify({ name, value, timestamp: Date.now() });
        navigator.sendBeacon('/api/metrics', data);
    }
}
```

### 8.3 自定义性能Hook

```javascript
// hooks/usePerformance.js
/**
 * 性能监控Hook
 * @author erik.zhou
 */
import { useEffect, useRef } from 'react';

function usePerformance(componentName) {
    const renderCount = useRef(0);
    const startTime = useRef(performance.now());
    
    useEffect(() => {
        renderCount.current += 1;
        const endTime = performance.now();
        const duration = endTime - startTime.current;
        
        console.log(`${componentName} 性能数据:`, {
            renderCount: renderCount.current,
            duration: duration.toFixed(2) + 'ms'
        });
        
        // 记录慢渲染
        if (duration > 16) {
            console.warn(`${componentName} 渲染较慢: ${duration.toFixed(2)}ms`);
        }
        
        startTime.current = performance.now();
    });
    
    return renderCount.current;
}

export default usePerformance;
```

```javascript
// components/MonitoredComponent.jsx
/**
 * 带性能监控的组件
 * @author erik.zhou
 */
import React from 'react';
import usePerformance from '../hooks/usePerformance';

function MonitoredComponent({ data }) {
    const renderCount = usePerformance('MonitoredComponent');
    
    return (
        <div>
            <p>渲染次数: {renderCount}</p>
            <ul>
                {data.map(item => (
                    <li key={item.id}>{item.name}</li>
                ))}
            </ul>
        </div>
    );
}

export default MonitoredComponent;
```

---

## 9. 最佳实践

### 9.1 性能优化清单

```javascript
/**
 * React性能优化清单
 * @author erik.zhou
 */

const performanceChecklist = {
    渲染优化: [
        '使用React.memo包裹纯组件',
        '使用useMemo缓存计算结果',
        '使用useCallback缓存回调函数',
        '避免在render中创建新对象/数组',
        '合理拆分组件，避免过大组件'
    ],
    
    代码分割: [
        '使用React.lazy进行路由级代码分割',
        '懒加载重型组件',
        '按需加载第三方库',
        '使用动态import'
    ],
    
    状态管理: [
        '状态下沉到最近的使用组件',
        '拆分Context避免不必要的重渲染',
        '使用状态管理库（Redux/Zustand）',
        '避免在Context中存放频繁变化的值'
    ],
    
    列表优化: [
        '使用虚拟列表处理大量数据',
        '使用稳定的key值',
        '实现分页或无限滚动',
        '避免在列表项中使用index作为key'
    ],
    
    事件处理: [
        '使用事件委托',
        '对频繁触发的事件使用防抖/节流',
        '使用被动事件监听器',
        '及时清理事件监听器'
    ],
    
    资源加载: [
        '图片懒加载',
        '使用WebP等现代图片格式',
        '预加载关键资源',
        '使用CDN加速静态资源'
    ]
};
```

### 9.2 常见性能陷阱

```javascript
// components/PerformancePitfalls.jsx
/**
 * 常见性能陷阱示例
 * @author erik.zhou
 */
import React, { useState, useMemo, useCallback } from 'react';

// 陷阱1: 在render中创建函数
function Pitfall1Bad() {
    return (
        <button onClick={() => console.log('点击')}>
            点击
        </button>
    );
}

function Pitfall1Good() {
    const handleClick = useCallback(() => {
        console.log('点击');
    }, []);
    
    return <button onClick={handleClick}>点击</button>;
}

// 陷阱2: 在render中创建对象
function Pitfall2Bad({ data }) {
    return <ChildComponent style={{ color: 'red' }} />;
}

const STYLE = { color: 'red' };
function Pitfall2Good({ data }) {
    return <ChildComponent style={STYLE} />;
}

// 陷阱3: 不必要的状态
function Pitfall3Bad({ items }) {
    const [filteredItems, setFilteredItems] = useState([]);
    
    // 不需要状态，可以直接计算
    return <div>{filteredItems.length}</div>;
}

function Pitfall3Good({ items, filter }) {
    const filteredItems = useMemo(() => 
        items.filter(item => item.name.includes(filter)),
        [items, filter]
    );
    
    return <div>{filteredItems.length}</div>;
}

// 陷阱4: 过度使用Context
// 应该拆分Context或使用状态管理库
```

### 9.3 性能优化决策树

```javascript
/**
 * 性能优化决策流程
 * @author erik.zhou
 */

const optimizationDecisionTree = `
1. 是否存在性能问题？
   └─ 否 → 不需要优化
   └─ 是 → 继续

2. 使用Profiler定位问题
   └─ 组件渲染次数过多？
      └─ 是 → 使用React.memo/useMemo/useCallback
      └─ 否 → 继续
   
   └─ 单次渲染时间过长？
      └─ 是 → 优化计算逻辑/使用Web Worker
      └─ 否 → 继续
   
   └─ 列表渲染慢？
      └─ 是 → 使用虚拟列表/分页
      └─ 否 → 继续
   
   └─ 首屏加载慢？
      └─ 是 → 代码分割/懒加载/预加载
      └─ 否 → 继续
   
   └─ 状态更新导致大范围重渲染？
      └─ 是 → 状态下沉/拆分Context
      └─ 否 → 继续

3. 验证优化效果
   └─ 使用Profiler对比优化前后
   └─ 监控Core Web Vitals
   └─ 用户体验是否改善
`;
```

---

## 10. 实战案例

### 10.1 大型表格优化

```javascript
// components/OptimizedTable.jsx
/**
 * 优化的大型表格
 * @author erik.zhou
 */
import React, { useMemo, memo } from 'react';
import { useVirtual } from 'react-virtual';

// 表格行组件
const TableRow = memo(function TableRow({ row, columns }) {
    return (
        <tr>
            {columns.map(column => (
                <td key={column.key}>
                    {row[column.key]}
                </td>
            ))}
        </tr>
    );
});

function OptimizedTable({ data, columns }) {
    const parentRef = React.useRef();
    
    // 虚拟滚动
    const rowVirtualizer = useVirtual({
        size: data.length,
        parentRef,
        estimateSize: React.useCallback(() => 50, [])
    });
    
    return (
        <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
            <table>
                <thead>
                    <tr>
                        {columns.map(column => (
                            <th key={column.key}>{column.title}</th>
                        ))}
                    </tr>
                </thead>
                <tbody style={{ height: `${rowVirtualizer.totalSize}px` }}>
                    {rowVirtualizer.virtualItems.map(virtualRow => (
                        <TableRow
                            key={virtualRow.index}
                            row={data[virtualRow.index]}
                            columns={columns}
                        />
                    ))}
                </tbody>
            </table>
        </div>
    );
}

export default OptimizedTable;
```

### 10.2 复杂表单优化

```javascript
// components/OptimizedForm.jsx
/**
 * 优化的复杂表单
 * @author erik.zhou
 */
import React, { memo } from 'react';
import { useForm } from 'react-hook-form';

// 表单字段组件
const FormField = memo(function FormField({ name, label, register, error }) {
    console.log(`${name} 字段渲染`);
    
    return (
        <div>
            <label>{label}</label>
            <input {...register(name)} />
            {error && <span>{error.message}</span>}
        </div>
    );
});

function OptimizedForm() {
    const { register, handleSubmit, formState: { errors } } = useForm();
    
    const onSubmit = (data) => {
        console.log('提交数据:', data);
    };
    
    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            <FormField
                name="username"
                label="用户名"
                register={register}
                error={errors.username}
            />
            
            <FormField
                name="email"
                label="邮箱"
                register={register}
                error={errors.email}
            />
            
            <FormField
                name="password"
                label="密码"
                register={register}
                error={errors.password}
            />
            
            <button type="submit">提交</button>
        </form>
    );
}

export default OptimizedForm;
```


### 10.3 实时数据流优化

```javascript
// components/OptimizedDataStream.jsx
/**
 * 优化的实时数据流组件
 * @author erik.zhou
 */
import React, { useState, useEffect, useCallback, useMemo } from 'react';
import { useThrottle } from '../hooks/useThrottle';

function OptimizedDataStream() {
    const [messages, setMessages] = useState([]);
    const [filter, setFilter] = useState('');
    const [isPaused, setIsPaused] = useState(false);
    
    // 节流更新消息
    const throttledAddMessage = useThrottle((newMessage) => {
        setMessages(prev => {
            // 限制消息数量，避免内存溢出
            const updated = [newMessage, ...prev];
            return updated.slice(0, 1000);
        });
    }, 100);
    
    useEffect(() => {
        if (isPaused) {
            return;
        }
        
        // 模拟WebSocket连接
        const ws = new WebSocket('wss://example.com/stream');
        
        ws.onmessage = (event) => {
            const message = JSON.parse(event.data);
            throttledAddMessage(message);
        };
        
        return () => {
            ws.close();
        };
    }, [isPaused, throttledAddMessage]);
    
    // 过滤消息
    const filteredMessages = useMemo(() => {
        if (!filter) {
            return messages;
        }
        return messages.filter(msg => 
            msg.content.toLowerCase().includes(filter.toLowerCase())
        );
    }, [messages, filter]);
    
    // 统计信息
    const stats = useMemo(() => ({
        total: messages.length,
        filtered: filteredMessages.length,
        rate: messages.length > 0 ? (filteredMessages.length / messages.length * 100).toFixed(2) : 0
    }), [messages.length, filteredMessages.length]);
    
    return (
        <div className="data-stream">
            <div className="controls">
                <input
                    type="text"
                    value={filter}
                    onChange={(e) => setFilter(e.target.value)}
                    placeholder="过滤消息..."
                />
                
                <button onClick={() => setIsPaused(!isPaused)}>
                    {isPaused ? '继续' : '暂停'}
                </button>
                
                <button onClick={() => setMessages([])}>
                    清空
                </button>
                
                <div className="stats">
                    总计: {stats.total} | 显示: {stats.filtered} | 比率: {stats.rate}%
                </div>
            </div>
            
            <MessageList messages={filteredMessages} />
        </div>
    );
}

// 消息列表组件
const MessageList = React.memo(function MessageList({ messages }) {
    return (
        <div className="message-list" style={{ height: '500px', overflow: 'auto' }}>
            {messages.map((message, index) => (
                <MessageItem key={message.id || index} message={message} />
            ))}
        </div>
    );
});

// 消息项组件
const MessageItem = React.memo(function MessageItem({ message }) {
    return (
        <div className="message-item">
            <span className="timestamp">{new Date(message.timestamp).toLocaleTimeString()}</span>
            <span className="content">{message.content}</span>
        </div>
    );
});

export default OptimizedDataStream;
```

### 10.4 图片画廊优化

```javascript
// components/OptimizedGallery.jsx
/**
 * 优化的图片画廊
 * @author erik.zhou
 */
import React, { useState, useCallback, useMemo } from 'react';
import LazyImage from './LazyImage';

function OptimizedGallery({ images }) {
    const [selectedImage, setSelectedImage] = useState(null);
    const [loadedImages, setLoadedImages] = useState(new Set());
    
    // 图片加载完成回调
    const handleImageLoad = useCallback((imageId) => {
        setLoadedImages(prev => new Set([...prev, imageId]));
    }, []);
    
    // 缩略图组件
    const Thumbnail = React.memo(function Thumbnail({ image, onClick }) {
        return (
            <div className="thumbnail" onClick={onClick}>
                <LazyImage
                    src={image.thumbnail}
                    alt={image.title}
                    placeholder="/placeholder.jpg"
                    onLoad={() => handleImageLoad(image.id)}
                />
                <div className="overlay">
                    <span>{image.title}</span>
                </div>
            </div>
        );
    });
    
    // 图片网格
    const imageGrid = useMemo(() => {
        return images.map(image => (
            <Thumbnail
                key={image.id}
                image={image}
                onClick={() => setSelectedImage(image)}
            />
        ));
    }, [images]);
    
    return (
        <div className="gallery">
            <div className="gallery-grid">
                {imageGrid}
            </div>
            
            {selectedImage && (
                <ImageModal
                    image={selectedImage}
                    onClose={() => setSelectedImage(null)}
                />
            )}
            
            <div className="loading-stats">
                已加载: {loadedImages.size} / {images.length}
            </div>
        </div>
    );
}

// 图片模态框
const ImageModal = React.memo(function ImageModal({ image, onClose }) {
    return (
        <div className="modal-overlay" onClick={onClose}>
            <div className="modal-content" onClick={(e) => e.stopPropagation()}>
                <button className="close-button" onClick={onClose}>×</button>
                <img src={image.fullSize} alt={image.title} />
                <div className="image-info">
                    <h3>{image.title}</h3>
                    <p>{image.description}</p>
                </div>
            </div>
        </div>
    );
});

export default OptimizedGallery;
```

### 10.5 实时搜索优化

```javascript
// components/OptimizedSearch.jsx
/**
 * 优化的实时搜索
 * @author erik.zhou
 */
import React, { useState, useEffect, useMemo, useCallback } from 'react';
import useDebounce from '../hooks/useDebounce';

function OptimizedSearch({ data }) {
    const [searchTerm, setSearchTerm] = useState('');
    const [results, setResults] = useState([]);
    const [loading, setLoading] = useState(false);
    const [cache, setCache] = useState(new Map());
    
    // 防抖搜索词
    const debouncedSearchTerm = useDebounce(searchTerm, 300);
    
    // 搜索函数
    const performSearch = useCallback(async (term) => {
        if (!term) {
            setResults([]);
            return;
        }
        
        // 检查缓存
        if (cache.has(term)) {
            setResults(cache.get(term));
            return;
        }
        
        setLoading(true);
        
        try {
            // 模拟API调用
            const response = await fetch(`/api/search?q=${encodeURIComponent(term)}`);
            const data = await response.json();
            
            // 更新缓存
            setCache(prev => new Map(prev).set(term, data));
            setResults(data);
        } catch (error) {
            console.error('搜索失败:', error);
        } finally {
            setLoading(false);
        }
    }, [cache]);
    
    useEffect(() => {
        performSearch(debouncedSearchTerm);
    }, [debouncedSearchTerm, performSearch]);
    
    // 高亮搜索词
    const highlightText = useCallback((text, highlight) => {
        if (!highlight) {
            return text;
        }
        
        const parts = text.split(new RegExp(`(${highlight})`, 'gi'));
        return parts.map((part, index) => 
            part.toLowerCase() === highlight.toLowerCase() 
                ? <mark key={index}>{part}</mark> 
                : part
        );
    }, []);
    
    return (
        <div className="search-container">
            <input
                type="text"
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
                placeholder="搜索..."
                className="search-input"
            />
            
            {loading && <div className="loading">搜索中...</div>}
            
            <SearchResults 
                results={results} 
                searchTerm={debouncedSearchTerm}
                highlightText={highlightText}
            />
        </div>
    );
}

// 搜索结果组件
const SearchResults = React.memo(function SearchResults({ results, searchTerm, highlightText }) {
    if (results.length === 0 && searchTerm) {
        return <div className="no-results">未找到结果</div>;
    }
    
    return (
        <ul className="search-results">
            {results.map(result => (
                <SearchResultItem
                    key={result.id}
                    result={result}
                    searchTerm={searchTerm}
                    highlightText={highlightText}
                />
            ))}
        </ul>
    );
});

// 搜索结果项
const SearchResultItem = React.memo(function SearchResultItem({ result, searchTerm, highlightText }) {
    return (
        <li className="result-item">
            <h4>{highlightText(result.title, searchTerm)}</h4>
            <p>{highlightText(result.description, searchTerm)}</p>
        </li>
    );
});

export default OptimizedSearch;
```

### 10.6 无限滚动列表优化

```javascript
// components/OptimizedInfiniteList.jsx
/**
 * 优化的无限滚动列表
 * @author erik.zhou
 */
import React, { useState, useEffect, useRef, useCallback } from 'react';

function OptimizedInfiniteList() {
    const [items, setItems] = useState([]);
    const [page, setPage] = useState(1);
    const [loading, setLoading] = useState(false);
    const [hasMore, setHasMore] = useState(true);
    const [error, setError] = useState(null);
    
    const observerRef = useRef();
    const loadingRef = useRef(false);
    
    // 加载数据
    const loadMore = useCallback(async () => {
        if (loadingRef.current || !hasMore) {
            return;
        }
        
        loadingRef.current = true;
        setLoading(true);
        setError(null);
        
        try {
            const response = await fetch(`/api/items?page=${page}&limit=20`);
            const data = await response.json();
            
            setItems(prev => [...prev, ...data.items]);
            setHasMore(data.hasMore);
            setPage(prev => prev + 1);
        } catch (err) {
            setError('加载失败，请重试');
            console.error('加载失败:', err);
        } finally {
            setLoading(false);
            loadingRef.current = false;
        }
    }, [page, hasMore]);
    
    // 最后一个元素的ref回调
    const lastItemRef = useCallback(node => {
        if (loading) {
            return;
        }
        
        if (observerRef.current) {
            observerRef.current.disconnect();
        }
        
        observerRef.current = new IntersectionObserver(
            entries => {
                if (entries[0].isIntersecting && hasMore) {
                    loadMore();
                }
            },
            { threshold: 0.1 }
        );
        
        if (node) {
            observerRef.current.observe(node);
        }
    }, [loading, hasMore, loadMore]);
    
    // 初始加载
    useEffect(() => {
        loadMore();
    }, []);
    
    return (
        <div className="infinite-list">
            <ul>
                {items.map((item, index) => {
                    const isLastItem = items.length === index + 1;
                    
                    return (
                        <ListItem
                            key={item.id}
                            item={item}
                            ref={isLastItem ? lastItemRef : null}
                        />
                    );
                })}
            </ul>
            
            {loading && (
                <div className="loading-indicator">
                    <div className="spinner"></div>
                    <p>加载中...</p>
                </div>
            )}
            
            {error && (
                <div className="error-message">
                    <p>{error}</p>
                    <button onClick={loadMore}>重试</button>
                </div>
            )}
            
            {!hasMore && (
                <div className="end-message">
                    没有更多内容了
                </div>
            )}
        </div>
    );
}

// 列表项组件
const ListItem = React.forwardRef(function ListItem({ item }, ref) {
    return (
        <li ref={ref} className="list-item">
            <div className="item-content">
                <h3>{item.title}</h3>
                <p>{item.description}</p>
                <span className="item-meta">{item.date}</span>
            </div>
        </li>
    );
});

export default OptimizedInfiniteList;
```

---

## 总结

### 核心要点

React性能优化是一个系统工程，需要从多个维度进行考虑和实施：

1. **渲染优化**
   - 使用React.memo避免不必要的重渲染
   - 使用useMemo缓存计算结果
   - 使用useCallback缓存回调函数
   - 避免在render中创建新对象和数组

2. **代码分割**
   - 路由级代码分割
   - 组件级懒加载
   - 第三方库按需加载
   - 动态导入优化

3. **状态管理**
   - 状态下沉到最近的使用组件
   - 拆分Context避免大范围重渲染
   - 合理使用状态管理库
   - 避免过度提升状态

4. **列表优化**
   - 虚拟列表处理大量数据
   - 使用稳定的key值
   - 实现分页或无限滚动
   - 优化列表项渲染

5. **事件处理**
   - 使用事件委托
   - 防抖和节流优化
   - 被动事件监听
   - 及时清理事件监听器

6. **资源加载**
   - 图片懒加载
   - 预加载关键资源
   - 使用Web Workers处理复杂计算
   - CDN加速静态资源

7. **性能监控**
   - 使用React Profiler分析性能
   - 监控Core Web Vitals
   - 建立性能监控体系
   - 持续优化和改进

### 优化原则

1. **测量优先**: 先测量找到瓶颈，再针对性优化
2. **避免过早优化**: 不要在没有性能问题时就进行优化
3. **权衡取舍**: 性能优化可能增加代码复杂度，需要权衡
4. **持续监控**: 建立性能监控体系，及时发现和解决问题

### 学习路径

1. **基础阶段**
   - 理解React渲染机制
   - 掌握基本优化技巧（memo、useMemo、useCallback）
   - 学习使用React DevTools Profiler

2. **进阶阶段**
   - 深入理解Fiber架构
   - 掌握代码分割和懒加载
   - 学习虚拟列表等高级技巧
   - 实践性能监控

3. **高级阶段**
   - 系统性能优化方案设计
   - 性能监控体系建设
   - 复杂场景性能优化
   - 性能优化最佳实践总结

### 推荐资源

**官方文档**
- [React官方文档 - 性能优化](https://react.dev/learn/render-and-commit)
- [React Profiler API](https://react.dev/reference/react/Profiler)
- [Web Vitals](https://web.dev/vitals/)

**工具库**
- [react-window](https://github.com/bvaughn/react-window) - 虚拟列表
- [react-virtual](https://github.com/tannerlinsley/react-virtual) - 虚拟滚动
- [use-debounce](https://github.com/xnimorz/use-debounce) - 防抖Hook

**学习资源**
- [React性能优化指南](https://kentcdodds.com/blog/optimize-react-re-renders)
- [Web性能优化](https://web.dev/fast/)
- [Chrome DevTools性能分析](https://developer.chrome.com/docs/devtools/performance/)

---

## 附录

### A. 常用API速查

```javascript
/**
 * React性能优化API速查
 * @author erik.zhou
 */

// React.memo
const MemoComponent = React.memo(Component, arePropsEqual);

// useMemo
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

// useCallback
const memoizedCallback = useCallback(() => {
    doSomething(a, b);
}, [a, b]);

// React.lazy
const LazyComponent = React.lazy(() => import('./Component'));

// Suspense
<Suspense fallback={<Loading />}>
    <LazyComponent />
</Suspense>

// Profiler
<Profiler id="App" onRender={onRenderCallback}>
    <App />
</Profiler>

// useTransition (React 18+)
const [isPending, startTransition] = useTransition();

// useDeferredValue (React 18+)
const deferredValue = useDeferredValue(value);
```

### B. 性能优化配置模板

```javascript
// webpack.config.js
/**
 * Webpack性能优化配置
 * @author erik.zhou
 */
module.exports = {
    mode: 'production',
    
    optimization: {
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    priority: 10
                },
                common: {
                    minChunks: 2,
                    priority: 5,
                    reuseExistingChunk: true
                }
            }
        },
        runtimeChunk: 'single',
        minimize: true
    },
    
    performance: {
        maxEntrypointSize: 512000,
        maxAssetSize: 512000,
        hints: 'warning'
    }
};
```

```javascript
// vite.config.js
/**
 * Vite性能优化配置
 * @author erik.zhou
 */
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    'react-vendor': ['react', 'react-dom'],
                    'router': ['react-router-dom'],
                    'ui': ['antd']
                }
            }
        },
        chunkSizeWarningLimit: 1000
    },
    
    server: {
        hmr: {
            overlay: false
        }
    }
});
```

### C. 常见问题FAQ

**Q1: 什么时候应该使用React.memo？**

A: 当组件满足以下条件时考虑使用React.memo：
- 组件经常以相同的props重新渲染
- 组件渲染成本较高
- 组件是纯组件（相同props产生相同输出）

**Q2: useMemo和useCallback的区别？**

A: 
- useMemo缓存计算结果（值）
- useCallback缓存函数本身
- useCallback(fn, deps) 等价于 useMemo(() => fn, deps)

**Q3: 虚拟列表适用于什么场景？**

A: 
- 需要渲染大量列表项（通常>1000项）
- 列表项高度固定或可预测
- 用户需要滚动浏览整个列表

**Q4: 如何选择代码分割的粒度？**

A: 
- 路由级分割：适用于大多数应用
- 组件级分割：适用于重型组件（图表、编辑器等）
- 避免过度分割导致请求过多

**Q5: Context性能问题如何解决？**

A: 
- 拆分Context，按功能域划分
- 使用useMemo缓存Context value
- 考虑使用状态管理库（Redux、Zustand）
- 状态下沉到最近的使用组件

### D. 性能优化工具清单

**开发工具**
- React DevTools Profiler - React性能分析
- Chrome DevTools Performance - 整体性能分析
- Lighthouse - 综合性能评分
- webpack-bundle-analyzer - 打包分析

**监控工具**
- Web Vitals - 核心性能指标
- Sentry - 错误和性能监控
- New Relic - APM性能监控
- Google Analytics - 用户体验监控

**优化库**
- react-window / react-virtual - 虚拟列表
- use-debounce - 防抖Hook
- lodash - 工具函数（按需引入）
- immer - 不可变数据

**构建工具**
- Webpack 5 - 模块打包
- Vite - 快速构建
- esbuild - 极速打包
- SWC - Rust编译器

---

**课程总结**: 本教程系统讲解了React性能优化的各个方面，从渲染优化到代码分割，从状态管理到性能监控，提供了大量实战案例和最佳实践。掌握这些技巧，能够显著提升React应用的性能和用户体验。

**@author erik.zhou**  
**最后更新**: 2026-03-02
