# Redis, Messaging, and Real-Time Features

Comprehensive guide covering Redis caching, message queues, WebSockets, email, file storage, and other essential backend concepts for Kotlin Android developers.

## 🎯 Overview

This guide covers advanced backend features you'll need in production applications. If you've used **SharedPreferences** for caching, **Firebase Cloud Messaging** for push notifications, or **WebSockets** for real-time features, you'll find Spring equivalents here.

## 📋 Android vs Backend Features

| Feature | Android | Spring Backend |
|---------|---------|----------------|
| Simple Caching | SharedPreferences | Redis |
| Complex Caching | Room Database | Redis + Spring Cache |
| Push Notifications | FCM | WebSockets, SSE, FCM Admin SDK |
| Background Jobs | WorkManager | Spring Batch, @Scheduled, Quartz |
| Message Queue | N/A | RabbitMQ, Kafka |
| Real-time Data | WebSocket, Firebase Realtime DB | WebSocket, Server-Sent Events |
| Email | Intent to email app | JavaMail, SendGrid, AWS SES |
| File Storage | Internal/External storage | S3, Google Cloud Storage, Local |
| Search | SQLite FTS | Elasticsearch, Full-text search |

---

## 1. Redis Caching

### 1.1 What is Redis?

**Redis** = Remote Dictionary Server - In-memory data store used for:
- Caching (most common use)
- Session storage
- Real-time analytics
- Message broker
- Rate limiting

**Think of it as:** SharedPreferences/Room but on the server, shared across all users, super fast (in-memory), with automatic expiration.

### 1.2 Setup Redis with Spring Boot

**Dependencies (Maven):**
```xml
<dependencies>
    <!-- Spring Data Redis -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <!-- Lettuce (Redis client) - included by default -->
    <!-- Or use Jedis -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>

    <!-- Spring Cache abstraction -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
</dependencies>
```

**Gradle:**
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'org.springframework.boot:spring-boot-starter-cache'
}
```

**Configuration (application.yml):**
```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: ${REDIS_PASSWORD:}  # Optional
    timeout: 60000
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: -1ms

  cache:
    type: redis
    redis:
      time-to-live: 600000  # 10 minutes default TTL
```

**Install Redis (Development):**
```bash
# Using Docker (easiest)
docker run -d -p 6379:6379 --name redis redis:alpine

# Or using package manager
# macOS
brew install redis
brew services start redis

# Ubuntu
sudo apt-get install redis-server
sudo systemctl start redis
```

### 1.3 Basic Redis Configuration

```java
package com.example.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.*;

import java.time.Duration;

@Configuration
@EnableCaching
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // JSON serialization
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        serializer.setObjectMapper(objectMapper);

        // Key serializer
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());

        // Value serializer
        template.setValueSerializer(serializer);
        template.setHashValueSerializer(serializer);

        template.afterPropertiesSet();
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))  // Default TTL
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            )
            .disableCachingNullValues();

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}
```

### 1.4 Spring Cache Annotations (Simple Caching)

**Service with caching:**
```java
package com.example.service;

import com.example.model.Product;
import com.example.repository.ProductRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
@Slf4j
@CacheConfig(cacheNames = "products")  // Default cache name
public class ProductService {

    private final ProductRepository productRepository;

    // Cache the result
    @Cacheable(key = "#id")
    public Product getProduct(Long id) {
        log.info("Fetching product from database: {}", id);
        return productRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Product not found"));
    }

    // Cache with custom key
    @Cacheable(key = "'category:' + #category")
    public List<Product> getProductsByCategory(String category) {
        log.info("Fetching products by category from database: {}", category);
        return productRepository.findByCategory(category);
    }

    // Cache with condition
    @Cacheable(key = "#id", condition = "#id > 100")
    public Product getExpensiveProduct(Long id) {
        return productRepository.findById(id).orElse(null);
    }

    // Update cache after saving
    @CachePut(key = "#product.id")
    public Product updateProduct(Product product) {
        log.info("Updating product in database: {}", product.getId());
        return productRepository.save(product);
    }

