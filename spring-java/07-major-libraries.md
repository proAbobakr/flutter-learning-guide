# Major Spring Ecosystem Libraries

Explore the rich Spring ecosystem and essential third-party libraries for building production-ready applications.

## 🎯 Overview

The Spring ecosystem offers a comprehensive set of libraries for various needs, from data access to cloud-native microservices. As an Android developer, you're familiar with libraries like Room, Retrofit, and WorkManager. Spring has similar and more powerful alternatives for server-side development.

## 📋 Quick Comparison: Android vs Spring Libraries

| Android Library | Spring Equivalent | Purpose |
|-----------------|-------------------|---------|
| Room | Spring Data JPA | Database ORM |
| SharedPreferences | Spring Data Redis | Key-value storage |
| Retrofit | RestTemplate/WebClient | HTTP client |
| WorkManager | Spring Batch | Background jobs |
| RxJava/Flow | Spring WebFlux | Reactive programming |
| JUnit + Mockito | JUnit + Mockito + Spring Test | Testing |
| Timber | SLF4J + Logback | Logging |
| Moshi/Gson | Jackson | JSON serialization |

## 1. Spring Data

Spring Data simplifies data access across various databases. Think of it as Room on steroids!

### Spring Data JPA (SQL Databases)

**Android Room:**
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getUserById(id: Long): User?

    @Query("SELECT * FROM users WHERE email = :email")
    suspend fun findByEmail(email: String): User?

    @Insert
    suspend fun insert(user: User)
}
```

**Spring Data JPA:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Method name becomes query automatically!
    Optional<User> findByEmail(String email);

    List<User> findByAgeGreaterThan(int age);

    @Query("SELECT u FROM User u WHERE u.lastName = ?1")
    List<User> findByLastName(String lastName);

    // Pagination and sorting built-in
    Page<User> findByActive(boolean active, Pageable pageable);
}
```

**Entity Definition:**
```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String firstName;

    @Column(nullable = false)
    private String lastName;

    private int age;

    private boolean active = true;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    // One user can have many orders
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders = new ArrayList<>();

    // Constructors, getters, setters
}
```

**Usage in Service:**
```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    public List<User> getActiveUsers() {
        return userRepository.findByActive(true);
    }

    // Pagination example
    public Page<User> getUsers(int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("lastName"));
        return userRepository.findAll(pageable);
    }
}
```

**Configuration (application.properties):**
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### Spring Data MongoDB (NoSQL)

**Entity:**
```java
@Document(collection = "products")
public class Product {

    @Id
    private String id;

    @Indexed(unique = true)
    private String sku;

    private String name;
    private String description;
    private BigDecimal price;

    private List<String> tags;
    private Map<String, Object> attributes;

    @CreatedDate
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;

    // Constructors, getters, setters
}
```

**Repository:**
```java
@Repository
public interface ProductRepository extends MongoRepository<Product, String> {

    List<Product> findByNameContaining(String name);

    List<Product> findByPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);

    List<Product> findByTagsContaining(String tag);

    @Query("{ 'price': { $gte: ?0, $lte: ?1 } }")
    List<Product> findProductsInPriceRange(BigDecimal min, BigDecimal max);

    // Full text search
    @Query("{ $text: { $search: ?0 } }")
    List<Product> searchProducts(String searchText);
}
```

**Configuration:**
```properties
spring.data.mongodb.uri=mongodb://localhost:27017/ecommerce
spring.data.mongodb.database=ecommerce
```

### Spring Data Redis (Caching)

**Android SharedPreferences:**
```kotlin
val prefs = context.getSharedPreferences("user_prefs", Context.MODE_PRIVATE)
prefs.edit().putString("user_token", token).apply()
val token = prefs.getString("user_token", null)
```

**Spring Data Redis:**
```java
@Configuration
@EnableCaching
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```

**Using Redis for Caching:**
```java
@Service
public class ProductService {

    private final ProductRepository productRepository;
    private final RedisTemplate<String, Object> redisTemplate;

    public ProductService(ProductRepository productRepository,
                         RedisTemplate<String, Object> redisTemplate) {
        this.productRepository = productRepository;
        this.redisTemplate = redisTemplate;
    }

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(String id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
    }

    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }

    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(String id) {
        productRepository.deleteById(id);
    }

    // Manual cache operations
    public void cacheUserSession(String sessionId, UserSession session) {
        redisTemplate.opsForValue().set(
            "session:" + sessionId,
            session,
            Duration.ofMinutes(30)
        );
    }

    public UserSession getUserSession(String sessionId) {
        return (UserSession) redisTemplate.opsForValue()
            .get("session:" + sessionId);
    }
}
```

**Configuration:**
```properties
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=
spring.cache.type=redis
spring.cache.redis.time-to-live=600000
```

### Spring Data Elasticsearch (Search)

**Document:**
```java
@Document(indexName = "articles")
public class Article {

    @Id
    private String id;

    @Field(type = FieldType.Text, analyzer = "standard")
    private String title;

    @Field(type = FieldType.Text, analyzer = "standard")
    private String content;

    @Field(type = FieldType.Keyword)
    private String author;

    @Field(type = FieldType.Keyword)
    private List<String> tags;

    @Field(type = FieldType.Date)
    private Instant publishedAt;

    // Constructors, getters, setters
}
```

**Repository:**
```java
@Repository
public interface ArticleRepository extends ElasticsearchRepository<Article, String> {

    List<Article> findByTitle(String title);

    List<Article> findByAuthor(String author);

    List<Article> findByTagsContaining(String tag);

    @Query("{\"bool\": {\"must\": [{\"match\": {\"title\": \"?0\"}}]}}")
    List<Article> searchByTitle(String title);
}
```

