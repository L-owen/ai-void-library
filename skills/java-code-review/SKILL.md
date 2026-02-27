---
name: java-code-review
description: Java 代码审查技能。五维度分析：功能逻辑、安全性、性能、架构扩展性、稳定性。基于 IntelliJ 平台最佳实践。触发条件：用户说 "review Java"、"Java 代码审查"、提供 .java 文件、或要求 Java 代码审查。输入：文件列表、diff 内容。输出：五维度审查结果、HTML 报告。
---

# Java Code Review

基于 IntelliJ 平台开发的 Java 代码审查技能，专注于代码质量分析。

## 审查维度

1. **功能逻辑** - 业务逻辑正确性、空指针处理、边界条件、异常处理
2. **安全性** - SQL 注入、XSS/CSRF、权限控制、敏感数据处理
3. **性能** - PSI 树遍历、缓存策略、N+1 查询、资源释放、并发
4. **架构扩展性** - SOLID 原则、设计模式、模块化、接口设计
5. **稳定性** - 异常处理、资源泄露、并发安全、日志记录、容错降级

## 审查流程

### 输入

- 文件列表（`.java` 文件）
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
- 无法确定影响范围（如：修改了私有方法但不确定调用方）
- 缺少上下文信息（如：只看到方法签名，没有实现逻辑）
- 需要理解继承/接口关系（如：修改接口方法，需查看实现类）
- 拿不准的问题（基于现有信息无法判断）

**如何获取**：
```
mcp__github__get_file_contents(owner, repo, path, ref)
```

**加载后**：解析文件结构 → 定位修改位置 → 分析影响范围 → 重新评估问题

**报告要求**：说明是否加载了完整上下文，哪些判断基于 diff、哪些基于完整文件。

### 按维度审查

| 维度 | 参考文档 |
|-----|---------|
| 功能逻辑 | `references/stability-checks.md` |
| 安全性 | `references/security-checklist.md` |
| 性能 | `references/performance-patterns.md` |
| 架构扩展性 | `references/architecture-principles.md` |
| 稳定性 | `references/stability-checks.md` |

**IntelliJ 平台特有**：`references/intellij-best-practices.md`

### 问题等级

| 等级 | 示例 |
|-----|-----|
| Critical | SQL 注入、资源泄露、死锁、系统崩溃风险 |
| High | N+1 查询、违反 SOLID、异常处理缺失、空指针风险 |
| Medium | 代码重复、命名不合理、缺少日志 |
| Low | 代码风格、注释不足、小优化建议 |

## 生成报告

1. 读取 `assets/review-report-template.html`
2. 替换占位符：`{PR_TITLE}`, `{CRITICAL_COUNT}`, `{FUNCTIONAL_ISSUES}` 等
3. 生成 Issue Card（结构见 `references/issue-card-structure.md`）
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
