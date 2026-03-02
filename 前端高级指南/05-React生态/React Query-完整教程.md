# React Query - 完整教程

## 课程信息
- **课程名称**: React Query完整教程
- **难度级别**: 中高级
- **预计学时**: 6小时
- **核心内容**: 数据获取、缓存管理、乐观更新、无限滚动
- **@author**: erik.zhou

---

## 目录
1. [React Query概述](#1-react-query概述)
2. [基础使用](#2-基础使用)
3. [查询管理](#3-查询管理)
4. [变更操作](#4-变更操作)
5. [缓存策略](#5-缓存策略)
6. [乐观更新](#6-乐观更新)
7. [无限滚动](#7-无限滚动)
8. [性能优化](#8-性能优化)
9. [最佳实践](#9-最佳实践)
10. [实战案例](#10-实战案例)

---

## 1. React Query概述

### 1.1 什么是React Query

React Query是一个强大的数据获取和状态管理库，专注于服务端状态管理。

**核心特点**:
- 自动缓存和后台更新
- 智能重试机制
- 乐观更新支持
- 无限滚动和分页
- 请求去重
- 开发工具完善

### 1.2 为什么使用React Query

```javascript
// 传统方式的问题
const traditionalProblems = {
    样板代码多: '需要手动管理loading、error、data状态',
    缓存困难: '需要自己实现缓存逻辑',
    重复请求: '同一数据可能被多次请求',
    数据同步: '难以保持数据最新',
    错误处理: '需要在每个组件中处理错误'
};

// React Query的优势
const reactQueryBenefits = {
    自动缓存: '智能缓存和后台更新',
    请求去重: '自动合并相同请求',
    数据同步: '自动保持数据新鲜',
    错误重试: '内置重试机制',
    开发体验: 'DevTools支持，调试方便'
};
```

### 1.3 安装配置

```bash
# 安装React Query
npm install @tanstack/react-query

# 安装DevTools（可选）
npm install @tanstack/react-query-devtools

# 或使用yarn
yarn add @tanstack/react-query
yarn add @tanstack/react-query-devtools
```

```javascript
// App.jsx
/**
 * React Query配置
 * @author erik.zhou
 */
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

// 创建QueryClient实例
const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            staleTime: 1000 * 60 * 5, // 5分钟
            cacheTime: 1000 * 60 * 10, // 10分钟
            retry: 3,
            refetchOnWindowFocus: false
        }
    }
});

function App() {
    return (
        <QueryClientProvider client={queryClient}>
            <YourApp />
            <ReactQueryDevtools initialIsOpen={false} />
        </QueryClientProvider>
    );
}

export default App;
```

---

## 2. 基础使用

### 2.1 useQuery基础

```javascript
// hooks/useUsers.js
/**
 * 获取用户列表Hook
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';

async function fetchUsers() {
    const response = await fetch('/api/users');
    if (!response.ok) {
        throw new Error('获取用户失败');
    }
    return response.json();
}

function useUsers() {
    return useQuery({
        queryKey: ['users'],
        queryFn: fetchUsers
    });
}

export default useUsers;
```

```javascript
// components/UserList.jsx
/**
 * 用户列表组件
 * @author erik.zhou
 */
import React from 'react';
import useUsers from '../hooks/useUsers';

function UserList() {
    const { data, isLoading, isError, error } = useUsers();
    
    if (isLoading) {
        return <div>加载中...</div>;
    }
    
    if (isError) {
        return <div>错误: {error.message}</div>;
    }
    
    return (
        <div>
            <h2>用户列表</h2>
            {data.map(user => (
                <div key={user.id}>
                    <h3>{user.name}</h3>
                    <p>{user.email}</p>
                </div>
            ))}
        </div>
    );
}

export default UserList;
```

### 2.2 带参数的查询

```javascript
// hooks/useUser.js
/**
 * 获取单个用户Hook
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';

async function fetchUser(userId) {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
        throw new Error('获取用户失败');
    }
    return response.json();
}

function useUser(userId) {
    return useQuery({
        queryKey: ['user', userId],
        queryFn: () => fetchUser(userId),
        enabled: !!userId // 只有userId存在时才执行查询
    });
}

export default useUser;
```

```javascript
// components/UserDetail.jsx
/**
 * 用户详情组件
 * @author erik.zhou
 */
import React from 'react';
import { useParams } from 'react-router-dom';
import useUser from '../hooks/useUser';

function UserDetail() {
    const { userId } = useParams();
    const { data: user, isLoading, isError, error } = useUser(userId);
    
    if (isLoading) {
        return <div>加载中...</div>;
    }
    
    if (isError) {
        return <div>错误: {error.message}</div>;
    }
    
    return (
        <div>
            <h2>{user.name}</h2>
            <p>邮箱: {user.email}</p>
            <p>电话: {user.phone}</p>
        </div>
    );
}

export default UserDetail;
```

### 2.3 查询状态

```javascript
// components/QueryStatus.jsx
/**
 * 查询状态示例
 * @author erik.zhou
 */
import React from 'react';
import { useQuery } from '@tanstack/react-query';

function QueryStatus() {
    const {
        data,
        isLoading,      // 首次加载
        isFetching,     // 任何时候的获取状态
        isError,        // 是否有错误
        error,          // 错误对象
        isSuccess,      // 是否成功
        status,         // 'loading' | 'error' | 'success'
        fetchStatus     // 'fetching' | 'paused' | 'idle'
    } = useQuery({
        queryKey: ['data'],
        queryFn: fetchData
    });
    
    return (
        <div>
            <div>状态: {status}</div>
            <div>获取状态: {fetchStatus}</div>
            {isLoading && <div>首次加载中...</div>}
            {isFetching && <div>获取数据中...</div>}
            {isError && <div>错误: {error.message}</div>}
            {isSuccess && <div>数据: {JSON.stringify(data)}</div>}
        </div>
    );
}

export default QueryStatus;
```

---

## 3. 查询管理

### 3.1 查询键（Query Keys）

```javascript
// queryKeys.js
/**
 * 查询键管理
 * @author erik.zhou
 */

// 简单查询键
const userKeys = {
    all: ['users'],
    lists: () => [...userKeys.all, 'list'],
    list: (filters) => [...userKeys.lists(), filters],
    details: () => [...userKeys.all, 'detail'],
    detail: (id) => [...userKeys.details(), id]
};

// 使用示例
function useUsers(filters) {
    return useQuery({
        queryKey: userKeys.list(filters),
        queryFn: () => fetchUsers(filters)
    });
}

function useUser(userId) {
    return useQuery({
        queryKey: userKeys.detail(userId),
        queryFn: () => fetchUser(userId)
    });
}

export { userKeys };
```

### 3.2 依赖查询

```javascript
// hooks/useDependentQueries.js
/**
 * 依赖查询示例
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';

function useDependentQueries(userId) {
    // 第一个查询：获取用户信息
    const { data: user } = useQuery({
        queryKey: ['user', userId],
        queryFn: () => fetchUser(userId),
        enabled: !!userId
    });
    
    // 第二个查询：依赖第一个查询的结果
    const { data: projects } = useQuery({
        queryKey: ['projects', user?.id],
        queryFn: () => fetchUserProjects(user.id),
        enabled: !!user?.id // 只有user存在时才执行
    });
    
    return { user, projects };
}

export default useDependentQueries;
```

### 3.3 并行查询

```javascript
// hooks/useParallelQueries.js
/**
 * 并行查询示例
 * @author erik.zhou
 */
import { useQueries } from '@tanstack/react-query';

function useParallelQueries(userIds) {
    const queries = useQueries({
        queries: userIds.map(id => ({
            queryKey: ['user', id],
            queryFn: () => fetchUser(id)
        }))
    });
    
    // 检查所有查询是否完成
    const isLoading = queries.some(query => query.isLoading);
    const isError = queries.some(query => query.isError);
    const data = queries.map(query => query.data);
    
    return { data, isLoading, isError };
}

export default useParallelQueries;
```

### 3.4 查询重试

```javascript
// hooks/useRetryQuery.js
/**
 * 查询重试配置
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';

function useRetryQuery() {
    return useQuery({
        queryKey: ['data'],
        queryFn: fetchData,
        retry: 3, // 重试3次
        retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
        // 自定义重试条件
        retryOnMount: true,
        // 只在特定错误时重试
        retry: (failureCount, error) => {
            if (error.status === 404) {
                return false; // 404不重试
            }
            return failureCount < 3;
        }
    });
}

export default useRetryQuery;
```

---

## 4. 变更操作

### 4.1 useMutation基础

```javascript
// hooks/useCreateUser.js
/**
 * 创建用户Mutation
 * @author erik.zhou
 */
import { useMutation, useQueryClient } from '@tanstack/react-query';

async function createUser(userData) {
    const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
        throw new Error('创建用户失败');
    }
    
    return response.json();
}

function useCreateUser() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: createUser,
        onSuccess: () => {
            // 使相关查询失效，触发重新获取
            queryClient.invalidateQueries({ queryKey: ['users'] });
        }
    });
}

export default useCreateUser;
```

```javascript
// components/CreateUserForm.jsx
/**
 * 创建用户表单
 * @author erik.zhou
 */
import React, { useState } from 'react';
import useCreateUser from '../hooks/useCreateUser';

function CreateUserForm() {
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');
    
    const { mutate, isPending, isError, error, isSuccess } = useCreateUser();
    
    const handleSubmit = (e) => {
        e.preventDefault();
        mutate({ name, email });
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                value={name}
                onChange={(e) => setName(e.target.value)}
                placeholder="姓名"
            />
            <input
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                placeholder="邮箱"
            />
            <button type="submit" disabled={isPending}>
                {isPending ? '创建中...' : '创建用户'}
            </button>
            {isError && <div>错误: {error.message}</div>}
            {isSuccess && <div>创建成功！</div>}
        </form>
    );
}

export default CreateUserForm;
```

### 4.2 更新和删除

```javascript
// hooks/useUserMutations.js
/**
 * 用户CRUD Mutations
 * @author erik.zhou
 */
import { useMutation, useQueryClient } from '@tanstack/react-query';

// 更新用户
function useUpdateUser() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: ({ id, data }) => 
            fetch(`/api/users/${id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data)
            }).then(res => res.json()),
        onSuccess: (data, variables) => {
            // 更新缓存中的用户数据
            queryClient.setQueryData(['user', variables.id], data);
            // 使用户列表失效
            queryClient.invalidateQueries({ queryKey: ['users'] });
        }
    });
}

// 删除用户
function useDeleteUser() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: (userId) =>
            fetch(`/api/users/${userId}`, {
                method: 'DELETE'
            }),
        onSuccess: (data, userId) => {
            // 从缓存中移除用户
            queryClient.removeQueries({ queryKey: ['user', userId] });
            // 使用户列表失效
            queryClient.invalidateQueries({ queryKey: ['users'] });
        }
    });
}

export { useUpdateUser, useDeleteUser };
```

### 4.3 Mutation回调

```javascript
// hooks/useMutationCallbacks.js
/**
 * Mutation回调示例
 * @author erik.zhou
 */
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useMutationCallbacks() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: createUser,
        
        // 开始前
        onMutate: async (newUser) => {
            console.log('开始创建用户:', newUser);
            
            // 取消正在进行的查询
            await queryClient.cancelQueries({ queryKey: ['users'] });
            
            // 保存之前的数据（用于回滚）
            const previousUsers = queryClient.getQueryData(['users']);
            
            return { previousUsers };
        },
        
        // 成功时
        onSuccess: (data, variables, context) => {
            console.log('创建成功:', data);
            queryClient.invalidateQueries({ queryKey: ['users'] });
        },
        
        // 失败时
        onError: (error, variables, context) => {
            console.error('创建失败:', error);
            // 回滚到之前的数据
            if (context?.previousUsers) {
                queryClient.setQueryData(['users'], context.previousUsers);
            }
        },
        
        // 无论成功失败都执行
        onSettled: (data, error, variables, context) => {
            console.log('操作完成');
        }
    });
}

