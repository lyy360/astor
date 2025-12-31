---
title: 'React Router 完全指南'
description: '深入学习 React Router v6，掌握路由配置、嵌套路由、动态路由、路由守卫等核心概念'
pubDate: 'Dec 31 2024'
heroImage: '../../assets/blog-placeholder-1.jpg'
---

## 什么是 React Router？

React Router 是 React 生态系统中最流行的路由解决方案，它允许我们在单页应用（SPA）中实现多页面导航体验，无需页面刷新。

## 安装

```bash
npm install react-router-dom
# 或
pnpm add react-router-dom
```

## 基础配置

### 1. 创建路由配置

在 React Router v6 中，推荐使用 `createBrowserRouter` 来创建路由：

```tsx
// src/router/index.tsx
import { createBrowserRouter } from 'react-router-dom';
import App from '../App';
import Home from '../pages/Home';
import About from '../pages/About';
import NotFound from '../pages/NotFound';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      {
        index: true,
        element: <Home />,
      },
      {
        path: 'about',
        element: <About />,
      },
      {
        path: '*',
        element: <NotFound />,
      },
    ],
  },
]);
```

### 2. 在入口文件中使用

```tsx
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { RouterProvider } from 'react-router-dom';
import { router } from './router';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

## 嵌套路由

嵌套路由是 React Router 的核心特性之一，它允许我们创建层级化的页面结构。

```tsx
// 路由配置
{
  path: 'dashboard',
  element: <DashboardLayout />,
  children: [
    {
      index: true,
      element: <DashboardHome />,
    },
    {
      path: 'settings',
      element: <Settings />,
    },
    {
      path: 'profile',
      element: <Profile />,
    },
  ],
}

// DashboardLayout.tsx
import { Outlet, NavLink } from 'react-router-dom';

export default function DashboardLayout() {
  return (
    <div className="dashboard">
      <nav>
        <NavLink to="/dashboard">首页</NavLink>
        <NavLink to="/dashboard/settings">设置</NavLink>
        <NavLink to="/dashboard/profile">个人信息</NavLink>
      </nav>
      <main>
        {/* 子路由会在这里渲染 */}
        <Outlet />
      </main>
    </div>
  );
}
```

## 动态路由

动态路由允许我们根据 URL 参数渲染不同的内容。

```tsx
// 路由配置
{
  path: 'users/:userId',
  element: <UserDetail />,
}

// UserDetail.tsx
import { useParams } from 'react-router-dom';

export default function UserDetail() {
  const { userId } = useParams<{ userId: string }>();

  return (
    <div>
      <h1>用户详情</h1>
      <p>用户ID: {userId}</p>
    </div>
  );
}
```

## 编程式导航

使用 `useNavigate` Hook 进行编程式导航：

```tsx
import { useNavigate } from 'react-router-dom';

export default function LoginForm() {
  const navigate = useNavigate();

  const handleLogin = async () => {
    // 登录逻辑...

    // 登录成功后跳转
    navigate('/dashboard');

    // 带参数跳转
    navigate('/dashboard', { state: { from: 'login' } });

    // 替换当前历史记录
    navigate('/dashboard', { replace: true });

    // 后退
    navigate(-1);
  };

  return <button onClick={handleLogin}>登录</button>;
}
```

## 路由守卫

实现受保护的路由，只有登录用户才能访问：

```tsx
// components/ProtectedRoute.tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

interface ProtectedRouteProps {
  children: React.ReactNode;
}

export default function ProtectedRoute({ children }: ProtectedRouteProps) {
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    // 保存当前位置，登录后可以重定向回来
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
}

// 在路由配置中使用
{
  path: 'dashboard',
  element: (
    <ProtectedRoute>
      <DashboardLayout />
    </ProtectedRoute>
  ),
}
```

## 路由加载数据 (Loader)

React Router v6.4+ 支持在路由级别加载数据：

```tsx
// 路由配置
{
  path: 'users/:userId',
  element: <UserDetail />,
  loader: async ({ params }) => {
    const response = await fetch(`/api/users/${params.userId}`);
    if (!response.ok) {
      throw new Response('User not found', { status: 404 });
    }
    return response.json();
  },
  errorElement: <UserError />,
}

// UserDetail.tsx
import { useLoaderData } from 'react-router-dom';

interface User {
  id: string;
  name: string;
  email: string;
}

export default function UserDetail() {
  const user = useLoaderData() as User;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

## 常用 Hooks 总结

| Hook | 用途 |
|------|------|
| `useNavigate` | 编程式导航 |
| `useParams` | 获取 URL 参数 |
| `useSearchParams` | 获取/设置查询字符串 |
| `useLocation` | 获取当前位置信息 |
| `useLoaderData` | 获取 loader 加载的数据 |
| `useRouteError` | 获取路由错误信息 |

## 最佳实践

1. **使用懒加载**：对于大型应用，使用 `React.lazy` 和 `Suspense` 实现路由懒加载
2. **集中管理路由**：将所有路由配置放在一个文件中便于管理
3. **使用 TypeScript**：为路由参数添加类型定义
4. **错误边界**：为每个路由配置 `errorElement` 处理错误情况

```tsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));

// 路由配置
{
  path: 'dashboard',
  element: (
    <Suspense fallback={<div>加载中...</div>}>
      <Dashboard />
    </Suspense>
  ),
}
```

## 总结

React Router v6 提供了强大而灵活的路由解决方案，通过嵌套路由、动态路由、路由守卫和数据加载等特性，我们可以轻松构建复杂的单页应用。掌握这些核心概念，将为你的 React 开发之路打下坚实的基础。

