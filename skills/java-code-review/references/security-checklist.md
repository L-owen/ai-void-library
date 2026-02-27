# Security Checklist

本文档提供 Java 代码安全审查的检查清单。

## 1. SQL Injection (SQL 注入)

### ⚠️ 高危问题

```java
// ❌ 危险：SQL 注入漏洞
String query = "SELECT * FROM users WHERE username = '" + username + "'";
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);

// ✅ 安全：使用 PreparedStatement
String query = "SELECT * FROM users WHERE username = ?";
PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, username);
ResultSet rs = pstmt.executeQuery(query);
```

### 检查要点
- 所有 SQL 查询是否使用 PreparedStatement 或参数化查询
- 是否有字符串拼接构造 SQL 的情况
- 动态 SQL 是否通过白名单验证

## 2. XSS (Cross-Site Scripting)

### ⚠️ 高危问题

```java
// ❌ 危险：直接输出用户输入
out.println("<div>" + userInput + "</div>");

// ✅ 安全：HTML 转义
out.println("<div>" + StringEscapeUtils.escapeHtml4(userInput) + "</div>");

// ✅ 或使用框架提供的转义
model.addAttribute("content", HtmlUtils.htmlEscape(userInput));
```

### 检查要点
- 所有输出到 HTML 的用户输入是否经过转义
- 是否使用模板引擎的自动转义功能
- JSON 响应中的用户数据是否正确处理

## 3. CSRF (Cross-Site Request Forgery)

### ✅ 防护措施

```java
// Spring Security 示例
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public CsrfTokenRepository csrfTokenRepository() {
        HttpSessionCsrfTokenRepository repository = new HttpSessionCsrfTokenRepository();
        repository.setHeaderName("X-CSRF-TOKEN");
        return repository;
    }
}

// 在表单中包含 CSRF token
<form>
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
    <!-- 其他表单字段 -->
</form>
```

### 检查要点
- 状态改变的操作是否有 CSRF 保护
- 是否使用框架提供的 CSRF 防护
- Token 是否正确生成和验证

## 4. 认证和授权

### ⚠️ 常见问题

```java
// ❌ 不安全：硬编码的凭据
private static final String PASSWORD = "admin123";

// ❌ 不安全：弱密码策略
if (password.length() < 6) {
    throw new Exception("Password too short");
}

// ✅ 安全：使用 BCrypt 加密
PasswordEncoder encoder = new BCryptPasswordEncoder();
String encodedPassword = encoder.encode(rawPassword);
```

### 检查要点
- 密码是否使用强哈希算法（BCrypt、Argon2）
- 是否有会话超时机制
- 敏感操作是否有二次认证
- 权限检查是否在每个端点都执行

## 5. 敏感数据处理

### ⚠️ 高危问题

```java
// ❌ 危险：日志中输出敏感信息
logger.info("User logged in: " + user.getPassword());

// ❌ 危险：异常中泄露敏感信息
throw new RuntimeException("Invalid user: " + user.getEmail());

// ✅ 安全：脱敏处理
logger.info("User logged in: " + user.getUsername());

// ✅ 安全：不泄露内部细节
throw new UnauthorizedException("Authentication failed");
```

### 检查要点
- 日志中是否包含密码、token、信用卡号等敏感信息
- 异常消息是否泄露内部实现细节
- 敏感配置是否使用环境变量或密钥管理系统
- 临时文件中的敏感数据是否安全清理

## 6. 加密和数据保护

### ✅ 推荐做法

```java
// 使用 Java Cryptography Architecture
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
SecretKey key = new SecretKeySpec(keyBytes, "AES");
GCMParameterSpec gcmSpec = new GCMParameterSpec(GCM_TAG_LENGTH, iv);
cipher.init(Cipher.ENCRYPT_MODE, key, gcmSpec);
byte[] encrypted = cipher.doFinal(plaintext);
```

### 检查要点
- 是否使用强加密算法（AES-256、RSA-4096）
- 加密模式是否安全（GCM、CBC 带 HMAC）
- 密钥管理是否安全（不硬编码、使用密钥管理系统）
- HTTPS 是否正确配置（禁用弱协议、正确证书验证）