export default useMutationCallbacks;
```

---

## 5. 缓存策略

### 5.1 缓存时间配置

```javascript
// config/queryClient.js
/**
 * QueryClient缓存配置
 * @author erik.zhou
 */
import { QueryClient } from '@tanstack/react-query';

const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            // staleTime: 数据被认为是新鲜的时间
            staleTime: 1000 * 60 * 5, // 5分钟
            
            // cacheTime: 数据在缓存中保留的时间
            cacheTime: 1000 * 60 * 10, // 10分钟
            
            // 窗口重新获得焦点时重新获取
            refetchOnWindowFocus: true,
            
            // 组件挂载时重新获取
            refetchOnMount: true,
            
            // 网络重新连接时重新获取
            refetchOnReconnect: true,
            
            // 重试次数
            retry: 3,
            
            // 重试延迟
            retryDelay: (attemptIndex) => 
                Math.min(1000 * 2 ** attemptIndex, 30000)
        }
    }
});

export default queryClient;
```

### 5.2 手动缓存管理

```javascript
// hooks/useCacheManagement.js
/**
 * 手动缓存管理
 * @author erik.zhou
 */
import { useQueryClient } from '@tanstack/react-query';

function useCacheManagement() {
    const queryClient = useQueryClient();
    
    // 获取缓存数据
    const getCachedData = (queryKey) => {
        return queryClient.getQueryData(queryKey);
    };
    
    // 设置缓存数据
    const setCachedData = (queryKey, data) => {
        queryClient.setQueryData(queryKey, data);
    };
    
    // 更新缓存数据
    const updateCachedData = (queryKey, updater) => {
        queryClient.setQueryData(queryKey, (oldData) => {
            return updater(oldData);
        });
    };
    
    // 使缓存失效
    const invalidateCache = (queryKey) => {
        queryClient.invalidateQueries({ queryKey });
    };
    
    // 移除缓存
    const removeCache = (queryKey) => {
        queryClient.removeQueries({ queryKey });
    };
    
    // 重置所有缓存
    const resetAllCache = () => {
        queryClient.clear();
    };
    
    return {
        getCachedData,
        setCachedData,
        updateCachedData,
        invalidateCache,
        removeCache,
        resetAllCache
    };
}