**Service with Advanced Search:**
```java
@Service
public class ArticleSearchService {

    private final ElasticsearchRestTemplate elasticsearchTemplate;

    public ArticleSearchService(ElasticsearchRestTemplate template) {
        this.elasticsearchTemplate = template;
    }

    public List<Article> searchArticles(String query) {
        Criteria criteria = new Criteria("title")
            .matches(query)
            .or("content").matches(query);

        Query searchQuery = new CriteriaQuery(criteria);

        SearchHits<Article> searchHits = elasticsearchTemplate
            .search(searchQuery, Article.class);

        return searchHits.stream()
            .map(SearchHit::getContent)
            .collect(Collectors.toList());
    }
}
```

**Configuration:**
```properties
spring.elasticsearch.uris=http://localhost:9200
spring.elasticsearch.username=elastic
spring.elasticsearch.password=changeme
```

## 2. Spring Cloud (Microservices)

Building distributed systems? Spring Cloud has you covered!

### Service Discovery with Eureka

**Eureka Server:**
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

**Eureka Server Configuration (application.yml):**
```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

**Eureka Client (Service):**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class ProductServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}
```

**Client Configuration:**
```yaml
spring:
  application:
    name: product-service

server:
  port: 8081

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

### Spring Cloud Config Server

Centralized configuration management (like Firebase Remote Config).

**Config Server:**
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

**Config Server Configuration:**
```yaml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo
          default-label: main
          search-paths: '{application}'

server:
  port: 8888
```

**Config Client:**
```yaml
spring:
  application:
    name: product-service
  config:
    import: "optional:configserver:http://localhost:8888"
  cloud:
    config:
      fail-fast: true
      retry:
        max-attempts: 5
```

**Refreshable Configuration:**
```java
@RestController
@RefreshScope  // Allows dynamic refresh
public class ProductController {

    @Value("${product.max-price:1000}")
    private BigDecimal maxPrice;

    @Value("${product.discount-rate:0.1}")
    private double discountRate;

    @GetMapping("/config")
    public Map<String, Object> getConfig() {
        return Map.of(
            "maxPrice", maxPrice,
            "discountRate", discountRate
        );
    }
}
```

### Spring Cloud Gateway (API Gateway)

**Gateway Configuration:**
```java
@SpringBootApplication
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

**Route Configuration (application.yml):**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service
          uri: lb://PRODUCT-SERVICE  # Load balanced
          predicates:
            - Path=/api/products/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: productServiceCircuitBreaker
                fallbackUri: forward:/fallback/products

        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

**Custom Global Filter:**
```java
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        if (!request.getHeaders().containsKey("Authorization")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        String token = request.getHeaders().getFirst("Authorization");
        // Validate token

        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;  // High priority
    }
}
```

### Circuit Breaker with Resilience4j

```java
@Service
public class ProductService {

    private final RestTemplate restTemplate;

    @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
    @Retry(name = "productService")
    @RateLimiter(name = "productService")
    public Product getProductFromExternalService(String id) {
        return restTemplate.getForObject(
            "http://external-api/products/" + id,
            Product.class
        );
    }

    public Product getProductFallback(String id, Exception e) {
        // Return cached or default product
        return Product.builder()
            .id(id)
            .name("Product Unavailable")
            .price(BigDecimal.ZERO)
            .build();
    }
}
```

**Resilience4j Configuration:**
```yaml
resilience4j:
  circuitbreaker:
    instances:
      productService:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 5
        permitted-number-of-calls-in-half-open-state: 3
        wait-duration-in-open-state: 10s
        failure-rate-threshold: 50

  retry:
    instances:
      productService:
        max-attempts: 3
        wait-duration: 1s

  ratelimiter:
    instances:
      productService:
        limit-for-period: 10
        limit-refresh-period: 1s
        timeout-duration: 0
```

## 3. Spring Batch (Background Jobs)

Similar to Android WorkManager but for server-side batch processing.

**Android WorkManager:**
```kotlin
val workRequest = OneTimeWorkRequestBuilder<DataSyncWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(workRequest)
```

**Spring Batch Job:**
```java
@Configuration
@EnableBatchProcessing
public class BatchConfiguration {

    @Bean
    public Job importUserJob(JobRepository jobRepository,
                            Step importUserStep) {
        return new JobBuilder("importUserJob", jobRepository)
            .start(importUserStep)
            .build();
    }

    @Bean
    public Step importUserStep(JobRepository jobRepository,
                              PlatformTransactionManager transactionManager,
                              ItemReader<User> reader,
                              ItemProcessor<User, User> processor,
                              ItemWriter<User> writer) {
        return new StepBuilder("importUserStep", jobRepository)
            .<User, User>chunk(100, transactionManager)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
    }

    @Bean
    public ItemReader<User> reader() {
        return new FlatFileItemReaderBuilder<User>()
            .name("userItemReader")
            .resource(new ClassPathResource("users.csv"))
            .delimited()
            .names("firstName", "lastName", "email")
            .targetType(User.class)
            .build();
    }

    @Bean
    public ItemProcessor<User, User> processor() {
        return user -> {
            // Transform data
            user.setEmail(user.getEmail().toLowerCase());
            user.setActive(true);
            return user;
        };
    }

    @Bean
    public ItemWriter<User> writer(UserRepository repository) {
        return new RepositoryItemWriterBuilder<User>()
            .repository(repository)
            .methodName("save")
            .build();
    }
}
```

**Job Launcher:**
```java
@Service
public class BatchJobService {

    private final JobLauncher jobLauncher;
    private final Job importUserJob;

    public BatchJobService(JobLauncher jobLauncher, Job importUserJob) {
        this.jobLauncher = jobLauncher;
        this.importUserJob = importUserJob;
    }

    public void runUserImportJob() throws Exception {
        JobParameters params = new JobParametersBuilder()
            .addString("JobID", String.valueOf(System.currentTimeMillis()))
            .toJobParameters();

        JobExecution execution = jobLauncher.run(importUserJob, params);
        System.out.println("Exit Status: " + execution.getStatus());
    }
}
```

