Here are 50 scenario-based interview questions with answers covering Java, Spring Boot, and REST API for a Lead Java Developer role.

---

## Java Core & Advanced

**1. Your application is experiencing memory leaks. How do you diagnose and fix them?**

Use tools like Java VisualVM, Eclipse MAT, or JProfiler to analyze heap dumps. Common causes include unclosed streams/connections, static collections holding references, inner class references to outer class, and improper use of caches. Fix by closing resources in `finally` blocks or using try-with-resources, using `WeakHashMap` for caches, and nullifying unused references.

---

**2. You have a multi-threaded application where two threads are stuck waiting for each other. What is this and how do you resolve it?**

This is a deadlock. Diagnose using `jstack` to get a thread dump and look for "deadlock" entries. Resolve by: enforcing a consistent lock ordering across threads, using `tryLock()` with timeouts from `java.util.concurrent.locks.Lock`, or restructuring code to avoid nested locks.

---

**3. Your team is debating between using `synchronized` and `ReentrantLock`. When would you prefer one over the other?**

Use `synchronized` for simple use cases — it's cleaner and the JVM optimizes it well. Use `ReentrantLock` when you need: tryLock with timeout, interruptible locking, fairness policy, or multiple condition variables (`Condition`). `ReentrantLock` gives more fine-grained control.

---

**4. A junior developer used `HashMap` in a multi-threaded environment and is seeing inconsistent data. How do you address this?**

`HashMap` is not thread-safe. Solutions: use `ConcurrentHashMap` (preferred for high concurrency — uses segment-level locking), use `Collections.synchronizedMap()` (coarser locking), or restructure to avoid shared mutable state. Explain why `Hashtable` is legacy and should be avoided.

---

**5. You need to process a list of 1 million records as fast as possible. How do you approach this in Java?**

Use `parallelStream()` which internally uses the ForkJoinPool. For more control, use `ExecutorService` with a thread pool sized to available processors (`Runtime.getRuntime().availableProcessors()`). Batch the records, process in parallel, then aggregate results. Be cautious of thread-safety in shared state during parallel processing.

---

**6. You see `ConcurrentModificationException` in production. What caused it and how do you fix it?**

It's thrown when a collection is modified while being iterated (using for-each or iterator). Fix by: using `Iterator.remove()` instead of `collection.remove()`, using `CopyOnWriteArrayList` for read-heavy scenarios, or collecting items to remove and removing them after iteration.

---

**7. What is the difference between `Callable` and `Runnable`, and when would you use each?**

`Runnable.run()` returns void and cannot throw checked exceptions. `Callable.call()` returns a value and can throw checked exceptions. Use `Callable` when you need a result from a thread (combined with `Future`) or need to propagate exceptions. Use `Runnable` for fire-and-forget tasks.

---

**8. A developer used `String` concatenation inside a loop for building a large string. What's wrong and how do you fix it?**

Each `+` creates a new `String` object due to immutability, leading to O(n²) memory and time complexity. Fix by using `StringBuilder` (single-threaded) or `StringBuffer` (multi-threaded). For joining collections, use `String.join()` or `Collectors.joining()`.

---

**9. Explain how Java's Garbage Collector works and when would you tune it?**

The JVM uses generational GC: objects are first allocated in the Young Generation (Eden + Survivor spaces). Surviving objects are promoted to Old Generation. Full GC collects both. Modern collectors: G1GC (default since Java 9), ZGC, and Shenandoah for low-latency. Tune when you see long GC pauses — adjust heap size (`-Xms`, `-Xmx`), survivor ratios, and choose the right GC algorithm based on latency vs. throughput needs.

---

**10. What are the key differences between Java 8 streams and traditional loops? When would you avoid streams?**

Streams provide declarative, functional-style processing with lazy evaluation and easy parallelism. Avoid streams when: you need to modify local variables (closure restriction), you need early exit with complex logic (plain loop is clearer), or performance profiling shows streams are slower for very simple operations on small collections.

---

**11. Your application uses a lot of Optional but a colleague says it's being misused. What are common misuses?**

