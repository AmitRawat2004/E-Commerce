# Architecture & Project Structure Explanation

This document explains the architecture, file structure, and how all components connect in the E-Commerce REST API.

## 🎯 Main Entry Point

### `ECommerceApplication.java`
**Location**: `src/main/java/com/personal/E_Commerce/ECommerceApplication.java`

This is the **main file** that starts the entire Spring Boot application. It contains:
- `@SpringBootApplication` annotation that enables auto-configuration
- `main()` method that launches the application
- Spring Boot automatically scans and loads all components in this package and sub-packages

**What happens when it runs:**
1. Spring Boot starts the embedded Tomcat server
2. Scans for all `@Component`, `@Service`, `@Repository`, `@Controller` annotations
3. Loads configuration from `application.properties`
4. Connects to MySQL database
5. Sets up security filters
6. Makes the API available on port 8080

---

## 📂 Project Structure & Flow

### 1. **Model Layer** (`model/`)
**Purpose**: Define database entities and relationships

**Files:**
- `User.java` - User entity with username, email, password, role
- `Product.java` - Product entity with name, description, price, stock
- `Order.java` - Order entity linked to User, contains OrderItems
- `OrderItem.java` - Links Order to Product with quantity and price
- `Role.java` - Enum: USER, ADMIN
- `OrderStatus.java` - Enum: PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED

**How it connects:**
- These entities map to database tables via JPA annotations (`@Entity`, `@Table`)
- Relationships defined with `@ManyToOne`, `@OneToMany`
- Hibernate automatically creates/updates tables based on these entities

---

### 2. **Repository Layer** (`repository/`)
**Purpose**: Data access layer - interface with database

**Files:**
- `UserRepository.java` - Extends `JpaRepository<User, Long>`
- `ProductRepository.java` - Extends `JpaRepository<Product, Long>`
- `OrderRepository.java` - Extends `JpaRepository<Order, Long>`
- `OrderItemRepository.java` - Extends `JpaRepository<OrderItem, Long>`

**How it connects:**
- Spring Data JPA automatically implements these interfaces
- Provides CRUD operations (save, findById, findAll, delete, etc.)
- Used by Service layer to interact with database
- No SQL code needed - Spring generates it automatically

**Example Flow:**
```
Service → Repository → Database
```

---

### 3. **DTO Layer** (`dto/`)
**Purpose**: Data Transfer Objects - define request/response structures

**Request DTOs:**
- `LoginRequest.java` - Username and password for login
- `RegisterRequest.java` - User registration data
- `ProductRequest.java` - Product creation/update data
- `OrderRequest.java` - Order creation with items

**Response DTOs:**
- `AuthResponse.java` - JWT token and user info after login/register
- `ProductResponse.java` - Product data for responses
- `OrderResponse.java` - Order data with items
- `OrderItemResponse.java` - Order item details

**How it connects:**
- Controllers receive Request DTOs from HTTP requests
- Services convert DTOs to Entities for database operations
- Services convert Entities back to Response DTOs for HTTP responses
- Prevents exposing internal entity structure to API consumers

**Example Flow:**
```
HTTP Request → Controller (DTO) → Service (Entity) → Repository → Database
Database → Repository → Service (Entity) → Controller (DTO) → HTTP Response
```

---

### 4. **Service Layer** (`service/`)
**Purpose**: Business logic and orchestration

**Files:**
- `AuthService.java` - Handles registration, login, password encoding
- `ProductService.java` - Product CRUD operations
- `OrderService.java` - Order creation, stock management, order retrieval

**How it connects:**
- Controllers call Services
- Services use Repositories to access database
- Services contain business rules (e.g., check stock before creating order)
- Services convert between DTOs and Entities

**Example Flow:**
```
Controller → Service → Repository → Database
```

**Key Operations:**
- `AuthService.register()` - Validates, encodes password, saves user, generates JWT
- `OrderService.createOrder()` - Validates stock, creates order, updates product stock
- `ProductService.createProduct()` - Validates data, saves product

---

### 5. **Controller Layer** (`controller/`)
**Purpose**: Handle HTTP requests and responses

**Files:**
- `AuthController.java` - `/api/auth/*` endpoints (register, login)
- `ProductController.java` - `/api/products/*` endpoints (CRUD operations)
- `OrderController.java` - `/api/orders/*` endpoints (create, view orders)
- `AdminController.java` - `/api/admin/*` endpoints (admin-only operations)

**How it connects:**
- Receives HTTP requests (GET, POST, PUT, DELETE)
- Validates request DTOs using `@Valid`
- Calls appropriate Service methods
- Returns HTTP responses with Response DTOs
- Handles exceptions and returns appropriate status codes

**Example Flow:**
```
HTTP Request → Controller → Service → Repository → Database
HTTP Response ← Controller ← Service ← Repository ← Database
```

**Request Flow Example:**
```
POST /api/orders
{
  "orderItems": [{"productId": 1, "quantity": 2}]
}
↓
OrderController.createOrder()
↓
OrderService.createOrder()
↓
OrderRepository.save()
↓
Database
```

---

### 6. **Security Layer** (`security/`)
**Purpose**: Authentication and authorization

**Files:**
- `SecurityConfig.java` - Main security configuration
- `JwtUtil.java` - JWT token generation and validation
- `JwtAuthenticationFilter.java` - Intercepts requests, validates JWT tokens
- `CustomUserDetailsService.java` - Loads user details for authentication

**How it connects:**

