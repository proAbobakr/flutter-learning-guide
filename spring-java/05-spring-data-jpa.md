# Spring Data JPA - Database Access for Kotlin Android Developers

Learn how Spring Data JPA simplifies database operations in Spring applications - think of it as Android's Room Database on steroids!

## 🎯 Overview

**Spring Data JPA** = JPA (Java Persistence API) + Hibernate (ORM) + Spring Data Repositories

If you've worked with Room Database in Android, you'll find Spring Data JPA familiar but more powerful. Both provide abstraction over SQL databases, but JPA offers more flexibility and advanced features.

## 📋 Room vs JPA Comparison

| Aspect | Room Database | Spring Data JPA |
|--------|---------------|-----------------|
| Framework | Android-specific | Java/Spring standard |
| ORM Engine | SQLite-optimized | Hibernate (configurable) |
| Compile-time checks | Yes (annotation processor) | Mostly runtime |
| Database Support | SQLite | MySQL, PostgreSQL, Oracle, H2, etc. |
| Query Language | SQL | JPQL, SQL, Criteria API |
| Relationships | Limited | Full support with lazy/eager loading |
| Caching | Manual | Built-in (1st & 2nd level cache) |
| Migrations | Manual (or Room migrations) | Flyway, Liquibase integration |
| Learning Curve | Moderate | Steeper |
| Performance | Optimized for mobile | Enterprise-grade |

## 1. JPA and Hibernate Basics

### What is JPA?

**JPA (Java Persistence API)** is a specification for ORM (Object-Relational Mapping) in Java. It's like an interface that defines how to map Java objects to database tables.

**Hibernate** is the most popular implementation of JPA - think of it as the "engine" that does the actual work.

```
┌─────────────────────────────────────────────┐
│         Your Application Code               │
│     (User.java, UserRepository.java)        │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│           Spring Data JPA                   │
│   (Repository implementations, @Query)      │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│              JPA (Specification)            │
│         (Interface/Contract)                │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│            Hibernate (Implementation)       │
│     (SQL Generation, Caching, etc.)         │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│              JDBC Driver                    │
│      (MySQL, PostgreSQL, etc.)              │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│              Database                       │
│        (MySQL, PostgreSQL, etc.)            │
└─────────────────────────────────────────────┘
```

### Room vs JPA Architecture

**Room Architecture:**
```
Android App
    ↓
Room Database (@Database)
    ↓
DAO Interface (@Dao)
    ↓
Entity (@Entity)
    ↓
SQLite Database
```

**Spring Data JPA Architecture:**
```
Spring Application
    ↓
Repository Interface (extends JpaRepository)
    ↓
Entity (@Entity with JPA annotations)
    ↓
JPA/Hibernate
    ↓
JDBC
    ↓
Any Relational Database
```

### Setup Spring Data JPA

**Step 1: Add Dependencies (pom.xml)**

```xml
<dependencies>
    <!-- Spring Data JPA Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Database Driver - Choose one -->
    <!-- MySQL -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Or PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Or H2 (In-memory database for testing) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

**Step 2: Configure Database (application.properties)**

```properties
# MySQL Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/myapp_db
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate Properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Connection Pool (HikariCP - default in Spring Boot)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
```

**ddl-auto Options:**
- `none` - No schema generation (production)
- `validate` - Validate schema, no changes
- `update` - Update schema (add new tables/columns)
- `create` - Drop and recreate schema on startup
- `create-drop` - Drop schema when SessionFactory closes

**Room Equivalent:**
```kotlin
// In Room, you'd define database like this:
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

## 2. Entity Mapping and Annotations

### Basic Entity

**Room Entity (Android):**
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,

    @ColumnInfo(name = "user_name")
    val userName: String,

    @ColumnInfo(name = "email")
    val email: String,

    @ColumnInfo(name = "created_at")
    val createdAt: Long
)
```

**JPA Entity (Spring):**
```java
package com.example.myapp.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "user_name", nullable = false, length = 50)
    private String userName;

    @Column(name = "email", unique = true, nullable = false)
    private String email;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    // Constructors
    public User() {}

    public User(String userName, String email) {
        this.userName = userName;
        this.email = email;
        this.createdAt = LocalDateTime.now();
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUserName() { return userName; }
    public void setUserName(String userName) { this.userName = userName; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }

    @Override
    public String toString() {
        return "User{id=" + id + ", userName='" + userName + "', email='" + email + "'}";
    }
}
```

### Common JPA Annotations

| Annotation | Purpose | Room Equivalent |
|------------|---------|-----------------|
| `@Entity` | Marks class as JPA entity | `@Entity` |
| `@Table` | Specifies table name | `@Entity(tableName)` |
| `@Id` | Primary key | `@PrimaryKey` |
| `@GeneratedValue` | Auto-increment strategy | `autoGenerate = true` |
| `@Column` | Column properties | `@ColumnInfo` |
| `@Transient` | Don't persist this field | `@Ignore` |
| `@Temporal` | Date/time mapping | - |
| `@Enumerated` | Enum mapping | - |
| `@Lob` | Large object (BLOB/CLOB) | - |
| `@Embedded` | Embed object | `@Embedded` |

### Advanced Entity Example

```java
@Entity
@Table(
    name = "products",
    indexes = {
        @Index(name = "idx_product_name", columnList = "name"),
        @Index(name = "idx_product_category", columnList = "category")
    },
    uniqueConstraints = {
        @UniqueConstraint(name = "uk_product_sku", columnNames = "sku")
    }
)
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name", nullable = false, length = 100)
    private String name;

    @Column(name = "sku", unique = true, nullable = false, length = 20)
    private String sku;

    @Column(name = "price", nullable = false, precision = 10, scale = 2)
    private Double price;

    @Enumerated(EnumType.STRING)
    @Column(name = "category", length = 30)
    private ProductCategory category;

    @Column(name = "description", columnDefinition = "TEXT")
    private String description;

    @Column(name = "in_stock")
    private Boolean inStock = true;

    @Column(name = "quantity")
    private Integer quantity = 0;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_at", updatable = false)
    private Date createdAt;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "updated_at")
    private Date updatedAt;

    @Transient  // Not persisted to database
    private String tempCalculation;

    @PrePersist
    protected void onCreate() {
        createdAt = new Date();
        updatedAt = new Date();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = new Date();
    }

    // Constructors, getters, setters...
}