Common misuses: calling `optional.get()` without checking (defeats the purpose), using `Optional` as method parameters or fields (it's designed for return types only), using `isPresent()` + `get()` instead of `map()`/`orElse()`, and wrapping collections in `Optional` (return an empty collection instead).

---

**12. How does the Java memory model (JMM) affect multi-threaded programming?**

The JMM defines how threads interact through memory — specifically around visibility and ordering of reads/writes. Without proper synchronization, changes made by one thread may not be visible to another due to CPU caching. Use `volatile` for simple visibility guarantees (no atomicity), `synchronized`/`Lock` for atomicity + visibility, and `java.util.concurrent.atomic` classes for atomic operations.

---

## Spring Boot

**13. Your Spring Boot application starts slowly. How do you diagnose and improve startup time?**

Enable `spring.main.lazy-initialization=true` to delay bean creation. Use Spring Boot Actuator's `/actuator/startup` endpoint to identify slow beans. Reduce classpath scanning with explicit `@ComponentScan` packages. Exclude unnecessary auto-configurations. Consider using Spring Native (GraalVM) for AOT compilation.

---

**14. How would you implement a circuit breaker in a Spring Boot microservice?**

Use Resilience4j (preferred over Hystrix which is in maintenance mode). Annotate the method with `@CircuitBreaker(name = "serviceName", fallbackMethod = "fallback")`. Configure thresholds in `application.yml` — failure rate threshold, wait duration in open state, permitted calls in half-open state. Provide a fallback method that returns a default/cached response.

---

**15. A bean is not being injected and you're getting `NoSuchBeanDefinitionException`. How do you debug this?**

Check: the class has `@Component`/`@Service`/`@Repository` etc., it's within the component scan package, there's no conditional annotation (`@ConditionalOnProperty`) preventing its creation, and if it's a configuration bean, the `@Configuration` class is properly loaded. Use `ApplicationContext.getBeanDefinitionNames()` to list all registered beans.

---

**16. How would you handle database transactions that span multiple service calls?**

For local transactions, use Spring's `@Transactional` which manages the transaction boundary. For distributed transactions across microservices, avoid 2PC (two-phase commit) as it's problematic at scale. Instead use the Saga pattern — either choreography (event-driven) or orchestration (a central saga orchestrator manages steps and compensating transactions on failure).

---

**17. Your Spring Boot app has a performance bottleneck in database queries. How do you approach this?**

Enable SQL logging (`spring.jpa.show-sql=true`, Hibernate statistics). Use `EXPLAIN ANALYZE` on slow queries. Add proper indexes. Fix N+1 problems by using `JOIN FETCH` in JPQL or `@EntityGraph`. Use projections/DTOs instead of full entity fetches. Enable second-level caching with Ehcache or Redis for read-heavy data. Use pagination for large result sets.

---

**18. How do you manage different configurations for dev, staging, and production environments?**

Use Spring Profiles — `application-dev.yml`, `application-prod.yml` etc., activated via `spring.profiles.active`. For sensitive values, use environment variables or a secrets manager (AWS Secrets Manager, HashiCorp Vault) integrated via Spring Cloud Vault or Spring Cloud AWS. Never hardcode credentials in config files.

---

**19. How would you implement a custom Spring Boot auto-configuration?**

Create a `@Configuration` class with `@ConditionalOn*` annotations. Register it in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Spring Boot 3.x) or `META-INF/spring.factories` (Spring Boot 2.x). This allows your library to auto-configure beans when added as a dependency without user configuration.

---

**20. A `@Transactional` method is not rolling back on an exception. Why?**

Common reasons: the exception is a checked exception (by default, `@Transactional` only rolls back on unchecked/runtime exceptions — fix with `rollbackFor = Exception.class`), the method is called from within the same class (self-invocation bypasses the proxy), or the bean is not managed by Spring. Also ensure the datasource supports transactions.

---

**21. How do you implement caching in Spring Boot and what are the pitfalls?**

Use `@EnableCaching` and `@Cacheable`, `@CachePut`, `@CacheEvict`. Configure a `CacheManager` (e.g., Redis, Caffeine). Pitfalls: cache stampede (many threads hitting DB on cache miss simultaneously — use locking), stale data (define appropriate TTL and eviction strategy), caching mutable objects (return copies or use immutable DTOs), and not caching at the right layer.

---

