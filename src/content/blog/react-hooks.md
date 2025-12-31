---
title: 'React Hooks 完全手册'
description: '全面掌握 React Hooks，包括 useState、useEffect、useContext、useReducer、useMemo、useCallback 等核心 Hooks'
pubDate: 'Dec 29 2024'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

## 什么是 React Hooks？

Hooks 是 React 16.8 引入的新特性，它让你可以在不编写 class 的情况下使用 state 和其他 React 特性。Hooks 使函数组件变得更加强大和灵活。

## 为什么使用 Hooks？

- 🎯 **更简洁的代码**：告别 class 组件的繁琐语法
- 🔄 **更好的逻辑复用**：自定义 Hooks 让逻辑复用变得简单
- 📦 **更好的代码组织**：相关逻辑可以放在一起
- 🔍 **更容易理解**：函数式编程思维更直观

## 基础 Hooks

### useState - 状态管理

`useState` 是最基础的 Hook，用于在函数组件中添加状态。

```tsx
import { useState } from 'react';

// 基础用法
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(prev => prev - 1)}>-1</button>
    </div>
  );
}

// 对象状态
interface User {
  name: string;
  age: number;
}

function UserForm() {
  const [user, setUser] = useState<User>({ name: '', age: 0 });

  const updateName = (name: string) => {
    // 使用展开运算符保持其他属性不变
    setUser(prev => ({ ...prev, name }));
  };

  const updateAge = (age: number) => {
    setUser(prev => ({ ...prev, age }));
  };

  return (
    <form>
      <input
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
        placeholder="姓名"
      />
      <input
        type="number"
        value={user.age}
        onChange={(e) => updateAge(Number(e.target.value))}
        placeholder="年龄"
      />
    </form>
  );
}

// 惰性初始化
function ExpensiveComponent() {
  // 只在首次渲染时执行
  const [data, setData] = useState(() => {
    const initialData = someExpensiveComputation();
    return initialData;
  });

  return <div>{data}</div>;
}
```

### useEffect - 副作用处理

`useEffect` 用于处理副作用，如数据获取、订阅、手动 DOM 操作等。

```tsx
import { useState, useEffect } from 'react';

// 基础用法 - 每次渲染后执行
function Logger({ value }: { value: string }) {
  useEffect(() => {
    console.log('Value changed:', value);
  });

  return <div>{value}</div>;
}

// 带依赖数组 - 仅在依赖变化时执行
function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]); // 只有 userId 变化时才重新获取

  if (loading) return <div>加载中...</div>;
  return <div>{user?.name}</div>;
}

// 空依赖数组 - 仅在挂载时执行一次
function OnMountComponent() {
  useEffect(() => {
    console.log('组件已挂载');
  }, []);

  return <div>Hello</div>;
}

// 清理函数 - 处理订阅/定时器等
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // 清理函数：组件卸载或依赖变化前执行
    return () => {
      clearInterval(intervalId);
    };
  }, []);

  return <div>运行时间: {seconds} 秒</div>;
}

// 事件监听
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return <div>窗口大小: {size.width} x {size.height}</div>;
}
```

### useContext - 跨组件状态共享

`useContext` 让你可以订阅 React Context，实现跨组件的状态共享。

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. 定义 Context 类型
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// 2. 创建 Context
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// 3. 创建 Provider 组件
function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => (prev === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 4. 创建自定义 Hook（推荐）
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// 5. 在组件中使用
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button onClick={toggleTheme}>
      当前主题: {theme === 'light' ? '☀️ 亮色' : '🌙 暗色'}
    </button>
  );
}

function ThemedCard() {
  const { theme } = useTheme();

  return (
    <div className={`card ${theme}`}>
      <p>这是一个主题化的卡片</p>
    </div>
  );
}

// 6. 在应用根部使用 Provider
function App() {
  return (
    <ThemeProvider>
      <ThemeToggle />
      <ThemedCard />
    </ThemeProvider>
  );
}
```

## 进阶 Hooks

### useReducer - 复杂状态管理

当状态逻辑复杂时，`useReducer` 比 `useState` 更适合：

```tsx
import { useReducer } from 'react';

// 定义状态类型
interface TodoState {
  todos: Array<{ id: number; text: string; completed: boolean }>;
  filter: 'all' | 'active' | 'completed';
}

// 定义 Action 类型
type TodoAction =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: number }
  | { type: 'DELETE_TODO'; payload: number }
  | { type: 'SET_FILTER'; payload: TodoState['filter'] };

// 初始状态
const initialState: TodoState = {
  todos: [],
  filter: 'all',
};

// Reducer 函数
function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: Date.now(), text: action.payload, completed: false },
        ],
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        ),
      };
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload),
      };
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload,
      };
    default:
      return state;
  }
}

// 使用 useReducer
function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <div>
      <input
        onKeyDown={(e) => {
          if (e.key === 'Enter' && e.currentTarget.value) {
            dispatch({ type: 'ADD_TODO', payload: e.currentTarget.value });
            e.currentTarget.value = '';
          }
        }}
        placeholder="添加待办事项"
      />

      <div>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'all' })}>
          全部
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'active' })}>
          进行中
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'completed' })}>
          已完成
        </button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
              删除
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### useMemo - 计算缓存

