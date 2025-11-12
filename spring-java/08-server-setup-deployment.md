# Server Setup and Deployment

Learn how to set up, configure, and deploy Spring Boot applications to production - from local development to cloud infrastructure.

## 🎯 Overview

**Deployment** = Getting your Spring Boot app from your laptop to production servers where users can access it.

Think of it like preparing an Android app for production: you configure build variants, sign the APK, test on different devices, and deploy to Google Play Store. Similarly, Spring Boot apps need environment configuration, packaging, containerization, and deployment to cloud platforms.

## 📋 Deployment Journey

| Phase | Android Development | Spring Boot Backend |
|-------|---------------------|---------------------|
| Local Dev | Run on emulator | Run locally with embedded server |
| Configuration | build.gradle variants | application.properties profiles |
| Packaging | Build APK/AAB | Build executable JAR |
| Containerization | N/A (OS handles it) | Docker container |
| Deployment | Google Play Store | AWS, GCP, Azure, Heroku |
| Monitoring | Firebase Analytics | Prometheus, Grafana, Actuator |
| CI/CD | GitHub Actions → Play Store | GitHub Actions → Cloud |

---

## 1. Local Development Environment Setup

### 1.1 Prerequisites

**Required Tools:**
```bash
# Java Development Kit (JDK)
java -version  # Should be 17 or higher
javac -version

# Maven or Gradle
mvn -version
gradle -version

# IDE (IntelliJ IDEA, VS Code, Eclipse)
# Similar to Android Studio but for backend development

# Git
git --version

# Optional but recommended
docker --version
docker-compose --version
```

### 1.2 Install Java (if not installed)

**On macOS:**
```bash
# Using Homebrew
brew install openjdk@17

# Add to PATH
echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**On Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install openjdk-17-jdk

# Verify installation
java -version
```

**On Windows:**
```powershell
# Download from Oracle or use chocolatey
choco install openjdk17
```

### 1.3 IDE Setup

**IntelliJ IDEA (Recommended)**
- Download: https://www.jetbrains.com/idea/
- Install Spring Boot plugin (usually pre-installed)
- Configure JDK: File → Project Structure → SDK

**VS Code**
- Install extensions:
  - Spring Boot Extension Pack
  - Java Extension Pack
  - Maven for Java

### 1.4 Create a New Spring Boot Project

**Option 1: Spring Initializr (Web)**
```bash
# Visit https://start.spring.io/
# Or use curl
curl https://start.spring.io/starter.zip \
  -d dependencies=web,data-jpa,mysql,actuator \
  -d language=java \
  -d type=maven-project \
  -d bootVersion=3.2.0 \
  -d baseDir=myapp \
  -d groupId=com.example \
  -d artifactId=myapp \
  -o myapp.zip

unzip myapp.zip
cd myapp
```

**Option 2: Using Spring Boot CLI**
```bash
# Install Spring Boot CLI
brew tap spring-io/tap
brew install spring-boot

# Create project
spring init --dependencies=web,data-jpa,mysql,actuator myapp
cd myapp
```

**Option 3: Using IntelliJ IDEA**
1. File → New → Project
2. Select "Spring Initializr"
3. Configure project settings
4. Select dependencies
5. Create

---

## 2. Database Setup

### 2.1 H2 Database (In-Memory) - Development

Perfect for quick development and testing, similar to using SQLite in Android.

**Add Dependency (pom.xml):**
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

**Configuration (application.properties):**
```properties
# H2 Database
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# H2 Console (access at http://localhost:8080/h2-console)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```

**Access H2 Console:**
- URL: http://localhost:8080/h2-console
- JDBC URL: jdbc:h2:mem:testdb
- Username: sa
- Password: (empty)

### 2.2 MySQL Setup

**Install MySQL:**
```bash
# macOS
brew install mysql
brew services start mysql

# Ubuntu
sudo apt update
sudo apt install mysql-server
sudo systemctl start mysql
sudo systemctl enable mysql

# Set root password
sudo mysql_secure_installation
```

**Create Database:**
```sql
-- Login to MySQL
mysql -u root -p

-- Create database
CREATE DATABASE myapp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'your_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON myapp_db.* TO 'myapp_user'@'localhost';
FLUSH PRIVILEGES;

-- Verify
SHOW DATABASES;
USE myapp_db;
```

**Add MySQL Dependency (pom.xml):**
```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

**Configuration (application.properties):**
```properties
# MySQL Database
spring.datasource.url=jdbc:mysql://localhost:3306/myapp_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=myapp_user
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
spring.jpa.properties.hibernate.format_sql=true
```

### 2.3 PostgreSQL Setup

**Install PostgreSQL:**
```bash
# macOS
brew install postgresql@15
brew services start postgresql@15

# Ubuntu
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**Create Database:**
```bash
# Switch to postgres user
sudo -u postgres psql

# In PostgreSQL prompt
CREATE DATABASE myapp_db;
CREATE USER myapp_user WITH ENCRYPTED PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE myapp_db TO myapp_user;

# PostgreSQL 15+ requires additional permissions
\c myapp_db
GRANT ALL ON SCHEMA public TO myapp_user;

\q  # Exit
```

**Add PostgreSQL Dependency (pom.xml):**
```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

**Configuration (application.properties):**
```properties
# PostgreSQL Database
spring.datasource.url=jdbc:postgresql://localhost:5432/myapp_db
spring.datasource.username=myapp_user
spring.datasource.password=your_password
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
```

### 2.4 Docker Database Setup (Recommended for Development)

**docker-compose.yml for multiple databases:**
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: myapp-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp_db
      MYSQL_USER: myapp_user
      MYSQL_PASSWORD: your_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - myapp-network

  postgres:
    image: postgres:15
    container_name: myapp-postgres
    environment:
      POSTGRES_DB: myapp_db
      POSTGRES_USER: myapp_user
      POSTGRES_PASSWORD: your_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - myapp-network

  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - myapp-network

volumes:
  mysql_data:
  postgres_data:
  redis_data:

networks:
  myapp-network:
    driver: bridge
```

**Usage:**
```bash
# Start all databases
docker-compose up -d

# Start specific database
docker-compose up -d mysql

# Stop all
docker-compose down

# Stop and remove volumes (delete data)
docker-compose down -v

# View logs
docker-compose logs -f mysql
```

---

## 3. Application Configuration for Different Environments

### 3.1 Spring Profiles

Spring profiles allow you to have different configurations for different environments, similar to Android's build variants (debug/release).

**Profile Structure:**
```
src/main/resources/
├── application.properties              # Default/common config
├── application-dev.properties          # Development
├── application-test.properties         # Testing
├── application-staging.properties      # Staging
└── application-prod.properties         # Production
```

### 3.2 Configuration Files

**application.properties (Default/Common):**
```properties
# Application
spring.application.name=myapp
server.port=8080

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG

# Actuator
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=when-authorized
```

**application-dev.properties:**
```properties
# Development Environment
spring.profiles.active=dev

# Database - H2 for fast development
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=
spring.h2.console.enabled=true

# JPA
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging - Verbose
logging.level.root=DEBUG
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

# Development tools
spring.devtools.restart.enabled=true
spring.devtools.livereload.enabled=true

# Disable security for easier testing (if using Spring Security)
# spring.security.user.name=admin
# spring.security.user.password=admin
```

