# Spring Core & Dependency Injection

Learn how Spring's Inversion of Control (IoC) container and Dependency Injection work, with comparisons to Dagger/Hilt.

## 🎯 Overview

Spring Framework's core feature is **Dependency Injection (DI)**, which is similar to Dagger/Hilt in Android. If you've used Dagger or Hilt, you already understand the fundamental concepts!

## 📋 Quick Comparison: Dagger/Hilt vs Spring

| Concept | Dagger/Hilt (Android) | Spring Framework |
|---------|----------------------|------------------|
| Container | Hilt Component | ApplicationContext |
| Define Dependencies | `@Module` with `@Provides` | `@Configuration` with `@Bean` |
| Inject Dependencies | `@Inject` | `@Autowired` or `@Inject` |
| Scope | `@Singleton`, `@ActivityScoped` | `@Singleton`, `@Prototype`, `@RequestScoped` |
| Component | `@HiltAndroidApp`, `@AndroidEntryPoint` | `@SpringBootApplication`, `@Component` |
| Qualifier | `@Named`, `@Qualifier` | `@Qualifier`, `@Primary` |

## 1. What is Dependency Injection?

### The Problem

**Without DI (Tight Coupling):**
```java
public class UserService {
    private UserRepository repository;

    public UserService() {
        // Tightly coupled to implementation
        this.repository = new UserRepositoryImpl();
    }

    public User findUser(Long id) {
        return repository.findById(id);
    }
}
```

**Issues:**
- Hard to test (can't mock repository)
- Hard to change implementation
- Violates Single Responsibility Principle

### The Solution: Dependency Injection

**With DI:**
```java
public class UserService {
    private final UserRepository repository;

    // Dependencies injected via constructor
    public UserService(UserRepository repository) {
        this.repository = repository;
    }

    public User findUser(Long id) {
        return repository.findById(id);
    }
}
```

**Benefits:**
- Easy to test (inject mocks)
- Loose coupling
- Flexible implementation swapping

## 2. Spring IoC Container

The Spring IoC Container manages object creation and dependency injection.

```
┌─────────────────────────────────────────────┐
│        Spring IoC Container                  │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Bean A  │  │  Bean B  │  │  Bean C  │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
│       │             │              │         │
│       └─────────────┼──────────────┘         │
│                     │                        │
│              Dependencies Managed            │
│              Lifecycle Managed               │
└─────────────────────────────────────────────┘
```

### Key Concepts

**Bean:** An object managed by Spring IoC Container
**ApplicationContext:** The Spring IoC Container interface
**BeanFactory:** Lower-level container interface (rarely used directly)

## 3. Dependency Injection Types

### Constructor Injection (Recommended)

**Spring:**
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    // Constructor injection - Spring automatically injects dependencies
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public void registerUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}
```

**Hilt (Android):**
```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository,
    private val emailService: EmailService
) : ViewModel() {

    fun registerUser(user: User) {
        userRepository.save(user)
        emailService.sendWelcomeEmail(user)
    }
}
```

**Why Constructor Injection?**
- Dependencies are immutable (final/val)
- Required dependencies are explicit
- Easy to test
- Null safety

### Field Injection (Not Recommended)

**Spring:**
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // Field injection

    @Autowired
    private EmailService emailService;

    // No-arg constructor required
}
```

**Issues:**
- Can't make fields final
- Harder to test
- Hidden dependencies
- Can lead to circular dependencies

### Setter Injection (For Optional Dependencies)

**Spring:**
```java
@Service
public class UserService {
    private UserRepository userRepository;
    private EmailService emailService;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Autowired(required = false)  // Optional dependency
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

## 4. Component Scanning and Stereotype Annotations

### Spring Stereotypes

```java
@Component      // Generic component
@Service        // Business logic layer
@Repository     // Data access layer
@Controller     // Web controller
@RestController // REST API controller
@Configuration  // Configuration class
```

### Component Scanning

**Enable component scanning:**
```java
@SpringBootApplication  // Includes @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**How it works:**
```
src/main/java/
└── com.example.myapp/
    ├── Application.java              [@SpringBootApplication]
    ├── controller/
    │   └── UserController.java       [@RestController] ✓ Scanned
    ├── service/
    │   └── UserService.java          [@Service] ✓ Scanned
    └── repository/
        └── UserRepository.java       [@Repository] ✓ Scanned
```

### Example: Full Stack

**Repository Layer:**
```java
@Repository
public class UserRepositoryImpl implements UserRepository {

    private final JdbcTemplate jdbcTemplate;

    public UserRepositoryImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public User findById(Long id) {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            new Object[]{id},
            new UserRowMapper()
        );
    }

    @Override
    public void save(User user) {
        jdbcTemplate.update(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            user.getName(), user.getEmail()
        );
    }
}
```