enum ProductCategory {
    ELECTRONICS,
    CLOTHING,
    BOOKS,
    FOOD,
    OTHER
}
```

### Embedded Objects

**Scenario:** You want to group related fields without creating a separate table.

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    private String country;

    // Constructors, getters, setters...
}

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street", column = @Column(name = "home_street")),
        @AttributeOverride(name = "city", column = @Column(name = "home_city")),
        @AttributeOverride(name = "state", column = @Column(name = "home_state")),
        @AttributeOverride(name = "zipCode", column = @Column(name = "home_zip")),
        @AttributeOverride(name = "country", column = @Column(name = "home_country"))
    })
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street", column = @Column(name = "work_street")),
        @AttributeOverride(name = "city", column = @Column(name = "work_city")),
        @AttributeOverride(name = "state", column = @Column(name = "work_state")),
        @AttributeOverride(name = "zipCode", column = @Column(name = "work_zip")),
        @AttributeOverride(name = "country", column = @Column(name = "work_country"))
    })
    private Address workAddress;

    // Getters, setters...
}
```

**Resulting Table:**
```
users table:
- id
- name
- email
- home_street, home_city, home_state, home_zip, home_country
- work_street, work_city, work_state, work_zip, work_country
```

**Room Equivalent:**
```kotlin
data class Address(
    val street: String,
    val city: String,
    val state: String,
    val zipCode: String
)

@Entity
data class User(
    @PrimaryKey val id: Long,
    val name: String,
    @Embedded(prefix = "home_") val homeAddress: Address,
    @Embedded(prefix = "work_") val workAddress: Address
)
```

## 3. Relationships

This is where JPA really shines compared to Room! JPA handles relationships automatically with lazy/eager loading.

### @OneToMany and @ManyToOne

**Scenario:** One user can have many posts.

```java
// Parent Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String email;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Post> posts = new ArrayList<>();

    // Helper methods for bidirectional relationship
    public void addPost(Post post) {
        posts.add(post);
        post.setUser(this);
    }

    public void removePost(Post post) {
        posts.remove(post);
        post.setUser(null);
    }

    // Getters, setters...
}

// Child Entity
@Entity
@Table(name = "posts")
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @Column(columnDefinition = "TEXT")
    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    private LocalDateTime createdAt;

    // Getters, setters...
}
```

**Resulting Tables:**
```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255),
    email VARCHAR(255)
);

CREATE TABLE posts (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    user_id BIGINT NOT NULL,
    created_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Room Equivalent (More Complex):**
```kotlin
@Entity
data class User(
    @PrimaryKey val id: Long,
    val username: String,
    val email: String
)

@Entity(
    foreignKeys = [ForeignKey(
        entity = User::class,
        parentColumns = ["id"],
        childColumns = ["userId"],
        onDelete = ForeignKey.CASCADE
    )]
)
data class Post(
    @PrimaryKey val id: Long,
    val title: String,
    val content: String,
    val userId: Long
)

// Relation class
data class UserWithPosts(
    @Embedded val user: User,
    @Relation(
        parentColumn = "id",
        entityColumn = "userId"
    )
    val posts: List<Post>
)

@Dao
interface UserDao {
    @Transaction
    @Query("SELECT * FROM User WHERE id = :userId")
    fun getUserWithPosts(userId: Long): UserWithPosts
}
```

### Cascade Types

| Cascade Type | Description | When Parent is... |
|--------------|-------------|-------------------|
| `PERSIST` | Persist child when parent persisted | Saved → Child saved |
| `MERGE` | Merge child when parent merged | Updated → Child updated |
| `REMOVE` | Remove child when parent removed | Deleted → Child deleted |
| `REFRESH` | Refresh child when parent refreshed | Refreshed → Child refreshed |
| `DETACH` | Detach child when parent detached | Detached → Child detached |
| `ALL` | All of the above | Any operation → Applies to child |

### Fetch Types

| Fetch Type | When | Use Case |
|------------|------|----------|
| `EAGER` | Load immediately with parent | Small collections, always needed |
| `LAZY` | Load only when accessed | Large collections, sometimes needed |

**Default Fetch Types:**
- `@OneToMany` → LAZY
- `@ManyToOne` → EAGER
- `@ManyToMany` → LAZY
- `@OneToOne` → EAGER

### @ManyToMany

**Scenario:** Students can enroll in many courses, and courses can have many students.

```java
@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();

    // Helper methods
    public void enrollCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }

    public void dropCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }

    // Getters, setters...
}

@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String code;
    private String name;
    private Integer credits;

    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();

    // Getters, setters...
}
```

**Resulting Tables:**
```sql
CREATE TABLE students (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255)
);

CREATE TABLE courses (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(20),
    name VARCHAR(255),
    credits INT
);

-- Join Table (automatically created)
CREATE TABLE student_course (
    student_id BIGINT,
    course_id BIGINT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

### @ManyToMany with Extra Attributes

**Scenario:** You want to store enrollment date in the join table.

```java
// Junction Entity
@Entity
@Table(name = "enrollments")
public class Enrollment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "student_id")
    private Student student;

    @ManyToOne
    @JoinColumn(name = "course_id")
    private Course course;

    @Column(name = "enrolled_at")
    private LocalDateTime enrolledAt;

    @Column(name = "grade")
    private String grade;

    // Constructors, getters, setters...
}

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "student", cascade = CascadeType.ALL)
    private List<Enrollment> enrollments = new ArrayList<>();

    // Getters, setters...
}

@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL)
    private List<Enrollment> enrollments = new ArrayList<>();

    // Getters, setters...
}
```

### @OneToOne

**Scenario:** One user has one profile (and vice versa).

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String email;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private UserProfile profile;

    public void setProfile(UserProfile profile) {
        this.profile = profile;
        profile.setUser(this);
    }

    // Getters, setters...
}