**22. How would you implement event-driven communication within a Spring Boot application?**

Use Spring's `ApplicationEventPublisher` for synchronous in-process events. Annotate listeners with `@EventListener` or `@TransactionalEventListener` (triggers after transaction commit). For async processing, add `@Async` to the listener. For cross-service events, use Spring Cloud Stream with Kafka or RabbitMQ as the broker.

---

**23. Your microservice needs to call three external APIs and aggregate the results. How do you do this efficiently?**

Use `CompletableFuture` to make all three calls in parallel, then `CompletableFuture.allOf()` to wait for all to complete and aggregate. With Spring WebFlux/WebClient, use `Mono.zip()` to combine reactive streams. This reduces total latency from sum of all call times to the slowest single call time.

---

**24. How do you prevent a slow downstream service from bringing down your Spring Boot application?**

Use a combination of: timeout configuration on your HTTP client (RestTemplate/WebClient), circuit breaker (Resilience4j) to stop calling a failing service, bulkhead pattern to limit concurrent calls, and a fallback response. This prevents thread pool exhaustion and cascading failures.

---

**25. How would you implement a scheduled job in Spring Boot and ensure it doesn't run concurrently?**

Use `@Scheduled(cron = "0 0 * * * *")` with `@EnableScheduling`. To prevent concurrent execution, add `@Scheduled` with `fixedDelay` instead of `fixedRate` (waits for completion before scheduling next), or use `ShedLock` library which uses a distributed lock (stored in DB/Redis) to ensure only one instance runs the job across a cluster.

---

## REST API Design & Implementation

**26. A client reports that your API is slow. How do you diagnose and optimize it?**

Add request tracing (Spring Sleuth/Micrometer + Zipkin). Check Actuator metrics for response time percentiles. Profile: is it slow DB queries, external API calls, or CPU-heavy processing? Solutions: add caching, optimize queries, use async processing for long-running tasks (return 202 Accepted with a job ID), add pagination, or use CDN for static content.

---

**27. How would you version your REST API and what are the trade-offs of each approach?**

Three main approaches: URI versioning (`/api/v1/users`) — most visible and cacheable but pollutes URLs; Header versioning (`Accept: application/vnd.app.v1+json`) — clean URLs but harder to test in browser; Query parameter (`/api/users?version=1`) — easy to default. URI versioning is most common in practice. Always maintain backward compatibility within a version.

---

**28. How do you handle partial updates in a REST API — PATCH vs PUT?**

`PUT` replaces the entire resource (client sends full representation). `PATCH` applies partial updates (client sends only changed fields). For PATCH, use JSON Merge Patch (RFC 7396) or JSON Patch (RFC 6902). In Spring, use `@PatchMapping`. Challenge with PATCH: distinguishing between "field not provided" vs "field explicitly set to null" — handle with `Map<String, Object>` or nullable wrapper types.

---

**29. Your REST API needs to handle file uploads of up to 500MB. How do you implement this?**

Use multipart file upload with `@RequestParam MultipartFile`. Configure `spring.servlet.multipart.max-file-size=500MB` and `max-request-size=500MB`. For large files, use chunked/streaming upload to avoid loading the entire file into memory — use `InputStream` directly. Store to object storage (S3) asynchronously, return a 202 with an upload ID, and let the client poll for status.

---

**30. How do you secure your REST API?**

Use Spring Security with JWT (stateless) or OAuth2 (delegated authorization). For JWT: validate signature, expiry, and claims on every request via a filter. Use HTTPS everywhere. Implement rate limiting (Bucket4j or API gateway). Apply `@PreAuthorize` for method-level security. Sanitize inputs, use CORS configuration, add security headers (via Spring Security defaults). Store secrets securely — never in code.

---

**31. How do you implement pagination and sorting in a REST API?**

Use Spring Data's `Pageable` with `@PageableDefault`. Accept `page`, `size`, and `sort` as query parameters. Return response with metadata: `{ "content": [...], "totalElements": 100, "totalPages": 10, "number": 0 }`. For cursor-based pagination (better for large datasets), use a cursor (e.g., last record ID) instead of offset — avoids the "deep pagination" performance problem with SQL OFFSET.

---

