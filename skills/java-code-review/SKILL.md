---
name: java-code-review
description: Java 代码审查技能。用于审查 Java 代码变更，关注功能逻辑、安全性、性能、架构扩展性和稳定性五个维度。基于 IntelliJ 平台开发最佳实践。当调用此技能时，传入文件变更列表和 diff 内容，返回五维度审查结果和 HTML 报告。
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

### 审查原则
- 只审查新增和修改的代码（`+` 开头的行）
- 优先关注严重和高优先级问题

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
