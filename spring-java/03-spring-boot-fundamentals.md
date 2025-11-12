# Spring Boot Fundamentals

Learn how Spring Boot simplifies Spring application development with auto-configuration and convention over configuration.

## 🎯 Overview

**Spring Boot** = Spring Framework + Convention over Configuration + Embedded Server

Think of Spring Boot as the modern, opinionated way to build Spring applications - similar to how Jetpack Compose simplified Android UI development.

## 📋 Spring vs Spring Boot

| Aspect | Spring Framework | Spring Boot |
|--------|------------------|-------------|
| Configuration | Manual XML/Java config | Auto-configuration |
| Dependencies | Manage each dependency | Starter dependencies |
| Server | Deploy to external server (Tomcat) | Embedded server |
| Setup Time | Hours | Minutes |
| Boilerplate | Lots | Minimal |
| Production Ready | Need to configure | Built-in features |

## 1. Why Spring Boot?

### Traditional Spring Setup (Manual)

```xml
<!-- pom.xml - Traditional Spring -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.x</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.x</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.x</version>
    </dependency>
    <!-- ... many more individual dependencies -->
</dependencies>
```

### Spring Boot Setup (Simplified)

```xml
<!-- pom.xml - Spring Boot -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- Version managed by Spring Boot parent -->
    </dependency>
</dependencies>
```

One starter = All related dependencies with compatible versions!

## 2. Spring Boot Starters

Starters are dependency descriptors that bring in everything you need for a specific functionality.

### Common Starters

| Starter | Purpose | Includes |
|---------|---------|----------|
| `spring-boot-starter-web` | Web applications + REST | Spring MVC, Tomcat, Jackson |
| `spring-boot-starter-data-jpa` | JPA with Hibernate | JPA, Hibernate, JDBC |
| `spring-boot-starter-security` | Security | Spring Security |
| `spring-boot-starter-test` | Testing | JUnit, Mockito, AssertJ |
| `spring-boot-starter-validation` | Validation | Hibernate Validator |
| `spring-boot-starter-actuator` | Production monitoring | Health checks, metrics |
| `spring-boot-starter-data-mongodb` | MongoDB | Spring Data MongoDB |
| `spring-boot-starter-data-redis` | Redis | Spring Data Redis, Lettuce |
| `spring-boot-starter-mail` | Email | JavaMail |
| `spring-boot-starter-websocket` | WebSocket | WebSocket support |

### Example: pom.xml with Spring Boot

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <name>My Spring Boot App</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Web + REST API -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Database (JPA + MySQL) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Development Tools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Gradle Example (build.gradle)

```groovy
plugins {
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.3'
    id 'java'
}

group = 'com.example'
version = '1.0.0'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    runtimeOnly 'com.mysql:mysql-connector-j'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

## 3. The Main Application Class

**@SpringBootApplication** is a combination of:
- `@Configuration`: Tags the class as a source of bean definitions
- `@EnableAutoConfiguration`: Enables Spring Boot's auto-configuration
- `@ComponentScan`: Enables component scanning

```java
package com.example.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

**What happens when you run this?**
```
1. Spring Boot starts
2. Creates ApplicationContext
3. Scans for @Component classes in com.example.myapp package
4. Auto-configures beans based on classpath
5. Starts embedded Tomcat server (default port 8080)
6. Application is ready!
```

### Customizing the Main Class

```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        // Customize before running
        SpringApplication app = new SpringApplication(MyApplication.class);

        // Set custom port
        app.setDefaultProperties(Collections.singletonMap("server.port", "9090"));

        // Add listeners
        app.addListeners(new ApplicationStartupListener());

        app.run(args);
    }

    @Bean
    public CommandLineRunner demo(UserRepository repository) {
        // Runs after application starts
        return args -> {
            System.out.println("Application started!");
            // Initialize data, etc.
        };
    }
}
```

## 4. Auto-Configuration Magic

Spring Boot automatically configures beans based on:
1. **Classpath dependencies** - What JARs are present
2. **Properties** - Configuration in application.properties
3. **Other beans** - What you've already defined

### How Auto-Configuration Works

