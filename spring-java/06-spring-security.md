# Spring Security - Complete Guide

Learn how to secure your Spring Boot applications with authentication, authorization, JWT, and OAuth2. Perfect for Android developers familiar with Firebase Auth, Room encryption, and OAuth flows.

## 🎯 Overview

Spring Security is a powerful authentication and access control framework. If you've used **Firebase Authentication**, **Android Keystore**, or **OAuth2** in Android apps, you'll find Spring Security familiar but more comprehensive.

## 📋 Quick Comparison to Android Security

| Concept | Android | Spring Security |
|---------|---------|-----------------|
| Authentication | Firebase Auth, AccountManager | Spring Security Authentication |
| Authorization | Permission checks | @PreAuthorize, @Secured |
| Token Storage | SharedPreferences (encrypted) | JWT in headers/cookies |
| OAuth2 | Google Sign-In SDK | Spring OAuth2 Client |
| Password Storage | Never do this! | BCrypt/Argon2 hashing |
| Session | ActivityLifecycle | HTTP Session/JWT |
| Secure Storage | Android Keystore | Database with encryption |
| Role-Based Access | Manual checks | Built-in RBAC |

## 1. Authentication vs Authorization

### Authentication (Who are you?)
Authentication verifies the identity of a user - like checking someone's ID card.

```
User: "I'm John Doe"
System: "Prove it - show me your password/token"
User: *provides credentials*
System: "Yes, you are John Doe"
```

### Authorization (What can you do?)
Authorization determines what an authenticated user can access - like checking if someone has a VIP pass.

```
User: "I want to delete this user"
System: "You're authenticated as John, but do you have ADMIN role?"
User: "Yes, I'm an admin"
System: "OK, you can delete users"
```

### Android Analogy

```kotlin
// Authentication - Firebase
FirebaseAuth.getInstance().signInWithEmailAndPassword(email, password)
    .addOnSuccessListener {
        // User authenticated
    }

// Authorization - Manual checks
fun deleteUser(userId: String) {
    val currentUser = FirebaseAuth.getInstance().currentUser
    if (currentUser?.getIdToken()?.claims?.get("admin") == true) {
        // Authorized to delete
    } else {
        throw UnauthorizedException()
    }
}
```

## 2. Security Configuration Basics

### Dependencies (pom.xml)

```xml
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JWT Support -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.12.3</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.12.3</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.12.3</version>
        <scope>runtime</scope>
    </dependency>

    <!-- For OAuth2 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-client</artifactId>
    </dependency>
</dependencies>
```

### Basic Security Configuration

```java
package com.example.myapp.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()  // Public endpoints
                .requestMatchers("/admin/**").hasRole("ADMIN")  // Admin only
                .anyRequest().authenticated()  // Everything else requires auth
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .logout(logout -> logout
                .permitAll()
            );

        return http.build();
    }
}
```

### Security Flow Diagram

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ 1. Request to /api/users
       ▼
┌─────────────────────────────┐
│  Security Filter Chain      │
│  ┌─────────────────────┐   │
│  │ Authentication      │   │ 2. Check if authenticated
│  │ Filter              │   │
│  └──────────┬──────────┘   │
│             │               │
│             ▼               │
│  ┌─────────────────────┐   │
│  │ Authorization       │   │ 3. Check permissions
│  │ Filter              │   │
│  └──────────┬──────────┘   │
└─────────────┼──────────────┘
              │
              ▼ 4. If authorized
       ┌─────────────┐
       │ Controller  │
       └─────────────┘
```

## 3. In-Memory Authentication

Perfect for development and testing.

```java
package com.example.myapp.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class InMemorySecurityConfig {

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder.encode("password123"))
            .roles("USER")
            .build();

        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder.encode("admin123"))
            .roles("USER", "ADMIN")
            .build();

        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(basic -> {})  // Enable HTTP Basic Auth
            .csrf(csrf -> csrf.disable());  // Disable CSRF for API

        return http.build();
    }
}
```

**Test with curl:**

```bash
# User endpoint - success
curl -u user:password123 http://localhost:8080/api/users

# Admin endpoint with user credentials - 403 Forbidden
curl -u user:password123 http://localhost:8080/api/admin/users

# Admin endpoint with admin credentials - success
curl -u admin:admin123 http://localhost:8080/api/admin/users
```

## 4. Database Authentication with UserDetailsService

### User Entity

```java
package com.example.myapp.model;

import jakarta.persistence.*;
import lombok.Data;
import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "users")
@Data
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private boolean enabled = true;

    @Column(nullable = false)
    private boolean accountNonExpired = true;

    @Column(nullable = false)
    private boolean accountNonLocked = true;

    @Column(nullable = false)
    private boolean credentialsNonExpired = true;

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

### Role Entity

```java
package com.example.myapp.model;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "roles")
@Data
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String name;  // ROLE_USER, ROLE_ADMIN, etc.

    private String description;
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
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}
```

### Custom UserDetailsService