    // Remove from cache after deletion
    @CacheEvict(key = "#id")
    public void deleteProduct(Long id) {
        log.info("Deleting product from database: {}", id);
        productRepository.deleteById(id);
    }

    // Clear entire cache
    @CacheEvict(allEntries = true)
    public void clearCache() {
        log.info("Clearing all product cache");
    }

    // Multiple cache operations
    @Caching(
        evict = {
            @CacheEvict(value = "products", allEntries = true),
            @CacheEvict(value = "productCategories", allEntries = true)
        }
    )
    public void refreshProducts() {
        log.info("Refreshing product data");
        // Refresh logic
    }
}
```

**Android Comparison:**
```kotlin
// Similar to SharedPreferences in Android
class ProductService(
    private val repository: ProductRepository,
    private val sharedPrefs: SharedPreferences
) {
    fun getProduct(id: Long): Product {
        // Check cache first
        val cached = sharedPrefs.getString("product_$id", null)
        if (cached != null) {
            return Json.decodeFromString(cached)
        }

        // Fetch from database
        val product = repository.findById(id)

        // Save to cache
        sharedPrefs.edit().putString("product_$id", Json.encodeToString(product)).apply()

        return product
    }
}
```

### 1.5 Direct Redis Operations (RedisTemplate)

```java
package com.example.service;

import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

import java.util.*;
import java.util.concurrent.TimeUnit;

@Service
@RequiredArgsConstructor
public class RedisService {

    private final RedisTemplate<String, Object> redisTemplate;

    // ===== String Operations =====

    public void setValue(String key, Object value) {
        redisTemplate.opsForValue().set(key, value);
    }

    public void setValueWithExpiry(String key, Object value, long timeout, TimeUnit unit) {
        redisTemplate.opsForValue().set(key, value, timeout, unit);
    }

    public Object getValue(String key) {
        return redisTemplate.opsForValue().get(key);
    }

    public Boolean deleteKey(String key) {
        return redisTemplate.delete(key);
    }

    public Boolean hasKey(String key) {
        return redisTemplate.hasKey(key);
    }

    public Boolean expire(String key, long timeout, TimeUnit unit) {
        return redisTemplate.expire(key, timeout, unit);
    }

    // ===== Hash Operations (like HashMap) =====

    public void hashPut(String key, String field, Object value) {
        redisTemplate.opsForHash().put(key, field, value);
    }

    public Object hashGet(String key, String field) {
        return redisTemplate.opsForHash().get(key, field);
    }

    public Map<Object, Object> hashGetAll(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    public Boolean hashExists(String key, String field) {
        return redisTemplate.opsForHash().hasKey(key, field);
    }

    public Long hashDelete(String key, Object... fields) {
        return redisTemplate.opsForHash().delete(key, fields);
    }

    // ===== List Operations =====

    public Long listRightPush(String key, Object value) {
        return redisTemplate.opsForList().rightPush(key, value);
    }

    public Object listLeftPop(String key) {
        return redisTemplate.opsForList().leftPop(key);
    }

    public List<Object> listRange(String key, long start, long end) {
        return redisTemplate.opsForList().range(key, start, end);
    }

    public Long listSize(String key) {
        return redisTemplate.opsForList().size(key);
    }

    // ===== Set Operations =====

    public Long setAdd(String key, Object... values) {
        return redisTemplate.opsForSet().add(key, values);
    }

    public Set<Object> setMembers(String key) {
        return redisTemplate.opsForSet().members(key);
    }

    public Boolean setIsMember(String key, Object value) {
        return redisTemplate.opsForSet().isMember(key, value);
    }

    public Long setRemove(String key, Object... values) {
        return redisTemplate.opsForSet().remove(key, values);
    }

    // ===== Sorted Set Operations (with scores) =====

    public Boolean sortedSetAdd(String key, Object value, double score) {
        return redisTemplate.opsForZSet().add(key, value, score);
    }

    public Set<Object> sortedSetRange(String key, long start, long end) {
        return redisTemplate.opsForZSet().range(key, start, end);
    }

    public Set<Object> sortedSetReverseRange(String key, long start, long end) {
        return redisTemplate.opsForZSet().reverseRange(key, start, end);
    }

    public Long sortedSetRank(String key, Object value) {
        return redisTemplate.opsForZSet().rank(key, value);
    }

    // ===== Counter Operations =====

    public Long increment(String key) {
        return redisTemplate.opsForValue().increment(key);
    }

    public Long incrementBy(String key, long delta) {
        return redisTemplate.opsForValue().increment(key, delta);
    }

    public Long decrement(String key) {
        return redisTemplate.opsForValue().decrement(key);
    }
}
```

### 1.6 Practical Redis Examples

**Example 1: User Session Storage**
```java
@Service
@RequiredArgsConstructor
public class SessionService {

