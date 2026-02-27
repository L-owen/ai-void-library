# React Best Practices

React 组件开发最佳实践参考指南。

## 组件设计原则

### 单一职责原则
每个组件应该只负责一个功能点。
- ✅ Good: `UserAvatar`, `UserProfile`, `UserSettings`
- ❌ Bad: `UserComponent` (混合了头像、资料、设置等多个功能)

### 组件组合优于继承
使用 composition 而非 inheritance：
```tsx
// ✅ Good
function Dialog({ title, children }) {
  return (
    <div className="dialog">
      <div className="dialog-header">{title}</div>
      <div className="dialog-body">{children}</div>
    </div>
  )
}

// ❌ Bad
class Dialog extends Component {
  // 复杂的继承逻辑
}
```

## Hooks 最佳实践

### Hooks 使用规则
1. 只在 React 函数组件中调用 Hooks
2. 只在顶层调用 Hooks，不要在循环、条件或嵌套函数中调用
3. 自定义 Hook 必须以 `use` 开头

### 常见 Hooks 陷阱

#### useState 的函数式更新
```tsx
// ✅ Good
const [count, setCount] = useState(0)
setCount(prev => prev + 1)

// ❌ Bad (当基于前一个状态更新时)
setCount(count + 1)
```

#### useEffect 的依赖数组
```tsx
// ✅ Good - 完整的依赖数组
useEffect(() => {
  const subscription = props.source.subscribe()
  return () => subscription.unsubscribe()
}, [props.source]) // ✅ props.source 在依赖中

// ❌ Bad - 缺少依赖
useEffect(() => {
  const subscription = props.source.subscribe()
  return () => subscription.unsubscribe()
}, []) // ❌ 缺少 props.source
```

#### 避免过度的 useEffect
```tsx
// ❌ Bad - 不必要的 useEffect
const [count, setCount] = useState(0)
useEffect(() => {
  setCount(count + 1)
}, [count])

// ✅ Good - 直接在事件处理中
const handleClick = () => setCount(c => c + 1)
```

### 自定义 Hook 设计
```tsx
// ✅ Good - 清晰的自定义 Hook
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      return initialValue
    }
  })

  const setValue = useCallback((value) => {
    setStoredValue(value)
    window.localStorage.setItem(key, JSON.stringify(value))
  }, [key])

  return [storedValue, setValue]
}
```

## 状态管理

### Props Drilling 避免方案
```tsx
// ❌ Bad - Props drilling
function App() {
  const user = { name: 'John' }
  return <Header user={user} />
}

function Header({ user }) {
  return <Navigation user={user} />
}

function Navigation({ user }) {
  return <UserMenu user={user} />
}

// ✅ Good - 使用 Context
const UserContext = createContext<User | null>(null)

function App() {
  const user = { name: 'John' }
  return (
    <UserContext.Provider value={user}>
      <Header />
    </UserContext.Provider>
  )
}
```

### 状态提升
当多个组件需要共享状态时，将状态提升到它们的最近共同父组件：
```tsx
// ✅ Good
function Parent() {
  const [value, setValue] = useState('')
  return (
    <>
      <ChildA value={value} onChange={setValue} />
      <ChildB value={value} />
    </>
  )
}
```

## 性能优化

### React.memo 适当使用
```tsx
// ✅ Good - 对等 props 的组件使用 memo
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* 复杂渲染 */}</div>
}, (prevProps, nextProps) => {
  // 自定义比较逻辑
  return prevProps.data.id === nextProps.data.id
})
```

### useMemo 和 useCallback 的合理使用
```tsx
// ✅ Good - 缓存昂贵的计算
const sortedList = useMemo(() => {
  return list.sort((a, b) => a.id - b.id)
}, [list])

// ✅ Good - 缓存传递给 memo 子组件的回调
const handleClick = useCallback(() => {
  doSomething(id)
}, [id])

// ❌ Bad - 不必要的 useMemo/useCallback
const value = useMemo(() => 1 + 1, []) // 简单计算不需要缓存
```

### 避免在渲染中创建对象/数组
```tsx
// ❌ Bad - 每次渲染都创建新对象
<div style={{ margin: 10 }} />

// ✅ Good - 将样式定义在组件外部
const containerStyle = { margin: 10 }
<div style={containerStyle} />
```

### 列表渲染的 key
```tsx
// ✅ Good - 使用稳定的、唯一的 ID
{items.map(item => (
  <Item key={item.id} {...item} />
))}

// ❌ Bad - 使用索引作为 key（当列表会重新排序时）
{items.map((item, index) => (
  <Item key={index} {...item} />
))}

// ❌ Bad - 使用随机值作为 key
{items.map(item => (
  <Item key={Math.random()} {...item} />
))}
```

## 组件通信

### Props 类型定义
```tsx
// ✅ Good - 明确的 props 类型
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  onClick: () => void
  children: React.ReactNode
}

function Button({ variant, size = 'md', disabled, onClick, children }: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  )
}
```

### 事件处理
```tsx
// ✅ Good - 正确的事件类型
function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
  setValue(event.target.value)
}

function handleClick(event: React.MouseEvent<HTMLButtonElement>) {
  event.preventDefault()
  // ...
}
```

## 错误边界

### 实现错误边界
```tsx
// ✅ Good
class ErrorBoundary extends React.Component<
  { children: ReactNode },
  { hasError: boolean }
> {
  constructor(props: { children: ReactNode }) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true }
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>
    }
    return this.props.children
  }
}
```

## Refs 使用

### useRef 的正确使用
```tsx
// ✅ Good - 访问 DOM
const inputRef = useRef<HTMLInputElement>(null)
useEffect(() => {
  inputRef.current?.focus()
}, [])

// ✅ Good - 保存可变值（不触发重新渲染）
const timerRef = useRef<NodeJS.Timeout | null>(null)
useEffect(() => {
  timerRef.current = setInterval(() => {}, 1000)
  return () => {
    if (timerRef.current) {
      clearInterval(timerRef.current)
    }
  }
}, [])

// ❌ Bad - 不应该用 ref 代替 state
const countRef = useRef(0)
countRef.current += 1 // 不会触发重新渲染
```

### forwardRef 使用
```tsx
// ✅ Good
const FancyInput = forwardRef<HTMLInputElement, FancyInputProps>(
  (props, ref) => {
    return <input ref={ref} className="fancy" {...props} />
  }
)
```

## 代码组织

### 组件文件结构
```
components/
  UserProfile/
    index.tsx         // 主组件
    types.ts          // 类型定义
    hooks.ts          // 自定义 hooks
    utils.ts          // 工具函数
    __tests__/        // 测试文件
```

### 导入顺序
```tsx
// 1. React 相关
import React, { useState, useEffect } from 'react'

// 2. 第三方库
import { useRouter } from 'next/router'
import clsx from 'clsx'

// 3. 组件
import { Button } from '@/components/ui/Button'
import { UserAvatar } from './UserAvatar'

// 4. 类型
import type { User } from '@/types'

// 5. 工具函数和样式
import { formatDate } from '@/utils/date'
import styles from './UserProfile.module.css'
```
