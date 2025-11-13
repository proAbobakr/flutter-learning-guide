# Apache Kafka: A Complete Guide from Scratch

## 🎯 What is Apache Kafka?

Apache Kafka is a **distributed event streaming platform** designed for high-throughput, fault-tolerant, real-time data streaming. Think of it as a super-powered message queue that can handle millions of messages per second.

### Simple Analogy
Imagine Kafka as a **sophisticated postal service**:
- **Publishers (Producers)** send letters (messages) to different mailbox categories (topics)
- **Post offices (Brokers)** store and organize these letters
- **Subscribers (Consumers)** pick up letters from the mailboxes they're interested in
- Letters are kept for a certain period, so multiple subscribers can read the same letter

### Key Characteristics
- ✅ **Distributed**: Runs on a cluster of machines for scalability
- ✅ **Fault-tolerant**: Data is replicated across multiple nodes
- ✅ **High-throughput**: Handles millions of messages per second
- ✅ **Persistent**: Messages are stored on disk, not just in memory
- ✅ **Real-time**: Low latency (milliseconds)

---

## 🏗️ Core Concepts

### 1. **Producer**
A producer is an application that **publishes messages** to Kafka topics.

```java
// Kotlin Producer Example
import org.apache.kafka.clients.producer.KafkaProducer
import org.apache.kafka.clients.producer.ProducerConfig
import org.apache.kafka.clients.producer.ProducerRecord
import java.util.Properties

class KafkaMessageProducer {
    private val producer: KafkaProducer<String, String>

    init {
        val props = Properties().apply {
            put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092")
            put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer")
            put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer")
            put(ProducerConfig.ACKS_CONFIG, "all") // Wait for all replicas
        }
        producer = KafkaProducer(props)
    }

    fun sendMessage(topic: String, key: String, message: String) {
        val record = ProducerRecord(topic, key, message)

        // Asynchronous send
        producer.send(record) { metadata, exception ->
            if (exception != null) {
                println("Error sending message: ${exception.message}")
            } else {
                println("Message sent successfully!")
                println("Topic: ${metadata.topic()}")
                println("Partition: ${metadata.partition()}")
                println("Offset: ${metadata.offset()}")
            }
        }
    }

    fun close() {
        producer.close()
    }
}

// Usage
fun main() {
    val producer = KafkaMessageProducer()

    // Send a user registration event
    producer.sendMessage(
        topic = "user-events",
        key = "user-123",
        message = """{"userId":"123","event":"registration","timestamp":1234567890}"""
    )

    producer.close()
}
```

### 2. **Consumer**
A consumer is an application that **subscribes to topics** and processes messages.

```kotlin
// Kotlin Consumer Example
import org.apache.kafka.clients.consumer.KafkaConsumer
import org.apache.kafka.clients.consumer.ConsumerConfig
import java.time.Duration
import java.util.Properties

class KafkaMessageConsumer {
    private val consumer: KafkaConsumer<String, String>

    init {
        val props = Properties().apply {
            put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092")
            put(ConsumerConfig.GROUP_ID_CONFIG, "my-consumer-group")
            put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer")
            put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer")
            put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest")
            put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true")
        }
        consumer = KafkaConsumer(props)
    }

    fun subscribe(topics: List<String>) {
        consumer.subscribe(topics)
    }

    fun consumeMessages() {
        try {
            while (true) {
                val records = consumer.poll(Duration.ofMillis(100))

                for (record in records) {
                    println("Received message:")
                    println("  Topic: ${record.topic()}")
                    println("  Partition: ${record.partition()}")
                    println("  Offset: ${record.offset()}")
                    println("  Key: ${record.key()}")
                    println("  Value: ${record.value()}")
                    println("  Timestamp: ${record.timestamp()}")

                    // Process the message
                    processMessage(record.value())
                }
            }
        } catch (e: Exception) {
            println("Error consuming messages: ${e.message}")
        } finally {
            consumer.close()
        }
    }

    private fun processMessage(message: String) {
        // Your business logic here
        println("Processing: $message")
    }
}

// Usage
fun main() {
    val consumer = KafkaMessageConsumer()
    consumer.subscribe(listOf("user-events"))
    consumer.consumeMessages()
}
```

### 3. **Topic**
A **topic** is a category or feed name to which records are published. Like a folder for specific types of messages.