    private final RedisTemplate<String, Object> redisTemplate;
    private static final String SESSION_PREFIX = "session:";
    private static final long SESSION_TIMEOUT = 30; // minutes

    public void createSession(String sessionId, UserSession userSession) {
        String key = SESSION_PREFIX + sessionId;
        redisTemplate.opsForValue().set(key, userSession, SESSION_TIMEOUT, TimeUnit.MINUTES);
    }

    public UserSession getSession(String sessionId) {
        String key = SESSION_PREFIX + sessionId;
        return (UserSession) redisTemplate.opsForValue().get(key);
    }

    public void extendSession(String sessionId) {
        String key = SESSION_PREFIX + sessionId;
        redisTemplate.expire(key, SESSION_TIMEOUT, TimeUnit.MINUTES);
    }

    public void deleteSession(String sessionId) {
        String key = SESSION_PREFIX + sessionId;
        redisTemplate.delete(key);
    }
}

@Data
@AllArgsConstructor
public class UserSession implements Serializable {
    private Long userId;
    private String username;
    private Set<String> roles;
    private LocalDateTime loginTime;
}
```

**Example 2: Rate Limiting (API Throttling)**
```java
@Service
@RequiredArgsConstructor
public class RateLimiterService {

    private final RedisTemplate<String, Object> redisTemplate;

    /**
     * Check if request is allowed based on rate limit
     * @param key User identifier (IP, userId, etc.)
     * @param maxRequests Maximum requests allowed
     * @param windowSeconds Time window in seconds
     * @return true if allowed, false if rate limit exceeded
     */
    public boolean isAllowed(String key, int maxRequests, int windowSeconds) {
        String rateLimitKey = "rate_limit:" + key;

        Long requests = redisTemplate.opsForValue().increment(rateLimitKey);

        if (requests == 1) {
            // First request, set expiry
            redisTemplate.expire(rateLimitKey, windowSeconds, TimeUnit.SECONDS);
        }

        return requests <= maxRequests;
    }

    /**
     * Get remaining requests for a key
     */
    public long getRemainingRequests(String key, int maxRequests) {
        String rateLimitKey = "rate_limit:" + key;
        Long requests = (Long) redisTemplate.opsForValue().get(rateLimitKey);

        if (requests == null) {
            return maxRequests;
        }

        return Math.max(0, maxRequests - requests);
    }
}

// Usage in controller
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
public class ApiController {

    private final RateLimiterService rateLimiter;

    @GetMapping("/data")
    public ResponseEntity<?> getData(HttpServletRequest request) {
        String clientIp = request.getRemoteAddr();

        // Allow 100 requests per minute
        if (!rateLimiter.isAllowed(clientIp, 100, 60)) {
            return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
                .body("Rate limit exceeded. Try again later.");
        }

        // Process request
        return ResponseEntity.ok("Your data");
    }
}
```

**Example 3: Leaderboard with Sorted Sets**
```java
@Service
@RequiredArgsConstructor
public class LeaderboardService {

    private final RedisTemplate<String, Object> redisTemplate;
    private static final String LEADERBOARD_KEY = "game:leaderboard";

    public void updateScore(String username, int score) {
        redisTemplate.opsForZSet().add(LEADERBOARD_KEY, username, score);
    }

