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