```java
package com.example.myapp.security;

import com.example.myapp.model.User;
import com.example.myapp.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Set;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    @Transactional
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() ->
                new UsernameNotFoundException("User not found: " + username));

        Set<GrantedAuthority> authorities = user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority(role.getName()))
            .collect(Collectors.toSet());

        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getUsername())
            .password(user.getPassword())
            .authorities(authorities)
            .accountExpired(!user.isAccountNonExpired())
            .accountLocked(!user.isAccountNonLocked())
            .credentialsExpired(!user.isCredentialsNonExpired())
            .disabled(!user.isEnabled())
            .build();
    }
}
```

### Security Configuration with Database Auth

```java
package com.example.myapp.config;

import com.example.myapp.security.CustomUserDetailsService;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@RequiredArgsConstructor
public class DatabaseSecurityConfig {

    private final CustomUserDetailsService userDetailsService;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);  // Strength factor
    }

    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        authProvider.setUserDetailsService(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authenticationProvider(authenticationProvider())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**", "/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .csrf(csrf -> csrf.disable());

        return http.build();
    }
}
```

## 5. JWT (JSON Web Token) Implementation

### JWT Structure

```
JWT = Header.Payload.Signature

Header:    {"alg": "HS512", "typ": "JWT"}
Payload:   {"sub": "user123", "roles": ["ROLE_USER"], "exp": 1234567890}
Signature: HMACSHA512(base64(header) + "." + base64(payload), secret)

Example JWT:
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyMTIzIiwicm9sZXMiOlsiUk9MRV9VU0VSIl0sImV4cCI6MTIzNDU2Nzg5MH0.dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

### JWT Authentication Flow

```
┌─────────┐                                  ┌─────────┐
│ Client  │                                  │  Server │
└────┬────┘                                  └────┬────┘
     │                                            │
     │  1. POST /api/auth/login                  │
     │     {username, password}                  │
     ├──────────────────────────────────────────>│
     │                                            │
     │                                   2. Validate credentials
     │                                      against database
     │                                            │
     │  3. Return JWT token                      │
     │     {token: "eyJhbG..."}                   │
     │<──────────────────────────────────────────┤
     │                                            │
     │  4. Store token (memory/storage)          │
     │                                            │
     │  5. API Request with token                │
     │     Authorization: Bearer eyJhbG...       │
     ├──────────────────────────────────────────>│
     │                                            │
     │                                   6. Validate JWT
     │                                      - Verify signature
     │                                      - Check expiration
     │                                      - Extract user info
     │                                            │
     │  7. Return protected resource             │
     │<──────────────────────────────────────────┤
     │                                            │
```

### JWT Configuration Properties

```properties
# application.properties
jwt.secret=your-256-bit-secret-key-change-this-in-production
jwt.expiration=86400000
jwt.refresh-expiration=604800000
```

### JWT Utility Class

```java
package com.example.myapp.security.jwt;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import io.jsonwebtoken.security.SignatureException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.stream.Collectors;

@Component
@Slf4j
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Value("${jwt.expiration}")
    private long jwtExpirationMs;

    private SecretKey getSigningKey() {
        byte[] keyBytes = jwtSecret.getBytes(StandardCharsets.UTF_8);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    public String generateToken(Authentication authentication) {
        UserDetails userPrincipal = (UserDetails) authentication.getPrincipal();

        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userPrincipal.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));

        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpirationMs);

        return Jwts.builder()
            .subject(userPrincipal.getUsername())
            .claims(claims)
            .issuedAt(now)
            .expiration(expiryDate)
            .signWith(getSigningKey(), Jwts.SIG.HS512)
            .compact();
    }

    public String generateTokenFromUsername(String username) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpirationMs);

        return Jwts.builder()
            .subject(username)
            .issuedAt(now)
            .expiration(expiryDate)
            .signWith(getSigningKey(), Jwts.SIG.HS512)
            .compact();
    }

    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parser()
            .verifyWith(getSigningKey())
            .build()
            .parseSignedClaims(token)
            .getPayload();

        return claims.getSubject();
    }

    public boolean validateToken(String authToken) {
        try {
            Jwts.parser()
                .verifyWith(getSigningKey())
                .build()
                .parseSignedClaims(authToken);
            return true;
        } catch (SignatureException e) {
            log.error("Invalid JWT signature: {}", e.getMessage());
        } catch (MalformedJwtException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            log.error("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            log.error("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            log.error("JWT claims string is empty: {}", e.getMessage());
        }
        return false;
    }
}
```

### JWT Authentication Filter

```java
package com.example.myapp.security.jwt;

import com.example.myapp.security.CustomUserDetailsService;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
@RequiredArgsConstructor
@Slf4j
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtTokenProvider tokenProvider;
    private final CustomUserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        try {
            String jwt = getJwtFromRequest(request);

            if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
                String username = tokenProvider.getUsernameFromToken(jwt);

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );

                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                );

                SecurityContextHolder.getContext().setAuthentication(authentication);
                log.debug("Set authentication for user: {}", username);
            }
        } catch (Exception e) {
            log.error("Cannot set user authentication: {}", e.getMessage());
        }

        filterChain.doFilter(request, response);
    }

    private String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

