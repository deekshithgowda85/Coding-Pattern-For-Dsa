
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

