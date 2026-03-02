# Redux Toolkit - 完整教程

## 目录
1. [Redux Toolkit简介](#redux-toolkit简介)
2. [核心API](#核心api)
3. [异步处理](#异步处理)
4. [RTK Query](#rtk-query)
5. [最佳实践](#最佳实践)

## Redux Toolkit简介

### 什么是Redux Toolkit
Redux Toolkit（RTK）是Redux官方推荐的工具集，简化了Redux的使用。

### 安装
```bash
npm install @reduxjs/toolkit react-redux

# 或使用yarn
yarn add @reduxjs/toolkit react-redux
```

### 核心优势
- 简化Store配置
- 内置Immer，简化不可变更新
- 内置Redux Thunk
- 自动生成Action Creators
- 更好的TypeScript支持

## 核心API

### configureStore
```javascript
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counter/counterSlice';
import userReducer from './features/user/userSlice';

const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer
  },
  // 自动包含redux-thunk和devtools
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // 忽略特定action
        ignoredActions: ['your/action/type']
      }
    }),
  devTools: process.env.NODE_ENV !== 'production'
});

export default store;
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### createSlice
```javascript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
  status: 'idle' | 'loading' | 'failed';
}

const initialState: CounterState = {
  value: 0,
  status: 'idle'
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    // 自动生成action creator
    increment: (state) => {
      // 使用Immer，可以直接修改state
      state.value += 1;
    },
    
    decrement: (state) => {
      state.value -= 1;
    },
    
    // 带payload的action
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
    
    // 使用prepare回调自定义payload
    addTodo: {
      reducer: (state, action: PayloadAction<{ id: string; text: string }>) => {
        state.todos.push(action.payload);
      },
      prepare: (text: string) => {
        return {
          payload: {
            id: nanoid(),
            text
          }
        };
      }
    }
  },
  
  // 处理其他slice的actions
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.status = 'idle';
        state.user = action.payload;
      })
      .addCase(fetchUser.rejected, (state) => {
        state.status = 'failed';
      });
  }
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

### 在组件中使用
```jsx
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './counterSlice';
import type { RootState } from '../../app/store';

function Counter() {
  const count = useSelector((state: RootState) => state.counter.value);
  const dispatch = useDispatch();
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
    </div>
  );
}
```

### createSelector（Reselect）
```javascript
import { createSelector } from '@reduxjs/toolkit';
import type { RootState } from '../../app/store';

// 基础selector
const selectTodos = (state: RootState) => state.todos.items;
const selectFilter = (state: RootState) => state.todos.filter;

// 记忆化selector
export const selectFilteredTodos = createSelector(
  [selectTodos, selectFilter],
  (todos, filter) => {
    switch (filter) {
      case 'completed':
        return todos.filter(todo => todo.completed);
      case 'active':
        return todos.filter(todo => !todo.completed);
      default:
        return todos;
    }
  }
);

// 带参数的selector
export const selectTodoById = createSelector(
  [selectTodos, (state: RootState, todoId: string) => todoId],
  (todos, todoId) => todos.find(todo => todo.id === todoId)
);

// 使用
function TodoList() {
  const filteredTodos = useSelector(selectFilteredTodos);
  const todo = useSelector((state) => selectTodoById(state, '123'));
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

## 异步处理

### createAsyncThunk
```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import axios from 'axios';

// 定义异步thunk
export const fetchUserById = createAsyncThunk(
  'users/fetchById',
  async (userId: string, thunkAPI) => {
    try {
      const response = await axios.get(`/api/users/${userId}`);
      return response.data;
    } catch (error) {
      return thunkAPI.rejectWithValue(error.response.data);
    }
  }
);

// 带条件的thunk
export const fetchUserIfNeeded = createAsyncThunk(
  'users/fetchIfNeeded',
  async (userId: string, thunkAPI) => {
    const state = thunkAPI.getState() as RootState;
    const user = state.users.entities[userId];
    
    if (user) {
      return user;
    }
    
    const response = await axios.get(`/api/users/${userId}`);
    return response.data;
  },
  {
    condition: (userId, { getState }) => {
      const state = getState() as RootState;
      const user = state.users.entities[userId];
      
      // 如果已有数据，不发起请求
      if (user) {
        return false;
      }
    }
  }
);

// 在slice中处理
const userSlice = createSlice({
  name: 'users',
  initialState: {
    entities: {},
    loading: 'idle',
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserById.pending, (state) => {
        state.loading = 'pending';
      })
      .addCase(fetchUserById.fulfilled, (state, action) => {
        state.loading = 'idle';
        state.entities[action.payload.id] = action.payload;
      })
      .addCase(fetchUserById.rejected, (state, action) => {
        state.loading = 'idle';
        state.error = action.payload;
      });
  }
});
```

### 使用异步thunk
```jsx
import { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchUserById } from './userSlice';

