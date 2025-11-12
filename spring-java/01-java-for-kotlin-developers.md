# Java Fundamentals for Kotlin Developers

This guide helps you understand Java by comparing it with Kotlin concepts you already know.

## 🎯 Overview

Java and Kotlin are both JVM languages, so they share many similarities. Kotlin was designed to improve upon Java, so many Kotlin features are more concise versions of Java patterns.

## 📋 Quick Comparison Table

| Feature | Kotlin | Java |
|---------|--------|------|
| Null Safety | Built-in (`String?`) | Optional (since Java 8) |
| Type Inference | `val name = "John"` | `String name = "John";` |
| Data Classes | `data class User(val name: String)` | Requires full class with getters/setters |
| Extension Functions | Built-in | Not available (use utility classes) |
| Coroutines | Built-in | CompletableFuture, Virtual Threads |
| Properties | Built-in | Getter/Setter methods |
| Semicolons | Optional | Required |
| Constructor | Primary + secondary | Multiple constructors |
| Functional Programming | First-class | Improved since Java 8 |

## 1. Basic Syntax

### Variable Declaration

**Kotlin:**
```kotlin
val immutable = "Cannot change"  // final in Java
var mutable = "Can change"
```

**Java:**
```java
final String immutable = "Cannot change";
String mutable = "Can change";
```

### Type Inference

**Kotlin:**
```kotlin
val number = 42              // Int inferred
val message = "Hello"        // String inferred
val list = listOf(1, 2, 3)  // List<Int> inferred
```

**Java (10+):**
```java
var number = 42;              // int inferred
var message = "Hello";        // String inferred
var list = List.of(1, 2, 3); // List<Integer> inferred
```

### String Interpolation

**Kotlin:**
```kotlin
val name = "John"
val age = 25
val message = "My name is $name and I'm $age years old"
val expression = "Next year I'll be ${age + 1}"
```

**Java:**
```java
String name = "John";
int age = 25;

// Traditional concatenation
String message = "My name is " + name + " and I'm " + age + " years old";

// String.format()
String formatted = String.format("My name is %s and I'm %d years old", name, age);

// Text Blocks (Java 15+)
String textBlock = """
    My name is %s and I'm %d years old
    """.formatted(name, age);
```

## 2. Functions and Methods

### Function Declaration

**Kotlin:**
```kotlin
fun greet(name: String): String {
    return "Hello, $name"
}

// Single expression function
fun greet(name: String) = "Hello, $name"

// Default parameters
fun greet(name: String = "Guest") = "Hello, $name"

// Named parameters
greet(name = "John")
```

**Java:**
```java
public String greet(String name) {
    return "Hello, " + name;
}

// No single expression syntax

// Default parameters not supported - use method overloading
public String greet(String name) {
    return "Hello, " + name;
}

public String greet() {
    return greet("Guest");
}

// No named parameters
greet("John");
```

## 3. Classes and Objects

### Simple Class

**Kotlin:**
```kotlin
class Person(val name: String, var age: Int) {
    fun introduce() = "I'm $name, $age years old"
}

val person = Person("John", 25)
println(person.name)  // Direct property access
```

**Java:**
```java
public class Person {
    private final String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String introduce() {
        return "I'm " + name + ", " + age + " years old";
    }
}

Person person = new Person("John", 25);
System.out.println(person.getName());  // Use getter method
```

### Data Class

**Kotlin:**
```kotlin
data class User(
    val id: Long,
    val name: String,
    val email: String
)

// Automatically generates: equals(), hashCode(), toString(), copy()
val user1 = User(1, "John", "john@example.com")
val user2 = user1.copy(name = "Jane")
```

**Java (Record - Java 14+):**
```java
public record User(Long id, String name, String email) {
    // Automatically generates: constructor, getters, equals(), hashCode(), toString()
}

User user1 = new User(1L, "John", "john@example.com");
// No copy() method - must create new instance
User user2 = new User(user1.id(), "Jane", user1.email());
```