**Scheduled Batch Job:**
```java
@Component
public class ScheduledJobs {

    private final JobLauncher jobLauncher;
    private final Job dailyReportJob;

    public ScheduledJobs(JobLauncher jobLauncher, Job dailyReportJob) {
        this.jobLauncher = jobLauncher;
        this.dailyReportJob = dailyReportJob;
    }

    @Scheduled(cron = "0 0 2 * * ?")  // Run at 2 AM daily
    public void runDailyReport() throws Exception {
        JobParameters params = new JobParametersBuilder()
            .addLocalDateTime("runDate", LocalDateTime.now())
            .toJobParameters();

        jobLauncher.run(dailyReportJob, params);
    }
}
```

## 4. Spring WebFlux (Reactive Programming)

**Android RxJava/Flow:**
```kotlin
fun getUsers(): Flow<User> = flow {
    val users = userRepository.getAllUsers()
    users.forEach { emit(it) }
}

viewModelScope.launch {
    userService.getUsers()
        .catch { e -> handleError(e) }
        .collect { user -> updateUI(user) }
}
```

**Spring WebFlux:**
```java
@RestController
@RequestMapping("/api/reactive/users")
public class ReactiveUserController {

    private final ReactiveUserRepository userRepository;

    public ReactiveUserController(ReactiveUserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping
    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }

    @GetMapping("/{id}")
    public Mono<User> getUserById(@PathVariable String id) {
        return userRepository.findById(id)
            .switchIfEmpty(Mono.error(new UserNotFoundException(id)));
    }

    @PostMapping
    public Mono<User> createUser(@RequestBody User user) {
        return userRepository.save(user);
    }

    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamUsers() {
        return userRepository.findAll()
            .delayElements(Duration.ofSeconds(1));
    }
}
```

**Reactive Repository:**
```java
@Repository
public interface ReactiveUserRepository
    extends ReactiveMongoRepository<User, String> {

    Flux<User> findByLastName(String lastName);

    Mono<User> findByEmail(String email);

    @Query("{ 'age': { $gte: ?0, $lte: ?1 } }")
    Flux<User> findByAgeBetween(int minAge, int maxAge);
}
```

**Reactive Service with WebClient:**
```java
@Service
public class ReactiveProductService {

    private final WebClient webClient;

    public ReactiveProductService(WebClient.Builder builder) {
        this.webClient = builder
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }

    public Mono<Product> getProduct(String id) {
        return webClient.get()
            .uri("/products/{id}", id)
            .retrieve()
            .bodyToMono(Product.class)
            .timeout(Duration.ofSeconds(5))
            .retry(3);
    }

    public Flux<Product> getAllProducts() {
        return webClient.get()
            .uri("/products")
            .retrieve()
            .bodyToFlux(Product.class);
    }

    public Mono<Product> createProduct(Product product) {
        return webClient.post()
            .uri("/products")
            .bodyValue(product)
            .retrieve()
            .bodyToMono(Product.class);
    }
}
```

**Reactive Operators:**
```java
@Service
public class ReactiveDataService {

    public Flux<User> processUsers() {
        return userRepository.findAll()
            .filter(user -> user.isActive())
            .map(user -> {
                user.setEmail(user.getEmail().toLowerCase());
                return user;
            })
            .flatMap(user -> enrichUserData(user))
            .take(10)
            .timeout(Duration.ofSeconds(30));
    }

    public Mono<UserStats> getUserStats() {
        return userRepository.findAll()
            .collectList()
            .map(users -> {
                UserStats stats = new UserStats();
                stats.setTotalUsers(users.size());
                stats.setActiveUsers((int) users.stream()
                    .filter(User::isActive)
                    .count());
                return stats;
            });
    }

    private Mono<User> enrichUserData(User user) {
        return externalApiService.getUserDetails(user.getId())
            .map(details -> {
                user.setAdditionalInfo(details);
                return user;
            });
    }
}
```

## 5. Spring Integration

Enterprise Integration Patterns for message-driven architectures.

**Message Gateway:**
```java
@MessagingGateway
public interface OrderGateway {

    @Gateway(requestChannel = "orderInputChannel")
    void submitOrder(Order order);

    @Gateway(requestChannel = "orderProcessChannel", replyTimeout = 5000)
    OrderConfirmation processOrder(Order order);
}
```

**Integration Flow:**
```java
@Configuration
public class OrderIntegrationConfig {

    @Bean
    public IntegrationFlow orderProcessingFlow() {
        return IntegrationFlows
            .from("orderInputChannel")
            .filter(Order.class, order -> order.getAmount() > 0)
            .transform(Order.class, this::enrichOrder)
            .route(Order.class, order -> {
                if (order.getAmount() > 1000) {
                    return "highValueChannel";
                } else {
                    return "standardChannel";
                }
            })
            .get();
    }

    @Bean
    public IntegrationFlow highValueOrderFlow() {
        return IntegrationFlows
            .from("highValueChannel")
            .handle("orderService", "processHighValueOrder")
            .channel("orderOutputChannel")
            .get();
    }

    @Bean
    public IntegrationFlow standardOrderFlow() {
        return IntegrationFlows
            .from("standardChannel")
            .handle("orderService", "processStandardOrder")
            .channel("orderOutputChannel")
            .get();
    }

    private Order enrichOrder(Order order) {
        // Add timestamp, validation, etc.
        order.setProcessedAt(Instant.now());
        return order;
    }
}
```

**File Integration:**
```java
@Configuration
public class FileIntegrationConfig {

    @Bean
    public IntegrationFlow fileReadingFlow() {
        return IntegrationFlows
            .from(Files.inboundAdapter(new File("/input"))
                    .patternFilter("*.csv"),
                e -> e.poller(Pollers.fixedDelay(5000)))
            .transform(Files.toStringTransformer())
            .split(s -> s.delimiters("\n"))
            .transform(String.class, this::parseLine)
            .aggregate()
            .channel("processedChannel")
            .get();
    }

    private Record parseLine(String line) {
        // Parse CSV line
        return new Record(line);
    }
}
```