    public void incrementScore(String username, int points) {
        redisTemplate.opsForZSet().incrementScore(LEADERBOARD_KEY, username, points);
    }

    public List<LeaderboardEntry> getTopPlayers(int count) {
        Set<ZSetOperations.TypedTuple<Object>> topScores =
            redisTemplate.opsForZSet().reverseRangeWithScores(LEADERBOARD_KEY, 0, count - 1);

        List<LeaderboardEntry> leaderboard = new ArrayList<>();
        int rank = 1;

        for (ZSetOperations.TypedTuple<Object> score : topScores) {
            leaderboard.add(new LeaderboardEntry(
                rank++,
                (String) score.getValue(),
                score.getScore().intValue()
            ));
        }

        return leaderboard;
    }

    public LeaderboardEntry getPlayerRank(String username) {
        Long rank = redisTemplate.opsForZSet().reverseRank(LEADERBOARD_KEY, username);
        Double score = redisTemplate.opsForZSet().score(LEADERBOARD_KEY, username);

        if (rank == null || score == null) {
            return null;
        }

        return new LeaderboardEntry(rank.intValue() + 1, username, score.intValue());
    }

    public long getTotalPlayers() {
        Long size = redisTemplate.opsForZSet().size(LEADERBOARD_KEY);
        return size != null ? size : 0;
    }
}

@Data
@AllArgsConstructor
public class LeaderboardEntry {
    private int rank;
    private String username;
    private int score;
}
```

**Example 4: Distributed Lock**
```java
@Service
@RequiredArgsConstructor
public class DistributedLockService {

    private final RedisTemplate<String, Object> redisTemplate;

    /**
     * Acquire lock
     * @param lockKey Lock identifier
     * @param lockValue Unique value (usually UUID)
     * @param expireTime Lock expiry in seconds
     * @return true if lock acquired
     */
    public boolean acquireLock(String lockKey, String lockValue, long expireTime) {
        Boolean success = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, lockValue, expireTime, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(success);
    }

    /**
     * Release lock
     */
    public boolean releaseLock(String lockKey, String lockValue) {
        String script =
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "    return redis.call('del', KEYS[1]) " +
            "else " +
            "    return 0 " +
            "end";

        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Collections.singletonList(lockKey),
            lockValue
        );

        return result != null && result == 1L;
    }
}

// Usage example
@Service
@RequiredArgsConstructor
public class OrderService {

    private final DistributedLockService lockService;
    private final OrderRepository orderRepository;

    public void processOrder(Long orderId) {
        String lockKey = "order:lock:" + orderId;
        String lockValue = UUID.randomUUID().toString();

        try {
            // Try to acquire lock (wait up to 10 seconds)
            if (lockService.acquireLock(lockKey, lockValue, 30)) {
                try {
                    // Process order (only one instance can do this)
                    Order order = orderRepository.findById(orderId).orElseThrow();
                    order.setStatus("PROCESSING");
                    orderRepository.save(order);

                    // Do payment processing...

                    order.setStatus("COMPLETED");
                    orderRepository.save(order);
                } finally {
                    // Always release lock
                    lockService.releaseLock(lockKey, lockValue);
                }
            } else {
                throw new RuntimeException("Could not acquire lock for order: " + orderId);
            }
        } catch (Exception e) {
            // Handle error
            throw new RuntimeException("Order processing failed", e);
        }
    }
}
```

---

## 2. Message Queues

### 2.1 RabbitMQ (Message Broker)

**Why Message Queues?**
- Asynchronous processing
- Decouple services
- Load balancing
- Guaranteed delivery
- Handle traffic spikes

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

**Configuration:**
```java
@Configuration
public class RabbitMQConfig {

    public static final String QUEUE_NAME = "order.queue";
    public static final String EXCHANGE_NAME = "order.exchange";
    public static final String ROUTING_KEY = "order.created";