export default useCacheManagement;
```

---

## 6. 乐观更新

### 6.1 基础乐观更新

```javascript
// hooks/useOptimisticUpdate.js
/**
 * 乐观更新示例
 * @author erik.zhou
 */
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useOptimisticUpdate() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: updateTodo,
        
        onMutate: async (newTodo) => {
            // 取消正在进行的查询
            await queryClient.cancelQueries({ queryKey: ['todos'] });
            
            // 保存之前的数据
            const previousTodos = queryClient.getQueryData(['todos']);
            
            // 乐观更新：立即更新UI
            queryClient.setQueryData(['todos'], (old) => {
                return old.map(todo => 
                    todo.id === newTodo.id ? newTodo : todo
                );
            });
            
            // 返回上下文对象（用于回滚）
            return { previousTodos };
        },
        
        onError: (err, newTodo, context) => {
            // 发生错误时回滚
            queryClient.setQueryData(['todos'], context.previousTodos);
        },
        
        onSettled: () => {
            // 无论成功失败，都重新获取数据
            queryClient.invalidateQueries({ queryKey: ['todos'] });
        }
    });
}

export default useOptimisticUpdate;
```

### 6.2 复杂乐观更新

```javascript
// hooks/useComplexOptimisticUpdate.js
/**
 * 复杂乐观更新（列表添加）
 * @author erik.zhou
 */
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useAddTodo() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: createTodo,
        
        onMutate: async (newTodo) => {
            await queryClient.cancelQueries({ queryKey: ['todos'] });
            
            const previousTodos = queryClient.getQueryData(['todos']);
            
            // 乐观添加新项（使用临时ID）
            queryClient.setQueryData(['todos'], (old) => [
                ...old,
                { ...newTodo, id: `temp-${Date.now()}`, status: 'pending' }
            ]);
            
            return { previousTodos };
        },
        
        onSuccess: (data) => {
            // 成功后替换临时数据
            queryClient.setQueryData(['todos'], (old) => 
                old.map(todo => 
                    todo.id.toString().startsWith('temp-') ? data : todo
                )
            );
        },
        
        onError: (err, newTodo, context) => {
            queryClient.setQueryData(['todos'], context.previousTodos);
        }
    });
}

export default useAddTodo;
```

### 6.3 错误回滚

```javascript
// hooks/useOptimisticWithRollback.js
/**
 * 带回滚的乐观更新
 * @author erik.zhou
 */
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { toast } from 'react-toastify';