**Java (Traditional):**
```java
public class User {
    private final Long id;
    private final String name;
    private final String email;

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Getters
    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }

    // equals(), hashCode(), toString() - must implement manually
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(id, user.id) &&
               Objects.equals(name, user.name) &&
               Objects.equals(email, user.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, email);
    }

    @Override
    public String toString() {
        return "User{id=" + id + ", name='" + name + "', email='" + email + "'}";
    }
}
```

**💡 Use Lombok to reduce boilerplate:**
```java
import lombok.Data;
import lombok.AllArgsConstructor;

@Data
@AllArgsConstructor
public class User {
    private final Long id;
    private final String name;
    private final String email;
}
```

## 4. Null Safety

### Nullable Types

**Kotlin:**
```kotlin
var nullableName: String? = null
var nonNullName: String = "John"

// Safe call
val length = nullableName?.length

// Elvis operator
val length = nullableName?.length ?: 0

// Safe cast
val person = obj as? Person

// Not-null assertion (avoid!)
val length = nullableName!!.length
```

**Java (Optional - Java 8+):**
```java
String nullableName = null;
String nonNullName = "John";

// Optional
Optional<String> optionalName = Optional.ofNullable(nullableName);

// Safe access
int length = optionalName.map(String::length).orElse(0);

// Or traditional null check
if (nullableName != null) {
    int len = nullableName.length();
}

// Annotations (for static analysis)
@Nullable String nullableName;
@NonNull String nonNullName;
```

## 5. Collections

### Lists

**Kotlin:**
```kotlin
// Immutable
val immutableList = listOf(1, 2, 3)

// Mutable
val mutableList = mutableListOf(1, 2, 3)
mutableList.add(4)

// Array
val array = arrayOf(1, 2, 3)
```

**Java:**
```java
// Immutable (Java 9+)
List<Integer> immutableList = List.of(1, 2, 3);

// Mutable
List<Integer> mutableList = new ArrayList<>(List.of(1, 2, 3));
mutableList.add(4);

// Array
Integer[] array = {1, 2, 3};
// or
Integer[] array = new Integer[]{1, 2, 3};
```

### Maps

**Kotlin:**
```kotlin
val map = mapOf(
    "key1" to "value1",
    "key2" to "value2"
)

val mutableMap = mutableMapOf<String, String>()
mutableMap["key1"] = "value1"

// Access
val value = map["key1"]
```

**Java:**
```java
// Immutable (Java 9+)
Map<String, String> map = Map.of(
    "key1", "value1",
    "key2", "value2"
);

// Mutable
Map<String, String> mutableMap = new HashMap<>();
mutableMap.put("key1", "value1");

// Access
String value = map.get("key1");
```

### Collection Operations

**Kotlin:**
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

val doubled = numbers.map { it * 2 }
val evens = numbers.filter { it % 2 == 0 }
val sum = numbers.sum()
val first = numbers.firstOrNull { it > 3 }
```

**Java (Streams - Java 8+):**
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

List<Integer> doubled = numbers.stream()
    .map(n -> n * 2)
    .collect(Collectors.toList());

List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

Optional<Integer> first = numbers.stream()
    .filter(n -> n > 3)
    .findFirst();
```

## 6. Lambda Expressions and Functional Programming

**Kotlin:**
```kotlin
// Lambda
val sum = { a: Int, b: Int -> a + b }
println(sum(2, 3))  // 5

// Function type
val operation: (Int, Int) -> Int = { a, b -> a + b }

// Higher-order function
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

calculate(5, 3) { a, b -> a + b }  // 8
```

**Java:**
```java
// Lambda (Java 8+)
BiFunction<Integer, Integer, Integer> sum = (a, b) -> a + b;
System.out.println(sum.apply(2, 3));  // 5

// Method reference
BiFunction<Integer, Integer, Integer> sum = Integer::sum;

// Functional interface
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// Using functional interface
public int calculate(int a, int b, Calculator calculator) {
    return calculator.calculate(a, b);
}

calculate(5, 3, (a, b) -> a + b);  // 8
```