**Service Layer:**
```java
@Service
public class UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public User getUser(Long id) {
        return userRepository.findById(id);
    }

    public void registerUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}
```

**Controller Layer:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }

    @PostMapping
    public void createUser(@RequestBody User user) {
        userService.registerUser(user);
    }
}
```

## 5. Java-based Configuration

### @Configuration and @Bean

**Dagger Module (Android):**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
}
```

**Spring Configuration:**
```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
        return restTemplate;
    }

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### When to use @Bean vs @Component?

**Use @Component (and stereotypes):**
- For your own classes
- When you can add annotations to the class

**Use @Bean:**
- For third-party library classes
- When you need complex initialization
- When you need multiple instances with different configurations

## 6. Bean Scopes

```java
@Singleton    // Default - one instance per container
@Prototype    // New instance every time
@Request      // One instance per HTTP request
@Session      // One instance per HTTP session
@Application  // One instance per ServletContext
```

**Examples:**

```java
@Service
@Scope("singleton")  // Default, can omit
public class UserService {
    // Single instance shared across application
}

@Component
@Scope("prototype")
public class ReportGenerator {
    // New instance created for each injection
}

@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestContext {
    // New instance for each HTTP request
}
```

**Comparison with Hilt:**

| Hilt Scope | Spring Scope | Lifetime |
|------------|--------------|----------|
| `@Singleton` | `@Singleton` | Application lifecycle |
| `@ActivityScoped` | `@RequestScoped` | Request lifecycle |
| `@ViewModelScoped` | `@Prototype` | New instance each time |

## 7. Qualifiers

When you have multiple beans of the same type, use qualifiers to specify which one to inject.

**Hilt (Android):**
```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptor

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class LoggingInterceptor

@Module
@InstallIn(SingletonComponent::class)
object InterceptorModule {

    @Provides
    @AuthInterceptor
    fun provideAuthInterceptor(): Interceptor {
        return AuthInterceptor()
    }

    @Provides
    @LoggingInterceptor
    fun provideLoggingInterceptor(): Interceptor {
        return HttpLoggingInterceptor()
    }
}
```

**Spring:**
```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Qualifier("mysqlDataSource")
    public DataSource mysqlDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/mydb")
            .build();
    }

    @Bean
    @Qualifier("postgresDataSource")
    public DataSource postgresDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://localhost:5432/mydb")
            .build();
    }
}

@Service
public class UserService {

    private final DataSource dataSource;

    public UserService(@Qualifier("mysqlDataSource") DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

### @Primary Annotation

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Primary  // This one will be injected by default
    public DataSource mysqlDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/mydb")
            .build();
    }

    @Bean
    public DataSource postgresDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://localhost:5432/mydb")
            .build();
    }
}

