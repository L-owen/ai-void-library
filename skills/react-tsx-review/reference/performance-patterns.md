# Performance Patterns

React + TypeScript 应用性能优化参考指南。

## 组件渲染优化

### React.memo 使用

```tsx
// ✅ Good - 对纯展示组件使用 memo
const UserCard = memo(({ user }: { user: User }) => {
  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  )
})

// ✅ Good - 自定义比较函数
const ExpensiveList = memo(
  ({ items }: { items: Item[] }) => {
    return (
      <div>
        {items.map(item => (
          <Item key={item.id} {...item} />
        ))}
      </div>
    )
  },
  (prevProps, nextProps) => {
    // 只比较第一个元素的变化
    return prevProps.items[0]?.id === nextProps.items[0]?.id
  }
)
```

### 避免不必要的渲染

```tsx
// ❌ Bad - 每次父组件更新都会创建新的 onClick
function Parent() {
  const [count, setCount] = useState(0)
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <Child onClick={() => console.log('clicked')} />
    </>
  )
}

// ✅ Good - 使用 useCallback
function Parent() {
  const [count, setCount] = useState(0)

  const handleClick = useCallback(() => {
    console.log('clicked')
  }, [])

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <Child onClick={handleClick} />
    </>
  )
}
```

## 计算优化

### useMemo 使用

```tsx
// ✅ Good - 缓存昂贵的计算
function SortedList({ items }: { items: Item[] }) {
  const sortedItems = useMemo(() => {
    console.log('Sorting items...')
    return items.slice().sort((a, b) => a.id - b.id)
  }, [items])

  return (
    <ul>
      {sortedItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
}

// ❌ Bad - 简单计算不需要 useMemo
const doubled = useMemo(() => count * 2, [count])
// ✅ Good - 直接计算
const doubled = count * 2

// ❌ Bad - 每次都创建新对象
const style = useMemo(() => ({ margin: 10 }), [])
// ✅ Good - 移到组件外部
const containerStyle = { margin: 10 }
```

### 大列表优化

```tsx
// ✅ Good - 使用虚拟滚动
import { FixedSizeList } from 'react-window'

function VirtualizedList({ items }: { items: Item[] }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          <Item data={items[index]} />
        </div>
      )}
    </FixedSizeList>
  )
}

// ✅ Good - 使用 react-window 的 VariableSizeList
// 对于不同高度的项
import { VariableSizeList } from 'react-window'

function VariableList({ items }: { items: Item[] }) {
  const getItemSize = (index: number) => {
    return items[index].expanded ? 100 : 50
  }

  return (
    <VariableSizeList
      height={600}
      itemCount={items.length}
      itemSize={getItemSize}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          <Item data={items[index]} />
        </div>
      )}
    </VariableSizeList>
  )
}
```

## Code Splitting

### 路由级别的代码分割

```tsx
// ✅ Good - 使用 React.lazy 和 Suspense
import { lazy, Suspense } from 'react'

const Dashboard = lazy(() => import('./pages/Dashboard'))
const Settings = lazy(() => import('./pages/Settings'))

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  )
}
```

### 组件级别的代码分割

```tsx
// ✅ Good - 按需加载重型组件
const HeavyChart = lazy(() => import('./HeavyChart'))

function Dashboard() {
  const [showChart, setShowChart] = useState(false)

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<Loading />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  )
}
```

## 状态管理优化

### Context 优化

```tsx
// ❌ Bad - 整个 state 在一个 context 中，会导致不必要的重渲染
const AppContext = createContext<{
  state: AppState
  dispatch: Dispatch<Action>
} | null>(null)

function AppProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, initialState)

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  )
}

// ✅ Good - 分离 context
const StateContext = createContext<AppState | null>(null)
const DispatchContext = createContext<Dispatch<Action> | null>(null)

function AppProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, initialState)

  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  )
}

// 组件可以选择性地订阅
function Component() {
  const dispatch = useContext(DispatchContext)
  // 只有 dispatch 改变时才重渲染
}
```

### 使用选择器模式

```tsx
// ✅ Good - 只订阅需要的状态片段
function useUserSelector<T>(selector: (state: AppState) => T): T {
  const state = useContext(StateContext)!
  return useMemo(() => selector(state), [state, selector])
}

function UserName() {
  const name = useUserSelector(state => state.user.name)
  return <div>{name}</div>
}
```

## 图片优化

### 懒加载图片