function useOptimisticWithRollback() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: async (updatedItem) => {
            const response = await fetch(`/api/items/${updatedItem.id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(updatedItem)
            });
            
            if (!response.ok) {
                throw new Error('更新失败');
            }
            
            return response.json();
        },
        
        onMutate: async (updatedItem) => {
            await queryClient.cancelQueries({ queryKey: ['items'] });
            
            const previousItems = queryClient.getQueryData(['items']);
            
            // 乐观更新
            queryClient.setQueryData(['items'], (old) => 
                old.map(item => 
                    item.id === updatedItem.id ? updatedItem : item
                )
            );
            
            // 显示乐观更新提示
            toast.info('正在保存...');
            
            return { previousItems };
        },
        
        onSuccess: () => {
            toast.success('保存成功！');
        },
        
        onError: (err, updatedItem, context) => {
            // 回滚数据
            queryClient.setQueryData(['items'], context.previousItems);
            toast.error(`保存失败: ${err.message}`);
        },
        
        onSettled: () => {
            queryClient.invalidateQueries({ queryKey: ['items'] });
        }
    });
}

export default useOptimisticWithRollback;
```

```javascript
// components/OptimisticUpdateDemo.jsx
/**
 * 乐观更新演示组件
 * @author erik.zhou
 */
import React from 'react';
import { useQuery } from '@tanstack/react-query';
import useOptimisticWithRollback from '../hooks/useOptimisticWithRollback';

function OptimisticUpdateDemo() {
    const { data: items } = useQuery({
        queryKey: ['items'],
        queryFn: fetchItems
    });
    
    const updateMutation = useOptimisticWithRollback();
    
    const handleToggle = (item) => {
        updateMutation.mutate({
            ...item,
            completed: !item.completed
        });
    };
    
    return (
        <div>
            <h2>待办事项</h2>
            {items?.map(item => (
                <div key={item.id}>
                    <input
                        type="checkbox"
                        checked={item.completed}
                        onChange={() => handleToggle(item)}
                    />
                    <span style={{ 
                        textDecoration: item.completed ? 'line-through' : 'none' 
                    }}>
                        {item.title}
                    </span>
                </div>
            ))}
        </div>
    );
}

export default OptimisticUpdateDemo;
```

---

## 7. 无限滚动

### 7.1 useInfiniteQuery基础

```javascript
// hooks/useInfiniteUsers.js
/**
 * 无限滚动用户列表
 * @author erik.zhou
 */
import { useInfiniteQuery } from '@tanstack/react-query';

async function fetchUsers({ pageParam = 1 }) {
    const response = await fetch(`/api/users?page=${pageParam}&limit=10`);
    if (!response.ok) {
        throw new Error('获取用户失败');
    }
    return response.json();
}

function useInfiniteUsers() {
    return useInfiniteQuery({
        queryKey: ['users', 'infinite'],
        queryFn: fetchUsers,
        getNextPageParam: (lastPage, allPages) => {
            // 如果还有更多数据，返回下一页页码
            return lastPage.hasMore ? allPages.length + 1 : undefined;
        },
        getPreviousPageParam: (firstPage, allPages) => {
            // 如果需要向前加载
            return firstPage.page > 1 ? firstPage.page - 1 : undefined;
        }
    });
}

export default useInfiniteUsers;
```

### 7.2 分页加载

```javascript
// components/InfiniteUserList.jsx
/**
 * 无限滚动用户列表组件
 * @author erik.zhou
 */
import React from 'react';
import useInfiniteUsers from '../hooks/useInfiniteUsers';

function InfiniteUserList() {
    const {
        data,
        fetchNextPage,
        hasNextPage,
        isFetchingNextPage,
        isLoading,
        isError,
        error
    } = useInfiniteUsers();
    
    if (isLoading) {
        return <div>加载中...</div>;
    }
    
    if (isError) {
        return <div>错误: {error.message}</div>;
    }
    
    return (
        <div>
            <h2>用户列表</h2>
            {data.pages.map((page, pageIndex) => (
                <div key={pageIndex}>
                    {page.users.map(user => (
                        <div key={user.id}>
                            <h3>{user.name}</h3>
                            <p>{user.email}</p>
                        </div>
                    ))}
                </div>
            ))}
            
            <button
                onClick={() => fetchNextPage()}
                disabled={!hasNextPage || isFetchingNextPage}
            >
                {isFetchingNextPage
                    ? '加载中...'
                    : hasNextPage
                    ? '加载更多'
                    : '没有更多了'}
            </button>
        </div>
    );
}

export default InfiniteUserList;
```

### 7.3 滚动触发加载

```javascript
// hooks/useInfiniteScroll.js
/**
 * 滚动触发无限加载Hook
 * @author erik.zhou
 */
import { useEffect, useRef } from 'react';
import { useInfiniteQuery } from '@tanstack/react-query';

function useInfiniteScroll(queryKey, queryFn) {
    const observerRef = useRef();
    
    const {
        data,
        fetchNextPage,
        hasNextPage,
        isFetchingNextPage
    } = useInfiniteQuery({
        queryKey,
        queryFn,
        getNextPageParam: (lastPage) => lastPage.nextCursor
    });
    
    useEffect(() => {
        const observer = new IntersectionObserver(
            (entries) => {
                if (entries[0].isIntersecting && hasNextPage && !isFetchingNextPage) {
                    fetchNextPage();
                }
            },
            { threshold: 1.0 }
        );
        
        if (observerRef.current) {
            observer.observe(observerRef.current);
        }
        
        return () => observer.disconnect();
    }, [fetchNextPage, hasNextPage, isFetchingNextPage]);
    
    return { data, observerRef, isFetchingNextPage };
}

export default useInfiniteScroll;
```

```javascript
// components/ScrollLoadList.jsx
/**
 * 滚动加载列表组件
 * @author erik.zhou
 */
import React from 'react';
import useInfiniteScroll from '../hooks/useInfiniteScroll';

async function fetchPosts({ pageParam = 0 }) {
    const response = await fetch(`/api/posts?cursor=${pageParam}`);
    return response.json();
}

function ScrollLoadList() {
    const { data, observerRef, isFetchingNextPage } = useInfiniteScroll(
        ['posts', 'infinite'],
        fetchPosts
    );
    
    return (
        <div>
            <h2>文章列表</h2>
            {data?.pages.map((page, pageIndex) => (
                <div key={pageIndex}>
                    {page.posts.map(post => (
                        <article key={post.id}>
                            <h3>{post.title}</h3>
                            <p>{post.content}</p>
                        </article>
                    ))}
                </div>
            ))}
            
            {/* 观察目标元素 */}
            <div ref={observerRef} style={{ height: '20px' }}>
                {isFetchingNextPage && <div>加载更多...</div>}
            </div>
        </div>
    );
}

export default ScrollLoadList;
```

---

## 8. 性能优化

### 8.1 选择器优化

```javascript
// hooks/useOptimizedQuery.js
/**
 * 使用选择器优化性能
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';

function useOptimizedQuery() {
    // 不使用选择器（每次都重新渲染）
    const { data: allUsers } = useQuery({
        queryKey: ['users'],
        queryFn: fetchUsers
    });
    
    // 使用选择器（只在选中的数据变化时重新渲染）
    const { data: activeUsers } = useQuery({
        queryKey: ['users'],
        queryFn: fetchUsers,
        select: (data) => data.filter(user => user.active)
    });
    
    return { allUsers, activeUsers };
}

export default useOptimizedQuery;
```

```javascript
// hooks/useSelectOptimization.js
/**
 * 选择器优化示例
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';
import { useMemo } from 'react';

function useSelectOptimization(userId) {
    // 方式1：使用select选择器
    const { data: user } = useQuery({
        queryKey: ['users'],
        queryFn: fetchUsers,
        select: (users) => users.find(u => u.id === userId)
    });
    
    // 方式2：使用useMemo（不推荐）
    const { data: allUsers } = useQuery({
        queryKey: ['users'],
        queryFn: fetchUsers
    });
    
    const userMemo = useMemo(() => {
        return allUsers?.find(u => u.id === userId);
    }, [allUsers, userId]);
    
    return { user, userMemo };
}

export default useSelectOptimization;
```

### 8.2 预取数据

```javascript
// hooks/usePrefetch.js
/**
 * 数据预取
 * @author erik.zhou
 */
import { useQueryClient } from '@tanstack/react-query';

function usePrefetch() {
    const queryClient = useQueryClient();
    
    // 预取数据
    const prefetchUser = (userId) => {
        queryClient.prefetchQuery({
            queryKey: ['user', userId],
            queryFn: () => fetchUser(userId),
            staleTime: 1000 * 60 * 5 // 5分钟内不会重新获取
        });
    };
    
    // 确保数据存在
    const ensureUserData = async (userId) => {
        await queryClient.ensureQueryData({
            queryKey: ['user', userId],
            queryFn: () => fetchUser(userId)
        });
    };
    
    return { prefetchUser, ensureUserData };
}

export default usePrefetch;
```

```javascript
// components/UserListWithPrefetch.jsx
/**
 * 带预取的用户列表
 * @author erik.zhou
 */
import React from 'react';
import { useQuery } from '@tanstack/react-query';
import { Link } from 'react-router-dom';
import usePrefetch from '../hooks/usePrefetch';

function UserListWithPrefetch() {
    const { data: users } = useQuery({
        queryKey: ['users'],
        queryFn: fetchUsers
    });
    
    const { prefetchUser } = usePrefetch();
    
    return (
        <div>
            <h2>用户列表</h2>
            {users?.map(user => (
                <Link
                    key={user.id}
                    to={`/users/${user.id}`}
                    onMouseEnter={() => prefetchUser(user.id)}
                >
                    {user.name}
                </Link>
            ))}
        </div>
    );
}

export default UserListWithPrefetch;
```

### 8.3 查询取消

```javascript
// hooks/useCancellableQuery.js
/**
 * 可取消的查询
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';

async function fetchWithCancel({ queryKey, signal }) {
    const [, userId] = queryKey;
    
    const response = await fetch(`/api/users/${userId}`, {
        signal // 传递AbortSignal
    });
    
    if (!response.ok) {
        throw new Error('获取用户失败');
    }
    
    return response.json();
}

function useCancellableQuery(userId) {
    return useQuery({
        queryKey: ['user', userId],
        queryFn: fetchWithCancel,
        enabled: !!userId
    });
}

export default useCancellableQuery;
```

```javascript
// hooks/useSearchWithDebounce.js
/**
 * 带防抖的搜索查询
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';
import { useState, useEffect } from 'react';

function useSearchWithDebounce(searchTerm, delay = 500) {
    const [debouncedTerm, setDebouncedTerm] = useState(searchTerm);
    
    useEffect(() => {
        const timer = setTimeout(() => {
            setDebouncedTerm(searchTerm);
        }, delay);
        
        return () => clearTimeout(timer);
    }, [searchTerm, delay]);
    
    return useQuery({
        queryKey: ['search', debouncedTerm],
        queryFn: () => searchUsers(debouncedTerm),
        enabled: debouncedTerm.length > 0
    });
}

export default useSearchWithDebounce;
```

---

## 9. 最佳实践

### 9.1 查询键管理

```javascript
// constants/queryKeys.js
/**
 * 统一管理查询键
 * @author erik.zhou
 */

const queryKeys = {
    // 用户相关
    users: {
        all: ['users'],
        lists: () => [...queryKeys.users.all, 'list'],
        list: (filters) => [...queryKeys.users.lists(), { filters }],
        details: () => [...queryKeys.users.all, 'detail'],
        detail: (id) => [...queryKeys.users.details(), id],
        search: (term) => [...queryKeys.users.all, 'search', term]
    },
    
    // 文章相关
    posts: {
        all: ['posts'],
        lists: () => [...queryKeys.posts.all, 'list'],
        list: (filters) => [...queryKeys.posts.lists(), { filters }],
        details: () => [...queryKeys.posts.all, 'detail'],
        detail: (id) => [...queryKeys.posts.details(), id],
        infinite: (filters) => [...queryKeys.posts.all, 'infinite', { filters }]
    },
    
    // 评论相关
    comments: {
        all: ['comments'],
        list: (postId) => [...queryKeys.comments.all, 'list', postId],
        detail: (id) => [...queryKeys.comments.all, 'detail', id]
    }
};

export default queryKeys;
```

### 9.2 错误处理

```javascript
// utils/errorHandler.js
/**
 * 统一错误处理
 * @author erik.zhou
 */

class ApiError extends Error {
    constructor(message, status, data) {
        super(message);
        this.status = status;
        this.data = data;
    }
}

async function handleResponse(response) {
    if (!response.ok) {
        const error = await response.json();
        throw new ApiError(
            error.message || '请求失败',
            response.status,
            error
        );
    }
    return response.json();
}

export { ApiError, handleResponse };
```

```javascript
// hooks/useErrorHandling.js
/**
 * 错误处理Hook
 * @author erik.zhou
 */
import { useQuery } from '@tanstack/react-query';
import { handleResponse } from '../utils/errorHandler';
import { toast } from 'react-toastify';

function useErrorHandling() {
    return useQuery({
        queryKey: ['data'],
        queryFn: async () => {
            const response = await fetch('/api/data');
            return handleResponse(response);
        },
        onError: (error) => {
            // 根据错误类型显示不同提示
            if (error.status === 401) {
                toast.error('请先登录');
                // 跳转到登录页
            } else if (error.status === 403) {
                toast.error('没有权限');
            } else if (error.status === 404) {
                toast.error('数据不存在');
            } else if (error.status >= 500) {
                toast.error('服务器错误，请稍后重试');
            } else {
                toast.error(error.message);
            }
        },
        retry: (failureCount, error) => {
            // 4xx错误不重试
            if (error.status >= 400 && error.status < 500) {
                return false;
            }
            return failureCount < 3;
        }
    });
}

export default useErrorHandling;
```

### 9.3 TypeScript集成

```typescript
// types/api.ts
/**
 * API类型定义
 * @author erik.zhou
 */

interface User {
    id: number;
    name: string;
    email: string;
    avatar?: string;
}

interface PaginatedResponse<T> {
    data: T[];
    total: number;
    page: number;
    pageSize: number;
}

interface ApiError {
    message: string;
    code: string;
    details?: Record<string, any>;
}

export type { User, PaginatedResponse, ApiError };
```

```typescript
// hooks/useTypedQuery.ts
/**
 * 类型安全的查询Hook
 * @author erik.zhou
 */
import { useQuery, UseQueryOptions } from '@tanstack/react-query';
import type { User, ApiError } from '../types/api';

async function fetchUser(userId: number): Promise<User> {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
        throw new Error('获取用户失败');
    }
    return response.json();
}

function useTypedQuery(
    userId: number,
    options?: Omit<UseQueryOptions<User, ApiError>, 'queryKey' | 'queryFn'>
) {
    return useQuery<User, ApiError>({
        queryKey: ['user', userId],
        queryFn: () => fetchUser(userId),
        ...options
    });
}

export default useTypedQuery;
```

```typescript
// hooks/useTypedMutation.ts
/**
 * 类型安全的Mutation Hook
 * @author erik.zhou
 */
import { useMutation, UseMutationOptions } from '@tanstack/react-query';
import type { User, ApiError } from '../types/api';

interface CreateUserInput {
    name: string;
    email: string;
}

async function createUser(input: CreateUserInput): Promise<User> {
    const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(input)
    });
    
    if (!response.ok) {
        throw new Error('创建用户失败');
    }
    
    return response.json();
}

function useTypedMutation(
    options?: UseMutationOptions<User, ApiError, CreateUserInput>
) {
    return useMutation<User, ApiError, CreateUserInput>({
        mutationFn: createUser,
        ...options
    });
}

export default useTypedMutation;
```

---

## 10. 实战案例

### 10.1 用户管理系统

```javascript
// features/users/api.js
/**
 * 用户API
 * @author erik.zhou
 */

const userApi = {
    getUsers: async (filters) => {
        const params = new URLSearchParams(filters);
        const response = await fetch(`/api/users?${params}`);
        if (!response.ok) throw new Error('获取用户列表失败');
        return response.json();
    },
    
    getUser: async (userId) => {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('获取用户失败');
        return response.json();
    },
    
    createUser: async (userData) => {
        const response = await fetch('/api/users', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(userData)
        });
        if (!response.ok) throw new Error('创建用户失败');
        return response.json();
    },
    
    updateUser: async ({ id, data }) => {
        const response = await fetch(`/api/users/${id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        });
        if (!response.ok) throw new Error('更新用户失败');
        return response.json();
    },
    
    deleteUser: async (userId) => {
        const response = await fetch(`/api/users/${userId}`, {
            method: 'DELETE'
        });
        if (!response.ok) throw new Error('删除用户失败');
    }
};

