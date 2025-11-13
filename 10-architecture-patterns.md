# Architecture Patterns: Microservices vs Monolith

## 🎯 Overview

Understanding backend architecture patterns is crucial for Flutter developers, as the way your backend is structured directly impacts how you design your mobile app's network layer, state management, and data synchronization strategies.

## 📚 Table of Contents

1. [Monolithic Architecture](#monolithic-architecture)
2. [Microservices Architecture](#microservices-architecture)
3. [Detailed Comparison](#detailed-comparison)
4. [When to Use Each](#when-to-use-each)
5. [Impact on Flutter Apps](#impact-on-flutter-apps)
6. [Hybrid Approaches](#hybrid-approaches)

---

## Monolithic Architecture

### What is a Monolith?

A **monolithic architecture** is a traditional software design pattern where all components of an application are built, deployed, and run as a single, unified unit.

### Structure

```
┌─────────────────────────────────────┐
│      Monolithic Application         │
│                                     │
│  ┌──────────────────────────────┐  │
│  │   Presentation Layer (UI)    │  │
│  └──────────────────────────────┘  │
│  ┌──────────────────────────────┐  │
│  │   Business Logic Layer       │  │
│  │  • User Management           │  │
│  │  • Product Catalog           │  │
│  │  • Order Processing          │  │
│  │  • Payment Processing        │  │
│  │  • Inventory Management      │  │
│  └──────────────────────────────┘  │
│  ┌──────────────────────────────┐  │
│  │   Data Access Layer          │  │
│  └──────────────────────────────┘  │
│  ┌──────────────────────────────┐  │
│  │   Single Database            │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
```

### Characteristics

- **Single Codebase**: All code lives in one repository
- **Single Deployment Unit**: Deploy everything together
- **Shared Database**: One database for all features
- **Tight Coupling**: Components are interdependent
- **Single Technology Stack**: Usually one programming language

### Example: E-commerce Monolith

```kotlin
// Android/Backend analogy - Monolithic Spring Boot application
@RestController
class MonolithicECommerceController {

    // All features in one application
    @GetMapping("/api/users/{id}")
    fun getUser(@PathVariable id: Long): User {
        return userService.findById(id)
    }

    @GetMapping("/api/products")
    fun getProducts(): List<Product> {
        return productService.getAllProducts()
    }

    @PostMapping("/api/orders")
    fun createOrder(@RequestBody order: Order): Order {
        return orderService.createOrder(order)
    }

    @PostMapping("/api/payments")
    fun processPayment(@RequestBody payment: Payment): PaymentResult {
        return paymentService.process(payment)
    }

    @GetMapping("/api/inventory/{productId}")
    fun checkInventory(@PathVariable productId: Long): Inventory {
        return inventoryService.check(productId)
    }
}

// All services in same codebase
class OrderService {
    // Can directly call other services
    fun createOrder(order: Order): Order {
        val product = productService.getProduct(order.productId)
        val inventory = inventoryService.checkStock(order.productId)
        val payment = paymentService.charge(order.amount)
        // All in same transaction
        return orderRepository.save(order)
    }
}
```

### Advantages

✅ **Simple to Develop**: Start quickly with straightforward development
✅ **Easy to Test**: Can test entire application as one unit
✅ **Simple Deployment**: One artifact to deploy
✅ **Easy Debugging**: All code in one place, easy to trace
✅ **No Network Latency**: In-process calls are fast
✅ **Strong Consistency**: Single database transactions (ACID)
✅ **Easier Data Joins**: Can join across all tables

### Disadvantages

❌ **Scalability**: Must scale entire application even if only one feature needs it
❌ **Technology Lock-in**: Hard to change technology stack
❌ **Large Codebase**: Can become unwieldy over time
❌ **Longer Build Times**: Entire app must be rebuilt
❌ **Risk of Failure**: One bug can bring down entire system
❌ **Team Coordination**: Multiple teams working on same codebase causes conflicts
❌ **Difficult Updates**: Must redeploy everything for small changes

---

## Microservices Architecture

### What are Microservices? (From Scratch)

**Microservices** is an architectural style where an application is built as a collection of small, independent services that:

- Run in their own processes
- Communicate via lightweight protocols (HTTP/REST, gRPC, message queues)
- Are independently deployable
- Are organized around business capabilities
- Can use different technologies

Think of it like building with LEGO blocks instead of carving from a single block of wood.

### Structure

```
┌─────────────────────────────────────────────────────────────┐
│                     API Gateway                             │
│            (Single entry point for clients)                 │
└─────────────────────────────────────────────────────────────┘
           │         │         │         │         │
           ▼         ▼         ▼         ▼         ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
    │  User    │ │ Product  │ │  Order   │ │ Payment  │ │Inventory │
    │ Service  │ │ Service  │ │ Service  │ │ Service  │ │ Service  │
    ├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤
    │ Node.js  │ │ Kotlin   │ │ Python   │ │  Go      │ │  Java    │
    ├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤
    │ MongoDB  │ │PostgreSQL│ │PostgreSQL│ │PostgreSQL│ │  MySQL   │
    └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
```

### Core Principles

1. **Single Responsibility**: Each service does one thing well
2. **Independence**: Services can be developed, deployed, and scaled independently
3. **Decentralization**: No central governing body for technology choices
4. **Failure Isolation**: One service failure doesn't crash the entire system
5. **Business-Driven**: Services align with business domains (Domain-Driven Design)

### Example: E-commerce Microservices

#### User Service (Kotlin)
```kotlin
// Separate service running on its own
@RestController
@RequestMapping("/api/users")
class UserServiceController {

    @GetMapping("/{id}")
    fun getUser(@PathVariable id: Long): User {
        return userRepository.findById(id)
            .orElseThrow { UserNotFoundException(id) }
    }

    @PostMapping
    fun createUser(@RequestBody user: User): User {
        return userRepository.save(user)
    }
}

// Its own database
interface UserRepository : JpaRepository<User, Long>
```

#### Order Service (Python/Flask)
```python
# Different technology, separate service
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

@app.route('/api/orders', methods=['POST'])
def create_order():
    order_data = request.json

    # Call Product Service via HTTP
    product = requests.get(
        f'http://product-service/api/products/{order_data["product_id"]}'
    ).json()

    # Call Inventory Service
    inventory = requests.post(
        f'http://inventory-service/api/inventory/reserve',
        json={'product_id': order_data['product_id'], 'quantity': 1}
    ).json()

    if not inventory['available']:
        return jsonify({'error': 'Out of stock'}), 400

    # Call Payment Service
    payment = requests.post(
        f'http://payment-service/api/payments',
        json={'amount': product['price']}
    ).json()

    # Save order to its own database
    order = save_order(order_data)
    return jsonify(order), 201

if __name__ == '__main__':
    app.run(port=5001)
```

#### Payment Service (Go)
```go
// Yet another technology, separate service
package main

import (
    "encoding/json"
    "net/http"
)

type Payment struct {
    ID     string  `json:"id"`
    Amount float64 `json:"amount"`
    Status string  `json:"status"`
}

func processPayment(w http.ResponseWriter, r *http.Request) {
    var payment Payment
    json.NewDecoder(r.Body).Decode(&payment)

    // Process payment logic
    payment.Status = "completed"
    payment.ID = generateID()

    // Store in its own database
    saveToDatabase(payment)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(payment)
}

func main() {
    http.HandleFunc("/api/payments", processPayment)
    http.ListenAndServe(":5002", nil)
}
```

### Communication Patterns

#### 1. Synchronous (REST/HTTP)
```dart
// Flutter calling microservices via API Gateway
class OrderService {
  final Dio _dio = Dio(BaseOptions(baseUrl: 'https://api.myapp.com'));

  Future<Order> createOrder(CreateOrderRequest request) async {
    // API Gateway routes to Order Service
    final response = await _dio.post('/orders', data: request.toJson());
    return Order.fromJson(response.data);
  }
}
```

#### 2. Asynchronous (Message Queue)
```
Order Service → [RabbitMQ/Kafka] → Email Service
                                 → Inventory Service
                                 → Analytics Service
```

### Service Discovery

```yaml
# Kubernetes service discovery example
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
    - port: 80
      targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 5001
```

### Advantages

✅ **Independent Scaling**: Scale only the services that need it
✅ **Technology Flexibility**: Use the best tool for each job
✅ **Faster Deployment**: Deploy services independently
✅ **Team Autonomy**: Teams can work independently
✅ **Fault Isolation**: One service crash doesn't affect others
✅ **Easier Maintenance**: Smaller codebases are easier to understand
✅ **Continuous Deployment**: Can deploy updates without downtime
✅ **Better Resource Utilization**: Optimize each service separately

### Disadvantages

❌ **Complexity**: Distributed systems are harder to build and manage
❌ **Network Latency**: Inter-service calls over network are slower
❌ **Data Consistency**: No distributed transactions (eventual consistency)
❌ **Testing Complexity**: Must test service interactions
❌ **Operational Overhead**: Need to manage many services
❌ **Debugging Difficulty**: Tracing issues across services is hard
❌ **Infrastructure Costs**: More resources needed for operation
❌ **Learning Curve**: Requires knowledge of distributed systems

---

## Detailed Comparison

### Development Perspective

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Initial Setup** | Fast and simple | Complex, requires infrastructure |
| **Development Speed** | Fast initially | Slower initially, faster long-term |
| **Code Sharing** | Easy (same codebase) | Harder (need libraries/contracts) |
| **Refactoring** | Easy within service | Difficult across services |
| **Technology** | One stack | Multiple stacks possible |
| **Learning Curve** | Low | High |

### Deployment Perspective

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Deployment** | Single artifact | Multiple artifacts |
| **Build Time** | Slow (entire app) | Fast (individual services) |
| **Rollback** | All or nothing | Can rollback individual services |
| **Infrastructure** | Simple | Complex (orchestration needed) |
| **CI/CD** | Simple pipeline | Complex, multiple pipelines |
| **Versioning** | Single version | Multiple service versions |

### Runtime Perspective

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Performance** | Fast (in-process calls) | Network overhead |
| **Scaling** | Vertical/Horizontal (entire app) | Independent horizontal scaling |
| **Reliability** | Single point of failure | Better fault isolation |
| **Monitoring** | Simpler | Complex (distributed tracing needed) |
| **Data Consistency** | Strong (ACID) | Eventual (CAP theorem) |
| **Resource Usage** | Lower | Higher (multiple instances) |

### Team Perspective

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Team Structure** | Can be shared codebase | Independent teams per service |
| **Coordination** | High (merge conflicts) | Low (service boundaries) |
| **Ownership** | Shared | Clear ownership |
| **Onboarding** | Learn entire system | Learn specific services |
| **Skill Requirements** | Standard development | DevOps, distributed systems |

---

## When to Use Each

### Choose Monolith When:

✅ **Starting a New Project**
- You're validating an idea/MVP
- Market fit is uncertain
- Need to launch quickly

✅ **Small Team**
- Team of 1-10 developers
- Everyone can understand the codebase
- Simple coordination

✅ **Simple Domain**
- Application logic is straightforward
- Doesn't need independent scaling
- Few distinct business domains

✅ **Limited Traffic**
- Can be handled by vertical scaling
- Predictable load patterns
- Not expecting massive scale

✅ **Limited Resources**
- Small infrastructure budget
- Limited DevOps expertise
- Can't maintain complex infrastructure

### Example Scenarios for Monolith:
- Internal business tools
- Small to medium SaaS applications
- Content management systems
- Startup MVPs
- Personal projects

### Choose Microservices When:

✅ **Large, Complex Domain**
- Multiple distinct business capabilities
- Complex business logic per domain
- Natural service boundaries exist

✅ **Large Teams**
- 20+ developers
- Multiple teams need autonomy
- Conway's Law considerations

✅ **Independent Scaling Needs**
- Different features have different load patterns
- Need to scale specific features independently
- Cost optimization through targeted scaling

✅ **Different Technology Requirements**
- Some services benefit from specific technologies
- Need to use specialized tools
- Want technology flexibility

✅ **High Availability Requirements**
- Need to deploy without downtime
- Critical that failure isolation exists
- 99.99%+ uptime requirements

✅ **Organizational Maturity**
- Strong DevOps culture
- CI/CD pipelines in place
- Monitoring and observability expertise
- Cloud-native infrastructure

### Example Scenarios for Microservices:
- Large e-commerce platforms (Amazon, Alibaba)
- Streaming services (Netflix, Spotify)
- Social media platforms (Twitter, LinkedIn)
- Banking/Financial applications
- Enterprise SaaS at scale

---

## Impact on Flutter Apps

### For Monolithic Backends

#### Network Layer (Simple)
```dart
// Single base URL, straightforward API client
class ApiClient {
  final Dio _dio = Dio(
    BaseOptions(baseUrl: 'https://api.myapp.com')
  );

  // All endpoints from single server
  Future<User> getUser(int id) async {
    final response = await _dio.get('/users/$id');
    return User.fromJson(response.data);
  }

  Future<List<Product>> getProducts() async {
    final response = await _dio.get('/products');
    return (response.data as List)
        .map((json) => Product.fromJson(json))
        .toList();
  }

  Future<Order> createOrder(CreateOrderRequest request) async {
    final response = await _dio.post('/orders', data: request.toJson());
    return Order.fromJson(response.data);
  }
}
```

#### Error Handling (Simple)
```dart
// Consistent error format
class ApiError {
  final int code;
  final String message;

  ApiError.fromJson(Map<String, dynamic> json)
    : code = json['code'],
      message = json['message'];
}
```

### For Microservices Backends

#### Network Layer (Complex)
```dart
// Multiple service clients or API Gateway
class ServiceClients {
  // Option 1: Direct service calls
  final UserServiceClient userService;
  final ProductServiceClient productService;
  final OrderServiceClient orderService;

  // Option 2: API Gateway (Recommended)
  final ApiGatewayClient apiGateway;
}

// API Gateway approach
class ApiGatewayClient {
  final Dio _dio = Dio(
    BaseOptions(baseUrl: 'https://gateway.myapp.com')
  );

  // Gateway routes to microservices
  Future<User> getUser(int id) async {
    final response = await _dio.get('/api/users/$id');
    return User.fromJson(response.data);
  }

  Future<List<Product>> getProducts() async {
    final response = await _dio.get('/api/products');
    return (response.data as List)
        .map((json) => Product.fromJson(json))
        .toList();
  }
}
```

#### Handling Service Failures
```dart
// Circuit breaker pattern for resilience
class ResilientApiClient {
  final CircuitBreaker _circuitBreaker = CircuitBreaker(
    failureThreshold: 5,
    recoveryTimeout: Duration(seconds: 30),
  );

  Future<User> getUser(int id) async {
    return _circuitBreaker.execute(() async {
      try {
        final response = await _dio.get('/api/users/$id');
        return User.fromJson(response.data);
      } on DioException catch (e) {
        if (e.response?.statusCode == 503) {
          // Service unavailable, use cached data
          return await _cacheService.getUser(id);
        }
        rethrow;
      }
    });
  }
}
```

#### Caching Strategy
```dart
// More aggressive caching for microservices
class CachingRepository {
  final ApiGatewayClient _api;
  final CacheManager _cache;

  // Cache-first strategy
  Future<User> getUser(int id) async {
    // Try cache first
    final cached = await _cache.getUser(id);
    if (cached != null && !cached.isExpired) {
      return cached;
    }

    try {
      // Fetch from network
      final user = await _api.getUser(id);
      await _cache.saveUser(user);
      return user;
    } catch (e) {
      // If network fails, return stale cache
      if (cached != null) {
        return cached;
      }
      rethrow;
    }
  }
}
```

#### State Management Considerations
```dart
// Riverpod example with microservices
final userServiceProvider = Provider((ref) => UserServiceClient());
final productServiceProvider = Provider((ref) => ProductServiceClient());
final orderServiceProvider = Provider((ref) => OrderServiceClient());

// Handle partial failures
final checkoutProvider = StateNotifierProvider<CheckoutNotifier, CheckoutState>(
  (ref) => CheckoutNotifier(
    userService: ref.watch(userServiceProvider),
    productService: ref.watch(productServiceProvider),
    orderService: ref.watch(orderServiceProvider),
  ),
);

class CheckoutNotifier extends StateNotifier<CheckoutState> {
  CheckoutNotifier({
    required this.userService,
    required this.productService,
    required this.orderService,
  }) : super(CheckoutState.initial());

  final UserServiceClient userService;
  final ProductServiceClient productService;
  final OrderServiceClient orderService;

  Future<void> checkout() async {
    state = state.copyWith(status: CheckoutStatus.loading);

    try {
      // Call multiple services
      final user = await userService.getCurrentUser();
      final cart = await productService.getCart();

      // Handle partial success
      try {
        final order = await orderService.createOrder(cart, user);
        state = state.copyWith(
          status: CheckoutStatus.success,
          order: order,
        );
      } catch (orderError) {
        // Order service failed but others succeeded
        state = state.copyWith(
          status: CheckoutStatus.partialFailure,
          error: 'Order service unavailable. Please try again.',
          userData: user,
          cartData: cart,
        );
      }
    } catch (e) {
      state = state.copyWith(
        status: CheckoutStatus.error,
        error: e.toString(),
      );
    }
  }
}
```

### API Gateway Pattern for Flutter

```dart
// API Gateway abstracts microservices complexity
/*
  Mobile App → API Gateway → Multiple Microservices

  Benefits:
  - Single entry point
  - Authentication/Authorization centralized
  - Request aggregation (BFF pattern)
  - Protocol translation
  - Rate limiting
*/

class ApiGatewayService {
  final Dio _dio;

  ApiGatewayService(String baseUrl)
      : _dio = Dio(BaseOptions(baseUrl: baseUrl));

  // Gateway aggregates data from multiple services
  Future<DashboardData> getDashboard() async {
    // Single call to gateway
    final response = await _dio.get('/api/dashboard');

    // Gateway internally calls:
    // - User Service
    // - Order Service
    // - Product Service
    // - Notification Service
    // And aggregates response

    return DashboardData.fromJson(response.data);
  }
}
```

---

## Hybrid Approaches

### Modular Monolith

A middle ground that combines benefits of both:

```
┌─────────────────────────────────────┐
│   Modular Monolithic Application    │
│                                     │
│  ┌────────────┐  ┌────────────┐    │
│  │   User     │  │  Product   │    │
│  │   Module   │  │  Module    │    │
│  └────────────┘  └────────────┘    │
│  ┌────────────┐  ┌────────────┐    │
│  │   Order    │  │  Payment   │    │
│  │   Module   │  │  Module    │    │
│  └────────────┘  └────────────┘    │
│                                     │
│  Clear module boundaries            │
│  Single deployment                  │
│  Shared database (optional)         │
└─────────────────────────────────────┘
```

**Benefits:**
- Clear boundaries (can extract to microservices later)
- Simpler infrastructure
- Better than unstructured monolith
- Easier testing and maintenance

### Strangler Fig Pattern

Gradually migrate from monolith to microservices:

```
Phase 1: Start with monolith
┌──────────────┐
│   Monolith   │
└──────────────┘

Phase 2: Extract one service
┌──────────────┐  ┌──────────┐
│   Monolith   │  │ Service A│
└──────────────┘  └──────────┘

Phase 3: Extract more services
┌──────────────┐  ┌──────────┐  ┌──────────┐
│   Monolith   │  │ Service A│  │ Service B│
└──────────────┘  └──────────┘  └──────────┘

Phase 4: Eventually microservices
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│Service A │  │Service B │  │Service C │  │Service D │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
```

---

## Key Takeaways

### For Flutter Developers:

1. **Start Simple**: Unless you're working on a large-scale app, your backend is likely a monolith
2. **API Gateway**: If working with microservices, ensure there's an API gateway
3. **Error Handling**: Microservices require more sophisticated error handling
4. **Caching**: Essential for microservices to handle service unavailability
5. **Testing**: Test with service failures in mind for microservices backends

### Architecture Decision Framework:

```
Start Small (Monolith)
    ↓
Grow Organically (Modular Monolith)
    ↓
Scale Selectively (Hybrid: Monolith + Key Microservices)
    ↓
Full Microservices (When necessary and team is ready)
```

### Questions to Ask:

1. **Team Size**: Can we manage multiple services?
2. **Scale**: Do we need independent scaling?
3. **Complexity**: Is our domain complex enough?
4. **Resources**: Can we afford the operational overhead?
5. **Time**: Can we invest in the infrastructure?

---

## Comparison with Android Development

### Monolith ≈ Single Module Android App
```
app/
  ├── ui/
  ├── data/
  ├── domain/
  └── All features in one module
```

### Microservices ≈ Multi-Module Android App
```
:app (UI layer)
:feature:users
:feature:products
:feature:orders
:core:network
:core:database
Each module is independent
```

The principles are similar:
- **Monolith/Single Module**: Simple, easy to start, can become unwieldy
- **Microservices/Multi-Module**: More complex, better separation, scales better

---

## Additional Resources

- [Martin Fowler - Microservices](https://martinfowler.com/articles/microservices.html)
- [Sam Newman - Building Microservices](https://samnewman.io/books/building_microservices_2nd_edition/)
- [Microservices.io - Patterns](https://microservices.io/patterns/index.html)
- [Flutter BFF Pattern](https://flutter.dev/docs/development/data-and-backend/networking)

---

**Next**: [Best Practices & Tips](./09-best-practices.md) | **Previous**: [Navigation & Routing](./08-navigation-routing.md)
