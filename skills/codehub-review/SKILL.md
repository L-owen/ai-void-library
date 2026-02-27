---
name: codehub-review
description: 系统化的代码审查工作流程。用于 MR/PR 代码审查，根据变更复杂度选择审查深度（直接/深度/本地），集成语言特定的审查技能。触发条件：用户说 "review"、"审查"、提供 MR/PR 链接、或要求代码审查。本技能专注于审查流程，不定义具体编码规则。
---

# CodeHub Review

本技能提供系统化的 Code Review 工作流程，通过 codehub-review-mcp 获取 MR 信息并进行分层 review。核心特点是**流程驱动**而非规则驱动。

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

使用 codehub-review-mcp 工具获取：
- MR 元数据（ID、标题、作者、分支）
- 变更文件列表和 Diff 统计
- 评估复杂度（文件数、行数、跨文件影响）

详细流程参见 [references/review-workflow.md](references/review-workflow.md)

### [阶段 1.5] 按需获取完整上下文 🆕

**核心策略**：初始审查使用 diff，让 agent 自主评估是否需要完整上下文。

**按需加载触发条件**（当以下情况时主动获取完整文件）：
- 无法确定影响范围（如：修改了私有方法但不确定调用方）
- 缺少上下文信息（如：只看到方法签名，没有完整实现）
- 需要理解继承/依赖关系（如：修改接口方法，需查看实现类）
- 拿不准的问题（基于现有信息无法判断）

**如何获取完整文件**：
```
mcp__github__get_file_contents(owner, repo, path, ref)
```

**报告要求**：明确说明是否加载了完整上下文及原因。

### [阶段 2] 分层 Review

根据复杂度选择审查深度：

#### [2A] 直接 Review（< 100 行）
1. 检测文件扩展名
2. 如果是 `.java`：使用 Skill 工具调用 `java-code-review`
3. 如果是 `.tsx`, `.ts`, `.jsx`：使用 Skill 工具调用 `react-tsx-review`
4. 整合结果

#### [2B] 深度 Review（100-500 行）
1. 检测文件扩展名
2. 如果是 `.java`：使用 Skill 工具调用 `java-code-review`
3. 如果是 `.tsx`, `.ts`, `.jsx`：使用 Skill 工具调用 `react-tsx-review`
4. 读取完整上下文，跨文件检查
5. 整合结果

#### [2C] 本地 Review（> 500 行）
1. 下载 patch，应用到本地
2. 检测文件类型
3. 静态分析 + 测试
4. 深度 review

详细步骤参见 [references/review-workflow.md](references/review-workflow.md)

### [阶段 3] 生成报告

包含：
- 摘要（变更概述、主要发现）
- 问题列表（按严重程度分组）
- 正面反馈
- 后续行动

## 语言特定技能集成

### 检测和调用规则

当检测到文件扩展名时，使用 Skill 工具调用对应技能：

**Java 文件**：调用 `java-code-review`，传入文件列表、diff、MR 信息（owner/repo/ref）

**TSX/TS 文件**：调用 `react-tsx-review`，传入文件列表、diff、MR 信息（owner/repo/ref）

说明：初始只传递 diff，agent 会根据需要自行调用 `get_file_contents` 获取完整文件。

### 技能对比

| 文件类型 | 调用技能 | 审查维度 |
|---------|---------|---------|
| `.java` | `java-code-review` | 功能逻辑、安全性、性能、架构、稳定性 |
| `.tsx`, `.ts`, `.jsx` | `react-tsx-review` | React 最佳实践、TypeScript、性能、安全、质量 |

### 结果整合

- 按严重程度（Critical/High/Medium/Low）合并问题
- 标注来源：`[Java Review - 安全性]`、`[React Review - TypeScript]`

## MCP 工具

- `get_mr_info` - 获取 MR 元数据
- `get_mr_diff` - 获取 Diff
- `get_file_contents` - 按需获取完整文件内容（agent 自主判断何时调用）
- `download_diff` - 下载 patch（> 500 行复杂变更使用）

参见 [references/mcp-tools-guide.md](references/mcp-tools-guide.md)

## 关键原则

1. **流程驱动** - 提供流程，不定义规则
2. **语言感知** - 检测语言，调用专业技能
3. **分层处理** - 根据复杂度选择深度
4. **建设性反馈** - 提供可行建议
