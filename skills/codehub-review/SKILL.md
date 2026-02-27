---
name: codehub-review
description: 系统化的 Code Review 工作流程，用于 MR/PR 代码审查。使用 codehub-mcp-server 获取信息，根据变更复杂度选择审查深度（直接/深度/本地），并集成语言特定的审查技能（java-code-review、react-tsx-review）。本技能专注于审查流程和工作流，不定义具体的编码规则。
---

# CodeHub Review

本技能提供系统化的 Code Review 工作流程，通过 codehub-mcp-server 获取 MR 信息并进行分层 review。核心特点是**流程驱动**而非规则驱动。

## 触发条件

- "Review 这个 MR/PR"
- 提供一个 MR/PR 链接
- 要求审查某次代码变更

## Review 流程决策树

```
用户请求 Review
    ↓
[阶段 1] 获取 MR 信息并评估复杂度
    ↓
    ├─ 简单 (< 100 行) → [阶段 2A] 直接 Review
    ├─ 中等 (100-500 行) → [阶段 2B] 深度 Review
    └─ 复杂 (> 500 行) → [阶段 2C] 本地上下文 Review
    ↓
[阶段 3] 生成 Review 报告
```

## 工作流程

### [阶段 1] 获取 MR 信息

使用 codehub-mcp-server 工具获取：
- MR 元数据（ID、标题、作者、分支）
- 变更文件列表和 Diff 统计
- 评估复杂度（文件数、行数、跨文件影响）

详细流程参见 [references/review-workflow.md](references/review-workflow.md)

### [阶段 2] 分层 Review

根据复杂度选择审查深度：

#### [2A] 直接 Review（< 100 行）
检测语言 → 调用对应 review skill → 整合结果

#### [2B] 深度 Review（100-500 行）
检测语言 → 调用对应 review skill → 读取完整上下文 → 跨文件检查 → 整合结果

#### [2C] 本地 Review（> 500 行）
下载 patch → 应用到本地 → 静态分析 + 测试 → 深度 review

详细步骤参见 [references/review-workflow.md](references/review-workflow.md)

### [阶段 3] 生成报告

包含：
- 摘要（变更概述、主要发现）
- 问题列表（按严重程度分组）
- 正面反馈
- 后续行动

## 语言特定技能集成

检测文件扩展名后调用对应技能：

| 文件类型 | 调用技能 | 审查维度 |
|---------|---------|---------|
| `.java` | `java-code-review` | 功能逻辑、安全性、性能、架构、稳定性 |
| `.tsx`, `.ts`, `.jsx` | `react-tsx-review` | React 最佳实践、TypeScript、性能、安全、质量 |

**调用方式**：
```
Skill tool: skill="java-code-review" 或 "react-tsx-review"
传入：文件列表、Diff 内容、PR 元数据
返回：分类的问题列表、HTML 报告
```

**结果整合**：
- 按严重程度（Critical/High/Medium/Low）合并问题
- 标注来源：`[Java Review - 安全性]`、`[React Review - TypeScript]`

## MCP 工具

- `get_mr_info` - 获取 MR 元数据
- `get_mr_diff` - 获取 Diff
- `download_diff` - 下载 patch

参见 [references/mcp-tools-guide.md](references/mcp-tools-guide.md)

## 关键原则

1. **流程驱动** - 提供流程，不定义规则
2. **语言感知** - 检测语言，调用专业技能
3. **分层处理** - 根据复杂度选择深度
4. **建设性反馈** - 提供可行建议
