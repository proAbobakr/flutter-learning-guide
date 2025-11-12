# Building REST APIs with Spring MVC

Learn how to build RESTful web services with Spring MVC, similar to creating backend APIs for your Android apps.

## 🎯 Overview

Spring MVC is the web framework for building REST APIs in Spring. If you've used **Retrofit** to consume APIs or **Ktor Server** to create them, you'll find Spring MVC familiar.

## 📋 Quick Comparison

| Concept | Retrofit (Client) | Ktor Server | Spring MVC |
|---------|-------------------|-------------|------------|
| Endpoint | `@GET("/users/{id}")` | `get("/users/{id}")` | `@GetMapping("/users/{id}")` |
| Path Variable | `@Path("id")` | `{id}` parameter | `@PathVariable Long id` |
| Query Param | `@Query("name")` | `parameters["name"]` | `@RequestParam String name` |
| Request Body | `@Body User user` | `receive<User>()` | `@RequestBody User user` |
| Response | `Call<User>` | `call.respond(user)` | `return user` |

## 1. REST Basics

### HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Update/Replace resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Delete resource | Yes | No |

### RESTful URL Design

```
Good:
GET    /api/users              - List all users
GET    /api/users/123          - Get user by ID
POST   /api/users              - Create new user
PUT    /api/users/123          - Update user
DELETE /api/users/123          - Delete user
GET    /api/users/123/posts    - Get posts by user

Bad:
GET    /api/getAllUsers
POST   /api/createUser
GET    /api/user?id=123
```

## 2. Creating Your First REST Controller

### Simple Controller

```java
package com.example.myapp.controller;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/hello")
public class HelloController {

    @GetMapping
    public String sayHello() {
        return "Hello, World!";
    }

    @GetMapping("/{name}")
    public String sayHelloToName(@PathVariable String name) {
        return "Hello, " + name + "!";
    }

    @GetMapping("/greet")
    public String greet(@RequestParam String name,
                       @RequestParam(defaultValue = "en") String lang) {
        if ("es".equals(lang)) {
            return "¡Hola, " + name + "!";
        }
        return "Hello, " + name + "!";
    }
}
```

**Test:**
```bash
curl http://localhost:8080/api/hello
# Output: Hello, World!

curl http://localhost:8080/api/hello/John
# Output: Hello, John!

curl http://localhost:8080/api/hello/greet?name=John&lang=es
# Output: ¡Hola, John!
```

## 3. Complete CRUD REST API

### Domain Model

```java
package com.example.myapp.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.Data;

@Entity
@Table(name = "users")
@Data
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    @Column(unique = true)
    private String email;

    @Min(value = 18, message = "Age must be at least 18")
    private Integer age;

    @Column(name = "created_at")
    private java.time.LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = java.time.LocalDateTime.now();
    }
}
```

### DTO (Data Transfer Object)

```java
package com.example.myapp.dto;

import jakarta.validation.constraints.*;
import lombok.Data;

@Data
public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50)
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;

    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Age must be at least 18")
    private Integer age;
}

@Data
public class UserResponse {
    private Long id;
    private String name;
    private String email;
    private Integer age;
    private String createdAt;
}
```

### Repository

```java
package com.example.myapp.repository;

import com.example.myapp.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

    boolean existsByEmail(String email);
}
```

### Service Layer

```java
package com.example.myapp.service;

import com.example.myapp.dto.*;
import com.example.myapp.exception.ResourceNotFoundException;
import com.example.myapp.model.User;
import com.example.myapp.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public List<UserResponse> getAllUsers() {
        return userRepository.findAll().stream()
            .map(this::mapToResponse)
            .collect(Collectors.toList());
    }

    public UserResponse getUserById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
        return mapToResponse(user);
    }

    @Transactional
    public UserResponse createUser(CreateUserRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new IllegalArgumentException("Email already exists");
        }

        User user = new User();
        user.setName(request.getName());
        user.setEmail(request.getEmail());
        user.setAge(request.getAge());

        User savedUser = userRepository.save(user);
        return mapToResponse(savedUser);
    }

    @Transactional
    public UserResponse updateUser(Long id, CreateUserRequest request) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));

        user.setName(request.getName());
        user.setEmail(request.getEmail());
        user.setAge(request.getAge());

        User updatedUser = userRepository.save(user);
        return mapToResponse(updatedUser);
    }

    @Transactional
    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User not found with id: " + id);
        }
        userRepository.deleteById(id);
    }

    private UserResponse mapToResponse(User user) {
        UserResponse response = new UserResponse();
        response.setId(user.getId());
        response.setName(user.getName());
        response.setEmail(user.getEmail());
        response.setAge(user.getAge());
        response.setCreatedAt(user.getCreatedAt().toString());
        return response;
    }
}
```

