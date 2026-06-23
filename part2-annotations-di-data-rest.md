
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

