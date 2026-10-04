# Spring Boot 101 — Ground Up Learning Guide

> A first-principles guide for engineers learning Spring Boot.
> Written for: **Spring Boot 4** (Java 21 minimum). Spring Boot 3 differences are called out explicitly.
> Examples are drawn from the SME service codebase.

---

## Table of Contents

1. [What Is Spring Boot?](#1-what-is-spring-boot)
2. [The IoC Container — The Core Idea](#2-the-ioc-container)
3. [Dependency Injection — How Beans Find Each Other](#3-dependency-injection)
4. [Bean Lifecycle — Birth to Death](#4-bean-lifecycle)
5. [Auto-Configuration — The Magic Explained](#5-auto-configuration)
6. [Configuration — Properties, Profiles, @Value, @ConfigurationProperties](#6-configuration)
7. [The Web Layer — Controllers & Filters](#7-the-web-layer)
8. [The Data Layer — Spring Data & MongoDB](#8-the-data-layer)
9. [Security — Authentication & Authorization](#9-security)
10. [Messaging — Spring Kafka](#10-messaging--spring-kafka)
11. [AOP — Cross-Cutting Concerns](#11-aop--cross-cutting-concerns)
12. [Exception Handling](#12-exception-handling)
13. [Validation](#13-validation)
14. [Caching](#14-caching)
15. [Scheduling & Async](#15-scheduling--async)
16. [Testing](#16-testing)
17. [Spring Boot 3 → 4: What Changes](#17-spring-boot-3--4-what-changes)
18. [Java 21 Features That Matter for Spring Boot](#18-java-21-features-that-matter-for-spring-boot)
19. [Glossary](#19-glossary)
20. [Further Reading](#20-further-reading)

---

## 1. What Is Spring Boot?

To understand Spring Boot, you need to understand the layers beneath it.

### The Layers

```
┌──────────────────────────────────────────────────────┐
│  Spring Boot                                          │
│  "Convention over configuration"                      │
│  Auto-configures everything. You just add deps.       │
├──────────────────────────────────────────────────────┤
│  Spring Framework                                     │
│  IoC container, DI, MVC, Security, Data, AOP, etc.   │
│  The engine. Powerful but verbose to configure.       │
├──────────────────────────────────────────────────────┤
│  Jakarta EE (formerly Java EE)                        │
│  Standard Java specs: Servlet, JPA, Validation, etc. │
│  Spring implements these specs.                       │
├──────────────────────────────────────────────────────┤
│  JVM / Java                                           │
└──────────────────────────────────────────────────────┘
```

**Spring Framework** is the actual engine — it manages your objects, handles HTTP, talks to databases, secures endpoints. It's been around since 2003 and is excellent. But configuring it from scratch requires hundreds of lines of XML or Java config.

**Spring Boot** is a thin layer on top that says: "I know what 90% of apps need. Let me set it up automatically based on what JARs you have on the classpath." You add `spring-boot-starter-web` to your `pom.xml` and you get a running Tomcat server, JSON serialization, error handling, and health endpoints — zero configuration.

**Jakarta EE** is the standard. Spring implements the standard specs (`@Inject`, Servlet API, Bean Validation) so that your code doesn't depend on Spring internals. This is why your code uses `@Inject` (Jakarta) instead of only `@Autowired` (Spring) — code written against Jakarta specs is portable.

### Spring Boot 4 vs Spring Boot 3

| Feature | Spring Boot 3 | Spring Boot 4 |
|---------|--------------|---------------|
| Minimum Java | 17 | **21** |
| Spring Framework | 6.x | **7.x** |
| Jakarta EE | 10 | **11** |
| Virtual threads | Opt-in | **Default** for web & Kafka |
| RestTemplate | Deprecated | **Removed** — use `RestClient` |
| OTel integration | Via Micrometer bridge | **First-class, built-in** |
| Structured logging | Via logstash encoder | **Built-in** (`logging.structured.format`) |

**What does this mean for your SME service?** You're on Spring Boot 3.5.x today. The code in this guide shows Spring Boot 4 style where it differs, so you're learning the direction of travel.

### @SpringBootApplication — What It Actually Does

```java
// Your SmeApplication.java
@SpringBootApplication(exclude = {
    MongoAutoConfiguration.class,        // "don't auto-configure Mongo — we do it manually"
    MongoDataAutoConfiguration.class,
    MongoReactiveAutoConfiguration.class
})
@EnableCaching      // activate Spring's @Cacheable/@CacheEvict AOP proxy
@EnableRetry        // activate @Retryable AOP proxy
@ComponentScan({"com.niyo.sme.commons.*", "com.niyo*"})  // scan multiple modules
public class SmeApplication {
    public static void main(String[] args) {
        SpringApplication.run(SmeApplication.class, args);
    }
}
```

`@SpringBootApplication` is actually three annotations combined:

```java
@SpringBootConfiguration   // = @Configuration, marks this as a config class
@EnableAutoConfiguration   // "scan META-INF/spring/... and auto-configure"
@ComponentScan             // "scan this package and subpackages for @Component beans"
```

The `exclude` shows a key Spring Boot pattern: **opt out of specific auto-configurations** when you need custom control. The SME service manages MongoDB manually (custom read preference, write concern, converters) so it excludes Spring Boot's default MongoDB auto-configuration and provides its own `DatabaseConfiguration`.

---

## 2. The IoC Container

**IoC** (Inversion of Control) is the central idea of Spring. Understanding it deeply makes everything else click.

### Without IoC (The Old Way)

```java
// You create and wire everything yourself
public class PayoutsService {
    private CardHostClient cardHostClient;
    private PayoutsRepository repository;
    private KafkaTemplate kafkaTemplate;

    public PayoutsService() {
        // You're responsible for creating dependencies
        ObjectMapper mapper = new ObjectMapper();
        OkHttpClient http = new OkHttpClient.Builder().build();
        this.cardHostClient = new CardHostClient(http, mapper, "https://card-host.internal");
        this.repository = new PayoutsRepository(new MongoClient("mongodb://localhost:27017"));
        this.kafkaTemplate = new KafkaTemplate(new DefaultKafkaProducerFactory(props));
    }
}
```

Problems:
- `PayoutsService` knows how to build its own dependencies — tight coupling
- Testing is hard — you can't swap `CardHostClient` for a mock without changing `PayoutsService`
- Configuration is scattered — MongoDB URI is hardcoded inside `PayoutsService`
- Each class that needs `KafkaTemplate` creates its own copy

### With IoC (Spring's Way)

```java
// You declare what you need. Spring creates and injects it.
@Service
public class PayoutsServiceImpl implements PayoutsService {

    private final CardHostClient cardHostClient;
    private final PayoutsRepository repository;
    private final KafkaTemplate<String, Object> kafkaTemplate;

    // Spring reads this constructor and injects each parameter
    public PayoutsServiceImpl(CardHostClient cardHostClient,
                               PayoutsRepository repository,
                               KafkaTemplate<String, Object> kafkaTemplate) {
        this.cardHostClient = cardHostClient;
        this.repository = repository;
        this.kafkaTemplate = kafkaTemplate;
    }
}
```

Now:
- `PayoutsServiceImpl` doesn't know how `CardHostClient` is built — loose coupling
- In tests, inject a mock `CardHostClient` — no code change needed
- Configuration lives in `application.yml` — one place for all config
- Spring creates ONE `KafkaTemplate` and shares it everywhere (singleton by default)

**The IoC container** is the thing that does this — it's a registry of objects (called **beans**) and their wiring. At startup, Spring:
1. Scans your code for `@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`
2. Creates instances of each (respecting dependencies)
3. Wires them together
4. Manages their lifecycle (startup, shutdown)

The result is a graph of connected objects, assembled by Spring, that your application code can use.

### The ApplicationContext

The container itself is called the `ApplicationContext`. It's the object that holds all your beans:

```java
// You rarely interact with it directly, but conceptually:
ApplicationContext context = SpringApplication.run(SmeApplication.class, args);

// Every bean is accessible from it
PayoutsService payouts = context.getBean(PayoutsService.class);
KafkaTemplate kafka    = context.getBean(KafkaTemplate.class);
```

In production, Spring Boot creates an `AnnotationConfigServletWebServerApplicationContext` (for web apps) or `AnnotationConfigApplicationContext` (for non-web, like your workers). Both work the same way — the type just determines whether an embedded server starts.

---

## 3. Dependency Injection

**DI** (Dependency Injection) is how the IoC container wires beans together. There are three styles. Spring Boot 4 has a strong preference for one of them.

### Style 1: Constructor Injection (Preferred)

```java
@Service
public class CorporateServiceImpl implements CorporateService {

    private final SmeRepository smeRepository;
    private final KafkaTemplate<String, Object> kafkaTemplate;
    private final DataDogMetricsService metricsService;

    // Spring sees ONE constructor → injects automatically, no @Autowired needed
    public CorporateServiceImpl(SmeRepository smeRepository,
                                 KafkaTemplate<String, Object> kafkaTemplate,
                                 DataDogMetricsService metricsService) {
        this.smeRepository = smeRepository;
        this.kafkaTemplate = kafkaTemplate;
        this.metricsService = metricsService;
    }
}
```

With Lombok (which you use):
```java
@Service
@RequiredArgsConstructor   // generates the constructor above from all final fields
public class CorporateServiceImpl implements CorporateService {

    private final SmeRepository smeRepository;
    private final KafkaTemplate<String, Object> kafkaTemplate;
    private final DataDogMetricsService metricsService;
}
```

**Why constructor injection is best:**
- Fields are `final` — the object is immutable after construction (thread-safe)
- Circular dependencies fail fast at startup, not at runtime
- Easy to test — just `new CorporateServiceImpl(mockRepo, mockKafka, mockMetrics)`
- No Spring annotations needed in the class body — the class is a plain Java class

### Style 2: Field Injection (Your codebase uses this — avoid in new code)

```java
// Found in your workers and services
@Service
public class JobApplicationReceivedNotifyWorker {

    @Inject private PortalNotificationService portalNotificationService;
    @Inject private ServiceUtils serviceUtils;
    @Autowired private KafkaUtils kafkaUtils;
}
```

`@Inject` (from Jakarta) and `@Autowired` (from Spring) do the same thing. Spring Boot 4 still supports both but the Spring team's recommended style is constructor injection.

**Why field injection is problematic:**
- Fields can't be `final` — the object is mutable after construction
- Testing requires Spring context or reflection hacks (`ReflectionTestUtils.setField()`)
- Circular dependencies fail at runtime, not at startup
- Harder to see what a class actually needs (just look at the constructor for constructor injection)

### Style 3: Setter Injection (Rarely used)

```java
@Service
public class MyService {
    private SomeOptionalDep dep;

    @Autowired(required = false)  // inject only if bean exists
    public void setDep(SomeOptionalDep dep) {
        this.dep = dep;
    }
}
```

Use this only for truly **optional** dependencies.

### @Inject vs @Autowired — Which to Use?

| Annotation | Package | Part of |
|------------|---------|---------|
| `@Inject` | `jakarta.inject` | Jakarta EE standard |
| `@Autowired` | `org.springframework` | Spring-specific |

Both work identically for injection. `@Inject` is the standard (portable), `@Autowired` is Spring-only (has `required = false` feature). Spring Boot 4 recommendation: use constructor injection and you don't need either in the class body.

### Qualifying When Multiple Beans Exist

```java
// Two beans of the same type:
@Bean
public KafkaListenerContainerFactory<?> kafkaListenerContainerFactory() { ... }  // @Primary

@Bean
public KafkaListenerContainerFactory<?> retryListenerContainerFactory() { ... }

// Injection:
@Autowired
@Qualifier("retryListenerContainerFactory")   // pick the non-primary one
private KafkaListenerContainerFactory<?> retryFactory;

// Or on a @KafkaListener:
@KafkaListener(containerFactory = "retryListenerContainerFactory", ...)
public void listen(...) { }
```

`@Primary` marks the default bean when multiple exist. `@Qualifier` overrides the default.

---

## 4. Bean Lifecycle

Understanding the lifecycle explains when to use `@PostConstruct`, `@PreDestroy`, and `InitializingBean`.

### The Full Lifecycle

```
1. Spring scans and finds @Component / @Bean
2. Spring instantiates the class (calls constructor)
3. Spring injects dependencies (@Autowired fields, if any)
4. Spring calls @PostConstruct method (your init logic)
5. Bean is ready — serves requests
   ...time passes...
6. Application shutdown begins
7. Spring calls @PreDestroy method (your cleanup logic)
8. Bean is garbage collected
```

### @PostConstruct — Init After Injection

```java
// From your KafkaConfig.java
@Configuration
@EnableKafka
public class KafkaConfig {

    @Value("${kafka.secret-arn}")
    private String kafkaSecretManagerARN;

    @Inject
    private AwsService awsService;

    private String consJaasCfg;  // computed, not injectable

    @PostConstruct
    public void initKafkaConfig() throws IOException {
        // At this point: @Value fields are set, @Inject fields are injected
        // So we can safely use them to do initialization
        String secret = awsService.getSecretForAws(kafkaSecretManagerARN);
        this.consJaasCfg = buildJaasConfig(secret);
        // Also initializes topic-to-class mappings for Kafka deserialization
    }
}
```

**Why not do this in the constructor?** Because `@Value` fields and `@Inject` fields are populated AFTER the constructor runs. At constructor time, `awsService` is still `null`.

```
Constructor() runs → awsService is null here
@Inject awsService is populated
@Value fields are populated
@PostConstruct runs → awsService is ready here ✅
```

### @PreDestroy — Cleanup Before Shutdown

```java
@Service
public class KafkaProducerService {

    private final KafkaTemplate<String, Object> kafkaTemplate;

    @PreDestroy
    public void shutdown() {
        // Flush all pending messages before the JVM exits
        kafkaTemplate.flush();
        log.info("KafkaProducerService shut down — all messages flushed");
    }
}
```

### Bean Scopes

By default, every Spring bean is a **singleton** — one instance shared by the entire application. This is almost always what you want for services, repositories, and config classes.

| Scope | Annotation | Instances | Use Case |
|-------|-----------|-----------|----------|
| **Singleton** | (default) | 1 per application | Services, repositories, config |
| **Prototype** | `@Scope("prototype")` | 1 per injection point | Stateful objects (rarely needed) |
| **Request** | `@RequestScope` | 1 per HTTP request | Request-scoped data (auth context) |
| **Session** | `@SessionScope` | 1 per HTTP session | Session data (avoid — use JWT instead) |

```java
// Request-scoped bean — new instance for every HTTP request
@Component
@RequestScope
public class RequestContext {
    private String correlationId;
    private String userId;
    // getters/setters
}

// Inject request-scoped bean into a singleton service
@Service
public class MyService {
    // Spring injects a PROXY that forwards calls to the current request's instance
    @Inject
    private RequestContext requestContext;
}
```

**Important for your service**: Because workers process Kafka messages (not HTTP requests), request-scoped beans don't work in the worker module. Context must be passed explicitly or via MDC.

---

## 5. Auto-Configuration

This is Spring Boot's superpower. Understanding it lets you debug "why is Spring Boot doing X?" and "how do I override it?"

### How It Works

When Spring Boot starts, it reads a file called `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` from every JAR on the classpath. Each line is a class name. Spring Boot loads these classes conditionally.

```
# Inside spring-boot-autoconfigure.jar
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
# ... ~150 more
```

Each auto-configuration class is annotated with `@Conditional*` annotations that check:

```java
@AutoConfiguration
@ConditionalOnClass(MongoClient.class)          // "only if MongoDB driver JAR is present"
@ConditionalOnMissingBean(MongoClient.class)    // "only if the user hasn't defined their own"
@EnableConfigurationProperties(MongoProperties.class)
public class MongoAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(MongoClientFactory.class)
    public MongoClient mongo(MongoProperties properties) {
        return MongoClients.create(properties.determineUri());
    }
}
```

**The key condition: `@ConditionalOnMissingBean`**. If YOU define a `MongoClient` bean, Spring Boot's auto-configuration backs off and doesn't create its own.

This is exactly what your `DatabaseConfiguration` does:

```java
// DatabaseConfiguration.java
@Configuration
public class DatabaseConfiguration extends AbstractMongoClientConfiguration {

    @Bean
    @Override
    public MongoClient mongoClient() {
        // Your custom MongoClient with WriteConcern.MAJORITY etc.
        return MongoClients.create(settings);
    }
}

// + SmeApplication.java excludes the auto-config entirely:
@SpringBootApplication(exclude = {MongoAutoConfiguration.class, ...})
```

### Common @Conditional Annotations

| Annotation | Meaning |
|------------|---------|
| `@ConditionalOnClass(Foo.class)` | Only configure if `Foo` is on classpath |
| `@ConditionalOnMissingClass("com.Foo")` | Only configure if `Foo` is NOT on classpath |
| `@ConditionalOnBean(Foo.class)` | Only configure if a `Foo` bean exists |
| `@ConditionalOnMissingBean(Foo.class)` | Only configure if NO `Foo` bean exists (back-off) |
| `@ConditionalOnProperty("my.feature.enabled")` | Only configure if property is set to true |
| `@ConditionalOnWebApplication` | Only configure in a web application |
| `@ConditionalOnExpression("${app.enabled:true}")` | SpEL expression must be true |

### Debugging Auto-Configuration

```bash
# Run with this flag to see every auto-configuration decision
java -jar sme-service.jar --debug

# Output includes:
# ============================
# CONDITIONS EVALUATION REPORT
# ============================
# Positive matches:  (auto-configurations that DID activate)
#   KafkaAutoConfiguration matched:
#     - @ConditionalOnClass found required class 'KafkaTemplate' (OnClassCondition)
#
# Negative matches: (auto-configurations that did NOT activate, and why)
#   MongoAutoConfiguration:
#     Did not match:
#       - @ConditionalOnMissingBean (types: MongoClient) found beans 'mongoClient' (OnBeanCondition)
```

Or check `/actuator/conditions` endpoint (already exposed in your service) for the same info at runtime.

---

## 6. Configuration

Your application needs values from the outside world (DB URIs, API keys, timeouts). Spring Boot has multiple mechanisms, in increasing order of sophistication.

### 6.1 application.yml and Profiles

```yaml
# src/main/resources/config/application.yml (base — all environments)
spring:
  application:
    name: sme-service-apis
  data:
    mongodb:
      database: sme_db

server:
  port: 8080
  servlet:
    context-path: /sme

kafka:
  bootstrap-server: localhost:9092
  topic-prefix: com.sme.dev.
```

```yaml
# application-dev.yml (overrides for local development)
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017

kafka:
  bootstrap-server: localhost:9092
```

```yaml
# application-prod.yml (overrides for production)
spring:
  data:
    mongodb:
      uri: ${MONGO_URI}   # from environment variable
```

**How profiles work:**

```bash
# Activate a profile at startup:
java -jar app.jar --spring.profiles.active=prod

# Or via environment variable:
SPRING_PROFILES_ACTIVE=prod java -jar app.jar

# Multiple profiles:
java -jar app.jar --spring.profiles.active=prod,feature-x
```

Spring Boot loads `application.yml` first, then `application-{profile}.yml` on top. Values in the profile file override the base. Your `SmeApplication` sets `dev` as the default:

```java
app.setDefaultProperties(Collections.singletonMap("spring.profiles.default", "dev"));
```

### 6.2 @Value — Simple Property Injection

```java
// From FrsServiceProperties.java in your codebase
@Configuration
public class FrsServiceProperties {

    @Value("${frs.api.baseurl}")                   // required — fails at startup if missing
    private String baseUrl;

    @Value("${frs.api.enableLogs:false}")           // optional — defaults to false
    private boolean enableLogs;

    @Value("${frs.api.timeout:30}")                 // optional — defaults to 30
    private int timeoutSeconds;

    @Value("${allowed.cors.origins:*}")             // default to *
    private String allowedOrigins;

    // SpEL expressions work too:
    @Value("#{${max.retries:3} * 1000}")            // multiply property by 1000
    private int maxRetryMs;
}
```

**When to use `@Value`:**
- Simple scalar values (strings, ints, booleans)
- A handful of properties in a class
- One-off config values

**When NOT to use `@Value`:**
- Groups of related properties → use `@ConfigurationProperties`
- Properties that need validation → use `@ConfigurationProperties` with `@Validated`

### 6.3 @ConfigurationProperties — The Right Way for Groups

```java
// Spring Boot 4 — preferred for grouped config
@ConfigurationProperties(prefix = "kafka")
@Validated  // enables @NotBlank, @Min etc. on fields
public record KafkaProperties(
    @NotBlank String bootstrapServer,
    @NotBlank String topicPrefix,
    @NotBlank String groupIdPrefix,
    @Min(1) int concurrency,
    @Min(1) int maxPollRecords
) {}

// Register it:
@SpringBootApplication
@ConfigurationPropertiesScan   // scans for all @ConfigurationProperties classes
public class SmeApplication { ... }

// Use it:
@Service
@RequiredArgsConstructor
public class KafkaConfig {
    private final KafkaProperties kafkaProperties;

    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
                  kafkaProperties.bootstrapServer());  // type-safe, IDE-completable
    }
}
```

In Spring Boot 4, `@ConfigurationProperties` works beautifully with **Java records** (immutable, concise, validated). This is the recommended style.

```yaml
# application.yml
kafka:
  bootstrap-server: localhost:9092
  topic-prefix: com.sme.dev.
  group-id-prefix: sme-dev-
  concurrency: 3
  max-poll-records: 50
```

**Benefits of `@ConfigurationProperties` over `@Value`:**
- IDE auto-completion (with the `spring-boot-configuration-processor` annotation processor)
- Type safety at startup — wrong type = fail fast, not at first use
- Bean Validation — `@NotBlank`, `@Min`, `@Max`
- Relaxed binding — `my-property`, `MY_PROPERTY`, `myProperty` all map to `myProperty`

### 6.4 Environment Variables and Secrets

Properties can come from environment variables. Spring Boot maps them automatically:

```
Environment variable:    SPRING_DATA_MONGODB_URI
Equivalent property:     spring.data.mongodb.uri
```

For secrets (API keys, DB passwords), your service fetches them from AWS Secrets Manager at startup via `@PostConstruct` in `KafkaConfig`. This is a valid pattern — the secret is never in `application.yml` or committed to git.

In Spring Boot 4, **Spring Cloud AWS** has first-class integration for fetching properties directly from Secrets Manager and Parameter Store as if they were `application.yml` properties:

```yaml
# No @PostConstruct needed — Spring Boot fetches and decrypts automatically
spring:
  config:
    import: "aws-secretsmanager:/myapp/kafka-credentials"
```

---

## 7. The Web Layer

### 7.1 How an HTTP Request Flows Through Spring Boot

```
Browser/Client
      │  HTTP Request
      ▼
┌─────────────────────────────────────────────────────────┐
│  Embedded Tomcat (port 8080)                             │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│  Filter Chain (ordered)                                  │
│  1. MDCFilter (@Order HIGHEST_PRECEDENCE)                │
│     → sets correlationId, userId in MDC                  │
│  2. SmeAuthenticationFilter                              │
│     → validates JWT, sets SecurityContext                │
│  3. Other filters...                                     │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│  DispatcherServlet ("Front Controller")                  │
│  → finds the right @Controller method for this URL       │
│  → calls @Secured check via Spring Security AOP          │
│  → invokes the controller method                         │
│  → serializes return value to JSON                       │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│  @RestController method                                  │
│  → calls @Service                                        │
│  → @Service calls @Repository                            │
│  → returns ResponseEntity<T>                            │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
                      JSON Response
```

### 7.2 Filters vs Interceptors — What's the Difference?

| | Filter | Interceptor |
|--|--------|------------|
| Part of | Jakarta Servlet spec | Spring MVC |
| Scope | Wraps the entire Servlet, including Spring Security | Wraps only the `DispatcherServlet` handler |
| Can access | Raw `HttpServletRequest/Response` | `HandlerMethod` (the actual controller method) |
| Registered in | `Filter` interface or `@Component` with `@Order` | `WebMvcConfigurer.addInterceptors()` |
| Use for | Auth, logging, CORS, rate limiting, MDC | Request logging, controller-specific logic |

Your `MDCFilter` is a Filter because it must run before Spring Security (which is also a filter chain). An interceptor runs after the security filters.

```java
// MDCFilter.java — runs BEFORE Spring Security
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)   // runs first among all filters
public class MDCFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        try {
            // Set up MDC context for all downstream logging
            MDC.put("correlationId", extractOrGenerate(req));
            MDC.put("userId", currentUser());

            chain.doFilter(req, res);   // pass to next filter / spring security / controller
        } finally {
            MDC.clear();   // CRITICAL: thread pool reuses threads — always clean up
        }
    }
}
```

### 7.3 Controllers

```java
// From your LoanController.java
@RestController               // = @Controller + @ResponseBody (return values serialized to JSON)
@RequestMapping("/")          // base path (added to server.servlet.context-path = /sme)
@Slf4j
public class LoanController {

    @Inject
    private CorporateService corporateService;

    // GET /sme/v1/loan?page=0&size=20&sort=createdAt,desc
    @GetMapping("v1/loan")
    @Secured(AuthoritiesConstants.Code.LOAN_DOCUMENT_VIEW)   // role check
    public ResponseEntity<List<LoanDTO>> getAllLoanApplied(Pageable pageable) {
        var page = corporateService.getAllLoanRequests(pageable);
        if (page.isEmpty()) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.ok()
            .headers(PaginationUtil.generatePaginationHttpHeaders(page))
            .body(page.getContent());
    }

    // POST /sme/v1/loan/weblink/email?corporateCode=CORP-1&emailIds=a@b.com,c@d.com
    @PostMapping("v1/loan/weblink/email")
    @Secured(AuthoritiesConstants.Code.LOAN_DOCUMENT_VIEW)
    public ResponseEntity<Void> sendLoanWebLink(
            @RequestParam(required = false) String corporateCode,
            @RequestParam(required = false) List<String> emailIds) {
        corporateService.emailTriggerForLoanWebLink(corporateCode, emailIds);
        return ResponseEntity.ok().build();
    }
}
```

**Controller annotation reference:**

| Annotation | Meaning |
|------------|---------|
| `@RestController` | This class handles HTTP, return values are JSON |
| `@RequestMapping("/api")` | URL prefix for all methods in this class |
| `@GetMapping("/path")` | Handle GET requests |
| `@PostMapping("/path")` | Handle POST requests |
| `@PutMapping`, `@PatchMapping`, `@DeleteMapping` | Other HTTP verbs |
| `@PathVariable` | Extract from URL: `/loan/{id}` → `@PathVariable String id` |
| `@RequestParam` | Extract from query string: `?page=0` → `@RequestParam int page` |
| `@RequestBody` | Deserialize JSON body to a Java object |
| `@ResponseStatus(HttpStatus.CREATED)` | Set default response status for the method |
| `Pageable` | Spring Data auto-binds `?page=0&size=20&sort=field,asc` |

### 7.4 ResponseEntity — Full Control Over HTTP Response

```java
// Simple: return status + body
return ResponseEntity.ok(dto);                    // 200 OK with body
return ResponseEntity.noContent().build();         // 204 No Content
return ResponseEntity.created(location).body(dto); // 201 Created with Location header
return ResponseEntity.badRequest().body(error);    // 400 Bad Request with error body

// Full control: status + headers + body
return ResponseEntity
    .status(HttpStatus.OK)
    .header("X-Custom-Header", "value")
    .header(HttpHeaders.CACHE_CONTROL, "no-cache")
    .body(payloadList);
```

### 7.5 HTTP Clients: RestClient (Spring Boot 4)

In Spring Boot 3, `RestTemplate` was the standard HTTP client. In Spring Boot 4, it is **removed**. The replacement is `RestClient` (introduced in Spring Boot 3.2, primary in 4).

```java
// Spring Boot 4 — RestClient (fluent, synchronous)
@Configuration
public class HttpClientConfig {

    @Bean
    public RestClient cardHostClient(RestClient.Builder builder) {
        return builder
            .baseUrl("https://card-host.internal")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }
}

// Usage
@Service
@RequiredArgsConstructor
public class CardHostService {

    private final RestClient cardHostClient;

    public DebitResponse debit(DebitRequest request) {
        return cardHostClient.post()
            .uri("/v1/debit")
            .body(request)
            .retrieve()
            .body(DebitResponse.class);
    }
}
```

**Your service uses Retrofit2 for HTTP clients** — that's a valid and popular alternative, especially for complex API clients with interceptors, converters, and generated interfaces. Retrofit stays relevant in Spring Boot 4. The key difference is that Spring's `RestTemplate` is gone — but Retrofit is a third-party library and is unaffected.

---

## 8. The Data Layer

### 8.1 Spring Data — The Big Idea

Spring Data is an abstraction over different databases (MongoDB, JPA/SQL, Redis, Elasticsearch) that provides:
1. **Repository interfaces** — write `findByEmail(String email)`, Spring generates the query
2. **Auditing** — `@CreatedDate`, `@LastModifiedDate` auto-populated
3. **Mapping** — Java objects ↔ database documents/rows

Your service uses **Spring Data MongoDB**. The same concepts (with minor differences) apply to JPA/SQL.

### 8.2 Documents and @Document

A MongoDB document maps to a Java class annotated with `@Document`:

```java
// A real pattern from your codebase
@Document(collection = "inward_remittances")   // collection name in MongoDB
@Data
@NoArgsConstructor
public class InwardRemittance extends AbstractAuditingEntity {

    @Id
    private String id;                          // maps to MongoDB _id field

    @Indexed(unique = true)                     // creates a unique index
    private String utr;

    @Indexed
    private String bankSequenceNo;

    private InwardRemittanceStatus status;      // enum → stored as String

    private ZonedDateTime processedAt;          // custom converter handles this

    @DBRef(lazy = true)                         // reference to another document (lazy loaded)
    private Corporate corporate;

    private List<TransactionHistory> transactionHistory;  // embedded documents

    @Field("txn_ref_no")                       // field name in MongoDB differs from Java name
    private String txnRefNo;
}
```

```java
// AbstractAuditingEntity — base class all your documents extend
@Data
public class AbstractAuditingEntity implements Serializable {

    @CreatedDate
    @Field("created_at")
    protected ZonedDateTime createdAt;     // Spring Data sets this on first save

    @Field("created_by")
    protected String createdBy;

    @LastModifiedDate
    @Field("last_modified_at")
    @JsonIgnore
    protected ZonedDateTime lastModifiedAt; // Spring Data updates this on every save

    @Field("last_modified_by")
    @JsonIgnore
    protected String lastModifiedBy;
}
```

For `@CreatedDate` / `@LastModifiedDate` to work, `@EnableMongoAuditing` must be on your `DatabaseConfiguration` — it is, and the auditor identity comes from `SpringSecurityAuditorAware` which reads the current JWT principal.

### 8.3 Repositories — Queries Without Writing Queries

```java
// From your InwardRemittanceRepository.java
public interface InwardRemittanceRepository extends MongoRepository<InwardRemittance, String> {

    // Spring Data reads the method name and generates the MongoDB query
    Optional<InwardRemittance> findByUtr(String utr);
    // → db.inward_remittances.findOne({ utr: utr })

    Optional<InwardRemittance> findByBankSequenceNo(String bankSequenceNo);
    // → db.inward_remittances.findOne({ bankSequenceNo: bankSequenceNo })

    List<InwardRemittance> findAllByStatusIn(Set<InwardRemittanceStatus> statuses);
    // → db.inward_remittances.find({ status: { $in: [...] } })

    // Nested document: transactionHistory is an embedded array
    Optional<InwardRemittance> findOneByTransactionHistory_ApiRefNo(String apiRefNo);
    // → db.inward_remittances.findOne({ "transactionHistory.apiRefNo": apiRefNo })

    // Combining range + multi-value + sort + pagination:
    List<InwardRemittance> findAllByTransactionHistory_TimeStampBetweenAndStatusInOrderByCreatedAtAsc(
            ZonedDateTime from, ZonedDateTime to,
            List<InwardRemittanceStatus> statuses,
            Pageable pageable);
    // → db.inward_remittances.find({
    //     "transactionHistory.timestamp": { $gte: from, $lte: to },
    //     status: { $in: [...] }
    //   }).sort({ createdAt: 1 }).skip(page*size).limit(size)
}
```

**Built-in methods from `MongoRepository<T, ID>`:**
```java
repository.save(entity);                    // insert or update
repository.findById("id");                  // returns Optional<T>
repository.findAll();                       // all documents
repository.findAll(PageRequest.of(0, 20));  // paginated
repository.deleteById("id");
repository.existsById("id");
repository.count();
```

### 8.4 Query Derivation Keywords

| Keyword | Example | MongoDB |
|---------|---------|---------|
| `findBy` | `findByEmail` | `{ email: value }` |
| `And` | `findByEmailAndStatus` | `{ email:, status: }` |
| `Or` | `findByEmailOrPhone` | `{ $or: [{email:}, {phone:}] }` |
| `In` | `findByStatusIn(Set<>)` | `{ status: { $in: [...] } }` |
| `Between` | `findByCreatedAtBetween` | `{ createdAt: { $gte:, $lte: } }` |
| `GreaterThan` | `findByAmountGreaterThan` | `{ amount: { $gt: } }` |
| `Like` | `findByNameLike` | `{ name: /pattern/ }` |
| `OrderBy` | `...OrderByCreatedAtAsc` | `.sort({ createdAt: 1 })` |
| `Distinct` | `findDistinctByStatus` | distinct query |

### 8.5 Custom Queries with @Query

When method name derivation becomes unwieldy:

```java
// Use @Query with MongoDB JSON query
@Query("{ 'status': ?0, 'corporate.code': ?1 }")
List<InwardRemittance> findByStatusAndCorporate(String status, String corporateCode);

// With Pageable
@Query("{ 'status': { $in: ?0 }, 'createdAt': { $gte: ?1 } }")
Page<InwardRemittance> findRecentByStatuses(List<String> statuses, ZonedDateTime since, Pageable pageable);
```

### 8.6 MongoTemplate — For Complex Operations

When even `@Query` isn't enough, use `MongoTemplate` directly:

```java
@Service
@RequiredArgsConstructor
public class SomeServiceImpl {

    private final MongoTemplate mongoTemplate;

    public List<InwardRemittance> findWithComplexCriteria(String corporateCode, ZonedDateTime from) {
        Query query = new Query();
        query.addCriteria(
            Criteria.where("corporate.code").is(corporateCode)
                .and("status").in(List.of("PENDING", "PROCESSING"))
                .and("createdAt").gte(from)
        );
        query.with(Sort.by(Sort.Direction.DESC, "createdAt"));
        query.limit(100);

        return mongoTemplate.find(query, InwardRemittance.class);
    }

    // Update without fetching the document first (efficient)
    public void markAsProcessed(String id, String txnRef) {
        Query query = new Query(Criteria.where("id").is(id));
        Update update = new Update()
            .set("status", InwardRemittanceStatus.PROCESSED)
            .set("txnRefNo", txnRef)
            .currentDate("lastModifiedAt");
        mongoTemplate.updateFirst(query, update, InwardRemittance.class);
    }
}
```

---

## 9. Security

Spring Security is complex. Here's the mental model you need.

### 9.1 The Security Filter Chain

Spring Security is entirely implemented as a chain of `Filter`s. Every HTTP request passes through them in order before reaching your controller:

```
Incoming Request
      │
      ▼
SecurityContextPersistenceFilter    ← loads SecurityContext (empty for stateless apps)
      │
      ▼
SmeAuthenticationFilter             ← YOUR custom filter (validates JWT)
      │ (sets Authentication in SecurityContext on success)
      ▼
ExceptionTranslationFilter          ← catches AccessDeniedException → 403, AuthException → 401
      │
      ▼
AuthorizationFilter                 ← checks authorizeHttpRequests rules
      │
      ▼
Your @Controller method
```

### 9.2 SecurityFilterChain — Configuration

```java
// WebSecurityConfig.java (Spring Boot 3+ lambda DSL)
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        // Disable CSRF — not needed for stateless JWT APIs
        .csrf(AbstractHttpConfigurer::disable)

        // No HTTP sessions — every request is self-authenticated via JWT
        .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

        // Security headers (HSTS, CSP, no X-Frame-Options)
        .headers(h -> h.defaultsDisabled()
            .httpStrictTransportSecurity(hsts -> hsts.maxAgeInSeconds(31536000).includeSubDomains(true))
            .contentSecurityPolicy(csp -> csp.policyDirectives("none")))

        // URL-level authorization rules
        .authorizeHttpRequests(auth -> auth
            .requestMatchers(AUTH_WHITELIST).permitAll()   // public endpoints
            .anyRequest().authenticated())                 // everything else requires auth

        // Handle auth failures
        .exceptionHandling(ex -> ex.authenticationEntryPoint(authenticationEntryPoint))

        // Plug in custom JWT filter BEFORE Spring's username/password filter
        .addFilterBefore(buildSmeAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)

        .build();
}
```

### 9.3 How JWT Auth Works (Your SmeAuthenticationFilter)

```java
public class SmeAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

    // Called for every request that matches the RequestMatcher
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, ...)
            throws AuthenticationException {

        // 1. Extract JWT from Authorization header
        String token = request.getHeader("Authorization");  // "Bearer eyJhbGc..."
        String tokenPayload = tokenHandler.extract(token);   // decode/validate

        // 2. Create an "unauthenticated" token (just carries the raw JWT data)
        SmeAuthenticationToken authToken = new SmeAuthenticationToken(tokenPayload);

        // 3. Delegate to AuthenticationManager → SmeAuthenticationProvider
        return getAuthenticationManager().authenticate(authToken);
        // → SmeAuthenticationProvider validates the token, loads user roles,
        //   returns a FULLY authenticated SmeAuthenticationToken
    }

    // Called by Spring Security when authenticate() returns a valid Authentication
    @Override
    protected void successfulAuthentication(HttpServletRequest req, HttpServletResponse res,
                                             FilterChain chain, Authentication auth)
            throws IOException, ServletException {
        SecurityContextHolder.getContext().setAuthentication(auth);
        chain.doFilter(req, res);   // IMPORTANT: continue to next filter / controller
    }
}
```

**Key concept: `Authentication` object.** After successful auth, Spring Security stores an `Authentication` in `SecurityContextHolder`. It contains:
- `getPrincipal()` → the username/userId
- `getAuthorities()` → the roles/permissions (e.g., `["ROLE_LOAN_VIEW", "ROLE_PAYOUT_INITIATE"]`)
- `isAuthenticated()` → true

Every subsequent check (like `@Secured`) reads from this object.

### 9.4 Method Security — @Secured

```java
// In WebSecurityConfig:
@EnableMethodSecurity(securedEnabled = true)

// On controller methods:
@GetMapping("v1/loan")
@Secured(AuthoritiesConstants.Code.LOAN_DOCUMENT_VIEW)   // "ROLE_LOAN_DOCUMENT_VIEW"
public ResponseEntity<List<LoanDTO>> getAllLoan(...) { ... }
```

When `@Secured` is used, Spring AOP wraps the controller method in a proxy. Before executing it, the proxy checks if the current `Authentication` in `SecurityContextHolder` has the required role. If not, it throws `AccessDeniedException` → caught by `ExceptionTranslationFilter` → returns HTTP 403.

**@Secured vs @PreAuthorize:**

```java
// @Secured — simple role check
@Secured("ROLE_ADMIN")

// @PreAuthorize — supports SpEL (Spring Expression Language)
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.name")
// → user must be ADMIN OR the userId param must equal their own userId
```

---

## 10. Messaging — Spring Kafka

### 10.1 The Mental Model

```
sme-service-apis (Producer)           Kafka Broker (Topics)         sme-service-workers (Consumer)
┌────────────────────────┐            ┌──────────────────┐          ┌────────────────────────────┐
│                        │            │ payouts-debit-    │          │                            │
│ PayoutsController      │──publish──→│  topic           │──pull───→│ PayoutsTransactionsWorker  │
│ POST /payouts/initiate │            │                  │          │ @KafkaListener              │
│                        │            │ payouts-success-  │          │                            │
│                        │──publish──→│  topic           │──pull───→│ PayoutsTransactionDelayed  │
└────────────────────────┘            └──────────────────┘          └────────────────────────────┘
         │                                                                        │
         │   KafkaTemplate.send("topic", payload)              @KafkaListener    │
         │   → serializes to JSON                              → deserializes    │
         │   → sends to broker                                 → calls method    │
```

### 10.2 Producing Messages

```java
@Service
@RequiredArgsConstructor
public class PayoutsKafkaProducer {

    private final KafkaTemplate<String, Object> kafkaTemplate;
    private final KafkaUtils kafkaUtils;

    public void publishDebitEvent(PayoutsTransaction transaction) {
        String topic = kafkaUtils.getTopicName(KafkaTopics.PAYOUTS_DEBIT_TOPIC);

        // send(topic, key, value)
        // key = used for partition assignment (same key → same partition → ordered)
        // value = serialized as JSON by JsonSerializer in ProducerFactory
        kafkaTemplate.send(topic, transaction.getCorporateId(), transaction);
    }

    // With callback to handle success/failure
    public void publishWithCallback(String topic, Object payload) {
        ListenableFuture<SendResult<String, Object>> future = kafkaTemplate.send(topic, payload);
        future.addCallback(
            result -> log.info("Published to partition {} offset {}",
                result.getRecordMetadata().partition(),
                result.getRecordMetadata().offset()),
            ex -> log.error("Failed to publish: {}", ex.getMessage())
        );
    }
}
```

**Topic naming convention in your service:**

```java
// KafkaUtils.getTopicName() prepends the environment prefix
// e.g., "com.sme.dev." + "payouts.debit" = "com.sme.dev.payouts.debit"

public String getTopicName(String topicSuffix) {
    return topicPrefix + topicSuffix;
}
```

This means the same code runs in dev (prefix `com.sme.dev.`) and prod (prefix `com.sme.prod.`) without change.

### 10.3 Consuming Messages

```java
// From JobApplicationReceivedNotifyWorker.java in your codebase
@Service
@Slf4j
public class JobApplicationReceivedNotifyWorker {

    @Inject private CorporateService corporateService;

    @KafkaListener(
        topics = "#{kafkaUtils.getTopicName('" + KafkaExternalTopics.SME_JOB_APPLICATION_SUBMISSION_TOPIC + "')}",
        groupId = "#{kafkaUtils.getGroupId('" + KafkaGroups.SME_JOB_APPLICATION_SUBMISSION_GROUP + "')}"
    )
    public void receiveJobApplication(
            @Header(value = "uuid", required = false) String uuid,  // from Kafka message headers
            @Payload JobApplicantReceivedDto payload) {             // deserialized JSON body

        if (payload == null || StringUtils.isBlank(payload.getCorporateCode())) {
            log.info("Skipping invalid message");
            return;   // acknowledge and skip — don't throw (would trigger retry)
        }

        try {
            CorporateDTO corp = corporateService.getSmeDataByCode(payload.getCorporateCode());
            sendNotification(corp, payload);
        } catch (IllegalStateException e) {
            log.info("Duplicate event, skipping: {}", e.getMessage());  // idempotency guard
        } catch (Exception e) {
            log.error("Processing failed: {}", e.getMessage(), e);
            // Don't rethrow — the DefaultErrorHandler in KafkaConfig retries once,
            // then moves the message to the dead-letter topic (if configured)
        }
    }
}
```

**SpEL in `@KafkaListener`:** `#{kafkaUtils.getTopicName(...)}` evaluates a Spring bean method at startup. This is how your topics resolve to the environment-prefixed names.

### 10.4 Error Handling — The DefaultErrorHandler

```java
// From KafkaConfig.java
factory.setCommonErrorHandler(
    new DefaultErrorHandler(new FixedBackOff(1000L, 1L))
    // FixedBackOff(intervalMs, maxAttempts): retry once after 1 second
);
```

What happens when a `@KafkaListener` method throws:
1. Spring Kafka catches the exception
2. `DefaultErrorHandler` waits 1000ms and retries once
3. If the retry also fails, Spring Kafka **commits the offset** (skips the message) and logs the error
4. Optional: configure a `DeadLetterPublishingRecoverer` to send failed messages to a `-dlt` topic

**Poison pill prevention:** Your `ConsumerFactory` wraps `JsonDeserializer` in `ErrorHandlingDeserializer`. If a message can't be deserialized (e.g., malformed JSON), instead of killing the consumer thread, it calls the error handler with a `DeserializationException` — the message is skipped safely.

### 10.5 Consumer Group and Partitions

```
Topic: com.sme.prod.payouts.debit (6 partitions)
Consumer Group: sme-prod-payouts-worker (3 instances)

Instance 1: consumes partitions 0, 1
Instance 2: consumes partitions 2, 3
Instance 3: consumes partitions 4, 5

Kafka assigns partitions automatically. When you add a 4th instance,
Kafka rebalances: each instance gets 1-2 partitions.
```

**`concurrency` in `KafkaListenerContainerFactory`:** In your `KafkaConfig`, `factory.setConcurrency(concurrency)` sets how many threads a single worker instance uses to consume. If concurrency=3 and there are 6 partitions, the one pod consumes all 6 partitions with 3 threads (2 partitions per thread).

---

## 11. AOP — Cross-Cutting Concerns

**AOP** (Aspect-Oriented Programming) lets you add behaviour to methods across your codebase **without changing those methods**. Spring uses it internally for `@Transactional`, `@Secured`, `@Cacheable`, `@Retryable`, and `@Observed`.

### The Mental Model

```
Without AOP — you have to write the same boilerplate in every method:

public CorporateDTO getCorporate(String code) {
    log.info("Cache miss, fetching...");            // boilerplate
    long start = System.currentTimeMillis();        // boilerplate
    try {
        CorporateDTO result = repository.findByCode(code);
        cache.put(code, result);                    // boilerplate
        return result;
    } finally {
        log.info("Took {}ms", System.currentTimeMillis() - start); // boilerplate
    }
}

// 300 service methods × this boilerplate = unmaintainable
```

```
With AOP — you declare the cross-cutting behaviour once as an Aspect.
The @Cacheable annotation on the method is just a marker.
Spring generates a PROXY class that wraps your method:

@Cacheable(value = "corporate-single", key = "#p0")
public CorporateDTO getCorporateForPayouts(String code) {
    return repository.findByCode(code);  // clean — just business logic
}

// Spring generates (conceptually):
public CorporateDTO getCorporateForPayouts_PROXY(String code) {
    if (cache.contains(code)) return cache.get(code);   // AOP adds this
    CorporateDTO result = original.getCorporateForPayouts(code);
    cache.put(code, result);                            // AOP adds this
    return result;
}
```

### How Spring AOP Works — Proxies

Spring creates a **proxy object** that wraps your bean. When you inject `CorporateService`, you actually get the proxy, not `CorporateServiceImpl` directly. The proxy intercepts method calls and runs the aspect logic:

```
Caller → Proxy → Aspect logic (before) → Your method → Aspect logic (after) → Caller
```

This means AOP **only works on public methods** called through the Spring proxy. Calling a method from within the same class bypasses the proxy:

```java
@Service
public class CorporateServiceImpl implements CorporateService {

    @Cacheable("corp")
    public CorporateDTO getCorporateForPayouts(String code) { ... }

    public void someOtherMethod(String code) {
        // WRONG: calls this.getCorporateForPayouts() — bypasses proxy → no cache
        CorporateDTO dto = getCorporateForPayouts(code);

        // RIGHT: inject self reference, or restructure
        CorporateDTO dto = corporateService.getCorporateForPayouts(code); // via proxy → cache works
    }
}
```

### @Cacheable and @CacheEvict (from your codebase)

```java
// CorporateServiceImpl.java
@Cacheable(value = "corporate-single", key = "#p0")
public CorporateDTO getCorporateForPayouts(String corporateCode) {
    // Spring checks the cache first
    // If miss: calls this method, caches result, returns it
    // If hit: returns cached value, this method body NEVER RUNS
    return repository.findByCode(corporateCode);
}

@CacheEvict(value = "corporate-single", key = "#p0")
public void corporateCacheEvict(String corporateCode) {
    // Spring removes the cached entry for this key
    // This method body is intentionally EMPTY — AOP does the work
}
```

**Cache key expressions:**
```java
@Cacheable(value = "loan", key = "#corporateCode + '-' + #status")
@Cacheable(value = "corp", key = "#p0")              // #p0 = first parameter
@Cacheable(value = "corp", key = "#root.args[0]")    // same, different syntax
@Cacheable(value = "corp", condition = "#code != null")  // only cache if condition is true
@Cacheable(value = "corp", unless = "#result == null")   // don't cache null results
```

---

## 12. Exception Handling

### Global Exception Handler with @RestControllerAdvice

Instead of try-catching exceptions in every controller method, define one class that handles all exceptions globally:

```java
@RestControllerAdvice   // = @ControllerAdvice + @ResponseBody
@Slf4j
public class GlobalExceptionHandler {

    // Handle your domain exceptions
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(ValidationException ex) {
        log.warn("Validation error: {}", ex.getMessage());
        return new ErrorResponse("VALIDATION_ERROR", ex.getMessage());
    }

    // Handle Spring Security auth failures
    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleAccessDenied(AccessDeniedException ex) {
        return new ErrorResponse("ACCESS_DENIED", "Insufficient permissions");
    }

    // Handle missing @RequestParam / @PathVariable
    @ExceptionHandler(MissingServletRequestParameterException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleMissingParam(MissingServletRequestParameterException ex) {
        return new ErrorResponse("MISSING_PARAMETER", ex.getParameterName() + " is required");
    }

    // Catch-all
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        return new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred");
    }
}

// Error response DTO — using Java record (Java 16+)
public record ErrorResponse(String code, String message) {}
```

### Spring Boot 4: Problem Details (RFC 9457)

Spring Boot 4 standardises error responses via **Problem Details** (formerly RFC 7807):

```java
// application.yml
spring:
  mvc:
    problemdetails:
      enabled: true  # enables RFC 9457 problem details format

// Now Spring automatically returns this for validation errors, 404s, etc.:
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "corporateCode must not be blank",
  "instance": "/sme/v1/payouts/initiate"
}

// Your custom exceptions can extend ProblemDetail:
@ExceptionHandler(ValidationException.class)
public ProblemDetail handle(ValidationException ex) {
    ProblemDetail detail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
    detail.setProperty("errorCode", ex.getCode());
    return detail;
}
```

---

## 13. Validation

Spring Boot integrates Jakarta Bean Validation. Annotate your DTOs, enable validation in controllers.

```java
// DTO with validation annotations
public record PayoutInitiateRequest(
    @NotBlank(message = "Corporate code is required")
    String corporateCode,

    @NotEmpty(message = "Employee list cannot be empty")
    @Size(max = 1000, message = "Cannot process more than 1000 employees at once")
    List<@Valid EmployeePayoutDTO> employees,

    @NotNull
    @FutureOrPresent(message = "Payout date cannot be in the past")
    LocalDate payoutDate
) {}

public record EmployeePayoutDTO(
    @NotBlank String employeeId,
    @Min(value = 1, message = "Amount must be at least ₹1")
    @Max(value = 10_000_000, message = "Amount exceeds single-transfer limit")
    double amount
) {}
```

```java
// Controller — add @Valid to trigger validation
@PostMapping("v1/payouts/initiate")
public ResponseEntity<Void> initiatePayout(@Valid @RequestBody PayoutInitiateRequest request) {
    // If validation fails, Spring throws MethodArgumentNotValidException → 400
    payoutsService.initiate(request);
    return ResponseEntity.accepted().build();
}
```

```java
// Service-level validation with @Validated (validates @Valid on method params)
@Service
@Validated
public class PayoutsServiceImpl implements PayoutsService {

    @Override
    public void initiate(@Valid PayoutInitiateRequest request) {
        // Also validated here — useful for service methods called internally
    }
}
```

**Validation annotations reference:**

| Annotation | Validates |
|------------|-----------|
| `@NotNull` | Not null (but can be empty string) |
| `@NotBlank` | Not null AND not empty AND not whitespace |
| `@NotEmpty` | Not null AND not empty (for strings and collections) |
| `@Size(min=, max=)` | String/collection length |
| `@Min`, `@Max` | Numeric range |
| `@Email` | Email format |
| `@Pattern(regexp=)` | Regex match |
| `@Future`, `@Past`, `@FutureOrPresent` | Date/time relative validation |
| `@Valid` | Recursively validate nested object |
| `@Positive`, `@PositiveOrZero` | > 0 or ≥ 0 |

---

## 14. Caching

Caching in Spring is an abstraction — you annotate methods, Spring handles the caching with whichever provider is configured (EhCache, Caffeine, Redis, etc.).

```java
// Enable caching (already in your SmeApplication.java)
@EnableCaching
```

```java
// Configure the cache provider — Caffeine (in-memory, fast, configurable eviction)
@Configuration
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager();
        manager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)               // max entries
            .expireAfterWrite(5, MINUTES)    // TTL
            .recordStats());                 // exposes hit/miss metrics to Micrometer
        return manager;
    }
}
```

```yaml
# Or configure via application.yml (Spring Boot auto-configuration)
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=1000,expireAfterWrite=5m
```

**Cache annotations summary:**

| Annotation | When to use |
|------------|------------|
| `@Cacheable` | Cache the result of a method (read-through) |
| `@CachePut` | Always execute the method AND update the cache |
| `@CacheEvict` | Remove entries from the cache (on write/delete) |
| `@Caching` | Combine multiple cache annotations on one method |

**Important caveat for multi-instance deployments**: Your service runs as multiple pods. In-memory cache (Caffeine) is per-pod. If one pod evicts a cache entry, other pods still have the stale cached value. For true cache consistency across pods, use **Redis**. Your service uses in-memory cache today, which is acceptable for read-heavy data that can tolerate brief stale reads.

---

## 15. Scheduling & Async

### @Scheduled — Run Code on a Timer

```java
@EnableScheduling  // on your @SpringBootApplication or @Configuration

@Service
public class BalanceSyncScheduler {

    @Scheduled(cron = "0 0 * * * *")          // every hour at :00
    public void syncCorporateBalances() {
        log.info("Starting hourly balance sync");
        corporateService.syncAllBalances();
    }

    @Scheduled(fixedDelay = 30_000)            // 30 seconds after previous run completes
    public void processPendingRemittances() {
        remittanceService.processAllPending();
    }

    @Scheduled(fixedRate = 60_000,             // every 60 seconds
               initialDelay = 10_000)          // wait 10 seconds after startup
    public void healthCheck() {
        externalService.ping();
    }
}
```

**Cron expression format:**
```
second minute hour day-of-month month day-of-week

0 0 9 * * MON-FRI     → 9:00 AM every weekday
0 0 * * * *            → every hour
0 */15 * * * *         → every 15 minutes
0 0 10 25 * *          → 10 AM on the 25th of every month (salary day?)
```

### @Async — Run Code in a Background Thread

```java
@EnableAsync  // on your @SpringBootApplication or @Configuration

@Service
public class NotificationService {

    // Returns immediately; method runs in a separate thread
    @Async
    public void sendEmailAsync(String to, String subject, String body) {
        emailClient.send(to, subject, body);  // doesn't block the caller
    }

    // If you need the result later, return CompletableFuture
    @Async
    public CompletableFuture<Boolean> verifyPanAsync(String pan) {
        boolean valid = karzaClient.verifyPan(pan);
        return CompletableFuture.completedFuture(valid);
    }
}

// Usage in a controller:
payoutController.initiateResponse → 200 OK (immediately)
notificationService.sendEmailAsync(email, subject, body);  // runs in background
```

**Thread pool for @Async:**
```java
@Configuration
public class AsyncConfig {

    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Async("taskExecutor")   // use the named executor
public void doWork() { }
```

**In Spring Boot 4 with Java 21 virtual threads:**
```java
// With virtual threads enabled, @Async uses virtual threads automatically
// No thread pool configuration needed — virtual threads are cheap (one per task)
spring:
  threads:
    virtual:
      enabled: true
```

---

## 16. Testing

### 16.1 The Testing Pyramid

```
            ┌──────────────────┐
            │   E2E / Contract  │  ← slowest, fewest, most realistic
            │      Tests       │
          ┌─┴──────────────────┴─┐
          │  Integration Tests    │  ← test multiple layers together
          │  @SpringBootTest     │
        ┌─┴────────────────────┴─┐
        │      Unit Tests        │  ← fast, isolated, most numerous
        │   Mockito / JUnit      │
        └────────────────────────┘
```

Your codebase has unit tests in `sme-service-commons/src/test/`. They use Mockito with no Spring context — fast to run, test one class at a time.

### 16.2 Unit Tests — Mockito

```java
// Pattern used in your codebase
@ExtendWith(MockitoExtension.class)
class CorporateServiceImplTest {

    @Mock
    private SmeRepository smeRepository;          // a fake SmeRepository
    @Mock
    private KafkaTemplate<String, Object> kafkaTemplate;

    @InjectMocks
    private CorporateServiceImpl corporateService; // inject the mocks above into this

    @Test
    void getCorporate_whenCodeExists_returnsDto() {
        // Arrange
        Sme sme = TestData.buildSme("CORP-001");
        when(smeRepository.findByCode("CORP-001")).thenReturn(Optional.of(sme));

        // Act
        CorporateDTO result = corporateService.getCorporate("CORP-001");

        // Assert
        assertThat(result).isNotNull();
        assertThat(result.getCode()).isEqualTo("CORP-001");
        verify(smeRepository, times(1)).findByCode("CORP-001");
    }

    @Test
    void getCorporate_whenCodeMissing_throwsException() {
        when(smeRepository.findByCode("MISSING")).thenReturn(Optional.empty());

        assertThatThrownBy(() -> corporateService.getCorporate("MISSING"))
            .isInstanceOf(ValidationException.class)
            .hasMessageContaining("Corporate not found");
    }
}
```

**Setting private fields (your codebase uses this for `@Value` fields):**
```java
@BeforeEach
void setup() {
    ReflectionTestUtils.setField(corporateService, "maxRetries", 3);
    ReflectionTestUtils.setField(corporateService, "baseUrl", "http://test-host");
}
```

### 16.3 Integration Tests — @SpringBootTest

```java
// Boots the full Spring context — slow, but tests real wiring
@SpringBootTest
@ActiveProfiles("test")
class PayoutsControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;  // simulates HTTP without actually starting a server

    @MockBean
    private CardHostClient cardHostClient;   // replace real HTTP client with mock

    @Test
    void initiatePayouts_validRequest_returns202() throws Exception {
        // Arrange
        when(cardHostClient.debit(any())).thenReturn(successResponse());

        // Act + Assert
        mockMvc.perform(post("/sme/v1/payouts/initiate")
                .contentType(MediaType.APPLICATION_JSON)
                .header("Authorization", "Bearer " + testJwt())
                .content(objectMapper.writeValueAsString(validRequest())))
            .andExpect(status().isAccepted())
            .andExpect(jsonPath("$.status").value("PENDING"));
    }
}
```

### 16.4 Slice Tests — Test One Layer

Spring Boot provides "slice" test annotations that boot only part of the context — faster than `@SpringBootTest`:

| Annotation | What It Loads |
|------------|--------------|
| `@WebMvcTest(MyController.class)` | Only the web layer (controllers, filters) — no service, no DB |
| `@DataMongoTest` | Only Spring Data MongoDB — repositories, entities, converters |
| `@JsonTest` | Only Jackson serialization/deserialization |
| `@MockBean` | Replaces a bean in the context with a Mockito mock |

```java
// Test just the controller layer — fast
@WebMvcTest(LoanController.class)
class LoanControllerTest {

    @Autowired private MockMvc mockMvc;

    @MockBean private CorporateService corporateService;  // mock the dependency

    @Test
    @WithMockUser(roles = "LOAN_DOCUMENT_VIEW")   // fake security context
    void getAllLoan_returnsList() throws Exception {
        when(corporateService.getAllLoanRequests(any()))
            .thenReturn(new PageImpl<>(List.of(new LoanDTO())));

        mockMvc.perform(get("/v1/loan"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray());
    }
}
```

---

## 17. Spring Boot 3 → 4: What Changes

This is what you need to know to write code that works in Spring Boot 4 (and won't need major rewrites).

### Java 21 Is the Minimum

No Java 17 support in Spring Boot 4. This means you get these features for free:
- Records (stable since Java 16)
- Pattern matching for `switch` (stable since Java 21)
- Virtual threads (stable since Java 21)
- Sequenced collections (`SequencedList`, `SequencedMap`)

### RestTemplate Is Gone

```java
// Spring Boot 3: deprecated warning
@Bean
public RestTemplate restTemplate() {    // shows deprecation
    return new RestTemplate();
}

// Spring Boot 4: does not exist — remove this from SmeApplication.java
// Use RestClient instead:
@Bean
public RestClient.Builder restClientBuilder() {
    return RestClient.builder();
}
```

**But Retrofit2 is fine** — it's a third-party library, not a Spring class. Your Retrofit-based HTTP clients (YBL, ICICI, Karza, etc.) don't need any changes.

### SecurityFilterChain Style Is Already Correct

Your `WebSecurityConfig` already uses the Spring Boot 3/4 lambda DSL. No changes needed. The old `WebSecurityConfigurerAdapter` (which was removed in Spring Boot 3) is still absent from your codebase — good.

### @SpringBootApplication Now Finds Beans Better

The `@ComponentScan({"com.niyo.sme.commons.*", "com.niyo*"})` workaround in your `SmeApplication` exists because of the multi-module structure. Spring Boot 4's auto-configuration improvements may reduce the need for explicit `@ComponentScan` — worth testing after upgrading.

### Jakarta EE 11 — No Change If You're Already on Jakarta

Your codebase uses `jakarta.inject.Inject`. Spring Boot 3 already moved from `javax.*` to `jakarta.*`. Spring Boot 4 upgrades to Jakarta EE 11, which means newer spec versions but the same package names. No code changes needed for the Jakarta migration since you already did it.

### Structured Logging Built-In

```yaml
# Spring Boot 4 — no logstash-logback-encoder needed
logging:
  structured:
    format:
      console: ecs   # Elastic Common Schema JSON
      # or: logstash, gelf, json
```

Your current setup uses `logstash-logback-encoder` in `logback-spring.xml`. In Spring Boot 4, you can replace that dependency and the logback XML with the single YAML line above.

### Virtual Threads Default for Tomcat and Kafka

```yaml
# Spring Boot 4 — virtual threads are on by default
# You don't need this in SB4, but it's how you opt in on SB3:
spring:
  threads:
    virtual:
      enabled: true
```

With virtual threads, your Tomcat server handles far more concurrent requests without thread pool exhaustion. Your Kafka consumers also use virtual threads — meaning `MAX_POLL_RECORDS_CONFIG = 50` may be worth tuning upward.

### OTel First-Class

```yaml
# Spring Boot 4 — built-in OTel, no micrometer-tracing-bridge-otel needed
management:
  tracing:
    enabled: true
    exporter:
      otlp:
        endpoint: http://otel-collector:4318
```

The `opentracing-util` dependency in your pom.xml is the old pre-OTel API — remove it in Spring Boot 4.

---

## 18. Java 21 Features That Matter for Spring Boot

### Records — Immutable Data Carriers

```java
// Old DTO style (your codebase has many of these)
@Data
@AllArgsConstructor
@NoArgsConstructor
public class DebitRequest {
    private String transactionId;
    private String corporateId;
    private double amount;
    private TransferType transferType;
}

// Java 21 record — same thing, 1 line, immutable, auto-equals/hashCode/toString
public record DebitRequest(
    String transactionId,
    String corporateId,
    double amount,
    TransferType transferType
) {}

// With validation (Spring Boot 4 supports @Valid on record components):
public record DebitRequest(
    @NotBlank String transactionId,
    @NotBlank String corporateId,
    @Positive double amount,
    @NotNull TransferType transferType
) {}
```

**Use records for**: DTOs, configuration classes (`@ConfigurationProperties`), query result objects, Kafka message payloads.

**Don't use records for**: Domain entities (MongoDB documents need mutable state for auditing fields), objects that need inheritance.

### Pattern Matching for Switch

```java
// Old style (your workers use if/else chains)
String status;
if (response.getStatus() == Status.SUCCESS) {
    status = "success";
} else if (response.getStatus() == Status.ERROR || response.getStatus() == Status.INVALID) {
    status = "failed";
} else if (response.getStatus() == Status.PENDING) {
    status = "pending";
} else {
    status = "unknown";
}

// Java 21 pattern matching switch — exhaustive, no break needed
String status = switch (response.getStatus()) {
    case SUCCESS                -> "success";
    case ERROR, INVALID         -> "failed";
    case PENDING                -> "pending";
    case null                   -> "error";
    default                     -> "unknown";
};
```

### Virtual Threads — What They Mean for You

A **virtual thread** is a lightweight thread managed by the JVM, not the OS. You can create millions of them. They're designed for I/O-bound work (exactly what your service does — MongoDB queries, HTTP calls, Kafka produce/consume).

```java
// Traditional platform threads: limited by OS
// Typical Tomcat default: 200 threads
// At 200 concurrent requests, the 201st request waits

// Virtual threads: JVM manages them, millions possible
// At 10,000 concurrent requests: JVM creates 10,000 virtual threads
// Each one parks when waiting for I/O, freeing the carrier thread

// With Spring Boot 4, this is automatic — no code change
// Tomcat uses virtual threads by default
// Kafka consumers use virtual threads by default
```

**Impact on your service:**
- `MAX_POLL_RECORDS_CONFIG = 50` (your current Kafka setting) can likely be increased — each message processing uses a virtual thread, so 200 concurrent messages ≠ 200 blocked OS threads
- Thread pool metrics (`executor.active`, `executor.pool.size`) become less meaningful — monitor request concurrency and Kafka lag instead
- `MDC.put()`/`MDC.clear()` still works correctly in Java 21 virtual threads — inherits from parent at creation

### Sequenced Collections (Java 21)

```java
// Java 21 — clear, consistent first/last access on any ordered collection
List<InwardRemittance> remittances = findAll();

// Old way:
InwardRemittance first = remittances.get(0);
InwardRemittance last  = remittances.get(remittances.size() - 1);

// Java 21 SequencedCollection:
InwardRemittance first = remittances.getFirst();
InwardRemittance last  = remittances.getLast();

// Also: SequencedMap for LinkedHashMap
Map<String, String> map = new LinkedHashMap<>();
map.firstEntry();   // instead of map.entrySet().iterator().next()
map.lastEntry();
```

---

## 19. Glossary

| Term | Definition |
|------|-----------|
| **@Async** | Spring annotation: execute method in a background thread pool (or virtual thread in SB4) |
| **@Autowired** | Spring annotation for dependency injection. Use constructor injection instead. |
| **@Bean** | Marks a method in a `@Configuration` class as producing a Spring-managed bean |
| **@Cacheable** | AOP annotation: cache the method return value. Spring skips the method on cache hits |
| **@CacheEvict** | AOP annotation: remove a cache entry. Method body can be empty — AOP does the work |
| **@Component** | Generic Spring bean marker. Specializations: `@Service`, `@Repository`, `@Controller` |
| **@ConfigurationProperties** | Bind a group of properties to a class. Better than multiple `@Value` fields |
| **@ConditionalOnMissingBean** | Auto-configuration backs off if the user has defined their own bean of this type |
| **@Document** | Spring Data MongoDB: maps a Java class to a MongoDB collection |
| **@EnableAutoConfiguration** | Triggers Spring Boot's classpath scanning and conditional auto-configuration |
| **@EnableKafka** | Activates scanning for `@KafkaListener` annotations |
| **@EnableMethodSecurity** | Activates `@Secured`, `@PreAuthorize`, `@PostAuthorize` on methods |
| **@EnableMongoAuditing** | Activates automatic population of `@CreatedDate`, `@LastModifiedDate` fields |
| **@Inject** | Jakarta standard annotation for dependency injection. Equivalent to `@Autowired` |
| **@KafkaListener** | Marks a method as a Kafka message consumer for specified topics and group |
| **@PostConstruct** | Called once after all dependencies are injected. Use for initialization that needs injected values |
| **@PreDestroy** | Called just before the Spring context shuts down. Use for cleanup |
| **@Primary** | When multiple beans of the same type exist, this one is injected by default |
| **@Qualifier** | Specifies which bean to inject when multiple candidates exist |
| **@Repository** | `@Component` specialization for data access layers. Enables persistence exception translation |
| **@RequestBody** | Deserialize the HTTP request body (JSON) into a Java object |
| **@RequestMapping** | Maps a URL pattern to a controller class or method |
| **@RequiredArgsConstructor** | Lombok: generates a constructor with all `final` fields as parameters. Enables constructor injection without boilerplate |
| **@RestController** | `@Controller` + `@ResponseBody`: every method return value is serialized to JSON |
| **@RestControllerAdvice** | Global exception handler for all `@RestController` classes |
| **@Retryable** | Spring Retry AOP annotation: retry the method on specified exception types |
| **@Scheduled** | Schedule a method to run on a cron expression or fixed rate/delay |
| **@Secured** | Method security: restrict access to users with specific roles |
| **@Service** | `@Component` specialization for business logic layer. Semantically signals service |
| **@SpringBootApplication** | Combines `@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| **@Transactional** | AOP annotation: wrap method in a DB transaction. Roll back on unchecked exceptions |
| **@Valid** | Trigger Jakarta Bean Validation on a method parameter or field |
| **@Value** | Inject a property value into a field. Supports default: `${prop:default}` |
| **ApplicationContext** | The Spring IoC container — holds all beans and their wiring |
| **AOP** | Aspect-Oriented Programming: add cross-cutting behaviour (caching, security, logging) without modifying target methods |
| **Auto-Configuration** | Spring Boot mechanism: activate configuration conditionally based on classpath and existing beans |
| **Bean** | Any object managed by the Spring IoC container |
| **Bean Scope** | Lifetime of a bean: Singleton (default), Prototype, Request, Session |
| **CompositeMeterRegistry** | Micrometer: fans out metric writes to multiple backends simultaneously |
| **DispatcherServlet** | Spring MVC front controller: routes HTTP requests to the right `@Controller` method |
| **DI** | Dependency Injection: the IoC container provides dependencies, classes don't create them |
| **EmbeddedServer** | Tomcat (or Netty for reactive) bundled inside the JAR. No external server needed |
| **Filter** | Jakarta Servlet: intercepts HTTP requests/responses at the Servlet level (before Spring MVC) |
| **HikariCP** | The default JDBC connection pool in Spring Boot. Used for SQL databases |
| **Interceptor** | Spring MVC: intercepts requests at the handler level (after Spring Security filters) |
| **IoC** | Inversion of Control: the framework (Spring) creates and wires your objects, not your code |
| **Kafka Consumer Group** | A set of consumers sharing topic consumption. Kafka assigns partitions across the group |
| **MongoRepository** | Spring Data interface providing CRUD + pagination for MongoDB. Extend it for your entity |
| **MongoTemplate** | Lower-level MongoDB access for complex queries that can't be expressed as derived methods |
| **Pageable** | Spring Data: encapsulates `page`, `size`, `sort`. Auto-bound from `?page=0&size=20` |
| **Problem Details** | RFC 9457 standard error response format. Built-in to Spring Boot 4 |
| **Proxy** | AOP: Spring wraps your bean in a proxy object. Method calls go through the proxy first |
| **Record** | Java 16+ immutable data class. Replaces Lombok `@Data` for DTOs and value objects |
| **RestClient** | Spring Boot 3.2+ HTTP client. Replaces deprecated `RestTemplate` in Spring Boot 4 |
| **SecurityContext** | Thread-local storage for the current user's `Authentication` object |
| **SecurityContextHolder** | Holder for the `SecurityContext`. Read it anywhere: `SecurityContextHolder.getContext().getAuthentication()` |
| **SecurityFilterChain** | The ordered chain of Spring Security filters. Configured via `HttpSecurity` DSL |
| **Singleton** | Default Spring bean scope: one instance shared by the entire application |
| **Slice Test** | A partial Spring context for testing one layer: `@WebMvcTest`, `@DataMongoTest`, etc. |
| **SpEL** | Spring Expression Language: `#{bean.method()}`, `${property}` — used in annotations |
| **Spring Data** | Abstraction over databases: derived query methods, auditing, pagination. Backends: Mongo, JPA, Redis, etc. |
| **Spring Framework** | The core engine: IoC, DI, MVC, Security, AOP. Spring Boot configures it automatically |
| **Starter** | A curated dependency group: `spring-boot-starter-web` pulls in Tomcat + MVC + Jackson + validation |
| **Virtual Threads** | Java 21 JVM-managed lightweight threads. Default in Spring Boot 4 for web and Kafka |

---

*For observability concepts (metrics, traces, logs, Prometheus, OpenTelemetry), see [`observability-101.md`](./observability-101.md).*
*For implementation patterns specific to the SME service, see [`observability-guide.md`](./observability-guide.md).*