### Security Configuration with JWT

```java
package com.example.myapp.config;

import com.example.myapp.security.jwt.JwtAuthenticationFilter;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true)
@RequiredArgsConstructor
public class JwtSecurityConfig {

    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(
                    "/api/auth/**",
                    "/api/public/**",
                    "/swagger-ui/**",
                    "/v3/api-docs/**"
                ).permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            );

        http.addFilterBefore(jwtAuthenticationFilter,
            UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

### Authentication Controller

```java
package com.example.myapp.controller;

import com.example.myapp.dto.auth.*;
import com.example.myapp.model.Role;
import com.example.myapp.model.User;
import com.example.myapp.repository.RoleRepository;
import com.example.myapp.repository.UserRepository;
import com.example.myapp.security.jwt.JwtTokenProvider;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.util.HashSet;
import java.util.Set;

@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final UserRepository userRepository;
    private final RoleRepository roleRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider tokenProvider;

    @PostMapping("/login")
    public ResponseEntity<JwtAuthResponse> login(@Valid @RequestBody LoginRequest request) {
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            )
        );

        SecurityContextHolder.getContext().setAuthentication(authentication);
        String token = tokenProvider.generateToken(authentication);

        return ResponseEntity.ok(new JwtAuthResponse(token, "Bearer"));
    }

    @PostMapping("/register")
    public ResponseEntity<MessageResponse> register(
            @Valid @RequestBody RegisterRequest request) {

        if (userRepository.existsByUsername(request.getUsername())) {
            return ResponseEntity.badRequest()
                .body(new MessageResponse("Username is already taken"));
        }

        if (userRepository.existsByEmail(request.getEmail())) {
            return ResponseEntity.badRequest()
                .body(new MessageResponse("Email is already in use"));
        }

        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));

        // Assign default role
        Role userRole = roleRepository.findByName("ROLE_USER")
            .orElseThrow(() -> new RuntimeException("Default role not found"));

        Set<Role> roles = new HashSet<>();
        roles.add(userRole);
        user.setRoles(roles);

        userRepository.save(user);

        return ResponseEntity.ok(new MessageResponse("User registered successfully"));
    }

    @GetMapping("/me")
    public ResponseEntity<UserInfoResponse> getCurrentUser() {
        Authentication authentication = SecurityContextHolder.getContext()
            .getAuthentication();

        String username = authentication.getName();
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new RuntimeException("User not found"));

        UserInfoResponse response = new UserInfoResponse(
            user.getId(),
            user.getUsername(),
            user.getEmail(),
            user.getRoles().stream()
                .map(Role::getName)
                .toList()
        );

        return ResponseEntity.ok(response);
    }
}
```

### DTOs for Authentication

```java
package com.example.myapp.dto.auth;

import jakarta.validation.constraints.*;
import lombok.Data;

// LoginRequest.java
@Data
public class LoginRequest {
    @NotBlank(message = "Username is required")
    private String username;

    @NotBlank(message = "Password is required")
    private String password;
}

// RegisterRequest.java
@Data
public class RegisterRequest {
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 20, message = "Username must be between 3 and 20 characters")
    private String username;

    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 6, max = 40, message = "Password must be between 6 and 40 characters")
    private String password;
}

// JwtAuthResponse.java
@Data
@AllArgsConstructor
public class JwtAuthResponse {
    private String accessToken;
    private String tokenType;
}

// UserInfoResponse.java
@Data
@AllArgsConstructor
public class UserInfoResponse {
    private Long id;
    private String username;
    private String email;
    private List<String> roles;
}

// MessageResponse.java
@Data
@AllArgsConstructor
public class MessageResponse {
    private String message;
}
```

### Testing JWT Authentication

```bash
# 1. Register a new user
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "email": "john@example.com",
    "password": "password123"
  }'

# Response: {"message": "User registered successfully"}

# 2. Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "password": "password123"
  }'

# Response:
# {
#   "accessToken": "eyJhbGciOiJIUzUxMiJ9...",
#   "tokenType": "Bearer"
# }

# 3. Access protected endpoint
curl http://localhost:8080/api/auth/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..."

# Response:
# {
#   "id": 1,
#   "username": "john",
#   "email": "john@example.com",
#   "roles": ["ROLE_USER"]
# }

# 4. Access without token - 401 Unauthorized
curl http://localhost:8080/api/users
```

### Android JWT Integration Example

```kotlin
// Retrofit Service
interface ApiService {
    @POST("api/auth/login")
    suspend fun login(@Body request: LoginRequest): JwtAuthResponse

    @GET("api/auth/me")
    suspend fun getCurrentUser(): UserInfoResponse
}

// JWT Interceptor
class JwtInterceptor(private val tokenManager: TokenManager) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val token = tokenManager.getToken()
        val request = if (token != null) {
            chain.request().newBuilder()
                .addHeader("Authorization", "Bearer $token")
                .build()
        } else {
            chain.request()
        }
        return chain.proceed(request)
    }
}

