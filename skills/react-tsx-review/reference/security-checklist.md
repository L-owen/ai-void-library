# Security Checklist

React + TypeScript 应用安全检查参考指南。

## XSS 防护

### dangerouslySetInnerHTML 谨慎使用

```tsx
// ❌ Bad - 直接渲染用户输入
function Comment({ content }: { content: string }) {
  return <div dangerouslySetInnerHTML={{ __html: content }} />
}

// ✅ Good - 使用 DOMPurify 清理
import DOMPurify from 'dompurify'

function Comment({ content }: { content: string }) {
  const clean = DOMPurify.sanitize(content)
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}

// ✅ Better - 避免使用 HTML，使用纯文本
function Comment({ content }: { content: string }) {
  return <div>{content}</div>
}
```

### URL 注入防护

```tsx
// ❌ Bad - 未验证的 URL
function Link({ url }: { url: string }) {
  return <a href={url}>Click</a>
}

// ✅ Good - 验证 URL
function isValidUrl(url: string): boolean {
  try {
    const parsed = new URL(url)
    return ['http:', 'https:'].includes(parsed.protocol)
  } catch {
    return false
  }
}

function Link({ url }: { url: string }) {
  if (!isValidUrl(url)) {
    return <a href="#">Invalid URL</a>
  }
  return <a href={url}>Click</a>
}
```

## CSRF 防护

### CSRF Token

```tsx
// ✅ Good - 在请求中包含 CSRF token
const csrfToken = document.querySelector('meta[name="csrf-token"]')?.getAttribute('content')

async function postData(url: string, data: unknown) {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken || '',
    },
    body: JSON.stringify(data),
  })
  return response.json()
}
```

### SameSite Cookie

```typescript
// ✅ Good - 服务器端设置 SameSite cookie
// Set-Cookie: sessionId=xxx; SameSite=Strict; Secure; HttpOnly
```

## 认证和授权

### 路由保护

```tsx
// ✅ Good - 路由级别保护
interface ProtectedRouteProps {
  children: ReactNode
  requiredPermission?: string
}

function ProtectedRoute({ children, requiredPermission }: ProtectedRouteProps) {
  const { user, isAuthenticated, hasPermission } = useAuth()

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />
  }

  if (requiredPermission && !hasPermission(requiredPermission)) {
    return <Navigate to="/403" replace />
  }

  return <>{children}</>
}

// 使用
<Route
  path="/admin"
  element={
    <ProtectedRoute requiredPermission="admin:access">
      <AdminPanel />
    </ProtectedRoute>
  }
/>
```

### 组件级别授权

```tsx
// ✅ Good - 组件级别权限控制
function PermissionGate({
  permission,
  fallback,
  children,
}: {
  permission: string
  fallback?: ReactNode
  children: ReactNode
}) {
  const { hasPermission } = useAuth()

  if (!hasPermission(permission)) {
    return fallback || null
  }

  return <>{children}</>
}

// 使用
<PermissionGate permission="user:delete" fallback={<span>无权限</span>}>
  <button onClick={handleDelete}>删除用户</button>
</PermissionGate>
```

## 敏感数据处理

### 不在客户端存储敏感信息

```tsx
// ❌ Bad - localStorage 存储敏感信息
localStorage.setItem('token', token)
localStorage.setItem('userPassword', password)

// ✅ Good - 使用 httpOnly cookie
// 服务器端设置:
// Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict

// ✅ Good - 使用内存状态
function AuthProvider({ children }: { children: ReactNode }) {
  const [token, setToken] = useState<string | null>(null)

  // token 只存在于内存中，页面刷新后需要重新登录
  return (
    <AuthContext.Provider value={{ token, setToken }}>
      {children}
    </AuthContext.Provider>
  )
}
```

### 日志中的敏感信息

