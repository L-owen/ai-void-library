# Stability Checks

本文档提供 Java 代码稳定性审查的检查清单。

## 1. 异常处理策略

### ⚠️ 常见问题

```java
// ❌ 吞掉异常
try {
    doSomething();
} catch (Exception e) {
    // 什么也不做
}

// ❌ 捕获过于宽泛
try {
    doSomething();
} catch (Exception e) {
    log.error("Error", e);
}

// ❌ 捕获后立即抛出新异常，丢失原始异常
try {
    doSomething();
} catch (IOException e) {
    throw new MyException("Failed");
}

// ✅ 正确：保留原始异常链
try {
    doSomething();
} catch (IOException e) {
    throw new MyException("Failed", e); // 保留 cause
}

// ✅ 正确：捕获具体异常
try {
    doSomething();
} catch (IOException e) {
    log.error("Failed to read file: {}", file.getPath(), e);
    throw new BusinessException("File processing failed", e);
}

// ✅ 正确：提供有意义的错误消息
try {
    processPayment(order);
} catch (PaymentDeclinedException e) {
    log.warn("Payment declined for order {}: {}", order.getId(), e.getReason());
    throw new UserVisibleException("Payment declined: " + e.getReason(), e);
}
```

### 检查要点
- 是否捕获具体异常而非 Exception 或 Throwable
- 是否记录原始异常信息（日志、cause chain）
- 是否提供有意义的错误消息
- 是否正确区分可恢复和不可恢复异常
- 是否避免使用异常进行正常流程控制

## 2. 资源泄露检查

### ⚠️ 高危问题

```java
// ❌ 流未关闭
public void readFile(String path) throws IOException {
    FileInputStream fis = new FileInputStream(path);
    // 使用流但忘记关闭
    // 如果发生异常，流不会被关闭
}

// ✅ 使用 try-with-resources
public void readFile(String path) throws IOException {
    try (FileInputStream fis = new FileInputStream(path);
         BufferedInputStream bis = new BufferedInputStream(fis)) {
        // 使用流
    } // 自动关闭
}

// ❌ 数据库连接未关闭
public List<User> getUsers() throws SQLException {
    Connection conn = dataSource.getConnection();
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery("SELECT * FROM users");
    // 忘记关闭连接
}

// ✅ 正确关闭数据库资源
public List<User> getUsers() throws SQLException {
    try (Connection conn = dataSource.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery("SELECT * FROM users")) {
        // 处理结果
    }
}

// ❌ 监听器未注销
public class MyComponent {
    public void init() {
        eventBus.register(this);
    }
    // 忘记注销监听器
}

// ✅ 使用 Disposable 管理生命周期
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

// ❌ 定时任务未取消
public class Scheduler {
    public void start() {
        executor.scheduleAtFixedRate(task, 0, 1, TimeUnit.SECONDS);
    }
    // 忘记取消任务
}

// ✅ 保存并取消任务
public class Scheduler implements Disposable {
    private ScheduledFuture<?> future;

    public void start() {
        future = executor.scheduleAtFixedRate(task, 0, 1, TimeUnit.SECONDS);
    }

    @Override
    public void dispose() {
        if (future != null) {
            future.cancel(false);
        }
    }
}
```

### 检查要点
- 所有 Closeable 资源是否使用 try-with-resources
- 数据库连接、Statement、ResultSet 是否正确关闭
- 监听器、回调是否在适当时机注销
- 定时任务、线程池是否在适当时机关闭
- 文件句柄、Socket 连接是否正确关闭

## 3. 空指针和边界条件

### ⚠️ 常见问题

```java
// ❌ 不检查 null
public void processUser(User user) {
    String email = user.getEmail(); // 可能 NPE
    sendEmail(email);
}

// ✅ 使用 Optional 或显式检查
public void processUser(User user) {
    if (user == null) {
        throw new IllegalArgumentException("User cannot be null");
    }
    String email = user.getEmail();
    if (email == null) {
        log.warn("User {} has no email", user.getId());
        return;
    }
    sendEmail(email);
}

// ✅ 使用 Optional
public void processUser(User user) {
    Optional.ofNullable(user)
        .map(User::getEmail)
        .ifPresent(this::sendEmail);
}

// ❌ 不检查集合大小
public Item getFirstItem(List<Item> items) {
    return items.get(0); // 可能 IndexOutOfBoundsException
}

// ✅ 检查集合大小
public Optional<Item> getFirstItem(List<Item> items) {
    if (items == null || items.isEmpty()) {
        return Optional.empty();
    }
    return Optional.of(items.get(0));
}

// ❌ 字符串操作不检查
public String getDomain(String email) {
    return email.substring(email.indexOf("@") + 1); // 可能 -1
}

// ✅ 正确验证
public Optional<String> getDomain(String email) {
    if (email == null || !email.contains("@")) {
        return Optional.empty();
    }
    return Optional.of(email.substring(email.indexOf("@") + 1));
}
```

### 检查要点
- 方法参数是否进行 null 检查
- 集合操作前是否检查大小
- 字符串操作是否验证输入（indexOf、substring）
- 数组访问是否检查边界
- 是否使用 @Nullable 和 @NonNull 注解

## 4. 并发问题

### ⚠️ 高危问题