export default userApi;
```

```javascript
// features/users/hooks.js
/**
 * 用户相关Hooks
 * @author erik.zhou
 */
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import userApi from './api';
import queryKeys from '../../constants/queryKeys';

function useUsers(filters) {
    return useQuery({
        queryKey: queryKeys.users.list(filters),
        queryFn: () => userApi.getUsers(filters),
        keepPreviousData: true
    });
}

function useUser(userId) {
    return useQuery({
        queryKey: queryKeys.users.detail(userId),
        queryFn: () => userApi.getUser(userId),
        enabled: !!userId
    });
}

function useCreateUser() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: userApi.createUser,
        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: queryKeys.users.all });
        }
    });
}

function useUpdateUser() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: userApi.updateUser,
        onMutate: async ({ id, data }) => {
            await queryClient.cancelQueries({ queryKey: queryKeys.users.detail(id) });
            
            const previousUser = queryClient.getQueryData(queryKeys.users.detail(id));
            
            queryClient.setQueryData(queryKeys.users.detail(id), (old) => ({
                ...old,
                ...data
            }));
            
            return { previousUser };
        },
        onError: (err, { id }, context) => {
            queryClient.setQueryData(
                queryKeys.users.detail(id),
                context.previousUser
            );
        },
        onSettled: (data, error, { id }) => {
            queryClient.invalidateQueries({ queryKey: queryKeys.users.detail(id) });
            queryClient.invalidateQueries({ queryKey: queryKeys.users.lists() });
        }
    });
}

