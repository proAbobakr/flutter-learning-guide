# Section 1: Dart for Kotlin Developers

## Introduction

Dart and Kotlin share many similarities, making the transition smooth for Android developers. Both are modern, object-oriented languages with strong type systems and support for functional programming concepts.

## Key Similarities

- ✅ Strong type inference
- ✅ Null safety
- ✅ Modern syntax
- ✅ First-class functions
- ✅ Async/await pattern
- ✅ Extension methods
- ✅ Collection operators

## Syntax Comparison

### 1. Variable Declaration

**Kotlin:**
```kotlin
// Immutable
val name: String = "John"
val age = 25  // Type inference

// Mutable
var count: Int = 0
var city = "New York"

// Late initialization
lateinit var user: User

// Lazy initialization
val database by lazy { Database.getInstance() }
```

**Dart:**
```dart
// Immutable
final String name = "John";
final age = 25;  // Type inference

// Mutable
int count = 0;
var city = "New York";

// Late initialization
late User user;

// Lazy initialization (using getter)
Database? _database;
Database get database => _database ??= Database.getInstance();
```

### 2. Null Safety

**Kotlin:**
```kotlin
// Nullable type
var name: String? = null

// Safe call
val length = name?.length

// Elvis operator
val len = name?.length ?: 0

// Non-null assertion
val definiteLength = name!!.length

// Safe cast
val user = obj as? User
```

**Dart:**
```dart
// Nullable type
String? name = null;

// Safe call
final length = name?.length;

// Null coalescing
final len = name?.length ?? 0;

// Non-null assertion
final definiteLength = name!.length;

// Safe cast
final user = obj as User?;
```

### 3. Functions

**Kotlin:**
```kotlin
// Regular function
fun greet(name: String): String {
    return "Hello, $name"
}

// Expression body
fun add(a: Int, b: Int) = a + b

// Named parameters
fun createUser(
    name: String,
    age: Int = 0,
    email: String? = null
) { }

// Lambda
val multiply = { a: Int, b: Int -> a * b }

// Higher-order function
fun processNumbers(numbers: List<Int>, operation: (Int) -> Int): List<Int> {
    return numbers.map(operation)
}
```

**Dart:**
```dart
// Regular function
String greet(String name) {
  return "Hello, $name";
}

// Expression body
int add(int a, int b) => a + b;

// Named parameters (with required annotation)
void createUser({
  required String name,
  int age = 0,
  String? email,
}) { }

// Anonymous function
final multiply = (int a, int b) => a * b;

// Higher-order function
List<int> processNumbers(List<int> numbers, int Function(int) operation) {
  return numbers.map(operation).toList();
}
```

### 4. Classes

**Kotlin:**
```kotlin
// Data class
data class User(
    val id: Int,
    val name: String,
    val email: String?
)

// Regular class with constructor
class Person(
    val name: String,
    var age: Int
) {
    // Secondary constructor
    constructor(name: String) : this(name, 0)

    // Method
    fun greet() = "Hello, I'm $name"

    // Computed property
    val isAdult: Boolean
        get() = age >= 18
}

// Singleton
object DatabaseConfig {
    const val DB_NAME = "mydb"
}

// Companion object
class User {
    companion object {
        fun create(name: String) = User(name)
    }
}
```

**Dart:**
```dart
// Class (no data class, but you can use packages like freezed)
class User {
  final int id;
  final String name;
  final String? email;

  User(this.id, this.name, this.email);

  // Named constructor
  User.guest() : id = 0, name = "Guest", email = null;

  // Equality (override manually or use packages)
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is User && id == other.id;

  @override
  int get hashCode => id.hashCode;
}

// Regular class
class Person {
  final String name;
  int age;

  Person(this.name, this.age);

  // Named constructor
  Person.withName(this.name) : age = 0;

  // Method
  String greet() => "Hello, I'm $name";

  // Computed property (getter)
  bool get isAdult => age >= 18;
}

// Static class members
class DatabaseConfig {
  static const String dbName = "mydb";
}

// Factory constructor (similar to companion object)
class User {
  static User create(String name) => User(name);

  // Or using factory
  factory User.createFactory(String name) {
    return User(name);
  }
}
```

### 5. Interfaces and Abstract Classes