```
Topic: "user-events"
├── Partition 0: [msg1, msg2, msg3, ...]
├── Partition 1: [msg4, msg5, msg6, ...]
└── Partition 2: [msg7, msg8, msg9, ...]
```

**Creating a Topic:**
```bash
# Command line
kafka-topics.sh --create \
  --topic user-events \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 2

# List all topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# Describe a topic
kafka-topics.sh --describe --topic user-events --bootstrap-server localhost:9092
```

### 4. **Partition**
Topics are divided into **partitions** for parallelism and scalability. Each partition is an ordered, immutable sequence of messages.

**Key Points:**
- Messages within a partition are **ordered** (FIFO)
- Messages across partitions are **not ordered**
- Each message in a partition gets a unique **offset** (sequential ID)
- More partitions = more parallelism = higher throughput

```
Topic: "orders"
├── Partition 0: [order1(offset:0), order4(offset:1), order7(offset:2)]
├── Partition 1: [order2(offset:0), order5(offset:1), order8(offset:2)]
└── Partition 2: [order3(offset:0), order6(offset:1), order9(offset:2)]
```

### 5. **Broker**
A Kafka **broker** is a server that stores and serves messages. A Kafka cluster consists of multiple brokers for fault tolerance.

```
Kafka Cluster
├── Broker 1 (Leader for Partition 0)
├── Broker 2 (Leader for Partition 1)
└── Broker 3 (Leader for Partition 2)
```

### 6. **Consumer Group**
Consumers work together in a **consumer group** to process messages from topics in parallel.

```
Topic "orders" (3 partitions)
│
└─► Consumer Group "order-processors"
    ├── Consumer A → reads from Partition 0
    ├── Consumer B → reads from Partition 1
    └── Consumer C → reads from Partition 2
```

**Key Rules:**
- Each partition is consumed by **exactly one consumer** in a group
- Multiple consumer groups can read the **same topic** independently

---

## 🔄 How Kafka Works

### Message Flow

```
┌──────────┐         ┌──────────────┐         ┌──────────┐
│ Producer │ ──────> │ Kafka Broker │ ──────> │ Consumer │
└──────────┘         │  (Topic)     │         └──────────┘
                     │  Partition 0 │
                     │  Partition 1 │
                     │  Partition 2 │
                     └──────────────┘
```

### Step-by-Step Process

1. **Producer sends message**
   - Serializes data (String, JSON, Avro, etc.)
   - Optionally specifies a key (for partitioning)
   - Kafka determines which partition to send to

2. **Partitioning Strategy**
   - **With Key**: `hash(key) % num_partitions` → Same key always goes to same partition
   - **Without Key**: Round-robin or sticky partitioning
   - **Custom**: You can implement your own partitioner

3. **Broker stores message**
   - Appends message to the partition log
   - Replicates to other brokers (if replication factor > 1)
   - Returns acknowledgment to producer

4. **Consumer reads messages**
   - Polls for new messages
   - Processes messages
   - Commits offset (marks message as processed)

---

## 💡 Practical Examples

### Example 1: E-Commerce Order Processing

```kotlin
// Order Event Model
data class OrderEvent(
    val orderId: String,
    val userId: String,
    val items: List<Item>,
    val totalAmount: Double,
    val timestamp: Long,
    val status: OrderStatus
)

enum class OrderStatus {
    CREATED, PAID, SHIPPED, DELIVERED, CANCELLED
}

data class Item(val productId: String, val quantity: Int, val price: Double)

// Order Service (Producer)
class OrderService(private val producer: KafkaMessageProducer) {

    fun createOrder(order: OrderEvent) {
        // Save to database
        saveOrderToDatabase(order)

        // Publish event to Kafka
        val orderJson = Json.encodeToString(order)
        producer.sendMessage(
            topic = "orders",
            key = order.orderId,
            message = orderJson
        )

        println("Order created and event published: ${order.orderId}")
    }

    private fun saveOrderToDatabase(order: OrderEvent) {
        // Database logic
    }
}

// Inventory Service (Consumer)
class InventoryService(private val consumer: KafkaMessageConsumer) {

    fun start() {
        consumer.subscribe(listOf("orders"))

        while (true) {
            val records = consumer.poll(Duration.ofMillis(100))

            for (record in records) {
                val order = Json.decodeFromString<OrderEvent>(record.value())

                when (order.status) {
                    OrderStatus.CREATED -> reserveInventory(order)
                    OrderStatus.CANCELLED -> releaseInventory(order)
                    else -> println("Status ${order.status} - no action needed")
                }
            }
        }
    }

    private fun reserveInventory(order: OrderEvent) {
        println("Reserving inventory for order: ${order.orderId}")
        order.items.forEach { item ->
            println("  - Product ${item.productId}: ${item.quantity} units")
        }
    }

    private fun releaseInventory(order: OrderEvent) {
        println("Releasing inventory for order: ${order.orderId}")
    }
}

// Email Service (Consumer)
class EmailService(private val consumer: KafkaMessageConsumer) {

    fun start() {
        consumer.subscribe(listOf("orders"))

        while (true) {
            val records = consumer.poll(Duration.ofMillis(100))

            for (record in records) {
                val order = Json.decodeFromString<OrderEvent>(record.value())
                sendEmail(order)
            }
        }
    }

    private fun sendEmail(order: OrderEvent) {
        when (order.status) {
            OrderStatus.CREATED -> println("Sending confirmation email for ${order.orderId}")
            OrderStatus.SHIPPED -> println("Sending shipping notification for ${order.orderId}")
            OrderStatus.DELIVERED -> println("Sending delivery confirmation for ${order.orderId}")
            else -> {}
        }
    }
}
```

