# Best Practices & Architecture Guide for Spring Boot

A comprehensive guide for Kotlin Android developers transitioning to Spring Boot backend development.

## 🎯 Overview

This guide covers enterprise-grade best practices, architectural patterns, and common pitfalls when building Spring Boot applications. As an Android developer, you'll find familiar patterns adapted to backend development.

## 📋 Table of Contents

1. [Clean Architecture in Spring Boot](#1-clean-architecture-in-spring-boot)
2. [SOLID Principles](#2-solid-principles-application)
3. [Design Patterns](#3-design-patterns)
4. [Project Structure](#4-project-structure-best-practices)
5. [Error Handling](#5-error-handling-strategies)
6. [Validation Patterns](#6-validation-patterns)
7. [Testing Strategies](#7-testing-strategies)
8. [API Design](#8-api-design-best-practices)
9. [Security Best Practices](#9-security-best-practices)
10. [Performance Optimization](#10-performance-optimization)
11. [Code Organization](#11-code-organization-and-naming-conventions)
12. [Common Pitfalls](#12-common-pitfalls-and-anti-patterns)
13. [Migration Path](#13-migration-path-from-android-to-spring)

---

## 1. Clean Architecture in Spring Boot

### 🏗️ Architecture Overview

Clean Architecture (similar to Android's MVVM/MVI patterns) separates concerns into distinct layers.

**Android Architecture (Familiar to You):**
```
Presentation Layer (UI/ViewModel)
    ↓
Domain Layer (Use Cases/Repository Interfaces)
    ↓
Data Layer (Repository Implementations/Data Sources)
```

**Spring Boot Clean Architecture:**
```
Presentation Layer (Controllers/DTOs)
    ↓
Application Layer (Services/Use Cases)
    ↓
Domain Layer (Entities/Business Logic)
    ↓
Infrastructure Layer (Repositories/External Services)
```

### Layer Responsibilities

#### 1. Presentation Layer (Controllers)

**❌ Bad - Fat Controller:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;

    @PostMapping
    public ResponseEntity<UserDTO> createUser(@RequestBody CreateUserRequest request) {
        // Business logic in controller - BAD!
        if (request.getEmail() == null || !request.getEmail().contains("@")) {
            return ResponseEntity.badRequest().build();
        }

        // Direct repository access - BAD!
        User user = new User();
        user.setEmail(request.getEmail());
        user.setName(request.getName());
        user.setCreatedAt(LocalDateTime.now());

        User saved = userRepository.save(user);

        // Side effects in controller - BAD!
        emailService.sendWelcomeEmail(saved.getEmail());

        return ResponseEntity.ok(new UserDTO(saved));
    }
}
```

**✅ Good - Thin Controller:**
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final CreateUserUseCase createUserUseCase;
    private final UserMapper userMapper;

    @PostMapping
    public ResponseEntity<UserResponse> createUser(
        @Valid @RequestBody CreateUserRequest request
    ) {
        CreateUserCommand command = userMapper.toCommand(request);
        User user = createUserUseCase.execute(command);
        UserResponse response = userMapper.toResponse(user);

        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(response);
    }
}
```

#### 2. Application Layer (Use Cases/Services)

**✅ Use Case Pattern:**
```java
@Service
@RequiredArgsConstructor
@Transactional
public class CreateUserUseCase {
    private final UserRepository userRepository;
    private final EmailService emailService;
    private final UserValidator userValidator;

    public User execute(CreateUserCommand command) {
        // 1. Validate
        userValidator.validateCreateUser(command);

        // 2. Check business rules
        if (userRepository.existsByEmail(command.getEmail())) {
            throw new UserAlreadyExistsException(command.getEmail());
        }

        // 3. Create domain entity
        User user = User.builder()
            .email(command.getEmail())
            .name(command.getName())
            .status(UserStatus.PENDING)
            .build();

        // 4. Persist
        User savedUser = userRepository.save(user);

        // 5. Side effects
        emailService.sendWelcomeEmailAsync(savedUser);

        return savedUser;
    }
}
```

**Similar to Android Use Case:**
```kotlin
// Android
class CreateUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
    private val emailService: EmailService
) {
    suspend operator fun invoke(command: CreateUserCommand): User {
        val user = User(
            email = command.email,
            name = command.name,
            status = UserStatus.PENDING
        )

        val savedUser = userRepository.save(user)
        emailService.sendWelcomeEmail(savedUser)

        return savedUser
    }
}
```

#### 3. Domain Layer (Entities & Business Logic)

**✅ Rich Domain Model:**
```java
@Entity
@Table(name = "users")
@NoArgsConstructor
@Getter
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    private String name;

    @Enumerated(EnumType.STRING)
    private UserStatus status;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @Column(name = "last_login_at")
    private LocalDateTime lastLoginAt;

    // Builder pattern
    @Builder
    public User(String email, String name, UserStatus status) {
        this.email = email;
        this.name = name;
        this.status = status;
        this.createdAt = LocalDateTime.now();
    }

    // Business logic methods
    public void activate() {
        if (this.status != UserStatus.PENDING) {
            throw new IllegalStateException(
                "Only pending users can be activated"
            );
        }
        this.status = UserStatus.ACTIVE;
    }

    public void recordLogin() {
        if (this.status != UserStatus.ACTIVE) {
            throw new IllegalStateException(
                "Only active users can login"
            );
        }
        this.lastLoginAt = LocalDateTime.now();
    }

    public boolean isActive() {
        return this.status == UserStatus.ACTIVE;
    }

    public boolean canLogin() {
        return this.status == UserStatus.ACTIVE;
    }
}
```

#### 4. Infrastructure Layer

**✅ Repository Interface (Domain Layer):**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    List<User> findByStatus(UserStatus status);
}
```

**✅ Custom Repository Implementation:**
```java
@Repository
@RequiredArgsConstructor
public class UserRepositoryImpl {
    private final EntityManager entityManager;

    public List<User> findActiveUsersWithRecentActivity(int days) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);

        query.select(user)
            .where(
                cb.and(
                    cb.equal(user.get("status"), UserStatus.ACTIVE),
                    cb.greaterThan(
                        user.get("lastLoginAt"),
                        LocalDateTime.now().minusDays(days)
                    )
                )
            );

        return entityManager.createQuery(query).getResultList();
    }
}
```

### Complete Clean Architecture Example

**Project Structure:**
```
src/main/java/com/example/app/
├── domain/
│   ├── model/
│   │   ├── User.java
│   │   └── UserStatus.java
│   ├── repository/
│   │   └── UserRepository.java
│   └── exception/
│       └── UserAlreadyExistsException.java
├── application/
│   ├── usecase/
│   │   ├── CreateUserUseCase.java
│   │   ├── GetUserUseCase.java
│   │   └── UpdateUserUseCase.java
│   ├── service/
│   │   ├── EmailService.java
│   │   └── UserValidator.java
│   └── dto/
│       ├── CreateUserCommand.java
│       └── UpdateUserCommand.java
├── infrastructure/
│   ├── persistence/
│   │   └── UserRepositoryImpl.java
│   ├── email/
│   │   └── SmtpEmailService.java
│   └── config/
│       └── DatabaseConfig.java
└── presentation/
    ├── controller/
    │   └── UserController.java
    ├── dto/
    │   ├── CreateUserRequest.java
    │   ├── UpdateUserRequest.java
    │   └── UserResponse.java
    ├── mapper/
    │   └── UserMapper.java
    └── exception/
        └── GlobalExceptionHandler.java
```

---

## 2. SOLID Principles Application

### Single Responsibility Principle (SRP)

**❌ Bad - Multiple Responsibilities:**
```java
@Service
public class UserService {
    // Handles CRUD, validation, email, logging, metrics - TOO MUCH!

    public User createUser(CreateUserRequest request) {
        // Validation logic
        if (request.getEmail() == null) {
            throw new ValidationException("Email required");
        }

        // Business logic
        User user = new User();
        user.setEmail(request.getEmail());

        // Persistence
        user = userRepository.save(user);

        // Email logic
        sendEmail(user.getEmail(), "Welcome!");

        // Logging
        logger.info("User created: {}", user.getId());

        // Metrics
        metricsService.incrementUserCount();

        return user;
    }

    private void sendEmail(String to, String subject) {
        // Email sending logic
    }
}
```

**✅ Good - Single Responsibility:**
```java
// 1. User Creation Use Case
@Service
@RequiredArgsConstructor
public class CreateUserUseCase {
    private final UserRepository userRepository;
    private final UserValidator validator;
    private final UserEventPublisher eventPublisher;

    @Transactional
    public User execute(CreateUserCommand command) {
        validator.validate(command);

        User user = User.builder()
            .email(command.getEmail())
            .name(command.getName())
            .build();

        User savedUser = userRepository.save(user);

        eventPublisher.publishUserCreated(savedUser);

        return savedUser;
    }
}

// 2. Validation Service
@Component
public class UserValidator {
    public void validate(CreateUserCommand command) {
        if (command.getEmail() == null || command.getEmail().isBlank()) {
            throw new ValidationException("Email is required");
        }

        if (!command.getEmail().matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
            throw new ValidationException("Invalid email format");
        }
    }
}

// 3. Event Publisher
@Component
@RequiredArgsConstructor
public class UserEventPublisher {
    private final ApplicationEventPublisher eventPublisher;

    public void publishUserCreated(User user) {
        UserCreatedEvent event = new UserCreatedEvent(user);
        eventPublisher.publishEvent(event);
    }
}

// 4. Email Event Listener
@Component
@RequiredArgsConstructor
public class UserEmailEventListener {
    private final EmailService emailService;

    @EventListener
    @Async
    public void handleUserCreated(UserCreatedEvent event) {
        emailService.sendWelcomeEmail(event.getUser());
    }
}
```

### Open/Closed Principle (OCP)

**❌ Bad - Not Open for Extension:**
```java
public class PriceCalculator {
    public double calculatePrice(String customerType, double basePrice) {
        if (customerType.equals("REGULAR")) {
            return basePrice;
        } else if (customerType.equals("PREMIUM")) {
            return basePrice * 0.9; // 10% discount
        } else if (customerType.equals("VIP")) {
            return basePrice * 0.8; // 20% discount
        }
        return basePrice;
    }
}
```

**✅ Good - Open for Extension, Closed for Modification:**
```java
// Strategy Pattern
public interface PricingStrategy {
    double calculatePrice(double basePrice);
}

@Component("regularPricing")
public class RegularPricingStrategy implements PricingStrategy {
    @Override
    public double calculatePrice(double basePrice) {
        return basePrice;
    }
}

@Component("premiumPricing")
public class PremiumPricingStrategy implements PricingStrategy {
    @Override
    public double calculatePrice(double basePrice) {
        return basePrice * 0.9;
    }
}

@Component("vipPricing")
public class VIPPricingStrategy implements PricingStrategy {
    @Override
    public double calculatePrice(double basePrice) {
        return basePrice * 0.8;
    }
}

@Service
@RequiredArgsConstructor
public class PriceCalculator {
    private final Map<String, PricingStrategy> strategies;

    public double calculatePrice(String customerType, double basePrice) {
        PricingStrategy strategy = strategies.get(customerType + "Pricing");

        if (strategy == null) {
            throw new IllegalArgumentException("Unknown customer type: " + customerType);
        }

        return strategy.calculatePrice(basePrice);
    }
}
```

### Liskov Substitution Principle (LSP)

**❌ Bad - Violates LSP:**
```java
public class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // Violates LSP!
    }

    @Override
    public void setHeight(int height) {
        this.width = height;  // Violates LSP!
        this.height = height;
    }
}

// This breaks!
Rectangle rect = new Square();
rect.setWidth(5);
rect.setHeight(10);
// Expected: 50, Actual: 100
```

**✅ Good - Respects LSP:**
```java
public interface Shape {
    int getArea();
}

@Value
public class Rectangle implements Shape {
    int width;
    int height;

    @Override
    public int getArea() {
        return width * height;
    }
}

@Value
public class Square implements Shape {
    int side;

    @Override
    public int getArea() {
        return side * side;
    }
}
```

### Interface Segregation Principle (ISP)

**❌ Bad - Fat Interface:**
```java
public interface Worker {
    void work();
    void eat();
    void sleep();
    void getPaid();
    void attendMeeting();
    void submitTimesheet();
}

// Robot worker doesn't eat or sleep!
public class RobotWorker implements Worker {
    @Override
    public void work() { /* work */ }

    @Override
    public void eat() { /* Not applicable */ }

    @Override
    public void sleep() { /* Not applicable */ }

    @Override
    public void getPaid() { /* Not applicable */ }

    @Override
    public void attendMeeting() { /* work */ }

    @Override
    public void submitTimesheet() { /* Not applicable */ }
}
```

**✅ Good - Segregated Interfaces:**
```java
public interface Workable {
    void work();
}

public interface Attendable {
    void attendMeeting();
}

public interface Biological {
    void eat();
    void sleep();
}

public interface Payable {
    void getPaid();
    void submitTimesheet();
}

public class HumanWorker implements Workable, Attendable, Biological, Payable {
    @Override
    public void work() { /* work */ }

    @Override
    public void attendMeeting() { /* attend */ }

    @Override
    public void eat() { /* eat */ }

    @Override
    public void sleep() { /* sleep */ }

    @Override
    public void getPaid() { /* get paid */ }

    @Override
    public void submitTimesheet() { /* submit */ }
}

public class RobotWorker implements Workable, Attendable {
    @Override
    public void work() { /* work */ }

    @Override
    public void attendMeeting() { /* attend */ }
}
```

### Dependency Inversion Principle (DIP)

**❌ Bad - Depends on Concretions:**
```java
@Service
public class OrderService {
    private final MySQLOrderRepository repository; // Concrete class!
    private final SmtpEmailService emailService;   // Concrete class!

    public OrderService() {
        this.repository = new MySQLOrderRepository();
        this.emailService = new SmtpEmailService();
    }

    public void createOrder(Order order) {
        repository.save(order);
        emailService.sendOrderConfirmation(order);
    }
}
```

**✅ Good - Depends on Abstractions:**
```java
// Abstractions
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(Long id);
}

public interface NotificationService {
    void sendOrderConfirmation(Order order);
}

// Service depends on abstractions
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository repository;
    private final NotificationService notificationService;

    public void createOrder(Order order) {
        Order savedOrder = repository.save(order);
        notificationService.sendOrderConfirmation(savedOrder);
    }
}

// Implementations
@Repository
public class JpaOrderRepository implements OrderRepository {
    // JPA implementation
}

@Service
public class EmailNotificationService implements NotificationService {
    // Email implementation
}

@Service
@Primary
public class MultiChannelNotificationService implements NotificationService {
    // Email + SMS + Push implementation
}
```

---

## 3. Design Patterns

### Factory Pattern

**Use Case:** Creating complex objects with different configurations.

**✅ Implementation:**
```java
// Product
public interface PaymentProcessor {
    PaymentResult process(Payment payment);
}

public class CreditCardProcessor implements PaymentProcessor {
    @Override
    public PaymentResult process(Payment payment) {
        // Credit card processing logic
        return new PaymentResult(true, "Credit card processed");
    }
}

public class PayPalProcessor implements PaymentProcessor {
    @Override
    public PaymentResult process(Payment payment) {
        // PayPal processing logic
        return new PaymentResult(true, "PayPal processed");
    }
}

public class CryptoProcessor implements PaymentProcessor {
    @Override
    public PaymentResult process(Payment payment) {
        // Cryptocurrency processing logic
        return new PaymentResult(true, "Crypto processed");
    }
}

// Factory
@Component
public class PaymentProcessorFactory {
    private final Map<PaymentMethod, PaymentProcessor> processors;

    public PaymentProcessorFactory(
        CreditCardProcessor creditCardProcessor,
        PayPalProcessor payPalProcessor,
        CryptoProcessor cryptoProcessor
    ) {
        this.processors = Map.of(
            PaymentMethod.CREDIT_CARD, creditCardProcessor,
            PaymentMethod.PAYPAL, payPalProcessor,
            PaymentMethod.CRYPTO, cryptoProcessor
        );
    }

    public PaymentProcessor getProcessor(PaymentMethod method) {
        PaymentProcessor processor = processors.get(method);
        if (processor == null) {
            throw new UnsupportedPaymentMethodException(method);
        }
        return processor;
    }
}

// Usage
@Service
@RequiredArgsConstructor
public class PaymentService {
    private final PaymentProcessorFactory processorFactory;

    public PaymentResult processPayment(Payment payment) {
        PaymentProcessor processor = processorFactory.getProcessor(
            payment.getMethod()
        );
        return processor.process(payment);
    }
}
```

### Builder Pattern

**Use Case:** Creating objects with many optional parameters.

**✅ Implementation:**
```java
@Getter
public class EmailMessage {
    private final String to;
    private final String subject;
    private final String body;
    private final List<String> cc;
    private final List<String> bcc;
    private final List<Attachment> attachments;
    private final EmailPriority priority;
    private final boolean htmlFormat;

    private EmailMessage(Builder builder) {
        this.to = builder.to;
        this.subject = builder.subject;
        this.body = builder.body;
        this.cc = builder.cc;
        this.bcc = builder.bcc;
        this.attachments = builder.attachments;
        this.priority = builder.priority;
        this.htmlFormat = builder.htmlFormat;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private String to;
        private String subject;
        private String body;
        private List<String> cc = new ArrayList<>();
        private List<String> bcc = new ArrayList<>();
        private List<Attachment> attachments = new ArrayList<>();
        private EmailPriority priority = EmailPriority.NORMAL;
        private boolean htmlFormat = false;

        public Builder to(String to) {
            this.to = to;
            return this;
        }

        public Builder subject(String subject) {
            this.subject = subject;
            return this;
        }

        public Builder body(String body) {
            this.body = body;
            return this;
        }

        public Builder cc(String... cc) {
            this.cc.addAll(Arrays.asList(cc));
            return this;
        }

        public Builder bcc(String... bcc) {
            this.bcc.addAll(Arrays.asList(bcc));
            return this;
        }

        public Builder addAttachment(Attachment attachment) {
            this.attachments.add(attachment);
            return this;
        }

        public Builder priority(EmailPriority priority) {
            this.priority = priority;
            return this;
        }

        public Builder htmlFormat(boolean htmlFormat) {
            this.htmlFormat = htmlFormat;
            return this;
        }

        public EmailMessage build() {
            if (to == null || to.isBlank()) {
                throw new IllegalStateException("Recipient is required");
            }
            if (subject == null) {
                throw new IllegalStateException("Subject is required");
            }
            return new EmailMessage(this);
        }
    }
}

// Usage
EmailMessage email = EmailMessage.builder()
    .to("user@example.com")
    .subject("Welcome!")
    .body("<h1>Welcome to our service</h1>")
    .cc("manager@example.com", "team@example.com")
    .priority(EmailPriority.HIGH)
    .htmlFormat(true)
    .addAttachment(welcomeGuide)
    .build();
```

### Strategy Pattern

**Use Case:** Different algorithms for the same operation.

**✅ Implementation:**
```java
// Strategy Interface
public interface DiscountStrategy {
    double applyDiscount(double price);
    boolean isApplicable(Customer customer);
}

// Concrete Strategies
@Component
public class NoDiscountStrategy implements DiscountStrategy {
    @Override
    public double applyDiscount(double price) {
        return price;
    }

    @Override
    public boolean isApplicable(Customer customer) {
        return customer.getMembershipLevel() == MembershipLevel.NONE;
    }
}

@Component
public class SeasonalDiscountStrategy implements DiscountStrategy {
    private static final double DISCOUNT_RATE = 0.15;

    @Override
    public double applyDiscount(double price) {
        return price * (1 - DISCOUNT_RATE);
    }

    @Override
    public boolean isApplicable(Customer customer) {
        LocalDate now = LocalDate.now();
        return now.getMonthValue() == 12; // December
    }
}

@Component
public class LoyaltyDiscountStrategy implements DiscountStrategy {
    private static final Map<MembershipLevel, Double> DISCOUNTS = Map.of(
        MembershipLevel.SILVER, 0.05,
        MembershipLevel.GOLD, 0.10,
        MembershipLevel.PLATINUM, 0.20
    );

    @Override
    public double applyDiscount(double price) {
        // Implemented in context
        return price;
    }

    public double applyDiscount(double price, MembershipLevel level) {
        double discount = DISCOUNTS.getOrDefault(level, 0.0);
        return price * (1 - discount);
    }

    @Override
    public boolean isApplicable(Customer customer) {
        return customer.getMembershipLevel() != MembershipLevel.NONE;
    }
}

// Context
@Service
@RequiredArgsConstructor
public class PricingService {
    private final List<DiscountStrategy> strategies;

    public double calculateFinalPrice(Customer customer, double basePrice) {
        return strategies.stream()
            .filter(strategy -> strategy.isApplicable(customer))
            .findFirst()
            .map(strategy -> strategy.applyDiscount(basePrice))
            .orElse(basePrice);
    }
}
```

### Observer Pattern (Event-Driven)

**Use Case:** Decoupling components through events.

**✅ Implementation:**
```java
// Event
@Getter
public class OrderPlacedEvent {
    private final Order order;
    private final LocalDateTime timestamp;

    public OrderPlacedEvent(Order order) {
        this.order = order;
        this.timestamp = LocalDateTime.now();
    }
}

// Publisher
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public Order placeOrder(PlaceOrderCommand command) {
        Order order = Order.builder()
            .customerId(command.getCustomerId())
            .items(command.getItems())
            .status(OrderStatus.PENDING)
            .build();

        Order savedOrder = orderRepository.save(order);

        // Publish event
        eventPublisher.publishEvent(new OrderPlacedEvent(savedOrder));

        return savedOrder;
    }
}

// Listeners
@Component
@RequiredArgsConstructor
@Slf4j
public class OrderNotificationListener {
    private final EmailService emailService;

    @EventListener
    @Async
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Sending order confirmation for order: {}",
            event.getOrder().getId());
        emailService.sendOrderConfirmation(event.getOrder());
    }
}

@Component
@RequiredArgsConstructor
@Slf4j
public class InventoryListener {
    private final InventoryService inventoryService;

    @EventListener
    @Async
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Updating inventory for order: {}",
            event.getOrder().getId());
        inventoryService.reserveItems(event.getOrder().getItems());
    }
}

@Component
@RequiredArgsConstructor
@Slf4j
public class AnalyticsListener {
    private final AnalyticsService analyticsService;

    @EventListener
    @Async
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Recording analytics for order: {}",
            event.getOrder().getId());
        analyticsService.recordOrderPlaced(event.getOrder());
    }
}
```

### Repository Pattern

**Already built into Spring Data JPA, but here's how to extend it:**

```java
// Base Repository Interface
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByStatus(UserStatus status);
}