@Service
public class UserService {
    // mysqlDataSource will be injected (it's @Primary)
    public UserService(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

## 8. Bean Lifecycle

```
Bean Lifecycle Flow:

1. Instantiation
   ↓
2. Populate Properties (DI)
   ↓
3. BeanNameAware.setBeanName()
   ↓
4. BeanFactoryAware.setBeanFactory()
   ↓
5. ApplicationContextAware.setApplicationContext()
   ↓
6. @PostConstruct / InitializingBean.afterPropertiesSet()
   ↓
7. Custom init-method
   ↓
8. Bean Ready for Use
   ↓
9. @PreDestroy / DisposableBean.destroy()
   ↓
10. Custom destroy-method
```

**Example:**
```java
@Component
public class DatabaseConnection {

    private Connection connection;

    @PostConstruct
    public void init() {
        System.out.println("Opening database connection...");
        // Initialize connection
        connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Closing database connection...");
        // Clean up resources
        try {
            if (connection != null) {
                connection.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public Connection getConnection() {
        return connection;
    }
}
```

**Alternative with interfaces:**
```java
@Component
public class DatabaseConnection implements InitializingBean, DisposableBean {

    private Connection connection;

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Opening database connection...");
        connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Closing database connection...");
        if (connection != null) {
            connection.close();
        }
    }
}
```

## 9. Circular Dependencies

**Problem:**
```java
@Service
public class ServiceA {
    private final ServiceB serviceB;

    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

@Service
public class ServiceB {
    private final ServiceA serviceA;

    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;  // Circular dependency!
    }
}
```

**Solutions:**

1. **Redesign (Best)** - Avoid circular dependencies
2. **Use @Lazy:**
```java
@Service
public class ServiceA {
    private final ServiceB serviceB;

    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

3. **Setter Injection:**
```java
@Service
public class ServiceA {
    private ServiceB serviceB;

    @Autowired
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

## 10. Practical Example: E-commerce Application

```java
// Domain Model
public class Product {
    private Long id;
    private String name;
    private BigDecimal price;
    // getters, setters, constructors
}

// Repository Layer
@Repository
public class ProductRepository {

    private final JdbcTemplate jdbcTemplate;

    public ProductRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Optional<Product> findById(Long id) {
        try {
            Product product = jdbcTemplate.queryForObject(
                "SELECT * FROM products WHERE id = ?",
                new Object[]{id},
                (rs, rowNum) -> new Product(
                    rs.getLong("id"),
                    rs.getString("name"),
                    rs.getBigDecimal("price")
                )
            );
            return Optional.ofNullable(product);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    public List<Product> findAll() {
        return jdbcTemplate.query(
            "SELECT * FROM products",
            (rs, rowNum) -> new Product(
                rs.getLong("id"),
                rs.getString("name"),
                rs.getBigDecimal("price")
            )
        );
    }

    public void save(Product product) {
        jdbcTemplate.update(
            "INSERT INTO products (name, price) VALUES (?, ?)",
            product.getName(), product.getPrice()
        );
    }
}

// Service Layer
@Service
public class ProductService {

    private final ProductRepository productRepository;
    private final PricingService pricingService;

    public ProductService(ProductRepository productRepository,
                         PricingService pricingService) {
        this.productRepository = productRepository;
        this.pricingService = pricingService;
    }

    public Product getProduct(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
    }

    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }

    public Product createProduct(Product product) {
        // Apply pricing rules
        BigDecimal finalPrice = pricingService.calculatePrice(product);
        product.setPrice(finalPrice);

        productRepository.save(product);
        return product;
    }
}

// Pricing Service
@Service
public class PricingService {

    private final TaxCalculator taxCalculator;

    public PricingService(TaxCalculator taxCalculator) {
        this.taxCalculator = taxCalculator;
    }

    public BigDecimal calculatePrice(Product product) {
        BigDecimal basePrice = product.getPrice();
        BigDecimal tax = taxCalculator.calculateTax(basePrice);
        return basePrice.add(tax);
    }
}

// Tax Calculator
@Component
public class TaxCalculator {

    private static final BigDecimal TAX_RATE = new BigDecimal("0.10"); // 10%

    public BigDecimal calculateTax(BigDecimal amount) {
        return amount.multiply(TAX_RATE);
    }
}

// Controller Layer
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.getProduct(id);
    }

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.createProduct(product);
    }
}

// Configuration
@Configuration
public class DatabaseConfig {

    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/ecommerce")
            .username("root")
            .password("password")
            .build();
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

## 11. Dependency Injection Flow Diagram

```
Application Startup
        │
        ▼
┌───────────────────┐
│ @SpringBootApp    │
│ Main Class        │
└────────┬──────────┘
         │
         ▼
┌─────────────────────────────────┐
│  Spring IoC Container           │
│                                  │
│  1. Component Scanning           │
│     - Find @Component classes    │
│     - Find @Configuration        │
│                                  │
│  2. Bean Definitions             │
│     - Create bean metadata       │
│                                  │
│  3. Dependency Resolution        │
│     - Analyze dependencies       │
│     - Determine injection order  │
│                                  │
│  4. Bean Instantiation           │
│     ProductController            │
│          │                       │
│          ├─> ProductService      │
│          │        │              │
│          │        ├─> ProductRepo│
│          │        │              │
│          │        └─> PricingServ│
│          │                │      │
│          │                └─> Tax│
│                                  │
│  5. Dependency Injection         │
│     - Constructor injection      │
│     - Property population        │
│                                  │
│  6. Lifecycle Callbacks          │
│     - @PostConstruct             │
│                                  │
│  7. Beans Ready                  │
└─────────────────────────────────┘
         │
         ▼
   Application Running
```

## 📊 Summary Comparison

| Feature | Dagger/Hilt | Spring |
|---------|-------------|--------|
| **Container** | Compile-time generation | Runtime container |
| **Performance** | Faster (compile-time) | Slower startup (runtime) |
| **Error Detection** | Compile-time | Runtime |
| **Learning Curve** | Steeper | Gentler |
| **Configuration** | More boilerplate | Less boilerplate |
| **Scopes** | Activity, Fragment, ViewModel | Request, Session, Singleton |
| **Use Case** | Mobile (resource-constrained) | Server (more resources) |

## 🎯 Key Takeaways

1. **Dependency Injection** reduces coupling and improves testability
2. **Constructor Injection** is the preferred method in Spring
3. **@Component** and stereotypes enable auto-discovery
4. **@Configuration** and **@Bean** for third-party dependencies
5. **Qualifiers** resolve ambiguity when multiple beans exist
6. **Scopes** control bean lifecycle and instances
7. Spring DI is similar to Dagger/Hilt but with runtime resolution

## 🚀 Next Steps

Now that you understand Spring Core and Dependency Injection, let's explore Spring Boot!

Continue to [Section 3: Spring Boot Fundamentals](./03-spring-boot-fundamentals.md)