@Entity
@Table(name = "user_profiles")
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne
    @JoinColumn(name = "user_id", nullable = false, unique = true)
    private User user;

    private String firstName;
    private String lastName;
    private String phone;
    private String bio;

    // Getters, setters...
}
```

**Shared Primary Key Approach:**
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    private UserProfile profile;

    // Getters, setters...
}

@Entity
@Table(name = "user_profiles")
public class UserProfile {
    @Id
    private Long id;

    @OneToOne
    @MapsId
    @JoinColumn(name = "id")
    private User user;

    private String firstName;
    private String lastName;

    // Getters, setters...
}
```

## 4. Repository Pattern and JpaRepository

Spring Data JPA provides repository interfaces that eliminate boilerplate code!

### Basic Repository

**Room DAO:**
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): List<User>

    @Query("SELECT * FROM users WHERE id = :id")
    fun getUserById(id: Long): User?

    @Insert
    fun insertUser(user: User): Long

    @Update
    fun updateUser(user: User)

    @Delete
    fun deleteUser(user: User)
}
```

**Spring Data JPA Repository:**
```java
package com.example.myapp.repository;

import com.example.myapp.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // That's it! You get CRUD operations for free!
}
```

**What you get automatically:**

```java
// No need to implement - all provided by JpaRepository!

// Save/Update
User save(User user);
List<User> saveAll(Iterable<User> users);

// Find
Optional<User> findById(Long id);
List<User> findAll();
List<User> findAllById(Iterable<Long> ids);

// Delete
void delete(User user);
void deleteById(Long id);
void deleteAll();
void deleteAllById(Iterable<Long> ids);

// Count/Exists
long count();
boolean existsById(Long id);
```

### Repository Hierarchy

```
                    Repository<T, ID>
                            ↑
                            |
                    CrudRepository<T, ID>
                            ↑
                            |
                    PagingAndSortingRepository<T, ID>
                            ↑
                            |
                    JpaRepository<T, ID>
                            ↑
                            |
                    YourCustomRepository
```

**Features by Interface:**

| Interface | Features |
|-----------|----------|
| `Repository` | Marker interface |
| `CrudRepository` | Basic CRUD operations |
| `PagingAndSortingRepository` | Pagination and sorting |
| `JpaRepository` | JPA-specific (batch operations, flushing) |

### Using the Repository

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    // Constructor injection (recommended)
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public User updateUser(Long id, User userDetails) {
        User user = getUserById(id);
        user.setUserName(userDetails.getUserName());
        user.setEmail(userDetails.getEmail());
        return userRepository.save(user);
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    public boolean userExists(Long id) {
        return userRepository.existsById(id);
    }

    public long getUserCount() {
        return userRepository.count();
    }
}
```

### Pagination and Sorting

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // Get users with pagination
    public Page<User> getUsers(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return userRepository.findAll(pageable);
    }

    // Get users with sorting
    public List<User> getUsersSorted() {
        Sort sort = Sort.by(Sort.Direction.DESC, "createdAt");
        return userRepository.findAll(sort);
    }

    // Get users with pagination and sorting
    public Page<User> getUsersPaginatedAndSorted(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy).descending());
        return userRepository.findAll(pageable);
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public ResponseEntity<Page<User>> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy) {

        Page<User> users = userService.getUsersPaginatedAndSorted(page, size, sortBy);
        return ResponseEntity.ok(users);
    }
}
```

**Page Object Contains:**
```java
Page<User> page = userRepository.findAll(pageable);

page.getContent();           // List<User> - Current page data
page.getTotalElements();     // Total number of items
page.getTotalPages();        // Total number of pages
page.getNumber();            // Current page number
page.getSize();              // Page size
page.hasNext();              // Has next page?
page.hasPrevious();          // Has previous page?
page.isFirst();              // Is first page?
page.isLast();               // Is last page?
```

## 5. Query Methods

Spring Data JPA can generate queries from method names - no SQL needed!

### Derived Query Methods

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // SELECT * FROM users WHERE user_name = ?
    User findByUserName(String userName);

    // SELECT * FROM users WHERE email = ?
    Optional<User> findByEmail(String email);

    // SELECT * FROM users WHERE user_name = ? AND email = ?
    List<User> findByUserNameAndEmail(String userName, String email);

    // SELECT * FROM users WHERE user_name = ? OR email = ?
    List<User> findByUserNameOrEmail(String userName, String email);

    // SELECT * FROM users WHERE created_at BETWEEN ? AND ?
    List<User> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    // SELECT * FROM users WHERE age < ?
    List<User> findByAgeLessThan(Integer age);

    // SELECT * FROM users WHERE age >= ?
    List<User> findByAgeGreaterThanEqual(Integer age);

    // SELECT * FROM users WHERE user_name LIKE ?
    List<User> findByUserNameContaining(String keyword);

    // SELECT * FROM users WHERE user_name LIKE ?
    List<User> findByUserNameStartingWith(String prefix);

    // SELECT * FROM users WHERE email LIKE ?
    List<User> findByEmailEndingWith(String suffix);

    // SELECT * FROM users WHERE email IS NULL
    List<User> findByEmailIsNull();

    // SELECT * FROM users WHERE email IS NOT NULL
    List<User> findByEmailIsNotNull();

    // SELECT * FROM users WHERE active = true
    List<User> findByActiveTrue();

    // SELECT * FROM users WHERE active = false
    List<User> findByActiveFalse();

    // SELECT * FROM users ORDER BY user_name ASC
    List<User> findAllByOrderByUserNameAsc();

    // SELECT * FROM users ORDER BY created_at DESC
    List<User> findAllByOrderByCreatedAtDesc();

    // SELECT * FROM users WHERE age > ? ORDER BY user_name
    List<User> findByAgeGreaterThanOrderByUserName(Integer age);

    // SELECT DISTINCT user_name FROM users
    List<String> findDistinctUserNameBy();

    // SELECT * FROM users WHERE user_name IN (?)
    List<User> findByUserNameIn(List<String> userNames);

    // SELECT * FROM users WHERE user_name NOT IN (?)
    List<User> findByUserNameNotIn(List<String> userNames);

    // Count queries
    long countByActive(Boolean active);

    // Exists queries
    boolean existsByEmail(String email);

    // Delete queries
    void deleteByUserName(String userName);
    long deleteByActiveAndCreatedAtBefore(Boolean active, LocalDateTime date);

    // Top/First queries
    User findFirstByOrderByCreatedAtDesc();
    List<User> findTop10ByOrderByCreatedAtDesc();
}
```

