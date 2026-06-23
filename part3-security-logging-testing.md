
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