### REST Controller

```java
package com.example.myapp.controller;

import com.example.myapp.dto.*;
import com.example.myapp.service.UserService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<List<UserResponse>> getAllUsers() {
        List<UserResponse> users = userService.getAllUsers();
        return ResponseEntity.ok(users);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUserById(@PathVariable Long id) {
        UserResponse user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserResponse createdUser = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody CreateUserRequest request) {
        UserResponse updatedUser = userService.updateUser(id, request);
        return ResponseEntity.ok(updatedUser);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

## 4. Exception Handling

### Custom Exceptions

```java
package com.example.myapp.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### Global Exception Handler

```java
package com.example.myapp.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgument(IllegalArgumentException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidationExceptions(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        Map<String, Object> response = new HashMap<>();
        response.put("status", HttpStatus.BAD_REQUEST.value());
        response.put("errors", errors);
        response.put("timestamp", LocalDateTime.now());

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}

// Error Response DTO
@Data
@AllArgsConstructor
class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
}
```

## 5. Request/Response Examples

### GET Request
```bash
curl http://localhost:8080/api/users
```

**Response:**
```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "age": 25,
    "createdAt": "2024-01-15T10:30:00"
  },
  {
    "id": 2,
    "name": "Jane Smith",
    "email": "jane@example.com",
    "age": 30,
    "createdAt": "2024-01-16T14:20:00"
  }
]
```

### POST Request
```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Alice Johnson",
    "email": "alice@example.com",
    "age": 28
  }'
```

**Response (201 Created):**
```json
{
  "id": 3,
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "age": 28,
  "createdAt": "2024-01-17T09:15:00"
}
```

### PUT Request
```bash
curl -X PUT http://localhost:8080/api/users/3 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Alice Williams",
    "email": "alice.williams@example.com",
    "age": 29
  }'
```

### DELETE Request
```bash
curl -X DELETE http://localhost:8080/api/users/3
```

**Response: 204 No Content**

### Validation Error Example
```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "A",
    "email": "invalid-email",
    "age": 15
  }'
```

**Response (400 Bad Request):**
```json
{
  "status": 400,
  "errors": {
    "name": "Name must be between 2 and 50 characters",
    "email": "Email should be valid",
    "age": "Age must be at least 18"
  },
  "timestamp": "2024-01-17T10:00:00"
}
```

## 6. Advanced Features

### Pagination

```java
@GetMapping
public ResponseEntity<Page<UserResponse>> getAllUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id,asc") String[] sort) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(parseSortParams(sort)));
    Page<UserResponse> users = userService.getAllUsers(pageable);
    return ResponseEntity.ok(users);
}

// Service method
public Page<UserResponse> getAllUsers(Pageable pageable) {
    return userRepository.findAll(pageable)
        .map(this::mapToResponse);
}
```

**Request:**
```bash
curl "http://localhost:8080/api/users?page=0&size=10&sort=name,asc"
```

**Response:**
```json
{
  "content": [...],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10
  },
  "totalPages": 5,
  "totalElements": 50,
  "last": false,
  "first": true
}
```

### Filtering/Search

```java
@GetMapping("/search")
public ResponseEntity<List<UserResponse>> searchUsers(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) String email,
        @RequestParam(required = false) Integer minAge) {

    List<UserResponse> users = userService.searchUsers(name, email, minAge);
    return ResponseEntity.ok(users);
}

// Repository method
@Query("SELECT u FROM User u WHERE " +
       "(:name IS NULL OR u.name LIKE %:name%) AND " +
       "(:email IS NULL OR u.email LIKE %:email%) AND " +
       "(:minAge IS NULL OR u.age >= :minAge)")
List<User> searchUsers(
    @Param("name") String name,
    @Param("email") String email,
    @Param("minAge") Integer minAge
);
```