```java
// ❌ 竞态条件
public class Counter {
    private int count = 0;

    public void increment() {
        count++; // 非原子操作
    }
}

// ✅ 使用原子类或同步
public class Counter {
    private final AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();
    }
}

// ❌ 死锁风险
public void transfer(Account from, Account to, double amount) {
    synchronized (from) {
        synchronized (to) {
            from.withdraw(amount);
            to.deposit(amount);
        }
    }
}

// ✅ 使用锁顺序避免死锁
public void transfer(Account from, Account to, double amount) {
    Account first = from.getId() < to.getId() ? from : to;
    Account second = from.getId() < to.getId() ? to : from;

    synchronized (first) {
        synchronized (second) {
            from.withdraw(amount);
            to.deposit(amount);
        }
    }
}

// ❌ 双重检查锁定错误实现
public class Singleton {
    private static Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) { // 第一次检查
            synchronized (Singleton.class) {
                if (instance == null) { // 第二次检查
                    instance = new Singleton(); // 问题：指令重排
                }
            }
        }
        return instance;
    }
}

// ✅ 正确的双重检查锁定
public class Singleton {
    private static volatile Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// 或使用枚举单例
public enum Singleton {
    INSTANCE;
}
```

### 检查要点
- 共享可变状态是否正确同步
- 是否存在竞态条件
- 锁的获取顺序是否一致（避免死锁）
- volatile 关键字是否正确使用
- 是否避免在锁内执行耗时操作

## 5. 日志记录和错误追踪

### ✅ 推荐做法

```java
// ✅ 使用适当的日志级别
logger.debug("Processing request: {}", request);  // 调试信息
logger.info("User {} logged in", user.getId());   // 重要业务事件
logger.warn("Retry attempt {} for {}", retryCount, operation); // 警告
logger.error("Failed to process payment", exception); // 错误

// ✅ 记录关键业务操作
logger.info("Order created: id={}, customer={}, total={}",
    order.getId(), order.getCustomerId(), order.getTotal());

// ✅ 记录异常时包含堆栈和上下文
try {
    processPayment(order);
} catch (PaymentException e) {
    logger.error("Payment failed for order {}: amount={}, gateway={}",
        order.getId(), order.getTotal(), paymentGateway, e);
    throw new OrderProcessingException("Payment failed", e);
}

// ✅ 性能日志
long startTime = System.nanoTime();
try {
    processLargeDataset(data);
} finally {
    long duration = System.nanoTime() - startTime;
    logger.info("Processed {} records in {} ms",
        data.size(), TimeUnit.NANOSECONDS.toMillis(duration));
}
```

### 检查要点
- 关键业务操作是否有日志记录
- 异常是否记录完整的堆栈跟踪和上下文信息
- 是否避免在循环中频繁记录日志
- 敏感信息是否从日志中排除（密码、token）
- 是否使用结构化日志（JSON 格式）

## 6. 容错和降级机制

### ✅ 推荐做法

```java
// ✅ 重试机制
@Retryable(
    value = {TransientException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2)
)
public void processWithRetry(Input input) {
    // 可能失败的操作
}

// ✅ 熔断器模式
@CircuitBreaker(
    name = "paymentService",
    fallbackMethod = "processPaymentFallback"
)
public PaymentResult processPayment(PaymentRequest request) {
    return paymentGateway.charge(request);
}

private PaymentResult processPaymentFallback(PaymentRequest request, Exception e) {
    log.warn("Payment service unavailable, using fallback");
    return PaymentResult.failed("Service temporarily unavailable");
}

// ✅ 超时控制
@TimeOut(value = 5, unit = TimeUnit.SECONDS)
public CompletableFuture<Result> processAsync(Input input) {
    return CompletableFuture.supplyAsync(() -> {
        return heavyComputation(input);
    });
}

// ✅ 优雅降级
public class RecommendationService {
    private final MLModelService mlModelService;
    private final SimpleRecommendationService simpleService;

    public List<Product> getRecommendations(User user) {
        try {
            return mlModelService.getPersonalizedProducts(user)
                .get(2, TimeUnit.SECONDS);
        } catch (Exception e) {
            log.warn("ML service failed, falling back to simple recommendations", e);
            return simpleService.getPopularProducts();
        }
    }
}
```

### 检查要点
- 外部服务调用是否有超时控制
- 是否实现重试机制（针对可恢复错误）
- 是否有熔断器防止级联失败
- 是否有降级方案处理服务不可用

## 7. 配置和参数验证

### ✅ 推荐做法

```java
// ✅ 使用 Bean Validation
public class CreateUserRequest {
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    private String username;

    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;

    @Pattern(regexp = "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d@$!%*#?&]{8,}$",
             message = "Password must be at least 8 characters with letters and numbers")
    private String password;
}

// ✅ 方法参数验证
public void transferMoney(@NotNull Account from, @NotNull Account to, @Positive double amount) {
    if (from.equals(to)) {
        throw new IllegalArgumentException("Source and destination accounts must be different");
    }
    if (amount > from.getBalance()) {
        throw new InsufficientFundsException(from.getBalance(), amount);
    }
    // 执行转账
}

// ✅ 配置验证
@ConfigurationProperties(prefix = "app.payment")
@Validated
public class PaymentProperties {
    @NotBlank
    private String gatewayUrl;

    @Min(1000)
    @Max(30000)
    private int timeoutMs = 5000;

    @Size(min = 1, max = 10)
    private List<String> supportedCurrencies;
}
```

### 检查要点
- 公共 API 参数是否进行验证
- 是否使用 Bean Validation 注解
- 配置属性是否有默认值和验证
- 错误消息是否清晰有用
