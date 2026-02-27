# Architecture Design Principles

本文档提供 Java 代码架构审查的设计原则和最佳实践。

## 1. SOLID 原则

### Single Responsibility Principle (单一职责原则)

```java
// ❌ 违反：类承担多个职责
public class UserService {
    public void registerUser(User user) { }
    public void sendEmail(User user) { }
    public void logAudit(User user) { }
}

// ✅ 推荐：每个类单一职责
public class UserService {
    private final EmailService emailService;
    private final AuditService auditService;

    public void registerUser(User user) {
        // 只处理用户注册逻辑
        emailService.sendWelcomeEmail(user);
        auditService.logUserRegistration(user);
    }
}
```

### Open/Closed Principle (开闭原则)

```java
// ❌ 违反：每次添加新支付方式都要修改类
public class PaymentProcessor {
    public void process(String type, double amount) {
        if (type.equals("credit")) {
            // ...
        } else if (type.equals("paypal")) {
            // ...
        }
    }
}

// ✅ 推荐：对扩展开放，对修改关闭
public interface PaymentStrategy {
    void process(double amount);
}

public class CreditCardPayment implements PaymentStrategy { }
public class PayPalPayment implements PaymentStrategy { }

public class PaymentProcessor {
    private final Map<String, PaymentStrategy> strategies;

    public PaymentProcessor(List<PaymentStrategy> strategyList) {
        this.strategies = strategyList.stream()
            .collect(Collectors.toMap(
                s -> s.getClass().getSimpleName(),
                Function.identity()
            ));
    }

    public void process(String type, double amount) {
        strategies.get(type).process(amount);
    }
}
```

### Liskov Substitution Principle (里氏替换原则)

```java
// ❌ 违反：子类改变了父类的行为契约
public class Rectangle {
    public void setWidth(double width) { }
    public void setHeight(double height) { }
    public double getArea() { return width * height; }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(double width) {
        super.setWidth(width);
        super.setHeight(width); // 违反了 Rectangle 的契约
    }
}

// ✅ 推荐：不滥用继承，使用组合
public interface Shape {
    double getArea();
}

public class Rectangle implements Shape { }
public class Square implements Shape { }
```

### Interface Segregation Principle (接口隔离原则)

```java
// ❌ 违反：臃肿的接口
public interface UserService {
    void login(String username, String password);
    void logout();
    void deleteUser();
    void resetPassword();
    void updateUserProfile();
    void exportUserData();
}

// ✅ 推荐：分离接口
public interface AuthService {
    void login(String username, String password);
    void logout();
}

public interface UserManagementService {
    void deleteUser();
    void resetPassword();
    void updateUserProfile();
}

public interface UserDataExportService {
    void exportUserData();
}
```

### Dependency Inversion Principle (依赖倒置原则)

```java
// ❌ 违反：依赖具体实现
public class OrderService {
    private MySQLDatabase database = new MySQLDatabase();

    public void saveOrder(Order order) {
        database.save(order);
    }
}

// ✅ 推荐：依赖抽象
public interface Database {
    void save(Order order);
}

public class OrderService {
    private final Database database;

    public OrderService(Database database) {
        this.database = database;
    }
}
```

## 2. 设计模式识别

### 单例模式（Singleton）

```java
// ✅ 推荐：线程安全的单例
public enum DatabaseConnection {
    INSTANCE;

    private Connection connection;

    public Connection getConnection() {
        if (connection == null) {
            connection = createConnection();
        }
        return connection;
    }
}
```

### 工厂模式（Factory）

```java
// ✅ 推荐：使用工厂创建对象
public interface PaymentFactory {
    PaymentStrategy createPayment(PaymentType type);
}

public class PaymentFactoryImpl implements PaymentFactory {
    @Override
    public PaymentStrategy createPayment(PaymentType type) {
        return switch (type) {
            case CREDIT_CARD -> new CreditCardPayment();
            case PAYPAL -> new PayPalPayment();
            case BANK_TRANSFER -> new BankTransferPayment();
        };
    }
}
```

### 策略模式（Strategy）

```java
// ✅ 推荐：使用策略模式消除条件判断
public interface DiscountStrategy {
    BigDecimal calculateDiscount(Order order);
}

public class VIPDiscountStrategy implements DiscountStrategy {
    @Override
    public BigDecimal calculateDiscount(Order order) {
        return order.getTotal().multiply(new BigDecimal("0.2"));
    }
}

public class RegularDiscountStrategy implements DiscountStrategy {
    @Override
    public BigDecimal calculateDiscount(Order order) {
        return order.getTotal().multiply(new BigDecimal("0.05"));
    }
}

public class DiscountCalculator {
    private final Map<CustomerType, DiscountStrategy> strategies;

    public BigDecimal calculate(Order order, CustomerType type) {
        return strategies.get(type).calculateDiscount(order);
    }
}
```

### 观察者模式（Observer）