## 6. Testing Libraries

### JUnit 5

**Android Test:**
```kotlin
@Test
fun `test user creation`() {
    val user = User("John", "Doe")
    assertEquals("John", user.firstName)
    assertEquals("Doe", user.lastName)
}
```

**JUnit 5:**
```java
@SpringBootTest
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("Should create user successfully")
    void testCreateUser() {
        // Given
        User user = new User();
        user.setEmail("test@example.com");
        user.setFirstName("John");
        user.setLastName("Doe");

        // When
        User created = userService.createUser(user);

        // Then
        assertNotNull(created.getId());
        assertEquals("test@example.com", created.getEmail());
    }

    @Test
    void testGetUserById() {
        // Given
        User saved = userRepository.save(new User("test@test.com", "Jane", "Smith"));

        // When
        Optional<User> found = userService.getUserById(saved.getId());

        // Then
        assertTrue(found.isPresent());
        assertEquals("Jane", found.get().getFirstName());
    }

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @AfterEach
    void tearDown() {
        userRepository.deleteAll();
    }
}
```

**Parameterized Tests:**
```java
class ValidationTest {

    @ParameterizedTest
    @ValueSource(strings = {"test@example.com", "user@test.org", "admin@company.net"})
    void testValidEmails(String email) {
        assertTrue(EmailValidator.isValid(email));
    }

    @ParameterizedTest
    @CsvSource({
        "John, Doe, 25",
        "Jane, Smith, 30",
        "Bob, Johnson, 35"
    })
    void testUserCreation(String firstName, String lastName, int age) {
        User user = new User(firstName, lastName, age);
        assertEquals(firstName, user.getFirstName());
        assertEquals(lastName, user.getLastName());
        assertEquals(age, user.getAge());
    }

    @ParameterizedTest
    @MethodSource("provideUsers")
    void testUserValidation(User user, boolean expected) {
        assertEquals(expected, UserValidator.isValid(user));
    }

    private static Stream<Arguments> provideUsers() {
        return Stream.of(
            Arguments.of(new User("John", "Doe", 25), true),
            Arguments.of(new User("", "Doe", 25), false),
            Arguments.of(new User("John", "", 25), false)
        );
    }
}
```

### Mockito

**Android Mockito:**
```kotlin
@Test
fun `test user repository`() {
    val mockRepo = mock<UserRepository>()
    `when`(mockRepo.getUserById(1)).thenReturn(User("John", "Doe"))

    val service = UserService(mockRepo)
    val user = service.getUser(1)

    assertEquals("John", user.firstName)
    verify(mockRepo).getUserById(1)
}
```

**Spring with Mockito:**
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    void testRegisterUser() {
        // Given
        User user = new User("test@example.com", "John", "Doe");
        when(userRepository.save(any(User.class))).thenReturn(user);

        // When
        User registered = userService.registerUser(user);

        // Then
        assertNotNull(registered);
        verify(userRepository).save(user);
        verify(emailService).sendWelcomeEmail(user);
    }

    @Test
    void testGetUserById_NotFound() {
        // Given
        when(userRepository.findById(1L)).thenReturn(Optional.empty());

        // When & Then
        assertThrows(UserNotFoundException.class, () -> {
            userService.getUserById(1L);
        });
    }

    @Test
    void testUpdateUser() {
        // Given
        User existing = new User("old@example.com", "John", "Doe");
        existing.setId(1L);

        User updated = new User("new@example.com", "John", "Smith");

        when(userRepository.findById(1L)).thenReturn(Optional.of(existing));
        when(userRepository.save(any(User.class))).thenReturn(updated);

        // When
        User result = userService.updateUser(1L, updated);

        // Then
        assertEquals("new@example.com", result.getEmail());
        assertEquals("Smith", result.getLastName());
        verify(userRepository).save(any(User.class));
    }
}
```

### MockMvc (Controller Testing)

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void testGetUser() throws Exception {
        // Given
        User user = new User("test@example.com", "John", "Doe");
        user.setId(1L);
        when(userService.getUserById(1L)).thenReturn(Optional.of(user));

        // When & Then
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.email").value("test@example.com"))
            .andExpect(jsonPath("$.firstName").value("John"))
            .andExpect(jsonPath("$.lastName").value("Doe"));
    }

    @Test
    void testCreateUser() throws Exception {
        // Given
        User user = new User("test@example.com", "John", "Doe");
        User created = new User("test@example.com", "John", "Doe");
        created.setId(1L);

        when(userService.createUser(any(User.class))).thenReturn(created);

        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(user)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.email").value("test@example.com"));
    }

    @Test
    void testGetUser_NotFound() throws Exception {
        // Given
        when(userService.getUserById(999L)).thenReturn(Optional.empty());

        // When & Then
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }
}
```

### TestContainers (Integration Testing)

**Real database for testing:**
```java
@SpringBootTest
@Testcontainers
class UserRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void testSaveAndFindUser() {
        // Given
        User user = new User("test@example.com", "John", "Doe");

        // When
        User saved = userRepository.save(user);
        Optional<User> found = userRepository.findById(saved.getId());

        // Then
        assertTrue(found.isPresent());
        assertEquals("test@example.com", found.get().getEmail());
    }
}
```

**Multiple Containers:**
```java
@SpringBootTest
@Testcontainers
class FullStackIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);

    @Container
    static MongoDBContainer mongodb = new MongoDBContainer("mongo:6");

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        // PostgreSQL
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);

        // Redis
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);

        // MongoDB
        registry.add("spring.data.mongodb.uri", mongodb::getReplicaSetUrl);
    }

    @Test
    void testFullStack() {
        // Test with real databases
    }
}
```

## 7. Useful Third-Party Libraries

### Lombok (Reduce Boilerplate)

