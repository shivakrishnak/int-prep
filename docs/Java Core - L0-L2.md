---
layout: default
title: "Java Core - L0 to L2"
nav_order: 2
---

## Table of contents
{: .no_toc }

1. TOC
{:toc}


# Java Core - L0 Orientation

## Why Java Exists and Design Philosophy

---

### 🎯 Model Answer

**30 seconds:**
> Java was created in 1995 at Sun Microsystems to solve the portability
> problem: write once, run anywhere. Software at the time had to be
> recompiled for each CPU and OS. Java introduced the JVM - a virtual
> machine that runs the same bytecode on any platform. The language was
> also designed for network and enterprise computing: automatic memory
> management (garbage collection), strong type system, and built-in
> support for networking. Today it powers Android, enterprise backends,
> financial systems, and big data platforms.

**3 minutes (Senior):**
> Java's five design principles are: (1) Simple - smaller syntax than
> C++, no pointers, no header files. (2) Object-oriented - everything is
> a class; the mental model is objects exchanging messages. (3) Portable
> - bytecode compiled once, run by any JVM on any OS. (4) Robust -
    > garbage collection eliminates manual memory management; strong static
    > type system catches errors at compile time; checked exceptions force
    > error handling. (5) Multithreaded - threading primitives built into
    > the language and standard library.
>
> The design trade-offs: verbose syntax in exchange for readability.
> Garbage collection eliminates memory bugs but introduces GC pauses.
> The JVM gives portability and JIT optimization but adds startup time.
> Checked exceptions force error handling but create API friction.
>
> Java's design influenced everything since: C#, Kotlin, Scala, and Groovy
> all borrow from Java. The JVM became a platform in its own right,
> hosting over 100 programming languages.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Why Java was created | 60 seconds |
| JVM portability mechanism | 2-3 minutes |
| Key design trade-offs | 2-3 minutes |
| Java vs alternatives | 2-3 minutes |
| Java enterprise ecosystem | 2-3 minutes |
| Modern Java evolution | 2-3 minutes |
| Java performance model | 2-3 minutes |

---

**Q1 (Why Java was created): Why was Java created and what problem
did it solve?**

A: Java was created in 1995 at Sun Microsystems to solve the "Write Once,
Run Anywhere" problem. Pre-Java software was platform-specific: C/C++
programs for Windows ran only on Windows (x86 binaries), Solaris programs
ran only on Solaris (SPARC), etc. Each OS/CPU combination required separate
compilation, distribution, and maintenance.

Java's solution: compile to JVM bytecode (a platform-independent binary),
not to native machine code. Any device with a JVM can run the bytecode.
This was critical for networked computing: a web applet could be downloaded
and run on any browser's JVM regardless of OS.

Secondary problems solved: C/C++ programs frequently had memory bugs
(buffer overflows, use-after-free) that caused crashes and security
vulnerabilities. Java's garbage collector and absence of pointers
eliminated this class of bugs.

*What separates good from great:* Java's portability argument was
compelling in 1995. By 2024, cloud deployment has changed the equation:
Docker containers make any compiled language "portable" in the deployment
sense. GraalVM Native Image compiles Java to native binaries (startup
in milliseconds, no JVM overhead). The original WORA argument is less
important; Java's dominance now rests on its ecosystem (frameworks,
tools, libraries) and JVM maturity, not portability alone.

---

**Q2 (JVM portability mechanism): How does the JVM provide
platform independence?**

A: The JVM is a specification for a virtual computing machine. Each
OS/CPU combination has its own JVM implementation (Windows JVM, Linux
JVM, macOS JVM). All JVMs understand and execute the same bytecode.

Compilation pipeline:
```
Java source (.java)
    |
    v  javac (Java compiler)
Java bytecode (.class files)
    |
    v  JVM (OS-specific)
  Linux JVM:    interprets or JIT-compiles to x86 instructions
  Windows JVM:  interprets or JIT-compiles to x86 instructions
  macOS JVM:    interprets or JIT-compiles to ARM/x86 instructions
  Android VM:   compiles to DEX format for Dalvik/ART
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Bytecode advantages:
- Verified before execution (bytecode verifier): prevents invalid operations
- JIT-compiled to native code at runtime for performance
- Profiling-guided optimization: JIT optimizes hot paths based on actual
  execution data

*What separates good from great:* The JVM's JIT compiler has decades
of optimization sophistication. It performs: inline caching (virtual
dispatch elimination), escape analysis (heap-to-stack promotion, lock
elision), dead code elimination, and loop vectorization. A JVM running
a hot method can produce machine code that rivals hand-tuned C++.
The catch: JIT takes time to "warm up" (observe execution patterns).
Startup performance suffers; long-running servers benefit most.

---

**Q3 (Key design trade-offs): What are Java's most important design
trade-offs?**

A:

| Decision | Benefit | Trade-off |
|---|---|---|
| Garbage collection | No memory bugs | GC pauses; less memory control |
| Strong typing | Compile-time errors | Verbosity; no duck typing |
| Checked exceptions | Explicit error handling | API friction (streams) |
| No multiple inheritance | No diamond problem | Less expressive hierarchy |
| JVM overhead | Portability + JIT | Startup time; memory footprint |
| No operator overload | Consistent semantics | Less expressive DSLs |

**The checked exception trade-off in practice:**
```java
// PROBLEM: checked exception in lambda
List<String> urls = List.of("http://...");
urls.stream()
    .map(url -> {
        try {
            return new URL(url); // throws MalformedURLException (checked)
        } catch (MalformedURLException e) {
            throw new RuntimeException(e); // ugly wrapper
        }
    });
// Modern Java: use RuntimeException wrappers or Vavr's Try
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline uice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* Java's design choices made sense
for 1995 enterprise programming. Some trade-offs aged poorly (verbose
null handling → Optional, verbose loops → streams, no value types →
Project Valhalla). The Java platform responds slowly to changes to
maintain backward compatibility - a design principle that has kept
Java relevant for 30 years.

---

**Q4 (Java vs alternatives): When would you choose Kotlin or Scala
over Java?**

A:

**Kotlin over Java:**
- More concise syntax (null safety, data classes, extension functions)
- Better Android support (first-class language for Android)
- Coroutines for structured concurrency (cleaner than CompletableFuture)
- Fully interoperable with Java (can call any Java library)
- Use when: new project, Android, team prefers conciseness

**Scala over Java:**
- Functional programming (case classes, pattern matching, immutable collections)
- Type system more powerful (higher-kinded types, implicits/given)
- Akka (actor model) ecosystem
- Use when: functional style, Spark (written in Scala), complex type constraints

**Java over alternatives:**
- Largest ecosystem and library support
- Most hiring market (easier to find developers)
- Best JVM tooling (IntelliJ, Gradle, Maven)
- Most stable API (30 years of backward compatibility)
- Use when: long-term enterprise projects, maximum hiring pool

*What separates good from great:* The choice should also consider
operational factors: who will maintain the code, onboarding speed
for new engineers, and tooling maturity. Scala's steep learning curve
means Scala teams are often smaller and more expert, making hiring
harder. Kotlin's gentle migration path (mix Kotlin and Java files in
the same project) makes it a low-risk choice for Java teams.

---

**Q5 (Java enterprise ecosystem): Name the core Java enterprise
ecosystem components.**

A:
| Layer| Technology| Purpose|
|---|------------------------------|-------------------------------------------|
| Build| Maven, Gradle| Dependency management, compilation, testing|
| Application framework| Spring Boot| DI, REST, data access|
| Web| Spring MVC, JAX-RS| HTTP request handling|
| Persistence| JPA/Hibernate, Spring Data| ORM, database access|
| Testing| JUnit 5, Mockito, Testcontainers| Unit, mock, integration testing|
| Messaging| Kafka, RabbitMQ clients| Async message processing|
| Observability| Micrometer, Prometheus, OpenTelemetry| Metrics, tracing, loggin
| Security| Spring Security| AuthN, AuthZ|
| Containers| Docker + Kubernetes| Deployment, scaling|

*What separates good from great:* The Spring ecosystem has become
the de facto standard for Java enterprise development. Understanding
how Spring's dependency injection (ApplicationContext, BeanFactory)
wires these components together is more valuable in interviews than
memorizing individual framework APIs.

---

**Q6 (Modern Java evolution): What are the most significant Java
improvements from Java 8 to Java 21?**

A:
| Version| Feature| Significance|
| Java 8 (2014)| Lambdas, Streams, Optional, java.time| Functional programming i
| Java 9 (2017)| Module system (Jigsaw), JShell| Strong encapsulation, REPL|
| Java 11 (2018 LTS)| var in lambdas, HTTP client, String methods| Last free Ora
| Java 14 (2020)| Records (preview)| Immutable data classes, reduced boilerplate
| Java 15 (2020)| Sealed classes (preview)| Restricted inheritance hierarchies|
| Java 16 (2021)| Pattern matching instanceof| Eliminates cast boilerplate|
| Java 17 (2021 LTS)| Records + Sealed final, sealed classes final| Long-term su
| Java 19 (2022)| Virtual threads (preview)| Lightweight concurrency|
| Java 21 (2023 LTS)| Virtual threads GA, Sequenced Collections| Official lightw

*What separates good from great:* The LTS version choice drives
production deployment. Most enterprises are on Java 11 or 17.
Java 21 is the current LTS. Key decision: Java 21 for new projects
(virtual threads, records, sealed classes all stable); Java 17 for
existing projects that haven't migrated. Java 8/11 are approaching
or past community support end dates.

---

**Q7 (Java performance model): How does Java achieve competitive
performance despite the JVM?**

A: Java achieves competitive performance through a multi-layer
optimization stack:

1. **JIT compilation**: HotSpot JVM identifies "hot" methods (called
   frequently) and compiles them to native machine code.
   Level 1-4 tiered compilation: interpreted -> client-compiled ->
   server-compiled (with profiling-guided optimizations).

2. **Inlining**: the most powerful optimization. Small, frequently
   called methods are inlined at the call site, eliminating call
   overhead and enabling further optimizations.

3. **Escape analysis**: determines if an object is accessed only within
   a method. If so: allocate on stack instead of heap (no GC overhead),
   eliminate unnecessary synchronization.

4. **Dead code elimination**: unreachable code removed; branches that
   never execute (in profiled data) eliminated.

5. **Loop vectorization**: loops operating on arrays auto-vectorized
   to SIMD instructions (SSE/AVX on x86).

Benchmarks: Java throughput vs C++ (long-running processes):
typically 80-120% of C++ performance on CPU-bound benchmarks.
Memory usage is higher (GC overhead, object headers).

*What separates good from great:* JIT optimization depends on "warm-up":
the JVM must observe execution before optimizing. In serverless
or short-lived processes, Java has poor cold-start performance. GraalVM
Native Image compiles Java ahead-of-time to native code, eliminating
JVM startup time. Spring Boot 3 / Micronaut / Quarkus support Native
Image for serverless-friendly Java.

---


## Java Ecosystem and JDK Structure

---

### 🎯 Model Answer

**30 seconds:**
> The JDK (Java Development Kit) is the complete developer package:
> compiler (`javac`), runtime (`java`), standard library, and tools.
> The JRE (Java Runtime Environment) is just the runtime (for deploying
> apps). The JVM is the virtual machine within the JRE that executes
> bytecode. OpenJDK is the open-source reference implementation used
> by most distributions (Temurin/Adoptium, Amazon Corretto, Azul Zulu).
> Oracle JDK is commercial with different licensing.

**3 minutes (Senior):**
> JDK structure: javac compiles .java to .class (bytecode). The jar
> tool packages .class files into a JAR archive. The JVM's class loader
> loads .class files on demand. The JVM's bytecode interpreter and JIT
> compiler execute the bytecode. The standard library (java.*, javax.*)
> provides the core APIs.
>
> JVM implementations: HotSpot (default, OpenJDK and Oracle JDK),
> OpenJ9 (IBM/Eclipse, lower memory footprint), GraalVM (ahead-of-time
> compilation for native image, polyglot). All implement the same JVM
> specification - bytecode runs on any.
>
> Java distributions: there are 17+ JDK distributions. Key ones:
> Eclipse Temurin (Adoptium) - free, community-maintained, most popular.
> Amazon Corretto - free, AWS-supported, used in AWS Lambda.
> Azul Zulu - commercial support option. Oracle JDK 11+ requires commercial
> license for production use.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| JDK vs JRE vs JVM | 60 seconds |
| Distribution choice | 2-3 minutes |
| Module system | 2-3 minutes |
| JDK tools | 2-3 minutes |
| Container JVM | 2-3 minutes |
| jlink custom JRE | 2-3 minutes |
| OpenJDK governance | 2-3 minutes |

---

**Q1 (JDK vs JRE vs JVM): Explain the relationship between JDK, JRE,
and JVM.**

A: Three nested components:

**JVM (innermost):** The virtual machine that loads and executes Java
bytecode. It includes: class loader (loads .class files), bytecode
verifier (security check), interpreter/JIT compiler (execution), and
garbage collector (memory management).

**JRE (middle):** The runtime environment = JVM + Java class library.
The class library includes all of `java.lang`, `java.util`, `java.io`,
etc. A user running a Java application needs the JRE.

**JDK (outermost):** The development kit = JRE + development tools.
Tools: `javac` (compiler), `jar` (archiver), `javadoc` (docs), `jdb`
(debugger), `jshell` (REPL), diagnostic tools (jstack, jmap, jcmd).
A developer writing Java code needs the JDK.

Since Java 11: the separate JRE download is no longer available. Only
the JDK is distributed. Use `jlink` to create a minimal JRE for
production deployment.

*What separates good from great:* With the Java Module System (Java 9+),
the standard library is broken into modules (java.base, java.sql,
java.xml, etc.). `jlink` can create a custom JRE containing only the
modules your application uses, dramatically reducing the runtime size.
A minimal Spring Boot app might have a custom JRE of ~50MB vs 300MB+
for a full JDK.

---

**Q2 (Distribution choice): How do you choose a JDK distribution for
a production system?**

A: Key selection criteria:

**1. License:** Oracle JDK 11+ requires a commercial license for
production (Java SE Subscription). OpenJDK-based distributions (Temurin,
Corretto, Zulu) are free for production.

**2. Support timeline:** LTS versions (11, 17, 21) receive security
patches for years. Match the support period to your project's lifecycle.
Eclipse Temurin: 8 years for LTS. Amazon Corretto: 8 years for LTS.

**3. Platform:** Amazon Corretto is AWS-native, includes AWS-specific
performance improvements, optimized for Lambda. Azul Zulu CE supports
the widest platform matrix.

**4. Performance requirements:** GraalVM for native image (serverless,
CLI tools). OpenJ9 for memory-constrained environments (IBM cloud).

**Standard choice for most projects:** Eclipse Temurin (Adoptium) -
most popular, free, well-maintained, broad platform support.

*What separates good from great:* For containerized deployments,
all major distributions behave similarly - the JDK is inside a Docker
image and the distribution matters only for the support contract.
The important operational choice is the JDK version (LTS cadence)
and the GC algorithm (`-XX:+UseG1GC` default in Java 9+, ZGC for
low-latency in Java 15+, Shenandoah for low-pause alternative).

---

**Q3 (Module system): When and why would you use the Java module system?**

A: The Java Module System (JPMS, introduced in Java 9) provides:
- Strong encapsulation: packages not in `exports` are inaccessible
  from outside the module, even via reflection (unless `opens`)
- Explicit dependencies: `requires` makes dependencies visible
- Service loading: `provides`/`uses` for ServiceLoader patterns

**When to use JPMS:**
- Building libraries or frameworks that need strong API encapsulation
- Applications requiring security isolation between components
- Building custom JREs with `jlink` (modules must be declared)

**When NOT to use JPMS:**
- Application code in a classpath-based project (adds complexity)
- When using many third-party libraries not modularized (requires
  `--add-opens` hacks)

**Practical reality (2024):**
Most applications run on the classpath, not the module path.
Spring Boot is primarily classpath-based. The module system is more
valuable for library authors (Guava, Jackson) than application developers.

*What separates good from great:* The module system's strong
encapsulation broke many libraries that used reflection (like Spring
and Hibernate). The workaround: `--add-opens` JVM arguments to open
packages for reflection. Spring Boot adds these automatically for its
known needs. But any custom framework code that uses reflection on
JDK internals needs explicit `--add-opens`.

---

**Q4 (JDK tools): What JDK diagnostic tools do you use for production
issues?**

A:
| Tool | Purpose | Example |
|---|---|---|
| `jstack <pid>` | Thread dump (deadlock, stuck threads) | `jstack 12345 > dump.txt` |
| `jmap -heap <pid>` | Heap statistics | `jmap -heap 12345` |
| `jcmd <pid> Thread.print` | Thread dump (preferred over jstack) | `jcmd 12345 Thread.print` |
| `jcmd <pid> VM.gc` | Force GC | `jcmd 12345 GC.run` |
| `jcmd <pid> JFR.start` | Start Flight Recorder | `jcmd 12345 JFR.start duration=60s` |
| `jstat -gc <pid>` | GC statistics | `jstat -gc 12345 1000 10` |
| `jps` | List JVM processes | `jps -lv` |

*What separates good from great:* In production, `jcmd` is preferred
over `jstack`, `jmap`, and `jinfo` (all three are deprecated in newer
JDKs). `jcmd` provides all their functionality. For containers:
`kubectl exec -it <pod> -- jcmd 1 Thread.print` (PID 1 is usually
the JVM in a container). For observability at scale: Spring Actuator
`/actuator/threaddump` and `/actuator/heapdump` expose these without
needing to exec into containers.

---

**Q5 (Container JVM): What JVM flags are essential when running Java
in containers?**

A:
```bash
# Essential container-aware flags:

# Memory: set heap to 75% of container memory limit
-XX:MaxRAMPercentage=75.0
# Or explicit heap size:
-Xms512m -Xmx1g

# GC algorithm (Java 15+: ZGC for low-latency, default G1GC for throughput):
-XX:+UseZGC          # sub-millisecond GC pauses
-XX:+UseG1GC         # default; good balance

# Thread stack size (reduce from default 512KB-1MB):
-Xss256k             # reduce for high thread count containers

# JIT compilation: restrict to container CPU count (Java 10+):
# Automatic since Java 10: reads /sys/fs/cgroup CPU quota

# Diagnostic output:
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/java/
-XX:+ExitOnOutOfMemoryError  # fail fast, let k8s restart

# JFR for continuous profiling (Java 11+):
-XX:StartFlightRecording=dumponexit=true,filename=/var/log/jfr/app.jfr
```

> **Code walkthrough:** This JFR for continuous profiling (Java 11+): example demonstrates shell script pattern using container. **KEY MECHANISM:** the shell executes commands sequentially; pipes pass stdout of one command to stdin of the next. **WHY IT MATTERS:** unquoted variables with spaces cause word splitting - IFS splits the value into multiple arguments. **TAKEAWAY: always double-quote variables: "$VAR"; use [[ ]] instead of [ ] for safer conditionals.**

*What separates good from great:* The single most important flag is
`-XX:MaxRAMPercentage=75.0`. Without it, the JVM on Java 10+ auto-
detects the container memory limit (correct), but defaults the heap
to 25% of that. For a 4GB container: 1GB heap. Most applications
need 70-80% of container memory as heap. The remaining 20-25% is for:
native thread stacks, code cache (JIT compiled code), metaspace,
and operating overhead.

---

**Q6 (jlink custom JRE): When would you use jlink to create a custom JRE?**

A: `jlink` creates a minimal JRE containing only the Java modules your
application uses, reducing image size.

```bash
# Analyze what modules your app needs:
jdeps --ignore-missing-deps --print-module-deps target/app.jar
# Output: java.base,java.sql,java.logging,java.xml

# Create custom JRE:
jlink \
  --module-path $JAVA_HOME/jmods \
  --add-modules java.base,java.sql,java.logging,java.xml \
  --strip-debug \
  --compress 2 \
  --no-header-files \
  --no-man-pages \
  --output custom-jre/

# Result: ~50MB custom JRE vs 300MB+ full JDK
# Use in Dockerfile:
# FROM eclipse-temurin:21 AS builder
# COPY custom-jre/ /opt/jre/
# FROM debian:slim
# COPY --from=builder /opt/jre /opt/jre
# RUN export PATH="/opt/jre/bin:$PATH"
```

> **Code walkthrough:** This RUN export PATH="/opt/jre/bin:$PATH" example demonstrates shell script pattern using container. **KEY MECHANISM:** the shell executes commands sequentially; pipes pass stdout of one command to stdin of the next. **WHY IT MATTERS:** unquoted variables with spaces cause word splitting - IFS splits the value into multiple arguments. **TAKEAWAY: always double-quote variables: "$VAR"; use [[ ]] instead of [ ] for safer conditionals.**

Use cases: container images (smaller base image), embedded devices,
CLI tools distributed as executables.

*What separates good from great:* `jlink` only works if your application
and ALL its dependencies declare module-info (are "modularized"). Most
libraries still use the classpath (unnamed module). In practice: only
applications with zero third-party dependencies (unlikely) or applications
using only well-modularized libraries can use jlink fully. GraalVM
Native Image is often a more practical solution for "small deployable"
requirements.

---

**Q7 (OpenJDK governance): How is OpenJDK governed and what does
that mean for production usage?**

A: OpenJDK is governed by the OpenJDK project under the Java Community
Process (JCP). Oracle is the primary contributor but the project has
contributions from Red Hat, SAP, Google, Azul, and others.

Release model (since Java 10):
- New Java feature release every 6 months (March and September)
- LTS releases: Java 11, 17, 21 (every 3 years minimum, starting to be
  every 2 years)
- Non-LTS releases receive patches only for 6 months (the next release)
- LTS releases receive security patches for years (varies by distribution)

Production guidance:
- Use LTS versions in production (11, 17, or 21 as of 2024)
- Non-LTS versions are for experimentation and preview features
- Each distribution has its own LTS support timeline

*What separates good from great:* The 6-month release cadence means
Java features now reach production faster. Preview features (annotations
@Preview) are available to test but may change before final release.
Important previews to track: Structured Concurrency, String Templates,
Project Loom completion, Project Valhalla (value types). Following
the JEP (Java Enhancement Proposal) list lets you plan for upcoming
language changes.

---

## Java Version History and LTS Releases


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| LTS selection for new project | 60 seconds |
| Java 8 to 11 migration | 2-3 minutes |
| Key features per LTS | 2-3 minutes |
| Java 21 new features | 2-3 minutes |
| Records and sealed | 2-3 minutes |
| Future of Java (roadmap) | 2-3 minutes |
| Java version in Docker | 2-3 minutes |

---

**Q1 (LTS selection): Which Java LTS version should you use for a
new project started today?**

A: Java 21 (LTS, released September 2023) for new projects.

Justification:
- Virtual threads (GA): simplifies high-concurrency I/O code
- Records (final since Java 16): immutable data classes without boilerplate
- Sealed classes (final since Java 17): closed inheritance hierarchies
- Pattern matching `switch` (preview in 21, final in 21): exhaustive type handling
- Sequenced Collections: consistent order API for List, Set, Deque
- Support: Eclipse Temurin 21 LTS until at least 2031

Migration effort from Java 17: minimal. Java 21 is backward compatible
with Java 17 code (as usual within minor LTS upgrades).

From Java 11: slightly more effort (sealed classes and records are new),
but the upgrade is straightforward for application code.

*What separates good from great:* In enterprise, organizational policy
often lags the latest LTS by 1-2 years. A project started today may
be on Java 17 by organizational policy. The strategic argument for
Java 21 is virtual threads: scaling to 100K concurrent requests without
reactive code is a significant operational simplification. Quantify
the value: "at our concurrency level, virtual threads save X% infrastructure
cost vs. reactive migration".

---

**Q2 (Java 8 to 11 migration): What are the main migration challenges
from Java 8 to Java 11?**

A: Key breaking changes from Java 8 to 11:

**1. Module system encapsulation (most common issue):**
Java 9 encapsulated JDK internal packages. Code using `sun.misc.Unsafe`,
`sun.nio.ch.*`, or `com.sun.*` needs `--add-opens`:
```bash
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/sun.nio.ch=ALL-UNNAMED
```
> **Code walkthrough:** This Upgrade to version of library that supports your Java version example demonstrates shell script pattern. **KEY MECHANISM:** the shell executes commands sequentially; pipes pass stdout of one command to stdin of the next. **WHY IT MATTERS:** unquoted variables with spaces cause word splitting - IFS splits the value into multiple arguments. **TAKEAWAY: always double-quote variables: "$VAR"; use [[ ]] instead of [ ] for safer conditionals.**

Spring Boot 2+ adds these automatically.

**2. Removed classes:**
- `javax.xml.ws.*`, `javax.xml.bind.*` (JAXB) moved to separate JAR
- `javafx.*` removed from JDK (now separate OpenJFX)
- `corba`, `applet`, `rmi.activation` modules removed

**3. `javax` to `jakarta` namespace (not Java 11, but Java EE 9+):**
When migrating to Jakarta EE, all `javax.*` imports become `jakarta.*`.
This is a Spring Boot 3.x migration concern (requires Spring Boot 2 -> 3).

**4. `--illegal-access` removed (Java 17):**
The migration workaround `--illegal-access=permit` was removed in Java 17.

**5. Test: run with OpenJDK 11, add `--add-opens` for any warnings.**

*What separates good from great:* The most common hidden issue is
`ReflectionHelper` patterns in frameworks. Run with `--illegal-access=warn`
on Java 11 to log all reflection violations. Address each one either
by updating the library version (modern versions are module-compatible)
or adding explicit `--add-opens`. Document all `--add-opens` in your
JVM startup arguments - they indicate technical debt to address.

---

**Q3 (Key features per LTS): Name the most interview-relevant features
introduced in Java 11, 17, and 21.**

A:

**Java 11 (2018):**
- `var` in lambda parameters: `(var s) -> s.length()`
- New `String` methods: `strip()`, `isBlank()`, `repeat()`, `lines()`
- `Files.readString()` and `Files.writeString()` for simple file I/O
- New `HttpClient` (non-blocking, HTTP/2 support)
- `Optional.isEmpty()`

**Java 17 (2021):**
- Records (final): `record Point(int x, int y) {}`
- Sealed classes (final): `sealed interface Shape permits Circle, Rect {}`
- Pattern matching `instanceof` (final): `if (x instanceof String s) ...`
- Text blocks (final since Java 15): multiline strings with `"""`
- Strong encapsulation of JDK internals (no `--illegal-access=permit`)

**Java 21 (2023):**
- Virtual threads (GA): `Thread.ofVirtual().start(() -> ...)`
- Sequenced Collections: `getFirst()`, `getLast()`, `reversed()` on collections
- Record patterns: `case Point(int x, int y) -> x + y`
- Pattern matching switch (final): switch on any type with patterns
- String templates (preview): `"Hello, \{name}!"` interpolation

*What separates good from great:* Records are the most interview-
significant Java 17 feature for application developers. They reduce
DTO/Value Object boilerplate from ~30 lines to 1 line. Understand:
records are implicitly final, all fields are final (immutable), and
they cannot extend other classes (only implement interfaces). This
makes them ideal for Command/Query objects, API request/response
DTOs, and domain value objects.

---

**Q4 (Java 21 new features): Walk through the most important Java 21
features for production backends.**

A:

**1. Virtual threads (Loom, GA):**
```java
// Before: platform thread pool, limited by thread count
ExecutorService pool = Executors.newFixedThreadPool(200);

// After: virtual thread per task, millions possible
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (var req : requests) {
        executor.submit(() -> processRequest(req));
    }
}
// Each virtual thread is ~2KB; 1M concurrent requests = ~2GB
// vs 1M platform threads = ~1TB (infeasible)
```

> **Code walkthrough:** This Upgrade to version of library that supports your Java version example demonstrates thread pool management using thread pool. **KEY MECHANISM:** the pool maintains a work queue; submitted tasks block until a thread is free. **WHY IT MATTERS:** unconfigured pool sizes exhaust threads under load or waste memory at rest. **TAKEAWAY: always name threads and bound queue size to detect saturation.**

**2. Sequenced Collections:**
```java
SequencedCollection<String> list = new ArrayList<>(List.of("a","b","c"));
list.getFirst();   // "a" - new in Java 21
list.getLast();    // "c" - new in Java 21
list.reversed();   // ["c","b","a"] - reversed view
list.addFirst("z"); // add at front
```

> **Code walkthrough:** This Upgrade to version of library that supports your Java version example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**3. Pattern matching switch (final):**
```java
String desc = switch (shape) {
    case Circle c -> "Circle with radius " + c.radius();
    case Rectangle r -> "Rectangle " + r.w() + "x" + r.h();
    default -> "Unknown shape";
};
```

> **Code walkthrough:** This Upgrade to version of library that supports your Java version example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Virtual threads are not a silver
bullet. They still have the pinning problem (synchronized blocks prevent
unmounting). Code that uses synchronized + blocking I/O will pin the
carrier thread. Migration: replace synchronized blocks with ReentrantLock
in hot I/O paths. JDBC drivers (blocking by design) are the main pinning
concern until JDBC Virtual Thread integration improves.

---

**Q5 (Records and sealed): When would you use records vs regular
classes, and sealed interfaces vs enums?**

A:

**Records vs regular classes:**
Use records for: immutable data holders, DTOs, value objects, command/
response objects, simple tuples.
Use regular classes for: mutable state, classes that extend other classes,
complex behavior, entities with lifecycle.


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// GOOD: record for DTO
record ApiResponse(int status, String body, Instant timestamp) {}