**application-test.properties:**
```properties
# Test Environment
spring.profiles.active=test

# Database - H2 for testing
spring.datasource.url=jdbc:h2:mem:testdb
spring.jpa.hibernate.ddl-auto=create-drop

# Logging - Minimal
logging.level.root=WARN
logging.level.com.example=INFO
```

**application-staging.properties:**
```properties
# Staging Environment
spring.profiles.active=staging

# Database - Similar to production
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

# JPA - Don't auto-create schema
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG

# Actuator - More endpoints for debugging
management.endpoints.web.exposure.include=health,info,metrics,env,loggers
```

**application-prod.properties:**
```properties
# Production Environment
spring.profiles.active=prod

# Database - Use environment variables
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000

# JPA - Never auto-create schema in production
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.open-in-view=false

# Logging - Structured logging
logging.level.root=WARN
logging.level.com.example=INFO
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
logging.file.name=/var/log/myapp/application.log
logging.file.max-size=10MB
logging.file.max-history=30

# Server
server.port=8080
server.compression.enabled=true
server.http2.enabled=true

# Actuator - Minimal exposure
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=never

# Security
server.error.include-message=never
server.error.include-stacktrace=never
```

### 3.3 YAML Configuration (Alternative)

**application.yml:**
```yaml
spring:
  application:
    name: myapp
  profiles:
    active: ${SPRING_PROFILE:dev}

server:
  port: 8080

---
# Development Profile
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password:
  h2:
    console:
      enabled: true
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

logging:
  level:
    root: DEBUG
    com.example: DEBUG

---
# Production Profile
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

logging:
  level:
    root: WARN
    com.example: INFO
  file:
    name: /var/log/myapp/application.log

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

### 3.4 Activating Profiles

**Method 1: Command Line**
```bash
# Run with specific profile
java -jar myapp.jar --spring.profiles.active=prod

# Multiple profiles
java -jar myapp.jar --spring.profiles.active=prod,mysql
```

**Method 2: Environment Variable**
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar myapp.jar
```

**Method 3: application.properties**
```properties
spring.profiles.active=dev
```

**Method 4: In Code**
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.setAdditionalProfiles("dev");
        app.run(args);
    }
}
```

**Method 5: IDE (IntelliJ IDEA)**
1. Run → Edit Configurations
2. Add to "Environment variables": `SPRING_PROFILES_ACTIVE=dev`
3. Or add to "Program arguments": `--spring.profiles.active=dev`

---

## 4. Building Executable JARs

### 4.1 Maven Build

**Build with Maven:**
```bash
# Clean and build
mvn clean package

# Skip tests
mvn clean package -DskipTests

# Build with specific profile
mvn clean package -Pprod

# The JAR will be in: target/myapp-0.0.1-SNAPSHOT.jar
```

**pom.xml configuration:**
```xml
<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
                <mainClass>com.example.myapp.MyApplication</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 4.2 Gradle Build

**Build with Gradle:**
```bash
# Clean and build
./gradlew clean build

# Skip tests
./gradlew clean build -x test

# The JAR will be in: build/libs/myapp-0.0.1-SNAPSHOT.jar
```

**build.gradle configuration:**
```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

bootJar {
    archiveFileName = "${archiveBaseName.get()}.${archiveExtension.get()}"
    mainClass = 'com.example.myapp.MyApplication'
}
```

### 4.3 Running the JAR

```bash
# Basic run
java -jar myapp.jar

# With profile
java -jar myapp.jar --spring.profiles.active=prod

# With custom port
java -jar myapp.jar --server.port=9090

# With JVM options
java -Xms512m -Xmx2048m -jar myapp.jar

# With environment variables
DB_URL=jdbc:mysql://localhost:3306/myapp \
DB_USERNAME=user \
DB_PASSWORD=pass \
java -jar myapp.jar --spring.profiles.active=prod
```

### 4.4 Creating Executable JAR (Unix/Linux)

Make the JAR directly executable on Unix systems:

**pom.xml:**
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

**Usage:**
```bash
# Make executable
chmod +x myapp.jar

# Run directly
./myapp.jar --spring.profiles.active=prod
```

### 4.5 Multi-Module Project Build

**Parent pom.xml:**
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>myapp-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>myapp-core</module>
        <module>myapp-api</module>
        <module>myapp-web</module>
    </modules>
</project>
```

**Build all modules:**
```bash
mvn clean package -DskipTests
```

---

## 5. Docker Containerization

### 5.1 Why Docker?

Docker containerizes your application with all its dependencies, ensuring it runs the same everywhere - similar to how an APK bundles your Android app with its resources.

**Benefits:**
- **Consistency**: Same environment everywhere (dev, test, prod)
- **Isolation**: Each container is independent
- **Portability**: Run anywhere Docker is installed
- **Scalability**: Easy to scale up/down
- **Resource Efficiency**: Lighter than virtual machines

### 5.2 Basic Dockerfile

**Dockerfile (Simple approach):**
```dockerfile
# Use official OpenJDK runtime as base image
FROM openjdk:17-jdk-slim

# Set working directory
WORKDIR /app

# Copy the JAR file
COPY target/myapp.jar app.jar

# Expose port
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Build and run:**
```bash
# Build the JAR first
mvn clean package -DskipTests

# Build Docker image
docker build -t myapp:latest .

# Run container
docker run -p 8080:8080 myapp:latest

# Run with environment variables
docker run -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DB_URL=jdbc:mysql://host.docker.internal:3306/myapp \
  -e DB_USERNAME=user \
  -e DB_PASSWORD=pass \
  myapp:latest

# Run in detached mode
docker run -d -p 8080:8080 --name myapp-container myapp:latest

# View logs
docker logs -f myapp-container

# Stop container
docker stop myapp-container

# Remove container
docker rm myapp-container
```

### 5.3 Multi-Stage Dockerfile (Optimized)

Build the application inside Docker for better portability and smaller image size.

**Dockerfile (Multi-stage with Maven):**
```dockerfile
# Stage 1: Build stage
FROM maven:3.9-openjdk-17-slim AS build

# Set working directory
WORKDIR /app

# Copy pom.xml and download dependencies (cached layer)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# Stage 2: Runtime stage
FROM openjdk:17-jdk-slim

# Create non-root user for security
RUN groupadd -r spring && useradd -r -g spring spring

# Set working directory
WORKDIR /app

# Copy JAR from build stage
COPY --from=build /app/target/myapp.jar app.jar

# Change ownership
RUN chown -R spring:spring /app

# Switch to non-root user
USER spring:spring

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Run the application
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
```

**Dockerfile (Multi-stage with Gradle):**
```dockerfile
# Stage 1: Build stage
FROM gradle:8-jdk17-alpine AS build

# Set working directory
WORKDIR /app

# Copy gradle files (cached layer)
COPY build.gradle settings.gradle ./
COPY gradle ./gradle

# Download dependencies
RUN gradle dependencies --no-daemon

# Copy source code
COPY src ./src

# Build the application
RUN gradle clean build -x test --no-daemon

# Stage 2: Runtime stage
FROM openjdk:17-jdk-slim

# Create non-root user
RUN groupadd -r spring && useradd -r -g spring spring

WORKDIR /app

# Copy JAR from build stage
COPY --from=build /app/build/libs/myapp.jar app.jar

RUN chown -R spring:spring /app
USER spring:spring

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-jar", "app.jar"]
```