### Query Method Keywords

| Keyword | Example | JPQL Snippet |
|---------|---------|--------------|
| `And` | `findByNameAndAge` | `WHERE name = ? AND age = ?` |
| `Or` | `findByNameOrAge` | `WHERE name = ? OR age = ?` |
| `Is, Equals` | `findByName` | `WHERE name = ?` |
| `Between` | `findByAgeBetween` | `WHERE age BETWEEN ? AND ?` |
| `LessThan` | `findByAgeLessThan` | `WHERE age < ?` |
| `LessThanEqual` | `findByAgeLessThanEqual` | `WHERE age <= ?` |
| `GreaterThan` | `findByAgeGreaterThan` | `WHERE age > ?` |
| `GreaterThanEqual` | `findByAgeGreaterThanEqual` | `WHERE age >= ?` |
| `After` | `findByDateAfter` | `WHERE date > ?` |
| `Before` | `findByDateBefore` | `WHERE date < ?` |
| `IsNull` | `findByAgeIsNull` | `WHERE age IS NULL` |
| `IsNotNull, NotNull` | `findByAgeIsNotNull` | `WHERE age IS NOT NULL` |
| `Like` | `findByNameLike` | `WHERE name LIKE ?` |
| `NotLike` | `findByNameNotLike` | `WHERE name NOT LIKE ?` |
| `StartingWith` | `findByNameStartingWith` | `WHERE name LIKE ?%` |
| `EndingWith` | `findByNameEndingWith` | `WHERE name LIKE %?` |
| `Containing` | `findByNameContaining` | `WHERE name LIKE %?%` |
| `OrderBy` | `findByAgeOrderByNameDesc` | `ORDER BY name DESC` |
| `Not` | `findByAgeNot` | `WHERE age <> ?` |
| `In` | `findByAgeIn(Collection ages)` | `WHERE age IN (?)` |
| `NotIn` | `findByAgeNotIn(Collection)` | `WHERE age NOT IN (?)` |
| `True` | `findByActiveTrue()` | `WHERE active = true` |
| `False` | `findByActiveFalse()` | `WHERE active = false` |
| `IgnoreCase` | `findByNameIgnoreCase` | `WHERE UPPER(name) = UPPER(?)` |

### @Query Annotation

For complex queries, use `@Query` with JPQL or native SQL.

**JPQL (Java Persistence Query Language):**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // JPQL - Uses entity names and properties
    @Query("SELECT u FROM User u WHERE u.userName = ?1")
    User findByUserName(String userName);

    // Named parameters (recommended)
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);

    // Complex query with JOIN
    @Query("SELECT u FROM User u LEFT JOIN FETCH u.posts WHERE u.id = :id")
    Optional<User> findUserWithPosts(@Param("id") Long id);

    // Aggregate functions
    @Query("SELECT COUNT(u) FROM User u WHERE u.active = :active")
    long countActiveUsers(@Param("active") Boolean active);

    // Custom projection
    @Query("SELECT new com.example.myapp.dto.UserDTO(u.id, u.userName, u.email) " +
           "FROM User u WHERE u.id = :id")
    UserDTO findUserDTOById(@Param("id") Long id);

    // Multiple conditions
    @Query("SELECT u FROM User u WHERE " +
           "(:userName IS NULL OR u.userName LIKE %:userName%) AND " +
           "(:email IS NULL OR u.email LIKE %:email%)")
    List<User> searchUsers(@Param("userName") String userName, @Param("email") String email);

    // Sorting with JPQL
    @Query("SELECT u FROM User u WHERE u.active = :active ORDER BY u.createdAt DESC")
    List<User> findActiveUsers(@Param("active") Boolean active);

    // Pagination with @Query
    @Query("SELECT u FROM User u WHERE u.userName LIKE %:keyword%")
    Page<User> searchUsersByKeyword(@Param("keyword") String keyword, Pageable pageable);
}
```

**Native SQL:**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE user_name = ?1", nativeQuery = true)
    User findByUserNameNative(String userName);

    // Native query with named parameters
    @Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
    User findByEmailNative(@Param("email") String email);

    // Complex native query
    @Query(value = "SELECT u.* FROM users u " +
                   "LEFT JOIN posts p ON u.id = p.user_id " +
                   "WHERE p.created_at > :date " +
                   "GROUP BY u.id " +
                   "HAVING COUNT(p.id) > :minPosts",
           nativeQuery = true)
    List<User> findActiveUsersWithPosts(@Param("date") LocalDateTime date,
                                        @Param("minPosts") int minPosts);

    // Native query with pagination
    @Query(value = "SELECT * FROM users WHERE active = true ORDER BY created_at DESC",
           countQuery = "SELECT COUNT(*) FROM users WHERE active = true",
           nativeQuery = true)
    Page<User> findActiveUsersNative(Pageable pageable);
}
```

### Modifying Queries

For UPDATE and DELETE operations:

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.active = :active WHERE u.id = :id")
    int updateUserActive(@Param("id") Long id, @Param("active") Boolean active);

    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.userName = :userName, u.email = :email WHERE u.id = :id")
    int updateUser(@Param("id") Long id,
                   @Param("userName") String userName,
                   @Param("email") String email);

    @Modifying
    @Transactional
    @Query("DELETE FROM User u WHERE u.active = false AND u.createdAt < :date")
    int deleteInactiveUsersBefore(@Param("date") LocalDateTime date);

    // Native update
    @Modifying
    @Transactional
    @Query(value = "UPDATE users SET last_login = NOW() WHERE id = :id", nativeQuery = true)
    int updateLastLogin(@Param("id") Long id);
}
```

### Projections

Return only specific fields instead of full entities:

**Interface Projection:**
```java
public interface UserSummary {
    Long getId();
    String getUserName();
    String getEmail();
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findAllProjectedBy();
    UserSummary findProjectedById(Long id);
}
```

**Class-based Projection (DTO):**
```java
public class UserDTO {
    private Long id;
    private String userName;
    private String email;