### Common Functional Interfaces

| Kotlin | Java Equivalent | Description |
|--------|-----------------|-------------|
| `() -> R` | `Supplier<R>` | Takes nothing, returns R |
| `(T) -> R` | `Function<T, R>` | Takes T, returns R |
| `(T) -> Boolean` | `Predicate<T>` | Takes T, returns boolean |
| `(T) -> Unit` | `Consumer<T>` | Takes T, returns nothing |
| `(T, U) -> R` | `BiFunction<T, U, R>` | Takes T and U, returns R |

## 7. Control Flow

### When vs Switch

**Kotlin:**
```kotlin
when (value) {
    1 -> println("One")
    2, 3 -> println("Two or Three")
    in 4..10 -> println("Between 4 and 10")
    else -> println("Something else")
}

// As expression
val result = when (value) {
    1 -> "One"
    2 -> "Two"
    else -> "Other"
}

// Without argument
when {
    x > 10 -> println("Greater than 10")
    x > 5 -> println("Greater than 5")
    else -> println("5 or less")
}
```

**Java (Enhanced Switch - Java 14+):**
```java
switch (value) {
    case 1 -> System.out.println("One");
    case 2, 3 -> System.out.println("Two or Three");
    default -> System.out.println("Something else");
}

// As expression
String result = switch (value) {
    case 1 -> "One";
    case 2 -> "Two";
    default -> "Other";
};

// Traditional switch
switch (value) {
    case 1:
        System.out.println("One");
        break;
    case 2:
    case 3:
        System.out.println("Two or Three");
        break;
    default:
        System.out.println("Something else");
}
```

### For Loops

**Kotlin:**
```kotlin
// Range
for (i in 1..5) {
    println(i)
}

// Until
for (i in 1 until 5) {
    println(i)
}

// Step
for (i in 1..10 step 2) {
    println(i)
}

// Collection
for (item in list) {
    println(item)
}

// With index
for ((index, item) in list.withIndex()) {
    println("$index: $item")
}
```

**Java:**
```java
// Range
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}

// Collection (enhanced for)
for (String item : list) {
    System.out.println(item);
}

// With index (traditional)
for (int i = 0; i < list.size(); i++) {
    System.out.println(i + ": " + list.get(i));
}

// Stream with index (Java 8+)
IntStream.range(0, list.size())
    .forEach(i -> System.out.println(i + ": " + list.get(i)));
```

## 8. Exception Handling

**Kotlin:**
```kotlin
// No checked exceptions!
fun readFile(path: String): String {
    return File(path).readText()  // No need to declare throws
}

try {
    val content = readFile("file.txt")
} catch (e: IOException) {
    println("Error: ${e.message}")
} finally {
    println("Done")
}

// Try as expression
val content = try {
    readFile("file.txt")
} catch (e: IOException) {
    "Default content"
}
```

**Java:**
```java
// Checked exceptions must be declared
public String readFile(String path) throws IOException {
    return Files.readString(Path.of(path));
}

try {
    String content = readFile("file.txt");
} catch (IOException e) {
    System.out.println("Error: " + e.getMessage());
} finally {
    System.out.println("Done");
}

// Try-with-resources (Java 7+)
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line = reader.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
```

## 9. Async Programming

### Kotlin Coroutines vs Java CompletableFuture

**Kotlin:**
```kotlin
// Coroutine
suspend fun fetchUser(id: Long): User {
    delay(1000)  // Suspends without blocking
    return userRepository.findById(id)
}

// Usage
lifecycleScope.launch {
    val user = fetchUser(1)
    println(user.name)
}

// Sequential
suspend fun getFullProfile() {
    val user = fetchUser(1)
    val posts = fetchPosts(user.id)
    return Profile(user, posts)
}

// Parallel
suspend fun getFullProfile() {
    coroutineScope {
        val userDeferred = async { fetchUser(1) }
        val postsDeferred = async { fetchPosts(1) }

        Profile(userDeferred.await(), postsDeferred.await())
    }
}
```

