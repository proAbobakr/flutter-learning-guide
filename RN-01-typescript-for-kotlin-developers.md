# TypeScript for Kotlin Developers

## Overview

TypeScript is a statically-typed superset of JavaScript that compiles to plain JavaScript. As a Kotlin developer, you'll find many familiar concepts in TypeScript, though with different syntax and some philosophical differences.

## Type System Comparison

### Basic Types

**Kotlin:**
```kotlin
val name: String = "John"
val age: Int = 30
val height: Double = 5.9
val isActive: Boolean = true
val value: Any = "could be anything"
```

**TypeScript:**
```typescript
const name: string = "John";
const age: number = 30;
const height: number = 5.9;
const isActive: boolean = true;
const value: any = "could be anything";
```

### Type Inference

Both languages support type inference:

**Kotlin:**
```kotlin
val name = "John"  // Inferred as String
val age = 30       // Inferred as Int
```

**TypeScript:**
```typescript
const name = "John";  // Inferred as string
const age = 30;       // Inferred as number
```

### Null Safety

**Kotlin:**
```kotlin
var name: String = "John"        // Cannot be null
var nullableName: String? = null // Can be null

// Safe call
val length = nullableName?.length

// Elvis operator
val length = nullableName?.length ?: 0

// Safe cast
val str: String? = value as? String
```

**TypeScript:**
```typescript
let name: string = "John";              // Cannot be null (in strict mode)
let nullableName: string | null = null; // Can be null

// Optional chaining
const length = nullableName?.length;

// Nullish coalescing
const length = nullableName?.length ?? 0;

// Type guard
const str = value as string;
```

### Arrays and Lists

**Kotlin:**
```kotlin
val numbers: List<Int> = listOf(1, 2, 3)
val mutableNumbers: MutableList<Int> = mutableListOf(1, 2, 3)

// Array
val array: Array<String> = arrayOf("a", "b", "c")
```

**TypeScript:**
```typescript
const numbers: number[] = [1, 2, 3];
// Or using generic syntax
const numbers: Array<number> = [1, 2, 3];

// Arrays are mutable by default
numbers.push(4);

// For immutability, use readonly
const readonlyNumbers: readonly number[] = [1, 2, 3];
```

### Maps and Objects

**Kotlin:**
```kotlin
val map: Map<String, Int> = mapOf(
    "one" to 1,
    "two" to 2
)

val mutableMap: MutableMap<String, Int> = mutableMapOf(
    "one" to 1,
    "two" to 2
)
```

**TypeScript:**
```typescript
// Object literal
const map: { [key: string]: number } = {
    one: 1,
    two: 2
};

// Map object (preferred for dynamic keys)
const map = new Map<string, number>([
    ["one", 1],
    ["two", 2]
]);

// Record utility type
const map: Record<string, number> = {
    one: 1,
    two: 2
};
```

## Classes and Interfaces

### Classes

**Kotlin:**
```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String?
)

class UserRepository(
    private val apiService: ApiService
) {
    fun getUser(id: Int): User {
        return apiService.fetchUser(id)
    }
}
```

**TypeScript:**
```typescript
// Interface for data
interface User {
    id: number;
    name: string;
    email?: string; // Optional property
}

// Class with constructor
class UserRepository {
    constructor(private apiService: ApiService) {}

    getUser(id: number): User {
        return this.apiService.fetchUser(id);
    }
}

// Type alias (alternative to interface)
type User = {
    id: number;
    name: string;
    email?: string;
};
```

### Interfaces

**Kotlin:**
```kotlin
interface ClickListener {
    fun onClick()
    fun onLongClick() {
        // Default implementation
    }
}

class Button : ClickListener {
    override fun onClick() {
        println("Clicked")
    }
}
```

**TypeScript:**
```typescript
interface ClickListener {
    onClick(): void;
    onLongClick?(): void; // Optional method
}

class Button implements ClickListener {
    onClick(): void {
        console.log("Clicked");
    }
}
```

## Functions

### Function Declaration

**Kotlin:**
```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}

// Expression body
fun add(a: Int, b: Int) = a + b

// Default parameters
fun greet(name: String = "Guest"): String {
    return "Hello, $name"
}

// Higher-order functions
fun performOperation(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}
```

