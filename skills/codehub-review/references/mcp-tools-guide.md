# CodeHub MCP 工具使用指南

## 概述

本文档描述 codehub-review-mcp 提供的工具及其在 Code Review 流程中的使用方法。

## 可用工具

### TODO: 待补充

目前 MCP 工具的具体信息待补充。当工具信息可用时，应包含：

1. **工具名称和描述**
2. **参数说明**
3. **返回值格式**
4. **使用示例**
5. **注意事项**

## 预期的工具类别

基于 Code Review 流程的需求，预期需要以下类型的工具：

### MR 信息获取工具

- `get_mr_info` - 获取 MR 基本信息
  - 参数: mr_url 或 mr_id
  - 返回: 提交者、时间、标题、描述等

- `get_mr_diff` - 获取 MR 的代码变更
  - 参数: mr_url 或 mr_id
  - 返回: 文件列表、diff 内容

- `get_mr_diff_stats` - 获取变更统计
  - 参数: mr_url 或 mr_id
  - 返回: 增删行数、文件数量等

### Diff 下载工具

- `download_diff` - 下载完整 diff 文件
  - 参数: mr_url 或 mr_id, output_path
  - 返回: 下载的文件路径

### 文件内容获取工具

- `get_file_content` - 获取特定文件的内容
  - 参数: file_path, ref (commit/branch)
  - 返回: 文件内容

## 使用流程

### 标准流程

```python
# 1. 获取 MR 信息
mr_info = mcp_tool.get_mr_info(mr_url)

# 2. 获取代码变更
diff_data = mcp_tool.get_mr_diff(mr_url)

# 3. 评估复杂度
complexity = assess_complexity(diff_data)

# 4. 如果复杂，下载 patch
if complexity == "high":
    diff_file = mcp_tool.download_diff(mr_url, output_path)
    apply_patch_locally(diff_file)
```

## 注意事项

- 错误处理和重试机制
- 大型 MR 的性能考虑
- 权限验证
- 网络超时处理

## 更新日志

- 2025-02-26: 初始版本，内容待补充
