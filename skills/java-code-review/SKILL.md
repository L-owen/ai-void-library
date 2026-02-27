---
name: java-code-review
description: Java 代码审查技能，用于 Pull Request 代码质量审查。重点关注功能逻辑、安全性、性能、架构扩展性和稳定性五个维度。基于 IntelliJ 平台开发最佳实践。当用户请求审查 Java 代码、检查 PR、分析代码质量或请求代码审查报告时使用此技能。
---

# Java Code Review

基于 IntelliJ 平台开发的 Java 代码审查技能，用于 Pull Request 代码质量审查。

## 审查维度

本技能从五个维度进行代码审查：

1. **功能逻辑** - 业务逻辑正确性、空指针处理、边界条件、异常处理
2. **安全性** - SQL 注入、XSS/CSRF、权限控制、敏感数据处理、加密
3. **性能** - PSI 树遍历优化、缓存策略、N+1 查询、资源释放、并发
4. **架构扩展性** - SOLID 原则、设计模式、模块化解耦、接口设计
5. **稳定性** - 异常处理、资源泄露、并发安全、日志记录、容错降级

## 工作流程

### Step 1: 获取 PR 信息

使用 GitHub 工具获取 PR 的详细信息：

```bash
# 获取 PR 的文件变更
gh pr view <pr_number> --json files,title,number,author,headRefName
```

需要提取的信息：
- PR 标题和编号
- 作者和分支
- 变更的文件列表
- 代码增删行数

### Step 2: 读取并分析代码

对于每个变更的 Java 文件：

1. 使用 `gh pr diff` 获取完整的 diff
2. 识别新增和修改的代码块
3. 分析每个变更是否符合最佳实践

**关键原则：**
- 只审查新增和修改的代码（关注 `+` 开头的行）
- 避免对未变更代码进行无谓的批评
- 优先关注严重和高优先级问题

### Step 3: 按维度审查代码

根据变更内容，参考对应的参考文档进行审查：

#### 功能逻辑审查
参考：`references/stability-checks.md`
- 空指针检查
- 边界条件处理
- 异常处理策略
- 业务逻辑正确性

#### 安全性审查
参考：`references/security-checklist.md`
- SQL 注入风险
- XSS/CSRF 防护
- 权限控制
- 敏感数据处理
- 加密和数据保护

#### 性能审查
参考：`references/performance-patterns.md`
- PSI 树遍历优化（IntelliJ 特有）
- 集合操作优化
- 缓存策略
- N+1 查询问题
- 资源释放
- 并发性能

#### 架构扩展性审查
参考：`references/architecture-principles.md`
- SOLID 原则遵循
- 设计模式使用
- 模块化和解耦
- 接口设计
- 依赖注入

#### 稳定性审查
参考：`references/stability-checks.md`
- 异常处理策略
- 资源泄露检查
- 并发问题
- 日志记录
- 容错降级机制

### Step 4: 评定问题严重等级

根据问题的影响程度评定等级：

**Critical（严重）**
- 安全漏洞（SQL 注入、XSS、权限绕过）
- 资源泄露（未关闭的流、连接、监听器）
- 数据损坏或丢失风险
- 严重并发问题（死锁、竞态条件）
- 系统崩溃风险

**High（高）**
- 性能瓶颈（N+1 查询、未优化的循环）
- 明显的架构缺陷（违反 SOLID 原则）
- 异常处理缺失
- 潜在的空指针异常

**Medium（中）**
- 代码重复
- 不合理的命名
- 缺少必要的日志
- 可优化的代码结构

**Low（低）**
- 代码风格问题
- 注释不足
- 小的优化建议

### Step 5: 生成 HTML 报告

使用模板生成结构化的 HTML 报告：