// AVOID: record for mutable entity
// records are immutable - cannot be JPA @Entity (needs setters)
// BAD:
@Entity
record User(Long id, String name) {} // doesn't work with JPA
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

**Sealed interfaces vs enums:**
Enums: type = singleton value; limited to pre-defined constants.
Sealed interfaces: type = class hierarchy; each branch can have different fields.

```java
// ENUM: when all variants have same structure
enum Direction { NORTH, SOUTH, EAST, WEST }

// SEALED: when variants have different data
sealed interface Result<T> permits Success, Failure {}
record Success<T>(T value) implements Result<T> {}
record Failure<T>(String error) implements Result<T> {}

// Exhaustive switch (compiler verifies all cases):
String msg = switch (result) {
    case Success<String> s -> "Got: " + s.value();
    case Failure<String> f -> "Error: " + f.error();
};
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Sealed interfaces + records +
pattern matching switch form a "functional union type" pattern in
Java. This enables algebraic data types (ADTs) that Scala and Haskell
have had for years. The exhaustive switch compiler check means: if you
add a new branch to the sealed hierarchy, every switch on that type
will fail to compile until the new case is handled. This is a powerful
refactoring safety net.

---

**Q6 (Future of Java roadmap): What major features are coming in
future Java versions?**

A:

**Project Loom (ongoing):**
- Virtual threads (GA in Java 21) - done
- Structured Concurrency (preview in Java 21, evolving) - near stable
- Scoped Values (replaces InheritableThreadLocal, preview) - evolving

**Project Valhalla (most anticipated):**
- Value types: classes with value semantics (no identity, no null,
  passed by value like primitives). Eliminates boxing of int/long.
- Generic specialization: `List<int>` instead of `List<Integer>`.
- Impacts all Java performance - eliminates major boxing overhead.
- Expected: Java 23+ (still in development).

**Project Panama:**
- Foreign Function & Memory API (GA in Java 22): safe, efficient native
  memory access and calling native libraries without JNI.
- Replacing `sun.misc.Unsafe` for performance-sensitive code.

**Project Amber (language features, ongoing):**
- String templates (preview in Java 21): `"Hello, \{name}!"`
- Unnamed patterns and variables: `_` for unused variables
- Unnamed classes and instance main methods (simplify beginner programs)

*What separates good from great:* Project Valhalla is the most
architecturally significant upcoming change. Value types will make
primitive-backed generics (`List<int>`) possible, eliminating a major
source of boxing overhead and memory bloat. The change will require
library updates (Collections framework, Stream API) and may be the
most impactful change to Java's performance model since generics in
Java 5.

---

**Q7 (Java version in Docker): How do you specify the Java version in
a Docker image?**

A:

```dockerfile
# BAD: anti-pattern shown for contrast
# This approach has the issues the GOOD example fixes
```

```dockerfile
# GOOD: use specific LTS + specific minor version for reproducibility
FROM eclipse-temurin:21.0.2_13-jdk-jammy AS builder
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests

# Runtime: smaller image (JRE suffix or jre tag)
FROM eclipse-temurin:21.0.2_13-jre-jammy
WORKDIR /app
COPY --from=builder /app/target/app.jar .

# Essential JVM flags for containers:
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0 \
               -XX:+UseZGC \
               -XX:+ExitOnOutOfMemoryError \
               -XX:HeapDumpPath=/var/log/ \
               -XX:+HeapDumpOnOutOfMemoryError"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

> **Code walkthrough:** GOOD pattern: This Essential JVM flags for containers: example demonstrates a key concept in practice using container. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Key practices:
- Pin to specific minor version (not just `21`) for reproducibility
- Use `jre` tag where available (smaller than `jdk`)
- Set `MaxRAMPercentage` to match your memory allocation
- Use `ExitOnOutOfMemoryError` to fail fast and let Kubernetes restart

*What separates good from great:* Distroless images (`gcr.io/distroless/java21`)
are the most secure option - no shell, no package manager, minimal attack
surface. But they make exec-into-container debugging impossible. The
trade-off: production images use distroless, staging/dev images use JDK
with shell for debugging. A two-stage Dockerfile with `--target` argument
can build both from the same Dockerfile.

---


# Java Core - L1 OOP Basics

## Inheritance and Polymorphism

---

### 🎯 Model Answer

**30 seconds:**
> Inheritance lets a subclass extend a parent class, inheriting its
> fields and methods. Polymorphism lets the same method call behave
> differently based on the actual runtime type. Java supports single
> class inheritance (`extends`) but multiple interface implementation
> (`implements`). Method overriding (same signature, different class)
> enables runtime polymorphism. Method overloading (same name, different
> parameters) is compile-time polymorphism. The `final` keyword prevents
> inheritance (class) or overriding (method).

**3 minutes (Senior):**
> Polymorphism via virtual dispatch: the JVM uses a virtual method table
> (vtable) to resolve method calls at runtime. When you call `animal.speak()`,
> the JVM looks up the actual type of `animal` in the vtable and calls
> the appropriate implementation. This enables programming to interfaces
> (dependency inversion).
>
> Liskov Substitution Principle (LSP): subclasses must be substitutable
> for their parent class without breaking correctness. Violating LSP
> (a Square extending Rectangle breaks width/height invariant) is a
> classic design error. Prefer composition over inheritance to avoid
> tight coupling.
>
> The `Object` class is the root of all Java classes. It provides:
> `equals()`, `hashCode()`, `toString()`, `clone()`, `getClass()`,
> `finalize()` (deprecated), `wait()`, `notify()`, `notifyAll()`.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Inheritance vs composition | 2 minutes |
| Virtual dispatch mechanism | 2 minutes |
| LSP explanation | 2 minutes |
| Overriding vs overloading | 90 seconds |
| covariant return type | 90 seconds |
| Constructor and inheritance | 2 minutes |
| final and abstract | 60 seconds |

---

**Q1 (Inheritance vs composition): When would you prefer composition
over inheritance?**

A: The rule: "prefer composition over inheritance" (Effective Java, Item 18).

**Use composition when:**
- The relationship is "has-a", not "is-a"
- You want to reuse behavior without inheriting the full interface
- The parent class may change (fragile base class problem)
- You need to use multiple implementations of the same behavior

**Use inheritance when:**
- The subclass genuinely IS the parent type (behavioral contract holds)
- You're extending abstract classes in frameworks (Template Method pattern)
- Override-and-extend is semantically correct

```java
// COMPOSITION: LoggingList wraps List, doesn't extend it:
class LoggingList<T> {
    private final List<T> delegate;
    LoggingList(List<T> list) { this.delegate = list; }
    boolean add(T item) {
        log("Adding: " + item);
        return delegate.add(item);
    }
    // forward other methods...
}
// If ArrayList adds a new method, LoggingList won't accidentally
// inherit it without logging.
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The "fragile base class" problem is
the core reason to prefer composition. If you extend a class and the
parent adds a new method that you haven't overridden, callers get the
parent's implementation - possibly breaking your invariants. Example:
if you extend `HashSet` to count insertions and the parent adds a new
bulk-add method that internally calls `add()`, your count stays correct.
But if it calls a different internal method, your count breaks.
Composition: you control ALL method delegations.

---

**Q2 (Virtual dispatch): How does the JVM implement virtual method
dispatch?**

A: The JVM implements virtual dispatch via virtual method tables (vtables).

For each class, the JVM creates a vtable: a table of method pointers,
one per virtual method. Subclasses inherit the parent's vtable and
override entries for their overridden methods.

```plaintext
Animal vtable:
  [0] speak -> Animal.speak
  [1] toString -> Object.toString
  ...

Dog vtable (extends Animal):
  [0] speak -> Dog.speak  // OVERRIDDEN
  [1] toString -> Object.toString
  ...

Call a.speak() where a is declared Animal:
  1. Load actual object reference (points to Dog vtable)
  2. Look up vtable[0] = Dog.speak
  3. Call Dog.speak
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

JIT optimization: JVM profiles call sites. If a call site always sees
the same type (monomorphic), JIT inlines the method directly - no vtable
lookup overhead.

*What separates good from great:* JIT inline caching makes polymorphism
nearly free in hot code paths. But megamorphic call sites (3+ different
types seen at one call site) cannot be inlined and have higher overhead.
In performance-critical loops, monomorphic dispatch (always the same
concrete type) is fastest. Micro-benchmarks that compare interface
calls to direct calls often see no difference because JIT inlining
eliminates the dispatch overhead.

---

**Q3 (LSP): Explain the Liskov Substitution Principle with a Java example.**

A: LSP: if `S` is a subtype of `T`, then objects of type `T` may be
replaced by objects of type `S` without altering the correctness of
the program.

Behavioral definition: a subclass must honor all behavioral contracts
of the parent class: preconditions cannot be strengthened, postconditions
cannot be weakened, invariants must be preserved.

```java
// LSP violation: ReadOnlyList extends ArrayList
// Parent contract: add() adds an element and returns true
// Child behavior: add() throws UnsupportedOperationException
class ReadOnlyList<T> extends ArrayList<T> {
    @Override
    public boolean add(T e) {
        throw new UnsupportedOperationException(); // breaks parent contract
    }
}

// Code that uses ArrayList:
void addItem(ArrayList<String> list, String item) {
    list.add(item); // throws for ReadOnlyList!
}
// ReadOnlyList cannot substitute for ArrayList without breaking code

// FIX: ReadOnlyList should implement List, not extend ArrayList
// Better: use Collections.unmodifiableList() - designed for this
List<String> readOnly = Collections.unmodifiableList(mutableList);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Java's `Collections.unmodifiableList()`
technically violates LSP for the same reason (throws on mutation), but
it's a documented design choice. The real lesson: throwing
`UnsupportedOperationException` from an interface method indicates a
poorly designed inheritance hierarchy. Modern Java uses `interface +
default methods` with explicit optional capabilities (e.g., `Spliterator`
with optional characteristics) instead of subclassing with unsupported
operations.

---

**Q4 (Overriding vs overloading): Contrast method overriding and
method overloading.**

A:

| Aspect | Overriding | Overloading |
|---|---|---|
| When resolved | Runtime (dynamic) | Compile time (static) |
| Relationship | Subclass changes parent's method | Same class, different params |
| Return type | Same or covariant | Can differ (if signature differs) |
| Exceptions | Can throw less (not more checked) | No restriction |
| `@Override` | Yes (validates) | Not applicable |
| Polymorphism type | Runtime polymorphism | Compile-time polymorphism |

```java
// Overriding: resolved at RUNTIME
Animal a = new Dog();
a.speak(); // Dog.speak() called - runtime type matters

// Overloading: resolved at COMPILE TIME
class Printer {
    void print(int n) { System.out.println("int: " + n); }
    void print(double d) { System.out.println("double: " + d); }
}
Printer p = new Printer();
p.print(5);    // print(int) - compiler sees int literal
p.print(5.0);  // print(double) - compiler sees double literal
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* A tricky overloading case: varargs
methods. `print(String... args)` is called when no specific overload
matches. Autoboxing and varargs interact in surprising ways:
`print(1, 2, 3)` matches `print(int...)` before `print(Integer...)`.
And overloading with inheritance: if you overload a method in a subclass,
the parent's overloads are ALSO visible. The compiler picks the most
specific applicable overload - can be surprising when the declared
type and actual type differ.

---

**Q5 (Covariant return): What is covariant return type overriding?**

A: In Java 5+, an overriding method can return a subtype of the parent
method's return type. This is covariant return type.