    @Bean
    public Queue orderQueue() {
        return new Queue(QUEUE_NAME, true); // durable
    }

    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange(EXCHANGE_NAME);
    }

    @Bean
    public Binding binding(Queue orderQueue, TopicExchange orderExchange) {
        return BindingBuilder.bind(orderQueue)
            .to(orderExchange)
            .with(ROUTING_KEY);
    }
}
```

**Producer (Send messages):**
```java
@Service
@RequiredArgsConstructor
public class OrderPublisher {

    private final RabbitTemplate rabbitTemplate;

    public void publishOrderCreated(Order order) {
        OrderEvent event = new OrderEvent(
            order.getId(),
            order.getUserId(),
            order.getTotalAmount(),
            LocalDateTime.now()
        );

        rabbitTemplate.convertAndSend(
            RabbitMQConfig.EXCHANGE_NAME,
            RabbitMQConfig.ROUTING_KEY,
            event
        );

        log.info("Published order created event: {}", order.getId());
    }
}
```

**Consumer (Receive messages):**
```java
@Component
@RequiredArgsConstructor
@Slf4j
public class OrderConsumer {

    private final EmailService emailService;
    private final NotificationService notificationService;

    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME)
    public void handleOrderCreated(OrderEvent event) {
        log.info("Received order created event: {}", event.getOrderId());

        try {
            // Send confirmation email
            emailService.sendOrderConfirmation(event);

            // Send push notification
            notificationService.notifyUser(event.getUserId(), "Order confirmed!");

            // Update inventory
            // ... other async tasks
        } catch (Exception e) {
            log.error("Error processing order event", e);
            throw new AmqpRejectAndDontRequeueException("Processing failed", e);
        }
    }
}
```

### 2.2 Kafka (Event Streaming)

**For high-throughput, distributed systems:**

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: my-app
      auto-offset-reset: earliest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

**Producer:**
```java
@Service
@RequiredArgsConstructor
public class EventProducer {

    private final KafkaTemplate<String, Object> kafkaTemplate;

    public void sendEvent(String topic, String key, Object event) {
        kafkaTemplate.send(topic, key, event)
            .addCallback(
                result -> log.info("Sent event to Kafka: {}", event),
                error -> log.error("Failed to send event", error)
            );
    }
}
```

**Consumer:**
```java
@Component
@Slf4j
public class EventConsumer {

    @KafkaListener(topics = "user.events", groupId = "my-app")
    public void handleUserEvent(UserEvent event) {
        log.info("Received user event: {}", event);
        // Process event
    }
}
```

---

## 3. WebSockets (Real-Time Communication)

**Like Firebase Realtime Database or Socket.IO:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

**Configuration:**
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic", "/queue");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
            .setAllowedOriginPatterns("*")
            .withSockJS();
    }
}
```

**Chat Controller:**
```java
@Controller
@RequiredArgsConstructor
public class ChatController {

    private final SimpMessagingTemplate messagingTemplate;

    @MessageMapping("/chat.send")
    @SendTo("/topic/messages")
    public ChatMessage sendMessage(ChatMessage message) {
        message.setTimestamp(LocalDateTime.now());
        return message;
    }

    @MessageMapping("/chat.private")
    public void sendPrivateMessage(@Payload ChatMessage message, @Header("simpSessionId") String sessionId) {
        messagingTemplate.convertAndSendToUser(
            message.getRecipient(),
            "/queue/private",
            message
        );
    }

    // Send notification to specific user
    public void notifyUser(String username, String notification) {
        messagingTemplate.convertAndSendToUser(username, "/queue/notifications", notification);
    }
}
```

**Android Client (using STOMP):**
```kotlin
// In Android app
class ChatActivity : AppCompatActivity() {
    private lateinit var stompClient: StompClient

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Connect to WebSocket
        stompClient = Stomp.over(Stomp.ConnectionProvider.OKHTTP, "ws://your-server.com/ws")
        stompClient.connect()

        // Subscribe to messages
        stompClient.topic("/topic/messages").subscribe { message ->
            val chatMessage = Gson().fromJson(message.payload, ChatMessage::class.java)
            // Update UI
            runOnUiThread {
                addMessageToChat(chatMessage)
            }
        }
    }

    fun sendMessage(text: String) {
        val message = ChatMessage(username, text)
        stompClient.send("/app/chat.send", Gson().toJson(message)).subscribe()
    }
}
```