### 5.4 Optimized Dockerfile with Layered JARs

Spring Boot 2.3+ supports layered JARs for better Docker caching.

**Dockerfile (Layered approach):**
```dockerfile
# Build stage
FROM maven:3.9-openjdk-17-slim AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

# Runtime stage
FROM openjdk:17-jdk-slim

# Create non-root user
RUN groupadd -r spring && useradd -r -g spring spring

# Copy layers separately for better caching
ARG DEPENDENCY=/app/target/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

RUN chown -R spring:spring /app
USER spring:spring

EXPOSE 8080

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-cp", "app:app/lib/*", \
  "com.example.myapp.MyApplication"]
```

### 5.5 .dockerignore File

Exclude unnecessary files from Docker build context:

**.dockerignore:**
```
# Build outputs
target/
build/
bin/
out/

# IDE files
.idea/
.vscode/
*.iml
*.ipr
*.iws

# Git
.git/
.gitignore

# Docker
Dockerfile
docker-compose.yml
.dockerignore

# Documentation
README.md
docs/

# Logs
logs/
*.log

# OS files
.DS_Store
Thumbs.db

# Testing
.gradle/
.mvn/
```

### 5.6 Docker Best Practices

**1. Use specific base image versions:**
```dockerfile
# Good
FROM openjdk:17.0.9-jdk-slim

# Bad - unpredictable
FROM openjdk:latest
```

**2. Minimize layers:**
```dockerfile
# Good - Single layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Bad - Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean
```

**3. Use multi-stage builds:**
- Separates build and runtime dependencies
- Results in smaller final images

**4. Run as non-root user:**
```dockerfile
USER spring:spring
```

**5. Add health checks:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
```

**6. Use .dockerignore:**
- Speeds up builds
- Reduces context size

---

## 6. Docker Compose for Multi-Container Apps

### 6.1 Basic Docker Compose

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Spring Boot Application
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_URL=jdbc:mysql://mysql:3306/myapp_db
      - DB_USERNAME=myapp_user
      - DB_PASSWORD=your_password
    depends_on:
      - mysql
    networks:
      - myapp-network
    restart: unless-stopped

  # MySQL Database
  mysql:
    image: mysql:8.0
    container_name: myapp-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp_db
      MYSQL_USER: myapp_user
      MYSQL_PASSWORD: your_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init-scripts:/docker-entrypoint-initdb.d
    networks:
      - myapp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql_data:

networks:
  myapp-network:
    driver: bridge
```

**Commands:**
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose up -d --build

# Scale application instances
docker-compose up -d --scale app=3
```

### 6.2 Full Stack Docker Compose

Complete application with database, cache, and reverse proxy:

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - myapp-network
    restart: unless-stopped

  # Spring Boot Application
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp
    expose:
      - "8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_URL=jdbc:postgresql://postgres:5432/myapp_db
      - DB_USERNAME=myapp_user
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - myapp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: myapp-postgres
    environment:
      POSTGRES_DB: myapp_db
      POSTGRES_USER: myapp_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    networks:
      - myapp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp_user -d myapp_db"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - myapp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # Prometheus Monitoring
  prometheus:
    image: prom/prometheus:latest
    container_name: myapp-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - myapp-network
    restart: unless-stopped

  # Grafana Dashboard
  grafana:
    image: grafana/grafana:latest
    container_name: myapp-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    depends_on:
      - prometheus
    networks:
      - myapp-network
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  prometheus_data:
  grafana_data:

networks:
  myapp-network:
    driver: bridge
```

**Environment file (.env):**
```env
# Database
DB_PASSWORD=your_secure_password_here

# Redis
REDIS_PASSWORD=redis_password_here

# Application
JWT_SECRET=your_jwt_secret_key_here

# Grafana
GRAFANA_PASSWORD=admin
```

**Nginx Configuration (nginx/nginx.conf):**
```nginx
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server app:8080;
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /actuator/health {
            proxy_pass http://backend/actuator/health;
            access_log off;
        }
    }
}
```

### 6.3 Development Docker Compose

With hot reload and debugging:

**docker-compose.dev.yml:**
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: myapp-dev
    ports:
      - "8080:8080"
      - "5005:5005"  # Debug port
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
    volumes:
      - ./src:/app/src:ro
      - ./target:/app/target
      - maven_cache:/root/.m2
    depends_on:
      - mysql
    networks:
      - myapp-network
    command: mvn spring-boot:run

  mysql:
    image: mysql:8.0
    container_name: myapp-mysql-dev
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp_db
    ports:
      - "3306:3306"
    volumes:
      - mysql_dev_data:/var/lib/mysql
    networks:
      - myapp-network

volumes:
  mysql_dev_data:
  maven_cache:

networks:
  myapp-network:
    driver: bridge
```

**Dockerfile.dev:**
```dockerfile
FROM maven:3.9-openjdk-17-slim

WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Expose ports
EXPOSE 8080 5005

# Run with Spring Boot DevTools
CMD ["mvn", "spring-boot:run"]
```

**Usage:**
```bash
# Development
docker-compose -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.yml up -d
```

---

## 7. Cloud Deployment

### 7.1 AWS Deployment

#### 7.1.1 AWS Elastic Beanstalk (Easiest)

**Prerequisites:**
```bash
# Install AWS CLI
brew install awscli  # macOS
# OR
pip install awscli

# Configure AWS credentials
aws configure
# Enter: AWS Access Key ID, Secret Access Key, Region

# Install EB CLI
pip install awsebcli
```

**Deploy to Elastic Beanstalk:**
```bash
# Initialize EB application
eb init -p corretto-17 myapp --region us-east-1

# Create environment and deploy
eb create myapp-prod --database.engine mysql --database.username admin

# Deploy updates
mvn clean package -DskipTests
eb deploy

# Open application in browser
eb open

# View logs
eb logs

# SSH into instance
eb ssh

# Set environment variables
eb setenv SPRING_PROFILES_ACTIVE=prod DB_PASSWORD=secretpass

# Terminate environment
eb terminate myapp-prod
```

**Configuration (.ebextensions/options.config):**
```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    SERVER_PORT: 5000
    SPRING_PROFILES_ACTIVE: prod
  aws:elasticbeanstalk:container:java:
    JvmOptions: "-Xms512m -Xmx2048m"
```

#### 7.1.2 AWS ECS (Container Service)

**1. Build and push Docker image to ECR:**
```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

# Create ECR repository
aws ecr create-repository --repository-name myapp --region us-east-1

# Build image
docker build -t myapp:latest .

# Tag image
docker tag myapp:latest <account-id>.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Push image
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

**2. Create ECS Task Definition (task-definition.json):**
```json
{
  "family": "myapp",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "<account-id>.dkr.ecr.us-east-1.amazonaws.com/myapp:latest",
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "SPRING_PROFILES_ACTIVE",
          "value": "prod"
        }
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/myapp",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -f http://localhost:8080/actuator/health || exit 1"
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3
      }
    }
  ]
}
```