`useMemo` 用于缓存计算结果，避免不必要的重复计算：

```tsx
import { useMemo, useState } from 'react';

function ExpensiveList({ items, filter }: { items: string[]; filter: string }) {
  // 只有 items 或 filter 变化时才重新计算
  const filteredItems = useMemo(() => {
    console.log('正在过滤...');
    return items.filter(item =>
      item.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  // 复杂计算示例
  const statistics = useMemo(() => {
    console.log('正在计算统计数据...');
    return {
      total: items.length,
      filtered: filteredItems.length,
      percentage: ((filteredItems.length / items.length) * 100).toFixed(1),
    };
  }, [items, filteredItems]);

  return (
    <div>
      <p>显示 {statistics.filtered} / {statistics.total} ({statistics.percentage}%)</p>
      <ul>
        {filteredItems.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

### useCallback - 函数缓存

`useCallback` 用于缓存函数引用，避免子组件不必要的重渲染：

```tsx
import { useCallback, useState, memo } from 'react';

// 使用 memo 包裹的子组件
const ExpensiveChild = memo(function ExpensiveChild({
  onClick
}: {
  onClick: () => void
}) {
  console.log('ExpensiveChild 渲染');
  return <button onClick={onClick}>点击我</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // 不使用 useCallback：每次渲染都创建新函数
  // const handleClick = () => {
  //   console.log('clicked');
  // };

  // 使用 useCallback：函数引用保持稳定
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []); // 依赖数组为空，函数永远不变

  // 如果需要访问最新的 count
  const handleIncrement = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // 使用函数式更新，不需要依赖 count

  return (
    <div>
      <p>Count: {count}</p>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="输入名字（不会导致子组件重渲染）"
      />
      <ExpensiveChild onClick={handleClick} />
      <button onClick={handleIncrement}>增加</button>
    </div>
  );
}
```

### useRef - 引用管理

`useRef` 用于保存可变值和访问 DOM 元素：

```tsx
import { useRef, useEffect, useState } from 'react';

// 1. 访问 DOM 元素
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>聚焦输入框</button>
    </div>
  );
}

// 2. 保存可变值（不触发重渲染）
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef<number | null>(null);

  const startTimer = () => {
    if (intervalRef.current) return; // 防止重复启动
    intervalRef.current = window.setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  useEffect(() => {
    return () => stopTimer(); // 清理
  }, []);

  return (
    <div>
      <p>时间: {seconds} 秒</p>
      <button onClick={startTimer}>开始</button>
      <button onClick={stopTimer}>停止</button>
    </div>
  );
}

// 3. 保存前一个值
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>当前值: {count}, 前一个值: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

## 自定义 Hooks

自定义 Hooks 是复用状态逻辑的强大方式：

```tsx
// useLocalStorage - 持久化状态
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// useDebounce - 防抖
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// useFetch - 数据获取
interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      setState(prev => ({ ...prev, loading: true, error: null }));
      try {
        const response = await fetch(url, { signal: abortController.signal });
        if (!response.ok) throw new Error('请求失败');
        const data = await response.json();
        setState({ data, loading: false, error: null });
      } catch (error) {
        if ((error as Error).name !== 'AbortError') {
          setState({ data: null, loading: false, error: error as Error });
        }
      }
    };

    fetchData();

    return () => abortController.abort();
  }, [url]);

  return state;
}

// 使用示例
function SearchUsers() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);

  const { data, loading, error } = useFetch<User[]>(
    `https://api.example.com/users?q=${debouncedQuery}`
  );

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="搜索用户..."
      />
      {loading && <p>搜索中...</p>}
      {error && <p>错误: {error.message}</p>}
      {data && (
        <ul>
          {data.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## Hooks 使用规则

1. **只在顶层调用 Hooks**：不要在循环、条件或嵌套函数中调用
2. **只在 React 函数中调用 Hooks**：函数组件或自定义 Hooks 中

```tsx
// ❌ 错误示例
function BadComponent({ condition }) {
  if (condition) {
    const [state, setState] = useState(0); // 错误！
  }
}

// ✅ 正确示例
function GoodComponent({ condition }) {
  const [state, setState] = useState(0);

  if (condition) {
    // 在这里使用 state
  }
}
```

## Hooks 速查表

| Hook | 用途 |
|------|------|
| `useState` | 状态管理 |
| `useEffect` | 副作用处理 |
| `useContext` | 使用 Context |
| `useReducer` | 复杂状态管理 |
| `useMemo` | 缓存计算结果 |
| `useCallback` | 缓存函数引用 |
| `useRef` | 引用管理 |
| `useLayoutEffect` | 同步副作用 |
| `useId` | 生成唯一 ID |
| `useTransition` | 非紧急更新 |
| `useDeferredValue` | 延迟更新 |

## 总结

React Hooks 彻底改变了我们编写 React 组件的方式，使代码更加简洁、可复用。掌握这些核心 Hooks，并学会创建自定义 Hooks，将使你的 React 开发技能更上一层楼！