function useDeleteUser() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: userApi.deleteUser,
        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: queryKeys.users.all });
        }
    });
}

export { useUsers, useUser, useCreateUser, useUpdateUser, useDeleteUser };
```

```javascript
// features/users/UserManagement.jsx
/**
 * 用户管理主组件
 * @author erik.zhou
 */
import React, { useState } from 'react';
import { useUsers, useCreateUser, useUpdateUser, useDeleteUser } from './hooks';

function UserManagement() {
    const [filters, setFilters] = useState({ page: 1, search: '' });
    const [editingUser, setEditingUser] = useState(null);
    
    const { data, isLoading, isError, error } = useUsers(filters);
    const createMutation = useCreateUser();
    const updateMutation = useUpdateUser();
    const deleteMutation = useDeleteUser();
    
    const handleCreate = (userData) => {
        createMutation.mutate(userData, {
            onSuccess: () => {
                alert('创建成功');
            }
        });
    };
    
    const handleUpdate = (id, userData) => {
        updateMutation.mutate({ id, data: userData }, {
            onSuccess: () => {
                setEditingUser(null);
                alert('更新成功');
            }
        });
    };
    
    const handleDelete = (userId) => {
        if (window.confirm('确定删除？')) {
            deleteMutation.mutate(userId, {
                onSuccess: () => {
                    alert('删除成功');
                }
            });
        }
    };
    
    if (isLoading) return <div>加载中...</div>;
    if (isError) return <div>错误: {error.message}</div>;
    
    return (
        <div>
            <h1>用户管理</h1>
            
            {/* 搜索框 */}
            <input
                type="text"
                placeholder="搜索用户..."
                value={filters.search}
                onChange={(e) => setFilters({ ...filters, search: e.target.value })}
            />
            
            {/* 用户列表 */}
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>姓名</th>
                        <th>邮箱</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody>
                    {data?.users.map(user => (
                        <tr key={user.id}>
                            <td>{user.id}</td>
                            <td>{user.name}</td>
                            <td>{user.email}</td>
                            <td>
                                <button onClick={() => setEditingUser(user)}>
                                    编辑
                                </button>
                                <button onClick={() => handleDelete(user.id)}>
                                    删除
                                </button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>
            
            {/* 分页 */}
            <div>
                <button
                    disabled={filters.page === 1}
                    onClick={() => setFilters({ ...filters, page: filters.page - 1 })}
                >
                    上一页
                </button>
                <span>第 {filters.page} 页</span>
                <button
                    disabled={!data?.hasMore}
                    onClick={() => setFilters({ ...filters, page: filters.page + 1 })}
                >
                    下一页
                </button>
            </div>
        </div>
    );
}

export default UserManagement;
```

### 10.2 文章列表与详情

```javascript
// features/posts/hooks.js
/**
 * 文章相关Hooks
 * @author erik.zhou
 */
import { useQuery, useInfiniteQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import queryKeys from '../../constants/queryKeys';

async function fetchPosts({ pageParam = 1 }) {
    const response = await fetch(`/api/posts?page=${pageParam}&limit=10`);
    if (!response.ok) throw new Error('获取文章失败');
    return response.json();
}

async function fetchPost(postId) {
    const response = await fetch(`/api/posts/${postId}`);
    if (!response.ok) throw new Error('获取文章失败');
    return response.json();
}

function useInfinitePosts() {
    return useInfiniteQuery({
        queryKey: queryKeys.posts.infinite({}),
        queryFn: fetchPosts,
        getNextPageParam: (lastPage) => {
            return lastPage.hasMore ? lastPage.nextPage : undefined;
        }
    });
}

function usePost(postId) {
    return useQuery({
        queryKey: queryKeys.posts.detail(postId),
        queryFn: () => fetchPost(postId),
        enabled: !!postId
    });
}

function useLikePost() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: async (postId) => {
            const response = await fetch(`/api/posts/${postId}/like`, {
                method: 'POST'
            });
            return response.json();
        },
        onMutate: async (postId) => {
            await queryClient.cancelQueries({ 
                queryKey: queryKeys.posts.detail(postId) 
            });
            
            const previousPost = queryClient.getQueryData(
                queryKeys.posts.detail(postId)
            );
            
            queryClient.setQueryData(queryKeys.posts.detail(postId), (old) => ({
                ...old,
                likes: old.likes + 1,
                isLiked: true
            }));
            
            return { previousPost };
        },
        onError: (err, postId, context) => {
            queryClient.setQueryData(
                queryKeys.posts.detail(postId),
                context.previousPost
            );
        }
    });
}

export { useInfinitePosts, usePost, useLikePost };
```

```javascript
// features/posts/PostList.jsx
/**
 * 文章列表组件（无限滚动）
 * @author erik.zhou
 */
import React, { useRef, useEffect } from 'react';
import { useInfinitePosts, useLikePost } from './hooks';

function PostList() {
    const {
        data,
        fetchNextPage,
        hasNextPage,
        isFetchingNextPage,
        isLoading
    } = useInfinitePosts();
    
    const likeMutation = useLikePost();
    const observerRef = useRef();
    
    useEffect(() => {
        const observer = new IntersectionObserver(
            (entries) => {
                if (entries[0].isIntersecting && hasNextPage && !isFetchingNextPage) {
                    fetchNextPage();
                }
            },
            { threshold: 1.0 }
        );
        
        if (observerRef.current) {
            observer.observe(observerRef.current);
        }
        
        return () => observer.disconnect();
    }, [fetchNextPage, hasNextPage, isFetchingNextPage]);
    
    if (isLoading) {
        return <div>加载中...</div>;
    }
    
    return (
        <div>
            <h1>文章列表</h1>
            {data?.pages.map((page, pageIndex) => (
                <div key={pageIndex}>
                    {page.posts.map(post => (
                        <article key={post.id}>
                            <h2>{post.title}</h2>
                            <p>{post.content}</p>
                            <button
                                onClick={() => likeMutation.mutate(post.id)}
                                disabled={likeMutation.isPending}
                            >
                                ❤️ {post.likes}
                            </button>
                        </article>
                    ))}
                </div>
            ))}
            
            <div ref={observerRef} style={{ height: '20px' }}>
                {isFetchingNextPage && <div>加载更多...</div>}
            </div>
        </div>
    );
}

export default PostList;
```

### 10.3 实时数据同步

```javascript
// features/realtime/hooks.js
/**
 * 实时数据同步Hook
 * @author erik.zhou
 */
import { useQuery, useQueryClient } from '@tanstack/react-query';
import { useEffect } from 'react';

function useRealtimeData(channel) {
    const queryClient = useQueryClient();
    
    const { data } = useQuery({
        queryKey: ['realtime', channel],
        queryFn: () => fetchInitialData(channel),
        staleTime: Infinity // 数据永不过期，通过WebSocket更新
    });
    
    useEffect(() => {
        // 建立WebSocket连接
        const ws = new WebSocket(`ws://api.example.com/${channel}`);
        
        ws.onmessage = (event) => {
            const update = JSON.parse(event.data);
            
            // 更新缓存数据
            queryClient.setQueryData(['realtime', channel], (old) => {
                if (!old) {
                    return update;
                }
                
                // 根据更新类型处理数据
                switch (update.type) {
                    case 'add':
                        return [...old, update.data];
                    case 'update':
                        return old.map(item =>
                            item.id === update.data.id ? update.data : item
                        );
                    case 'delete':
                        return old.filter(item => item.id !== update.data.id);
                    default:
                        return old;
                }
            });
        };
        
        ws.onerror = (error) => {
            console.error('WebSocket错误:', error);
        };
        
        return () => {
            ws.close();
        };
    }, [channel, queryClient]);
    
    return { data };
}

export default useRealtimeData;
```

```javascript
// features/realtime/RealtimeList.jsx
/**
 * 实时数据列表组件
 * @author erik.zhou
 */
import React from 'react';
import useRealtimeData from './hooks';

function RealtimeList() {
    const { data: messages } = useRealtimeData('chat');
    
    return (
        <div>
            <h2>实时消息</h2>
            <div>
                {messages?.map(message => (
                    <div key={message.id}>
                        <strong>{message.user}:</strong> {message.text}
                        <span>{new Date(message.timestamp).toLocaleTimeString()}</span>
                    </div>
                ))}
            </div>
        </div>
    );
}

export default RealtimeList;
```

---

## 总结

### 核心要点

1. **查询管理**
   - 使用 useQuery 获取数据
   - 合理设置 staleTime 和 cacheTime
   - 使用查询键管理缓存
   - 利用 select 优化性能

2. **变更操作**
   - 使用 useMutation 修改数据
   - 配合 onSuccess 更新缓存
   - 实现乐观更新提升体验
   - 错误时回滚数据

3. **无限滚动**
   - 使用 useInfiniteQuery
   - 配置 getNextPageParam
   - 结合 IntersectionObserver
   - 优化加载体验

4. **性能优化**
   - 使用选择器减少重渲染
   - 预取数据提升响应速度
   - 取消不必要的请求
   - 合理配置缓存策略

5. **最佳实践**
   - 统一管理查询键
   - 封装通用 Hook
   - 完善错误处理
   - TypeScript 类型安全

### 学习路径

1. **基础阶段**（1-2天）
   - 理解 React Query 核心概念
   - 掌握 useQuery 和 useMutation
   - 学习查询键管理
   - 了解缓存机制

2. **进阶阶段**（3-5天）
   - 掌握乐观更新
   - 学习无限滚动
   - 理解性能优化技巧
   - 实践错误处理

3. **实战阶段**（1-2周）
   - 构建完整的数据管理系统
   - 实现复杂的业务场景
   - 优化应用性能
   - 集成 TypeScript

4. **高级阶段**（持续学习）
   - 深入理解缓存策略
   - 掌握高级优化技巧
   - 学习源码实现
   - 探索最佳实践

### 推荐资源

**官方文档**
- [React Query 官方文档](https://tanstack.com/query/latest)
- [React Query DevTools](https://tanstack.com/query/latest/docs/devtools)
- [示例代码库](https://github.com/TanStack/query/tree/main/examples)

**学习资源**
- [React Query 教程系列](https://tkdodo.eu/blog/practical-react-query)
- [React Query 最佳实践](https://tkdodo.eu/blog/react-query-best-practices)
- [性能优化指南](https://tanstack.com/query/latest/docs/guides/performance)

**社区资源**
- [Discord 社区](https://discord.com/invite/WrRKjPJ)
- [GitHub Discussions](https://github.com/TanStack/query/discussions)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/react-query)

**视频教程**
- [React Query 入门教程](https://www.youtube.com/watch?v=DocXo3gqGdI)
- [React Query 进阶技巧](https://www.youtube.com/watch?v=r8Dg0KVnfMA)
- [实战项目开发](https://www.youtube.com/watch?v=lVLz_ASqAio)

---

## 附录

### A. 常用API速查

```javascript
// useQuery
const { data, isLoading, isError, error, refetch } = useQuery({
    queryKey: ['key'],
    queryFn: fetchData,
    staleTime: 5000,
    cacheTime: 10000,
    enabled: true,
    retry: 3,
    select: (data) => data.filter(item => item.active)
});

// useMutation
const { mutate, mutateAsync, isPending, isError, error } = useMutation({
    mutationFn: updateData,
    onSuccess: (data) => {},
    onError: (error) => {},
    onMutate: (variables) => {},
    onSettled: () => {}
});

// useInfiniteQuery
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey: ['key'],
    queryFn: fetchData,
    getNextPageParam: (lastPage) => lastPage.nextCursor
});