**Without Lombok:**
```java
public class User {
    private Long id;
    private String email;
    private String firstName;
    private String lastName;

    public User() {}

    public User(Long id, String email, String firstName, String lastName) {
        this.id = id;
        this.email = email;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    // ... more getters/setters

    @Override
    public boolean equals(Object o) { /* ... */ }

    @Override
    public int hashCode() { /* ... */ }

    @Override
    public String toString() { /* ... */ }
}
```

**With Lombok:**
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    private Long id;
    private String email;
    private String firstName;
    private String lastName;
}

// Usage
User user = User.builder()
    .email("test@example.com")
    .firstName("John")
    .lastName("Doe")
    .build();
```

**Common Lombok Annotations:**
```java
@Getter  // Generate getters
@Setter  // Generate setters
@ToString  // Generate toString()
@EqualsAndHashCode  // Generate equals() and hashCode()
@NoArgsConstructor  // Generate no-args constructor
@AllArgsConstructor  // Generate all-args constructor
@RequiredArgsConstructor  // Constructor for final fields
@Data  // Combines @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor
@Builder  // Builder pattern
@Slf4j  // Logger field

@Slf4j
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    public User createUser(User user) {
        log.info("Creating user: {}", user.getEmail());
        User created = userRepository.save(user);
        log.debug("User created with ID: {}", created.getId());
        return created;
    }
}
```

### MapStruct (Object Mapping)

**Manual Mapping:**
```java
public UserDTO toDTO(User user) {
    UserDTO dto = new UserDTO();
    dto.setId(user.getId());
    dto.setEmail(user.getEmail());
    dto.setFullName(user.getFirstName() + " " + user.getLastName());
    return dto;
}
```

**MapStruct:**
```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    @Mapping(target = "fullName", expression = "java(user.getFirstName() + \" \" + user.getLastName())")
    UserDTO toDTO(User user);

    @Mapping(target = "firstName", source = "dto.fullName", qualifiedByName = "extractFirstName")
    @Mapping(target = "lastName", source = "dto.fullName", qualifiedByName = "extractLastName")
    User toEntity(UserDTO dto);

    List<UserDTO> toDTOList(List<User> users);

    @Named("extractFirstName")
    default String extractFirstName(String fullName) {
        return fullName.split(" ")[0];
    }

    @Named("extractLastName")
    default String extractLastName(String fullName) {
        String[] parts = fullName.split(" ");
        return parts.length > 1 ? parts[1] : "";
    }
}

// Usage
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;

    public UserDTO getUser(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
        return userMapper.toDTO(user);
    }
}
```

### ModelMapper (Alternative to MapStruct)

```java
@Configuration
public class ModelMapperConfig {

    @Bean
    public ModelMapper modelMapper() {
        ModelMapper mapper = new ModelMapper();

        // Custom mapping
        mapper.typeMap(User.class, UserDTO.class).addMappings(map -> {
            map.map(src -> src.getFirstName() + " " + src.getLastName(),
                UserDTO::setFullName);
        });

        return mapper;
    }
}

@Service
@RequiredArgsConstructor
public class UserService {

    private final ModelMapper modelMapper;

    public UserDTO convertToDTO(User user) {
        return modelMapper.map(user, UserDTO.class);
    }

    public User convertToEntity(UserDTO dto) {
        return modelMapper.map(dto, User.class);
    }
}
```

### Jackson (JSON Processing)

```java
@Configuration
public class JacksonConfig {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();

        // Configuration
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

        // Register modules
        mapper.registerModule(new JavaTimeModule());

        return mapper;
    }
}
```

**Jackson Annotations:**
```java
@Data
public class User {

    @JsonProperty("user_id")
    private Long id;

    @JsonProperty("email_address")
    private String email;

    @JsonIgnore
    private String password;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createdAt;

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String middleName;

    @JsonAlias({"first_name", "firstname", "given_name"})
    private String firstName;
}
```

**Custom Serializer:**
```java
public class MoneySerializer extends JsonSerializer<BigDecimal> {

    @Override
    public void serialize(BigDecimal value, JsonGenerator gen,
                         SerializerProvider serializers) throws IOException {
        gen.writeString("$" + value.setScale(2, RoundingMode.HALF_UP).toString());
    }
}

public class Product {

    @JsonSerialize(using = MoneySerializer.class)
    private BigDecimal price;
}
```

### Apache Commons Libraries

```java
// Apache Commons Lang
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.RandomStringUtils;

@Service
public class UtilityService {

    public String sanitizeInput(String input) {
        if (StringUtils.isBlank(input)) {
            return "";
        }
        return StringUtils.trim(input);
    }

    public String generateToken() {
        return RandomStringUtils.randomAlphanumeric(32);
    }

    public boolean isValidEmail(String email) {
        return StringUtils.isNotBlank(email) &&
               email.contains("@") &&
               email.contains(".");
    }
}

// Apache Commons Collections
import org.apache.commons.collections4.CollectionUtils;

@Service
public class DataService {

    public <T> List<T> safeList(List<T> list) {
        return CollectionUtils.emptyIfNull(list);
    }

    public boolean hasCommonElements(List<String> list1, List<String> list2) {
        return CollectionUtils.containsAny(list1, list2);
    }
}
```

### Guava (Google Core Libraries)

```java
import com.google.common.collect.ImmutableList;
import com.google.common.collect.ImmutableMap;
import com.google.common.cache.CacheBuilder;
import com.google.common.cache.CacheLoader;
import com.google.common.cache.LoadingCache;

@Service
public class CacheService {

    private final LoadingCache<Long, User> userCache;

    public CacheService(UserRepository userRepository) {
        this.userCache = CacheBuilder.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build(new CacheLoader<Long, User>() {
                @Override
                public User load(Long id) {
                    return userRepository.findById(id)
                        .orElseThrow(() -> new UserNotFoundException(id));
                }
            });
    }

