# IntelliJ Platform Development Best Practices

本文档提供 IntelliJ 平台插件开发的最佳实践指南，用于代码审查时的参考。

## 1. PSI (Program Structure Interface) 使用

### ✅ 推荐做法

```java
// 使用 PsiTreeUtil 查找特定类型的元素
PsiMethod method = PsiTreeUtil.getParentOfType(element, PsiMethod.class);

// 使用 findChildOfClass 查找子元素
PsiExpression expr = psiClass.findChildOfClass(PsiExpression.class);

// 使用类型安全的访问
if (psiElement instanceof PsiMethod) {
    PsiMethod method = (PsiMethod) psiElement;
    // 处理方法
}
```

### ⚠️ 常见问题

- **问题**: 直接遍历 PSI 树而不使用 `PsiTreeUtil`
- **影响**: 性能差，代码易出错
- **建议**: 使用 `PsiTreeUtil` 提供的工具方法

```java
// ❌ 不推荐
for (PsiElement child = element.getFirstChild(); child != null; child = child.getNextSibling()) {
    if (child instanceof PsiMethod) {
        // ...
    }
}

// ✅ 推荐
Collection<PsiMethod> methods = PsiTreeUtil.findChildrenOfType(element, PsiMethod.class);
```

## 2. Read Action vs Write Action

### ✅ 正确使用

```java
// 读操作使用 ReadAction
ApplicationManager.getApplication().runReadAction(() -> {
    PsiFile file = PsiManager.getInstance(project).findFile(virtualFile);
    // 只读操作
});

// 写操作使用 WriteAction
ApplicationManager.getApplication().runWriteAction(() -> {
    psiElement.delete();
    // 修改 PSI 树的操作
});
```

### ⚠️ 常见问题

- **问题**: 在 Write Action 中执行耗时操作
- **影响**: UI 冻结，用户体验差
- **建议**: 在 Write Action 外部准备数据，只在 Write Action 中执行必要的修改

```java
// ❌ 不推荐
ApplicationManager.getApplication().runWriteAction(() -> {
    // 耗时的计算
    List<PsiElement> toDelete = findElementsToDelete(project);
    for (PsiElement e : toDelete) {
        e.delete();
    }
});

// ✅ 推荐
List<PsiElement> toDelete = ApplicationManager.getApplication().runReadAction(
    () -> findElementsToDelete(project)
);
ApplicationManager.getApplication().runWriteAction(() -> {
    for (PsiElement e : toDelete) {
        e.delete();
    }
});
```

## 3. EDT (Event Dispatch Thread) 使用

### ✅ 正确使用

```java
// 耗时操作在后台线程
Task.Backgroundable task = new Task.Backgroundable(project, "Title") {
    @Override
    public void run(@NotNull ProgressIndicator indicator) {
        // 后台线程执行耗时操作
        Result result = computeHeavyResult();
    }

    @Override
    public void onSuccess() {
        // EDT 线程更新 UI
        updateUI(result);
    }
};
task.queue();
```

### ⚠️ 常见问题

- **问题**: 在 EDT 线程执行耗时操作
- **影响**: UI 卡顿
- **建议**: 使用 `Task.Backgroundable` 或 `ApplicationManager.getApplication().executeOnPooledThread()`

## 4. ApplicationManager 和 Project 使用

### ✅ 推荐做法

```java
// 获取服务实例
MyProjectService service = project.getService(MyProjectService.class);
MyApplicationService appService = ApplicationManager.getApplication().getService(MyApplicationService.class);

// 检查项目是否有效
if (!project.isDisposed()) {
    // 使用 project
}
```

### ⚠️ 常见问题

- **问题**: 在 project 已经 dispose 后继续使用
- **影响**: NullPointerException 或其他异常
- **建议**: 使用前检查 `project.isDisposed()`

## 5. Extension Point 注册

### ✅ 正确注册

```xml
<!-- plugin.xml -->
<extensions defaultExtensionNs="com.intellij">
    <applicationService serviceImplementation="com.example.MyApplicationService"/>
    <projectService serviceImplementation="com.example.MyProjectService"/>
    <applicationConfigurable instance="com.example.MyConfigurable"/>
</extensions>
```

### ⚠️ 常见问题

- **问题**: 忘记在 plugin.xml 中注册服务或扩展点
- **影响**: 服务注入失败，功能无法使用
- **建议**: 确保所有服务和扩展点都在 plugin.xml 中正确声明

## 6. AnAction 实现

### ✅ 推荐做法

```java
public class MyAction extends AnAction {
    @Override
    public void actionPerformed(@NotNull AnActionEvent e) {
        Project project = e.getProject();
        if (project == null) return;

        // 检查可用性
        if (!isEnabled(e)) {
            return;
        }

        // 执行操作
        doAction(project);
    }

    @Override
    public void update(@NotNull AnActionEvent e) {
        // 根据上下文启用/禁用 Action
        boolean enabled = e.getProject() != null
            && e.getData(LangDataKeys.PSI_FILE) != null;
        e.getPresentation().setEnabledAndVisible(enabled);
    }
}
```

### ⚠️ 常见问题

- **问题**: 不实现 `update()` 方法，导致 Action 始终可用或不可用
- **影响**: 用户体验差，可能在无效上下文中执行操作
- **建议**: 总是实现 `update()` 方法来控制 Action 的可用性

## 7. 资源管理和生命周期

### ✅ 推荐做法

```java
// 使用 Disposable 管理资源
public class MyComponent implements Disposable {
    private final MessageBusConnection connection;

    public MyComponent(Project project) {
        connection = project.getMessageBus().connect(this);
        connection.subscribe(MyTopic.TOPIC, this::handleEvent);
    }

    @Override
    public void dispose() {
        connection.disconnect();
    }
}

// 检查 dispose 状态
public void doSomething() {
    if (isDisposed()) {
        return;
    }
    // 执行操作
}
```

### ⚠️ 常见问题

- **问题**: 忘记断开 MessageBus 连接
- **影响**: 内存泄漏
- **建议**: 使用 `Disposable` 管理 MessageBusConnection

## 8. 性能优化要点

1. **缓存 PSI 查询结果**: 避免重复遍历 PSI 树
2. **使用 `PsiTreeUtil`**: 而不是手动遍历
3. **批量操作**: 将多个 Write Action 合并为一个
4. **使用 `RecursionManager`**: 防止 PSI 树递归遍历栈溢出
5. **避免在 `equals()` 和 `hashCode()` 中使用 PSI 对象**: 可能导致性能问题

## 9. 线程安全

- **不要在后台线程持有 PSI 元素的引用**: PSI 可能在任何时候失效
- **使用 `SmartPsiElementPointer`**: 如果需要跨线程持有 PSI 引用
- **避免在 Write Action 中启动新线程**: 可能导致死锁