// Token Manager (SharedPreferences with encryption)
class TokenManager(context: Context) {
    private val prefs = EncryptedSharedPreferences.create(
        "auth_prefs",
        MasterKey.Builder(context).build(),
        context,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    fun saveToken(token: String) {
        prefs.edit().putString("jwt_token", token).apply()
    }

    fun getToken(): String? = prefs.getString("jwt_token", null)

    fun clearToken() {
        prefs.edit().remove("jwt_token").apply()
    }
}
```

## 6. OAuth2 Integration

### OAuth2 Flow Diagram

```
┌─────────┐                                  ┌──────────────┐        ┌─────────────┐
│ Client  │                                  │   Server     │        │   Google    │
│  App    │                                  │ (Your API)   │        │  (OAuth2)   │
└────┬────┘                                  └──────┬───────┘        └──────┬──────┘
     │                                              │                       │
     │  1. Click "Login with Google"               │                       │
     ├─────────────────────────────────────────────>                       │
     │                                              │                       │
     │  2. Redirect to Google                      │                       │
     ├──────────────────────────────────────────────────────────────────────>
     │                                              │                       │
     │  3. User logs in to Google                  │                       │
     │                                              │                       │
     │  4. Google returns authorization code       │                       │
     │<─────────────────────────────────────────────────────────────────────┤
     │                                              │                       │
     │  5. Send code to your server                │                       │
     ├─────────────────────────────────────────────>│                       │
     │                                              │                       │
     │                                              │  6. Exchange code     │
     │                                              │     for access token  │
     │                                              ├──────────────────────>│
     │                                              │                       │
     │                                              │  7. Return token      │
     │                                              │<──────────────────────┤
     │                                              │                       │
     │  8. Return JWT token                        │                       │
     │<─────────────────────────────────────────────┤                       │
     │                                              │                       │
```

### OAuth2 Configuration

```properties
# application.properties
spring.security.oauth2.client.registration.google.client-id=your-google-client-id
spring.security.oauth2.client.registration.google.client-secret=your-google-client-secret
spring.security.oauth2.client.registration.google.scope=profile,email

spring.security.oauth2.client.registration.github.client-id=your-github-client-id
spring.security.oauth2.client.registration.github.client-secret=your-github-client-secret
spring.security.oauth2.client.registration.github.scope=read:user,user:email
```

### OAuth2 Security Configuration

```java
package com.example.myapp.config;

import com.example.myapp.security.oauth2.OAuth2AuthenticationSuccessHandler;
import com.example.myapp.security.oauth2.OAuth2AuthenticationFailureHandler;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@RequiredArgsConstructor
public class OAuth2SecurityConfig {

    private final OAuth2AuthenticationSuccessHandler successHandler;
    private final OAuth2AuthenticationFailureHandler failureHandler;

    @Bean
    public SecurityFilterChain oauth2SecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login", "/oauth2/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .authorizationEndpoint(authorization -> authorization
                    .baseUri("/oauth2/authorize"))
                .redirectionEndpoint(redirection -> redirection
                    .baseUri("/oauth2/callback/*"))
                .successHandler(successHandler)
                .failureHandler(failureHandler)
            );

        return http.build();
    }
}
```

### OAuth2 Success Handler

```java
package com.example.myapp.security.oauth2;

import com.example.myapp.model.User;
import com.example.myapp.repository.UserRepository;
import com.example.myapp.security.jwt.JwtTokenProvider;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.Authentication;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler;
import org.springframework.stereotype.Component;
import org.springframework.web.util.UriComponentsBuilder;

import java.io.IOException;

@Component
@RequiredArgsConstructor
public class OAuth2AuthenticationSuccessHandler
        extends SimpleUrlAuthenticationSuccessHandler {

    private final JwtTokenProvider tokenProvider;
    private final UserRepository userRepository;

    @Override
    public void onAuthenticationSuccess(
            HttpServletRequest request,
            HttpServletResponse response,
            Authentication authentication) throws IOException, ServletException {

        OAuth2User oAuth2User = (OAuth2User) authentication.getPrincipal();

        String email = oAuth2User.getAttribute("email");
        String name = oAuth2User.getAttribute("name");

        // Find or create user
        User user = userRepository.findByEmail(email)
            .orElseGet(() -> createNewOAuth2User(email, name));

        // Generate JWT token
        String token = tokenProvider.generateTokenFromUsername(user.getUsername());

        // Redirect to frontend with token
        String targetUrl = UriComponentsBuilder.fromUriString("http://localhost:3000/oauth2/redirect")
            .queryParam("token", token)
            .build().toUriString();

        getRedirectStrategy().sendRedirect(request, response, targetUrl);
    }

    private User createNewOAuth2User(String email, String name) {
        User user = new User();
        user.setUsername(email);
        user.setEmail(email);
        user.setPassword(""); // OAuth2 users don't have passwords
        // Set default role
        return userRepository.save(user);
    }
}
```

## 7. Method-Level Security

### Enable Method Security

```java
@Configuration
@EnableMethodSecurity(
    prePostEnabled = true,   // Enable @PreAuthorize and @PostAuthorize
    securedEnabled = true,   // Enable @Secured
    jsr250Enabled = true     // Enable @RolesAllowed
)
public class MethodSecurityConfig {
}
```

### Using @PreAuthorize

```java
package com.example.myapp.service;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    // Only authenticated users
    @PreAuthorize("isAuthenticated()")
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    // Only ADMIN role
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        userRepository.deleteById(userId);
    }

    // Multiple roles (OR)
    @PreAuthorize("hasAnyRole('ADMIN', 'MODERATOR')")
    public void banUser(Long userId) {
        // Ban logic
    }

    // Check if user owns the resource
    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public User updateUser(Long userId, UpdateUserRequest request) {
        // Update logic
    }

    // Complex expressions
    @PreAuthorize("hasRole('ADMIN') and #user.enabled == true")
    public void promoteToModerator(User user) {
        // Promotion logic
    }

    // Check return value
    @PostAuthorize("returnObject.username == authentication.name or hasRole('ADMIN')")
    public User getUserById(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}
