# Learning Guides for Kotlin Android Developers

Comprehensive guides to help Kotlin Android developers transition to Flutter/Dart and Spring/Java development.

## 🎯 Overview

This repository contains two comprehensive learning guides specifically designed for Android developers with Kotlin experience:

1. **Flutter & Dart Guide** - Transition to mobile cross-platform development
2. **Spring & Java Guide** - Transition to backend development

Both guides draw parallels between Android/Kotlin concepts and their equivalents to accelerate your learning.

---

## 📱 Flutter & Dart Learning Guide

### 📚 Table of Contents

1. [Dart for Kotlin Developers](./01-dart-for-kotlin-developers.md)
   - Syntax comparison
   - Language features
   - Collections and data structures
   - Null safety
   - Async programming

2. [Project Structure](./02-project-structure.md)
   - Flutter project anatomy
   - Directory organization
   - Android vs Flutter comparison
   - Configuration files

3. [App Lifecycle](./03-app-lifecycle.md)
   - Widget lifecycle
   - App state lifecycle
   - Comparison with Android Activity/Fragment lifecycle

4. [Flutter UI Components](./04-flutter-ui-components.md)
   - Widget tree concept
   - StatelessWidget vs StatefulWidget
   - Major widgets (Scaffold, Container, Column, Row, etc.)
   - Layout system
   - Material Design vs Cupertino widgets

5. [Popular Libraries & Packages](./05-popular-libraries.md)
   - Essential packages
   - Community favorites
   - Platform-specific integrations

6. [Network Layer](./06-network-layer.md)
   - HTTP clients (dio, http)
   - REST API integration
   - Serialization/Deserialization
   - Error handling
   - Interceptors and middleware

7. [State Management](./07-state-management.md)
   - setState (basic)
   - Provider
   - Riverpod
   - BLoC
   - GetX
   - MobX
   - Comparison matrix and when to use each

8. [Navigation & Routing](./08-navigation-routing.md)
   - Navigator 1.0 vs 2.0
   - Route management
   - Deep linking
   - Comparison with Android Navigation Component

9. [Best Practices & Tips](./09-best-practices.md)
   - Performance optimization
   - Code organization
   - Testing strategies
   - Common pitfalls

## 🚀 Getting Started

### Prerequisites
- Basic understanding of Kotlin and Android development
- Flutter SDK installed
- IDE setup (Android Studio with Flutter plugin or VS Code)

### Installation
```bash
# Verify Flutter installation
flutter doctor

# Create a new Flutter project
flutter create my_first_app

# Run the app
cd my_first_app
flutter run
```

## 🎓 Learning Path

We recommend following the sections in order, especially if you're new to Flutter:

**Week 1**: Sections 1-3 (Dart basics, project structure, lifecycle)
**Week 2**: Sections 4-5 (UI components, libraries)
**Week 3**: Sections 6-7 (Network layer, state management)
**Week 4**: Sections 8-9 (Navigation, best practices)

## 📖 How to Use This Guide

Each section contains:
- **Concept Overview**: High-level explanation
- **Kotlin/Android Comparison**: How it relates to what you already know
- **Code Examples**: Practical, working examples
- **Best Practices**: Industry-standard approaches
- **Common Pitfalls**: What to avoid

### 🔗 Flutter Additional Resources

- [Official Flutter Documentation](https://flutter.dev/docs)
- [Dart Language Tour](https://dart.dev/guides/language/language-tour)
- [Flutter Widget Catalog](https://flutter.dev/docs/development/ui/widgets)
- [Pub.dev - Package Repository](https://pub.dev/)

---

## ☕ Spring Framework & Java Learning Guide

### 📚 Table of Contents

1. [Java Fundamentals for Kotlin Developers](./spring-java/01-java-for-kotlin-developers.md)
   - Syntax comparison
   - Language features and differences
   - Collections and data structures
   - Null handling
   - Async programming (CompletableFuture vs Coroutines)

2. [Spring Core & Dependency Injection](./spring-java/02-spring-core-di.md)
   - What is Spring Framework
   - Inversion of Control (IoC) Container
   - Dependency Injection patterns
   - Bean lifecycle
   - Comparison with Dagger/Hilt

3. [Spring Boot Fundamentals](./spring-java/03-spring-boot-fundamentals.md)
   - Spring Boot overview
   - Auto-configuration magic
   - Project structure and setup
   - Application properties and profiles
   - Starters and dependencies

4. [Building REST APIs with Spring MVC](./spring-java/04-rest-api-spring-mvc.md)
   - Spring MVC architecture
   - Controllers and request mapping
   - Request/Response handling
   - Validation and error handling
   - Comparison with Retrofit/Ktor Server

5. [Spring Data JPA](./spring-java/05-spring-data-jpa.md)
   - JPA and Hibernate basics
   - Entity mapping
   - Repository pattern
   - Query methods and JPQL
   - Comparison with Room Database

6. [Spring Security](./spring-java/06-spring-security.md)
   - Authentication and Authorization
   - Security configuration
   - JWT implementation
   - OAuth2 integration
   - Best practices

7. [Major Spring Ecosystem Libraries](./spring-java/07-major-libraries.md)
   - Spring Data (MongoDB, Redis, etc.)
   - Spring Cloud (Microservices)
   - Spring Batch
   - Spring WebFlux (Reactive)
   - Testing libraries

8. [Server Setup & Deployment](./spring-java/08-server-setup-deployment.md)
   - Local development setup
   - Database configuration
   - Docker containerization
   - Cloud deployment (AWS, GCP, Azure)
   - Monitoring and logging

9. [Best Practices & Architecture](./spring-java/09-best-practices.md)
   - Clean architecture in Spring
   - SOLID principles
   - Design patterns
   - Testing strategies
   - Performance optimization

### 🚀 Getting Started with Spring

#### Prerequisites
- Basic understanding of Kotlin and Android development
- JDK 17 or later installed
- IDE setup (IntelliJ IDEA or VS Code)
- Maven or Gradle knowledge

#### Quick Start
```bash
# Verify Java installation
java -version

# Create Spring Boot project using Spring Initializr
curl https://start.spring.io/starter.zip \
  -d dependencies=web,data-jpa,h2 \
  -d type=gradle-project \
  -d language=java \
  -d bootVersion=3.2.0 \
  -d baseDir=my-spring-app \
  -o my-spring-app.zip

# Extract and run
unzip my-spring-app.zip
cd my-spring-app
./gradlew bootRun

# Your server will start at http://localhost:8080
```

### 🎓 Spring Learning Path

We recommend following the sections in order:

**Week 1**: Sections 1-2 (Java basics, Spring Core & DI)
**Week 2**: Sections 3-4 (Spring Boot, REST APIs)
**Week 3**: Sections 5-6 (Spring Data JPA, Security)
**Week 4**: Sections 7-9 (Libraries, Deployment, Best Practices)

### 🔗 Spring Additional Resources

- [Spring Framework Documentation](https://spring.io/projects/spring-framework)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Guides](https://spring.io/guides)
- [Baeldung Spring Tutorials](https://www.baeldung.com/spring-tutorial)

**Ready to start backend development?** Begin with [Java for Kotlin Developers](./spring-java/01-java-for-kotlin-developers.md)

---

## 🤝 Contributing

Feel free to contribute to this guide by opening issues or pull requests!

## 📄 License

MIT License - Feel free to use this guide for learning and teaching purposes.

---

**Ready to start?** Begin with [Section 1: Dart for Kotlin Developers](./01-dart-for-kotlin-developers.md)
