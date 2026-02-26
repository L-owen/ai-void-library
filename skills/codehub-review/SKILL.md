---
name: codehub-review
description: Systematic code review workflow for merge requests using codehub-mcp-server. Use when user asks to review an MR/PR or provides a merge request link. Handles fetching MR information, assessing change complexity, reviewing code changes with appropriate depth, downloading and applying patches locally for complex changes, and integrating with coding standards skills for rule-based review. This skill focuses on the REVIEW PROCESS and WORKFLOW, not specific review rules which are provided by other skills.
---

# CodeHub Review

## 概述

本技能提供系统化的 Code Review 工作流程，通过 codehub-mcp-server 获取 MR 信息并进行分层 review。核心特点是**流程驱动**而非规则驱动——具体的编码规范由其他 skills 提供。

## 核心工作流程

### 触发条件

当用户说出以下内容时触发此技能：
- "Review 这个 MR" / "Review 这个 PR"
- "帮我看一下这个 merge request"
- 提供一个 MR/PR 链接
- 要求审查某次代码变更

### Review 流程决策树

```
用户请求 Review
    ↓
[阶段 1] 获取 MR 信息
    - 使用 MCP 工具获取元数据和变更
    ↓
评估变更复杂度
    ↓
    ├─ 简单 (< 100 行, 单文件) → [阶段 2A] 直接 Review
    ├─ 中等 (100-500 行, 多文件) → [阶段 2B] 深度 Review
    └─ 复杂 (> 500 行, 重构等) → [阶段 2C] 本地上下文 Review
    ↓
[阶段 3] 生成 Review 报告
```

### 工作流程详解

#### [阶段 1] 获取 MR 信息

使用 codehub-mcp-server 提供的工具获取：

1. **MR 元数据**
   - MR 链接/ID
   - 提交者和提交时间
   - MR 标题和描述
   - 目标分支

2. **代码变更**
   - 变更文件列表
   - Diff 统计（增删行数）

3. **评估复杂度**
   - 文件数量
   - 代码行数变化
   - 跨文件影响程度

详细流程参见 [references/review-workflow.md](references/review-workflow.md#阶段-1-获取-mr-信息)

#### [阶段 2A] 直接 Review（简单变更）

适用于：单文件、少量改动（< 100 行）、无跨文件影响

**执行步骤**:

1. 读取每个变更文件的 diff
2. 调用编码规范 skill 进行规则检查
3. 识别潜在问题
4. 生成结构化的 review 意见

#### [阶段 2B] 深度 Review（中等变更）

适用于：多文件、中等改动（100-500 行）、有限的跨文件影响

**执行步骤**:

1. 读取相关文件的完整内容（不仅是 diff）
2. 理解变更的上下文和影响范围
3. 按功能模块分组审查
4. 检查跨文件一致性
5. 调用编码规范 skill
6. 生成详细的 review 报告

#### [阶段 2C] 本地上下文 Review（复杂变更）

适用于：大量文件、大量改动（> 500 行）、复杂重构

**执行步骤**:

1. **下载并应用 Patch**
   - 使用 MCP 工具下载完整 diff
   - 将 patch 应用到本地工程
   - 验证应用成功

2. **本地分析**
   - 使用完整代码库进行静态分析
   - 运行测试（如果可用）
   - 理解整体架构和依赖关系

3. **深度 Review**
   - 结合完整上下文进行审查
   - 分析对系统的整体影响
   - 评估性能和安全影响

详细流程参见 [references/complex-change-handling.md](references/complex-change-handling.md)

#### [阶段 3] 生成 Review 结果

无论使用哪个子流程，都应生成包含以下内容的结果：

1. **摘要**
   - 变更概述
   - 主要发现
   - 总体评价

2. **问题列表**
   - 按严重程度分组（严重/中等/建议）
   - 每个问题包含：
     * 问题描述
     * 文件和行号
     * 具体改进建议
     * 引用的规范（如适用）

3. **正面反馈**
   - 做得好的地方
   - 值得学习的实践

4. **后续行动**
   - 必须修复的问题
   - 建议修改的问题
   - 可选的优化建议

## MCP 工具使用

本技能依赖 codehub-mcp-server 提供的工具。可用的工具包括：

- MR 信息获取工具（get_mr_info, get_mr_diff 等）
- Diff 下载工具（download_diff）
- 文件内容获取工具（get_file_content）

**注意**: 具体工具列表和参数待补充。参见 [references/mcp-tools-guide.md](references/mcp-tools-guide.md)

## 与编码规范技能的集成

本技能专注于 **review 流程**，具体的编码规则由其他 skills 提供。当需要进行规则检查时：

1. 识别适用的编码规范 skill
2. 调用该 skill 获取具体的审查规则
3. 应用规则检查代码
4. 将结果整合到 review 报告中

常见的编码规范技能包括：
- 代码风格检查
- 安全规范检查
- 性能最佳实践
- 语言特定的规范

## 关键原则

1. **理解优先于批评** - 先理解变更目的，再进行审查
2. **流程驱动** - 本技能提供 systematic 流程，不定义具体规则
3. **分层处理** - 根据变更复杂度选择合适的 review 深度
4. **规范集成** - 与编码规范技能协作，不重复定义规则
5. **建设性反馈** - 提供具体可行的改进建议
6. **上下文感知** - 复杂变更需要完整代码库支持

## 参考资料

本技能包含以下参考资料，在需要时阅读：

- **[review-workflow.md](references/review-workflow.md)** - 完整的 review 工作流程文档
- **[mcp-tools-guide.md](references/mcp-tools-guide.md)** - MCP 工具使用指南（TODO）
- **[complex-change-handling.md](references/complex-change-handling.md)** - 复杂变更的本地处理流程

## 使用示例

### 示例 1: 简单变更

**用户**: "Review 一下这个 MR: https://codehub.example.com/mr/123"

**Agent 流程**:
1. 使用 MCP 工具获取 MR #123 的信息
2. 识别为简单变更（单文件，50 行变更）
3. 直接读取 diff，调用编码规范 skill
4. 生成 review 意见

### 示例 2: 复杂变更

**用户**: "帮我看一下这个重构 MR: https://codehub.example.com/mr/456"

**Agent 流程**:
1. 获取 MR #456 信息
2. 识别为复杂变更（30 个文件，2000+ 行，架构重构）
3. 下载 diff，应用到本地
4. 运行测试，进行静态分析
5. 结合完整上下文深度 review
6. 生成综合报告
7. 清理本地变更