```

### Using @Secured

```java
import org.springframework.security.access.annotation.Secured;

@Service
public class ProductService {

    @Secured("ROLE_USER")
    public List<Product> getProducts() {
        return productRepository.findAll();
    }

    @Secured({"ROLE_ADMIN", "ROLE_MODERATOR"})
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }
}
```

### Using @RolesAllowed (JSR-250)

```java
import javax.annotation.security.RolesAllowed;

@Service
public class OrderService {

    @RolesAllowed("ROLE_USER")
    public Order createOrder(OrderRequest request) {
        // Create order
    }

    @RolesAllowed({"ROLE_ADMIN", "ROLE_FINANCE"})
    public void refundOrder(Long orderId) {
        // Refund logic
    }
}
```

### Custom Security Expressions

```java
package com.example.myapp.security;

import com.example.myapp.model.Post;
import com.example.myapp.repository.PostRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Component;

@Component("postSecurity")
@RequiredArgsConstructor
public class PostSecurityExpression {

    private final PostRepository postRepository;

    public boolean isOwner(Authentication authentication, Long postId) {
        String username = authentication.getName();
        Post post = postRepository.findById(postId).orElse(null);
        return post != null && post.getAuthor().getUsername().equals(username);
    }
}

// Usage in controller
@Service
public class PostService {

    @PreAuthorize("@postSecurity.isOwner(authentication, #postId)")
    public void deletePost(Long postId) {
        postRepository.deleteById(postId);
    }
}
```

## 8. CORS and CSRF Configuration

### CORS (Cross-Origin Resource Sharing)

CORS allows your API to be accessed from different domains (e.g., frontend running on localhost:3000 accessing API on localhost:8080).

```java
package com.example.myapp.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.Arrays;
import java.util.List;

@Configuration
public class CorsConfig {

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        // Allowed origins
        configuration.setAllowedOrigins(Arrays.asList(
            "http://localhost:3000",
            "http://localhost:4200",
            "https://myapp.com"
        ));

        // Allowed methods
        configuration.setAllowedMethods(Arrays.asList(
            "GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"
        ));

        // Allowed headers
        configuration.setAllowedHeaders(Arrays.asList(
            "Authorization",
            "Content-Type",
            "X-Requested-With"
        ));

        // Expose headers
        configuration.setExposedHeaders(List.of("Authorization"));

        // Allow credentials (cookies, authorization headers)
        configuration.setAllowCredentials(true);

        // Max age for preflight requests
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);

        return source;
    }

    // Alternative: WebMvcConfigurer approach
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("http://localhost:3000")
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowedHeaders("*")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}
```

### Integrate CORS with Security

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .cors(cors -> cors.configurationSource(corsConfigurationSource()))
        .csrf(csrf -> csrf.disable())
        // ... rest of configuration
        ;
    return http.build();
}
```

### CSRF (Cross-Site Request Forgery) Protection

For **stateless APIs with JWT**, CSRF is typically disabled:

```java
http.csrf(csrf -> csrf.disable())
```

For **session-based authentication** (using cookies), enable CSRF:

```java
http
    .csrf(csrf -> csrf
        .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
    );
```

### CSRF with Angular/React

```java
@Configuration
public class CsrfSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringRequestMatchers("/api/auth/**")  // Exempt login/register
            );
        return http.build();
    }
}
```

**Frontend (React/Angular):**

```javascript
// Get CSRF token from cookie
const csrfToken = document.cookie
  .split('; ')
  .find(row => row.startsWith('XSRF-TOKEN='))
  ?.split('=')[1];

// Include in requests
fetch('/api/users', {
  method: 'POST',
  headers: {
    'X-XSRF-TOKEN': csrfToken,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
});
```

## 9. Password Encoding with BCrypt

### Why BCrypt?

- **Adaptive**: Adjustable work factor (rounds)
- **Salted**: Automatically generates unique salt
- **Slow**: Intentionally slow to prevent brute-force attacks
- **Industry Standard**: Widely used and tested

