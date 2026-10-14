# Enterprise Inventory & Order Management System

A production-grade **Spring Boot** enterprise application demonstrating core **Java/JEE** concepts including Dependency Injection, Transaction Management, AOP, Spring Batch, and REST APIs.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Spring Boot 3.2.5 |
| Language | Java 17 |
| ORM / Data Access | Hibernate / JPA (Spring Data JPA) |
| Batch Processing | Spring Batch |
| AOP | Spring AOP (AspectJ) |
| Database | H2 (dev) / MySQL (prod) |
| Testing | JUnit 5 + Mockito |
| Build Tool | Maven |
| Boilerplate Reduction | Lombok |

---

## Key Concepts Demonstrated

### 1. Dependency Injection & IoC
Spring's IoC container manages all beans. Constructor injection is used throughout (preferred over field injection).
```java
@Service
@RequiredArgsConstructor  // Lombok generates constructor — Spring injects dependencies
public class ProductServiceImpl implements ProductService {
    private final ProductRepository productRepository; // injected by Spring
}
```

### 2. Transaction Management
`@Transactional` ensures atomic operations — if any step fails, everything rolls back.
```java
@Transactional(rollbackFor = Exception.class)
public OrderDTO placeOrder(OrderDTO orderDTO) {
    // Step 1: decrease stock
    // Step 2: create order record
    // If Step 2 fails → Step 1 is ROLLED BACK automatically
}
```

### 3. Aspect-Oriented Programming (AOP)
A single `LoggingAspect` class handles logging, execution timing, and exception tracking for **all service methods** — without touching service code.
```java
@Around("serviceLayer()")
public Object measureExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = pjp.proceed();
    log.info("Executed in {} ms", System.currentTimeMillis() - start);
    return result;
}
```

### 4. Spring Batch — Chunk-Oriented Processing
Processes inventory records in configurable chunks, each with its own transaction.
```
ItemReader → reads low-stock products from DB
ItemProcessor → validates and flags critical items
ItemWriter → writes batch report
Chunk size: 10 items per transaction
```

### 5. Test-Driven Development (TDD)
8 unit tests with JUnit 5 + Mockito covering success, failure, and edge cases.

---

## Project Structure

```
src/
├── main/java/com/akanksha/inventory/
│   ├── InventoryApplication.java       # Entry point
│   ├── aspect/
│   │   └── LoggingAspect.java          # AOP — centralised logging & timing
│   ├── batch/
│   │   └── InventoryBatchConfig.java   # Spring Batch job configuration
│   ├── controller/
│   │   ├── ProductController.java      # REST API endpoints for products
│   │   └── OrderController.java        # REST API endpoints for orders + batch trigger
│   ├── dto/
│   │   ├── ProductDTO.java
│   │   └── OrderDTO.java
│   ├── entity/
│   │   ├── Product.java                # JPA Entity with lifecycle hooks
│   │   └── Order.java                  # JPA Entity with ManyToOne relationship
│   ├── exception/
│   │   ├── GlobalExceptionHandler.java # Centralised error handling
│   │   └── ResourceNotFoundException.java
│   ├── repository/
│   │   ├── ProductRepository.java      # JPA Repository with custom JPQL queries
│   │   └── OrderRepository.java
│   └── service/
│       ├── ProductService.java         # Interface (IoC principle)
│       ├── ProductServiceImpl.java     # Implementation with @Transactional
│       └── OrderService.java           # Atomic order placement
└── test/java/com/akanksha/inventory/
    └── service/
        └── ProductServiceTest.java     # 8 JUnit 5 + Mockito unit tests
```

---

## Getting Started

### Prerequisites
- Java 17+
- Maven 3.8+

### Run (No Database Setup Needed)
```bash
# Clone the repository
git clone https://github.com/AkankshaKesarkar/inventory-management-system.git
cd inventory-management-system

# Build and run (uses H2 in-memory DB automatically)
mvn spring-boot:run
```

### Run Tests
```bash
mvn test
```

### Access
| URL | Description |
|---|---|
| `http://localhost:8080/api/products` | Product REST API |
| `http://localhost:8080/api/orders` | Order REST API |
| `http://localhost:8080/h2-console` | H2 DB Console (dev only) |

---

## REST API Reference

### Products
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/products` | Create a product |
| GET | `/api/products` | Get all products |
| GET | `/api/products/{id}` | Get product by ID |
| GET | `/api/products/category/{cat}` | Filter by category |
| GET | `/api/products/low-stock?threshold=10` | Get low-stock products |
| PUT | `/api/products/{id}` | Update product |
| DELETE | `/api/products/{id}` | Delete product |
| GET | `/api/products/inventory-value` | Total inventory value |

### Orders
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/orders` | Place an order (atomic) |
| GET | `/api/orders` | Get all orders |
| GET | `/api/orders/{id}` | Get order by ID |
| PUT | `/api/orders/{id}/cancel` | Cancel an order |
| POST | `/api/orders/batch/run-inventory-report` | Trigger batch job |

### Sample Request — Create Product
```json
POST /api/products
{
  "name": "Laptop",
  "category": "Electronics",
  "price": 75000.00,
  "stockQuantity": 50
}
```

### Sample Request — Place Order
```json
POST /api/orders
{
  "customerName": "Akanksha Kesarkar",
  "productId": 1,
  "quantity": 2
}
```

---

## Author

**Akanksha Ramchandra Kesarkar**  
B.E. Computer Science & Engineering | 2024  
📧 akankshakesarkar5@gmail.com  
🔗 [LinkedIn](https://linkedin.com/in/akanksha-kesarkar)