// Custom Repository Interface
public interface CustomUserRepository {
    List<User> findUsersWithComplexCriteria(UserSearchCriteria criteria);
    Page<User> searchUsers(String query, Pageable pageable);
}

// Custom Repository Implementation
@RequiredArgsConstructor
public class CustomUserRepositoryImpl implements CustomUserRepository {
    private final EntityManager entityManager;

    @Override
    public List<User> findUsersWithComplexCriteria(UserSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);

        List<Predicate> predicates = new ArrayList<>();

        if (criteria.getEmail() != null) {
            predicates.add(cb.like(
                cb.lower(user.get("email")),
                "%" + criteria.getEmail().toLowerCase() + "%"
            ));
        }

        if (criteria.getStatus() != null) {
            predicates.add(cb.equal(user.get("status"), criteria.getStatus()));
        }

        if (criteria.getCreatedAfter() != null) {
            predicates.add(cb.greaterThan(
                user.get("createdAt"),
                criteria.getCreatedAfter()
            ));
        }

        query.where(predicates.toArray(new Predicate[0]));

        return entityManager.createQuery(query).getResultList();
    }

    @Override
    public Page<User> searchUsers(String query, Pageable pageable) {
        // Full-text search implementation
        return null; // Implementation details
    }
}

// Combined Interface
public interface UserRepository
    extends JpaRepository<User, Long>, CustomUserRepository {
    // Spring Data JPA methods + custom methods
}
```

### Decorator Pattern

**Use Case:** Adding behavior to objects dynamically.

**✅ Implementation:**
```java
// Component
public interface NotificationService {
    void send(String message, String recipient);
}

