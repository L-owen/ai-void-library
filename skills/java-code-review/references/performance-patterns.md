# Performance Patterns and Anti-Patterns

本文档提供 Java 代码性能审查的检查清单和优化模式。

## 1. PSI Tree 遍历优化（IntelliJ 平台特有）

### ⚠️ 性能问题

```java
// ❌ 低效：多次遍历 PSI 树
public void analyze(PsiFile file) {
    PsiMethod[] methods = ((PsiJavaFile) file).getClasses()[0].getMethods();
    for (PsiMethod method : methods) {
        // 每次都重新查找
        PsiClass containingClass = PsiTreeUtil.getParentOfType(method, PsiClass.class);
    }
}

// ✅ 高效：缓存结果
public void analyze(PsiFile file) {
    PsiClass[] classes = ((PsiJavaFile) file).getClasses();
    for (PsiClass clazz : classes) {
        PsiMethod[] methods = clazz.getMethods();
        PsiClass containingClass = clazz; // 已知
        for (PsiMethod method : methods) {
            // 直接使用
        }
    }
}
```

### 检查要点
- 避免在循环中重复调用 `PsiTreeUtil.getParentOfType()`
- 使用 `PsiTreeUtil.findChildrenOfType()` 批量查找
- 考虑使用 `CachedValue` 缓存计算结果

## 2. 集合操作优化

### ⚠️ 常见问题

```java
// ❌ 低效：List 在循环中的 contains 操作
List<String> items = ...;
for (String key : keys) {
    if (items.contains(key)) { // O(n) 复杂度
        // ...
    }
}

// ✅ 高效：使用 HashSet
Set<String> itemSet = new HashSet<>(items);
for (String key : keys) {
    if (itemSet.contains(key)) { // O(1) 复杂度
        // ...
    }
}

// ❌ 低效：String 拼接在循环中
String result = "";
for (String item : items) {
    result += item; // 每次创建新的 String 对象
}

// ✅ 高效：使用 StringBuilder
StringBuilder sb = new StringBuilder();
for (String item : items) {
    sb.append(item);
}
String result = sb.toString();
```

### 检查要点
- 大数据集查找是否使用合适的集合类型（HashSet vs ArrayList）
- List.contains() 是否在大集合上使用
- String 拼接是否在循环中重复执行
- Map 的 key 对象是否正确实现了 hashCode() 和 equals()

## 3. 缓存策略

### ✅ 推荐模式

```java
// 使用 Caffeine 或 Guava Cache
private final Cache<String, ExpensiveResult> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(Duration.ofMinutes(10))
    .build();

public ExpensiveResult compute(String key) {
    return cache.get(key, k -> expensiveComputation(k));
}

// IntelliJ 平台使用 CachedValue
private final CachedValue<MyData> cachedData = CachedValuesManager.getManager(project).createCachedValue(
    () -> CachedValueProvider.Result.create(
        computeExpensiveData(),
        ModificationTracker.EVER_CHANGED
    ),
    false
);
```

### 检查要点
- 重复计算的昂贵操作是否使用缓存
- 缓存是否有过期策略
- 缓存大小是否有限制，防止内存溢出
- 缓存失效策略是否正确

## 4. N+1 查询问题

### ⚠️ 性能问题

```java
// ❌ 低效：N+1 查询
List<User> users = userRepository.findAll();
for (User user : users) {
    List<Order> orders = orderRepository.findByUserId(user.getId()); // N 次查询
    user.setOrders(orders);
}

// ✅ 高效：使用 JOIN FETCH
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();

// 或使用 EntityGraph
EntityGraph graph = entityManager.createEntityGraph(User.class);
graph.addAttributeNodes("orders");
List<User> users = entityManager.createQuery("SELECT u FROM User u", User.class)
    .setHint("javax.persistence.fetchgraph", graph)
    .getResultList();
```

### 检查要点
- 循环中是否执行数据库查询
- 是否使用批量查询或 JOIN FETCH
- 是否使用 @EntityGraph 或 DTO projection 优化查询

## 5. 资源释放

### ⚠️ 内存泄漏风险

```java
// ❌ 危险：未关闭资源
FileInputStream fis = new FileInputStream(file);
// 使用流但忘记关闭

// ✅ 安全：使用 try-with-resources
try (FileInputStream fis = new FileInputStream(file);
     BufferedInputStream bis = new BufferedInputStream(fis)) {
    // 使用流
} // 自动关闭

// ❌ 危险：监听器未注销
eventBus.register(this);
// 忘记注销

// ✅ 安全：使用 Disposable 管理
public class MyComponent implements Disposable {
    private final MessageBusConnection connection;

    public MyComponent(Project project) {
        connection = project.getMessageBus().connect(this);
        connection.subscribe(MyTopic.TOPIC, this::handleEvent);
    }

    @Override
    public void dispose() {
        connection.disconnect(); // 确保注销
    }
}
```