### BCrypt Work Factor

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);  // 12 rounds (2^12 iterations)
}

// Rounds comparison:
// 10 rounds: ~70ms
// 12 rounds: ~300ms (recommended)
// 14 rounds: ~1200ms
// 16 rounds: ~5000ms
```

### Password Encoding Examples

```java
@Service
@RequiredArgsConstructor
public class PasswordService {

    private final PasswordEncoder passwordEncoder;

    public String encodePassword(String rawPassword) {
        return passwordEncoder.encode(rawPassword);
    }

    public boolean matchesPassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }

    public void changePassword(User user, String oldPassword, String newPassword) {
        // Verify old password
        if (!passwordEncoder.matches(oldPassword, user.getPassword())) {
            throw new BadCredentialsException("Old password is incorrect");
        }

        // Encode and save new password
        user.setPassword(passwordEncoder.encode(newPassword));
        userRepository.save(user);
    }
}
```

### Password Validation

```java
package com.example.myapp.validator;

import jakarta.validation.Constraint;
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = StrongPasswordValidator.class)
public @interface StrongPassword {
    String message() default "Password must be at least 8 characters and contain uppercase, lowercase, digit, and special character";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

class StrongPasswordValidator implements ConstraintValidator<StrongPassword, String> {

    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        if (password == null) {
            return false;
        }

        return password.length() >= 8 &&
               password.matches(".*[A-Z].*") &&      // At least one uppercase
               password.matches(".*[a-z].*") &&      // At least one lowercase
               password.matches(".*\\d.*") &&        // At least one digit
               password.matches(".*[@#$%^&+=!].*");  // At least one special char
    }
}

// Usage in DTO
@Data
public class RegisterRequest {
    @NotBlank
    @StrongPassword
    private String password;
}
```

### Alternative: Argon2 (More Secure)

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.70</version>
</dependency>
```

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8();
}
```

## 10. Complete JWT Auth Flow Example

### Complete Project Structure

```
src/main/java/com/example/myapp/
├── config/
│   ├── JwtSecurityConfig.java
│   ├── CorsConfig.java
│   └── MethodSecurityConfig.java
├── controller/
│   ├── AuthController.java
│   ├── UserController.java
│   └── AdminController.java
├── dto/
│   ├── auth/
│   │   ├── LoginRequest.java
│   │   ├── RegisterRequest.java
│   │   ├── JwtAuthResponse.java
│   │   └── UserInfoResponse.java
│   └── user/
│       ├── UserResponse.java
│       └── UpdateUserRequest.java
├── model/
│   ├── User.java
│   └── Role.java
├── repository/
│   ├── UserRepository.java
│   └── RoleRepository.java
├── security/
│   ├── CustomUserDetailsService.java
│   └── jwt/
│       ├── JwtTokenProvider.java
│       └── JwtAuthenticationFilter.java
├── service/
│   ├── UserService.java
│   └── AuthService.java
└── exception/
    ├── GlobalExceptionHandler.java
    └── UnauthorizedException.java
```

### User Controller with Security

```java
package com.example.myapp.controller;

import com.example.myapp.dto.user.UpdateUserRequest;
import com.example.myapp.dto.user.UserResponse;
import com.example.myapp.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    public ResponseEntity<List<UserResponse>> getAllUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }

    @GetMapping("/{id}")
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<UserResponse> getUserById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    @PutMapping("/{id}")
    @PreAuthorize("#id == authentication.principal.id or hasRole('ADMIN')")
    public ResponseEntity<UserResponse> updateUser(
            @PathVariable Long id,
            @RequestBody UpdateUserRequest request,
            Authentication authentication) {
        return ResponseEntity.ok(userService.updateUser(id, request));
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }

    @GetMapping("/profile")
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<UserResponse> getCurrentUserProfile(Authentication authentication) {
        String username = authentication.getName();
        return ResponseEntity.ok(userService.getUserByUsername(username));
    }
}
```

### Admin Controller

```java
package com.example.myapp.controller;

import com.example.myapp.dto.admin.UserStatistics;
import com.example.myapp.service.AdminService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/admin")
@PreAuthorize("hasRole('ADMIN')")
@RequiredArgsConstructor
public class AdminController {

    private final AdminService adminService;

    @GetMapping("/statistics")
    public ResponseEntity<UserStatistics> getStatistics() {
        return ResponseEntity.ok(adminService.getUserStatistics());
    }

    @PostMapping("/users/{id}/disable")
    public ResponseEntity<Void> disableUser(@PathVariable Long id) {
        adminService.disableUser(id);
        return ResponseEntity.ok().build();
    }

    @PostMapping("/users/{id}/enable")
    public ResponseEntity<Void> enableUser(@PathVariable Long id) {
        adminService.enableUser(id);
        return ResponseEntity.ok().build();
    }