    public User getUser(Long id) {
        try {
            return userCache.get(id);
        } catch (Exception e) {
            throw new RuntimeException("Failed to load user", e);
        }
    }

    public ImmutableList<String> getRoles() {
        return ImmutableList.of("ADMIN", "USER", "GUEST");
    }

    public ImmutableMap<String, String> getConfig() {
        return ImmutableMap.of(
            "app.name", "MyApp",
            "app.version", "1.0.0"
        );
    }
}
```

## 8. Logging (SLF4J + Logback)

**Android Timber:**
```kotlin
Timber.d("User logged in: %s", username)
Timber.e(exception, "Failed to load data")
```

**Spring Logging:**
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class UserService {

    private static final Logger log = LoggerFactory.getLogger(UserService.class);

    public User createUser(User user) {
        log.info("Creating user with email: {}", user.getEmail());

        try {
            User created = userRepository.save(user);
            log.debug("User created successfully with ID: {}", created.getId());
            return created;
        } catch (Exception e) {
            log.error("Failed to create user: {}", user.getEmail(), e);
            throw e;
        }
    }
}

// With Lombok
@Slf4j
@Service
public class ProductService {

    public void processOrder(Order order) {
        log.info("Processing order: {}", order.getId());
        log.debug("Order details: {}", order);
        log.warn("Low stock for product: {}", order.getProductId());
        log.error("Payment failed for order: {}", order.getId());
    }
}
```

