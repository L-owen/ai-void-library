# Code Quality

React + TypeScript 代码质量参考指南。

## 代码可读性

### 命名规范

```tsx
// ✅ Good - 清晰的命名
const UserList = () => { }           // 组件: PascalCase
const getUserById = (id: number) => { }  // 函数: camelCase
const MAX_RETRY_COUNT = 3            // 常量: UPPER_SNAKE_CASE
interface UserProfile { }            // 接口: PascalCase
type Theme = 'light' | 'dark'        // 类型: PascalCase

// ❌ Bad - 不清晰的命名
const UL = () => { }                 // 缩写
const getData = (id: number) => { }  // 太通用
const temp = {}                      // 无意义的名称
```

### 函数设计

```tsx
// ✅ Good - 单一职责，参数清晰
function createUser({
  name,
  email,
  age,
}: {
  name: string
  email: string
  age: number
}): User {
  return { id: generateId(), name, email, age }
}

// ❌ Bad - 参数过多
function createUser(name: string, email: string, age: number, address: string, phone: string) {
  // ...
}

// ✅ Good - 使用对象参数
function createUser(data: {
  name: string
  email: string
  age: number
  address: string
  phone: string
}) {
  // ...
}
```

### 代码组织

```tsx
// ✅ Good - 清晰的组件结构
function UserProfile({ userId }: { userId: string }) {
  // 1. Hooks
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const navigate = useNavigate()

  // 2. 派生状态
  const isAdmin = user?.role === 'admin'
  const displayName = user ? `${user.firstName} ${user.lastName}` : 'Guest'

  // 3. 副作用
  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false))
  }, [userId])

  // 4. 事件处理器
  const handleEdit = () => navigate(`/users/${userId}/edit`)
  const handleDelete = () => deleteUser(userId)

  // 5. 渲染
  if (loading) return <Spinner />
  if (!user) return <NotFound />

  return (
    <div>
      <h1>{displayName}</h1>
      {/* ... */}
    </div>
  )
}
```

## DRY 原则（Don't Repeat Yourself）

### 提取可复用逻辑

```tsx
// ❌ Bad - 重复的代码
function UserList() {
  const [users, setUsers] = useState<User[]>([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .finally(() => setLoading(false))
  }, [])

  // ...
}

function ProductList() {
  const [products, setProducts] = useState<Product[]>([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetchProducts()
      .then(setProducts)
      .finally(() => setLoading(false))
  }, [])

  // ...
}

// ✅ Good - 提取自定义 hook
function useData<T>(
  fetcher: () => Promise<T[]>
): { data: T[]; loading: boolean } {
  const [data, setData] = useState<T[]>([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetcher()
      .then(setData)
      .finally(() => setLoading(false))
  }, [fetcher])

  return { data, loading }
}

function UserList() {
  const { data: users, loading } = useData(fetchUsers)
  // ...
}

function ProductList() {
  const { data: products, loading } = useData(fetchProducts)
  // ...
}
```

### 组件组合

```tsx
// ❌ Bad - 重复的布局代码
function UserPage() {
  return (
    <div className="page">
      <Header />
      <div className="content">
        <Sidebar />
        <main>{/* user content */}</main>
      </div>
      <Footer />
    </div>
  )
}

function ProductPage() {
  return (
    <div className="page">
      <Header />
      <div className="content">
        <Sidebar />
        <main>{/* product content */}</main>
      </div>
      <Footer />
    </div>
  )
}

// ✅ Good - 提取布局组件
function PageLayout({ children }: { children: ReactNode }) {
  return (
    <div className="page">
      <Header />
      <div className="content">
        <Sidebar />
        <main>{children}</main>
      </div>
      <Footer />
    </div>
  )
}

function UserPage() {
  return (
    <PageLayout>
      {/* user content */}
    </PageLayout>
  )
}
```

## 代码复杂度

### 避免深层嵌套

```tsx
// ❌ Bad - 深层嵌套
function UserList({ users }: { users: User[] }) {
  return (
    <div>
      {users.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          {user.posts && user.posts.length > 0 ? (
            <div>
              {user.posts.map(post => (
                <div key={post.id}>
                  <h4>{post.title}</h4>
                  {post.comments && post.comments.length > 0 ? (
                    <ul>
                      {post.comments.map(comment => (
                        <li key={comment.id}>{comment.text}</li>
                      ))}
                    </ul>
                  ) : null}
                </div>
              ))}
            </div>
          ) : null}
        </div>
      ))}
    </div>
  )
}

// ✅ Good - 拆分子组件
function CommentList({ comments }: { comments: Comment[] }) {
  return (
    <ul>
      {comments.map(comment => (
        <li key={comment.id}>{comment.text}</li>
      ))}
    </ul>
  )
}

function PostList({ posts }: { posts: Post[] }) {
  return posts.map(post => (
    <div key={post.id}>
      <h4>{post.title}</h4>
      {post.comments && post.comments.length > 0 && (
        <CommentList comments={post.comments} />
      )}
    </div>
  ))
}

function UserCard({ user }: { user: User }) {
  return (
    <div>
      <h3>{user.name}</h3>
      {user.posts && user.posts.length > 0 && <PostList posts={user.posts} />}
    </div>
  )
}

function UserList({ users }: { users: User[] }) {
  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  )
}
```

### 提取复杂逻辑