```
┌──────────────────────────────────────────────────┐
│  You add spring-boot-starter-data-jpa            │
└────────────────┬─────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────┐
│  Spring Boot detects Hibernate on classpath      │
└────────────────┬─────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────┐
│  Auto-configures:                                │
│  - DataSource                                    │
│  - EntityManagerFactory                          │
│  - TransactionManager                            │
│  - JPA Repositories                              │
└──────────────────────────────────────────────────┘
```

### Example: DataSource Auto-Configuration

**You add:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

**You configure:**
```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
```

**Spring Boot auto-configures:**
```java
// You DON'T need to write this - Spring Boot does it!
@Bean
public DataSource dataSource() {
    return DataSourceBuilder.create()
        .url("jdbc:mysql://localhost:3306/mydb")
        .username("root")
        .password("secret")
        .build();
}

@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {
    // Complex configuration done automatically
}

@Bean
public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
    return new JpaTransactionManager(emf);
}
```

### Conditional Auto-Configuration

Spring Boot uses `@Conditional` annotations:

```java
@Configuration
@ConditionalOnClass(DataSource.class)  // Only if DataSource is on classpath
@ConditionalOnMissingBean(DataSource.class)  // Only if you haven't defined one
public class DataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        // Auto-configure DataSource
    }
}
```

**Override auto-configuration:**
```java
@Configuration
public class MyDatabaseConfig {

    @Bean
    public DataSource dataSource() {
        // Your custom DataSource
        // Auto-configuration will back off!
        return new HikariDataSource();
    }
}
```

## 5. Application Properties and Configuration

### application.properties

Located in `src/main/resources/application.properties`

```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/api

# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Logging
logging.level.root=INFO
logging.level.com.example.myapp=DEBUG
logging.file.name=app.log

# Application-specific
app.name=My Application
app.version=1.0.0
```

### application.yml (Alternative)

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQL8Dialect

logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
  file:
    name: app.log

app:
  name: My Application
  version: 1.0.0
```

### Using Configuration in Code

**@Value:**
```java
@Service
public class AppInfoService {

    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String appVersion;

    @Value("${server.port}")
    private int serverPort;

    public String getInfo() {
        return String.format("%s v%s running on port %d",
            appName, appVersion, serverPort);
    }
}
```

**@ConfigurationProperties (Type-safe):**
```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {

    private String name;
    private String version;
    private Security security = new Security();

    // Getters and setters

    public static class Security {
        private String secret;
        private int tokenExpiry;

        // Getters and setters
    }
}
```

```properties
app.name=My Application
app.version=1.0.0
app.security.secret=mySecretKey
app.security.token-expiry=3600
```

```java
@Service
public class TokenService {

    private final AppProperties appProperties;

    public TokenService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }

    public String generateToken() {
        String secret = appProperties.getSecurity().getSecret();
        int expiry = appProperties.getSecurity().getTokenExpiry();
        // Generate token
    }
}
```

## 6. Profiles

Profiles allow different configurations for different environments (dev, test, prod).

### Define Profiles

**application-dev.properties:**
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb_dev
logging.level.root=DEBUG
```

**application-test.properties:**
```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.jpa.hibernate.ddl-auto=create-drop
```

**application-prod.properties:**
```properties
spring.datasource.url=jdbc:mysql://prod-server:3306/mydb_prod
logging.level.root=WARN
server.port=80
```

### Activate Profiles

**In application.properties:**
```properties
spring.profiles.active=dev
```

**Command line:**
```bash
java -jar myapp.jar --spring.profiles.active=prod
```

**Environment variable:**
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar myapp.jar
```

**In IDE (IntelliJ):**
```
Run > Edit Configurations > Environment Variables
SPRING_PROFILES_ACTIVE=dev
```

### Profile-specific Beans

```java
@Configuration
public class DatabaseConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setMaximumPoolSize(20);
        // Production settings
        return dataSource;
    }
}
```

**Component with profile:**
```java
@Service
@Profile("!prod")  // Active in all profiles except prod
public class MockEmailService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("Mock: Sending email to " + to);
    }
}