// Base Implementation
@Component
public class SimpleNotificationService implements NotificationService {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending: " + message + " to " + recipient);
    }
}

// Decorators
@Component
@Primary
@RequiredArgsConstructor
public class LoggingNotificationDecorator implements NotificationService {
    private final NotificationService delegate;
    private final Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public void send(String message, String recipient) {
        logger.info("Sending notification to {}", recipient);
        delegate.send(message, recipient);
        logger.info("Notification sent successfully");
    }
}

@Component
@RequiredArgsConstructor
public class RetryNotificationDecorator implements NotificationService {
    private final NotificationService delegate;
    private static final int MAX_RETRIES = 3;

    @Override
    public void send(String message, String recipient) {
        int attempt = 0;
        while (attempt < MAX_RETRIES) {
            try {
                delegate.send(message, recipient);
                return;
            } catch (Exception e) {
                attempt++;
                if (attempt >= MAX_RETRIES) {
                    throw e;
                }
            }
        }
    }
}
```

---

## 4. Project Structure Best Practices

### Package-by-Feature vs Package-by-Layer

**❌ Package-by-Layer (Old Approach):**
```
com.example.app/
├── controller/
│   ├── UserController.java
│   ├── OrderController.java
│   └── ProductController.java
├── service/
│   ├── UserService.java
│   ├── OrderService.java
│   └── ProductService.java
├── repository/
│   ├── UserRepository.java
│   ├── OrderRepository.java
│   └── ProductRepository.java
└── model/
    ├── User.java
    ├── Order.java
    └── Product.java
```

**✅ Package-by-Feature (Recommended):**
```
com.example.app/
├── user/
│   ├── User.java
│   ├── UserController.java
│   ├── UserService.java
│   ├── UserRepository.java
│   ├── dto/
│   │   ├── CreateUserRequest.java
│   │   └── UserResponse.java
│   └── exception/
│       └── UserNotFoundException.java
├── order/
│   ├── Order.java
│   ├── OrderController.java
│   ├── OrderService.java
│   ├── OrderRepository.java
│   └── dto/
├── product/
│   ├── Product.java
│   ├── ProductController.java
│   ├── ProductService.java
│   └── ProductRepository.java
└── common/
    ├── config/
    ├── security/
    ├── exception/
    └── util/