---

## 4. Email Sending

### 4.1 JavaMail with Gmail

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

```yaml
spring:
  mail:
    host: smtp.gmail.com
    port: 587
    username: ${EMAIL_USERNAME}
    password: ${EMAIL_PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
```

**Email Service:**
```java
@Service
@RequiredArgsConstructor
public class EmailService {

    private final JavaMailSender mailSender;

    public void sendSimpleEmail(String to, String subject, String text) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(text);
        message.setFrom("noreply@yourapp.com");

        mailSender.send(message);
    }

    public void sendHtmlEmail(String to, String subject, String htmlContent) {
        try {
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");

            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(htmlContent, true); // true = HTML
            helper.setFrom("noreply@yourapp.com");

            mailSender.send(message);
        } catch (MessagingException e) {
            throw new RuntimeException("Failed to send email", e);
        }
    }

    public void sendEmailWithAttachment(String to, String subject, String text, File attachment) {
        try {
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);

            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(text);
            helper.addAttachment(attachment.getName(), attachment);

            mailSender.send(message);
        } catch (MessagingException e) {
            throw new RuntimeException("Failed to send email", e);
        }
    }
}
```

**Email Template Example:**
```java
@Service
@RequiredArgsConstructor
public class OrderEmailService {

    private final EmailService emailService;
    private final TemplateEngine templateEngine;

    public void sendOrderConfirmation(Order order) {
        Context context = new Context();
        context.setVariable("order", order);
        context.setVariable("customer", order.getCustomer());
        context.setVariable("items", order.getItems());

        String htmlContent = templateEngine.process("email/order-confirmation", context);

        emailService.sendHtmlEmail(
            order.getCustomer().getEmail(),
            "Order Confirmation #" + order.getId(),
            htmlContent
        );
    }
}
```

---

## 5. File Storage

### 5.1 Amazon S3

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.20.0</version>
</dependency>
```

**S3 Service:**
```java
@Service
public class S3Service {

    private final S3Client s3Client;
    private final String bucketName;

    public S3Service(@Value("${aws.s3.bucket}") String bucketName) {
        this.bucketName = bucketName;
        this.s3Client = S3Client.builder()
            .region(Region.US_EAST_1)
            .build();
    }

    public String uploadFile(MultipartFile file) {
        try {
            String fileName = UUID.randomUUID() + "-" + file.getOriginalFilename();

            PutObjectRequest request = PutObjectRequest.builder()
                .bucket(bucketName)
                .key(fileName)
                .contentType(file.getContentType())
                .build();

            s3Client.putObject(request, RequestBody.fromBytes(file.getBytes()));

            return getFileUrl(fileName);
        } catch (IOException e) {
            throw new RuntimeException("Failed to upload file", e);
        }
    }

    public byte[] downloadFile(String fileName) {
        GetObjectRequest request = GetObjectRequest.builder()
            .bucket(bucketName)
            .key(fileName)
            .build();

        ResponseBytes<GetObjectResponse> responseBytes = s3Client.getObjectAsBytes(request);
        return responseBytes.asByteArray();
    }

    public void deleteFile(String fileName) {
        DeleteObjectRequest request = DeleteObjectRequest.builder()
            .bucket(bucketName)
            .key(fileName)
            .build();

        s3Client.deleteObject(request);
    }

    public String getFileUrl(String fileName) {
        return String.format("https://%s.s3.amazonaws.com/%s", bucketName, fileName);
    }

    public String generatePresignedUrl(String fileName, Duration expiration) {
        GetObjectRequest getObjectRequest = GetObjectRequest.builder()
            .bucket(bucketName)
            .key(fileName)
            .build();

        S3Presigner presigner = S3Presigner.builder()
            .region(Region.US_EAST_1)
            .build();

        PresignedGetObjectRequest presignedRequest = presigner.presignGetObject(
            GetObjectPresignRequest.builder()
                .getObjectRequest(getObjectRequest)
                .signatureDuration(expiration)
                .build()
        );

        return presignedRequest.url().toString();
    }
}
```

### 5.2 Local File Storage

```java
@Service
public class LocalFileStorageService {