**Logback Configuration (logback-spring.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Error File Appender -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/error.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/error-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- JSON Appender for structured logging -->
    <appender name="JSON_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.json</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>

    <!-- Logger configurations -->
    <logger name="com.example.myapp" level="DEBUG"/>
    <logger name="org.springframework.web" level="INFO"/>
    <logger name="org.hibernate" level="INFO"/>
    <logger name="org.hibernate.SQL" level="DEBUG"/>
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

    <!-- Profile-specific configurations -->
    <springProfile name="dev">
        <logger name="com.example.myapp" level="DEBUG"/>
    </springProfile>

    <springProfile name="prod">
        <logger name="com.example.myapp" level="INFO"/>
        <appender-ref ref="JSON_FILE"/>
    </springProfile>

</configuration>
```

**Structured Logging:**
```java
import net.logstash.logback.argument.StructuredArguments;

@Slf4j
@Service
public class OrderService {

    public void processOrder(Order order) {
        log.info("Processing order",
            StructuredArguments.kv("orderId", order.getId()),
            StructuredArguments.kv("userId", order.getUserId()),
            StructuredArguments.kv("amount", order.getAmount()),
            StructuredArguments.kv("status", order.getStatus())
        );
    }
}
```

## 9. API Documentation (Swagger/OpenAPI)

**Dependencies (build.gradle):**
```gradle
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0'
```

**Configuration:**
```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("E-commerce API")
                .version("1.0")
                .description("REST API for E-commerce application")
                .contact(new Contact()
                    .name("API Support")
                    .email("support@example.com"))
                .license(new License()
                    .name("Apache 2.0")
                    .url("http://www.apache.org/licenses/LICENSE-2.0")))
            .externalDocs(new ExternalDocumentation()
                .description("Full Documentation")
                .url("https://docs.example.com"))
            .addSecurityItem(new SecurityRequirement().addList("bearer-jwt"))
            .components(new Components()
                .addSecuritySchemes("bearer-jwt", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")));
    }
}
```

**Documented Controller:**
```java
@RestController
@RequestMapping("/api/products")
@Tag(name = "Products", description = "Product management APIs")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @Operation(
        summary = "Get all products",
        description = "Returns a list of all products with pagination support"
    )
    @ApiResponses(value = {
        @ApiResponse(
            responseCode = "200",
            description = "Successfully retrieved products",
            content = @Content(
                mediaType = "application/json",
                schema = @Schema(implementation = ProductListResponse.class)
            )
        ),
        @ApiResponse(
            responseCode = "500",
            description = "Internal server error"
        )
    })
    @GetMapping
    public Page<Product> getAllProducts(
        @Parameter(description = "Page number (0-indexed)", example = "0")
        @RequestParam(defaultValue = "0") int page,

        @Parameter(description = "Page size", example = "20")
        @RequestParam(defaultValue = "20") int size
    ) {
        return productService.getAllProducts(page, size);
    }

    @Operation(
        summary = "Get product by ID",
        description = "Returns a single product"
    )
    @ApiResponse(
        responseCode = "200",
        description = "Product found"
    )
    @ApiResponse(
        responseCode = "404",
        description = "Product not found"
    )
    @GetMapping("/{id}")
    public Product getProduct(
        @Parameter(description = "Product ID", required = true)
        @PathVariable Long id
    ) {
        return productService.getProduct(id);
    }

    @Operation(
        summary = "Create a new product",
        description = "Creates a new product and returns the created product"
    )
    @ApiResponse(
        responseCode = "201",
        description = "Product created successfully"
    )
    @ApiResponse(
        responseCode = "400",
        description = "Invalid input"
    )
    @PostMapping
    public ResponseEntity<Product> createProduct(
        @io.swagger.v3.oas.annotations.parameters.RequestBody(
            description = "Product to create",
            required = true,
            content = @Content(
                schema = @Schema(implementation = ProductRequest.class)
            )
        )
        @RequestBody @Valid ProductRequest request
    ) {
        Product created = productService.createProduct(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @Operation(summary = "Delete a product")
    @ApiResponse(responseCode = "204", description = "Product deleted")
    @ApiResponse(responseCode = "404", description = "Product not found")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Documented Model:**
```java
@Schema(description = "Product entity")
@Data
public class Product {

    @Schema(description = "Product ID", example = "1", accessMode = Schema.AccessMode.READ_ONLY)
    private Long id;

    @Schema(description = "Product name", example = "Laptop", required = true)
    @NotBlank
    private String name;

    @Schema(description = "Product description", example = "High-performance laptop")
    private String description;

    @Schema(description = "Product price", example = "999.99", required = true)
    @NotNull
    @Positive
    private BigDecimal price;

    @Schema(description = "Stock quantity", example = "50")
    @Min(0)
    private int stock;

    @Schema(description = "Product category", example = "Electronics")
    private String category;

    @Schema(description = "Product tags", example = "[\"tech\", \"computer\"]")
    private List<String> tags;

    @Schema(description = "Creation timestamp", accessMode = Schema.AccessMode.READ_ONLY)
    private LocalDateTime createdAt;
}
```

**Access Swagger UI:**
```
http://localhost:8080/swagger-ui.html
http://localhost:8080/v3/api-docs
```

**Configuration (application.properties):**
```properties
# Swagger UI customization
springdoc.swagger-ui.path=/api-docs
springdoc.swagger-ui.operationsSorter=method
springdoc.swagger-ui.tagsSorter=alpha
springdoc.swagger-ui.tryItOutEnabled=true

# OpenAPI settings
springdoc.api-docs.path=/v3/api-docs
springdoc.packages-to-scan=com.example.myapp.controller
springdoc.paths-to-match=/api/**
```

## 10. Practical Example: Complete E-commerce Service

**Dependencies (build.gradle):**
```gradle
dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-cache'

    // Database
    runtimeOnly 'org.postgresql:postgresql'

    // Lombok
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // MapStruct
    implementation 'org.mapstruct:mapstruct:1.5.5.Final'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.5.Final'

    // OpenAPI
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0'

    // Testing
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.testcontainers:postgresql:1.19.3'
    testImplementation 'org.testcontainers:junit-jupiter:1.19.3'
}
```

**Domain Model:**
```java
@Entity
@Table(name = "products")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String sku;

    @Column(nullable = false)
    private String name;

    @Column(length = 1000)
    private String description;

    @Column(nullable = false)
    private BigDecimal price;

    private int stock;

    @Enumerated(EnumType.STRING)
    private ProductStatus status;

    @ElementCollection
    private List<String> images;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    public enum ProductStatus {
        ACTIVE, INACTIVE, OUT_OF_STOCK
    }
}
```

**Repository:**
```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    Optional<Product> findBySku(String sku);

    List<Product> findByStatus(Product.ProductStatus status);

    @Query("SELECT p FROM Product p WHERE p.price BETWEEN :minPrice AND :maxPrice")
    List<Product> findByPriceRange(
        @Param("minPrice") BigDecimal minPrice,
        @Param("maxPrice") BigDecimal maxPrice
    );

    Page<Product> findByNameContainingIgnoreCase(String name, Pageable pageable);
}
```

**DTO:**
```java
@Data
@Builder
@Schema(description = "Product response")
public class ProductDTO {

    @Schema(description = "Product ID")
    private Long id;

    @Schema(description = "Stock keeping unit")
    private String sku;

    @Schema(description = "Product name")
    private String name;

    @Schema(description = "Product description")
    private String description;

    @Schema(description = "Product price")
    private BigDecimal price;

    @Schema(description = "Available stock")
    private int stock;

    @Schema(description = "Product status")
    private String status;

    @Schema(description = "Product images")
    private List<String> images;
}

@Data
public class CreateProductRequest {

    @NotBlank(message = "SKU is required")
    private String sku;

    @NotBlank(message = "Name is required")
    @Size(min = 3, max = 100)
    private String name;

    @Size(max = 1000)
    private String description;

    @NotNull(message = "Price is required")
    @Positive
    private BigDecimal price;

    @Min(0)
    private int stock;
}
```

**Mapper:**
```java
@Mapper(componentModel = "spring")
public interface ProductMapper {

    ProductDTO toDTO(Product product);

    List<ProductDTO> toDTOList(List<Product> products);

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "status", constant = "ACTIVE")
    Product toEntity(CreateProductRequest request);
}
```

**Service:**
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;
    private final ProductMapper productMapper;

    @Cacheable(value = "products", key = "#id")
    public ProductDTO getProduct(Long id) {
        log.info("Fetching product with ID: {}", id);

        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found: " + id));

        return productMapper.toDTO(product);
    }

    public Page<ProductDTO> getAllProducts(int page, int size) {
        log.info("Fetching all products - page: {}, size: {}", page, size);

        Pageable pageable = PageRequest.of(page, size, Sort.by("name"));
        Page<Product> products = productRepository.findAll(pageable);

        return products.map(productMapper::toDTO);
    }

    @CachePut(value = "products", key = "#result.id")
    public ProductDTO createProduct(CreateProductRequest request) {
        log.info("Creating product with SKU: {}", request.getSku());

        // Check if SKU already exists
        productRepository.findBySku(request.getSku())
            .ifPresent(p -> {
                throw new DuplicateSkuException("Product with SKU already exists: " + request.getSku());
            });

        Product product = productMapper.toEntity(request);
        Product saved = productRepository.save(product);

        log.info("Product created successfully with ID: {}", saved.getId());
        return productMapper.toDTO(saved);
    }

    @CachePut(value = "products", key = "#id")
    public ProductDTO updateProduct(Long id, CreateProductRequest request) {
        log.info("Updating product with ID: {}", id);

        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found: " + id));

        product.setName(request.getName());
        product.setDescription(request.getDescription());
        product.setPrice(request.getPrice());
        product.setStock(request.getStock());

        Product updated = productRepository.save(product);

        log.info("Product updated successfully: {}", id);
        return productMapper.toDTO(updated);
    }

    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        log.info("Deleting product with ID: {}", id);

        if (!productRepository.existsById(id)) {
            throw new ProductNotFoundException("Product not found: " + id);
        }

        productRepository.deleteById(id);
        log.info("Product deleted successfully: {}", id);
    }

    public List<ProductDTO> searchByPriceRange(BigDecimal minPrice, BigDecimal maxPrice) {
        log.info("Searching products in price range: {} - {}", minPrice, maxPrice);

        List<Product> products = productRepository.findByPriceRange(minPrice, maxPrice);
        return productMapper.toDTOList(products);
    }
}
```

**Controller:**
```java
@Slf4j
@RestController
@RequestMapping("/api/products")
@Tag(name = "Products", description = "Product management APIs")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @Operation(summary = "Get all products")
    @GetMapping
    public ResponseEntity<Page<ProductDTO>> getAllProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size
    ) {
        return ResponseEntity.ok(productService.getAllProducts(page, size));
    }

    @Operation(summary = "Get product by ID")
    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> getProduct(@PathVariable Long id) {
        return ResponseEntity.ok(productService.getProduct(id));
    }

    @Operation(summary = "Create a new product")
    @PostMapping
    public ResponseEntity<ProductDTO> createProduct(
        @Valid @RequestBody CreateProductRequest request
    ) {
        ProductDTO created = productService.createProduct(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @Operation(summary = "Update a product")
    @PutMapping("/{id}")
    public ResponseEntity<ProductDTO> updateProduct(
        @PathVariable Long id,
        @Valid @RequestBody CreateProductRequest request
    ) {
        return ResponseEntity.ok(productService.updateProduct(id, request));
    }

    @Operation(summary = "Delete a product")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }

    @Operation(summary = "Search products by price range")
    @GetMapping("/search")
    public ResponseEntity<List<ProductDTO>> searchByPrice(
        @RequestParam BigDecimal minPrice,
        @RequestParam BigDecimal maxPrice
    ) {
        return ResponseEntity.ok(productService.searchByPriceRange(minPrice, maxPrice));
    }
}
```

**Exception Handling:**
```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleProductNotFound(ProductNotFoundException ex) {
        log.error("Product not found: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.NOT_FOUND.value())
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(DuplicateSkuException.class)
    public ResponseEntity<ErrorResponse> handleDuplicateSku(DuplicateSkuException ex) {
        log.error("Duplicate SKU: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.CONFLICT.value())
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .build();

        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );

        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.BAD_REQUEST.value())
            .message("Validation failed")
            .errors(errors)
            .timestamp(LocalDateTime.now())
            .build();

        return ResponseEntity.badRequest().body(error);
    }
}

