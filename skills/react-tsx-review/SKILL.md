---
name: react-tsx-review
description: React + TypeScript 代码审查技能。五维度分析：React 最佳实践、TypeScript 类型安全、性能优化、安全规范、代码质量。触发条件：用户说 "review React"、"React 代码审查"、"TSX review"、提供 .tsx/.ts 文件、或要求 React/TSX 代码审查。输入：文件列表、diff 内容。输出：五维度审查结果、HTML 报告。
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
- MR 信息（owner/repo/ref，用于按需加载完整文件）

**说明**：初始使用 diff 审查，拿不准时主动获取完整文件内容。

### 审查原则

**不确定性优先**：基于 diff 审查，拿不准时获取完整上下文，不武断下结论。

**审查范围**：重点审查新增和修改的代码（`+` 开头的行），按需深入分析。

**质量优先**：优先关注严重问题，提供具体改进建议，不确定时标记为"建议验证"或加载更多上下文。

### 按需上下文加载

**触发条件**（满足任一即获取完整文件）：
- 无法确定影响范围（如：修改了 props 但不确定父组件如何传递）
- 缺少上下文信息（如：只看到 Hook 调用，没有依赖数组）
- 需要理解状态管理关系（如：修改了 state，但不确定哪些地方依赖）
- 拿不准的问题（如：不确定是否会导致重渲染或内存泄漏）

**如何获取**：
```
mcp__github__get_file_contents(owner, repo, path, ref)
```

**加载后**：解析组件结构 → 定位修改位置 → 分析影响范围 → 重新评估问题

**报告要求**：说明是否加载了完整上下文，哪些判断基于 diff、哪些基于完整文件。

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