function UserProfile({ userId }) {
  const dispatch = useDispatch();
  const user = useSelector(state => state.users.entities[userId]);
  const loading = useSelector(state => state.users.loading);
  const error = useSelector(state => state.users.error);
  
  useEffect(() => {
    dispatch(fetchUserById(userId));
  }, [dispatch, userId]);
  
  if (loading === 'pending') {
    return <div>加载中...</div>;
  }
  
  if (error) {
    return <div>错误: {error}</div>;
  }
  
  if (!user) {
    return <div>用户不存在</div>;
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### 取消异步请求
```javascript
import { useEffect } from 'react';
import { useDispatch } from 'react-redux';

function SearchComponent() {
  const dispatch = useDispatch();
  
  useEffect(() => {
    const promise = dispatch(fetchSearchResults(query));
    
    return () => {
      // 组件卸载时取消请求
      promise.abort();
    };
  }, [query, dispatch]);
  
  return <div>搜索结果</div>;
}
```

## RTK Query

### 基础配置
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

// 定义API服务
export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ 
    baseUrl: '/api',
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    }
  }),
  tagTypes: ['User', 'Post'],
  endpoints: (builder) => ({
    // 查询
    getUsers: builder.query<User[], void>({
      query: () => '/users',
      providesTags: ['User']
    }),
    
    getUserById: builder.query<User, string>({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }]
    }),
    
    // 变更
    createUser: builder.mutation<User, Partial<User>>({
      query: (user) => ({
        url: '/users',
        method: 'POST',
        body: user
      }),
      invalidatesTags: ['User']
    }),
    
    updateUser: builder.mutation<User, { id: string; data: Partial<User> }>({
      query: ({ id, data }) => ({
        url: `/users/${id}`,
        method: 'PUT',
        body: data
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }]
    }),
    
    deleteUser: builder.mutation<void, string>({
      query: (id) => ({
        url: `/users/${id}`,
        method: 'DELETE'
      }),
      invalidatesTags: (result, error, id) => [{ type: 'User', id }]
    })
  })
});

export const {
  useGetUsersQuery,
  useGetUserByIdQuery,
  useCreateUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation
} = api;
```

### 配置Store
```javascript
import { configureStore } from '@reduxjs/toolkit';
import { api } from './services/api';

const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware)
});

export default store;
```

### 使用查询
```jsx
import { useGetUsersQuery, useGetUserByIdQuery } from './services/api';

