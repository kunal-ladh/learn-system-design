# Spring Boot 4 + Java 21 LTS Migration & Optimization Guide

> **Service:** bharat-agent-onboarding  
> **Current Stack:** Spring Boot 3.5.13, Java 17, MongoDB, Retrofit/OkHttp, Kafka, AWS  
> **Target Stack:** Spring Boot 4.x, Java 21 LTS, Virtual Threads, RestClient, Resilience4j  
> **Created:** 2026-05-05

---

## Table of Contents

1. [Prerequisites & Compatibility](#1-prerequisites--compatibility)
2. [Java 21 — Language & Runtime Changes](#2-java-21--language--runtime-changes)
3. [Virtual Threads — The Game Changer](#3-virtual-threads--the-game-changer)
4. [Replace Retrofit/OkHttp with RestClient & @HttpExchange](#4-replace-retrofitokhttp-with-restclient--httpexchange)
5. [Resilience4j Circuit Breakers](#5-resilience4j-circuit-breakers)
6. [MongoDB Connection Pool Tuning](#6-mongodb-connection-pool-tuning)
7. [Observability — OpenTelemetry & Micrometer](#7-observability--opentelemetry--micrometer)
8. [Spring Security 7 Changes](#8-spring-security-7-changes)
9. [DTOs — Records & Immutability](#9-dtos--records--immutability)
10. [Configuration Properties — Type-Safe Config](#10-configuration-properties--type-safe-config)
11. [Dependency Cleanup](#11-dependency-cleanup)
12. [Structured Concurrency](#12-structured-concurrency)
13. [GraalVM Native Image (Optional)](#13-graalvm-native-image-optional)
14. [Global Exception Handling — ProblemDetail (RFC 7807)](#14-global-exception-handling--problemdetail-rfc-7807)
15. [Structured Logging](#15-structured-logging)
16. [Migration Checklist](#16-migration-checklist)

---

## 1. Prerequisites & Compatibility

### Hard Requirements for Spring Boot 4

| Requirement | Minimum Version |
|-------------|----------------|
| Java | 21 (LTS) — Java 17 will NOT compile |
| Gradle | 8.14+ or 9.x |
| Maven | 3.9+ |
| Spring Framework | 7.0 |
| Spring Security | 7.0 |
| Jakarta EE | 10 (jakarta.* namespace) |

### pom.xml Changes

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.6</version> <!-- Latest stable as of May 2026 -->
    <relativePath/>
</parent>

<properties>
    <java.version>21</java.version>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
    <!-- Use release flag for cross-compilation safety -->
    <maven.compiler.release>21</maven.compiler.release>
</properties>
```

### Key Breaking Changes in Boot 4

1. **All APIs deprecated in Boot 3.x are removed** — no grace period
2. **Virtual threads enabled by default** — changes concurrency model
3. **Spring Security lambda DSL is the only approach** — method chaining removed
4. **`spring.factories` auto-configuration loading removed** — uses `AutoConfiguration.imports`
5. **Observation API (Micrometer) replaces legacy metrics wiring**

---

## 2. Java 21 — Language & Runtime Changes

### 2.1 Pattern Matching for switch (finalized)

```java
// BEFORE (Java 17)
public String getStatusMessage(AgentStatus status) {
    if (status == AgentStatus.INVITED) return "Pending invitation";
    else if (status == AgentStatus.ACTIVE) return "Active agent";
    else if (status == AgentStatus.BLOCKED) return "Blocked";
    else return "Unknown";
}

// AFTER (Java 21)
public String getStatusMessage(AgentStatus status) {
    return switch (status) {
        case INVITED -> "Pending invitation";
        case ACTIVE -> "Active agent";
        case BLOCKED -> "Blocked";
        case DORMANT, SLEEPING, INACTIVE -> "Inactive";
    };
    // Compiler enforces exhaustiveness — add a new enum value and it won't compile
}
```

### 2.2 Record Patterns

```java
// Destructuring records in instanceof and switch
record AgentLocation(double lat, double lng) {}

if (location instanceof AgentLocation(var lat, var lng)) {
    validateCoordinates(lat, lng);
}
```

### 2.3 Sealed Classes (for domain modeling)

```java
public sealed interface AgentEvent permits InviteEvent, KycEvent, StatusChangeEvent {
    String agentCode();
    ZonedDateTime timestamp();
}

public record InviteEvent(String agentCode, ZonedDateTime timestamp, String invitedBy) 
    implements AgentEvent {}

public record StatusChangeEvent(String agentCode, ZonedDateTime timestamp, 
    AgentStatus from, AgentStatus to) implements AgentEvent {}
```

### 2.4 Sequenced Collections

```java
// Java 21 adds getFirst(), getLast(), reversed() to List, Set, Map
var latestAgent = agents.getLast();
var oldestAgent = agents.getFirst();
var reversed = agents.reversed();
```

---

## 3. Virtual Threads — The Game Changer

### 3.1 What Changes

In Spring Boot 4, virtual threads are the default. Every HTTP request is handled by a virtual thread.

**Traditional threads (Java 17):**
- 1 request = 1 platform thread (OS thread)
- ~1MB stack per thread
- Thread pool limits concurrency (e.g., Tomcat default: 200 threads)
- Blocking I/O = thread sits idle, pool exhausted under load

**Virtual threads (Java 21):**
- 1 request = 1 virtual thread (~few KB)
- Millions of concurrent virtual threads possible
- Blocking I/O = virtual thread yields, platform thread freed for other work
- No thread pool exhaustion under I/O-heavy workloads

### 3.2 What to Delete

```java
// DELETE: AsyncConfiguration.java — no longer needed
@Configuration
@EnableAsync
public class AsyncConfiguration {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);   // irrelevant with virtual threads
        executor.setMaxPoolSize(10);   // irrelevant with virtual threads
        executor.setQueueCapacity(10); // irrelevant with virtual threads
        executor.setTaskDecorator(new MDCTaskDecorator());
        return executor;
    }
}

// DELETE: MDCTaskDecorator.java — use Micrometer context propagation instead

// DELETE: @Async annotations — blocking on virtual threads is fine
@Async  // REMOVE THIS
public void sendInvitationSms(String agentCode, String mobile) { ... }
```

### 3.3 What to Change — synchronized → ReentrantLock

`synchronized` blocks PIN virtual threads to platform threads, defeating the purpose.

```java
// BAD — pins virtual thread
private final Object lock = new Object();
public void updateAgent(Agent agent) {
    synchronized (lock) {
        mongoTemplate.save(agent); // virtual thread pinned during I/O!
    }
}

// GOOD — ReentrantLock does NOT pin
private final ReentrantLock lock = new ReentrantLock();
public void updateAgent(Agent agent) {
    lock.lock();
    try {
        mongoTemplate.save(agent); // virtual thread can yield
    } finally {
        lock.unlock();
    }
}
```

### 3.4 What to Watch — Third-Party Libraries

| Library | Virtual Thread Safe? | Action |
|---------|---------------------|--------|
| MongoDB Java Driver 5.x | ✅ Yes | No action needed |
| Spring Data MongoDB | ✅ Yes | No action needed |
| OkHttp 4.x | ⚠️ Uses synchronized internally | Replace with RestClient |
| Kafka Client | ✅ Yes (since 3.7) | No action needed |
| AWS SDK v2 | ✅ Yes | No action needed |
| Retrofit | ⚠️ Depends on OkHttp | Replace with @HttpExchange |

### 3.5 application.yml — Virtual Threads Config

```yaml
# Spring Boot 4 — virtual threads are ON by default
# To explicitly control (e.g., for testing):
spring:
  threads:
    virtual:
      enabled: true  # default in Boot 4

# Tomcat no longer needs thread pool tuning
server:
  tomcat:
    # These become less relevant with virtual threads:
    # max-threads: 200  (remove or ignore)
    # min-spare-threads: 10  (remove or ignore)
    accept-count: 100
    max-connections: 10000  # can increase since threads are cheap
```

### 3.6 MDC Propagation with Virtual Threads

Virtual threads don't inherit `ThreadLocal` by default. Use Micrometer's context propagation:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>context-propagation</artifactId>
</dependency>
```

```java
// Register MDC propagation
@Configuration
public class ContextPropagationConfig {
    @Bean
    public ContextSnapshotFactory contextSnapshotFactory() {
        // Automatically propagates MDC, SecurityContext, etc.
        return ContextSnapshotFactory.builder().build();
    }
}
```

---

## 4. Replace Retrofit/OkHttp with RestClient & @HttpExchange

### 4.1 Why Replace

| Aspect | Retrofit/OkHttp | RestClient/@HttpExchange |
|--------|----------------|--------------------------|
| Dependencies | 6 extra JARs | Zero (built into Spring) |
| Virtual thread safe | ⚠️ OkHttp uses synchronized | ✅ JDK HttpClient is VT-safe |
| Observability | Manual instrumentation | Auto-instrumented by Micrometer |
| HTTP/2 | Requires config | Built-in |
| Boilerplate | 280 lines (RetrofitConfig) | ~50 lines total |
| Timeouts | Per-client OkHttpClient | Per-client or global |

### 4.2 Define HTTP Service Interfaces

```java
// Replace: IamApiClientService.java (Retrofit interface)
@HttpExchange
public interface IamApiClient {

    @GetExchange("/v1/users/exists")
    Boolean checkUserExistence(@RequestParam String mobile, @RequestParam String email);

    @GetExchange("/v1/users")
    List<IamUserResponseDto> fetchUsers(
        @RequestParam(required = false) String email,
        @RequestParam(required = false) String agentCode,
        @RequestParam(required = false) String mobile);
}
```

```java
// Replace: PiiVaultApiClientService.java
@HttpExchange
public interface PiiVaultApiClient {

    @PostExchange("/store")
    PiiVaultResponse storePiiData(@RequestBody PiiVaultRequest request);

    @PutExchange("/update/{schemaName}/{domainId}")
    void updatePiiData(
        @PathVariable String schemaName,
        @PathVariable String domainId,
        @RequestBody Object piiData);

    @GetExchange("/retrieve/{schemaName}/{domainId}")
    PiiVaultResponse retrievePiiData(
        @PathVariable String schemaName,
        @PathVariable String domainId);
}
```

### 4.3 Configure Clients (replaces entire RetrofitConfig.java)

```java
@Configuration
public class HttpClientConfig {

    @Bean
    public IamApiClient iamApiClient(
            RestClient.Builder builder,
            @Value("${iam.service.base-url}") String baseUrl,
            IamInterceptor iamInterceptor) {
        var client = builder
                .baseUrl(baseUrl)
                .requestInterceptor(toClientInterceptor(iamInterceptor))
                .defaultStatusHandler(HttpStatusCode::is4xxClientError, 
                    (req, res) -> handleClientError(res))
                .defaultStatusHandler(HttpStatusCode::is5xxServerError, 
                    (req, res) -> handleServerError(res))
                .build();
        return createProxy(client, IamApiClient.class);
    }

    @Bean
    public PiiVaultApiClient piiVaultApiClient(
            RestClient.Builder builder,
            @Value("${pii-vault.base-url}") String baseUrl,
            PiiVaultInterceptor piiVaultInterceptor) {
        var client = builder
                .baseUrl(baseUrl)
                .requestInterceptor(toClientInterceptor(piiVaultInterceptor))
                .build();
        return createProxy(client, PiiVaultApiClient.class);
    }

    @Bean
    public NsdlPbCsApiClient nsdlApiClient(
            RestClient.Builder builder,
            @Value("${service.nsdl.cs.base-url}") String baseUrl,
            NiyoNsdlInterceptor nsdlInterceptor) {
        var client = builder
                .baseUrl(baseUrl)
                .requestInterceptor(toClientInterceptor(nsdlInterceptor))
                .build();
        return createProxy(client, NsdlPbCsApiClient.class);
    }

    // Helper to create proxy from RestClient
    private <T> T createProxy(RestClient client, Class<T> serviceType) {
        return HttpServiceProxyFactory
                .builderFor(RestClientAdapter.create(client))
                .build()
                .createClient(serviceType);
    }

    // Adapter: convert OkHttp-style interceptor to Spring ClientHttpRequestInterceptor
    private ClientHttpRequestInterceptor toClientInterceptor(Object interceptor) {
        // Implement based on what your interceptors do (add headers, auth, etc.)
        return (request, body, execution) -> {
            // Add headers from your interceptor logic
            return execution.execute(request, body);
        };
    }
}
```

### 4.4 Timeout Configuration

```java
// Global defaults via application.yml
// Per-client overrides in the builder:
var client = builder
        .baseUrl(baseUrl)
        .connectTimeout(Duration.ofSeconds(3))   // was 2 MINUTES!
        .readTimeout(Duration.ofSeconds(10))     // was 2 MINUTES!
        .build();
```

**Recommended timeouts by service:**

| Service | Connect | Read | Rationale |
|---------|---------|------|-----------|
| IAM | 3s | 5s | Simple user lookup |
| PII Vault | 3s | 10s | Encryption overhead |
| NSDL | 3s | 15s | External bank API, slower |
| Digio | 3s | 30s | Document processing |
| Notification | 3s | 5s | Fire-and-forget style |
| Risk Engine | 3s | 10s | Scoring computation |

### 4.5 Interceptor Migration

Your current OkHttp interceptors add headers (auth tokens, tenant IDs, correlation IDs). Convert to Spring's `ClientHttpRequestInterceptor`:

```java
// BEFORE: OkHttp Interceptor
public class IamInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request().newBuilder()
                .addHeader("Authorization", "Bearer " + getToken())
                .addHeader("X-Correlation-Id", MDC.get("correlationId"))
                .build();
        return chain.proceed(request);
    }
}

// AFTER: Spring ClientHttpRequestInterceptor
@Component
public class IamClientInterceptor implements ClientHttpRequestInterceptor {
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body,
            ClientHttpRequestExecution execution) throws IOException {
        request.getHeaders().setBearerAuth(getToken());
        request.getHeaders().set("X-Correlation-Id", MDC.get("correlationId"));
        return execution.execute(request, body);
    }
}
```

---

## 5. Resilience4j Circuit Breakers

### 5.1 Why You Need This

Without circuit breakers, when a downstream service (NSDL, PII Vault) goes down:
- Each request blocks for the full read timeout (currently 2 minutes!)
- Thread pool exhausts → your service becomes unresponsive
- Cascading failure across all APIs

With circuit breakers:
- After N failures, circuit OPENS → requests fail fast (milliseconds)
- After a wait period, circuit goes HALF-OPEN → tests if service recovered
- If recovered, circuit CLOSES → normal operation resumes

### 5.2 Dependencies

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-micrometer</artifactId>
    <version>2.2.0</version>
</dependency>
```

### 5.3 Configuration

```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        sliding-window-type: COUNT_BASED
        sliding-window-size: 10
        minimum-number-of-calls: 5
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
        permitted-number-of-calls-in-half-open-state: 3
        automatic-transition-from-open-to-half-open-enabled: true
    instances:
      nsdl:
        base-config: default
        wait-duration-in-open-state: 60s  # NSDL is slow to recover
      pii-vault:
        base-config: default
        wait-duration-in-open-state: 10s  # Internal service, recovers fast
      iam:
        base-config: default
      digio:
        base-config: default
        wait-duration-in-open-state: 60s  # External service
      risk-engine:
        base-config: default

  retry:
    configs:
      default:
        max-attempts: 3
        wait-duration: 500ms
        retry-exceptions:
          - java.io.IOException
          - java.net.SocketTimeoutException
    instances:
      pii-vault:
        base-config: default
      notification:
        base-config: default
        max-attempts: 2

  timelimiter:
    configs:
      default:
        timeout-duration: 10s
    instances:
      nsdl:
        timeout-duration: 15s
      digio:
        timeout-duration: 30s
```

### 5.4 Usage in Service Layer

```java
@Service
@RequiredArgsConstructor
public class NsdlPbCsApiServiceImpl implements NsdlPbCsApiService {

    private final NsdlPbCsApiClient nsdlClient;

    @CircuitBreaker(name = "nsdl", fallbackMethod = "fetchCustomerDetailsFallback")
    @Retry(name = "nsdl")
    @Override
    public CustomerDetailsResponse fetchCustomerDetails(String mobileNo) {
        return nsdlClient.fetchByMobile(mobileNo);
    }

    private CustomerDetailsResponse fetchCustomerDetailsFallback(String mobileNo, Exception e) {
        log.warn("NSDL circuit breaker open or call failed for mobile lookup: {}", e.getMessage());
        throw new ServiceUnavailableException(
            "NSDL bank service is temporarily unavailable. Please try again later.");
    }
}
```

### 5.5 Monitoring Circuit Breaker State

Circuit breaker metrics are auto-exported to Prometheus via Micrometer:

```
# Prometheus metrics available at /actuator/prometheus
resilience4j_circuitbreaker_state{name="nsdl"} 0  # 0=CLOSED, 1=OPEN, 2=HALF_OPEN
resilience4j_circuitbreaker_failure_rate{name="nsdl"} 0.0
resilience4j_circuitbreaker_calls_seconds_count{name="nsdl",kind="successful"} 150
resilience4j_circuitbreaker_calls_seconds_count{name="nsdl",kind="failed"} 2
```

---

## 6. MongoDB Connection Pool Tuning

### 6.1 Why Tune for Virtual Threads

With virtual threads, you can have thousands of concurrent requests. But MongoDB's connection pool
has a fixed size. If all virtual threads try to use MongoDB simultaneously, they'll queue up waiting
for a connection.

### 6.2 Configuration

```java
@Configuration
public class MongoConfig {

    @Bean
    public MongoClientSettings mongoClientSettings(
            @Value("${spring.data.mongodb.uri}") String uri) {
        return MongoClientSettings.builder()
                .applyConnectionString(new ConnectionString(uri))
                .applyToConnectionPoolSettings(pool -> pool
                        .minSize(10)              // Keep 10 warm connections
                        .maxSize(50)              // Max 50 concurrent connections
                        .maxWaitTime(5, TimeUnit.SECONDS)  // Fail fast if pool exhausted
                        .maxConnectionIdleTime(30, TimeUnit.SECONDS)
                        .maxConnectionLifeTime(60, TimeUnit.MINUTES))
                .applyToSocketSettings(socket -> socket
                        .connectTimeout(3, TimeUnit.SECONDS)
                        .readTimeout(10, TimeUnit.SECONDS))
                .applyToClusterSettings(cluster -> cluster
                        .serverSelectionTimeout(5, TimeUnit.SECONDS))
                .build();
    }
}
```

### 6.3 Index Optimization

Ensure indexes exist for your common query patterns:

```java
@Document(collection = "agents")
@CompoundIndex(name = "mobile_hash_idx", def = "{'mobileHash': 1}", unique = true)
@CompoundIndex(name = "email_hash_idx", def = "{'emailHash': 1}")
@CompoundIndex(name = "status_lastOnboarding_idx", 
    def = "{'agentStatus': 1, 'lastOnboardingDate': 1}")
@CompoundIndex(name = "agentCode_idx", def = "{'agentCode': 1}", unique = true)
public class Agent { ... }
```

### 6.4 Monitoring

```yaml
management:
  metrics:
    mongo:
      command:
        enabled: true
      connectionpool:
        enabled: true
```

This exposes:
- `mongodb.driver.pool.size` — current pool size
- `mongodb.driver.pool.waitqueuesize` — threads waiting for connection
- `mongodb.driver.commands.duration` — query latency

---

## 7. Observability — OpenTelemetry & Micrometer

### 7.1 Dependencies

```xml
<!-- Spring Boot 4 ships this starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-opentelemetry</artifactId>
</dependency>

<!-- You already have this -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### 7.2 Configuration

```yaml
management:
  otlp:
    tracing:
      endpoint: http://localhost:4318/v1/traces  # OTLP collector (Datadog agent)
    metrics:
      export:
        endpoint: http://localhost:4318/v1/metrics
  tracing:
    sampling:
      probability: 1.0  # 100% in dev/UAT, reduce in prod (e.g., 0.1)
  observations:
    annotations:
      enabled: true  # enables @Observed annotation
```

### 7.3 What Gets Auto-Instrumented

With the OpenTelemetry starter, these are traced automatically (zero code changes):

| Component | Metrics | Traces |
|-----------|---------|--------|
| HTTP incoming requests | ✅ latency, count, errors | ✅ spans |
| RestClient outgoing calls | ✅ latency per service | ✅ spans |
| MongoDB queries | ✅ command duration | ✅ spans |
| Kafka producer/consumer | ✅ lag, throughput | ✅ spans |
| Spring Security auth | ✅ auth failures | ✅ spans |
| Circuit breaker state | ✅ via Resilience4j | N/A |

### 7.4 Custom Observations for Business Logic

```java
@Service
@RequiredArgsConstructor
public class AgentServiceImpl implements AgentService {

    private final ObservationRegistry observationRegistry;

    @Override
    public void editInviteAgent(String agentCode, EditInviteAgentRequestDto request) {
        Observation.createNotStarted("agent.invite.edit", observationRegistry)
                .lowCardinalityKeyValue("status", "started")
                .observe(() -> {
                    // your business logic
                    doEditInviteAgent(agentCode, request);
                });
    }
}

// Or use the annotation approach:
@Observed(name = "agent.invite.edit", 
    contextualName = "edit-agent-invite",
    lowCardinalityKeyValues = {"type", "invite-edit"})
public void editInviteAgent(String agentCode, EditInviteAgentRequestDto request) { ... }
```

### 7.5 Remove Manual Timing Logs

```java
// BEFORE — manual timing in logs (hard to aggregate, search, alert on)
long start = System.currentTimeMillis();
var result = iamApiService.checkUserExistence(mobile, null);
log.info("IAM check took {}ms", System.currentTimeMillis() - start);

// AFTER — automatic via Micrometer (searchable in Datadog, alertable)
// Just call the service. The RestClient auto-instruments the call.
var result = iamApiService.checkUserExistence(mobile, null);
// Metric: http.client.requests{uri="/v1/users/exists", status="200"} = 45ms
```

---

## 8. Spring Security 7 Changes

### 8.1 What's Removed

| Removed in Security 7 | Replacement |
|----------------------|-------------|
| `.and()` method chaining | Lambda DSL (already using) |
| `antMatchers()` | `requestMatchers()` (already using) |
| `authorizeRequests()` | `authorizeHttpRequests()` (already using) |
| `WebSecurityConfigurerAdapter` | `SecurityFilterChain` bean (already using) |
| `@EnableGlobalMethodSecurity` | `@EnableMethodSecurity` (already using) |

### 8.2 Your Current Config — Mostly Compatible

Your `WebSecurityConfig` is already using the lambda DSL and `SecurityFilterChain`. 
The main change needed:

```java
// Verify: @EnableMethodSecurity replaces @EnableGlobalMethodSecurity
@EnableMethodSecurity(securedEnabled = true)  // ✅ You already have this

// Verify: No .and() chaining — ✅ You're using lambdas

// NEW in Security 7: Default CSRF handling changed
// If you're disabling CSRF (which you are for JWT), explicitly state why:
.csrf(AbstractHttpConfigurer::disable)  // ✅ You already have this
```

### 8.3 One Change Needed — PathPatternRequestMatcher

Spring Security 7 deprecates `AntPathRequestMatcher` in favor of `PathPatternRequestMatcher`.
Your `SkipPathRequestMatcher` already uses `PathPatternRequestMatcher` — you're good.

---

## 9. DTOs — Records & Immutability

### 9.1 The Problem with @Data

| Issue | Impact |
|-------|--------|
| Mutable (setters generated) | Accidental modification after deserialization |
| toString() dumps all fields | PII leaks in logs |
| equals/hashCode on all fields | Often wrong for DTOs |
| No compile-time construction safety | Half-built objects possible |

### 9.2 Use Java Records for Request/Response DTOs

```java
// BEFORE
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
public class EditInviteAgentRequestDto {
    private String name;
    private String email;
    private String mobile;
}

// AFTER — Java Record
public record EditInviteAgentRequest(
    @Nullable String name,
    @Nullable @Email String email,
    @Nullable @Pattern(regexp = "^[6-9]\\d{9}$", message = "Invalid mobile format") String mobile
) {
    public boolean hasAtLeastOneField() {
        return StringUtils.isNotBlank(name) 
            || StringUtils.isNotBlank(email) 
            || StringUtils.isNotBlank(mobile);
    }

    // PII-safe toString (override the default)
    @Override
    public String toString() {
        return "EditInviteAgentRequest[name=%s, email=%s, mobile=%s]".formatted(
            name != null ? "***" : "null",
            email != null ? "***" : "null",
            mobile != null ? "***" : "null"
        );
    }
}
```

### 9.3 Records with Jackson — Zero Config Needed

Spring Boot 3.x+ (and 4.x) auto-configures Jackson to deserialize records via their canonical constructor. No `@NoArgsConstructor` needed.

```java
// Jackson handles this automatically:
// {"name": "John", "email": "john@example.com"} → EditInviteAgentRequest record

// For ignoring unknown properties globally (you already have this in BeanRegistry):
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
```

### 9.4 When Records Don't Work — Use @Value

Records can't be used when you need:
- Inheritance
- Builder pattern with many optional fields
- JPA/MongoDB entities (need no-arg constructor, mutable state)

For those cases, use Lombok `@Value` (immutable) instead of `@Data` (mutable):

```java
@Value
@Builder
@JsonIgnoreProperties(ignoreUnknown = true)
public class InviteAgentRequest {
    String name;           // private final (no setter)
    String email;          // private final (no setter)
    String mobile;         // private final (no setter)
    AgentType agentType;   // private final (no setter)
    // Lombok generates: all-args constructor, getters, equals, hashCode, toString
    // Does NOT generate: setters, no-arg constructor
}
```

### 9.5 Decision Matrix

| Use Case | Approach | Why |
|----------|----------|-----|
| Simple request DTO (≤5 fields) | `record` | Minimal, immutable, built-in |
| Complex request DTO (>5 fields, optional) | `@Value @Builder` | Builder pattern for readability |
| Response DTO | `record` | Always immutable |
| Domain entity (MongoDB) | Regular class | Needs mutability for Spring Data |
| Internal value object | `record` | Immutable, compact |
| Kafka event payload | `record` | Immutable, serializable |

---

## 10. Configuration Properties — Type-Safe Config

### 10.1 The Problem with @Value

```java
// Scattered across multiple classes, no validation, no IDE support
@Value("${iam.service.base-url}") private String iamBaseUrl;
@Value("${pii-vault.base-url}") private String piiVaultBaseUrl;
@Value("${service.nsdl.cs.base-url}") private String nsdlBaseUrl;
// If any of these are missing → NPE at runtime, not at startup
```

### 10.2 Use @ConfigurationProperties Records

```java
@ConfigurationProperties(prefix = "services")
@Validated
public record ServiceEndpoints(
    @NotBlank String iam,
    @NotBlank String piiVault,
    @NotBlank String digio,
    @NotBlank String nsdl,
    @NotBlank String riskEngine,
    @NotBlank String longCode,
    @NotBlank String civicDataHub,
    @NotBlank String pdfConverter
) {}

@ConfigurationProperties(prefix = "notification.service")
@Validated
public record NotificationConfig(
    @NotBlank String baseUrl,
    @NotBlank String tenantId,
    TemplateIds templateId
) {
    public record TemplateIds(
        String loginOtpSms,
        String loginOtpEmail,
        String invitationEmail,
        String invitationSms,
        String actionRequiredEmail
    ) {}
}

@ConfigurationProperties(prefix = "security.jwt")
@Validated
public record JwtConfig(
    @NotBlank String header,
    @NotBlank String prefix,
    @NotBlank String baseUrl,
    @Positive int expiry,
    @Positive int refreshExpiry
) {}
```

### 10.3 Enable in Application

```java
@SpringBootApplication
@ConfigurationPropertiesScan  // Scans for @ConfigurationProperties records
public class AgentOnboardingApplication { ... }
```

### 10.4 Corresponding YAML

```yaml
services:
  iam: ${IAM_BASE_URL:https://uatone.goniyo.com/iam/}
  pii-vault: ${PII_VAULT_BASE_URL:https://pii-vault.fnpaas.com/pii-vault/}
  digio: ${DIGIO_BASE_URL:https://ext.digio.in:444}
  nsdl: ${NSDL_BASE_URL:https://vpce-xxx.amazonaws.com/uat/nsdl-pb/cs/api/}
  risk-engine: ${RISK_ENGINE_BASE_URL:https://uatone.goniyo.com/}
  long-code: ${LONG_CODE_BASE_URL:https://platform-s2s.fnpaas.com/long-codes/}
  civic-data-hub: ${CIVIC_DATA_HUB_BASE_URL:https://platform-s2s.fnpaas.com/civic-data-hub/}
  pdf-converter: ${PDF_CONVERTER_BASE_URL:https://uatone.fnpaas.com/bharat-pdf-converter/}
```

### 10.5 Benefits

- **Fails at startup** if a required property is missing (not at runtime)
- **IDE autocomplete** for property names
- **Type-safe** — no string typos
- **Testable** — just construct the record in tests
- **Documented** — the record IS the documentation

---

## 11. Dependency Cleanup

### 11.1 Dependencies to REMOVE

| Dependency | Reason |
|-----------|--------|
| `com.squareup.okhttp3:okhttp` | Replaced by RestClient |
| `com.squareup.okhttp3:logging-interceptor` | Replaced by Micrometer |
| `com.squareup.retrofit2:retrofit` | Replaced by @HttpExchange |
| `com.squareup.retrofit2:converter-jackson` | Not needed |
| `com.squareup.retrofit2:converter-gson` | Not needed |
| `com.squareup.retrofit2:converter-scalars` | Not needed |
| `org.jetbrains.kotlin:kotlin-stdlib` | Only needed for OkHttp |
| `org.modelmapper:modelmapper` | Use MapStruct only |
| `org.modelmapper.extensions:modelmapper-jackson` | Use MapStruct only |
| `spring-boot-starter-web-services` | Not using SOAP |
| `spring-boot-starter-integration` | Not using Spring Integration |
| `javax.annotation:javax.annotation-api` | Jakarta only in Boot 4 |
| `javax.xml.bind:jaxb-api` | Use jakarta.xml.bind only |
| `javax.xml.ws:jaxws-api` | Not using SOAP |
| `com.sun.xml.ws:jaxws-rt` | Not using SOAP |
| `io.netty:netty-codec-http` | Not using Netty directly |
| `org.sonarsource.scanner.api:sonar-scanner-api` | CI-only, not compile dep |
| `spring-boot-devtools` | Don't ship to prod |

### 11.2 Dependencies to ADD

| Dependency | Purpose |
|-----------|---------|
| `spring-boot-starter-opentelemetry` | Distributed tracing + metrics export |
| `io.github.resilience4j:resilience4j-spring-boot3` | Circuit breakers |
| `io.micrometer:context-propagation` | MDC propagation with virtual threads |

### 11.3 Dependencies to UPDATE

| Dependency | Current | Target | Notes |
|-----------|---------|--------|-------|
| `spring-boot-starter-parent` | 3.5.13 | 4.0.6 | Major upgrade |
| `mapstruct` | 1.5.5.Final | 1.6.x | Boot 4 compatible |
| `springdoc-openapi` | 2.8.13 | 2.8.x+ | Verify Boot 4 compat |

### 11.4 Impact

- **~17 fewer JARs** in the final image
- **Faster builds** — less to download and compile
- **Smaller container** — fewer layers, less attack surface
- **Fewer CVEs** — less third-party code to patch

---

## 12. Structured Concurrency

### 12.1 The Problem — Sequential External Calls

In `editInviteAgent`, you make sequential calls:

```java
// Current flow — sequential, latency stacks:
validateEditAgentInviteRequestDto(request);           // ~5ms (local)
Agent agent = getAgent(agentCode);                    // ~20ms (MongoDB)
var agentPiiDto = agentUtils.getAgentPiiData(agent);  // ~50ms (PII Vault)
iamApiService.checkUserExistence(mobile, null);       // ~200ms (IAM)
blockAgentInviteIfAccountPresentInNSDL(mobile);       // ~300ms (NSDL)
piiVaultApiService.updatePiiDataToPiiVault(...);      // ~100ms (PII Vault)
agentDAOService.saveAgent(agent);                     // ~20ms (MongoDB)
// Total: ~695ms
```

### 12.2 Parallel Execution with StructuredTaskScope (Java 21)

```java
// IAM check and NSDL check are independent — run in parallel
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<Boolean> iamCheck = scope.fork(() -> 
        iamApiService.checkUserExistence(mobile, null));
    Subtask<CustomerDetails> nsdlCheck = scope.fork(() -> 
        nsdlPbCsApiService.fetchCustomerDetails(mobile));
    
    scope.join();           // Wait for both
    scope.throwIfFailed();  // Propagate any exception
    
    // Both completed — use results
    if (Boolean.TRUE.equals(iamCheck.get())) {
        throw new DuplicateAgentException(...);
    }
    validateNsdlResponse(nsdlCheck.get());
}
// Total: max(200ms, 300ms) = 300ms instead of 500ms
```

> **Note:** StructuredTaskScope is a preview feature in Java 21. It may be finalized in Java 23+.
> For production use today, you can use `CompletableFuture` with virtual thread executor:

### 12.3 Production-Ready Alternative — CompletableFuture + Virtual Threads

```java
private static final ExecutorService VIRTUAL_EXECUTOR = 
    Executors.newVirtualThreadPerTaskExecutor();

public void editInviteAgent(String agentCode, EditInviteAgentRequestDto request) {
    // ... validation ...
    
    Agent agent = getAgent(agentCode);
    
    // Parallel external checks
    var iamFuture = CompletableFuture.supplyAsync(
        () -> iamApiService.checkUserExistence(request.getMobile(), null), 
        VIRTUAL_EXECUTOR);
    var nsdlFuture = CompletableFuture.supplyAsync(
        () -> nsdlPbCsApiService.fetchCustomerDetails(request.getMobile()), 
        VIRTUAL_EXECUTOR);
    
    // Wait for both (virtual threads — blocking is fine)
    CompletableFuture.allOf(iamFuture, nsdlFuture).join();
    
    // Check results
    if (Boolean.TRUE.equals(iamFuture.join())) {
        throw new DuplicateAgentException(...);
    }
    validateNsdlResponse(nsdlFuture.join());
    
    // ... continue with updates ...
}
```

---

## 13. GraalVM Native Image (Optional)

### 13.1 When to Use

| Scenario | Native Image? | Why |
|----------|--------------|-----|
| Frequent pod scaling (HPA) | ✅ Yes | 0.1s startup vs 8-12s |
| Lambda/serverless | ✅ Yes | Cold start elimination |
| Long-running service, stable pods | ⚠️ Maybe | JIT is faster at steady-state |
| Heavy reflection (MongoDB, Jackson) | ⚠️ Careful | Needs reachability metadata |

### 13.2 Build Configuration

```xml
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <configuration>
        <buildArgs>
            <buildArg>--no-fallback</buildArg>
            <buildArg>-H:+ReportExceptionStackTraces</buildArg>
        </buildArgs>
    </configuration>
</plugin>
```

```bash
# Build native image
./mvnw -Pnative native:compile

# Or build native container image
./mvnw -Pnative spring-boot:build-image
```

### 13.3 Performance Comparison

| Metric | JVM (Java 21) | Native Image |
|--------|--------------|--------------|
| Startup time | 8-12s | 0.05-0.1s |
| Memory (RSS) | 350-500MB | 60-100MB |
| Peak throughput | Higher (JIT optimized) | ~80% of JVM |
| First request latency | High (JIT warmup) | Low (AOT compiled) |

### 13.4 Recommendation

For `bharat-agent-onboarding`:
- If pods are long-lived and stable → stick with JVM, benefit from JIT
- If you need fast autoscaling → native image
- Start with JVM + virtual threads, evaluate native later

---

## 14. Global Exception Handling — ProblemDetail (RFC 7807)

### 14.1 Standardized Error Responses

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(ValidationException.class)
    public ProblemDetail handleValidation(ValidationException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST, ex.getMessage());
        problem.setTitle("Validation Failed");
        problem.setProperty("errorCode", extractErrorCode(ex));
        problem.setProperty("timestamp", Instant.now());
        return problem;
    }

    @ExceptionHandler(DuplicateAgentException.class)
    public ProblemDetail handleDuplicate(DuplicateAgentException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.CONFLICT, ex.getMessage());
        problem.setTitle("Duplicate Agent");
        problem.setProperty("errorCode", ex.getErrorConstants().getErrorCode());
        return problem;
    }

    @ExceptionHandler(DataNotFoundException.class)
    public ProblemDetail handleNotFound(DataNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Resource Not Found");
        return problem;
    }

    @ExceptionHandler(ServiceUnavailableException.class)
    public ProblemDetail handleServiceUnavailable(ServiceUnavailableException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.SERVICE_UNAVAILABLE, ex.getMessage());
        problem.setTitle("Downstream Service Unavailable");
        problem.setProperty("retryAfterSeconds", 30);
        return problem;
    }
}
```

### 14.2 Response Format (RFC 7807)

```json
{
  "type": "about:blank",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Agent invite can only be edited when agent status is INVITED.",
  "instance": "/public/wb/v1/agent-invite/BBN0405269131815522",
  "errorCode": "E-AGENT-5004",
  "timestamp": "2026-05-05T09:15:30Z"
}
```

### 14.3 Enable in application.yml

```yaml
spring:
  mvc:
    problemdetail:
      enabled: true  # Default in Boot 4
```

---

## 15. Structured Logging

### 15.1 SLF4J 2.0 Fluent API (available in Spring Boot 3.5+/4.x)

```java
// BEFORE — message string interpolation (hard to search in Datadog)
log.info("Edit agent-invite request for agentCode: {}, updatedFields: {}", 
    agentCode, updatedFields);

// AFTER — structured key-value pairs (first-class JSON fields)
log.atInfo()
    .addKeyValue("agentCode", agentCode)
    .addKeyValue("action", "editInvite")
    .addKeyValue("updatedFields", updatedFields)
    .log("Agent invite edited successfully");
```

### 15.2 Output in JSON (logstash-logback-encoder)

```json
{
  "timestamp": 1714905330000,
  "level": "INFO",
  "message": "Agent invite edited successfully",
  "agentCode": "BBN0405269131815522",
  "action": "editInvite",
  "updatedFields": ["mobile", "name"],
  "correlationId": "BAO-abc123",
  "principal": "distributor@goniyo.com",
  "application": "bharat-agent-onboarding"
}
```

Key-value pairs become top-level JSON fields — directly searchable and filterable in Datadog/Grafana without regex parsing.

### 15.3 Logback Configuration for Key-Value Support

Your existing `logback-spring.xml` with `LoggingEventCompositeJsonEncoder` already supports this. Add the key-value provider:

```xml
<encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
    <providers>
        <timestamp>
            <fieldName>timestamp</fieldName>
            <pattern>[UNIX_TIMESTAMP_AS_NUMBER]</pattern>
        </timestamp>
        <!-- ADD THIS: outputs SLF4J key-value pairs as top-level JSON fields -->
        <keyValuePairs/>
        <pattern>
            <pattern>
                {
                "application": "${APP_NAME}",
                "level": "%level",
                "loggerName":"%c",
                "correlationId": "%mdc{correlationId}",
                "principal": "%mdc{userId}",
                "thread": "%t",
                "message": "%.-6000msg%n%rEx{65}"
                }
            </pattern>
        </pattern>
    </providers>
</encoder>
```

---

## 16. Migration Checklist

### Phase 1: Foundation (Week 1-2)

- [ ] Upgrade JDK to 21 in CI/CD, Docker images, local dev
- [ ] Update `pom.xml`: `java.version=21`, `maven.compiler.release=21`
- [ ] Upgrade Spring Boot to 4.0.x
- [ ] Fix all compilation errors from removed deprecated APIs
- [ ] Run full test suite, fix failures
- [ ] Verify application starts and basic APIs work

### Phase 2: Virtual Threads & Concurrency (Week 2-3)

- [ ] Delete `AsyncConfiguration.java`
- [ ] Delete `MDCTaskDecorator.java`
- [ ] Remove `@Async` annotations (or keep for specific fire-and-forget cases)
- [ ] Audit code for `synchronized` blocks → replace with `ReentrantLock`
- [ ] Add `io.micrometer:context-propagation` for MDC with virtual threads
- [ ] Load test to verify no thread pinning issues

### Phase 3: HTTP Client Migration (Week 3-4)

- [ ] Create `@HttpExchange` interfaces for each downstream service
- [ ] Create `HttpClientConfig` with `RestClient` builders
- [ ] Migrate interceptors to `ClientHttpRequestInterceptor`
- [ ] Set proper timeouts (3s connect, 10-30s read per service)
- [ ] Remove Retrofit/OkHttp dependencies
- [ ] Delete `RetrofitConfig.java`
- [ ] Integration test all downstream calls

### Phase 4: Resilience (Week 4)

- [ ] Add Resilience4j dependencies
- [ ] Configure circuit breakers for each downstream service
- [ ] Add `@CircuitBreaker` + `@Retry` to service methods
- [ ] Implement fallback methods
- [ ] Test circuit breaker behavior (simulate downstream failure)
- [ ] Verify Prometheus metrics for circuit breaker state

### Phase 5: Observability (Week 4-5)

- [ ] Add `spring-boot-starter-opentelemetry`
- [ ] Configure OTLP export to Datadog agent
- [ ] Verify traces appear in Datadog APM
- [ ] Remove manual timing logs
- [ ] Add `@Observed` to key business methods
- [ ] Configure sampling rate for production

### Phase 6: Code Quality (Week 5-6)

- [ ] Convert request/response DTOs to records
- [ ] Replace `@Value` with `@ConfigurationProperties` records
- [ ] Remove ModelMapper, use MapStruct everywhere
- [ ] Implement `ProblemDetail` global exception handler
- [ ] Adopt structured logging (key-value pairs)
- [ ] Remove dead dependencies
- [ ] Tune MongoDB connection pool

### Phase 7: Validation & Rollout (Week 6-7)

- [ ] Full regression test suite
- [ ] Load test with virtual threads (compare P50/P95/P99 latency)
- [ ] Verify Prometheus/Grafana dashboards
- [ ] Deploy to UAT, soak test for 48h
- [ ] Monitor for thread pinning, connection pool exhaustion
- [ ] Gradual production rollout (canary → 25% → 50% → 100%)

---

## Expected Results

| Metric | Before (Boot 3.5, Java 17) | After (Boot 4, Java 21) |
|--------|---------------------------|------------------------|
| P95 latency (multi-call APIs) | 800-1200ms | 300-500ms |
| Max concurrent requests | ~200 (thread pool) | 10,000+ (virtual threads) |
| Startup time | 8-12s | 8-12s (JVM) / 0.1s (native) |
| Memory per pod | 400-500MB | 300-400MB |
| Container image size | ~250MB | ~180MB (fewer deps) |
| Downstream failure recovery | 2 min timeout | 30s circuit breaker |
| Dependencies | ~80 JARs | ~60 JARs |

---

## References

- [Spring Boot 4.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Release-Notes)
- [Spring Boot 3→4 Migration Guide](https://ankurm.com/spring-boot-3-to-4-migration-guide/)
- [Virtual Threads with Spring Boot](https://katyella.com/blog/java-virtual-threads-spring-boot/)
- [Resilience4j Documentation](https://resilience4j.readme.io/docs)
- [Spring RestClient Documentation](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html)
- [RFC 7807 Problem Details](https://www.rfc-editor.org/rfc/rfc7807)
- [Micrometer Observation API](https://micrometer.io/docs/observation)