    public UserDTO(Long id, String userName, String email) {
        this.id = id;
        this.userName = userName;
        this.email = email;
    }

    // Getters...
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT new com.example.myapp.dto.UserDTO(u.id, u.userName, u.email) FROM User u")
    List<UserDTO> findAllUserDTOs();
}
```

**Dynamic Projections:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    <T> List<T> findByActive(Boolean active, Class<T> type);
}

// Usage
List<User> users = userRepository.findByActive(true, User.class);
List<UserSummary> summaries = userRepository.findByActive(true, UserSummary.class);
```

## 6. Transactions with @Transactional

Transactions ensure data consistency - either all operations succeed or all fail.

### Basic Transaction

```java
@Service
public class UserService {

    private final UserRepository userRepository;
    private final PostRepository postRepository;

    public UserService(UserRepository userRepository, PostRepository postRepository) {
        this.userRepository = userRepository;
        this.postRepository = postRepository;
    }

    @Transactional
    public User createUserWithPosts(User user, List<Post> posts) {
        // Save user
        User savedUser = userRepository.save(user);

        // Save posts
        for (Post post : posts) {
            post.setUser(savedUser);
            postRepository.save(post);
        }

        // If any operation fails, ALL changes are rolled back
        return savedUser;
    }

    @Transactional
    public void transferCredits(Long fromUserId, Long toUserId, int credits) {
        User fromUser = userRepository.findById(fromUserId)
            .orElseThrow(() -> new UserNotFoundException("User not found"));
        User toUser = userRepository.findById(toUserId)
            .orElseThrow(() -> new UserNotFoundException("User not found"));

        if (fromUser.getCredits() < credits) {
            throw new InsufficientCreditsException("Not enough credits");
        }

        // Deduct from sender
        fromUser.setCredits(fromUser.getCredits() - credits);
        userRepository.save(fromUser);

        // Add to receiver
        toUser.setCredits(toUser.getCredits() + credits);
        userRepository.save(toUser);

        // Both updates succeed or both fail
    }
}
```

### Transaction Properties

```java
@Transactional(
    propagation = Propagation.REQUIRED,    // Transaction propagation
    isolation = Isolation.DEFAULT,         // Isolation level
    timeout = 30,                          // Timeout in seconds
    readOnly = false,                      // Read-only transaction?
    rollbackFor = Exception.class,         // Rollback for these exceptions
    noRollbackFor = IllegalArgumentException.class  // Don't rollback for these
)
public void complexOperation() {
    // ...
}
```

### Propagation Types

| Propagation | Description |
|-------------|-------------|
| `REQUIRED` (default) | Use existing transaction or create new one |
| `REQUIRES_NEW` | Always create new transaction, suspend existing |
| `MANDATORY` | Must be called within existing transaction |
| `NEVER` | Must NOT be called within transaction |
| `NOT_SUPPORTED` | Execute non-transactionally |
| `SUPPORTS` | Execute within transaction if exists, otherwise non-transactional |
| `NESTED` | Execute within nested transaction |

**Example:**
```java
@Service
public class OrderService {

    @Transactional
    public void processOrder(Order order) {
        saveOrder(order);
        sendNotification(order);  // Should use separate transaction
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendNotification(Order order) {
        // Runs in separate transaction
        // If this fails, order is still saved
    }
}
```

### Isolation Levels

| Isolation | Description | Problems Prevented |
|-----------|-------------|-------------------|
| `DEFAULT` | Database default | Depends on DB |
| `READ_UNCOMMITTED` | Dirty reads allowed | None |
| `READ_COMMITTED` | Read only committed data | Dirty reads |
| `REPEATABLE_READ` | Same query returns same results | Dirty, non-repeatable reads |
| `SERIALIZABLE` | Full isolation | All concurrency issues |

### Read-Only Transactions

```java
@Service
public class UserService {

    @Transactional(readOnly = true)
    public List<User> getAllUsers() {
        return userRepository.findAll();
        // Optimized for reading
        // Hibernate won't check for changes
    }

    @Transactional(readOnly = true)
    public UserDTO getUserDetails(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found"));
        return convertToDTO(user);
    }
}
```

### Programmatic Transactions

```java
@Service
public class UserService {

    private final TransactionTemplate transactionTemplate;
    private final UserRepository userRepository;

    public UserService(PlatformTransactionManager transactionManager,
                       UserRepository userRepository) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
        this.userRepository = userRepository;
    }

    public User createUser(User user) {
        return transactionTemplate.execute(status -> {
            try {
                return userRepository.save(user);
            } catch (Exception e) {
                status.setRollbackOnly();
                throw e;
            }
        });
    }
}
```

### Transaction Best Practices

1. **Keep transactions short** - Don't include I/O operations
```java
// BAD
@Transactional
public void processOrder(Order order) {
    orderRepository.save(order);
    sendEmailToCustomer(order);  // I/O operation - slow!
    callExternalAPI(order);      // Network call - can fail!
}

// GOOD
@Transactional
public void processOrder(Order order) {
    orderRepository.save(order);
}

public void notifyCustomer(Order order) {
    sendEmailToCustomer(order);
}
```

2. **Use read-only for queries**
```java
@Transactional(readOnly = true)
public List<User> getAllUsers() {
    return userRepository.findAll();
}
```

3. **Specify rollback conditions**
```java
@Transactional(rollbackFor = Exception.class)
public void createUser(User user) {
    // Rollback for ANY exception, not just RuntimeException
}
```

## 7. Database Configuration

### Single Datasource Configuration

**application.properties:**
```properties
# Database Connection
spring.datasource.url=jdbc:mysql://localhost:3306/myapp_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Connection Pool (HikariCP)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.max-lifetime=1200000

# JPA Properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.properties.hibernate.jdbc.batch_size=20
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true

# Naming Strategy
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
spring.jpa.hibernate.naming.implicit-strategy=org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
```

### Multiple Datasources