### Example 2: Real-Time Analytics Pipeline

```kotlin
// Clickstream Event
data class ClickEvent(
    val userId: String,
    val pageUrl: String,
    val action: String,
    val timestamp: Long,
    val sessionId: String
)

// Click Tracker (Producer)
class ClickTracker {
    private val producer = KafkaMessageProducer()

    fun trackClick(userId: String, pageUrl: String, action: String) {
        val event = ClickEvent(
            userId = userId,
            pageUrl = pageUrl,
            action = action,
            timestamp = System.currentTimeMillis(),
            sessionId = generateSessionId(userId)
        )

        producer.sendMessage(
            topic = "clickstream",
            key = userId,
            message = Json.encodeToString(event)
        )
    }

    private fun generateSessionId(userId: String): String {
        // Session logic
        return "$userId-${System.currentTimeMillis() / 1000}"
    }
}

// Analytics Processor (Consumer with Aggregation)
class AnalyticsProcessor {
    private val consumer = KafkaMessageConsumer()
    private val pageViews = mutableMapOf<String, Int>()

    fun start() {
        consumer.subscribe(listOf("clickstream"))

        while (true) {
            val records = consumer.poll(Duration.ofMillis(100))

            for (record in records) {
                val event = Json.decodeFromString<ClickEvent>(record.value())
                processEvent(event)
            }

            // Print statistics every 1000 messages
            if (records.count() > 0) {
                printTopPages()
            }
        }
    }

    private fun processEvent(event: ClickEvent) {
        // Count page views
        pageViews[event.pageUrl] = (pageViews[event.pageUrl] ?: 0) + 1

        // Could also:
        // - Store in database
        // - Update real-time dashboard
        // - Trigger alerts
        // - Feed ML models
    }

    private fun printTopPages() {
        println("\n=== Top Pages ===")
        pageViews.entries
            .sortedByDescending { it.value }
            .take(5)
            .forEach { (page, count) ->
                println("$page: $count views")
            }
    }
}
```

### Example 3: Microservices Communication