    @Value("${file.upload.dir}")
    private String uploadDir;

    @PostConstruct
    public void init() {
        try {
            Files.createDirectories(Paths.get(uploadDir));
        } catch (IOException e) {
            throw new RuntimeException("Could not create upload directory", e);
        }
    }

    public String storeFile(MultipartFile file) {
        try {
            String fileName = UUID.randomUUID() + "-" + file.getOriginalFilename();
            Path filePath = Paths.get(uploadDir, fileName);
            Files.copy(file.getInputStream(), filePath);
            return fileName;
        } catch (IOException e) {
            throw new RuntimeException("Failed to store file", e);
        }
    }

    public Resource loadFile(String fileName) {
        try {
            Path filePath = Paths.get(uploadDir, fileName);
            Resource resource = new UrlResource(filePath.toUri());

            if (resource.exists() && resource.isReadable()) {
                return resource;
            } else {
                throw new RuntimeException("File not found: " + fileName);
            }
        } catch (MalformedURLException e) {
            throw new RuntimeException("File not found: " + fileName, e);
        }
    }

    public void deleteFile(String fileName) {
        try {
            Path filePath = Paths.get(uploadDir, fileName);
            Files.deleteIfExists(filePath);
        } catch (IOException e) {
            throw new RuntimeException("Failed to delete file", e);
        }
    }
}
```

---

## 6. Background Job Processing

### 6.1 @Scheduled Tasks

```java
@Component
@EnableScheduling
@Slf4j
public class ScheduledTasks {

    @Autowired
    private OrderService orderService;

    // Fixed rate - every 5 seconds
    @Scheduled(fixedRate = 5000)
    public void reportStatus() {
        log.info("Current time: {}", LocalDateTime.now());
    }

    // Fixed delay - 5 seconds after previous execution completes
    @Scheduled(fixedDelay = 5000)
    public void processQueue() {
        log.info("Processing queue...");
    }

    // Cron expression - Every day at 2 AM
    @Scheduled(cron = "0 0 2 * * ?")
    public void performDailyCleanup() {
        log.info("Running daily cleanup");
        orderService.cleanupOldOrders();
    }

    // Cron - Every Monday at 9 AM
    @Scheduled(cron = "0 0 9 * * MON")
    public void sendWeeklyReport() {
        log.info("Generating weekly report");
    }

    // Cron - Every 30 minutes
    @Scheduled(cron = "0 */30 * * * ?")
    public void syncData() {
        log.info("Syncing data");
    }
}
```

### 6.2 Async Execution

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {

    @Async
    public CompletableFuture<Void> sendNotificationAsync(String userId, String message) {
        // Long-running task
        log.info("Sending notification to {}", userId);
        // ... send notification
        return CompletableFuture.completedFuture(null);
    }

    @Async
    public void processInBackground(Order order) {
        // Fire and forget
        log.info("Processing order {} in background", order.getId());
    }
}
```

---

## 7. Full-Text Search with Elasticsearch

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

```java
@Document(indexName = "products")
@Data
public class ProductDocument {

    @Id
    private String id;

    @Field(type = FieldType.Text, analyzer = "standard")
    private String name;

    @Field(type = FieldType.Text, analyzer = "standard")
    private String description;

    @Field(type = FieldType.Keyword)
    private String category;

    @Field(type = FieldType.Double)
    private Double price;

    @Field(type = FieldType.Date)
    private LocalDateTime createdAt;
}

public interface ProductSearchRepository extends ElasticsearchRepository<ProductDocument, String> {

    List<ProductDocument> findByNameContaining(String name);

    List<ProductDocument> findByCategory(String category);

    @Query("{\"bool\": {\"must\": [{\"match\": {\"name\": \"?0\"}}], \"filter\": [{\"range\": {\"price\": {\"gte\": ?1, \"lte\": ?2}}}]}}")
    List<ProductDocument> findByNameAndPriceRange(String name, Double minPrice, Double maxPrice);
}

@Service
@RequiredArgsConstructor
public class ProductSearchService {

    private final ProductSearchRepository searchRepository;

    public List<ProductDocument> searchProducts(String query) {
        return searchRepository.findByNameContaining(query);
    }

    public void indexProduct(Product product) {
        ProductDocument doc = new ProductDocument();
        doc.setId(product.getId().toString());
        doc.setName(product.getName());
        doc.setDescription(product.getDescription());
        doc.setCategory(product.getCategory());
        doc.setPrice(product.getPrice());
        doc.setCreatedAt(LocalDateTime.now());

        searchRepository.save(doc);
    }
}
```

