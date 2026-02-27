# TypeScript Best Practices

TypeScript 在 React 项目中的最佳实践参考指南。

## 类型定义

### 接口 vs 类型别名

#### 使用 interface
- 定义对象形状
- 需要扩展时
- 定义类结构

```tsx
// ✅ Good - 使用 interface 定义对象形状
interface User {
  id: number
  name: string
  email: string
}

// ✅ Good - interface 可以扩展
interface AdminUser extends User {
  permissions: string[]
  role: 'admin'
}

// ✅ Good - 类实现 interface
class UserService implements IUserService {
  getUser(id: number): User {
    // ...
  }
}
```

#### 使用 type
- 联合类型
- 交叉类型
- 映射类型
- 条件类型

```tsx
// ✅ Good - 联合类型
type Theme = 'light' | 'dark' | 'auto'

// ✅ Good - 联合多个类型
type ButtonProps = HTMLButtonElement['props'] & {
  variant: 'primary' | 'secondary'
}

// ✅ Good - 条件类型
type NonNullable<T> = T extends null | undefined ? never : T
```

### Props 类型定义

```tsx
// ✅ Good - 使用 React.FunctionComponent
interface ButtonProps {
  variant: 'primary' | 'secondary'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  onClick: () => void
  children: React.ReactNode
}

const Button: React.FunctionComponent<ButtonProps> = ({
  variant,
  size = 'md',
  disabled = false,
  onClick,
  children
}) => {
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

// ✅ Good - 简洁的函数声明
function Button({ variant, children }: ButtonProps) {
  return <button className={`btn btn-${variant}`}>{children}</button>
}
```

### 避免使用 any

```tsx
// ❌ Bad - 使用 any
function process(data: any) {
  return data.value
}

// ✅ Good - 使用具体类型
interface Data {
  value: string
}

function process(data: Data) {
  return data.value
}

// ✅ Good - 使用泛型
function process<T extends { value: string }>(data: T): string {
  return data.value
}

// ✅ Good - 如果确实不知道类型，使用 unknown
function process(data: unknown) {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return (data as { value: string }).value
  }
  throw new Error('Invalid data')
}
```

### 泛型最佳实践

```tsx
// ✅ Good - 约束泛型
interface Identifiable {
  id: number | string
}

function findById<T extends Identifiable>(items: T[], id: number | string): T | undefined {
  return items.find(item => item.id === id)
}

// ✅ Good - 泛型默认值
interface ApiResponse<T = unknown> {
  data: T
  status: number
  message: string
}

// ✅ Good - 多个泛型
function mapObject<K extends string, V, R>(
  obj: Record<K, V>,
  mapper: (value: V, key: K) => R
): Record<K, R> {
  const result = {} as Record<K, R>
  for (const key in obj) {
    result[key] = mapper(obj[key], key)
  }
  return result
}
```

## 类型推断

### 让 TypeScript 推断类型

```tsx
// ✅ Good - TypeScript 推断类型
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
] // 推断为 { id: number; name: string; }[]

// ❌ Bad - 不必要的类型注解
const users: Array<{ id: number; name: string }> = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
]

// ✅ Good - 函数返回类型可以省略（简单情况）
function add(a: number, b: number) {
  return a + b
}

// ✅ Good - 公共 API 应该明确返回类型
interface UserService {
  getUser(id: number): Promise<User>  // ✅ 明确返回类型
}
```

## 类型守卫

### 使用类型守卫缩小类型范围

```tsx
// ✅ Good - typeof 类型守卫
function process(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase()  // TypeScript 知道这里是 string
  }
  return value.toFixed(2)  // TypeScript 知道这里是 number
}

// ✅ Good - instanceof 类型守卫
function errorToString(error: unknown): string {
  if (error instanceof Error) {
    return error.message
  }
  return String(error)
}

// ✅ Good - 自定义类型守卫
interface User {
  name: string
  email: string
}

interface Admin {
  name: string
  permissions: string[]
}

function isAdmin(user: User | Admin): user is Admin {
  return 'permissions' in user
}

function processUser(user: User | Admin) {
  if (isAdmin(user)) {
    console.log(user.permissions)  // ✅ Admin
  } else {
    console.log(user.email)  // ✅ User
  }
}
```