**3. Create and run service:**
```bash
# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Create ECS cluster
aws ecs create-cluster --cluster-name myapp-cluster

# Create service
aws ecs create-service \
  --cluster myapp-cluster \
  --service-name myapp-service \
  --task-definition myapp \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345],securityGroups=[sg-12345],assignPublicIp=ENABLED}"
```

#### 7.1.3 AWS EC2 (Manual Setup)

**1. Launch EC2 instance:**
- AMI: Amazon Linux 2 or Ubuntu
- Instance type: t3.medium or higher
- Security group: Allow ports 22 (SSH), 80 (HTTP), 443 (HTTPS)

**2. Connect and setup:**
```bash
# SSH into instance
ssh -i mykey.pem ec2-user@<public-ip>

# Install Java
sudo yum update -y
sudo yum install java-17-amazon-corretto -y

# Install Docker (optional)
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user

# Create application directory
sudo mkdir -p /opt/myapp
cd /opt/myapp

# Upload JAR (from local machine)
scp -i mykey.pem target/myapp.jar ec2-user@<public-ip>:/opt/myapp/
```

**3. Create systemd service (/etc/systemd/system/myapp.service):**
```ini
[Unit]
Description=My Spring Boot Application
After=syslog.target network.target

[Service]
User=ec2-user
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/java -Xms512m -Xmx2048m -jar /opt/myapp/myapp.jar
SuccessExitStatus=143
Restart=always
RestartSec=10

Environment="SPRING_PROFILES_ACTIVE=prod"
Environment="DB_URL=jdbc:mysql://your-rds-endpoint:3306/myapp"
Environment="DB_USERNAME=admin"
Environment="DB_PASSWORD=secretpass"

[Install]
WantedBy=multi-user.target
```

**4. Start service:**
```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable and start service
sudo systemctl enable myapp
sudo systemctl start myapp

# Check status
sudo systemctl status myapp

# View logs
sudo journalctl -u myapp -f
```

**5. Setup Nginx as reverse proxy:**
```bash
# Install Nginx
sudo yum install nginx -y

# Configure Nginx (/etc/nginx/conf.d/myapp.conf)
sudo tee /etc/nginx/conf.d/myapp.conf <<EOF
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }
}
EOF

# Start Nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 7.2 Google Cloud Platform (GCP)

#### 7.2.1 Google App Engine

**Prerequisites:**
```bash
# Install gcloud CLI
brew install google-cloud-sdk  # macOS

# Initialize and login
gcloud init
gcloud auth login
```

**app.yaml configuration:**
```yaml
runtime: java17
instance_class: F2

env_variables:
  SPRING_PROFILES_ACTIVE: "prod"
  DB_URL: "jdbc:mysql:///<database>?socketFactory=com.google.cloud.sql.mysql.SocketFactory&cloudSqlInstance=<project>:<region>:<instance>"

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.65

resources:
  cpu: 2
  memory_gb: 2

health_check:
  enable_health_check: true
  check_interval_sec: 30
  timeout_sec: 4
  unhealthy_threshold: 2
  healthy_threshold: 2
```

**Deploy:**
```bash
# Build JAR
mvn clean package -DskipTests

# Deploy
gcloud app deploy

# View application
gcloud app browse

# View logs
gcloud app logs tail -s default

# Set environment variables
gcloud app deploy --set-env-vars=DB_PASSWORD=secret
```

#### 7.2.2 Google Kubernetes Engine (GKE)

**1. Build and push to Google Container Registry:**
```bash
# Configure Docker for GCR
gcloud auth configure-docker

# Build image
docker build -t gcr.io/<project-id>/myapp:latest .

# Push image
docker push gcr.io/<project-id>/myapp:latest
```

**2. Create GKE cluster:**
```bash
gcloud container clusters create myapp-cluster \
  --num-nodes=3 \
  --machine-type=n1-standard-2 \
  --region=us-central1

# Get credentials
gcloud container clusters get-credentials myapp-cluster --region=us-central1
```

**3. Create Kubernetes deployment (k8s/deployment.yaml):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: gcr.io/<project-id>/myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

**4. Deploy to GKE:**
```bash
# Create secret for database password
kubectl create secret generic db-secret --from-literal=password=your_password

# Apply deployment
kubectl apply -f k8s/deployment.yaml

# Check status
kubectl get pods
kubectl get services

# View logs
kubectl logs -l app=myapp -f

# Scale deployment
kubectl scale deployment myapp --replicas=5

# Update deployment
docker build -t gcr.io/<project-id>/myapp:v2 .
docker push gcr.io/<project-id>/myapp:v2
kubectl set image deployment/myapp myapp=gcr.io/<project-id>/myapp:v2
```

### 7.3 Microsoft Azure

#### 7.3.1 Azure App Service

**Prerequisites:**
```bash
# Install Azure CLI
brew install azure-cli  # macOS

# Login
az login
```

**Deploy:**
```bash
# Create resource group
az group create --name myapp-rg --location eastus

# Create App Service plan
az appservice plan create \
  --name myapp-plan \
  --resource-group myapp-rg \
  --sku B2 \
  --is-linux

# Create web app
az webapp create \
  --resource-group myapp-rg \
  --plan myapp-plan \
  --name myapp-unique-name \
  --runtime "JAVA:17-java17"

# Deploy JAR
az webapp deploy \
  --resource-group myapp-rg \
  --name myapp-unique-name \
  --src-path target/myapp.jar \
  --type jar

# Configure app settings
az webapp config appsettings set \
  --resource-group myapp-rg \
  --name myapp-unique-name \
  --settings SPRING_PROFILES_ACTIVE=prod DB_PASSWORD=secret

# View logs
az webapp log tail --resource-group myapp-rg --name myapp-unique-name
```

#### 7.3.2 Azure Container Instances

**Deploy Docker container:**
```bash
# Create container registry
az acr create \
  --resource-group myapp-rg \
  --name myappregistry \
  --sku Basic

# Login to registry
az acr login --name myappregistry

# Build and push image
docker build -t myappregistry.azurecr.io/myapp:latest .
docker push myappregistry.azurecr.io/myapp:latest

# Deploy container
az container create \
  --resource-group myapp-rg \
  --name myapp-container \
  --image myappregistry.azurecr.io/myapp:latest \
  --cpu 2 \
  --memory 2 \
  --registry-username <username> \
  --registry-password <password> \
  --dns-name-label myapp-unique \
  --ports 8080 \
  --environment-variables SPRING_PROFILES_ACTIVE=prod

# View logs
az container logs --resource-group myapp-rg --name myapp-container
```

### 7.4 Heroku

**Prerequisites:**
```bash
# Install Heroku CLI
brew install heroku/brew/heroku  # macOS

# Login
heroku login
```

**Method 1: Using Heroku Git:**
```bash
# Create Heroku app
heroku create myapp-unique-name

# Add PostgreSQL
heroku addons:create heroku-postgresql:mini

# Configure environment
heroku config:set SPRING_PROFILES_ACTIVE=prod
heroku config:set JAVA_TOOL_OPTIONS="-Xmx512m -Xms256m"

# Create Procfile
echo "web: java -Dserver.port=\$PORT \$JAVA_OPTS -jar target/myapp.jar" > Procfile

# Deploy
git push heroku main

