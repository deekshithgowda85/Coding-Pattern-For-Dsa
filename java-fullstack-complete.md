# Complete Java Full Stack: Spring Boot, Microservices & AWS — Master Reference Guide

> Theory + Implementation for every topic. Each concept: **What → Why → When/Where → How**.

---

## Table of Contents
1. [Core Java Fundamentals](#1-core-java-fundamentals)
2. [Advanced Java: Streams, Lambdas & Optionals](#2-advanced-java)
3. [Multithreaded Web Server Project](#3-multithreaded-web-server)
4. [Introduction to Spring Boot](#4-spring-boot-introduction)
5. [Spring Boot Annotations Deep Dive](#5-annotations)
6. [Dependency Injection & Beans](#6-dependency-injection)
7. [Spring Boot Data Access](#7-data-access)
8. [RESTful APIs with Spring Boot](#8-restful-apis)
9. [Spring Boot Security & JWT](#9-security)
10. [Logging](#10-logging)
11. [Exception Handling](#11-exception-handling)
12. [Caching](#12-caching)
13. [Interceptors](#13-interceptors)
14. [Scheduling](#14-scheduling)
15. [Testing & Mockito](#15-testing)
16. [Microservices Introduction](#16-microservices)
17. [Eureka Service Discovery](#17-eureka)
18. [Distributed Tracing: Sleuth & Zipkin](#18-tracing)
19. [Spring Boot Profiles](#19-profiles)
20. [Spring Cloud Config Server](#20-config-server)
21. [Inter-Service Communication](#21-communication)
22. [API Gateway](#22-api-gateway)
23. [Circuit Breaker](#23-circuit-breaker)
24. [CQRS Pattern](#24-cqrs)
25. [Spring AOP, Proxies & CGLIB](#25-aop)
26. [Kafka & RabbitMQ](#26-messaging)
27. [Expense Tracker Project Design](#27-project)
28. [Deployment & Containerization](#28-deployment)
29. [Linux for Backend Developers](#29-linux)

---

# 1. Core Java Fundamentals

## 1.1 JVM, JDK, JRE

**What are they?**

- **JDK** (Java Development Kit): Full toolkit to *write and compile* Java. Contains compiler (`javac`), debugger, profiler, and the JRE.
- **JRE** (Java Runtime Environment): Everything needed to *run* Java programs — the JVM plus standard libraries. No compiler.
- **JVM** (Java Virtual Machine): The engine that actually *executes* bytecode. Platform-specific, but the bytecode it runs is platform-neutral.

**Why this architecture?**

Java's promise is **Write Once, Run Anywhere (WORA)**. Before Java, C/C++ compiled to machine-specific binaries. Java introduced bytecode (`.class` files) as an intermediate format that any JVM — on Windows, Linux, macOS, AWS — can run identically.

**How the JVM works internally:**

```
Your Code (.java)
     │
     ▼  javac (compiler)
Bytecode (.class)
     │
     ▼  JVM loads via Class Loader
Runtime Data Areas:
  ├── Method Area     (class metadata, static fields)
  ├── Heap            (all object instances — GC lives here)
  ├── Stack           (one per thread; holds stack frames)
  ├── PC Register     (current instruction pointer per thread)
  └── Native Stack    (for native C/C++ methods)
     │
     ▼
Execution Engine:
  ├── Interpreter     (executes bytecode line by line — slow)
  ├── JIT Compiler    (compiles hot paths to native code — fast)
  └── Garbage Collector
```

**JIT Compiler**: The JVM profiles your running app and identifies "hot" code paths (methods called thousands of times). It compiles those directly to native machine code. This is why Java apps get *faster* the longer they run.

**Garbage Collection & Heap Layout:**

```
Heap Memory
├── Young Generation  ← new objects start here
│   ├── Eden Space    ← first allocation
│   ├── Survivor S0
│   └── Survivor S1
└── Old Generation    ← objects that survive many GC cycles
```

- **Minor GC**: Cleans young generation. Fast, frequent.
- **Major/Full GC**: Cleans old generation. Slow, causes "stop-the-world" pause.

In your Expense Tracker project: bulk CSV imports creating thousands of objects in loops can cause GC pressure. Use streaming/pagination to avoid it.

---

## 1.2 Java Memory: Stack vs Heap

```java
public void processExpense() {
    double amount = 500.0;         // primitive — lives on STACK
    Expense expense = new Expense("Food", amount); // object on HEAP,
                                   // reference 'expense' on STACK
    // When method returns, stack frame pops.
    // 'expense' reference gone → object on heap eligible for GC
}
```

**Pass-by-value (Java ALWAYS passes by value):**

```java
public void updateAmount(double amount) {
    amount = 999.0;  // Does NOT affect caller — primitive copied
}

public void updateExpense(Expense expense) {
    expense.setAmount(999.0); // DOES affect object — reference is copied,
                               // but both copies point to same heap object
    expense = new Expense();  // Does NOT change caller's reference
}
```

---

## 1.3 OOP Pillars

### Encapsulation

Bundle data and behavior together, hide internals.

```java
// Bad: internals exposed
public class Expense {
    public double amount;        // anyone can set negative values
}

// Good: controlled access
public class Expense {
    private BigDecimal amount;

    public void setAmount(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0)
            throw new IllegalArgumentException("Amount must be positive");
        this.amount = amount;
    }
    public BigDecimal getAmount() { return amount; }
}
```

### Inheritance

```java
// Shared audit fields — avoid duplication across all entities
@MappedSuperclass
public abstract class BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(updatable = false)
    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() { createdAt = updatedAt = LocalDateTime.now(); }

    @PreUpdate
    protected void onUpdate() { updatedAt = LocalDateTime.now(); }
}

@Entity
public class Expense extends BaseEntity {
    private String description;   // inherits id, createdAt, updatedAt
    private BigDecimal amount;
}

@Entity
public class User extends BaseEntity {
    private String email;         // inherits id, createdAt, updatedAt
}
```

### Polymorphism

```java
public interface NotificationService {
    void send(String userId, String message);
}

@Service("email")
public class EmailNotificationService implements NotificationService {
    @Override
    public void send(String userId, String message) {
        // send email via SMTP
    }
}

@Service("push")
public class PushNotificationService implements NotificationService {
    @Override
    public void send(String userId, String message) {
        // send push notification via FCM
    }
}

// Expense service only depends on the abstraction
@Service
public class ExpenseAlertService {
    private final NotificationService notificationService;

    public ExpenseAlertService(@Qualifier("email") NotificationService ns) {
        this.notificationService = ns;
    }

    public void alertBudgetExceeded(String userId) {
        notificationService.send(userId, "Budget limit exceeded!");
        // Swap to push notifications: change @Qualifier — zero other changes
    }
}
```

### Abstraction — Template Method Pattern

```java
public abstract class ReportGenerator {
    // Template method — fixed algorithm, variable steps
    public final Report generate(DateRange range) {
        List<Expense> data = fetchData(range);         // common
        List<ExpenseDTO> processed = process(data);    // abstract
        return buildReport(processed);                 // abstract
    }

    private List<Expense> fetchData(DateRange range) {
        return expenseRepository.findByDateRange(range.start(), range.end());
    }

    protected abstract List<ExpenseDTO> process(List<Expense> raw);
    protected abstract Report buildReport(List<ExpenseDTO> processed);
}

public class MonthlyReportGenerator extends ReportGenerator {
    @Override
    protected List<ExpenseDTO> process(List<Expense> raw) {
        return raw.stream()
            .collect(Collectors.groupingBy(Expense::getCategory))
            .entrySet().stream()
            .map(e -> new ExpenseDTO(e.getKey(), sumAmounts(e.getValue())))
            .toList();
    }

    @Override
    protected Report buildReport(List<ExpenseDTO> processed) {
        return new MonthlyReport(processed);
    }
}
```

---

## 1.4 Exception Handling

```
Throwable
├── Error          (JVM problems: OutOfMemoryError — never catch)
└── Exception
    ├── Checked    (compiler forces handle/declare: IOException)
    └── RuntimeException (unchecked: NullPointerException, IllegalArgumentException)
```

In Spring Boot projects, **always use unchecked (Runtime) exceptions** to avoid polluting method signatures.

```java
// Custom exception hierarchy for Expense Tracker
public class ExpenseTrackerException extends RuntimeException {
    private final String errorCode;
    public ExpenseTrackerException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    public String getErrorCode() { return errorCode; }
}

public class ResourceNotFoundException extends ExpenseTrackerException {
    public ResourceNotFoundException(String resource, Long id) {
        super("RESOURCE_NOT_FOUND",
              String.format("%s with id %d not found", resource, id));
    }
}

public class InsufficientBudgetException extends ExpenseTrackerException {
    public InsufficientBudgetException(BigDecimal available, BigDecimal requested) {
        super("INSUFFICIENT_BUDGET",
              String.format("Available: %s, Requested: %s", available, requested));
    }
}
```

**Try-with-resources** — always use for AutoCloseable resources:

```java
// Bad: connection leaks if exception occurs
Connection conn = dataSource.getConnection();
try { /* process */ } finally { conn.close(); }

// Good: both resources always closed
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement("SELECT * FROM expenses")) {
    ResultSet rs = stmt.executeQuery();
    while (rs.next()) { /* process */ }
}
```

---

# 2. Advanced Java

## 2.1 Functional Interfaces

A functional interface has **exactly one abstract method**. It enables lambdas.

**Built-in functional interfaces (java.util.function):**

```java
// Predicate<T>: T → boolean  — USE FOR FILTERING
Predicate<Expense> isLarge   = e -> e.getAmount().compareTo(new BigDecimal("1000")) > 0;
Predicate<Expense> isFood    = e -> "FOOD".equals(e.getCategory());
Predicate<Expense> largeFoood = isLarge.and(isFood);
Predicate<Expense> largeOrFood = isLarge.or(isFood);
Predicate<Expense> notLarge  = isLarge.negate();

List<Expense> filtered = expenses.stream().filter(largeFoood).toList();

// Function<T,R>: T → R  — USE FOR MAPPING/TRANSFORMING
Function<Expense, ExpenseDTO> toDTO = expense -> new ExpenseDTO(
    expense.getId(), expense.getDescription(), expense.getAmount());

Function<String, String> trim      = String::trim;
Function<String, String> upper     = String::toUpperCase;
Function<String, String> trimUpper = trim.andThen(upper); // compose

// Consumer<T>: T → void  — USE FOR SIDE EFFECTS
Consumer<Expense> logIt  = e -> log.info("Processing: {}", e.getId());
Consumer<Expense> saveIt = e -> repository.save(e);
Consumer<Expense> logAndSave = logIt.andThen(saveIt);
expenses.forEach(logAndSave);

// Supplier<T>: () → T  — USE FOR LAZY/FACTORY
Supplier<Expense> defaultExpense = () -> new Expense("Uncategorized", BigDecimal.ZERO);
Expense e = optionalExpense.orElseGet(defaultExpense);

// BiFunction<T,U,R>: two inputs → R
BiFunction<Expense, String, String> formatWithCurrency =
    (expense, currency) -> currency + " " + expense.getAmount();
```

**Custom functional interface:**

```java
@FunctionalInterface
public interface ExpenseValidator {
    ValidationResult validate(Expense expense);

    default ExpenseValidator and(ExpenseValidator other) {
        return expense -> {
            ValidationResult r = this.validate(expense);
            return r.isValid() ? other.validate(expense) : r;
        };
    }
}

// Usage
ExpenseValidator amountCheck    = e -> e.getAmount().compareTo(BigDecimal.ZERO) > 0
    ? ValidationResult.ok() : ValidationResult.fail("Amount must be positive");
ExpenseValidator categoryCheck  = e -> e.getCategory() != null
    ? ValidationResult.ok() : ValidationResult.fail("Category required");
ExpenseValidator fullValidation = amountCheck.and(categoryCheck);
```

---

## 2.2 Lambda Expressions

```java
// Syntax variants
(TypeA a, TypeB b) -> { return result; }   // full
(a, b) -> result                            // type inferred, single expression
a -> a.getAmount()                          // single param, no parens needed
() -> new Expense()                         // no params

// Method references (shorthand lambdas)
Integer::parseInt          // static:   s -> Integer.parseInt(s)
repo::save                 // instance: e -> repo.save(e)
String::toUpperCase        // arbitrary: s -> s.toUpperCase()
Expense::new               // constructor: () -> new Expense()
```

```java
@Service
public class ExpenseAnalyticsService {

    public Map<String, BigDecimal> totalByCategory(List<Expense> expenses) {
        return expenses.stream()
            .collect(Collectors.groupingBy(
                Expense::getCategory,                      // method ref
                Collectors.reducing(BigDecimal.ZERO,
                    Expense::getAmount, BigDecimal::add)   // method refs
            ));
    }

    public Optional<Expense> mostExpensive(List<Expense> expenses) {
        return expenses.stream()
            .max(Comparator.comparing(Expense::getAmount));
    }

    public BigDecimal totalAboveThreshold(List<Expense> expenses, BigDecimal threshold) {
        return expenses.stream()
            .filter(e -> e.getAmount().compareTo(threshold) > 0)
            .map(Expense::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

---

## 2.3 Stream API

Streams are **lazy pipelines** for processing data. Not data structures — they process and discard.

```
Source → [Intermediate Ops (lazy)]* → Terminal Op (triggers execution)
```

```java
@Service
public class ExpenseReportService {

    // filter + map + collect
    public List<ExpenseDTO> getExpenseDTOs(List<Expense> expenses) {
        return expenses.stream()
            .filter(e -> e.getAmount().compareTo(BigDecimal.ZERO) > 0)
            .map(mapper::toDTO)
            .collect(Collectors.toList());
    }

    // statistics
    public ExpenseSummary getSummary(List<Expense> expenses) {
        DoubleSummaryStatistics stats = expenses.stream()
            .mapToDouble(e -> e.getAmount().doubleValue())
            .summaryStatistics();

        return new ExpenseSummary(stats.getSum(), stats.getAverage(),
                                   stats.getMax(), stats.getMin(), stats.getCount());
    }

    // groupingBy
    public Map<String, List<Expense>> groupByCategory(List<Expense> expenses) {
        return expenses.stream().collect(Collectors.groupingBy(Expense::getCategory));
    }

    // partitioningBy
    public Map<Boolean, List<Expense>> partitionByBudget(List<Expense> expenses,
                                                          BigDecimal limit) {
        return expenses.stream()
            .collect(Collectors.partitioningBy(e -> e.getAmount().compareTo(limit) <= 0));
    }

    // flatMap — flatten nested lists
    public List<Tag> getAllTags(List<Expense> expenses) {
        return expenses.stream()
            .flatMap(e -> e.getTags().stream())  // Stream<List<Tag>> → Stream<Tag>
            .distinct()
            .toList();
    }

    // reduce
    public BigDecimal totalAmount(List<Expense> expenses) {
        return expenses.stream()
            .map(Expense::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    // joining
    public String expenseSummaryString(List<Expense> expenses) {
        return expenses.stream()
            .map(e -> e.getDescription() + ": " + e.getAmount())
            .collect(Collectors.joining(", ", "[", "]"));
    }

    // anyMatch / allMatch / noneMatch
    public boolean hasAnyLargeExpense(List<Expense> expenses, BigDecimal limit) {
        return expenses.stream().anyMatch(e -> e.getAmount().compareTo(limit) > 0);
    }
}
```

**Lazy evaluation example:**

```java
// Nothing executes here — no terminal operation
Stream<Expense> stream = expenses.stream()
    .filter(e -> { System.out.println("filter"); return e.getAmount().intValue() > 100; })
    .map(e -> { System.out.println("map"); return e; });

// NOW everything executes
long count = stream.count();
```

---

## 2.4 Optional

**Why:** Eliminates `NullPointerException` by making absence explicit.

```java
// Repository returns Optional
Optional<User> user = userRepository.findById(id);

// BAD — defeats the purpose
if (user.isPresent()) { User u = user.get(); }

// GOOD — orElseThrow (most common in services)
User u = userRepository.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("User", id));

// GOOD — map: transform if present
Optional<String> email = userRepository.findById(id).map(User::getEmail);

// GOOD — flatMap: when transformation returns Optional
Optional<Address> address = userRepository.findById(id)
    .flatMap(User::getPrimaryAddress);  // getPrimaryAddress returns Optional<Address>

// GOOD — filter: conditional presence
Optional<User> admin = userRepository.findById(id)
    .filter(u -> u.getRole() == Role.ADMIN);

// GOOD — orElse: default value
String display = optName.orElse("Anonymous");

// GOOD — orElseGet: lazy default (expensive operation)
User guest = optUser.orElseGet(() -> guestService.createGuest());

// GOOD — ifPresent: side effect
optUser.ifPresent(u -> auditLog.record("Accessed: " + u.getId()));

// GOOD — ifPresentOrElse (Java 9+)
optUser.ifPresentOrElse(
    u -> log.info("Found user {}", u.getId()),
    () -> log.warn("User {} not found", id)
);
```

**Rules:**
- ✅ Return type from methods where value might be absent
- ❌ Never use as entity/DTO field (bad serialization)
- ❌ Never use in collections (`List<Optional<X>>` is an antipattern)

---

# 3. Multithreaded Web Server

## 3.1 Sockets & TCP

```java
// Basic single-threaded server (handles one client at a time — WRONG for production)
public class SimpleServer {
    public void start(int port) throws IOException {
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Listening on " + port);
            while (true) {
                Socket client = serverSocket.accept(); // BLOCKS until client connects
                handleClient(client);  // PROBLEM: next client waits here
            }
        }
    }

    private void handleClient(Socket socket) throws IOException {
        try (var reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             var writer = new PrintWriter(socket.getOutputStream(), true)) {
            String line = reader.readLine();
            System.out.println("Got: " + line);
            writer.println("HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\n\r\nHello!");
        }
    }
}
```

## 3.2 Multithreading

**Thread states:**
```
NEW → RUNNABLE ↔ BLOCKED/WAITING/TIMED_WAITING → TERMINATED
```

**Thread pool server (production approach):**

```java
public class ThreadPoolWebServer {
    private final ExecutorService pool;
    private final int port;

    public ThreadPoolWebServer(int port, int poolSize) {
        this.port = port;
        // Reuse N threads — no thread-per-request overhead
        this.pool = Executors.newFixedThreadPool(poolSize);
    }

    public void start() throws IOException {
        try (ServerSocket server = new ServerSocket(port)) {
            while (true) {
                Socket client = server.accept();
                pool.submit(() -> handleClient(client)); // delegate to pool
            }
        }
    }

    public void stop() { pool.shutdown(); }
}
```

**Race condition & synchronization:**

```java
// UNSAFE: count++ is read-increment-write (3 non-atomic ops)
public class UnsafeCounter {
    private int count = 0;
    public void increment() { count++; } // race condition!
}

// SAFE option 1: synchronized
public class SyncCounter {
    private int count = 0;
    public synchronized void increment() { count++; }
    public synchronized int get() { return count; }
}

// SAFE option 2: AtomicInteger (faster, lock-free)
public class AtomicCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    public void increment() { count.incrementAndGet(); }
    public int get() { return count.get(); }
}

// SAFE option 3: ReentrantLock (most flexible)
public class LockCounter {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    public void increment() {
        lock.lock();
        try { count++; }
        finally { lock.unlock(); } // ALWAYS in finally
    }
}
```

**HTTP Request Parser:**

```java
public class HttpRequest {
    private String method, path, version;
    private Map<String, String> headers = new HashMap<>();
    private String body;

    public static HttpRequest parse(BufferedReader reader) throws IOException {
        HttpRequest req = new HttpRequest();
        // Parse: "GET /api/expenses HTTP/1.1"
        String[] parts = reader.readLine().split(" ");
        req.method = parts[0]; req.path = parts[1]; req.version = parts[2];

        String line;
        while ((line = reader.readLine()) != null && !line.isEmpty()) {
            int colon = line.indexOf(':');
            req.headers.put(line.substring(0, colon).trim().toLowerCase(),
                            line.substring(colon + 1).trim());
        }

        String len = req.headers.get("content-length");
        if (len != null) {
            char[] buf = new char[Integer.parseInt(len)];
            reader.read(buf, 0, buf.length);
            req.body = new String(buf);
        }
        return req;
    }
}
```

---

# 4. Spring Boot Introduction

## 4.1 The Problem Spring Boot Solved

**Before Spring Boot**, a simple REST API required:
1. `web.xml` (100+ lines of XML)
2. `applicationContext.xml`, `spring-mvc.xml`, `security.xml`
3. Manual dependency version management (constant conflicts)
4. Deploy WAR to external Tomcat
5. Configure DataSource, TransactionManager, EntityManagerFactory manually

A "Hello World" took hours. One wrong XML tag = runtime crash.

**Spring Boot provides:**

- **Auto-configuration**: Detects classpath and configures beans automatically. Add JPA jar → Hibernate configured.
- **Opinionated defaults**: Sensible defaults for 90% of cases. Override only what you need.
- **Embedded server**: Tomcat bundled in your JAR. `java -jar app.jar` = running server.
- **Starter POMs**: Pre-packaged compatible dependency groups.

```
Traditional               Spring Boot
─────────────             ────────────
web.xml (200 lines)   →   @SpringBootApplication
spring-mvc.xml        →   Auto-configured
External Tomcat       →   Embedded Tomcat
WAR file              →   Executable JAR
Manual versions       →   Starter POMs
```

## 4.2 Maven pom.xml Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>

    <!-- PARENT: manages all dependency versions — no conflicts -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.expensetracker</groupId>
    <artifactId>expense-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Web: REST + embedded Tomcat -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- JPA: ORM with Hibernate -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- Security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- Bean Validation: @NotNull, @Email, etc. -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- PostgreSQL driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Lombok: reduces boilerplate (@Getter, @Builder, @Slf4j) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.3</version>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Creates executable fat JAR containing all dependencies -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 4.3 Layered Architecture

```
┌──────────────────────────────────┐
│          Client (HTTP)           │
└───────────────┬──────────────────┘
                │ HTTP Request
┌───────────────▼──────────────────┐
│       Controller Layer           │  ← Maps URLs, handles HTTP, returns response
│  @RestController, @GetMapping    │     Does NOT contain business logic
└───────────────┬──────────────────┘
                │ calls
┌───────────────▼──────────────────┐
│        Service Layer             │  ← Business logic, transactions, orchestration
│  @Service, @Transactional        │     Does NOT know about HTTP or SQL
└───────────────┬──────────────────┘
                │ calls
┌───────────────▼──────────────────┐
│      Repository Layer            │  ← Data access ONLY. No business logic.
│  @Repository, JpaRepository      │
└───────────────┬──────────────────┘
                │ SQL / NoSQL
┌───────────────▼──────────────────┐
│          Database                │
└──────────────────────────────────┘
```

**Why strict layers matter:**
- Swap PostgreSQL for MongoDB → only Repository changes
- Add REST API for mobile → Controller changes, Service untouched
- Change business rules → only Service changes

## 4.4 Entry Point

```java
@SpringBootApplication
// = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan
public class ExpenseTrackerApplication {
    public static void main(String[] args) {
        // Creates ApplicationContext, starts embedded Tomcat, registers beans
        SpringApplication.run(ExpenseTrackerApplication.class, args);
    }
}
```


# 5. Annotations

## 5.1 @Controller vs @RestController

```java
// @Controller — returns VIEW NAMES (Thymeleaf, JSP)
@Controller
public class WebController {
    @GetMapping("/dashboard")
    public String dashboard(Model model) {
        model.addAttribute("expenses", expenseService.getRecent());
        return "dashboard"; // resolves to templates/dashboard.html
    }
}

// @RestController = @Controller + @ResponseBody on every method
// Serializes return value directly to JSON — no view resolution
@RestController
@RequestMapping("/api/v1/expenses")
public class ExpenseController {
    @GetMapping
    public List<ExpenseDTO> getAll() {
        return expenseService.getAll(); // auto-serialized to JSON
    }
}
```

## 5.2 Request Mapping Annotations

```java
@RestController
@RequestMapping("/api/v1/expenses")
public class ExpenseController {

    private final ExpenseService expenseService;

    public ExpenseController(ExpenseService expenseService) {
        this.expenseService = expenseService;
    }

    // GET /api/v1/expenses?page=0&size=10&category=FOOD
    @GetMapping
    public ResponseEntity<Page<ExpenseDTO>> getAll(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(required = false) String category) {
        return ResponseEntity.ok(expenseService.findAll(page, size, category));
    }

    // GET /api/v1/expenses/42
    @GetMapping("/{id}")
    public ResponseEntity<ExpenseDTO> getById(@PathVariable Long id) {
        return ResponseEntity.ok(expenseService.findById(id));
    }

    // POST /api/v1/expenses
    @PostMapping
    public ResponseEntity<ExpenseDTO> create(
            @RequestBody @Valid CreateExpenseRequest request) {
        ExpenseDTO created = expenseService.create(request);
        URI location = URI.create("/api/v1/expenses/" + created.id());
        return ResponseEntity.created(location).body(created);
    }

    // PUT /api/v1/expenses/42  (full replacement)
    @PutMapping("/{id}")
    public ResponseEntity<ExpenseDTO> update(
            @PathVariable Long id,
            @RequestBody @Valid UpdateExpenseRequest request) {
        return ResponseEntity.ok(expenseService.update(id, request));
    }

    // PATCH /api/v1/expenses/42/amount  (partial)
    @PatchMapping("/{id}/amount")
    public ResponseEntity<ExpenseDTO> updateAmount(
            @PathVariable Long id,
            @RequestBody @Valid UpdateAmountRequest request) {
        return ResponseEntity.ok(expenseService.updateAmount(id, request.amount()));
    }

    // DELETE /api/v1/expenses/42
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        expenseService.delete(id);
        return ResponseEntity.noContent().build(); // 204
    }
}
```

**Parameter binding summary:**

| Annotation | Source | Example |
|---|---|---|
| `@PathVariable` | URL path `/{id}` | `GET /expenses/42` |
| `@RequestParam` | Query string | `GET /expenses?page=0` |
| `@RequestBody` | Request JSON body | `POST /expenses` |
| `@RequestHeader` | HTTP header | `X-Correlation-ID` |

## 5.3 @Autowired, @Component, @Service, @Repository

```java
// @Component — generic Spring-managed bean
@Component
public class ExpenseMapper {
    public ExpenseDTO toDTO(Expense expense) { /* mapping */ }
}

// @Service — @Component for business logic layer (semantic clarity)
@Service
public class ExpenseService {
    // Spring creates and manages this bean as singleton
}

// @Repository — @Component with DB exception translation
// Converts JDBC/JPA exceptions to Spring's DataAccessException hierarchy
@Repository
public interface ExpenseRepository extends JpaRepository<Expense, Long> { }

// Constructor injection (RECOMMENDED — dependencies are explicit, field can be final)
@Service
public class ExpenseService {
    private final ExpenseRepository repository;
    private final ExpenseMapper mapper;

    // @Autowired optional since Spring 4.3 when single constructor exists
    public ExpenseService(ExpenseRepository repository, ExpenseMapper mapper) {
        this.repository = repository;
        this.mapper = mapper;
    }
}
```

## 5.4 @Configuration and @Bean

```java
// When you need to configure third-party classes (no @Component on them)
@Configuration
public class SecurityConfig {

    // Spring manages this as a singleton bean
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("https://app.expensetracker.com"));
        config.setAllowedMethods(List.of("GET","POST","PUT","DELETE","PATCH"));
        config.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}
```

## 5.5 @Value and @ConfigurationProperties

```java
// @Value — single property injection
@Service
public class JwtService {
    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration-ms:86400000}") // 86400000ms default
    private long expirationMs;
}

// @ConfigurationProperties — bind group of properties to POJO (PREFERRED for groups)
// application.yml:
// app:
//   jwt:
//     secret: mySecret
//     expiration-ms: 86400000
//   max-upload-size: 5MB

@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    private Jwt jwt = new Jwt();
    private String maxUploadSize;

    public static class Jwt {
        private String secret;
        private long expirationMs;
        // getters/setters
    }
    // getters/setters
}
```

## 5.6 @Qualifier — Resolving Ambiguity

```java
@Service("postgresExpenseRepo")
public class PostgresExpenseRepository implements ExpenseRepository { }

@Service("mongoExpenseRepo")
public class MongoExpenseRepository implements ExpenseRepository { }

@Service
public class ExpenseService {
    private final ExpenseRepository repository;

    public ExpenseService(@Qualifier("postgresExpenseRepo") ExpenseRepository repository) {
        this.repository = repository;
    }
}
```

## 5.7 @Profile — Environment-Specific Beans

```java
// Only created when 'dev' profile is active
@Component
@Profile("dev")
public class DevDataSeeder implements CommandLineRunner {
    @Override
    public void run(String... args) {
        // Seed test data — only in dev
    }
}

// Real email in prod, mock in dev/qa
@Service
@Profile("prod")
public class SmtpEmailService implements EmailService {
    public void send(String to, String subject, String body) { /* SMTP */ }
}

@Service
@Profile("!prod")
public class MockEmailService implements EmailService {
    public void send(String to, String subject, String body) {
        log.info("[MOCK EMAIL] To: {}, Subject: {}", to, subject);
    }
}
```

## 5.8 @Entity and JPA Annotations

```java
@Entity
@Table(name = "expenses",
    indexes = {
        @Index(name = "idx_expense_user",     columnList = "user_id"),
        @Index(name = "idx_expense_date",     columnList = "expense_date"),
        @Index(name = "idx_expense_category", columnList = "category")
    })
@Getter @Setter @NoArgsConstructor @Builder
public class Expense extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 255)
    private String description;

    @Column(nullable = false, precision = 12, scale = 2)
    private BigDecimal amount;

    // Store as string "FOOD" not integer 0,1,2 — readable in DB, rename-safe
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 50)
    private ExpenseCategory category;

    @Column(nullable = false)
    private LocalDate expenseDate;

    // LAZY: don't load User when loading Expense — avoid N+1 problem
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @OneToMany(mappedBy = "expense", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Receipt> receipts = new ArrayList<>();
}
```

## 5.9 @Transactional

```java
@Service
public class ExpenseService {

    // Spring wraps this method in a DB transaction
    // Exception → automatic rollback
    @Transactional
    public ExpenseDTO createExpense(CreateExpenseRequest request, Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("User", userId));

        Budget budget = budgetRepository.findByUserId(userId)
            .orElseThrow(() -> new ResourceNotFoundException("Budget", userId));

        // Both run in same transaction
        Expense expense = expenseRepository.save(Expense.from(request, user));
        budget.deduct(request.amount()); // may throw InsufficientBudgetException
        budgetRepository.save(budget);

        // If deduct() throws: BOTH saves are rolled back
        return mapper.toDTO(expense);
    }

    // readOnly = true: optimization, disables dirty checking
    @Transactional(readOnly = true)
    public List<ExpenseDTO> getAll(Long userId) {
        return expenseRepository.findByUserId(userId)
            .stream().map(mapper::toDTO).toList();
    }

    // Propagation: what happens when transactional methods call each other
    @Transactional(propagation = Propagation.REQUIRES_NEW) // always new transaction
    public void logAuditEvent(AuditEvent event) {
        // runs in its own transaction — won't be rolled back with caller
        auditRepository.save(event);
    }
}
```

**Propagation types:**
- `REQUIRED` (default): join existing or create new
- `REQUIRES_NEW`: always new transaction, suspend existing
- `SUPPORTS`: join if exists, non-transactional if not
- `NOT_SUPPORTED`: always non-transactional, suspend existing

## 5.10 @EnableCaching / @Cacheable

```java
@SpringBootApplication
@EnableCaching  // activate caching infrastructure
public class ExpenseTrackerApplication { }

@Service
public class ExpenseAnalyticsService {

    // Result cached on first call; subsequent calls return cached value
    // Cache key: userId + "-" + yearMonth (SpEL expression)
    @Cacheable(value = "monthlyReports", key = "#userId + '-' + #yearMonth")
    public MonthlyReport getMonthlyReport(Long userId, YearMonth yearMonth) {
        // Expensive query — only runs on cache miss
        return reportRepository.compute(userId, yearMonth);
    }

    // Evict when data changes — keeps cache consistent
    @CacheEvict(value = "monthlyReports",
                key = "#userId + '-' + T(java.time.YearMonth).from(#expenseDate)")
    @Transactional
    public ExpenseDTO createExpense(CreateExpenseRequest req, Long userId,
                                    LocalDate expenseDate) {
        return mapper.toDTO(expenseRepository.save(Expense.from(req)));
    }

    // Evict all entries in cache
    @CacheEvict(value = "monthlyReports", allEntries = true)
    public void invalidateAll() { }

    // Always run method AND update cache
    @CachePut(value = "exchangeRates", key = "#currency")
    public ExchangeRate refreshRate(String currency) {
        return externalApiClient.fetchRate(currency);
    }
}
```

## 5.11 @Async

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean("taskExecutor")
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {
    // Runs in separate thread — caller does NOT wait
    @Async("taskExecutor")
    public CompletableFuture<Void> sendBudgetAlert(Long userId, BigDecimal amount) {
        emailService.sendBudgetExceededEmail(userId, amount); // slow — 2-3 seconds
        return CompletableFuture.completedFuture(null);
    }
}

@Service
public class ExpenseService {
    @Transactional
    public ExpenseDTO createExpense(CreateExpenseRequest request, Long userId) {
        Expense saved = expenseRepository.save(Expense.from(request));
        notificationService.sendBudgetAlert(userId, request.amount()); // fire-and-forget
        return mapper.toDTO(saved); // returns immediately, email sends in background
    }
}
```

## 5.12 @Scheduled

```java
@Configuration
@EnableScheduling
public class SchedulingConfig { }

@Component
@Slf4j
public class ExpenseTrackerScheduler {

    @Scheduled(cron = "0 0 1 * * *", zone = "Asia/Kolkata") // 1 AM daily
    public void generateDailyReports() {
        log.info("Generating daily reports...");
        try { reportService.generateForAllUsers(); }
        catch (Exception e) { log.error("Report generation failed", e); }
        // Never rethrow — scheduler stops on exception
    }

    @Scheduled(fixedRate = 30 * 60 * 1000)  // every 30 minutes
    public void checkBudgetAlerts() { budgetAlertService.checkAll(); }

    @Scheduled(fixedDelay = 60 * 1000)  // 60s after previous run completes
    public void cleanupExpiredTokens() { tokenService.deleteExpired(); }

    @Scheduled(cron = "0 0 8 * * MON")  // 8 AM every Monday
    public void sendWeeklySummary() { emailService.sendWeeklySummary(); }
}
```

**Cron format:** `second minute hour day-of-month month day-of-week`

---

# 6. Dependency Injection & Beans

## 6.1 What Is DI and Why It Matters

**Without DI (tightly coupled):**

```java
public class ExpenseService {
    // Service creates its own dependency — impossible to test without real DB
    private ExpenseRepository repo = new JpaExpenseRepository(
        new HikariDataSource("jdbc:postgresql://localhost/expenses", "user", "pass")
    );
}
```

**With DI (loosely coupled):**

```java
public class ExpenseService {
    private final ExpenseRepository repo; // interface — don't care which implementation

    public ExpenseService(ExpenseRepository repo) {
        this.repo = repo; // Spring injects it
    }
    // Tests can inject a mock. Production injects the JPA implementation.
}
```

## 6.2 Inversion of Control (IoC)

Traditional: your code calls libraries, creates objects, controls flow.
IoC: the **framework** creates objects, injects dependencies, calls your code.

The **IoC Container** (`ApplicationContext`) is Spring's DI engine:
1. Scans packages for `@Component`, `@Service`, `@Repository`, `@Controller`
2. Creates singleton instances (beans)
3. Resolves dependency graph
4. Injects dependencies (constructor/field/setter)

## 6.3 Bean Scopes

```java
// SINGLETON (default): one instance per ApplicationContext
@Service  // implicitly singleton
public class ExpenseService { }

// PROTOTYPE: new instance every time bean is requested
@Component
@Scope("prototype")
public class CsvExpenseParser {
    private final List<Expense> parsedRows = new ArrayList<>(); // mutable state OK
}

// REQUEST: one instance per HTTP request
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST,
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestContext {
    private String correlationId;
    private User currentUser;
}

// SESSION: one instance per HTTP session
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION,
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserPreferences {
    private String preferredCurrency = "INR";
    private String theme = "dark";
}
```

## 6.4 Bean Lifecycle

```java
@Component
@Slf4j
public class KafkaConsumerPool {

    @Autowired
    private KafkaProperties properties;

    @PostConstruct // Called AFTER all dependencies injected, BEFORE bean is used
    public void initialize() {
        log.info("Initializing Kafka consumer pool with {} threads",
                  properties.getConsumerThreads());
        // safe to use 'properties' here
        createConsumers(properties.getConsumerThreads());
    }

    @PreDestroy // Called before Spring destroys bean (app shutdown)
    public void shutdown() {
        log.info("Shutting down Kafka consumers...");
        consumers.forEach(KafkaConsumer::close);
    }
}
```

Lifecycle order:
1. Constructor called
2. `@Autowired` dependencies injected
3. `@PostConstruct` method called
4. Bean in use
5. `@PreDestroy` called on shutdown

---

# 7. Data Access

## 7.1 Spring Data JPA

**What is JPA?** Specification for ORM (Object-Relational Mapping) — maps Java classes to DB tables. Hibernate is the implementation.

**Why JPA?** Without it: 30+ lines of JDBC per query. With it: 0 lines for common operations.

**Entity design:**

```java
@Entity
@Table(name = "users")
@Getter @Setter @NoArgsConstructor @Builder
public class User extends BaseEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String passwordHash;

    @Enumerated(EnumType.STRING)
    private UserRole role = UserRole.USER;

    // LAZY: don't load expenses when loading user
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Expense> expenses = new ArrayList<>();

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    private Budget budget;
}
```

**Repository:**

```java
@Repository
public interface ExpenseRepository extends JpaRepository<Expense, Long> {
    // JpaRepository gives: save, findById, findAll, delete, count, existsById, etc.

    // --- Query Methods (Spring generates SQL from method name) ---

    List<Expense> findByUserId(Long userId);
    // → SELECT * FROM expenses WHERE user_id = ?

    List<Expense> findByUserIdAndCategory(Long userId, ExpenseCategory category);
    // → SELECT * FROM expenses WHERE user_id = ? AND category = ?

    List<Expense> findByUserIdAndExpenseDateBetween(Long userId, LocalDate from, LocalDate to);
    // → SELECT * FROM expenses WHERE user_id = ? AND expense_date BETWEEN ? AND ?

    List<Expense> findByUserIdOrderByAmountDesc(Long userId);
    // → SELECT * FROM expenses WHERE user_id = ? ORDER BY amount DESC

    Page<Expense> findByUserId(Long userId, Pageable pageable);
    // → paginated query with ORDER BY

    // --- Custom JPQL queries ---
    @Query("SELECT e FROM Expense e WHERE e.user.id = :userId " +
           "AND e.amount > :minAmount AND e.category IN :categories")
    List<Expense> findByCriteria(@Param("userId") Long userId,
                                  @Param("minAmount") BigDecimal minAmount,
                                  @Param("categories") List<ExpenseCategory> categories);

    // --- Native SQL for complex reports ---
    @Query(value = """
        SELECT category, SUM(amount) as total, COUNT(*) as cnt
        FROM expenses
        WHERE user_id = :userId AND EXTRACT(YEAR FROM expense_date) = :year
        GROUP BY category
        ORDER BY total DESC
        """, nativeQuery = true)
    List<CategoryTotalProjection> getCategoryTotals(@Param("userId") Long userId,
                                                     @Param("year") int year);

    // --- Projection (fetch only needed columns) ---
    @Query("SELECT e.category as category, SUM(e.amount) as total " +
           "FROM Expense e WHERE e.user.id = :userId GROUP BY e.category")
    List<CategorySummary> getCategorySummary(@Param("userId") Long userId);

    // --- Update/Delete queries ---
    @Modifying
    @Transactional
    @Query("UPDATE Expense e SET e.category = :newCat WHERE e.user.id = :userId " +
           "AND e.category = :oldCat")
    int recategorize(@Param("userId") Long userId,
                     @Param("oldCat") ExpenseCategory oldCat,
                     @Param("newCat") ExpenseCategory newCat);

    // Count / existence
    long countByUserIdAndCategory(Long userId, ExpenseCategory category);
    boolean existsByUserIdAndDescriptionAndExpenseDate(Long userId, String desc, LocalDate date);
}

// Projection interface
public interface CategorySummary {
    String getCategory();
    BigDecimal getTotal();
}

public interface CategoryTotalProjection {
    String getCategory();
    BigDecimal getTotal();
    Long getCnt();
}
```

**Service layer:**

```java
@Service
@Transactional
@Slf4j
public class ExpenseService {
    private final ExpenseRepository expenseRepository;
    private final UserRepository userRepository;
    private final ExpenseMapper mapper;

    public ExpenseService(ExpenseRepository expenseRepository,
                          UserRepository userRepository,
                          ExpenseMapper mapper) {
        this.expenseRepository = expenseRepository;
        this.userRepository = userRepository;
        this.mapper = mapper;
    }

    @Transactional(readOnly = true)
    public Page<ExpenseDTO> getExpenses(Long userId, int page, int size,
                                         String sortBy, String dir) {
        Sort sort = "desc".equalsIgnoreCase(dir)
            ? Sort.by(sortBy).descending()
            : Sort.by(sortBy).ascending();
        Pageable pageable = PageRequest.of(page, size, sort);
        return expenseRepository.findByUserId(userId, pageable).map(mapper::toDTO);
    }

    public ExpenseDTO createExpense(CreateExpenseRequest req, Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("User", userId));

        Expense expense = Expense.builder()
            .description(req.description())
            .amount(req.amount())
            .category(req.category())
            .expenseDate(req.expenseDate())
            .user(user)
            .build();

        return mapper.toDTO(expenseRepository.save(expense));
    }

    public void delete(Long id, Long userId) {
        Expense expense = expenseRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Expense", id));
        if (!expense.getUser().getId().equals(userId))
            throw new AccessDeniedException("Not your expense");
        expenseRepository.delete(expense);
    }
}
```

## 7.2 Spring JDBC — When to Use Instead of JPA

Use Spring JDBC/JdbcTemplate when:
- Complex aggregation reports (many joins, GROUP BY, window functions)
- Bulk insert/update (JPA is too slow for thousands of rows)
- Database-specific SQL features (CTEs, partitioning)

```java
@Repository
public class ExpenseAnalyticsRepository {
    private final JdbcTemplate jdbc;
    private final NamedParameterJdbcTemplate namedJdbc;

    public ExpenseAnalyticsRepository(JdbcTemplate jdbc,
                                       NamedParameterJdbcTemplate namedJdbc) {
        this.jdbc = jdbc;
        this.namedJdbc = namedJdbc;
    }

    public List<MonthlyTrend> getMonthlyTrend(Long userId, int year) {
        String sql = """
            SELECT
                EXTRACT(MONTH FROM expense_date) AS month,
                SUM(amount)     AS total,
                COUNT(*)        AS cnt,
                AVG(amount)     AS average
            FROM expenses
            WHERE user_id = ? AND EXTRACT(YEAR FROM expense_date) = ?
            GROUP BY EXTRACT(MONTH FROM expense_date)
            ORDER BY month
            """;

        return jdbc.query(sql,
            (rs, row) -> new MonthlyTrend(
                rs.getInt("month"),
                rs.getBigDecimal("total"),
                rs.getLong("cnt"),
                rs.getBigDecimal("average")
            ), userId, year);
    }

    // Batch insert: 500x faster than JPA for bulk operations
    public void bulkInsert(List<Expense> expenses) {
        String sql = "INSERT INTO expenses (description, amount, category, expense_date, user_id)" +
                     " VALUES (?, ?, ?, ?, ?)";
        jdbc.batchUpdate(sql, expenses, 500, (ps, expense) -> {
            ps.setString(1, expense.getDescription());
            ps.setBigDecimal(2, expense.getAmount());
            ps.setString(3, expense.getCategory().name());
            ps.setDate(4, Date.valueOf(expense.getExpenseDate()));
            ps.setLong(5, expense.getUser().getId());
        });
    }
}
```

---

# 8. RESTful APIs with Spring Boot

## 8.1 REST Principles

REST (Representational State Transfer) constraints:
- **Stateless**: Every request contains all needed info. No server-side session.
- **Uniform Interface**: Standard HTTP verbs (GET, POST, PUT, PATCH, DELETE)
- **Resource-based**: URLs = nouns, not actions

```
BAD (action-based RPC):          GOOD (resource-based REST):
POST /createExpense          →   POST   /api/v1/expenses
POST /getExpense?id=5        →   GET    /api/v1/expenses/5
POST /updateExpense?id=5     →   PUT    /api/v1/expenses/5
POST /deleteExpense?id=5     →   DELETE /api/v1/expenses/5
POST /getExpensesByUser?u=3  →   GET    /api/v1/users/3/expenses
```

## 8.2 DTOs — Never Expose Entities

```java
// Request DTO: what client sends
public record CreateExpenseRequest(
    @NotBlank(message = "Description is required")
    @Size(max = 255)
    String description,

    @NotNull
    @DecimalMin(value = "0.01", message = "Amount must be positive")
    BigDecimal amount,

    @NotNull
    ExpenseCategory category,

    @NotNull
    @PastOrPresent(message = "Cannot log future expenses")
    LocalDate expenseDate
) {}

// Response DTO: what API returns
@Builder
public record ExpenseDTO(
    Long id,
    String description,
    BigDecimal amount,
    String category,
    LocalDate expenseDate,
    LocalDateTime createdAt
) {}

// Mapper
@Component
public class ExpenseMapper {
    public ExpenseDTO toDTO(Expense expense) {
        return ExpenseDTO.builder()
            .id(expense.getId())
            .description(expense.getDescription())
            .amount(expense.getAmount())
            .category(expense.getCategory().name())
            .expenseDate(expense.getExpenseDate())
            .createdAt(expense.getCreatedAt())
            .build();
    }

    public Expense toEntity(CreateExpenseRequest req, User user) {
        return Expense.builder()
            .description(req.description())
            .amount(req.amount())
            .category(req.category())
            .expenseDate(req.expenseDate())
            .user(user)
            .build();
    }
}
```

## 8.3 HTTP Status Codes

| Code | Meaning | Use When |
|------|---------|----------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST (include Location header) |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation failure, malformed JSON |
| 401 | Unauthorized | Not authenticated (no/invalid token) |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource (email already exists) |
| 422 | Unprocessable | Semantic validation failure |
| 429 | Too Many Requests | Rate limit hit |
| 500 | Internal Server Error | Unhandled exception |
| 503 | Service Unavailable | Circuit breaker OPEN |

## 8.4 Pagination Response

```java
// GET /api/v1/expenses?page=0&size=10&sortBy=expenseDate&dir=desc
@GetMapping
public ResponseEntity<Page<ExpenseDTO>> getAll(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "expenseDate") String sortBy,
        @RequestParam(defaultValue = "desc") String dir) {
    return ResponseEntity.ok(expenseService.getExpenses(getCurrentUserId(), page, size, sortBy, dir));
}

// Client receives:
// {
//   "content": [...],
//   "pageable": { "pageNumber": 0, "pageSize": 10 },
//   "totalElements": 48,
//   "totalPages": 5,
//   "first": true,
//   "last": false,
//   "numberOfElements": 10
// }
```


# 9. Spring Boot Security

## 9.1 Security Architecture

```
HTTP Request
    │
    ▼ Servlet Filter Chain:
  ┌─ CorsFilter
  ├─ CsrfFilter            (disabled for stateless JWT APIs)
  ├─ UsernamePasswordAuthenticationFilter
  ├─ JwtAuthenticationFilter   ← our custom filter
  │       └─ extract JWT → validate → set SecurityContext
  ├─ ExceptionTranslationFilter
  └─ AuthorizationFilter
    │
    ▼
@RestController method
```

## 9.2 Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // enables @PreAuthorize on methods
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtFilter;
    private final UserDetailsService userDetailsService;

    public SecurityConfig(JwtAuthenticationFilter jwtFilter,
                          UserDetailsService userDetailsService) {
        this.jwtFilter = jwtFilter;
        this.userDetailsService = userDetailsService;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())  // JWT is stateless — no CSRF risk
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(s ->
                s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()       // login/register
                .requestMatchers("/actuator/health").permitAll()       // health check
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN") // admin only
                .anyRequest().authenticated()                          // all else: login required
            )
            // Run our JWT filter before Spring's default auth filter
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // cost factor 12
    }
}
```

## 9.3 UserDetails Implementation

```java
@Entity
@Table(name = "users")
@Getter @Setter @NoArgsConstructor @Builder
public class User extends BaseEntity implements UserDetails {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String passwordHash;

    @Enumerated(EnumType.STRING)
    private UserRole role;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_" + role.name()));
    }

    @Override public String getPassword()  { return passwordHash; }
    @Override public String getUsername()  { return email; }  // email is username
    @Override public boolean isAccountNonExpired()     { return true; }
    @Override public boolean isAccountNonLocked()      { return true; }
    @Override public boolean isCredentialsNonExpired() { return true; }
    @Override public boolean isEnabled()               { return true; }
}

@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        return userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + email));
    }
}
```

## 9.4 JWT — JSON Web Tokens

**Structure:** `Header.Payload.Signature`

```
eyJhbGciOiJIUzI1NiJ9               ← Header: {"alg":"HS256"}
.eyJzdWIiOiI0MiIsImVtYWlsIjoiYWxpY2VAZXhhbXBsZS5jb20iLCJleHAiOjE3MDAwMDB9
                                    ← Payload: {"sub":"42","email":"alice@...","exp":...}
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
                                    ← Signature: HMAC-SHA256(header+payload, secret)
```

**Why JWT?** Stateless — server stores nothing. Any server instance can validate by re-computing signature.

```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration-ms}")
    private long accessExpirationMs;

    @Value("${jwt.refresh-expiration-ms}")
    private long refreshExpirationMs;

    private SecretKey signingKey() {
        return Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
    }

    public String generateAccessToken(UserDetails user) {
        return buildToken(user, accessExpirationMs, Map.of());
    }

    public String generateRefreshToken(UserDetails user) {
        return buildToken(user, refreshExpirationMs, Map.of("type", "refresh"));
    }

    private String buildToken(UserDetails user, long expMs, Map<String, Object> extra) {
        return Jwts.builder()
            .claims(extra)
            .subject(user.getUsername())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expMs))
            .signWith(signingKey())
            .compact();
    }

    public String extractEmail(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public boolean isValid(String token, UserDetails user) {
        return extractEmail(token).equals(user.getUsername()) && !isExpired(token);
    }

    private boolean isExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }

    private <T> T extractClaim(String token, Function<Claims, T> resolver) {
        return resolver.apply(
            Jwts.parser().verifyWith(signingKey()).build()
                .parseSignedClaims(token).getPayload()
        );
    }
}
```

## 9.5 JWT Authentication Filter

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    public JwtAuthenticationFilter(JwtService jwtService,
                                    UserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain)
            throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        // No auth header or not Bearer token — skip
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            chain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);
        String email;

        try {
            email = jwtService.extractEmail(token);
        } catch (JwtException e) {
            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Invalid token");
            return;
        }

        // Authenticate only if not already authenticated
        if (email != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails user = userDetailsService.loadUserByUsername(email);

            if (jwtService.isValid(token, user)) {
                UsernamePasswordAuthenticationToken auth =
                    new UsernamePasswordAuthenticationToken(
                        user, null, user.getAuthorities());
                auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(request, response);
    }
}
```

## 9.6 Auth Controller + Refresh Tokens

```java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {
    private final AuthService authService;

    @PostMapping("/register")
    public ResponseEntity<AuthResponse> register(@RequestBody @Valid RegisterRequest req) {
        return ResponseEntity.status(HttpStatus.CREATED).body(authService.register(req));
    }

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody @Valid LoginRequest req) {
        return ResponseEntity.ok(authService.login(req));
    }

    @PostMapping("/refresh")
    public ResponseEntity<AuthResponse> refresh(@RequestBody RefreshTokenRequest req) {
        return ResponseEntity.ok(authService.refresh(req.refreshToken()));
    }

    @PostMapping("/logout")
    public ResponseEntity<Void> logout(HttpServletRequest request) {
        authService.logout(extractToken(request));
        return ResponseEntity.noContent().build();
    }
}

// Refresh token entity (stored in DB)
@Entity
@Table(name = "refresh_tokens")
public class RefreshToken {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false, length = 500)
    private String token;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @Column(nullable = false)
    private Instant expiresAt;

    private boolean revoked = false;
}

@Service
@Transactional
public class AuthService {

    public AuthResponse refresh(String refreshTokenStr) {
        RefreshToken rt = refreshTokenRepository.findByToken(refreshTokenStr)
            .orElseThrow(() -> new InvalidTokenException("Refresh token not found"));

        if (rt.isRevoked()) throw new InvalidTokenException("Token revoked");
        if (rt.getExpiresAt().isBefore(Instant.now()))
            throw new InvalidTokenException("Token expired");

        // ROTATE: revoke old, issue new (security best practice)
        rt.setRevoked(true);
        String newRefresh = createAndSaveRefreshToken(rt.getUser());
        String newAccess  = jwtService.generateAccessToken(rt.getUser());

        return new AuthResponse(newAccess, newRefresh, "Bearer", 900);
    }
}
```

## 9.7 Method-Level Security

```java
@Service
public class ExpenseService {

    // Only expense owner or ADMIN can update
    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public ExpenseDTO update(Long expenseId, Long userId, UpdateExpenseRequest req) {
        // ...
    }

    @PreAuthorize("hasRole('ADMIN')")
    public void adminDelete(Long expenseId) {
        expenseRepository.deleteById(expenseId);
    }
}
```

---

# 10. Logging

## 10.1 Logging Stack

```
Your Code (@Slf4j Logger)
      │
SLF4J API (abstraction — swap implementation without code changes)
      │
Logback (Spring Boot default implementation)
      │
  ├── ConsoleAppender   → stdout
  ├── FileAppender      → file
  └── RollingFileAppender → rotating files (daily/size-based)
```

## 10.2 Logging Best Practices

```java
@Slf4j  // Lombok: injects 'log' field
@Service
public class ExpenseService {

    public ExpenseDTO createExpense(CreateExpenseRequest req, Long userId) {
        // Use {} placeholders — lazy evaluation, no string concat if level disabled
        log.debug("Creating expense for user {}: {}", userId, req);
        log.info("User {} creating {} expense: {}", userId, req.category(), req.amount());

        try {
            Expense saved = expenseRepository.save(/* ... */);
            log.info("Expense {} created for user {}", saved.getId(), userId);
            return mapper.toDTO(saved);
        } catch (DataIntegrityViolationException e) {
            log.error("Failed to save expense for user {}: {}", userId, e.getMessage());
            throw new ExpenseTrackerException("DB_ERROR", "Could not save expense");
        }
    }

    // Warning: large expense might be budget risk
    private void checkAmount(BigDecimal amount, Long userId) {
        if (amount.compareTo(new BigDecimal("5000")) > 0)
            log.warn("Large expense detected: user={}, amount={}", userId, amount);
    }
}
```

## 10.3 Configuration

```yaml
# application.yml
logging:
  level:
    root: INFO
    com.expensetracker: DEBUG               # our code
    org.springframework.security: INFO
    org.hibernate.SQL: DEBUG                # show SQL queries
    org.hibernate.type.descriptor.sql: TRACE # show bind parameters
  file:
    name: logs/expense-tracker.log
  pattern:
    console: "%d{HH:mm:ss.SSS} [%thread] %-5level %-40logger{40} - %msg%n"
    file:    "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

## 10.4 MDC — Correlation IDs Across Logs

```java
@Component
public class MdcLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        // Use existing correlation ID or generate new one
        String correlationId = Optional.ofNullable(req.getHeader("X-Correlation-ID"))
            .orElse(UUID.randomUUID().toString());

        MDC.put("correlationId", correlationId);
        MDC.put("method", req.getMethod());
        MDC.put("uri", req.getRequestURI());

        // Add to response so clients can trace their requests
        res.setHeader("X-Correlation-ID", correlationId);

        try {
            chain.doFilter(req, res);
        } finally {
            MDC.clear(); // CRITICAL: prevent memory leaks in thread pools
        }
    }
}
// Pattern: "%d [%X{correlationId}] %-5level %logger - %msg%n"
// Output:  2024-01-15 [abc-123-def] INFO  ExpenseService - Expense 42 created
```

---

# 11. Exception Handling

## 11.1 @RestControllerAdvice

```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex, WebRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());
        return ErrorResponse.of(404, "Not Found", ex.getMessage(),
                                getPath(request), null);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(err -> errors.put(err.getField(), err.getDefaultMessage()));
        return ErrorResponse.of(400, "Validation Failed", "Request validation failed",
                                null, errors);
    }

    @ExceptionHandler(InsufficientBudgetException.class)
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ErrorResponse handleBudget(InsufficientBudgetException ex) {
        return ErrorResponse.of(422, "Business Rule Violation", ex.getMessage(), null, null);
    }

    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleForbidden(AccessDeniedException ex) {
        return ErrorResponse.of(403, "Forbidden", "Insufficient permissions", null, null);
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDuplicate(DataIntegrityViolationException ex) {
        return ErrorResponse.of(409, "Conflict", "Resource already exists", null, null);
    }

    // Catch-all — log the full stack trace
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneric(Exception ex, WebRequest request) {
        log.error("Unhandled exception at {}", getPath(request), ex);
        return ErrorResponse.of(500, "Internal Server Error",
                                "An unexpected error occurred", getPath(request), null);
    }

    private String getPath(WebRequest req) {
        return req.getDescription(false).replace("uri=", "");
    }
}

// Standard error response (RFC 7807 Problem Details style)
@Builder
public record ErrorResponse(
    int status,
    String error,
    String message,
    String path,
    LocalDateTime timestamp,
    Map<String, String> fieldErrors
) {
    public static ErrorResponse of(int status, String error, String message,
                                    String path, Map<String, String> fieldErrors) {
        return ErrorResponse.builder()
            .status(status).error(error).message(message)
            .path(path).timestamp(LocalDateTime.now()).fieldErrors(fieldErrors).build();
    }
}
```

---

# 12. Caching

## 12.1 Why Cache?

Database queries for monthly analytics can scan millions of rows. Caching stores the computed result so subsequent calls return in microseconds.

**Where to cache in Expense Tracker:**
- Monthly report: expensive aggregation — cache 60 mins (completed months)
- User profile: loaded every authenticated request — cache 30 mins
- Exchange rates: external API call — cache 5 mins
- Category totals: computed from many rows — cache 15 mins

## 12.2 Caffeine Cache Configuration

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager manager = new SimpleCacheManager();
        manager.setCaches(List.of(
            buildCache("monthlyReports",  500, 60),
            buildCache("userProfiles",   1000, 30),
            buildCache("exchangeRates",    10,  5),
            buildCache("categoryTotals",  500, 15)
        ));
        return manager;
    }

    private CaffeineCache buildCache(String name, int maxSize, int ttlMinutes) {
        return new CaffeineCache(name,
            Caffeine.newBuilder()
                .maximumSize(maxSize)
                .expireAfterWrite(ttlMinutes, TimeUnit.MINUTES)
                .recordStats()   // expose hit/miss metrics
                .build());
    }
}
```

## 12.3 Cache Annotations

```java
@Service
public class ExpenseAnalyticsService {

    // Cache the result; subsequent calls with same args return cached value immediately
    // condition: only cache past months (current month data still changing)
    @Cacheable(
        value = "monthlyReports",
        key = "#userId + '-' + #yearMonth",
        condition = "#yearMonth.isBefore(T(java.time.YearMonth).now())"
    )
    @Transactional(readOnly = true)
    public MonthlyReport getMonthlyReport(Long userId, YearMonth yearMonth) {
        log.debug("Cache MISS — computing report for user {} {}", userId, yearMonth);
        return reportRepository.compute(userId, yearMonth);
    }

    // Evict cache when expense changes — prevents stale data
    @CacheEvict(
        value = "monthlyReports",
        key = "#userId + '-' + T(java.time.YearMonth).from(#expenseDate)"
    )
    @Transactional
    public ExpenseDTO createExpense(CreateExpenseRequest req, Long userId, LocalDate expenseDate) {
        return mapper.toDTO(expenseRepository.save(Expense.from(req)));
    }

    // Multiple evictions on one operation
    @Caching(evict = {
        @CacheEvict(value = "monthlyReports", allEntries = true),
        @CacheEvict(value = "categoryTotals", allEntries = true)
    })
    @Transactional
    public void deleteExpense(Long id, Long userId) {
        expenseRepository.deleteById(id);
    }

    // Always execute + update cache (for refresh operations)
    @CachePut(value = "exchangeRates", key = "#currency")
    public ExchangeRate refreshExchangeRate(String currency) {
        return externalApiClient.fetchRate(currency);
    }
}
```

---

# 13. Interceptors

## 13.1 Filter vs Interceptor

| | Servlet Filter | Spring Interceptor |
|---|---|---|
| Level | Servlet container | Spring MVC |
| Spring context access | Limited | Full |
| Access to handler method | No | Yes |
| Use for | Low-level HTTP (CORS, GZIP) | Business concerns (logging, rate limiting) |

## 13.2 Request Logging Interceptor

```java
@Component
@Slf4j
public class RequestLoggingInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res,
                              Object handler) {
        req.setAttribute("startTime", System.currentTimeMillis());

        if (handler instanceof HandlerMethod hm) {
            log.info("→ {} {} [{}#{}]",
                req.getMethod(), req.getRequestURI(),
                hm.getBeanType().getSimpleName(), hm.getMethod().getName());
        }
        return true; // return false to abort — request won't reach controller
    }

    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse res,
                                 Object handler, Exception ex) {
        long duration = System.currentTimeMillis() - (Long) req.getAttribute("startTime");
        log.info("← {} {} | {} | {}ms",
            req.getMethod(), req.getRequestURI(), res.getStatus(), duration);

        if (duration > 2000)
            log.warn("SLOW REQUEST: {} {} took {}ms", req.getMethod(), req.getRequestURI(), duration);
    }
}

// Rate limiting interceptor
@Component
@Slf4j
public class RateLimitInterceptor implements HandlerInterceptor {
    private final RateLimiterService rateLimiter;

    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res,
                              Object handler) throws IOException {
        String key = getClientIp(req) + ":" + req.getRequestURI();
        if (!rateLimiter.isAllowed(key)) {
            log.warn("Rate limit exceeded: {}", key);
            res.setStatus(429);
            res.setHeader("Retry-After", "60");
            res.getWriter().write("{\"error\":\"Rate limit exceeded\",\"retryAfter\":60}");
            return false;
        }
        return true;
    }

    private String getClientIp(HttpServletRequest req) {
        String xff = req.getHeader("X-Forwarded-For");
        return xff != null ? xff.split(",")[0].trim() : req.getRemoteAddr();
    }
}

// Registration
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    private final RequestLoggingInterceptor logging;
    private final RateLimitInterceptor rateLimit;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(logging)
            .addPathPatterns("/api/**");

        registry.addInterceptor(rateLimit)
            .addPathPatterns("/api/v1/auth/**")
            .excludePathPatterns("/api/v1/auth/refresh");
    }
}
```

---

# 14. Scheduling

```java
// Cron: second minute hour day-of-month month day-of-week
// "0 0 1 * * *"    — 1 AM every day
// "0 0 9 * * MON-FRI" — 9 AM weekdays
// "0 0/15 * * * *" — every 15 minutes
// "0 0 0 1 * *"    — midnight first of every month

@Configuration
@EnableScheduling
public class SchedulingConfig { }

@Component
@Slf4j
public class ExpenseTrackerScheduler {
    private final ReportService reportService;
    private final TokenService tokenService;
    private final BudgetAlertService alertService;

    @Scheduled(cron = "0 0 1 * * *", zone = "Asia/Kolkata")
    public void dailyReports() {
        log.info("Starting daily report generation...");
        try {
            int count = reportService.generateAll();
            log.info("Reports generated for {} users", count);
        } catch (Exception e) {
            // DO NOT rethrow — scheduler stops executing if exception escapes
            log.error("Daily report generation failed", e);
        }
    }

    @Scheduled(cron = "0 0 * * * *")  // top of every hour
    public void cleanExpiredTokens() {
        int deleted = tokenService.deleteExpired();
        log.debug("Deleted {} expired tokens", deleted);
    }

    @Scheduled(fixedRate = 30 * 60 * 1000)  // every 30 mins
    public void checkBudgetAlerts() { alertService.checkAndNotify(); }

    @Scheduled(cron = "0 0 8 * * MON")  // 8 AM Monday
    public void weeklySummary() { reportService.sendWeeklySummaryEmails(); }
}
```

---

# 15. Testing & Mockito

## 15.1 Testing Pyramid

```
           ╱─────────────╲
          ╱  E2E / UI      ╲    Few, slow, expensive
         ╱─────────────────╲
        ╱  Integration Tests ╲  Some, moderate speed
       ╱─────────────────────╲
      ╱     Unit Tests         ╲ Many, fast, cheap — focus here
     ╱───────────────────────────╲
```

## 15.2 Unit Testing with Mockito

```java
@ExtendWith(MockitoExtension.class)
class ExpenseServiceTest {

    @Mock  // Mockito creates a fake implementation
    private ExpenseRepository expenseRepository;

    @Mock
    private UserRepository userRepository;

    @Mock
    private ExpenseMapper mapper;

    @InjectMocks  // Creates ExpenseService and injects @Mocks above
    private ExpenseService expenseService;

    private User testUser;
    private Expense testExpense;
    private ExpenseDTO testExpenseDTO;
    private CreateExpenseRequest request;

    @BeforeEach
    void setUp() {
        testUser = User.builder().id(1L).email("alice@example.com").build();
        testExpense = Expense.builder()
            .id(1L).description("Lunch").amount(new BigDecimal("12.50"))
            .category(ExpenseCategory.FOOD).expenseDate(LocalDate.now()).user(testUser).build();
        testExpenseDTO = new ExpenseDTO(1L, "Lunch", new BigDecimal("12.50"),
                                        "FOOD", LocalDate.now(), LocalDateTime.now());
        request = new CreateExpenseRequest("Lunch", new BigDecimal("12.50"),
                                           ExpenseCategory.FOOD, LocalDate.now());
    }

    @Test
    @DisplayName("createExpense — happy path returns DTO")
    void createExpense_success() {
        // ARRANGE — define mock behavior
        when(userRepository.findById(1L)).thenReturn(Optional.of(testUser));
        when(expenseRepository.save(any(Expense.class))).thenReturn(testExpense);
        when(mapper.toDTO(testExpense)).thenReturn(testExpenseDTO);

        // ACT
        ExpenseDTO result = expenseService.createExpense(request, 1L);

        // ASSERT
        assertNotNull(result);
        assertEquals("Lunch", result.description());
        assertEquals(new BigDecimal("12.50"), result.amount());

        // Verify collaborators were called correctly
        verify(userRepository).findById(1L);
        verify(expenseRepository).save(any(Expense.class));
        verify(mapper).toDTO(testExpense);
        verifyNoMoreInteractions(expenseRepository);
    }

    @Test
    @DisplayName("createExpense — user not found throws ResourceNotFoundException")
    void createExpense_userNotFound() {
        when(userRepository.findById(999L)).thenReturn(Optional.empty());

        assertThrows(ResourceNotFoundException.class,
            () -> expenseService.createExpense(request, 999L));

        verify(expenseRepository, never()).save(any());
    }

    @Test
    @DisplayName("getExpenses — returns paginated result with correct sort")
    void getExpenses_pagination() {
        Page<Expense> page = new PageImpl<>(List.of(testExpense),
            PageRequest.of(0, 10), 1);
        when(expenseRepository.findByUserId(eq(1L), any(Pageable.class))).thenReturn(page);
        when(mapper.toDTO(any())).thenReturn(testExpenseDTO);

        Page<ExpenseDTO> result = expenseService.getExpenses(1L, 0, 10, "expenseDate", "desc");

        assertEquals(1, result.getTotalElements());

        ArgumentCaptor<Pageable> captor = ArgumentCaptor.forClass(Pageable.class);
        verify(expenseRepository).findByUserId(eq(1L), captor.capture());
        assertEquals(Sort.Direction.DESC,
            captor.getValue().getSort().getOrderFor("expenseDate").getDirection());
    }

    @Test
    @DisplayName("delete — throws when expense belongs to different user")
    void delete_wrongUser_throws() {
        User otherUser = User.builder().id(2L).build();
        Expense otherExpense = Expense.builder().id(10L).user(otherUser).build();
        when(expenseRepository.findById(10L)).thenReturn(Optional.of(otherExpense));

        assertThrows(AccessDeniedException.class,
            () -> expenseService.delete(10L, 1L));

        verify(expenseRepository, never()).delete(any());
    }
}
```

## 15.3 Integration Testing

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.ANY) // use H2
@Transactional  // rollback after each test
class ExpenseControllerIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    private String bearerToken;

    @BeforeEach
    void setUp() {
        User user = userRepository.save(User.builder()
            .email("test@example.com").passwordHash(encoder.encode("password"))
            .role(UserRole.USER).build());
        bearerToken = jwtService.generateAccessToken(user);
    }

    @Test
    void createExpense_returns201WithLocation() {
        CreateExpenseRequest req = new CreateExpenseRequest("Dinner",
            new BigDecimal("45.00"), ExpenseCategory.FOOD, LocalDate.now());

        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(bearerToken);
        headers.setContentType(MediaType.APPLICATION_JSON);

        ResponseEntity<ExpenseDTO> response = restTemplate.exchange(
            "/api/v1/expenses", HttpMethod.POST,
            new HttpEntity<>(req, headers), ExpenseDTO.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getHeaders().getLocation());
        assertEquals("Dinner", response.getBody().description());
    }

    @Test
    void getExpenses_withoutToken_returns401() {
        ResponseEntity<Void> response = restTemplate.getForEntity("/api/v1/expenses", Void.class);
        assertEquals(HttpStatus.UNAUTHORIZED, response.getStatusCode());
    }
}
```

## 15.4 Repository Testing

```java
@DataJpaTest  // loads ONLY JPA layer — no web, no security
class ExpenseRepositoryTest {

    @Autowired
    private ExpenseRepository expenseRepository;

    @Autowired
    private TestEntityManager em;

    private User user;

    @BeforeEach
    void setUp() {
        user = em.persist(User.builder()
            .email("test@example.com").passwordHash("hash").role(UserRole.USER).build());
        em.flush();
    }

    @Test
    void findByUserIdAndCategory_returnsOnlyMatchingExpenses() {
        em.persist(Expense.builder().user(user).description("Lunch")
            .amount(BigDecimal.TEN).category(ExpenseCategory.FOOD)
            .expenseDate(LocalDate.now()).build());
        em.persist(Expense.builder().user(user).description("Taxi")
            .amount(BigDecimal.TEN).category(ExpenseCategory.TRANSPORT)
            .expenseDate(LocalDate.now()).build());
        em.flush();

        List<Expense> result = expenseRepository.findByUserIdAndCategory(
            user.getId(), ExpenseCategory.FOOD);

        assertEquals(1, result.size());
        assertEquals("Lunch", result.get(0).getDescription());
    }
}
```


# 16. Microservices Introduction

## 16.1 Monolith vs Microservices

```
MONOLITH                              MICROSERVICES
───────────────────────               ────────────────────────────────────────

┌───────────────────────┐             ┌──────────┐  ┌──────────┐  ┌─────────┐
│      Single App       │             │  User    │  │ Expense  │  │ Budget  │
│  ┌─────────────────┐  │             │ Service  │  │ Service  │  │ Service │
│  │   User Module   │  │             │ :8081    │  │ :8082    │  │ :8083   │
│  └─────────────────┘  │             └────┬─────┘  └────┬─────┘  └────┬────┘
│  ┌─────────────────┐  │                  └─────────────┴──────────────┘
│  │ Expense Module  │  │                               │
│  └─────────────────┘  │                        Message Bus /
│  ┌─────────────────┐  │                        API Gateway
│  │  Budget Module  │  │
│  └─────────────────┘  │  Deploy:  All or nothing
│       Single DB       │  Scale:   Scale everything even if only User is busy
└───────────────────────┘  Failure: One bug can bring down entire system
```

**Microservices advantages:**
- **Independent deployment**: Update Budget Service without touching User Service
- **Independent scaling**: 10x Expense Service, 1x Budget Service
- **Technology freedom**: Expense Service uses PostgreSQL, Budget Service uses MongoDB
- **Fault isolation**: Budget Service crashes, User login still works
- **Team ownership**: Each team owns their service end-to-end

**Microservices disadvantages:**
- Distributed system complexity (network failures, latency)
- Data consistency harder (no shared database = no ACID across services)
- Operational overhead (many services to monitor, deploy, secure)
- Debugging harder (request spans many services)

## 16.2 Expense Tracker Microservices Architecture

```
                    ┌────────────────────────┐
     Client  ──────►│      API Gateway       │ :8080
                    │  (Spring Cloud GW)     │
                    │  Auth, routing,        │
                    │  rate limiting, CORS   │
                    └──────────┬─────────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                       │
┌───────▼────────┐   ┌─────────▼──────┐   ┌───────────▼────────┐
│  User Service  │   │Expense Service │   │  Budget Service    │
│  :8081         │   │  :8082         │   │  :8083             │
│  MySQL         │   │  PostgreSQL    │   │  MongoDB           │
│  - register    │   │  - create      │   │  - set budget      │
│  - login       │   │  - list/filter │   │  - check limits    │
│  - profile     │   │  - analytics   │   │  - alert thresholds│
└────────────────┘   └────────────────┘   └────────────────────┘
        │                      │                       │
        └──────────────────────┼───────────────────────┘
                               │ Events (Kafka)
                    ┌──────────▼─────────────┐
                    │   Notification Service  │ :8084
                    │   Email, Push, SMS      │
                    └─────────────────────────┘

Infrastructure:
  ┌──────────────────────────────────────────┐
  │  Eureka Server    :8761  (discovery)     │
  │  Config Server    :8888  (config)        │
  │  Zipkin           :9411  (tracing)       │
  │  Kafka            :9092  (events)        │
  └──────────────────────────────────────────┘
```

---

# 17. Eureka Service Discovery

## 17.1 The Problem

In microservices, services call each other. Without discovery:

```yaml
# expense-service config — FRAGILE
user-service.url: http://10.0.1.15:8081  # what if IP changes?
                                           # what if 3 User Service instances run?
```

With Eureka, services register themselves by name. Callers ask "give me an instance of `user-service`" and Eureka provides a healthy one.

## 17.2 Eureka Architecture

```
┌─────────────────────────────────┐
│        Eureka Server            │  "Phone book" — knows every service
│         :8761                   │
└─────────┬───────────────────────┘
          │
  ①Register (on startup)
  ②Heartbeat (every 30s)
  ③Lookup (when caller needs another service)
          │
┌─────────┴─────────────────────┐
│                               │
▼                               ▼
┌────────────────┐   ┌──────────────────────┐
│  User Service  │   │   Expense Service    │
│  :8081 x3      │   │   :8082              │
│ (3 instances)  │   │  calls user-service  │
└────────────────┘   └──────────────────────┘
```

## 17.3 Eureka Server Setup

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceRegistryApplication.class, args);
    }
}
```

```yaml
# application.yml — Eureka Server
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false  # Don't register with itself
    fetch-registry: false         # Don't fetch its own registry
  server:
    wait-time-in-ms-when-sync-empty: 0  # Dev: don't wait for peers
```

## 17.4 Eureka Client (each microservice)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
# application.yml — expense-service
server:
  port: 8082

spring:
  application:
    name: expense-service  # ← Service is registered under this name

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true                    # register IP, not hostname
    lease-renewal-interval-in-seconds: 30      # heartbeat every 30s
    lease-expiration-duration-in-seconds: 90   # remove if no heartbeat for 90s
    instance-id: ${spring.application.name}:${server.port}:${random.value}
```

## 17.5 OpenFeign — Declarative HTTP Client

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients
public class ExpenseServiceApplication { }

// Declare interface — Feign generates implementation; Eureka resolves the URL
@FeignClient(
    name = "user-service",         // matches spring.application.name
    fallback = UserServiceFallback.class  // circuit breaker fallback
)
public interface UserServiceClient {

    @GetMapping("/api/v1/users/{id}")
    UserDTO getUserById(@PathVariable Long id);

    @PostMapping("/api/v1/users/validate")
    boolean validateUser(@RequestBody ValidateUserRequest request);

    @GetMapping("/api/v1/users/{id}/exists")
    boolean exists(@PathVariable Long id);
}

// Fallback — returned when user-service is down
@Component
public class UserServiceFallback implements UserServiceClient {
    @Override
    public UserDTO getUserById(Long id) {
        log.warn("User service unavailable — returning stub for id {}", id);
        return new UserDTO(id, "Unknown", "unknown@example.com");
    }

    @Override
    public boolean validateUser(ValidateUserRequest req) { return false; }

    @Override
    public boolean exists(Long id) { return false; }
}

// Usage in Expense Service
@Service
public class ExpenseService {
    private final UserServiceClient userClient;

    public ExpenseDTO createExpense(CreateExpenseRequest req, Long userId) {
        // Feign handles: URL resolution via Eureka, load balancing, serialization
        UserDTO user = userClient.getUserById(userId);
        // ...
    }
}
```

---

# 18. Distributed Tracing: Sleuth & Zipkin

## 18.1 The Problem

A request through your system might visit 5 services. If it's slow or fails, which service caused it?

```
GET /api/v1/expenses  (500ms total — too slow!)
         │
   API Gateway (5ms)
         │
   Expense Service (480ms ← is this the problem?)
         │
   User Service (10ms)  ←  or this?
         │
   Budget Service (5ms)
```

Without tracing, you check 5 different log files manually.

## 18.2 Trace ID + Span ID

```
ONE user request gets ONE trace ID.
Each service hop creates a NEW span ID.

Request: GET /api/v1/expenses
                          TraceID     SpanID   ParentSpan
API Gateway:              abc-123     0001     —
Expense Service:          abc-123     0002     0001
User Service:             abc-123     0003     0002
Budget Service:           abc-123     0004     0002

All logs with traceId=abc-123 → complete picture of one request's journey
```

## 18.3 Configuration (Spring Boot 3 / Micrometer Tracing)

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

```yaml
management:
  tracing:
    sampling:
      probability: 1.0  # 100% in dev; use 0.05-0.1 in prod (5-10%)

spring:
  zipkin:
    base-url: http://localhost:9411

logging:
  pattern:
    console: "%d{HH:mm:ss} [%X{traceId}/%X{spanId}] %-5level %logger{20} - %msg%n"
    # Output: 10:30:00 [abc123/0002] INFO  ExpenseService - Creating expense
```

**Start Zipkin:**
```bash
docker run -d -p 9411:9411 openzipkin/zipkin
# UI available at: http://localhost:9411
```

Zipkin UI shows:
- Timeline waterfall: which service called which, in what order
- Duration per span: identify the bottleneck
- Error highlighting: which span failed and why

---

# 19. Spring Boot Profiles

## 19.1 Why Profiles?

Same code runs differently in dev, QA, and production:

| Concern | Dev | QA | Prod |
|---------|-----|----|------|
| Database | H2 in-memory | Real PostgreSQL | RDS PostgreSQL |
| Email | Log to console | Send to test inbox | Send real emails |
| Log level | DEBUG | INFO | WARN |
| Cache TTL | 1 min | 15 min | 60 min |
| Seed data | Yes | Yes | Never |

## 19.2 Profile Config Files

```
src/main/resources/
├── application.yml          ← base config shared by all profiles
├── application-dev.yml      ← dev overrides
├── application-qa.yml       ← QA overrides
└── application-prod.yml     ← prod overrides
```

```yaml
# application.yml — base (shared)
spring:
  application:
    name: expense-service
  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: validate

server:
  port: 8082

---
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:expensedb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  h2:
    console:
      enabled: true

logging:
  level:
    com.expensetracker: DEBUG
    org.hibernate.SQL: DEBUG

---
# application-prod.yml
spring:
  datasource:
    url: ${DATABASE_URL}          # from environment variable
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000

logging:
  level:
    root: WARN
    com.expensetracker: INFO

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
```

## 19.3 Activating Profiles

```bash
# Command line
java -jar expense-service.jar --spring.profiles.active=prod

# Environment variable (Docker/Kubernetes friendly)
SPRING_PROFILES_ACTIVE=prod java -jar expense-service.jar

# In application.yml for development default
spring:
  profiles:
    active: dev
```

## 19.4 Profile-Specific Beans

```java
// Only in dev and qa — seeds test data
@Component
@Profile({"dev", "qa"})
public class TestDataSeeder implements CommandLineRunner {
    @Override
    public void run(String... args) {
        userRepository.save(User.builder()
            .email("demo@expensetracker.com").passwordHash(encoder.encode("demo123"))
            .role(UserRole.USER).build());
        // ... seed expenses, budgets, etc.
    }
}

// Two implementations of same interface, activated by profile
@Service
@Profile("prod")
public class SmtpEmailService implements EmailService {
    @Value("${smtp.host}") private String host;
    public void send(String to, String subject, String body) { /* real SMTP */ }
}

@Service
@Profile("!prod")
public class ConsoleEmailService implements EmailService {
    public void send(String to, String subject, String body) {
        log.info("=== MOCK EMAIL ===\nTo: {}\nSubject: {}\n{}\n=================",
                  to, subject, body);
    }
}
```

---

# 20. Spring Cloud Config Server

## 20.1 The Problem Without Config Server

10 microservices × 3 environments = 30 config files scattered across repos. To change a database URL:
1. Update 10 config files
2. Restart 10 services (or redeploy)
3. Risk inconsistency (forgot to update 2 services)

## 20.2 Architecture

```
Git Repository (config-repo)
├── application.yml               ← shared by ALL services
├── expense-service.yml           ← expense-service specific
├── expense-service-prod.yml      ← expense-service in prod
├── user-service.yml
└── user-service-prod.yml
         │
         ▼ Config Server reads from Git
┌──────────────────────────────┐
│     Config Server  :8888     │  Serves config over HTTP
└──────────┬───────────────────┘
           │ Each service fetches its config at startup
    ┌──────┼──────┐
    ▼      ▼      ▼
User Svc  Expense Svc  Budget Svc
```

## 20.3 Config Server Setup

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
# application.yml — Config Server
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/expense-tracker-config
          default-label: main
          clone-on-start: true
          search-paths: "{application}"  # look in subdirectory named after service
          # For private repo:
          # username: ${GIT_USERNAME}
          # password: ${GIT_TOKEN}
```

## 20.4 Client Configuration

```yaml
# application.yml in each microservice
spring:
  application:
    name: expense-service  # must match filename in config repo
  config:
    import: "optional:configserver:http://localhost:8888"
  cloud:
    config:
      fail-fast: true      # fail immediately if config server unavailable
      retry:
        max-attempts: 6
        initial-interval: 1500ms
        max-interval: 2000ms
```

## 20.5 Refreshing Config Without Restart

```bash
# 1. Update config in Git and push
# 2. Trigger refresh on specific service instance
POST http://expense-service:8082/actuator/refresh

# 3. Or refresh all services at once via Spring Cloud Bus + Kafka:
POST http://config-server:8888/actuator/busrefresh
```

```java
// Beans that need to pick up refreshed config
@RefreshScope  // bean is re-created when /actuator/refresh is called
@Service
public class JwtService {
    @Value("${jwt.secret}")
    private String secret;  // picks up new value after refresh
}
```

---

# 21. Inter-Service Communication

## 21.1 Synchronous (REST/HTTP)

Use when you need an **immediate response**:
- Create expense → validate user exists → return expense DTO
- Login → validate credentials → return JWT

```java
// OpenFeign with Circuit Breaker + Retry
@FeignClient(name = "budget-service")
public interface BudgetServiceClient {

    @GetMapping("/api/v1/budgets/{userId}/remaining")
    BigDecimal getRemainingBudget(@PathVariable Long userId);

    @PostMapping("/api/v1/budgets/{userId}/deduct")
    BudgetDTO deductAmount(@PathVariable Long userId, @RequestBody DeductRequest request);
}
```

**Feign configuration:**

```yaml
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 10000
        loggerLevel: BASIC
  circuitbreaker:
    enabled: true
```

## 21.2 Asynchronous (Messaging)

Use when you **don't need an immediate response** or want **loose coupling**:
- Expense created → send email notification (don't hold request)
- Expense created → recalculate monthly analytics
- User activity → audit log

```
SYNC (REST):
ExpenseService ──── HTTP ────► UserService
               ◄─── HTTP ────  (waits here, blocking thread)

ASYNC (Kafka):
ExpenseService ──── publish "expense.created" ────► Kafka
               returns immediately                      │
                                                        ▼ (whenever ready)
                                               NotificationService
                                               AnalyticsService
                                               AuditService
```

**When to choose which:**

| Use Sync When | Use Async When |
|---|---|
| Need result to continue | Fire-and-forget |
| Strong consistency needed | Eventual consistency OK |
| Simple request-response | Fan-out to multiple consumers |
| Real-time user feedback | Background processing |

---

# 22. API Gateway

## 22.1 Why API Gateway?

Without gateway:
- Client must know 10 different service URLs
- Auth logic duplicated in every service
- No centralized rate limiting, CORS, SSL termination

With gateway:
- Single entry point for all clients
- Centralized: auth, rate limiting, logging, CORS, SSL, request/response transformation

## 22.2 Spring Cloud Gateway Setup

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```yaml
# application.yml — API Gateway
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true               # auto-discover routes from Eureka
          lower-case-service-id: true
      routes:
        - id: user-service
          uri: lb://user-service       # lb:// = load-balanced via Eureka
          predicates:
            - Path=/api/v1/users/**
          filters:
            - StripPrefix=0

        - id: expense-service
          uri: lb://expense-service
          predicates:
            - Path=/api/v1/expenses/**
          filters:
            - StripPrefix=0
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200

        - id: auth-endpoints
          uri: lb://user-service
          predicates:
            - Path=/api/v1/auth/**
          filters:
            - StripPrefix=0
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10   # strict rate limit on auth
                redis-rate-limiter.burstCapacity: 20
```

## 22.3 Custom Global Auth Filter

```java
@Component
@Slf4j
public class JwtGatewayFilter implements GlobalFilter, Ordered {

    private final JwtService jwtService;

    // These paths don't require auth
    private static final List<String> PUBLIC_PATHS = List.of(
        "/api/v1/auth/login",
        "/api/v1/auth/register",
        "/api/v1/auth/refresh",
        "/actuator/health"
    );

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String path = exchange.getRequest().getURI().getPath();

        // Skip auth for public endpoints
        if (PUBLIC_PATHS.stream().anyMatch(path::startsWith)) {
            return chain.filter(exchange);
        }

        String authHeader = exchange.getRequest().getHeaders()
            .getFirst(HttpHeaders.AUTHORIZATION);

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return unauthorized(exchange, "Missing Authorization header");
        }

        String token = authHeader.substring(7);
        try {
            String userId = jwtService.extractUserId(token);
            String userRole = jwtService.extractRole(token);

            // Forward validated user context to downstream services
            ServerHttpRequest mutated = exchange.getRequest().mutate()
                .header("X-User-Id", userId)
                .header("X-User-Role", userRole)
                .build();

            return chain.filter(exchange.mutate().request(mutated).build());

        } catch (JwtException e) {
            log.warn("Invalid JWT token: {}", e.getMessage());
            return unauthorized(exchange, "Invalid token");
        }
    }

    private Mono<Void> unauthorized(ServerWebExchange exchange, String message) {
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        exchange.getResponse().getHeaders().setContentType(MediaType.APPLICATION_JSON);
        byte[] body = ("{\"error\":\"" + message + "\"}").getBytes();
        DataBuffer buf = exchange.getResponse().bufferFactory().wrap(body);
        return exchange.getResponse().writeWith(Mono.just(buf));
    }

    @Override
    public int getOrder() { return -1; } // highest priority
}
```

---

# 23. Circuit Breaker

## 23.1 Cascading Failures

```
User Service: response time = 5000ms (database overloaded)
      │
Expense Service: 500 threads waiting for User Service
      │
All threads exhausted → Expense Service stops responding
      │
API Gateway: can't reach Expense Service → returns 503
      │
All users see failures
```

The circuit breaker **fails fast** when a service is struggling, returning a fallback instead of piling up waiting threads.

## 23.2 Circuit Breaker States

```
                    ┌─ failure rate > threshold ──────► OPEN
                    │                                       │
               CLOSED                                 wait timeout
               (normal — all requests pass)                 │
                    │                                       ▼
                    └────── success ──────── HALF-OPEN (probe one request)
                                                  │              │
                                               success         failure
                                                  │              │
                                              → CLOSED       → OPEN
```

- **CLOSED**: All requests pass through normally
- **OPEN**: All requests immediately fail with fallback — no real calls to failing service
- **HALF-OPEN**: One probe request allowed to test recovery

## 23.3 Resilience4j

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      user-service:
        registerHealthIndicator: true
        slidingWindowSize: 10              # last 10 calls
        minimumNumberOfCalls: 5            # need at least 5 before evaluating
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 10s       # stay OPEN 10s before probing
        failureRateThreshold: 50           # 50%+ failures → OPEN
        slowCallDurationThreshold: 2s      # calls > 2s count as failures
        slowCallRateThreshold: 50          # 50%+ slow calls → OPEN

  retry:
    instances:
      user-service:
        maxAttempts: 3
        waitDuration: 500ms
        retryExceptions:
          - feign.FeignException.ServiceUnavailable
          - java.net.ConnectException

  timelimiter:
    instances:
      user-service:
        timeoutDuration: 3s
        cancelRunningFuture: true
```

```java
@Service
@Slf4j
public class ExpenseService {
    private final UserServiceClient userClient;
    private final UserCacheService userCache;

    @CircuitBreaker(name = "user-service", fallbackMethod = "getUserFallback")
    @Retry(name = "user-service")
    @TimeLimiter(name = "user-service")
    public CompletableFuture<UserDTO> getUserAsync(Long userId) {
        return CompletableFuture.supplyAsync(() -> userClient.getUserById(userId));
    }

    // Called when circuit is OPEN or request fails after retries
    // Must have same signature + Throwable as last param
    public CompletableFuture<UserDTO> getUserFallback(Long userId, Throwable t) {
        log.warn("User service unavailable for userId {}: {}", userId, t.getMessage());

        return CompletableFuture.supplyAsync(() ->
            userCache.getCached(userId)
                .orElse(new UserDTO(userId, "Unknown User", null))
        );
    }
}

// Monitor circuit breaker state
@EventListener
public void onCircuitBreakerStateChange(CircuitBreakerOnStateTransitionEvent event) {
    log.warn("Circuit breaker '{}' changed from {} to {}",
        event.getCircuitBreakerName(),
        event.getStateTransition().getFromState(),
        event.getStateTransition().getToState()
    );
    // Alert ops team if circuit opens
    if (event.getStateTransition().getToState() == CircuitBreaker.State.OPEN) {
        alertService.sendAlert("Circuit breaker OPEN: " + event.getCircuitBreakerName());
    }
}
```

---

# 24. CQRS Pattern

## 24.1 What Is CQRS?

Command Query Responsibility Segregation: **write operations** and **read operations** use **separate models**.

```
Traditional:
Client → ExpenseService → Single DB (same model for read and write)

CQRS:
Client ─── Command ──► CommandHandler → Write DB (normalized, ACID)
       └── Query ────► QueryHandler  → Read DB  (denormalized, fast)
```

## 24.2 Why CQRS in Expense Tracker?

**Write model needs:**
- Normalized tables (no duplication)
- ACID transactions (expense + budget updated atomically)
- Business rule validation

**Read model (analytics) needs:**
- Denormalized data (pre-joined, pre-aggregated)
- Fast queries with no joins
- Possibly different database (ClickHouse for analytics, MongoDB for flexible queries)

## 24.3 Implementation

```java
// ===== COMMAND SIDE (writes) =====

// Commands are immutable value objects describing intent
public record CreateExpenseCommand(
    Long userId, String description, BigDecimal amount,
    ExpenseCategory category, LocalDate expenseDate) {}

public record DeleteExpenseCommand(Long expenseId, Long userId) {}

// Command handlers contain business logic + emit events
@Service
@Transactional
public class ExpenseCommandHandler {
    private final ExpenseRepository expenseRepo;
    private final UserRepository userRepo;
    private final ApplicationEventPublisher eventPublisher;

    public Long handle(CreateExpenseCommand cmd) {
        User user = userRepo.findById(cmd.userId())
            .orElseThrow(() -> new ResourceNotFoundException("User", cmd.userId()));

        Expense expense = Expense.builder()
            .description(cmd.description()).amount(cmd.amount())
            .category(cmd.category()).expenseDate(cmd.expenseDate())
            .user(user).build();

        expense = expenseRepo.save(expense);

        // Publish domain event — read model will update itself
        eventPublisher.publishEvent(new ExpenseCreatedEvent(
            expense.getId(), cmd.userId(), cmd.amount(),
            cmd.category().name(), cmd.expenseDate(), Instant.now()));

        return expense.getId();
    }

    public void handle(DeleteExpenseCommand cmd) {
        Expense expense = expenseRepo.findById(cmd.expenseId())
            .orElseThrow(() -> new ResourceNotFoundException("Expense", cmd.expenseId()));

        if (!expense.getUser().getId().equals(cmd.userId()))
            throw new AccessDeniedException("Not your expense");

        expenseRepo.delete(expense);
        eventPublisher.publishEvent(new ExpenseDeletedEvent(
            cmd.expenseId(), cmd.userId(), expense.getAmount(),
            expense.getCategory().name(), Instant.now()));
    }
}

// ===== QUERY SIDE (reads) =====

// Read model — denormalized, optimized for queries
@Document(collection = "expense_summary_views")
@Getter @Setter
public class ExpenseSummaryView {
    @Id private String id;           // userId:yearMonth
    private Long userId;
    private String yearMonth;        // "2024-01"
    private BigDecimal totalAmount;
    private int expenseCount;
    private Map<String, BigDecimal> byCategory;  // pre-aggregated
    private BigDecimal largestExpense;
    private BigDecimal averageExpense;
    private LocalDateTime lastUpdated;
}

// Query objects
public record GetMonthlySummaryQuery(Long userId, YearMonth yearMonth) {}
public record GetExpensesByDateRangeQuery(Long userId, LocalDate from, LocalDate to,
                                           int page, int size) {}

// Query handlers — only read, never write
@Service
public class ExpenseQueryHandler {
    private final ExpenseSummaryRepository summaryRepo;

    public ExpenseSummaryView handle(GetMonthlySummaryQuery query) {
        String id = query.userId() + ":" + query.yearMonth();
        return summaryRepo.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Summary", query.userId()));
    }
}

// Projection — listens to events and updates read model
@Service
@Slf4j
public class ExpenseSummaryProjection {
    private final ExpenseSummaryRepository summaryRepo;

    @EventListener
    @Async
    public void on(ExpenseCreatedEvent event) {
        log.debug("Updating read model for event: expense {} created", event.expenseId());

        String id = event.userId() + ":" + event.expenseDate().withDayOfMonth(1);
        ExpenseSummaryView summary = summaryRepo.findById(id)
            .orElseGet(() -> {
                ExpenseSummaryView s = new ExpenseSummaryView();
                s.setId(id);
                s.setUserId(event.userId());
                s.setYearMonth(event.expenseDate().toString().substring(0, 7));
                s.setByCategory(new HashMap<>());
                s.setTotalAmount(BigDecimal.ZERO);
                return s;
            });

        summary.setTotalAmount(summary.getTotalAmount().add(event.amount()));
        summary.setExpenseCount(summary.getExpenseCount() + 1);
        summary.getByCategory().merge(event.category(), event.amount(), BigDecimal::add);
        summary.setAverageExpense(
            summary.getTotalAmount().divide(
                BigDecimal.valueOf(summary.getExpenseCount()), 2, RoundingMode.HALF_UP));
        summary.setLastUpdated(LocalDateTime.now());

        summaryRepo.save(summary);
    }

    @EventListener
    @Async
    public void on(ExpenseDeletedEvent event) {
        // Update read model on delete
        String id = event.userId() + ":" + event.occurredAt().toString().substring(0, 7);
        summaryRepo.findById(id).ifPresent(summary -> {
            summary.setTotalAmount(summary.getTotalAmount().subtract(event.amount()));
            summary.setExpenseCount(summary.getExpenseCount() - 1);
            summary.getByCategory().computeIfPresent(event.category(),
                (k, v) -> v.subtract(event.amount()));
            summaryRepo.save(summary);
        });
    }
}

// Controller using both sides
@RestController
@RequestMapping("/api/v1/expenses")
public class ExpenseController {
    private final ExpenseCommandHandler commandHandler;
    private final ExpenseQueryHandler queryHandler;

    @PostMapping
    public ResponseEntity<Long> create(@RequestBody @Valid CreateExpenseRequest req,
                                        @RequestHeader("X-User-Id") Long userId) {
        Long id = commandHandler.handle(new CreateExpenseCommand(
            userId, req.description(), req.amount(), req.category(), req.expenseDate()));
        return ResponseEntity.created(URI.create("/api/v1/expenses/" + id)).body(id);
    }

    @GetMapping("/analytics/monthly")
    public ResponseEntity<ExpenseSummaryView> monthly(
            @RequestHeader("X-User-Id") Long userId,
            @RequestParam String yearMonth) {
        return ResponseEntity.ok(queryHandler.handle(
            new GetMonthlySummaryQuery(userId, YearMonth.parse(yearMonth))));
    }
}
```


# 25. Spring AOP, Proxies & CGLIB

## 25.1 What Is AOP?

AOP (Aspect-Oriented Programming) separates **cross-cutting concerns** from business logic.

Cross-cutting concerns appear in every layer but shouldn't be mixed with business logic:
- Logging every method entry/exit
- Security checks
- Performance monitoring
- Transaction management
- Caching

Without AOP: copy-paste the same logging/security code in 100 methods.
With AOP: write it once, apply declaratively with annotations.

## 25.2 AOP Terminology

```
Aspect    — The "what": the cross-cutting concern (e.g., "logging")
Advice    — The "when": Before, After, Around the method
Pointcut  — The "where": which methods to intercept (expression)
JoinPoint — The actual moment of execution (the method being intercepted)
Weaving   — Process of applying aspects to target objects
```

## 25.3 Spring Proxies and CGLIB — The Mechanism

When you annotate a class with `@Transactional` or `@Cacheable`, Spring does NOT modify your class bytecode. Instead it creates a **proxy** — a wrapper object that intercepts method calls.

```
You declare:  @Autowired ExpenseService expenseService;

What you get: ExpenseService$$SpringCGLIB$$0 (proxy)
              ┌──────────────────────────────────────────────┐
              │  createExpense(...) {                        │
              │    // BEFORE advice                          │
              │    transactionManager.begin();               │
              │    try {                                     │
              │      result = realService.createExpense();   │ ← your code
              │      transactionManager.commit();            │
              │    } catch (RuntimeException e) {           │
              │      transactionManager.rollback();          │
              │      throw e;                                │
              │    }                                         │
              │  }                                           │
              └──────────────────────────────────────────────┘
```

**CGLIB**: Creates runtime subclass of your class (no interface needed).
**JDK Proxy**: Creates object implementing same interfaces (requires interface).

### Critical Self-Invocation Limitation

```java
@Service
public class ExpenseService {

    @Transactional
    public void methodA() {
        this.methodB(); // PROBLEM! Calls real object, bypasses proxy
                        // @Transactional on methodB() has NO effect
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void methodB() {
        // This transaction annotation is IGNORED when called via this.methodB()
    }
}

// Fix: inject the proxy (self-injection)
@Service
public class ExpenseService {
    @Autowired
    private ExpenseService self; // Spring injects the proxy

    public void methodA() {
        self.methodB(); // Now calls through proxy — @Transactional works
    }
}
```

## 25.4 Custom AOP Aspects

```java
// Performance monitoring aspect
@Aspect
@Component
@Slf4j
public class PerformanceAspect {

    // Pointcut: all public methods in service classes
    @Pointcut("execution(public * com.expensetracker.*.service.*Service.*(..))")
    public void serviceLayerMethods() {}

    // Around: runs before AND after
    @Around("serviceLayerMethods()")
    public Object monitorPerformance(ProceedingJoinPoint pjp) throws Throwable {
        String className  = pjp.getTarget().getClass().getSimpleName();
        String methodName = pjp.getSignature().getName();
        long start = System.currentTimeMillis();

        log.debug("→ {}.{}", className, methodName);
        try {
            Object result = pjp.proceed(); // execute actual method
            long ms = System.currentTimeMillis() - start;
            log.debug("← {}.{} completed in {}ms", className, methodName, ms);

            if (ms > 500)
                log.warn("SLOW: {}.{} took {}ms", className, methodName, ms);

            return result;
        } catch (Exception e) {
            long ms = System.currentTimeMillis() - start;
            log.error("{}.{} FAILED after {}ms: {}", className, methodName, ms, e.getMessage());
            throw e;
        }
    }
}

// Audit logging aspect
@Aspect
@Component
@Slf4j
public class AuditAspect {
    private final AuditLogRepository auditRepo;

    // Intercept all methods annotated with @Audited
    @Around("@annotation(audited)")
    public Object audit(ProceedingJoinPoint pjp, Audited audited) throws Throwable {
        String username = SecurityContextHolder.getContext()
            .getAuthentication().getName();

        Object result = pjp.proceed();

        auditRepo.save(AuditLog.builder()
            .username(username)
            .action(audited.action())
            .resource(audited.resource())
            .timestamp(LocalDateTime.now())
            .build());

        return result;
    }
}

// @Before — runs before method, can't modify return value
@Aspect
@Component
public class ValidationAspect {
    @Before("execution(* com.expensetracker.*.service.*Service.create*(..))")
    public void logCreation(JoinPoint jp) {
        log.info("Creating resource via: {}", jp.getSignature().getName());
    }
}

// @AfterReturning — runs after method succeeds, can inspect return value
@Aspect
@Component
public class CacheUpdateAspect {
    @AfterReturning(
        pointcut = "execution(* com.expensetracker.expense.service.ExpenseService.createExpense(..))",
        returning = "result")
    public void onExpenseCreated(Object result) {
        if (result instanceof ExpenseDTO dto) {
            log.debug("Expense created: {}", dto.id());
        }
    }
}

// @AfterThrowing — runs when method throws
@Aspect
@Component
public class ErrorTrackingAspect {
    @AfterThrowing(
        pointcut = "execution(* com.expensetracker..*Service.*(..))",
        throwing = "ex")
    public void trackError(JoinPoint jp, Exception ex) {
        errorTrackingService.record(jp.getSignature().toString(), ex);
    }
}
```

**Custom annotation for AOP:**

```java
// Custom annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Audited {
    String action();
    String resource() default "";
}

// Usage — clean, declarative
@Service
public class ExpenseService {

    @Audited(action = "CREATE", resource = "EXPENSE")
    @Transactional
    public ExpenseDTO createExpense(CreateExpenseRequest req, Long userId) {
        // audit aspect fires automatically — no manual audit code here
        return mapper.toDTO(expenseRepository.save(Expense.from(req)));
    }

    @Audited(action = "DELETE", resource = "EXPENSE")
    @Transactional
    public void deleteExpense(Long id, Long userId) {
        expenseRepository.deleteById(id);
    }
}
```

## 25.5 @EnableAspectJAutoProxy

```java
@SpringBootApplication
@EnableAspectJAutoProxy  // enables AOP proxy creation (usually auto-enabled in Spring Boot)
public class ExpenseTrackerApplication { }
```

---

# 26. Kafka & RabbitMQ

## 26.1 Why Event Streaming?

When a user creates an expense, synchronously calling 4 services is slow and fragile:

```
SYNC (bad for this use case):
ExpenseService ─► BudgetService    (waits 50ms)
               ─► NotificationSvc  (waits 200ms) ← email takes time
               ─► AnalyticsSvc     (waits 100ms)
               ─► AuditSvc         (waits 30ms)
Total: 380ms blocking the user's request thread
```

```
ASYNC with Kafka (good):
ExpenseService ─► publish "expense.created" ─► Kafka Topic
               returns in 5ms

Meanwhile (in parallel, independently):
BudgetService     ─ consumes ─► updates budget
NotificationSvc   ─ consumes ─► sends email
AnalyticsSvc      ─ consumes ─► updates analytics
AuditSvc          ─ consumes ─► logs audit event
```

## 26.2 Kafka vs RabbitMQ

| Aspect | Kafka | RabbitMQ |
|--------|-------|----------|
| Model | Log/Topic (events stored) | Queue (consumed + deleted) |
| Message replay | Yes — re-read old events | No |
| Throughput | Very high (millions/sec) | High (thousands/sec) |
| Order guarantee | Per partition | Per queue (with constraints) |
| Best for | Event sourcing, activity tracking, analytics streams | Task queues, notifications, RPC |
| Retention | Configurable (days/forever) | Until consumed (or TTL) |

**In Expense Tracker:**
- **Kafka**: Real-time activity tracking, audit events, analytics pipeline
- **RabbitMQ**: Email/push notifications (simple task queue)

## 26.3 Apache Kafka Integration

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all          # wait for all replicas to acknowledge — durable
      retries: 3
      properties:
        enable.idempotence: true  # prevent duplicate messages on retry
    consumer:
      group-id: expense-tracker-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      auto-offset-reset: earliest
      enable-auto-commit: false   # manual commit for at-least-once guarantee
      properties:
        spring.json.trusted.packages: "com.expensetracker.*"
```

**Domain Events:**

```java
public record ExpenseCreatedEvent(
    Long expenseId, Long userId, BigDecimal amount,
    String category, LocalDate expenseDate, Instant occurredAt) {}

public record UserActivityEvent(
    Long userId, String action, String resourceType,
    Long resourceId, Map<String, Object> metadata, Instant occurredAt) {}
```

**Kafka Producer:**

```java
@Service
@Slf4j
public class ExpenseEventPublisher {
    private final KafkaTemplate<String, Object> kafka;

    private static final String EXPENSE_EVENTS  = "expense-events";
    private static final String ACTIVITY_EVENTS = "user-activity";

    public void publishExpenseCreated(Expense expense) {
        ExpenseCreatedEvent event = new ExpenseCreatedEvent(
            expense.getId(), expense.getUser().getId(),
            expense.getAmount(), expense.getCategory().name(),
            expense.getExpenseDate(), Instant.now());

        // Key = userId → all events for one user go to same partition (ordering)
        kafka.send(EXPENSE_EVENTS, expense.getUser().getId().toString(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Failed to publish ExpenseCreatedEvent for expense {}: {}",
                        expense.getId(), ex.getMessage());
                } else {
                    log.debug("Published expense {} to partition {} offset {}",
                        expense.getId(),
                        result.getRecordMetadata().partition(),
                        result.getRecordMetadata().offset());
                }
            });
    }

    public void publishUserActivity(Long userId, String action, String type, Long resourceId) {
        UserActivityEvent event = new UserActivityEvent(
            userId, action, type, resourceId, Map.of(), Instant.now());
        kafka.send(ACTIVITY_EVENTS, userId.toString(), event);
    }
}
```

**Kafka Consumer:**

```java
@Service
@Slf4j
public class ExpenseEventConsumer {
    private final NotificationService notificationService;
    private final BudgetService budgetService;

    @KafkaListener(
        topics = "expense-events",
        groupId = "notification-group",
        concurrency = "3"   // 3 consumer threads in parallel
    )
    public void onExpenseCreated(
            ConsumerRecord<String, ExpenseCreatedEvent> record,
            Acknowledgment ack) {

        ExpenseCreatedEvent event = record.value();
        log.info("Processing expense.created for user {} expense {}",
            event.userId(), event.expenseId());

        try {
            notificationService.sendExpenseConfirmation(event);
            budgetService.deductFromBudget(event.userId(), event.amount());
            ack.acknowledge(); // manual commit ONLY after successful processing
        } catch (Exception e) {
            log.error("Failed to process expense.created event {}: {}",
                event.expenseId(), e.getMessage());
            // don't ack → message will be redelivered
            throw e;
        }
    }
}

// Kafka error handling configuration
@Configuration
public class KafkaConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object>
            kafkaListenerContainerFactory(ConsumerFactory<String, Object> cf) {

        ConcurrentKafkaListenerContainerFactory<String, Object> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(cf);
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);

        // Retry 3 times with 1s delay; then send to dead-letter topic
        factory.setCommonErrorHandler(new DefaultErrorHandler(
            new DeadLetterPublishingRecoverer(kafkaTemplate),
            new FixedBackOff(1000L, 3L)
        ));
        return factory;
    }
}
```

## 26.4 RabbitMQ Integration

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    listener:
      simple:
        acknowledge-mode: manual
        retry:
          enabled: true
          max-attempts: 3
          initial-interval: 1s
```

```java
@Configuration
public class RabbitMQConfig {

    // Queue + DLQ (Dead Letter Queue) for failed messages
    @Bean
    public Queue notificationQueue() {
        return QueueBuilder.durable("notification.queue")
            .withArgument("x-dead-letter-exchange", "")
            .withArgument("x-dead-letter-routing-key", "notification.dlq")
            .withArgument("x-message-ttl", 300000) // 5-minute TTL
            .build();
    }

    @Bean Queue deadLetterQueue() { return new Queue("notification.dlq", true); }

    @Bean
    public DirectExchange notificationExchange() {
        return new DirectExchange("notification.exchange");
    }

    @Bean
    public Binding notificationBinding(Queue notificationQueue,
                                        DirectExchange notificationExchange) {
        return BindingBuilder.bind(notificationQueue)
            .to(notificationExchange).with("notification.email");
    }

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory cf) {
        RabbitTemplate tmpl = new RabbitTemplate(cf);
        tmpl.setMessageConverter(messageConverter());
        tmpl.setConfirmCallback((correlationData, ack, cause) -> {
            if (!ack) log.error("Message not confirmed: {}", cause);
        });
        return tmpl;
    }
}

// Publisher
@Service
public class NotificationPublisher {
    private final RabbitTemplate rabbitTemplate;

    public void queueEmailNotification(EmailNotificationRequest req) {
        rabbitTemplate.convertAndSend(
            "notification.exchange", "notification.email", req);
        log.debug("Queued email to: {}", req.to());
    }
}

// Consumer
@Service
@Slf4j
public class EmailNotificationConsumer {

    @RabbitListener(queues = "notification.queue")
    public void handleEmail(EmailNotificationRequest req, Channel channel,
                             @Header(AmqpHeaders.DELIVERY_TAG) long tag)
            throws IOException {
        try {
            log.info("Sending email to: {}", req.to());
            emailService.send(req.to(), req.subject(), req.body());
            channel.basicAck(tag, false); // acknowledge success
        } catch (Exception e) {
            log.error("Failed to send email to {}: {}", req.to(), e.getMessage());
            channel.basicNack(tag, false, false); // reject → routes to DLQ
        }
    }
}
```

---

# 27. Expense Tracker Project Design

## 27.1 High-Level Design (HLD)

```
System: Expense tracking with budgets, analytics, and notifications

Functional requirements:
  ✓ User registration and authentication (JWT)
  ✓ Create, read, update, delete expenses
  ✓ Filter expenses by date, category, amount
  ✓ Set monthly budgets per category
  ✓ Budget alerts (email when 80% spent)
  ✓ Monthly/yearly analytics reports
  ✓ Real-time activity tracking
  ✓ Export expenses (CSV/PDF)

Non-functional requirements:
  ✓ 99.9% uptime
  ✓ <200ms p95 API response
  ✓ 10,000 concurrent users
  ✓ Data encrypted at rest and in transit
```

## 27.2 Service Breakdown

```
User Service (MySQL)
  ├── Register, login, logout
  ├── JWT access + refresh tokens
  ├── User profile management
  └── Role management (USER, ADMIN)

Expense Service (PostgreSQL)
  ├── CRUD expenses
  ├── Pagination + filtering
  ├── Bulk CSV import
  └── Publishes events to Kafka

Budget Service (MongoDB)
  ├── Set monthly budgets by category
  ├── Track spending against budget
  ├── Alert thresholds (50%, 80%, 100%)
  └── Consumes expense events

Analytics Service (ClickHouse or PostgreSQL)
  ├── Monthly/yearly trends
  ├── Category breakdown
  ├── Spending forecasting
  └── CQRS read model (pre-computed)

Notification Service (stateless)
  ├── Email (SMTP/SES)
  ├── Push (FCM)
  └── Consumes from RabbitMQ

API Gateway (Spring Cloud Gateway)
  ├── JWT validation
  ├── Route to services
  ├── Rate limiting
  └── Request logging
```

## 27.3 Database Design (SQL)

```sql
-- users table (User Service - MySQL)
CREATE TABLE users (
    id            BIGINT PRIMARY KEY AUTO_INCREMENT,
    email         VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name    VARCHAR(100),
    last_name     VARCHAR(100),
    role          ENUM('USER', 'ADMIN') DEFAULT 'USER',
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email)
);

-- refresh_tokens table
CREATE TABLE refresh_tokens (
    id         BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id    BIGINT NOT NULL,
    token      VARCHAR(500) UNIQUE NOT NULL,
    expires_at DATETIME NOT NULL,
    revoked    BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- expenses table (Expense Service - PostgreSQL)
CREATE TABLE expenses (
    id           BIGSERIAL PRIMARY KEY,
    user_id      BIGINT NOT NULL,
    description  VARCHAR(255) NOT NULL,
    amount       DECIMAL(12, 2) NOT NULL CHECK (amount > 0),
    category     VARCHAR(50) NOT NULL,
    expense_date DATE NOT NULL,
    notes        TEXT,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_expenses_user_id        ON expenses(user_id);
CREATE INDEX idx_expenses_date           ON expenses(expense_date DESC);
CREATE INDEX idx_expenses_user_category  ON expenses(user_id, category);
CREATE INDEX idx_expenses_user_date      ON expenses(user_id, expense_date DESC);

-- receipts table
CREATE TABLE receipts (
    id         BIGSERIAL PRIMARY KEY,
    expense_id BIGINT NOT NULL REFERENCES expenses(id) ON DELETE CASCADE,
    file_key   VARCHAR(500) NOT NULL,  -- S3 key
    file_name  VARCHAR(255) NOT NULL,
    file_size  BIGINT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## 27.4 MongoDB Schema (Budget Service)

```javascript
// Budget document
{
  "_id": "42:2024-01",        // userId:yearMonth
  "userId": 42,
  "yearMonth": "2024-01",
  "totalBudget": 50000.00,
  "totalSpent": 32500.75,
  "remaining": 17499.25,
  "categories": {
    "FOOD": { "budget": 15000, "spent": 8905.50, "percentage": 59.4 },
    "TRANSPORT": { "budget": 5000, "spent": 3200.00, "percentage": 64.0 },
    "ENTERTAINMENT": { "budget": 8000, "spent": 4500.25, "percentage": 56.3 }
  },
  "alerts": [
    { "type": "50_PERCENT", "sentAt": ISODate("2024-01-20T10:00:00Z") },
    { "type": "80_PERCENT", "sentAt": ISODate("2024-01-28T10:00:00Z") }
  ],
  "updatedAt": ISODate("2024-01-31T23:59:00Z")
}
```

## 27.5 Security Flow

```
1. Registration:
   POST /api/v1/auth/register
   Body: { email, password, firstName, lastName }
   → Validate → BCrypt hash password → Save user
   → Return: 201 Created

2. Login:
   POST /api/v1/auth/login
   Body: { email, password }
   → Load user → BCrypt verify → Generate JWT
   → Return: { accessToken (15min), refreshToken (7days), user }

3. Authenticated Request:
   GET /api/v1/expenses
   Header: Authorization: Bearer <accessToken>
   → API Gateway: validate JWT → extract userId
   → Forward with X-User-Id header → Expense Service
   → Return: expenses for that user

4. Token Refresh:
   POST /api/v1/auth/refresh
   Body: { refreshToken }
   → Validate not revoked/expired
   → Revoke old refresh token
   → Issue new accessToken + refreshToken (rotation)
   → Return: { accessToken, refreshToken }
```

---

# 28. Deployment & Containerization

## 28.1 Creating Executable JARs

```bash
# Build
mvn clean package -DskipTests

# Output: target/expense-service-0.0.1-SNAPSHOT.jar (fat JAR, ~80MB)

# Run with profile
java -jar target/expense-service.jar \
  --spring.profiles.active=prod \
  --server.port=8082

# Override any property at runtime
java -jar target/expense-service.jar \
  --spring.datasource.url=jdbc:postgresql://proddb:5432/expenses \
  --jwt.secret=${JWT_SECRET}
```

## 28.2 WAR Deployment (legacy)

```java
// Extend SpringBootServletInitializer for WAR
@SpringBootApplication
public class ExpenseTrackerApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(ExpenseTrackerApplication.class);
    }
    public static void main(String[] args) {
        SpringApplication.run(ExpenseTrackerApplication.class, args);
    }
}
```

```xml
<!-- pom.xml: change packaging -->
<packaging>war</packaging>

<!-- Mark embedded Tomcat as provided — external Tomcat will be used -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

## 28.3 Dockerfile — Multi-Stage Build

```dockerfile
# Stage 1: Build
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /build

# Cache dependencies layer (only re-runs if pom.xml changes)
COPY pom.xml .
COPY mvnw .
COPY .mvn .mvn
RUN ./mvnw dependency:go-offline -q

# Build application
COPY src ./src
RUN ./mvnw clean package -DskipTests -q

# Stage 2: Runtime (much smaller image — no JDK, no Maven)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Security: run as non-root
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
USER appuser

COPY --from=builder /build/target/*.jar app.jar

# JVM tuning for containers
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+UseG1GC", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]

EXPOSE 8082

HEALTHCHECK --interval=30s --timeout=5s --start-period=60s --retries=3 \
  CMD wget -qO- http://localhost:8082/actuator/health | grep -q '"status":"UP"' || exit 1
```

```bash
# Build image
docker build -t expense-service:1.0.0 .

# Run container
docker run -d \
  --name expense-service \
  -p 8082:8082 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DATABASE_URL=jdbc:postgresql://db:5432/expenses \
  -e JWT_SECRET=your-secret \
  expense-service:1.0.0
```

## 28.4 Docker Compose — Full Stack Local Dev

```yaml
# docker-compose.yml
version: '3.8'

services:
  # === DATABASES ===
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: expenses
      POSTGRES_USER: expense_user
      POSTGRES_PASSWORD: expense_pass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U expense_user -d expenses"]
      interval: 10s
      timeout: 5s
      retries: 5

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: users
      MYSQL_USER: user_service
      MYSQL_PASSWORD: user_pass
      MYSQL_ROOT_PASSWORD: root_pass
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      retries: 5

  mongodb:
    image: mongo:6.0
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # === MESSAGING ===
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin

  # === INFRASTRUCTURE ===
  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://localhost:8761/actuator/health"]
      interval: 30s
      retries: 3

  config-server:
    build: ./config-server
    ports:
      - "8888:8888"
    environment:
      - GIT_URI=https://github.com/your-org/config-repo
    depends_on:
      - eureka-server

  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"

  # === MICROSERVICES ===
  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/
    depends_on:
      - eureka-server
      - config-server

  user-service:
    build: ./user-service
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/users
      - SPRING_DATASOURCE_USERNAME=user_service
      - SPRING_DATASOURCE_PASSWORD=user_pass
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/
    depends_on:
      mysql:
        condition: service_healthy
      eureka-server:
        condition: service_healthy

  expense-service:
    build: ./expense-service
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/expenses
      - SPRING_DATASOURCE_USERNAME=expense_user
      - SPRING_DATASOURCE_PASSWORD=expense_pass
      - SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/
    depends_on:
      postgres:
        condition: service_healthy
      kafka:
        condition: service_started
      eureka-server:
        condition: service_healthy

  budget-service:
    build: ./budget-service
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATA_MONGODB_URI=mongodb://mongodb:27017/budgets
      - SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/

  notification-service:
    build: ./notification-service
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_RABBITMQ_HOST=rabbitmq
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/

volumes:
  postgres_data:
  mysql_data:
  mongo_data:
```

## 28.5 Health Checks with Actuator

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,loggers
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true           # /actuator/health/liveness + /actuator/health/readiness
  health:
    db:
      enabled: true
    kafka:
      enabled: true
    redis:
      enabled: true
  info:
    git:
      mode: full
```

```java
// Custom health indicator
@Component
public class ExternalApiHealthIndicator implements HealthIndicator {

    private final ExchangeRateApiClient apiClient;

    @Override
    public Health health() {
        try {
            apiClient.ping();
            return Health.up()
                .withDetail("exchange-rate-api", "reachable")
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("exchange-rate-api", "unreachable")
                .withException(e)
                .build();
        }
    }
}
```

---

# 29. Linux for Backend Developers

## 29.1 Essential File System Commands

```bash
# Navigation
pwd                         # print working directory
ls -la                      # list all files with permissions and size
cd /opt/expense-tracker     # change directory
cd ~                        # go to home directory
cd ..                       # go up one level

# File operations
cp app.jar app.jar.bak      # copy
mv app.jar /opt/app.jar     # move / rename
rm -rf /tmp/old-logs/       # delete recursively (IRREVERSIBLE — be careful!)
mkdir -p /var/log/expense/  # create directory tree
ln -s /opt/app/current /opt/app/latest  # symbolic link

# File viewing
cat application.yml         # print entire file
less large-file.log         # paginated view (q to quit)
head -n 50 app.log          # first 50 lines
tail -n 100 app.log         # last 100 lines
tail -f app.log             # FOLLOW: live view (Ctrl+C to stop)
tail -f app.log | grep ERROR  # live view filtered to errors only

# Search
grep "ERROR" app.log                    # lines containing ERROR
grep -i "exception" *.log              # case-insensitive, all .log files
grep -n "NullPointer" app.log          # show line numbers
grep -r "DatabaseException" /var/log/  # recursive through directories
grep -A 5 "ERROR" app.log             # 5 lines AFTER each match (context)
grep -B 3 "ERROR" app.log             # 3 lines BEFORE each match

# Count
wc -l app.log               # line count
grep -c "ERROR" app.log     # count ERROR occurrences
```

## 29.2 Process Management

```bash
# View processes
ps aux                      # all processes
ps aux | grep java          # filter to java processes
top                         # real-time CPU/memory (q to quit)
htop                        # better top (apt install htop)
jps                         # Java processes only (JDK tool)

# Start / stop
java -jar app.jar &         # start in background
nohup java -jar app.jar > app.log 2>&1 &  # persist after logout

# Control signals
kill -15 <pid>              # SIGTERM: graceful shutdown (app handles it)
kill -9 <pid>               # SIGKILL: force kill (no cleanup — last resort)
kill -2 <pid>               # SIGINT: like Ctrl+C

# Systemd service management (production)
systemctl start expense-service
systemctl stop expense-service
systemctl restart expense-service
systemctl status expense-service
systemctl enable expense-service    # start on boot
journalctl -u expense-service       # view service logs
journalctl -u expense-service -f    # follow service logs
journalctl -u expense-service --since "2024-01-15 10:00:00"
```

## 29.3 Networking

```bash
# Check what's running on a port
netstat -tlnp | grep 8082           # what's on port 8082?
ss -tlnp                            # modern alternative to netstat
lsof -i :8082                       # process using port 8082

# Test connectivity
ping google.com
curl http://localhost:8082/actuator/health
curl -v -X POST http://localhost:8082/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password"}'
wget -qO- http://localhost:8082/actuator/health

# View network interfaces
ip addr show
ifconfig
```

## 29.4 Environment Variables

```bash
# Set in current session
export DATABASE_URL="jdbc:postgresql://localhost/expenses"
export JWT_SECRET="super-secret-key"

# View value
echo $DATABASE_URL

# Unset
unset DATABASE_URL

# Persistent (add to ~/.bashrc or ~/.profile)
echo 'export DATABASE_URL="jdbc:postgresql://localhost/expenses"' >> ~/.bashrc
source ~/.bashrc    # reload without logging out

# Pass to java process
DATABASE_URL="jdbc:postgresql://..." JWT_SECRET="..." java -jar app.jar

# System-wide for services (/etc/environment)
DATABASE_URL=jdbc:postgresql://proddb:5432/expenses
JWT_SECRET=your-production-secret
```

## 29.5 File Permissions

```bash
# Permission format: rwxrwxrwx (owner group others)
# r=4, w=2, x=1
ls -la app.jar
# -rw-r--r-- 1 appuser appgroup 80M Jan 15 app.jar

chmod 755 startup.sh        # owner=rwx, group=rx, others=rx (executable)
chmod 644 application.yml   # owner=rw, group=r, others=r (config file)
chmod +x startup.sh         # add execute permission
chown appuser:appgroup app.jar  # change owner:group
chown -R appuser /opt/expense-tracker/  # recursive
```

## 29.6 Disk and Memory Monitoring

```bash
# Disk
df -h                           # disk space per filesystem
du -sh /opt/expense-tracker/    # directory total size
du -sh /var/log/*               # size of each log file

# Memory
free -h                         # RAM usage (free/used/available)
cat /proc/meminfo               # detailed memory info

# CPU
nproc                           # number of CPU cores
uptime                          # load average (1min, 5min, 15min)

# Java-specific diagnostics
jstack <pid>                    # thread dump (great for diagnosing hangs)
jmap -heap <pid>                # heap memory summary
jstat -gc <pid> 1000 10         # GC stats: every 1s, 10 times
jcmd <pid> VM.flags             # all JVM flags in effect
```

## 29.7 Log Analysis in Production

```bash
# Count errors per minute
grep "ERROR" app.log | awk '{print $1, $2}' | cut -c1-16 | sort | uniq -c

# Find all unique exception types
grep "Exception" app.log | grep -oP '\w+Exception' | sort | uniq -c | sort -rn

# Find slowest requests (assuming log format includes response time)
grep "duration" app.log | sort -t= -k2 -rn | head -20

# Check if specific user has errors
grep "userId=42" app.log | grep ERROR

# Find requests that took > 1000ms
grep -E "duration=[0-9]{4,}" app.log

# Archive and compress old logs
tar -czf logs-2024-01.tar.gz /var/log/expense/2024-01/
gzip app.log.2024-01-15       # compress single file
```

---

# Appendix A: Spring Boot Starters Reference

| Starter | What's Included | When to Use |
|---|---|---|
| `spring-boot-starter-web` | Spring MVC, Tomcat, Jackson | REST APIs |
| `spring-boot-starter-data-jpa` | Spring Data JPA, Hibernate | SQL databases |
| `spring-boot-starter-data-mongodb` | Spring Data MongoDB | Document databases |
| `spring-boot-starter-data-redis` | Lettuce Redis client | Caching, sessions |
| `spring-boot-starter-security` | Spring Security | Auth/authz |
| `spring-boot-starter-validation` | Hibernate Validator | Input validation |
| `spring-boot-starter-cache` | Cache abstraction | Any caching |
| `spring-boot-starter-actuator` | Health, metrics, info | Monitoring |
| `spring-boot-starter-test` | JUnit5, Mockito, AssertJ | Testing |
| `spring-cloud-starter-gateway` | API Gateway + WebFlux | API Gateway |
| `spring-cloud-starter-netflix-eureka-client` | Eureka client | Service registration |
| `spring-cloud-starter-netflix-eureka-server` | Eureka server | Service registry |
| `spring-cloud-starter-openfeign` | Feign HTTP client | Inter-service calls |
| `spring-cloud-starter-config` | Config client | External config |
| `spring-cloud-config-server` | Config server | Central config server |
| `spring-kafka` | Kafka producer/consumer | Event streaming |
| `spring-boot-starter-amqp` | RabbitMQ | Message queues |
| `resilience4j-spring-boot3` | Circuit breaker, retry | Fault tolerance |

---

# Appendix B: Microservice Patterns Summary

| Pattern | Problem Solved | Tool/Implementation |
|---|---|---|
| API Gateway | Single entry point | Spring Cloud Gateway |
| Service Discovery | Dynamic URL resolution | Netflix Eureka |
| Client-Side LB | Distribute load across instances | Spring Cloud LoadBalancer |
| Circuit Breaker | Prevent cascading failures | Resilience4j |
| Distributed Tracing | Debug cross-service requests | Micrometer + Zipkin |
| Centralized Config | Consistent configuration | Spring Cloud Config |
| Event-Driven | Loose coupling, async | Kafka, RabbitMQ |
| CQRS | Separate read/write models | Custom implementation |
| Saga | Distributed transactions | Choreography via events |
| Bulkhead | Isolate failures | Resilience4j ThreadPool |
| Retry | Transient failure recovery | Resilience4j Retry |
| Rate Limiting | Protect from overload | Spring Gateway + Redis |
| Dead Letter Queue | Handle failed messages | RabbitMQ DLQ, Kafka DLT |

---

# Appendix C: SQL vs NoSQL Decision Guide

| Concern | Use SQL (PostgreSQL/MySQL) | Use NoSQL (MongoDB/Redis) |
|---|---|---|
| Data relationships | Complex joins, foreign keys | Embedded documents, flat data |
| Consistency | ACID transactions required | Eventual consistency OK |
| Schema | Stable, well-defined schema | Schema-less, evolving structure |
| Query pattern | Ad-hoc complex queries | Known access patterns |
| Scale | Vertical primarily | Horizontal sharding |
| Use in project | Expenses, Users (structured) | Budgets (flexible), Cache |

---

*End of Java Full Stack Master Reference Guide*
*Covers: Core Java → Advanced Java → Spring Boot → Microservices → Deployment*