**TypeScript:**
```typescript
function add(a: number, b: number): number {
    return a + b;
}

// Arrow function
const add = (a: number, b: number): number => a + b;

// Default parameters
function greet(name: string = "Guest"): string {
    return `Hello, ${name}`;
}

// Higher-order functions
function performOperation(
    a: number,
    b: number,
    operation: (a: number, b: number) => number
): number {
    return operation(a, b);
}

// Usage
const result = performOperation(5, 3, (a, b) => a + b);
```

### Lambda Functions

**Kotlin:**
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

val doubled = numbers.map { it * 2 }
val filtered = numbers.filter { it > 2 }
val sum = numbers.reduce { acc, num -> acc + num }
```

**TypeScript:**
```typescript
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(it => it * 2);
const filtered = numbers.filter(it => it > 2);
const sum = numbers.reduce((acc, num) => acc + num);
```

## Async Programming

### Coroutines vs Promises/Async-Await

**Kotlin (Coroutines):**
```kotlin
suspend fun fetchUser(id: Int): User {
    delay(1000) // Simulates network delay
    return apiService.getUser(id)
}

fun loadUserData() {
    viewModelScope.launch {
        try {
            val user = fetchUser(1)
            println("User: $user")
        } catch (e: Exception) {
            println("Error: ${e.message}")
        }
    }
}

// Parallel execution
suspend fun loadMultipleUsers(): List<User> {
    return coroutineScope {
        val user1 = async { fetchUser(1) }
        val user2 = async { fetchUser(2) }
        listOf(user1.await(), user2.await())
    }
}
```

**TypeScript (Promises/Async-Await):**
```typescript
async function fetchUser(id: number): Promise<User> {
    await new Promise(resolve => setTimeout(resolve, 1000)); // Simulates delay
    return apiService.getUser(id);
}

async function loadUserData() {
    try {
        const user = await fetchUser(1);
        console.log("User:", user);
    } catch (e) {
        console.log("Error:", e.message);
    }
}

// Parallel execution
async function loadMultipleUsers(): Promise<User[]> {
    const [user1, user2] = await Promise.all([
        fetchUser(1),
        fetchUser(2)
    ]);
    return [user1, user2];
}

// Promises without async/await
function fetchUser(id: number): Promise<User> {
    return apiService.getUser(id)
        .then(user => {
            console.log("User:", user);
            return user;
        })
        .catch(error => {
            console.log("Error:", error);
            throw error;
        });
}
```

## Generics

**Kotlin:**
```kotlin
class Box<T>(val value: T)

fun <T> identity(value: T): T = value

// With constraints
fun <T : Comparable<T>> max(a: T, b: T): T {
    return if (a > b) a else b
}
```

**TypeScript:**
```typescript
class Box<T> {
    constructor(public value: T) {}
}

function identity<T>(value: T): T {
    return value;
}

// With constraints
function max<T extends { compareTo(other: T): number }>(a: T, b: T): T {
    return a.compareTo(b) > 0 ? a : b;
}

// Generic constraints with interfaces
interface Comparable<T> {
    compareTo(other: T): number;
}

function max<T extends Comparable<T>>(a: T, b: T): T {
    return a.compareTo(b) > 0 ? a : b;
}
```

## Advanced Types

### Union and Intersection Types

**TypeScript:**
```typescript
// Union type (OR)
type StringOrNumber = string | number;

function printId(id: string | number) {
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    } else {
        console.log(id.toFixed(2));
    }
}

// Intersection type (AND)
interface Nameable {
    name: string;
}

interface Ageable {
    age: number;
}

type Person = Nameable & Ageable;

const person: Person = {
    name: "John",
    age: 30
};
```

**Kotlin equivalent:**
```kotlin
// Sealed classes for union types
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
}

// Multiple interfaces for intersection
interface Nameable {
    val name: String
}

interface Ageable {
    val age: Int
}

data class Person(
    override val name: String,
    override val age: Int
) : Nameable, Ageable
```

### Type Guards

**TypeScript:**
```typescript
// typeof guard
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    return padding + value;
}

// instanceof guard
class Dog {
    bark() { console.log("Woof!"); }
}

class Cat {
    meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
        animal.bark();
    } else {
        animal.meow();
    }
}

// Custom type guard
interface Fish {
    swim: () => void;
}