**Kotlin:**
```kotlin
// Interface
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")  // Default implementation
}

// Abstract class
abstract class Vehicle {
    abstract val maxSpeed: Int
    abstract fun drive()

    fun stop() {
        println("Stopping")
    }
}

// Implementation
class Car : Vehicle(), Clickable {
    override val maxSpeed = 200

    override fun drive() {
        println("Driving")
    }

    override fun click() {
        println("Car clicked")
    }
}
```

**Dart:**
```dart
// Abstract class (Dart doesn't have explicit interfaces)
abstract class Clickable {
  void click();

  // Default implementation
  void showOff() {
    print("I'm clickable!");
  }
}

// Abstract class
abstract class Vehicle {
  int get maxSpeed;
  void drive();

  void stop() {
    print("Stopping");
  }
}

// Implementation (use 'implements' for interface-like behavior)
class Car extends Vehicle implements Clickable {
  @override
  int get maxSpeed => 200;

  @override
  void drive() {
    print("Driving");
  }

  @override
  void click() {
    print("Car clicked");
  }

  @override
  void showOff() {
    print("I'm clickable!");
  }
}

// You can also implement any class as an interface
class Button {
  void press() {}
}

class CustomButton implements Button {
  @override
  void press() {
    print("Custom press");
  }
}
```

### 6. Collections

**Kotlin:**
```kotlin
// List
val immutableList = listOf(1, 2, 3)
val mutableList = mutableListOf(1, 2, 3)

// Map
val map = mapOf("a" to 1, "b" to 2)
val mutableMap = mutableMapOf("a" to 1)

// Set
val set = setOf(1, 2, 3)

// Operations
val doubled = immutableList.map { it * 2 }
val filtered = immutableList.filter { it > 1 }
val sum = immutableList.reduce { acc, i -> acc + i }
```

**Dart:**
```dart
// List
final immutableList = const [1, 2, 3];  // Compile-time constant
final list = [1, 2, 3];  // Can't reassign but elements can change
final List<int> mutableList = [1, 2, 3];

// Map
final map = const {"a": 1, "b": 2};
final mutableMap = {"a": 1, "b": 2};

// Set
final set = const {1, 2, 3};

// Operations
final doubled = list.map((e) => e * 2).toList();
final filtered = list.where((e) => e > 1).toList();
final sum = list.reduce((acc, i) => acc + i);

// Spread operator
final combined = [...list, 4, 5];
```

### 7. Async Programming

**Kotlin:**
```kotlin
// Suspend function
suspend fun fetchUser(id: Int): User {
    delay(1000)
    return User(id, "John")
}

// Coroutine scope
fun loadData() {
    lifecycleScope.launch {
        try {
            val user = fetchUser(1)
            // Update UI
        } catch (e: Exception) {
            // Handle error
        }
    }
}

// Async/await
suspend fun getTotalData(): Int {
    val result1 = async { fetchData1() }
    val result2 = async { fetchData2() }
    return result1.await() + result2.await()
}

// Flow
fun userFlow(): Flow<User> = flow {
    while (true) {
        emit(fetchUser())
        delay(1000)
    }
}
```

**Dart:**
```dart
// Async function
Future<User> fetchUser(int id) async {
  await Future.delayed(Duration(seconds: 1));
  return User(id, "John");
}

// Async/await
Future<void> loadData() async {
  try {
    final user = await fetchUser(1);
    // Update UI
  } catch (e) {
    // Handle error
  }
}

// Parallel execution
Future<int> getTotalData() async {
  final results = await Future.wait([
    fetchData1(),
    fetchData2(),
  ]);
  return results[0] + results[1];
}

// Stream (similar to Flow)
Stream<User> userStream() async* {
  while (true) {
    yield await fetchUser();
    await Future.delayed(Duration(seconds: 1));
  }
}

// Listen to stream
userStream().listen((user) {
  print(user.name);
});
```

### 8. Extension Functions

**Kotlin:**
```kotlin
// Extension function
fun String.isEmail(): Boolean {
    return this.contains("@")
}

// Extension property
val String.firstChar: Char
    get() = this[0]

// Usage
val email = "test@example.com"
if (email.isEmail()) {
    println("Valid email")
}
```

**Dart:**
```dart
// Extension
extension StringExtensions on String {
  bool isEmail() {
    return this.contains("@");
  }

  // Extension getter
  String get firstChar => this[0];
}

// Usage
final email = "test@example.com";
if (email.isEmail()) {
  print("Valid email");
}

// You can also extend nullable types
extension NullableStringExtension on String? {
  bool get isNullOrEmpty => this == null || this!.isEmpty;
}
```