# Open app
heroku open

# View logs
heroku logs --tail

# Scale dynos
heroku ps:scale web=2
```

**Method 2: Using Heroku Container Registry:**
```bash
# Login to container registry
heroku container:login

# Build and push
heroku container:push web --app myapp-unique-name

# Release
heroku container:release web --app myapp-unique-name
```

**heroku.yml:**
```yaml
build:
  docker:
    web: Dockerfile
run:
  web: java -Dserver.port=$PORT $JAVA_OPTS -jar app.jar
```

---

## 8. Environment Variables and Secrets Management

### 8.1 Environment Variables

**Best Practices:**
1. **Never commit secrets to version control**
2. **Use different secrets for each environment**
3. **Rotate secrets regularly**
4. **Use secrets management services**

**Loading Environment Variables in Spring Boot:**

**application.properties:**
```properties
# Use ${VAR_NAME:default_value} syntax
spring.datasource.url=${DB_URL:jdbc:h2:mem:testdb}
spring.datasource.username=${DB_USERNAME:sa}
spring.datasource.password=${DB_PASSWORD:}

jwt.secret=${JWT_SECRET}
jwt.expiration=${JWT_EXPIRATION:86400000}

# External API keys
stripe.api.key=${STRIPE_API_KEY}
sendgrid.api.key=${SENDGRID_API_KEY}
aws.access.key=${AWS_ACCESS_KEY}
aws.secret.key=${AWS_SECRET_KEY}
```

**Configuration Class:**
```java
@Configuration
public class AppConfig {

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Value("${jwt.expiration:86400000}")  // Default value
    private Long jwtExpiration;

    @Bean
    public JwtTokenProvider jwtTokenProvider() {
        return new JwtTokenProvider(jwtSecret, jwtExpiration);
    }
}
```

**@ConfigurationProperties approach:**
```java
@ConfigurationProperties(prefix = "app")
@Configuration
public class AppProperties {

    private Security security = new Security();
    private Database database = new Database();

    // Getters and setters

    public static class Security {
        private String jwtSecret;
        private Long jwtExpiration;
        // Getters and setters
    }

    public static class Database {
        private String url;
        private String username;
        private String password;
        // Getters and setters
    }
}
```

**application.properties:**
```properties
app.security.jwt-secret=${JWT_SECRET}
app.security.jwt-expiration=86400000
app.database.url=${DB_URL}
app.database.username=${DB_USERNAME}
app.database.password=${DB_PASSWORD}
```

### 8.2 Local Development (.env file)

**Not for production!** Use for local development only.

**Install dependency (pom.xml):**
```xml
<dependency>
    <groupId>me.paulschwarz</groupId>
    <artifactId>spring-dotenv</artifactId>
    <version>4.0.0</version>
</dependency>
```

**.env file:**
```env
# Database
DB_URL=jdbc:mysql://localhost:3306/myapp_db
DB_USERNAME=myapp_user
DB_PASSWORD=localpass123

# JWT
JWT_SECRET=my_super_secret_jwt_key_for_development_only

# External APIs
STRIPE_API_KEY=sk_test_xxxxxxxxxxxxx
SENDGRID_API_KEY=SG.xxxxxxxxxxxxx

# AWS
AWS_ACCESS_KEY=AKIAXXXXXXXXXXXXXXXX
AWS_SECRET_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_REGION=us-east-1
```

**Add to .gitignore:**
```gitignore
.env
.env.local
.env.*.local
```

### 8.3 AWS Secrets Manager

**Add dependency (pom.xml):**
```xml
<dependency>
    <groupId>com.amazonaws.secretsmanager</groupId>
    <artifactId>aws-secretsmanager-jdbc</artifactId>
    <version>1.0.12</version>
</dependency>
```

**Create secret in AWS:**
```bash
aws secretsmanager create-secret \
  --name myapp/prod/db \
  --secret-string '{"username":"admin","password":"secretpass","host":"mydb.123456789.us-east-1.rds.amazonaws.com","port":"3306","dbname":"myapp"}'
```

**Configuration:**
```java
@Configuration
public class AwsSecretsConfig {

    @Value("${aws.secretsmanager.secret-name}")
    private String secretName;

    @Bean
    public DataSource dataSource() {
        AWSSecretsManager client = AWSSecretsManagerClientBuilder
            .standard()
            .withRegion(Regions.US_EAST_1)
            .build();

        GetSecretValueRequest request = new GetSecretValueRequest()
            .withSecretId(secretName);

        GetSecretValueResult result = client.getSecretValue(request);
        String secret = result.getSecretString();

        JSONObject secretJson = new JSONObject(secret);

        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(String.format("jdbc:mysql://%s:%s/%s",
            secretJson.getString("host"),
            secretJson.getString("port"),
            secretJson.getString("dbname")));
        config.setUsername(secretJson.getString("username"));
        config.setPassword(secretJson.getString("password"));

        return new HikariDataSource(config);
    }
}
```

**Or use Spring Cloud AWS:**
```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-secrets-manager</artifactId>
    <version>3.0.0</version>
</dependency>
```

**application.properties:**
```properties
aws.secretsmanager.secret-name=myapp/prod/db
spring.config.import=aws-secretsmanager:myapp/prod/db
```

### 8.4 HashiCorp Vault

**Add dependency (pom.xml):**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-vault-config</artifactId>
</dependency>
```

**bootstrap.properties:**
```properties
spring.application.name=myapp
spring.cloud.vault.host=vault.example.com
spring.cloud.vault.port=8200
spring.cloud.vault.scheme=https
spring.cloud.vault.authentication=TOKEN
spring.cloud.vault.token=${VAULT_TOKEN}

spring.cloud.vault.kv.enabled=true
spring.cloud.vault.kv.backend=secret
spring.cloud.vault.kv.application-name=myapp
```

**Store secrets in Vault:**
```bash
# Write secret
vault kv put secret/myapp \
  db.password=secretpass \
  jwt.secret=jwtsecret \
  stripe.api.key=sk_test_xxx

# Read secret
vault kv get secret/myapp
```

### 8.5 Kubernetes Secrets

**Create secret:**
```bash
# From literal values
kubectl create secret generic app-secrets \
  --from-literal=db-password=secretpass \
  --from-literal=jwt-secret=jwtsecret

# From file
kubectl create secret generic app-secrets \
  --from-file=db-password=./secrets/db-password.txt \
  --from-file=jwt-secret=./secrets/jwt-secret.txt

# From YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db-password: c2VjcmV0cGFzcw==  # base64 encoded
  jwt-secret: and0c2VjcmV0==       # base64 encoded
EOF
```

**Use in deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        env:
        # Individual secrets
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db-password
        # All secrets from a secret
        envFrom:
        - secretRef:
            name: app-secrets
        # Mount as files
        volumeMounts:
        - name: secrets
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: secrets
        secret:
          secretName: app-secrets
```

---

## 9. Monitoring and Logging

### 9.1 Spring Boot Actuator

**Add dependency (pom.xml):**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Configuration (application.properties):**
```properties
# Actuator endpoints
management.endpoints.web.exposure.include=health,info,metrics,prometheus,loggers,env
management.endpoints.web.base-path=/actuator
management.endpoint.health.show-details=when-authorized
management.endpoint.health.probes.enabled=true