// QueryClient
queryClient.invalidateQueries({ queryKey: ['key'] });
queryClient.setQueryData(['key'], newData);
queryClient.getQueryData(['key']);
queryClient.removeQueries({ queryKey: ['key'] });
queryClient.clear();
```

### B. 配置模板

```javascript
// config/queryClient.js
/**
 * QueryClient完整配置模板
 * @author erik.zhou
 */
import { QueryClient } from '@tanstack/react-query';

const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            // 数据新鲜时间（毫秒）
            staleTime: 1000 * 60 * 5,
            
            // 缓存时间（毫秒）
            cacheTime: 1000 * 60 * 10,
            
            // 重试次数
            retry: 3,
            
            // 重试延迟函数
            retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
            
            // 窗口聚焦时重新获取
            refetchOnWindowFocus: false,
            
            // 组件挂载时重新获取
            refetchOnMount: true,
            
            // 网络重连时重新获取
            refetchOnReconnect: true,
            
            // 保留之前的数据
            keepPreviousData: false,
            
            // 结构化共享
            structuralSharing: true
        },
        mutations: {
            // 重试次数
            retry: 0,
            
            // 重试延迟
            retryDelay: 1000
        }
    }
});

export default queryClient;
```

### C. 常见问题

**Q1: 如何避免重复请求？**

```javascript
// React Query 自动处理重复请求
function Component1() {
    const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
    // ...
}