### 检查要点
- 所有 Closeable 资源是否使用 try-with-resources
- 监听器、回调是否在适当时机注销
- ThreadLocal 是否正确清理
- WeakReference 使用是否正确

## 6. 并发性能

### ⚠️ 常见问题

```java
// ❌ 低效：过度同步
public class Counter {
    private synchronized void increment() {
        count++;
    }
}

// ✅ 高效：使用原子类
private final AtomicLong count = new AtomicLong();

public void increment() {
    count.incrementAndGet();
}

// ❌ 低效：锁竞争严重
private final Object lock = new Object();
private final List<String> list = new ArrayList<>();

public void add(String item) {
    synchronized (lock) {
        list.add(item);
    }
}

// ✅ 高效：使用并发集合
private final ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();

public void add(String item) {
    queue.offer(item);
}
```

### 检查要点
- 是否存在不必要的同步
- 是否可以使用并发集合（ConcurrentHashMap、ConcurrentLinkedQueue）
- 是否可以使用原子类（AtomicLong、AtomicReference）
- 锁的粒度是否合理

## 7. I/O 性能

### ✅ 推荐做法

```java
// ✅ 使用缓冲 I/O
try (BufferedReader br = new BufferedReader(new FileReader(file));
     BufferedWriter bw = new BufferedWriter(new FileWriter(output))) {
    String line;
    while ((line = br.readLine()) != null) {
        bw.write(line);
        bw.newLine();
    }
}

// ✅ 批量操作
PreparedStatement pstmt = connection.prepareStatement(sql);
for (Item item : items) {
    pstmt.setString(1, item.getName());
    pstmt.addBatch();
    if (batchSize++ % 1000 == 0) {
        pstmt.executeBatch(); // 每 1000 条执行一次
    }
}
pstmt.executeBatch();
```

### 检查要点
- 是否使用缓冲 I/O（BufferedReader、BufferedInputStream）
- 数据库操作是否使用批量操作
- 大文件处理是否使用流式处理
- 网络请求是否使用连接池

## 8. 内存优化

### ⚠️ 常见问题

```java
// ❌ 内存浪费：创建不必要的对象
String s = new String("hello"); // 创建了两个 String 对象

// ✅ 推荐
String s = "hello"; // 使用字符串常量

// ❌ 内存浪费：自动装箱
Long sum = 0L; // 使用 Long 对象
for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i; // 每次都创建新的 Long 对象
}

// ✅ 推荐
long sum = 0L; // 使用原始类型
for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
}
```

### 检查要点
- 是否优先使用原始类型而非包装类
- 是否避免创建不必要的中间对象
- Stream 操作是否合理使用（避免过度链式调用）
- 大对象是否及时释放引用

## 9. 懒加载模式

### ✅ 推荐做法

```java
// ✅ 懒加载初始化
private volatile ExpensiveObject instance;

public ExpensiveObject getInstance() {
    ExpensiveObject result = instance;
    if (result == null) {
        synchronized (this) {
            result = instance;
            if (result == null) {
                instance = result = createExpensiveObject();
            }
        }
    }
    return result;
}

// 或使用 Supplier
private final Supplier<ExpensiveObject> instance = Suppliers.memoize(
    () -> new ExpensiveObject()
);

public ExpensiveObject getInstance() {
    return instance.get();
}
```

### 检查要点
- 昂贵对象的初始化是否延迟到首次使用
- 懒加载是否线程安全

## 10. 数据库索引和查询

### ⚠️ 性能问题

```sql
-- ❌ 低效：全表扫描
SELECT * FROM users WHERE LOWER(name) = 'john';

-- ✅ 推荐：使用函数索引或应用层转换
SELECT * FROM users WHERE name = 'John';

-- ❌ 低效：SELECT *
SELECT * FROM users WHERE id = ?;

-- ✅ 推荐：只查询需要的字段
SELECT id, name, email FROM users WHERE id = ?;
```

### 检查要点
- WHERE 子句中的字段是否有索引
- 是否避免 SELECT *，只查询需要的字段
- 是否使用 EXPLAIN 分析查询计划
- 是否有适当的复合索引

## 11. 异步处理

### ✅ 推荐做法

```java
// ✅ 使用 CompletableFuture 进行异步处理
public CompletableFuture<Result> processDataAsync(Input input) {
    return CompletableFuture.supplyAsync(() -> {
        return heavyComputation(input);
    }, executor);
}

// 或使用响应式编程
public Mono<Result> processData(Input input) {
    return Mono.fromCallable(() -> heavyComputation(input))
        .subscribeOn(Schedulers.boundedElastic());
}
```

### 检查要点
- 耗时操作是否可以使用异步处理
- 是否正确使用线程池（避免创建过多线程）
- 异步操作是否正确处理异常