**32. A client is making too many API calls and overloading your server. How do you implement rate limiting?**

Use Bucket4j with Spring (in-memory or backed by Redis for distributed rate limiting). Apply rate limits per API key or per user. Return `429 Too Many Requests` with `Retry-After` header. Implement at the API gateway level (Kong, AWS API Gateway) for better scalability. Provide different rate limits per subscription tier.

---

**33. How do you implement idempotency in your REST API?**

For POST requests, have clients send a unique `Idempotency-Key` header. Store this key with the response in Redis/DB (with TTL). On duplicate requests with the same key, return the cached response instead of re-processing. PUT and DELETE are naturally idempotent by design. This is critical for payment and order APIs where duplicate processing can cause serious issues.

---

**34. How do you design error responses in your REST API?**

Use RFC 7807 (Problem Details for HTTP APIs) format: `{ "type": "uri", "title": "Validation Error", "status": 400, "detail": "Email is invalid", "instance": "/api/users/123" }`. Use `@ControllerAdvice` with `@ExceptionHandler` to centralize error handling. Map domain exceptions to appropriate HTTP status codes. Never expose stack traces or internal details in production responses.

---

**35. How do you handle long-running operations in a REST API?**

Use async processing pattern: client sends request → server returns `202 Accepted` with a job ID and a status URL → client polls `GET /jobs/{id}` for status → when complete, response contains result or a link to it. Alternatively, use webhooks (server calls back a client-provided URL) or WebSockets/SSE for real-time progress updates.

---

**36. How do you implement HATEOAS and is it always necessary?**

HATEOAS (Hypermedia as the Engine of Application State) means responses include links to related actions: `{ "id": 1, "_links": { "self": "/orders/1", "cancel": "/orders/1/cancel" } }`. Use Spring HATEOAS library. It's theoretically the highest REST maturity level but in practice, many successful APIs don't implement it. It's most valuable for public APIs where clients shouldn't hardcode URLs. For internal microservices, it's often overkill.

---

**37. How do you implement API documentation and keep it in sync with the code?**

Use SpringDoc OpenAPI (Springdoc-openapi) which auto-generates OpenAPI 3.0 specs from code. Annotate with `@Operation`, `@ApiResponse`, `@Schema` for richer docs. Expose Swagger UI at `/swagger-ui.html`. For contract-first approach, write the OpenAPI spec first and generate server stubs. Use CI checks to fail the build if the spec drifts from the implementation.

---

**38. Your API receives a request with invalid data. How do you handle validation?**

Use Bean Validation (`@Valid`, `@NotNull`, `@Size`, `@Email` etc.) on DTOs. Spring automatically invokes validation and throws `MethodArgumentNotValidException` on failure. Handle this in `@ControllerAdvice` and return 400 with field-level error details. For custom business rules, use `@Validated` with custom `ConstraintValidator` implementations. Always validate at the API boundary, not just in the service layer.

---

## Architecture & Advanced Scenarios

**39. How would you design a microservice that processes high-volume financial transactions?**

Use an event-driven architecture with Kafka for guaranteed delivery and ordering within partitions. Implement idempotency with a unique transaction ID. Use the Outbox pattern to ensure database write and event publication are atomic (write to DB and an outbox table in one transaction; a separate process publishes events). Apply circuit breakers for downstream calls. Use CQRS — separate write and read models for scalability.

---

**40. How do you implement distributed tracing across your microservices?**

Use Micrometer Tracing (Spring Boot 3.x) with a Zipkin or Jaeger backend. Each request gets a unique `traceId` that propagates across services via HTTP headers (`traceparent` in W3C Trace Context standard) or message headers. Log the `traceId` in every log statement using MDC. This lets you trace a full request flow across all services in one view.

---

**41. How would you approach migrating a monolith to microservices?**

Use the Strangler Fig pattern — incrementally extract functionality rather than a big-bang rewrite. Start with bounded contexts that have clear boundaries and low coupling. Set up an API gateway to route traffic to old monolith or new services. Share the database initially (anti-pattern but pragmatic), then gradually give each service its own database. Test thoroughly at each extraction step. Have a rollback plan.

---

**42. How do you manage secrets and sensitive configuration in a production Spring Boot app?**

