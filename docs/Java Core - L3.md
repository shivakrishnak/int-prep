---
layout: default
title: "Java Core - L3"
nav_order: 3
---

## Table of contents
{: .no_toc }

1. TOC
{:toc}

---

# Java Core - L3 Advanced Generics

## Generic Wildcards and PECS

---

### 🎯 Model Answer

**30 seconds:**
> PECS = "Producer Extends, Consumer Super" - the rule for generic
> wildcards. Use `? extends T` when reading FROM a collection (the
> collection PRODUCES T values for you). Use `? super T` when writing
> TO a collection (the collection CONSUMES T values from you).
> `List<? extends Number>` is read-only from Number perspective.
> `List<? super Integer>` accepts Integer and supertypes.
> Wildcards solve the covariance problem: `List<String>` is NOT a
> subtype of `List<Object>`, but `List<String>` IS a subtype of
> `List<? extends Object>`.

**3 minutes (Senior):**
> The invariance problem: why does Java make generics invariant?
> If `List<String>` were a subtype of `List<Object>`, you could add
> an Integer to a List<String> (via the List<Object> reference).
> Java arrays are covariant (`String[]` IS `Object[]`) - this is why
> `String[] arr = new String[1]; Object[] objs = arr; objs[0] = 42`
> compiles but throws `ArrayStoreException` at runtime.
>
> Bounded wildcards enable safe covariance for reading (`? extends`)
> and safe contravariance for writing (`? super`). The `Collections.copy()`
> signature: `copy(List<? super T> dest, List<? extends T> src)` -
> PECS in action. `src` produces T values (extends), `dest` consumes
> T values (super).
>
> Unbounded wildcard `<?>`: a list of unknown type. Can only read as
> Object, can only write null. Used when you don't care about the
> element type: `printList(List<?> list)`.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "PECS and wildcards - let me cover the invariance problem,
upper-bounded (extends, for reading), lower-bounded (super, for writing),
unbounded wildcard, and the Collections.copy example."

**(2) First principles:** "From first principles: covariance (read-only)
is safe - if List<Dog> is a List<Animal>, we can read Animals from it
safely. Contravariance (write-only) is also safe - if List<Animal>
is a List<Dog>, we can put Dogs in it safely. Doing both (read AND write)
is unsafe - that's why generic types are invariant by default."

**(3) Bridge:** "PECS is like a pantry rule. A pantry that PRODUCES
food (? extends Food) can give you any specific food - you consume what
it produces. A pantry that CONSUMES food (? super Fruit) will accept
any fruit or more general item - it's flexible about what you put in."

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| PECS mnemonic explained | 2 minutes |
| Why generics are invariant | 2-3 minutes |
| Unbounded wildcard uses | 2 minutes |
| Wildcard vs bounded type param | 2 minutes |
| Collections.copy signature | 2 minutes |
| Wildcard capture | 2-3 minutes |
| Return type wildcards | 2 minutes |
| Lower-bounded wildcard | 2 minutes |
| Type inference with wildcards | 2 minutes |

---

**Q1 (PECS mnemonic explained): Explain PECS with an example.**

A: PECS = Producer Extends, Consumer Super.

A "Producer" collection is one you READ FROM (it produces T values for you):
```java
// Producer: reads integers, produces Number for calculation
double sum(List<? extends Number> nums) { // extends = producer
    return nums.stream().mapToDouble(Number::doubleValue).sum();
}
sum(List.of(1, 2, 3));     // List<Integer> - Integer extends Number
sum(List.of(1.0, 2.0));   // List<Double> - Double extends Number
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

A "Consumer" collection is one you WRITE TO (it consumes T values from you):
```java
// Consumer: accepts Integer and supertypes
void fill(List<? super Integer> list, int n) { // super = consumer
    for (int i = 0; i < n; i++) list.add(i); // adds Integer to list
}
fill(new ArrayList<Integer>(), 5); // Integer can be added
fill(new ArrayList<Number>(), 5);  // Number list accepts Integer
fill(new ArrayList<Object>(), 5);  // Object list accepts Integer
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Kafka messaging. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

"Neither" - both read and write: use unbounded type parameter `<T>`.
"Both" contexts: use `<T>` so you have the type name to work with.

*What separates good from great:* The PECS rule comes from Effective Java
(Item 31). The formal terms are covariance (extends = read-only) and
contravariance (super = write-only). Java arrays chose covariance without
the read-only restriction (unsafe). Java generics chose invariance by
default (safe, but restrictive). Wildcards add controlled variance at
use-sites. Kotlin and Scala have declaration-site variance (annotate the
class definition to be always covariant or contravariant). Declaration-site
is simpler for API consumers; use-site is more flexible for API authors.

---

**Q2 (Why generics are invariant): Why is `List<String>` not a subtype
of `List<Object>` in Java?**

A: Because allowing this subtype relationship would create a type hole -
a way to put non-String objects into a List<String> through the List<Object>
reference.

```java
// If List<String> were a List<Object>:
List<String> strings = new ArrayList<>();
List<Object> objects = strings; // hypothetically allowed
objects.add(42); // 42 is an Object - would compile
String s = strings.get(0); // ClassCastException! Integer is not String

// Java prevents this at compile time by making generics INVARIANT.
// The compile error on line 2 above: "incompatible types"
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Arrays DO allow covariance (and regret it):
```java
String[] sa = new String[1];
Object[] oa = sa; // allowed (covariant arrays)
oa[0] = 42;  // COMPILES but throws ArrayStoreException at runtime!
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Java arrays were designed this way before generics, for compatibility
with pre-generics code (like `Arrays.sort(Object[])`). The runtime check
(ArrayStoreException) is the price.

*What separates good from great:* The array design decision is widely
considered a mistake. Generic collections correct it by using invariance
(compile-time safety instead of runtime checks). The Liskov Substitution
Principle is violated by covariant arrays: a `String[]` is NOT a
`Object[]` in the LSP sense because you can do things with `Object[]`
(add non-String) that you can't do with `String[]`. Generics enforce LSP
correctly. This is why Effective Java advises preferring generic collections
to arrays (Item 28).

---

**Q3 (Unbounded wildcard uses): When do you use `List<?>`?**

A: Use `List<?>` (unbounded wildcard) when:

1. **The method doesn't depend on the element type:**
```java
void printAll(List<?> list) {
    for (Object obj : list) System.out.println(obj); // elements as Object
}
printAll(List.of("a", "b"));   // OK
printAll(List.of(1, 2, 3));    // OK
printAll(List.of(new Object())); // OK - any list
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

2. **Checking size, clear, contains with Object:**
```java
boolean hasDuplicates(List<?> list) {
    return list.size() != new HashSet<>(list).size();
}
int indexOf(List<?> list, Object o) {
    return list.indexOf(o); // contains/indexOf take Object, not T
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

3. **instanceof check for raw type:**
```java
if (obj instanceof List) { // can't do instanceof List<String>
    List<?> list = (List<?>) obj; // safe wildcard cast
    // list.add(element); // COMPILE ERROR - correctly prevents unsafe adds
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

`List<?>` limits you to: read as Object, write null only, call methods
that don't care about element type (size, clear, isEmpty).

*What separates good from great:* `List<?>` and `List<Object>` look
similar but differ critically: `List<Object>` accepts ONLY `List<Object>`
references. `List<?>` accepts `List<Object>`, `List<String>`,
`List<Integer>` - any List. For method parameters that only OBSERVE
a list (don't modify): `List<?>` is the correct signature. It's more
permissive for callers while being safer (prevents accidental writes).
A method taking `List<Object>` forces callers to have exactly `List<Object>`,
which is rare - they usually have `List<SpecificType>`.

---

**Q4 (Wildcard vs bounded type parameter): When do you use `<T extends X>`
vs `<? extends X>`?**

A:

Use **bounded type parameter `<T extends X>`** when:
- You need to reference the type in multiple places (input AND output)
- The type must be consistent across parameters
- The type is used in the return type

```java
// T used in both parameter and return type:
<T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
// Caller: String s = max("apple", "banana"); // T=String inferred

// T relates two parameters:
<T> void copy(List<? super T> dest, List<? extends T> src) { ... }
// Both wildcards relate through T
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Use **wildcard `? extends X`** when:
- You only need to use the type once (single parameter position)
- The type doesn't need a name (not used in return type)
- You're writing a read-only parameter

```java
// Only used once, read-only, no return type dependency:
double sum(List<? extends Number> nums) { ... }
// Equivalent to:
<T extends Number> double sum(List<T> nums) { ... }
// But wildcard is preferred: simpler, communicates "read-only"
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Effective Java (Item 31) calls this
the "rule of thumb" for wildcards: "use bounded wildcards to increase
API flexibility." The general guideline: prefer wildcards in method
parameters if you don't need the type name elsewhere. Named type
parameters are for methods where the type must appear in multiple
positions or the return type. APIs with wildcards in parameters are
more flexible for callers; APIs with type parameters are more expressive
for method bodies that need to do typed operations.

---

**Q5 (Collections.copy signature): Analyze the signature
`<T> void copy(List<? super T> dest, List<? extends T> src)`.**

A: This is the canonical PECS example from the JDK.

```java
// Analysis:
// - T: the element type being copied
// - src: List<? extends T>
//   - src is a PRODUCER of T (we read T values from src)
//   - ? extends T: the actual type in src is T or a subtype of T
//   - We can read T from src safely (any subtype can be used as T)
//   - We CANNOT write to src (unknown actual subtype)
// - dest: List<? super T>
//   - dest is a CONSUMER of T (we write T values to dest)
//   - ? super T: dest holds T or a supertype of T
//   - We CAN write T to dest (T is a subtype of dest's element type)
//   - We cannot read from dest as T (might be supertype, needs cast)

// Example call:
List<Number> numbers = new ArrayList<>(List.of(1.0, 2.0, 3.0));
List<Integer> integers = List.of(10, 20, 30);

Collections.copy(numbers, integers);
// T = Integer
// dest: List<? super Integer> - numbers (List<Number>) qualifies
//   because Number is a supertype of Integer
// src: List<? extends Integer> - integers (List<Integer>) qualifies
//   because Integer extends itself

// After: numbers = [10, 20, 30]
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Kafka messaging. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Without PECS, the signature would be `<T> void copy(List<T> dest, List<T> src)`.
This would require `dest` and `src` to have EXACTLY the same type T.
You couldn't copy from `List<Integer>` to `List<Number>`.

*What separates good from great:* The PECS principle was not invented for
Java - it's the application of the Liskov Substitution Principle to
parameterized types. In type theory: covariant types can be used where
supertypes are expected (extends = covariant = producer). Contravariant
types can be used where subtypes are expected (super = contravariant =
consumer). Java's wildcards implement "use-site variance" - the variance
is declared at the usage of the type, not at the declaration of the
generic class.

---

**Q6 (Wildcard capture): What is wildcard capture?**

A: When the compiler "captures" a wildcard as a concrete type variable
for use in the method body.

```java
// Problem: can't swap elements via List<?> without capture
void swap(List<?> list, int i, int j) {
    Object temp = list.get(i);
    list.set(i, list.get(j)); // COMPILE ERROR: set expects '?'
    list.set(j, temp);        // COMPILE ERROR: same
}

// Solution: capture the wildcard via a private helper:
void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j); // compiler captures '?' as type T
}

private <T> void swapHelper(List<T> list, int i, int j) {
    T temp = list.get(i);   // T captured - same type as list elements
    list.set(i, list.get(j)); // OK: set and get same type T
    list.set(j, temp);
}
// The compiler verifies: the '?' in swap is consistent with T in swapHelper
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

The compiler performs "wildcard capture" when `swap` calls `swapHelper`:
the actual type of `?` (unknown at compile time) is captured as T in
the helper. The type checker can verify consistency without knowing
the actual type.

*What separates good from great:* Wildcard capture is how the JDK
implements many collection utility methods. `Collections.swap`,
`Collections.reverse`, and similar methods use this pattern. The pattern:
public method with `List<?>` parameter (flexible API), private helper
with `<T>` (typed implementation). The compiler automatically performs
the capture. Understanding wildcard capture distinguishes candidates
who have read the collections source code from those who have only used it.

---

**Q7 (Return type wildcards): Why should you avoid wildcards in return types?**

A: Wildcards in return types force callers to deal with wildcards in their
own code, propagating complexity.

```java
// AVOID: wildcard in return type
List<? extends Number> getNumbers() { return List.of(1, 2, 3); }

// Caller forced to deal with wildcard:
List<? extends Number> nums = getNumbers(); // must use wildcard
// nums.add(4); // COMPILE ERROR
// List<Number> plainNums = getNumbers(); // COMPILE ERROR
// The caller is stuck with an un-addable list

// PREFER: concrete return type
List<Integer> getIntegers() { return List.of(1, 2, 3); }
List<Number> getNumbers() { return new ArrayList<>(List.of(1, 2, 3)); }

// If the actual type varies: use a bounded type parameter on the method:
<T extends Number> List<T> getValues(Class<T> type) {
    return new ArrayList<>(); // concrete type at call site
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Effective Java (Item 31): "Do not use wildcard types as return types."
Using wildcards in return types leaks the API's implementation complexity
to callers.

*What separates good from great:* The exception: `<T>` type parameters
in return types are fine (they give callers flexibility via type inference).
Wildcards in return types are different: they force callers to use
wildcard syntax. As an API design rule: if you find yourself putting
`? extends` or `? super` in a return type, reconsider. Either: make
the method generic with a bounded type parameter, or commit to a specific
concrete type. The goal: callers get the most specific usable type
from your API without needing to know about the implementation's variance.

---

**Q8 (Lower-bounded wildcard): When exactly do you use `? super T`?**

A: `? super T` (lower-bounded wildcard) means "T or any ancestor of T."
Used when the method writes T values to a container.

```java
// Consumer pattern: writing Integer values to any Integer-compatible list
void fillRange(List<? super Integer> list, int start, int count) {
    for (int i = start; i < start + count; i++) {
        list.add(i); // Integer can be added to List<Integer>,
                     // List<Number>, List<Object>
    }
}

// TreeSet with custom comparator:
TreeSet<? super Integer> set = new TreeSet<>(Comparator.naturalOrder());
// Can't create a TreeSet<? super Integer> directly - use concrete type
// Lower bound is usually for PARAMETERS, not variable declarations

// Comparator.naturalOrder() return type:
// static <T extends Comparable<? super T>> Comparator<T> naturalOrder()
// "? super T" for the Comparable bound: T itself or any supertype
// of T can be Comparable - allows T to inherit compareTo from a parent class

// Real use: Comparator.comparing with Function:
// static <T, U extends Comparable<? super U>> Comparator<T> comparing(
//     Function<? super T, ? extends U> keyExtractor)
// "? super T": key extractor accepts T or supertype (flexible input)
// "? extends U": result is U or subtype (safe use as U)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using Kafka messaging. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `Comparable<? super T>` appears in
many JDK signatures. It means: T is Comparable to something that T
extends. This allows a `Dog` class to inherit `compareTo` from `Animal`
(declared as `Animal implements Comparable<Animal>`) and still work in
`TreeSet<Dog>`. Without `? super`, you'd need `Dog` to explicitly implement
`Comparable<Dog>`, preventing inheritance of the comparison logic.
This subtlety is tested in senior interviews.

---

**Q9 (Type inference with wildcards): How does Java's type inference
interact with wildcards?**

A:
```java
// Type inference fills in T from context:
List<String> strings = new ArrayList<>();  // T=String inferred
List<Integer> ints = Collections.emptyList(); // T=Integer inferred

// Diamond operator (Java 7): infer type from left side
Map<String, List<Integer>> map = new HashMap<>(); // inferred

// Wildcard and inference:
List<? extends Number> nums = List.of(1, 2, 3); // ? captured as Integer

// Method inference with wildcards:
<T> T first(List<? extends T> list) { return list.get(0); }
Number n = first(List.of(1, 2, 3)); // T=Number, ? extends Number satisfied
Object o = first(List.of("a"));      // T=Object, ? extends Object satisfied
String s = first(List.of("a", "b")); // T=String

// Target typing (Java 8): infer from method parameter type
Collections.sort(strings);          // T=String inferred from strings
Collections.sort(strings, Comparator.reverseOrder()); // T=String inferred

// When inference fails: provide explicit type argument
List<Number> mixed = Collections.<Number>emptyList(); // explicit T=Number
// Without explicit: Collections.emptyList() may infer as List<Object>
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Java 8 improved type inference
significantly (target typing). Before Java 8: `Collections.emptyList()`
often required an explicit type argument. With Java 8: the compiler
infers from the assignment target. Inference failures are rare but
notable: when calling a method with a wildcard return type as an
argument to another generic method, the compiler may fail to infer
the type and require explicit annotation. Understanding this helps
debug "cannot infer type arguments" compile errors.

---

### ⚖️ Comparison Table

| Wildcard Type | Read Access | Write Access | Use Case |
|---|---|---|---|
| `List<T>` | As T | As T | Both read and write, type matters |
| `List<? extends T>` | As T | None (except null) | Read-only (producer) |
| `List<? super T>` | As Object only | As T | Write-only (consumer) |
| `List<?>` | As Object only | None (except null) | Type-agnostic operations |
| `List<Object>` | As Object | As Object | Explicitly heterogeneous |

---

## Effective Java Item Clusters

---

### 🎯 Model Answer

**30 seconds:**
> "Effective Java" by Joshua Bloch (3rd edition, 2018) is the definitive
> Java best-practices guide. Key clusters: (1) Static factory methods
> over constructors (readability, caching, subtyping). (2) Builder
> pattern for objects with many parameters. (3) Favor composition over
> inheritance. (4) Program to interfaces, not implementations.
> (5) Prefer immutable classes. (6) Use generics, not raw types.
> These items represent distilled experience from the Java platform
> team and remain the benchmark for professional Java code.

**3 minutes (Senior):**
> The clusters matter because they address recurring design problems:
> Static factories: `Optional.of()`, `Collections.emptyList()` are static
> factories. They can return cached instances (empty collections), have
> meaningful names, return subtypes without exposing them, and reduce
> verbosity. Builders: `HttpRequest.newBuilder()...build()` - construction
> with named parameters, validation at build time, optional parameters.
>
> Composition over inheritance: `extends` creates tight coupling and
> violates encapsulation (subclass depends on superclass internals).
> Composition wraps the underlying object; changes in the wrapped class
> don't automatically break the wrapper. Item 18: "Inheritance violates
> encapsulation." Real-world: `InstrumentedSet` wrapping `HashSet` vs
> extending it (the counting-add example in Effective Java).
>
> Immutability: thread-safe by default, simple, cacheable. Strings,
> Integer, LocalDate are immutable. Making a class immutable: all fields
> final, no setters, defensive copies of mutable fields in constructors
> and accessors.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Effective Java item clusters - let me cover the key
patterns: static factory methods, builders, composition over inheritance,
immutability, and programming to interfaces."

**(2) First principles:** "From first principles: these items address
the most common design mistakes in Java. Each one is a specific trade-off
with a recommended resolution based on decades of Java ecosystem experience."

**(3) Bridge:** "Effective Java items are like a professional
electrician's code book. You could wire a house your own way, but the
code represents accumulated safety wisdom. These patterns represent
accumulated Java safety and maintainability wisdom."

### 📘 Concept Explanation

**Cluster 1: Object Creation and Destruction (Items 1-9)**

Key items: 1 (static factory), 2 (builder), 3 (singleton), 5 (DI),
6 (avoid unnecessary objects), 7 (eliminate obsolete references).

**Cluster 2: Methods Common to All Objects (Items 10-14)**

Key items: 10 (equals contract), 11 (hashCode contract), 12 (toString),
13 (clone), 14 (Comparable).

**Cluster 3: Classes and Interfaces (Items 15-25)**

Key items: 15 (minimize accessibility), 16 (accessor methods), 17 (immutable),
18 (composition over inheritance), 19 (design for inheritance), 21 (interface defaults).

**Cluster 4: Generics (Items 26-33)**

Key items: 26 (no raw types), 27 (unchecked warnings), 28 (lists over arrays),
29 (generic types), 30 (generic methods), 31 (bounded wildcards), 33 (typesafe containers).

---


### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Static factory benefits | 2 minutes |
| Builder pattern when needed | 2 minutes |
| Composition vs inheritance | 3 minutes |
| Immutability benefits | 2 minutes |
| Program to interfaces | 2 minutes |
| Equals/hashCode contract | 2 minutes |
| Item 7: obsolete references | 2 minutes |
| Item 17: immutable class rules | 2 minutes |
| Item 28: lists over arrays | 2 minutes |

---

**Q1 (Static factory benefits): What are the advantages of static factory
methods over constructors?**

A: Item 1 from Effective Java lists five advantages:

**1. They have names (communicate intent):**
```java
new BigInteger(int, int, Random) // which arg is which?
BigInteger.probablePrime(bitLength, random) // clear!
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**2. Not required to create new object each call (caching):**
```java
Integer.valueOf(127) // returns cached instance for -128..127
Boolean.valueOf(true) // always returns same Boolean.TRUE object
// constructors MUST create new objects
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**3. Can return any subtype (implementation hiding):**
```java
// Collections.emptyList() returns Collections.EMPTY_LIST - package-private class
// Caller sees List<T>, doesn't need to know the actual class
List<String> empty = Collections.emptyList(); // java.util.Collections.EmptyList
// This class is not public - hidden implementation
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**4. The returned class can vary based on parameters:**
```java
EnumSet.of(Day.MON, Day.TUE) // returns RegularEnumSet (long bit field)
EnumSet.noneOf(Day.class)    // may return JumboEnumSet for large enums
// Optimal implementation chosen for you
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**5. The returned class need not exist when writing the factory:**
Service Provider Framework pattern: `DriverManager.getConnection(url)`.

*What separates good from great:* The naming convention for static factories:
`of` (conversion with same type), `from` (type conversion), `valueOf`
(more verbose `of`), `instance` or `getInstance` (for singletons/shared
instances), `create` or `newInstance` (guaranteed new instance), `getType`
(factory in a different class, e.g., `Files.newBufferedReader`). Following
these conventions makes the API recognizable to Java programmers.

---

**Q2 (Builder pattern when needed): When should you use the Builder pattern?**

A: Use Builder when (Item 2):

**1. Constructor has 4+ parameters (especially optional ones):**

```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// BAD: telescoping constructors
NutritionFacts cola = new NutritionFacts(240, 8, 100, 0, 35, 27);
// Parameters: servingSize, servings, calories, fat, sodium, carbs
// Which number is sodium? 0 fat? 35 sodium?

// GOOD: Builder
NutritionFacts cola = new NutritionFacts.Builder(240, 8) // required
    .calories(100)
    .sodium(35)
    .carbohydrates(27)
    .build();
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

**2. JavaBeans pattern would leave object in inconsistent state:**
```java
// JavaBeans: setters can leave partially constructed object
NutritionFacts cola = new NutritionFacts();
cola.setServingSize(240); // incomplete object accessible here!
cola.setServings(8);      // still incomplete
cola.setCalories(100);    // now valid? or not?
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**3. Class hierarchy with builders:**
```java
abstract class Pizza {
    abstract static class Builder<T extends Builder<T>> {
        abstract Pizza build();
    }
}
class NYPizza extends Pizza {
    enum Size { SMALL, MEDIUM, LARGE }
    static class Builder extends Pizza.Builder<Builder> {
        Builder size(Size size) { ... return this; }
    }
}
// Covariant return types and self-type pattern enable fluent subclass builders
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using enum. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Lombok's `@Builder` annotation generates
the Builder boilerplate at compile time. Spring Boot's configuration
objects use builders. Jackson uses builders for ObjectMapper configuration.
The test for when to use Builder: "Would a reasonable developer look at
this constructor call and know what each argument does?" If no: use Builder.
Builders are especially valuable when many optional parameters exist -
you only set what's relevant, defaults handle the rest.

---

**Q3 (Composition vs inheritance): When is inheritance appropriate
vs composition?**

A: The LSP test: is the relationship genuinely IS-A with full behavioral
compatibility?

**Inheritance is appropriate:**
```java
// Stack IS-A AbstractSequentialList? NO! (stack shouldn't expose get(i))
// A Vector IS-A Stack? NO! (Bloch's example of Java's design mistake)

// Car IS-A Vehicle? YES (if Vehicle defines all operations Car supports)
abstract class Vehicle {
    abstract void start();
    abstract void stop();
    // All operations that ALL vehicles support
}
class Car extends Vehicle {
    // Liskov: anywhere Vehicle is used, Car works identically
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Composition is appropriate:**
```java
// Logger that wraps an existing logger:
class TimedLogger implements Logger {
    private final Logger delegate;      // composition
    private final Clock clock;

    TimedLogger(Logger delegate, Clock clock) {
        this.delegate = delegate;
        this.clock = clock;
    }

    @Override
    public void log(String msg) {
        delegate.log("[" + clock.now() + "] " + msg); // adds timing
    }
    // All other Logger methods delegate:
    // delegate.setLevel, delegate.isEnabled, etc.
}
// Change: if Logger adds a new method, TimedLogger delegates it
// without any change to TimedLogger (doesn't break)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The "fragile base class problem": when
a subclass's behavior breaks after a change to the base class that didn't
intend to affect subclasses. Composition avoids this: the wrapped object's
internals are irrelevant to the wrapper. Real-world: `Properties extends
Hashtable` in the JDK is a design mistake (admitted by Bloch). Properties
inherits `put(Object, Object)` from Hashtable, allowing non-String values
even though Properties should only hold String->String. Fixing this would
be a backward incompatible change. The lesson: wrong use of inheritance
creates permanent API debt.

---

**Q4 (Immutability benefits): What makes a class immutable and why
prefer immutability?**

A: **Five rules for immutable classes (Item 17):**
1. Don't provide methods that modify state (no setters)
2. Ensure the class can't be extended (final class or private constructors)
3. Make all fields final
4. Make all fields private
5. Ensure exclusive access to mutable components (defensive copies)

```java
// Immutable class:
public final class Point {
    private final double x;
    private final double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    // "Wither" methods return new instances instead of mutating:
    public Point translate(double dx, double dy) {
        return new Point(x + dx, y + dy);
    }

    public double getX() { return x; }
    public double getY() { return y; }
    // equals, hashCode, toString...
}

// Defensive copy for mutable fields:
public final class DateRange {
    private final Date start; // Date is mutable!
    private final Date end;

    public DateRange(Date start, Date end) {
        this.start = new Date(start.getTime()); // defensive copy IN
        this.end = new Date(end.getTime());
    }
    public Date getStart() {
        return new Date(start.getTime()); // defensive copy OUT
    }
    // Better: use LocalDate or Instant (immutable) instead of Date!
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Benefits of immutability:**
- Thread-safe: no synchronization needed (read-only access is always safe)
- Cache-friendly: hash code computed once, shared instances safe
- Simple reasoning: state never changes - no "what changed my object?"
- Failure atomicity: operations either fully succeed or fully fail (no partial state)

*What separates good from great:* The biggest practical benefit in modern
Java: immutable objects work safely in concurrent environments without
any synchronization. `String`, `Integer`, `LocalDate`, `BigDecimal` are
immutable - safe to share across threads. Mutable objects (`Date`,
`ArrayList`) require external synchronization for concurrent access.
Java records (Java 16) enforce the immutability pattern for data carriers:
final fields, only accessor methods, generated equals/hashCode/toString.
For domain objects that are purely data: records are the modern idiomatic
choice over manual immutable classes.

---

**Q5 (Program to interfaces): What does "program to interfaces, not
implementations" mean in practice?**

A: Declare variables, parameters, and return types using interface types.
Reserve concrete types for instantiation (and local variables where the
type is clear from context).


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// BAD: concrete types in API
ArrayList<String> getNames() { return new ArrayList<>(); }
void process(HashMap<String, User> users) { ... }

// GOOD: interface types in API
List<String> getNames() { return new ArrayList<>(); }
void process(Map<String, User> users) { ... }

// BAD: concrete type in variable declaration
ArrayList<String> names = new ArrayList<>();

// GOOD: interface type (diamond infers the concrete type)
List<String> names = new ArrayList<>();

// Benefit: easy to change implementation:
// List<String> names = new LinkedList<>(); // change only one place
// If "names" was ArrayList everywhere: change in many places

// Interface in API: callers can pass any Map implementation:
process(new HashMap<>()); // works
process(new TreeMap<>());  // works
process(new ConcurrentHashMap<>()); // works
// If signature used HashMap: callers couldn't pass TreeMap!

// When to use concrete type:
// - When the concrete type's extra methods are needed:
ArrayDeque<String> deque = new ArrayDeque<>();
deque.push("first"); // ArrayDeque.push() not in Deque? (it is, via Deque)
// But: if you need ArrayDeque-specific methods not in Deque,
// use ArrayDeque in the variable type
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **WHAT BREAKS: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* This item applies to ALL types, not
just collections. `InputStream` not `FileInputStream` in method parameters
(caller decides the source). `Executor` not `ThreadPoolExecutor`. `Clock`
not `SystemClock` (enables test injection of a mock clock). The practical
test: "Does this API care HOW something is implemented, or just WHAT
it does?" If "what it does": use the interface. The secondary benefit:
testability - interfaces can be mocked (`Mockito.mock(List.class)`) while
final implementations can't.

---

**Q6 (Equals/hashCode contract): What are the rules for equals() and
hashCode()?**

A: **equals() contract (5 rules, Item 10):**
1. **Reflexive:** `x.equals(x)` = true
2. **Symmetric:** `x.equals(y)` = `y.equals(x)`
3. **Transitive:** if `x.equals(y)` and `y.equals(z)` then `x.equals(z)`
4. **Consistent:** same result for same unchanged objects
5. **Null:** `x.equals(null)` = false (never throws NPE)

**hashCode() contract (Item 11):**
1. Consistent with equals: if `x.equals(y)` then `x.hashCode() == y.hashCode()`
2. NOT required that unequal objects have different hash codes (collisions allowed)
3. Consistent across JVM runs UNLESS fields used in computation change

```java
// Common violation: symmetry broken by mixed-type equals
class Point {
    int x, y;
    @Override public boolean equals(Object o) {
        if (o instanceof Point p) return x == p.x && y == p.y;
        if (o instanceof ColorPoint cp) return x == cp.x && y == cp.y; // wrong!
        return false;
    }
}
class ColorPoint extends Point {
    Color color;
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint cp)) return false;
        return super.equals(cp) && color.equals(cp.color);
    }
}
// point.equals(colorPoint) = true (ignores color)
// colorPoint.equals(point) = false (not a ColorPoint)
// ASYMMETRIC! Violates contract.

// Fix: don't mix types in equals. Use composition.
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The impossibility result (Bloch, citing
Liskov): you CANNOT extend an instantiable class and add a value component
while preserving the equals contract. The solution: use composition
(ColorPoint HAS-A Point, not IS-A Point) and add a `asPoint()` view
method. Java records handle this correctly: equals compares all components
of the same record type, never mixing with supertypes.

---

**Q7 (Item 7 obsolete references): What are obsolete references and
how do you handle them?**

A: An "obsolete reference" is a reference your program holds but will
never access again. The GC can't collect the referenced object. This
is Java's version of a memory leak.

```java
// Classic example: a custom stack implementation
class Stack<E> {
    private Object[] elements;
    private int size;

    E pop() {
        if (size == 0) throw new EmptyStackException();
        return (E) elements[--size];
        // BUG: elements[size] still holds the reference!
        // Object won't be GC'd even after pop()!
    }

    // FIX: null out the obsolete reference:
    E popFixed() {
        if (size == 0) throw new EmptyStackException();
        @SuppressWarnings("unchecked")
        E result = (E) elements[--size];
        elements[size] = null; // eliminate obsolete reference
        return result;
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**Common sources of obsolete references:**
1. Caches: entries that are no longer needed but still in the cache.
   Fix: use `WeakHashMap` for cache keys; entries expire when key has no strong reference.
2. Listeners: registered but never deregistered.
   Fix: `removeEventListener` in `close()` or `AutoCloseable`.
3. Callbacks: stored but never cleaned up.
   Fix: WeakReference-based callback registry.

*What separates good from great:* Java's GC doesn't prevent memory leaks;
it prevents dangling pointer bugs. Memory leaks in Java come from holding
references to objects longer than needed. Tools for diagnosis: Eclipse MAT
(Memory Analyzer Tool), VisualVM heap dump analysis, `-XX:+HeapDumpOnOutOfMemoryError`.
The leak signature: heap usage grows monotonically even under steady load.
Profiler shows a specific class accumulating instances. Finding it in a
heap dump: sort by class instance count, look for what's holding references.

---

**Q8 (Item 17 immutable class rules): How do you make a Java class fully
immutable?**

A: The five rules plus practical implementation:

```java
public final class ImmutablePerson {
    // Rule 3: all fields final
    // Rule 4: all fields private
    private final String name;
    private final LocalDate birthDate;
    private final List<String> nicknames; // mutable - needs defense

    // Rule 5: defensive copies of mutable fields in constructor
    public ImmutablePerson(String name, LocalDate birthDate,
                           List<String> nicknames) {
        this.name = Objects.requireNonNull(name);
        this.birthDate = Objects.requireNonNull(birthDate);
        // Defensive copy of mutable list:
        this.nicknames = List.copyOf(nicknames); // Java 10: unmodifiable copy
        // or: Collections.unmodifiableList(new ArrayList<>(nicknames));
    }

    // Rule 1: no setters, no mutating methods
    // "Wither" method returns new instance:
    public ImmutablePerson withName(String newName) {
        return new ImmutablePerson(newName, birthDate, nicknames);
    }

    // Rule 5: defensive copies in accessors
    // (not needed here: nicknames is already unmodifiable)
    public List<String> getNicknames() { return nicknames; }

    // Rule 2: class is final (can't be extended to add mutable state)
}

// Alternatively: make constructor private + static factory
// to enable returning cached instances or subclasses
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `List.copyOf()` (Java 10) creates an
unmodifiable copy in one call - the defensive copy IN the constructor.
Returning the unmodifiable list from the accessor means callers can't
mutate it. For deeper immutability: if the list contains mutable objects
(e.g., `List<Address>` where Address is mutable), you'd need to
deep-copy each element too. Truly immutable classes work deeply: all
reachable objects are also immutable. Java records achieve this automatically
for their components if the component types are immutable.

---

**Q9 (Item 28 lists over arrays): Why does Effective Java say to prefer
lists over arrays?**

A: Item 28: "Prefer lists to arrays."

**Three reasons:**

**1. Arrays are covariant, generics are invariant:**
```java
String[] sa = new String[1];
Object[] oa = sa;       // OK at compile time
oa[0] = 42;            // ArrayStoreException at runtime!

List<String> ls = new ArrayList<>();
List<Object> lo = ls;  // COMPILE ERROR: correctly rejected!
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**2. Arrays are reifiable (runtime type), generics are erased:**
```java
String[] sa = new String[1]; // new String[]: runtime type is String[]
// Generic array:
List<String>[] lsa = new List<String>[1]; // COMPILE ERROR: generic array!
// You'd get: List[]   (erased) - type safety lost
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

**3. Lists provide type safety, arrays don't:**
```java
// With arrays: ClassCastException at runtime
// With generics: compile-time error
// Prefer compile-time errors
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// Practical conversion:
// BAD: generic array creation
@SuppressWarnings("unchecked")
T[] array = (T[]) new Object[n]; // need unchecked cast, type safety lost

// GOOD: generic list
List<T> list = new ArrayList<>(n); // no unchecked cast, fully type-safe
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The legitimate uses for arrays in modern
Java: performance-critical code where boxing overhead of `List<Integer>`
vs `int[]` matters. JDK internals use arrays (ArrayList's backing store
is `Object[]`). For public APIs: `List<T>` is safer and more flexible.
For primitive numeric work: primitive arrays (`int[]`, `double[]`) are
always preferred over `List<Integer>` to avoid boxing. Java's `Arrays`
utility class (sort, fill, copyOf, binarySearch) provides collection-like
operations on primitive arrays.

---

### ⚖️ Comparison Table

| Pattern | Static Factory | Constructor | Builder |
|---|---|---|---|
| Can have name | Yes | No | Via method names |
| Can return cached | Yes | No | No |
| Can return subtype | Yes | No | Yes (factory method) |
| Enforces invariants | At factory time | At construction | At build time |
| Optional parameters | Methods | Overloading/null | Optional setters |
| Readability (many params) | Poor | Poor | Excellent |
| Suitable for | Small APIs | Few, clear params | 4+ params |

---


# Java Core - L3 DateTime and Records

## Java Date and Time API

---

### 🎯 Model Answer

**30 seconds:**
> Java 8 introduced `java.time` - a comprehensive, immutable date/time API.
> Key types: `LocalDate` (date only), `LocalTime` (time only), `LocalDateTime`
> (both, no timezone), `ZonedDateTime` (with timezone), `Instant` (machine
> timestamp, UTC epoch seconds), `Duration` (time-based amount), `Period`
> (date-based amount). All are immutable. `DateTimeFormatter` for formatting/parsing.
> The old `java.util.Date` and `Calendar` are mutable, not thread-safe, and
> replaced by `java.time` in modern code.

**3 minutes (Senior):**
> Key design: `Instant` is a machine timestamp (epoch seconds + nanos since
> 1970-01-01T00:00:00Z). `ZonedDateTime` is a human timestamp with timezone
> rules (DST, offsets). Always use `Instant` for storing/comparing times,
> `ZonedDateTime` for user display. Timezone pitfalls: `ZoneId.of("Asia/Kolkata")`
> (name-based, observes DST rules) vs `ZoneOffset.of("+05:30")` (fixed offset,
> no DST). Storing `ZoneOffset` loses historical DST info.
>
> `DateTimeFormatter` is thread-safe (immutable) - create once as a static
> constant. `SimpleDateFormat` is NOT thread-safe - never share instances.
> `ChronoUnit` for field-based arithmetic: `ChronoUnit.DAYS.between(d1, d2)`.
> Business day calculations require `TemporalAdjusters` or a library like
> ThreeTenExtra. `Duration.between(t1, t2)` for time-based differences.
> `Period.between(d1, d2)` for date-based (days/months/years).

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Java date/time API - let me cover LocalDate/Time,
Instant, ZonedDateTime, formatting, arithmetic, and the key differences
from the old Date/Calendar API."

**(2) First principles:** "From first principles: time has two views -
machine time (a number: nanoseconds since epoch, timezone-agnostic) and
human time (2024-01-15 in Tokyo timezone). Java time provides both.
Immutability ensures thread safety and prevents the 'share and mutate'
bugs common with Date/Calendar."

**(3) Bridge:** "java.time is like a precision watch collection. Instant
is a stopwatch (pure elapsed time). LocalDateTime is a wall clock
(shows 3pm but doesn't say where). ZonedDateTime is a world clock
(3pm in New York, accounting for DST). Each tool for its purpose."

---


### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Instant vs LocalDateTime | 2 minutes |
| Timezone best practices | 2 minutes |
| DateTimeFormatter thread safety | 2 minutes |
| Duration vs Period | 2 minutes |
| DST handling | 2-3 minutes |
| Database integration | 2 minutes |
| java.util.Date migration | 2 minutes |
| TemporalAdjusters | 90 seconds |
| Calendar vs java.time | 2 minutes |

---

**Q1 (Instant vs LocalDateTime): When do you use Instant vs LocalDateTime?**

A: **Instant:** a point on the timeline, independent of timezone.
Expressed as seconds + nanoseconds since 1970-01-01T00:00:00Z (UTC epoch).
Use for: logging, event timestamps, database storage, comparing events,
measuring durations.

**LocalDateTime:** a date-time WITHOUT timezone information. Ambiguous
without knowing where.
Use for: representing schedule times in a specific local context,
birth dates where timezone doesn't matter, business hours, alarm times.

```java
// Logging (when did this happen): Instant
log.info("Request received at {}", Instant.now()); // UTC, unambiguous

// User birthday: LocalDate (no time, no timezone)
LocalDate birthday = LocalDate.of(1990, Month.JUNE, 15);
// Age calculation:
long age = ChronoUnit.YEARS.between(birthday, LocalDate.now());

// Meeting schedule: ZonedDateTime (timezone matters for DST)
ZonedDateTime meeting = ZonedDateTime.of(
    2024, 1, 15, 14, 30, 0, 0,
    ZoneId.of("America/New_York"));

// Store in DB: Instant -> TIMESTAMP
repository.save(event.withTimestamp(Instant.now()));
// Or: ZonedDateTime.toInstant() before storing
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The database storage decision is critical.
TIMESTAMP in SQL stores a moment in time (UTC-equivalent). Java's `Instant`
maps perfectly. `TIMESTAMP WITH TIME ZONE` in PostgreSQL also stores moments.
`TIMESTAMP WITHOUT TIME ZONE` stores "wall clock time" with no timezone -
matches `LocalDateTime`. A common production bug: storing `ZonedDateTime`
as text (with timezone string) in a VARCHAR. Later changes to timezone
rules (governments change DST rules) make old data display incorrectly.
Correct: store `Instant` (epoch seconds) + timezone ID string separately.

---

**Q2 (Timezone best practices): What are the best practices for
handling timezones in Java?**

A:
1. **Store as UTC/Instant:** database timestamps in UTC, expose as `Instant`
2. **Convert at the boundary:** convert to user timezone only in the UI layer
3. **Never assume system timezone:** `ZoneId.systemDefault()` changes based on deployment
4. **Use zone names, not offsets:** `Asia/Kolkata` not `+05:30` (offset changes with DST)
5. **Test DST transitions:** March/November in US, October/April in EU

```java
// Store user's timezone preference:
String userTimezone = "America/New_York"; // from user profile
ZoneId userZone = ZoneId.of(userTimezone); // validate at registration

// Convert for display:
Instant eventInstant = event.getTimestamp(); // UTC from DB
ZonedDateTime userTime = eventInstant.atZone(userZone);
String display = userTime.format(DateTimeFormatter
    .ofPattern("MMM d, yyyy h:mm a z")); // "Jan 15, 2024 9:00 AM EST"

// WRONG: storing timezone offset:
ZoneOffset offset = ZoneOffset.of("+05:30"); // India Standard Time
// Problem: India doesn't observe DST currently, but what if rules change?
// A zone NAME like "Asia/Kolkata" is future-proof; an offset is not.

// Time arithmetic: add 24 hours or 1 day?
ZonedDateTime now = ZonedDateTime.now(ZoneId.of("America/New_York"));
// "Same time tomorrow" in user's timezone (handles DST):
ZonedDateTime tomorrow = now.plusDays(1); // adds calendar day
// vs:
Instant plus24h = now.toInstant().plus(Duration.ofHours(24)); // adds 24h
// On DST "spring forward" day: plusDays(1) = 23h later in UTC!
// Most users expect "tomorrow 3pm" = same wall clock time tomorrow
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The `plusDays(1)` vs `plus(24h)` choice
is significant for user-facing scheduling. `ZonedDateTime.plusDays(1)` adds
a "calendar day" - the result is the same wall clock time tomorrow, even
if that's only 23 or 25 hours due to DST. `Instant.plus(Duration.ofHours(24))`
adds exactly 86400 seconds. Which is correct? For recurring alarms, calendar
events, "daily at 9am": use `plusDays(1)` (calendar semantics). For
monitoring cooldowns, retry delays, timeout windows: use Instant arithmetic
(exact seconds).

---

**Q3 (DateTimeFormatter thread safety): Why is DateTimeFormatter
thread-safe but SimpleDateFormat is not?**

A: `DateTimeFormatter` is immutable - all state (pattern, locale, zone)
is set at construction and never changes. Multiple threads can share one
instance safely.

`SimpleDateFormat` maintains mutable internal state during formatting
(calendar fields, format buffers) - shared access corrupts this state.

```java
// WRONG: shared SimpleDateFormat
private static final SimpleDateFormat SDF =
    new SimpleDateFormat("yyyy-MM-dd");
// Thread 1: sdf.format(date1) - modifies internal calendar
// Thread 2: sdf.format(date2) - corrupts Thread 1's in-progress format
// Result: garbled dates, wrong output, or exceptions

// FIX 1: ThreadLocal (one per thread, no sharing)
private static final ThreadLocal<SimpleDateFormat> TL_SDF =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
TL_SDF.get().format(date); // each thread has its own instance

// CORRECT: DateTimeFormatter is immutable, share freely
private static final DateTimeFormatter DTF =
    DateTimeFormatter.ofPattern("yyyy-MM-dd");
DTF.format(LocalDate.now()); // safe from any thread
DTF.format(LocalDate.now()); // same instance, concurrent calls OK

// Formatting:
LocalDate date = LocalDate.of(2024, 1, 15);
String s = date.format(DTF); // "2024-01-15"
String s = DTF.format(date); // same result, method on formatter

// Parsing:
LocalDate parsed = LocalDate.parse("2024-01-15", DTF);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The `ThreadLocal<SimpleDateFormat>`
workaround was the pre-Java 8 standard for shared date formatting. With
Java 8+: use `DateTimeFormatter` - it's both correct and more convenient.
A code review flag: any static field of type `SimpleDateFormat` without
`ThreadLocal` wrapping is a concurrency bug waiting to surface under load.
This bug is often caught only in production when concurrent requests
corrupt each other's date formatting.

---

**Q4 (Duration vs Period): When do you use Duration vs Period?**

A: **Duration:** time-based amount (hours, minutes, seconds, nanoseconds).
Works with `Instant`, `LocalTime`, `LocalDateTime`, `ZonedDateTime`.

**Period:** date-based amount (years, months, days).
Works with `LocalDate` and `LocalDateTime`.

```java
// Duration: exact time difference
Instant start = Instant.parse("2024-01-15T10:00:00Z");
Instant end   = Instant.parse("2024-01-15T14:30:00Z");
Duration dur = Duration.between(start, end);
dur.toHours();        // 4
dur.toMinutes();      // 270 (TOTAL minutes, not the "minutes part"!)
dur.toMinutesPart();  // 30 (Java 9: the minutes component 0-59)
dur.toSecondsPart();  // 0  (Java 9: the seconds component 0-59)

// Period: calendar date difference
LocalDate d1 = LocalDate.of(2024, 1, 1);
LocalDate d2 = LocalDate.of(2025, 3, 15);
Period p = Period.between(d1, d2);
p.getYears();    // 1
p.getMonths();   // 2
p.getDays();     // 14
// "1 year, 2 months, and 14 days"

// ChronoUnit for total count:
long totalDays = ChronoUnit.DAYS.between(d1, d2); // 440

// Duration factory methods:
Duration.ofHours(3);
Duration.ofMinutes(90);
Duration.ofSeconds(3600);
Duration.parse("PT3H30M"); // ISO-8601 duration format

// Period factory:
Period.ofDays(7);
Period.ofMonths(3);
Period.of(1, 2, 3); // 1 year, 2 months, 3 days
Period.parse("P1Y2M3D"); // ISO-8601 period format
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The `Duration` vs `Period` distinction
matters in banking and finance: a loan with "3 month term" uses `Period.ofMonths(3)` (calendar months, different number of days). An SLA
of "72 hours" uses `Duration.ofHours(72)` (exact 72 * 3600 seconds).
Mixing them causes subtle bugs: "add 1 month" from Jan 31 = Feb 28
(Period) vs "+30 days" (Duration) = March 1 or 2. These are different
dates! `Period.addTo(LocalDate)` handles month-end correctly (Feb 28/29
on overflow).

---

**Q5 (DST handling): How do you handle daylight saving time transitions?**

A:
```java
// "Spring forward": clock skips 02:00-03:00 in America/New_York on 2024-03-10
ZoneId nyZone = ZoneId.of("America/New_York");
LocalDateTime gapTime = LocalDateTime.of(2024, 3, 10, 2, 30);

// LENIENT resolution (default): adjusts to next valid time
ZonedDateTime zdt = gapTime.atZone(nyZone);
System.out.println(zdt); // 2024-03-10T03:30-04:00[America/New_York]
// 02:30 became 03:30 (jumped forward)

// STRICT resolution: throws exception for invalid time
try {
    ZonedDateTime strict = ZonedDateTime.ofStrict(gapTime, ZoneOffset.of("-05:00"), nyZone);
} catch (DateTimeException e) {
    System.out.println("Time does not exist: " + e.getMessage());
}

// "Fall back": clock goes back at 02:00 in America/New_York on 2024-11-03
// 01:30 occurs twice (EST and EDT overlap)
LocalDateTime overlap = LocalDateTime.of(2024, 11, 3, 1, 30);
ZonedDateTime earlier = ZonedDateTime.of(overlap, ZoneOffset.of("-04:00"), nyZone); // EDT
ZonedDateTime later   = ZonedDateTime.of(overlap, ZoneOffset.of("-05:00"), nyZone); // EST

// Java chooses earlier occurrence by default with atZone()

// Best practice: work in UTC, convert only for display
// DST issues vanish when you store/compute in Instant:
Instant eventInstant = Instant.parse("2024-03-10T07:30:00Z");
// This is always 07:30 UTC, regardless of DST
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* DST bugs are the most common datetime
production issues. They manifest as: appointments shifted by 1 hour,
scheduler jobs running twice or not at all, SLA calculations off by
an hour. The architectural fix: all internal computation in UTC (Instant),
all display in user timezone. Never compute durations or time differences
using `ZonedDateTime` arithmetic (DST transitions make it unpredictable).
Use `Duration.between(instant1, instant2)` for exact elapsed time.

---

**Q6 (Database integration): How do you map java.time types to database columns?**

A:

| java.time Type | SQL Type | JDBC Method |
|---|---|---|
| `Instant` | `TIMESTAMP` | `setObject(n, instant)` (JDBC 4.2) |
| `LocalDate` | `DATE` | `setObject(n, localDate)` (JDBC 4.2) |
| `LocalTime` | `TIME` | `setObject(n, localTime)` (JDBC 4.2) |
| `LocalDateTime` | `TIMESTAMP` | `setObject(n, localDateTime)` (JDBC 4.2) |
| `ZonedDateTime` | `TIMESTAMP WITH TIME ZONE` | Via `OffsetDateTime` |

```java
// JPA / Hibernate (automatic mapping in Hibernate 5+):
@Entity
class Event {
    @Column(name = "created_at")
    private Instant createdAt; // maps to TIMESTAMP (UTC)

    @Column(name = "event_date")
    private LocalDate eventDate; // maps to DATE

    @Column(name = "start_time")
    private LocalDateTime startTime; // maps to TIMESTAMP (without TZ)
}

// Spring Data JPA: works out of the box with java.time types
// No @Temporal annotation needed (that was for java.util.Date/Calendar)

// Legacy java.util.Date ↔ java.time conversion:
Date legacyDate = Date.from(instant);          // Instant -> Date
Instant inst = legacyDate.toInstant();         // Date -> Instant

java.sql.Date sqlDate = java.sql.Date.valueOf(localDate); // LocalDate -> SQL Date
LocalDate localDate = sqlDate.toLocalDate();              // SQL Date -> LocalDate

java.sql.Timestamp ts = Timestamp.from(instant);  // Instant -> SQL Timestamp
Instant inst = ts.toInstant();                     // SQL Timestamp -> Instant
```

> **Code walkthrough:** This Unknown example demonstrates metadata declaration. **KEY MECHANISM:** annotations are processed at compile-time or runtime via reflection. **WHY IT MATTERS:** annotation processing adds compile time; runtime reflection disables JIT optimizations. **TAKEAWAY: prefer compile-time annotation processors (APT) over runtime reflection for performance.**

*What separates good from great:* PostgreSQL stores TIMESTAMP WITH TIME ZONE
as UTC internally (converts from local time at insert). TIMESTAMP WITHOUT
TIME ZONE stores the literal values with no conversion. When querying
TIMESTAMP WITH TIME ZONE from JDBC 4.2: use `getObject(n, OffsetDateTime.class)`,
then convert to `Instant` or `ZonedDateTime`. Hibernate 5.x maps `Instant`
to `TIMESTAMP` (as UTC), which is correct. Older Hibernate versions
needed `@Type(type="instant")` or custom converters.

---

**Q7 (java.util.Date migration): How do you migrate from java.util.Date
and Calendar to java.time?**

A:
```java
// java.util.Date -> java.time:
Date d = new Date();
Instant instant = d.toInstant();                  // precise conversion
LocalDate localDate = instant.atZone(ZoneId.systemDefault())
    .toLocalDate();                               // requires timezone

// Calendar -> java.time:
Calendar cal = Calendar.getInstance();
ZonedDateTime zdt = cal.toInstant()
    .atZone(cal.getTimeZone().toZoneId());
LocalDateTime ldt = zdt.toLocalDateTime();

// java.time -> java.util.Date (for legacy APIs):
Date fromInstant = Date.from(instant);
Date fromLocal = Date.from(localDate.atStartOfDay()
    .atZone(ZoneId.systemDefault()).toInstant());

// SimpleDateFormat -> DateTimeFormatter:
// OLD: new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
// NEW:
DateTimeFormatter newFmt =
    DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
// Parsing:
LocalDateTime ldt = LocalDateTime.parse(str, newFmt);
// Formatting:
String s = ldt.format(newFmt);

// Strategy for legacy code:
// 1. Convert to java.time at system boundaries (IO, DB, REST)
// 2. Process internally with java.time
// 3. Convert back to legacy types at output boundaries if needed
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The migration strategy is architectural:
don't mix `Date` and `java.time` in the same domain model. Define conversion
utilities at integration boundaries (DAO layer, REST controllers). Spring's
`@DateTimeFormat` and Jackson's `JavaTimeModule` (for JSON) automate
the conversion for web layer. For JDBC: ensure your driver supports JDBC 4.2
(all major drivers since 2014). The remaining `Date` usage after migration:
only in legacy library APIs that you can't change (`java.util.Timer`,
some old JMX APIs).

---

**Q8 (TemporalAdjusters): What is TemporalAdjusters and when do you use it?**

A: `TemporalAdjusters` provides factory methods for common date adjustments:
finding specific days (next Monday, last day of month, first business day).

```java
LocalDate today = LocalDate.now();

// Built-in adjusters:
LocalDate lastDayOfMonth = today.with(TemporalAdjusters.lastDayOfMonth());
LocalDate firstDayOfNextMonth = today.with(TemporalAdjusters.firstDayOfNextMonth());
LocalDate firstMondayOfMonth = today.with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY));
LocalDate lastFridayOfMonth = today.with(TemporalAdjusters.lastInMonth(DayOfWeek.FRIDAY));
LocalDate nextMonday = today.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
LocalDate nextOrSameMonday = today.with(TemporalAdjusters.nextOrSame(DayOfWeek.MONDAY));

// Custom adjuster (lambda):
TemporalAdjuster nextBusinessDay = temporal -> {
    LocalDate date = LocalDate.from(temporal);
    DayOfWeek dow = date.getDayOfWeek();
    int daysToAdd = switch (dow) {
        case FRIDAY -> 3;
        case SATURDAY -> 2;
        default -> 1;
    };
    return date.plusDays(daysToAdd);
};
LocalDate nextBD = today.with(nextBusinessDay);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `TemporalAdjusters` enables a clean,
composable way to express business date logic without raw date arithmetic.
The alternative (manual date loop checking day of week) is error-prone.
For proper business day calculations with holidays: use a library like
`jollyday` (reads holiday calendars from XML) or your company's own
business calendar service. `TemporalAdjusters` handles weekday logic;
public holidays require data.

---

**Q9 (Calendar vs java.time): Why was Calendar replaced by java.time?**

A: `java.util.Calendar` was problematic in multiple ways:

1. **Mutable:** shared Calendar instances cause thread-safety bugs
2. **Confusing month numbering:** months are 0-based (JANUARY=0)
3. **Mixed concerns:** Calendar handles both parsing AND date arithmetic
4. **Inconsistent API:** `get(Calendar.MONTH)` vs `get(Calendar.DAY_OF_MONTH)`
5. **No date-only type:** Calendar always includes time (set time to midnight as workaround)
6. **Timezone handling:** `TimeZone` class with confusing DST support

```java
// Calendar's famous month bug:
Calendar cal = Calendar.getInstance();
cal.set(2024, 0, 15); // January! 0 = January, 11 = December
// java.time:
LocalDate date = LocalDate.of(2024, 1, 15); // 1 = January - intuitive!
LocalDate date = LocalDate.of(2024, Month.JANUARY, 15); // even better

// Calendar mutation bug:
Calendar cal = Calendar.getInstance();
processDate(cal); // might change cal's date!
cal.get(Calendar.DAY_OF_MONTH); // might return modified value

// java.time immutability:
LocalDate date = LocalDate.now();
LocalDate result = processDate(date); // original date unchanged
// processDate cannot modify date (immutable)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY ice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* `java.time` was designed by the same
team that wrote Joda-Time (the de facto replacement for Calendar before
Java 8). It incorporates lessons from 10+ years of Joda-Time usage. The
key design decisions: immutability (thread safety), separate types for
date vs time vs datetime vs timezone-aware datetime, and human-readable
month/day constants. The Joda-Time migration to java.time is almost
mechanical: `org.joda.time.LocalDate` -> `java.time.LocalDate`, etc.

---

### ⚖️ Comparison Table

| Type| Has Date| Has Time| Has Timezone| Use When|
|----------------|--------|--------|-------------|----------------------|
| `LocalDate`| Yes| No| No| Birthdays, deadlines|
| `LocalTime`| No| Yes| No| Business hours, alarms|
| `LocalDateTime`| Yes| Yes| No| Schedule without TZ|
| `OffsetDateTime`| Yes| Yes| Fixed offset| DB with offset TZ|
| `ZonedDateTime`| Yes| Yes| Full TZ rules| User-facing events|
| `Instant`| UTC only| UTC only| UTC| Timestamps, storage|

---

## Records, Sealed Classes, and Pattern Matching

---

### 🎯 Model Answer

**30 seconds:**
> Java 16 Records: `record Point(int x, int y) {}` - a compact syntax for
> immutable data carriers. Auto-generates: constructor, accessors, `equals`,
> `hashCode`, `toString`. Java 17 Sealed Classes: `sealed interface Shape
> permits Circle, Square, Triangle {}` - restricts which classes can extend.
> Java 21 Pattern Matching for switch: `switch (shape) { case Circle c ->
> c.radius(); case Square s -> s.side(); }` - exhaustive matching on
> type hierarchies. Together: concise algebraic data types with compile-time
> exhaustiveness.

**3 minutes (Senior):**
> Records are ideal for DTOs, value objects, immutable domain concepts.
> Compact canonical constructor for validation: `record Range(int min, int max)
> Range { if (min > max) throw new IllegalArgumentException(); } }`.
> Records can implement interfaces but cannot extend classes (implicitly
> extend Record). Can have static fields and methods, instance methods,
> but no instance state beyond components.
>
> Sealed types + records + pattern matching = algebraic data types in Java.
> Example: `sealed interface Expr permits Num, Add, Mul` with records for
> each case. Pattern matching evaluates exhaustively (compiler warns if
> case missed). This enables interpreter patterns without reflection.
> Java 21 `instanceof` pattern: `if (shape instanceof Circle c && c.radius() > 5

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Records, sealed classes, pattern matching - let me
cover records as immutable data carriers, sealed types for restricted
hierarchies, and pattern matching for exhaustive type-based switching."

**(2) First principles:** "From first principles: data transfer objects
need immutability and value-based equality. Records provide both with
minimal boilerplate. Sealed types express 'a Shape is either Circle, Square,
or Triangle - nothing else', enabling exhaustive handling in switch."

**(3) Bridge:** "Records are like named tuples with methods. Sealed types
are like enums for class hierarchies. Pattern matching switch is like
a type-aware switch/case that the compiler verifies is complete."

---


### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Records vs POJOs | 2 minutes |
| Sealed classes purpose | 2 minutes |
| Pattern matching exhaustiveness | 2 minutes |
| Records with validation | 90 seconds |
| Records limitations | 2 minutes |
| Algebraic data types | 2-3 minutes |
| Records vs Lombok | 2 minutes |
| Deconstruction patterns | 2 minutes |
| Switch expressions | 2 minutes |

---

**Q1 (Records vs POJOs): When do you use records vs regular classes?**

A: **Use records when:**
- The class is primarily a transparent data carrier
- Immutability is desired (value semantics)
- Equality should be based on all field values (component equality)
- Boilerplate reduction is valuable (DTOs, value objects, return types)

**Use regular classes when:**
- Mutable state is required (JPA entities, builder state)
- Extending another class is required (records implicitly extend Record)
- Hiding fields from equals/hashCode (password, internal cache)
- Complex construction logic beyond validation (factories, builders)
- Framework requirements (JPA, Jackson with no-arg constructor)

```java
// Record: perfect for coordinate
record GeoCoord(double lat, double lon) {}

// Record: HTTP response body (immutable, equality by content)
record ApiResponse<T>(int status, T data, String message) {}

// Regular class: JPA entity (mutable, no-arg constructor, partial equality)
@Entity
class Product {
    @Id Long id; // equality by id only, not all fields
    String name;
    BigDecimal price;
}

// Records work with Jackson 2.12+ without extra config:
// @JsonProperty auto-maps component names
record UserDTO(String name, String email) {}
// ObjectMapper.readValue(json, UserDTO.class) works!
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using generic type. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Records changed the "standard" for small
data classes. Before records: Lombok `@Value` (immutable) or `@Data` (mutable).
After records: use records for immutable data, plain classes with Lombok
`@Data` for mutable. The key difference from Lombok `@Value`: records
are a JVM concept (bytecode has RECORD attribute), not just boilerplate
generation. Reflection can inspect component names. Jackson's `RecordNamingStrategy`
knows about record components. IDE and tool support is first-class.

---

**Q2 (Sealed classes purpose): What problem do sealed classes solve?**

A: Sealed classes solve the "open world problem" for class hierarchies.
Without sealed: anyone can subclass your type, making exhaustive handling impossible.

```java
// Without sealed: unknown subtypes
interface Shape { double area(); }
// Caller:
double describe(Shape s) {
    if (s instanceof Circle c) return c.area();
    if (s instanceof Square sq) return sq.area();
    // Must have "else" - someone could add Triangle, Pentagon, etc.
    else throw new IllegalArgumentException("Unknown shape: " + s);
    // Runtime error for new subtypes - not caught at compile time!
}

// With sealed: exhaustive, compiler-verified
sealed interface Shape permits Circle, Square, Triangle {}
double describe(Shape s) {
    return switch (s) {
        case Circle c  -> c.area();
        case Square sq -> sq.area();
        case Triangle t -> t.area();
        // NO DEFAULT needed - compiler knows all cases!
        // Adding Square2 to Shape without handling it -> COMPILE ERROR
    };
}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Sealed types are the Java equivalent of
Rust's enums with data, Haskell's ADTs, or Kotlin's sealed classes. They
enable "close the world" reasoning: you can prove that your switch handles
ALL possible cases. This matters for: error types (a `Result` is either
`Success` or `Failure`), state machines, AST nodes in compilers/interpreters,
protocol message types. The alternative before sealed: checked exceptions
(declare all failure types), but checked exceptions are for method signatures,
not type hierarchies.

---

**Q3 (Pattern matching exhaustiveness): How does the compiler verify
exhaustiveness in switch expressions?**

A: For a sealed type, the compiler knows all permitted subtypes at compile time.
A switch expression (not statement) must cover all cases or have a default.

```java
sealed interface Status permits Active, Inactive, Suspended {}
record Active() implements Status {}
record Inactive() implements Status {}
record Suspended(String reason) implements Status {}

// Exhaustive switch expression (no default needed):
String describe(Status s) {
    return switch (s) {
        case Active a    -> "Active user";
        case Inactive i  -> "Inactive user";
        case Suspended su -> "Suspended: " + su.reason();
    };
}
// If you remove the Suspended case: COMPILE ERROR
// "the switch expression does not cover all possible input values"

// Adding a new permitted type forces ALL switch expressions to handle it:
// sealed interface Status permits Active, Inactive, Suspended, PendingVerification {}
// -> All switches on Status fail to compile until PendingVerification is added!
// This is the compile-time safety guarantee.

// Switch statement (NOT expression) does NOT require exhaustiveness:
switch (s) {
    case Active a    -> System.out.println("active");
    // case Inactive i -> ... // missing case: compile WARNING, not error
} // statements can silently ignore cases - prefer expressions!
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* The exhaustiveness checking is only
guaranteed for SWITCH EXPRESSIONS (not statements) and only when the
switched type is sealed. For non-sealed types: use `default` case or
accept the "missing cases" warning. The architectural value: when you
add a new domain concept (new `Status` type), the compiler forces all
processing code to acknowledge it. You can't accidentally miss a case
in a serializer, deserializer, or state machine handler.

---

**Q4 (Records with validation): How do you add validation to records?**

A:
```java
// Compact canonical constructor: runs after components are assigned
// Components are implicitly available (no param list needed)
record EmailAddress(String value) {
    EmailAddress { // compact constructor: no "(String value)" needed
        Objects.requireNonNull(value, "email required");
        if (!value.matches("[^@]+@[^@]+\\.[^@]+")) {
            throw new IllegalArgumentException(
                "Invalid email: " + value);
        }
        value = value.toLowerCase().strip(); // normalize (reassigns param)
    }
    // After compact constructor: this.value is normalized
}

// Explicit canonical constructor (more verbose but clearer):
record Range(int min, int max) {
    Range(int min, int max) {
        if (min > max) throw new IllegalArgumentException(
            "min " + min + " > max " + max);
        this.min = min;
        this.max = max;
    }
}

// Defensive copy for mutable components (rare in records - prefer immutable):
record Snapshot(List<String> items) {
    Snapshot {
        items = List.copyOf(items); // defensive copy + unmodifiable
    }
    // Accessor returns the unmodifiable copy:
    // List<String> items() { return items; } // auto-generated
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The compact canonical constructor is
the preferred form: it's concise and makes the intent clear (validation/
normalization only). The `value = normalized(value)` assignment in the
compact constructor looks like it assigns a field, but it assigns the
PARAMETER - the framework then assigns the parameter to the final field.
This is the only place in Java where you can reassign a parameter that
"becomes" a final field. Understanding this mechanism is important for
correct validation code.

---

**Q5 (Records limitations): What can't you do with records?**

A:
1. **Cannot extend classes** (records implicitly extend `java.lang.Record`)
2. **Cannot declare instance variables** beyond components (only static)
3. **Cannot have mutable state** (all components are final)
4. **Cannot use as JPA entities** (no no-arg constructor, immutable)
5. **Cannot implement @FunctionalInterface** if it declares abstract methods beyond what records provide

```java
// Cannot extend a class:
record Point3D(int x, int y, int z) extends Point {} // COMPILE ERROR
// Can only extend java.lang.Record (implicit)

// Cannot add instance variables:
record Pair<A, B>(A first, B second) {
    private int count; // COMPILE ERROR: only static fields allowed!
    private static int instanceCount = 0; // OK: static
}

// CAN implement interfaces:
record Point(int x, int y) implements Comparable<Point> {
    @Override
    public int compareTo(Point other) {
        int xCmp = Integer.compare(x, other.x);
        return xCmp != 0 ? xCmp : Integer.compare(y, other.y);
    }
}

// CAN override component accessor:
record Name(String value) {
    public String value() { // override auto-generated accessor
        return value.strip(); // normalize on access
    }
}

// CAN have static factory methods:
record Color(int r, int g, int b) {
    static final Color RED = new Color(255, 0, 0);
    static Color of(String hex) {
        // parse hex color string
        return new Color(/* ... */);
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Records being unable to extend other
classes is by design: they ARE the "product type" (all components, all
the time). Subclassing would allow adding components in a non-records way,
breaking the value semantics. If you need inheritance: use sealed interfaces
with records implementing them (as shown in the Expr example). The
interface approach gives you the polymorphism of inheritance without the
fragile base class problem.

---

**Q6 (Algebraic data types): Explain algebraic data types in Java.**

A: An ADT combines two primitives:
- **Sum type** ("or"): A value is ONE of several options (Circle OR Square OR Triangle)
- **Product type** ("and"): A value has ALL of its parts (Circle has a radius AND nothing else)

```java
// Sum type: sealed interface (A is B or C or D)
sealed interface Result<T> permits Success, Failure {}

// Product types: records (the concrete variants)
record Success<T>(T value) implements Result<T> {}
record Failure<T>(String error, Exception cause) implements Result<T> {}

// Pattern matching handles the sum type exhaustively:
<T, R> R fold(Result<T> result,
              Function<T, R> onSuccess,
              Function<String, R> onFailure) {
    return switch (result) {
        case Success<T> s -> onSuccess.apply(s.value());
        case Failure<T> f -> onFailure.apply(f.error());
    };
}

// Usage: type-safe error handling without exceptions:
Result<User> findUser(Long id) {
    try {
        return new Success<>(repo.findById(id));
    } catch (EntityNotFoundException e) {
        return new Failure<>("User not found: " + id, e);
    }
}

String display = fold(findUser(42L),
    user -> "Found: " + user.getName(),
    error -> "Error: " + error);
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using error handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* ADTs with sealed types and pattern
matching represent a fundamental shift in how Java handles heterogeneous
data. The `Result<T>` type shown is equivalent to Kotlin's `Result<T>`,
Rust's `Result<T, E>`, or Scala's `Either[A, B]`. It makes the "this can
fail" case explicit in the return type, forcing callers to handle both
success and failure. No more "returns null on failure" or "throws checked
exception" (which can be ignored). The compile-time exhaustiveness check
means: adding a third case (`Partial<T>`) forces all callers to handle it.

---

**Q7 (Records vs Lombok): When do you use records vs Lombok?**

A:

| Aspect | Records (Java 16+) | Lombok `@Value` |
|---|---|---|
| Immutability | Built-in (components final) | Via `@Value` |
| Extends classes | No (extends Record) | Yes |
| JVM concept | Yes (RECORD attribute) | No (compile-time codegen) |
| Deconstruction patterns | Yes (Java 21) | No |
| Sealed type compatibility | Yes | Limited |
| Build tool dependency | None | Lombok jar required |
| IDE support | Native | Plugin needed |
| Customization | Compact constructor | Lombok annotations |

**Decision:**
- New code, Java 16+: prefer records for immutable data carriers
- Need to extend a class: Lombok `@Data` or manual
- JPA entities: Lombok `@Data` or manual (not records)
- Complex validation/customization: compact constructor or Lombok `@Builder`

*What separates good from great:* Records and Lombok serve different needs.
Records are a language feature with JVM support - reflection, pattern matching,
and future features will first-class support records. Lombok is annotation
processing - works at compile time but tools need to understand it. For
new codebases on Java 17+: use records for data classes, eliminate Lombok
for this use case. For existing codebases with heavy Lombok usage: migrate
incrementally (Lombok `@Value` -> records is usually mechanical). Keep
Lombok `@Builder` for complex construction scenarios not covered by records.

---

**Q8 (Deconstruction patterns): How does record deconstruction work in
pattern matching?**

A: Record deconstruction (Java 21) allows binding record components
directly in case labels:

```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}

void process(Object obj) {
    switch (obj) {
        // Deconstruct record:
        case Point(int x, int y) ->
            System.out.printf("Point at (%d, %d)%n", x, y);

        // Nested deconstruction:
        case Line(Point(int x1, int y1), Point(int x2, int y2)) ->
            System.out.printf("Line from (%d,%d) to (%d,%d)%n",
                x1, y1, x2, y2);

        // Guard (when clause):
        case Point(int x, int y) when x == y ->
            System.out.println("On diagonal: " + x);

        default -> System.out.println("Unknown: " + obj);
    }
}
// Note: when two Point cases: the specific (diagonal) guard must come first!

// For List (Java 21 preview):
// case List<Integer> list when list.isEmpty() -> ...
// Matching on list contents requires external iteration (no built-in list deconstruction yet)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Deconstruction patterns eliminate the
common pattern of matching a record type then immediately extracting
components: `case Point p -> { int x = p.x(); int y = p.y(); }`.
With deconstruction: `case Point(int x, int y)` binds x and y directly.
Nested deconstruction for data trees is particularly powerful - compare
with the Expr evaluator example. The `when` guard (`when x == y`) adds
conditions beyond type matching. This is Java's approach to what Scala
and Haskell call "pattern matching" - the fundamental tool for structural
decomposition of algebraic data.

---

**Q9 (Switch expressions): How do switch expressions differ from
switch statements?**

A:

| Aspect | Switch Statement | Switch Expression |
|---|---|---|
| Returns value | No | Yes (entire switch yields a value) |
| Fall-through | Yes (needs `break`) | No (each branch is independent) |
| Exhaustiveness | Not required | Required (compiler error if incomplete) |
| Arrow form | Traditional with break | `case X -> expr` (no fall-through) |
| Colon form | `case X:` with break | `case X: yield value;` with `yield` |

```java
int numLetters = switch (day) { // switch EXPRESSION
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
    // ALL DayOfWeek values covered -> no default needed!
};

// With blocks (arrow form):
String category = switch (score) {
    case int s when s >= 90 -> "A";
    case int s when s >= 80 -> "B";
    case int s when s >= 70 -> "C";
    default -> "F";
};

// With yield (colon form for multi-statement blocks):
String result = switch (status) {
    case ACTIVE: {
        auditLog("Access");
        yield "active";      // use yield, not return!
    }
    case INACTIVE -> "inactive"; // arrow form: no yield needed
    default -> "unknown";
};
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Switch expressions (Java 14, standard)
are always preferred over switch statements for assignments - the compiler
enforces exhaustiveness and eliminates fall-through bugs (accidental omission
of `break` between cases is one of Java's most common bugs). The arrow form
(`->`) is cleaner for simple cases. The colon form (`case X: ... yield v;`)
is needed when the case requires multiple statements but should produce
a value. `yield` in a switch expression is analogous to `return` but
specific to switch blocks.

---

### ⚖️ Comparison Table

| Feature | Records | Sealed Classes | Pattern Matching |
|---|---|---|---|
| Java version | 16 | 17 | 16 (instanceof), 21 (switch) |
| Purpose | Immutable data carrier | Closed type hierarchy | Exhaustive type dispatch |
| Boilerplate reduction | High | Medium | Medium |
| Compiler enforcement | Component equality | Exhaustive permits | Exhaustive cases |
| Serialization | JSON: auto; Java: manual | N/A | N/A |
| JPA compatibility | No (no-arg ctor) | Yes | N/A |

---


---

# Java Core - L3 IO and NIO

## Java IO Streams

---

### 🎯 Model Answer

**30 seconds:**
> Java IO (`java.io`) provides stream-based I/O: byte streams
> (`InputStream`/`OutputStream`) and character streams (`Reader`/`Writer`).
> Streams are unidirectional and sequential. The decorator pattern layers
> functionality: `new BufferedReader(new FileReader(path))` adds buffering
> to unbuffered file reading. Character streams handle encoding conversion
> (between bytes and characters). Always close streams in `finally` or
> via `try-with-resources` to release file handles.

**3 minutes (Senior):**
> The IO class hierarchy is large but follows the decorator pattern:
> base classes (`FileInputStream`, `FileReader`) + decorator wrappers
> (`BufferedInputStream`, `DataInputStream`, `ObjectInputStream`).
> `BufferedInputStream` reads 8KB at a time from disk instead of one
> byte; dramatically improves performance.
>
> Character streams vs byte streams: `FileReader` reads characters using
> the platform's default encoding - a trap on systems with non-UTF-8
> defaults. Always specify encoding explicitly:
> `new InputStreamReader(fis, StandardCharsets.UTF_8)`.
>
> `ObjectInputStream`/`ObjectOutputStream` provide Java serialization.
> Deserializing untrusted data is a critical security vulnerability -
> arbitrary code execution via gadget chains. Never deserialize data
> from untrusted sources without validation. Use `ObjectInputFilter` (Java 9+)
> to restrict deserializable classes.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Java IO streams - let me cover the byte/character
stream hierarchy, the decorator pattern for layering, encoding pitfalls,
buffering importance, and try-with-resources."

**(2) First principles:** "From first principles: file data is bytes.
Reading one byte at a time from disk is slow (each system call has
overhead). Buffering batches reads into larger chunks. Character streams
abstract the byte-to-character conversion, letting you work with text
regardless of the underlying encoding."

**(3) Bridge:** "IO streams are like postal mail: you send and receive
one letter at a time (unbuffered) or batch them into packages (buffered).
Character streams are like translated mail - the bytes are letters in
foreign language; the character stream translates them for you."

---


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Byte vs character streams | 2 minutes |
| Decorator pattern in IO | 2 minutes |
| try-with-resources semantics | 2 minutes |
| Encoding pitfalls | 2 minutes |
| Buffering and performance | 2 minutes |
| IO vs NIO for files | 2 minutes |
| Serialization security | 2-3 minutes |
| Stream closing order | 2 minutes |
| Large file processing | 2 minutes |

---

**Q1 (Byte vs character streams): What is the difference between byte
streams and character streams?**

A: **Byte streams** (`InputStream`/`OutputStream`): deal in raw bytes (0-255).
No encoding awareness. Work for any binary data: images, audio, serialized objects.

**Character streams** (`Reader`/`Writer`): deal in Java `char` values (UTF-16).
Built on top of byte streams via `InputStreamReader`/`OutputStreamWriter`.
Handle encoding conversion (bytes <-> chars). Work for text data.

```java
// Byte stream: reads raw bytes
try (InputStream is = new FileInputStream("image.jpg")) {
    byte[] buffer = new byte[8192];
    int bytesRead;
    while ((bytesRead = is.read(buffer)) != -1) {
        process(buffer, 0, bytesRead); // process raw bytes
    }
}

// Character stream: reads decoded text
try (Reader r = new InputStreamReader(
        new FileInputStream("text.txt"), UTF_8)) {
    char[] buffer = new char[8192];
    int charsRead;
    while ((charsRead = r.read(buffer)) != -1) {
        process(new String(buffer, 0, charsRead)); // process text
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

Rule: use byte streams for binary data. Use character streams (with
explicit encoding) for text data.

*What separates good from great:* Java `char` is UTF-16 internally.
`InputStreamReader` converts from external encoding (e.g., UTF-8) to
Java's internal UTF-16. `OutputStreamWriter` converts back. For files
with multi-byte characters (emoji, CJK characters), be aware: a single
Unicode code point may be one `char` (BMP) or two `char` values (surrogate
pair, above U+FFFF). Java's `String.length()` returns char count (UTF-16
units), not code point count. `"emoji".codePointCount(0, s.length())`
gives actual Unicode character count.

---

**Q2 (Decorator pattern in IO): How does the decorator pattern apply
to Java IO?**

A: The decorator pattern wraps an object to add functionality while
preserving the same interface.

```java
// All decorators take a stream in their constructor:
InputStream base = new FileInputStream("data.bin"); // component

// Decorator: adds buffering
InputStream buffered = new BufferedInputStream(base);

// Decorator: adds ability to read Java primitives
DataInputStream data = new DataInputStream(buffered);

// Decorator chain: FileInputStream -> BufferedInputStream -> DataInputStream
// Each layer adds functionality without knowing about others

// The same with OutputStreams:
OutputStream os = new FileOutputStream("out.bin");
os = new BufferedOutputStream(os);        // add buffering
os = new DataOutputStream(os);           // add typed writes (primitives)
os = new GZIPOutputStream(os);           // add GZIP compression

// Writing to a compressed file:
try (DataOutputStream dos = new DataOutputStream(
        new GZIPOutputStream(
            new BufferedOutputStream(new FileOutputStream("data.gz"))))) {
    dos.writeInt(42);
    dos.writeUTF("compressed data!");
}
// Layers: FileOutputStream (base) -> BufferedOutputStream (buffering)
//      -> GZIPOutputStream (compression) -> DataOutputStream (typed write)
// Data flows: int -> DataOutputStream serializes -> GZIPOutputStream compresses
//           -> BufferedOutputStream batches -> FileOutputStream writes
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The decorator pattern in IO allows
arbitrary composition of behaviors without combinatorial explosion of
subclasses. If you had N functionalities (buffering, compression, encryption,
data types) and M stream types (file, network, memory): without decorator
you'd need N*M subclasses. With decorator: N+M classes, combine at
construction time. This is why `new GZIPOutputStream(new BufferedOutputStream(socket.getOutputStream()))` works perfectly - each layer is unaware
of the others. The cost: deeply nested constructors are verbose; the NIO
Files API wraps this complexity into simpler factory methods.

---

**Q3 (try-with-resources semantics): Explain how try-with-resources
handles exceptions.**

A:
```java
// Syntax:
try (ResourceA a = ...; ResourceB b = ...) {
    // use a and b
} catch (Exception e) {
    // handle
}
// Resources closed in reverse order: b first, then a
// Even if the try block throws OR if previous close() throws

// Exception handling with multiple exceptions:
try (AutoCloseable outer = new Outer();  // closed second
     AutoCloseable inner = new Inner()) { // closed first
    throw new RuntimeException("from body");
} catch (Exception e) {
    // e is "from body" (the primary exception)
    // If outer.close() ALSO throws:
    //   e.getSuppressed()[0] = exception from outer.close()
    //   e.getSuppressed()[1] = exception from inner.close()
    // Primary exception always takes precedence!
}

// Without try-with-resources (old way - bug: exception from finally
// suppresses exception from try):
InputStream is = new FileInputStream("f");
try {
    throw new IOException("read failed");
} finally {
    is.close(); // if this ALSO throws IOException:
                // the original "read failed" is SWALLOWED!
    // The finally exception replaces the try exception!
}
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using error handling. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The suppressed exception mechanism
(introduced with try-with-resources in Java 7) is a correct solution
to a pre-existing bug. Before Java 7: a `finally` block that throws
replaces the original exception - the original failure is lost, making
debugging impossible. With try-with-resources: the primary exception
survives, and any `close()` exceptions are attached as suppressed.
When debugging failures in code using resources: always check
`e.getSuppressed()` in the stack trace - these are often clues to
cascading failures.

---

**Q4 (Encoding pitfalls): What are the encoding pitfalls in Java IO?**

A:

**Pitfall 1: Platform-default encoding**
```java
// WRONG: FileReader uses platform default encoding
Reader r = new FileReader("config.properties");
// On Windows with Windows-1252 default: reads bytes as Windows-1252
// On Linux with UTF-8 default: reads as UTF-8
// Same file, different results!

// CORRECT:
Reader r = new InputStreamReader(
    new FileInputStream("config.properties"),
    StandardCharsets.UTF_8); // explicit encoding
// Or:
Reader r = Files.newBufferedReader(
    Path.of("config.properties"), StandardCharsets.UTF_8);
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

**Pitfall 2: PrintStream encoding (System.out)**
```java
// System.out is a PrintStream with platform encoding
System.out.println("日本語"); // may display as "???" on non-UTF-8 terminals
// In production servers (Linux/UTF-8): usually OK
// In CI/CD pipelines or Windows: may corrupt
// Fix for tests:
PrintStream out = new PrintStream(System.out, true, StandardCharsets.UTF_8);
// Or: set JVM default: -Dfile.encoding=UTF-8 (Java 17)
//     or: -Dstdout.encoding=UTF-8 (Java 18+)
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

**Pitfall 3: BOM in UTF-8 files**
```java
// Some files (especially from Windows Notepad) start with UTF-8 BOM:
// EF BB BF (the BOM bytes)
// BufferedReader.readLine() does NOT strip BOM!
String firstLine = br.readLine();
if (firstLine.startsWith("\uFEFF")) { // BOM as Unicode
    firstLine = firstLine.substring(1);
}
// Or use Apache Commons IO's BOMInputStream
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The file.encoding system property was
the conventional way to set default encoding but was "soft" in older Java
(not always respected). Java 17 introduced `java.nio.charset.Charset.defaultCharset()`
being settable via system property. Java 18 made UTF-8 the default on all
platforms (`-Dfile.encoding=UTF-8` no longer needed on most JDKs). This
was a major portability improvement. In modern Java (18+): `FileReader`
uses UTF-8 by default on all platforms. But for explicit, portable code:
always specify encoding regardless of Java version.

---

**Q5 (Buffering and performance): Why is buffering critical for IO performance?**

A:
```java
// Measuring unbuffered vs buffered read:
// Unbuffered: one system call per byte
try (InputStream is = new FileInputStream("large.bin")) {
    int b;
    while ((b = is.read()) != -1) { // system call for each byte!
        process((byte) b);
    }
    // For a 1MB file: ~1,000,000 system calls
    // Typical system call: ~1-2 microseconds
    // Total: ~1-2 seconds just in system call overhead!
}

// Buffered: one system call per 8KB
try (InputStream is = new BufferedInputStream(
        new FileInputStream("large.bin"))) {
    int b;
    while ((b = is.read()) != -1) { // reads from buffer; OS call every 8KB
        process((byte) b);
    }
    // For a 1MB file: ~128 system calls
    // Total: <1 millisecond in system call overhead
}

// Block read: most efficient
try (InputStream is = new BufferedInputStream(
        new FileInputStream("large.bin"))) {
    byte[] buffer = new byte[8192];
    int bytesRead;
    while ((bytesRead = is.read(buffer)) != -1) {
        process(buffer, 0, bytesRead); // process a block at a time
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* Buffering is the single most impactful
IO optimization. The `read()` method on `BufferedInputStream` reads from
an in-memory buffer (refilled when empty by reading 8192 bytes from the
OS in one system call). The performance difference: on Linux, `read(2)`
system call for 1 byte vs 8192 bytes is roughly the same cost (~1µs).
Reading a 1MB file: 1 call vs 1M calls = 1000x difference. NIO
`FileChannel.read(ByteBuffer)` can be even faster: direct byte buffers
avoid a copy between kernel and user space.

---

**Q6 (IO vs NIO for files): When do you use IO vs NIO for file operations?**

A:

| Use Case | IO API | NIO (Files) API |
|---|---|---|
| Read entire small file | `FileReader` + loop | `Files.readAllLines()` or `Files.readString()` |
| Read large file line by line | `BufferedReader` + loop | `Files.lines()` (lazy stream) |
| Write text | `PrintWriter` / `BufferedWriter` | `Files.write()` or `Files.writeString()` |
| Copy files | Manual read-write loop | `Files.copy()` |
| Move / rename | `File.renameTo()` (unreliable) | `Files.move()` (atomic on same filesystem) |
| Check if file exists | `new File(path).exists()` | `Files.exists(path)` |
| File metadata | `File.length()`, `lastModified()` | `Files.size()`, `Files.getLastModifiedTime()` |
| Walk directory | `File.listFiles()` + recursion | `Files.walk()` or `Files.walkFileTree()` |
| Binary file processing | IO streams | NIO `FileChannel` + `ByteBuffer` |

```java
// Modern NIO for common operations:
// Read entire file (small files only):
String content = Files.readString(Path.of("config.txt"), UTF_8);

// Write file (overwrites; use APPEND for append):
Files.writeString(Path.of("out.txt"), content, UTF_8,
    StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING);

// Copy:
Files.copy(source, dest, StandardCopyOption.REPLACE_EXISTING);

// Walk directory:
Files.walk(Path.of("src"), 10) // max depth 10
    .filter(p -> p.toString().endsWith(".java"))
    .forEach(System.out::println);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `Files.move()` with `ATOMIC_MOVE` is
the only reliable way to atomically rename or replace a file on a single
filesystem. `File.renameTo()` returns a boolean on failure (no exception),
silently fails across filesystems, and is not atomic. Production pattern:
write to a temp file, then `Files.move(temp, dest, ATOMIC_MOVE)` to avoid
readers seeing a partially-written file. This is used by databases (WAL),
configuration management (Ansible), and deployment tools.

---

**Q7 (Serialization security): Why is Java serialization a security risk?**

A: Java object deserialization executes code during the deserialization
process. If an attacker can inject a crafted byte stream:

1. `readObject()` methods of objects in the stream are called during deserialization
2. Classes already on the classpath (gadgets) may have `readObject()` that
   triggers dangerous operations
3. By chaining gadgets, attackers can achieve arbitrary code execution (RCE)

```java
// Known vulnerable deserialization:
ObjectInputStream ois = new ObjectInputStream(untrustedInput);
Object obj = ois.readObject(); // DANGEROUS if input is untrusted!

// Real exploit classes (before patches):
// - Apache Commons Collections: InvokerTransformer chain
// - Spring Framework: various gadget chains
// - JDK itself: UnicastRef, JMX gadgets

// Mitigations:
// 1. Don't deserialize untrusted input (best defense)
// 2. Java 9+ ObjectInputFilter:
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "com.myapp.*;!*"); // only allow your own classes
// 3. Java 9+ JEP 290: set JVM-wide filter:
// -Djdk.serialFilter=maxbytes=10000;maxdepth=5;!*

// Safe alternatives:
// JSON: ObjectMapper.readValue(json, MyClass.class) - no code execution
// Protobuf: generated parser with no dynamic dispatch
// Avro/Thrift: schema-driven, no arbitrary code execution
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* Java serialization's fundamental problem:
`readObject()` is a code execution point, not just data deserialization.
CVE-2015-4852 (WebLogic), CVE-2015-7501 (JBoss), and others caused billions
in security incidents. The fix requires not just patching vulnerable libraries
but architectural changes: stop accepting Java-serialized data from untrusted
sources. Modern systems: JSON over HTTP (Jackson), gRPC (Protobuf), or
message queues with schema registries (Avro) - none execute arbitrary code
during parsing. The `ObjectInputFilter` (Java 9+) is a defense-in-depth
measure, not a complete fix: an allowlist is safe; a blocklist will always
miss gadget classes.

---

**Q8 (Stream closing order): What is the correct order to close nested
IO streams?**

A: With `try-with-resources`: resources are closed in REVERSE declaration
order (innermost stream first). With manual close: close outermost first
(which propagates close to inner).

```java
// try-with-resources: reverse order (correct)
try (InputStream fis = new FileInputStream("file");         // closed last
     BufferedInputStream bis = new BufferedInputStream(fis); // closed second
     DataInputStream dis = new DataInputStream(bis)) {       // closed first
    dis.readInt();
}
// Close order: dis, bis, fis

// Manual: close outermost (propagates down):
DataInputStream dis = new DataInputStream(
    new BufferedInputStream(new FileInputStream("file")));
dis.close(); // calls BufferedInputStream.close() -> FileInputStream.close()

// RISK: in try-with-resources, if any constructor throws,
// only already-declared resources are closed:
try (InputStream fis = new FileInputStream("file"); // declared
     ObjectInputStream ois = new ObjectInputStream(fis)) { // if this throws:
    // fis is closed (it was declared)
    // ois is not open (constructor failed)
    // Correct: fis is properly closed
}

// RISK: if you don't use try-with-resources and constructor throws:
InputStream fis = new FileInputStream("file"); // opened
ObjectInputStream ois;
try {
    ois = new ObjectInputStream(fis); // might throw
} catch (IOException e) {
    fis.close(); // must manually close fis if ois construction fails!
}
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* The `try-with-resources` resource variable
scope is important: only resources declared IN the parentheses are
automatically closed. If you open a resource inside the try block (not
in the declaration), it won't be auto-closed. For chained stream construction:
if any inner constructor throws, any already-constructed outer streams
that aren't in the resource variable list need manual cleanup. This is
why `try-with-resources` with each stream separately declared is safer
than nesting all in one `new`:
```java
// Safer (each declared):
try (FileInputStream fis = new FileInputStream("f");
     InputStreamReader isr = new InputStreamReader(fis, UTF_8);
     BufferedReader br = new BufferedReader(isr)) { ... }
// vs: new BufferedReader(new InputStreamReader(new FileInputStream("f")))
// Only the outermost is in the resource variable - inner leaks on constructor failure
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

---

**Q9 (Large file processing): How do you process a file that doesn't
fit in memory?**

A:

```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// Files.readAllLines() - NOT for large files: loads ALL lines into memory
List<String> lines = Files.readAllLines(path, UTF_8); // OOM for 10GB file!

// GOOD: Files.lines() - lazy Stream, processes one line at a time:
long count;
try (Stream<String> stream = Files.lines(path, UTF_8)) { // lazy
    count = stream
        .filter(line -> line.contains("ERROR"))
        .count(); // only "ERROR" lines are processed per line
} // stream and underlying reader closed

// For very large binary files: NIO FileChannel with direct ByteBuffer
try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024); // 1MB direct
    while (channel.read(buffer) > 0) {
        buffer.flip(); // prepare for reading
        while (buffer.hasRemaining()) {
            process(buffer.get()); // process byte
        }
        buffer.clear(); // prepare for next read
    }
}

// Memory-mapped files: OS maps file into virtual address space
try (FileChannel channel = FileChannel.open(path)) {
    MappedByteBuffer mmap = channel.map(
        FileChannel.MapMode.READ_ONLY, 0, channel.size());
    // Entire file accessible as a ByteBuffer
    // OS pages in data on demand - no explicit read loop
    // Fastest for random access patterns on large files
}
```

> **Code walkthrough:** GOOD pattern: This Unknown example demonstrates Java Strice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**

*What separates good from great:* `Files.lines()` is the right tool for
large text files: it buffers reads but doesn't hold all lines in memory.
The `Stream` must be closed (use try-with-resources) to release the
underlying file handle. Memory-mapped files (`MappedByteBuffer`) are
the fastest for read-heavy, random-access patterns on large files:
the OS handles page-in/page-out transparently. Used by: JVM class loading,
database engines (SQLite, H2), search engines (Lucene). The downside:
`MappedByteBuffer` is hard to unmap explicitly in Java (a known limitation;
requires reflection or closing the `FileChannel` and waiting for GC).

---

### ⚖️ Comparison Table

| Aspect| IO (java.io)| NIO (java.nio)| NIO2 (Files API)|
|---|-----------------|---------------------------------|----------------------|
| Paradigm| Stream (one byte/char at a time)| Channel + Buffer (block-oriented)|
| Blocking| Blocking| Non-blocking available| Blocking (convenience)|
| Performance| Good with buffering| Best for large files| Good|
| Ease of use| Moderate| Complex| Simplest|
| API style| Imperative| Low-level| Modern/functional|
| Best for| Text processing, protocols| High-throughput binary| File management|

---


## NIO Files API

---

### 🎯 Model Answer

**30 seconds:**
> `java.nio.file.Files` and `java.nio.file.Path` (NIO2, Java 7) are
> the modern Java file system API. Key operations: `Files.readString()`,
> `Files.writeString()`, `Files.copy()`, `Files.move()`, `Files.delete()`,
> `Files.exists()`, `Files.walk()`, `Files.lines()`. `Path` replaces
> the legacy `java.io.File`. `Paths.get()` or `Path.of()` (Java 11)
> create Paths. NIO2 provides atomic operations (`ATOMIC_MOVE`), proper
> exceptions (not just boolean returns), and tree walking. `WatchService`
> monitors file system changes.

**3 minutes (Senior):**
> Key improvements over `java.io.File`: (1) Proper exception handling -
> `File.delete()` returns boolean; `Files.delete()` throws `IOException`
> with reason. (2) Atomic operations - `Files.move(src, dest, ATOMIC_MOVE)`
> is atomic on POSIX filesystems. (3) Symbolic links - `Files.readSymbolicLink()
> link following options. (4) Metadata - `Files.readAttributes()`, `BasicFileAtt
> (5) Directory traversal - `Files.walk()` (depth-first), `Files.walkFileTree()`
>
> `WatchService`: registers a path to receive events (`ENTRY_CREATE`,
> `ENTRY_MODIFY`, `ENTRY_DELETE`). Uses OS-level notifications (inotify
> on Linux, FSEvents on macOS, ReadDirectoryChangesW on Windows) - not
> polling. Used by: Spring Boot DevTools (auto-restart), Webpack dev server,
> configuration reload systems.
>
> Path operations: `path.resolve()` (append), `path.relativize()` (compute
> relative path), `path.normalize()` (remove `.`, `..`), `path.toAbsolutePath()`
> `path.getParent()`, `path.getFileName()`.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "NIO Files API - let me cover Path operations, the Files
utility class, WatchService for file system events, and the improvements
over java.io.File."

**(2) First principles:** "From first principles: file system operations
need: reliable error reporting (not just booleans), atomic operations
to avoid partial writes, and notification-based change detection (not
polling). NIO2 provides all three."

**(3) Bridge:** "NIO2 is the 'grown-up' file API. `java.io.File` is
like a pocket knife - convenient but limited, with silent failures.
NIO2 is the full toolbox: proper errors, atomic operations, notifications,
and rich metadata access."

---


### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Path vs File comparison | 2 minutes |
| Atomic file write pattern | 2 minutes |
| WatchService internals | 2 minutes |
| Files.walk resource management | 2 minutes |
| Copy options | 2 minutes |
| Directory traversal options | 2 minutes |
| Temp files and paths | 90 seconds |
| File attributes | 2 minutes |
| Symbolic links handling | 2 minutes |

---

**Q1 (Path vs File comparison): What are the improvements of Path/Files
over java.io.File?**

A:

| Aspect | java.io.File | java.nio.file.Path + Files |
|---|---|---|
| Error reporting | boolean return (delete, mkdir) | Throws IOException with detail |
| Atomic operations | None | `Files.move(ATOMIC_MOVE)` |
| Symbolic links | Partial support | Full support, follow/no-follow options |
| File attributes | `length()`, `lastModified()` | `BasicFileAttributes`, extended attrs |
| Directory listing | `listFiles()` (loads all into array) | `Files.list()` (lazy stream) |
| Tree walking | Manual recursion | `Files.walk()`, `Files.walkFileTree()` |
| Change notification | Polling only | `WatchService` (OS events) |
| Interoperability | Standard | Integrates with NIO channels |

```java
// File: silent failure
File f = new File("/read-only/file");
f.delete(); // returns false - WHY did it fail?

// Files: informative exception
try {
    Files.delete(Path.of("/read-only/file"));
} catch (NoSuchFileException e) {
    log.error("File does not exist: {}", e.getFile());
} catch (AccessDeniedException e) {
    log.error("Permission denied: {}", e.getFile());
} catch (DirectoryNotEmptyException e) {
    log.error("Directory not empty: {}", e.getFile());
}

// Files.deleteIfExists: delete if present, no exception if absent:
Files.deleteIfExists(path); // throws only on failure (not if missing)
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using SQL. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The exception hierarchy under
`java.nio.file` is much richer than generic `IOException`. `NoSuchFileException`,
`AccessDeniedException`, `DirectoryNotEmptyException`,
`NotDirectoryException`, `AtomicMoveNotSupportedException` - each is
catchable separately for precise error handling. In production code:
catch specific exceptions to provide clear user-facing messages and
distinguish "retry-able" failures (temporary lock) from permanent ones
(permission denied). The boolean return from `File.delete()` was a
historical mistake that makes error handling impossible.

---

**Q2 (Atomic file write pattern): How do you atomically update a file?**

A: The pattern: write to a temp file, then atomically rename to target.

```java
/**
 * Atomically replaces target with new content.
 * Readers see old or new content, never partial writes.
 */
public static void atomicWrite(Path target, String content)
        throws IOException {
    // Create temp file in SAME directory as target (important!)
    // Same directory = same filesystem = atomic rename possible
    Path temp = Files.createTempFile(
        target.getParent(),  // same dir as target
        ".tmp-",             // prefix
        null);               // suffix (auto-generated)

    try {
        Files.writeString(temp, content, UTF_8,
            StandardOpenOption.WRITE,
            StandardOpenOption.TRUNCATE_EXISTING);

        // Attempt atomic move:
        try {
            Files.move(temp, target,
                StandardCopyOption.REPLACE_EXISTING,
                StandardCopyOption.ATOMIC_MOVE);
        } catch (AtomicMoveNotSupportedException e) {
            // Fallback: non-atomic but still replace-existing
            // (window of partial visibility exists)
            Files.move(temp, target,
                StandardCopyOption.REPLACE_EXISTING);
        }
    } catch (IOException e) {
        Files.deleteIfExists(temp); // clean up
        throw e;
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using SQL. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* `Files.createTempFile(parent, ...)` is
important: the temp file must be on the same filesystem as the target.
`Files.createTempFile()` without a parent uses the system temp dir
(`/tmp` on Linux) which may be a different filesystem from `/etc/app/`.
Cross-filesystem move is NOT atomic (copies then deletes). Another
production pattern: databases use WAL (Write-Ahead Log) which is the
same principle - write changes to a separate log file first, then apply
to the main file.

---

**Q3 (WatchService internals): How does WatchService work under the hood?**

A: `WatchService` delegates to OS-level file change notification mechanisms:

| OS | Mechanism | Details |
|---|---|---|
| Linux | `inotify` | Kernel watches inodes; instant notification |
| macOS | `kqueue` / FSEvents | JDK 8: polling fallback; JDK 17+: FSEvents |
| Windows | `ReadDirectoryChangesW` | Win32 API for directory change events |

```java
// Registration:
WatchService ws = FileSystems.getDefault().newWatchService();
path.register(ws, ENTRY_CREATE, ENTRY_MODIFY, ENTRY_DELETE);
// Under the hood: JVM calls inotify_add_watch(fd, path, mask) on Linux

// Consuming events:
WatchKey key = ws.take(); // blocks until event (or poll() for non-blocking)
// Under the hood: JVM calls read(inotify_fd) which blocks until event

// Event details:
for (WatchEvent<?> event : key.pollEvents()) {
    Path changed = (Path) event.context(); // only FILENAME, not full path
    int count = event.count(); // >1 if repeated events coalesced
}
key.reset(); // MUST reset or key goes invalid; no more events for this key
// Under the hood: continues the inotify watch

// Cancel (stop watching):
key.cancel();
ws.close(); // releases inotify file descriptor
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using SQL. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The WatchKey lifecycle is critical:
after `take()`, the key is in "signaled" state and won't receive events.
`key.reset()` returns it to "ready" state. Forgetting `key.reset()`
means: you process the first batch of events, then never get more.
Symptom: file changes detected initially, then missed. Also: WatchService
monitors directories, not individual files. To watch `config.json`:
register its parent directory and filter events by filename.

---

**Q4 (Files.walk resource management): What resource leak can occur
with Files.walk()?**

A: `Files.walk()` opens a directory stream internally. This stream holds
a file descriptor. Without closing: the FD leaks.

```java
// LEAK: no try-with-resources
Stream<Path> paths = Files.walk(dir);
paths.forEach(System.out::println);
// Stream is not closed! DirectoryStream FD leaks.
// GC eventually finalizes it, but non-deterministic.

// CORRECT: try-with-resources
try (Stream<Path> paths = Files.walk(dir)) {
    paths.forEach(System.out::println);
} // DirectoryStream closed here

// Also: if early-terminating the stream:
try (Stream<Path> paths = Files.walk(dir)) {
    Optional<Path> first = paths
        .filter(p -> p.endsWith("pom.xml"))
        .findFirst(); // stream not fully consumed - still need to close!
}

// Files.find: walk with built-in filter (more efficient than walk+filter):
try (Stream<Path> found = Files.find(dir, 10,
        (path, attrs) -> attrs.isRegularFile()
                         && path.toString().endsWith(".log"))) {
    found.forEach(System.out::println);
}
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

**Also:** `Files.list()` (single directory, not recursive) has the same
requirement: close the returned stream.

*What separates good from great:* File descriptor exhaustion (`Too many
open files`) is a production failure mode in Java services. Default FD
limits on Linux: 1024 per process (adjustable). A service that calls
`Files.walk()` or `Files.list()` in a loop without closing will exhaust
FDs quickly under load. Diagnosis: `lsof -p <pid> | wc -l` (count open FDs),
`lsof -p <pid> | grep DIR` (directory streams). `-XX:+PrintGC` and heap
dumps won't show this - it's an OS resource leak, not heap memory.

---

**Q5 (Copy options): What StandardCopyOption values exist and when
do you use each?**

A:
```java
// REPLACE_EXISTING: overwrite target if it exists
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
// Without REPLACE_EXISTING: throws FileAlreadyExistsException if target exists

// COPY_ATTRIBUTES: copy file attributes (timestamps, permissions)
Files.copy(source, target,
    StandardCopyOption.REPLACE_EXISTING,
    StandardCopyOption.COPY_ATTRIBUTES);
// Without: target gets current time as creation time

// ATOMIC_MOVE: move atomically (same filesystem only)
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
// Throws AtomicMoveNotSupportedException if cross-filesystem

// LinkOption.NOFOLLOW_LINKS: don't follow symlinks
Files.copy(symlink, target, LinkOption.NOFOLLOW_LINKS);
// Copies the symlink itself, not the file it points to

// Combination for directory copy (Files.copy only copies one file/dir):
Files.copy(source, target,
    StandardCopyOption.REPLACE_EXISTING,
    StandardCopyOption.COPY_ATTRIBUTES);
// For recursive directory copy: use Files.walk + copy each file
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `Files.copy()` does NOT recursively
copy directories - it only copies a single file or creates an empty
directory. For recursive directory copy: combine `Files.walk()` with
individual `Files.copy()` calls for each file. This is a common interview
question: "How do you copy a directory tree?" Answer: walk the source,
for each file/directory: create the relative path in the target, copy
the file. The `walkFileTree` with a `FileVisitor` is the traditional
approach; `Files.walk()` stream with manual copy is the modern approach.

---

**Q6 (Directory traversal options): What are the options for traversing
a directory tree?**

A:
```java
// 1. Files.walk(path): depth-first, Stream<Path>
// Simple, functional, closes properly with try-with-resources
try (Stream<Path> walk = Files.walk(root)) {
    walk.filter(Files::isRegularFile)
        .forEach(this::processFile);
}

// 2. Files.find(path, depth, matcher): walk with built-in filter
// More efficient than walk+filter when filter is simple:
try (Stream<Path> found = Files.find(root, 10,
        (p, attrs) -> attrs.isRegularFile() && attrs.size() > 1024 * 1024)) {
    found.forEach(System.out::println); // files > 1MB
}

// 3. Files.walkFileTree: event-based visitor (pre/post-visit):
Files.walkFileTree(root, new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
        process(file, attrs);
        return FileVisitResult.CONTINUE;
    }
    @Override
    public FileVisitResult visitFileFailed(Path file, IOException e) {
        log.error("Cannot visit: {} - {}", file, e.getMessage());
        return FileVisitResult.CONTINUE; // skip, don't abort
    }
    @Override
    public FileVisitResult preVisitDirectory(Path dir,
            BasicFileAttributes attrs) {
        if (dir.getFileName().toString().startsWith(".")) {
            return FileVisitResult.SKIP_SUBTREE; // skip hidden dirs
        }
        return FileVisitResult.CONTINUE;
    }
});

// FileVisitResult options:
// CONTINUE, SKIP_SUBTREE (skip this directory's contents),
// SKIP_SIBLINGS (skip other files in same directory), TERMINATE
```

> **Code walkthrough:** This Unknown example demonstrates Java Stream pipeline. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **TAKEAWAY: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* `visitFileFailed` in `walkFileTree`
handles permission-denied errors gracefully. `Files.walk()` by default
throws on permission-denied entries; `walkFileTree` with `visitFileFailed`
returning `CONTINUE` skips inaccessible paths. For production directory
scans (backup tools, search indexers): always use `walkFileTree` with
proper `visitFileFailed` handling. `SKIP_SUBTREE` is essential for
performance: skipping `node_modules`, `.git`, build output directories.

---

**Q7 (Temp files and paths): How do you create temp files safely?**

A:
```java
// Creates temp file with unique name in system temp dir:
Path temp = Files.createTempFile("prefix-", ".tmp");
// e.g., /tmp/prefix-8429347230423.tmp

// Creates temp file in specific directory:
Path temp = Files.createTempFile(dir, "prefix-", ".tmp");

// Temp directory:
Path tempDir = Files.createTempDirectory("myapp-");

// Auto-delete on JVM exit (legacy approach):
temp.toFile().deleteOnExit(); // unreliable: only on normal JVM exit!

// CORRECT: try-with-resources for temp file lifetime:
Path temp = Files.createTempFile("work-", ".json");
try {
    Files.writeString(temp, json, UTF_8);
    processFile(temp);
} finally {
    Files.deleteIfExists(temp); // explicit cleanup
}

// For temp directory with cleanup:
Path tempDir = Files.createTempDirectory("workdir-");
try {
    // use tempDir
} finally {
    // Recursively delete:
    Files.walk(tempDir)
        .sorted(Comparator.reverseOrder()) // delete files before dirs
        .forEach(p -> { try { Files.delete(p); }
                         catch (IOException e) { /* log */ } });
}
```

> **Code walkthrough:** This Unknown example demonstrates exception handling using SQL. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* `deleteOnExit()` is unreliable: only
called on normal JVM shutdown (not on kill -9, OutOfMemoryError, or
StackOverflow). Temp files from `deleteOnExit()` accumulate if the JVM
crashes. Pattern: use explicit deletion in a `finally` block or
try-with-resources. For large-scale temp file usage (many parallel
operations): use a temp directory under your application's work dir
(not `/tmp`) and clean it up on application startup (orphaned temps
from previous crashes). `/tmp` is typically `tmpfs` (in-memory on Linux)
- large temp files fill RAM, not disk.

---

**Q8 (File attributes): How do you read and set file attributes?**

A:
```java
// BasicFileAttributes: available on all filesystems
BasicFileAttributes attrs = Files.readAttributes(
    path, BasicFileAttributes.class);
attrs.creationTime()     // FileTime
attrs.lastModifiedTime() // FileTime
attrs.lastAccessTime()   // FileTime
attrs.size()             // in bytes
attrs.isRegularFile()
attrs.isDirectory()
attrs.isSymbolicLink()
attrs.fileKey()          // filesystem-specific ID (inode on UNIX)

// PosixFileAttributes: POSIX filesystems (Linux, macOS)
PosixFileAttributes posix = Files.readAttributes(
    path, PosixFileAttributes.class);
posix.owner()            // UserPrincipal
posix.group()            // GroupPrincipal
posix.permissions()      // Set<PosixFilePermission>

// Read/write permissions:
Set<PosixFilePermission> perms = posix.permissions();
// {OWNER_READ, OWNER_WRITE, GROUP_READ, OTHERS_READ}

// Set permissions:
Files.setPosixFilePermissions(path,
    PosixFilePermissions.fromString("rwxr-xr-x"));
// or:
Files.setAttribute(path, "posix:permissions",
    PosixFilePermissions.fromString("rw-r--r--"));

// Set timestamps:
Files.setLastModifiedTime(path, FileTime.fromMillis(System.currentTimeMillis()));
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `BasicFileAttributes.fileKey()` returns
the inode number on UNIX filesystems (as `Object` - cast to `Long` after
checking). Two `Path` objects with the same `fileKey` are the same
underlying file (handles hard links and symlinks). This is how `Files.isSameFile(p1, p2)` works. In deduplication tools and backup systems: use `fileKey`
to detect hard links (same inode = same physical file blocks, don't copy twice).
`Files.isSameFile()` does the inode comparison correctly even for symlinks
pointing to the same file with different paths.

---

**Q9 (Symbolic links handling): How does NIO2 handle symbolic links?**

A:
```java
// By default: Files methods FOLLOW symlinks
// (operate on the target of the link)
Files.size(symlink);      // size of target file
Files.readAttributes(symlink, BasicFileAttributes.class).isSymbolicLink();
// Returns false! Attributes of the TARGET (regular file)

// NOFOLLOW_LINKS: operate on the symlink itself
Files.readAttributes(symlink, BasicFileAttributes.class,
    LinkOption.NOFOLLOW_LINKS).isSymbolicLink(); // returns true

// Read symlink target:
Path target = Files.readSymbolicLink(symlink);
// Returns the path the symlink points to

// Create symlink:
Files.createSymbolicLink(link, target);
// link -> target

// Check if a path is a symlink (without following):
Files.isSymbolicLink(path); // same as NOFOLLOW readAttributes

// Walk: follows symlinks by default? NO (avoids infinite loops)
Files.walk(root); // by default: FileVisitOption.NOFOLLOW_LINKS
Files.walk(root, FileVisitOption.FOLLOW_LINKS); // follows (may cycle!)
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

Detecting cycles when following symlinks:
```java
Files.walkFileTree(root,
    EnumSet.of(FileVisitOption.FOLLOW_LINKS),
    Integer.MAX_VALUE,
    new SimpleFileVisitor<>() {
        @Override
        public FileVisitResult visitFileFailed(Path file, IOException e) {
            if (e instanceof FileSystemLoopException) {
                log.warn("Cycle detected: {}", file);
                return FileVisitResult.CONTINUE; // skip the cycle
            }
            throw e;
        }
    });
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `Files.walk()` does NOT follow symlinks
by default - a safe default that prevents infinite loops (`a -> b -> a`).
`FileVisitOption.FOLLOW_LINKS` opts into symlink following, and
`FileSystemLoopException` (subclass of IOException) signals a detected
cycle (by comparing `fileKey` values). In container environments (Docker):
symlinks are common for configuration injection, library versioning
(`/usr/lib/libssl.so -> libssl.so.1.1`), and JDK installation
(`/usr/bin/java -> /etc/alternatives/java`). Knowing how to handle them
correctly matters for tools that inspect containers.

---

### ⚖️ Comparison Table

| Operation | java.io.File | java.nio.file.Files |
|---|---|---|
| Delete | `delete()` returns boolean | `delete()` throws IOException |
| Atomic move | Not supported | `move(ATOMIC_MOVE)` |
| Check existence | `exists()` | `exists()` with follow/no-follow |
| Directory list | `listFiles()` (array) | `list()` (lazy stream) |
| Tree walk | Manual recursion | `walk()`, `walkFileTree()` |
| Change watch | Polling | `WatchService` (OS events) |
| File metadata | Limited | `readAttributes(BasicFileAttributes)` |
| Path operations | String concatenation | `resolve()`, `relativize()`, etc. |

---


---

# Java Core - L3 String Processing

## String Processing and Regular Expressions

---

### 🎯 Model Answer

**30 seconds:**
> Java Strings are immutable UTF-16 sequences. String operations that
> "modify" a string create new objects. `StringBuilder` is mutable for
> repeated concatenation. For regex: `Pattern.compile(regex)` returns a
> thread-safe `Pattern`; `pattern.matcher(str)` returns a non-thread-safe
> `Matcher` (per-thread). Key regex risk: catastrophic backtracking -
> `(a+)+b` on "aaaaac" causes exponential time. Always pre-compile patterns
> as static finals. `String.format` is 5-10x slower than concatenation for
> simple cases; use it only for complex formatting.

**3 minutes (Senior):**
> String immutability supports the String pool: identical literals share
> references. `intern()` moves a runtime string into the pool. Java 9+
> compact strings: ASCII-only strings stored as byte[] (Latin-1 encoding)
> instead of char[], halving memory for common ASCII strings.
>
> StringBuilder: single-threaded mutable string. StringBuffer: same but
> synchronized (rarely needed - prefer StringBuilder). String concatenation
> in loops: Java compiler converts `s += x` to `new StringBuilder().append(s).append(x).toString()`
> per iteration - O(n^2) for n iterations. Pre-allocate StringBuilder with
> expected capacity to minimize resizing.
>
> Regex performance: `Pattern.compile()` builds NFA (expensive); do once as
> static final. `matches()` requires FULL string match; `find()` searches
> for substring match. Possessive quantifiers (`a++`) and atomic groups
> (`(?>...)`) prevent backtracking. For input validation (user data):
> always set a character limit before applying regex to prevent ReDoS attacks.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "String processing - let me cover String immutability
and pool, StringBuilder vs StringBuffer, regex with Pattern/Matcher,
performance traps, and ReDoS security."

**(2) First principles:** "Strings being immutable means concatenation
allocates new objects. For building strings: StringBuilder pools mutations
before creating the final string. Regex patterns are compiled state
machines - compile once, run many times."

**(3) Bridge:** "String processing is like carpentry. String concatenation
is a single cut. StringBuilder is a workbench where you assemble pieces
before showing the final result. Regex is a CNC machine - expensive
to set up (compile), fast to run repeatedly."

---

### 📘 Concept Explanation

**String pool and immutability:**
```
"Hello" literal -> String pool -> single shared instance
new String("Hello") -> heap object (outside pool)
"Hello".intern() -> look up pool, return pooled reference
```


---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| String immutability benefits | 2 minutes |
| StringBuilder vs StringBuffer | 90 seconds |
| String pool and intern() | 2 minutes |
| Regex Pattern/Matcher | 2 minutes |
| ReDoS and security | 2-3 minutes |
| String.format performance | 2 minutes |
| Text blocks (Java 15) | 90 seconds |
| Locale-sensitive operations | 2 minutes |
| String comparison | 90 seconds |

---

**Q1 (String immutability): What are the benefits of String immutability?**

A:
1. **Thread safety:** shared strings need no synchronization
2. **String pool:** identical literals share one instance (memory savings)
3. **Hashcode caching:** `String.hashCode()` is computed once and cached (used as HashMap keys)
4. **Security:** passwords/filenames can't be modified by callers; interned strings stable
5. **Simplicity:** method contracts are clearer without defensive copies

```java
// Thread safety - no synchronization needed:
static final String CONFIG_KEY = "database.url"; // shared across threads
// Mutation would be unsafe: any thread could corrupt CONFIG_KEY

// Hashcode caching (see String source):
// private int hash; // cached on first call
String key = "somekey";
map.get(key); // hashCode computed and cached on first call
map.get(key); // subsequent calls use cached value (0 check: 0 not cached)

// Security: password as char[] is better than String for clearing:
char[] password = {'s','e','c','r','e','t'};
// ... use password ...
Arrays.fill(password, '\0'); // zero out after use - preventss heap dump exposure
// vs: String password = "secret" -> can't clear! stays in pool
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The security implication is subtle.
`String` in the pool can outlive the use - a heap dump taken after the
user logged out still contains the password string. `char[]` passwords
can be zeroed after use, removing them from memory. This is why
`JPasswordField.getPassword()` returns `char[]`, not `String`. Modern
secure coding: use `char[]` for passwords/secrets, zero after use.

---

**Q2 (StringBuilder vs StringBuffer): When do you use StringBuilder vs StringBuffer?**

A: **StringBuilder** (Java 5): not thread-safe, high performance.
**StringBuffer** (Java 1.0): all methods synchronized, thread-safe, slower.

In practice: always use `StringBuilder` unless you specifically need
a thread-safe mutable string shared between threads (rare).

```java
// SingleThread: StringBuilder
void buildLog(List<Event> events) {
    StringBuilder sb = new StringBuilder(events.size() * 100);
    for (Event e : events) sb.append(e).append('\n');
    return sb.toString();
}

// Multi-thread scenario where StringBuffer MIGHT be used:
// (but usually better to use a different design)
class LogAggregator {
    private final StringBuffer buffer = new StringBuffer();
    void addLog(String entry) { buffer.append(entry).append('\n'); }
    String getLog()  { return buffer.toString(); }
}
// Better: ConcurrentLinkedQueue<String>, then join at read time

// Realistic answer: StringBuffer is legacy. In 20 years of Java development,
// you'll almost never need it. StringBuilder for everything.
```

> **Code walkthrough:** This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **TAKEAWAY: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* The JVM JIT can sometimes "destack"
`StringBuilder` allocations in tight loops (escape analysis). If the
StringBuilder doesn't escape the method, the JIT may eliminate the allocation
entirely, placing the buffer on the stack. This is an optimization you don't
control but is worth knowing: prefer local StringBuilder variables that don't
escape for maximum JIT optimization opportunity.

---

**Q3 (String pool and intern()): How does the String pool work and when do you intern()?**

A: The String pool (String intern table) is a hash table of canonical
string references in the JVM heap (Java 8+, was PermGen before).
String literals are automatically interned. `intern()` looks up or inserts
a string into the pool.

```java
String a = "hello"; // from pool (literal)
String b = "hello"; // same pool reference
System.out.println(a == b); // true (same object)

String c = new String("hello"); // new heap object, NOT from pool
System.out.println(a == c); // false (different objects)
System.out.println(a.equals(c)); // true (same content)

String d = c.intern(); // look up "hello" in pool -> returns a
System.out.println(a == d); // true (d is the pooled reference)

// Performance use case: intern large numbers of repeated strings
// (e.g., parsing CSV with repeated category values)
// WARNING: interning millions of unique strings = OutOfMemoryError
// (pool never GCed for strong references)

// Safer alternative: use an explicit cache:
Map<String, String> cache = new HashMap<>();
String intern(String s) {
    return cache.computeIfAbsent(s, k -> k); // own pool, GC-able
}
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Interning is a space-time trade-off.
If you have millions of record fields that are frequently the same string
(stock symbols, country codes, status strings), interning eliminates
duplicate objects. Netflix's Hollow library uses this for large dataset
optimizations. The risk: the JVM String pool's default table size is 60013
(Java 8 and earlier); adjust with `-XX:StringTableSize=N` for large pools.
Java 11+ uses a default of 65536. For production analytics with many distinct
strings: don't intern (memory pressure); for repeated lookup values: interning
reduces memory.

---

**Q4 (Regex Pattern/Matcher): How do you use Pattern and Matcher efficiently?**

A:
```java
// WRONG: re-compiling pattern every call
boolean isValid(String email) {
    return email.matches("[^@]+@[^@]+"); // compiles pattern EVERY call!
}

// CORRECT: static final Pattern (compile once)
private static final Pattern EMAIL =
    Pattern.compile("^[a-zA-Z0-9._%+\\-]+@[a-zA-Z0-9.\\-]+\\.[a-zA-Z]{2,}$");

boolean isValid(String email) {
    if (email == null || email.length() > 254) return false;
    return EMAIL.matcher(email).matches(); // reuse Pattern, new Matcher OK
}
// Matcher is NOT thread-safe - create new per call (cheap)
// Pattern IS thread-safe - one static instance is fine

// Extracting groups:
Pattern p = Pattern.compile("(\\d+)\\s+(\\w+)");
Matcher m = p.matcher("42 items found");
if (m.find()) {
    String count = m.group(1); // "42"
    String unit  = m.group(2); // "items"
}

// Find all matches:
Pattern WORD = Pattern.compile("\\b\\w+\\b");
Matcher wm = WORD.matcher("Hello World Java");
while (wm.find()) {
    System.out.println(wm.group() + " at " + wm.start());
}

// Replace with function (Java 9):
String result = Pattern.compile("\\d+")
    .matcher("abc 123 def 456")
    .replaceAll(mr -> String.valueOf(Integer.parseInt(mr.group()) * 2));
// "abc 246 def 912"
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* `Pattern.compile()` is expensive because
it builds a finite automaton from the regex string. In microbenchmarks, it's
100-1000x slower than `Matcher.matches()`. In a web request handler called
1000 times/second, recompiling a regex on every request wastes CPU.
Use `static final Pattern` as the default. The `Matcher.replaceAll(Function)`
in Java 9 enables dynamic replacement without needing to manually loop with
`find()` and `appendReplacement()`/`appendTail()` - cleaner and less error-prone.

---

**Q5 (ReDoS): What is ReDoS and how do you prevent it?**

A: **ReDoS** (Regular Expression Denial of Service): a regex that takes
exponential time on certain inputs, blocking the thread.

Cause: nested quantifiers on ambiguous patterns cause catastrophic
backtracking. The regex engine tries every possible match path.

```java
// Vulnerable patterns (exponential on adversarial input):
// (a+)+b   -> "aaaaaac" = exponential backtracking
// (a|aa)+  -> similar
// (.*a){20} -> exponential on strings with many 'a's

// Attack: send "aaaaaaaaaaaaaaaaaac" (many a's, no b)
// Engine tries every combination -> thread blocked for seconds/minutes

// PREVENTION:
// 1. Bound input length (before regex):
if (input.length() > 1000) throw new IllegalArgumentException("Too long");

// 2. Use possessive quantifiers (Java supports):
Pattern.compile("(a++)++b"); // a++ never gives back what it matched

// 3. Atomic groups (same as possessive):
Pattern.compile("(?>a+)+b");

// 4. Avoid nested quantifiers on ambiguous patterns:
// Instead of (a+)+ use a+ (they match the same but without catastrophic backtracking risk)

// 5. Use a regex linter/validator:
// - ReDoS Checker: https://devina.io/redos-checker
// - vuln-regex-detector: npm package for CI/CD

// Production example: OWASP recommends
// - All user input: length limit + character whitelist
// - Complex validation: explicit parser over regex
// - Use java.util.regex timeout wrapper if needed:
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Boolean> future = executor.submit(
    () -> UNSAFE_PATTERN.matcher(userInput).matches());
try {
    return future.get(100, TimeUnit.MILLISECONDS); // timeout!
} catch (TimeoutException e) {
    future.cancel(true);
    throw new ValidationException("Input validation timeout");
}
```

> **Code walkthrough:** This Unknown example demonstrates thread pool management using thread pool. **KEY MECHANISM:** the pool maintains a work queue; submitted tasks block until a thread is free. **WHY IT MATTERS:** unconfigured pool sizes exhaust threads under load or waste memory at rest. **TAKEAWAY: always name threads and bound queue size to detect saturation.**

*What separates good from great:* ReDoS is a real attack vector. In 2016,
a regex in the Node.js `moment` library caused a severe ReDoS vulnerability.
In 2019, Cloudflare had an outage partly caused by a catastrophic backtracking
regex in their WAF. Java's `java.util.regex` has NO built-in timeout.
At scale, a single malicious request can cause a thread to hang for minutes.
The timeout wrapper pattern (ExecutorService + Future.get with timeout) is
the only safe mitigation for user-provided content validated by complex regex.
For greenfield code: prefer explicit parsers (parser combinators, Antlr) for
complex grammar validation.

---

**Q6 (String.format performance): When is String.format appropriate?**

A: `String.format()` parses the format string on every call and uses
varargs (boxing primitives). For high-frequency logging or string building:
prefer concatenation or StringBuilder.


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// Benchmark: 1 million iterations
// String.format: ~800ms
// StringBuilder.append: ~15ms
// String concat (JIT-optimized): ~20ms

// LOW FREQUENCY: String.format is fine (human-readable, error-resistant)
String errorMsg = String.format(
    "User %s (id=%d) failed login at %s from %s",
    username, userId, timestamp, ipAddress);
// Great for error messages - rare, readability matters

// HIGH FREQUENCY: avoid String.format in hot paths
// BAD: logging in tight loop
for (Item item : millionItems) {
    log.debug(String.format("Processing item %d: %s", item.id(), item.name()));
}

// GOOD: use SLF4J's lazy formatting (evaluates only if debug enabled!)
for (Item item : millionItems) {
    log.debug("Processing item {}: {}", item.id(), item.name());
    // SLF4J: if DEBUG disabled, no String created at all!
}

// Text blocks (Java 15): excellent for multi-line strings
String json = """
    {
      "name": "%s",
      "age": %d
    }
    """.formatted(name, age); // .formatted() on text blocks (Java 15)
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **WHAT BREAKS: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The SLF4J pattern (`log.debug("x {}", y)`)
is far more important than `String.format` for performance. If debug logging
is disabled (production default), SLF4J never evaluates the format string
or calls `toString()` on the arguments. With `String.format("x %s", obj)`:
`obj.toString()` is always called (even when logging is disabled), then the
format string is parsed, then the result is created - all wasted. This is
why SLF4J's parameterized logging is a performance best practice, not just
style.

---

**Q7 (Text blocks): How do text blocks work in Java 15+?**

A: Text blocks are multi-line string literals that strip common leading
whitespace and use `\n` as line terminator:

```java
// Before: escaping nightmare
String json = "{\n    \"name\": \"Alice\",\n    \"age\": 30\n}";

// After: text block (Java 15, standard)
String json = """
    {
        "name": "Alice",
        "age": 30
    }
    """;
// Incidental whitespace stripped based on closing """ position

// Position of closing """ determines stripping:
String a = """
    Hello
    World
    """; // trailing newline included; strips 4 spaces
String b = """
    Hello
    World
    """.stripIndent(); // explicit stripping

// Escape sequences in text blocks:
String s = """
    Line 1\
    Line 2
    """;
// \<newline> = line continuation: "Line 1Line 2\n"

// \s (trailing whitespace anchor):
String s = """
    column1   \s
    column2   \s
    """;
// \s prevents stripping of intentional trailing spaces

// Formatted:
String sql = """
    SELECT *
    FROM users
    WHERE id = %d
    """.formatted(userId);
```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using SQL. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* Text blocks improve correctness, not
just readability. The stripping algorithm is deterministic: indentation
common to ALL content lines (and the closing `"""`) is stripped. This means
the indented text block inside a method body doesn't produce leading spaces
in the output. `\s` at line end is the escape to preserve intentional
trailing whitespace - useful for fixed-width data, CSV generation, protocol
messages where trailing spaces are significant.

---

**Q8 (Locale-sensitive): When does Locale matter for string operations?**

A:

```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```


```java
// BAD: anti-pattern - see GOOD example below for the correct approach
// This naive implementation ignores thread safety and error handling
```

```java
// BAD: locale-sensitive comparison in code logic
String country = userInput.toLowerCase();
if (country.equals("turkey")) { // wrong!
// "Turkey".toLowerCase() in Turkish locale = "turkey" but
// "I".toLowerCase(Locale.of("tr","TR")) = "ı" (dotless i)
// "I".toLowerCase() may return "i" or "ı" depending on default Locale!

// GOOD: locale-independent for programmatic comparisons
String country = userInput.toLowerCase(Locale.ROOT);
// Locale.ROOT: no locale-specific mappings, A-Z only

// GOOD: locale-specific for user display
String userDisplay = name.toUpperCase(Locale.forLanguageTag("tr-TR"));

// Locale.ROOT vs Locale.ENGLISH:
// Locale.ROOT: stable, no locale rules (use for code-facing operations)
// Locale.ENGLISH: English locale rules (may differ on some edge chars)

// String comparison ignoring case:
// BAD: "Turkey".equalsIgnoreCase("turkey") - may fail with Turkish locale
// GOOD:
boolean match = s1.toLowerCase(Locale.ROOT)
    .equals(s2.toLowerCase(Locale.ROOT));
// Or: Collator for full locale-aware comparison
Collator collator = Collator.getInstance(Locale.GERMAN);
collator.setStrength(Collator.PRIMARY); // ignore case and accents
int cmp = collator.compare("straße", "STRASSE"); // 0 (same in German)
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates exception handling. **KEY MECHANISM:** the JVM checks catch clauses in order; finally always executes for cleanup. **WHY IT MATTERS:** swallowing exceptions silently hides failures that corrupt downstream state. **WHAT BREAKS: log or rethrow every exception; empty catch blocks are defects.**

*What separates good from great:* Locale bugs are among the hardest to
reproduce - they only manifest on machines with non-English system locales.
The famous Turkish test: set JVM locale to Turkish (`java -Duser.language=tr`),
then `"FILE".toLowerCase().equals("file")` returns `false` because "I"
lowercases to "ı" (dotless i). This breaks string comparisons in HTTP
header parsing, configuration key comparisons, and enum name lookups.
Production rule: every `toLowerCase()` / `toUpperCase()` in production code
needs an explicit Locale. `Locale.ROOT` for internal identifiers.
User's locale for display.

---

**Q9 (String comparison): How do you correctly compare strings?**

A:
```java
// == compares references, not content:
String a = new String("hello");
String b = new String("hello");
a == b      // false (different objects)
a.equals(b) // true  (same content)

// Case-insensitive content comparison:
// BAD: locale-sensitive
a.equalsIgnoreCase(b);
// BETTER: explicit locale
a.toLowerCase(Locale.ROOT).equals(b.toLowerCase(Locale.ROOT));

// Null-safe comparison (Java 7+):
Objects.equals(a, b); // returns false if either is null, not NPE

// Comparing with a known constant (yoda style to avoid NPE):
"expected".equals(userInput); // NPE-safe (left side is non-null)
// vs: userInput.equals("expected") -> NPE if userInput is null

// Ordering strings (for sorting):
List<String> names = List.of("charlie", "Alice", "bob");
// Case-insensitive sort:
names.stream()
    .sorted(String.CASE_INSENSITIVE_ORDER)
    .forEach(System.out::println); // Alice, bob, charlie

// Locale-aware sort (for user display):
Collator de = Collator.getInstance(Locale.GERMAN);
names.sort(de); // handles German umlauts correctly
```

> **Code walkthrough:** BAD pattern: This Unknown example demonstrates Java Stream pipeline using Stream. **KEY MECHANISM:** the stream is lazy - intermediate ops build a pipeline, terminal op drives it. **WHY IT MATTERS:** calling terminal op twice throws IllegalStateException; parallel() on small data adds overhead. **WHAT BREAKS: collect() or findFirst() triggers the pipeline; reuse by wrapping in Supplier.**

*What separates good from great:* `Objects.equals(a, b)` is the standard
null-safe equality check. In Spring/JPA code, null checks before
`.equals()` are boilerplate: `a != null && a.equals(b)` - replaced by
`Objects.equals(a, b)`. For sorting user-visible data: always use
`Collator` with the user's locale, not natural String ordering (which is
Unicode codepoint order, not alphabetical in any language). "ü" sorts
after "z" in Unicode order but between "u" and "v" in German locale.

---

### ⚖️ Comparison Table

| Tool | Thread Safe | Use Case | Performance |
|---|---|---|---|
| String concat (`+`) | Yes (immutable) | 1-2 concatenations | Fine |
| StringBuilder | No | Loop building, single thread | Fastest |
| StringBuffer | Yes | Shared mutable string | Slower (sync) |
| String.join | Yes | Static list join | Good |
| Collectors.joining | Yes | Stream collection | Good |
| String.format | Yes | Complex formatting | Slow |
| Text block | Yes | Multi-line literals | Same as String |

---

### 🏛️ System Design

*(Omit: ★★☆ level - system design not required)*

---

### 📊 Diagram

*(Omit: non-visual concept)*

---

---

## Java Annotations

---

### 🎯 Model Answer

**30 seconds:**
> Annotations are metadata markers on code elements (classes, methods,
> fields, parameters, packages). Built-in: `@Override`, `@Deprecated`,
> `@SuppressWarnings`, `@FunctionalInterface`. Custom annotations:
> `@interface MyAnnotation { String value() default ""; }`. Retention:
> `SOURCE` (discarded at compile), `CLASS` (in bytecode, not loaded),
> `RUNTIME` (available via reflection). Most framework annotations
> (`@Autowired`, `@Entity`, `@Test`) are RUNTIME retention. Annotation
> processors run at compile time (Lombok, MapStruct).

**3 minutes (Senior):**
> `@Retention(RUNTIME)` enables runtime processing via reflection:
> `method.getAnnotation(Transactional.class)`. `@Target` restricts where
> an annotation can be used (`ElementType.METHOD`, `FIELD`, `TYPE`, etc.).
> `@Repeatable` (Java 8) allows the same annotation multiple times on one element.
>
> Two processing models: (1) annotation processors (APT, JSR 269) run at
> compile time via `javac` - produce new source files or validate. Lombok,
> MapStruct, Dagger use this. (2) Runtime reflection - Spring, Hibernate,
> JUnit read `RUNTIME` retention annotations after class loading.
>
> Performance: reflection-based annotation reading has overhead; frameworks
> cache the results after first scan. Spring scans class annotations once
> at startup, caches to `BeanDefinition`. Hibernate scans entity annotations
> once per `SessionFactory` creation. Avoid reading annotations in hot paths
> without caching.

**Framework:** WHAT → WHY → HOW → TRADE-OFF → EXAMPLE

**Blank Mind Recovery:**

**(1) Restate:** "Annotations - let me cover definition, meta-annotations,
retention policies, custom annotations, compile-time processors vs runtime
reflection."

**(2) First principles:** "Annotations add metadata without changing code
behavior. The metadata needs a consumer: either the compiler, an APT
processor at compile time, or reflection at runtime."

**(3) Bridge:** "Annotations are like sticky notes on code. `SOURCE`
annotations are notes the compiler reads then throws away. `CLASS` annotations
are permanent notes in the file but no one reads them at runtime. `RUNTIME`
annotations are notes anyone can read while the program runs."

---

### 📘 Concept Explanation

**Meta-annotations (annotations on annotations):**
```java
@Retention(RetentionPolicy.RUNTIME)  // when available
@Target({ElementType.METHOD, ElementType.TYPE}) // where usable
@Documented                          // include in Javadoc
@Inherited                           // subclasses inherit from superclass
@Repeatable(MyAnnotations.class)     // can be used multiple times
public @interface MyAnnotation {
    String value() default "";       // element named "value" (special!)
    int timeout() default 30;
    Class<?>[] targets() default {};
}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

**Defining and using custom annotations:**
```java
// Define:
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Validate {
    int minLength() default 0;
    int maxLength() default Integer.MAX_VALUE;
    String pattern() default ".*";
    String message() default "Validation failed";
}

// Use:
class User {
    @Validate(minLength=2, maxLength=50, message="Name must be 2-50 chars")
    private String name;

    @Validate(pattern="[^@]+@[^@]+\\.[^@]+", message="Invalid email")
    private String email;
}

// Process at runtime via reflection:
void validate(Object obj) throws ValidationException {
    for (Field field : obj.getClass().getDeclaredFields()) {
        Validate v = field.getAnnotation(Validate.class);
        if (v == null) continue;
        field.setAccessible(true);
        String value = (String) field.get(obj);
        if (value == null || value.length() < v.minLength()) {
            throw new ValidationException(
                field.getName() + ": " + v.message());
        }
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

---

### 💻 Code Example

> **Code walkthrough:** The custom `@RateLimit` annotation with AOP processingice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**
> shows the standard framework pattern: define the annotation (metadata),
> implement the processor (behavior), wire them together (AOP / annotation
> processor). The annotation carries the configuration (max calls, period);
> the processor implements the logic. This separates concern: calling code
> expresses intent (`@RateLimit(max=100)`), infrastructure implements it.

```java
// Custom annotation for rate limiting:
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RateLimit {
    int maxCalls() default 100;
    int perSeconds() default 60;
    String message() default "Rate limit exceeded";
}

// Usage:
@Service
class PaymentService {
    @RateLimit(maxCalls=10, perSeconds=60, message="Too many payment attempts")
    public Receipt processPayment(PaymentRequest request) { ... }

    @RateLimit(maxCalls=1000, perSeconds=60)
    public PaymentStatus getStatus(String id) { ... }
}

// AOP processor (Spring):
@Aspect
@Component
class RateLimitAspect {
    private final Map<String, RateLimiter> limiters = new ConcurrentHashMap<>();

    @Around("@annotation(rateLimit)")
    public Object enforceLimitAspect(ProceedingJoinPoint pjp,
                                      RateLimit rateLimit) throws Throwable {
        String key = pjp.getSignature().toLongString();
        RateLimiter limiter = limiters.computeIfAbsent(key, k ->
            RateLimiter.create(
                (double) rateLimit.maxCalls() / rateLimit.perSeconds()));

        if (!limiter.tryAcquire()) {
            throw new TooManyRequestsException(rateLimit.message());
        }
        return pjp.proceed();
    }
}

// Compile-time annotation processor (validates at compile time):
@SupportedAnnotationTypes("com.example.RateLimit")
@SupportedSourceVersion(SourceVersion.RELEASE_17)
public class RateLimitProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations,
                           RoundEnvironment roundEnv) {
        for (Element element : roundEnv.getElementsAnnotatedWith(
                RateLimit.class)) {
            RateLimit rl = element.getAnnotation(RateLimit.class);
            if (rl.maxCalls() <= 0) {
                processingEnv.getMessager().printMessage(
                    Diagnostic.Kind.ERROR,
                    "maxCalls must be > 0", element);
            }
        }
        return true;
    }
}
```

> **Code walkthrough:** The AOP aspect uses `@annotation(rateLimit)` toice. **KEY MECHANISM:** the runtime executes these instructions in sequence with specific memory and execution semantics. **WHY IT MATTERS:** misapplying this pattern causes subtle bugs that only manifest under production load. **TAKEAWAY: understand the execution model before using this pattern in production code.**
> intercept any method annotated with `@RateLimit` and inject the annotation
> instance as a parameter (`rateLimit`). The `computeIfAbsent` on
> `ConcurrentHashMap` creates a separate `RateLimiter` (Guava) per method
> signature. The compile-time processor (`AbstractProcessor`) runs during
> `javac` - it catches invalid annotation values before runtime. Most
> framework validation (Bean Validation, Lombok) uses this approach: compile
> errors instead of runtime surprises.

---

### 🎓 Answers by Seniority

**Junior / Mid (0-5 years):**
> `@Retention(RUNTIME)` is required for annotations read via reflection.
> `@Target` restricts where the annotation can appear. Custom annotations:
> `@interface MyAnnotation { String value(); }`. The `value()` element is
> special: `@MyAnnotation("x")` is shorthand for `@MyAnnotation(value="x")`.
> Annotations don't do anything by themselves - they need a processor
> (framework, APT, or your own reflection code) to act on them.

---

**Senior / Staff (5+ years):**
> Annotation processors vs runtime reflection is an architectural choice.
> APT (compile-time): zero runtime cost, strong validation at compile time,
> produces new source files (Lombok, MapStruct, Dagger). Runtime reflection
> (Spring, Hibernate): flexible, can respond to dynamic state, but has
> startup cost and requires JVM module access (Java 9+ `--add-opens`).
> Java 9+ platform modules restrict reflective access; frameworks like Spring
> need `--add-opens java.base/java.lang` or native proxy-based injection.
> In Java 17+ with sealed classes: many patterns using reflection can be
> replaced with type-safe alternatives.

---

### ⚠️ Common Misconceptions

**Misconception 1: "Annotations add runtime behavior themselves."**
Annotations are pure metadata. `@Transactional` doesn't make a method
transactional - Spring's `TransactionInterceptor` reads the annotation
and wraps the method in a transaction. Remove Spring context: `@Transactional`
does nothing. The annotation is a marker; the framework provides behavior.

**Misconception 2: "`@Inherited` works for interfaces and methods."**
`@Inherited` only applies to class-level annotations (meta-annotation
`@Target(TYPE)`), and only for class inheritance (not interface implementation).
`@Inherited` on a superclass annotation IS inherited by subclasses.
But: if an interface is annotated, the implementing class does NOT inherit
that annotation. Spring's `@Transactional` on an interface is a common
mistake - Spring (proxy-based) may not pick it up depending on proxy type.

---

### 🚨 Failure Modes and Diagnosis

**Failure: annotation not picked up by framework.**
```java
// Example: @Transactional on interface not working
interface UserService {
    @Transactional // WRONG: Spring CGLIB proxy doesn't inherit interface annotations
    User save(User user);
}
// Diagnosis: transactions not started, no exception but data not committed

// Fix: put @Transactional on the implementing class method:
@Service
class UserServiceImpl implements UserService {
    @Transactional // CORRECT: Spring reads from concrete class
    public User save(User user) { return repo.save(user); }
}

// OR: use AspectJ weaving instead of CGLIB proxy (full bytecode instrumentation)
// @EnableTransactionManagement(mode = AdviceMode.ASPECTJ)
```
> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

Diagnosis: enable debug logging for `org.springframework.transaction`.
Check `TransactionSynchronizationManager.isActualTransactionActive()` in
the method body.

---

### 🎯 Interview Deep-Dive

| Question Category | Time to Answer |
|---|---|
| Annotation processing models | 2 minutes |
| Retention policies | 90 seconds |
| Custom annotation creation | 2 minutes |
| @Repeatable | 90 seconds |
| APT vs runtime reflection | 2-3 minutes |
| Framework annotation pitfalls | 2 minutes |
| Annotation inheritance | 2 minutes |
| Bean Validation annotations | 2 minutes |
| Security risks | 2 minutes |

---

**Q1 (Processing models): What are the two models for processing annotations?**

A: **1. Compile-time Annotation Processing (APT, JSR 269):**
Runs during `javac` compilation. `AbstractProcessor` subclass registered
in `META-INF/services/javax.annotation.processing.Processor`. Can generate
new source files, emit compile errors/warnings, cannot modify existing source.
Examples: Lombok (modifies AST via compiler internals), MapStruct
(generates mapper implementations), Dagger (generates DI code).

**2. Runtime Reflection:**
Reads `RUNTIME` retention annotations after class loading via
`Class.getAnnotation()`, `Method.getAnnotation()`, etc. Flexible but
has overhead. AOP frameworks (Spring AOP, AspectJ) build proxy objects
that intercept method calls and read annotations. JPA providers (Hibernate)
scan entity classes at `SessionFactory` creation.

```java
// Reading annotation at runtime:
Method m = SomeClass.class.getMethod("myMethod");
RateLimit rl = m.getAnnotation(RateLimit.class);
if (rl != null) {
    System.out.println("Max calls: " + rl.maxCalls());
}

// Scanning all fields for annotation:
Field[] fields = clazz.getDeclaredFields();
for (Field f : fields) {
    if (f.isAnnotationPresent(Validate.class)) {
        // process field
    }
}

// Annotation processor skeleton (APT):
@AutoService(Processor.class) // Guava AutoService: auto-registers
@SupportedAnnotationTypes("com.example.Validate")
class ValidateProcessor extends AbstractProcessor { ... }
```

> **Code walkthrough:** This Unknown example demonstrates metadata declaration. **KEY MECHANISM:** annotations are processed at compile-time or runtime via reflection. **WHY IT MATTERS:** annotation processing adds compile time; runtime reflection disables JIT optimizations. **TAKEAWAY: prefer compile-time annotation processors (APT) over runtime reflection for performance.**

*What separates good from great:* The choice between APT and runtime
reflection is primarily a "when do you want the cost?" question.
APT: cost at build time (slower builds), zero runtime overhead.
Runtime reflection: fast builds, startup cost (Spring context startup
can be seconds), per-call overhead (usually cached). For serverless
(AWS Lambda, Azure Functions): startup time matters - compile-time
injection (Quarkus, Micronaut with APT) beats Spring's runtime reflection
by 10x on cold start. This is why Quarkus uses Jandex (bytecode index)
and APT-style processing to eliminate Spring-style runtime scanning.

---

**Q2 (Retention policies): Explain the three retention policies.**

A:
- `SOURCE`: annotation present in source only, discarded by compiler.
  Use: IDE hints, documentation (`@Override` validates at compile, then gone).
- `CLASS` (default): stored in bytecode (.class file) but NOT loaded into JVM at runtime. Use: bytecode tooling (ASM, Byte Buddy can read them without loading class).
- `RUNTIME`: loaded into JVM, accessible via reflection. Use: ALL framework annotations (`@Autowired`, `@Transactional`, `@Entity`, `@Test`).

```java
@Retention(RetentionPolicy.SOURCE)  // compile-time only
public @interface Todo { String value(); } // reminder for developers

@Retention(RetentionPolicy.CLASS)   // default (rarely needed explicitly)
public @interface BytecodeMarker { String value(); }

@Retention(RetentionPolicy.RUNTIME) // most framework annotations
public @interface Cacheable {
    String region() default "default";
    int ttlSeconds() default 3600;
}
// Spring can call:
// method.getAnnotation(Cacheable.class).ttlSeconds()
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using interface. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* `CLASS` retention is the rare middle
ground: useful for bytecode instrumentation tools that process .class
files without loading them (javap, ASM, ProGuard, R8). These tools read
the bytecode file directly, not through JVM class loading. ProGuard uses
CLASS retention annotations to identify methods to preserve. If you're
writing a bytecode agent that processes .class files at build time (not
runtime), CLASS retention adds metadata without JVM overhead.

---

**Q3 (Custom annotation): How do you create and use a custom annotation?**

A:
```java
// Step 1: Define the annotation
@Retention(RetentionPolicy.RUNTIME) // required for runtime processing
@Target({ElementType.METHOD, ElementType.TYPE})
@Documented                         // optional: appear in Javadoc
public @interface Audit {
    // Elements (like methods in an interface, but annotation elements):
    String action() default "";         // optional (has default)
    AuditLevel level() default AuditLevel.INFO; // enum element
    boolean logParams() default false;  // boolean element
    Class<?>[] excludeTypes() default {}; // array element

    // No "value()" -> must always use named elements
    // @Audit(action="CREATE", level=AuditLevel.WARN)
}
// If only one element named "value()": @Audit("CREATE") works

// Step 2: Use the annotation
@Service
class OrderService {
    @Audit(action="CREATE_ORDER", logParams=true)
    public Order createOrder(OrderRequest request) { ... }

    @Audit(action="CANCEL_ORDER", level=AuditLevel.WARN)
    public void cancelOrder(Long orderId) { ... }
}

// Step 3: Process the annotation (AOP + Spring)
@Aspect
@Component
class AuditAspect {
    @Around("@annotation(audit)")
    public Object log(ProceedingJoinPoint pjp, Audit audit)
            throws Throwable {
        log.info("AUDIT: action={} level={}", audit.action(), audit.level());
        if (audit.logParams()) {
            log.info("PARAMS: {}", Arrays.toString(pjp.getArgs()));
        }
        try {
            Object result = pjp.proceed();
            log.info("AUDIT: action={} SUCCESS", audit.action());
            return result;
        } catch (Exception e) {
            log.warn("AUDIT: action={} FAILED: {}", audit.action(), e.getMessage());
            throw e;
        }
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates null-safe value wrapping using Spring annotation. **KEY MECHANISM:** Optional.of() throws NPE on null; Optional.ofNullable() wraps null safely. **WHY IT MATTERS:** calling get() without isPresent() check produces NoSuchElementException. **TAKEAWAY: prefer orElseThrow() with a meaningful message over bare get().**

*What separates good from great:* Custom annotations work best when they
express business intent cleanly. `@Audit(action="CREATE_ORDER")` is more
readable than manually injecting an AuditLogger into every method.
The annotation becomes the "what", the AOP aspect the "how". This separation
lets you: change the audit implementation (log to DB, to Kafka, to external
system) without touching any service code. Add auditing to new methods
with one annotation. Remove auditing from methods by removing the annotation.
Test audit behavior independently of business logic.

---

**Q4 (@Repeatable): How does @Repeatable work?**

A: `@Repeatable` (Java 8) allows the same annotation to appear multiple
times on the same element. Requires a "container annotation":

```java
// Container annotation (holds array of the repeatable annotation):
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Schedules {
    Schedule[] value(); // must have value() returning array of Schedule
}

// Repeatable annotation:
@Repeatable(Schedules.class) // points to container
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Schedule {
    String cron();
    String timezone() default "UTC";
}

// Usage: multiple @Schedule on same method
@Schedule(cron = "0 0 9 * * MON-FRI")            // 9am weekdays
@Schedule(cron = "0 0 12 * * SAT", timezone="EST") // noon Saturday EST
public void sendReport() { ... }

// Reading: must use getAnnotationsByType (not getAnnotation) for repeatables:
Schedule[] schedules = method.getAnnotationsByType(Schedule.class);
for (Schedule s : schedules) {
    System.out.println(s.cron() + " @ " + s.timezone());
}
// getAnnotation(Schedule.class) returns null if multiple applied!
// (It finds Schedules container, not Schedule)

// Or: getAnnotation(Schedules.class) returns the container
Schedules container = method.getAnnotation(Schedules.class);
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using container. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* `@Repeatable` solves the "wrapper array
annotation" boilerplate. Before Java 8: `@Schedules({@Schedule("cron1"), @Schedule("cron2")})`.
After: just use `@Schedule` twice. The key trap: reading with `getAnnotation()`
vs `getAnnotationsByType()`. When an element has multiple `@Schedule`,
the compiler wraps them in `@Schedules` container. `getAnnotation(Schedule.class)`
looks for a bare `Schedule` - finds nothing (only the container exists).
`getAnnotationsByType(Schedule.class)` unwraps the container and returns
all `Schedule` instances. Always use `getAnnotationsByType` for `@Repeatable` annotations.

---

**Q5 (APT vs runtime): Trade-offs between APT and runtime reflection for
annotation processing.**

A:

| Aspect | Annotation Processing (APT) | Runtime Reflection |
|---|---|---|
| When | Compile time | Runtime (startup or call time) |
| Cost | Build time | Startup time or per-call |
| Source generation | Yes (Lombok, MapStruct) | No |
| Error detection | Compile errors | Runtime exceptions |
| Framework examples | Lombok, MapStruct, Dagger | Spring, Hibernate, JUnit |
| JVM module access | Not needed | Needs --add-opens in Java 9+ |
| Serverless cold start | Fast (no scanning) | Slow (classpath scanning) |
| Debugging | Hard (generated code) | Easy (step through reflective code) |

```java
// MapStruct (APT): generates implementation at compile time
@Mapper
interface UserMapper {
    UserDTO toDto(User user);
    User toEntity(UserDTO dto);
}
// Compile: generates UserMapperImpl.java
// Runtime: no reflection needed - calls generated code directly

// Spring (runtime): reads annotations at startup
@Service
class UserService {
    @Autowired UserRepository repo; // injected at startup via reflection
}
// Startup: Spring scans classpath, reads @Service, @Autowired, wires beans
// No code generation; flexible but slower to start

// Quarkus (hybrid): processes at compile time, runs AOT
// @Inject fields: resolved at build time, fast startup
// This is why Quarkus starts in 50ms vs Spring's 2000ms+
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using Spring annotation. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* The performance difference matters at
scale. In a microservices architecture with 50 services, Spring's runtime
scanning adds 1-3 seconds to each deployment startup. For Kubernetes with
rolling updates across 10 pods: 50-150 extra seconds of startup across
the fleet. Quarkus/Micronaut's compile-time processing: 100-500ms startup
because there's no classpath scanning or proxy generation. The trade-off:
APT frameworks are less flexible (no dynamic bean registration) but
dramatically faster. Choose based on startup time requirements.

---

**Q6 (Framework annotation pitfalls): Common annotation misuse with frameworks.**

A:
```java
// PITFALL 1: @Transactional on private methods (Spring CGLIB ignored)
@Service
class OrderService {
    public void processOrder(Order order) {
        saveOrderInternal(order); // calls self - bypasses proxy!
    }

    @Transactional // IGNORED: Spring proxy can't intercept self-calls
    private void saveOrderInternal(Order order) {
        repo.save(order);
    }
    // Fix: extract to separate Spring bean, or use AspectJ weaving
}

// PITFALL 2: @Cacheable on method called from within same class
@Service
class ProductService {
    public Product getWithRelations(Long id) {
        Product p = findById(id); // bypasses @Cacheable proxy!
        // ...
    }

    @Cacheable("products")
    public Product findById(Long id) { return repo.findById(id); }
    // Fix: inject self: @Autowired ProductService self; self.findById(id);
    // Or: AopContext.currentProxy() (requires exposeProxy=true)
}

// PITFALL 3: @Async on same class method call (same proxy issue)
// All three pitfalls share the same root cause: Spring's CGLIB/JDK proxy
// wraps the bean; direct calls within the class bypass the proxy

// PITFALL 4: Bean Validation on service layer (controller validation not propagated)
@Service
class UserService {
    public void createUser(@Valid UserDTO dto) { // @Valid has no effect without a trigger!
        // @Valid requires a Spring method validation interceptor
    }
    // Fix: @Validated on class + @Valid on params
    // OR: validate in controller (auto-applied with @RequestBody @Valid)
}

@Service
@Validated // enables method-level validation in Spring
class UserService {
    public void createUser(@Valid UserDTO dto) { // NOW it works
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

*What separates good from great:* The "same-class method call bypasses AOP
proxy" is one of the most common Spring bugs. It's invisible in unit tests
(no proxy) and only manifests in integration tests. The spring documentation
mentions it, but developers often miss it. Production symptoms: transactions
not committed, caches not populated, methods not running async. The root
cause is always the same: calling `this.method()` goes directly to the
target object, not through the proxy wrapper. Solutions: (1) separate
bean, (2) self-injection, (3) AspectJ full weaving (works for all cases
including `final` methods and `private` methods).

---

**Q7 (Annotation inheritance): How does annotation inheritance work in Java?**

A:
- `@Inherited` on an annotation meta-type: class-level annotations on a
  superclass are inherited by subclasses.
- Interface annotations: NOT inherited by implementing classes (even with `@Inherited`).
- Method annotations: NOT inherited when overriding.

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface CategoryMarker { String value(); }

@CategoryMarker("service")
class BaseService {}

class UserService extends BaseService {}
// UserService.class.getAnnotation(CategoryMarker.class) != null (inherited!)

// But interfaces:
@CategoryMarker("api")
interface UserApi {}

class UserApiImpl implements UserApi {}
// UserApiImpl.class.getAnnotation(CategoryMarker.class) == null
// Interface annotations NOT inherited even with @Inherited!

// Method annotations: NOT inherited
class Base {
    @Transactional
    public void save() {}
}
class Child extends Base {
    @Override
    public void save() {} // @Transactional NOT present here!
    // Spring will NOT apply transaction to Child.save() calls!
    // Fix: re-annotate in Child, or put @Transactional on Child class
}
```

> **Code walkthrough:** This Unknown example demonstrates Spring declarative transaction using @Transactional. **KEY MECHANISM:** Spring wraps the method in a proxy that begins/commits a DB transaction. **WHY IT MATTERS:** calling @Transactional from the same class bypasses the proxy - no transaction. **TAKEAWAY: never self-invoke @Transactional methods; inject the bean instead.**

*What separates good from great:* Spring has a meta-annotation search
(`AnnotationUtils.findAnnotation()`) that walks the class and interface
hierarchy looking for annotations. `method.getAnnotation()` only checks
the specific method. `AnnotationUtils.findAnnotation(method, Transactional.class)`
checks the method, then superclass methods, then interfaces. This is why
Spring's `@Transactional` on an interface SOMETIMES works with JDK proxy
(which creates a proxy for the interface): Spring finds the annotation
via interface hierarchy search. But with CGLIB proxy (subclass-based),
interface annotations are not visible to the proxy lookup. Knowing this
distinction separates engineers who debug Spring transaction issues from
those who cargo-cult annotations.

---

**Q8 (Bean Validation): How do Bean Validation annotations work?**

A: Bean Validation (JSR 380, Hibernate Validator) provides annotations for
field-level constraints. Spring MVC automatically triggers validation when
`@Valid` or `@Validated` is on a controller method parameter.

```java
// Constraint annotations:
class UserRegistrationRequest {
    @NotNull @Size(min=2, max=50)
    private String name;

    @NotBlank @Email
    private String email;

    @NotNull @Size(min=8, max=100)
    @Pattern(regexp="(?=.*\\d)(?=.*[a-z])(?=.*[A-Z]).+",
             message="Must contain uppercase, lowercase, digit")
    private String password;

    @Min(18) @Max(120)
    private Integer age;

    @Past // date in the past
    private LocalDate birthDate;
}

// In controller: triggers validation, 400 on failure
@PostMapping("/users")
ResponseEntity<UserDTO> register(@Valid @RequestBody UserRegistrationRequest req) { ... }

// Custom validator:
@Constraint(validatedBy = UniqueEmailValidator.class)
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UniqueEmail {
    String message() default "Email already registered";
    Class<?>[] groups() default {};  // required by Bean Validation spec
    Class<? extends Payload>[] payload() default {}; // required
}

class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    @Autowired UserRepository repo; // Spring injects it!
    @Override
    public boolean isValid(String email, ConstraintValidatorContext ctx) {
        return email == null || !repo.existsByEmail(email);
    }
}
```

> **Code walkthrough:** This Unknown example demonstrates contract definition using Spring annotation. **KEY MECHANISM:** the JVM uses dynamic dispatch for all interface method calls. **WHY IT MATTERS:** interfaces with default methods can conflict at compile time via diamond problem. **TAKEAWAY: interfaces define contracts; prefer them over abstract classes for unrelated types.**

*What separates good from great:* Custom `ConstraintValidator` beans can be
Spring-managed (injection works!). This enables database-backed validation
(`@UniqueEmail` checking the DB) in the validation layer. The `groups()`
attribute enables validation groups: different validation rules for create
vs update (`@NotNull(groups=Create.class)`). `@Valid` triggers recursive
validation of nested objects; `@Validated` (Spring) enables group selection.
Validation failure throws `MethodArgumentNotValidException` in controllers
(auto-handled by Spring's default exception handler: 400 with field errors).

---

**Q9 (Security risks): What are the security risks of annotation-based
runtime reflection?**

A:
1. **Annotation data injection:** annotation values (strings) come from
   code, but custom annotation processors may pass them to SQL/OS commands.
   ```java
   @Table(name = userInput) // annotation values are compile-time constants
   // Actually safe: annotation values must be compile-time constants
   // Real risk: annotation processor that reads other config files
   ```

> **Code walkthrough:** This Unknown example demonstrates metadata declaration. **KEY MECHANISM:** annotations are processed at compile-time or runtime via reflection. **WHY IT MATTERS:** annotation processing adds compile time; runtime reflection disables JIT optimizations. **TAKEAWAY: prefer compile-time annotation processors (APT) over runtime reflection for performance.**

2. **Reflection bypass of access controls:** `field.setAccessible(true)` bypasses `private`.
   ```java
   // Malicious code (if allowed to run):
   Field secretField = Config.class.getDeclaredField("apiKey");
   secretField.setAccessible(true); // bypasses private!
   String apiKey = (String) secretField.get(config);
   // Java 9 module system prevents this for JDK classes without --add-opens
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

3. **Deserialization of annotated types (Jackson):**
   ```java
   // @JsonTypeInfo allows polymorphic deserialization
   // Without @JsonTypeInfo restrictions: arbitrary class instantiation!
   // CVE-2017-7525: Jackson deserializing malicious JSON could execute code
   // Fix: use @JsonTypeInfo with @JsonSubTypes (whitelist approach)
   // Or: disable default typing (objectMapper.disableDefaultTyping())
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

4. **Spring SpEL injection in annotation values:**
   ```java
   // @PreAuthorize, @Value can execute SpEL expressions
   // If user input reaches SpEL expression: Remote Code Execution!
   @PreAuthorize("hasRole('" + userInput + "')") // NEVER do this!
   // SpEL evaluates: #{T(java.lang.Runtime).getRuntime().exec('...')}
   ```

> **Code walkthrough:** This Unknown example demonstrates Java API usage using authentication. **KEY MECHANISM:** the JVM compiles to bytecode that runs on the JVM; JIT compiles hot paths to native. **WHY IT MATTERS:** unchecked assumptions about thread safety cause data races under concurrent load. **TAKEAWAY: document thread-safety guarantees on every shared mutable class.**

*What separates good from great:* The Jackson polymorphic deserialization
vulnerability (CVE-2017-7525 and related CVEs) was among the most severe
Java security issues in 2017-2019. The root cause: `@JsonTypeInfo` with
`Id.CLASS` allows the JSON input to specify which Java class to instantiate.
An attacker sends a JSON with a "gadget class" (JNDI lookup, Spring EL
evaluation) as the type. Mitigation: use `@JsonSubTypes` whitelist,
or `MapperFeature.ALLOW_COERCION_OF_SCALARS`, or Jackson's default typing
allowlist. Any annotation that executes code or creates objects from
runtime input is a security boundary.

---

### ⚖️ Comparison Table

| Annotation Processing Model | Timing | Error Detection | Flexibility | Examples |
|---|---|---|---|---|
| APT (JSR 269) | Compile time | Compile errors | Low (static) | Lombok, MapStruct |
| Runtime reflection | JVM startup | Runtime exceptions | High (dynamic) | Spring, Hibernate |
| Bytecode instrumentation | Load time | Runtime | Medium | AspectJ, Byte Buddy |
| Hybrid (AOT) | Build + run | Both | Medium | Quarkus, Micronaut |

---