```java
// ✅ 推荐：使用事件驱动架构
public interface EventListener<T> {
    void onEvent(T event);
}

public class EventBus {
    private final Map<Class<?>, List<EventListener>> listeners = new ConcurrentHashMap<>();

    public <T> void subscribe(Class<T> eventType, EventListener<T> listener) {
        listeners.computeIfAbsent(eventType, k -> new CopyOnWriteArrayList<>())
            .add(listener);
    }

    public <T> void publish(T event) {
        List<EventListener> eventListeners = listeners.get(event.getClass());
        if (eventListeners != null) {
            eventListeners.forEach(l -> l.onEvent(event));
        }
    }
}
```

### 装饰器模式（Decorator）

```java
// ✅ 推荐：使用装饰器动态添加功能
public interface DataSource {
    void write(String data);
    String read();
}

public class FileDataSource implements DataSource {
    @Override
    public void write(String data) { }
    @Override
    public String read() { return ""; }
}

public class CompressionDecorator implements DataSource {
    private final DataSource dataSource;

    public CompressionDecorator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public void write(String data) {
        String compressed = compress(data);
        dataSource.write(compressed);
    }

    @Override
    public String read() {
        String data = dataSource.read();
        return decompress(data);
    }
}
```

## 3. 模块化和解耦

### ✅ 推荐做法

```java
// ✅ 清晰的模块边界
// module-api
public interface UserService {
    UserDTO getUserById(Long id);
    UserDTO createUser(UserDTO user);
}

// module-implementation
@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;

    @Override
    public UserDTO getUserById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
        return userMapper.toDTO(user);
    }
}

// module-consumer (只依赖 API)
@RestController
public class UserController {
    private final UserService userService;
}
```

### 检查要点
- 模块之间是否通过接口而非实现类交互
- 是否避免循环依赖
- 是否使用 DTO 对象隔离领域模型
- 是否有清晰的分层架构（Controller → Service → Repository）

## 4. 接口设计

### ✅ 推荐做法

```java
// ✅ 清晰的接口命名和职责
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByActiveTrueAndLastLoginBefore(LocalDateTime date);
}

// ✅ 清晰的方法命名
public interface OrderService {
    Order createOrder(CreateOrderRequest request);  // 清晰的意图
    void cancelOrder(Long orderId, CancellationReason reason);
    Order getOrder(Long orderId);
    List<Order> getOrdersByCustomer(Long customerId);
}

// ❌ 不好的命名
public interface OrderManager {
    void doOrder(Order o);  // 不清晰
    Order order(Long id);   // 动词作为名词使用
}
```

### 检查要点
- 接口方法命名是否清晰表达意图
- 方法参数是否合理（避免过多参数）
- 返回类型是否明确（避免返回 null，考虑 Optional）
- 接口是否单一职责

## 5. 依赖注入和控制反转

### ✅ 推荐做法

```java
// ✅ 构造器注入（推荐）
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// ✅ 使用框架的依赖注入
@Component
public class OrderProcessor {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;

    @Autowired
    public OrderProcessor(
        PaymentService paymentService,
        InventoryService inventoryService,
        NotificationService notificationService
    ) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
    }
}
```

### 检查要点
- 是否使用依赖注入而非 new 创建对象
- 依赖是否通过接口而非具体类注入
- 是否避免循环依赖

## 6. 领域驱动设计（DDD）

### ✅ 推荐做法

```java
// ✅ 领域对象封装业务逻辑
@Entity
public class Order {
    private OrderStatus status;
    private List<OrderItem> items;

    public void addItem(Product product, int quantity) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot add items to non-draft order");
        }
        items.add(new OrderItem(product, quantity));
    }

    public void complete() {
        if (items.isEmpty()) {
            throw new IllegalStateException("Cannot complete empty order");
        }
        this.status = OrderStatus.COMPLETED;
    }

    public Money getTotal() {
        return items.stream()
            .map(OrderItem::getTotal)
            .reduce(Money.ZERO, Money::add);
    }
}

// ✅ 领域服务处理跨聚合根的业务逻辑
@Service
public class OrderDomainService {
    public void reserveInventory(Order order) {
        for (OrderItem item : order.getItems()) {
            inventoryService.reserve(item.getProduct(), item.getQuantity());
        }
    }
}
```

### 检查要点
- 领域对象是否封装业务逻辑而非贫血模型
- 是否正确使用聚合根管理一致性边界
- 是否使用值对象（Value Object）替代原始类型

## 7. 错误处理和异常设计

### ✅ 推荐做法

```java
// ✅ 自定义异常层次结构
public abstract class DomainException extends RuntimeException {
    private final ErrorCode errorCode;

    protected DomainException(ErrorCode errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
}

public class UserNotFoundException extends DomainException {
    public UserNotFoundException(Long userId) {
        super(ErrorCode.USER_NOT_FOUND, "User not found: " + userId);
    }
}

// ✅ 使用 Result 模式避免异常
public class Result<T, E> {
    private final T value;
    private final E error;

    public static <T, E> Result<T, E> success(T value) {
        return new Result<>(value, null);
    }

    public static <T, E> Result<T, E> failure(E error) {
        return new Result<>(null, error);
    }

    public boolean isSuccess() {
        return error == null;
    }
}
```

### 检查要点
- 异常层次结构是否清晰
- 是否避免使用异常进行流程控制
- 是否提供有意义的错误消息和错误码