```

### Complete Project Structure

**✅ Enterprise Structure:**
```
my-spring-app/
├── src/
│   ├── main/
│   │   ├── java/com/example/app/
│   │   │   ├── config/
│   │   │   │   ├── SecurityConfig.java
│   │   │   │   ├── DatabaseConfig.java
│   │   │   │   ├── CacheConfig.java
│   │   │   │   └── AsyncConfig.java
│   │   │   ├── common/
│   │   │   │   ├── exception/
│   │   │   │   │   ├── BusinessException.java
│   │   │   │   │   ├── ResourceNotFoundException.java
│   │   │   │   │   └── GlobalExceptionHandler.java
│   │   │   │   ├── dto/
│   │   │   │   │   ├── ApiResponse.java
│   │   │   │   │   ├── PageResponse.java
│   │   │   │   │   └── ErrorResponse.java
│   │   │   │   └── util/
│   │   │   │       ├── DateUtils.java
│   │   │   │       └── ValidationUtils.java
│   │   │   ├── user/
│   │   │   │   ├── domain/
│   │   │   │   │   ├── User.java
│   │   │   │   │   └── UserStatus.java
│   │   │   │   ├── application/
│   │   │   │   │   ├── CreateUserUseCase.java
│   │   │   │   │   ├── GetUserUseCase.java
│   │   │   │   │   └── UpdateUserUseCase.java
│   │   │   │   ├── infrastructure/
│   │   │   │   │   ├── UserRepository.java
│   │   │   │   │   └── UserCacheRepository.java
│   │   │   │   ├── presentation/
│   │   │   │   │   ├── UserController.java
│   │   │   │   │   ├── dto/
│   │   │   │   │   │   ├── CreateUserRequest.java
│   │   │   │   │   │   ├── UpdateUserRequest.java
│   │   │   │   │   │   └── UserResponse.java
│   │   │   │   │   └── mapper/
│   │   │   │   │       └── UserMapper.java
│   │   │   │   └── exception/
│   │   │   │       ├── UserNotFoundException.java
│   │   │   │       └── UserAlreadyExistsException.java
│   │   │   ├── order/
│   │   │   │   └── [similar structure]
│   │   │   └── Application.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       ├── db/migration/
│   │       │   ├── V1__Create_users_table.sql
│   │       │   └── V2__Create_orders_table.sql
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       ├── java/com/example/app/
│       │   ├── user/
│       │   │   ├── UserControllerTest.java
│       │   │   ├── CreateUserUseCaseTest.java
│       │   │   └── UserRepositoryTest.java
│       │   └── integration/
│       │       └── UserIntegrationTest.java
│       └── resources/
│           └── application-test.yml
├── .gitignore
├── pom.xml (or build.gradle)
└── README.md
```

---

## 5. Error Handling Strategies

### Global Exception Handler

**✅ Centralized Error Handling:**
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
        ResourceNotFoundException ex,
        WebRequest request
    ) {
        log.error("Resource not found: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.NOT_FOUND.value())
            .error("Not Found")
            .message(ex.getMessage())
            .path(request.getDescription(false))
            .build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
        ValidationException ex,
        WebRequest request
    ) {
        log.warn("Validation error: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.BAD_REQUEST.value())
            .error("Validation Failed")
            .message(ex.getMessage())
            .path(request.getDescription(false))
            .build();

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(
        MethodArgumentNotValidException ex,
        WebRequest request
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.BAD_REQUEST.value())
            .error("Validation Failed")
            .message("Input validation failed")
            .path(request.getDescription(false))
            .validationErrors(errors)
            .build();

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusiness(
        BusinessException ex,
        WebRequest request
    ) {
        log.warn("Business rule violation: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.UNPROCESSABLE_ENTITY.value())
            .error("Business Rule Violation")
            .message(ex.getMessage())
            .path(request.getDescription(false))
            .errorCode(ex.getErrorCode())
            .build();

        return ResponseEntity
            .status(HttpStatus.UNPROCESSABLE_ENTITY)
            .body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(
        Exception ex,
        WebRequest request
    ) {
        log.error("Unexpected error occurred", ex);

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
            .error("Internal Server Error")
            .message("An unexpected error occurred")
            .path(request.getDescription(false))
            .build();

        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error);
    }

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(
        AccessDeniedException ex,
        WebRequest request
    ) {
        log.warn("Access denied: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.FORBIDDEN.value())
            .error("Forbidden")
            .message("You don't have permission to access this resource")
            .path(request.getDescription(false))
            .build();

        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }
}
```

### Custom Exception Hierarchy

**✅ Exception Structure:**
```java
// Base Exception
@Getter
public abstract class AppException extends RuntimeException {
    private final String errorCode;
    private final HttpStatus httpStatus;

    protected AppException(
        String message,
        String errorCode,
        HttpStatus httpStatus
    ) {
        super(message);
        this.errorCode = errorCode;
        this.httpStatus = httpStatus;
    }
}

// Business Exceptions
public class ResourceNotFoundException extends AppException {
    public ResourceNotFoundException(String resource, Object id) {
        super(
            String.format("%s not found with id: %s", resource, id),
            "RESOURCE_NOT_FOUND",
            HttpStatus.NOT_FOUND
        );
    }
}

public class ResourceAlreadyExistsException extends AppException {
    public ResourceAlreadyExistsException(String resource, String field, Object value) {
        super(
            String.format("%s already exists with %s: %s", resource, field, value),
            "RESOURCE_ALREADY_EXISTS",
            HttpStatus.CONFLICT
        );
    }
}

public class BusinessException extends AppException {
    public BusinessException(String message, String errorCode) {
        super(message, errorCode, HttpStatus.UNPROCESSABLE_ENTITY);
    }
}

public class InvalidOperationException extends AppException {
    public InvalidOperationException(String message) {
        super(
            message,
            "INVALID_OPERATION",
            HttpStatus.BAD_REQUEST
        );
    }
}
```

### Error Response DTO

**✅ Consistent Error Response:**
```java
@Data
@Builder
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private String errorCode;
    private Map<String, String> validationErrors;
    private String traceId; // For distributed tracing
}
```

### Result Pattern (Alternative to Exceptions)

**✅ For Business Logic:**
```java
@Value
public class Result<T> {
    T data;
    boolean success;
    String errorMessage;
    String errorCode;

    public static <T> Result<T> success(T data) {
        return new Result<>(data, true, null, null);
    }

    public static <T> Result<T> failure(String errorMessage, String errorCode) {
        return new Result<>(null, false, errorMessage, errorCode);
    }

    public boolean isFailure() {
        return !success;
    }
}

// Usage
@Service
@RequiredArgsConstructor
public class TransferService {
    private final AccountRepository accountRepository;

    public Result<Transfer> transferMoney(
        Long fromAccountId,
        Long toAccountId,
        BigDecimal amount
    ) {
        Optional<Account> fromAccount = accountRepository.findById(fromAccountId);
        if (fromAccount.isEmpty()) {
            return Result.failure("Source account not found", "ACCOUNT_NOT_FOUND");
        }

        Optional<Account> toAccount = accountRepository.findById(toAccountId);
        if (toAccount.isEmpty()) {
            return Result.failure("Destination account not found", "ACCOUNT_NOT_FOUND");
        }

        if (fromAccount.get().getBalance().compareTo(amount) < 0) {
            return Result.failure("Insufficient funds", "INSUFFICIENT_FUNDS");
        }

        // Perform transfer
        Transfer transfer = performTransfer(fromAccount.get(), toAccount.get(), amount);
        return Result.success(transfer);
    }
}
```

---

## 6. Validation Patterns

### Bean Validation (JSR-380)

**✅ DTO Validation:**
```java
@Data
public class CreateUserRequest {
    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;

    @NotBlank(message = "Password is required")
    @Pattern(
        regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=])(?=\\S+$).{8,}$",
        message = "Password must contain at least 8 characters, one uppercase, one lowercase, one number and one special character"
    )
    private String password;

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be less than 120")
    private Integer age;

    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;

    @NotNull(message = "Terms acceptance is required")
    @AssertTrue(message = "You must accept the terms and conditions")
    private Boolean acceptedTerms;
}

// Controller
@PostMapping
public ResponseEntity<UserResponse> createUser(
    @Valid @RequestBody CreateUserRequest request
) {
    // @Valid triggers validation automatically
    // If validation fails, MethodArgumentNotValidException is thrown
}
```

### Custom Validators

**✅ Custom Annotation:**
```java
// Annotation
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator
@Component
@RequiredArgsConstructor
public class UniqueEmailValidator
    implements ConstraintValidator<UniqueEmail, String> {

    private final UserRepository userRepository;

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) {
            return true; // Let @NotNull handle this
        }
        return !userRepository.existsByEmail(email);
    }
}

// Usage
@Data
public class CreateUserRequest {
    @NotBlank
    @Email
    @UniqueEmail
    private String email;
}
```

### Validation Groups