1. 读取 `assets/review-report-template.html`
2. 读取 `assets/styles.css`
3. 替换模板中的占位符：
   - `{PR_TITLE}`, `{PR_NUMBER}`, `{PR_AUTHOR}`, `{PR_BRANCH}`, `{REVIEW_DATE}`
   - `{CRITICAL_COUNT}`, `{HIGH_COUNT}`, `{MEDIUM_COUNT}`, `{LOW_COUNT}`, `{TOTAL_ISSUES}`
   - `{FILES_COUNT}`, `{ADDITIONS}`, `{DELETIONS}`
   - `{CRITICAL_ISSUES}`, `{FUNCTIONAL_ISSUES}`, `{SECURITY_ISSUES}`, `{PERFORMANCE_ISSUES}`, `{ARCHITECTURE_ISSUES}`, `{STABILITY_ISSUES}`
4. 为每个问题生成 Issue Card HTML
5. 输出最终的 HTML 报告

## Issue Card HTML 结构

每个问题的 HTML 格式：

```html
<div class="issue-card [critical|high|medium|low]">
    <div class="issue-header">
        <h3 class="issue-title">问题标题</h3>
        <div class="issue-meta">
            <span class="issue-badge [critical|high|medium|low]">[严重等级]</span>
            <span class="issue-badge category">[维度]</span>
        </div>
    </div>
    <div class="issue-location">
        <strong>位置:</strong> <code>文件路径:行号</code>
    </div>
    <div class="issue-description">
        问题的详细描述，说明为什么这是一个问题以及可能的影响。
    </div>
    <div class="issue-suggestion">
        <h4>修复建议:</h4>
        <ul>
            <li>具体的修复步骤</li>
            <li>代码示例</li>
        </ul>
    </div>
    <div class="code-block">
        <span class="code-line"><span class="code-line-number">123</span>问题代码片段</span>
        <span class="code-line code-line-highlight"><span class="code-line-number">124</span>高亮显示的问题行</span>
    </div>
</div>
```

## 参考文档使用指南

### references/intellij-best-practices.md
**何时阅读：** 当代码涉及 IntelliJ 平台开发时

包含内容：
- PSI (Program Structure Interface) 使用
- Read Action vs Write Action
- EDT 线程使用
- ApplicationManager 和 Project 使用
- Extension Point 和 AnAction 最佳实践
- 资源管理和生命周期

### references/security-checklist.md
**何时阅读：** 进行安全性审查时

包含内容：
- SQL Injection 防护
- XSS 和 CSRF 防护
- 认证和授权
- 敏感数据处理
- 加密和数据保护
- 反序列化安全
- 依赖安全
- 文件上传安全

### references/performance-patterns.md
**何时阅读：** 进行性能审查时

包含内容：
- PSI Tree 遍历优化
- 集合操作优化
- 缓存策略
- N+1 查询问题
- 资源释放
- 并发性能
- I/O 性能
- 内存优化

### references/architecture-principles.md
**何时阅读：** 进行架构审查时

包含内容：
- SOLID 原则详解
- 设计模式识别（单例、工厂、策略、观察者、装饰器）
- 模块化和解耦
- 接口设计
- 依赖注入和控制反转
- 领域驱动设计（DDD）
- 错误处理和异常设计

### references/stability-checks.md
**何时阅读：** 进行稳定性审查时

包含内容：
- 异常处理策略
- 资源泄露检查
- 空指针和边界条件
- 并发问题
- 日志记录和错误追踪
- 容错和降级机制
- 配置和参数验证

## 输出文件

最终的审查报告应保存为：
```
review-report-pr-<pr_number>-<timestamp>.html
```

包含完整的样式表（内联 CSS）以便于分享和查看。

## 审查原则

1. **建设性** - 提供具体的改进建议，而非单纯的批评
2. **优先级** - 优先标记严重和高优先级问题
3. **上下文** - 考虑代码的实际使用场景和业务需求
4. **可操作性** - 每个问题都应提供明确的修复方案
5. **谦逊** - 如果不确定，标记为建议而非问题
