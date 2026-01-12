# Spring Boot Application Runtime Architecture

This document illustrates the typical runtime architecture of a Spring Boot application.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CLIENT REQUESTS                                    │
│                    (Browser, Mobile App, API Client)                         │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         EMBEDDED SERVER                                      │
│                    (Tomcat / Jetty / Undertow)                              │
│                         Port: 8080                                           │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FILTER CHAIN                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Security  │→ │    CORS     │→ │   Logging   │→ │   Custom    │        │
│  │   Filter    │  │   Filter    │  │   Filter    │  │   Filters   │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       DISPATCHER SERVLET                                     │
│                    (Front Controller Pattern)                                │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       HANDLER MAPPING                                        │
│              (Maps URL patterns to Controller methods)                       │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     INTERCEPTORS (HandlerInterceptor)                        │
│            preHandle() → Controller → postHandle() → afterCompletion()       │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CONTROLLER LAYER                                      │
│                         @RestController                                      │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  @GetMapping, @PostMapping, @PutMapping, @DeleteMapping              │   │
│  │  @RequestBody, @PathVariable, @RequestParam                          │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SERVICE LAYER                                        │
│                           @Service                                           │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Business Logic, Validation, Transaction Management (@Transactional) │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       REPOSITORY LAYER                                       │
│                    @Repository / JpaRepository                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Data Access, CRUD Operations, Custom Queries (@Query)               │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  Database  │  │   Redis    │  │  MongoDB   │  │    S3      │            │
│  │  (MySQL/   │  │   Cache    │  │            │  │            │            │
│  │  Postgres) │  │            │  │            │  │            │            │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                    SPRING APPLICATION CONTEXT                                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  IoC Container (Dependency Injection)                               │    │
│  │  ├── Bean Factory                                                   │    │
│  │  ├── Bean Lifecycle Management                                      │    │
│  │  └── Auto-configuration                                             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Cross-cutting Concerns                                             │    │
│  │  ├── @Aspect (AOP)                                                  │    │
│  │  ├── Exception Handling (@ControllerAdvice)                         │    │
│  │  ├── Validation (@Valid, @Validated)                                │    │
│  │  └── Actuator (Health, Metrics, Info)                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Key Components

| Layer | Responsibility |
|-------|----------------|
| **Embedded Server** | Handles HTTP connections (Tomcat is the default) |
| **Filter Chain** | Pre/post processing of requests (security, CORS, logging) |
| **DispatcherServlet** | Front controller that routes requests to appropriate handlers |
| **Handler Mapping** | Maps URL patterns to controller methods |
| **Interceptors** | Execute logic before/after controller methods |
| **Controller** | Handles HTTP requests, performs input validation |
| **Service** | Contains business logic, manages transactions |
| **Repository** | Handles data persistence and database queries |
| **Application Context** | Manages beans, dependency injection, and lifecycle |

## Request Flow

1. **Request Arrival**: Client request arrives at the embedded server (Tomcat/Jetty/Undertow)
2. **Filter Chain**: Request passes through the filter chain (security, CORS, logging, etc.)
3. **DispatcherServlet**: The front controller receives the request and determines the handler
4. **Handler Mapping**: URL pattern is matched to the appropriate controller method
5. **Interceptors**: `preHandle()` methods execute before the controller
6. **Controller**: Processes the request, validates input, calls service layer
7. **Service**: Executes business logic, manages transactions, calls repository
8. **Repository**: Interacts with the database to persist or retrieve data
9. **Response Flow**: Response flows back through interceptors (`postHandle()`, `afterCompletion()`)
10. **Client Response**: Final response is sent back to the client

## Spring Application Context

The Spring Application Context is the IoC (Inversion of Control) container that manages:

### Bean Management
- **Bean Factory**: Creates and manages bean instances
- **Bean Lifecycle**: Handles initialization and destruction callbacks
- **Auto-configuration**: Automatically configures beans based on classpath and properties

### Cross-cutting Concerns
- **AOP (@Aspect)**: Aspect-Oriented Programming for logging, security, transactions
- **Exception Handling (@ControllerAdvice)**: Global exception handling
- **Validation (@Valid, @Validated)**: Input validation using Bean Validation API
- **Actuator**: Production-ready features for monitoring and management

## Common Annotations

### Controller Layer
```java
@RestController
@RequestMapping("/api/v1")
public class UserController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) { ... }

    @PostMapping("/users")
    public User createUser(@RequestBody @Valid UserRequest request) { ... }
}
```

### Service Layer
```java
@Service
@Transactional
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### Repository Layer
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
}
```

## Configuration

### application.yml Example
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
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

## Related Resources

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Framework Reference](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