    @PostMapping("/users/{id}/roles/{roleName}")
    public ResponseEntity<Void> assignRole(
            @PathVariable Long id,
            @PathVariable String roleName) {
        adminService.assignRoleToUser(id, roleName);
        return ResponseEntity.ok().build();
    }
}
```

### Exception Handling for Security

```java
package com.example.myapp.exception;

import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.MalformedJwtException;
import io.jsonwebtoken.security.SignatureException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.core.AuthenticationException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;

@RestControllerAdvice
@Slf4j
public class SecurityExceptionHandler {

    @ExceptionHandler(BadCredentialsException.class)
    public ResponseEntity<ErrorResponse> handleBadCredentials(BadCredentialsException ex) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(new ErrorResponse(
                HttpStatus.UNAUTHORIZED.value(),
                "Invalid username or password",
                LocalDateTime.now()
            ));
    }

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(new ErrorResponse(
                HttpStatus.FORBIDDEN.value(),
                "You don't have permission to access this resource",
                LocalDateTime.now()
            ));
    }

    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthentication(AuthenticationException ex) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(new ErrorResponse(
                HttpStatus.UNAUTHORIZED.value(),
                "Authentication failed: " + ex.getMessage(),
                LocalDateTime.now()
            ));
    }

    @ExceptionHandler({SignatureException.class, MalformedJwtException.class})
    public ResponseEntity<ErrorResponse> handleInvalidJwt(Exception ex) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(new ErrorResponse(
                HttpStatus.UNAUTHORIZED.value(),
                "Invalid JWT token",
                LocalDateTime.now()
            ));
    }

    @ExceptionHandler(ExpiredJwtException.class)
    public ResponseEntity<ErrorResponse> handleExpiredJwt(ExpiredJwtException ex) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(new ErrorResponse(
                HttpStatus.UNAUTHORIZED.value(),
                "JWT token has expired",
                LocalDateTime.now()
            ));
    }
}
```

## 11. Security Best Practices

### 1. Environment Variables for Secrets

```properties
# application.properties (DON'T store secrets here in production!)
jwt.secret=${JWT_SECRET:default-secret-key-for-development-only}
jwt.expiration=${JWT_EXPIRATION:86400000}

spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:5432/myapp}
spring.datasource.username=${DB_USERNAME:postgres}
spring.datasource.password=${DB_PASSWORD:password}
```

### 2. Refresh Token Implementation

```java
@Entity
@Table(name = "refresh_tokens")
@Data
public class RefreshToken {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne
    @JoinColumn(name = "user_id", referencedColumnName = "id")
    private User user;

    @Column(nullable = false, unique = true)
    private String token;

    @Column(nullable = false)
    private Instant expiryDate;
}

@Service
@RequiredArgsConstructor
public class RefreshTokenService {

    @Value("${jwt.refresh-expiration}")
    private Long refreshTokenDurationMs;

    private final RefreshTokenRepository refreshTokenRepository;
    private final UserRepository userRepository;

    public RefreshToken createRefreshToken(Long userId) {
        RefreshToken refreshToken = new RefreshToken();

        refreshToken.setUser(userRepository.findById(userId).get());
        refreshToken.setExpiryDate(Instant.now().plusMillis(refreshTokenDurationMs));
        refreshToken.setToken(UUID.randomUUID().toString());

        return refreshTokenRepository.save(refreshToken);
    }

    public Optional<RefreshToken> findByToken(String token) {
        return refreshTokenRepository.findByToken(token);
    }

    public RefreshToken verifyExpiration(RefreshToken token) {
        if (token.getExpiryDate().compareTo(Instant.now()) < 0) {
            refreshTokenRepository.delete(token);
            throw new TokenRefreshException(token.getToken(),
                "Refresh token was expired. Please make a new signin request");
        }
        return token;
    }
}

// Controller endpoint
@PostMapping("/refresh")
public ResponseEntity<JwtAuthResponse> refreshToken(
        @Valid @RequestBody TokenRefreshRequest request) {

    String requestRefreshToken = request.getRefreshToken();

    return refreshTokenService.findByToken(requestRefreshToken)
        .map(refreshTokenService::verifyExpiration)
        .map(RefreshToken::getUser)
        .map(user -> {
            String token = tokenProvider.generateTokenFromUsername(user.getUsername());
            return ResponseEntity.ok(new JwtAuthResponse(token, "Bearer"));
        })
        .orElseThrow(() -> new TokenRefreshException(requestRefreshToken,
            "Refresh token is not in database!"));
}
```

### 3. Rate Limiting

```xml
<dependency>
    <groupId>com.bucket4j</groupId>
    <artifactId>bucket4j-core</artifactId>
    <version>8.7.0</version>
</dependency>
```

```java
@Component
public class RateLimitingFilter extends OncePerRequestFilter {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    private Bucket createNewBucket() {
        Bandwidth limit = Bandwidth.classic(100, Refill.greedy(100, Duration.ofMinutes(1)));
        return Bucket.builder()
            .addLimit(limit)
            .build();
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        String key = request.getRemoteAddr();
        Bucket bucket = cache.computeIfAbsent(key, k -> createNewBucket());