**SecurityConfig.java:**
- Defines which endpoints are public (`/api/auth/**`)
- Defines which endpoints require authentication
- Defines role-based access (`/api/admin/**` requires ADMIN role)
- Configures JWT filter to run before controllers
- Sets up password encoder (BCrypt)

**JwtAuthenticationFilter.java:**
- Runs before every request (except public endpoints)
- Extracts JWT token from `Authorization: Bearer <token>` header
- Validates token using `JwtUtil`
- Sets authentication in Spring Security context
- Allows request to proceed if valid, blocks if invalid

**JwtUtil.java:**
- Generates JWT tokens with username and role
- Validates tokens and extracts user information
- Uses secret key from `application.properties`

**CustomUserDetailsService.java:**
- Loads user from database by username
- Used by Spring Security for authentication

**Authentication Flow:**
```
1. User sends login request → AuthController
2. AuthService validates credentials
3. JwtUtil generates token
4. Token returned to user
5. User includes token in subsequent requests: Authorization: Bearer <token>
6. JwtAuthenticationFilter intercepts request
7. JwtUtil validates token
8. SecurityContext set with user details
9. Controller receives authenticated request
```

---

### 7. **Exception Handling** (`exception/`)
**Purpose**: Global exception handling

**File:**
- `GlobalExceptionHandler.java` - Catches all exceptions and returns consistent error responses

**How it connects:**
- Uses `@RestControllerAdvice` to catch exceptions from all controllers
- Handles validation errors (`@Valid` failures)
- Handles runtime exceptions (e.g., "Product not found")
- Returns appropriate HTTP status codes and error messages

**Example:**
```
Service throws RuntimeException("Product not found")
↓
GlobalExceptionHandler catches it
↓
Returns HTTP 400 with {"error": "Product not found"}
```

---

## 🔄 Complete Request Flow Example

### Example: Creating an Order

```
1. HTTP Request
   POST /api/orders
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   {
     "orderItems": [{"productId": 1, "quantity": 2}]
   }

2. JwtAuthenticationFilter
   - Extracts token from Authorization header
   - Validates token using JwtUtil
   - Loads user details using CustomUserDetailsService
   - Sets authentication in SecurityContext

3. SecurityConfig
   - Checks if /api/orders requires authentication (YES - USER/ADMIN)
   - Verifies user has required role
   - Allows request to proceed

4. OrderController.createOrder()
   - Receives OrderRequest DTO
   - Validates DTO using @Valid
   - Calls OrderService.createOrder()

5. OrderService.createOrder()
   - Gets authenticated user from SecurityContext
   - For each order item:
     - Fetches Product using ProductRepository
     - Checks if stock is sufficient
     - Creates OrderItem
     - Updates product stock
   - Calculates total amount
   - Saves Order using OrderRepository
   - Converts Order entity to OrderResponse DTO

6. OrderRepository.save()
   - Saves Order entity to database
   - Hibernate generates SQL INSERT statements

7. Database
   - Stores order and order items
   - Updates product stock

8. Response flows back:
   Database → Repository → Service → Controller → HTTP Response
   
9. HTTP Response
   HTTP 201 Created
   {
     "id": 1,
     "userId": 1,
     "totalAmount": 199.98,
     "status": "PENDING",
     "orderItems": [...]
   }
```

---

## 🔗 Component Dependencies

```
ECommerceApplication (Main)
    ↓
SecurityConfig (Security Setup)
    ↓
JwtAuthenticationFilter (Request Interceptor)
    ↓
Controllers (HTTP Handlers)
    ↓
Services (Business Logic)
    ↓
Repositories (Data Access)
    ↓
Models/Entities (Database Tables)
    ↓
MySQL Database
```

---

## 📊 Database Relationships

```
User (1) ────< (Many) Order
                    │
                    │ (1)
                    │
                    └───< (Many) OrderItem
                                │
                                │ (Many)
                                │
                                └───> (1) Product
```

**Explanation:**
- One User can have many Orders
- One Order can have many OrderItems
- One OrderItem belongs to one Product
- One Product can be in many OrderItems

---

## 🎯 Key Design Patterns

1. **Layered Architecture**: Controller → Service → Repository → Database
2. **DTO Pattern**: Separate request/response objects from entities
3. **Repository Pattern**: Abstract data access layer
4. **Dependency Injection**: Spring manages object creation and dependencies
5. **Filter Pattern**: JWT filter intercepts requests before controllers

---

## 🔐 Security Flow Summary

```
Public Endpoints:
/api/auth/** → No authentication required

Protected Endpoints:
/api/products (GET) → Public
/api/products (POST/PUT/DELETE) → Requires ADMIN role
/api/orders/** → Requires USER or ADMIN role
/api/admin/** → Requires ADMIN role

Authentication Process:
1. User registers/logs in → Gets JWT token
2. Token included in Authorization header
3. JwtAuthenticationFilter validates token
4. SecurityConfig checks role permissions
5. Request proceeds if authorized
```

---

## 📝 Configuration Files

### `application.properties`
- Database connection settings
- JWT secret and expiration
- JPA/Hibernate settings
- Server port configuration

### `pom.xml`
- Maven dependencies
- Spring Boot version
- Java version
- All required libraries (JWT, MySQL, Security, etc.)

---

This architecture ensures:
- **Separation of Concerns**: Each layer has a specific responsibility
- **Maintainability**: Easy to modify individual components
- **Security**: Centralized authentication and authorization
- **Scalability**: Can easily add new features or endpoints
- **Testability**: Each layer can be tested independently