```tsx
// ❌ Bad - 复杂的内联逻辑
function UserForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    age: '',
  })

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target
    let validatedValue = value

    if (name === 'email') {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
      if (!emailRegex.test(value)) {
        // 错误处理
      }
    }

    if (name === 'age') {
      const age = parseInt(value)
      if (isNaN(age) || age < 0 || age > 120) {
        // 错误处理
      }
    }

    setForm(prev => ({ ...prev, [name]: validatedValue }))
  }

  // ...
}

// ✅ Good - 提取验证逻辑
const validators = {
  email: (value: string) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!emailRegex.test(value)) {
      throw new Error('Invalid email format')
    }
    return value
  },

  age: (value: string) => {
    const age = parseInt(value)
    if (isNaN(age) || age < 0 || age > 120) {
      throw new Error('Invalid age')
    }
    return age
  },
}

function UserForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    age: '',
  })

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target
    try {
      const validatedValue = validators[name as keyof typeof validators]?.(value) ?? value
      setForm(prev => ({ ...prev, [name]: validatedValue }))
    } catch (error) {
      // 处理验证错误
    }
  }

  // ...
}
```

## 类型安全

### 避免类型断言

```tsx
// ❌ Bad - 过度使用类型断言
const user = data as User
const email = (data as any).email

// ✅ Good - 类型守卫
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    'email' in value
  )
}

if (isUser(data)) {
  // data 现在是 User 类型
}
```

### 使用联合类型

```tsx
// ❌ Bad - 使用可选属性
interface User {
  id: number
  name: string
  email?: string
  phone?: string
  // 问题: email 和 phone 可能为 undefined
}

// ✅ Good - 使用联合类型
type ContactMethod = { type: 'email'; value: string } | { type: 'phone'; value: string }

interface User {
  id: number
  name: string
  contacts: ContactMethod[]
}
```

## 注释和文档

### 何时需要注释

```tsx
// ❌ Bad - 注释重复代码
// 获取用户
const user = getUser()
// 设置用户
setUser(user)

// ✅ Good - 解释"为什么"而非"是什么"
// 使用延迟加载以提高初始加载性能
const HeavyChart = lazy(() => import('./HeavyChart'))

// ✅ Good - 记录复杂逻辑的意图
// 使用二分查找快速定位用户在已排序列表中的位置
function findUserIndex(sortedUsers: User[], targetUserId: number): number {
  // ...
}

// ✅ Good - 标记 TODO 和 FIXME
// TODO: 添加错误处理和重试逻辑
// FIXME: 这个临时方案在生产环境可能导致内存泄漏
```

### JSDoc 注释

```tsx
/**
 * 计算两个日期之间的天数差
 * @param date1 - 第一个日期
 * @param date2 - 第二个日期
 * @returns 天数差（绝对值）
 * @throws {Error} 如果传入无效日期
 * @example
 * ```ts
 * const days = daysBetween(new Date('2024-01-01'), new Date('2024-01-10'))
 * console.log(days) // 9
 * ```
 */
function daysBetween(date1: Date, date2: Date): number {
  const diff = Math.abs(date1.getTime() - date2.getTime())
  return Math.ceil(diff / (1000 * 60 * 60 * 24))
}
```

## 错误处理

### 优雅的错误处理

```tsx
// ✅ Good - 统一的错误处理
class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string
  ) {
    super(message)
    this.name = 'ApiError'
  }
}

async function fetchUser(id: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`)
    if (!response.ok) {
      throw new ApiError(response.status, 'Failed to fetch user')
    }
    return await response.json()
  } catch (error) {
    if (error instanceof ApiError) {
      // 处理 API 错误
    }
    // 处理其他错误
    throw error
  }
}

// ✅ Good - 错误边界
class ErrorBoundary extends React.Component<
  { children: ReactNode },
  { hasError: boolean; error?: Error }
> {
  state = { hasError: false }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />
    }
    return this.props.children
  }
}
```

## 测试友好

### 可测试的代码

```tsx
// ❌ Bad - 难以测试（依赖全局状态）
function UserList() {
  const [users, setUsers] = useState<User[]>([])

  useEffect(() => {
    // 直接调用 fetch，难以 mock
    fetch('/api/users')
      .then(res => res.json())
      .then(setUsers)
  }, [])

  return <div>{/* ... */}</div>
}

// ✅ Good - 易于测试（依赖注入）
function UserList({
  fetcher = fetchUsers
}: {
  fetcher?: () => Promise<User[]>
}) {
  const { data: users, loading } = useData(fetcher)

  if (loading) return <Spinner />
  return <div>{/* ... */}</div>
}

// 测试
it('should render users', async () => {
  const mockUsers = [{ id: 1, name: 'John' }]
  const fetcher = jest.fn().mockResolvedValue(mockUsers)

  render(<UserList fetcher={fetcher} />)

  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument()
  })
})
```

## 代码质量检查清单

### 可读性
- [ ] 使用清晰的命名
- [ ] 函数职责单一
- [ ] 避免深层嵌套
- [ ] 合理的代码组织
- [ ] 必要的注释

### 可维护性
- [ ] 遵循 DRY 原则
- [ ] 合理的抽象层次
- [ ] 模块化设计
- [ ] 一致的代码风格

### 类型安全
- [ ] 避免使用 any
- [ ] 使用类型守卫而非断言
- [ ] 完整的类型定义
- [ ] 启用严格模式

### 错误处理
- [ ] 统一的错误处理
- [ ] 错误边界
- [ ] 优雅的错误恢复
- [ ] 有意义的错误消息

### 测试友好
- [ ] 依赖注入
- [ ] 纯函数优先
- [ ] 可测试的组件
- [ ] Mock 友好