```kotlin
// Event-Driven Architecture Example

// User Service publishes events
class UserService {
    private val producer = KafkaMessageProducer()

    fun registerUser(email: String, name: String): String {
        val userId = generateUserId()

        // Save user to database
        saveUser(userId, email, name)

        // Publish event
        val event = mapOf(
            "eventType" to "USER_REGISTERED",
            "userId" to userId,
            "email" to email,
            "name" to name,
            "timestamp" to System.currentTimeMillis()
        )

        producer.sendMessage(
            topic = "user-events",
            key = userId,
            message = Json.encodeToString(event)
        )

        return userId
    }

    private fun generateUserId() = "user-${System.currentTimeMillis()}"
    private fun saveUser(userId: String, email: String, name: String) { /* DB logic */ }
}

// Multiple services react to the same event

// Notification Service
class NotificationService {
    fun start() {
        val consumer = KafkaMessageConsumer()
        consumer.subscribe(listOf("user-events"))

        while (true) {
            val records = consumer.poll(Duration.ofMillis(100))
            for (record in records) {
                val event = Json.decodeFromString<Map<String, Any>>(record.value())
                if (event["eventType"] == "USER_REGISTERED") {
                    sendWelcomeEmail(event["email"] as String)
                }
            }
        }
    }

    private fun sendWelcomeEmail(email: String) {
        println("Sending welcome email to $email")
    }
}

// Analytics Service
class AnalyticsService {
    fun start() {
        val consumer = KafkaMessageConsumer()
        consumer.subscribe(listOf("user-events"))

        while (true) {
            val records = consumer.poll(Duration.ofMillis(100))
            for (record in records) {
                val event = Json.decodeFromString<Map<String, Any>>(record.value())
                if (event["eventType"] == "USER_REGISTERED") {
                    trackRegistration(event["userId"] as String)
                }
            }
        }
    }

    private fun trackRegistration(userId: String) {
        println("Tracking registration for analytics: $userId")
    }
}

// Recommendation Service
class RecommendationService {
    fun start() {
        val consumer = KafkaMessageConsumer()
        consumer.subscribe(listOf("user-events"))

        while (true) {
            val records = consumer.poll(Duration.ofMillis(100))
            for (record in records) {
                val event = Json.decodeFromString<Map<String, Any>>(record.value())
                if (event["eventType"] == "USER_REGISTERED") {
                    initializeUserProfile(event["userId"] as String)
                }
            }
        }
    }

    private fun initializeUserProfile(userId: String) {
        println("Creating recommendation profile for $userId")
    }
}
```

---

## 🎯 Common Use Cases

### 1. **Activity Tracking**
- User clicks, page views, searches
- Real-time analytics dashboards
- Example: LinkedIn, Facebook, Twitter

### 2. **Log Aggregation**
- Collecting logs from multiple services
- Centralized logging system
- Example: ELK Stack (Elasticsearch, Logstash, Kafka, Kibana)

### 3. **Stream Processing**
- Real-time data transformations
- Fraud detection
- Example: Credit card transactions, payment processing

### 4. **Event Sourcing**
- Store all changes as events
- Rebuild application state from events
- Example: Banking systems, audit logs

### 5. **Microservices Communication**
- Asynchronous service communication
- Decoupling services
- Example: E-commerce order processing

### 6. **Metrics & Monitoring**
- Application metrics collection
- Infrastructure monitoring
- Example: Datadog, New Relic

### 7. **Change Data Capture (CDC)**
- Track database changes
- Sync data between systems
- Example: Debezium with MySQL/PostgreSQL

---

## ⚖️ Kafka vs Other Messaging Systems

| Feature | Kafka | RabbitMQ | AWS SQS | Redis Pub/Sub |
|---------|-------|----------|---------|---------------|
| **Type** | Distributed log | Message broker | Queue service | In-memory pub/sub |
| **Throughput** | Very High (millions/sec) | High | Medium | Very High |
| **Persistence** | Disk (configurable retention) | Optional | Yes (14 days max) | No (in-memory only) |
| **Message Ordering** | Per partition | Per queue | FIFO queues only | No |
| **Replay Messages** | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Consumer Groups** | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Scalability** | Horizontal | Vertical + Horizontal | Managed | Horizontal |
| **Use Case** | Event streaming, logs | Task queues, RPC | Simple queuing | Caching, real-time |

---

## 🚀 Setting Up Kafka

### Docker Compose Setup (Easiest)

```yaml
# docker-compose.yml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
```

```bash
# Start Kafka
docker-compose up -d

# Check if running
docker-compose ps

# View logs
docker-compose logs -f kafka
```

### Gradle Dependencies (Kotlin/Android)

```kotlin
// build.gradle.kts
dependencies {
    implementation("org.apache.kafka:kafka-clients:3.6.0")

    // For JSON serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")

    // Coroutines support
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
}
```

---

## 🛠️ Best Practices

### 1. **Producer Best Practices**

```kotlin
// ✅ Good: Configure appropriate acknowledgments
properties.put(ProducerConfig.ACKS_CONFIG, "all") // Wait for all replicas

// ✅ Good: Enable idempotence to prevent duplicates
properties.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true")

// ✅ Good: Set appropriate batch size for throughput
properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384)
properties.put(ProducerConfig.LINGER_MS_CONFIG, 10) // Wait 10ms to batch

// ✅ Good: Handle failures with retries
properties.put(ProducerConfig.RETRIES_CONFIG, 3)

// ✅ Good: Compress messages
properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy")
```

### 2. **Consumer Best Practices**

