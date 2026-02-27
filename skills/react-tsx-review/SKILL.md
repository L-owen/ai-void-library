---
name: react-tsx-review
description: React + TypeScript 代码审查技能。当用户说 "review React"、"React 代码审查"、"TSX review"、审查 .tsx/.ts 文件、或包含 "React"+"review" 时触发。五维度分析：React 最佳实践、TypeScript 类型安全、性能优化、安全规范、代码质量。传入文件列表和 diff，返回审查结果和 HTML 报告。
---

# React + TypeScript Code Review

React + TypeScript 前端代码审查技能，专注于 React 组件代码质量分析。

## 审查维度

1. **React 最佳实践** - 组件设计、Hooks 规则、状态管理、错误边界
2. **TypeScript 类型安全** - 类型定义、泛型使用、类型守卫、避免 any
3. **性能优化** - 渲染优化、懒加载、虚拟化、缓存策略
4. **安全规范** - XSS/CSRF 防护、认证授权、敏感数据处理
5. **代码质量** - 可读性、可维护性、DRY 原则、命名规范

## 审查流程

### 输入
- 文件列表（`.tsx`, `.ts`, `.jsx` 文件）
- Diff 内容（新增和修改的代码）
- PR 元数据（可选）

### 审查原则
- 只审查新增和修改的代码（`+` 开头的行）
- 优先关注严重和高优先级问题

### 按维度审查

| 维度 | 参考文档 |
|-----|---------|
| React 最佳实践 | `reference/react-best-practices.md` |
| TypeScript 类型安全 | `reference/typescript-best-practices.md` |
| 性能优化 | `reference/performance-patterns.md` |
| 安全规范 | `reference/security-checklist.md` |
| 代码质量 | `reference/code-quality.md` |

### 问题等级

| 等级 | 示例 |
|-----|-----|
| Critical | XSS 漏洞、token 存 localStorage、内存泄漏、无限重渲染 |
| High | 不必要重渲染、过度使用 any、未清理副作用、违反 Hooks 规则 |
| Medium | 代码重复、缺少类型、不当使用 memo/useMemo |
| Low | 命名不规范、代码风格、小优化建议 |

## 生成报告

1. 读取 `assets/review-report-template.html`
2. 替换占位符：`{PR_TITLE}`, `{CRITICAL_COUNT}`, `{REACT_ISSUES}` 等
3. 生成 Issue Card（结构见 `reference/issue-card-structure.md`）
4. 输出：`review-report-pr-<pr_number>-<timestamp>.html`

## 输出格式

```typescript
{
  critical: Issue[],
  high: Issue[],
  medium: Issue[],
  low: Issue[],
  summary: { total, byDimension },
  positiveFeedback: string[]
}
```

## 审查原则

1. **建设性** - 提供具体改进建议
2. **优先级** - 优先标记严重问题
3. **上下文** - 考虑实际使用场景
4. **可操作性** - 明确修复方案
5. **谦逊** - 不确定时标记为建议