**Scenario:** You need to connect to multiple databases (e.g., MySQL for users, PostgreSQL for analytics).

**application.properties:**
```properties
# Primary Database (MySQL)
spring.datasource.primary.jdbc-url=jdbc:mysql://localhost:3306/myapp_db
spring.datasource.primary.username=root
spring.datasource.primary.password=secret
spring.datasource.primary.driver-class-name=com.mysql.cj.jdbc.Driver

# Secondary Database (PostgreSQL)
spring.datasource.secondary.jdbc-url=jdbc:postgresql://localhost:5432/analytics_db
spring.datasource.secondary.username=postgres
spring.datasource.secondary.password=secret
spring.datasource.secondary.driver-class-name=org.postgresql.Driver
```

**Primary Database Configuration:**
```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.example.myapp.repository.primary",
    entityManagerFactoryRef = "primaryEntityManagerFactory",
    transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryDatabaseConfig {

    @Primary
    @Bean(name = "primaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean(name = "primaryEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(
            EntityManagerFactoryBuilder builder,
            @Qualifier("primaryDataSource") DataSource dataSource) {
        return builder
            .dataSource(dataSource)
            .packages("com.example.myapp.model.primary")
            .persistenceUnit("primary")
            .build();
    }

    @Primary
    @Bean(name = "primaryTransactionManager")
    public PlatformTransactionManager transactionManager(
            @Qualifier("primaryEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

**Secondary Database Configuration:**
```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.example.myapp.repository.secondary",
    entityManagerFactoryRef = "secondaryEntityManagerFactory",
    transactionManagerRef = "secondaryTransactionManager"
)
public class SecondaryDatabaseConfig {

    @Bean(name = "secondaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondaryEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(
            EntityManagerFactoryBuilder builder,
            @Qualifier("secondaryDataSource") DataSource dataSource) {
        return builder
            .dataSource(dataSource)
            .packages("com.example.myapp.model.secondary")
            .persistenceUnit("secondary")
            .build();
    }

    @Bean(name = "secondaryTransactionManager")
    public PlatformTransactionManager transactionManager(
            @Qualifier("secondaryEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

**Project Structure with Multiple Datasources:**
```
src/main/java/com/example/myapp/
├── model/
│   ├── primary/        // Entities for primary DB
│   │   ├── User.java
│   │   └── Post.java
│   └── secondary/      // Entities for secondary DB
│       └── Analytics.java
├── repository/
│   ├── primary/
│   │   ├── UserRepository.java
│   │   └── PostRepository.java
│   └── secondary/
│       └── AnalyticsRepository.java
└── config/
    ├── PrimaryDatabaseConfig.java
    └── SecondaryDatabaseConfig.java
```

### Database Connection Pool Configuration

**HikariCP (Default in Spring Boot):**
```properties
spring.datasource.hikari.maximum-pool-size=20        # Max connections
spring.datasource.hikari.minimum-idle=10            # Min idle connections
spring.datasource.hikari.connection-timeout=30000   # Wait for connection (ms)
spring.datasource.hikari.idle-timeout=600000        # Idle connection timeout (ms)
spring.datasource.hikari.max-lifetime=1800000       # Max connection lifetime (ms)
spring.datasource.hikari.auto-commit=true
spring.datasource.hikari.pool-name=MyAppHikariCP
```

**Alternative: Apache DBCP2:**
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
</dependency>
```

```properties
spring.datasource.type=org.apache.commons.dbcp2.BasicDataSource
spring.datasource.dbcp2.max-total=20
spring.datasource.dbcp2.max-idle=10
spring.datasource.dbcp2.min-idle=5
```

## 8. Complete CRUD Example with Relationships

Let's build a complete blog system with Users, Posts, and Comments!

### Entities

**User.java:**
```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username", unique = true, nullable = false, length = 50)
    private String username;

    @Column(name = "email", unique = true, nullable = false)
    private String email;

    @Column(name = "password", nullable = false)
    private String password;

    @Column(name = "bio", columnDefinition = "TEXT")
    private String bio;

    @Column(name = "active")
    private Boolean active = true;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Post> posts = new ArrayList<>();

    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // Helper methods
    public void addPost(Post post) {
        posts.add(post);
        post.setAuthor(this);
    }

    public void removePost(Post post) {
        posts.remove(post);
        post.setAuthor(null);
    }

    // Constructors, getters, setters, toString...
}
```

**Post.java:**
```java
@Entity
@Table(name = "posts")
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "title", nullable = false, length = 200)
    private String title;

    @Column(name = "content", columnDefinition = "TEXT", nullable = false)
    private String content;

    @Column(name = "slug", unique = true, nullable = false)
    private String slug;

    @Column(name = "published")
    private Boolean published = false;

    @Column(name = "view_count")
    private Integer viewCount = 0;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;

    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();

    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "post_tags",
        joinColumns = @JoinColumn(name = "post_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
        if (slug == null) {
            slug = generateSlug(title);
        }
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    private String generateSlug(String title) {
        return title.toLowerCase()
            .replaceAll("[^a-z0-9\\s-]", "")
            .replaceAll("\\s+", "-");
    }

    // Helper methods
    public void addComment(Comment comment) {
        comments.add(comment);
        comment.setPost(this);
    }

    public void removeComment(Comment comment) {
        comments.remove(comment);
        comment.setPost(null);
    }

    public void addTag(Tag tag) {
        tags.add(tag);
        tag.getPosts().add(this);
    }

    public void removeTag(Tag tag) {
        tags.remove(tag);
        tag.getPosts().remove(this);
    }

    // Constructors, getters, setters...
}
```

**Comment.java:**
```java
@Entity
@Table(name = "comments")
public class Comment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "content", columnDefinition = "TEXT", nullable = false)
    private String content;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }

    // Constructors, getters, setters...
}
```

**Tag.java:**
```java
@Entity
@Table(name = "tags")
public class Tag {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name", unique = true, nullable = false, length = 50)
    private String name;

    @ManyToMany(mappedBy = "tags")
    private Set<Post> posts = new HashSet<>();

