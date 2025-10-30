# Microservices Communication with Spring Cloud OpenFeign

## Project Overview

This project demonstrates two microservices communicating using **Spring Cloud OpenFeign** - a declarative REST client that simplifies inter-service communication.

### Services Architecture

1. **purchase-billing-service** (Port 8080)
   - Manages shopping carts and cart items
   - Provides REST endpoints for cart operations

2. **user-service** (Port varies)
   - Manages user information
   - Communicates with purchase-billing-service via Feign Client

---

## Feign Client Implementation

### What is Feign?

Spring Cloud OpenFeign is a declarative web service client that makes writing HTTP clients easier. Instead of manually creating REST clients with RestTemplate or WebClient, you simply define a Java interface and annotate it.

### Setup Steps

#### 1. Add Dependency (user-service)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 2. Enable Feign Clients

Add `@EnableFeignClients` annotation to your main application class:

```java
@SpringBootApplication
@EnableFeignClients
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

#### 3. Create Feign Client Interface

```java
@FeignClient(name = "purchase-billing-service", url = "http://localhost:8080")
public interface UserFeignService {
    @PostMapping("/shopping-carts")
    ResponseEntity<ShoppingCart> createShoppingCart(@RequestBody ShoppingCart shoppingCart);
}
```

**Key Annotations Explained:**

- `@FeignClient`: Marks this interface as a Feign client
  - `name`: Logical name for the service (used for service discovery)
  - `url`: Direct URL to the target service (hardcoded for simple setup)

- `@PostMapping`: Defines the HTTP method and endpoint path
- `@RequestBody`: Maps the ShoppingCart object to the request body

#### 4. Create DTO for Communication

```java
package com.example.user_service.feign.dto;

public class ShoppingCart {
    private Long id;
    private List<CartItem> items;
    // Constructors, getters, setters
}
```

**Important:** The DTO in user-service should match the structure expected by purchase-billing-service.

---

## How Feign Communication Works

### Request Flow

```
User Service → Feign Client → HTTP Request → Purchase Billing Service
                    ↓
              Serializes DTO to JSON
                    ↓
              POST http://localhost:8080/shopping-carts
                    ↓
              Purchase Billing Service processes request
                    ↓
              Response (ShoppingCart JSON)
                    ↓
              Feign deserializes to ShoppingCart object
                    ↓
              Returns ResponseEntity<ShoppingCart>
```

### Usage Example

In any service or controller within user-service:

```java
@Service
public class UserService {
    
    private final UserFeignService feignService;
    
    public UserService(UserFeignService feignService) {
        this.feignService = feignService;
    }
    
    public ShoppingCart createCartForUser(User user) {
        ShoppingCart cart = new ShoppingCart();
        // Set cart properties
        
        ResponseEntity<ShoppingCart> response = feignService.createShoppingCart(cart);
        return response.getBody();
    }
}
```

---

## Advantages of Using Feign

1. **Declarative Syntax**: No boilerplate code for HTTP clients
2. **Spring Integration**: Seamless integration with Spring Boot
3. **Automatic Serialization**: Handles JSON conversion automatically
4. **Service Discovery Ready**: Can integrate with Eureka/Consul when `url` is removed
5. **Load Balancing**: Built-in support with Spring Cloud LoadBalancer

---

## Configuration Options

### application.properties (user-service)

```properties
# Feign client timeout configuration
feign.client.config.default.connectTimeout=5000
feign.client.config.default.readTimeout=5000

# Feign logging level
logging.level.com.example.user_service.feign=DEBUG
```

### Service Discovery Alternative

For production environments, replace hardcoded URLs with service discovery:

```java
@FeignClient(name = "purchase-billing-service")  // Remove url parameter
public interface UserFeignService {
    @PostMapping("/shopping-carts")
    ResponseEntity<ShoppingCart> createShoppingCart(@RequestBody ShoppingCart shoppingCart);
}
```

This requires:
- Eureka Server running
- Both services registered with Eureka
- `spring-cloud-starter-netflix-eureka-client` dependency
## Running the Services

1. **Start purchase-billing-service** on port 8080
2. **Start user-service** on any available port
3. User-service will automatically connect to purchase-billing-service via Feign
