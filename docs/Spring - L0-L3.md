---
layout: default
title: "Spring - L0 to L3"
nav_order: 11
---

## Table of contents
{: .no_toc }

1. TOC
{:toc}


# Spring Framework Overview

### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions, 60-90 seconds each.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is Inversion of Control and why does it matter?**

IoC is the design principle where control of object creation and lifecycle
is inverted - instead of your code calling `new` to create dependencies,
an external container creates them and gives them to you.

It matters for three reasons. First, testability: when dependencies are
injected, you can replace them with mocks in tests. Second, flexibility:
you can swap implementations without changing callers. Third, decoupling:
your business code depends on abstractions (interfaces), not concrete
implementations.

The Hollywood Principle captures it: "Don't call us, we'll call you."
Your objects don't go fetch their dependencies - the container delivers them.

*What separates good from great:* Great answers connect IoC to the SOLID
principles - specifically Dependency Inversion (D in SOLID). IoC is the
runtime mechanism that enables the Dependency Inversion Principle at the
architectural level.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the difference between BeanFactory and ApplicationContext?**

BeanFactory is the base Spring container - it lazily creates beans on first
request and provides core DI functionality. ApplicationContext extends
BeanFactory and adds: eager singleton instantiation at startup, event
publishing, internationalization (MessageSource), AOP auto-proxy creation,
and @PostConstruct / @PreDestroy lifecycle callbacks.

In practice, you always use ApplicationContext. BeanFactory is the
interface you program against; ApplicationContext is what you run.

*What separates good from great:* Spring Boot uses
AnnotationConfigServletWebServerApplicationContext for web applications
and AnnotationConfigApplicationContext for non-web. The choice is made
automatically by SpringApplication based on classpath detection.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What happens if you define two beans of the same type?**

Spring throws NoUniqueBeanDefinitionException at the injection point.

Resolution strategies:
1. @Primary on one bean - marks it as the default choice
2. @Qualifier("beanName") at the injection point - selects by name
3. @Profile to activate different beans in different environments

*What separates good from great:* @Primary creates implicit coupling -
any code injecting that type gets the primary bean. @Qualifier is more
explicit and safer in large codebases.

---

**[MID] Q4 - [CONCEPTUAL] How does Spring Boot differ from Spring Framework?**

Spring Framework provides the IoC container, DI, AOP, MVC, data access,
and all core abstractions. It requires explicit configuration.

Spring Boot adds opinionated auto-configuration: it detects what is on
your classpath and automatically configures beans with sensible defaults.
Boot also provides the parent POM with curated dependency versions so
you do not fight dependency conflicts.

Relationship: Boot = Framework + Auto-config + Embedded server + Actuator.

*What separates good from great:* Auto-configuration is just @Configuration
classes annotated with @ConditionalOnClass, @ConditionalOnMissingBean -
there is no magic. Spring Boot's auto-configuration source is readable
and override-able.

---

**[MID] Q5 - [CONCEPTUAL] What is the Spring bean lifecycle?**

The lifecycle phases in order:
1. Instantiation: Spring calls the constructor.
2. Dependency injection: injects @Autowired fields/methods.
3. Aware callbacks: BeanNameAware, ApplicationContextAware if implemented.
4. BeanPostProcessor.postProcessBeforeInitialization() for all BPPs.
5. @PostConstruct (or InitializingBean.afterPropertiesSet()).
6. Custom init-method if specified.
7. BeanPostProcessor.postProcessAfterInitialization() - AOP proxies created here.
8. Bean is ready for use.
9. @PreDestroy on shutdown (or DisposableBean.destroy()).
10. Custom destroy-method if specified.

*What separates good from great:* AOP proxy creation happens in step 7
(postProcessAfterInitialization). This is why @Transactional and @Async
do not work when you call a method on `this` inside the same bean - you
are bypassing the proxy.

---

**[MID] Q6 - [TRADE-OFF] Why is constructor injection preferred over field injection?**

Three concrete reasons:
1. Immutability: constructor-injected dependencies can be `final`.
2. Testability: create with `new MyService(mockDep)` - no Spring needed.
3. Mandatory dependencies visible: constructor signature documents what
   is required.

Field injection (@Autowired on fields) requires Spring to set private
fields via reflection after construction - the object exists in an invalid
state between construction and injection.

*What separates good from great:* Circular dependency detection. Constructor
injection causes Spring to fail fast at startup with a clear cycle error.
Field/setter injection silently defers wiring and can hide cycles.

---

**[SENIOR] Q7 - [CONCEPTUAL] What is the difference between @Component, @Service,**
       @Repository, and @Controller?

All four are specializations of @Component - functionally identical for
bean registration. The difference is semantic and enables framework features:

- @Component: generic stereotype for any Spring-managed bean
- @Service: marks business logic layer; no extra behaviour in core Spring
- @Repository: marks data access layer; activates persistence exception
  translation (wraps JPA/JDBC exceptions into DataAccessException hierarchy)
- @Controller: marks MVC controller; enables dispatcher servlet routing
- @RestController: @Controller + @ResponseBody

*What separates good from great:* @Repository's exception translation is
real Spring behaviour - it wraps vendor-specific exceptions into Spring's
DataAccessException hierarchy, making your service layer independent of the
data access technology.

---


# Spring vs EJB - The Simplicity Revolution


### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [HANDS-ON] Why was Spring created?**

Rod Johnson published "Expert One-on-One J2EE Design and Development" in
2002 demonstrating that enterprise features (transactions, data access) could
be implemented with plain Java objects in a lightweight container - no EJB
container required. The core problem with EJB 2.x was invasiveness: your
classes had to extend framework base classes, making them impossible to test
outside the container. Spring proved you could have enterprise capability
without framework inheritance. The test cycle dropped from minutes to
milliseconds, and the developer community adopted it rapidly.

*What separates good from great:* Name the book and the year. Mention that
Rod Johnson also co-founded Interface21 (later SpringSource, acquired by
VMware in 2009). This chain shows Spring was a deliberate architectural
argument, not accidental success.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What does "non-invasive framework" mean?**

A non-invasive framework does not require your business classes to extend its
types or implement its interfaces. Your domain objects stay framework-free;
the framework hooks in from the outside via reflection, annotations, or code
generation.

Spring is non-invasive because you can take any @Service class, strip the
@Service annotation, and the class is still a perfectly valid Java object.
Compare EJB 2.x where removing extends SessionBean breaks everything.

The practical benefit: domain objects can be tested as plain Java, used in
non-Spring contexts, and their logic is readable without framework knowledge.

*What separates good from great:* Note that annotations still create some
coupling. True zero-coupling uses XML config or @Bean factory methods in
@Configuration classes. For domain model objects (entities, value objects),
Spring enforces no coupling at all.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How did Spring influence the EJB 3.0 specification?**

EJB 3.0 (JSR-220, 2006) directly adopted Spring's ideas: POJOs with
annotations instead of interface inheritance, dependency injection via @EJB
and (later) @Inject, JPA replacing CMP entity beans. The spec authors
acknowledged Spring had demonstrated the right approach.

By the time EJB 3.0 shipped, Spring had four years of adoption momentum and
a richer ecosystem. EJB 3.0 was better than EJB 2.x but could not displace
Spring's installed base.

*What separates good from great:* CDI (JSR-299, 2009) was the Jakarta EE
answer to Spring's DI. CDI is excellent - type-safe, producer methods,
interceptors - but Spring's ecosystem (Boot, Data, Security, Cloud) is what
keeps Spring dominant.

---

**[MID] Q4 - [CONCEPTUAL] What is the POJO principle in Spring development?**

POJO (Plain Old Java Object) means a class that does not extend any framework
class and implements no framework interface beyond what the domain requires. In
Spring, your business services, repositories, and domain models should all be
POJOs.

The POJO principle has three consequences:
1. Unit tests create business objects with plain new and mock dependencies.
2. The class can be understood without framework documentation.
3. Migrating frameworks requires changing configuration, not rewriting
   business logic.

Modern Spring reinforces this: prefer @PostConstruct over InitializingBean,
@PreDestroy over DisposableBean, constructor injection over BeanFactoryAware.

*What separates good from great:* Entities in JPA are POJOs. Spring Data
repositories are interfaces. Neither has framework base classes - the
architecture is working as intended.

---

**[MID] Q5 - [TRADE-OFF] What is the key trade-off of Spring's proxy-based approach?**

Spring implements AOP (and therefore @Transactional, @Async, @Cacheable) by
wrapping beans in proxy objects - JDK dynamic proxies for interfaces or CGLIB
subclass proxies for concrete classes.

The trade-off: this only intercepts calls from outside the bean. When method A
calls method B in the same bean, the call goes directly to the target object,
bypassing the proxy. This means:

- Calling @Transactional from within the same class does NOT start a transaction
- Calling @Async from within the same class does NOT run asynchronously
- @Cacheable on an internal method does NOT cache

This is the most common Spring production bug and the most common interview
trap question.

*What separates good from great:* Solutions are: self-inject the bean with
@Lazy to break the circular dependency, or refactor so the cross-cutting
method is on a separate bean. For @Transactional, use TransactionTemplate
programmatically as another option.

---

**[MID] Q6 - [TRADE-OFF] When would you NOT choose Spring?**

Three legitimate scenarios:

1. Native image / serverless cold starts: Spring's classpath scanning takes
   seconds. Quarkus or Micronaut compile-time DI are better for latency-
   sensitive serverless. Spring Boot 3+ native image support is improving.

2. Tiny utilities and scripts: Spring startup overhead is not worth it for
   a CLI tool running 100ms.

3. Jakarta EE-mandated environments: some enterprises mandate Jakarta EE
   containers (WebSphere, WildFly) for compliance reasons.

*What separates good from great:* "Spring is always the answer" is a red
flag showing lack of trade-off thinking. Mentioning the trade-offs shows
engineering judgment.

---

**[SENIOR] Q7 - [ARCHITECTURE] What is Spring's module structure?**

Spring Framework modules:
- spring-core: IoC container, DI, utilities
- spring-beans: Bean definitions and BeanFactory
- spring-context: ApplicationContext, events, i18n
- spring-aop: Proxy-based AOP
- spring-web / spring-webmvc: HTTP and MVC
- spring-webflux: Reactive web (Project Reactor)
- spring-data: Repository abstractions
- spring-security: Authentication and authorization
- spring-tx: Transaction management
- spring-test: TestContext framework, MockMvc

Spring Boot auto-configures whichever modules are on the classpath. The
spring-boot-starter-* POMs are the practical mechanism for module selection.
The Spring Boot BOM prevents version conflicts.

*What separates good from great:* A microservice can use spring-web +
spring-tx without pulling in spring-security or spring-batch. Modular
design means you include only what you need.

---

---

# Spring Ecosystem Map


### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [BEHAVIORAL] Name the main Spring projects and what each does.**

Core four every Spring developer needs:
- Spring Framework: IoC container, AOP, MVC, WebFlux, JDBC template
- Spring Boot: auto-configuration, embedded servers, starter POMs, Actuator
- Spring Security: authentication, authorization, OAuth2, CSRF, CORS
- Spring Data: repository abstraction over JPA, MongoDB, Redis, Elasticsearch

Important for microservices:
- Spring Cloud: config server, service discovery, circuit breakers, API gateway
- Spring Batch: bulk data processing with jobs, steps, readers/writers
- Spring Integration: enterprise integration patterns (message channels,
  adapters, transformers)

*What separates good from great:* Mention the spring.io/projects lifecycle
column - active vs maintenance vs community. Spring Cloud Netflix (Eureka,
Ribbon, Hystrix) is in maintenance mode; prefer Spring Cloud LoadBalancer
and Resilience4j circuit breaker.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is a Spring Boot starter and how does it work?**

A starter is a POM dependency that:
1. Pulls in the transitive dependencies needed for a feature
2. Provides auto-configuration classes that activate when those dependencies
   are on the classpath

Example: spring-boot-starter-web pulls in Spring MVC, Tomcat, Jackson, and
Hibernate Validator. The associated auto-configuration activates a
DispatcherServlet, ObjectMapper bean, and default error handling.

Under the hood: auto-configurations are listed in
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
(Spring Boot 2.7+). Boot reads this file and registers those as @Configuration
candidates at startup.

*What separates good from great:* Creating a custom starter involves the same
pattern: create a module with your auto-configuration class, register it in the
imports file, and add appropriate @Conditional annotations.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What does /actuator provide?**

Spring Boot Actuator exposes production-ready operational endpoints:
- /actuator/health: liveness and readiness probes (used by Kubernetes)
- /actuator/metrics: application metrics (integrates with Micrometer)
- /actuator/conditions: which auto-configurations fired and why
- /actuator/env: resolved property values (secure this endpoint)
- /actuator/beans: all Spring beans in context
- /actuator/mappings: all @RequestMapping routes
- /actuator/threaddump: current thread state

Security best practice: expose only /health and /info publicly; require
authentication for all others.

*What separates good from great:* /actuator/health has health indicators
contributed by each auto-configured component (DataSource, Redis, Kafka).
Custom health indicators let you add domain-specific checks. Kubernetes
uses /actuator/health/liveness vs /actuator/health/readiness for distinct
probe types.

---

**[MID] Q4 - [CONCEPTUAL] How does Spring Data simplify data access?**

Spring Data provides repository interfaces with generated implementations.
You declare the interface and method names; Spring Data generates JPQL/SQL
at startup. Example: findByNameContainingIgnoreCase("spring") generates a
LIKE query automatically.

Spring Data generates a proxy implementing the interface - no boilerplate
DAO required. Key abstractions: CrudRepository (basic CRUD), JpaRepository
(adds pagination, flush, batch operations).

*What separates good from great:* Spring Data projections - you declare a
result interface with only the columns you need, and Spring Data generates
SELECT with only those columns. Crucial for performance when you only need
2 columns out of 20.

---

**[MID] Q5 - [CONCEPTUAL] What is Spring Cloud and when do you need it?**

Spring Cloud provides microservice cross-cutting concerns not in Spring Boot:
- Config Server: centralized configuration management
- Eureka: service registration and discovery
- Spring Cloud Gateway: API gateway with routing and rate limiting
- Circuit Breaker (Resilience4j): prevent cascade failures
- Distributed tracing: Micrometer Tracing + Zipkin/Tempo

When you need it: multiple Spring Boot microservices needing to discover each
other and handle partial failures - when NOT running in Kubernetes.

When you do NOT need it: in Kubernetes, which provides service discovery (DNS),
config management (ConfigMaps/Secrets). Many teams use only Spring Boot on
Kubernetes and skip Spring Cloud for discovery/config entirely.

*What separates good from great:* Spring Cloud's main remaining value in
Kubernetes is circuit breaker (Resilience4j) and distributed tracing - not
service discovery or config management which Kubernetes handles natively.

---

**[MID] Q6 - [ARCHITECTURE] How does Spring Security integrate with the ecosystem?**

Spring Security integrates at the MVC/WebFlux layer via a filter chain
(servlet filters for MVC, WebFilter for WebFlux). Every incoming request
passes through the security filter chain before reaching controllers.

Auto-configuration (activated by spring-boot-starter-security) sets up basic
auth, CSRF protection, session management, and a form login page by default.
You override by defining a SecurityFilterChain @Bean.

Integration with other Spring projects:
- Spring Session: replaces HttpSession with distributed session store (Redis)
  without code changes
- Spring OAuth2: client, resource server, and authorization server support
- Spring Data: @PreAuthorize can reference Spring Data predicates

*What separates good from great:* Spring Security's method-level security
(@PreAuthorize) uses Spring EL evaluated by a SpEL-based aspect. This is
why method security does not work on @Bean methods in @Configuration classes -
proxies are not applied there.

---

**[SENIOR] Q7 - [CONCEPTUAL] What changed in Spring Boot 3 / Spring Framework 6?**

Spring Boot 3 (November 2022) - key changes:
1. Java 17 minimum required.
2. Migrated from javax.* to jakarta.* package names (Jakarta EE 10).
   Any code using javax.servlet.*, javax.persistence.* must be updated.
3. Hibernate 6 required (also migrated to Jakarta).
4. GraalVM native image support via Spring AOT (ahead-of-time compilation).
5. Observability improvements: Micrometer Tracing replaces Sleuth.
6. Virtual threads (Project Loom) support via Spring MVC thread-per-request
   model with virtual threads (Spring Boot 3.2+).

Practical migration impact: package rename is the biggest breaking change.
Third-party libraries must support Jakarta EE 10 to work with Boot 3.

*What separates good from great:* The jakarta.* migration means Spring and
Jakarta EE now share the same package namespace, making interop and future
migration between Spring and Jakarta EE easier. Spring Boot 3's AOT
compilation generates bean definitions at build time, reducing startup
reflection and enabling smaller native images.

---

# Dependency Injection


### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is the difference between DI and IoC?**

IoC (Inversion of Control) is the broad principle: instead of application
code controlling the creation and lifecycle of its dependencies, control is
inverted to a container or framework. DI (Dependency Injection) is the most
common implementation of IoC: the container injects dependencies into your
objects.

Other IoC patterns include: Service Locator (objects look up dependencies),
Template Method (framework calls your hook methods), Event Listeners (framework
calls your handlers). DI is the cleanest because it leaves your objects with
no coupling to the container.

*What separates good from great:* The Service Locator is also IoC but is an
anti-pattern because objects still reach out to the container to get
dependencies - the coupling shifts from "new ConcreteType" to "registry.get
(AbstractType)" but is still coupling. DI eliminates the coupling entirely.

---

**[JUNIOR] Q2 - [TRADE-OFF] Why is constructor injection preferred over field injection?**

Four concrete reasons:
1. **Explicit dependencies**: constructor signature shows all required
   dependencies. Field injection hides them.
2. **Immutability**: constructor-injected fields can be final. Field-injected
   fields cannot.
3. **Testability**: `new MyService(mockDep)` works without any framework.
   Field injection requires Spring or Mockito's @InjectMocks to set fields.
4. **Fail-fast**: circular dependencies detected at startup. Field injection
   defers detection.