**✅ Different Validation Rules for Different Operations:**
```java
public interface CreateValidation {}
public interface UpdateValidation {}

@Data
public class UserDTO {
    @Null(groups = CreateValidation.class, message = "ID must be null for new users")
    @NotNull(groups = UpdateValidation.class, message = "ID is required for updates")
    private Long id;

    @NotBlank(groups = {CreateValidation.class, UpdateValidation.class})
    @Email(groups = {CreateValidation.class, UpdateValidation.class})
    private String email;

    @NotBlank(groups = CreateValidation.class, message = "Password is required")
    @Null(groups = UpdateValidation.class, message = "Use password change endpoint")
    private String password;
}

// Controller
@PostMapping
public ResponseEntity<UserResponse> createUser(
    @Validated(CreateValidation.class) @RequestBody UserDTO userDTO
) {
    // Only CreateValidation constraints are checked
}

@PutMapping("/{id}")
public ResponseEntity<UserResponse> updateUser(
    @PathVariable Long id,
    @Validated(UpdateValidation.class) @RequestBody UserDTO userDTO
) {
    // Only UpdateValidation constraints are checked
}
```

### Business Validation

**✅ Service-Level Validation:**
```java
@Component
public class OrderValidator {

    public void validateCreateOrder(CreateOrderCommand command) {
        List<String> errors = new ArrayList<>();

        if (command.getItems() == null || command.getItems().isEmpty()) {
            errors.add("Order must contain at least one item");
        }

        if (command.getItems() != null) {
            for (OrderItemCommand item : command.getItems()) {
                if (item.getQuantity() <= 0) {
                    errors.add("Quantity must be positive for item: " + item.getProductId());
                }
                if (item.getPrice().compareTo(BigDecimal.ZERO) <= 0) {
                    errors.add("Price must be positive for item: " + item.getProductId());
                }
            }
        }

        BigDecimal total = calculateTotal(command.getItems());
        if (total.compareTo(new BigDecimal("10000")) > 0) {
            errors.add("Order total exceeds maximum allowed amount");
        }

        if (!errors.isEmpty()) {
            throw new ValidationException(String.join(", ", errors));
        }
    }

    private BigDecimal calculateTotal(List<OrderItemCommand> items) {
        return items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

---

## 7. Testing Strategies

### Unit Tests

**✅ Service Unit Test:**
```java
@ExtendWith(MockitoExtension.class)
class CreateUserUseCaseTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private UserValidator userValidator;

    @Mock
    private UserEventPublisher eventPublisher;

    @InjectMocks
    private CreateUserUseCase createUserUseCase;

    @Test
    void shouldCreateUserSuccessfully() {
        // Given
        CreateUserCommand command = CreateUserCommand.builder()
            .email("test@example.com")
            .name("Test User")
            .build();

        User expectedUser = User.builder()
            .id(1L)
            .email(command.getEmail())
            .name(command.getName())
            .status(UserStatus.PENDING)
            .build();

        when(userRepository.existsByEmail(command.getEmail()))
            .thenReturn(false);
        when(userRepository.save(any(User.class)))
            .thenReturn(expectedUser);

        // When
        User result = createUserUseCase.execute(command);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getEmail()).isEqualTo(command.getEmail());
        assertThat(result.getName()).isEqualTo(command.getName());
        assertThat(result.getStatus()).isEqualTo(UserStatus.PENDING);

        verify(userValidator).validate(command);
        verify(userRepository).existsByEmail(command.getEmail());
        verify(userRepository).save(any(User.class));
        verify(eventPublisher).publishUserCreated(expectedUser);
    }

    @Test
    void shouldThrowExceptionWhenEmailAlreadyExists() {
        // Given
        CreateUserCommand command = CreateUserCommand.builder()
            .email("existing@example.com")
            .name("Test User")
            .build();

        when(userRepository.existsByEmail(command.getEmail()))
            .thenReturn(true);

        // When & Then
        assertThatThrownBy(() -> createUserUseCase.execute(command))
            .isInstanceOf(UserAlreadyExistsException.class)
            .hasMessageContaining("existing@example.com");

        verify(userValidator).validate(command);
        verify(userRepository).existsByEmail(command.getEmail());
        verify(userRepository, never()).save(any(User.class));
        verify(eventPublisher, never()).publishUserCreated(any(User.class));
    }

    @Test
    void shouldThrowExceptionWhenValidationFails() {
        // Given
        CreateUserCommand command = CreateUserCommand.builder()
            .email("invalid-email")
            .name("")
            .build();

        doThrow(new ValidationException("Invalid email"))
            .when(userValidator).validate(command);

        // When & Then
        assertThatThrownBy(() -> createUserUseCase.execute(command))
            .isInstanceOf(ValidationException.class)
            .hasMessage("Invalid email");

        verify(userValidator).validate(command);
        verify(userRepository, never()).existsByEmail(anyString());
        verify(userRepository, never()).save(any(User.class));
    }
}
```

### Integration Tests

**✅ Repository Integration Test:**
```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void shouldFindUserByEmail() {
        // Given
        User user = User.builder()
            .email("test@example.com")
            .name("Test User")
            .status(UserStatus.ACTIVE)
            .build();
        entityManager.persist(user);
        entityManager.flush();

        // When
        Optional<User> found = userRepository.findByEmail("test@example.com");

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("test@example.com");
        assertThat(found.get().getName()).isEqualTo("Test User");
    }

    @Test
    void shouldReturnEmptyWhenUserNotFound() {
        // When
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");

        // Then
        assertThat(found).isEmpty();
    }

    @Test
    void shouldFindUsersByStatus() {
        // Given
        User activeUser1 = createUser("active1@example.com", UserStatus.ACTIVE);
        User activeUser2 = createUser("active2@example.com", UserStatus.ACTIVE);
        User pendingUser = createUser("pending@example.com", UserStatus.PENDING);

        entityManager.persist(activeUser1);
        entityManager.persist(activeUser2);
        entityManager.persist(pendingUser);
        entityManager.flush();

        // When
        List<User> activeUsers = userRepository.findByStatus(UserStatus.ACTIVE);

        // Then
        assertThat(activeUsers).hasSize(2);
        assertThat(activeUsers)
            .extracting(User::getStatus)
            .containsOnly(UserStatus.ACTIVE);
    }

    private User createUser(String email, UserStatus status) {
        return User.builder()
            .email(email)
            .name("Test User")
            .status(status)
            .build();
    }
}
```

**✅ API Integration Test:**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateUserSuccessfully() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
            .email("newuser@example.com")
            .name("New User")
            .password("SecurePass123!")
            .build();

        // When
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
            "/api/users",
            request,
            UserResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody()).isNotNull();
        assertThat(response.getBody().getEmail()).isEqualTo("newuser@example.com");
        assertThat(response.getBody().getName()).isEqualTo("New User");

        // Verify in database
        Optional<User> savedUser = userRepository.findByEmail("newuser@example.com");
        assertThat(savedUser).isPresent();
    }

    @Test
    void shouldReturnBadRequestForInvalidEmail() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
            .email("invalid-email")
            .name("Test User")
            .password("SecurePass123!")
            .build();

        // When
        ResponseEntity<ErrorResponse> response = restTemplate.postForEntity(
            "/api/users",
            request,
            ErrorResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
        assertThat(response.getBody()).isNotNull();
        assertThat(response.getBody().getMessage()).contains("Email must be valid");
    }

    @Test
    void shouldReturnConflictWhenEmailAlreadyExists() {
        // Given
        User existingUser = User.builder()
            .email("existing@example.com")
            .name("Existing User")
            .status(UserStatus.ACTIVE)
            .build();
        userRepository.save(existingUser);

        CreateUserRequest request = CreateUserRequest.builder()
            .email("existing@example.com")
            .name("New User")
            .password("SecurePass123!")
            .build();

        // When
        ResponseEntity<ErrorResponse> response = restTemplate.postForEntity(
            "/api/users",
            request,
            ErrorResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CONFLICT);
    }
}
```

### E2E Tests

**✅ End-to-End Test:**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserE2ETest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @Test
    void completeUserLifecycle() {
        // 1. Create user
        CreateUserRequest createRequest = CreateUserRequest.builder()
            .email("lifecycle@example.com")
            .name("Lifecycle User")
            .password("SecurePass123!")
            .build();

        ResponseEntity<UserResponse> createResponse = restTemplate.postForEntity(
            "/api/users",
            createRequest,
            UserResponse.class
        );

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        Long userId = createResponse.getBody().getId();

        // 2. Get user
        ResponseEntity<UserResponse> getResponse = restTemplate.getForEntity(
            "/api/users/" + userId,
            UserResponse.class
        );

        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().getEmail()).isEqualTo("lifecycle@example.com");

        // 3. Update user
        UpdateUserRequest updateRequest = UpdateUserRequest.builder()
            .name("Updated Name")
            .build();

        restTemplate.put(
            "/api/users/" + userId,
            updateRequest
        );

        ResponseEntity<UserResponse> updatedResponse = restTemplate.getForEntity(
            "/api/users/" + userId,
            UserResponse.class
        );

        assertThat(updatedResponse.getBody().getName()).isEqualTo("Updated Name");

        // 4. Delete user
        restTemplate.delete("/api/users/" + userId);

        ResponseEntity<ErrorResponse> deletedResponse = restTemplate.getForEntity(
            "/api/users/" + userId,
            ErrorResponse.class
        );

        assertThat(deletedResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    }
}
```

### Test Configuration

**✅ Test Properties:**
```yaml
# src/test/resources/application-test.yml
spring:
  datasource:
    url: jdbc:tc:postgresql:15:///testdb
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  mail:
    host: localhost
    port: 3025