## 严格模式配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,                    // 启用所有严格类型检查选项
    "noImplicitAny": true,             // 禁止隐式 any
    "strictNullChecks": true,          // 严格的 null 检查
    "strictFunctionTypes": true,       // 严格的函数类型检查
    "strictPropertyInitialization": true,  // 严格的类属性初始化
    "noImplicitThis": true,            // 禁止 this 为 any
    "alwaysStrict": true,              // 总是严格模式
    "noUnusedLocals": true,            // 检查未使用的局部变量
    "noUnusedParameters": true,        // 检查未使用的参数
    "noImplicitReturns": true,         // 检查所有代码路径是否返回
    "noFallthroughCasesInSwitch": true  // switch 穿透检查
  }
}
```

## 常见类型陷阱

### 可选属性 vs undefined

```tsx
// ❌ Bad
interface User {
  name: string
  email?: string  // email 可能是 string | undefined
}

function getUserEmail(user: User): string {
  return user.email  // ❌ 可能是 undefined
}

// ✅ Good
function getUserEmail(user: User): string {
  return user.email ?? ''  // ✅ 提供默认值
}

// ✅ Good
function getUserEmail(user: User): string | undefined {
  return user.email  // ✅ 返回类型包含 undefined
}
```

### 类型断言谨慎使用

```tsx
// ❌ Bad - 滥用类型断言
const value = data as User

// ✅ Good - 使用类型守卫
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  )
}

if (isUser(data)) {
  // data 现在是 User 类型
}
```

### 数组类型

```tsx
// ✅ Good
const numbers: number[] = [1, 2, 3]
const numbers: Array<number> = [1, 2, 3]

// ✅ Good - 只读数组
const readOnly: readonly number[] = [1, 2, 3]
const readOnly: ReadonlyArray<number> = [1, 2, 3]
const readOnly = [1, 2, 3] as const  // 最严格的只读

// ✅ Good - 元组
const tuple: [string, number] = ['hello', 42]
```

## 工具类型

### 常用工具类型

```tsx
// Partial - 所有属性可选
type PartialUser = Partial<User>

// Required - 所有属性必需
type RequiredUser = Required<User>

// Readonly - 所有属性只读
type ReadonlyUser = Readonly<User>

// Pick - 选择部分属性
type UserSummary = Pick<User, 'id' | 'name'>

// Omit - 排除部分属性
type CreateUserInput = Omit<User, 'id'>

// Record - 构建对象类型
type UsersById = Record<number, User>

// Exclude - 排除联合类型成员
type ThemeWithoutAuto = Exclude<Theme, 'auto'>

// Extract - 提取联合类型成员
type StringOrNumber = Extract<string | number | boolean, string | number>

// NonNullable - 排除 null 和 undefined
type NonNullableString = NonNullable<string | null | undefined>
```

## React 相关类型

### 事件类型

```tsx
// ✅ Good - 正确的事件类型
function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
  setValue(event.target.value)
}

function handleClick(event: React.MouseEvent<HTMLButtonElement>) {
  event.preventDefault()
}

function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
  event.preventDefault()
}

function handleInput(event: React.FormEvent<HTMLInputElement>) {
  const target = event.target as HTMLInputElement  // 类型断言
  setValue(target.value)
}
```

### Ref 类型

```tsx
// ✅ Good
const inputRef = useRef<HTMLInputElement>(null)
const divRef = useRef<HTMLDivElement>(null)

useEffect(() => {
  inputRef.current?.focus()
}, [])
```

### Props 类型

```tsx
// ✅ Good - 使用 React.ComponentProps
type ButtonProps = React.ComponentProps<'button'> & {
  variant: 'primary' | 'secondary'
}

// ✅ Good - 继承原生元素属性
interface InputProps extends React.ComponentProps<'input'> {
  label: string
  error?: string
}

function Input({ label, error, ...props }: InputProps) {
  return (
    <div>
      <label>{label}</label>
      <input {...props} />
      {error && <span className="error">{error}</span>}
    </div>
  )
}
```

## 类型导入/导出

```tsx
// ✅ Good - 类型导入
import type { User, UserService } from './types'
import { userService } from './services'

// ✅ Good - 类型导出
export type { User, Admin }

// ✅ Good - 内联类型导出
export interface User {
  id: number
  name: string
}
```