        if (bucket.tryConsume(1)) {
            filterChain.doFilter(request, response);
        } else {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Too many requests");
        }
    }
}
```

### 4. Audit Logging

```java
@Entity
@Table(name = "audit_log")
@Data
public class AuditLog {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String action;
    private String resource;
    private String ipAddress;
    private LocalDateTime timestamp;
    private boolean success;
}

@Aspect
@Component
@Slf4j
public class SecurityAuditAspect {

    @Autowired
    private AuditLogRepository auditLogRepository;

    @AfterReturning(
        pointcut = "@annotation(org.springframework.security.access.prepost.PreAuthorize)",
        returning = "result"
    )
    public void logSecureMethodAccess(JoinPoint joinPoint, Object result) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        AuditLog log = new AuditLog();
        log.setUsername(auth.getName());
        log.setAction(joinPoint.getSignature().getName());
        log.setResource(joinPoint.getTarget().getClass().getSimpleName());
        log.setTimestamp(LocalDateTime.now());
        log.setSuccess(true);

        auditLogRepository.save(log);
    }
}
```

## 12. Android vs Spring Security Comparison

### Authentication Patterns

| Pattern | Android | Spring Security |
|---------|---------|-----------------|
| **Login Flow** | Firebase Auth, Manual API calls | Built-in authentication with UserDetailsService |
| **Token Storage** | EncryptedSharedPreferences | In-memory (stateless) or Session |
| **Token Transmission** | Retrofit interceptor | Security filter chain |
| **Session Management** | Manual token refresh | Automatic with Spring Session |
| **Logout** | Clear local token | Invalidate session/blacklist JWT |

### Authorization Patterns

```kotlin
// Android - Manual checks
fun deleteUser(userId: String) {
    if (currentUser.hasRole("ADMIN")) {
        // Delete user
    } else {
        throw UnauthorizedException()
    }
}

// Spring - Declarative
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long userId) {
    userRepository.deleteById(userId);
}
```

### Security Storage

```kotlin
// Android - KeyStore for sensitive data
val keyStore = KeyStore.getInstance("AndroidKeyStore")
val secretKey = keyStore.getKey("my_key", null) as SecretKey

val cipher = Cipher.getInstance("AES/GCM/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey)
val encrypted = cipher.doFinal(plainText.toByteArray())

// Spring - BCrypt for passwords
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

String hashedPassword = passwordEncoder.encode(rawPassword);
```

### OAuth2 Flow Comparison

```kotlin
// Android - Google Sign-In
val gso = GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
    .requestIdToken(getString(R.string.default_web_client_id))
    .requestEmail()
    .build()

val googleSignInClient = GoogleSignIn.getClient(this, gso)
startActivityForResult(googleSignInClient.signInIntent, RC_SIGN_IN)

// Spring - OAuth2 Login
@Configuration
public class OAuth2Config {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        http.oauth2Login(oauth2 -> oauth2
            .successHandler(successHandler)
        );
        return http.build();
    }
}
```

## 🎯 Key Takeaways

1. **Authentication vs Authorization**: Authentication verifies identity, authorization controls access
2. **JWT**: Stateless token-based authentication, perfect for REST APIs
3. **BCrypt**: Industry-standard password hashing with adjustable work factor
4. **Method Security**: Use `@PreAuthorize` for fine-grained access control
5. **CORS**: Essential for frontend-backend communication across domains
6. **OAuth2**: Delegate authentication to trusted providers (Google, GitHub)
7. **Security Filters**: Process every request through filter chain
8. **UserDetailsService**: Custom authentication with database users
9. **Password Encoding**: NEVER store plain text passwords
10. **Best Practices**: Use environment variables, implement refresh tokens, add rate limiting

## 🔒 Security Checklist

- [ ] Use HTTPS in production
- [ ] Store JWT secret in environment variables
- [ ] Implement refresh tokens
- [ ] Add rate limiting
- [ ] Enable CORS with specific origins
- [ ] Use strong password encoding (BCrypt with 12+ rounds)
- [ ] Implement password validation rules
- [ ] Add audit logging for sensitive operations
- [ ] Use method-level security annotations
- [ ] Validate all user inputs
- [ ] Implement account lockout after failed attempts
- [ ] Add email verification
- [ ] Implement password reset flow
- [ ] Use HTTPS-only cookies for tokens
- [ ] Set appropriate JWT expiration times
- [ ] Implement logout/token blacklisting
- [ ] Regular security dependency updates

## 🚀 Next Steps

Now that you understand Spring Security, you can:

1. Implement authentication in your Spring Boot applications
2. Secure REST APIs with JWT
3. Add OAuth2 social login
4. Implement role-based access control
5. Build secure microservices

## 📚 Additional Resources

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [JWT.io](https://jwt.io/) - JWT debugger and documentation
- [OWASP Security Cheat Sheets](https://cheatsheetseries.owasp.org/)
- [OAuth 2.0 Specification](https://oauth.net/2/)

Continue to [Section 7: Spring Data JPA Advanced](./07-spring-data-jpa-advanced.md)