Never store secrets in `application.properties` or version control. Use: environment variables (injected by the platform), Spring Cloud Vault (HashiCorp Vault integration), AWS Secrets Manager with Spring Cloud AWS, or Kubernetes Secrets (base64 encoded — pair with encryption at rest). Rotate secrets regularly. Audit access to secrets. Use different secrets per environment.

---

**43. A developer on your team pushed code that broke production. How do you handle this as a lead?**

Immediately focus on restoring service — rollback the deployment using your CI/CD pipeline. Once stable, do a blameless post-mortem: what happened, why, how to prevent it. Improve: add the missing test case, improve code review guidelines, strengthen CI/CD gates (automated tests, static analysis). Foster a culture where developers can raise concerns and mistakes are learning opportunities, not punishments.

---

**44. How do you design for high availability in a Spring Boot application?**

Deploy multiple instances behind a load balancer. Externalize state (sessions in Redis, data in DB — don't use in-memory state). Use health checks (`/actuator/health`) for load balancer routing. Implement circuit breakers for dependencies. Design for graceful shutdown (`server.shutdown=graceful`). Use a database with read replicas. Apply the 12-factor app principles. Target 99.9%+ uptime with proper retry and fallback strategies.

---

**45. How do you implement a multi-tenant SaaS application in Spring Boot?**

Three strategies: separate databases per tenant (highest isolation, expensive), separate schemas per tenant (good balance), or shared schema with `tenant_id` column (cost-effective but risk of data leakage). Use a `TenantContext` (ThreadLocal) to hold current tenant, resolved from JWT claim or subdomain. Apply row-level security at the DB level. Use Hibernate's multi-tenancy support. Ensure queries always filter by tenant ID — use a base entity or Hibernate filter.

---

**46. How do you implement a global exception handler in Spring Boot and what should it cover?**

Use `@ControllerAdvice` + `@ExceptionHandler`. Cover: `MethodArgumentNotValidException` (400 validation errors), `HttpMessageNotReadableException` (400 malformed JSON), custom domain exceptions mapped to appropriate 4xx codes, `AccessDeniedException` (403), `AuthenticationException` (401), and a catch-all `Exception` handler (500) that logs the error but returns a generic message. Always log with traceId for correlation.

---

**47. What strategies do you use for database schema migrations in a Spring Boot app?**

Use Flyway or Liquibase — never let Hibernate auto-generate DDL in production (`spring.jpa.hibernate.ddl-auto=validate` only). Version migration scripts sequentially. Make migrations backward compatible (don't drop/rename columns immediately — use expand-contract pattern: add new column, migrate data, update code, then remove old column). Run migrations automatically on startup or as a separate step in CI/CD. Test migrations against a copy of production data.

---

**48. How do you handle backward compatibility when changing a REST API?**

Never remove or rename fields in a response (add new fields, keep old ones). Never change field types. For breaking changes, create a new API version. Use API versioning strategy consistently. Communicate deprecation timelines clearly to clients. Add `Deprecation` and `Sunset` headers to deprecated endpoints. Keep deprecated endpoints running until the sunset date. Automate contract tests (Pact) to catch breaking changes in CI.

---

**49. How do you ensure your microservices don't have tight coupling through shared databases?**

Each microservice must own its data — no other service queries its database directly. Services communicate via APIs or events. If Service A needs data from Service B, it calls Service B's API or subscribes to events Service B publishes. If joining data from two services is needed, use API composition or the CQRS pattern to maintain a materialized view. Shared DB is the fastest path to a distributed monolith.

---

**50. As a Lead Developer, how do you ensure code quality across your team?**

Establish coding standards and document them. Use static analysis tools (SonarQube, Checkstyle, SpotBugs) in CI — fail builds on quality gate breaches. Enforce code coverage thresholds (meaningful coverage, not just line %). Conduct thorough, constructive code reviews focused on design, not style (style is for linters). Pair program on complex features. Hold regular tech talks and knowledge-sharing sessions. Lead by example — write clean, well-tested code yourself. Create a psychologically safe environment where the team can raise concerns freely.

---

These 50 questions span the full depth expected of a Lead Java Developer — from low-level JVM internals to distributed systems design and team leadership. Good luck with your interviews!