### File Upload

```java
@PostMapping("/upload")
public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
    if (file.isEmpty()) {
        return ResponseEntity.badRequest().body("File is empty");
    }

    try {
        String filename = file.getOriginalFilename();
        Path path = Paths.get("uploads/" + filename);
        Files.write(path, file.getBytes());

        return ResponseEntity.ok("File uploaded: " + filename);
    } catch (IOException e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body("Failed to upload file");
    }
}
```

### File Download

```java
@GetMapping("/download/{filename}")
public ResponseEntity<Resource> downloadFile(@PathVariable String filename) {
    try {
        Path path = Paths.get("uploads/" + filename);
        Resource resource = new UrlResource(path.toUri());

        if (resource.exists()) {
            return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION,
                    "attachment; filename=\"" + resource.getFilename() + "\"")
                .body(resource);
        } else {
            return ResponseEntity.notFound().build();
        }
    } catch (MalformedURLException e) {
        return ResponseEntity.badRequest().build();
    }
}
```

### Custom Headers

```java
@GetMapping("/{id}")
public ResponseEntity<UserResponse> getUserById(@PathVariable Long id) {
    UserResponse user = userService.getUserById(id);

    return ResponseEntity.ok()
        .header("X-Custom-Header", "CustomValue")
        .header("X-Request-ID", UUID.randomUUID().toString())
        .body(user);
}
```

### Response Status

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)  // Always return 201
public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
    return userService.createUser(request);
}

@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)  // Always return 204
public void deleteUser(@PathVariable Long id) {
    userService.deleteUser(id);
}
```

## 7. Content Negotiation

```java
@GetMapping(value = "/{id}", produces = {
    MediaType.APPLICATION_JSON_VALUE,
    MediaType.APPLICATION_XML_VALUE
})
public ResponseEntity<User> getUserById(@PathVariable Long id) {
    User user = userService.getUserById(id);
    return ResponseEntity.ok(user);
}
```

**Request:**
```bash
# JSON response
curl -H "Accept: application/json" http://localhost:8080/api/users/1

# XML response
curl -H "Accept: application/xml" http://localhost:8080/api/users/1
```

## 8. API Versioning

### URI Versioning
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { ... }
```

### Header Versioning
```java
@GetMapping(headers = "X-API-VERSION=1")
public List<UserResponse> getUsersV1() { ... }

@GetMapping(headers = "X-API-VERSION=2")
public List<UserResponse> getUsersV2() { ... }
```

## 9. Testing REST Controllers

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void shouldGetAllUsers() throws Exception {
        List<UserResponse> users = List.of(
            new UserResponse(1L, "John", "john@example.com", 25, "2024-01-15T10:30:00")
        );

        when(userService.getAllUsers()).thenReturn(users);

        mockMvc.perform(get("/api/users"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$[0].name").value("John"))
            .andExpect(jsonPath("$[0].email").value("john@example.com"));
    }

    @Test
    void shouldCreateUser() throws Exception {
        CreateUserRequest request = new CreateUserRequest();
        request.setName("John");
        request.setEmail("john@example.com");
        request.setAge(25);

        UserResponse response = new UserResponse(1L, "John", "john@example.com", 25, "2024-01-15T10:30:00");

        when(userService.createUser(any())).thenReturn(response);

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("John"));
    }
}
```

## 🎯 Key Takeaways

1. **@RestController** combines @Controller and @ResponseBody
2. **@RequestMapping** maps HTTP requests to handler methods
3. **@PathVariable** extracts values from URI path
4. **@RequestParam** extracts query parameters
5. **@RequestBody** binds request body to object
6. **@Valid** triggers validation
7. **@RestControllerAdvice** handles exceptions globally
8. **ResponseEntity** provides full control over HTTP response

## 🚀 Next Steps

Now that you can build REST APIs, let's learn Spring Data JPA for database operations!

Continue to [Section 5: Spring Data JPA](./05-spring-data-jpa.md)