### 9. Enum Classes

**Kotlin:**
```kotlin
enum class State {
    IDLE, LOADING, SUCCESS, ERROR
}

// With properties
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}

// Usage
val state = State.LOADING
when (state) {
    State.IDLE -> println("Idle")
    State.LOADING -> println("Loading")
    State.SUCCESS -> println("Success")
    State.ERROR -> println("Error")
}
```

**Dart:**
```dart
// Simple enum
enum State {
  idle,
  loading,
  success,
  error
}

// Enhanced enum (Dart 2.17+)
enum Color {
  red(0xFF0000),
  green(0x00FF00),
  blue(0x0000FF);

  final int rgb;
  const Color(this.rgb);
}

// Usage
final state = State.loading;
switch (state) {
  case State.idle:
    print("Idle");
    break;
  case State.loading:
    print("Loading");
    break;
  case State.success:
    print("Success");
    break;
  case State.error:
    print("Error");
    break;
}
```

### 10. Sealed Classes / Pattern Matching

**Kotlin:**
```kotlin
// Sealed class
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Exception) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

// Usage
fun handleResult(result: Result<User>) {
    when (result) {
        is Result.Success -> println("Data: ${result.data}")
        is Result.Error -> println("Error: ${result.exception.message}")
        Result.Loading -> println("Loading...")
    }
}
```

**Dart:**
```dart
// Sealed class (using freezed package recommended)
// Manual approach:
abstract class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Error<T> extends Result<T> {
  final Exception exception;
  const Error(this.exception);
}

class Loading<T> extends Result<T> {
  const Loading();
}

// Usage
void handleResult(Result<User> result) {
  if (result is Success<User>) {
    print("Data: ${result.data}");
  } else if (result is Error) {
    print("Error: ${result.exception}");
  } else if (result is Loading) {
    print("Loading...");
  }
}

// Better with freezed package (see Section 5)
```

## Important Differences

### 1. Constructor Syntax
- Dart uses `this.parameter` shorthand in constructors
- Dart has named constructors instead of secondary constructors

### 2. Privacy
- Kotlin: `private`, `protected`, `internal`, `public`
- Dart: Only library-level privacy using `_` prefix (e.g., `_privateMethod`)

### 3. Mixins
Dart has powerful mixin support:

```dart
mixin LoggerMixin {
  void log(String message) {
    print('[LOG] $message');
  }
}

class UserService with LoggerMixin {
  void createUser() {
    log('Creating user');
  }
}
```

### 4. Cascade Notation
Dart has cascade notation for chaining operations:

```dart
var paint = Paint()
  ..color = Colors.blue
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;
```

## Best Practices

1. **Use `final` by default**: Like `val` in Kotlin
2. **Embrace null safety**: Always handle nullable types properly
3. **Use `const` for constants**: Compile-time constants improve performance
4. **Leverage type inference**: But be explicit when it improves readability
5. **Use named parameters**: For better API design
6. **Async/await over `.then()`**: More readable asynchronous code

## Common Pitfalls

1. **Forgetting `.toList()`**: Many Dart collection operations return `Iterable`, not `List`
2. **Privacy**: Remember `_` prefix is library-level, not class-level
3. **Equality**: Must override `==` operator manually (or use packages like `equatable`)
4. **Late initialization**: Unlike Kotlin's `lateinit`, Dart's `late` throws at runtime if accessed before initialization

## Practice Exercise

Try converting this Kotlin code to Dart:

```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String?
)

class UserRepository {
    private val users = mutableListOf<User>()

    suspend fun fetchUsers(): List<User> {
        delay(1000)
        return users.toList()
    }

    fun addUser(user: User) {
        users.add(user)
    }
}
```

**Solution:**
```dart
class User {
  final int id;
  final String name;
  final String? email;

  User(this.id, this.name, this.email);

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is User && id == other.id;

  @override
  int get hashCode => id.hashCode;
}

class UserRepository {
  final List<User> _users = [];

  Future<List<User>> fetchUsers() async {
    await Future.delayed(Duration(seconds: 1));
    return List.from(_users);
  }

  void addUser(User user) {
    _users.add(user);
  }
}
```

## Next Steps

Now that you understand Dart basics, let's dive into Flutter project structure in [Section 2: Project Structure](./02-project-structure.md).
