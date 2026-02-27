# Issue Card HTML 结构

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

## CSS 类说明

- `.issue-card` - 问题卡片容器，根据严重程度有不同样式
- `.critical/.high/.medium/.low` - 严重程度样式类
- `.issue-badge` - 徽章样式（严重等级和分类徽章）
- `.code-line-highlight` - 高亮的问题代码行