# Health indicators
management.health.livenessstate.enabled=true
management.health.readinessstate.enabled=true

# Info endpoint
management.info.env.enabled=true
management.info.java.enabled=true
management.info.os.enabled=true

# Metrics
management.metrics.export.prometheus.enabled=true
management.metrics.distribution.percentiles-histogram.http.server.requests=true
```

**Available endpoints:**
- `GET /actuator` - List all endpoints
- `GET /actuator/health` - Application health status
- `GET /actuator/info` - Application info
- `GET /actuator/metrics` - Application metrics
- `GET /actuator/metrics/{metric}` - Specific metric
- `GET /actuator/env` - Environment properties
- `GET /actuator/loggers` - Logger configuration
- `POST /actuator/loggers/{logger}` - Change log level
- `GET /actuator/prometheus` - Metrics in Prometheus format

**Custom health indicator:**
```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    private final UserRepository userRepository;

    public DatabaseHealthIndicator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public Health health() {
        try {
            long count = userRepository.count();
            return Health.up()
                .withDetail("database", "accessible")
                .withDetail("userCount", count)
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("database", "not accessible")
                .withException(e)
                .build();
        }
    }
}
```

**Custom metrics:**
```java
@Service
public class UserService {

    private final UserRepository userRepository;
    private final Counter userCreationCounter;
    private final Timer userCreationTimer;

    public UserService(
            UserRepository userRepository,
            MeterRegistry meterRegistry) {
        this.userRepository = userRepository;
        this.userCreationCounter = Counter.builder("users.created")
            .description("Number of users created")
            .register(meterRegistry);
        this.userCreationTimer = Timer.builder("users.creation.time")
            .description("Time to create a user")
            .register(meterRegistry);
    }

    public User createUser(UserDto dto) {
        return userCreationTimer.record(() -> {
            User user = new User();
            // ... create user
            User saved = userRepository.save(user);
            userCreationCounter.increment();
            return saved;
        });
    }
}
```

### 9.2 Logging Configuration

**Logback configuration (logback-spring.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

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
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- JSON Appender for structured logging -->
    <appender name="JSON" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.json</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>

    <!-- Development Profile -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
        <logger name="com.example" level="DEBUG"/>
        <logger name="org.springframework.web" level="DEBUG"/>
        <logger name="org.hibernate.SQL" level="DEBUG"/>
    </springProfile>

    <!-- Production Profile -->
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="FILE"/>
            <appender-ref ref="JSON"/>
        </root>
        <logger name="com.example" level="INFO"/>
        <logger name="org.springframework" level="WARN"/>
    </springProfile>
</configuration>
```

**Add JSON logging dependency (pom.xml):**
```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

**Usage in code:**
```java
@RestController
@Slf4j  // Lombok annotation
public class UserController {

    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        log.info("Fetching user with id: {}", id);

        try {
            User user = userService.findById(id);
            log.debug("User found: {}", user);
            return ResponseEntity.ok(user);
        } catch (UserNotFoundException e) {
            log.error("User not found: {}", id, e);
            return ResponseEntity.notFound().build();
        } catch (Exception e) {
            log.error("Error fetching user: {}", id, e);
            return ResponseEntity.status(500).build();
        }
    }
}
```

### 9.3 Prometheus Integration

**Add dependency (pom.xml):**
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

**Configuration (application.properties):**
```properties
management.metrics.export.prometheus.enabled=true
management.endpoint.prometheus.enabled=true
management.endpoints.web.exposure.include=prometheus
```

**Prometheus configuration (prometheus/prometheus.yml):**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['app:8080']
        labels:
          application: 'myapp'
          environment: 'production'

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'
```

**Alert rules (prometheus/alerts.yml):**
```yaml
groups:
  - name: application_alerts
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_server_requests_seconds_count{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} requests/sec"

      - alert: HighMemoryUsage
        expr: jvm_memory_used_bytes / jvm_memory_max_bytes > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value | humanizePercentage }}"

      - alert: ApplicationDown
        expr: up{job="spring-boot-app"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Application is down"
          description: "Application has been down for more than 1 minute"
```

### 9.4 Grafana Dashboards

**Grafana provisioning (grafana/provisioning/datasources/prometheus.yml):**
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

**Dashboard configuration (grafana/provisioning/dashboards/dashboard.yml):**
```yaml
apiVersion: 1

providers:
  - name: 'Spring Boot'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

**Import existing dashboards:**
1. Open Grafana (http://localhost:3000)
2. Go to Dashboards → Import
3. Import dashboard IDs:
   - **4701** - JVM (Micrometer)
   - **11378** - Spring Boot 2.1 Statistics
   - **12856** - Spring Boot APM Dashboard

**Custom Grafana dashboard JSON (grafana/provisioning/dashboards/myapp.json):**
```json
{
  "dashboard": {
    "title": "MyApp Dashboard",
    "panels": [
      {
        "title": "HTTP Request Rate",
        "targets": [
          {
            "expr": "rate(http_server_requests_seconds_count[5m])"
          }
        ],
        "type": "graph"
      },
      {
        "title": "HTTP Error Rate",
        "targets": [
          {
            "expr": "rate(http_server_requests_seconds_count{status=~\"5..\"}[5m])"
          }
        ],
        "type": "graph"
      },
      {
        "title": "JVM Memory Used",
        "targets": [
          {
            "expr": "jvm_memory_used_bytes"
          }
        ],
        "type": "graph"
      },
      {
        "title": "CPU Usage",
        "targets": [
          {
            "expr": "system_cpu_usage"
          }
        ],
        "type": "gauge"
      }
    ]
  }
}
```

### 9.5 ELK Stack (Elasticsearch, Logstash, Kibana)

**Add dependency (pom.xml):**
```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

**Logback configuration for Logstash:**
```xml
<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>logstash:5000</destination>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"app":"myapp","environment":"production"}</customFields>
    </encoder>
</appender>
```

**Docker Compose with ELK:**
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.10.0
    container_name: logstash
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - "5000:5000"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.10.0
    container_name: kibana
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch_data:
```

**Logstash pipeline (logstash/pipeline/logstash.conf):**
```conf
input {
  tcp {
    port => 5000
    codec => json
  }
}