logging:
  level:
    com.example.app: DEBUG
    org.hibernate.SQL: DEBUG
```

---

## 8. API Design Best Practices

### RESTful API Design

**✅ Resource Naming:**
```java
// ✅ Good - Plural nouns for collections
GET    /api/users
POST   /api/users
GET    /api/users/{id}
PUT    /api/users/{id}
DELETE /users/{id}
PATCH  /api/users/{id}

// ✅ Nested resources
GET    /api/users/{userId}/orders
POST   /api/users/{userId}/orders
GET    /api/users/{userId}/orders/{orderId}

// ✅ Actions as resources when necessary
POST   /api/users/{id}/activate
POST   /api/users/{id}/deactivate
POST   /api/orders/{id}/cancel
POST   /api/orders/{id}/refund

// ❌ Bad - Avoid verbs in URLs
POST   /api/createUser
GET    /api/getUser/{id}
POST   /api/deleteUser/{id}
```

### HTTP Status Codes

**✅ Proper Status Code Usage:**
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    // 200 OK - Successful GET, PUT, PATCH
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUser(id));
    }

    // 201 Created - Successful POST (resource created)
    @PostMapping
    public ResponseEntity<UserResponse> createUser(@RequestBody CreateUserRequest request) {
        UserResponse user = userService.createUser(request);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(user.getId())
            .toUri();
        return ResponseEntity.created(location).body(user);
    }

    // 204 No Content - Successful DELETE
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }

    // 400 Bad Request - Invalid input
    // Handled by @Valid and GlobalExceptionHandler

    // 404 Not Found - Resource doesn't exist
    // Thrown by service layer as ResourceNotFoundException

    // 409 Conflict - Resource already exists
    // Thrown by service layer as ResourceAlreadyExistsException

    // 422 Unprocessable Entity - Business rule violation
    // Thrown by service layer as BusinessException
}
```

### API Versioning

**✅ URL Versioning:**
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
    // Version 1 implementation
}

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    // Version 2 implementation with breaking changes
}
```

### Pagination & Sorting

**✅ Pagination Implementation:**
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @GetMapping
    public ResponseEntity<PageResponse<UserResponse>> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "id,asc") String[] sort
    ) {
        Pageable pageable = PageRequest.of(
            page,
            size,
            Sort.by(parseSortParams(sort))
        );

        Page<UserResponse> userPage = userService.getUsers(pageable);

        PageResponse<UserResponse> response = PageResponse.<UserResponse>builder()
            .content(userPage.getContent())
            .pageNumber(userPage.getNumber())
            .pageSize(userPage.getSize())
            .totalElements(userPage.getTotalElements())
            .totalPages(userPage.getTotalPages())
            .first(userPage.isFirst())
            .last(userPage.isLast())
            .build();

        return ResponseEntity.ok(response);
    }

    private Sort.Order[] parseSortParams(String[] sort) {
        return Arrays.stream(sort)
            .map(s -> {
                String[] parts = s.split(",");
                String property = parts[0];
                Sort.Direction direction = parts.length > 1 && parts[1].equalsIgnoreCase("desc")
                    ? Sort.Direction.DESC
                    : Sort.Direction.ASC;
                return new Sort.Order(direction, property);
            })
            .toArray(Sort.Order[]::new);
    }
}

// Response DTO
@Data
@Builder
public class PageResponse<T> {
    private List<T> content;
    private int pageNumber;
    private int pageSize;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;
}
```

### Filtering & Searching

**✅ Query Parameters:**
```java
@GetMapping("/search")
public ResponseEntity<List<UserResponse>> searchUsers(
    @RequestParam(required = false) String email,
    @RequestParam(required = false) String name,
    @RequestParam(required = false) UserStatus status,
    @RequestParam(required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate createdAfter,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size
) {
    UserSearchCriteria criteria = UserSearchCriteria.builder()
        .email(email)
        .name(name)
        .status(status)
        .createdAfter(createdAfter)
        .build();

    Pageable pageable = PageRequest.of(page, size);
    Page<UserResponse> results = userService.searchUsers(criteria, pageable);

    return ResponseEntity.ok(results.getContent());
}
```

### HATEOAS (Hypermedia)

**✅ Adding Links:**
```java
@GetMapping("/{id}")
public ResponseEntity<EntityModel<UserResponse>> getUser(@PathVariable Long id) {
    UserResponse user = userService.getUser(id);

    EntityModel<UserResponse> resource = EntityModel.of(user);

    resource.add(linkTo(methodOn(UserController.class).getUser(id)).withSelfRel());
    resource.add(linkTo(methodOn(UserController.class).getUsers(0, 20, new String[]{"id,asc"})).withRel("users"));
    resource.add(linkTo(methodOn(OrderController.class).getUserOrders(id, 0, 20)).withRel("orders"));

    return ResponseEntity.ok(resource);
}
```

---

## 9. Security Best Practices

### Spring Security Configuration

**✅ Security Configuration:**
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/users/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/users/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.PUT, "/api/users/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/users/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authenticationEntryPoint())
                .accessDeniedHandler(accessDeniedHandler())
            );

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    public AuthenticationManager authenticationManager(
        AuthenticationConfiguration config
    ) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### JWT Authentication

**✅ JWT Service:**
```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private long expiration;

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));

        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    private <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

### Method-Level Security

**✅ Using @PreAuthorize:**
```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @PreAuthorize("hasRole('USER') or hasRole('ADMIN')")
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }

    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    public User updateUser(Long id, UpdateUserRequest request) {
        // User can update their own profile or admin can update any
        User user = getUser(id);
        user.setName(request.getName());
        return userRepository.save(user);
    }

    @PreAuthorize("@userSecurity.canAccessOrder(#orderId, authentication)")
    public Order getOrder(Long orderId) {
        // Custom security check
        return orderRepository.findById(orderId)
            .orElseThrow(() -> new ResourceNotFoundException("Order", orderId));
    }
}

@Component
public class UserSecurity {
    @Autowired
    private OrderRepository orderRepository;

    public boolean canAccessOrder(Long orderId, Authentication authentication) {
        Order order = orderRepository.findById(orderId).orElse(null);
        if (order == null) {
            return false;
        }

        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        return order.getUserEmail().equals(userDetails.getUsername()) ||
               authentication.getAuthorities().stream()
                   .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
    }
}
```

### Input Sanitization

**✅ Prevent XSS:**
```java
@Component
public class HtmlSanitizer {
    private final Policy policy;

    public HtmlSanitizer() {
        this.policy = new HtmlPolicyBuilder()
            .allowElements("p", "br", "strong", "em", "u")
            .allowTextIn("p")
            .toFactory();
    }

    public String sanitize(String input) {
        if (input == null) {
            return null;
        }
        return policy.sanitize(input);
    }
}

@Service
@RequiredArgsConstructor
public class PostService {
    private final HtmlSanitizer htmlSanitizer;
    private final PostRepository postRepository;

    public Post createPost(CreatePostRequest request) {
        String sanitizedContent = htmlSanitizer.sanitize(request.getContent());

        Post post = Post.builder()
            .title(request.getTitle())
            .content(sanitizedContent)
            .build();

        return postRepository.save(post);
    }
}
```

### SQL Injection Prevention

**✅ Always use parameterized queries:**
```java
// ✅ Good - Parameterized query
@Query("SELECT u FROM User u WHERE u.email = :email")
Optional<User> findByEmail(@Param("email") String email);

// ✅ Good - Criteria API
public List<User> searchUsers(String email) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> user = query.from(User.class);

    query.where(cb.equal(user.get("email"), email));

    return entityManager.createQuery(query).getResultList();
}

// ❌ Bad - String concatenation (vulnerable to SQL injection)
@Query(value = "SELECT * FROM users WHERE email = '" + email + "'", nativeQuery = true)
List<User> findByEmailUnsafe(String email);
```

### Rate Limiting

**✅ Using Bucket4j:**
```java
@Component
public class RateLimitingFilter extends OncePerRequestFilter {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {

        String key = getClientKey(request);
        Bucket bucket = resolveBucket(key);

        if (bucket.tryConsume(1)) {
            filterChain.doFilter(request, response);
        } else {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Too many requests");
        }
    }

    private Bucket resolveBucket(String key) {
        return cache.computeIfAbsent(key, k -> createNewBucket());
    }