**Java (CompletableFuture - Java 8+):**
```java
// CompletableFuture
public CompletableFuture<User> fetchUser(Long id) {
    return CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return userRepository.findById(id);
    });
}

// Usage
fetchUser(1L).thenAccept(user -> {
    System.out.println(user.getName());
});

// Sequential
public CompletableFuture<Profile> getFullProfile() {
    return fetchUser(1L)
        .thenCompose(user -> fetchPosts(user.getId())
            .thenApply(posts -> new Profile(user, posts)));
}

// Parallel
public CompletableFuture<Profile> getFullProfile() {
    CompletableFuture<User> userFuture = fetchUser(1L);
    CompletableFuture<List<Post>> postsFuture = fetchPosts(1L);

    return userFuture.thenCombine(postsFuture, Profile::new);
}
```

**Java (Virtual Threads - Java 21+):**
```java
// Virtual threads - similar to coroutines
public User fetchUser(Long id) throws InterruptedException {
    Thread.sleep(1000);  // Doesn't block OS thread!
    return userRepository.findById(id);
}

// Usage with virtual thread
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        try {
            User user = fetchUser(1L);
            System.out.println(user.getName());
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    });
}
```

## 10. Annotations

**Kotlin:**
```kotlin
@Entity
@Table(name = "users")
data class User(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @Column(nullable = false)
    val name: String
)
```

**Java:**
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    // Constructors, getters, setters...
}
```

## 11. Sealed Classes vs Enums

**Kotlin (Sealed Class):**
```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Exception) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

fun handle(result: Result<User>) {
    when (result) {
        is Result.Success -> println(result.data)
        is Result.Error -> println(result.exception)
        Result.Loading -> println("Loading...")
    }
}
```

**Java (Sealed Class - Java 17+):**
```java
public sealed interface Result<T> permits Success, Error, Loading {

    record Success<T>(T data) implements Result<T> {}

    record Error(Exception exception) implements Result {
        @Override
        public Object data() { return null; }
    }

    final class Loading implements Result {
        private static final Loading INSTANCE = new Loading();
        private Loading() {}
        public static Loading getInstance() { return INSTANCE; }
    }
}

// Usage with pattern matching (Java 17+)
public void handle(Result<User> result) {
    switch (result) {
        case Success<User> s -> System.out.println(s.data());
        case Error e -> System.out.println(e.exception());
        case Loading l -> System.out.println("Loading...");
    }
}
```

## 📊 Summary Diagram

```
Kotlin Feature              Java Equivalent                      Since
├─ val/var                  final/non-final                      Always
├─ Type inference           var (limited)                        Java 10
├─ String templates         String.format() or concat            Always
├─ Data class               Record or Lombok                     Java 14 / Library
├─ Null safety              Optional                             Java 8
├─ Extension functions      Static utility methods               Always
├─ Higher-order functions   Functional interfaces                Java 8
├─ Coroutines               CompletableFuture/Virtual Threads    Java 8 / Java 21
├─ when expression          Switch expressions                   Java 14
├─ Sealed classes           Sealed classes                       Java 17
├─ Smart casts              Pattern matching                     Java 16+
└─ Properties               Getter/Setter methods                Always
```

## 🎯 Key Takeaways

1. **Java is more verbose** - requires more boilerplate code
2. **Null safety** - Java relies on Optional or annotations, Kotlin has it built-in
3. **Properties** - Java uses getters/setters, Kotlin has direct property access
4. **Async** - Kotlin coroutines are more intuitive than CompletableFuture
5. **Modern Java** - Java 17+ has caught up with many Kotlin features
6. **Interoperability** - Both compile to JVM bytecode and work together seamlessly

## 🚀 Next Steps

Now that you understand Java basics, let's dive into Spring Framework and Dependency Injection!

Continue to [Section 2: Spring Core & Dependency Injection](./02-spring-core-di.md)