```java
class Animal {
    Animal getInstance() { return new Animal(); }
}

class Dog extends Animal {
    @Override
    Dog getInstance() { // covariant: Dog is a subtype of Animal
        return new Dog();
    }
}

// Usage:
Dog d = new Dog();
Dog result = d.getInstance(); // no cast needed (return type is Dog)

// Without covariant return (old style):
Animal result = d.getInstance(); // had to accept Animal, then cast
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Used extensively in builder patterns and fluent APIs:
```java
abstract class Builder<T extends Builder<T>> {
    abstract T self(); // returns concrete subtype
    T withName(String n) { this.name = n; return self(); }
}
class PersonBuilder extends Builder<PersonBuilder> {
    @Override PersonBuilder self() { return this; }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Covariant return enables cleaner
API design without casting. The `Comparable<T>` pattern and builder
hierarchies commonly use it. The JVM supports covariant return by
creating a bridge method (synthetic) in the bytecode that calls the
actual overriding method - transparent to the developer. In code
reviews: look for unnecessary casts after method calls - often a
sign that covariant return could simplify the code.

---

**Q6 (Constructor and inheritance): How does constructor chaining work
in Java inheritance?**

A: Constructor chaining rules:
1. Every constructor must call either `this(...)` (sibling constructor)
   or `super(...)` (parent constructor) as the FIRST statement.
2. If neither is specified, the compiler inserts `super()` - calling
   the no-arg parent constructor.
3. If the parent has no no-arg constructor, the compiler reports an error.

```java
class Vehicle {
    String type;
    int wheels;
    Vehicle(String type, int wheels) {
        this.type = type;
        this.wheels = wheels;
    }
    // No no-arg constructor!
}

class Car extends Vehicle {
    String model;
    Car(String model) {
        super("car", 4); // MUST call parent constructor explicitly
        this.model = model;
    }
    Car(String model, int extraWheels) {
        this(model); // chains to Car(String model)
    }
}

// Object lifecycle during new Car("Tesla"):
// 1. Object memory allocated
// 2. Car(String) calls super("car", 4)
// 3. Vehicle("car", 4) constructor runs
//    - Vehicle.type = "car", Vehicle.wheels = 4
// 4. Car constructor resumes
//    - Car.model = "Tesla"
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The "call overridable method from
constructor" anti-pattern: if `Vehicle()` calls `this.validate()` and
Dog overrides `validate()`, Dog's validate runs before Dog's fields are
initialized. This causes NPE or incorrect values. Rule: constructors should
only call `private`, `final`, or `static` methods (which cannot be
overridden and therefore behave predictably during construction).

---

**Q7 (final and abstract): Contrast final, abstract, and sealed class
modifiers.**

A:

| Modifier | Class | Method | Field |
|---|---|---|---|
| `final` | Cannot be extended | Cannot be overridden | Cannot be reassigned |
| `abstract` | Cannot be instantiated | No implementation; must override | N/A |
| `sealed` (Java 17) | Limited set of permitted subclasses | N/A | N/A |

```java
// final class: String, Integer, System
final class ImmutablePoint {
    final int x, y; // final fields too - fully immutable
    ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
}

// abstract class: template method pattern
abstract class ReportGenerator {
    // Template method:
    final void generate() { // final: can't override the skeleton
        fetchData();
        String content = formatContent(); // abstract step
        writeOutput(content);
    }
    abstract String formatContent(); // subclasses must implement
    private void fetchData() { ... }
    private void writeOutput(String c) { ... }
}

// sealed class: restricted hierarchy (Java 17)
sealed interface Shape permits Circle, Rectangle, Triangle {}
final class Circle implements Shape { double radius; }
final class Rectangle implements Shape { double w, h; }
final class Triangle implements Shape { double base, height; }
// No other class can implement Shape!
```

> **Code walkthrough:** This Unknown example demonstrates contract definition usice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* Sealed classes enable exhaustive
pattern matching. The compiler verifies that a `switch` on a sealed
type covers all permitted subtypes - no `default` branch needed, and
adding a new subtype forces you to update all switches (compile error).
This is the Java equivalent of ML/Haskell algebraic data types. Use
sealed + records for domain modeling where the set of possible states
is closed and known at design time.

---


## Abstract Classes and Interfaces

---

### 🎯 Model Answer

**30 seconds:**
> Interfaces define a contract (what a class can do) without
> implementation. Abstract classes provide partial implementation
> and force subclasses to complete the rest. Key rule: a class
> can implement many interfaces but extend only one abstract class.
> Java 8 added default methods to interfaces (code in an interface),
> and Java 9 added private methods. Use interface when you want to
> define a capability (Comparable, Serializable, Runnable).
> Use abstract class when you want to share code while enforcing a
> contract (Template Method pattern).

**3 minutes (Senior):**
> Interface vs abstract class decision:
> - Interface: no state (except constants), pure contract, multiple
    >   implementation, type token
> - Abstract class: state (fields), constructors, shared implementation,
    >   template method pattern, one extends only
>
> Java 8 default methods reduced the gap: interfaces can now have code.
> But interfaces still cannot have instance fields or constructors.
> The diamond problem with default methods: if two interfaces both
> define `default void log()`, the implementing class must override
> to resolve ambiguity.
>
> Functional interfaces (exactly one abstract method): work with
> lambdas. Examples: `Runnable`, `Callable`, `Comparator`, `Function`.
> `@FunctionalInterface` annotation validates single abstract method.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Interface vs abstract class | 90 seconds |
| Default method conflicts | 2 minutes |
| Functional interfaces | 2 minutes |
| Template method pattern | 2 minutes |
| Interface segregation | 2 minutes |
| Abstract class constructor | 90 seconds |
| Marker interfaces | 90 seconds |

---

**Q1 (Interface vs abstract): When do you choose interface over
abstract class?**

A:

**Choose Interface when:**
- Defining a capability, not an identity (Runnable, Comparable, Printable)
- Multiple implementations needed from unrelated class hierarchies
- Type token / marker purpose (Serializable, Cloneable)
- Functional interface for lambda use
- API contract that must not restrict the implementor's class hierarchy

**Choose Abstract Class when:**
- Sharing state (instance fields) across subclasses
- Template Method pattern: algorithm skeleton with pluggable steps
- Providing non-trivial default implementations that need state
- Building framework extension points with infrastructure

```java
// Interface: capability (any class can be Comparable)
interface Comparable<T> { int compareTo(T other); }
// String, Integer, Date all implement Comparable - unrelated hierarchies

// Abstract class: is-a with shared state
abstract class AbstractHttpClient {
    private final String baseUrl; // shared state
    protected final HttpSession session = createSession();
    AbstractHttpClient(String baseUrl) { this.baseUrl = baseUrl; }
    abstract Response doRequest(Request req);
    final Response get(String path) {
        return doRequest(new Request("GET", baseUrl + path));
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* The Java SDK itself shows the pattern:
`AbstractList` (abstract class, shared state + template), `List`
(interface, contract). `AbstractList` reduces effort to implement a
custom List by implementing most methods via `get()` and `size()`.
This combination - define interface + provide abstract base - is called
the Interface + Skeletal Implementation pattern (Effective Java, Item 20).
Spring uses it extensively: `AbstractBeanFactory`, `AbstractController`, etc.

---

**Q2 (Default method conflicts): How does Java resolve conflicting
default methods?**

A: Three rules in priority order:

1. **Class or superclass wins:** if a class or any superclass provides
   an implementation, it takes priority over all interface defaults.
2. **More specific interface wins:** if one interface extends another
   and both provide defaults, the more specific (child) interface wins.
3. **Must override:** if neither rule resolves, the class must explicitly
   override.

```java
interface A { default void m() { System.out.println("A"); } }
interface B extends A {
    @Override default void m() { System.out.println("B"); }
}
class C implements A, B {
    // B wins (more specific than A)
}
new C().m(); // prints "B"

interface X { default void m() { System.out.println("X"); } }
interface Y { default void m() { System.out.println("Y"); } }
class D implements X, Y {
    // Compile error: neither X nor Y is more specific
    @Override public void m() {
        X.super.m(); // explicitly delegate to X
        // or:
        // Y.super.m(); // or to Y
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Default methods were added for API
evolution: existing interfaces could add new methods without breaking
all existing implementations (e.g., `Collection.forEach()`,
`Map.computeIfAbsent()`). The diamond conflict problem is rare in
practice because most interfaces in the same type hierarchy share
ancestry. The conflict only arises with truly unrelated interfaces
providing the same default method - which is usually an API design
mistake.

---

**Q3 (Functional interfaces): Explain functional interfaces and
their role with lambdas.**

A: A functional interface is an interface with exactly one abstract
method (SAM - Single Abstract Method). Default methods and static
methods don't count. The `@FunctionalInterface` annotation is
optional but recommended (compiler validates SAM requirement).

```java
@FunctionalInterface
interface Validator<T> {
    boolean validate(T value);           // single abstract method
    default Validator<T> and(Validator<T> other) { // default - ok
        return v -> this.validate(v) && other.validate(v);
    }
    static <T> Validator<T> notNull() {  // static - ok
        return v -> v != null;
    }
}

// Lambda = anonymous implementation of functional interface:
Validator<String> notEmpty = s -> !s.isEmpty();
Validator<String> notTooLong = s -> s.length() <= 100;
Validator<String> combined = notEmpty.and(notTooLong);

combined.validate("");       // false
combined.validate("hello");  // true
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

Standard functional interfaces in `java.util.function`:
- `Function<T,R>`: T -> R (transform)
- `Consumer<T>`: T -> void (side effect)
- `Supplier<T>`: () -> T (produce)
- `Predicate<T>`: T -> boolean (test)
- `BiFunction<T,U,R>`: (T,U) -> R (two inputs)
- `UnaryOperator<T>`: T -> T (same type in/out)

*What separates good from great:* Functional interface composition is
a powerful pattern. `Predicate.and()`, `Predicate.or()`, `Function.andThen()`,
`Function.compose()` build pipelines without creating named classes.
But over-composing function objects creates opaque call chains that are
hard to debug. Rule: use function composition for simple transformations;
use named methods for complex business logic (better stack traces,
easier testing).

---

**Q4 (Template method pattern): Where is the Template Method pattern
used in Java's standard library?**

A: Template Method: define the algorithm skeleton in a base class
(often `final`), let subclasses fill in specific steps.

**Java stdlib examples:**
- `AbstractList.sort()`: uses the list's own element comparison, calls
  `listIterator()` which subclasses provide
- `Thread.run()`: the `Thread` class's `run()` calls `target.run()` if
  set, or the subclass overrides `run()` directly
- `HttpServlet.service()`: dispatches to `doGet()`, `doPost()` etc.
  which subclasses override
- `AbstractQueuedSynchronizer (AQS)`: `acquire()` calls `tryAcquire()`
  which ReentrantLock, Semaphore, CountDownLatch override

```java
// Spring's Template Method: JdbcTemplate
public class JdbcTemplate {
    // Template method:
    public <T> T query(String sql, ResultSetExtractor<T> rse) {
        // acquire connection, create statement, execute, handle errors
        // ... boilerplate ...
        return rse.extractData(resultSet); // your code here
    }
}
// User provides only the extraction logic (lambda):
String name = jdbc.query(
    "SELECT name FROM user WHERE id = ?",
    rs -> rs.getString("name"),
    id
);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using SQL. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The Template Method pattern captures
the "Hollywood Principle" - don't call us, we'll call you. The framework
controls the flow; the application code provides customization points.
This is the foundation of Spring, Hibernate, and most Java frameworks.
Understanding Template Method helps you understand why framework classes
use `abstract` and why you override specific methods rather than writing
top-level code.

---

**Q5 (Interface segregation): What is Interface Segregation Principle
and how does Java support it?**

A: Interface Segregation Principle (ISP): clients should not be forced
to depend on interfaces they don't use. Large "fat" interfaces should
be split into smaller, focused interfaces.

```java
// FAT interface - violates ISP:
interface Employee {
    void work();
    void takeBreak();
    void attendMeeting();
    void writeTimesheet();
    void performReview(); // managers only!
    void approveBudget(); // finance only!
}

// SEGREGATED interfaces - obey ISP:
interface Worker { void work(); }
interface Reviewer { void performReview(); }
interface BudgetApprover { void approveBudget(); }

class Developer implements Worker, Reviewer { ... }
class FinanceManager implements Worker, Reviewer, BudgetApprover { ... }
// Each class implements only what it needs
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

Java naturally supports ISP: multiple interface implementation means you
can compose small interfaces without forcing unrelated implementations.
Functional interfaces (single method) are the extreme case of ISP.

*What separates good from great:* Applying ISP requires identifying
"role interfaces" vs "header interfaces". A role interface captures
a specific behavior (Auditable, Printable); a header interface captures
all methods of a class (a 1:1 mapping). Role interfaces (ISP) enable
better decoupling. The `java.util.Collection` hierarchy is a good example:
`Iterable` (just iterate), `Collection` (add/remove), `List` (indexed),
`RandomAccess` (marker for efficient indexed access) - each additional
interface adds one capability.

---

**Q6 (Abstract class constructor): Can abstract classes have constructors
and what is their purpose?**

A: Yes. Abstract classes can and should have constructors. They cannot
be called directly (`new AbstractClass()` is a compile error) but are
called by subclass constructors via `super(...)`.

Purpose: initialize the shared state (fields) that the abstract class
defines and subclasses inherit.

```java
abstract class Shape {
    private final String color;
    private final boolean filled;

    // Constructor initializes shared state:
    protected Shape(String color, boolean filled) {
        this.color = Objects.requireNonNull(color);
        this.filled = filled;
    }

    public String getColor() { return color; }
    public boolean isFilled() { return filled; }
    public abstract double area();
}

class Circle extends Shape {
    private final double radius;
    Circle(String color, boolean filled, double radius) {
        super(color, filled); // must call abstract class constructor
        this.radius = radius;
    }
    @Override public double area() { return Math.PI * radius * radius; }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Abstract class constructors can
enforce invariants for ALL subclasses. `Objects.requireNonNull(color)`
in `Shape`'s constructor means NO subclass can create a shape with
null color - the check runs before any subclass code. This is a powerful
way to enforce domain invariants centrally. Contrast with interfaces:
no constructor, no shared state, no centralized invariant enforcement.

---

**Q7 (Marker interfaces): What is a marker interface and when should
you use one vs annotations?**

A: A marker interface has no methods - it exists only to tag a class
as having some property.

Classic Java marker interfaces:
- `Serializable`: marks a class as safe to serialize
- `Cloneable`: marks a class as supporting `clone()` (poorly designed)
- `RandomAccess`: marks a List as supporting fast random access

```java
// Marker interface approach:
interface Cacheable {}
class UserDTO implements Cacheable { ... }
class ProductDTO implements Cacheable { ... }

// Type-safe check:
if (obj instanceof Cacheable) {
    cacheService.cache(obj);
}
// Can be used as generic type bound:
<T extends Cacheable> void cache(T obj) { ... }
```

> **Code walkthrough:** This Unknown example demonstrates contract definition usice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

**Marker interface vs annotation:**
|               | Marker Interface     | Annotation                     |  
|-------------|--------------------|------------------------------|
| Type check    | `instanceof`         | reflection or framework        |  
| Generic bound | `<T extends Marker>` | not possible                   |  
| Applies to    | Classes              | Classes, methods, fields, etc. |  
| Inherited     | Yes                  | `@Inherited` needed            |  
| Retention     | Compile + runtime    | Configurable                   |

Prefer annotations when: marking methods, fields, or constructors;
specifying metadata (configuration values); the mark is checked by
a framework at compile or annotation processor time.
Prefer marker interfaces when: compile-time type safety is needed;
the mark should be constrainable with generics.

*What separates good from great:* `Serializable` is a famous poor
example. It's a marker with no methods, but serialization behavior
depends on `serialVersionUID`, `readObject()`, `writeObject()` -
none of which are in the interface. Java can't verify that a class
actually serializes correctly just from the marker. Annotations with
retention `RUNTIME` are almost always clearer and more powerful
for framework metadata. Modern Java uses annotations extensively
(JPA `@Entity`, Spring `@Component`) and avoids marker interfaces
for new APIs.

---

## Exception Handling Basics

### 🎯 Model Answer


**30 seconds:**
> Java exceptions are objects that represent error conditions. Checked
> exceptions (extend Exception) must be declared in `throws` or caught -
> the compiler enforces this. Unchecked exceptions (extend RuntimeException)
> need not be declared. `try-catch-finally` handles exceptions; `finally`
> always runs. `try-with-resources` (Java 7) automatically closes
> `AutoCloseable` resources. Key rule: catch specific exception types,
> not bare `Exception`. Log the root cause, don't swallow exceptions.

**3 minutes (Senior):**
> Checked vs unchecked debate: checked exceptions force callers to handle
> errors, documenting failure modes in the API. But they pollute method
> signatures and are incompatible with lambdas (can't throw checked
> exceptions from `Function<T,R>`). Modern Java consensus: use unchecked
> (RuntimeException) for most application code; checked only for recoverable
> conditions where the caller MUST handle (e.g., `IOException` for files).
>
> Exception chaining: always wrap exceptions with context when re-throwing.
> `throw new ServiceException("Failed to load user " + id, e)` preserves
> the original cause. Lost cause = lost debugging context.
>
> Try-with-resources handles `Closeable`/`AutoCloseable` objects. Suppressed
> exceptions: if both the try body AND the close() throw, the close()
> exception is suppressed (added to the primary exception). Check
> `e.getSuppressed()` in debugging.



---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Checked vs unchecked | 2 minutes |
| try-with-resources | 2 minutes |
| Exception hierarchy design | 2-3 minutes |
| Suppressed exceptions | 90 seconds |
| Exception chaining | 90 seconds |
| finally guarantees | 2 minutes |
| Multi-catch syntax | 60 seconds |

---

**Q1 (Checked vs unchecked): When should you use checked vs unchecked
exceptions?**

A: The question of when to use checked vs unchecked is one of Java's
most debated design topics.

**Use CHECKED exceptions when:**
- The caller CAN and SHOULD handle the exception
- Recovery is possible and meaningful
- The failure is part of the normal contract (not a bug)
- Classic examples: `IOException` (file may not exist - check first),
  `InterruptedException` (thread interruption is expected)

**Use UNCHECKED exceptions when:**
- The error is a programming bug (NPE, illegal argument)
- The caller cannot meaningfully recover
- The code is used in lambda/stream contexts
- The failure is infrastructure/unexpected (DB connection dropped)

**Modern practice:**
```java
// Checked: file may not exist - callers should handle
public byte[] readConfig(Path path) throws IOException { ... }

// Unchecked: invalid argument is a programming error
public void setAge(int age) {
    if (age < 0) throw new IllegalArgumentException(
        "Age must be non-negative: " + age);
    // no throws declaration needed
}

// Spring/framework pattern: wrap checked as unchecked
try {
    return jdbcTemplate.queryForObject(sql, String.class, id);
} catch (DataAccessException e) { // Spring's unchecked wrapper
    throw new UserNotFoundException("User not found: " + id, e);
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The lambda incompatibility is the
strongest argument against checked exceptions for modern code. A
`Function<String, Integer>` cannot throw `ParseException` without
a wrapper. Many codebases use a utility:
`@FunctionalInterface interface ThrowingFunction<T,R> { R apply(T t) throws Exception; }`
with a static `wrap(ThrowingFunction)` adapter. This is boilerplate
that checked exceptions force upon functional programming. Kotlin and
C# made all exceptions unchecked for this reason.

---

**Q2 (try-with-resources): How does try-with-resources work and what
is the suppressed exception mechanism?**

A: `try-with-resources` (Java 7, JEP 334) automatically calls `close()`
on resources declared in the try header, in reverse declaration order.

```java
try (Resource1 r1 = ...; Resource2 r2 = ...) {
    // body
} // calls: r2.close(), then r1.close()
  // regardless of whether body threw or returned normally
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

**Suppressed exception scenario:**
```java
try (var conn = openConnection()) {
    throw new BusinessException("Bad data"); // primary exception
    // conn.close() is called; if it also throws:
    // ConnectionCloseException is SUPPRESSED
}
// Result: catch gets BusinessException
// BusinessException.getSuppressed() = [ConnectionCloseException]
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

Examining suppressed:
```java
try {
    ...
} catch (BusinessException e) {
    log.error("Primary: {}", e.getMessage(), e);
    for (Throwable suppressed : e.getSuppressed()) {
        log.error("Suppressed: {}", suppressed.getMessage(), suppressed);
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

Classes implementing `AutoCloseable` (not `Closeable`) can throw any
exception from `close()`. `Closeable` restricts to `IOException`.

*What separates good from great:* Suppressed exceptions are frequently
ignored in debugging. A `ConnectionCloseException` being silently
swallowed means the developer doesn't notice a connection leak.
Always log `getSuppressed()` in global exception handlers / logging
advice. Frameworks like Spring AOP can add cross-cutting logging of
suppressed exceptions. Also: `close()` should not throw for idempotency;
trying to close an already-closed resource should be a no-op, not an
exception.

---

**Q3 (Exception hierarchy design): How do you design a domain exception
hierarchy?**

A: A well-designed exception hierarchy provides:
- Catch at the right level of abstraction
- Specific handling for specific failures
- Generic handling for the entire domain

```java
// Domain exception hierarchy:
public class OrderException extends RuntimeException {
    // catch-all for order failures
    public OrderException(String message) { super(message); }
    public OrderException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class PaymentException extends OrderException {
    private final String paymentId;
    public PaymentException(String paymentId, String message, Throwable cause) {
        super(message, cause);
        this.paymentId = paymentId;
    }
    public String getPaymentId() { return paymentId; }
}

public class InsufficientFundsException extends PaymentException {
    private final BigDecimal available;
    private final BigDecimal required;
    // carries diagnostic context
}

// Catching at different levels:
try {
    orderService.processOrder(order);
} catch (InsufficientFundsException e) {
    return ResponseEntity.status(402).body("Insufficient funds: " + e.getRequired());
} catch (PaymentException e) {
    return ResponseEntity.status(402).body("Payment failed: " + e.getPaymentId());
} catch (OrderException e) {
    return ResponseEntity.status(500).body("Order processing failed");
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* Include diagnostic context in custom
exceptions: not just a message but the IDs, values, and state that
caused the failure. `InsufficientFundsException` carrying `available`
and `required` amounts means the error handler can generate precise
user messages and can be logged with full context without re-fetching
data. Exception classes are part of the API contract; changing them
is a breaking change. Design exception hierarchies as carefully as
regular class hierarchies.

---

**Q4 (Suppressed exceptions): When would you need to check getSuppressed()?**

A: Suppressed exceptions occur when: the try body throws, AND a
`close()` call in try-with-resources ALSO throws. The close exception
is attached to the primary exception as suppressed.

Scenarios where this matters in production:
1. **Database connection close fails:** if `close()` fails after a
   query exception, you miss the connection leak.
2. **File close fails:** OS file handle not released properly.
3. **Network socket close throws:** underlying network issue.

```java
// Utility to log all exceptions including suppressed:
static void logFullException(Logger log, Exception e) {
    log.error("Primary exception: ", e);
    Throwable[] suppressed = e.getSuppressed();
    for (int i = 0; i < suppressed.length; i++) {
        log.error("Suppressed[{}]: ", i, suppressed[i]);
    }
    if (e.getCause() != null) {
        log.error("Cause: ", e.getCause());
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Spring's `DataAccessUtils` and most framework exception translators
preserve suppressed exceptions in their wrapping.

*What separates good from great:* In production incident analysis,
missing the suppressed exception can lead to wrong root cause analysis.
If the DB query failed AND the connection close failed (indicating
connection pool exhaustion or network partition), both pieces of
information are needed. A good error reporting framework logs the
full exception chain including suppressed exceptions and causes.
OpenTelemetry's exception recording includes suppressed by default.

---

**Q5 (Exception chaining): Why is exception chaining important and how
do you use it?**

A: Exception chaining (introduced in Java 1.4) preserves the original
exception as the "cause" of a wrapping exception. The full chain appears
in the stack trace: `Caused by: ...`.

```java
// Layer 1: DAO throws specific exception
public User findUser(long id) {
    try {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id=?", USER_MAPPER, id);
    } catch (EmptyResultDataAccessException e) {
        throw new UserNotFoundException(
            "User not found: " + id, e); // chain preserved
    }
}

// Layer 2: Service wraps with business context
public UserProfile getProfile(long userId) {
    try {
        User user = userDao.findUser(userId);
        return buildProfile(user);
    } catch (UserNotFoundException e) {
        throw new ProfileLoadException(
            "Cannot load profile for userId=" + userId, e);
    }
}

// Stack trace shows:
// ProfileLoadException: Cannot load profile for userId=42
//   at UserService.getProfile(UserService.java:45)
// Caused by: UserNotFoundException: User not found: 42
//   at UserDAO.findUser(UserDAO.java:23)
// Caused by: EmptyResultDataAccessException: ...
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using SQL. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* Exception messages should include
the DATA that was being processed, not just the operation. "User not
found" is less useful than "User not found: id=42" when scanning logs
for a specific customer's issue. The message is your primary debugging
tool when you have a thread dump but no debugger. Treat exception
messages as your first line of log evidence.

---

**Q6 (finally guarantees): What are the guarantee and non-guarantee
of finally blocks?**

A:

**finally ALWAYS runs in:**
- Normal completion of try block
- Exception in try block (whether caught or not)
- `return` statement in try or catch block
- `break`/`continue` in try block

**finally does NOT run when:**
- `System.exit()` is called (JVM terminates)
- JVM is killed (kill -9, power off)
- Infinite loop in try block (finally waits but runs eventually if break)
- `Runtime.halt()` or `Runtime.getRuntime().halt(0)`

```java
// finally with return (tricky):
int getValue() {
    try {
        return 1;     // sets return value to 1
    } finally {
        return 2;     // OVERRIDES return 1! Returns 2
    }
}
// getValue() returns 2
// This is a code smell - never return from finally
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

**Prefer try-with-resources over finally for resources:**

```java
// BAD: null check without Optional
User user = findUser(id);
if (user != null) {
    return user.getName();
}
return null; // callers must null-check return value
```

```java
// BAD: manual finally, verbose, error-prone:
Connection conn = null;
try {
    conn = getConnection();
    // ...
} finally {
    if (conn != null) {
        try { conn.close(); } catch (SQLException e) { /* suppressed */ }
    }
}

// GOOD: try-with-resources handles this correctly:
try (Connection conn = getConnection()) {
    // ...
}
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **WHAT BREAKS: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The `return` from `finally` override
is a famous Java gotcha. If `finally` returns a value, it silently
overrides the try/catch's return or exception. This can hide exceptions:
if the try body throws an exception and the finally block returns
normally, the exception is LOST. Code review checklist: flag any
`return` statement inside a `finally` block.

---

**Q7 (Multi-catch syntax): What is multi-catch syntax and when is
it useful?**

A: Multi-catch (Java 7+, JEP 334) allows catching multiple unrelated
exception types in a single catch block using `|`:

```java
// BEFORE Java 7: duplicate handling:
try {
    riskyOperation();
} catch (IOException e) {
    log.error("IO error", e);
    throw new ServiceException("Operation failed", e);
} catch (SQLException e) {
    log.error("IO error", e);    // same handling - duplicated!
    throw new ServiceException("Operation failed", e);
}

// Java 7+: multi-catch:
try {
    riskyOperation();
} catch (IOException | SQLException e) {
    // e is effectively final - cannot be reassigned
    log.error("Operation error", e);
    throw new ServiceException("Operation failed", e);
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

Multi-catch type: the inferred type is the common supertype. If
`IOException` and `SQLException` both extend `Exception`, `e` is
of type `Exception` in the multi-catch block.

**When to use:**
- Same handling for unrelated exceptions (different inheritance hierarchy)
- Reducing boilerplate without a common base exception class
- When you cannot or don't want to catch the common supertype (Exception)

*What separates good from great:* Multi-catch implicitly makes `e`
effectively final - you cannot reassign it. This is intentional: if
you could reassign `e` to a different exception type, the runtime type
would be ambiguous. This restriction also encourages cleaner code:
you should handle the exception as received, not transform it into
a different type inside the catch block. If transformation is needed,
catch separately and rethrow explicitly.

---


# Java Core - L1 Types and Strings

## Primitive Types and Autoboxing

---

### 🎯 Model Answer

**30 seconds:**
> Java has 8 primitive types - value types stored on the stack with no
> object overhead: `byte` (1 byte), `short` (2), `int` (4), `long` (8),
> `float` (4), `double` (8), `char` (2), `boolean` (1 bit/1 byte JVM-
> dependent). Each has a wrapper class (Integer, Long, Double, etc.)
> that adds object behavior for use in collections. Autoboxing is the
> compiler's automatic conversion between primitive and wrapper:
> `Integer x = 5` (boxing), `int y = x` (unboxing). The trap: autoboxing
> `null` unboxes to NullPointerException; excessive boxing kills performance.

**3 minutes (Senior):**
> Primitives exist because Java's object model has significant overhead:
> every object has a 16-byte header, a reference indirection, and heap
> allocation. An array of 1 million ints is 4MB on the stack or heap
> directly; an Integer[] is 4MB of references + 16MB of objects = 20MB.
> That's 5x more memory and far worse cache performance.
>
> Autoboxing traps: (1) `Integer x = null; int y = x;` throws NPE.
> (2) `==` comparison: `Integer.valueOf(127) == Integer.valueOf(127)` is
> true (cache), but `Integer.valueOf(128) == Integer.valueOf(128)` is false
> (different objects). (3) Autoboxing in loops creates massive GC pressure.
>
> Performance: use `int[]` not `Integer[]` for numeric arrays. Use
> `int` not `Integer` for local variables and parameters. Use
> `IntStream` not `Stream<Integer>` for numeric streams.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Primitive vs wrapper types | 60 seconds |
| Autoboxing mechanics | 2 minutes |
| Integer cache behavior | 2 minutes |
| Float/double precision | 2 minutes |
| Boxing performance impact | 2-3 minutes |
| Null unboxing trap | 90 seconds |
| When to use each type | 2 minutes |

---

**Q1 (Primitive vs wrapper): Why does Java have both primitive and
wrapper types?**

A: Primitives exist for performance. Java's object model requires every
object to have a header (16 bytes on 64-bit JVM) plus the data. An
`int` (4 bytes) vs `Integer` (20 bytes) is a 5x size difference.
For numeric arrays: `int[1000]` is 4KB; `Integer[1000]` is 4KB of
references + 20KB of Integer objects = 24KB total. The reference
indirection also destroys CPU cache locality.

Wrappers are needed for: collections (`List<Integer>` - generics can't
use primitives), nullability (`Integer` can be null; `int` cannot),
reflection (primitives have no `Class.forName()`), utility methods
(`Integer.parseInt()`, `Integer.toBinaryString()`).

Project Valhalla (future Java) will introduce generic specialization,
allowing `List<int>` - eliminating this trade-off entirely.

*What separates good from great:* Know the specific use cases that
require wrappers: nullable numeric columns from databases, map keys
(HashMap requires objects), generic type parameters, and concurrent
structures (AtomicInteger wraps int for CAS operations). When you
see `Integer` vs `int` in an API, the choice reveals the API's
contract: `Integer` means "may be null" or "used generically";
`int` means "always has a value, used for computation."

---

**Q2 (Autoboxing mechanics): Explain exactly what the compiler does
for autoboxing and unboxing.**

A: Autoboxing is a compile-time syntactic transformation. The compiler
inserts explicit boxing/unboxing calls:

```java
// Source code:
Integer x = 5;          // autoboxing
int y = x;              // autounboxing
List<Integer> l = ...; l.add(3); // autoboxing in method call
int v = l.get(0);       // autounboxing from method return

// Compiler generates (bytecode equivalent):
Integer x = Integer.valueOf(5);
int y = x.intValue();
l.add(Integer.valueOf(3));
int v = l.get(0).intValue();
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

`Integer.valueOf()` uses the cache for -128 to 127; outside that range
it calls `new Integer(n)` (deprecated in Java 9).

Unboxing on null: `x.intValue()` where `x == null` throws NPE. The
compiler generates the unboxing call without null check. This is why
null unboxing produces NPE with no obvious cause in the stack trace.

*What separates good from great:* The compiler warns about some boxing
inefficiencies but not all. Production issue: `Integer i = Integer.parseInt(...)` -
then `if (i != null && i > 0)` - the `i > 0` comparison unboxes `i`.
If `i` is null, the null check should prevent NPE, but... no: `&&`
evaluates left to right with short-circuit, so this is safe. But
`if (i > 0 && i != null)` reversed order is NOT safe - `i > 0`
unboxes first, NPE if null. Consistent null-check-first is the rule.

---

**Q3 (Integer cache): What is the Integer cache and why can it cause bugs?**

A: `Integer.valueOf()` caches Integer objects for values -128 to 127
(inclusive). These are static final cached instances. Any call to
`Integer.valueOf()` (including autoboxing) for values in this range
returns the SAME object instance.

```java
Integer a = 100;   // Integer.valueOf(100) - cached instance
Integer b = 100;   // Integer.valueOf(100) - same cached instance
a == b;            // true (same object reference)

Integer c = 200;   // Integer.valueOf(200) - new object
Integer d = 200;   // Integer.valueOf(200) - different new object
c == d;            // false (different object references)
c.equals(d);       // true (same value)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

The cache exists for performance (small values are very common, caching
avoids allocating the same Integer repeatedly) and is specified in the
JLS (Java Language Specification, Section 5.1.7).

The cache range is configurable: `-XX:AutoBoxCacheMax=<N>` can extend
the upper bound. But it's a non-standard JVM property; don't rely on it.

*What separates good from great:* The cache bug is one of the most
common Java interview traps. The dangerous form: code that works in
development (small IDs, -128 to 127) but fails in production (real
IDs, larger numbers). `Integer id1 = order.getId(); Integer id2 = event.getOrderId(); if (id1 == id2) {...}` - works for IDs 1-127, fails
silently for ID 128+. Rule: always use `.equals()` for reference type
comparison, never `==`. Use `==` only for reference identity check.

---

**Q4 (Float/double precision): Why shouldn't you use float/double for
currency or financial calculations?**

A: `float` and `double` use IEEE 754 binary floating-point
representation. Most decimal fractions (0.1, 0.2, 0.3) CANNOT be
represented exactly in binary floating point.

```
0.1 in binary is: 0.00011001100110011... (infinite repeating)
Stored as double: 0.1000000000000000055511151231257827021181583404541015625
0.2 in binary: 0.001100110011001100... (infinite repeating)
0.1 + 0.2 = 0.30000000000000004 (not 0.3)
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

**For currency use `BigDecimal`:**

```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// BAD: float/double for money
double price = 0.10;
double tax = 0.02;
double total = price + tax; // 0.12000000000000001 - wrong!

// GOOD: BigDecimal with String constructor (not double constructor)
BigDecimal price = new BigDecimal("0.10"); // exact
BigDecimal tax = new BigDecimal("0.02");   // exact
BigDecimal total = price.add(tax);         // 0.12 - exact!
// DO NOT use: new BigDecimal(0.10) - inherits double imprecision!
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

Use `double` for: physics simulations, statistics, rendering, any
case where small rounding errors are acceptable.

*What separates good from great:* `BigDecimal` operations require
specifying `MathContext` for division and `RoundingMode` for any
operation that might produce infinite precision:
`price.divide(quantity, 2, RoundingMode.HALF_UP)`. The `scale`
(decimal places) must be managed explicitly. A common pattern:
store monetary values as `long` cents (`1050` = $10.50) to avoid
BigDecimal overhead in high-throughput systems, converting to
BigDecimal only for display.

---

**Q5 (Boxing performance): How does autoboxing affect performance in
a high-throughput system?**

A: Autoboxing impacts: heap allocation rate, GC frequency, and CPU
cache locality.

Allocation cost: each boxing operation allocates an object on the heap.
For a counter that receives 100K events/second, `Long count = 0L;
count += 1;` creates and discards 100K Long objects per second - GC
must collect them all.

Cache locality: `int[]` stores values contiguously - 100 ints = 400
bytes, fits in L1 cache. `Integer[]` stores 100 pointers (400 bytes)
pointing to 100 scattered Integer objects = ~2000 bytes, many cache
misses.

**Profiling boxing:**
```bash
# JVM allocation profiling:
java -XX:+PrintGCDetails -jar app.jar
# High "young gen" GC = boxing in hot path

# JFR allocation events:
jcmd <pid> JFR.start settings=profile duration=30s
# Look for: java.lang.Integer, java.lang.Long in top allocations
```

> **Code walkthrough:** This Look for: java.lang.Integer, java.lang.Long in top allocations example demonstrates shell script pattern. **KEY MECHANISM:** the shell executes commands sequentially; pipes pass stdout of one command to stdin of the next. **WHY IT MATTERS:** unquoted variables with spaces cause word splitting - IFS splits the value into multiple arguments. **TAKEAWAY: always double-quote variables: "$VAR"; use [[ ]] instead of [ ] for safer conditionals.**

**Fixes:**
- `int/long` local variables and parameters
- `IntStream`/`LongStream` instead of `Stream<Integer>`
- `int[]`/`long[]` instead of `List<Integer>`
- `EnumMap`, `HashMap` with primitive value libraries (Eclipse Collections)

*What separates good from great:* JVM JIT can eliminate boxing for
short-lived objects via escape analysis (if the Integer never leaves
the method, the JVM may allocate on stack or eliminate entirely).
But this is not guaranteed and hard to verify. Eliminate boxing
in hot paths explicitly - don't rely on JIT to fix it. Use
async-profiler or JFR allocation profiling to identify boxing in
production hot paths.

---

**Q6 (Null unboxing trap): Describe a production bug caused by null
unboxing.**

A: Scenario: a REST endpoint receives a JSON body with optional numeric
field. Jackson deserializes it to a DTO. If the JSON field is absent,
Jackson sets the field to `null` (for wrapper types).

```java
// DTO:
public class OrderRequest {
    private Integer quantity; // nullable from JSON
}

// Service:
public void processOrder(OrderRequest req) {
    int qty = req.getQuantity(); // NullPointerException if absent
    if (qty > 0) {
        // ...
    }
}
```

> **Code walkthrough:** This Look for: java.lang.Integer, java.lang.Long in top allocations example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

This produces NPE with stack trace pointing to the assignment line,
not the null source. Confusing for developers who see `int qty = ...`
and don't immediately think "NPE here".

**Fix patterns:**
```java
// Option 1: null check
int qty = req.getQuantity() != null ? req.getQuantity() : 0;

// Option 2: Optional
int qty = Optional.ofNullable(req.getQuantity()).orElse(0);

// Option 3: change DTO to primitive + default value
// (Jackson uses 0 as default for int)
private int quantity = 0; // never null
```

> **Code walkthrough:** This Look for: java.lang.Integer, java.lang.Long in top allocations example demonstrates null-safe value wrapping using Optional. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

*What separates good from great:* The best fix is domain-driven:
if `quantity` being absent is a validation error (required field),
add a `@NotNull` validation constraint and reject the request early.
If absent means "default to X", use a default value in the primitive
field or a `@JsonProperty` default. The NPE is a symptom; the root
cause is missing input validation at the system boundary.

---

**Q7 (When to use each type): Walk through the decision: int, Integer,
long, BigDecimal - when to use each?**

A:

| Use Case | Type | Reason |
|---|---|---|
| Loop counters, indices | `int` | Primitive, no boxing |
| Numeric computation | `int/long` | Primitive arithmetic |
| Large numbers (>2 billion) | `long` | int max = ~2.1 billion |
| Monetary amounts | `BigDecimal` | Exact decimal |
| DB nullable int column | `Integer` | Can be null |
| Collection element | `Integer` / `Long` | Generics require objects |
| Atomic counter (concurrency) | `AtomicLong` | Thread-safe primitives |
| High-freq numeric array | `int[]` / `long[]` | No boxing, cache-friendly |
| Bit flags | `int` with bit ops | Efficient flag representation |

```java
// Database nullable column pattern:
@Column(name = "discount_percent", nullable = true)
private Integer discountPercent; // null = no discount applied

// Compute with null safety:
int effectiveDiscount =
    discountPercent != null ? discountPercent : 0;
```

> **Code walkthrough:** This Look for: java.lang.Integer, java.lang.Long in top ice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* The choice between `int` and `Integer`
communicates intent in APIs. A method signature `void process(int count)`
says "count is always present." `void process(Integer count)` says "count
might be null - handle it." APIs with `Integer` parameters that don't
handle null are bugs waiting to happen. Review method signatures in
code reviews: any `Integer`/`Long` parameter should have documented
nullability semantics.

---

## String Class and String Pool

---

### 🎯 Model Answer

**30 seconds:**
> `String` in Java is immutable: once created, its character data
> cannot change. The JVM maintains a String pool (intern pool) for
> string literals - identical literal strings share one object.
> Operations like `concat`, `substring`, and `replace` create new
> String objects, not modify the original. For repeated string
> construction, use `StringBuilder` (single-threaded) or
> `StringBuffer` (thread-safe, but slower). Immutability makes
> Strings safe to share across threads without synchronization.

**3 minutes (Senior):**
> String immutability enforces the invariant that strings used as
> HashMap keys cannot be mutated after insertion (critical for
> hash-based correctness). The String pool allows literal comparison
> with `==` for pooled strings, but `new String("x")` bypasses the
> pool, creating a distinct object. `String.intern()` can add any
> string to the pool.
>
> Java 9+ changed String internal representation: `byte[]` (compact
> strings) instead of `char[]`. ASCII strings use 1 byte per character;
> non-ASCII uses 2 bytes. This halves memory for ASCII strings.
> Java 11 introduced `String.repeat()`, `strip()` (Unicode-aware), and
> `isBlank()`. Java 15 introduced text blocks (triple-quote strings).
> `StringBuilder` is not thread-safe. `StringBuffer` is synchronized.
> For concurrent string building: use local `StringBuilder` per thread
> (shared-nothing pattern).

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Why String is immutable | 60 seconds |
| String pool mechanics | 2 minutes |
| StringBuilder vs StringBuffer | 90 seconds |
| String comparison traps | 2 minutes |
| Java 9 compact strings | 2 minutes |
| String memory optimization | 2-3 minutes |
| Text blocks | 90 seconds |

---

**Q1 (Why immutable): Why is String immutable in Java?**

A: Four reasons:

1. **Thread safety:** immutable objects can be shared across threads
   without synchronization. A String passed to multiple threads cannot
   be corrupted.

2. **HashMap key safety:** if a String used as a HashMap key could be
   mutated, its hash code would change, making it unfindable in the map.
   Immutability guarantees the key's hash never changes.

3. **Security:** passwords, file paths, network addresses validated as
   Strings cannot be changed between validation and use (no TOCTOU race).

4. **String pool:** caching String objects in a pool only works if they
   can never change. If `"hello"` could be mutated, sharing it would
   corrupt all holders.

*What separates good from great:* The security argument is the deepest.
Consider: `File f = new File(path)` where `path` is a String argument.
If String were mutable, an attacker could give you a path you validate,
then mutate it to a different path before `f` is used. Java's immutable
String makes this attack impossible. C strings (char[]) are mutable,
which is one reason C programs are more vulnerable to path traversal
attacks.

---

**Q2 (String pool): How does the String pool work and when does
intern() help?**

A: The String pool (intern pool) is a JVM-level cache of String objects.
All string literals in bytecode are automatically interned. `String.intern()`
allows runtime-created strings to be added to the pool.

The pool is stored in the JVM's heap (Java 7+ - previously PermGen,
which caused PermGen OOM with many interned strings).

**When intern() helps:** when you have millions of repeated strings with
high duplication rate (e.g., field names, status codes, category names
read from database).

```java
// Memory scenario: 1M records, each with "ACTIVE"/"INACTIVE" status
// WITHOUT intern: 1M String objects (~40 bytes each) = ~40MB
// WITH intern: 2 String objects + 1M references (~8 bytes each) = ~8MB

List<String> statuses = dbQuery.getStatuses();
for (int i = 0; i < statuses.size(); i++) {
    statuses.set(i, statuses.get(i).intern()); // deduplicate
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**When NOT to use intern:** unique strings (user names, URLs) - interning
adds JNI overhead without reducing duplicates. Consider a `Map<String,
String>` deduplication cache instead.

*What separates good from great:* Java 8+ has the `-XX:StringTableSize`
JVM flag to resize the string table (default 60013 buckets). For
massive intern() use, increase it: `-XX:StringTableSize=1000003`.
`jcmd <pid> VM.stringtable` reports table statistics. Modern best
practice: prefer explicit object identity (enums, IDs) over string
interning for semantic deduplication.

---

**Q3 (StringBuilder vs StringBuffer): When do you use StringBuilder
vs StringBuffer?**

A: Nearly always `StringBuilder`. `StringBuffer` is thread-safe
(all methods synchronized) but adds synchronization overhead even
in single-threaded use. Java's designers have stated that `StringBuffer`
is effectively obsolete.

The correct pattern for concurrent string building is not `StringBuffer`
but rather: thread-local `StringBuilder` (each thread has its own),
or synchronize at a higher level, or use `String.join()` for simple cases.

```java
// WRONG: StringBuffer for thread safety
class LogFormatter {
    private final StringBuffer sb = new StringBuffer(); // over-engineered
    synchronized void append(String s) { sb.append(s); }
}

// RIGHT: local StringBuilder per operation
class LogFormatter {
    String format(List<String> entries) {
        StringBuilder sb = new StringBuilder(); // local, no sharing
        for (String e : entries) sb.append(e).append('\n');
        return sb.toString();
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates mutex locking using concurrency primitive. **KEY MECHANISM:** the JVM acquires the intrinsic lock on the object monitor before entering the block. **WHY IT MATTERS:** a thread holding the lock blocks all other threads - a bottleneck at scale. **TAKEAWAY: prefer ReentrantLock or ConcurrentHashMap over synchronized for hot paths.**

`StringBuffer` exists for historical compatibility. It was the only
option before Java 5. `StringBuilder` was added in Java 5 as the
non-synchronized alternative.

*What separates good from great:* The performance difference is measurable
but rarely the bottleneck. More important: `StringBuffer` in code signals
the original author was worried about thread safety. If you inherit code
using `StringBuffer`, investigate the threading model - were they correct
to be concerned? Is there shared mutable state that needs fixing? The
`StringBuffer` is often a symptom of a larger concurrency design issue.

---

**Q4 (String comparison traps): What are the common String comparison
pitfalls?**

A:

**Trap 1: `==` instead of `.equals()`**
```java
// WRONG:
if (status == "ACTIVE") { ... }  // almost certainly wrong

// RIGHT:
if ("ACTIVE".equals(status)) { ... }  // null-safe and value-based
if (Objects.equals(status, "ACTIVE")) { ... } // null-safe both sides
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Trap 2: `.equals()` on potentially null reference**
```java
String s = null;
s.equals("test"); // NPE!
"test".equals(s); // safe - returns false
Objects.equals(s, "test"); // safe - handles null on both sides
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Trap 3: case sensitivity**
```java
// WRONG for case-insensitive comparison:
if (type.equals("json")) { ... }

// RIGHT:
if ("json".equalsIgnoreCase(type)) { ... }
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Trap 4: `compareTo()` for alphabetical sort (locale issues)**
```java
// WRONG for locale-sensitive text:
list.sort(String::compareTo);

// RIGHT for user-facing text:
list.sort(Collator.getInstance(Locale.US)::compare);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* In a code review, any `string1 ==
string2` comparison is a bug unless one of the operands is interned
or is a known literal and you are explicitly checking object identity.
The `Objects.equals(a, b)` pattern is the safest and most readable
for equality with nullable strings. Spring Security commonly compares
strings with `StringUtils.hasText()` (null + blank check) and
`passwordEncoder.matches()` (timing-safe comparison for passwords).

---

**Q5 (Compact strings): What changed in Java 9 String representation?**

A: Before Java 9: `String` stored characters as `char[]`, where each
`char` is 2 bytes (UTF-16). All strings, even pure ASCII, used 2 bytes
per character.

Java 9 (JEP 254 - Compact Strings): `String` stores characters as
`byte[]` with an encoding flag:
- If all characters fit in Latin-1 (codepoints 0-255): 1 byte/char
- Otherwise (any char > 255): 2 bytes/char (UTF-16 encoded)

Impact: ASCII and Latin-1 strings use half the memory. Significant
for typical English-language applications where most strings are ASCII.

Benchmark (from JEP 254): 10-15% reduction in heap for typical server
workloads (mostly ASCII strings).

No API change: the change is internal, transparent to application code.
Disabled with `-XX:-CompactStrings` (rarely needed, e.g., heavy
non-Latin string workloads where the encoding check has overhead).

*What separates good from great:* The encoding flag is stored as a byte
field in the String object. String operations check this flag and use
fast paths for Latin-1 strings (byte operations) vs UTF-16 strings
(char operations). For applications serving non-Latin content (Chinese,
Japanese, Arabic), compact strings provide less benefit and have a small
overhead for the flag check. Profile before assuming compact strings
help in your specific workload.

---

**Q6 (String memory optimization): How do you reduce String memory
usage in a JVM with millions of strings?**

A:

**1. Avoid unnecessary string creation:**

```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// BAD: new String per log message
logger.info(new StringBuilder("User ").append(id).toString());
// GOOD: use format strings (lazy evaluation):
logger.info("User {}", id); // SLF4J - only formats if level enabled
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

**2. Deduplication via G1GC:**
JVM flag: `-XX:+UseStringDeduplication` (Java 8u20+, requires G1GC)
GC identifies identical String objects and replaces their backing
`byte[]` with a shared copy. Reduces live heap for duplicate strings
without changing references.
```bash
java -XX:+UseG1GC -XX:+UseStringDeduplication \
     -XX:+PrintStringDeduplicationStatistics -jar app.jar
```

> **Code walkthrough:** This Unknown example demonstrates shell script pattern. **KEY MECHANISM:** the shell executes commands sequentially; pipes pass stdout of one command to stdin of the next. **WHY IT MATTERS:** unquoted variables with spaces cause word splitting - IFS splits the value into multiple arguments. **TAKEAWAY: always double-quote variables: "$VAR"; use [[ ]] instead of [ ] for safer conditionals.**

**3. Manual deduplication via intern():**
For high-duplication fields (status codes, categories), intern strings
at the point of creation.

**4. Use byte[] or char[] for very large text:**
For in-memory document processing, work with `byte[]` buffers instead
of String; convert to String only at boundaries.

*What separates good from great:* `-XX:+UseStringDeduplication` is the
easiest win for applications with many duplicate strings (e.g., a cache
storing JSON with repeated field names). It runs during GC, no code
change needed. Check with `-XX:+PrintStringDeduplicationStatistics`
to see deduplication rate. If 40%+ of strings are deduplicated, it's
worth enabling. It does add a small GC overhead (comparing string contents
during GC), so measure the trade-off.

---

**Q7 (Text blocks): What are text blocks and when should you use them?**

A: Text blocks (final in Java 15, JEP 355) are multiline string literals
using triple-quote `"""` syntax. The compiler strips common leading
whitespace and normalizes line endings.

```java
// BEFORE (Java 14 and earlier):
String json = "{\n"
    + "  \"name\": \"Alice\",\n"
    + "  \"age\": 30\n"
    + "}";

// AFTER Java 15+:
String json = """
    {
      "name": "Alice",
      "age": 30
    }
    """;
// ^ trailing """ on its own line adds a trailing newline
// No trailing newline:
String json = """
    {
      "name": "Alice"
    }"""; // """ on same line as last content

// Indentation: the compiler removes leading whitespace common to all
// lines, aligned to the closing """. Incidental whitespace stripped.
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Use cases: SQL queries, JSON/XML templates, HTML templates, multiline
test assertions.

**String templates (Java 21 preview):**
```java
// Interpolation syntax (preview in Java 21):
String name = "Alice";
int age = 30;
String msg = STR."Hello \{name}, you are \{age} years old.";
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY ice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* Text blocks eliminate escape hell
for SQL and JSON in tests. Important: the closing `"""` position
controls indentation stripping. `"""` on a new line with 4 spaces of
indentation = 4 spaces stripped from all lines. If a line has less
indentation than the closing `"""`, a compile error occurs. Use your
IDE's formatter to handle this automatically - IntelliJ auto-formats
text block indentation correctly.

---


---

## equals() and hashCode() Contract

---

### 🎯 Model Answer

**30 seconds:**
> `equals()` and `hashCode()` must be consistent: if two objects are
> equal (`a.equals(b)` is true), they MUST have the same `hashCode()`.
> Breaking this contract causes objects to "disappear" from HashMaps
> and HashSets - you put an object in and can never find it again.
> Both methods are inherited from `Object` (identity equality/identity
> hash). Always override both together - never one without the other.
> Use `Objects.equals()` and `Objects.hash()` (utility methods) to
> implement them correctly.

**3 minutes (Senior):**
> The contract is: (1) equals is reflexive, symmetric, transitive,
> consistent, and x.equals(null) == false. (2) If a.equals(b) then
> a.hashCode() == b.hashCode(). The converse is not required: hash
> collision (same hash, different objects) is allowed and handled by
> the hash bucket's linked list/tree.
>
> hashCode design: good hash functions minimize collisions. Java
> recommends `Objects.hash(field1, field2, ...)` which uses prime
> multiplication (31 * result + field.hashCode()). All-zero hashCode()
> is legal but terrible for HashMap performance (all objects end
> up in bucket 0).
>
> Mutable fields in equals/hashCode: never use mutable fields as
> HashMap keys. If you change a field used in hashCode after insertion,
> the object is in the wrong bucket - it's lost. Make key classes
> immutable or don't include mutable fields.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**First principles:** "From first principles: HashMap stores objects
in buckets by hash code, then uses equals() to find the exact object
within a bucket. If hashCode() returns different values for equal objects,
they end up in different buckets - equals() is never called to find them."


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| equals/hashCode contract | 60 seconds |
| HashMap mechanics | 2-3 minutes |
| Mutable key bug | 2-3 minutes |
| hashCode quality | 2 minutes |
| Records and equals | 90 seconds |
| Symmetric equals pitfall | 2 minutes |
| equals with inheritance | 2-3 minutes |

---

**Q1 (Contract): State the equals/hashCode contract.**

A: Two rules from the Java specification:

**equals() contract:**
- Reflexive: `x.equals(x)` is always true
- Symmetric: `x.equals(y)` iff `y.equals(x)`
- Transitive: if `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`
- Consistent: same result on repeated calls (no side effects)
- Null safety: `x.equals(null)` always returns false (never NPE)

**hashCode() contract:**
- Consistent: repeated calls return same int (unless object state changes)
- If `x.equals(y)` then `x.hashCode() == y.hashCode()`
  (the ONLY hard requirement linking the two)
- Optional but strongly recommended: if `!x.equals(y)` then different hash
  (minimize collisions for performance)

*What separates good from great:* The subtle point is: the contract
does NOT require that unequal objects have different hash codes. That's
just a recommendation for performance. What IS required: if you override
equals to make two objects "equal", you MUST also ensure their hash codes
are the same. HashMap's algorithm first computes hash to find the bucket,
then calls equals within the bucket. A broken contract means the get
never reaches the equals call.

---

**Q2 (HashMap mechanics): How does HashMap use equals() and hashCode()
internally?**

A: HashMap uses a two-phase lookup:

```
Phase 1 (hashCode): which bucket?
  bucket = (n - 1) & hash(key.hashCode())
  n = table length (power of 2)
  hash() = additional spreading to reduce collisions

Phase 2 (equals): which entry in the bucket?
  for each entry in bucket:
      if entry.hash == hash && (entry.key == key
                                || key.equals(entry.key))
          return entry.value
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

Java 8 improvement: when a bucket's linked list exceeds 8 entries,
it's converted to a tree (TreeNode, Red-Black tree) for O(log n) vs
O(n) lookup on heavily collided buckets. This requires keys to be
`Comparable` for treeification.

**If hashCode is broken:**
- All keys may hash to the same bucket -> O(n) lookup (or O(log n))
- The worst case: `hashCode()` returns 0 for everything -> one giant
  bucket -> HashMap degrades to O(n) linked list

*What separates good from great:* The Java 8 treeification of buckets
was motivated by hash DoS attacks. An adversary who knows your hash
function can craft keys that all collide into one bucket, making HashMap
O(n) per get. Java 8 treeifies to O(log n), limiting the damage.
For security-sensitive code: use `String.hashCode()` which is not secret
(predictable). Spring Security's `Hmac` is used to hash sensitive
request parameters before putting in maps.

---

**Q3 (Mutable key bug): Describe the mutable HashMap key bug with example.**

A:
```java
// The trap:
class User {
    int id;
    String name;

    @Override public boolean equals(Object o) {
        if (!(o instanceof User)) return false;
        return id == ((User) o).id && name.equals(((User) o).name);
    }
    @Override public int hashCode() {
        return Objects.hash(id, name); // name is MUTABLE!
    }
}

Map<User, String> roles = new HashMap<>();
User alice = new User(1, "Alice");
roles.put(alice, "ADMIN");   // stored in bucket = hash("Alice")

alice.name = "ALICE";        // mutate the name!
roles.get(alice);            // looks in bucket = hash("ALICE")
                             // different bucket -> returns null!
roles.containsKey(alice);    // false - "lost" the entry
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

**Fix options:**
1. Make key class immutable: `final class User { final String name; }`
2. Don't use mutable fields in hashCode: only use `id`
3. Don't use this class as a map key (use the ID directly)

*What separates good from great:* This bug is insidious because the
map doesn't throw an exception - it silently returns null. If you
subsequently `put` with the mutated key, you create a SECOND entry
in the map with the new hash, while the original entry still exists
in the old bucket. Iterating the map would reveal two entries for
"the same object". Production diagnosis: `HashMap.size()` keeps
growing, or lookups always miss despite puts succeeding.

---

**Q4 (hashCode quality): What makes a good hashCode implementation?**

A: A good hash function minimizes collisions and distributes entries
evenly across buckets.

**Properties of a good hashCode:**
1. Fast to compute
2. Even distribution across the int range
3. Different for "obviously different" objects
4. Consistent with equals

**Effective Java recipe (still the best):**
```java
@Override public int hashCode() {
    int result = 17;              // non-zero prime seed
    result = 31 * result + field1.hashCode();
    result = 31 * result + (field2 != null ? field2.hashCode() : 0);
    result = 31 * result + field3;  // primitive
    return result;
}

// Modern equivalent:
@Override public int hashCode() {
    return Objects.hash(field1, field2, field3);
    // Uses 31*result + ... internally
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Why 31?** It's an odd prime, close to a power of 2, and JVMs optimize
`31 * i` as `(i << 5) - i` (shift + subtract, no multiply needed).

**Bad hashCodes:**
```java
return 0; // all objects in one bucket -> O(n) everywhere
return id; // only uses one field - misses other distinguishing data
return field1.hashCode() ^ field2.hashCode(); // XOR is symmetric;
          // Point(1,2).hashCode() == Point(2,1).hashCode() -> collision
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* For immutable classes: cache the
hash code. `String` computes hash on first call and caches it in a field.
For mutable classes: never cache (may change). Java's `record` generates
field-based hashCode automatically - the standard implementation using
`Objects.hash()`. Use records for value objects instead of manual
hashCode.

---

**Q5 (Records and equals): How do records handle equals() and hashCode()?**

A: Records (Java 14+, final in Java 16) automatically generate `equals()`,
`hashCode()`, and `toString()` based on all record components.

```java
record Point(int x, int y) {}

// Auto-generated (equivalent to):
@Override public boolean equals(Object o) {
    if (!(o instanceof Point p)) return false;
    return x == p.x && y == p.y;
}
@Override public int hashCode() {
    return Objects.hash(x, y);
}
@Override public String toString() {
    return "Point[x=" + x + ", y=" + y + "]";
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Records are ideal as HashMap keys: immutable (fields are final),
correct equals/hashCode, and minimal boilerplate.

You can override the generated methods:
```java
record CaseInsensitiveName(String value) {
    @Override public boolean equals(Object o) {
        if (!(o instanceof CaseInsensitiveName n)) return false;
        return value.equalsIgnoreCase(n.value);
    }
    @Override public int hashCode() {
        return value.toLowerCase().hashCode();
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* When customizing equals for a record,
you MUST also customize hashCode to maintain the contract. The compiler
does not check this - it's your responsibility. Also: record equality
compares ALL components. If you want partial equality (two records equal
if their IDs match, ignoring other fields), records are not ideal -
use a regular class with explicit equals/hashCode.

---

**Q6 (Symmetric equals pitfall): What's the symmetric equals pitfall
with inheritance?**

A: The Liskov Substitution Principle conflicts with equals in inheritance
hierarchies. A common mistake: making a subclass's equals compare with
the parent class.

```java
class Animal { String species; }
class Dog extends Animal { String breed; }

// BROKEN: asymmetric equals
@Override // in Dog:
public boolean equals(Object o) {
    if (o instanceof Animal a) { // accepts Animal!
        return species.equals(a.species);
    }
    return false;
}

Animal a = new Animal("Canis");
Dog d = new Dog("Canis", "Labrador");
a.equals(d); // false (Animal.equals uses Object identity)
d.equals(a); // true (Dog.equals accepts Animal!)
// Violates symmetry!
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

**Fix: use getClass() instead of instanceof for inheritance:**
```java
// In Dog:
@Override public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Dog dog = (Dog) o;
    return Objects.equals(species, dog.species)
        && Objects.equals(breed, dog.breed);
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Effective Java recommends: "There is no way to extend an instantiable
class and add a value component while preserving the equals contract."
Use composition over inheritance for value objects.

*What separates good from great:* `instanceof` vs `getClass()` is a
classic Effective Java debate. `instanceof` allows subclass instances
to be "equal" to parent instances (useful for polymorphism), but
violates symmetry when the subclass adds value fields. `getClass()`
is strict: only same-class objects can be equal (correct contract,
less polymorphic). For sealed hierarchies with pattern matching,
record-based approaches eliminate this entirely: each case is a
distinct record type with its own equals.

---

**Q7 (equals with inheritance): What is the canonical equals
implementation pattern for a non-final class?**

A:
```java
public class Person {
    private final String name;    // immutable field
    private final int age;        // immutable field

    public Person(String name, int age) {
        this.name = Objects.requireNonNull(name);
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;          // 1. identity check (fast path)
        if (!(o instanceof Person)) return false; // 2. type check
        Person other = (Person) o;           // 3. safe cast
        return age == other.age              // 4. compare primitives first
            && Objects.equals(name, other.name); // 5. null-safe reference
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);      // consistent with equals
    }

    @Override
    public String toString() {
        return "Person[name=" + name + ", age=" + age + "]";
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Step explanation:
1. `this == o`: fast path, same reference always equal
2. `instanceof`: handles null (false if o is null) and type check
3. Cast: safe after instanceof check
4. Primitive first: cheap comparison, fail fast
5. `Objects.equals`: null-safe reference comparison

*What separates good from great:* Always implement `toString()` alongside
equals/hashCode. It has no contract requirements but is invaluable for
debugging. `System.out.println(person)` that shows `Person@7f3e5c` vs
`Person[name=Alice, age=30]` is the difference between hours of debugging.
IDEs generate all three together; records generate all three automatically.
In a code review, a class that overrides equals but not toString is
a missed opportunity.

---

# Java Core - L2 Collections

## Collections Framework Design

---

### 🎯 Model Answer

**30 seconds:**
> The Java Collections Framework (JCF) is a unified hierarchy of
> interfaces and implementations for storing and manipulating groups
> of objects. Core interfaces: `Collection` (add/remove/iterate),
> `List` (ordered, indexed, allows duplicates), `Set` (no duplicates),
> `Map` (key-value pairs, not a Collection), `Queue`/`Deque`
> (ordered processing). Key implementations: `ArrayList` (fast random
> access), `LinkedList` (efficient insert/delete), `HashMap` (O(1) avg
> lookup), `TreeMap` (sorted), `HashSet`, `ArrayDeque`. The framework
> is backed by an interface-first design: always declare as `List` not
> `ArrayList` for flexibility.

**3 minutes (Senior):**
> The framework design (by Joshua Bloch) follows key principles:
> interface-first declarations, algorithm separation (Collections utility
> methods), fail-fast iterators (ConcurrentModificationException on
> structural changes), and consistent time complexity guarantees.
>
> `Iterable` and `Iterator` enable the enhanced for loop. `Comparable`
> and `Comparator` provide sorting. `Collections` utility class provides
> algorithms: `sort`, `binarySearch`, `shuffle`, `reverse`, `unmodifiable*`,
> `synchronized*`. Java 9 added factory methods: `List.of()`, `Set.of()`,
> `Map.of()` - immutable collections with null-rejection.
>
> Trade-offs: `ArrayList` vs `LinkedList` is the most common interview
> question - but in practice, `ArrayList` wins almost always due to
> cache locality. The only legitimate `LinkedList` use case is as a
> `Deque` (double-ended queue), not as a `List`.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

** First principles:** "From first principles: programs need
standard ways to group objects. Without a framework, every developer
would build their own list and map. The JCF provides standard,
tested, optimized implementations with a consistent interface."


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Collection hierarchy | 90 seconds |
| Interface-first principle | 60 seconds |
| List.of() vs unmodifiable | 2 minutes |
| Collection selection | 2 minutes |
| ConcurrentModificationException | 2 minutes |
| Sorting and ordering | 2 minutes |
| Java 21 Sequenced Collections | 90 seconds |
| Fail-fast iterators | 2 minutes |
| Collections utility methods | 90 seconds |

---

**Q1 (Collection hierarchy): Walk through the Java Collections
Framework hierarchy.**

A: The JCF has two root hierarchies:

**`Iterable` / `Collection` tree:**
- `Iterable`: supports for-each loops (`iterator()`)
- `Collection`: adds `add()`, `remove()`, `contains()`, `size()`, `isEmpty()`
- `List`: adds `get(i)`, `set(i)`, `add(i)`, `indexOf()`
- `Set`: no duplicates; same methods as `Collection`
- `SortedSet` / `NavigableSet`: adds `first()`, `last()`, `headSet()`,
  `tailSet()`, `subSet()`
- `Queue`: adds `offer()`, `poll()`, `peek()` (FIFO semantics)
- `Deque`: extends `Queue` with `addFirst()`, `addLast()`,
  `pollFirst()`, `pollLast()` (both ends)

**`Map` tree (separate from `Collection`):**
- `Map`: `put()`, `get()`, `remove()`, `containsKey()`, `keySet()`,
  `values()`, `entrySet()`
- `SortedMap` / `NavigableMap`: sorted by key, range operations

**Java 21 additions (`SequencedCollection`):**
- `SequencedCollection` extends `Collection`: adds `getFirst()`,
  `getLast()`, `addFirst()`, `addLast()`, `reversed()`
- `SequencedSet` extends `SequencedCollection` and `Set`
- `SequencedMap` extends `Map`: adds `firstEntry()`, `lastEntry()`, `reversed()`

*What separates good from great:* The JCF design separated algorithms
from data structures (unlike C++ STL templates). `Collections.sort()`
works on any `List`; `Collections.binarySearch()` on any sorted `List`.
This was ahead of its time in 1998. The interface-first design means
you can substitute implementations (e.g., replace `ArrayList` with
`LinkedList` or `CopyOnWriteArrayList`) by changing one line. Modern
Java streams operate on any `Collection` via the `Spliterator` interface.

---

**Q2 (Interface-first principle): Why should you declare `List<String>
names` instead of `ArrayList<String> names`?**

A: Declaring the interface type:
1. **Flexibility:** you can change the implementation in one place
2. **Encapsulation:** code only uses the interface contract, not
   ArrayList-specific methods (`trimToSize()`, `ensureCapacity()`)
3. **Testability:** tests can pass mock implementations
4. **Communication:** the declaration says "I need a list"; not "I need
   an ArrayList"


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// GOOD: interface type in field, local variable, parameter, return
List<String> process(List<String> items) {
    List<String> result = new ArrayList<>();
    // switch to CopyOnWriteArrayList? Change ONE line above
    ...
    return result;
}

// BAD: concrete type everywhere
ArrayList<String> process(ArrayList<String> items) {
    ArrayList<String> result = new ArrayList<>();
    // now everything depends on ArrayList specifically
}
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **WHAT BREAKS: interfaces define contracts; prefer them over abstract classes for unrelated types.**

**Exceptions: concrete type is appropriate when:**
- You specifically need concrete-type methods not in the interface
  (`ArrayDeque.peekFirst()` vs `Deque.peekFirst()` - actually same here)
- You're explicitly documenting implementation choice for performance
  (`private final ConcurrentHashMap<>` to document thread-safety choice)

*What separates good from great:* API boundary declarations (method
parameters, return types) are the most important. Using `List` in a
public method signature means callers can pass any List. Using `ArrayList`
unnecessarily restricts callers and leaks implementation detail. However:
in a `private` method used only within the class, the concrete type
in a `local variable` is mostly a style choice - the JVM doesn't care.
Focus the interface-first discipline on public API boundaries.

---

**Q3 (List.of() vs unmodifiable): What's the difference between
List.of() and Collections.unmodifiableList()?**

A:

| Aspect | `List.of()` | `Collections.unmodifiableList()` |
|---|---|---|
| Truly immutable | YES | NO (original can mutate) |
| Null elements | Rejects (NPE) | Allows |
| Backed by original | NO (own storage) | YES (wrapper) |
| Iteration order | JVM-dependent (small) | Same as wrapped |
| Memory | Compact (JVM optimized) | Extra wrapper object |
| Thread-safe | YES | Reads yes; writes blocked |

```java
List<String> src = new ArrayList<>(Arrays.asList("a","b","c"));
List<String> unmod = Collections.unmodifiableList(src);
List<String> immut = List.of("a", "b", "c");

src.add("d");
unmod.get(3); // "d" - unmod sees the mutation!
// immut never changes

unmod.add("e"); // UnsupportedOperationException
immut.add("e"); // UnsupportedOperationException

unmod.contains(null); // false - null not in list
immut.contains(null); // NullPointerException - List.of() rejects null
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Practical choice:**
- `List.of()`: return values, constants, test data - truly immutable
- `Collections.unmodifiableList()`: when you need a read-only VIEW of
  a mutable list (rare; usually defensive copy is better)
- `List.copyOf(other)` (Java 10): immutable copy of another collection

*What separates good from great:* Returning a mutable collection from
a method is a common encapsulation violation. Callers can modify the
returned list and affect the object's internal state:
`order.getItems().clear()` could empty an order's items if `getItems()`
returns the internal list. Fix: return `List.copyOf(items)` or
`Collections.unmodifiableList(items)`. `List.copyOf` is safer
(truly immutable). The tradeoff: defensive copy has O(n) allocation;
unmodifiable wrapper has O(1) but the underlying list is still mutable.

---

**Q4 (Collection selection): How do you choose the right collection
implementation?**

A:

Decision tree:

**Need key-value pairs?** -> `Map` family
- Unordered, fast: `HashMap`
- Insertion order: `LinkedHashMap`
- Sorted by key: `TreeMap`
- Enum keys: `EnumMap`
- Concurrent: `ConcurrentHashMap`

**Need no duplicates?** -> `Set` family
- Unordered, fast: `HashSet`
- Insertion order: `LinkedHashSet`
- Sorted: `TreeSet`

**Need ordered sequence?** -> `List` or `Queue` family
- Random access: `ArrayList` (almost always)
- Queue/stack operations: `ArrayDeque` (never LinkedList)
- Concurrent reads: `CopyOnWriteArrayList`
- Priority-based: `PriorityQueue`

**The `LinkedList` myth:**
```java
// WRONG: using LinkedList as a List (almost always worse than ArrayList)
List<String> list = new LinkedList<>(); // worse cache locality

// RIGHT: use ArrayDeque for queue/stack:
Deque<String> queue = new ArrayDeque<>();
queue.addLast("a");   // enqueue
queue.pollFirst();    // dequeue

// RIGHT: use ArrayList for list:
List<String> list = new ArrayList<>();
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The performance case against LinkedList
is compelling. Modern CPUs have 64-byte cache lines. `ArrayList` stores
elements contiguously; iterating it is a sequential memory scan with
perfect prefetching. `LinkedList` nodes are scattered in heap memory;
iterating it follows pointers, causing cache misses. For a list of 1000
elements, iteration is 2-10x faster with `ArrayList` even though
`LinkedList` has O(1) insertions. The only competitive use case for
`LinkedList` is as a `Deque` - but `ArrayDeque` is still faster.

---

**Q5 (ConcurrentModificationException): Explain fail-fast iterators
and how to avoid CME.**

A: Fail-fast iterators track a `modCount` (modification count) on the
collection. When the iterator is created, it captures `modCount`.
On each `next()` call, it checks if `modCount` changed. If yes:
`ConcurrentModificationException`.

This detects structural modifications (add, remove, clear - anything
that changes the list size) made while iterating.

**Safe modification patterns:**
```java
List<String> items = new ArrayList<>(List.of("a","b","c","d"));

// Option 1: Iterator.remove() - safe
Iterator<String> it = items.iterator();
while (it.hasNext()) {
    if (it.next().startsWith("a")) it.remove(); // safe
}

// Option 2: removeIf (Java 8+) - cleanest
items.removeIf(s -> s.startsWith("a"));

// Option 3: stream filter + collect - creates new list
List<String> kept = items.stream()
    .filter(s -> !s.startsWith("a"))
    .collect(Collectors.toList());

// Option 4: indexed loop with decreasing index
for (int i = items.size() - 1; i >= 0; i--) {
    if (items.get(i).startsWith("a")) items.remove(i);
}
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* CME is a best-effort detection
mechanism, not a hard guarantee. The Javadoc says: "It is not generally
permissible for one thread to modify a Collection while another thread
is iterating over it." Concurrent access requires thread-safe collections
(`CopyOnWriteArrayList` for read-heavy, `Collections.synchronizedList()`
with external synchronization, or `ConcurrentHashMap` for maps). CME
also occurs in single-threaded code if you modify while iterating -
the most common case.

---

**Q6 (Sorting and ordering): How do you sort a collection in Java?**

A:

```java
List<String> names = new ArrayList<>(List.of("Charlie","Alice","Bob"));

// Method 1: Collections.sort() - mutates list
Collections.sort(names);                         // [Alice, Bob, Charlie]
Collections.sort(names, Comparator.reverseOrder()); // [Charlie, Bob, Alice]

// Method 2: List.sort() - mutates list (Java 8+)
names.sort(Comparator.naturalOrder());
names.sort(Comparator.comparingInt(String::length)); // by length
names.sort(Comparator.comparingInt(String::length)
    .thenComparing(Comparator.naturalOrder())); // length, then alpha

// Method 3: Stream.sorted() - creates new sorted stream (no mutation)
List<String> sorted = names.stream()
    .sorted(Comparator.comparingInt(String::length))
    .collect(Collectors.toList());

// Sorting objects:
record Person(String name, int age) {}
List<Person> people = List.of(
    new Person("Alice", 30), new Person("Bob", 25)
);
List<Person> byAge = people.stream()
    .sorted(Comparator.comparingInt(Person::age))
    .toList(); // Java 16+

// Reverse a sorted:
List<Person> byAgeDesc = people.stream()
    .sorted(Comparator.comparingInt(Person::age).reversed())
    .toList();
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* Java's sort algorithm is `TimSort`
(a hybrid merge sort / insertion sort), O(n log n) worst case with O(n)
for nearly-sorted data. It's stable: equal elements maintain their
original relative order. This matters when sorting by one field and
wanting secondary sort to be preserved from a previous sort. For parallel
sorting of large arrays: `Arrays.parallelSort()` uses fork/join for
O(n log n) with multiple cores.

---

**Q7 (Java 21 Sequenced Collections): What are Sequenced Collections
in Java 21?**

A: Java 21 (JEP 431) added three new interfaces to unify "has a defined
encounter order" semantics:

```java
// Before Java 21: inconsistent APIs
list.get(0);              // List - get first
deque.peekFirst();        // Deque - peek first
sortedSet.first();        // SortedSet - get first
// No common way to get "first" of any ordered collection!

// Java 21: SequencedCollection interface
interface SequencedCollection<E> extends Collection<E> {
    E getFirst();     // get first element
    E getLast();      // get last element
    void addFirst(E e);
    void addLast(E e);
    E removeFirst();
    E removeLast();
    SequencedCollection<E> reversed(); // reversed view
}
// ArrayList, LinkedList, ArrayDeque all implement this now
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

**Practical use:**
```java
List<String> list = new ArrayList<>(List.of("a","b","c"));
list.getFirst();  // "a" - new Java 21 method
list.getLast();   // "c" - new Java 21 method
list.reversed();  // reversed view ["c","b","a"]
list.addFirst("z"); // adds at front: ["z","a","b","c"]
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `SequencedCollection.reversed()`
returns a VIEW, not a copy. Mutating the original is reflected in the
reversed view and vice versa. For an immutable reversed copy:
`List.copyOf(list.reversed())`. The sequenced collection interfaces
unify what previously required different APIs for each collection type,
reducing the need to check documentation for "how do I get the last
element of this particular type?"

---

**Q8 (Fail-fast iterators): What is the difference between fail-fast
and fail-safe iterators?**

A:

| Property | Fail-Fast (ArrayList, HashMap) | Fail-Safe (CopyOnWrite, Concurrent) |
|---|---|---|
| How works | Throw CME if modified during iteration | Operate on snapshot or don't throw |
| Memory | No copy | Copy (CopyOnWrite) or segment lock |
| Sees updates | Yes (same underlying data) | No (snapshot) or eventually |
| Thread safety | No | Yes |
| Performance | Lower overhead | Higher overhead |

**Fail-safe examples:**
```java
// CopyOnWriteArrayList: iterates on snapshot taken at iterator creation
List<String> cowList = new CopyOnWriteArrayList<>(List.of("a","b"));
for (String s : cowList) {
    cowList.add("x"); // no CME! iterator sees original ["a","b"]
}
// After loop: cowList = ["a","b","x","x"]

// ConcurrentHashMap: iterates without copying, no CME, may see partial updates
Map<String, Integer> map = new ConcurrentHashMap<>();
for (Map.Entry<String, Integer> e : map.entrySet()) {
    map.put("new", 1); // no CME, but may or may not be seen in iteration
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using SQL. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* `CopyOnWriteArrayList` is only
appropriate when writes are rare and reads are very frequent. Each write
creates a complete copy of the underlying array: O(n) write cost. For
an application with 100 readers and 1 writer: great. For 50% writes:
terrible performance. Real-world uses: event listener lists (many reads,
rare adds/removes), role/permission lists (loaded at startup, rarely changed).

---

**Q9 (Collections utility methods): What are the most useful methods
in the Collections utility class?**

A:
```java
List<Integer> nums = new ArrayList<>(List.of(3,1,4,1,5,9,2,6));

// Sorting:
Collections.sort(nums);              // natural order
Collections.sort(nums, Comparator.reverseOrder());

// Searching (requires sorted list):
Collections.sort(nums);
int pos = Collections.binarySearch(nums, 4); // >= 0 if found

// Reordering:
Collections.shuffle(nums);           // randomize
Collections.reverse(nums);           // reverse in place

// Stats:
Collections.max(nums);               // max element
Collections.min(nums);               // min element
Collections.frequency(nums, 1);      // count occurrences

// Defensive wrappers:
List<String> immutable = Collections.unmodifiableList(mutableList);
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
Set<String> emptySet = Collections.emptySet(); // empty immutable
List<String> singletonList = Collections.singletonList("only"); // 1 element

// Fill and copy:
Collections.fill(nums, 0);           // set all to 0
List<Integer> dest = new ArrayList<>(Collections.nCopies(8, 0));
Collections.copy(dest, nums);        // copy nums into dest

// Disjoint check:
boolean noCommon = Collections.disjoint(list1, list2);
```

> **Code walkthrough:** This Unknown example demonstrates mutex locking using coice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* `Collections.emptyList()`,
`Collections.emptySet()`, `Collections.emptyMap()` return singleton
empty collections (not new objects each time). For method return values
when there are no results, prefer these over `new ArrayList<>()`.
Similarly, `Collections.singletonList()` is more efficient than
`new ArrayList<>` with one element for read-only single-element lists.
These are micro-optimizations, but they signal idiomatic Java coding.

---

### ⚖️ Comparison Table

| Collection| Order| Duplicates| Null| Thread-Safe| Get O| Add O|
|-------|-------------|----------|------------|-----------|---------|----------|
| ArrayList| Insertion| Yes| Yes| No| O(1)| O(1) amort|
| LinkedList| Insertion| Yes| Yes| No| O(n)| O(1)|
| ArrayDeque| Insertion| Yes| No| No| O(1) ends| O(1) amort|
| HashSet| None| No| One null| No| -| O(1) avg|
| LinkedHashSet| Insertion| No| One null| No| -| O(1) avg|
| TreeSet| Sorted| No| No| No| -| O(log n)|
| HashMap| None| Keys no| One null key| No| O(1) avg| O(1) avg|
| LinkedHashMap| Insert/access| Keys no| One null key| No| O(1) avg| O(1) avg|
| TreeMap| Key sorted| Keys no| No null key| No| O(log n)| O(log n)|
| ConcurrentHashMap| None| Keys no| No| Yes| O(1) avg| O(1) avg|
| CopyOnWriteArrayList| Insertion| Yes| Yes| Yes| O(1)| O(n)|


---

# Java Core - L2 Generics

## Generics and Type Erasure

---

### 🎯 Model Answer

**30 seconds:**
> Generics enable type-safe containers and algorithms without casting.
> `List<String>` guarantees every element is a String - no `ClassCastException`
> at runtime. Java generics use TYPE ERASURE: the type parameter is
> removed by the compiler and not present in bytecode. At runtime,
> `List<String>` and `List<Integer>` are both just `List`. This means:
> no `new T()`, no `instanceof List<String>`, no `T.class`. The compiler
> inserts necessary casts and checks before erasure. The trade-off:
> backward compatibility (old and new code interoperate) at the cost of
> limited runtime type information.

**3 minutes (Senior):**
> Type erasure trade-offs: you CANNOT do `new T[]` (use cast of
> `Object[]`), `new T()` (use a factory or `Class<T>`), or
> `instanceof T` (use `Class<T>.isInstance()`). You CAN use `T` in
> method signatures and return types (useful at compile time). Reifiable
> types are those whose type info is preserved at runtime: raw types,
> non-parameterized types, unbounded wildcards (`?`). Parameterized types
> are not reifiable.
>
> Raw types (e.g., `List` without parameters): unsafe, discouraged,
> exist for backward compatibility with pre-generics code. Mixing raw
> and parameterized types produces "unchecked" warnings.
>
> The `@SuppressWarnings("unchecked")` annotation suppresses the warning
> when you know the cast is safe (e.g., in generic container internals).
> Should be used sparingly and documented.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Generics and type erasure - let me cover what generics
provide (compile-time type safety), how type erasure works, its
limitations, and common generic patterns."

**(2) First principles:** "From first principles: before generics,
`ArrayList` held `Object` and every read required a cast that could fail
at runtime. Generics move that failure to compile time, making programs
safer. Type erasure was chosen for backward compatibility - the JVM
doesn't need to know about generics."

**(3) Bridge:** "Generics are like clothing size labels that the factory
(compiler) uses to sort and verify, but the label is removed before
the product ships (runtime). The product (bytecode) itself has no label,
but everything was checked before the label was removed."

---


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Type erasure explained | 2 minutes |
| Why type erasure was chosen | 2 minutes |
| Generic array limitation | 2 minutes |
| Bounded type parameters | 2 minutes |
| Raw types dangers | 2 minutes |
| Super type token | 2-3 minutes |
| Generic method vs class | 90 seconds |
| @SuppressWarnings unchecked | 90 seconds |
| Heap pollution | 2 minutes |

---

**Q1 (Type erasure explained): Explain type erasure with a concrete example.**

A: Type erasure is the compiler process of removing type parameter information
from bytecode, replacing parameterized types with raw types and inserting
casts where needed.

```java
// Compiled source:
public <T extends Number> double sum(List<T> nums) {
    double total = 0;
    for (T n : nums) {
        total += n.doubleValue(); // T is known to extend Number
    }
    return total;
}

// After erasure (what bytecode represents):
public double sum(List nums) {  // T replaced by upper bound: Number
    double total = 0;
    for (Number n : nums) {     // cast inserted by compiler
        total += n.doubleValue();
    }
    return total;
}
// The method's erasure signature: double sum(List)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using SQL. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Consequences:
- Method overloading by generic type only is impossible:
  ```java
  void process(List<String> s) {} // erasure: process(List)
  void process(List<Integer> i) {} // COMPILE ERROR: same erasure!
  ```
> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

- Generic type info NOT in bytecode (no `T.class`)
- `instanceof` cannot check parameterized type

*What separates good from great:* Type erasure was a deliberate compatibility
choice. Alternative: reified generics (C# style) where `List<int>` and
`List<String>` are truly different types at runtime. Java chose erasure
to allow old `List` (raw) code to interoperate with new `List<String>`
code via the raw type escape hatch. The cost: every interaction with
generic code from reflection requires extra work (TypeToken pattern).
C# .NET has reified generics and avoids boxing for value types in
`List<int>` - Project Valhalla aims to bring this to Java.

---

**Q2 (Why type erasure): Why did Java choose type erasure for generics?**

A: Java 5 (2004) needed to add generics while maintaining 100% backward
compatibility. The goal: existing pre-generics code (running on JDK 1.4)
must still run unmodified on JDK 5.

**Backward compatibility requirements:**
- Old code: `List list = new ArrayList();` (raw type)
- New code: `List<String> list = new ArrayList<>();`
- Both must compile and run on the same JVM

With type erasure: at runtime, `List<String>` is just `List`.
Old `List` methods work on new `List<String>` objects and vice versa.

```java
// Interoperability: old code using new generics code:
List<String> modern = new ArrayList<>();
modern.add("hello");

List legacy = modern; // raw type - loses type info
String s = (String) legacy.get(0); // old-style cast - works!
legacy.add(42); // compiles! unchecked warning - heap pollution
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Alternative (reified generics) would have:**
- Required JVM bytecode changes
- Broken binary compatibility
- New JVM class file format

Type erasure kept the JVM unchanged - only the compiler changed.

*What separates good from great:* The backward compatibility decision
was correct for Java's ecosystem scale (millions of JARs). But it created
the erasure limitations we live with today. The lesson: early design
decisions about type systems have extremely long-lasting consequences.
Kotlin (on the JVM) uses the same erasure but adds `reified` for inline
functions (which are inlined by the compiler, so the type info is
available at the call site). This is a clever escape hatch within the
JVM's limitations.

---

**Q3 (Generic array limitation): Why can't you create a generic array?
What are the workarounds?**

A: `new T[n]` is not allowed because arrays are covariant and generics
are invariant - combining them creates a type-unsafe situation.

```java
// Why it's disallowed:
// Suppose T[] was allowed:
T[] array = new T[10];  // if T = String at runtime
// But due to erasure, actual runtime type is Object[]:
Object[] arr = (Object[]) array; // legal for array covariance
arr[0] = 42;                     // no ArrayStoreException!
T elem = array[0];               // ClassCastException when reading!
// The compiler can't prevent this -> disallowed entirely

// Workaround 1: Object array + unchecked cast (ArrayList approach)
@SuppressWarnings("unchecked")
T[] arr = (T[]) new Object[n]; // flagged but safe if managed carefully

// Workaround 2: use List<T> instead of T[]
List<T> list = new ArrayList<>(n);

// Workaround 3: Class<T> token to create typed array
T[] arr = (T[]) java.lang.reflect.Array.newInstance(clazz, n);

// Workaround 4: have caller supply the array (Collection pattern)
T[] toArray(T[] arr) { ... } // caller provides typed array
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using authentication. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `ArrayList`'s internal `elementData`
is `Object[]` (never `T[]`) for exactly this reason. The `toArray(T[])`
method on `Collection` takes a typed array from the caller, who knows
the actual type at the call site: `list.toArray(new String[0])`. Passing
`new String[0]` (zero-length) is idiomatic - the actual array returned
may be larger; the argument's type is used for runtime reflection.

---

**Q4 (Bounded type parameters): What are bounded type parameters and
when do you use them?**

A: Bounded type parameters restrict what types can be used as the type argument.

**Upper bound (`extends`):**
```java
// T must be Number or a subclass:
<T extends Number> double sum(List<T> nums) {
    return nums.stream().mapToDouble(Number::doubleValue).sum();
}
// Works with List<Integer>, List<Double>, List<BigDecimal>
// Won't compile with List<String>
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

**Multiple bounds:**
```java
// T must extend Number AND implement Comparable:
<T extends Number & Comparable<T>> T max(List<T> list) {
    return list.stream().max(Comparator.naturalOrder())
               .orElseThrow();
}
// Works with Integer (extends Number, implements Comparable<Integer>)
// Won't work with AtomicInteger (extends Number, NOT Comparable)
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

**Recursive type bound (Comparable pattern):**
```java
// T compared to itself:
<T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Multiple bounds have one important
rule: at most ONE class bound (the first), rest must be interfaces.
`<T extends ArrayList & Comparable<T>>` is legal (ArrayList is a class,
Comparable is interface). `<T extends ArrayList & LinkedList>` is not
(two class bounds). This mirrors Java's single inheritance rule.
Wildcard bounds (`? extends`, `? super`) are different from type
parameter bounds and apply the PECS principle (covered in L3).

---

**Q5 (Raw types dangers): What are the risks of using raw types?**

A: Raw types (e.g., `List` without type parameter) bypass all generic
type checking. They exist for backward compatibility with pre-Java 5 code.

```java
// Raw type hazards:
List rawList = new ArrayList();   // raw type
rawList.add("hello");
rawList.add(42);                  // compiles! no type check

// Mixing raw and generic:
List<String> typed = rawList;     // unchecked assignment warning
String s = typed.get(1);         // ClassCastException at runtime!
                                  // 42 is not a String

// Raw type loses method return type info:
List rawList2 = List.of("a", "b");
String s2 = (String) rawList2.get(0); // requires cast
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Compile with `-Xlint:unchecked` to see all raw type usage warnings.
The compiler generates warnings but still compiles (backward compat).

*What separates good from great:* Raw types should NEVER appear in new
code. The only legitimate use: `instanceof` check against raw type
(can't check parameterized type due to erasure): `if (obj instanceof List)`.
A second legitimate use: `Class` literals: `List.class` (you can't write
`List<String>.class`). In code reviews: raw types in new code are a red
flag. Enable `-Xlint:unchecked` in build config and treat raw type warnings
as errors.

---

**Q6 (Super type token): What is the super type token pattern?**

A: Generic type info is erased at runtime. But a SUBCLASS that specializes
a generic superclass preserves the type info in its class file metadata.

```java
// Problem: generic type info lost at runtime
Type t = new ArrayList<String>() {}.getClass()
    .getGenericSuperclass(); // ParameterizedType!
// Anonymous class that extends ArrayList<String> preserves
// the String type parameter in its class file!

// Super type token pattern (Guava's TypeToken / Spring's ParameterizedTypeReference):
public abstract class TypeRef<T> {
    final Type type;
    protected TypeRef() {
        ParameterizedType superType =
            (ParameterizedType) getClass().getGenericSuperclass();
        type = superType.getActualTypeArguments()[0];
    }
}
// Usage:
TypeRef<List<String>> ref = new TypeRef<List<String>>() {};
// ref.type = ParameterizedType: List<String>
// The anonymous subclass preserves List<String> in its metadata!

// Spring RestTemplate uses ParameterizedTypeReference:
ResponseEntity<List<User>> resp = restTemplate.exchange(
    "/api/users",
    HttpMethod.GET,
    null,
    new ParameterizedTypeReference<List<User>>() {}
);
List<User> users = resp.getBody(); // typed! no cast needed
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using authentication. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The super type token is a clever workaround
for type erasure. The anonymous subclass `new TypeRef<List<String>>() {}`
has its parent type `TypeRef<List<String>>` encoded in its class file
metadata. This survives erasure because it's class HIERARCHY info, not
local variable type info. Jackson's `TypeReference<T>` uses the same
trick for deserialization: `mapper.readValue(json, new TypeReference<List<User>>() {})`.

---

**Q7 (Generic method vs class): When do you use a generic method
instead of a generic class?**

A:
```java
// Generic class: type parameter applies to the entire class
class Repository<T> {
    T findById(Long id) { ... }
    void save(T entity) { ... }
    List<T> findAll() { ... }
    // T is the same for all methods - it's a class-level concept
}

// Generic method: type parameter applies only to one method
class CollectionUtils {
    // This method works with any type - no need for a generic class:
    public static <T> List<T> repeat(T item, int times) {
        return Collections.nCopies(times, item);
    }

    // Multiple unrelated type parameters:
    public static <K, V> Map<V, K> invertMap(Map<K, V> original) {
        Map<V, K> inverted = new HashMap<>();
        original.forEach((k, v) -> inverted.put(v, k));
        return inverted;
    }
}
// Usage: type inference from arguments
List<String> strings = CollectionUtils.repeat("hello", 3);
Map<Integer, String> inverted = CollectionUtils.invertMap(codeToName);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Rule:** use generic class when the type parameter represents an
"owned" concept (Repository<User>). Use generic method when the type
is only relevant to that specific operation.

*What separates good from great:* Generic methods allow type INFERENCE -
the compiler deduces the type argument from the method arguments. For
static utility methods (algorithms, transformations), generic methods
are almost always preferable to wrapping in a generic class. Generic
methods can also have multiple independent type parameters (`<K, V>`)
which generic classes could handle but would require both parameters
to be specified at the class level, reducing reusability.

---

**Q8 (Suppress warnings unchecked): When is `@SuppressWarnings("unchecked")`
appropriate?**

A: `@SuppressWarnings("unchecked")` is appropriate ONLY when you can
verify the cast is safe through the program's invariants.


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// GOOD: documented safe cast in generic container:
class TypedCache<T> {
    private final Object[] storage;
    @SuppressWarnings("unchecked")
    T get(int index) {
        // Safe: only T is put in via put(T)
        return (T) storage[index];
    }
}

// GOOD: known framework interaction:
@SuppressWarnings("unchecked")
List<String> fromFramework = (List<String>) session.getAttribute("list");
// Documented: "list" attribute is always List<String>

// BAD: suppressing without understanding:
@SuppressWarnings("unchecked")
List<String> risky = (List<String>) getRandomObject(); // could fail!

// Rule: apply at the narrowest scope possible
// Don't suppress at class or method level - suppress just the line:
@SuppressWarnings("unchecked") // only this line
T item = (T) storage[index];  // not the whole method
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage using container. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Every `@SuppressWarnings("unchecked")
` should have a comment explaining WHY the cast is safe. Code review
question: "Why is this cast safe?" If the author can't explain it, the
annotation should be removed and the code redesigned. In security audits,
unchecked cast suppressions are red flags - each one needs justification.
A `@SuppressWarnings` without a comment is a code smell.

---

**Q9 (Heap pollution): What is heap pollution and how does it occur?**

A: Heap pollution: a variable of a parameterized type refers to an object
that is not of that parameterized type. The compiler cannot detect this
at runtime because of type erasure.

```java
// Heap pollution via varargs:
@SafeVarargs
static <T> void addToList(List<T> list, T... items) {
    for (T item : items) list.add(item);
}

// Heap pollution via raw type:
List<String> strings = new ArrayList<>();
List rawList = strings;       // raw type alias
rawList.add(42);               // compiles! puts Integer in List<String>
strings.get(0);                // returns "?" from original items
strings.get(strings.size()-1); // ClassCastException when reading!
// The heap now contains an Integer where String is expected - POLLUTED

// Safe varargs: @SafeVarargs guarantees method doesn't store to the
// varargs array in an unsafe way
@SafeVarargs
static <T> List<T> listOf(T... items) {
    return Arrays.asList(items); // only reads, doesn't store differently
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using gice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* `@SafeVarargs` is needed on generic
varargs methods because `T...` (varargs) creates a `T[]` internally -
which is actually `Object[]` after erasure. The compiler warns you that
it can't verify the method doesn't pollute the heap. `@SafeVarargs`
suppresses this warning when you guarantee: the method only reads from
the varargs array (doesn't store non-T values into it). `Collections.addAll()`,
`Arrays.asList()` are examples of `@SafeVarargs` methods in the JDK.

---

### ⚖️ Comparison Table

| Aspect| Generics| Pre-Generics (raw types)|
|------------------|----------------------------------|------------------------|
| Type checking| Compile-time| Runtime (cast required)|
| ClassCastException| Rare (caught at compile time)| Common|
| Readability| `List<String>` is self-documenting| `List` requires Javadoc|
| Collection API usage| No cast on get()| Explicit cast on get()|
| Runtime type info| Erased| N/A|
| Backward compat| Via raw types| Yes|

---

## Optional API

---

### 🎯 Model Answer

**30 seconds:**
> `Optional<T>` is a container that may or may not hold a value - a
> better alternative to returning `null` for "value may be absent."
> Created with `Optional.of(value)`, `Optional.ofNullable(value)`, or
> `Optional.empty()`. Key methods: `isPresent()`, `get()` (throws if
> empty), `orElse(default)`, `orElseGet(supplier)`, `orElseThrow()`,
> `map(fn)`, `flatMap(fn)`, `filter(predicate)`. Use Optional as a
> RETURN TYPE from methods to communicate "this may be absent." Do NOT
> use as a field, method parameter, or collection element.

**3 minutes (Senior):**
> The primary purpose: force callers to explicitly handle the "absent"
> case instead of ignoring a potential null. `Optional.of(null)` throws
> NPE (use `ofNullable` for possibly-null sources). `orElse()` evaluates
> its argument eagerly - even if the Optional is present. Use `orElseGet()`
> with a supplier for expensive defaults.
>
> Optional chains: `optional.map(String::toUpperCase).orElse("N/A")`
> is cleaner than `value != null ? value.toUpperCase() : "N/A"`.
> `flatMap`: when the mapping function itself returns an Optional:
> `user.flatMap(User::getAddress).map(Address::getCity).orElse("Unknown")`.
>
> Anti-patterns: `optional.isPresent() && optional.get()...` (defeats
> the purpose - just use `ifPresent()` or `map()`). Optional fields
> in classes (Optional is not Serializable). Optional collection
> elements (use `filter` on the stream instead).

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Optional API - let me cover its purpose, key methods,
proper use cases, and the anti-patterns to avoid."

**(2) First principles:** "From first principles: `null` is a single
pointer value that conflates 'not initialized', 'not found', 'not
applicable', and 'error'. Optional makes the 'not found' case explicit
in the type system, forcing callers to handle it."

**(3) Bridge:** "Optional is like a gift box that may or may not contain
a gift. You can peek (isPresent), open it safely (orElse - you get the
gift or a default), or chain operations on the contents (map). You don't
need to check if the box is empty before every operation - the box
handles it."

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Optional purpose | 60 seconds |
| orElse vs orElseGet | 2 minutes |
| map vs flatMap | 2 minutes |
| Optional anti-patterns | 2 minutes |
| Optional in APIs | 2 minutes |
| Stream and Optional | 2 minutes |
| Optional performance | 90 seconds |
| Optional vs null | 2 minutes |
| Java 9+ Optional additions | 90 seconds |

---

**Q1 (Optional purpose): What problem does Optional solve?**

A: `null` is ambiguous. When a method returns `null`, it can mean:
(a) "not found", (b) "not applicable", (c) "error occurred",
(d) "not yet initialized". Callers must guess which and handle
(or forget to handle) the case.

Tony Hoare called null "my billion-dollar mistake." In Java, forgetting
to check for null causes `NullPointerException` - one of the most common
Java runtime errors.

`Optional<T>` makes "not found" (or "absent") explicit in the type.
A method returning `Optional<User>` tells callers: "this might not
be there - handle both cases." The caller's code must explicitly deal
with the Optional to get the value; you can't accidentally pass an
Optional where a User is expected without unwrapping.

*What separates good from great:* Optional was introduced in Java 8
inspired by Haskell's `Maybe` and Scala's `Option`. But Java's Optional
is intentionally limited compared to those: not Serializable, no special
null handling in the JVM. The intent: use Optional ONLY for return types
from API methods where absence is a normal result. Not for all nullable
values (which would be unworkable). Not for fields (use @Nullable annotation
with static analysis tools like NullAway or FindBugs for field nullability).

---

**Q2 (orElse vs orElseGet): When is `orElseGet()` preferred over `orElse()`?**

A: `orElse(T other)`: the `other` argument is ALWAYS evaluated, even
if the Optional is present. It's like: `optional.isPresent() ? optional.get() : other`.

`orElseGet(Supplier<T> supplier)`: the supplier is called ONLY if the
Optional is empty. It's like: `optional.isPresent() ? optional.get() : supplier.get()`.

```java
// orElse is fine for cheap constants/literals:
String name = findName().orElse("Unknown"); // "Unknown" is a literal
String name = findName().orElse(DEFAULT_NAME); // field, no computation

// orElseGet is required for expensive operations:
User user = findUser(id)
    .orElseGet(() -> createDefaultUser(id)); // expensive if called

// Performance trap:
Optional<String> present = Optional.of("value");
present.orElse(generateReport()); // generateReport() ALWAYS runs!
present.orElseGet(this::generateReport); // only runs if empty
```

> **Code walkthrough:** This Unknown example demonstrates null-safe value wrapping using Optional. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

*What separates good from great:* The `orElse` vs `orElseGet` mistake
is common in production code and can cause significant performance
degradation. A real-world example: `findCachedUser().orElse(loadFromDatabase())`
- `loadFromDatabase()` runs even when the cache hits! This negates the
  entire purpose of the cache. The fix is one word: `orElseGet`.
  In code review: any `orElse(methodCall())` where the method does I/O,
  computation, or has side effects should be flagged for `orElseGet`.

---

**Q3 (map vs flatMap): When do you use `map` vs `flatMap` on Optional?**

A: `map(Function<T,R>)`: transform the value if present; the mapping
function returns a plain value R. Result: `Optional<R>`.

`flatMap(Function<T,Optional<R>>)`: the mapping function returns an
Optional itself. Without flatMap, you'd get `Optional<Optional<R>>`.
flatMap flattens to `Optional<R>`.

```java
// map: mapping function returns a plain value
Optional<String> name = findUser(id)
    .map(User::getName);  // User::getName returns String (not Optional)

// flatMap: mapping function returns Optional
Optional<Address> address = findUser(id)
    .flatMap(User::getAddress); // User::getAddress returns Optional<Address>

// Chain of flatMaps:
Optional<String> city = findUser(id)
    .flatMap(User::getAddress)      // Optional<Address>
    .flatMap(Address::getCity)      // Optional<String>
    .map(String::toUpperCase);      // String -> String (use map, not flatMap)

// Wrong: using map when flatMap is needed:
Optional<Optional<Address>> wrong = findUser(id)
    .map(User::getAddress);  // User::getAddress returns Optional<Address>
                             // so map returns Optional<Optional<Address>>!
```

> **Code walkthrough:** This Unknown example demonstrates null-safe value wrapping. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

*What separates good from great:* The map/flatMap distinction directly
mirrors the same distinction in streams and functional programming monads.
If you understand that Optional is a monad (a container with `map` and
`flatMap`), the rules are consistent: `map` for pure transformation,
`flatMap` for operations that produce the same container type. This
pattern appears in `CompletableFuture.thenApply(map)` vs
`thenCompose(flatMap)`, and in reactive streams.

---

**Q4 (Optional anti-patterns): What are the main Optional anti-patterns?**

A:

**1. Using `get()` without `isPresent()` - same as NPE:**

```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// BAD:
String name = findUser(id).get(); // NoSuchElementException if empty!
// GOOD:
String name = findUser(id).orElseThrow();
String name = findUser(id).orElseThrow(
    () -> new UserNotFoundException(id));
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

**2. Optional as method parameter:**
```java
// BAD: callers can pass Optional.empty() or null
void save(Optional<Attachment> attachment) { ... }
// GOOD: two methods or @Nullable annotation
void save() { ... }
void save(Attachment attachment) { ... }
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates null-safe value wrapping using Optional. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **WHAT BREAKS: prefer orElseThrow() with a meaningful message over bare get().**

**3. Optional as field:**
```java
// BAD: not Serializable, extra overhead
class User { private Optional<String> middleName; }
// GOOD: nullable field with documentation
class User { @Nullable private String middleName; }
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates null-safe value wrapping. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **WHAT BREAKS: prefer orElseThrow() with a meaningful message over bare get().**

**4. isPresent + get instead of ifPresent/map:**
```java
// BAD: defeats Optional purpose, verbose
if (opt.isPresent()) { process(opt.get()); }
// GOOD: functional style
opt.ifPresent(this::process);
opt.map(this::transform).ifPresent(this::process);
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates null-safe value wrapping. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **WHAT BREAKS: prefer orElseThrow() with a meaningful message over bare get().**

**5. Optional in collections:**
```java
// BAD: Optional as list element - just use filter instead
List<Optional<User>> users = ...;
// GOOD:
List<User> users = optionals.stream()
    .filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());
// Or Java 9+:
optionals.stream().flatMap(Optional::stream).collect(Collectors.toList());
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **WHAT BREAKS: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The Optional anti-patterns reveal
whether someone uses Optional idiomatically (as a functional container)
or defensively (as a verbose null check). The clearest signal: code
using `if (opt.isPresent()) { opt.get()... }` was written by someone
who learned `if (x != null) { x... }` and mechanically translated it.
Refactoring to `opt.ifPresent()` or `opt.map().orElse()` is not just
style - it's intent-driven code that communicates "this operation only
happens if the value is present."

---

**Q5 (Optional in APIs): When should a method return `Optional<T>`
vs `T` vs throwing an exception?**

A: Three cases:

**Return `Optional<T>` when:**
- Absence is a NORMAL, expected result (not an error)
- Examples: `findById(id)` (entity may not exist), `findFirst()` (list may be empty)
- Caller should handle both present and absent without exceptional code path

**Return `T` (possibly null) when:**
- Null is an accepted representation in the domain (though discouraged)
- Internal/private methods where callers are known to handle null
- Performance-critical paths where Optional boxing is unacceptable

**Throw exception when:**
- Absence is an ERROR condition (caller guaranteed existence)
- Examples: `getById(id)` (asserts entity exists; throws `EntityNotFoundException`)
- Required configuration is missing (startup-time failure)

```java
// Spring Data pattern:
Optional<User> findById(Long id);   // may not exist - Optional
User getById(Long id);              // must exist - throws if not

// Our code:
Optional<User> findUser(Long id) {
    return userRepo.findById(id);   // normal: user may not exist
}
User loadUser(Long id) {
    return findUser(id)
        .orElseThrow(() -> new UserNotFoundException(id)); // contract: must exist
}
```

> **Code walkthrough:** This Unknown example demonstrates null-safe value wrapping. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

*What separates good from great:* The naming convention `find*` (returns
Optional or null) vs `get*` (returns value or throws) is useful for
communicating the contract at the method signature level. Spring Data,
Hibernate, and most frameworks follow this convention. In API reviews:
a `findById` that throws instead of returning Optional, or a `getById`
that returns null instead of throwing, is a contract mismatch that causes
bugs.

---

**Q6 (Stream and Optional): How do streams and Optional interact in Java 9+?**

A:
```java
// Java 9: Optional.stream() - convert Optional to a 0-or-1 element Stream
Optional<User> user = findUser(id);
Stream<User> userStream = user.stream(); // empty stream or 1-element stream

// The key use case: flatMap with Optional in a stream:
List<Optional<User>> optUsers = List.of(
    Optional.of(new User("Alice")),
    Optional.empty(),
    Optional.of(new User("Bob"))
);

// Java 8 (verbose):
List<User> presentUsers = optUsers.stream()
    .filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());

// Java 9+ (clean):
List<User> presentUsers = optUsers.stream()
    .flatMap(Optional::stream) // empty Optionals disappear
    .collect(Collectors.toList()); // [Alice, Bob]

// Java 9: Optional.ifPresentOrElse:
findUser(id).ifPresentOrElse(
    user -> log.info("Found user: {}", user.getName()),
    () -> log.warn("User {} not found", id)
);

// Java 9: Optional.or() - supply alternative Optional:
Optional<User> userFromCache = cacheRepo.findUser(id);
Optional<User> user = userFromCache
    .or(() -> dbRepo.findUser(id)); // try DB if not in cache
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* `flatMap(Optional::stream)` is the
idiomatic Java 9+ pattern for filtering out empty Optionals from a stream.
Before Java 9, the `filter + map + get` pattern was required and was
error-prone (the `get()` could throw if filter was wrong). The Java 9
`or()` method is particularly useful for cache-then-database fallback
patterns: try cache first, then DB, without nested if statements.

---

**Q7 (Optional performance): Does Optional have a performance overhead?**

A: Yes, a small but measurable overhead in hot paths:

1. **Object allocation:** each `Optional.of(value)` creates an `Optional`
   object (~24 bytes on 64-bit JVM with header + reference). For methods
   called millions of times per second, this adds GC pressure.

2. **JIT optimization:** HotSpot JIT can optimize Optional away via
   escape analysis if the Optional doesn't escape the method:
   ```java
   Optional<String> opt = findName(); // may be optimized away by JIT
   return opt.orElse("default"); // JIT may inline and avoid allocation
   ```
> **Code walkthrough:** This Unknown example demonstrates null-safe value wrapping. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

This optimization is not guaranteed.

**When Optional overhead matters:**
- Tight inner loops: don't use Optional inside a loop called 100M times
- Primitive values: use `OptionalInt`, `OptionalLong`, `OptionalDouble`
  (avoid boxing + Optional wrapping):
  ```java
  OptionalInt maxAge = people.stream()
      .mapToInt(Person::getAge)  // IntStream
      .max();                    // OptionalInt (no boxing)
  ```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The Optional overhead is negligible
for typical business logic (HTTP request handling, database queries).
The cost of one Optional allocation is ~100ns; a database query is ~1ms.
The performance concern is only relevant for numeric computations and
very hot paths. Use `OptionalInt`/`OptionalLong`/`OptionalDouble` for
primitive streams to avoid boxing overhead - they're the correct API
for stream operations on primitives anyway.

---

**Q8 (Optional vs null): Should you always use Optional instead of null?**

A: No - Optional has specific use cases. The full recommendation:

**Use Optional for:**
- Public API return types where absence is expected
- Domain objects expressing "may or may not have a value"
- Chaining transformations on possibly-absent values

**Use null (with @Nullable annotation) for:**
- Fields: Optional is not Serializable; `@Nullable String` is cleaner
- Method parameters: overloading or builder pattern is cleaner
- High-performance code: avoid allocation overhead in hot paths
- Private/internal code where nulls are well-controlled

**Use exceptions for:**
- "Not found" that's an error (not an expected state)
- Required configuration or contract violations

**Reality of Java codebases (2024):**
Most existing Java code uses null extensively (especially older code).
Modern code should use Optional for public API return types. Static
analysis tools (NullAway, SpotBugs, IntelliJ's nullable annotations)
handle field/parameter nullability without Optional overhead.

*What separates good from great:* The "use Optional everywhere" approach
fails because Optional was not designed for fields (performance, Serializable),
parameters (verbose, unhelpful), or collection elements (use filter).
Kotlin took a different approach: nullable types (`String?` vs `String`)
at the language level, making null safety a first-class concern throughout.
Many Java teams adopt @Nullable/@NonNull annotations with NullAway for
field/parameter nullability and reserve Optional for return types.
This hybrid gives the best of both worlds.

---

**Q9 (Java 9+ Optional additions): What new Optional methods were added
in Java 9-11?**

A:

| Method | Version | Description |
|---|---|---|
| `or(Supplier<Optional<T>>)` | Java 9 | Fallback to another Optional if empty |
| `ifPresentOrElse(Consumer, Runnable)` | Java 9 | Handle both present and empty |
| `stream()` | Java 9 | Convert to 0 or 1 element Stream |
| `isEmpty()` | Java 11 | Returns true if no value present (not `!isPresent()`) |

```java
// or(): chain fallback Optionals
Optional<Config> config = fromEnvVar("DB_URL")
    .or(() -> fromSystemProperty("db.url"))
    .or(() -> Optional.of(Config.DEFAULT));

// ifPresentOrElse(): handle both cases in one call
userOptional.ifPresentOrElse(
    user -> metrics.userFound(user.getId()),
    () -> metrics.userNotFound()
);

// isEmpty(): cleaner than !isPresent()
if (optional.isEmpty()) {
    throw new IllegalStateException("Required value missing");
}

// stream(): unwrap in stream pipelines
List<User> users = ids.stream()
    .map(this::findUser)        // Stream<Optional<User>>
    .flatMap(Optional::stream)  // Stream<User> (empties removed)
    .collect(Collectors.toList());
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The `or()` method enables the "fallback chain"
pattern cleanly - try source A, then B, then C. Before Java 9, this required
nested `isPresent` checks or `.map(...).orElseGet(...)` chains. The `isEmpty()`
method was added purely for readability - `optional.isEmpty()` reads more
naturally than `!optional.isPresent()` in conditions like `if (optional.isEmpty()) throw ...`.

---

### ⚖️ Comparison Table

| Approach | Null | Optional | Exception |
|---|---|---|---|
| Absence semantics | Implicit | Explicit in type | N/A (error, not absence) |
| Compile-time enforcement | No | Yes (unwrap required) | No |
| Serializable | Yes | No | Yes |
| Performance | Fastest | Small overhead | High (exception creation) |
| Chaining | Verbose null checks | Clean (map/flatMap) | Try-catch blocks |
| Use case | Fields, internal | Public return types | Error conditions |

---

---

# Java Core - L2 HashMap Internals

## HashMap Internals and Hash Collision

---

### 🎯 Model Answer

**30 seconds:**
> `HashMap` stores entries in a hash table - an array of "buckets".
> `put(k,v)` computes `k.hashCode()`, spreads it with a secondary hash,
> and selects a bucket: `bucket = hash & (capacity - 1)`. If two keys
> land in the same bucket (collision), they form a linked list (Java 7)
> or a red-black tree when the list exceeds 8 nodes (Java 8). Average
> O(1) for `get/put`, worst case O(log n) with treeification (Java 8)
> vs O(n) without. The load factor (default 0.75) controls when the
> table resizes - at 75% fill, capacity doubles and all entries rehash.

**3 minutes (Senior):**
> Java 8's treeification (JEP 180) was motivated by hash DoS attacks:
> an adversary knowing the hash function could craft keys that all land
> in one bucket, degrading HashMap to O(n) per lookup. With treeification,
> the worst case becomes O(log n). Treeification requires keys to be
> `Comparable` - if not, the tree falls back to identity hash comparison.
>
> Performance factors: initial capacity choice matters. Default 16.
> With load factor 0.75, first resize at 12 entries. For 1000 entries:
> 7 resizes. Pre-size with `new HashMap<>(1000 / 0.75 + 1)` = 1334 to
> avoid any resize. Resizing is O(n) and rehashes every entry.
>
> HashMap is not thread-safe. Multiple threads modifying concurrently:
> lost updates, infinite loops in Java 6 (fixed in Java 8), data
> corruption. Use `ConcurrentHashMap` for concurrent access.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

*Adapting up:* Discuss the HashMap secondary hash function (`(h = key.hashCode()) ^ (h >>> 16)` - XOR of high and low bits to spread entropy), the power-of-2 table size optimization (allows bitwise AND instead of modulo), and `ConcurrentHashMap`'s segment/stripe design evolution from Java 7 (16 segments) to Java 8 (CAS on individual nodes).

**Blank Mind Recovery:**

**(1) Restate:** "HashMap internals - let me cover the bucket array
structure, hash computation, collision handling, treeification in
Java 8, and load factor/resizing."

**(2) First principles:** "From first principles: we want O(1) key
lookup. An array gives O(1) by index, but we need to map arbitrary
keys to array indices. That's the hash function's job. Collisions
are inevitable (pigeonhole principle), so we need a strategy to
handle them."

**(3) Bridge:** "HashMap is like a parking garage with numbered slots.
The license plate (key) is hashed to a slot number. If the slot is
taken (collision), you chain cars behind it (linked list/tree). When
the garage is 75% full, you build a bigger garage and move everyone."

---

### 📘 Concept Explanation

**Internal structure (Java 8):**
```
HashMap state:
  Node<K,V>[] table     // the bucket array (size = power of 2)
  int size              // number of key-value pairs
  int threshold         // size * loadFactor = resize trigger
  float loadFactor      // default 0.75
  int modCount          // structural modification count (for CME)

Node<K,V>:
  int hash              // cached hashCode
  K key
  V value
  Node<K,V> next        // linked list chain (or TreeNode for trees)

TreeNode<K,V> extends Node<K,V>:
  // Red-Black tree node for buckets with > 8 entries
  TreeNode parent, left, right, prev
  boolean red
```

> **Code walkthrough:** This L2 HashMap Internals example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

**put(k, v) algorithm:**
```
1. if table is null: initialize table to size 16

2. hash = hash(key.hashCode())
   // spread: h ^ (h >>> 16)
   // ensures upper bits affect lower bits (reduces collisions)

3. i = (table.length - 1) & hash
   // bucket index: bitwise AND (fast modulo for power-of-2)

4. if table[i] == null:
   table[i] = new Node(hash, key, value, null)   // empty bucket

5. else (bucket occupied):
   if (table[i].hash == hash && table[i].key.equals(key)):
       replace value   // exact same key

   else if table[i] is TreeNode:
       tree.putTreeVal(...)   // insert into tree

   else (linked list):
       walk list; if key found, replace; if end reached, append Node
       if list length >= TREEIFY_THRESHOLD (8):
           treeifyBin(table, i)   // convert to red-black tree

6. if ++size > threshold:
   resize()   // double capacity, rehash all entries
```

> **Code walkthrough:** This L2 HashMap Internals example demonstrates a key concept in practice using SQL. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

---

### 💻 Code Example

> **Code walkthrough:** The pre-sizing example shows the correct formulaice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**
> to avoid any resize operations. The hash DoS attack example explains
> why Java 8 added treeification. The thread-safety failure shows why
> concurrent HashMap access without synchronization corrupts state in
> Java 8+ (deadlock / lost updates; pre-Java 8 could cause infinite loops
> during concurrent resize).

```java
// PRE-SIZING: avoid resize with known entry count
int expectedEntries = 1000;
// Correct: initialCapacity = entries / loadFactor + 1
// (ensures capacity * loadFactor > entries, no resize needed)
Map<String, User> users = new HashMap<>(
    (int)(expectedEntries / 0.75) + 1  // = 1335
);

// HASH COLLISION: all keys in one bucket - bad performance
// (Before Java 8: O(n) per lookup; Java 8+: O(log n) after tree)
Map<String, String> map = new HashMap<>();
// All these hash to bucket 0 if hash function is broken:
for (int i = 0; i < 100; i++) {
    map.put("key" + i, "value" + i);  // normal case: ~1 per bucket
}

// THREAD-SAFETY FAILURE:
Map<String, Integer> counter = new HashMap<>(); // NOT thread-safe
// Multiple threads calling counter.put() concurrently:
// - Lost updates (two threads write same bucket simultaneously)
// - ConcurrentModificationException during concurrent resize

// FIX: use ConcurrentHashMap:
Map<String, Integer> safeCounter = new ConcurrentHashMap<>();
safeCounter.computeIfAbsent("key", k -> 0);
safeCounter.compute("key", (k, v) -> v == null ? 1 : v + 1); // atomic

// Or for counting: ConcurrentHashMap.merge():
safeCounter.merge("key", 1, Integer::sum); // atomic add
```

> **Code walkthrough:** `computeIfAbsent`, `compute`, and `merge` areice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**
> atomic operations in `ConcurrentHashMap` - they execute atomically
> on the affected bucket without external synchronization. `merge("key",
> 1, Integer::sum)` atomically sets the value to 1 if absent, or
> computes `existingValue + 1` if present. This is the idiomatic
> concurrent counter pattern. `ConcurrentHashMap` achieves this by
> using synchronized blocks only on the affected bucket, not the entire
> map.

---

### 🎓 Answers by Seniority

**Junior / Mid (0-5 years):**
> HashMap uses a hash table: hash codes map keys to bucket indices.
> Collisions are handled by linked lists (Java 7) or trees for long
> chains (Java 8). Default capacity 16, load factor 0.75. The table
> doubles when 75% full. Use `equals()` and `hashCode()` correctly or
> keys can't be found. Not thread-safe - use `ConcurrentHashMap` for
> concurrent access.

---

**Senior / Staff (5+ years):**
> Java 8 treeification (bucket list -> red-black tree at 8 nodes) was
> a security fix for hash DoS attacks, improving worst-case from O(n)
> to O(log n). The secondary hash (`h ^ h >>> 16`) spreads entropy from
> upper bits into lower bits - critical because bucket selection uses
> only the lower bits (`hash & (capacity-1)`). Poor hashCode() (e.g.,
> all low bits the same) concentrates keys in few buckets. For
> performance-critical maps: consider Guava's `ImmutableMap` (array-
> backed, no boxing), Eclipse Collections `UnifiedMap` (lower memory),
> or JDK's `EnumMap` for enum keys (array, O(1) everything).

---

### ⚠️ Common Misconceptions

**Misconception 1: "HashMap is O(1) always."**
Average O(1) with good hash distribution. Worst case:
O(n) Java 7 (all keys in one bucket), O(log n) Java 8+
(treeification). The load factor and quality of hashCode
both affect actual performance.

**Misconception 2: "Collections.synchronizedMap(map) is as good as
ConcurrentHashMap."**
`synchronizedMap` wraps the map in a monitor lock - only one thread
can access it at a time. `ConcurrentHashMap` uses bucket-level
synchronization (for writes) and lock-free reads - much higher
concurrency for read-heavy workloads.

---

### 🚨 Failure Modes and Diagnosis

**Failure: HashMap with all-zero hashCode causes O(n) or O(log n) degradation.**
```java
// Bad hashCode: always returns 0
class BadKey {
    int id;
    @Override public int hashCode() { return 0; } // terrible!
    @Override public boolean equals(Object o) { ... }
}
Map<BadKey, String> map = new HashMap<>();
// All entries land in bucket 0: one bucket with n entries
// get() walks the whole bucket: O(log n) with treeification
```
> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

Diagnosis: `jstack` thread dump showing threads spending time in
`HashMap.getEntry()`; heap dump showing one bucket with thousands
of nodes.

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| HashMap put algorithm | 2-3 minutes |
| Collision handling | 2 minutes |
| Java 8 treeification | 2 minutes |
| Load factor and resizing | 2 minutes |
| HashMap vs ConcurrentHashMap | 2-3 minutes |
| HashMap thread-safety failures | 2 minutes |
| LinkedHashMap use cases | 2 minutes |
| EnumMap advantages | 90 seconds |
| Pre-sizing formula | 90 seconds |
| HashMap vs Hashtable | 60 seconds |
| hashCode null key | 60 seconds |
| HashMap internal hash function | 2 minutes |

---

**Q1 (HashMap put algorithm): Walk through what happens when you call
`map.put("key", "value")` on a HashMap.**

A:
1. `key.hashCode()` is called. For String, this is a polynomial rolling
   hash of the characters.

2. A secondary hash is applied: `h ^ (h >>> 16)`. This XORs the high
   16 bits of the hash code into the low 16 bits. Purpose: the bucket
   index uses only `log2(capacity)` bits. For a 16-bucket table: only
   the lowest 4 bits matter. XOR spreads entropy from all 32 bits.

3. Bucket index: `i = (capacity - 1) & finalHash`. For capacity=16:
   `i = 15 & hash = hash % 16`.

4. Examine `table[i]`:
  - null: new `Node(hash, key, value, null)` placed here
  - `Node` with equal key (hash match + equals): update value
  - `Node` with different key (collision): walk the chain

5. At end of chain: add new `Node`. If chain length >= 8:
   convert to red-black tree (`treeifyBin`).

6. `size++`. If `size > threshold (= capacity * loadFactor)`:
   call `resize()` - double capacity, rehash all entries.

*What separates good from great:* The resize is the expensive operation
to avoid. Each resize: allocates new array, walks every entry, recomputes
bucket for each entry (but cheats: since new capacity is 2x old, the
new bucket is either same index or `old_index + old_capacity` - no full
hash recomputation needed). Amortized cost of n puts: O(n).

---

**Q2 (Collision handling): How does HashMap handle key collisions?**

A: Collision = two keys have the same bucket index (same `hash &
(capacity-1)`). Note: this doesn't require same hash code; different
hash codes can map to the same bucket.

**Chaining strategy (HashMap's approach):**
Multiple entries in the same bucket form a chain. Java 7: singly-linked
list. Java 8: singly-linked list until 8 nodes, then red-black tree.

```
Bucket 3: [entry(k1,v1)] -> [entry(k2,v2)] -> null  (linked list)
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

**get() on a collision bucket:**
1. Compute bucket index
2. Walk the chain: check `hash == e.hash && (key == e.key || key.equals(e.key))`
3. Return value if found, null otherwise

**Why check hash first?**
`hash == e.hash` is a cheap int comparison. `equals()` may be expensive
(String compares characters). Short-circuit: most entries in a bucket
won't have matching hash codes even if their bucket indices match.

*What separates good from great:* The quality of hash distribution is
the most important performance factor. A perfect hash puts exactly one
entry per bucket: zero collisions, pure O(1). A pathological hash
puts all entries in one bucket: O(log n) in Java 8. Real-world: with
default load factor 0.75 and good hashCode, average bucket occupancy
is 0.75 entries (most buckets empty, few have 1 entry, very rare to
have 2+). Poisson distribution predicts ~2% of buckets will have 2+
entries in this scenario.

---

**Q3 (Java 8 treeification): Why did Java 8 add tree buckets to HashMap?**

A: Java 8 (JEP 180: "Handle Frequent HashMap Collisions with Balanced
Trees") added treeification for two reasons:

**1. Algorithmic complexity improvement:**
Before: O(n) worst-case per lookup (one bucket with n entries)
After: O(log n) worst-case per lookup (treeified bucket)

**2. Hash DoS attack mitigation:**
An adversary who knows the HashMap hash function (it's in the JDK source)
can craft keys that all hash to the same bucket. Sending 10,000 crafted
keys through a REST API can degrade `HashMap.get()` to O(n):
10,000 operations of O(10,000) = 100,000,000 operations.
Java 8 treeification reduces this to O(10,000 * log(10,000)) ≈ 130,000 ops.

**Treeification details:**
- Threshold: 8 entries in a bucket triggers treeification
- Untreeify threshold: 6 entries (after removals, tree -> list)
- Minimum table size for treeification: 64 (small tables may resize first)
- Requires keys to be `Comparable` for tree ordering; falls back to
  identity-based ordering if keys don't implement `Comparable`

*What separates good from great:* For String keys, Java 8 also introduced
a "hash randomization" concept (randomized hash seed) in some JVM
implementations to prevent reproducible hash DoS. However, `HashMap`
itself uses a deterministic hash in the standard JDK. The randomization
is in web frameworks that handle untrusted input (e.g., Tomcat uses
`useBodyEncodingForURI` and request parameter limiting). Defense in depth:
the treeification handles the performance degradation; input size limits
handle the attack surface.

---

**Q4 (Load factor and resizing): What is the load factor and how does
it affect HashMap performance?**

A: Load factor = `size / capacity`. Default: 0.75. Resize threshold:
when `size > capacity * loadFactor`.

**Effects of load factor:**
- Lower load factor (e.g., 0.5): fewer collisions (more empty buckets),
  faster lookups, but wastes memory (half the buckets empty on average)
- Higher load factor (e.g., 0.9): more collisions, slower lookups,
  but less memory waste

**Resize mechanics:**
```java
void resize() {
    int newCapacity = oldCapacity * 2;
    Node<K,V>[] newTable = new Node[newCapacity];
    // Rehash all entries:
    for (each entry in table) {
        int newBucket = (newCapacity - 1) & entry.hash;
        // Move to newTable
    }
    table = newTable;
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

**Java 8 resize optimization:**
New bucket index = old index OR (old index + oldCapacity).
Since capacity doubles (power of 2), only 1 new bit is added to the
bucket calculation. No need to recompute hash:
```plaintext
old capacity = 16 (0001 0000)
new capacity = 32 (0010 0000)
new bit = hash & oldCapacity (the bit that changed)
if new bit == 0: new index = old index
if new bit == 1: new index = old index + oldCapacity
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* The default 0.75 load factor is a
good empirical compromise. For read-heavy, memory-available caches:
use 0.5 for fewer collisions. For memory-constrained environments with
predictable data: use 0.9. The initial capacity choice is more impactful
than load factor tuning. Formula: `initialCapacity = n / loadFactor + 1`
where n = expected entries. For 10K entries with 0.75 load factor:
`10000 / 0.75 + 1 = 13335` initial capacity.

---

**Q5 (HashMap vs ConcurrentHashMap): Compare HashMap and
ConcurrentHashMap for concurrent use.**

A:

| Aspect | HashMap | ConcurrentHashMap |
|---|---|---|
| Thread safety | No | Yes |
| Null keys | One allowed | Not allowed |
| Null values | Allowed | Not allowed |
| Read performance | Fastest | Near-HashMap (no lock for reads) |
| Write performance | Fastest | Good (bucket-level lock) |
| Atomic operations | No | computeIfAbsent, merge, compute |
| Size accuracy | Exact | Approximate (size()) |
| Iteration | Fail-fast | Weakly consistent |

**ConcurrentHashMap Java 8 internals:**
- Reads: lock-free (volatile access)
- Writes: `synchronized (bucket)` - only locks the specific bucket node
- Atomic CAS (Compare-And-Swap) for bucket insertion
- No global lock: 16+ threads can write to different buckets simultaneously

**Why not `Collections.synchronizedMap()`?**
```java
// synchronizedMap: global lock, one thread at a time:
Map<K,V> syncMap = Collections.synchronizedMap(new HashMap<>());
syncMap.put(k1, v1); // locks entire map
syncMap.get(k2);     // also locks entire map - no read concurrency

// ConcurrentHashMap: fine-grained, parallel reads:
ConcurrentHashMap<K,V> chm = new ConcurrentHashMap<>();
chm.put(k1, v1); // locks bucket[i]
chm.get(k2);     // no lock (volatile read) - parallel with puts!
```

> **Code walkthrough:** This Unknown example demonstrates mutex locking using concurrency primitive. **KEY MECHANISM:** the JVM acquires the intrinsic lock on the object monitor before entering the block. **WHY IT MATTERS:** a thread holding the lock blocks all other threads - a bottleneck at scale. **TAKEAWAY: prefer ReentrantLock or ConcurrentHashMap over synchronized for hot paths.**

*What separates good from great:* ConcurrentHashMap's `size()` method
returns an approximate count (uses the internal `CounterCell` accumulator,
similar to `LongAdder`). For exact counting in concurrent systems: maintain
a separate `AtomicLong` counter. Also: ConcurrentHashMap doesn't support
null keys or values - this is intentional. Null in a regular HashMap
means "key absent" (return null from get). In ConcurrentHashMap, null
could mean "key absent" OR "value is null" - ambiguous under concurrency.
Eliminating null forces explicit absence representation.

---

**Q6 (HashMap thread-safety failures): What failures can occur when
multiple threads use HashMap concurrently?**

A:

**1. Lost updates:**
```
Thread A: get("counter") -> 5
Thread B: get("counter") -> 5
Thread A: put("counter", 6)   // +1
Thread B: put("counter", 6)   // +1, overwrites A's write
// Expected: 7. Actual: 6. Lost update.
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice using SQL. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

**2. ConcurrentModificationException:**
Thread A iterates; Thread B puts (changes modCount).
Thread A's iterator detects modCount change -> CME.

**3. Infinite loop (Java 6 and earlier - fixed in Java 8):**
Pre-Java 8 HashMap resize used head insertion for the new chain.
Two concurrent threads doing resize could create a circular linked list.
Any subsequent get() on an entry in that bucket enters an infinite loop.
(Java 8 uses tail insertion and preserves order, fixing this.)

**4. Partial initialization:**
Partially constructed entries visible due to memory ordering.
Without volatile or synchronization: another thread may see a node
with null key or hash = 0 during construction.

*What separates good from great:* The infinite loop bug (pre-Java 8) was
a production catastrophe for teams that discovered HashMap "isn't thread
safe in theory" was actually "HashMap can hang your server" in practice.
The symptoms: CPU at 100%, thread dump shows all threads stuck in
`HashMap.get()`. The fix (Java 8 tail insertion) was a silent correctness
improvement alongside the tree buckets. Even with Java 8, HashMap is not
safe for concurrent access - the lost update and CME issues remain.

---

**Q7 (LinkedHashMap use cases): When would you use LinkedHashMap?**

A: `LinkedHashMap` extends `HashMap` by maintaining a doubly-linked list
through all entries in INSERTION order (or access order if specified).

**Use case 1: Preserve insertion order**
```java
Map<String, Integer> config = new LinkedHashMap<>();
config.put("host", 0);
config.put("port", 1);
config.put("timeout", 2);
// Iteration order matches insertion: host, port, timeout
// HashMap would give random order
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using SQL. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Use case 2: LRU Cache skeleton**
```java
// access-order LinkedHashMap: most-recently-accessed last
Map<String, Object> lruCache = new LinkedHashMap<>(16, 0.75f, true) {
    private static final int MAX_SIZE = 100;
    @Override
    protected boolean removeEldestEntry(Map.Entry<String,Object> eldest) {
        return size() > MAX_SIZE; // evict when full
    }
};
// get() moves entry to end (most recent); removeEldestEntry evicts head
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

**Use case 3: JSON serialization order**
JSON object properties have no required order, but tools
(logs, diffs) are easier to read with consistent field order.
`LinkedHashMap` for DTO-to-JSON conversion preserves field order.

*What separates good from great:* The LRU cache use case is common in
interviews. `LinkedHashMap(capacity, loadFactor, true)` (access-order
mode) reorders entries on every `get()` - the accessed entry moves to
the tail. Override `removeEldestEntry()` to evict when size exceeds
the limit. The complexity: `get/put/remove` are still O(1) but with
pointer maintenance for the linked list. Caffeine cache (modern Java
caching library) is a production-grade LRU/LFU/W-TinyLFU implementation
with better concurrency than `LinkedHashMap`.

---

**Q8 (EnumMap advantages): What are the advantages of EnumMap over HashMap
for enum keys?**

A: `EnumMap` uses a plain array indexed by the enum ordinal. No hashing,
no collision, no boxing.


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

// GOOD: EnumMap for enum keys
Map<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MON, "Sprint planning");
schedule.put(Day.FRI, "Retrospective");

// Performance comparison:
// HashMap<Day, V>: hash Day -> box if needed -> find bucket
// EnumMap<Day, V>: array[Day.ordinal()] -> O(1), no boxing, no hashing

// Size: 7 (Days.values().length) array slots vs HashMap's 16+ buckets
// Memory: compact, no Node objects, no pointer chains
// Iteration: always in enum declaration order (predictable)
```

> **Code walkthrough:** GOOD pattern: This Unknown example demonstrates Java API usage using enum. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**When to use EnumMap:**
- Map keys are always from a known enum
- Performance-critical code with enum keys
- Need predictable iteration order (enum declaration order)
- Particularly useful for: state machines (state -> handler), case
  dispatch (status -> action), configuration by environment

*What separates good from great:* `EnumMap` is the correct choice
whenever your keys are an enum - it's not a micro-optimization; it's
semantically more correct. The map size is bounded by the enum constants
(no growth). The ordinal-based storage is cache-line friendly.
`EnumSet` is the corresponding `Set` implementation for enum values
(uses a bitfield internally for extremely compact storage and O(1) ops).

---

**Q9 (Pre-sizing formula): What's the formula to pre-size a HashMap
to avoid resizing?**

A:
```java
// Formula: initialCapacity = (expectedEntries / loadFactor) + 1
// This ensures capacity * loadFactor >= expectedEntries

int expected = 1000;
float loadFactor = 0.75f;
int initialCapacity = (int)(expected / loadFactor) + 1; // 1335

Map<String, User> map = new HashMap<>(initialCapacity);
// HashMap rounds up to next power of 2: 2048
// With 1000 entries: 1000 < 2048 * 0.75 = 1536 -> no resize!

// Alternative: let Guava compute it
Map<String, User> guavaMap = Maps.newHashMapWithExpectedSize(1000);
// Guava uses (expected * 4 / 3) + 1 = same formula
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Why round up to power of 2?**
HashMap requires capacity to be a power of 2 for the `(n-1) & hash`
bitwise bucket calculation. It automatically rounds up to the next
power of 2.

*What separates good from great:* Pre-sizing matters for batch operations:
loading 100K records from a database into a HashMap. Without pre-sizing:
8+ resize/rehash operations during loading, temporary 2x memory usage
during each resize. With pre-sizing: single allocation, zero resizes.
The temporary 2x memory during resize can cause OOM in memory-constrained
environments. Profile-driven pre-sizing (from initial `COUNT(*)` query)
is a standard pattern in high-throughput data processing.

---

**Q10 (HashMap vs Hashtable): When would you use Hashtable instead of HashMap?**

A: Almost never. `Hashtable` is legacy (Java 1.0) and obsolete.

| Aspect | HashMap | Hashtable |
|---|---|---|
| Thread safety | No | Yes (synchronized) |
| Null keys | One allowed | Not allowed |
| Null values | Allowed | Not allowed |
| Iteration | Iterator (fail-fast) | Enumeration (not fail-fast) |
| Performance | Faster (no sync) | Slower (global sync) |
| Inheritance | AbstractMap | Dictionary (legacy) |
| Modern use | Yes | No - use ConcurrentHashMap |

`Hashtable`'s synchronization is coarse-grained (entire table locked).
`ConcurrentHashMap` is the modern thread-safe replacement with much
better concurrency.

*What separates good from great:* `Hashtable` exists only for backward
compatibility. You might encounter it in legacy code (pre-Java 5).
When modernizing: replace `Hashtable` with `ConcurrentHashMap` (for
concurrent access) or `HashMap` (for single-threaded). The same applies
to `Vector` -> `ArrayList` (or `CopyOnWriteArrayList`) and `Stack` ->
`ArrayDeque`.

---

**Q11 (hashCode null key): How does HashMap handle null keys?**

A: HashMap allows exactly one null key, stored at bucket index 0.

```java
map.put(null, "value");  // stored at index 0
map.get(null);           // returns "value"
map.containsKey(null);   // true
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Implementation:
```java
// HashMap special-cases null key:
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// null -> hash = 0 -> bucket[0]
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**ConcurrentHashMap does NOT allow null keys or null values.**
Reason: in concurrent access, `map.get(key) == null` is ambiguous:
did get return null because the key is absent, or because the value
is null? With non-concurrent HashMap, you can `containsKey()` to
distinguish. Under concurrent access, the state may change between
`get()` and `containsKey()`.

*What separates good from great:* Using null as a map key is a code
smell. It typically means "absent" key - but the map already has the
concept of absent keys. If null key means "unknown", use an `Optional`
or a sentinel value instead: `Map.getOrDefault(key, DEFAULT)`. Null
values similarly: if null value means "deleted" or "absent", use
`Optional<V>` as the value type or remove the entry instead.

---

**Q12 (HashMap internal hash function): Why does HashMap use
`h ^ (h >>> 16)` as a secondary hash?**

A: The secondary hash (also called hash "spreading"):

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Why needed:** Bucket index uses only the lower bits of the hash:
`(capacity - 1) & hash`. For a 16-bucket table: only 4 bits matter.
For a 256-bucket table: only 8 bits matter. If the hash code has poor
entropy in its low bits (common for integer keys, class-generated hash
codes), many entries cluster in few buckets.

**XOR with upper bits:**
`h ^ (h >>> 16)` mixes the high 16 bits of `h` into the low 16 bits.
Now the low bits contain entropy from the entire 32-bit hash code.
This reduces clustering without additional computation.

**Example:**
```
key.hashCode() = 0xFFFF_0001
h >>> 16 = 0x0000_FFFF
h ^ (h >>> 16) = 0xFFFF_FFFE
// Low 4 bits: 0001 -> 1110 (changed by upper bits)
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* This secondary hash is a practical
engineering compromise. A full cryptographic hash would give near-perfect
distribution but be much slower. The XOR spreading is O(1), requires
two CPU instructions, and significantly reduces clustering for common
hashCode implementations (integer keys, String hash codes). This is
the kind of detail that shows you've read the HashMap source code -
a strong signal in senior-level interviews.

---

### ⚖️ Comparison Table

| Map Type| Order| Null key| Thread-safe| Get O| Add O| Use When|
|---|---------|--------|------------|--------|--------|------------------------|
| HashMap| None| One| No| O(1) avg| O(1) avg| General use|
| LinkedHashMap| Insert/access| One| No| O(1) avg| O(1) avg| LRU, ordered output
| TreeMap| Sorted| No| No| O(log n)| O(log n)| Sorted key range queries|
| EnumMap| Enum order| No| No| O(1)| O(1)| Enum key maps|
| ConcurrentHashMap| None| No| Yes| O(1) avg| O(1) avg| Concurrent access|
| Hashtable| None| No| Yes (global)| O(1) avg| O(1) avg| Legacy only|

---


---

## Comparable and Comparator

---

### 🎯 Model Answer

**30 seconds:**
> `Comparable<T>` provides the NATURAL ordering of a class: implemented
> inside the class (`compareTo()`) to say "objects of this type are
> naturally ordered this way." `Comparator<T>` is an EXTERNAL ordering:
> a separate object defining a specific comparison strategy. Use
> `Comparable` for the primary, natural order (String alphabetical,
> Integer numeric). Use `Comparator` when you need multiple orderings,
> ad-hoc ordering, or you can't modify the class. `Comparator.comparing()`,
> `.thenComparing()`, `.reversed()` build readable comparison chains.

**3 minutes (Senior):**
> `compareTo()` contract: returns negative (this < other), zero (equal),
> positive (this > other). Must be consistent with equals: if
> `a.compareTo(b) == 0` then `a.equals(b)` should be true (TreeMap,
> TreeSet correctness depends on this). Violating this creates invisible
> bugs in sorted collections.
>
> Java 8 `Comparator` factory methods:
> `Comparator.comparing(Person::getName)` - by name
> `.thenComparing(Person::getAge)` - then by age
> `.reversed()` - reverse the order
> `Comparator.nullsFirst(Comparator.naturalOrder())` - nulls first
>
> `Arrays.sort()` and `Collections.sort()` use `TimSort` which requires
> consistent total order (anti-symmetry, transitivity, totality).
> Inconsistent comparators cause undefined behavior in sort.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Comparable vs Comparator - let me cover the difference,
the compareTo contract, Comparator.comparing() factory methods, and
when to use each."

**(2) First principles:** "From first principles: to sort objects, we
need to answer 'is A < B, A == B, or A > B?' This can be inherent to
the type (String alphabetical) or external (sort users by age for one
use case, by name for another)."

**(3) Bridge:** "Comparable is like a person who has a birth certificate
(natural age ordering). Comparator is like a judge at a talent show
who applies specific scoring criteria. You need both: the natural order
exists, but sometimes you need custom ordering criteria."

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Comparable vs Comparator | 90 seconds |
| compareTo contract | 2 minutes |
| Subtraction comparator bug | 90 seconds |
| Comparator.comparing() chains | 2 minutes |
| Consistent with equals | 2 minutes |
| Natural order primitives | 60 seconds |
| Reverse ordering | 90 seconds |
| Null-safe comparison | 90 seconds |
| Custom multi-field sort | 2 minutes |

---

**Q1 (Comparable vs Comparator): When do you use Comparable vs Comparator?**

A: **Comparable:** implement on a class to define its natural order.
One ordering per class. The class "knows" how to compare itself.
Examples: `String` (alphabetical), `Integer` (numeric), `LocalDate` (chronological).

**Comparator:** an external object defining a specific ordering.
Multiple comparators possible per class. The class doesn't need to implement anything.
Use when: multiple orderings needed, can't modify the class, ad-hoc ordering.

```java
// Natural order: Person comparable by ID (the primary order)
class Person implements Comparable<Person> {
    final int id;
    @Override public int compareTo(Person other) {
        return Integer.compare(this.id, other.id);
    }
}

// External orderings: sort by different fields
Comparator<Person> byName = Comparator.comparing(p -> p.name);
Comparator<Person> byAge = Comparator.comparingInt(p -> p.age);
Comparator<Person> byIdDesc = Comparator.comparingInt(
    (Person p) -> p.id).reversed();

// Using in sort:
people.sort(byName);              // sort by name
people.sort(byAge.thenComparing(byName)); // by age, then name
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The Comparable/Comparator split is
a design choice about where ordering logic belongs. If ordering is
inherent to the domain concept (higher ID = newer, lower price = better
deal), `Comparable` makes sense. If ordering is context-dependent
(sort users by name for the UI, by signup date for analytics, by
spending for loyalty), `Comparator` is the right choice. Records
do NOT auto-generate `Comparable` - you must implement it explicitly
if needed. This is intentional: what's the "natural" order of a record
with multiple fields is a domain decision.

---

**Q2 (compareTo contract): What are the requirements of compareTo()?**

A: `compareTo()` must satisfy four properties:

1. **Anti-symmetry:** `sgn(a.compareTo(b)) == -sgn(b.compareTo(a))`
   If a < b, then b > a.

2. **Transitivity:** if `a.compareTo(b) > 0` and `b.compareTo(c) > 0`,
   then `a.compareTo(c) > 0`.

3. **Consistency:** if `a.compareTo(b) == 0`, then `sgn(a.compareTo(c))
   == sgn(b.compareTo(c))` for all c.

4. **Consistency with equals (strongly recommended):**
   `(a.compareTo(b) == 0) == a.equals(b)`. Required for `TreeMap`/`TreeSet`
   to work correctly.

```java
// Transitivity violation example:
// a=1.0, b=NaN, c=2.0
// 1.0.compareTo(NaN) -> -1  (1.0 < NaN)
// NaN.compareTo(2.0) -> -1  (NaN < 2.0)
// 1.0.compareTo(2.0) -> -1  (1.0 < 2.0)
// This is consistent! But:
// NaN.compareTo(NaN) -> 0 (NaN == NaN for ordering)
// Double.compareTo handles NaN correctly; < operator does not
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Violating transitivity causes
undefined behavior in sorting algorithms. Java's `TimSort` may
throw `IllegalArgumentException` if it detects a violated comparator
contract (Java 7+: "Comparison method violates its general contract!").
This is a correctness check, not a performance optimization. If you
see this exception in production: find the inconsistent comparator
in your sorted collection or stream sort.

---

**Q3 (Subtraction comparator bug): Why is `(a, b) -> a - b` wrong as
a comparator?**

A: Integer overflow makes the subtraction comparator incorrect for
extreme values:

```java
// The bug:
Comparator<Integer> subtraction = (a, b) -> a - b;

int a = Integer.MIN_VALUE;  // -2147483648
int b = 1;
a - b;  // = -2147483648 - 1
        // = 2147483647 (integer overflow)
        // Comparator says a > b (positive result)
        // But a is actually LESS than b!

// Concrete sorting failure:
List<Integer> list = Arrays.asList(Integer.MIN_VALUE, 0, 1, 2);
list.sort((a, b) -> a - b);
// Expected: [MIN_VALUE, 0, 1, 2]
// Actual: unpredictable due to overflow!

// FIXES:
Comparator<Integer> correct1 = Integer::compare;
Comparator<Integer> correct2 = Comparator.comparingInt(i -> i);
Comparator<Integer> correct3 = (a, b) -> Integer.compare(a, b);
// Integer.compare uses subtraction internally with long cast:
// (x < y) ? -1 : ((x == y) ? 0 : 1) - no overflow
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The subtraction comparator works
for small bounded values (ages 0-150, scores 0-100) because overflow
never occurs in practice. This makes it a silent bug - tests pass,
production works for years, then an edge case hits. Code review flag:
any comparator implemented as `(a, b) -> a - b` or `(a, b) -> b - a`
should be rewritten. The correct forms are all one-liners:
`Comparator.comparingInt()`, `Integer.compare()`, `Comparator.naturalOrder()`.

---

**Q4 (Comparator.comparing() chains): Build a comparator that sorts
employees by department ascending, then by salary descending, with
null departments last.**

A:
```java
record Employee(String dept, String name, int salary) {}

Comparator<Employee> comp =
    Comparator.comparing(
        Employee::dept,
        Comparator.nullsLast(Comparator.naturalOrder()) // nulls last
    )
    .thenComparingInt(e -> -e.salary()); // negate = descending

// Test:
List<Employee> emps = List.of(
    new Employee("Eng", "Alice", 90000),
    new Employee("Eng", "Bob", 80000),
    new Employee(null, "Carol", 70000),
    new Employee("HR", "Dave", 60000)
);
emps.stream()
    .sorted(comp)
    .forEach(System.out::println);
// Eng Alice 90000
// Eng Bob 80000
// HR Dave 60000
// null Carol 70000
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

Alternatively using explicit reversed:
```java
Comparator<Employee> comp2 =
    Comparator.comparing(
        Employee::dept,
        Comparator.nullsLast(Comparator.naturalOrder()))
    .thenComparing(
        Comparator.comparingInt(Employee::salary).reversed());
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The `.reversed()` method reverses
the ENTIRE comparator up to that point. If you chain `.comparing().thenComparing().reversed()`, all criteria are reversed. To reverse
only the second criterion: use a new `Comparator.comparingX(...).reversed()`
as the argument to `thenComparing()`. Understanding this scoping rule
is essential for complex multi-field sorts. In production code: extract
named comparators for readability when the chain gets complex.

---

**Q5 (Consistent with equals): What happens when compareTo is not
consistent with equals in a TreeSet?**

A:
```java
// Case-insensitive comparator, case-sensitive equals:
TreeSet<String> set = new TreeSet<>(String.CASE_INSENSITIVE_ORDER);
set.add("hello");
set.add("Hello");  // compareTo returns 0 (case-insensitive match)
                   // TreeSet says "already in set"!
set.size();        // 1, not 2!
set.contains("HELLO"); // true
set.contains("hello"); // true
// But set only has one element!

// TreeSet uses compareTo for all operations (not equals)
// When compareTo returns 0, it's treated as "same element"
// regardless of what equals() says
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

The Javadoc for `Comparable` says: "It is strongly recommended, but
not strictly required, that `(x.compareTo(y) == 0) == x.equals(y)`."
The "strictly required" part is what causes the TreeSet behavior.

*What separates good from great:* This inconsistency is intentional
for `String.CASE_INSENSITIVE_ORDER` - you want a case-insensitive set
where "hello" and "Hello" are the same element. When the inconsistency
is unintentional, it's a subtle bug. `TreeMap` has the same issue:
inserting `"hello"` and `"Hello"` with a case-insensitive map only
stores one entry. To diagnose: check `set.size()` vs expected count
after population; if too small, check compareTo/equals consistency.

---

**Q6 (Natural order primitives): What is the natural order of Java's
primitive wrapper types?**

A: All primitive wrapper types implement `Comparable<T>` with numeric ordering:
- `Byte`, `Short`, `Integer`, `Long`, `Float`, `Double`: numeric ascending
- `Character`: Unicode code point order (same as `char` cast to `int`)
- `Boolean`: `false` < `true` (false=0, true=1)

```java
// Natural order examples:
List<Integer> nums = Arrays.asList(3, 1, 4, 1, 5);
Collections.sort(nums); // [1, 1, 3, 4, 5]

List<Character> chars = Arrays.asList('z', 'A', 'a', 'Z');
Collections.sort(chars); // [A, Z, a, z] (uppercase before lowercase in ASCII)

List<String> strs = Arrays.asList("banana", "Apple", "cherry");
Collections.sort(strs); // [Apple, banana, cherry] (uppercase before lowercase)
// For case-insensitive sort:
strs.sort(String.CASE_INSENSITIVE_ORDER); // [Apple, banana, cherry]
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* String's natural order is lexicographic
based on Unicode code points. This means uppercase letters come before
lowercase in ASCII (A=65, a=97). For locale-aware text sorting:
`Collator.getInstance(Locale)` provides locale-appropriate ordering
(e.g., Swedish has 'å' after 'z', not after 'a'). For user-facing
sort: always use `Collator` unless the data is purely technical
(IDs, codes) where lexicographic order is sufficient.

---

**Q7 (Reverse ordering): How do you reverse a natural or custom order?**

A:
```java
// Reverse natural order:
List<Integer> nums = new ArrayList<>(List.of(3,1,4,1,5));

// Option 1: Comparator.reverseOrder()
nums.sort(Comparator.reverseOrder()); // [5,4,3,1,1]

// Option 2: Collections.sort + Collections.reverse
Collections.sort(nums);
Collections.reverse(nums);

// Option 3: .reversed() on an existing comparator
Comparator<Integer> asc = Comparator.naturalOrder();
Comparator<Integer> desc = asc.reversed();
nums.sort(desc);

// TreeSet with reverse natural order:
TreeSet<Integer> descSet = new TreeSet<>(Comparator.reverseOrder());
descSet.addAll(List.of(3,1,4,1,5));
// Iterates: 5, 4, 3, 1 (descending, no duplicate 1)

// PriorityQueue max-heap (reverse natural order):
PriorityQueue<Integer> maxHeap =
    new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(3); maxHeap.offer(1); maxHeap.offer(5);
maxHeap.poll(); // 5 (max element)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `PriorityQueue` is min-heap by default
(natural order). For max-heap: `new PriorityQueue<>(Comparator.reverseOrder())`.
This is a common interview question: "implement a k-largest elements
algorithm". Solution: min-heap of size k; if a new element is larger
than the heap's min, replace the min with the new element. Result:
heap contains the k largest elements. `PriorityQueue` is O(log k)
per insert with the bounded heap trick.

---

**Q8 (Null-safe comparison): How do you sort a collection that contains
null elements?**

A:
```java
// Collections.sort(list) throws NPE if list contains null
List<String> list = Arrays.asList("c", null, "a", null, "b");
// Collections.sort(list); // NullPointerException!

// Fix: Comparator.nullsFirst or nullsLast
list.sort(Comparator.nullsFirst(Comparator.naturalOrder()));
// [null, null, a, b, c]

list.sort(Comparator.nullsLast(Comparator.naturalOrder()));
// [a, b, c, null, null]

// Null-safe custom comparator:
Comparator<Person> byName = Comparator.nullsLast(
    Comparator.comparing(Person::getName,
        Comparator.nullsLast(Comparator.naturalOrder()))
);
// Handles null Person AND null Person.getName()!

// For null-safe sorting of primitives from nullable sources:
people.stream()
    .sorted(Comparator.comparingInt(p ->
        p.getAge() != null ? p.getAge() : Integer.MAX_VALUE))
    .collect(Collectors.toList());
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* Database query results frequently
contain null values for optional fields. Any stream sort on such data
without null handling will NPE in production when a null row appears.
Standard defensive practice: always use `nullsFirst()/nullsLast()` when
sorting data from external sources. Define the null policy explicitly
in the comparator (not silently letting NPE propagate). For domain
objects: prefer `Optional<T>` return types from accessors to make null
handling explicit at the type level.

---

**Q9 (Custom multi-field sort): Implement a comparator to sort
employees by: manager first, then by years of service descending,
then by name ascending.**

A:
```java
record Employee(String name, boolean isManager,
                int yearsOfService) {}

Comparator<Employee> comp =
    // Managers first: false < true, so reversed (true first):
    Comparator.comparing(Employee::isManager).reversed()
    // Then years of service descending:
    .thenComparingInt(e -> -e.yearsOfService())
    // Then name ascending (natural order):
    .thenComparing(Employee::name);

// Equivalent, more explicit:
Comparator<Employee> comp2 =
    Comparator.<Employee, Boolean>comparing(Employee::isManager,
        Comparator.reverseOrder())           // true > false
    .thenComparing(
        Comparator.comparingInt(Employee::yearsOfService).reversed())
    .thenComparing(Employee::name);

// Test:
List<Employee> staff = List.of(
    new Employee("Alice", false, 3),
    new Employee("Bob",   true,  5),
    new Employee("Carol", true,  8),
    new Employee("Dave",  false, 7)
);
staff.stream().sorted(comp2).forEach(System.out::println);
// Carol (manager, 8 years)
// Bob   (manager, 5 years)
// Dave  (non-manager, 7 years)
// Alice (non-manager, 3 years)
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* When negating for descending order
(`e -> -e.yearsOfService()`), be aware of integer overflow for
`Integer.MIN_VALUE` - the same subtraction bug in comparator form.
For safety: use `Comparator.comparingInt(...).reversed()` instead of
negation. Also: the `comparing(Employee::isManager, Comparator.reverseOrder())`
pattern is clearer than negating or chaining `.reversed()` after
because it explicitly documents "isManager, sorted descending".

---

### ⚖️ Comparison Table

| Aspect | Comparable | Comparator |
|---|---|---|
| Location | Inside the class | External class or lambda |
| Number of orderings | One per class | Unlimited |
| Modifying class needed | Yes | No |
| Java 8 factory methods | No | Comparator.comparing(), etc. |
| Used by | Collections.sort, TreeSet/Map | Sort overloads, sorted() |
| Null handling | Not supported | nullsFirst()/nullsLast() |
| Thread safe | N/A (stateless) | N/A if stateless |

---

---

# Java Core - L2 Lambdas and Streams

## Lambda Expressions and Functional Interfaces

---

### 🎯 Model Answer

**30 seconds:**
> A lambda expression is an anonymous function: `(params) -> body`.
> It can be used anywhere a FUNCTIONAL INTERFACE (an interface with
> exactly one abstract method) is expected. The compiler infers the
> target type from context. Java provides built-in functional interfaces
> in `java.util.function`: `Function<T,R>`, `Predicate<T>`,
> `Consumer<T>`, `Supplier<T>`, `BiFunction<T,U,R>`, and primitive
> variants. Lambdas capture effectively-final local variables from
> enclosing scope. Method references (`Class::method`) are shorthand
> for single-method lambdas.

**3 minutes (Senior):**
> Lambdas desugar to static synthetic methods at compile time (Java 8+
> via `invokedynamic`). Each lambda invocation does NOT create a new
> class per call - the JVM generates a class at runtime on first use
> (invokeDynamic bootstraps this). The key implication: lambdas do not
> have their own `this` (they capture the enclosing `this`). Inner
> anonymous classes have their own `this`.
>
> Effective finality: variables captured by lambdas must not be
> reassigned after capture. `int count = 0; Runnable r = () -> count++;`
> is a compile error. Workaround: `AtomicInteger count = new AtomicInteger();`
> The REFERENCE to count is effectively final; the integer value changes.
>
> Functional interfaces designed for different uses: `Function` (in -> out),
> `Predicate` (in -> boolean), `Consumer` (in -> void), `Supplier` (() -> out),
> `UnaryOperator<T>` (T -> T), `BinaryOperator<T>` (T,T -> T).

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Lambda expressions - let me cover syntax, functional
interfaces, method references, variable capture rules, and the common
functional interface types."

**(2) First principles:** "From first principles: passing behavior as
data requires either subclassing (verbose), anonymous classes (still
verbose), or first-class functions (lambdas). Java 8 added lambdas to
enable functional programming patterns without the ceremony."

**(3) Bridge:** "A lambda is like a recipe card you hand to a chef
(a method that accepts a functional interface). Before Java 8, you had
to create a whole cookbook (class) to write one recipe. Now you can
just write the recipe inline."

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Lambda syntax and types | 2 minutes |
| Functional interfaces | 2 minutes |
| Method references | 2 minutes |
| Effective finality | 2 minutes |
| Lambda vs anonymous class | 2 minutes |
| invokedynamic internals | 2-3 minutes |
| Predicate/Function composition | 2 minutes |
| Custom functional interface | 90 seconds |
| Lambda in concurrent context | 2 minutes |

---

**Q1 (Lambda syntax and types): Describe the different lambda syntax forms.**

A:
```java
// Form 1: single param, single expression (no parens, no return)
Predicate<String> p1 = s -> s.isEmpty();

// Form 2: single param, block (needs return if non-void)
Function<String, String> f1 = s -> {
    if (s == null) return "";
    return s.trim();
};

// Form 3: multiple params
BiFunction<String, Integer, String> f2 = (str, n) -> str.repeat(n);

// Form 4: no params
Supplier<String> s1 = () -> "constant";

// Form 5: explicit types (compiler usually infers)
Function<String, Integer> f3 = (String s) -> s.length();

// Form 6: throws checked exception (must wrap or define @FunctionalInterface)
// Supplier<String> s2 = () -> Files.readString(path); // won't compile!
// Either: custom @FunctionalInterface that declares throws IOException,
// or wrap in try-catch:
Supplier<String> s2 = () -> {
    try { return Files.readString(path); }
    catch (IOException e) { throw new UncheckedIOException(e); }
};
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The checked exception limitation is
a real friction point. Java's standard functional interfaces don't declare
checked exceptions. Solutions: (1) wrap in unchecked exception
(`UncheckedIOException`), (2) define custom functional interface with
`throws`, (3) use Vavr's `CheckedFunction` or similar library. In
production: wrapping with `UncheckedIOException` or `RuntimeException`
with a cause is standard. The cause chain remains intact for logging.

---

**Q2 (Functional interfaces): What is a functional interface? List
the key JDK functional interfaces.**

A: A functional interface has exactly ONE abstract method. It may have
default and static methods. Annotated with `@FunctionalInterface` (optional
but recommended - enforces the one-abstract-method constraint).

```java
// JDK functional interfaces:
// java.util.function package

Function<T, R>        // T -> R (transform)
BiFunction<T,U,R>     // (T,U) -> R
UnaryOperator<T>      // T -> T (extends Function<T,T>)
BinaryOperator<T>     // (T,T) -> T

Predicate<T>          // T -> boolean (test)
BiPredicate<T,U>      // (T,U) -> boolean

Consumer<T>           // T -> void (action)
BiConsumer<T,U>       // (T,U) -> void

Supplier<T>           // () -> T (factory)

Runnable              // () -> void (in java.lang)
Callable<V>           // () -> V throws Exception (in java.util.concurrent)

// Primitive specializations (avoid autoboxing):
IntFunction<R>        // int -> R
ToIntFunction<T>      // T -> int
IntUnaryOperator      // int -> int
IntBinaryOperator     // (int, int) -> int
// Also: Long, Double variants
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using Kafka messaging. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Choosing the right functional interface
avoids unnecessary boxing. `mapToInt(Function<T, Integer>)` would box
every `int` result into `Integer`. `mapToInt(ToIntFunction<T>)` avoids
this. When designing APIs that accept callbacks: prefer the specific
primitive variant if the type is known (`IntSupplier` instead of
`Supplier<Integer>`). For custom functional interfaces: define them
only when none of the JDK interfaces fits, and annotate with
`@FunctionalInterface` for documentation and compiler enforcement.

---

**Q3 (Method references): Explain the four kinds of method references.**

A:

**1. Static method reference: `ClassName::staticMethod`**
```java
// s -> Integer.parseInt(s) -> Integer::parseInt
Function<String, Integer> parse = Integer::parseInt;
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**2. Instance method on a specific instance: `instance::instanceMethod`**
```java
String prefix = "Hello";
Predicate<String> starts = prefix::startsWith; // prefix bound at creation
// s -> prefix.startsWith(s)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**3. Instance method on any instance of the class: `ClassName::instanceMethod`**
```java
// The target instance is the first argument:
Function<String, String> upper = String::toUpperCase;
// s -> s.toUpperCase()

Comparator<String> comp = String::compareTo;
// (s1, s2) -> s1.compareTo(s2)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**4. Constructor reference: `ClassName::new`**
```java
Supplier<List<String>> listFactory = ArrayList::new;
// () -> new ArrayList<>()

BiFunction<String, Integer, StringBuilder> sb = StringBuilder::new;
// (str, capacity) -> not directly, but: () -> new StringBuilder()
// The matching depends on the functional interface's parameter count
```

> **Code walkthrough:** This Unknown example demonstrates contract definition. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Type 3 (unbound instance method) is
the most confusing. `String::toUpperCase` has zero explicit parameters
in the method signature, but the lambda form `s -> s.toUpperCase()` has
one parameter. This is because the "receiver" (the object on which the
method is called) becomes the first parameter in the functional interface.
This is why `String::compareTo` works as a `Comparator<String>` -
`Comparator.compare(s1, s2)` maps to `s1.compareTo(s2)`.

---

**Q4 (Effective finality): What is "effectively final" and why does
Java require it for lambda captures?**

A: "Effectively final" means a variable whose value is never changed
after initialization (it could be declared `final` but isn't required to be).

Java 8 relaxed the rule from "must be declared `final`" to "must be
effectively final" - same semantics, less verbosity.

```java
int x = 10;           // effectively final (never reassigned)
int y = 20;
y = 30;               // NOT effectively final (reassigned)

Runnable r = () -> {
    System.out.println(x); // OK: x is effectively final
    System.out.println(y); // COMPILE ERROR: y is not effectively final
};
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Why required?**
Lambda runs in a different context (possibly different thread, certainly
after the local variable goes out of scope). Java could capture a COPY
of the value (like closures in other languages), which it does. But if
the local variable could change, the lambda would see an inconsistent
view: `y = 30; Runnable r = () -> print(y); y = 40;` - which `y` should
the lambda print?

The requirement for effective finality makes the captured value unambiguous:
the lambda gets the value at capture time and it never changes.

*What separates good from great:* The restriction is specific to LOCAL
VARIABLES. Instance fields can be mutated inside lambdas (they're accessed
via `this` reference, which is effectively final). The mutation of instance
fields in lambdas is legal but can cause thread-safety issues in concurrent
streams. `Stream.parallel()` with lambdas that modify shared instance
state without synchronization is a common production bug.

---

**Q5 (Lambda vs anonymous class): What are the differences between
lambdas and anonymous classes?**

A:

| Aspect | Anonymous Class | Lambda |
|---|---|---|
| `this` reference | Refers to the anonymous class | Refers to enclosing class |
| `super` keyword | References anonymous class's super | Not applicable |
| Can have state (fields) | Yes | No (captures only) |
| Can extend a class | Yes | No (functional interface only) |
| Can be serialized | If implements Serializable | Unreliable; avoid |
| Performance | New class generated at compile time | invokedynamic; better at runtime |
| Shadowing outer variables | Yes (same-name field shadows outer) | No (compile error) |
| Single abstract method | No restriction | Required |

```java
// Lambda captures enclosing this:
class Outer {
    String name = "Outer";
    Runnable lambda = () -> System.out.println(this.name); // "Outer"
    Runnable anon = new Runnable() {
        String name = "Anon";
        public void run() {
            System.out.println(this.name);  // "Anon"
            System.out.println(Outer.this.name); // "Outer"
        }
    };
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The `this` difference is the practical
one for real code. In event listener patterns (Swing, Android), an anonymous
class registered as a listener needs to reference itself to unregister:
`button.removeActionListener(this)` - `this` being the anonymous class.
A lambda can't do this - you'd need a variable reference:
`ActionListener listener = e -> handle(e); button.addActionListener(listener);
button.removeActionListener(listener);`

---

**Q6 (invokedynamic internals): How does the JVM implement lambdas
internally?**

A: Java 8 lambdas use the `invokedynamic` bytecode instruction
(originally added in Java 7 for dynamic languages on the JVM).

**The process:**
```
1. Compiler encounters lambda expression
2. Compiler generates:
   - invokedynamic instruction (at the lambda use site)
   - A synthetic static method containing the lambda body
     (e.g., lambda$myMethod$0)
   
3. First execution of invokedynamic:
   - JVM calls the bootstrap method (LambdaMetafactory.metafactory())
   - LambdaMetafactory generates a class implementing the functional interface
     (at RUNTIME, not compile time - this is the "dynamic" part)
   - The generated class's method delegates to the synthetic static method
   
4. Subsequent executions:
   - Non-capturing lambdas: same instance reused (constant)
   - Capturing lambdas: new instance per call (stores captured values)
```

> **Code walkthrough:** This Unknown example demonstrates a key concept in practice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

**Why invokedynamic instead of anonymous class at compile time?**
- Forward compatibility: JVM is free to optimize lambda implementation
  in future versions without recompiling code
- Lazy class generation: classes only created when first used
- No proliferation of .class files: lambdas don't generate separate
  .class files in the JAR (anonymous classes do: `Outer$1.class`)

*What separates good from great:* The invokedynamic approach is one
reason Java 8 lambdas outperform equivalent anonymous classes in
benchmarks. Anonymous classes create a new .class file per use site,
loaded at startup. Lambdas generate one class per lambda expression
type (not per call), loaded lazily on first use. JVM profiling tools
(async-profiler, JFR) show lambda execution under the synthetic method
name (`lambda$method$0`). Knowing this helps interpret profiles.

---

**Q7 (Predicate/Function composition): How do you compose lambdas?**

A:
```java
// Function composition (andThen = left-to-right, compose = right-to-left):
Function<String, String> trim = String::trim;
Function<String, Integer> length = String::length;
Function<String, Integer> trimThenLength = trim.andThen(length);
// trimThenLength.apply("  hello  ") = 5

// Predicate combination:
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositiveEven = isPositive.and(isEven);
Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);
Predicate<Integer> isNotPositive = isPositive.negate();
Predicate<Object> isNull = Predicate.not(Objects::nonNull); // Java 11

// Consumer chaining (andThen):
Consumer<String> print = System.out::println;
Consumer<String> log = s -> logger.info("Processing: {}", s);
Consumer<String> printAndLog = print.andThen(log);
// Executes print first, then log

// Predicate.not() (Java 11): negate a method reference
List<String> strings = List.of("hello", "", "world", "");
strings.stream()
    .filter(Predicate.not(String::isEmpty)) // != isEmpty()
    .collect(Collectors.toList()); // ["hello", "world"]
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* Function composition is mathematical
function composition: `f.andThen(g)` means `g(f(x))`. Use composition
to build processing pipelines from small reusable pieces. Real-world:
a validation pipeline for incoming data:
`Validator.isNotNull().and(Validator.hasMinLength(3)).and(Validator.matchesPattern("[A-Z]+"))`.
Each validator is a `Predicate<String>` - reusable across different
fields, composable for different field validation rules.

---

**Q8 (Custom functional interface): When should you define your own
functional interface?**

A: Define a custom functional interface when:

1. **Semantic clarity**: `BiFunction<String, String, String>` is opaque;
   `StringMerger` communicates domain meaning.

2. **Checked exceptions**: need to declare `throws IOException`.

3. **Primitive performance**: specialized to avoid boxing (though prefer
   JDK primitives like `IntUnaryOperator`).

4. **Documentation**: custom name in IDE, javadocs, error messages.

```java
// Custom functional interface with checked exception:
@FunctionalInterface
interface FileTransformer {
    String transform(String content, Path source)
        throws IOException;
    // Can have default methods:
    default FileTransformer andThen(FileTransformer next) {
        return (content, path) ->
            next.transform(this.transform(content, path), path);
    }
}

// Usage: cleaner API with domain-specific name
void processFiles(List<Path> files, FileTransformer transformer)
        throws IOException {
    for (Path file : files) {
        String content = Files.readString(file);
        String result = transformer.transform(content, file);
        Files.writeString(file, result);
    }
}
// Caller:
processFiles(paths, (content, path) ->
    content.replace("deprecated_api", "new_api"));
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* The `@FunctionalInterface` annotation
is documentation and a compile-time guard. Without it: if you accidentally
add a second abstract method, the interface still compiles but lambdas
can no longer be used with it. With the annotation: the compiler errors
immediately. As a library author: always annotate intended functional
interfaces. For the `throws` use case: consider whether the checked
exception is truly appropriate or whether wrapping in a RuntimeException
(with the cause) is cleaner for callers.

---

**Q9 (Lambda in concurrent context): What are the thread-safety
considerations for lambdas?**

A: Lambdas themselves are not inherently thread-safe or unsafe - it
depends on what state they access.

```java
// SAFE: pure function (no shared mutable state):
Function<Integer, Integer> square = n -> n * n;
// Parallel streams with pure lambdas are safe:
List<Integer> squares = nums.parallelStream()
    .map(n -> n * n)
    .collect(Collectors.toList());

// UNSAFE: shared mutable state in parallel stream:
List<Integer> results = new ArrayList<>(); // NOT thread-safe!
nums.parallelStream()
    .map(n -> n * n)
    .forEach(results::add); // DATA RACE: multiple threads add simultaneously
// results may be incomplete, have nulls, or throw ConcurrentModificationException

// SAFE: use thread-safe collectors:
List<Integer> results = nums.parallelStream()
    .map(n -> n * n)
    .collect(Collectors.toList()); // Collectors.toList() is thread-safe

// SAFE with explicit synchronization:
List<Integer> syncResults = Collections.synchronizedList(new ArrayList<>());
nums.parallelStream().map(n -> n * n).forEach(syncResults::add); // OK

// PREFERRED: accumulate then return with collectors:
Map<String, Long> count = words.parallelStream()
    .collect(Collectors.groupingByConcurrent(  // parallel-safe
        Function.identity(),
        Collectors.counting()));
```

> **Code walkthrough:** This Unknown example demonstrates mutex locking using concurrency primitive. **KEY MECHANISM:** the JVM acquires the intrinsic lock on the object monitor before entering the block. **WHY IT MATTERS:** a thread holding the lock blocks all other threads - a bottleneck at scale. **TAKEAWAY: prefer ReentrantLock or ConcurrentHashMap over synchronized for hot paths.**

*What separates good from great:* The `parallel()` + mutable collection
pattern is one of the most common parallel stream bugs. `ArrayList`
is not thread-safe; concurrent adds can corrupt the backing array.
The correct mental model: treat parallel stream lambdas as pure functions
that take input and return output. Use collectors to aggregate - they're
designed for parallel execution. If you need side effects: make them
thread-safe (ConcurrentHashMap, AtomicInteger, synchronized) or use
`forEachOrdered()` to force sequential execution in order.

---

### ⚖️ Comparison Table

| Aspect | Lambda | Anonymous Class | Method Reference |
|---|---|---|---|
| Syntax verbosity | Low | High | Lowest |
| `this` binding | Enclosing class | Own class | Enclosing class |
| State | Captured only | Fields allowed | Captured only |
| Functional interface | Required | Optional | Required |
| Checked exceptions | Wrapping needed | Declare in interface | Same as target |
| invokedynamic | Yes | No (compile-time class) | Yes |

---


---

## Streams API Pipelines

---

### 🎯 Model Answer

**30 seconds:**
> A Stream is a sequence of elements supporting sequential and parallel
> aggregate operations. A pipeline has three parts: (1) SOURCE (collection,
> array, IO), (2) INTERMEDIATE OPERATIONS (lazy: filter, map, sorted,
> distinct, flatMap, limit, skip, peek) - each returns a new Stream,
> (3) TERMINAL OPERATION (eager: collect, forEach, reduce, count, min,
> max, findFirst, anyMatch, allMatch) - triggers execution. Streams
> are not collections: they do not store data, process lazily, and can
> only be consumed ONCE.

**3 minutes (Senior):**
> Lazy evaluation: intermediate operations do nothing until a terminal
> is called. With `filter().map().findFirst()`: the stream does not
> filter all elements first, then map all. Instead: it tries the first
> element through filter and map; if findFirst is satisfied, stops.
> Short-circuit operations (`findFirst`, `limit`, `anyMatch`) can process
> far fewer elements than the full source.
>
> Parallel streams: `stream.parallel()` splits the source and processes
> chunks in the `ForkJoinPool.commonPool()`. Works well for CPU-bound,
> stateless operations on large datasets. Works poorly for I/O-bound,
> stateful (sorted), or small datasets (overhead exceeds benefit).
>
> Collectors: `toList()`, `groupingBy()`, `joining()`, `partitioningBy()`,
> `toMap()`, `counting()`, `summarizingInt()`. `Collectors.groupingBy()
> ` is O(n) - one pass - not O(n log n) like sorting.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Streams API - let me cover the source-intermediate-
terminal pipeline structure, lazy evaluation, common operations, collectors,
and parallel streams."

**(2) First principles:** "From first principles: processing a collection
requires iteration. Before streams, you'd write a for loop with accumulation
variables. Streams abstract the iteration, support lazy evaluation (only
process what's needed), and enable parallel processing without explicit
thread management."

**(3) Bridge:** "A stream pipeline is like an assembly line with conveyor
belts. The raw materials come in (source), pass through processing
stations (intermediate operations), and the finished product comes out
(terminal). The conveyor only moves when the final station requests
the next part (lazy)."

---


### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Stream pipeline lazy evaluation | 2 minutes |
| flatMap explained | 2 minutes |
| Collectors groupingBy | 2 minutes |
| Stream vs Collection | 2 minutes |
| Parallel stream trade-offs | 2-3 minutes |
| reduce vs collect | 2 minutes |
| Infinite streams | 2 minutes |
| Stream ordering | 2 minutes |
| Custom Collector | 2-3 minutes |

---

**Q1 (Stream pipeline lazy evaluation): Explain lazy evaluation in streams
with a concrete example.**

A: Intermediate operations return a new Stream without processing elements.
The processing begins only when the terminal operation is invoked.

```java
Stream<Integer> stream = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .stream()
    .filter(n -> {
        System.out.println("filter: " + n);
        return n % 2 == 0;
    })
    .map(n -> {
        System.out.println("map: " + n);
        return n * n;
    });
// At this point: NOTHING has been printed. No elements processed.

// Terminal operation triggers processing:
Optional<Integer> first = stream.findFirst();
// Output:
// filter: 1   (1 fails filter)
// filter: 2   (2 passes filter)
// map: 2      (2 is mapped to 4)
// DONE: findFirst returns Optional[4]
// Elements 3-10 are NEVER processed!
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

The stream processed only 3 elements (1, 2 for filter; 2 for map) out of 10.
Without laziness, all 10 would be filtered, then all matching mapped.

*What separates good from great:* Lazy evaluation enables two critical
optimizations: (1) Short-circuit: `findFirst()`, `limit(n)`, `anyMatch()`
can terminate before processing all elements. An infinite stream:
`Stream.iterate(0, n -> n + 1).filter(n -> n > 100).findFirst()` terminates
at 101. (2) Fusion: the JVM can fuse sequential filter+map+filter into
a single pass, avoiding intermediate collections. Benchmarks show fused
pipelines are often competitive with optimized for-loops.

---

**Q2 (flatMap explained): Explain flatMap with a real-world example.**

A: `map` applies a function and wraps the result in the stream.
`flatMap` applies a function that returns a stream, then flattens
(concatenates) all resulting streams into one.

```java
// Real-world: each order has a list of items - get all items:
record OrderItem(String productId, int quantity) {}
record Order(String orderId, List<OrderItem> items) {}

List<Order> orders = getOrders();

// map gives Stream<List<OrderItem>> - nested, not what we want:
Stream<List<OrderItem>> nested = orders.stream()
    .map(Order::items); // each Order maps to a List<OrderItem>

// flatMap gives Stream<OrderItem> - flat, usable:
List<OrderItem> allItems = orders.stream()
    .flatMap(order -> order.items().stream()) // flatten each list
    .collect(Collectors.toList());

// Distinct product IDs across all orders:
Set<String> productIds = orders.stream()
    .flatMap(order -> order.items().stream())
    .map(OrderItem::productId)
    .collect(Collectors.toSet());

// flatMap for Optional streams (Java 9):
List<Optional<String>> optionals = List.of(
    Optional.of("a"), Optional.empty(), Optional.of("b")
);
List<String> values = optionals.stream()
    .flatMap(Optional::stream) // removes empties
    .collect(Collectors.toList()); // ["a", "b"]
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* `flatMap` is the functional equivalent
of the monad bind operation. Understanding it deeply: `map` is 1-to-1,
`flatMap` is 1-to-many-then-flatten. The flattening is exactly ONE level:
`Stream<Stream<Stream<T>>>` flatMapped once is `Stream<Stream<T>>`.
Real-world patterns requiring flatMap: expanding relationships (orders to
items, sentences to words, files to lines), filtering with Optional in
streams, expanding enum values to their related items.

---

**Q3 (Collectors groupingBy): How does `Collectors.groupingBy()` work
and what are its variants?**

A:
```java
// Basic groupingBy: Map<K, List<V>>
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::dept));
// {"Engineering": [Alice, Bob], "HR": [Carol, Dave]}

// Downstream collector (count instead of List):
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::dept,
        Collectors.counting()));
// {"Engineering": 2, "HR": 2}

// Downstream: map values to names:
Map<String, List<String>> namesByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::dept,
        Collectors.mapping(Employee::name, Collectors.toList())));

// Multi-level grouping:
Map<String, Map<Integer, List<Employee>>> byDeptAndLevel = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::dept,
        Collectors.groupingBy(Employee::level)));

// partitioningBy: splits into two groups (true/false):
Map<Boolean, List<Employee>> partition = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.salary() > 60000));
// {true: [high earners], false: [others]}

// toMap: specify key and value functions
Map<String, Integer> nameToSalary = employees.stream()
    .collect(Collectors.toMap(
        Employee::name,
        Employee::salary,
        (existing, replacement) -> existing)); // merge fn: handle duplicate keys
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* `groupingBy` performs a single O(n)
pass and builds the map incrementally - it does NOT sort first. The
result map's entry order is unspecified (HashMap). For sorted keys: use
`groupingBy(key, TreeMap::new, downstream)`. For stable insertion order:
`groupingBy(key, LinkedHashMap::new, downstream)`. The `merge function`
parameter in `toMap` is required when duplicate keys are possible -
without it, `toMap` throws `IllegalStateException` on duplicate keys
in Java 8. Always provide a merge function for production code.

---

**Q4 (Stream vs Collection): When do you use a Stream vs a Collection?**

A:

| Aspect | Collection | Stream |
|---|---|---|
| Stores data | Yes | No (processes from source) |
| Iterable | Yes (multiple times) | Once (terminal exhausts it) |
| Modifiable | Yes (add/remove) | No |
| Lazy | No (all in memory) | Yes |
| Parallel | Requires external sync | `parallel()` built-in |
| Infinite | No | Yes (generate/iterate) |
| Intermediate results | Yes | No |
| Random access | Some (List) | No |

**Use Collection when:**
- Need to store results for later use
- Need random access or multiple iterations
- Passing data between components

**Use Stream when:**
- One-pass processing (filter, map, aggregate)
- Lazy evaluation needed (short-circuit, large data)
- Composing transformations
- Parallel processing

```java
// One-pass aggregation: Stream
double avgSalary = employees.stream()
    .mapToInt(Employee::salary)
    .average().orElse(0);

// Store results + later access: collect to List
List<String> names = employees.stream()
    .map(Employee::name).collect(Collectors.toList());
names.get(0); // random access - needs Collection
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* Streams cannot be used as method
arguments that are iterated multiple times. A common mistake: returning
a Stream from a method and calling two terminal operations on it (first
call consumes it, second throws IllegalStateException). Stream as a
return type is appropriate only for terminal-once use; return a Collection
or Supplier<Stream> for reusable data. Spring Data's `Stream<T> findAll()`
requires the transaction to remain open for the full stream consumption -
a common pitfall in non-transactional service code.

---

**Q5 (Parallel stream trade-offs): When should you use parallel streams?**

A: Parallel streams split the source, process chunks on ForkJoinPool threads,
and merge results. Beneficial when:

**Good fit for parallel:**
- Large data (> 10,000 elements typically)
- CPU-intensive, stateless operations
- Data structures that split well: ArrayList, arrays (random access)
- No ordering constraint (unordered operations)

**Poor fit for parallel:**
- Small data (overhead exceeds benefit)
- I/O-bound operations (blocks ForkJoinPool threads)
- Stateful operations requiring ordering (sorted())
- Poorly-splittable sources: LinkedList, IO streams
- Operations with side effects on shared state


```java
// BAD: blocking the calling thread defeats async purpose
CompletableFuture<String> future = fetchDataAsync();
String result = future.get(); // blocks caller thread
process(result); // sequential, not async
```

```java
// BAD: parallel for small list (overhead dominates):
List<Integer> small = List.of(1, 2, 3, 4, 5);
small.parallelStream().map(n -> n * 2).collect(Collectors.toList());
// Sequential is faster for 5 elements

// GOOD: parallel for large CPU-bound work:
List<Image> images = loadImages(); // 10,000 images
List<Image> processed = images.parallelStream()
    .map(this::applyFilters) // CPU-bound: works well in parallel
    .collect(Collectors.toList());

// DANGEROUS: I/O in parallel on common pool:
// Blocks ForkJoinPool.commonPool() threads on network calls
urls.parallelStream()
    .map(url -> httpClient.get(url)) // BLOCKS COMMON POOL THREAD
    .collect(Collectors.toList()); // other parallel ops in JVM starved
// Use: CompletableFuture or virtual threads (Java 21) instead
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates async pipeline construction using CompletableFuture. **KEY MECHANISM:** the JVM schedules continuations via ForkJoinPool when each stage completes. **WHY IT MATTERS:** callback chains execute on wrong threads causing ClassCastException in Spring context. **WHAT BREAKS: always specify executor on thenApplyAsync to control thread context.**

*What separates good from great:* The ForkJoinPool.commonPool() is
shared across all parallel streams in the JVM, `CompletableFuture`
executions, and user code that submits to it. Blocking it with I/O
starves other parallel work. The solution for I/O-bound parallel work:
use `CompletableFuture.supplyAsync()` with a dedicated executor, or
Java 21 virtual threads (`Thread.ofVirtual()`). Always measure parallel
vs sequential performance with realistic data sizes before making the
switch. JMH (Java Microbenchmark Harness) is the standard tool.

---

**Q6 (reduce vs collect): When do you use reduce vs collect?**

A:
```java
// reduce: fold elements into a single value (immutable accumulation)
int sum = IntStream.rangeClosed(1, 100).reduce(0, Integer::sum);
// identity=0, accumulator=(acc, n) -> acc + n
// Result: 5050

// collect: mutable reduction into a container
List<String> names = employees.stream()
    .map(Employee::name)
    .collect(Collectors.toList()); // mutable container built up

// Parallel reduce: identity must be an identity for the accumulator:
// reduce(0, Integer::sum) is correct: 0 is the identity for sum
// reduce("", String::concat) is DANGEROUS in parallel:
//   partial results can be concatenated in unpredictable orders
//   AND is O(n^2) due to string creation
// Better: collect(Collectors.joining()) for String concatenation

// Three-arg reduce for parallel (identity, accumulator, combiner):
int sumParallel = employees.parallelStream()
    .reduce(
        0,                           // identity
        (acc, e) -> acc + e.salary(), // accumulator
        Integer::sum                 // combiner (merges partial results)
    );
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

**Rule:** use `reduce` for immutable accumulation into a single value
(sum, product, max). Use `collect` for mutable container building
(List, Map, StringBuilder). For parallel: `reduce` is safe if identity
is correct and accumulator is associative and stateless.

*What separates good from great:* The two-arg `reduce(identity, accumulator)`
works in parallel by splitting the stream, reducing each part (using
identity as the starting value), then combining. The identity MUST truly
be an identity: `reduce(0, ...)` for sum, `reduce(1, ...)` for product,
`reduce("", ...)` for concatenation. Providing a non-identity as the
"identity" causes wrong results in parallel (partial results are combined
with the identity again). This is the most common `reduce` bug.

---

**Q7 (Infinite streams): How do you create and use infinite streams?**

A:
```java
// Stream.iterate: first element = seed, rest = f(previous)
Stream<Integer> naturals = Stream.iterate(0, n -> n + 1);
// 0, 1, 2, 3, 4, ...

// Java 9: Stream.iterate with predicate (like a for loop)
Stream<Integer> under100 = Stream.iterate(0, n -> n < 100, n -> n + 1);
// like: for (int n = 0; n < 100; n++)

// Stream.generate: each element from a Supplier
Stream<Double> randoms = Stream.generate(Math::random);
Stream<String> uuids = Stream.generate(() -> UUID.randomUUID().toString());

// Use with short-circuit or limit:
List<Integer> first10Evens = Stream.iterate(0, n -> n + 2)
    .limit(10)
    .collect(Collectors.toList()); // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

// Fibonacci:
Stream.iterate(new int[]{0, 1}, f -> new int[]{f[1], f[0] + f[1]})
    .limit(10)
    .mapToInt(f -> f[0])
    .forEach(System.out::println); // 0 1 1 2 3 5 8 13 21 34

// Real-world: paginated API calls:
Stream.iterate(1, page -> page + 1)
    .map(page -> api.getUsers(page, 100))
    .takeWhile(users -> !users.isEmpty())  // Java 9: stop when empty page
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The paginated API use case is a real
production pattern for consuming paginated REST APIs. `takeWhile` (Java 9)
is the key: it stops the infinite iteration when the predicate becomes false.
Before Java 9, you'd use `generate` with a `AtomicBoolean` flag or implement
a custom `Spliterator`. The `takeWhile`/`dropWhile` (Java 9) additions
fill the gap between `limit(n)` (count-based) and `filter` (which never
short-circuits for non-matching tails).

---

**Q8 (Stream ordering): What is encounter order and how does it affect
parallel streams?**

A: Encounter order = the order in which elements appear in the source.
Some sources have encounter order (List, arrays - ordered). Some don't
(HashSet, HashMap - unordered).

```java
// Ordered source: order is preserved
List<Integer> nums = List.of(1, 2, 3, 4, 5);
nums.stream().map(n -> n * 2).collect(Collectors.toList());
// [2, 4, 6, 8, 10] - order preserved

// Unordered source: no order guarantee
Set<Integer> numSet = new HashSet<>(nums);
numSet.stream().map(n -> n * 2).collect(Collectors.toList());
// any order: [8, 2, 10, 4, 6] or similar

// Parallel with ordered source: order maintained, but may be slower:
nums.parallelStream().collect(Collectors.toList()); // order preserved but slower
// Worker threads must synchronize to maintain order

// Unordered hint: improve parallel performance when order doesn't matter:
nums.parallelStream()
    .unordered()                // hint: don't need encounter order
    .filter(n -> n % 2 == 0)   // may process in any order
    .collect(Collectors.toList()); // [4, 2] or [2, 4] - any order, faster

// findFirst vs findAny:
Optional<Integer> first = nums.parallelStream().findFirst(); // forces order
Optional<Integer> any = nums.parallelStream().findAny(); // no order - faster
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* Maintaining encounter order in parallel
streams requires coordination between threads (they must produce output
in order). `unordered()` tells the stream it can abandon order, potentially
improving parallel performance. For pipelines where order doesn't matter
(collecting to a Set, computing aggregates), always add `unordered()` in
parallel mode. For pipelines where order matters (collecting to a List
that's displayed to users): don't add `unordered()`, accept the ordering
cost.

---

**Q9 (Custom Collector): How do you implement a custom Collector?**

A:
```java
// Collector<T, A, R>: T=input, A=accumulator, R=result
// Implement 5 methods: supplier, accumulator, combiner, finisher, characteristics

// Example: collect to an immutable list (Guava-style):
Collector<String, List<String>, List<String>> toImmutableList =
    Collector.of(
        ArrayList::new,                      // supplier: creates mutable container
        List::add,                           // accumulator: adds element
        (left, right) -> {                   // combiner: merge two containers (parallel)
            left.addAll(right);
            return left;
        },
        Collections::unmodifiableList,       // finisher: convert to final result
        Collector.Characteristics.UNORDERED  // optional characteristics
    );

// Characteristics:
// CONCURRENT: combiner not used; accumulator handles concurrent access
// UNORDERED: order of accumulation doesn't matter
// IDENTITY_FINISH: finisher is identity (no-op); result = accumulator type

// Real-world: collect statistics:
Collector<Integer, int[], IntSummaryStatistics> stats =
    Collector.of(
        () -> new int[]{0, 0, Integer.MAX_VALUE, Integer.MIN_VALUE}, // [count, sum, min, max]
        (acc, n) -> {
            acc[0]++; acc[1] += n;
            acc[2] = Math.min(acc[2], n);
            acc[3] = Math.max(acc[3], n);
        },
        (left, right) -> new int[]{
            left[0] + right[0],
            left[1] + right[1],
            Math.min(left[2], right[2]),
            Math.max(left[3], right[3])
        },
        acc -> new IntSummaryStatistics(acc[0], acc[2], acc[3], acc[1])
    );
```

> **Code walkthrough:** This Unknown example demonstrates null-safe value wrapping using container. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

*What separates good from great:* Custom collectors are rare but
powerful for building domain-specific aggregations. The combiner is
ONLY called for parallel streams - it merges two partial results.
If your collector is not parallel-safe: don't add CONCURRENT.
The finisher allows transformation of the internal mutable accumulator
into the final immutable result type. For the IDENTITY_FINISH optimization:
the JVM skips the finisher call entirely, saving one function invocation
per pipeline execution.

---

### ⚖️ Comparison Table

| Approach | for-loop | Stream | Parallel Stream |
|---|---|---|---|
| Readability | Low-medium | High | High |
| Laziness | No | Yes | Yes |
| Short-circuit | Manual (break) | Built-in | Built-in |
| Thread safety | Manual | N/A (sequential) | Requires care |
| Performance (small data) | Fastest | Comparable | Slower (overhead) |
| Performance (large data) | Good | Good | Faster (if CPU-bound) |
| Infinite data | Manual | Yes (generate/iterate) | Yes |
| Composability | Low | High | High |

---