```tsx
// ✅ Good - 使用 Intersection Observer
function LazyImage({ src, alt }: { src: string; alt: string }) {
  const [imageSrc, setImageSrc] = useState<string>()
  const imgRef = useRef<HTMLImageElement>(null)

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImageSrc(src)
          observer.disconnect()
        }
      },
      { rootMargin: '100px' }
    )

    if (imgRef.current) {
      observer.observe(imgRef.current)
    }

    return () => observer.disconnect()
  }, [src])

  return (
    <img ref={imgRef} src={imageSrc} alt={alt} style={{ opacity: imageSrc ? 1 : 0 }} />
  )
}
```

### 响应式图片

```tsx
// ✅ Good - 使用 srcset
function ResponsiveImage({ image }: { image: Image }) {
  return (
    <img
      srcSet={`
        ${image.url_small} 320w,
        ${image.url_medium} 640w,
        ${image.url_large} 1024w
      `}
      sizes="(max-width: 640px) 320px, (max-width: 1024px) 640px, 1024px"
      src={image.url_medium}
      alt={image.alt}
      loading="lazy"
    />
  )
}
```

## 避免内存泄漏

### 清理副作用

```tsx
// ✅ Good - 正确的清理
function DataFetcher() {
  const [data, setData] = useState<Data | null>(null)

  useEffect(() => {
    let cancelled = false

    const fetchData = async () => {
      const result = await api.fetchData()
      if (!cancelled) {
        setData(result)
      }
    }

    fetchData()

    return () => {
      cancelled = true
    }
  }, [])

  return <div>{/* ... */}</div>
}

// ✅ Good - AbortController
function DataFetcher() {
  useEffect(() => {
    const controller = new AbortController()

    fetch('/api/data', { signal: controller.signal })
      .then(res => res.json())
      .then(setData)

    return () => {
      controller.abort()
    }
  }, [])

  return <div>{/* ... */}</div>
}
```

## 性能监控

### 使用 React DevTools Profiler

```tsx
// ✅ Good - 使用 Profiler 组件
import { Profiler, ProfilerOnRenderCallback } from 'react'

const onRenderCallback: ProfilerOnRenderCallback = (
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime,
  interactions
) => {
  console.log('Profiler:', {
    id,
    phase,
    actualDuration,
    baseDuration,
  })
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Navigation />
      <MainContent />
    </Profiler>
  )
}
```

### 自定义性能监控

```tsx
// ✅ Good - 使用 Performance API
function usePerformanceMonitor(componentName: string) {
  useEffect(() => {
    performance.mark(`${componentName}-mount-start`)

    return () => {
      performance.mark(`${componentName}-mount-end`)
      performance.measure(
        `${componentName}-mount`,
        `${componentName}-mount-start`,
        `${componentName}-mount-end`
      )

      const measure = performance.getEntriesByName(`${componentName}-mount`)[0]
      console.log(`${componentName} mounted in ${measure.duration}ms`)
    }
  }, [componentName])
}
```

## Bundle 优化

### Tree shaking

```tsx
// ❌ Bad - 导入整个库
import _ from 'lodash'

// ✅ Good - 只导入需要的函数
import debounce from 'lodash/debounce'
// 或者使用 ES Module 版本
import { debounce } from 'lodash-es'

// ✅ Good - 使用更小的替代库
import debounce from 'debounce'
```

### 动态导入

```tsx
// ✅ Good - 动态导入大型依赖
function Chart() {
  const [chartLib, setChartLib] = useState<typeof ChartJS | null>(null)

  useEffect(() => {
    import('chart.js').then(mod => {
      setChartLib(mod.default)
    })
  }, [])

  if (!chartLib) return <Loading />

  return <canvas ref={canvasRef} />
}
```

## 优化清单

### 渲染性能
- [ ] 使用 React.memo 避免不必要的重渲染
- [ ] 使用 useCallback 缓存事件处理器
- [ ] 使用 useMemo 缓存昂贵的计算
- [ ] 避免在渲染中创建新对象/数组/函数
- [ ] 为列表项提供稳定的 key
- [ ] 对长列表使用虚拟滚动

### 加载性能
- [ ] 路由级别的代码分割
- [ ] 按需加载重型组件
- [ ] 图片懒加载
- [ ] 使用响应式图片
- [ ] Tree shaking 未使用的代码

### 运行时性能
- [ ] 正确清理副作用
- [ ] 避免内存泄漏
- [ ] 使用 requestAnimationFrame 优化动画
- [ ] 防抖/节流用户输入

### 监控
- [ ] 使用 React DevTools Profiler
- [ ] 监控组件渲染时间
- [ ] 监控 bundle 大小
- [ ] 设置性能预算