## 7. 反序列化安全

### ⚠️ 高危问题

```java
// ❌ 危险：反序列化不受信任的数据
ObjectInputStream ois = new ObjectInputStream(inputStream);
Object obj = ois.readObject();

// ✅ 安全：使用安全的反序列化
// 使用 JSON 或其他安全格式
ObjectMapper mapper = new ObjectMapper();
MyClass obj = mapper.readValue(jsonInput, MyClass.class);

// 或使用白名单过滤
validFilter.setFilter(new SimpleFilterProvider()
    .addFilter("myFilter", SimpleBeanFilter.serializeAllExcept("sensitiveField")));
```

### 检查要点
- 是否反序列化不受信任的输入
- 是否使用白名单限制可反序列化的类
- 是否可以使用更安全的格式（JSON、XML）

## 8. 依赖安全

### 检查要点
- 是否使用已知漏洞的依赖
- 是否定期更新依赖到最新安全版本
- 是否使用依赖检查工具（OWASP Dependency-Check、Snyk）

### 推荐工具
```bash
# Maven 依赖检查
mvn org.owasp:dependency-check-maven:check

# Gradle 依赖检查
gradle dependencyCheck
```

## 9. 文件上传安全

### ⚠️ 常见问题

```java
// ❌ 不安全：不验证文件类型
File file = new File(uploadDir + filename);
multipartFile.transferTo(file);

// ✅ 安全：验证文件类型和内容
String contentType = multipartFile.getContentType();
if (!ALLOWED_TYPES.contains(contentType)) {
    throw new BadRequestException("Invalid file type");
}

// 验证文件内容（魔数）
byte[] header = new byte[8];
inputStream.read(header);
if (!isValidFileType(header)) {
    throw new BadRequestException("Invalid file content");
}

// 使用安全的文件名
String safeFilename = Paths.get(filename).getFileName().toString();
Path target = uploadDir.resolve(safeFilename).normalize();
if (!target.startsWith(uploadDir)) {
    throw new BadRequestException("Invalid filename");
}
```

### 检查要点
- 是否验证文件类型（扩展名 + 内容）
- 是否限制文件大小
- 是否使用安全的文件名（防止路径遍历）
- 上传的文件是否存储在 webroot 外

## 10. URL 重定向

### ⚠️ 常见问题

```java
// ❌ 不安全：开放重定向
response.sendRedirect(request.getParameter("next"));

// ✅ 安全：验证重定向 URL
String nextUrl = request.getParameter("next");
if (isValidRedirectUrl(nextUrl)) {
    response.sendRedirect(nextUrl);
}

private boolean isValidRedirectUrl(String url) {
    try {
        URI uri = new URI(url);
        return ALLOWED_HOSTS.contains(uri.getHost());
    } catch (URISyntaxException e) {
        return false;
    }
}
```

### 检查要点
- 重定向 URL 是否经过验证
- 是否限制为允许的主机列表
- 是否使用相对 URL

## 11. LDAP 注入

### ⚠️ 高危问题

```java
// ❌ 危险：LDAP 注入
String filter = "(uid=" + username + ")";
DirContext ctx = new InitialDirContext(env);
NamingEnumeration<SearchResult> results = ctx.search(base, filter, controls);

// ✅ 安全：使用参数化查询
String filter = "(uid={0})";
String[] args = {username};
DirContext ctx = new InitialDirContext(env);
NamingEnumeration<SearchResult> results = ctx.search(base, filter, args, controls);
```

### 检查要点
- LDAP 查询是否使用参数化
- 是否验证和清理输入

## 12. 命令注入

### ⚠️ 高危问题

```java
// ❌ 危险：命令注入
String cmd = "ls " + directory;
Runtime.getRuntime().exec(cmd);

// ✅ 安全：使用 ProcessBuilder 和参数数组
ProcessBuilder pb = new ProcessBuilder("ls", directory);
Process process = pb.start();
```

### 检查要点
- 是否使用 `Runtime.exec()` 或 `ProcessBuilder` 执行命令
- 命令参数是否使用数组而非字符串拼接
- 是否对输入进行白名单验证