    private Bucket createNewBucket() {
        Bandwidth limit = Bandwidth.builder()
            .capacity(100)
            .refillGreedy(100, Duration.ofMinutes(1))
            .build();

        return Bucket.builder()
            .addLimit(limit)
            .build();
    }

    private String getClientKey(HttpServletRequest request) {
        // Use IP address or authenticated user ID
        String ip = request.getRemoteAddr();
        String userId = request.getUserPrincipal() != null
            ? request.getUserPrincipal().getName()
            : ip;
        return userId;
    }
}
```

---

## 10. Performance Optimization

### Database Optimization

**✅ N+1 Query Problem:**
```java
// ❌ Bad - N+1 queries
@GetMapping("/orders")
public List<OrderDTO> getOrders() {
    List<Order> orders = orderRepository.findAll(); // 1 query

    return orders.stream()
        .map(order -> {
            // N additional queries for each order's items
            List<OrderItem> items = orderItemRepository.findByOrderId(order.getId());
            return new OrderDTO(order, items);
        })
        .collect(Collectors.toList());
}

// ✅ Good - Single query with JOIN FETCH
@Entity
public class Order {
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
}

@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    @Query("SELECT o FROM Order o LEFT JOIN FETCH o.items WHERE o.id IN :ids")
    List<Order> findAllWithItems(@Param("ids") List<Long> ids);

    @Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items")
    List<Order> findAllWithItems();
}
```

### Caching

**✅ Spring Cache:**
```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "products", "categories");
    }
}

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    @Cacheable(value = "users", key = "#id")
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }

    @CachePut(value = "users", key = "#result.id")
    public User updateUser(Long id, UpdateUserRequest request) {
        User user = getUser(id);
        user.setName(request.getName());
        return userRepository.save(user);
    }

    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    @CacheEvict(value = "users", allEntries = true)
    public void clearUserCache() {
        // Clear all user cache entries
    }
}
```

**✅ Redis Cache:**
```java
@Configuration
@EnableCaching
public class RedisCacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()
                )
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}
```

### Async Processing

**✅ @Async Methods:**
```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
@RequiredArgsConstructor
public class NotificationService {
    private final EmailService emailService;
    private final SmsService smsService;

    @Async
    public CompletableFuture<Void> sendWelcomeNotifications(User user) {
        return CompletableFuture.allOf(
            sendWelcomeEmail(user),
            sendWelcomeSms(user)
        );
    }

    @Async
    public CompletableFuture<Void> sendWelcomeEmail(User user) {
        emailService.send(user.getEmail(), "Welcome!", "Welcome to our platform");
        return CompletableFuture.completedFuture(null);
    }

    @Async
    public CompletableFuture<Void> sendWelcomeSms(User user) {
        if (user.getPhoneNumber() != null) {
            smsService.send(user.getPhoneNumber(), "Welcome to our platform!");
        }
        return CompletableFuture.completedFuture(null);
    }
}
```

### Database Connection Pooling

**✅ HikariCP Configuration:**
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      pool-name: HikariPool
      auto-commit: false
      leak-detection-threshold: 60000
```

### Pagination for Large Datasets

**✅ Cursor-Based Pagination:**
```java
@GetMapping("/users/cursor")
public ResponseEntity<CursorPageResponse<UserResponse>> getUsersCursor(
    @RequestParam(required = false) Long cursor,
    @RequestParam(defaultValue = "20") int limit
) {
    List<User> users;

    if (cursor == null) {
        users = userRepository.findTopNOrderById(limit + 1);
    } else {
        users = userRepository.findTopNAfterCursor(cursor, limit + 1);
    }

    boolean hasNext = users.size() > limit;
    if (hasNext) {
        users = users.subList(0, limit);
    }

    Long nextCursor = hasNext && !users.isEmpty()
        ? users.get(users.size() - 1).getId()
        : null;

    List<UserResponse> responses = users.stream()
        .map(UserMapper::toResponse)
        .collect(Collectors.toList());

    CursorPageResponse<UserResponse> response = CursorPageResponse.<UserResponse>builder()
        .data(responses)
        .nextCursor(nextCursor)
        .hasNext(hasNext)
        .build();

    return ResponseEntity.ok(response);
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT u FROM User u ORDER BY u.id ASC")
    List<User> findTopNOrderById(Pageable pageable);

    @Query("SELECT u FROM User u WHERE u.id > :cursor ORDER BY u.id ASC")
    List<User> findTopNAfterCursor(@Param("cursor") Long cursor, Pageable pageable);
}
```

---

## 11. Code Organization and Naming Conventions

### Package Naming

**✅ Conventions:**
```
com.company.product.feature.layer

Examples:
com.example.ecommerce.user.domain
com.example.ecommerce.user.application
com.example.ecommerce.order.infrastructure
```

### Class Naming

**✅ Conventions:**
```java
// Controllers
UserController, OrderController, ProductController

// Services/Use Cases
CreateUserUseCase, UpdateOrderUseCase, ProcessPaymentService

// Repositories
UserRepository, OrderRepository

// Entities
User, Order, Product

// DTOs
CreateUserRequest, UpdateUserRequest, UserResponse
CreateOrderCommand, UpdateOrderCommand

// Exceptions
UserNotFoundException, OrderAlreadyProcessedException

// Validators
UserValidator, OrderValidator

// Mappers
UserMapper, OrderMapper

// Configurations
SecurityConfig, DatabaseConfig, CacheConfig
```

### Method Naming

**✅ Conventions:**
```java
// CRUD operations
create(), update(), delete(), findById(), findAll()

// Boolean methods
isActive(), hasPermission(), canAccess(), exists()

// Getters/Setters
getUser(), setUser()

// Business operations
processOrder(), cancelOrder(), refundPayment()

// Validation
validateUser(), validateOrder()

// Conversion
toDTO(), toEntity(), toResponse()
```

### Variable Naming

**✅ Conventions:**
```java
// Local variables - camelCase
User user = ...;
OrderItem orderItem = ...;
List<Product> products = ...;

// Constants - UPPER_SNAKE_CASE
private static final int MAX_RETRY_ATTEMPTS = 3;
private static final String DEFAULT_CURRENCY = "USD";

// Private fields - camelCase with descriptive names
private final UserRepository userRepository;
private final EmailService emailService;

// Boolean variables - use 'is', 'has', 'can'
boolean isActive;
boolean hasPermission;
boolean canProceed;
```

### Comment Conventions

**✅ JavaDoc for Public APIs:**
```java
/**
 * Creates a new user in the system.
 *
 * @param command the user creation command containing user details
 * @return the created user entity
 * @throws UserAlreadyExistsException if a user with the same email already exists
 * @throws ValidationException if the input data is invalid
 */
public User createUser(CreateUserCommand command) {
    // Implementation
}
```

**✅ Inline Comments:**
```java
// Good: Explain WHY, not WHAT
// Using pessimistic lock to prevent double-booking
User user = userRepository.findByIdWithLock(id);

// Bad: Redundant comment
// Get user by id
User user = userRepository.findById(id);
```

---

## 12. Common Pitfalls and Anti-Patterns

### Anti-Pattern: God Class

**❌ Bad:**
```java
@Service
public class UserService {
    // Handles everything related to users - too much responsibility

    public User createUser() { }
    public void sendEmail() { }
    public void validateUser() { }
    public void generateReport() { }
    public void exportToExcel() { }
    public void calculateStatistics() { }
    public void processPayment() { }
    // ... 50 more methods
}
```

**✅ Good:**
```java
@Service
public class CreateUserUseCase { }

@Service
public class UserEmailService { }

@Component
public class UserValidator { }

@Service
public class UserReportService { }

@Service
public class UserStatisticsService { }
```

### Anti-Pattern: Anemic Domain Model

**❌ Bad:**
```java
@Entity
public class Order {
    private Long id;
    private OrderStatus status;
    private BigDecimal total;
    // Only getters and setters - no business logic
}

@Service
public class OrderService {
    public void processOrder(Order order) {
        // All business logic here
        if (order.getStatus() == OrderStatus.PENDING) {
            order.setStatus(OrderStatus.PROCESSING);
            // Calculate total
            // Apply discounts
            // Validate items
        }
    }
}
```

**✅ Good:**
```java
@Entity
public class Order {
    private Long id;
    private OrderStatus status;
    private BigDecimal total;

    // Business logic in domain model
    public void process() {
        if (this.status != OrderStatus.PENDING) {
            throw new InvalidOperationException(
                "Only pending orders can be processed"
            );
        }
        this.status = OrderStatus.PROCESSING;
    }

    public void cancel() {
        if (this.status == OrderStatus.DELIVERED) {
            throw new InvalidOperationException(
                "Delivered orders cannot be cancelled"
            );
        }
        this.status = OrderStatus.CANCELLED;
    }

    public boolean canBeCancelled() {
        return this.status != OrderStatus.DELIVERED;
    }
}
```

### Anti-Pattern: Repository in Controller

**❌ Bad:**
```java
@RestController
public class UserController {
    @Autowired
    private UserRepository userRepository; // Direct repository access!

    @PostMapping("/users")
    public User createUser(@RequestBody User user) {
        return userRepository.save(user); // No business logic layer
    }
}
```