interface Bird {
    fly: () => void;
}

function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
    if (isFish(pet)) {
        pet.swim();
    } else {
        pet.fly();
    }
}
```

### Utility Types

TypeScript provides many built-in utility types:

```typescript
interface User {
    id: number;
    name: string;
    email: string;
    age: number;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;

// Required - makes all properties required
type RequiredUser = Required<PartialUser>;

// Pick - select specific properties
type UserPreview = Pick<User, 'id' | 'name'>;

// Omit - exclude specific properties
type UserWithoutEmail = Omit<User, 'email'>;

// Record - create object type with specific keys and values
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>;

// Readonly - makes all properties readonly
type ReadonlyUser = Readonly<User>;
```

## String Templates

**Kotlin:**
```kotlin
val name = "John"
val age = 30
val message = "My name is $name and I'm $age years old"
val calculation = "Sum: ${2 + 2}"
```

**TypeScript:**
```typescript
const name = "John";
const age = 30;
const message = `My name is ${name} and I'm ${age} years old`;
const calculation = `Sum: ${2 + 2}`;
```

## Destructuring

**Kotlin:**
```kotlin
data class Point(val x: Int, val y: Int)

val point = Point(10, 20)
val (x, y) = point

// List destructuring
val (first, second) = listOf(1, 2, 3)

// Map destructuring
for ((key, value) in map) {
    println("$key -> $value")
}
```

**TypeScript:**
```typescript
interface Point {
    x: number;
    y: number;
}

const point: Point = { x: 10, y: 20 };
const { x, y } = point;

// Array destructuring
const [first, second] = [1, 2, 3];

// Renaming
const { x: posX, y: posY } = point;

// Default values
const { x = 0, y = 0 } = {} as Partial<Point>;

// Rest operator
const { x, ...rest } = { x: 1, y: 2, z: 3 };
```

## Enum

**Kotlin:**
```kotlin
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}

enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```

**TypeScript:**
```typescript
enum Direction {
    NORTH,
    SOUTH,
    EAST,
    WEST
}

enum Color {
    RED = 0xFF0000,
    GREEN = 0x00FF00,
    BLUE = 0x0000FF
}

// String enums
enum Direction {
    NORTH = "NORTH",
    SOUTH = "SOUTH",
    EAST = "EAST",
    WEST = "WEST"
}

// Const enum (for better performance)
const enum Direction {
    NORTH,
    SOUTH,
    EAST,
    WEST
}
```

## Key Differences to Remember

1. **Type Annotations**: TypeScript uses `:` before types, Kotlin uses `:` after variable name
2. **Null Safety**: TypeScript uses `| null`, Kotlin uses `?`
3. **Immutability**: TypeScript uses `const` and `readonly`, Kotlin uses `val` and immutable collections
4. **Async**: TypeScript uses Promises/async-await, Kotlin uses Coroutines
5. **Collections**: TypeScript arrays are mutable by default, Kotlin has separate mutable/immutable
6. **Modules**: TypeScript uses ES6 imports/exports, Kotlin uses package/import
7. **Compilation**: TypeScript compiles to JavaScript, Kotlin compiles to JVM bytecode (or JS/Native)

## Best Practices

1. **Enable strict mode** in `tsconfig.json` for better type safety
2. **Use interfaces over type aliases** for object shapes (better error messages)
3. **Prefer const over let** for immutability
4. **Use optional chaining (`?.`) and nullish coalescing (`??`)** for null handling
5. **Avoid `any` type** - use `unknown` if you need a type-safe alternative
6. **Use type guards** to narrow union types
7. **Leverage utility types** instead of duplicating types

## Common Pitfalls

1. **`this` binding**: Arrow functions vs regular functions have different `this` behavior
2. **Truthy/falsy values**: `0`, `""`, `false`, `null`, `undefined` are all falsy
3. **`==` vs `===`**: Always use `===` (strict equality)
4. **Array methods**: `map`, `filter`, etc. return new arrays (immutable)
5. **Hoisting**: Variables declared with `var` are hoisted, use `const`/`let` instead

## Next Steps

Now that you understand TypeScript basics, let's move to [Project Structure & Lifecycle](./RN-02-project-structure-lifecycle.md) to see how React Native projects are organized.