    // Constructors, getters, setters...
}
```

### Repositories

**UserRepository.java:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);

    @Query("SELECT u FROM User u LEFT JOIN FETCH u.posts WHERE u.id = :id")
    Optional<User> findUserWithPosts(@Param("id") Long id);

    List<User> findByActiveTrue();

    @Query("SELECT u FROM User u WHERE u.createdAt BETWEEN :start AND :end")
    List<User> findUsersCreatedBetween(@Param("start") LocalDateTime start,
                                       @Param("end") LocalDateTime end);
}
```

**PostRepository.java:**
```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {

    Optional<Post> findBySlug(String slug);
    List<Post> findByPublishedTrue();
    List<Post> findByAuthorId(Long authorId);

    @Query("SELECT p FROM Post p LEFT JOIN FETCH p.comments WHERE p.id = :id")
    Optional<Post> findPostWithComments(@Param("id") Long id);

    @Query("SELECT p FROM Post p LEFT JOIN FETCH p.tags WHERE p.id = :id")
    Optional<Post> findPostWithTags(@Param("id") Long id);

    @Query("SELECT p FROM Post p WHERE p.published = true ORDER BY p.createdAt DESC")
    Page<Post> findPublishedPosts(Pageable pageable);

    @Query("SELECT p FROM Post p WHERE p.published = true AND " +
           "(LOWER(p.title) LIKE LOWER(CONCAT('%', :keyword, '%')) OR " +
           "LOWER(p.content) LIKE LOWER(CONCAT('%', :keyword, '%')))")
    List<Post> searchPosts(@Param("keyword") String keyword);

    @Query("SELECT p FROM Post p JOIN p.tags t WHERE t.name = :tagName AND p.published = true")
    List<Post> findByTagName(@Param("tagName") String tagName);

    @Modifying
    @Transactional
    @Query("UPDATE Post p SET p.viewCount = p.viewCount + 1 WHERE p.id = :id")
    void incrementViewCount(@Param("id") Long id);
}
```

**CommentRepository.java:**
```java
@Repository
public interface CommentRepository extends JpaRepository<Comment, Long> {

    List<Comment> findByPostId(Long postId);
    List<Comment> findByAuthorId(Long authorId);

    @Query("SELECT c FROM Comment c WHERE c.post.id = :postId ORDER BY c.createdAt DESC")
    List<Comment> findByPostIdOrderByCreatedAtDesc(@Param("postId") Long postId);

    long countByPostId(Long postId);
}
```

**TagRepository.java:**
```java
@Repository
public interface TagRepository extends JpaRepository<Tag, Long> {

    Optional<Tag> findByName(String name);
    boolean existsByName(String name);

    @Query("SELECT t FROM Tag t LEFT JOIN FETCH t.posts WHERE t.id = :id")
    Optional<Tag> findTagWithPosts(@Param("id") Long id);
}
```

### Services

**UserService.java:**
```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional
    public User createUser(User user) {
        if (userRepository.existsByUsername(user.getUsername())) {
            throw new DuplicateUserException("Username already exists");
        }
        if (userRepository.existsByEmail(user.getEmail())) {
            throw new DuplicateUserException("Email already exists");
        }
        return userRepository.save(user);
    }

    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }

    @Transactional(readOnly = true)
    public User getUserByUsername(String username) {
        return userRepository.findByUsername(username)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + username));
    }

    @Transactional(readOnly = true)
    public User getUserWithPosts(Long id) {
        return userRepository.findUserWithPosts(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }

    @Transactional(readOnly = true)
    public List<User> getAllActiveUsers() {
        return userRepository.findByActiveTrue();
    }

    @Transactional
    public User updateUser(Long id, User userDetails) {
        User user = getUserById(id);
        user.setEmail(userDetails.getEmail());
        user.setBio(userDetails.getBio());
        return userRepository.save(user);
    }

    @Transactional
    public void deleteUser(Long id) {
        User user = getUserById(id);
        userRepository.delete(user);
    }

    @Transactional
    public void deactivateUser(Long id) {
        User user = getUserById(id);
        user.setActive(false);
        userRepository.save(user);
    }
}
```

**PostService.java:**
```java
@Service
public class PostService {

    private final PostRepository postRepository;
    private final UserRepository userRepository;
    private final TagRepository tagRepository;

    public PostService(PostRepository postRepository,
                       UserRepository userRepository,
                       TagRepository tagRepository) {
        this.postRepository = postRepository;
        this.userRepository = userRepository;
        this.tagRepository = tagRepository;
    }

    @Transactional
    public Post createPost(Long authorId, Post post, Set<String> tagNames) {
        User author = userRepository.findById(authorId)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + authorId));

        post.setAuthor(author);

        // Handle tags
        if (tagNames != null && !tagNames.isEmpty()) {
            for (String tagName : tagNames) {
                Tag tag = tagRepository.findByName(tagName)
                    .orElseGet(() -> {
                        Tag newTag = new Tag();
                        newTag.setName(tagName);
                        return tagRepository.save(newTag);
                    });
                post.addTag(tag);
            }
        }

        return postRepository.save(post);
    }

    @Transactional(readOnly = true)
    public Post getPostById(Long id) {
        return postRepository.findById(id)
            .orElseThrow(() -> new PostNotFoundException("Post not found: " + id));
    }

    @Transactional(readOnly = true)
    public Post getPostBySlug(String slug) {
        return postRepository.findBySlug(slug)
            .orElseThrow(() -> new PostNotFoundException("Post not found: " + slug));
    }

    @Transactional
    public Post getPostByIdAndIncrementViews(Long id) {
        Post post = getPostById(id);
        postRepository.incrementViewCount(id);
        post.setViewCount(post.getViewCount() + 1);
        return post;
    }

    @Transactional(readOnly = true)
    public Post getPostWithComments(Long id) {
        return postRepository.findPostWithComments(id)
            .orElseThrow(() -> new PostNotFoundException("Post not found: " + id));
    }

    @Transactional(readOnly = true)
    public Page<Post> getPublishedPosts(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return postRepository.findPublishedPosts(pageable);
    }

    @Transactional(readOnly = true)
    public List<Post> searchPosts(String keyword) {
        return postRepository.searchPosts(keyword);
    }

    @Transactional(readOnly = true)
    public List<Post> getPostsByTag(String tagName) {
        return postRepository.findByTagName(tagName);
    }

    @Transactional
    public Post updatePost(Long id, Post postDetails) {
        Post post = getPostById(id);
        post.setTitle(postDetails.getTitle());
        post.setContent(postDetails.getContent());
        post.setPublished(postDetails.getPublished());
        return postRepository.save(post);
    }

    @Transactional
    public void publishPost(Long id) {
        Post post = getPostById(id);
        post.setPublished(true);
        postRepository.save(post);
    }

    @Transactional
    public void deletePost(Long id) {
        Post post = getPostById(id);
        postRepository.delete(post);
    }
}
```