```kotlin
// ✅ Good: Use meaningful consumer group IDs
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "order-processing-service-v1")

// ✅ Good: Control commit behavior
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false") // Manual commits

// Process and commit
consumer.poll(Duration.ofMillis(100)).forEach { record ->
    processMessage(record)
    consumer.commitSync() // Commit after processing
}

// ✅ Good: Set appropriate fetch sizes
properties.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500)
properties.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, 1024)

// ✅ Good: Handle rebalancing gracefully
consumer.subscribe(topics, object : ConsumerRebalanceListener {
    override fun onPartitionsRevoked(partitions: Collection<TopicPartition>) {
        println("Partitions revoked: $partitions")
        // Commit offsets or clean up
    }

    override fun onPartitionsAssigned(partitions: Collection<TopicPartition>) {
        println("Partitions assigned: $partitions")
        // Initialize resources
    }
})
```

### 3. **Message Key Selection**

```kotlin
// ✅ Good: Use keys for related events to maintain order
producer.send(ProducerRecord("orders",
    userId,  // Key: All events for same user go to same partition
    orderJson))

// ❌ Bad: No key means random partition assignment
producer.send(ProducerRecord("orders", orderJson))
```

### 4. **Error Handling**

```kotlin
// ✅ Good: Implement retry logic with backoff
class ResilientKafkaProducer {
    fun sendWithRetry(topic: String, key: String, message: String, maxRetries: Int = 3) {
        var attempt = 0
        var lastException: Exception? = null

        while (attempt < maxRetries) {
            try {
                producer.send(ProducerRecord(topic, key, message)).get()
                return // Success
            } catch (e: Exception) {
                lastException = e
                attempt++
                Thread.sleep((2.0.pow(attempt) * 1000).toLong()) // Exponential backoff
            }
        }

        // Failed after retries - handle appropriately
        handleFailedMessage(topic, key, message, lastException)
    }

    private fun handleFailedMessage(topic: String, key: String,
                                   message: String, error: Exception?) {
        // Log to dead letter queue, alerting system, etc.
        println("Failed to send message after retries: ${error?.message}")
    }
}
```

### 5. **Monitoring & Observability**

```kotlin
// ✅ Good: Add metrics and logging
class MonitoredKafkaProducer {
    private var messagesSent = 0
    private var messagesFailed = 0

    fun sendMessage(topic: String, key: String, message: String) {
        val startTime = System.currentTimeMillis()

        producer.send(ProducerRecord(topic, key, message)) { metadata, exception ->
            val duration = System.currentTimeMillis() - startTime

            if (exception != null) {
                messagesFailed++
                logger.error("Failed to send message", exception)
                metrics.recordFailure(topic, duration)
            } else {
                messagesSent++
                logger.info("Message sent to ${metadata.topic()}:${metadata.partition()}")
                metrics.recordSuccess(topic, duration)
            }
        }
    }

    fun getStats(): String {
        return "Sent: $messagesSent, Failed: $messagesFailed"
    }
}
```

---

## 🎓 Key Takeaways

1. **Kafka is a distributed log**, not just a message queue
2. **Partitions enable parallelism** - more partitions = more throughput
3. **Consumer groups allow scaling** - add more consumers to process faster
4. **Messages are persistent** - can be replayed and reprocessed
5. **Order is guaranteed per partition**, not across partitions
6. **Use keys wisely** - they determine partitioning and ordering
7. **Kafka is pull-based** - consumers pull messages at their own pace
8. **Horizontal scalability** - add more brokers and partitions as needed

---

## 📚 Further Learning

### Official Resources
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent Kafka Tutorials](https://docs.confluent.io/kafka-tutorials/kafka-tutorial/content/overview.html)
- [Kafka: The Definitive Guide (Book)](https://www.confluent.io/resources/kafka-the-definitive-guide/)

### Kafka Ecosystem Tools
- **Kafka Streams**: Stream processing library
- **Kafka Connect**: Integration with external systems (databases, S3, etc.)
- **Schema Registry**: Manage Avro/JSON schemas
- **KSQL**: SQL queries on Kafka streams
- **Kafka Manager/Kafdrop**: Web UI for managing Kafka clusters

### Practice Projects
1. Build a real-time chat application
2. Create a log aggregation system
3. Implement event sourcing for a banking app
4. Build a real-time analytics dashboard
5. Create a microservices architecture with Kafka

---

**Ready to dive deeper?** Start with the [official Kafka quickstart](https://kafka.apache.org/quickstart) and build your first producer-consumer application!
