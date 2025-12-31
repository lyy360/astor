---
title: 'Redux Toolkit 实战教程'
description: '从零开始学习 Redux Toolkit，包括 Store 配置、Slice 创建、异步操作和 TypeScript 集成'
pubDate: 'Dec 30 2024'
heroImage: '../../assets/blog-placeholder-2.jpg'
---

## 什么是 Redux Toolkit？

Redux Toolkit (RTK) 是 Redux 官方推荐的编写 Redux 逻辑的方式。它简化了 Redux 的配置和使用，内置了 Immer、Redux Thunk 等常用工具，大大减少了样板代码。

## 为什么选择 Redux Toolkit？

- 📦 **简化配置**：一行代码完成 Store 配置
- ✂️ **减少样板代码**：告别繁琐的 action types 和 action creators
- 🔒 **默认不可变更新**：内置 Immer，可以"直接修改"状态
- 🚀 **内置异步处理**：createAsyncThunk 简化异步操作
- 📝 **完美的 TypeScript 支持**

## 安装

```bash
npm install @reduxjs/toolkit react-redux
# 或
pnpm add @reduxjs/toolkit react-redux
```

## 核心概念

### 1. 创建 Slice

Slice 是 Redux Toolkit 的核心概念，它将 reducer 和 action 合二为一：

```tsx
// src/store/features/counterSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
  status: 'idle' | 'loading' | 'failed';
}

const initialState: CounterState = {
  value: 0,
  status: 'idle',
};

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    // 增加
    increment: (state) => {
      // Redux Toolkit 允许我们"直接修改"状态
      // 实际上使用 Immer 来保证不可变性
      state.value += 1;
    },
    // 减少
    decrement: (state) => {
      state.value -= 1;
    },
    // 增加指定数量
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
    // 重置
    reset: (state) => {
      state.value = 0;
    },
  },
});

// 导出 actions
export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions;

// 导出 reducer
export default counterSlice.reducer;
```

### 2. 配置 Store

```tsx
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counterSlice';
import userReducer from './features/userSlice';
import todosReducer from './features/todosSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
    todos: todosReducer,
  },
  // 中间件配置（可选）
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: false,
    }),
  // 开启 Redux DevTools（生产环境建议关闭）
  devTools: process.env.NODE_ENV !== 'production',
});

// 导出类型
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### 3. 提供 Store

```tsx
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './store';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

## 在组件中使用

### 创建类型化的 Hooks

```tsx
// src/store/hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

// 使用这些 hooks 代替普通的 useDispatch 和 useSelector
export const useAppDispatch: () => AppDispatch = useDispatch;
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### 在组件中使用

```tsx
// src/components/Counter.tsx
import { useAppDispatch, useAppSelector } from '../store/hooks';
import { increment, decrement, incrementByAmount, reset } from '../store/features/counterSlice';

export default function Counter() {
  const count = useAppSelector((state) => state.counter.value);
  const dispatch = useAppDispatch();

  return (
    <div className="counter">
      <h2>计数器: {count}</h2>
      <div className="buttons">
        <button onClick={() => dispatch(decrement())}>-1</button>
        <button onClick={() => dispatch(increment())}>+1</button>
        <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
        <button onClick={() => dispatch(reset())}>重置</button>
      </div>
    </div>
  );
}
```

## 异步操作：createAsyncThunk

处理异步操作是前端开发的常见需求，Redux Toolkit 提供了 `createAsyncThunk` 来简化异步逻辑：

```tsx
// src/store/features/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserState {
  currentUser: User | null;
  users: User[];
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  currentUser: null,
  users: [],
  loading: false,
  error: null,
};

// 创建异步 thunk
export const fetchUsers = createAsyncThunk(
  'user/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      if (!response.ok) {
        throw new Error('请求失败');
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue((error as Error).message);
    }
  }
);

export const fetchUserById = createAsyncThunk(
  'user/fetchUserById',
  async (userId: number, { rejectWithValue }) => {
    try {
      const response = await fetch(
        `https://jsonplaceholder.typicode.com/users/${userId}`
      );
      if (!response.ok) {
        throw new Error('用户不存在');
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue((error as Error).message);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    logout: (state) => {
      state.currentUser = null;
    },
  },
  // 处理异步 actions
  extraReducers: (builder) => {
    builder
      // fetchUsers
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action: PayloadAction<User[]>) => {
        state.loading = false;
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      })
      // fetchUserById
      .addCase(fetchUserById.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUserById.fulfilled, (state, action: PayloadAction<User>) => {
        state.loading = false;
        state.currentUser = action.payload;
      })
      .addCase(fetchUserById.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });
  },
});