**CommentService.java:**
```java
@Service
public class CommentService {

    private final CommentRepository commentRepository;
    private final PostRepository postRepository;
    private final UserRepository userRepository;

    public CommentService(CommentRepository commentRepository,
                          PostRepository postRepository,
                          UserRepository userRepository) {
        this.commentRepository = commentRepository;
        this.postRepository = postRepository;
        this.userRepository = userRepository;
    }

    @Transactional
    public Comment createComment(Long postId, Long authorId, Comment comment) {
        Post post = postRepository.findById(postId)
            .orElseThrow(() -> new PostNotFoundException("Post not found: " + postId));
        User author = userRepository.findById(authorId)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + authorId));

        comment.setPost(post);
        comment.setAuthor(author);

        return commentRepository.save(comment);
    }

    @Transactional(readOnly = true)
    public List<Comment> getCommentsByPostId(Long postId) {
        return commentRepository.findByPostIdOrderByCreatedAtDesc(postId);
    }

    @Transactional(readOnly = true)
    public long getCommentCountByPostId(Long postId) {
        return commentRepository.countByPostId(postId);
    }

    @Transactional
    public void deleteComment(Long id) {
        Comment comment = commentRepository.findById(id)
            .orElseThrow(() -> new CommentNotFoundException("Comment not found: " + id));
        commentRepository.delete(comment);
    }
}
```

### REST Controllers

**UserController.java:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User createdUser = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }

    @GetMapping("/{id}/posts")
    public ResponseEntity<User> getUserWithPosts(@PathVariable Long id) {
        User user = userService.getUserWithPosts(id);
        return ResponseEntity.ok(user);
    }

    @GetMapping
    public ResponseEntity<List<User>> getAllActiveUsers() {
        List<User> users = userService.getAllActiveUsers();
        return ResponseEntity.ok(users);
    }

    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id,
                                           @Valid @RequestBody User user) {
        User updatedUser = userService.updateUser(id, user);
        return ResponseEntity.ok(updatedUser);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

**PostController.java:**
```java
@RestController
@RequestMapping("/api/posts")
public class PostController {

    private final PostService postService;

    public PostController(PostService postService) {
        this.postService = postService;
    }

    @PostMapping
    public ResponseEntity<Post> createPost(
            @RequestParam Long authorId,
            @Valid @RequestBody PostRequest request) {
        Post post = new Post();
        post.setTitle(request.getTitle());
        post.setContent(request.getContent());

        Post createdPost = postService.createPost(authorId, post, request.getTags());
        return ResponseEntity.status(HttpStatus.CREATED).body(createdPost);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Post> getPostById(@PathVariable Long id) {
        Post post = postService.getPostByIdAndIncrementViews(id);
        return ResponseEntity.ok(post);
    }

    @GetMapping("/slug/{slug}")
    public ResponseEntity<Post> getPostBySlug(@PathVariable String slug) {
        Post post = postService.getPostBySlug(slug);
        return ResponseEntity.ok(post);
    }

    @GetMapping
    public ResponseEntity<Page<Post>> getPublishedPosts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        Page<Post> posts = postService.getPublishedPosts(page, size);
        return ResponseEntity.ok(posts);
    }

    @GetMapping("/search")
    public ResponseEntity<List<Post>> searchPosts(@RequestParam String keyword) {
        List<Post> posts = postService.searchPosts(keyword);
        return ResponseEntity.ok(posts);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Post> updatePost(@PathVariable Long id,
                                           @Valid @RequestBody Post post) {
        Post updatedPost = postService.updatePost(id, post);
        return ResponseEntity.ok(updatedPost);
    }

    @PatchMapping("/{id}/publish")
    public ResponseEntity<Void> publishPost(@PathVariable Long id) {
        postService.publishPost(id);
        return ResponseEntity.noContent().build();
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletePost(@PathVariable Long id) {
        postService.deletePost(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Exception Handling

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(DuplicateUserException.class)
    public ResponseEntity<ErrorResponse> handleDuplicateUser(DuplicateUserException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An error occurred: " + ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

## 🎯 Key Takeaways

1. **Spring Data JPA** eliminates boilerplate database code
2. **Entities** map Java objects to database tables with annotations
3. **Relationships** (@OneToMany, @ManyToOne, etc.) are handled automatically
4. **JpaRepository** provides CRUD operations out of the box
5. **Derived queries** generate SQL from method names
6. **@Query** allows custom JPQL or native SQL
7. **@Transactional** ensures data consistency
8. **Pagination and sorting** are built-in
9. **Multiple datasources** can be configured when needed
10. **Lazy loading** improves performance for related entities

## 📚 Comparison Summary: Room vs JPA

| Feature | Room | JPA |
|---------|------|-----|
| Setup complexity | Simple | Moderate |
| Relationships | Manual | Automatic |
| Query generation | Limited | Extensive |
| Lazy loading | No | Yes |
| Caching | Manual | Built-in |
| Transaction support | Basic | Advanced |
| Migration tools | Room migrations | Flyway, Liquibase |
| Performance | Mobile-optimized | Enterprise-grade |
| Learning curve | Easy | Moderate |

## 🚀 Next Steps

Now that you understand Spring Data JPA, you can:
- Build complex database schemas with relationships
- Implement efficient queries with pagination
- Use transactions for data consistency
- Configure multiple datasources
- Integrate with Spring Security for user management

Continue exploring Spring Boot features to build production-ready applications!