filter {
  if [level] == "ERROR" {
    mutate {
      add_tag => ["error"]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "myapp-logs-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => rubydebug
  }
}
```

---

## 10. CI/CD with GitHub Actions

### 10.1 Basic Build and Test

**.github/workflows/build.yml:**
```yaml
name: Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: 'maven'

    - name: Build with Maven
      run: mvn clean install -DskipTests

    - name: Run tests
      run: mvn test

    - name: Generate test coverage report
      run: mvn jacoco:report

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./target/site/jacoco/jacoco.xml
```

### 10.2 Docker Build and Push

**.github/workflows/docker.yml:**
```yaml
name: Docker Build and Push

on:
  push:
    branches: [ main ]
    tags:
      - 'v*'

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: 'maven'

    - name: Build JAR
      run: mvn clean package -DskipTests

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### 10.3 Deploy to AWS

**.github/workflows/deploy-aws.yml:**
```yaml
name: Deploy to AWS

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: 'maven'

    - name: Build JAR
      run: mvn clean package -DskipTests

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build and push Docker image to ECR
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: myapp
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster myapp-cluster \
          --service myapp-service \
          --force-new-deployment

    # Alternative: Deploy to Elastic Beanstalk
    # - name: Deploy to Elastic Beanstalk
    #   uses: einaregilsson/beanstalk-deploy@v21
    #   with:
    #     aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
    #     aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    #     application_name: myapp
    #     environment_name: myapp-prod
    #     version_label: ${{ github.sha }}
    #     region: us-east-1
    #     deployment_package: target/myapp.jar
```

### 10.4 Deploy to Heroku

**.github/workflows/deploy-heroku.yml:**
```yaml
name: Deploy to Heroku

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Deploy to Heroku
      uses: akhileshns/heroku-deploy@v3.13.15
      with:
        heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
        heroku_app_name: myapp-unique-name
        heroku_email: your-email@example.com

    # Alternative: Docker deployment
    # - name: Login to Heroku Container Registry
    #   env:
    #     HEROKU_API_KEY: ${{ secrets.HEROKU_API_KEY }}
    #   run: heroku container:login

    # - name: Build and push Docker image
    #   env:
    #     HEROKU_API_KEY: ${{ secrets.HEROKU_API_KEY }}
    #   run: |
    #     heroku container:push web --app myapp-unique-name
    #     heroku container:release web --app myapp-unique-name
```

### 10.5 Deploy to Kubernetes

**.github/workflows/deploy-k8s.yml:**
```yaml
name: Deploy to Kubernetes

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: 'maven'

    - name: Build JAR
      run: mvn clean package -DskipTests

    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'latest'

    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig.yaml
        export KUBECONFIG=kubeconfig.yaml

    - name: Login to Google Container Registry
      run: |
        echo "${{ secrets.GCP_SA_KEY }}" | base64 -d > gcp-key.json
        gcloud auth activate-service-account --key-file gcp-key.json
        gcloud auth configure-docker

    - name: Build and push Docker image
      env:
        PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
      run: |
        docker build -t gcr.io/$PROJECT_ID/myapp:${{ github.sha }} .
        docker push gcr.io/$PROJECT_ID/myapp:${{ github.sha }}

    - name: Deploy to Kubernetes
      env:
        PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
      run: |
        kubectl set image deployment/myapp myapp=gcr.io/$PROJECT_ID/myapp:${{ github.sha }}
        kubectl rollout status deployment/myapp
```

### 10.6 Complete CI/CD Pipeline

**.github/workflows/cicd.yml:**
```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  JAVA_VERSION: '17'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Job 1: Code Quality Checks
  code-quality:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK
      uses: actions/setup-java@v4
      with:
        java-version: ${{ env.JAVA_VERSION }}
        distribution: 'temurin'
        cache: 'maven'

    - name: Run Checkstyle
      run: mvn checkstyle:check

    - name: Run PMD
      run: mvn pmd:check

    - name: Run SpotBugs
      run: mvn spotbugs:check

  # Job 2: Build and Test
  build-and-test:
    runs-on: ubuntu-latest
    needs: code-quality

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: testdb
        ports:
          - 3306:3306
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3

    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK
      uses: actions/setup-java@v4
      with:
        java-version: ${{ env.JAVA_VERSION }}
        distribution: 'temurin'
        cache: 'maven'

    - name: Build with Maven
      run: mvn clean install -DskipTests

    - name: Run unit tests
      run: mvn test

    - name: Run integration tests
      run: mvn verify -P integration-tests

    - name: Generate coverage report
      run: mvn jacoco:report

    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./target/site/jacoco/jacoco.xml

    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: jar-file
        path: target/*.jar

  # Job 3: Security Scan
  security-scan:
    runs-on: ubuntu-latest
    needs: build-and-test
    steps:
    - uses: actions/checkout@v4

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

  # Job 4: Build Docker Image
  build-docker:
    runs-on: ubuntu-latest
    needs: [build-and-test, security-scan]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v4

    - name: Download artifacts
      uses: actions/download-artifact@v3
      with:
        name: jar-file
        path: target/

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}

  # Job 5: Deploy to Staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: build-docker
    environment:
      name: staging
      url: https://staging.myapp.com

    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying to staging..."
        # Add your deployment commands here

  # Job 6: Deploy to Production (with approval)
  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production
      url: https://myapp.com

    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # Add your deployment commands here

    - name: Notify deployment
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: 'Deployment to production completed!'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 11. Performance Tuning

### 11.1 JVM Tuning

**JVM Options:**
```bash
java \
  -Xms512m \                          # Initial heap size
  -Xmx2048m \                         # Maximum heap size
  -XX:+UseG1GC \                      # Use G1 Garbage Collector
  -XX:MaxGCPauseMillis=200 \          # Maximum GC pause time
  -XX:+UseStringDeduplication \       # Deduplicate strings
  -XX:+OptimizeStringConcat \         # Optimize string concatenation
  -XX:+UseCompressedOops \            # Compressed object pointers
  -XX:+HeapDumpOnOutOfMemoryError \   # Dump heap on OOM
  -XX:HeapDumpPath=/var/log/myapp \   # Heap dump location
  -Djava.security.egd=file:/dev/./urandom \  # Faster startup
  -jar myapp.jar
```

**For containers:**
```bash
java \
  -XX:+UseContainerSupport \          # Container-aware JVM
  -XX:MaxRAMPercentage=75.0 \         # Use 75% of container memory
  -XX:InitialRAMPercentage=50.0 \     # Initial memory allocation
  -XX:MinRAMPercentage=50.0 \         # Minimum memory percentage
  -jar myapp.jar
```

**Dockerfile with optimized JVM:**
```dockerfile
FROM openjdk:17-jdk-slim

ENV JAVA_OPTS="-XX:+UseContainerSupport \
  -XX:MaxRAMPercentage=75.0 \
  -XX:+UseG1GC \
  -XX:+UseStringDeduplication \
  -Djava.security.egd=file:/dev/./urandom"

COPY target/myapp.jar app.jar

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### 11.2 Database Connection Pooling

**HikariCP configuration (application.properties):**
```properties
# HikariCP is the default in Spring Boot
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.leak-detection-threshold=60000

# Additional optimizations
spring.datasource.hikari.connection-test-query=SELECT 1
spring.datasource.hikari.pool-name=MyAppHikariPool
```

**Configuration class:**
```java
@Configuration
public class DatabaseConfig {

    @Bean
    public HikariConfig hikariConfig() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(env.getProperty("spring.datasource.url"));
        config.setUsername(env.getProperty("spring.datasource.username"));
        config.setPassword(env.getProperty("spring.datasource.password"));

        // Pool size based on formula: connections = ((core_count * 2) + effective_spindle_count)
        config.setMaximumPoolSize(10);
        config.setMinimumIdle(5);

        // Connection timeout
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);

        // Performance
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        config.addDataSourceProperty("useServerPrepStmts", "true");

        return config;
    }

    @Bean
    public DataSource dataSource(HikariConfig hikariConfig) {
        return new HikariDataSource(hikariConfig);
    }
}
```

### 11.3 Caching

**Add Redis caching:**

**Dependencies (pom.xml):**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**Configuration:**
```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(60))
            .disableCachingNullValues()
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();

        // Different TTL for different caches
        cacheConfigurations.put("users",
            RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(30)));
        cacheConfigurations.put("products",
            RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofHours(1)));

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(cacheConfiguration())
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
}
```

**Usage:**
```java
@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        // This will be cached
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    @CachePut(value = "users", key = "#user.id")
    public User update(User user) {
        // Updates cache
        return userRepository.save(user);
    }

    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) {
        // Removes from cache
        userRepository.deleteById(id);
    }

    @CacheEvict(value = "users", allEntries = true)
    public void deleteAll() {
        // Clears entire cache
        userRepository.deleteAll();
    }
}
```

### 11.4 Async Processing

**Enable async:**
```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```

**Usage:**
```java
@Service
public class EmailService {

    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject, String body) {
        // This runs asynchronously
        // ... send email logic
        return CompletableFuture.completedFuture(null);
    }
}

@RestController
public class UserController {

    private final EmailService emailService;

    @PostMapping("/users")
    public ResponseEntity<User> createUser(@RequestBody UserDto dto) {
        User user = userService.create(dto);

        // Send email asynchronously (non-blocking)
        emailService.sendEmail(user.getEmail(), "Welcome", "Welcome to our app!");

        return ResponseEntity.ok(user);
    }
}
```

### 11.5 Database Query Optimization

**Use projections:**
```java
// Instead of fetching entire entity
public interface UserProjection {
    Long getId();
    String getName();
    String getEmail();
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserProjection> findAllProjectedBy();
}
```

**Use @Query with specific fields:**
```java
@Query("SELECT new com.example.dto.UserDto(u.id, u.name, u.email) FROM User u")
List<UserDto> findAllUsers();
```

**Batch operations:**
```java
@Transactional
public void createUsers(List<User> users) {
    int batchSize = 50;
    for (int i = 0; i < users.size(); i++) {
        userRepository.save(users.get(i));
        if (i % batchSize == 0 && i > 0) {
            entityManager.flush();
            entityManager.clear();
        }
    }
}
```

**application.properties:**
```properties
# Batch processing
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true

# Query optimization
spring.jpa.properties.hibernate.query.plan_cache_max_size=2048
spring.jpa.properties.hibernate.query.plan_parameter_metadata_max_size=128

# Connection optimization
spring.jpa.open-in-view=false
```

---

## 12. Production Best Practices

### 12.1 Health Checks

**Kubernetes liveness and readiness probes:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

**Configuration:**
```properties
management.endpoint.health.probes.enabled=true
management.health.livenessstate.enabled=true
management.health.readinessstate.enabled=true
```

### 12.2 Graceful Shutdown

**application.properties:**
```properties
# Graceful shutdown with 30 second timeout
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=30s
```

**Kubernetes configuration:**
```yaml
spec:
  containers:
  - name: myapp
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 10"]
  terminationGracePeriodSeconds: 40
```

### 12.3 Rate Limiting

**Using Bucket4j:**

**Dependency (pom.xml):**
```xml
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-core</artifactId>
    <version>8.7.0</version>
</dependency>
```

**Implementation:**
```java
@Component
public class RateLimitFilter implements Filter {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String ip = httpRequest.getRemoteAddr();

        Bucket bucket = cache.computeIfAbsent(ip, k -> createBucket());

        if (bucket.tryConsume(1)) {
            chain.doFilter(request, response);
        } else {
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setStatus(429);
            httpResponse.getWriter().write("Too many requests");
        }
    }

    private Bucket createBucket() {
        // 100 requests per minute
        Bandwidth limit = Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(1)));
        return Bucket.builder()
            .addLimit(limit)
            .build();
    }
}
```

### 12.4 Security Headers

**Configuration:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers(headers -> headers
                .contentSecurityPolicy(csp ->
                    csp.policyDirectives("default-src 'self'"))
                .frameOptions(frame ->
                    frame.deny())
                .xssProtection(xss ->
                    xss.headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK))
                .contentTypeOptions(Customizer.withDefaults())
                .referrerPolicy(referrer ->
                    referrer.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN))
                .permissionsPolicy(permissions ->
                    permissions.policy("geolocation=(), microphone=(), camera=()"))
            );

        return http.build();
    }
}
```

### 12.5 Request/Response Logging

**Logbook configuration:**

**Dependency (pom.xml):**
```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>logbook-spring-boot-starter</artifactId>
    <version>3.5.0</version>