function UserList() {
  const { data: users, isLoading, isError, error } = useGetUsersQuery();
  
  if (isLoading) return <div>加载中...</div>;
  if (isError) return <div>错误: {error.message}</div>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

function UserDetail({ userId }) {
  const { 
    data: user, 
    isLoading, 
    isFetching,
    refetch 
  } = useGetUserByIdQuery(userId, {
    // 轮询
    pollingInterval: 3000,
    // 跳过查询
    skip: !userId,
    // 重新获取策略
    refetchOnMountOrArgChange: true
  });
  
  return (
    <div>
      {isLoading && <div>加载中...</div>}
      {isFetching && <div>更新中...</div>}
      {user && (
        <>
          <h1>{user.name}</h1>
          <button onClick={refetch}>刷新</button>
        </>
      )}
    </div>
  );
}
```

### 使用变更
```jsx
import { useCreateUserMutation, useUpdateUserMutation } from './services/api';

function CreateUserForm() {
  const [createUser, { isLoading, isSuccess, isError, error }] = 
    useCreateUserMutation();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      const result = await createUser({
        name: 'John Doe',
        email: 'john@example.com'
      }).unwrap();
      
      console.log('创建成功:', result);
    } catch (err) {
      console.error('创建失败:', err);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit" disabled={isLoading}>
        {isLoading ? '创建中...' : '创建用户'}
      </button>
      {isSuccess && <div>创建成功！</div>}
      {isError && <div>错误: {error.message}</div>}
    </form>
  );
}
```

### 乐观更新
```javascript
export const api = createApi({
  // ...
  endpoints: (builder) => ({
    updateUser: builder.mutation({
      query: ({ id, data }) => ({
        url: `/users/${id}`,
        method: 'PUT',
        body: data
      }),
      // 乐观更新
      async onQueryStarted({ id, data }, { dispatch, queryFulfilled }) {
        // 更新缓存
        const patchResult = dispatch(
          api.util.updateQueryData('getUserById', id, (draft) => {
            Object.assign(draft, data);
          })
        );
        
        try {
          await queryFulfilled;
        } catch {
          // 失败时回滚
          patchResult.undo();
        }
      }
    })
  })
});
```

### 手动缓存更新
```javascript
function UserComponent() {
  const dispatch = useDispatch();
  
  const handleUpdate = () => {
    // 手动更新缓存
    dispatch(
      api.util.updateQueryData('getUsers', undefined, (draft) => {
        draft.push({ id: '123', name: 'New User' });
      })
    );
    
    // 使缓存失效
    dispatch(api.util.invalidateTags(['User']));
    
    // 预取数据
    dispatch(api.endpoints.getUserById.initiate('123'));
  };
  
  return <button onClick={handleUpdate}>更新</button>;
}
```

## 最佳实践

### 文件组织
```
src/
├── app/
│   ├── store.ts
│   └── hooks.ts
├── features/
│   ├── counter/
│   │   ├── counterSlice.ts
│   │   └── Counter.tsx
│   └── user/
│       ├── userSlice.ts
│       ├── userAPI.ts
│       └── UserProfile.tsx
└── services/
    └── api.ts
```

### 自定义Hooks
```typescript
// app/hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Slice模式
```javascript
// features/todos/todosSlice.ts
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

// 使用Entity Adapter管理规范化数据
const todosAdapter = createEntityAdapter({
  selectId: (todo) => todo.id,
  sortComparer: (a, b) => b.createdAt - a.createdAt
});

const todosSlice = createSlice({
  name: 'todos',
  initialState: todosAdapter.getInitialState({
    loading: 'idle',
    filter: 'all'
  }),
  reducers: {
    todoAdded: todosAdapter.addOne,
    todosReceived: todosAdapter.setAll,
    todoUpdated: todosAdapter.updateOne,
    todoRemoved: todosAdapter.removeOne,
    setFilter: (state, action) => {
      state.filter = action.payload;
    }
  }
});

// 导出selectors
export const {
  selectAll: selectAllTodos,
  selectById: selectTodoById,
  selectIds: selectTodoIds
} = todosAdapter.getSelectors((state) => state.todos);

export const { todoAdded, todosReceived, todoUpdated, todoRemoved, setFilter } = 
  todosSlice.actions;

export default todosSlice.reducer;
```

### 错误处理
```javascript
const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    error: null,
    loading: false
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || '未知错误';
      });
  }
});
```

### 性能优化
```javascript
// 1. 使用shallowEqual避免不必要的重渲染
import { shallowEqual } from 'react-redux';

const todos = useSelector(selectTodos, shallowEqual);

// 2. 使用createSelector记忆化计算
const selectExpensiveComputation = createSelector(
  [selectData],
  (data) => expensiveOperation(data)
);

// 3. 拆分大的slice
// 不好
const bigSlice = createSlice({
  name: 'big',
  initialState: { users: {}, posts: {}, comments: {} },
  // ...
});

// 好
const usersSlice = createSlice({ name: 'users', /* ... */ });
const postsSlice = createSlice({ name: 'posts', /* ... */ });
const commentsSlice = createSlice({ name: 'comments', /* ... */ });
```

## 常见问题

### Q1: Redux Toolkit vs Redux？
RTK是Redux的官方工具集，简化了配置和使用，推荐使用RTK。

### Q2: 何时使用RTK Query？
当需要频繁进行API调用和缓存管理时，RTK Query是很好的选择。

### Q3: 如何处理复杂的异步逻辑？
使用createAsyncThunk或自定义middleware。

---

**@author erik.zhou**