export const { clearError, logout } = userSlice.actions;
export default userSlice.reducer;
```

### 在组件中使用异步 Thunk

```tsx
// src/components/UserList.tsx
import { useEffect } from 'react';
import { useAppDispatch, useAppSelector } from '../store/hooks';
import { fetchUsers, clearError } from '../store/features/userSlice';

export default function UserList() {
  const dispatch = useAppDispatch();
  const { users, loading, error } = useAppSelector((state) => state.user);

  useEffect(() => {
    dispatch(fetchUsers());
  }, [dispatch]);

  if (loading) {
    return <div className="loading">加载中...</div>;
  }

  if (error) {
    return (
      <div className="error">
        <p>错误: {error}</p>
        <button onClick={() => dispatch(clearError())}>清除错误</button>
        <button onClick={() => dispatch(fetchUsers())}>重试</button>
      </div>
    );
  }

  return (
    <ul className="user-list">
      {users.map((user) => (
        <li key={user.id}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </li>
      ))}
    </ul>
  );
}
```

## RTK Query：更强大的数据获取方案

RTK Query 是 Redux Toolkit 内置的数据获取和缓存解决方案：

```tsx
// src/store/api/userApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

interface User {
  id: number;
  name: string;
  email: string;
}

export const userApi = createApi({
  reducerPath: 'userApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://jsonplaceholder.typicode.com' }),
  tagTypes: ['User'],
  endpoints: (builder) => ({
    // 查询所有用户
    getUsers: builder.query<User[], void>({
      query: () => '/users',
      providesTags: ['User'],
    }),
    // 查询单个用户
    getUserById: builder.query<User, number>({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }],
    }),
    // 添加用户
    addUser: builder.mutation<User, Partial<User>>({
      query: (newUser) => ({
        url: '/users',
        method: 'POST',
        body: newUser,
      }),
      invalidatesTags: ['User'],
    }),
    // 更新用户
    updateUser: builder.mutation<User, { id: number; data: Partial<User> }>({
      query: ({ id, data }) => ({
        url: `/users/${id}`,
        method: 'PUT',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }],
    }),
  }),
});

// 导出自动生成的 hooks
export const {
  useGetUsersQuery,
  useGetUserByIdQuery,
  useAddUserMutation,
  useUpdateUserMutation,
} = userApi;
```

### 在 Store 中配置 RTK Query

```tsx
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { userApi } from './api/userApi';
import counterReducer from './features/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    [userApi.reducerPath]: userApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(userApi.middleware),
});
```

### 使用 RTK Query Hooks

```tsx
// src/components/UserListWithRTKQuery.tsx
import { useGetUsersQuery, useAddUserMutation } from '../store/api/userApi';

export default function UserListWithRTKQuery() {
  const { data: users, isLoading, isError, refetch } = useGetUsersQuery();
  const [addUser, { isLoading: isAdding }] = useAddUserMutation();

  const handleAddUser = async () => {
    try {
      await addUser({
        name: '新用户',
        email: 'new@example.com',
      }).unwrap();
    } catch (error) {
      console.error('添加用户失败:', error);
    }
  };

  if (isLoading) return <div>加载中...</div>;
  if (isError) return <div>加载失败</div>;

  return (
    <div>
      <button onClick={handleAddUser} disabled={isAdding}>
        {isAdding ? '添加中...' : '添加用户'}
      </button>
      <button onClick={refetch}>刷新</button>
      <ul>
        {users?.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 最佳实践

1. **使用 TypeScript**：Redux Toolkit 对 TypeScript 有出色的支持
2. **按功能组织代码**：使用 features 目录组织不同功能的 slice
3. **使用 RTK Query**：对于 API 请求，优先使用 RTK Query
4. **避免过度使用全局状态**：只将真正需要共享的状态放入 Redux
5. **使用 Redux DevTools**：开发时善用 DevTools 调试

## 总结

Redux Toolkit 极大地简化了 Redux 的使用体验，通过 `createSlice`、`createAsyncThunk` 和 RTK Query，我们可以用更少的代码实现更强大的状态管理。掌握这些工具，将使你的 React 应用开发更加高效。