@Data
@Builder
public class ErrorResponse {
    private int status;
    private String message;
    private Map<String, String> errors;
    private LocalDateTime timestamp;
}
```

**Integration Test:**
```java
@SpringBootTest
@Testcontainers
@AutoConfigureMockMvc
class ProductControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        productRepository.deleteAll();
    }

    @Test
    void testCreateProduct() throws Exception {
        CreateProductRequest request = CreateProductRequest.builder()
            .sku("LAP-001")
            .name("Gaming Laptop")
            .description("High-performance gaming laptop")
            .price(new BigDecimal("1299.99"))
            .stock(10)
            .build();

        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.sku").value("LAP-001"))
            .andExpect(jsonPath("$.name").value("Gaming Laptop"))
            .andExpect(jsonPath("$.price").value(1299.99));
    }

    @Test
    void testGetProduct() throws Exception {
        Product product = Product.builder()
            .sku("LAP-002")
            .name("Business Laptop")
            .price(new BigDecimal("899.99"))
            .stock(5)
            .status(Product.ProductStatus.ACTIVE)
            .build();

        Product saved = productRepository.save(product);

        mockMvc.perform(get("/api/products/" + saved.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(saved.getId()))
            .andExpect(jsonPath("$.sku").value("LAP-002"));
    }

    @Test
    void testGetProduct_NotFound() throws Exception {
        mockMvc.perform(get("/api/products/999"))
            .andExpect(status().isNotFound());
    }
}
```

## 📊 Library Comparison Summary

| Category | Android | Spring | When to Use |
|----------|---------|--------|-------------|
| **ORM** | Room | Spring Data JPA | Always for SQL databases |
| **NoSQL** | - | Spring Data MongoDB | Document storage, flexible schema |
| **Caching** | SharedPreferences | Spring Data Redis | Session data, caching |
| **Search** | - | Elasticsearch | Full-text search, analytics |
| **HTTP Client** | Retrofit | RestTemplate/WebClient | External API calls |
| **Reactive** | RxJava/Flow | Spring WebFlux | High concurrency, streaming |
| **Background Jobs** | WorkManager | Spring Batch | Bulk processing, scheduled tasks |
| **Microservices** | - | Spring Cloud | Distributed systems |
| **Testing** | JUnit + Mockito | JUnit + Mockito + TestContainers | All testing scenarios |
| **Logging** | Timber | SLF4J + Logback | Production logging |
| **JSON** | Moshi/Gson | Jackson | API serialization |
| **Mapping** | Manual | MapStruct/ModelMapper | DTO conversion |
| **Validation** | Manual | Bean Validation | Input validation |
| **API Docs** | - | Swagger/OpenAPI | API documentation |

## 🎯 Key Takeaways

1. **Spring Data** provides powerful abstractions for various data stores
2. **Spring Cloud** enables building microservices architectures
3. **Spring Batch** handles complex batch processing jobs
4. **Spring WebFlux** brings reactive programming to Spring
5. **TestContainers** enables real integration testing
6. **Lombok** dramatically reduces boilerplate code
7. **MapStruct** provides type-safe bean mapping
8. **SLF4J + Logback** offer production-grade logging
9. **OpenAPI** automatically generates API documentation
10. Choose libraries based on your specific needs

## 🚀 Next Steps

Congratulations! You now have a comprehensive understanding of the Spring ecosystem. Practice by:

1. Building a microservices application with Spring Cloud
2. Implementing reactive endpoints with WebFlux
3. Creating batch jobs for data processing
4. Setting up comprehensive testing with TestContainers
5. Documenting your APIs with OpenAPI

---

**Ready to dive deeper?** Check out the [Spring Documentation](https://spring.io/projects/spring-boot) for more advanced topics!
