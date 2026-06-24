
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