function Component2() {
    // 不会发起新请求，使用缓存数据
    const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
    // ...
}
```

**Q2: 如何处理依赖查询？**

```javascript
function useDependentQuery(userId) {
    const { data: user } = useQuery({
        queryKey: ['user', userId],
        queryFn: () => fetchUser(userId)
    });
    
    const { data: posts } = useQuery({
        queryKey: ['posts', user?.id],
        queryFn: () => fetchPosts(user.id),
        enabled: !!user?.id // 关键：只有user存在时才执行
    });
    
    return { user, posts };
}
```

**Q3: 如何实现轮询？**

```javascript
function usePolling() {
    return useQuery({
        queryKey: ['data'],
        queryFn: fetchData,
        refetchInterval: 5000, // 每5秒轮询一次
        refetchIntervalInBackground: true // 后台也轮询
    });
}
```

**Q4: 如何取消请求？**

```javascript
async function fetchWithCancel({ queryKey, signal }) {
    const response = await fetch('/api/data', { signal });
    return response.json();
}

function useCancellable() {
    return useQuery({
        queryKey: ['data'],
        queryFn: fetchWithCancel // React Query自动传入signal
    });
}
```

**Q5: 如何处理分页？**

```javascript
function usePagination() {
    const [page, setPage] = useState(1);
    
    return useQuery({
        queryKey: ['data', page],
        queryFn: () => fetchData(page),
        keepPreviousData: true // 保留上一页数据，避免闪烁
    });
}
```

**Q6: 如何预加载数据？**

```javascript
function usePrefetch() {
    const queryClient = useQueryClient();
    
    const prefetch = (id) => {
        queryClient.prefetchQuery({
            queryKey: ['item', id],
            queryFn: () => fetchItem(id)
        });
    };
    
    return { prefetch };
}
```

**Q7: 如何处理乐观更新失败？**

```javascript
function useOptimistic() {
    const queryClient = useQueryClient();
    
    return useMutation({
        mutationFn: updateItem,
        onMutate: async (newItem) => {
            await queryClient.cancelQueries({ queryKey: ['items'] });
            const previous = queryClient.getQueryData(['items']);
            queryClient.setQueryData(['items'], (old) => [...old, newItem]);
            return { previous };
        },
        onError: (err, newItem, context) => {
            // 回滚到之前的数据
            queryClient.setQueryData(['items'], context.previous);
        },
        onSettled: () => {
            // 重新获取确保数据一致
            queryClient.invalidateQueries({ queryKey: ['items'] });
        }
    });
}
```

**Q8: 如何处理并发请求？**

```javascript
function useParallel() {
    const results = useQueries({
        queries: [
            { queryKey: ['users'], queryFn: fetchUsers },
            { queryKey: ['posts'], queryFn: fetchPosts },
            { queryKey: ['comments'], queryFn: fetchComments }
        ]
    });
    
    const isLoading = results.some(r => r.isLoading);
    const data = results.map(r => r.data);
    
    return { data, isLoading };
}
```

**Q9: 如何实现搜索防抖？**

```javascript
function useSearch(term) {
    const [debouncedTerm, setDebouncedTerm] = useState(term);
    
    useEffect(() => {
        const timer = setTimeout(() => setDebouncedTerm(term), 500);
        return () => clearTimeout(timer);
    }, [term]);
    
    return useQuery({
        queryKey: ['search', debouncedTerm],
        queryFn: () => search(debouncedTerm),
        enabled: debouncedTerm.length > 0
    });
}
```

**Q10: 如何处理认证失败？**

```javascript
const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            onError: (error) => {
                if (error.status === 401) {
                    // 清除所有缓存
                    queryClient.clear();
                    // 跳转到登录页
                    window.location.href = '/login';
                }
            }
        }
    }
});
```

### D. 性能优化清单

- [ ] 合理设置 staleTime 和 cacheTime
- [ ] 使用 select 选择器减少重渲染
- [ ] 启用 keepPreviousData 避免加载闪烁
- [ ] 使用 prefetchQuery 预加载数据
- [ ] 实现乐观更新提升用户体验
- [ ] 避免在循环中使用 useQuery
- [ ] 使用 useQueries 处理并发请求
- [ ] 合理使用 enabled 控制查询执行
- [ ] 实现请求取消避免资源浪费
- [ ] 使用 DevTools 监控性能

### E. 调试技巧

```javascript
// 1. 启用 DevTools
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
    return (
        <QueryClientProvider client={queryClient}>
            <YourApp />
            <ReactQueryDevtools initialIsOpen={false} />
        </QueryClientProvider>
    );
}

// 2. 查看查询状态
const query = useQuery({ queryKey: ['data'], queryFn: fetchData });
console.log('Query State:', {
    status: query.status,
    fetchStatus: query.fetchStatus,
    data: query.data,
    error: query.error
});

// 3. 监听查询变化
queryClient.getQueryCache().subscribe((event) => {
    console.log('Query Event:', event);
});

// 4. 查看缓存内容
console.log('Cache:', queryClient.getQueryCache().getAll());

// 5. 手动触发查询
queryClient.refetchQueries({ queryKey: ['data'] });
```

---

## 结语

React Query 是现代 React 应用中不可或缺的数据管理工具。通过本教程的学习，你应该已经掌握了：

- React Query 的核心概念和使用方法
- 查询管理和缓存策略
- 变更操作和乐观更新
- 无限滚动和分页加载
- 性能优化技巧
- 实战项目开发经验

记住，React Query 的强大之处在于它的简单性和灵活性。从简单的数据获取开始，逐步探索更高级的特性，你会发现它能够优雅地解决大多数数据管理问题。

继续实践，不断优化，祝你在 React Query 的学习之路上越走越远！

---

**@author**: erik.zhou  
**最后更新**: 2026-02-27  
**版本**: v1.0.0