@Service
@Profile("prod")
public class RealEmailService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        // Actually send email via SMTP
    }
}
```

## 7. Project Structure

### Standard Spring Boot Structure

```
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── myapp/
│   │   │               ├── MyApplication.java          [@SpringBootApplication]
│   │   │               ├── controller/
│   │   │               │   └── UserController.java     [@RestController]
│   │   │               ├── service/
│   │   │               │   └── UserService.java        [@Service]
│   │   │               ├── repository/
│   │   │               │   └── UserRepository.java     [Interface extends JpaRepository]
│   │   │               ├── model/
│   │   │               │   └── User.java               [@Entity]
│   │   │               ├── dto/
│   │   │               │   └── UserDTO.java            [Data Transfer Object]
│   │   │               ├── config/
│   │   │               │   └── AppConfig.java          [@Configuration]
│   │   │               └── exception/
│   │   │                   └── GlobalExceptionHandler.java  [@ControllerAdvice]
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── application-dev.properties
│   │       ├── application-prod.properties
│   │       ├── static/                 [Static resources: CSS, JS, images]
│   │       └── templates/              [Thymeleaf templates if using]
│   └── test/
│       └── java/
│           └── com/
│               └── example/
│                   └── myapp/
│                       ├── MyApplicationTests.java
│                       ├── controller/
│                       │   └── UserControllerTest.java
│                       └── service/
│                           └── UserServiceTest.java
├── pom.xml  (or build.gradle)
└── README.md
```

**Comparison with Android:**

| Spring Boot | Android |
|-------------|---------|
| `controller/` | `ui/` (Activities/Fragments) |
| `service/` | `domain/` or `viewmodel/` |
| `repository/` | `data/repository/` |
| `model/` | `data/model/` |
| `dto/` | `ui/model/` (UI models) |
| `config/` | `di/` (Dagger modules) |

## 8. Spring Boot DevTools

Provides development-time features:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**Features:**
- **Automatic Restart** - Restarts app when files change
- **LiveReload** - Browser auto-refresh
- **Property Defaults** - Development-friendly defaults
- **Remote Debug** - Debug apps running remotely

## 9. Packaging and Running

### Create Executable JAR

```bash
# Maven
./mvnw clean package

# Gradle
./gradlew build
```

Produces: `target/myapp-1.0.0.jar` or `build/libs/myapp-1.0.0.jar`

### Run the JAR

```bash
java -jar target/myapp-1.0.0.jar
```

### Run with Profile

```bash
java -jar target/myapp-1.0.0.jar --spring.profiles.active=prod
```

### Run with Custom Properties

```bash
java -jar target/myapp-1.0.0.jar \
    --server.port=9090 \
    --spring.datasource.url=jdbc:mysql://localhost:3306/customdb
```

## 10. Common Configuration Examples

### CORS Configuration

```java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("http://localhost:3000")
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowedHeaders("*")
                    .allowCredentials(true);
            }
        };
    }
}
```

### Scheduler Configuration

```java
@Configuration
@EnableScheduling
public class SchedulerConfig {
}

@Component
public class ScheduledTasks {

    @Scheduled(fixedRate = 5000)  // Every 5 seconds
    public void reportCurrentTime() {
        System.out.println("Current time: " + new Date());
    }

    @Scheduled(cron = "0 0 2 * * ?")  // 2 AM every day
    public void performDailyCleanup() {
        System.out.println("Running daily cleanup...");
    }
}
```

### Async Configuration

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class AsyncService {

    @Async
    public CompletableFuture<String> processAsync() {
        // Long-running task
        return CompletableFuture.completedFuture("Done!");
    }
}
```

## 🎯 Key Takeaways

1. **Spring Boot** simplifies Spring development with auto-configuration
2. **Starters** bundle related dependencies with compatible versions
3. **@SpringBootApplication** = @Configuration + @EnableAutoConfiguration + @ComponentScan
4. **application.properties** centralizes configuration
5. **Profiles** enable environment-specific configurations
6. **DevTools** improves development experience
7. **Executable JAR** contains embedded server - no deployment needed

## 🚀 Next Steps

Now that you understand Spring Boot fundamentals, let's build REST APIs!

Continue to [Section 4: Building REST APIs with Spring MVC](./04-rest-api-spring-mvc.md)