Constructor injection is the official Spring team recommendation (documented
in Spring's own style guide).

*What separates good from great:* Lombok's @RequiredArgsConstructor generates
a constructor for all final fields, eliminating the constructor boilerplate
while keeping constructor injection semantics.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is a circular dependency and how do you fix it?**

A circular dependency exists when Bean A depends on Bean B and Bean B depends
on Bean A (directly or transitively). With constructor injection, Spring
detects this at startup and throws BeanCurrentlyInCreationException.

Detection: the error message names the full cycle.

Fixes:
1. **Redesign**: extract a shared dependency C that A and B both depend on.
   This is the best fix - circular dependencies often indicate a design smell.
2. **@Lazy on one injection**: defer one side's initialization.
   `OrderService(@Lazy PaymentService ps)` - PaymentService is created on
   first use rather than at startup.
3. **Setter injection on one side**: Spring can use setter injection to
   break constructor cycles by first creating both objects (without full
   wiring) then calling setters.

*What separates good from great:* Spring Boot 2.6+ prohibits circular
dependencies by default. You can re-enable them with
spring.main.allow-circular-references=true, but the right answer is to
redesign to remove the cycle.

---

**[MID] Q4 - [CONCEPTUAL] What is @Qualifier and when do you use it?**

@Qualifier disambiguates when Spring finds multiple beans of the same type
and does not know which to inject. You annotate the injection point with
@Qualifier("beanName") to specify which bean to use.

```java
@Autowired
@Qualifier("fastPaymentService")
private PaymentService paymentService;
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

@Qualifier can also be used on bean definitions:
```java
@Bean
@Qualifier("fastPaymentService")
public PaymentService fastPaymentService() { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Alternative to @Qualifier: @Primary marks one bean as the default choice
without requiring injection-point annotations.

*What separates good from great:* Custom qualifier annotations are more
type-safe: create `@FastPayment` as a meta-annotation of @Qualifier. This
gives you compile-time safety vs string-based @Qualifier("fastPayment").

---

**[MID] Q5 - [CONCEPTUAL] How does Spring resolve which bean to inject when there are multiple candidates?**

Spring's resolution algorithm in order:
1. Type match: find all beans matching the declared type.
2. @Primary: if one candidate is @Primary, use it.
3. @Qualifier: if injection point has @Qualifier, filter by name.
4. Name match: if the field/parameter name matches a bean name, prefer it.
5. Exception: if still ambiguous, throw NoUniqueBeanDefinitionException.

If no bean matches the type at all: NoSuchBeanDefinitionException.

*What separates good from great:* In practice, relying on name-matching
(step 4) is fragile - refactoring the field name breaks the wiring silently.
Always use @Primary or @Qualifier for explicit disambiguation in production
code.

---

**[MID] Q6 - [CONCEPTUAL] Can you inject a list of all beans of the same type?**

Yes. Declare `List<SomeInterface>` as the injection point and Spring injects
all beans implementing that interface.

```java
@Service
public class NotificationDispatcher {
    private final List<NotificationChannel> channels;

    public NotificationDispatcher(
            List<NotificationChannel> channels) {
        this.channels = channels;
    }

    public void notifyAll(String message) {
        channels.forEach(c -> c.send(message));
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using goroutine. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Spring collects all beans implementing NotificationChannel (EmailChannel,
SmsChannel, PushChannel, etc.) and injects them as a list. Adding a new
channel requires only creating a new @Service that implements the interface -
no changes to the dispatcher.

*What separates good from great:* You can inject Map<String, Interface> to
get a map from bean name to bean. Useful when you need to select a strategy
by name at runtime.

---

**[SENIOR] Q7 - [CONCEPTUAL] What is @Autowired(required = false) used for?**

@Autowired(required = false) marks an optional dependency. If no matching bean
exists, Spring leaves the field null rather than throwing
NoSuchBeanDefinitionException.

Use cases:
- Optional features: SMS notification only if an SmsSender bean is configured
- Backwards compatibility: new dependency added to an existing service that
  some deployments may not have configured

```java
@Autowired(required = false)
private MetricsReporter metricsReporter;

public void process(Order order) {
    // work...
    if (metricsReporter != null) {
        metricsReporter.record("order.processed");
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The modern alternative to @Autowired(required
= false) is using Optional<T> as the injection type:
`Optional<SmsSender> smsSender`. This forces you to explicitly handle the
absent case and makes the optionality visible in the type signature.

---



# Spring Bean


---

### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is a Spring bean?**

A Spring bean is any Java object whose full lifecycle is managed by the Spring
IoC container. "Managed" means Spring creates it, injects its dependencies,
calls initialization callbacks, serves it during the application's runtime,
and calls destruction callbacks on shutdown.

Any class annotated with @Component (or @Service, @Repository, @Controller)
is auto-discovered by classpath scanning and registered as a bean. Classes
returned by @Bean methods in @Configuration classes are also registered.

*What separates good from great:* The key distinction is managed vs unmanaged.
Your domain objects (Order, Customer, Product) are typically created with `new`
and are NOT beans. Your infrastructure and service classes ARE beans. The rule
of thumb: if Spring should manage the lifecycle, make it a bean.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the default bean scope and why?**

The default scope is **singleton**: one instance per Spring ApplicationContext.
Every request for that bean type returns the same instance.

Why singleton is the default: services are typically stateless (they hold
no request-specific data, only collaborators). A stateless service is safe
to share across all threads. Creating a new service instance per request would
waste memory and add GC pressure without any benefit.

The contract: singleton beans MUST be thread-safe because they are shared.
This means no mutable instance fields that vary per request.

*What separates good from great:* Mention that Spring's singleton is per-
ApplicationContext, not per-JVM. In a test suite where each test creates its
own ApplicationContext (rare), you can have multiple "singleton" instances
of the same type across tests.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How do you register a bean in Spring?**

Three ways, in order of preference:

1. **@Component scanning** (most common for classes you own):
    - Annotate with @Component / @Service / @Repository / @Controller
    - Ensure the class is in a package covered by @ComponentScan
      (Spring Boot's @SpringBootApplication scans the main class package)

2. **@Bean method** (for programmatic control or third-party classes):
    - Declare in a @Configuration class
    - Method name becomes bean name; return type becomes bean type
    - Full control over construction arguments

3. **XML** (legacy - avoid in new code):
    - `<bean id="..." class="..."/>` in applicationContext.xml

The difference matters: @Component relies on reflection to instantiate; @Bean
gives you explicit control and is required for classes you cannot annotate
(e.g., HikariDataSource, JdbcTemplate).

*What separates good from great:* @Bean methods in @Configuration are proxied
by Spring - calling one @Bean method from another @Bean method does NOT create
a new instance; it returns the registered singleton. This is how Spring prevents
you from accidentally creating multiple DataSource instances.

---

**[MID] Q4 - [CONCEPTUAL] What happens to a bean on application shutdown?**

On shutdown (when ApplicationContext.close() is called or the JVM receives
SIGTERM with a registered shutdown hook):

1. Spring publishes a ContextClosedEvent.
2. For each singleton bean (in reverse dependency order):
   a. Calls @PreDestroy method if present.
   b. Calls DisposableBean.destroy() if implemented.
   c. Calls custom destroy-method if specified in @Bean(destroyMethod).
3. Bean instances are removed from the singleton cache.

Spring Boot registers a shutdown hook automatically. @PreDestroy is the
cleanest way to release resources (close connections, cancel timers,
flush buffers).

*What separates good from great:* Prototype beans are NOT destroyed by
Spring. Since Spring does not track prototype instances after handing them
out, it cannot call @PreDestroy on them. You are responsible for destroying
prototype beans. This is a critical production gotcha for connection
management.

---

**[MID] Q5 - [CONCEPTUAL] What is @PostConstruct and when do you use it?**

@PostConstruct marks a method to be called after dependency injection is
complete but before the bean is put into service. It runs after all @Autowired
dependencies have been injected.

Common use cases:
- Validate configuration (throw if required config is missing)
- Pre-load a cache (populate in-memory cache from database at startup)
- Establish connections (connect to external system and verify)
- Register the bean with an external system

```java
@Service
public class ProductCacheService {
    private final ProductRepository repository;
    private Map<Long, Product> cache;

    public ProductCacheService(
            ProductRepository repository) {
        this.repository = repository;
    }

    @PostConstruct
    public void initCache() {
        // Runs after repository is injected
        this.cache = repository.findAll().stream()
            .collect(toMap(Product::getId,
                           Function.identity()));
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* @PostConstruct runs inside the
singleton creation process, before the bean is exposed to other beans.
This means slow @PostConstruct methods delay startup. For background
warm-up tasks, use ApplicationReadyEvent or CommandLineRunner instead
so they run after the context is fully initialized.

---

**[MID] Q6 - [CONCEPTUAL] What is the difference between @Bean and @Component?**

**@Component** (and @Service, @Repository, @Controller):
- Applied to the class itself
- Discovered via classpath scanning
- Spring calls the constructor using reflection
- Use when you own and can annotate the class

**@Bean**:
- Applied to a method in a @Configuration class
- Programmatic bean creation
- You write the creation code explicitly
- Use for: third-party classes, conditional creation, complex initialization,
  multiple beans of the same type with different configuration

Key difference: @Bean gives you full control over how the object is created.
@Component surrenders that control to Spring's reflection-based instantiation.

*What separates good from great:* @Configuration classes themselves are beans
(Spring subclasses them via CGLIB for proxy purposes). This is why @Bean
methods called from other @Bean methods return the singleton - the call goes
through the CGLIB proxy which intercepts it and returns the registered bean.

---

**[SENIOR] Q7 - [CONCEPTUAL] What is a BeanDefinition?**

A BeanDefinition is the metadata object that describes how to create a bean -
before any instance is actually created. It contains: the bean class name, scope,
constructor arguments, property values, init method name, destroy method name,
and whether it is lazy or eager.

Spring creates BeanDefinitions at startup (during context refresh, before
singleton instantiation). BeanFactoryPostProcessors can modify BeanDefinitions
at this stage - for example, replacing ${placeholder} values with actual
property values from application.properties.

Understanding BeanDefinitions explains why errors at startup happen in two
phases: BeanDefinition errors (duplicate bean names, invalid configuration)
happen before any object is created; bean instantiation errors happen during
the singleton creation phase.

*What separates good from great:* You can register BeanDefinitions
programmatically using `BeanDefinitionRegistryPostProcessor`. This is the
mechanism Spring Boot's auto-configuration uses: conditional logic decides
which BeanDefinitions to register, and then the normal instantiation process
creates the beans.

---

# ApplicationContext


---

### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is the ApplicationContext and how does it differ from BeanFactory?**

ApplicationContext extends BeanFactory. BeanFactory provides the minimal
container: lazy bean creation and basic DI. ApplicationContext adds:
- Eager singleton instantiation at refresh time
- BeanPostProcessor auto-registration (for AOP proxies, annotation processing)
- ApplicationEvent publishing and listener registration
- MessageSource for internationalization
- ResourcePatternResolver for classpath scanning
- Environment and PropertySource abstraction

In practice, always work with ApplicationContext. BeanFactory is only used
in extremely resource-constrained environments where you need the bare minimum.

*What separates good from great:* Every web ApplicationContext
(AnnotationConfigServletWebServerApplicationContext) starts the embedded
server as part of the onRefresh() step during refresh. This is why
@SpringBootApplication can be run as a standalone jar.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What happens during the ApplicationContext refresh?**

The refresh is the most important phase of Spring startup. Key steps:

1. prepareBeanFactory: register default BPPs and processors
2. invokeBeanFactoryPostProcessors: run all BFPPs, which includes:
    - ConfigurationClassPostProcessor: processes all @Configuration classes,
      @ComponentScan, @Import, @Bean methods
    - PropertySourcesPlaceholderConfigurer: resolves @Value expressions
3. registerBeanPostProcessors: collect all BPPs (for later use)
4. finishBeanFactoryInitialization: instantiates all non-lazy singletons
    - For each: create, inject, call BPPs (pre-init), @PostConstruct,
      call BPPs (post-init where AOP proxies are created)
5. finishRefresh: publish ContextRefreshedEvent, start lifecycle beans

*What separates good from great:* The order explains Spring behaviours.
Component scanning happens in step 2 (BFPP phase), not at context creation.
AOP proxies are created in step 4's BPP post-init phase. Everything follows
from this order.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is the ContextRefreshedEvent and when is it fired?**

ContextRefreshedEvent is published when the ApplicationContext refresh
is complete - all singletons are created, wired, and initialized, and the
context is ready to use.

Use case: tasks that must run after ALL beans are initialized but before
accepting external traffic.

```java
@Component
public class StartupValidator
        implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(
            ContextRefreshedEvent event) {
        // All beans are ready - validate invariants
        validateConfiguration();
    }
}
// Modern equivalent:
@Component
public class StartupValidator {
    @EventListener(ContextRefreshedEvent.class)
    public void validate() { validateConfiguration(); }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* In web applications, ContextRefreshedEvent
fires when the parent context refreshes AND again when the web (child) context
refreshes. Check event.getApplicationContext() to avoid double execution.
ApplicationReadyEvent (Spring Boot) fires only once after the full application
is ready including all lifecycle beans.

---

**[MID] Q4 - [HANDS-ON] What are the main ApplicationContext implementations?**

Key implementations in order of use frequency:

1. **AnnotationConfigServletWebServerApplicationContext** (Spring Boot web):
   annotation-driven + embedded servlet server. Most common.

2. **AnnotationConfigApplicationContext**: annotation-driven, no web layer.
   Used for standalone apps, scheduled jobs, CLI tools.

3. **AnnotationConfigReactiveWebServerApplicationContext**
   (Spring Boot WebFlux): annotation-driven + embedded reactive server.

4. **ClassPathXmlApplicationContext**: XML configuration from classpath.
   Legacy - only in older projects.

5. **GenericApplicationContext** (Spring tests): programmatic context
   for unit-testing infrastructure code.

Spring Boot selects automatically based on classpath: spring-webmvc on
classpath -> servlet context; spring-webflux only -> reactive context;
neither -> non-web context.

*What separates good from great:* Mentioning how Spring Boot's auto-detection
works demonstrates understanding of the framework's internals vs just using
@SpringBootApplication as a magic annotation.

---

**[MID] Q5 - [CONCEPTUAL] How can you interact with the ApplicationContext programmatically?**

Three legitimate use cases for programmatic context access:

1. **Dynamic bean lookup** (infrastructure code):
   `context.getBean(SomeService.class)` or `context.getBean("name", Class)`

2. **Publishing events**: `context.publishEvent(new MyEvent(this))`

3. **Registering beans dynamically** (rare):
   Cast to ConfigurableApplicationContext, get the BeanFactory,
   and register a BeanDefinition.

Accessing via injection:
- Implement ApplicationContextAware interface
- @Autowired ApplicationContext context (field or constructor)

When to use it: Only for infrastructure/framework code (custom starters,
plugin systems, test utilities). Business code should use constructor
injection exclusively.

*What separates good from great:* `context.getBeansOfType(Interface.class)`
returns a Map<String, Interface> of all beans implementing an interface.
Useful for plugin-style architectures where you want to discover all
registered implementations.

---

**[MID] Q6 - [CONCEPTUAL] How does Spring Boot's SpringApplication.run() work?**

SpringApplication.run() is a factory that:
1. Deduces the ApplicationContext type from the classpath (web vs non-web).
2. Loads SpringApplicationRunListeners from spring.factories.
3. Creates the Environment and loads application.properties/yml.
4. Creates the ApplicationContext of the deduced type.
5. Calls context.refresh() which triggers the full startup lifecycle.
6. Calls all ApplicationRunner and CommandLineRunner beans.
7. Publishes ApplicationReadyEvent.
8. Returns the ready ApplicationContext.

The key: steps 5-7 happen synchronously before run() returns. When run()
returns, the application is fully started and ready to handle requests.

*What separates good from great:* spring-boot-autoconfigure's
spring.factories (or AutoConfiguration.imports in Boot 2.7+) is read in
step 5 during the BFPP phase. Each auto-configuration class is evaluated
for its @Conditional conditions - only matching ones create beans.

---

**[SENIOR] Q7 - [CONCEPTUAL] What is context hierarchy and how is it used in Spring MVC?**

A parent-child context hierarchy is two ApplicationContexts where the child
inherits beans from the parent but the parent cannot see the child's beans.

Classic Spring MVC setup (before Spring Boot):
- Root ApplicationContext (parent): services, repositories, data access
- Web ApplicationContext (child): controllers, view resolvers, MVC config

This separation allowed the web context to be reloaded without reloading
the service layer, and enforced separation of concerns.

Spring Boot collapses this into a single ApplicationContext for simplicity.
The hierarchy is only relevant when using Spring Boot with traditional WAR
deployment, or when building multi-module applications with explicit context
hierarchies.

*What separates good from great:* The hierarchy affects @Transactional - a
transactional annotation on a controller (in the child context) might not
see the transaction manager (in the parent context) depending on how BPPs
are configured. This was a classic Spring MVC gotcha that Boot's single
context eliminated.

---

# Spring Boot Auto-configuration


---

### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is Spring Boot auto-configuration?**

beans based on classpath content, existing beans, and property values.
Implementation: @Configuration classes annotated with @Conditional variants
(@ConditionalOnClass, @ConditionalOnMissingBean, @ConditionalOnProperty etc.)
listed in META-INF/spring/AutoConfiguration.imports.

At startup, Spring Boot reads this list, evaluates conditions for each, and
activates only those where all conditions pass. The result: sensible defaults
for common setups with no configuration code required.

*What separates good from great:* Auto-configuration is not unique to Spring Boot
- it is just Spring's @Configuration + @Conditional mechanism used systematically.
  Every Spring Framework application could implement the same pattern. Boot just
  provides 100+ pre-written configurations and the framework to discover them.

---

**[JUNIOR] Q2 - [CONCEPTUAL] How does Spring Boot auto-configuration know what to configure?**

Two mechanisms work together:

1. **Classpath detection**: @ConditionalOnClass checks if a specific class is
   present. If spring-boot-starter-web is included, Spring MVC and Tomcat
   classes are present, so WebMvcAutoConfiguration activates.

2. **Bean absence check**: @ConditionalOnMissingBean ensures auto-configuration
   only activates when you haven't defined a conflicting bean. If you define
   your own DataSource, DataSourceAutoConfiguration is skipped.

The discovery mechanism: SpringFactoriesLoader reads
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
from all JARs on the classpath. This file lists auto-configuration class names.
Spring Boot 3 changed from spring.factories to this dedicated file for clarity.

*What separates good from great:* The SpringFactoriesLoader mechanism is also
used for ApplicationListeners, ApplicationContextInitializers, and FailureAnalyzers.
Understanding this mechanism explains how third-party libraries plug into Spring
Boot without any registration code in your application.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How do you override or disable auto-configuration?**

Three ways:

1. **Define your own bean** (implicit override):
   Spring Boot's @ConditionalOnMissingBean checks for an existing bean before
   auto-configuring. Define a @Bean of the same type - auto-config skips.

2. **Explicit exclude**:
   ```java
   @SpringBootApplication(exclude = {
       DataSourceAutoConfiguration.class
   })
   ```
> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Or in application.properties:
`spring.autoconfigure.exclude=...DataSourceAutoConfiguration`

3. **Conditional properties**:
   Many auto-configurations check @ConditionalOnProperty.
   Setting the right property can enable or disable them.

*What separates good from great:* When excluding auto-configuration, you take
responsibility for providing the beans it would have created. Excluding
DataSourceAutoConfiguration without providing your own DataSource will cause
NoSuchBeanDefinitionException when anything tries to inject a DataSource.

---

**[MID] Q4 - [DEBUGGING] How do you debug which auto-configurations are active?**

Three methods:

1. **--debug flag**: `java -jar app.jar --debug` or `spring.main.debug=true`.
   Prints the CONDITIONS EVALUATION REPORT showing positive matches (activated),
   negative matches (not activated and why), and unconditional classes.

2. **/actuator/conditions endpoint**: Same report as /actuator/conditions.
   Requires spring-boot-starter-actuator and exposure configuration.

3. **@SpringBootTest(properties = "logging.level.root=DEBUG")**: In tests,
   activate debug logging to see condition evaluation in test output.

Reading the conditions report: "Positive matches" shows what fired. "Negative
matches" shows what did not and which condition failed. Mostly you read the
negative matches to find why something expected did not activate.

*What separates good from great:* The conditions report is not just for
debugging - it is useful for security audits. Check which auto-configurations
are active in a production build to ensure no unexpected feature is enabled.

---

**[MID] Q5 - [CONCEPTUAL] What is the difference between @ConditionalOnClass and @ConditionalOnMissingClass?**

@ConditionalOnClass(X.class): configuration activates only if class X IS on
the classpath. Used to tie auto-configuration to library presence.

@ConditionalOnMissingClass("com.example.X"): configuration activates only if
class X is NOT on the classpath. Note: the parameter is a String to avoid
compilation failure when the class is absent - you cannot reference a class
in a @ConditionalOnClass annotation if the class might not compile.

Common pattern: provide a default implementation that is replaced when a
specific library is present:
```java
@ConditionalOnMissingClass("com.fasterxml.jackson.databind.ObjectMapper")
class FallbackSerializerConfig { ... }

@ConditionalOnClass(ObjectMapper.class)
class JacksonSerializerConfig { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Both annotations use String form when there
is a risk the class might not be on the classpath during compilation of the
auto-configuration module itself. The condition evaluation happens at runtime
where the class is either present or not.

---

**[MID] Q6 - [CONCEPTUAL] What is a Spring Boot starter and how does it relate to auto-configuration?**

A starter is a convenience POM that:
1. Pulls in the right set of transitive dependencies for a feature
2. Includes (directly or transitively) the spring-boot-autoconfigure artifact
   which contains the auto-configuration classes

Example: spring-boot-starter-web includes:
- spring-webmvc (Spring MVC classes)
- spring-boot-starter-tomcat (embedded Tomcat)
- jackson-databind (JSON serialization)
- hibernate-validator (bean validation)
- spring-boot-autoconfigure (contains WebMvcAutoConfiguration etc.)

The starter handles dependency management; auto-configuration handles bean
creation. They are complementary: the starter puts the right classes on the
classpath; auto-configuration detects those classes and creates the beans.

*What separates good from great:* Custom starters for internal libraries follow
the same pattern. Create a -autoconfigure module with the auto-configuration
class (registered in AutoConfiguration.imports) and a -starter module with
the dependency on your library and the -autoconfigure module. Teams consuming
your library get auto-configuration for free.

---

**[SENIOR] Q7 - [CONCEPTUAL] What are the performance implications of auto-configuration?**

Auto-configuration has two startup costs:

1. **Classpath scanning**: Spring Boot reads AutoConfiguration.imports from
   all JARs, evaluates conditions for each class. With ~130 auto-configuration
   candidates and their conditional checks, this adds 50-200ms to startup.

2. **Bean creation**: Each activated auto-configuration creates beans, which
   have their own initialization cost.

Optimizations:
- `spring.jmx.enabled=false`: disable JMX if not needed (saves 50ms)
- `spring.data.jpa.repositories.bootstrap-mode=lazy`: lazy repository bootstrap
- Exclude unused auto-configurations explicitly
- Spring Boot 3 AOT: pre-processes conditions at build time, generating source
  for beans instead of runtime reflection. Dramatically reduces startup time
  for GraalVM native images.
- @SpringBootApplication(proxyBeanMethods = false): faster context creation
  when inter-@Bean method calls are not needed.

*What separates good from great:* Spring Boot 3 AOT compilation is the future
of Boot performance. It moves condition evaluation and bean definition generation
to build time, producing a pre-computed bean factory that starts without
classpath scanning. This is what enables fast cold starts in serverless and
sub-100ms startup in GraalVM native images.

---


# @SpringBootApplication


---

### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What three annotations does @SpringBootApplication combine?**

1. @Configuration: the annotated class is a source of @Bean definitions.
   Spring processes it with ConfigurationClassPostProcessor.

2. @EnableAutoConfiguration: activates Spring Boot's auto-configuration
   mechanism. Imports AutoConfigurationImportSelector which reads
   AutoConfiguration.imports and processes all matching @Configuration candidates.

3. @ComponentScan: scans the annotated class's package and all sub-packages
   for @Component-annotated classes and registers them as bean definitions.

Each can be used independently when you need to configure them differently
from @SpringBootApplication's defaults.

*What separates good from great:* @SpringBootApplication is itself annotated
with @SpringBootConfiguration (not @Configuration directly). @SpringBootConfiguration
extends @Configuration and adds the TestAutoConfigurationExcludeFilter - used
by @SpringBootTest to avoid scanning test configurations in production code.

---

**[JUNIOR] Q2 - [CONCEPTUAL] Why does the placement of the Application class matter?**

@ComponentScan (included in @SpringBootApplication) scans the package of the
annotated class and all its sub-packages. If Application.java is in
com.example.order, Spring scans com.example.order.** but NOT com.example.payment.

Spring Boot convention: place Application.java in the root package
(com.example, not com.example.order) so all modules are scanned:
```
com.example.Application     <- root package
com.example.order.*         <- scanned
com.example.payment.*       <- scanned
com.example.user.*          <- scanned
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Violating this convention causes NoSuchBeanDefinitionException for components
in packages not under the application root.

*What separates good from great:* In multi-module Maven projects, the
Application class may be in a different module than some components. Use
scanBasePackages or create a configuration class with @ComponentScan in
the relevant module.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is the difference between @SpringBootApplication and @SpringBootTest?**

@SpringBootApplication is used in production code to mark the main entry point.
@SpringBootTest is a test annotation that creates a full Spring application
context for integration tests.

@SpringBootTest finds the @SpringBootApplication class automatically (searches
upward from the test class's package) and uses it to configure the test context.
It can start the full embedded server (webEnvironment = RANDOM_PORT) or create
just the context without a server.

*What separates good from great:* @SpringBootTest is expensive - it creates the
full context including auto-configuration. For unit tests, use @ExtendWith(Mock
itoExtension.class) without Spring. For slice tests (just MVC, just data layer),
use @WebMvcTest, @DataJpaTest, @JsonTest etc. These start only the relevant
slice of auto-configuration, making tests faster.

---

**[MID] Q4 - [CONCEPTUAL] What is proxyBeanMethods = false in @SpringBootApplication?**

By default, @Configuration classes are subclassed by CGLIB. This ensures that
@Bean methods called from other @Bean methods return the singleton bean (not
a new instance). CGLIB subclassing adds startup overhead.

proxyBeanMethods = false skips CGLIB subclassing. @Bean methods are treated
as regular factory methods - inter-method calls create new instances rather
than returning singletons.

Use false when:
- None of your @Bean methods call other @Bean methods in the same class
- You want faster context startup (skips CGLIB proxy generation)
- You are building a GraalVM native image (CGLIB proxies have restrictions)

Use true (default) when:
- You have inter-@Bean method calls that should return singletons

Spring Boot uses proxyBeanMethods = false in its own internal auto-configuration
classes for performance.

*What separates good from great:* @SpringBootApplication itself sets
proxyBeanMethods = false in recent Spring Boot versions. This means adding
@Bean methods to your Application class where one @Bean method calls another
may not work as expected - the called @Bean creates a new instance each time.

---

**[MID] Q5 - [CONCEPTUAL] How do you scan multiple packages with @SpringBootApplication?**

Use the scanBasePackages attribute:
```java
@SpringBootApplication(
    scanBasePackages = {
        "com.example.orders",
        "com.example.shared",
        "com.thirdparty.components"
    }
)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Or use scanBasePackageClasses for type-safe references:
```java
@SpringBootApplication(
    scanBasePackageClasses = {
        OrderService.class,     // scans com.example.orders
        SharedConfig.class      // scans com.example.shared
    }
)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

scanBasePackageClasses is safer: it is not affected by package rename
refactoring since it references a class in the package.

*What separates good from great:* If the packages you need to scan are in
a different Maven module, the better pattern is to have a @Configuration
class in that module with its own @ComponentScan, and import it with @Import
or include it in AutoConfiguration.imports. This keeps the configuration
local to the module it belongs to.

---

**[MID] Q6 - [CONCEPTUAL] Can you have multiple @SpringBootApplication classes?**

You can have multiple classes annotated with @SpringBootApplication but you
should only run one at a time. Each call to SpringApplication.run() creates
a new ApplicationContext starting from that class as the configuration root.

In a multi-module project, each module may have its own @SpringBootApplication
for testing purposes. Spring's test framework finds the nearest
@SpringBootApplication class when loading test contexts.

The rule: one @SpringBootApplication per deployed unit (per JVM process).
Multiple per codebase for testing is fine.

*What separates good from great:* Spring Boot test caching uses the combination
of @SpringBootApplication class + test annotations as the context cache key.
Two tests using the same @SpringBootApplication class share a cached context.
Tests with different configurations get different contexts.

---

**[SENIOR] Q7 - [CONCEPTUAL] What is the difference between @SpringBootApplication and @SpringBootConfiguration?**

@SpringBootConfiguration is a specialization of @Configuration with:
- @Configuration behaviour: the class is a bean definition source
- TestAutoConfigurationExcludeFilter: prevents test @Configuration classes
  from being loaded in the production context

@SpringBootApplication includes @SpringBootConfiguration (not plain @Configuration).
This means:
- @SpringBootTest finds @SpringBootApplication classes by looking for
  @SpringBootConfiguration on the classpath
- Test @Configuration classes in your test sources are excluded from the
  production context

You rarely use @SpringBootConfiguration directly. It is present because
@SpringBootApplication is a composed annotation and needs @Configuration
behaviour - it uses @SpringBootConfiguration to get the TestConfiguration
exclusion too.

*What separates good from great:* The exclusion of test configurations
(TestAutoConfigurationExcludeFilter) prevents accidentally loading test
@Configuration or test @Bean definitions into the production context.
This matters in build systems where production and test classes are on
the same classpath during compilation.

---

# Embedded Server


---

### 🎯 Interview Deep-Dive

**Timing:** Easy ★☆☆ - 7 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is an embedded server in Spring Boot?**

An embedded server is a servlet container (Tomcat, Jetty, Undertow) or
reactive server (Netty) bundled inside the Spring Boot application JAR and
started programmatically as part of the ApplicationContext refresh.

The result: a self-contained executable JAR that includes the application,
all dependencies, AND the server. Running `java -jar app.jar` starts both
the application and the server simultaneously.

This is the default Spring Boot deployment model - no separate server
installation required.

*What separates good from great:* The embedded server is started in
onRefresh() of AnnotationConfigServletWebServerApplicationContext. The
EmbeddedWebServerFactoryCustomizerAutoConfiguration creates the appropriate
server factory based on classpath detection. The server starts before
ContextRefreshedEvent fires, so by the time your application listener
receives that event, the server is already accepting connections.

---

**[JUNIOR] Q2 - [CONCEPTUAL] How do you change the default Tomcat server to Jetty?**

Exclude Tomcat starter, add Jetty starter in pom.xml:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Zero code changes required. Spring Boot's auto-configuration detects
JettyServletWebServerFactory on the classpath and uses it.

When to use Jetty: higher throughput for WebSocket-heavy applications;
slightly lower memory footprint than Tomcat.

*What separates good from great:* Undertow (Red Hat's server) is the lowest
footprint option and handles very high concurrency well. Used by Red Hat
internally. The performance difference between Tomcat, Jetty, and Undertow
is small for most applications - choose based on operational familiarity.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How does graceful shutdown work in Spring Boot?**

Graceful shutdown (Spring Boot 2.3+):
Configure: `server.shutdown=graceful` in application.properties.

When a SIGTERM is received:
1. Kubernetes/load balancer readiness probe starts failing (application sets
   readiness to DOWN).
2. Load balancer stops routing new requests to the pod.
3. Spring Boot waits for in-flight requests to complete (default 30 sec timeout).
4. After all requests complete (or timeout), the context closes and server stops.
5. Process exits.

Configuration:
```properties
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=30s
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* The Kubernetes deployment spec should set
terminationGracePeriodSeconds longer than the Spring Boot shutdown timeout.
If Kubernetes kills the pod before Spring Boot finishes, you still get abrupt
termination. A common production mistake is setting Spring Boot graceful shutdown
to 60s but Kubernetes terminationGracePeriodSeconds to 30s.

---

**[MID] Q4 - [CONCEPTUAL] What is the difference between a fat JAR and a WAR?**

**Fat JAR** (Spring Boot default):
- Self-contained executable JAR with all dependencies including embedded server
- Run with: `java -jar app.jar`
- Includes: your code + all JARs (Spring, Tomcat, etc.) + Spring Boot Loader
- For Docker: one file, one command, no external dependencies

**WAR** (traditional deployment):
- Web Application Archive without embedded server
- Deploy to an external Tomcat/JBoss/WebSphere
- Server is NOT included - must be installed separately
- For WAR: extend SpringBootServletInitializer, set packaging to war

When to use WAR: enterprise environments requiring centralised application
server management, server-level SSL termination, or shared resources across
applications on one server.

*What separates good from great:* Layered JARs (Spring Boot 2.3+) split the
fat JAR into layers: dependencies (rarely changes), Spring Boot loader, snapshot
dependencies, application code. Docker layers cache each layer separately -
only the application code layer is rebuilt on code changes, making Docker builds
much faster.

---

**[MID] Q5 - [CONCEPTUAL] How do you configure Tomcat connection pool and thread settings?**

```properties
# Thread pool
server.tomcat.threads.max=200       # default 200
server.tomcat.threads.min-spare=10  # default 10

# Connection settings
server.tomcat.accept-count=100  # queue depth when all threads busy
server.tomcat.max-connections=8192  # total connections
server.tomcat.connection-timeout=20000  # ms

# Compression
server.compression.enabled=true
server.compression.min-response-size=2048
```

> **Code walkthrough:** This Compression example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Choosing max-threads: a general rule is CPU-count * 2 for CPU-bound work;
for I/O-bound work (database calls, HTTP calls) you can go much higher
(200-500) since threads spend most time waiting.

*What separates good from great:* Virtual threads (Spring Boot 3.2+ with
Java 21+): `spring.threads.virtual.enabled=true` enables virtual threads
for Tomcat. Virtual threads are extremely cheap so max-threads is no longer
the limiting factor - you can handle tens of thousands of concurrent requests
without tuning. This is a paradigm shift for thread-per-request servlet model.

---

**[SENIOR] Q6 - [MECHANISM] How do you add HTTPS to a Spring Boot embedded server?**

Configure SSL in application.properties:
```properties
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.jks
server.ssl.key-store-password=secret
server.ssl.key-store-type=JKS
server.ssl.key-alias=myapp
server.port=8443
```

> **Code walkthrough:** This Compression example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

For production, prefer externally managed SSL (load balancer, Kubernetes
ingress controller) over application-managed SSL. Terminating SSL at the
load balancer is the standard pattern - the application sees plain HTTP
internally.

*What separates good from great:* If you need both HTTP and HTTPS, Spring
Boot only supports one port natively. For dual ports, configure a second
connector programmatically via WebServerFactoryCustomizer. For redirecting
HTTP to HTTPS, a filter (ChannelProcessingFilter) is the standard approach.

---

**[SENIOR] Q7 - [MECHANISM] What is the Spring Boot fat JAR structure?**

```
app.jar
|-- META-INF/
|   |-- MANIFEST.MF  (Main-Class: JarLauncher)
|-- BOOT-INF/
|   |-- classes/    <- your compiled .class files
|   |-- lib/        <- all dependency JARs (including Tomcat)
|   |   |-- tomcat-embed-core-10.x.jar
|   |   |-- spring-web-6.x.jar
|   |   |-- ... (hundreds of JARs)
|   |-- classpath.idx  (ordered list for class loading)
|   |-- layers.idx     (layered JAR metadata)
|-- org/
    |-- springframework/boot/loader/
        |-- JarLauncher.class  <- Boot's custom class loader
        |-- LaunchedURLClassLoader.class
```

> **Code walkthrough:** This Compression example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

JarLauncher is the real Main-Class. It sets up a custom classloader that
can load classes from nested JARs (JARs inside the fat JAR), then calls
your Application.main().

*What separates good from great:* The Spring Boot Loader solves a
fundamental Java problem: the standard JVM cannot load classes from JARs
nested inside JARs. Boot's custom classloader makes this work without
exploding the JAR at startup, which was the older approach. Understanding
this explains why Spring Boot apps are immediately runnable with java -jar.

---

# Bean Scopes


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What are the Spring bean scopes?**

Five built-in scopes:
1. **Singleton** (default): one instance per ApplicationContext. Shared
   across all threads. Must be stateless/thread-safe.
2. **Prototype**: new instance for each call to getBean() or each injection
   point. Spring does not manage lifecycle after handoff.
3. **Request** (web): one instance per HTTP request. Destroyed when request
   ends. Uses RequestContextHolder (ThreadLocal) internally.
4. **Session** (web): one instance per HTTP session. Lives until session
   expires or is invalidated.
5. **Application** (web): one instance per ServletContext. Like singleton
   but web-aware.

Plus WebSocket scope for WebSocket sessions.

*What separates good from great:* Custom scopes via the Scope interface.
Spring Cloud @RefreshScope is the most well-known example: a custom scope
that destroys and recreates beans on config refresh.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the difference between singleton and prototype scope?**

Singleton:
- One instance per ApplicationContext
- Created eagerly at context refresh
- Returned from singleton cache on every getBean() call
- Thread-safe requirement: mandatory
- @PreDestroy called on context close

Prototype:
- New instance on every getBean() call or injection
- Created lazily (only when requested)
- Spring does NOT cache or destroy prototype instances
- Thread-safe requirement: none (each caller has its own)
- @PreDestroy NOT called (Spring does not track prototypes)

Decision: if the bean holds state that varies per use, prototype. If it holds
only collaborators, singleton.

*What separates good from great:* The @PreDestroy caveat for prototypes is
often forgotten. If prototype beans hold resources (connections, file handles),
you MUST close them yourself. Spring will not do it.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How do you inject a prototype bean into a singleton?**

The naive approach (direct injection) gives you one prototype instance,
created at singleton startup - defeating the purpose of prototype scope.

Three correct approaches:

1. **ObjectFactory<T>** (cleanest):
   ```java
   @Autowired ObjectFactory<MyPrototype> factory;
   // In method: factory.getObject() returns fresh instance
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

2. **@Lookup method injection**:
   ```java
   @Service public abstract class MyService {
       public void process() {
           MyPrototype p = createPrototype();
       }
       @Lookup public abstract MyPrototype createPrototype();
   }
   ```
> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Spring generates a CGLIB subclass implementing createPrototype()
as a getBean() call.

3. **ApplicationContext.getBean()**:
   Direct context access; Service Locator anti-pattern but explicit.

*What separates good from great:* @Lookup is the most elegant for cases where
you need the fresh prototype frequently. ObjectFactory is preferred when you
want to be explicit about getting a new instance at the call site.

---

**[MID] Q4 - [CONCEPTUAL] How does proxyMode work for request and session scoped beans?**

When you declare a request-scoped bean, Spring needs to inject it into singleton
beans. The problem: the singleton is created at startup, but request-scoped
beans don't exist yet (no request at startup).

Solution: Spring injects a PROXY (CGLIB subclass or JDK proxy) that looks up
the real scoped instance on each method call. The proxy delegates to the current
request's real instance via RequestContextHolder (ThreadLocal).

```java
@Component
@Scope(
  value = WebApplicationContext.SCOPE_REQUEST,
  proxyMode = ScopedProxyMode.TARGET_CLASS
)
public class RequestContext { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

ScopedProxyMode.TARGET_CLASS: CGLIB proxy for concrete classes.
ScopedProxyMode.INTERFACES: JDK proxy for interfaces.

Without proxyMode: injecting a request-scoped bean into a singleton throws
BeanCreationException at startup.

*What separates good from great:* The proxy mechanism is the same mechanism
Spring uses for @Transactional and @Async - the proxy intercepts calls and
adds behaviour. Understanding that proxies are the universal Spring extension
mechanism explains many Spring behaviours.

---

**[MID] Q5 - [CONCEPTUAL] What is the @RefreshScope in Spring Cloud?**

@RefreshScope is a custom Spring scope that allows beans to be refreshed
(destroyed and recreated) when configuration changes.

When /actuator/refresh is called:
1. The refresh scope cache is cleared.
2. Next time a @RefreshScope bean is accessed, Spring creates a new instance.
3. The new instance picks up the latest configuration values.

This is how Spring Cloud Config Server works: change a value in the config
repository, call /actuator/refresh, and beans get the new values without
restarting the application.

Implementation: RefreshScope implements the Scope interface. It maintains a
cache of instances keyed by bean name. On refresh, it clears the cache.
The ScopedProxy mechanism ensures that singleton beans holding references
to refresh-scoped beans always go through the proxy, which checks the cache.

*What separates good from great:* @RefreshScope only refreshes the specific
bean - it does not restart the entire context. Beans that receive their
configuration via @Value need @RefreshScope to pick up new values. Beans
using Environment.getProperty() always get fresh values without @RefreshScope.

---

**[MID] Q6 - [CONCEPTUAL] What is the Application scope and when would you use it?**

Application scope creates one bean instance per ServletContext. In a standard
Spring Boot application with one embedded server, application scope is
functionally identical to singleton scope.

Application scope is meaningful when:
- Multiple Spring ApplicationContexts share the same ServletContext (rare)
- You need a bean visible to both Spring and the raw servlet context
  (via servlet context attributes)

In practice, application scope is rarely used. Singleton suffices for all
application-wide shared state. Application scope's main use case is legacy
integration with servlet-level code that accesses beans via
servletContext.getAttribute().

*What separates good from great:* Application scope beans are stored as
ServletContext attributes, not in Spring's singleton cache. This means they
can be accessed by non-Spring code via ServletContext, which singleton beans
cannot. For greenfield Spring Boot applications, this distinction does not matter.

---

**[SENIOR] Q7 - [CONCEPTUAL] How do you declare a custom scope?**

```java
// 1. Implement the Scope interface
public class TenantScope implements Scope {
    // Map of tenant ID -> bean instances
    private Map<String, Map<String, Object>> tenantBeans
        = new ConcurrentHashMap<>();

    @Override
    public Object get(String name,
                      ObjectFactory<?> factory) {
        String tenantId = TenantContext.currentTenantId();
        return tenantBeans
            .computeIfAbsent(tenantId, k -> new HashMap<>())
            .computeIfAbsent(name, k -> factory.getObject());
    }

    @Override
    public Object remove(String name) { /* ... */ }
    // ... other interface methods
}

// 2. Register the custom scope
@Configuration
public class TenantScopeConfig {
    @Bean
    public static CustomScopeConfigurer scopeConfigurer() {
        CustomScopeConfigurer configurer =
            new CustomScopeConfigurer();
        configurer.addScope("tenant", new TenantScope());
        return configurer;
    }
}

// 3. Use the custom scope
@Component
@Scope(value = "tenant",
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class TenantConfig { ... }
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using Spring annotation. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Custom scopes unlock powerful patterns:
per-tenant isolation, per-job state in batch processing, per-WebSocket session
state. Spring Cloud's @RefreshScope and @RequestScope are both implemented
as custom scopes. The Scope interface is small (4 methods) and straightforward
to implement.

---

**[SENIOR] Q8 - [CONCEPTUAL] What happens when a singleton bean depends on a wider-scoped bean**
(request/session)?

Without proxyMode on the narrower-scoped bean: Spring throws BeanCreationException
at startup because it tries to inject a non-existent request-scoped bean into
a singleton during context refresh (before any request exists).

With proxyMode: Spring injects a CGLIB/JDK proxy into the singleton. The proxy
is created at startup time. On each method call to the proxy, it:
1. Gets the current scope identifier (request ID, session ID)
2. Looks up the real scoped instance in the scope's storage
3. Delegates the method call to that instance

This allows singletons to "use" narrower-scoped beans transparently.

*What separates good from great:* The opposite case (wide scope in narrow scope)
is not a problem - a singleton injected into a request-scoped bean works fine
because the singleton is always available. The problem is only narrow->wide
dependency direction.

---

**[SENIOR] Q9 - [CONCEPTUAL] How do you test beans with non-singleton scopes?**

For request-scoped beans in tests:

1. **@WebMvcTest**: creates a web test context with request scope active.

2. **MockHttpServletRequest**: manually create a request context:
   ```java
   @BeforeEach
   void setUp() {
       MockHttpServletRequest request
           = new MockHttpServletRequest();
       RequestContextHolder.setRequestAttributes(
           new ServletRequestAttributes(request));
   }
   @AfterEach
   void tearDown() {
       RequestContextHolder.resetRequestAttributes();
   }
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

3. **@SpringBootTest(webEnvironment=MOCK)**: creates a mock web environment
   with request scope support.

For prototype beans: just use `new` or ApplicationContext.getBean() - no
special setup needed.

*What separates good from great:* @Scope("request") beans in unit tests are
often replaced with mocks rather than activating request scope. If the bean
under test depends on a request-scoped bean, mock the request-scoped bean
with @MockBean or provide it via constructor injection - the test does not
need a full web context.

---

# Bean Lifecycle


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is the full Spring bean lifecycle?**

Complete sequence for a singleton bean:
1. Constructor (object created, fields null)
2. Dependency injection (all @Autowired satisfied)
3. Aware callbacks (BeanNameAware, ApplicationContextAware if implemented)
4. BeanPostProcessor.postProcessBeforeInitialization() for all registered BPPs
5. @PostConstruct method
6. InitializingBean.afterPropertiesSet() (if implemented)
7. Custom initMethod (if @Bean(initMethod="...") specified)
8. BeanPostProcessor.postProcessAfterInitialization() <- AOP proxies created here
9. Bean ready; stored in singleton cache

On shutdown:
10. @PreDestroy method
11. DisposableBean.destroy() (if implemented)
12. Custom destroyMethod

*What separates good from great:* The reason step 8 is critical: after this
step, the bean reference in the context may be a proxy (not your original
object). When other beans inject this bean, they receive the proxy.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the difference between @PostConstruct and @Bean(initMethod)?**

Both run after dependency injection. The difference:

@PostConstruct:
- Annotation on a method inside the bean class
- Standard JSR-250 annotation (not Spring-specific)
- Works for any Spring-managed bean (component-scanned or @Bean)
- Cannot be used for third-party classes you cannot annotate

@Bean(initMethod = "initMethodName"):
- Specified on the @Bean factory method in @Configuration
- Required for third-party classes that have init methods but are not
  Spring-annotated
- The method can have any name matching the bean's actual method

Example: `@Bean(initMethod = "start", destroyMethod = "stop")` for
a HazelcastInstance which has start/stop lifecycle methods.

*What separates good from great:* For classes you own, prefer @PostConstruct.
For third-party classes, use @Bean initMethod. Never implement InitializingBean
in new code - it's Spring-specific and @PostConstruct achieves the same result.

---

**[JUNIOR] Q3 - [CONCEPTUAL] Why does @Transactional not work in @PostConstruct?**

The AOP proxy that implements @Transactional is created in BeanPostProcessor
post-initialization (step 8 in lifecycle). @PostConstruct runs at step 5 -
BEFORE the proxy exists.

When @PostConstruct calls a @Transactional method on `this`, it calls the
real object directly, not through the proxy. No transaction is started.
If the @Transactional method fails, no rollback occurs.

Solution: Use ApplicationReadyEvent listener for transactional initialization:
```java
@EventListener(ApplicationReadyEvent.class)
@Transactional
public void transactionalInit() {
    // Called via proxy - transaction works correctly
}
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

ApplicationReadyEvent fires after full context refresh including proxy creation.

*What separates good from great:* This is the same root cause as the self-
invocation problem: calling this.transactionalMethod() bypasses the proxy.
Whether from @PostConstruct or from another method in the same class, the
result is the same: the proxy is bypassed.

---

**[MID] Q4 - [HANDS-ON] What is a BeanPostProcessor and when would you write one?**

A BeanPostProcessor is a Spring extension point that processes every bean
instance before and after initialization. It receives every bean in the context
and can:
- Return the original bean (no-op)
- Return a different object (the proxy mechanism works this way)
- Throw an exception to veto bean creation

Two methods:
- postProcessBeforeInitialization(bean, beanName): called before @PostConstruct
- postProcessAfterInitialization(bean, beanName): called after @PostConstruct

AbstractAutoProxyCreator (which creates @Transactional proxies) is a BPP. All
Spring AOP works through BPPs.

When to write a custom BPP: custom annotation processing (scan all beans for
a custom annotation and apply behaviour), bean validation, custom metric
registration, custom lifecycle tracking.

*What separates good from great:* BPPs must be registered as beans themselves.
Spring detects them early in the refresh cycle (before other beans are created)
so they are available to process all beans. Creating a BPP that depends on
another bean can cause issues because Spring must create the BPP before the
dependency is available.

---

**[MID] Q5 - [CONCEPTUAL] What is the BeanFactoryPostProcessor and how is it different from BeanPostProcessor?**

BeanFactoryPostProcessor (BFPP) operates on BeanDefinitions BEFORE any bean
instance is created. BeanPostProcessor (BPP) operates on bean INSTANCES during
their lifecycle.

BFPP lifecycle:
- All BFPPs run during context refresh before singleton instantiation
- Can modify, add, or remove BeanDefinitions
- Cannot create regular beans (no DI available yet)

BPP lifecycle:
- Runs during bean instantiation, once per bean
- Can wrap beans in proxies or validate them
- Has access to the fully-initialized container

Key BFPP implementations: ConfigurationClassPostProcessor (processes
@Configuration, @ComponentScan, @Import), PropertySourcesPlaceholderConfigurer
(resolves ${...} in BeanDefinitions).

*What separates good from great:* Both are beans discovered early. The
critical difference is timing: BFPP modifies the recipe (BeanDefinitions);
BPP modifies the cooked dish (bean instances). If you need to change how a
bean is configured (change a property value, add a new @Bean definition),
use BFPP. If you need to wrap or validate bean instances, use BPP.

---

**[MID] Q6 - [CONCEPTUAL] What happens if @PostConstruct throws an exception?**

If @PostConstruct throws, Spring throws BeanCreationException and the entire
application context refresh fails. The application does not start.

This is intentional: Spring considers an application with a failed initialization
to be in an invalid state that should not serve traffic. Fail-fast at startup
is better than a partially initialized application accepting requests.

Best practices:
- Use @PostConstruct for assertions and validation that must pass
- Wrap recoverable errors in try-catch if startup should continue
- Do not use @PostConstruct for non-essential warm-up (use ApplicationReadyEvent)
- Log clearly what failed so operations can diagnose quickly

*What separates good from great:* In Kubernetes, a failed startup causes the
container to restart (CrashLoopBackOff). A clear startup error log message
with the configuration issue is the difference between a 5-minute fix and
a 2-hour investigation. @PostConstruct validation methods should throw
exceptions with descriptive messages: "Required property 'api.key' is missing"
not just NullPointerException.

---

**[SENIOR] Q7 - [CONCEPTUAL] What is the order of initialization callbacks?**

Within one bean, the order is:
1. @PostConstruct method (may have multiple - all called)
2. InitializingBean.afterPropertiesSet()
3. Custom initMethod specified in @Bean(initMethod="...")

For multiple beans: Spring initializes beans in dependency order (dependencies
first). You can also use @DependsOn to express initialization dependencies
that are not reflected in @Autowired.

Between @PostConstruct methods in the same bean: if multiple @PostConstruct
methods exist, the order is not guaranteed by Spring. If order matters, use
a single @PostConstruct that calls other methods in the desired order.

*What separates good from great:* @Order and @Priority affect AOP advisor
ordering and event listener ordering, but NOT bean initialization order.
Bean initialization order is determined by dependency graph, not @Order.
If you want one bean to initialize before another, declare an explicit
@Autowired dependency or use @DependsOn.

---

**[SENIOR] Q8 - [CONCEPTUAL] What is @DependsOn and when do you use it?**

@DependsOn forces Spring to initialize one bean before another, even when
there is no @Autowired dependency between them. This is for implicit dependencies
that cannot be expressed via injection.

Examples:
1. A service that requires a database schema to exist, but the schema
   migration bean (Flyway) is not in its dependency chain.
2. A bean that registers itself with a global registry during @PostConstruct -
   other beans that use the registry need it initialized first.

```java
@Bean
@DependsOn("flywayInitializer")
public OrderRepository orderRepository() {
    return new OrderRepositoryImpl(dataSource());
}

@Bean("flywayInitializer")
public Flyway flyway() {
    return Flyway.configure()
        .dataSource(dataSource())
        .load();
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* @DependsOn is a code smell in most cases.
Implicit dependencies that require @DependsOn usually indicate a design issue:
the ordering constraint should be expressed as an explicit bean dependency.
Legitimate use cases: legacy code integration, third-party library initialization,
global registries.

---

**[SENIOR] Q9 - [CONCEPTUAL] How does ApplicationReadyEvent differ from ContextRefreshedEvent**
for post-startup initialization?

ContextRefreshedEvent:
- Fires when the ApplicationContext is refreshed (all beans created and wired)
- Fires on EVERY refresh (multiple times in web applications - parent and child
  context each refresh)
- @Transactional works at this point (proxies are created)
- No embedded server started yet

ApplicationReadyEvent (Spring Boot only):
- Fires after the entire application is ready to serve requests
- Fires exactly ONCE per application startup
- Embedded server is started and accepting connections
- Fired AFTER CommandLineRunner and ApplicationRunner beans have run

For most post-startup tasks: prefer ApplicationReadyEvent for Spring Boot
applications. It fires once, after everything is ready. For ContextRefreshedEvent,
check event.getApplicationContext() to avoid double-processing.

*What separates good from great:* ApplicationStartedEvent fires after
CommandLineRunner/ApplicationRunner beans have run. ApplicationReadyEvent fires
after Spring Boot's lifecycle management. For true "ready to serve traffic"
initialization (e.g., connect to external services), ApplicationReadyEvent is
semantically correct.

---

# Spring Data JPA Repository


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [HANDS-ON] How does Spring Data JPA generate repository implementations?**

At application startup:
1. JpaRepositoriesAutoConfiguration activates (classpath: JPA, Spring Data JPA)
2. JpaRepositoryFactoryBean scans all repository interfaces
3. For each interface, it creates a JDK dynamic proxy
4. Standard methods (findAll, save, etc.) delegate to SimpleJpaRepository
5. Derived query methods: PartTree parser tokenizes the method name into
   Subject + Predicate, then builds a JPA Criteria query or JPQL string
6. @Query methods: literal JPQL/SQL validated by JPA provider at startup
7. The proxy is registered as a Spring bean

If a derived method name has an invalid property reference, Spring throws
PropertyReferenceException during context refresh - fail-fast, not at runtime.

*What separates good from great:* SimpleJpaRepository is the actual
implementation - you can read its source code to understand exactly what
each standard method does. For example, save() calls either persist() or
merge() based on isNew() check (entity has null ID or @Version is null).

---

**[JUNIOR] Q2 - [DEBUGGING] What is the N+1 query problem and how do you diagnose it?**

N+1: loading N entities and then executing 1 additional query per entity
to load an associated collection.

```
1 query: SELECT * FROM orders WHERE email = 'a@b.com'
-> returns 100 orders

100 queries: SELECT * FROM order_items WHERE order_id = ?
-> executed for each order when items are accessed
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice using SQL. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Total: 101 queries instead of 1 or 2.

Diagnosis:
1. spring.jpa.show-sql=true: prints all executed queries
2. p6spy: intercepts JDBC calls, shows actual SQL with parameters
3. Hibernate statistics: hibernate.generate_statistics=true shows
   "Statements.prepared" count per request
4. In production: slow query log + application performance monitoring

Fix options (ranked by preference):
1. DTO projection: SELECT only what you need, no entity loading
2. JOIN FETCH: `SELECT o FROM Order o LEFT JOIN FETCH o.items WHERE ...`
3. @EntityGraph on repository method: `@EntityGraph(attributePaths = {"items"})`
4. Batch size: `@BatchSize(size=100)` on the collection - loads in batches

*What separates good from great:* Eager fetching (FetchType.EAGER) does NOT
fix N+1 - it moves it to a different query. EAGER runs a separate query per
entity at load time (even when you don't need the association). JOIN FETCH is
the correct fix for cases where you always need the association.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is the difference between @Query JPQL and native SQL?**

**JPQL (@Query)**:
- Queries entity names and property names, not table/column names
- Database-agnostic (works with any JPA provider/database)
- JPA provider translates to SQL at startup (validated, fails fast)
- Supports entity associations naturally (LEFT JOIN FETCH o.items)

**Native SQL (@Query(nativeQuery = true))**:
- Raw SQL against actual table and column names
- Database-specific syntax (PostgreSQL, MySQL differences)
- Not validated by JPA - SQL errors manifest at runtime
- Required for database-specific features: JSON queries, full-text search,
  window functions, CTEs, LATERAL joins

Use JPQL for standard queries. Use native SQL for:
- Complex reporting requiring database-specific features
- Queries where JPA's abstraction adds overhead or complexity
- Performance-critical queries that need exact SQL control

*What separates good from great:* Native queries with DTO projections are the
most performant read path: exact SQL, exact columns selected, no entity
overhead. For high-throughput reporting (millions of rows), native SQL +
DTO is preferable to JPQL + entity loading.

---

**[MID] Q4 - [BEHAVIORAL] What is a DTO projection in Spring Data JPA?**

A DTO projection is an interface (or class) with getters matching the fields
selected in the query. Spring Data JPA maps query results to the projection
interface automatically - no entity is instantiated.

Benefits:
1. Only requested columns loaded from database (smaller result set)
2. No Hibernate entity tracking overhead (no first-level cache entry)
3. No lazy loading surprises (projections are flat, no associations)
4. Can include computed fields from the query

Interface projection:
```java
public interface OrderSummary {
    Long getId();
    String getCustomerEmail();
    BigDecimal getTotal();
}
List<OrderSummary> findByStatus(OrderStatus s);
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

Class projection (DTO constructor):
```java
public record OrderSummary(Long id,
                           String customerEmail,
                           BigDecimal total) {}

@Query("SELECT new com.example.OrderSummary(o.id, " +
       "o.customerEmail, o.total) " +
       "FROM Order o WHERE o.status = :status")
List<OrderSummary> findSummariesByStatus(
    @Param("status") OrderStatus status);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using SQL. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Interface projections use JDK proxies
(one proxy per result row). For large result sets, class projections (DTO with
constructor) are slightly faster because they create plain objects instead
of proxies. For millions of rows, use Stream<T> return type to avoid loading
all rows into memory at once.

---

**[MID] Q5 - [CONCEPTUAL] How does pagination work in Spring Data JPA?**

Repository method returning Page<T>:
```java
Page<Order> findByStatus(OrderStatus status,
                         Pageable pageable);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Calling code:
```java
Pageable page = PageRequest.of(
    0,  // page number (0-based)
    20, // page size
    Sort.by("createdAt").descending()
);
Page<Order> result = orderRepository
    .findByStatus(PENDING, page);

result.getContent();      // List<Order> for this page
result.getTotalElements();// total rows (runs COUNT query)
result.getTotalPages();   // total pages
result.hasNext();         // more pages?
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Page<T> executes TWO queries: one for data, one for COUNT. For large tables,
COUNT(*) can be expensive. Use Slice<T> instead of Page<T> when you only
need hasNext() (infinite scroll) - it avoids the COUNT query.

*What separates good from great:* The COUNT query generated by Spring Data is
often not optimal. For complex queries with JOINs, the COUNT query repeats the
full JOIN which is expensive. Use a custom countQuery in @Query:
`@Query(value="...", countQuery="SELECT COUNT(o.id) FROM Order o WHERE ...")`

---

**[MID] Q6 - [CONCEPTUAL] What is @Modifying and when do you need it?**

@Modifying marks a @Query method as a write operation (UPDATE or DELETE).
Without it, Spring Data throws InvalidDataAccessApiUsageException for DML queries.

```java
@Modifying
@Transactional
@Query("UPDATE Order o SET o.status = :status " +
       "WHERE o.id = :id")
int updateStatus(@Param("id") Long id,
                 @Param("status") OrderStatus status);

@Modifying
@Transactional
@Query("DELETE FROM Order o " +
       "WHERE o.status = :status " +
       "AND o.createdAt < :cutoff")
int deleteOldOrdersByStatus(
    @Param("status") OrderStatus status,
    @Param("cutoff") LocalDateTime cutoff);
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

@Modifying(clearAutomatically = true): clears the first-level cache (persistence
context) after the update. Without this, the cache may contain stale data -
you updated via JPQL but the cached entity still has the old value.

*What separates good from great:* @Modifying bulk updates bypass Hibernate's
entity lifecycle callbacks (@PreUpdate, etc.) and the first-level cache. Use
clearAutomatically = true to ensure subsequent reads see updated data. For
entity validation and auditing, updating entities via save() is cleaner - bulk
@Modifying is for performance-critical mass updates.

---

**[SENIOR] Q7 - [CONCEPTUAL] How does @Transactional interact with Spring Data repositories?**

SimpleJpaRepository methods are @Transactional (read-only for queries,
read-write for save/delete). This means each call is its own transaction.

Problem: you need atomic multi-step operations:
```java
// BAD: two separate transactions
orderRepository.save(order); // TX1
paymentRepository.save(payment); // TX2
// If TX2 fails, TX1 is already committed - inconsistency!
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

Correct pattern: @Transactional on the service method:
```java
@Service
public class OrderService {
    @Transactional
    public void createOrderWithPayment(
            Order order, Payment payment) {
        orderRepository.save(order);    // in TX
        paymentRepository.save(payment);// in TX
        // If anything fails, both roll back
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

With @Transactional on the service: Spring starts a transaction before the
method, repository calls join the existing transaction
(default propagation = REQUIRED), transaction commits or rolls back when
the service method returns.

*What separates good from great:* @Transactional only works on proxied method
calls. If OrderService.createOrderWithPayment() calls another @Transactional
method in the SAME class directly (this.otherMethod()), it bypasses the proxy
and the second @Transactional is ignored. This is the self-invocation problem.

---

**[SENIOR] Q8 - [ARCHITECTURE] What is the Specification pattern and when do you use it?**

The Specification pattern (Criteria API wrapper) enables building dynamic
predicates at runtime. Use it when query conditions are optional at runtime
(search forms with many optional filters).

```java
public interface OrderRepository extends
        JpaRepository<Order, Long>,
        JpaSpecificationExecutor<Order> {}

// Dynamic specifications
where(hasEmail(req.getEmail()))
    .and(hasStatus(req.getStatus()))
    .and(afterDate(req.getAfterDate()))
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

Pros: type-safe, composable, handles nulls via null predicate convention
Cons: verbose code compared to QueryDSL; complex queries can be hard to read

Alternatives:
- QueryDSL: generates metamodel from entities, type-safe predicate DSL
- @Query with JPQL: when conditions are known at compile time
- jOOQ: for complex SQL, generates type-safe DSL from database schema

*What separates good from great:* Specifications have an N+1 problem when
combining with collections (items in an order). The specification joins trigger
multiple rows per order (one per item), and the result set has duplicates.
Fix with query.distinct(true) in the specification.

---

**[SENIOR] Q9 - [HANDS-ON] What are custom repository implementations?**

For operations that don't fit derived queries, @Query, or Specification, you
can add custom implementation to a repository:

```java
// Custom interface
public interface OrderRepositoryCustom {
    List<Order> findWithComplexQuery(
        OrderSearchParams params);
}

// Implementation
public class OrderRepositoryImpl
        implements OrderRepositoryCustom {
    @PersistenceContext
    private EntityManager em;

    @Override
    public List<Order> findWithComplexQuery(
            OrderSearchParams params) {
        // Direct EntityManager - full JPA power
        CriteriaBuilder cb = em.getCriteriaBuilder();
        // ... complex criteria query
    }
}

// Repository interface combines both
public interface OrderRepository extends
        JpaRepository<Order, Long>,
        OrderRepositoryCustom {}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

Spring Data detects the Impl suffix and wires them together. The proxy
delegates to OrderRepositoryImpl for the custom method.

*What separates good from great:* The Impl class can also use JdbcTemplate
for native SQL operations that are too complex for JPA. Injecting JdbcTemplate
into a custom repository implementation gives you full access to native SQL
within the clean repository abstraction.

---

# @ConfigurationProperties


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is @ConfigurationProperties and how does it differ from @Value?**

@ConfigurationProperties binds a group of properties with a common prefix to
a POJO. @Value injects individual property values.

Key differences:

| | @ConfigurationProperties | @Value |
|---|---|---|
| Scope | Group of related properties | Single property |
| Type safety | Full type conversion | String by default |
| Validation | @Validated + JSR-303 | None (runtime NPE) |
| IDE support | Full autocomplete | Limited |
| SpEL support | No | Yes (#{...}) |
| Immutability | Constructor binding | No |

When to use @Value: single property, SpEL expressions, conditional expressions
with defaults `${prop:default}`.

When to use @ConfigurationProperties: everything else.

*What separates good from great:* @ConfigurationProperties classes are plain
beans with no Spring dependencies. They can be instantiated and tested in
isolation: `new PaymentProperties(url, timeout, retries, key)`. This makes
configuration testing clean.

---

**[JUNIOR] Q2 - [CONCEPTUAL] How does relaxed binding work?**

Spring Boot supports multiple naming conventions for the same property:

```
app.payment.api-key=secret        # kebab-case (recommended)
app.payment.apiKey=secret         # camelCase
app.payment.api_key=secret        # underscore
APP_PAYMENT_API_KEY=secret        # env var (UPPER_SNAKE)
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

All of these bind to the Java field `private String apiKey`.

Rules:
- Recommended in properties files: kebab-case
- Recommended in environment variables: UPPER_SNAKE_CASE
- Both work regardless of Java field name case

Relaxed binding applies to @ConfigurationProperties only, NOT @Value.
`@Value("${app.payment.api-key}")` uses the exact key - no relaxed binding.

*What separates good from great:* Environment variables use underscore as
separator because dots are reserved in many shell environments. SPRING_DATASOURCE_URL
in an environment variable maps to spring.datasource.url in properties.
This is how Kubernetes ConfigMap and Secret environment variables work
with Spring Boot configuration.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How do you validate @ConfigurationProperties?**

Add @Validated to the class and JSR-303 annotations to fields:

```java
@ConfigurationProperties("app")
@Validated
public class AppProperties {
    @NotEmpty
    @URL
    private String baseUrl;

    @NotNull
    @Positive
    private Integer maxConnections;

    @Valid   // cascade validation to nested
    private NestedConfig nested = new NestedConfig();
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

If validation fails at startup: BindException wraps the ConstraintViolations.
Spring Boot prints each failed constraint with the property name and invalid value.

*What separates good from great:* @Valid cascades validation to nested
configuration classes. Without @Valid, constraints on the nested class fields
are not evaluated. This is a common oversight when using nested configuration.

---

**[MID] Q4 - [CONCEPTUAL] What is constructor binding and how is it different from setter binding?**

**Setter binding** (default):
- Spring creates the object using the default constructor
- Populates fields via setter methods
- Fields can be mutable (not final)

**Constructor binding** (`@ConstructorBinding` or Java record):
- Spring uses a constructor with all configuration fields as parameters
- Fields can be final (immutable configuration)
- Required for records
- Available since Spring Boot 2.2

```java
// Constructor binding - immutable
@ConfigurationProperties("server")
@ConstructorBinding  // required in Boot 2.x (not in 3.x)
public class ServerProperties {
    private final String host;
    private final int port;

    public ServerProperties(String host, int port) {
        this.host = host;
        this.port = port;
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

In Spring Boot 3.x, constructor binding is detected automatically for classes
with a single constructor that has all properties - @ConstructorBinding is not
needed.

*What separates good from great:* Immutable configuration is inherently thread-safe
and easier to reason about. When configuration is final, you know it cannot
change after construction. Prefer constructor binding (or records) for all new
@ConfigurationProperties classes.

---

**[MID] Q5 - [CONCEPTUAL] How does @ConfigurationProperties support type conversion?**

Spring Boot includes converters for many types:

- Duration: "5s", "100ms", "1h30m" -> Duration
- DataSize: "10MB", "500KB" -> DataSize
- Period: "1y", "2m", "10d" -> Period
- File and Path: absolute or relative paths
- InetAddress: "192.168.1.1" or hostname

These are registered automatically via ApplicationConversionService.

Custom converter:
```java
@Bean
@ConfigurationPropertiesBinding
public Converter<String, MyType> myTypeConverter() {
    return source -> MyType.parse(source);
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

@ConfigurationPropertiesBinding marks the converter as intended for
@ConfigurationProperties binding (not MVC type conversion).

*What separates good from great:* DataSize is useful for configuring buffer
sizes, heap limits, etc.: `server.max-http-header-size=8KB`. Without this
type, you'd need to parse the string manually or use bytes. Type-safe
configuration values communicate intent and prevent units confusion
(milliseconds vs seconds).

---

**[MID] Q6 - [CONCEPTUAL] How do you use @ConfigurationProperties for multi-profile configuration?**

application.yml with profiles:
```yaml
app:
  payment:
    url: https://sandbox.pay.com
    timeout: 10s  # generous in dev

---
spring:
  config:
    activate:
      on-profile: production
app:
  payment:
    url: https://pay.example.com
    timeout: 5s  # tighter in production
```

> **Code walkthrough:** This Unknown example demonstrates YAML configuration pattern. **KEY MECHANISM:** YAML parsers are whitespace-sensitive; indentation errors cause silent value misinterpretation. **WHY IT MATTERS:** unquoted strings starting with special chars (*, &, ?, |) trigger YAML parser errors. **TAKEAWAY: quote strings containing YAML special chars; validate YAML before deploying to production.**

The @ConfigurationProperties class is the same - Spring loads different property
values based on active profile. No Java code changes needed.

For programmatic profile selection:
```java
@Component
@Profile("production")
public class ProductionPaymentProperties
        extends PaymentProperties { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* application.yml's multi-document format
(---) allows profile-specific sections in one file. For complex configuration,
application-{profile}.properties files are cleaner: application-production.properties
overrides only the production-specific values.

---

**[SENIOR] Q7 - [CONCEPTUAL] How do you test @ConfigurationProperties?**

Option 1 - @SpringBootTest (integration):
```java
@SpringBootTest
@TestPropertySource(properties = {
    "app.payment.url=https://test.example.com",
    "app.payment.api-key=test-key"
})
class PaymentPropertiesTest {
    @Autowired
    PaymentProperties props;
    // Full Spring context - slow but realistic
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Option 2 - @ConfigurationPropertiesTest (slice test):
```java
@ExtendWith(SpringExtension.class)
@EnableConfigurationProperties(PaymentProperties.class)
@TestPropertySource(properties = {
    "app.payment.url=https://test.example.com",
    "app.payment.api-key=test-key"
})
class PaymentPropertiesTest {
    @Autowired
    PaymentProperties props;
    // Minimal context - only binds properties
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Option 3 - Direct constructor (unit test):
```java
// If using constructor binding or records
PaymentProperties props = new PaymentProperties(
    "https://test.example.com",
    Duration.ofSeconds(5),
    3,
    "test-key"
);
// No Spring needed at all
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Option 3 (direct construction) is the fastest
and most appropriate for unit testing your configuration class's logic
(compact constructor defaults, validation). Use @SpringBootTest for integration
tests that verify the full property binding chain.

---

**[SENIOR] Q8 - [CONCEPTUAL] What is the difference between application.properties and application.yml?**

Both serve the same purpose - externalized configuration. Differences:

**application.properties:**
- Key=value format: `app.payment.url=https://pay.com`
- Flat - nested properties expressed with dots
- Better for simple configurations
- No data structure features

**application.yml:**
- YAML format with indentation for nesting
- Natural for deeply nested configuration
- Multi-document format for profiles (---)
- Lists and maps more readable
- Watch out: YAML is indentation-sensitive (TabError if tabs instead of spaces)

Choose .yml for: complex nested configuration, multiple profiles in one file.
Choose .properties for: simple flat configuration, environments where YAML
parsing issues arise.

*What separates good from great:* When both application.properties and
application.yml exist, Spring Boot loads both. Properties file values override
YAML values with the same key. This is useful for defaults in YAML and
environment-specific overrides in properties.

---

**[SENIOR] Q9 - [CONCEPTUAL] How do you access properties programmatically (without binding)?**

Option 1 - Environment.getProperty():
```java
@Service
public class DynamicConfigService {
    private final Environment env;

    public String getConfig(String key) {
        return env.getProperty(key, "default");
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Option 2 - @PropertySource + @Value:
```java
@Configuration
@PropertySource("classpath:custom.properties")
public class CustomConfig {
    @Value("${custom.prop}")
    private String customProp;
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Option 3 - ApplicationContext.getEnvironment():
```java
String value = applicationContext
    .getEnvironment()
    .getProperty("my.prop");
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Environment.getProperty() is useful for: dynamic property lookup based on
runtime values, checking if a property exists without failing, accessing
properties in components that cannot use @ConfigurationProperties.

*What separates good from great:* Environment.getProperty() always returns
the latest value from the property sources. Unlike @Value fields (set at
injection time), Environment.getProperty() reflects changes if property
sources are refreshed (Spring Cloud Config Server scenarios).

---

# @Autowired and Injection Types


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What are the three types of dependency injection in Spring?**

1. **Constructor injection**: dependencies provided via constructor parameters.
   Spring calls the constructor. Recommended for all required dependencies.

2. **Setter injection**: dependencies provided via setter methods annotated
   with @Autowired. For optional dependencies that can be changed post-construction.

3. **Field injection**: dependencies injected directly into fields via reflection.
   @Autowired on a field. Convenient but discouraged for new code.

*What separates good from great:* Spring also supports method injection
(@Lookup) for prototype beans, and @ConfigurationProperties which binds
properties to POJOs. These are specialised injection forms.

---

**[JUNIOR] Q2 - [TRADE-OFF] Why is constructor injection preferred over field injection?**

Constructor injection advantages:
1. **Immutability**: fields can be final - set once, never changed.
2. **Explicitness**: all dependencies visible in constructor signature.
3. **Testability**: `new Service(mockDep1, mockDep2)` - no Spring needed.
4. **Null safety**: constructors can validate and throw if dep is null.
5. **Circular detection**: circular constructor deps fail at startup.
6. **No reflection**: Spring calls the constructor normally.

Field injection disadvantages:
1. Fields cannot be final.
2. Object is in partially constructed state between constructor and injection.
3. Testing requires Spring context or reflection.
4. Circular dependencies silently allowed.

*What separates good from great:* The "partially constructed state" issue is
subtle. Between the constructor returning and Spring finishing field injection,
the object exists with null fields. Constructor injection eliminates this window.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How does @Qualifier work and when do you use it?**

@Qualifier specifies which bean to inject when multiple beans of the same
type are available.

```java
@Bean("primary")
@Primary
public DataSource primaryDataSource() { /* ... */ }

@Bean("replica")
public DataSource replicaDataSource() { /* ... */ }

// Gets replicaDataSource specifically
@Service
public class ReportService {
    public ReportService(
            @Qualifier("replica") DataSource ds) {
        /* ... */
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

@Primary marks the default: if no @Qualifier is specified, the @Primary bean
is chosen. @Qualifier overrides @Primary.

Custom qualifier annotations: create your own annotation meta-annotated with
@Qualifier for type-safe qualification without string names.

*What separates good from great:* Creating custom qualifier annotations is
cleaner than string qualifiers because they are refactor-safe.

---

**[MID] Q4 - [CONCEPTUAL] How do you handle optional dependencies in Spring?**

Three approaches:

1. **@Autowired(required = false) on setter**:
   ```java
   @Autowired(required = false)
   public void setEmailService(EmailService s) {
       this.emailService = s;
   }
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

2. **Optional<T> constructor parameter**:
   ```java
   public MyService(Optional<EmailService> email) {
       this.emailService = email.orElse(null);
   }
   ```

> **Code walkthrough:** This Unknown example demonstrates null-safe value wrapping. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

3. **@Autowired(required = false) on field** (avoid in new code):
   ```java
   @Autowired(required = false)
   private EmailService emailService; // null if no bean
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Best approach for new code: constructor with Optional<T> parameter.
It makes the optionality explicit in the class contract.

*What separates good from great:* Optional<T> in constructor makes the
optionality explicit - a developer reading the constructor immediately knows
EmailService is optional.

---

**[MID] Q5 - [CONCEPTUAL] What is @Primary and when do you use it?**

@Primary marks a bean as the default candidate when multiple beans of the
same type exist and no @Qualifier is specified.

Use case: primary DataSource and read-replica DataSource. Most code uses
primary. Annotate primary bean with @Primary. Code needing replica qualifies
with @Qualifier("replica").

@Primary has implications for auto-configuration: defining a @Primary DataSource
causes DataSourceAutoConfiguration to skip (ConditionalOnMissingBean). This
is the correct way to override auto-configured beans.

*What separates good from great:* @Primary is the override mechanism for
auto-configuration beans without explicit exclusion.

---

**[MID] Q6 - [CONCEPTUAL] How does Spring resolve injection when multiple beans exist?**

Resolution order:
1. **Type match**: find all beans of the required type.
2. **@Primary**: if exactly one @Primary bean, use it.
3. **@Qualifier**: if injection point has @Qualifier, use matching name.
4. **Name match**: if field/parameter name matches a bean name, use it.
5. **NoUniqueBeanDefinitionException**: if none resolve uniquely.

The name-match fallback is fragile. Changing a bean name can break injection
points relying on name matching. Always use @Qualifier explicitly when choosing
from multiple candidates.

*What separates good from great:* Implicit name matching is a fallback, not
a design strategy. Use @Qualifier for self-documenting, refactor-safe injection.

---

**[SENIOR] Q7 - [CONCEPTUAL] How do you break a circular dependency in Spring?**

A circular dependency is usually a design problem. Correct fix: redesign.

If needed:
1. **Extract shared functionality** into a third service C (best solution).
2. **@Lazy on one injection point**:
   ```java
   @Service
   public class ServiceA {
       public ServiceA(@Lazy ServiceB b) { /* ... */ }
   }
   ```
> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Spring creates a proxy for ServiceB, breaking the constructor cycle.

3. **Setter/field injection on one side**: Spring uses early bean references.
   Not recommended - the design issue remains.

Spring Boot 2.6+: circular dependencies disallowed by default
(spring.main.allow-circular-references=false). Fix the design.

*What separates good from great:* @Lazy is legitimate when the circular
dependency is inherent to the domain model and refactoring is not feasible.

---

**[SENIOR] Q8 - [CONCEPTUAL] What is the difference between @Autowired, @Resource, and @Inject?**

**@Autowired (Spring):**
- Resolves by type first, then name
- Supports required=false
- Spring-specific

**@Resource (JSR-250 / Jakarta):**
- Resolves by name first, then type
- name attribute specifies explicit name
- Standard Java annotation

**@Inject (JSR-330 / Jakarta):**
- Resolves by type (same as @Autowired)
- No required equivalent - always required
- Standard Java annotation

Recommendation: use @Autowired for consistency in Spring applications.
Use @Inject/@Resource if minimizing Spring coupling is a goal.

*What separates good from great:* JSR-330 annotations allow classes to be
portable between Spring and Jakarta EE/CDI containers.

---

**[SENIOR] Q9 - [CONCEPTUAL] How does constructor injection work with Lombok?**

@RequiredArgsConstructor generates a constructor for all final fields:

```java
@Service
@RequiredArgsConstructor  // generates constructor
public class OrderService {
    private final OrderRepository repo;
    private final PaymentService payment;
    // Lombok generates:
    // public OrderService(OrderRepository repo,
    //                     PaymentService payment) { ... }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Spring 4.3+ finds the single generated constructor and uses it without
@Autowired. @RequiredArgsConstructor + @NonNull adds null checks in the
generated constructor.

*What separates good from great:* @RequiredArgsConstructor with @NonNull on
fields adds null safety without the boilerplate of manual Objects.requireNonNull.

---

# Spring MVC Request Lifecycle


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is DispatcherServlet and what is its role?**

DispatcherServlet is a standard Java Servlet registered with the embedded
servlet container. Every HTTP request to a Spring MVC application passes through
it. This is the Front Controller pattern.

Responsibilities:
1. Receive all HTTP requests
2. Delegate to HandlerMapping to find the handler
3. Execute HandlerInterceptors (preHandle)
4. Delegate to HandlerAdapter to invoke the handler
5. Execute HandlerInterceptors (postHandle)
6. Resolve the response (view or message converter)
7. Execute HandlerInterceptors (afterCompletion)

Spring Boot auto-configures DispatcherServlet via
DispatcherServletAutoConfiguration. Mapped to "/" by default.

*What separates good from great:* DispatcherServlet is itself a Spring bean,
configured in the WebApplicationContext. It reads HandlerMapping, HandlerAdapter,
ViewResolver, and other MVC beans from the context. You customize its behavior
by defining these beans.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is HandlerMapping and what does it do?**

HandlerMapping maps incoming requests (URL + HTTP method) to a handler.
In @RequestMapping-based Spring MVC, the handler is a HandlerMethod (a
reference to the specific @Controller method).

Key implementations:
- RequestMappingHandlerMapping: maps @RequestMapping, @GetMapping, etc.
- SimpleUrlHandlerMapping: maps URLs to handler beans by name
- RouterFunctionMapping: maps functional route definitions

HandlerMapping returns a HandlerExecutionChain: the handler + any
HandlerInterceptors registered for the matched path pattern.

*What separates good from great:* Multiple HandlerMappings can coexist.
DispatcherServlet iterates them by @Order. The first one that returns a
non-null HandlerExecutionChain wins. This is how @RequestMapping controllers
and router functions can coexist.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is the difference between HandlerInterceptor and Servlet Filter?**

**Servlet Filter:**
- Part of the Servlet specification
- Runs BEFORE DispatcherServlet
- Applies to all requests (static resources, error dispatches, everything)
- Raw HttpServletRequest/Response access only
- Spring Security runs here

**HandlerInterceptor:**
- Spring MVC-specific
- Runs INSIDE DispatcherServlet, AFTER routing
- Only applies to requests handled by DispatcherServlet
- Access to handler method and ModelAndView

Use filter for: headers for all responses, authentication infrastructure.
Use interceptor for: controller-aware logic, audit logging of endpoints called.

*What separates good from great:* Spring Security's DelegatingFilterProxy is
a plain Filter that delegates to a Spring-managed Filter bean. This bridges
Servlet Filter and Spring context - security runs in the filter chain while
being a Spring bean with full DI support.

---

**[MID] Q4 - [HANDS-ON] How do you implement centralized exception handling?**

**@ControllerAdvice + @ExceptionHandler** (preferred):
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handle(
            EntityNotFoundException ex) {
        return new ErrorResponse(ex.getMessage());
    }
}
```
> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Catches exceptions from any @Controller in the application.

Selection order: controller-level @ExceptionHandler first, then
@ControllerAdvice global handlers. Most specific exception type wins.

*What separates good from great:* ResponseEntityExceptionHandler is a
convenience base class for @ControllerAdvice that handles all Spring MVC
standard exceptions with RFC 7807 Problem Detail responses.

---

**[MID] Q5 - [CONCEPTUAL] What is HandlerMethodArgumentResolver?**

Resolves @Controller method parameters from the HTTP request. Each @RequestParam,
@PathVariable, @RequestBody has a corresponding resolver.

Custom resolver - bind authenticated user:
```java
@Component
public class CurrentUserResolver
        implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(
            MethodParameter parameter) {
        return parameter.hasParameterAnnotation(
            CurrentUser.class);
    }

    @Override
    public Object resolveArgument(
            MethodParameter parameter,
            ModelAndViewContainer mav,
            NativeWebRequest req,
            WebDataBinderFactory binder) {
        Authentication auth = SecurityContextHolder
            .getContext().getAuthentication();
        return auth.getPrincipal();
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates metadata declaration using Spring annotation. **KEY MECHANISM:** annotations are processed at compile-time or runtime via reflection. **WHY IT MATTERS:** annotation processing adds compile time; runtime reflection disables JIT optimizations. **TAKEAWAY: prefer compile-time annotation processors (APT) over runtime reflection for performance.**

*What separates good from great:* Spring Security's @AuthenticationPrincipal
is implemented this way. Understanding this mechanism means you can implement
the same pattern for tenant ID from header, pagination from query params, etc.

---

**[MID] Q6 - [CONCEPTUAL] What is the difference between @Controller and @RestController?**

@RestController = @Controller + @ResponseBody on the class.

@Controller:
- Each method must explicitly have @ResponseBody to return JSON/XML
- Or return a view name resolved by ViewResolver

@RestController:
- @ResponseBody applied to ALL methods automatically
- Method return values always serialized to response body
- Cannot return view names (they would be serialized as strings)

Choose @Controller when mixing REST and view rendering.
Choose @RestController for pure REST API controllers.

*What separates good from great:* A common mistake is using @RestController and
returning a String (page name) expecting ViewResolver to render a template.
The String is serialized as a JSON string instead.

---

**[SENIOR] Q7 - [CONCEPTUAL] How does @RequestBody parsing work?**

@RequestBody triggers HttpMessageConverter to deserialize the request body.

Process:
1. Spring inspects Content-Type header.
2. Finds HttpMessageConverter (e.g., MappingJackson2HttpMessageConverter
   for application/json).
3. Converter reads and deserializes the body.
4. If @Valid is present, JSR-303 validation runs.

Failure: HttpMessageNotReadableException thrown -> 400 Bad Request.

Customizing Jackson: define an ObjectMapper bean. Spring Boot auto-config
picks it up via JacksonAutoConfiguration.

*What separates good from great:* @RequestBody reads the input stream once.
The stream is not resettable. If you need to read the body multiple times
(interceptor + controller), use ContentCachingRequestWrapper.

---

**[SENIOR] Q8 - [CONCEPTUAL] How do you add custom headers to all responses?**

Option 1 - Servlet Filter (for ALL requests):
```java
@Component
public class SecurityHeadersFilter
        extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(
            HttpServletRequest req,
            HttpServletResponse resp,
            FilterChain chain) throws Exception {
        resp.setHeader("X-Content-Type-Options",
            "nosniff");
        resp.setHeader("X-Frame-Options", "DENY");
        chain.doFilter(req, resp);
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Option 2 - HandlerInterceptor (MVC requests only):
```java
@Override
public void afterCompletion(...) {
    resp.setHeader("X-Request-Id",
        MDC.get("requestId"));
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Security headers should use Servlet Filter to apply to all responses
including error pages and static content.

*What separates good from great:* Spring Security's HeaderWriterFilter adds
security headers automatically (X-Content-Type-Options, X-Frame-Options,
HSTS). Configure via HttpSecurity.headers() rather than writing your own filter.

---

**[SENIOR] Q9 - [SYSTEM DESIGN] How do you handle multipart file uploads?**

```java
@RestController
@RequestMapping("/uploads")
public class FileUploadController {
    @PostMapping(
        consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public UploadResponse upload(
            @RequestParam("file") MultipartFile file,
            @RequestParam("description") String desc) {
        if (file.isEmpty()) {
            throw new IllegalArgumentException(
                "File cannot be empty");
        }
        // Security: validate content type
        if (!isAllowedType(file.getContentType())) {
            throw new IllegalArgumentException(
                "File type not allowed");
        }
        String filename = storage.store(file);
        return new UploadResponse(filename, desc);
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Configuration:
```
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* getContentType() returns the MIME type from
HTTP headers - user-controlled and not trustworthy for security decisions.
Use Apache Tika to detect the actual content type from file bytes. Using
Content-Type header for security validation is an OWASP-listed vulnerability
(Unrestricted File Upload). Also: store files outside the webroot, use UUID
filenames (not the original - path traversal prevention).

---

# Spring AOP and Proxies


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] How does Spring AOP work?**

Spring AOP works by creating proxy objects around managed beans. When a Spring
bean has methods with AOP-applicable annotations (@Transactional, @Cacheable,
@Async), Spring creates a proxy class that:
1. Extends the target class (CGLIB) or implements its interfaces (JDK proxy)
2. Overrides public methods to insert interceptor chain
3. Registers the proxy (not the real object) in the ApplicationContext

At call time: caller calls proxy method -> proxy runs before advice ->
proxy calls target method via super (CGLIB) or delegation (JDK) ->
proxy runs after/around advice.

*What separates good from great:* AOP proxy creation happens in the
BeanPostProcessor post-initialization phase (AbstractAutoProxyCreator).
This is step 8 in the bean lifecycle - AFTER @PostConstruct. Any call
from @PostConstruct goes to the real object, not the proxy.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the self-invocation problem and how do you fix it?**

Self-invocation: calling a proxied method from within the same class via `this`.
`this` refers to the real object, not the proxy. The proxy never intercepts
the call. Any advice (@Transactional, @Cacheable, etc.) on the called method
is silently ignored.

Example:
```java
@Transactional
public void outer() {
    this.inner(); // inner() @Transactional NOT applied
}

@Transactional(propagation = REQUIRES_NEW)
public void inner() { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

Fixes (best to worst):
1. Extract inner() to a separate @Service - cleanest design
2. @Autowired @Lazy private MyService self; - self-injection hack
3. AopContext.currentProxy() with @EnableAspectJAutoProxy(exposeProxy=true)
4. AspectJ LTW - works for self-invocation but requires Java agent

*What separates good from great:* The need to work around self-invocation
usually indicates a design issue. If you need separate transactions for
multiple operations in one class, a separate service boundary is the right
structural answer.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is the difference between JDK proxy and CGLIB proxy?**

**JDK dynamic proxy:**
- Requires the target class to implement at least one interface
- The proxy implements the same interfaces as the target
- Uses java.lang.reflect.Proxy
- Cannot proxy methods not in an interface
- Spring default before Spring Boot 2.0

**CGLIB proxy:**
- Subclasses the target class (no interface required)
- Overrides public and protected methods
- Cannot proxy final classes or final/static/private methods
- Spring Boot 2.0+ default (proxy-target-class=true)
- Slightly higher startup cost (bytecode generation)

Spring Boot 2.0 changed the default to CGLIB to solve a common injection bug:
@Autowired MyService service (where MyService is a class, not interface) failed
with JDK proxy because the proxy only implements the interface, not MyService.

*What separates good from great:* Spring Boot 3 improved CGLIB with "class-data
sharing" - CGLIB generated classes are reused across tests, speeding up test
context startup. For GraalVM native images, CGLIB proxies are pre-generated
at build time by the AOT engine.

---

**[MID] Q4 - [CONCEPTUAL] What AOP advice types does Spring support?**

Spring AOP supports five advice types matching the standard AOP vocabulary:

- **@Before**: runs before the method. Cannot veto execution.
  ```java
  @Before("execution(* com.example..*.*(..))")
  public void logBefore(JoinPoint jp) { ... }
  ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

- **@After**: runs after the method (always - both success and exception).
  Like a finally block.

- **@AfterReturning**: runs after successful return. Has access to return value.

- **@AfterThrowing**: runs after an exception is thrown. Has access to exception.

- **@Around**: wraps the entire execution. Must call pjp.proceed() to invoke
  the method. Most powerful - can modify arguments, return value, or suppress
  exceptions.

*What separates good from great:* @Around should be used only when you need
to control execution (conditional proceed, modify return value, handle exceptions).
For simple before/after logging, @Before and @After are cleaner because they
have narrower responsibility.

---

**[MID] Q5 - [HANDS-ON] How do you write a custom aspect?**

```java
// 1. Custom annotation as joinpoint marker
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AuditLog {
    String action();
}

// 2. Aspect class with @Around advice
@Aspect
@Component
public class AuditAspect {

    @Around("@annotation(auditLog)")
    public Object audit(
            ProceedingJoinPoint pjp,
            AuditLog auditLog) throws Throwable {

        String user = SecurityContextHolder.getContext()
            .getAuthentication().getName();
        String action = auditLog.action();

        try {
            Object result = pjp.proceed();
            auditService.log(user, action, "SUCCESS");
            return result;
        } catch (Exception e) {
            auditService.log(user, action, "FAILURE");
            throw e;
        }
    }
}

// 3. Usage
@Service
public class OrderService {
    @AuditLog(action = "CREATE_ORDER")
    public Order createOrder(OrderRequest req) { ... }
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using Spring annotation. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The aspect method signature includes the
AuditLog parameter - Spring binds the annotation instance to it, giving you
access to annotation attributes. Without this, you'd need pjp.getMethod()
.getAnnotation(AuditLog.class) to access the annotation.

---

**[MID] Q6 - [HANDS-ON] What is a pointcut expression and how do you write one?**

A pointcut expression defines which joinpoints (method executions) an advice
applies to. Common expressions:

**execution()** - most common:
```java
// All public methods in all classes:
execution(public * *(..))

// All methods in a specific package:
execution(* com.example.service.*.*(..))

// Specific method signature:
execution(* com.example.OrderService.create*(..))
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**@annotation()** - match by annotation:
```java
// Any method with @Transactional
@annotation(org.springframework.transaction
    .annotation.Transactional)
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

**within()** - match by type:
```java
// All methods in a class hierarchy
within(com.example.service..*)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Combining with &&, ||, !**:
```java
// Public methods in service package with @Transactional
execution(public * com.example.service..*.*(..)) &&
@annotation(Transactional)
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

*What separates good from great:* Overly broad pointcut expressions (execution(* *(..)))
have significant performance impact - they intercept every method call.
Always scope pointcuts as narrowly as possible. Profile AOP overhead if you
have broad pointcuts in high-throughput code.

---

**[SENIOR] Q7 - [CONCEPTUAL] How does @Async work with Spring AOP?**

@Async submits the annotated method's execution to a thread pool instead of
running it on the calling thread. Spring AOP creates a proxy that:
1. Intercepts the method call
2. Wraps the method in a Runnable/Callable
3. Submits to the configured AsyncTaskExecutor
4. Returns immediately (void or CompletableFuture to caller)

```java
@Service
public class EmailService {
    @Async  // runs in thread pool
    public CompletableFuture<Boolean> sendEmail(
            String to, String content) {
        // executed in separate thread
        boolean success = smtpClient.send(to, content);
        return CompletableFuture.completedFuture(success);
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates async pipeline construction using CompletableFuture. **KEY MECHANISM:** the JVM schedules continuations via ForkJoinPool when each stage completes. **WHY IT MATTERS:** callback chains execute on wrong threads causing ClassCastException in Spring context. **TAKEAWAY: always specify executor on thenApplyAsync to control thread context.**

Requirements:
- @EnableAsync on a @Configuration class
- The annotated method must return void or CompletableFuture
- Self-invocation limitation applies (same as @Transactional)

*What separates good from great:* Without a custom TaskExecutor, Spring uses
SimpleAsyncTaskExecutor which creates a new thread per call - no thread pool.
Always configure a bounded ThreadPoolTaskExecutor for @Async in production.
Also: exceptions in @Async void methods are swallowed unless you configure
an AsyncUncaughtExceptionHandler.

---

**[SENIOR] Q8 - [CONCEPTUAL] What is the difference between Spring AOP and AspectJ?**

**Spring AOP:**
- Proxy-based (CGLIB or JDK proxy)
- Intercepts only Spring bean method executions
- Does not work for: self-invocation, constructor calls, field access,
  new object creation, static methods
- No agent or compiler required
- Sufficient for 95% of use cases

**AspectJ:**
- Bytecode weaving (compile-time or load-time)
- Works for ALL Java constructs: self-invocation, constructors, fields, statics
- Requires AspectJ compiler (CTW) or Java agent (LTW)
- More powerful but more complex to configure
- Spring integrates with AspectJ via @EnableLoadTimeWeaving for LTW

Use Spring AOP for: @Transactional, @Async, @Cacheable, custom aspects on
Spring beans.
Use AspectJ for: self-invocation must work, weaving non-Spring objects,
aspect on new object creation.

*What separates good from great:* Spring's @Transactional, @Cacheable, @Async,
@Secured are ALL implemented as Spring AOP aspects. Understanding this means
understanding all their limitations (self-invocation, proxy requirements) from
first principles. You don't need to memorize which limitation applies to which
annotation.

---

**[SENIOR] Q9 - [CONCEPTUAL] How do you order multiple aspects on the same method?**

When multiple aspects apply to the same method, their order is controlled by:

1. **@Order annotation** on the @Aspect class:
   ```java
   @Aspect
   @Component
   @Order(1)  // lower = higher priority (runs first)
   public class SecurityAspect { ... }

   @Aspect
   @Component
   @Order(2)  // runs second
   public class AuditAspect { ... }
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

2. **Ordered interface**: implement org.springframework.core.Ordered.

Order applies as a stack:
- Higher priority aspects wrap lower priority aspects
- @Order(1) preHandle runs before @Order(2) preHandle
- @Order(1) postHandle runs after @Order(2) postHandle
- Think of it as nested try/finally blocks

Default order when not specified: undefined (any order).

*What separates good from great:* Spring Security's @PreAuthorize checks run
in a specific order relative to @Transactional. Security check should happen
OUTSIDE the transaction (unauthorized requests don't open a DB transaction).
Spring Security's MethodSecurityInterceptor has lower order number
(higher priority) than Spring's TransactionInterceptor to ensure this.

---

# @Transactional and Transaction Management


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What does @Transactional do?**

@Transactional demarcates a transaction boundary. Spring's TransactionInterceptor
(a BeanPostProcessor-created proxy) wraps the annotated method:

1. Before method: begin transaction (open connection, setAutoCommit(false))
2. Method executes (all DB operations in the same connection)
3. Normal return: commit
4. RuntimeException thrown: rollback, re-throw exception
5. Checked exception thrown: commit by default (can be changed with rollbackFor)
6. Connection returned to pool

This ensures atomicity: multiple repository saves in one service method are
one atomic database operation.

*What separates good from great:* @Transactional on a @Service method is the
correct place for transactional boundary. Repository methods have their own
@Transactional, but they each start/commit their own transaction. Without a
service-level transaction, two repository calls in one service method use two
separate transactions.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What are the transaction propagation modes?**

Key propagation values:

- **REQUIRED** (default): join existing transaction; create new if none.
  Most common - service methods join or create.

- **REQUIRES_NEW**: always create a new transaction; suspend existing.
  Use for: audit logging, notifications (must persist independently).

- **SUPPORTS**: join existing transaction if present; no transaction if not.
  Use for: read operations that work in or out of a transaction.

- **MANDATORY**: must have existing transaction; throws if none.
  Use for: methods that should never be called without a transaction context.

- **NOT_SUPPORTED**: execute without transaction; suspend existing.
  Use for: bulk operations where transaction overhead is not worth it.

- **NEVER**: must NOT have a transaction; throws if one exists.

- **NESTED**: create a savepoint in the existing transaction.
  Nested rollback only rolls back to savepoint, not entire transaction.
  Only supported by some databases and JDBC drivers.

*What separates good from great:* REQUIRES_NEW creates a separate database
connection. In high-throughput systems, too many REQUIRES_NEW calls can
exhaust the connection pool. NESTED uses savepoints on the same connection,
which is more efficient but has limited database support.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is the default rollback behavior and how do you change it?**

Default: rollback on RuntimeException (unchecked) and Error.
Default: NO rollback on checked Exception.

To change:
```java
// Roll back on ALL exceptions
@Transactional(rollbackFor = Exception.class)

// Roll back on specific checked exception
@Transactional(rollbackFor = MyCheckedException.class)

// Don't roll back on specific runtime exception
@Transactional(
    noRollbackFor = OptimisticLockingException.class)
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

Recommendation: always use rollbackFor = Exception.class if your service
methods throw or propagate checked exceptions.

*What separates good from great:* The "checked exceptions don't roll back"
default comes from J2EE history where checked exceptions indicated "expected
business errors" that were handled gracefully. Modern Spring convention uses
RuntimeException for domain errors and checked exceptions for external I/O.
The default works with this convention, but mixing checked exceptions with
domain errors requires explicit rollbackFor.

---

**[MID] Q4 - [CONCEPTUAL] What is transaction isolation and what are the levels?**

Isolation defines what concurrent transactions can see from each other's
uncommitted work.

Anomalies that isolation levels prevent:
- **Dirty read**: reading uncommitted data from another transaction
- **Non-repeatable read**: same row read twice returns different values
  (another transaction committed a change between reads)
- **Phantom read**: same query returns different rows (another transaction
  inserted/deleted rows)

Isolation levels:

| Level | Dirty Read | Non-Repeatable | Phantom |
|---|---|---|---|
| READ_UNCOMMITTED | Possible | Possible | Possible |
| READ_COMMITTED | Prevented | Possible | Possible |
| REPEATABLE_READ | Prevented | Prevented | Possible |
| SERIALIZABLE | Prevented | Prevented | Prevented |

Higher isolation = fewer anomalies = more contention = lower throughput.

Database defaults: PostgreSQL = READ_COMMITTED, MySQL InnoDB = REPEATABLE_READ.
Spring default = DEFAULT (use database default).

*What separates good from great:* PostgreSQL uses MVCC for REPEATABLE_READ
and SERIALIZABLE - readers don't block writers. This means READ_COMMITTED is
almost always appropriate. READ_UNCOMMITTED is rarely useful in production
(reads uncommitted data from other transactions - too risky).

---

**[MID] Q5 - [CONCEPTUAL] How does @Transactional(readOnly = true) affect behavior?**

readOnly = true sets a hint on the transaction:

1. **Hibernate flush mode**: set to FlushMode.NEVER or FlushMode.MANUAL.
   Hibernate skips dirty checking (no flush before queries). Dirty checking
   scans all loaded entities for changes - skipping it is a performance win
   for read-heavy methods with large entity graphs.

2. **Connection hint**: some JDBC drivers and connection pools (via
   ReadOnlyAwareDataSourceRouter) use the hint to route to read replicas.

3. **No optimization for non-Hibernate code**: JdbcTemplate ignores readOnly.

When to use: service methods that only read data, no writes. Makes the intent
explicit and gets a performance benefit from Hibernate.

*What separates good from great:* Read replica routing with @Transactional
(readOnly = true) is a common pattern for scaling read traffic. AbstractRoutingDataSource
or LazyConnectionDataSourceProxy inspect the readOnly flag and route to the
appropriate DataSource. This requires no code changes in the service layer - only
DataSource configuration changes.

---

**[MID] Q6 - [CONCEPTUAL] What is @TransactionalEventListener?**

@TransactionalEventListener publishes Spring application events AFTER the
current transaction commits. Contrast with @EventListener which fires immediately.

Use case: send email notification after order creation:

```java
// BAD: calling @Transactional method from same class
// Spring proxy is bypassed - no transaction started
public void processOrder(Order order) {
    saveOrder(order); // self-call bypasses proxy
}
@Transactional
public void saveOrder(Order order) { /* ... */ }
```

```java
// BAD: @EventListener fires BEFORE transaction commits
// Email sent even if order save fails and rolls back
@EventListener
public void onOrderCreated(OrderCreatedEvent event) {
    emailService.sendConfirmation(event.getOrder());
}

// GOOD: fires only after transaction commits
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT)
public void onOrderCreated(OrderCreatedEvent event) {
    emailService.sendConfirmation(event.getOrder());
    // Order is committed - safe to notify customer
}
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **WHAT BREAKS: never self-invoke @Transactional methods; inject the bean instead.**

TransactionPhase options:
- AFTER_COMMIT (default): fires if transaction committed
- AFTER_ROLLBACK: fires if transaction rolled back
- AFTER_COMPLETION: fires regardless
- BEFORE_COMMIT: fires before commit (within the transaction)

*What separates good from great:* @TransactionalEventListener AFTER_COMMIT
fires outside the transaction. If the email service throws, the exception does
not roll back the order (transaction already committed). Design your event
listeners to be idempotent and handle failures independently (retry queue).

---

**[SENIOR] Q7 - [CONCEPTUAL] How do you handle transactions programmatically?**

TransactionTemplate for programmatic control:
```java
@Service
public class OrderService {
    private final TransactionTemplate txTemplate;

    public OrderService(
            PlatformTransactionManager txManager) {
        this.txTemplate = new TransactionTemplate(
            txManager);
        txTemplate.setIsolationLevel(
            TransactionDefinition.ISOLATION_READ_COMMITTED);
    }

    public Order createOrder(OrderRequest req) {
        return txTemplate.execute(status -> {
            Order order = orderRepository.save(
                new Order(req));
            paymentRepository.save(
                new Payment(order));
            return order;
            // Automatically commits or rolls back
        });
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

When to use programmatic over declarative:
- Fine-grained control over transaction boundaries within a method
- Conditional transaction creation
- The transactional boundary is not at the method level

*What separates good from great:* TransactionTemplate.execute() returns the
result of the lambda. TransactionTemplate.executeWithoutResult() is for void
operations. The status parameter lets you manually mark the transaction for
rollback: status.setRollbackOnly() without throwing an exception.

---

**[SENIOR] Q8 - [CONCEPTUAL] What is optimistic vs pessimistic locking in JPA?**

**Optimistic locking** (@Version on entity field):
- No database lock held
- Entity has a @Version field (incrementing number or timestamp)
- On save: UPDATE ... WHERE id = ? AND version = ?
- If another transaction incremented version: UPDATE affects 0 rows
- JPA detects 0 rows updated: throws OptimisticLockingException
- Cost: exception on conflict; retry required
- Best for: low-contention resources, high-throughput systems

**Pessimistic locking** (@Lock annotation):
- Database row lock held for the transaction duration

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Account> findById(Long id);
```

> **Code walkthrough:** `@Lock(PESSIMISTIC_WRITE)` annotates aice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**
> Spring Data repository method to issue `SELECT ... FOR UPDATE`.
> The database row lock is held until the transaction commits,
> blocking concurrent writers entirely. This prevents phantom
> reads and lost updates but reduces throughput under contention.
> Use only for high-conflict resources like account balances where
> optimistic lock retries would loop indefinitely.

- SELECT ... FOR UPDATE: blocks other writes until lock released
- No conflicts possible, but lower throughput
- Best for: high-contention resources (account balances), guarantee-critical

Optimistic is default and preferred. Pessimistic for financial accounts,
inventory counts where conflicts would be costly.

*What separates good from great:* Optimistic locking requires retry logic in the
service layer. A failed optimistic lock should be retried, not surfaced to the
user as a 500 error. Spring Retry (@Retryable(OptimisticLockingFailureException.class))
adds automatic retry with configurable backoff.

---

**[SENIOR] Q9 - [CONCEPTUAL] How does @Transactional interact with Spring Data JPA?**

Spring Data JPA's SimpleJpaRepository:
- All write methods (save, delete) are @Transactional (read-write)
- All read methods (findAll, findById) are @Transactional(readOnly = true)

When you call orderRepository.save() from a non-transactional service:
- SimpleJpaRepository.save() opens its own transaction, commits, returns

When you call orderRepository.save() from a @Transactional service:
- SimpleJpaRepository.save() sees the existing transaction (REQUIRED propagation)
- Joins the existing transaction
- The commit happens when the service method returns

This is why @Transactional on the service method is essential for atomicity:
individual repository calls have their own transactions by default. The service
@Transactional wraps them all in one outer transaction.

*What separates good from great:* @Transactional(readOnly = true) on a service
method that calls multiple repository find methods is both a performance
optimization and a consistency guarantee. All reads in the method see a
consistent snapshot (no phantom reads within the read-only transaction scope).

---

# Spring Profiles


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is the default profile and when does it activate?**

The "default" profile activates when NO other profiles are active.
application-default.properties is loaded for this profile.

Use it to make the application safe to run without any profile configuration:
- default profile: H2 in-memory database, mock external services, DEBUG logging
- Developers can run the app without setting any env vars

If any profile IS active, the default profile is NOT active. This is important:
if you add config to application-default.properties and activate "local" profile,
default config no longer loads. Avoid putting critical defaults in application-default.properties
that you also need in other profiles.

*What separates good from great:* Spring Boot logs active profiles at startup:
"The following profiles are active: prod". This is your first debugging step
when profile configuration seems wrong. If it shows "No active profile set,
falling back to 1 default profile: default", no profiles were set.

---

**[JUNIOR] Q2 - [CONCEPTUAL] How does @Profile interact with @Conditional?**

@Profile is implemented as @Conditional(ProfileCondition.class). This means:
- @Profile and @Conditional can be combined
- @Conditional is more powerful (can check any condition: classpath, bean
  existence, property value, environment)

```java
// @Profile is syntactic sugar for:
@Conditional(ProfileCondition.class)
public @interface Profile { ... }

// More complex: active only in prod
// AND when property is set
@Bean
@Profile("prod")
@ConditionalOnProperty(
    name = "feature.cache.enabled",
    havingValue = "true")
public CacheService cacheService() { ... }
```

> **Code walkthrough:** This application-prod.properties (prod overrides) example demonstrates contract definition using Spring annotation. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

@ConditionalOnProperty is the most commonly combined annotation. This enables
feature flags that work across environments.

*What separates good from great:* Spring Security's SecurityAutoConfiguration
uses @Conditional extensively. Understanding @Conditional is key to
understanding how Spring Boot auto-configuration works. Every auto-configuration
class is a @Configuration with @Conditional guards.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How do you activate profiles in different environments?**

Priority order (highest wins, lower is overridden):

1. Command-line argument: `java -jar app.jar
   --spring.profiles.active=prod`

2. JVM system property: `java -Dspring.profiles.active=prod -jar app.jar`

3. Environment variable: `SPRING_PROFILES_ACTIVE=prod`
   (Spring Boot converts _ to . so this maps to spring.profiles.active)

4. In application.properties: `spring.profiles.active=dev`
   (lowest priority - gets overridden by env var)

5. Programmatic: `SpringApplication.setAdditionalProfiles("prod")`

Kubernetes deployment:
```yaml
env:
  - name: SPRING_PROFILES_ACTIVE
    value: prod
```

> **Code walkthrough:** This application-prod.properties (prod overrides) example demonstrates YAML configuration pattern. **KEY MECHANISM:** YAML parsers are whitespace-sensitive; indentation errors cause silent value misinterpretation. **WHY IT MATTERS:** unquoted strings starting with special chars (*, &, ?, |) trigger YAML parser errors. **TAKEAWAY: quote strings containing YAML special chars; validate YAML before deploying to production.**

Docker Compose:
```yaml
environment:
  - SPRING_PROFILES_ACTIVE=dev
```

> **Code walkthrough:** This application-prod.properties (prod overrides) example demonstrates YAML configuration pattern. **KEY MECHANISM:** YAML parsers are whitespace-sensitive; indentation errors cause silent value misinterpretation. **WHY IT MATTERS:** unquoted strings starting with special chars (*, &, ?, |) trigger YAML parser errors. **TAKEAWAY: quote strings containing YAML special chars; validate YAML before deploying to production.**

*What separates good from great:* SPRING_PROFILES_ACTIVE (env var) is the
best choice for containers: it works across Docker, Kubernetes, and CI/CD
without modifying the JAR. JVM args require modifying the entry point command.
Properties file activation is for development defaults only (overridable by env).

---

**[MID] Q4 - [CONCEPTUAL] How do profile groups work in Spring Boot 2.4+?**

Profile groups allow one profile activation to trigger multiple related profiles:

```properties
# application.properties
spring.profiles.group.prod=\
  prod-db,prod-security,prod-observability
spring.profiles.group.dev=\
  dev-db,dev-mocks
```

> **Code walkthrough:** This application.properties example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

When `SPRING_PROFILES_ACTIVE=prod`:
- prod profile is active
- prod-db, prod-security, prod-observability are also activated
- application-prod-db.properties, application-prod-security.properties, etc. are loaded

Use case: separate concerns. prod-db owns database config. prod-security owns
security config. prod-observability owns metrics/tracing config. Each can be
managed by different teams.

*What separates good from great:* Profile groups replace the old
spring.profiles.include which had confusing semantics. spring.profiles.include
was profile-scoped (could only be used inside a profile-specific document).
Groups are centrally defined in base config and easier to reason about.

---

**[SENIOR] Q5 - [DEBUGGING] What changed in Spring Boot 2.4 regarding profile loading?**

Spring Boot 2.4 introduced significant changes to config file loading:

**Breaking changes:**
1. Multi-document YAML: spring.profiles changed to
   `spring.config.activate.on-profile`
2. spring.profiles.include no longer works in profile-specific YAML documents
3. Profile-specific files have higher precedence than multi-document properties

Before (Spring Boot < 2.4):
```yaml
server.port: 8080
---
spring.profiles: prod
server.port: 80
```

> **Code walkthrough:** This application.properties example demonstrates YAML configuration pattern. **KEY MECHANISM:** YAML parsers are whitespace-sensitive; indentation errors cause silent value misinterpretation. **WHY IT MATTERS:** unquoted strings starting with special chars (*, &, ?, |) trigger YAML parser errors. **TAKEAWAY: quote strings containing YAML special chars; validate YAML before deploying to production.**

After (Spring Boot 2.4+):
```yaml
server.port: 8080
---
spring.config.activate.on-profile: prod
server.port: 80
```

> **Code walkthrough:** This application.properties example demonstrates YAML configuration pattern. **KEY MECHANISM:** YAML parsers are whitespace-sensitive; indentation errors cause silent value misinterpretation. **WHY IT MATTERS:** unquoted strings starting with special chars (*, &, ?, |) trigger YAML parser errors. **TAKEAWAY: quote strings containing YAML special chars; validate YAML before deploying to production.**

Migration: add spring.config.use-legacy-processing=true to restore old behavior
during migration.

*What separates good from great:* Spring Boot 2.4's config changes were the
most controversial Spring Boot release change in years. Many teams hit upgrade
failures. The spring.config.use-legacy-processing flag was added specifically
for migration. Knowing this exists - and why it was needed - signals production
experience.

---

**[SENIOR] Q6 - [DEBUGGING] How do you test with Spring Profiles?**

@ActiveProfiles in tests activates specific profiles:

```java
@SpringBootTest
@ActiveProfiles("test")
class OrderServiceIntegrationTest {
    // Loads application-test.properties
    // Activates @Profile("test") beans
}
```

```java
// application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.jpa.hibernate.ddl-auto=create-drop
payment.service.mock=true
```

> **Code walkthrough:** This application.properties example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Profile-specific test beans:
```java
@TestConfiguration
public class TestServiceConfig {

    @Bean
    @Primary
    @Profile("test")
    public PaymentService mockPaymentService() {
        return Mockito.mock(PaymentService.class);
    }
}
```

> **Code walkthrough:** This application.properties example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* @Primary ensures the mock bean wins over
any non-mock bean for the same type in tests. Combining @TestConfiguration
(only loaded in tests) with @Profile("test") and @Primary is the pattern for
replacing production beans in integration tests without modifying production code.

---

**[SENIOR] Q7 - [DEBUGGING] Can properties be set in profiles using environment variables?**

Yes - profile-specific properties can reference environment variables:

```properties
# application-prod.properties
spring.datasource.url=\
  jdbc:postgresql://${DB_HOST}/orders
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}

# Fallback with default:
spring.datasource.url=\
  jdbc:postgresql://${DB_HOST:localhost}/orders
```

> **Code walkthrough:** This Fallback with default: example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Property placeholder resolution: Spring resolves ${VARIABLE_NAME} from:
1. System properties
2. Environment variables
3. application.properties

This pattern separates structure (in properties files, committed to git) from
secrets (in environment variables, injected at runtime by CI/CD or secrets manager).

*What separates good from great:* The ${VAR:default} syntax provides a local
development default. DB_HOST defaults to localhost if not set. This lets
developers run without setting all env vars. Production sets all variables
explicitly. Never commit actual secrets to properties files - only the ${VAR}
placeholder syntax.

---

**[STAFF] Q8 - [DEBUGGING] How do Spring Cloud Config and profiles interact?**

Spring Cloud Config Server adds a remote source for profile-specific properties:

Application with spring-cloud-starter-config fetches:
1. {app}-{profile}.properties from Config Server
2. {app}.properties from Config Server
3. Local application-{profile}.properties (override)
4. Local application.properties (lowest)

Config Server reads from Git, S3, Vault, or local filesystem.
Properties from Config Server are treated as higher priority than local files.

```yaml
# bootstrap.yml (loaded before application context)
spring:
  application:
    name: order-service
  config:
    import: "configserver:http://config-server:8888"
  profiles:
    active: prod
```

> **Code walkthrough:** This bootstrap.yml (loaded before application context) example demonstrates YAML configuration pattern. **KEY MECHANISM:** YAML parsers are whitespace-sensitive; indentation errors cause silent value misinterpretation. **WHY IT MATTERS:** unquoted strings starting with special chars (*, &, ?, |) trigger YAML parser errors. **TAKEAWAY: quote strings containing YAML special chars; validate YAML before deploying to production.**

*What separates good from great:* Spring Cloud Config enables runtime
configuration refresh: change a property in Git, push, and running applications
can pick up the change via /actuator/refresh (with @RefreshScope on beans that
use the property). This is how feature flags can change without restarts in
microservice architectures.

---

**[STAFF] Q9 - [DEBUGGING] How do you verify which profile is active in a running application?**

Multiple ways:

1. Startup logs (Spring Boot always logs at INFO):
   `The following profiles are active: prod`

2. Inject Environment and check:
   ```java
   @Autowired Environment env;
   String[] profiles = env.getActiveProfiles();
   ```

> **Code walkthrough:** This bootstrap.yml (loaded before application context) example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

3. /actuator/env endpoint:
   Returns all environment properties including
   activeProfiles array

4. /actuator/info with management.info.env.enabled=true:
   Can expose active profiles as info

5. Programmatically in @Profile-annotated beans:
   The absence of a bean (NoSuchBeanDefinitionException
   or Optional.empty()) tells you that profile is inactive

*What separates good from great:* In Kubernetes, verifying active profile
is part of deployment validation. A pod that starts successfully but with
the wrong profile will use wrong infrastructure silently. Add a startup
log message from your own code that logs the active profiles and key
configuration values (sanitized - no secrets) as a deployment sanity check.

---

# Conditional Beans and @ConditionalOnX


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] How does Spring Boot auto-configuration use @Conditional internally?**

Every Spring Boot auto-configuration class is a @Configuration annotated with
one or more @Conditional guards:

```java
// From spring-boot-autoconfigure source
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({DataSource.class,
    EmbeddedDatabaseType.class})
@ConditionalOnMissingBean(type = {
    "io.r2dbc.spi.ConnectionFactory"})
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({DataSourcePoolMetadataProvidersConfiguration.class,
    DataSourceCheckpointRestoreConfiguration.class})
public class DataSourceAutoConfiguration {
    ...
    @ConditionalOnMissingBean(DataSource.class)
    @ConditionalOnSingleCandidate(EmbeddedDatabaseType.class)
    static class EmbeddedDatabaseConfiguration { ... }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

DataSourceAutoConfiguration only activates if:
1. DataSource class is on classpath (JPA dependency pulls it in)
2. No reactive R2DBC ConnectionFactory exists
3. Within it, embedded database config only if no DataSource bean already exists

*What separates good from great:* Reading Spring Boot auto-configuration source
code is the best way to understand how Spring Boot works. The source is readable
and the conditions tell the exact story of why something activates. Start with
DataSourceAutoConfiguration, JacksonAutoConfiguration, or WebMvcAutoConfiguration.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the difference between @ConditionalOnBean and @ConditionalOnMissingBean?**

@ConditionalOnBean - "register me only IF this bean exists":
- Use case: your bean depends on another (optional) bean being available
- Example: SecurityAuditService - only create if AuditRepository bean exists

@ConditionalOnMissingBean - "register me only IF this bean does NOT exist":
- Use case: provide a default that can be overridden
- Example: provide default CacheManager only if no CacheManager is defined

Order sensitivity: both are sensitive to evaluation order.
@ConditionalOnMissingBean works correctly for auto-configuration because Spring
Boot processes user @Configuration before auto-configuration classes.

*What separates good from great:* The common mistake: placing @ConditionalOnBean
on a bean that both depends on another bean AND is in the same configuration class.
Bean ordering within a single @Configuration class is not guaranteed. The condition
may be evaluated before the depended-on bean is registered. Use separate
@Configuration classes with @AutoConfigureAfter to establish ordering.

---

**[JUNIOR] Q3 - [HANDS-ON] How do you create a custom auto-configuration?**

Three files needed:

1. Auto-configuration class:
```java
@AutoConfiguration  // Spring Boot 3+
@ConditionalOnClass(MyLibraryClient.class)
@EnableConfigurationProperties(MyLibraryProperties.class)
public class MyLibraryAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyLibraryClient myLibraryClient(
            MyLibraryProperties props) {
        return new MyLibraryClient(
            props.getEndpoint(),
            props.getApiKey());
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

2. Properties class:
```java
@ConfigurationProperties(prefix = "my.library")
public class MyLibraryProperties {
    private String endpoint;
    private String apiKey;
    // getters + setters
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

3. Registration file (Spring Boot 3):
   `META-INF/spring/org.springframework.boot
   .autoconfigure.AutoConfiguration.imports`:
```
com.example.MyLibraryAutoConfiguration
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* @AutoConfiguration (Spring Boot 3) adds
proxyBeanMethods=false by default (avoids CGLIB proxy overhead) and registers
the class for deferred loading. @AutoConfigureBefore/@AutoConfigureAfter control
ordering between auto-configurations. Adding @AutoConfigureOrder provides numeric
ordering within the same level.

---

**[MID] Q4 - [DEBUGGING] How do you debug why an auto-configuration did or did not activate?**

Three methods:

1. /actuator/conditions endpoint (production-safe):
   GET /actuator/conditions
   -> positiveMatches: activated configurations + reasons
   -> negativeMatches: not-activated + which condition failed

2. Debug flag (development):
   java -jar app.jar --debug
   OR debug=true in application.properties
   Prints the full Conditions Evaluation Report to console at startup.

3. ConditionEvaluationReport bean:
```java
@Autowired
ConditionEvaluationReport report;

report.getConditionAndOutcomesBySource()
    .forEach((src, outcomes) ->
        log.info("{}: {}", src, outcomes));
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The conditions report shows not just IF a
configuration activated, but WHY or WHY NOT. "Did not match: @ConditionalOnMissingBean
(types: javax.sql.DataSource; SearchStrategy: all) found beans: dataSource" tells
you exactly which bean caused the condition to fail and what type it was.

---

**[MID] Q5 - [CONCEPTUAL] How does @ConditionalOnProperty work with matchIfMissing?**

@ConditionalOnProperty with matchIfMissing controls behavior when the property
is absent:

```java
// Only activate if property equals "true"
// If property is absent: NOT activated (default)
@ConditionalOnProperty(
    name = "feature.advanced.enabled",
    havingValue = "true")
public class AdvancedFeatureConfig { ... }

// Activate by default unless explicitly disabled
// If property is absent: ACTIVATED
// Only deactivated if: feature.cache.enabled=false
@ConditionalOnProperty(
    name = "feature.cache.enabled",
    havingValue = "true",
    matchIfMissing = true)
public class DefaultCacheConfig { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

matchIfMissing=true = "opt-out" feature (active unless you turn it off)
matchIfMissing=false = "opt-in" feature (inactive unless you turn it on)

*What separates good from great:* Spring Boot's own auto-configurations
use both patterns. matchIfMissing=true is used for features that should be
active by default (HikariCP connection pool). matchIfMissing=false is used
for optional features (H2 console: spring.h2.console.enabled=true is required
explicitly). Choosing the right default is a UX decision for library authors.

---

**[MID] Q6 - [CONCEPTUAL] What is @ConditionalOnWebApplication and when does it matter?**

@ConditionalOnWebApplication activates only when the application context is
a web application context:

```java
@ConditionalOnWebApplication(
    type = ConditionalOnWebApplication.Type.SERVLET)
public class MvcAutoConfiguration { ... }

@ConditionalOnWebApplication(
    type = ConditionalOnWebApplication.Type.REACTIVE)
public class WebFluxAutoConfiguration { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Types:
- SERVLET: traditional Spring MVC / Tomcat
- REACTIVE: Spring WebFlux / Netty
- ANY: either type

This prevents web-specific beans from loading in:
- Batch applications (no web context)
- CLI applications with SpringApplicationBuilder
- Non-web components of a larger system

*What separates good from great:* Spring Boot detects the web application
type automatically by checking classpath: DispatcherServlet = Servlet,
DispatcherHandler = Reactive, neither = non-web. You can override with
spring.main.web-application-type=none to disable web context even if
web dependencies are on the classpath.

---

**[SENIOR] Q7 - [CONCEPTUAL] How do @ConditionalOnX annotations interact when multiple are present?**

Multiple @ConditionalOnX annotations on a single class/method are AND-combined.
All conditions must be true for the bean to register:

```java
@Bean
@ConditionalOnClass(RedisCacheManager.class)
@ConditionalOnMissingBean(CacheManager.class)
@ConditionalOnProperty(
    name = "spring.cache.type",
    havingValue = "redis")
public CacheManager redisCacheManager(...) { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

All three must be true:
- RedisCacheManager on classpath
- No CacheManager already defined
- spring.cache.type=redis

For OR logic, use @ConditionalOnExpression with SpEL:
```java
@ConditionalOnExpression(
    "${feature.redis.enabled:false} || "
    + "${feature.cache.type:'none'}"
    + " == 'redis'")
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Or implement a custom Condition:
```java
@Conditional(RedisOrHazelcastCondition.class)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* @AnyNestedCondition and @AllNestedConditions
are meta-conditions that allow composing conditions with OR and AND logic
using nested condition classes. This is cleaner than SpEL for complex conditions
and testable as plain Java.

---

**[SENIOR] Q8 - [CONCEPTUAL] How do you override a Spring Boot auto-configured bean?**

Three patterns:

Pattern 1 - Define the same type (most common):
```java
// Spring Boot provides a default DataSource.
// You define one -> Boot backs off via
// @ConditionalOnMissingBean(DataSource.class)
@Bean
public DataSource dataSource() {
    HikariConfig config = new HikariConfig();
    config.setJdbcUrl(customUrl);
    return new HikariDataSource(config);
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Pattern 2 - Exclude auto-configuration explicitly:
```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class
})
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Pattern 3 - Use application.properties overrides:
```properties
# Many auto-configs expose properties to
# customize behavior without full override
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.minimum-idle=10
```

> **Code walkthrough:** This customize behavior without full override example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* Excluding auto-configuration should be a
last resort. It breaks the intent of auto-configuration and requires you to
configure everything manually. Prefer defining a bean of the same type (triggers
@ConditionalOnMissingBean) or using the exposed configuration properties. If
a Spring Boot auto-configuration doesn't have a property for what you need,
the right approach is to check if there is a property that was added in a newer
version - auto-configurations evolve with each Spring Boot release.

---

**[STAFF] Q9 - [MECHANISM] How do you write unit tests for @Conditional configurations?**

Test auto-configuration with ApplicationContextRunner:

```java
class MyAutoConfigurationTest {

    private final ApplicationContextRunner contextRunner =
        new ApplicationContextRunner()
            .withConfiguration(
                AutoConfigurations.of(
                    MyAutoConfiguration.class));

    @Test
    void backsOffWhenBeanAlreadyDefined() {
        contextRunner
            .withUserConfiguration(
                UserProvidedConfig.class)
            .run(ctx -> {
                assertThat(ctx)
                    .hasSingleBean(MyService.class);
                assertThat(ctx)
                    .getBean(MyService.class)
                    .isInstanceOf(CustomMyService.class);
            });
    }

    @Test
    void providesDefaultWhenNoUserBean() {
        contextRunner
            .run(ctx -> {
                assertThat(ctx)
                    .hasSingleBean(MyService.class);
                assertThat(ctx)
                    .getBean(MyService.class)
                    .isInstanceOf(DefaultMyService.class);
            });
    }

    @Configuration
    static class UserProvidedConfig {
        @Bean public MyService myService() {
            return new CustomMyService();
        }
    }
}
```

> **Code walkthrough:** This customize behavior without full override example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* ApplicationContextRunner is the correct
testing tool for auto-configurations. It creates a lightweight application
context without starting a full Spring Boot application. withPropertyValues(),
withClassLoader(), withSystemProperties() let you control the condition inputs.
The fluent assertions from assertThat(ctx) are specific to context content.
This is the testing pattern used by Spring Boot's own test suite.

---

# Spring Security Filter Chain


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] How does Spring Security integrate with Spring Boot?**

Spring Boot auto-configures Spring Security via SecurityAutoConfiguration.
When spring-boot-starter-security is on the classpath:

1. SecurityAutoConfiguration imports SpringBootWebSecurityConfiguration
2. A default SecurityFilterChain bean is created (form login, HTTP Basic enabled)
3. A default user with random password is created (printed at startup)
4. All endpoints require authentication

To override: define your own SecurityFilterChain @Bean. Your bean replaces
the auto-configured default.

*What separates good from great:* Spring Boot 2.7+ changed SecurityFilterChain
to use the component model (no need to extend WebSecurityConfigurerAdapter).
WebSecurityConfigurerAdapter is deprecated. Modern configuration is done
by defining @Bean SecurityFilterChain(HttpSecurity) methods.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the SecurityContext and how is it stored?**

SecurityContext holds the Authentication for the current request.
Storage: ThreadLocal (default) via SecurityContextHolder.

For each request:
1. SecurityContextHolderFilter loads SecurityContext from session (if any)
2. Authentication filter populates SecurityContext
3. Business logic reads from SecurityContextHolder.getContext()
4. SecurityContextHolderFilter saves context to session (if stateful) and
   clears the ThreadLocal

SecurityContextHolder strategies:
- MODE_THREADLOCAL (default): ThreadLocal per thread
- MODE_INHERITABLETHREADLOCAL: inherited by child threads
- MODE_GLOBAL: single global context (testing only)

*What separates good from great:* For reactive (WebFlux) applications,
SecurityContext is stored in Reactor context, not ThreadLocal. Never use
SecurityContextHolder in reactive code - use ReactiveSecurityContextHolder instead.

---

**[JUNIOR] Q3 - [CONCEPTUAL] What is the difference between hasRole() and hasAuthority()?**

**hasRole("ADMIN")**:
- Checks for an authority with "ROLE_" prefix: "ROLE_ADMIN"
- Spring Security's convention: roles are stored with ROLE_ prefix
- Users loaded with UserDetailsService should have authority "ROLE_ADMIN"

**hasAuthority("ADMIN")**:
- Checks for an authority matching exactly: "ADMIN"
- No prefix convention

This causes a common bug: user has authority "ROLE_ADMIN" but access rule
uses hasRole("ROLE_ADMIN") which checks for "ROLE_ROLE_ADMIN" - access denied.

Rule: use hasRole("X") for roles, grant authorities as "ROLE_X".
Or use hasAuthority() exclusively with exact authority strings.

*What separates good from great:* Spring Security 6 simplified this by making
hasRole() and hasAuthority() more explicit. When using JWT claims for roles,
you typically store them without prefix and use hasAuthority(). Configure the
JwtAuthenticationConverter to extract claims correctly.

---

**[MID] Q4 - [CONCEPTUAL] How does Spring Security handle authentication?**

Authentication flow:

1. AuthenticationFilter extracts credentials from request
   (JWT Bearer header, Basic Auth header, form data)

2. Creates unauthenticated Authentication token
   (e.g., UsernamePasswordAuthenticationToken with credentials but not authenticated)

3. Passes to AuthenticationManager (usually ProviderManager)

4. ProviderManager iterates AuthenticationProvider list
    - DaoAuthenticationProvider: loads UserDetails, verifies password
    - JwtAuthenticationProvider: validates JWT
    - LdapAuthenticationProvider: verifies against LDAP

5. Provider returns authenticated Authentication
   (with principal, credentials, authorities, isAuthenticated=true)

6. Authenticated Authentication stored in SecurityContext

Custom AuthenticationProvider:
```java
@Component
public class ApiKeyAuthProvider
        implements AuthenticationProvider {
    @Override
    public Authentication authenticate(
            Authentication auth) throws AuthenticationException {
        String apiKey = auth.getCredentials().toString();
        ApiKeyUser user = apiKeyService.lookup(apiKey);
        if (user == null) throw new BadCredentialsException(
            "Invalid API key");
        return new UsernamePasswordAuthenticationToken(
            user, null, user.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authType) {
        return ApiKeyAuthenticationToken.class
            .isAssignableFrom(authType);
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* AuthenticationProvider.supports() allows
multiple providers to coexist for different authentication types. ProviderManager
iterates providers, calling authenticate() only on those where supports() returns
true. This is how JWT + Basic + API key can all work in the same application.

---

**[MID] Q5 - [CONCEPTUAL] What is the difference between authentication and authorization in Spring Security?**

**Authentication** - WHO are you?
- Verifies identity
- Runs in authentication filters (before DispatcherServlet)
- Result: populated SecurityContext with authenticated principal
- Failure: 401 Unauthorized

**Authorization** - WHAT are you allowed to do?
- Checks permission based on identity
- Runs in AuthorizationFilter (Servlet layer) or AOP (@PreAuthorize, @PostAuthorize)
- HttpSecurity rules: URL-level authorization
- @PreAuthorize: method-level authorization
- Failure: 403 Forbidden

Order: authentication always before authorization. Authorization reads from
SecurityContext which authentication populated.

*What separates good from great:* Having both URL-level and method-level
authorization is defense in depth. URL-level catches requests early (no
controller code runs). Method-level catches direct service calls not going
through the HTTP layer (batch jobs, event listeners).

---

**[MID] Q6 - [CONCEPTUAL] How do you configure multiple SecurityFilterChain beans?**

Multiple SecurityFilterChain beans allow different security configurations
for different URL patterns:

```java
@Bean
@Order(1)  // checked first
public SecurityFilterChain apiSecurity(
        HttpSecurity http) throws Exception {
    return http
        .securityMatcher("/api/**")  // only for /api/**
        .csrf(AbstractHttpConfigurer::disable)
        .sessionManagement(s -> s.sessionCreationPolicy(
            SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(a -> a
            .anyRequest().authenticated())
        .oauth2ResourceServer(
            c -> c.jwt(Customizer.withDefaults()))
        .build();
}

@Bean
@Order(2)  // checked second (lower priority)
public SecurityFilterChain webSecurity(
        HttpSecurity http) throws Exception {
    return http
        .securityMatcher("/**")  // all other URLs
        .csrf(Customizer.withDefaults())
        .formLogin(Customizer.withDefaults())
        .build();
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

FilterChainProxy checks chains in @Order order. First matching chain handles
the request. API routes use JWT (stateless). Web routes use form login (session).

*What separates good from great:* Spring Security 6 moved from
antMatcher/mvcMatcher to requestMatcher. securityMatcher() accepts
RequestMatcher instances. Using Spring MVC's MvcRequestMatcher (vs AntPathRequestMatcher)
ensures the matcher uses the same path matching as your controllers - preventing
bypass via path variation.

---

**[SENIOR] Q7 - [CONCEPTUAL] How does CSRF protection work and when should you disable it?**

CSRF (Cross-Site Request Forgery) attacks trick authenticated browser users into
making malicious requests using their session cookies.

Spring Security's CSRF protection:
1. Generates a CSRF token stored in the session
2. Requires the token in a header or form field for state-changing requests
   (POST, PUT, DELETE, PATCH)
3. Requests without the token get 403 Forbidden

When to DISABLE CSRF:
- Stateless REST APIs (JWT in Authorization header, no session cookies)
- When clients are not browsers (mobile apps, other backends)
- When your frontend sends the CSRF token in the request (Angular, React with
  Spring Security CSRF support)

When to KEEP CSRF enabled:
- Traditional web applications using session cookies
- Any endpoint called from a browser where cookies carry the session

```java
// Disable for stateless API
.csrf(AbstractHttpConfigurer::disable)

// Or disable only for specific paths
.csrf(csrf -> csrf
    .ignoringRequestMatchers("/api/webhooks/**"))
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* SameSite cookie attribute provides additional
CSRF protection at the browser level. Setting session cookie to SameSite=Strict
prevents cross-site requests from including the cookie at all. This is a defense-
in-depth approach: CSRF tokens + SameSite cookie together.

---

**[SENIOR] Q8 - [HANDS-ON] How do you implement @PreAuthorize method security?**

@PreAuthorize checks authorization before the method executes:

```java
@Configuration
@EnableMethodSecurity  // enables @PreAuthorize
public class MethodSecurityConfig {}

@Service
public class OrderService {

    @PreAuthorize("hasRole('USER')")
    public List<Order> getMyOrders(String userId) {
        return orderRepository.findByUserId(userId);
    }

    // SpEL with parameters
    @PreAuthorize("hasRole('ADMIN') or " +
                  "#userId == authentication.name")
    public Order getOrder(Long orderId, String userId) {
        return orderRepository.findById(orderId)...;
    }

    @PostAuthorize("returnObject.userId == " +
                   "authentication.name")
    public Order getOrderById(Long id) {
        // method runs, then return value is checked
        return orderRepository.findById(id)...;
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

@PreAuthorize evaluates SpEL: `authentication.name`, `principal.username`,
`hasRole()`, `hasAuthority()`, method parameters (#paramName).

*What separates good from great:* @PreAuthorize uses Spring AOP (BeanPostProcessor).
This means: only works on Spring beans, not on new-created objects; self-invocation
bypasses it; must be on public methods (CGLIB). For fine-grained data-level
authorization, consider a domain-object security framework (ACL module).

---

**[SENIOR] Q9 - [DEBUGGING] How do you debug Spring Security issues?**

Enable security debug logging:
```properties
logging.level.org.springframework.security=DEBUG
# Or only filter chain decisions:
logging.level.org.springframework.security.web=DEBUG
```

> **Code walkthrough:** This Or only filter chain decisions: example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

This logs:
- Which SecurityFilterChain matched the request
- Which filters ran and in what order
- Authentication decisions and their reasoning
- Authorization decisions

Spring Security's built-in debug mode:
```java
@EnableWebSecurity(debug = true)
```
> **Code walkthrough:** This Or only filter chain decisions: example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

This logs request details and filter invocations at DEBUG level.

Key things to look for in debug output:
1. "Security filter chain: [" ... "]" - which chain matched
2. "An Authentication object was found" - authentication succeeded
3. "Voter ... voted ..." - authorization decision
4. "Access is denied" - what rule blocked access

*What separates good from great:* The most common debug step is checking what
authorities are on the Authentication object:
```java
Authentication auth = SecurityContextHolder.getContext()
    .getAuthentication();
log.debug("Authorities: {}", auth.getAuthorities());
```
> **Code walkthrough:** This Or only filter chain decisions: example demonstrates Java API usage using authentication. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

If the authority list doesn't match your hasRole/hasAuthority checks,
that's your bug.

---

# Spring Boot Actuator


---

### 🎯 Interview Deep-Dive

**Timing:** Medium ★★☆ - 9 questions.

---

**[JUNIOR] Q1 - [CONCEPTUAL] What is Spring Boot Actuator and why do you use it?**

Actuator provides production-ready operational endpoints for Spring Boot applications:
- Health checking (Kubernetes probes, monitoring systems)
- Metrics collection (JVM, HTTP, database, custom business metrics)
- Application diagnostics (/conditions, /beans, /env)
- Log level management (/loggers)
- Thread and heap dumps for debugging

Why use it: it provides the operational visibility every production application
needs, auto-configured with zero code. Replacing Actuator manually would require
implementing health endpoints, Prometheus integration, thread dump endpoints etc.

*What separates good from great:* Actuator uses the same MVC/WebFlux
infrastructure as your application. Its endpoints are regular Spring MVC
endpoints registered on a separate path (/actuator). This means Spring Security
can protect them normally.

---

**[JUNIOR] Q2 - [CONCEPTUAL] What is the difference between liveness and readiness in Kubernetes probes?**

**Liveness probe** - should the container be restarted?
- Failing liveness: Kubernetes kills and restarts the container
- Use for: unrecoverable internal states (deadlock, OOM, corrupted state)
- Spring Boot endpoint: /actuator/health/liveness
- DO NOT include external dependency checks - will cause restart loop

**Readiness probe** - should the container receive traffic?
- Failing readiness: Kubernetes removes the container from the Service load balancer
- Container continues running - can recover and become ready
- Use for: startup warm-up, temporary dependency unavailability
- Spring Boot endpoint: /actuator/health/readiness
- INCLUDE external dependency checks here

Configure in application.properties:
```properties
management.endpoint.health.probes.enabled=true
management.health.livenessstate.enabled=true
management.health.readinessstate.enabled=true
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* Custom ReadinessHealthIndicators allow
programmatic readiness control:
```java
ApplicationAvailability.setReadinessState(
    ReadinessState.REFUSING_TRAFFIC);
```
> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

This is how graceful shutdown works - the app signals not-ready before
the server stops accepting connections.

---

**[JUNIOR] Q3 - [CONCEPTUAL] How do you secure Actuator endpoints?**

Option 1 - Separate management port (best for production):
```properties
management.server.port=8081  # not exposed externally
```
> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

No authentication needed - the port is only accessible internally.

Option 2 - Spring Security rules:
```java
@Bean
public SecurityFilterChain actuatorSecurity(
        HttpSecurity http) throws Exception {
    return http
        .securityMatcher("/actuator/**")
        .authorizeHttpRequests(a -> a
            .requestMatchers("/actuator/health",
                "/actuator/info").permitAll()
            .anyRequest().hasRole("ADMIN"))
        .httpBasic(Customizer.withDefaults())
        .build();
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Option 3 - Expose only safe endpoints:
```properties
management.endpoints.web.exposure.include=health,info
```
> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Minimize attack surface by only exposing needed endpoints.

*What separates good from great:* Production best practice: management port
on an internal network + Spring Security requiring ADMIN role. Defense in depth:
even if someone reaches the management port, they need valid credentials.

---

**[MID] Q4 - [HANDS-ON] How do you create a custom HealthIndicator?**

Implement HealthIndicator and return Health.up() or Health.down() with details:

```java
@Component
public class ExternalServiceHealthIndicator
        implements HealthIndicator {

    private final ExternalServiceClient client;

    @Override
    public Health health() {
        try {
            // Lightweight connectivity check
            ResponseEntity<Void> response =
                client.ping();
            return Health.up()
                .withDetail("status",
                    response.getStatusCode())
                .build();
        } catch (Exception e) {
            return Health.down(e)
                .withDetail("error",
                    e.getMessage())
                .build();
        }
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using Spring annotation. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

For Kubernetes readiness (not liveness):
Implement ReadinessHealthIndicator to contribute only to /health/readiness.

*What separates good from great:* HealthIndicator.health() is called on every
health check request. Ensure it is fast (< 1 second) to avoid delaying
Kubernetes probes. For expensive checks, add caching: cache the result for
10 seconds. If the check itself fails with an exception after 5 seconds,
Kubernetes readiness timeout may fire before the check returns.

---

**[MID] Q5 - [CONCEPTUAL] What is Micrometer and how does it integrate with Actuator?**

Micrometer is a metrics facade - a vendor-neutral API for recording metrics
that delegates to backend-specific implementations.

Supported backends: Prometheus, Datadog, New Relic, CloudWatch, Graphite,
Influx, Wavefront, and more.

Spring Boot auto-configures a MeterRegistry based on what's on the classpath.
Auto-instrumented metrics:
- JVM: heap, GC, threads, classes (JvmMetrics)
- HTTP server: request count, duration, errors (WebMvcMetrics)
- Database: HikariCP pool metrics (HikariMetrics)
- Cache: hit/miss/size (CacheMetrics)

Access in code:
```java
@Autowired MeterRegistry registry;

Counter.builder("my.counter").register(registry)
       .increment();
Gauge.builder("queue.size", queue, Queue::size)
     .register(registry);
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using Spring annotation. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* Micrometer's global registry
(Metrics.globalRegistry) allows recording metrics without injecting MeterRegistry.
But injecting MeterRegistry is cleaner and testable. For high-throughput code,
pre-build meters at construction time rather than looking them up by name on
each call - meter lookup is not free.

---

**[MID] Q6 - [CONCEPTUAL] How do you expose custom application info via Actuator?**

/actuator/info exposes application information. Contribute via InfoContributor:

```java
@Component
public class AppInfoContributor
        implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("app", Map.of(
            "name", "Order Service",
            "version", "2.1.0",
            "buildTime", Instant.now().toString()
        ));
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Or via application.properties:
```properties
management.info.env.enabled=true
info.app.name=Order Service
info.app.version=@project.version@  # Maven placeholder
info.app.description=Handles order processing
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Git info (via spring-boot-maven-plugin):
```properties
management.info.git.enabled=true
management.info.git.mode=full  # includes branch, commit hash
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* Including git commit hash in /actuator/info
is a crucial production practice. When an incident occurs, you immediately know
which commit is deployed. spring-boot-maven-plugin generates git.properties with
branch, commit ID, commit time, and dirty flag. Enable it with a few config lines.

---

**[SENIOR] Q7 - [DEBUGGING] How do you use /actuator/loggers to change log levels at runtime?**

/actuator/loggers lists current log levels. POST to change them:

```
# Get all loggers
GET /actuator/loggers

# Get specific logger
GET /actuator/loggers/com.example.orders

# Change to DEBUG at runtime (no restart!)
POST /actuator/loggers/com.example.orders
Content-Type: application/json
{"configuredLevel": "DEBUG"}

# Reset to default
POST /actuator/loggers/com.example.orders
{"configuredLevel": null}
```

> **Code walkthrough:** This Reset to default example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

This is critical for production debugging: enable DEBUG logging for a specific
package without restarting the application. After debugging, reset to INFO.

*What separates good from great:* Log level changes via /actuator/loggers are
in-memory only and lost on restart. For persistent changes in production, use
Spring Cloud Config Server which can push configuration changes to running
applications via /actuator/refresh (with @RefreshScope on the beans).

---

**[STAFF] Q8 - [MECHANISM] What does /actuator/conditions show?**

/actuator/conditions shows the Conditions Evaluation Report - exactly what
/debug output shows at startup, but accessible at runtime.

Response includes:
- **positiveMatches**: auto-configurations that fired, and which conditions matched
- **negativeMatches**: auto-configurations that did NOT fire, and which condition failed
- **unconditionalClasses**: classes loaded without conditions

Use cases:
- Verify that expected auto-configuration activated
- Debug why an expected auto-configuration did not fire
- Security audit: check what features are enabled in production

Example debug:
"Why doesn't my application use my custom DataSource?"
-> Check positiveMatches for DataSourceAutoConfiguration
-> ConditionalOnMissingBean: says a DataSource bean already exists (your bean)
-> Confirms your custom bean is being used

*What separates good from great:* /actuator/conditions is the most useful
debugging endpoint for Spring Boot issues. Before reading Spring Boot source
code to understand why something is or isn't configured, check /actuator/conditions.

---

**[STAFF] Q9 - [MECHANISM] How do you add timing metrics to business methods with Micrometer?**

Option 1 - @Timed annotation (requires AspectJ or Micrometer's aspect):
```java
@Service
public class OrderService {
    @Timed(value = "orders.create",
           description = "Order creation time",
           percentiles = {0.5, 0.95, 0.99})
    public Order createOrder(OrderRequest req) { ... }
}
```

> **Code walkthrough:** This Reset to default example demonstrates Java API usage using Spring annotation. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Option 2 - Manual Timer (more control):
```java
@Service
public class OrderService {
    private final Timer createTimer;

    public OrderService(MeterRegistry registry) {
        this.createTimer = Timer
            .builder("orders.create")
            .tag("type", "manual")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(registry);
    }

    public Order createOrder(OrderRequest req) {
        return createTimer.record(
            () -> doCreateOrder(req));
    }
}
```

> **Code walkthrough:** This Reset to default example demonstrates exception handling using Spring annotation. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

Option 3 - LongTaskTimer for long-running operations:
```java
LongTaskTimer batchTimer = LongTaskTimer
    .builder("batch.processing.active")
    .register(registry);

// Shows how long current batches have been running
batchTimer.record(() -> runBatch(items));
```

> **Code walkthrough:** This Reset to default example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* Percentile metrics (p50, p95, p99) require
client-side aggregation (publishPercentiles) which uses more memory, or server-
side aggregation in Prometheus (publishPercentileHistogram). For Prometheus,
use publishPercentileHistogram=true and let Prometheus calculate percentiles with
histogram_quantile(). This scales better than pre-computing percentiles in-process.

---