---

## 🎯 Complete Backend Example

**E-commerce Order Processing System:**

```java
@Service
@RequiredArgsConstructor
@Transactional
public class CompleteOrderService {

    private final OrderRepository orderRepository;
    private final RedisService redisService;
    private final OrderPublisher orderPublisher;
    private final EmailService emailService;
    private final NotificationService notificationService;
    private final S3Service s3Service;

    public Order createOrder(CreateOrderRequest request) {
        // 1. Create order in database
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setItems(request.getItems());
        order.setTotalAmount(calculateTotal(request.getItems()));
        order.setStatus("PENDING");
        order = orderRepository.save(order);

        // 2. Cache order for quick access
        redisService.setValueWithExpiry(
            "order:" + order.getId(),
            order,
            30,
            TimeUnit.MINUTES
        );

        // 3. Publish event to message queue for async processing
        orderPublisher.publishOrderCreated(order);

        // 4. Send confirmation email asynchronously
        emailService.sendOrderConfirmation(order);

        // 5. Send push notification
        notificationService.notifyUser(
            order.getUserId(),
            "Your order #" + order.getId() + " has been confirmed!"
        );

        // 6. Update inventory (async)
        updateInventoryAsync(order.getItems());

        return order;
    }

    public Order getOrder(Long orderId) {
        // Try cache first
        String cacheKey = "order:" + orderId;
        Order cachedOrder = (Order) redisService.getValue(cacheKey);

        if (cachedOrder != null) {
            return cachedOrder;
        }

        // Fallback to database
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));

        // Update cache
        redisService.setValueWithExpiry(cacheKey, order, 30, TimeUnit.MINUTES);

        return order;
    }

    @Async
    public void updateInventoryAsync(List<OrderItem> items) {
        // Update inventory in background
        log.info("Updating inventory for {} items", items.size());
    }

    // Called by scheduled job
    @Scheduled(cron = "0 */5 * * * ?") // Every 5 minutes
    public void processPayments() {
        List<Order> pendingOrders = orderRepository.findByStatus("PENDING");

        for (Order order : pendingOrders) {
            try {
                // Process payment
                boolean paymentSuccess = processPayment(order);

                if (paymentSuccess) {
                    order.setStatus("PAID");
                    orderRepository.save(order);

                    // Update cache
                    redisService.setValue("order:" + order.getId(), order);

                    // Notify warehouse
                    orderPublisher.publishOrderPaid(order);
                }
            } catch (Exception e) {
                log.error("Failed to process payment for order: {}", order.getId(), e);
            }
        }
    }
}
```

---

## 🎯 Key Takeaways

1. **Redis** - Fast in-memory cache, session storage, rate limiting
2. **Message Queues** - Async processing, decouple services (RabbitMQ, Kafka)
3. **WebSockets** - Real-time bidirectional communication
4. **Email** - Transactional emails with templates
5. **File Storage** - S3 for production, local for development
6. **Scheduled Jobs** - Background tasks with @Scheduled
7. **Elasticsearch** - Full-text search at scale
8. **Async Processing** - Non-blocking operations with @Async

## 🚀 Next Steps

You now have the tools to build production-ready backend systems! Continue refining your knowledge with the other guides in this series.

Return to [Spring Java Guide Overview](./README.md)