```tsx
// ❌ Bad - 日志中包含敏感信息
console.log('User login:', {
  email: user.email,
  password: user.password,  // ❌ 不要记录密码
  token: user.token,        // ❌ 不要记录完整 token
})

// ✅ Good - 脱敏处理
console.log('User login:', {
  email: maskEmail(user.email),
  userId: user.id,
})

function maskEmail(email: string): string {
  const [name, domain] = email.split('@')
  return `${name[0]}***@${domain}`
}
```

## 输入验证

### 客户端验证

```tsx
// ✅ Good - 表单验证
interface LoginFormData {
  email: string
  password: string
}

const loginSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
})

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  })

  const onSubmit = async (data: LoginFormData) => {
    // 客户端验证通过后发送到服务器
    await api.login(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}

      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}

      <button type="submit">Login</button>
    </form>
  )
}
```

### 服务器端验证

```typescript
// ⚠️ 重要: 客户端验证不能替代服务器端验证
// 服务器端必须重新验证所有输入

// server.ts
app.post('/api/login', async (req, res) => {
  // 服务器端验证
  const result = loginSchema.safeParse(req.body)
  if (!result.success) {
    return res.status(400).json({ error: result.error })
  }

  // 处理登录逻辑
})
```

## API 安全

### 请求头安全

```tsx
// ✅ Good - 设置安全请求头
async function apiRequest(url: string, data: unknown) {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Requested-With': 'XMLHttpRequest',  // 防止 CSRF
    },
    credentials: 'same-origin',  // 包含 cookies
    body: JSON.stringify(data),
  })
  return response.json()
}
```

### 错误处理

```tsx
// ✅ Good - 不在错误中暴露敏感信息
async function fetchData() {
  try {
    const response = await fetch('/api/data')
    if (!response.ok) {
      throw new Error('Failed to fetch data')
    }
    return await response.json()
  } catch (error) {
    // ❌ Bad - console.error 可能暴露敏感信息
    // console.error('Request failed:', error)

    // ✅ Good - 记录错误但不暴露细节给用户
    console.error('Request failed')
    throw new Error('An error occurred. Please try again.')
  }
}
```

## Content Security Policy (CSP)

### 配置 CSP

```tsx
// ✅ Good - 设置 CSP 头
// 服务器端配置:
// Content-Security-Policy:
//   default-src 'self';
//   script-src 'self' 'unsafe-inline' 'unsafe-eval';
//   style-src 'self' 'unsafe-inline';
//   img-src 'self' data: https:;
//   connect-src 'self' https://api.example.com;
//   font-src 'self';

// 或者使用 nonce
const nonce = crypto.randomUUID()
// Content-Security-Policy: script-src 'self' 'nonce-{nonce}'
```

## 依赖安全

### 定期更新依赖

```json
// package.json
{
  "scripts": {
    "audit": "npm audit",
    "audit:fix": "npm audit fix",
    "outdated": "npm outdated",
    "update": "npm update"
  }
}
```

### 使用 Snyk 或 Dependabot

```yaml
# .github/dependabot.yml
version: 2
dependencies:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

## 安全检查清单

### XSS 防护
- [ ] 避免使用 dangerouslySetInnerHTML
- [ ] 如果必须使用，先使用 DOMPurify 清理
- [ ] 验证和清理用户输入
- [ ] URL 验证和编码

### CSRF 防护
- [ ] 使用 CSRF token
- [ ] 设置 SameSite cookie
- [ ] 验证请求来源

### 认证和授权
- [ ] 路由级别权限控制
- [ ] 组件级别权限控制
- [ ] 正确处理认证失败

### 敏感数据保护
- [ ] 不在 localStorage 存储敏感信息
- [ ] 使用 httpOnly cookie 存储 token
- [ ] 日志脱敏
- [ ] 错误信息不暴露敏感细节

### API 安全
- [ ] 使用 HTTPS
- [ ] 设置安全请求头
- [ ] 服务器端验证所有输入
- [ ] 实施速率限制
- [ ] 记录安全事件

### 依赖安全
- [ ] 定期更新依赖
- [ ] 运行安全审计
- [ ] 使用 Dependabot 或类似工具