</dependency>
```

**application.properties:**
```properties
logbook.format.style=http
logbook.exclude=/actuator/**
logbook.obfuscate.headers=Authorization,X-Secret
logbook.obfuscate.parameters=password,secret
```

### 12.6 Database Migration with Flyway

**Dependency (pom.xml):**
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

**Configuration:**
```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true
spring.flyway.validate-on-migrate=true
```

**Migration file structure:**
```
src/main/resources/
└── db/
    └── migration/
        ├── V1__create_users_table.sql
        ├── V2__add_email_to_users.sql
        └── V3__create_products_table.sql
```

**V1__create_users_table.sql:**
```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
```

### 12.7 Production Checklist

**Before deploying to production:**

- [ ] Environment-specific configuration files
- [ ] Secrets stored securely (not in code)
- [ ] Database migrations tested
- [ ] Health check endpoints configured
- [ ] Logging configured (with log rotation)
- [ ] Monitoring and alerting set up
- [ ] Error tracking (Sentry, Rollbar)
- [ ] Rate limiting implemented
- [ ] HTTPS/SSL configured
- [ ] Security headers configured
- [ ] CORS configured properly
- [ ] Backup strategy in place
- [ ] Disaster recovery plan
- [ ] Load testing performed
- [ ] Performance benchmarks established
- [ ] Documentation updated
- [ ] CI/CD pipeline tested
- [ ] Rollback procedure documented

---

## Summary

This guide covered comprehensive server setup and deployment for Spring Boot applications:

1. **Local Development**: Setting up Java, IDE, and creating Spring Boot projects
2. **Database Setup**: MySQL, PostgreSQL, H2, and Docker configurations
3. **Configuration Management**: Spring profiles for different environments
4. **Building JARs**: Maven and Gradle build processes
5. **Docker**: Containerization with optimized Dockerfiles
6. **Docker Compose**: Multi-container application orchestration
7. **Cloud Deployment**: AWS, GCP, Azure, and Heroku deployment strategies
8. **Secrets Management**: Environment variables and cloud secret services
9. **Monitoring**: Actuator, Prometheus, Grafana, and ELK stack
10. **CI/CD**: GitHub Actions workflows for automated deployment
11. **Performance**: JVM tuning, caching, and database optimization
12. **Best Practices**: Health checks, security, and production readiness

**Key Takeaways:**
- Use Docker for consistent environments across development and production
- Implement proper monitoring and logging from the start
- Automate deployments with CI/CD pipelines
- Never commit secrets to version control
- Test your deployment process in staging before production
- Monitor application performance and set up alerts
- Have a rollback strategy ready

**Next Steps:**
- Set up your local development environment
- Create a Dockerfile for your application
- Configure CI/CD with GitHub Actions
- Deploy to a cloud platform
- Set up monitoring and logging
- Implement security best practices

You now have a complete roadmap from local development to production deployment!