**✅ Good:**
```java
@RestController
@RequiredArgsConstructor
public class UserController {
    private final CreateUserUseCase createUserUseCase;

    @PostMapping("/users")
    public ResponseEntity<UserResponse> createUser(
        @Valid @RequestBody CreateUserRequest request
    ) {
        User user = createUserUseCase.execute(request);
        return ResponseEntity.created(location).body(toResponse(user));
    }
}
```

### Anti-Pattern: Transaction Script

**❌ Bad:**
```java
@Service
public class TransferService {
    public void transfer(Long from, Long to, BigDecimal amount) {
        // Long procedural method
        Account fromAccount = accountRepository.findById(from).get();
        Account toAccount = accountRepository.findById(to).get();

        if (fromAccount.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException();
        }

        fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
        toAccount.setBalance(toAccount.getBalance().add(amount));

        accountRepository.save(fromAccount);
        accountRepository.save(toAccount);

        // Log transaction
        // Send notification
        // Update statistics
        // ... more procedural code
    }
}
```

**✅ Good:**
```java
@Service
@RequiredArgsConstructor
public class TransferMoneyUseCase {
    private final AccountRepository accountRepository;
    private final TransactionLogger transactionLogger;
    private final NotificationService notificationService;

    @Transactional
    public Transfer execute(TransferCommand command) {
        Account source = findAccount(command.getSourceId());
        Account target = findAccount(command.getTargetId());

        // Domain logic
        source.withdraw(command.getAmount());
        target.deposit(command.getAmount());

        accountRepository.save(source);
        accountRepository.save(target);

        Transfer transfer = Transfer.between(source, target, command.getAmount());

        // Side effects
        transactionLogger.log(transfer);
        notificationService.notifyTransferCompleted(transfer);

        return transfer;
    }

    private Account findAccount(Long id) {
        return accountRepository.findById(id)
            .orElseThrow(() -> new AccountNotFoundException(id));
    }
}

// Domain model with business logic
@Entity
public class Account {
    public void withdraw(BigDecimal amount) {
        if (balance.compareTo(amount) < 0) {
            throw new InsufficientFundsException(id, balance, amount);
        }
        this.balance = this.balance.subtract(amount);
    }

    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidAmountException(amount);
        }
        this.balance = this.balance.add(amount);
    }
}
```

### Anti-Pattern: String-based Errors

**❌ Bad:**
```java
if (user == null) {
    return "User not found";
}

if (amount < 0) {
    throw new RuntimeException("Invalid amount");
}
```

**✅ Good:**
```java
if (user == null) {
    throw new UserNotFoundException(userId);
}

if (amount.compareTo(BigDecimal.ZERO) < 0) {
    throw new InvalidAmountException(amount);
}
```

### Anti-Pattern: Returning Null

**❌ Bad:**
```java
public User findUser(Long id) {
    return userRepository.findById(id).orElse(null); // Returns null!
}

// Caller has to check for null everywhere
User user = userService.findUser(id);
if (user != null) {
    // use user
}
```

**✅ Good:**
```java
// Option 1: Return Optional
public Optional<User> findUser(Long id) {
    return userRepository.findById(id);
}

// Option 2: Throw exception
public User getUser(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}
```

### Anti-Pattern: Premature Optimization

**❌ Bad:**
```java
// Over-engineering for scale you don't have yet
@Service
public class UserService {
    @Cacheable
    @Async
    @Retry(maxAttempts = 3)
    @CircuitBreaker
    public CompletableFuture<User> getUser(Long id) {
        // Simple database lookup doesn't need all this!
        return CompletableFuture.completedFuture(
            userRepository.findById(id).orElseThrow()
        );
    }
}
```

**✅ Good:**
```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    // Add optimizations when you have metrics showing they're needed
}
```

---

## 13. Migration Path from Android to Spring

### Conceptual Mapping

| Android Concept | Spring Boot Equivalent | Notes |
|----------------|------------------------|-------|
| `Activity`/`Fragment` | `@RestController` | Handles user interaction, but via HTTP |
| `ViewModel` | `@Service` / Use Case | Business logic layer |
| `Repository` | `@Repository` / Spring Data JPA | Data access layer |
| `Room Database` | `JPA` / `Hibernate` | ORM framework |
| `LiveData` | `ResponseEntity` / SSE | Data observation (different paradigm) |
| `Retrofit` | `RestTemplate` / `WebClient` | HTTP client |
| `Dagger/Hilt` | Spring DI (`@Autowired`) | Dependency injection |
| `Coroutines` | `@Async` / `CompletableFuture` | Async programming |
| `WorkManager` | `@Scheduled` / Spring Batch | Background tasks |
| `SharedPreferences` | `application.properties` | Configuration |
| `Navigation` | HTTP routing (`@RequestMapping`) | Navigation between screens |

### Android to Spring Patterns

**Android Repository Pattern:**
```kotlin
// Android
class UserRepository @Inject constructor(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) {
    suspend fun getUser(id: Long): User {
        return try {
            val remoteUser = remoteDataSource.getUser(id)
            localDataSource.saveUser(remoteUser)
            remoteUser
        } catch (e: Exception) {
            localDataSource.getUser(id)
        }
    }
}
```

**Spring Repository Pattern:**
```java
// Spring
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findById(Long id);
}

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final UserCacheRepository cacheRepository;

    public User getUser(Long id) {
        return cacheRepository.find(id)
            .orElseGet(() -> {
                User user = userRepository.findById(id)
                    .orElseThrow(() -> new UserNotFoundException(id));
                cacheRepository.save(user);
                return user;
            });
    }
}
```

### Learning Path

**Phase 1: Fundamentals (Week 1-2)**
1. Java basics (if coming from Kotlin-only background)
2. Spring Core & Dependency Injection
3. Spring Boot basics
4. REST API fundamentals

**Phase 2: Data Layer (Week 3-4)**
1. JPA & Hibernate
2. Database design
3. Spring Data JPA
4. Database migrations (Flyway/Liquibase)

**Phase 3: Business Logic (Week 5-6)**
1. Service layer design
2. Transaction management
3. Validation
4. Error handling

**Phase 4: Advanced Topics (Week 7-8)**
1. Security (Spring Security, JWT)
2. Testing strategies
3. Caching
4. Async processing

**Phase 5: Production Ready (Week 9-10)**
1. Monitoring & Logging
2. Performance optimization
3. Deployment strategies
4. CI/CD pipelines

### Quick Start Project

**✅ Hello World API:**
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public ResponseEntity<Map<String, String>> hello(
        @RequestParam(defaultValue = "World") String name
    ) {
        Map<String, String> response = Map.of(
            "message", "Hello, " + name + "!",
            "timestamp", LocalDateTime.now().toString()
        );
        return ResponseEntity.ok(response);
    }
}
```

### Resources for Android Developers

**📚 Recommended Learning Order:**
1. **Spring Initializr** (start.spring.io) - Generate starter projects
2. **Spring Guides** (spring.io/guides) - Official tutorials
3. **Baeldung** (baeldung.com) - Comprehensive Spring tutorials
4. **Spring Boot Reference Documentation** - Official docs

**🔧 Tools:**
- IntelliJ IDEA (familiar if you used Android Studio)
- Postman (API testing)
- DBeaver (database management)
- Docker (containerization)

**💡 Key Mindset Shifts:**
1. **Synchronous by default** - Unlike Android where UI must be async, Spring handles threads for you
2. **Stateless services** - No activity lifecycle, each request is independent
3. **Database-centric** - Data persistence is primary, not an afterthought
4. **Server-side validation** - Trust nothing from the client
5. **Horizontal scaling** - Design for multiple instances

---

## Summary

This guide covered enterprise best practices for Spring Boot development from an Android developer's perspective:

1. **Clean Architecture** - Separating concerns into layers (similar to Android MVVM)
2. **SOLID Principles** - Writing maintainable, extensible code
3. **Design Patterns** - Factory, Builder, Strategy, Observer, Repository
4. **Project Structure** - Package-by-feature for better organization
5. **Error Handling** - Global exception handling with custom exceptions
6. **Validation** - Bean Validation and custom validators
7. **Testing** - Unit, Integration, and E2E testing strategies
8. **API Design** - RESTful conventions, versioning, pagination
9. **Security** - Authentication, authorization, input sanitization
10. **Performance** - Caching, async processing, database optimization
11. **Code Organization** - Naming conventions and best practices
12. **Anti-Patterns** - Common mistakes to avoid
13. **Migration Path** - Your journey from Android to Spring Boot

**Next Steps:**
- Build a simple CRUD API
- Add authentication
- Implement caching
- Write comprehensive tests
- Deploy to production

Remember: Spring Boot development shares many concepts with Android (DI, repositories, layered architecture), but operates in a different paradigm (stateless, request-response, database-centric). Leverage your Android knowledge while embracing backend-specific patterns.

Happy coding! 🚀
