# E-Commerce REST API

A complete e-commerce REST API built with Spring Boot, featuring JWT authentication, MySQL database, and role-based access control (Admin and User roles).

## 🚀 Features

- **JWT Authentication** - Secure token-based authentication
- **Role-Based Access Control** - Admin and User roles with different permissions
- **MySQL Database** - Persistent data storage with JPA/Hibernate
- **RESTful API** - Clean REST endpoints for all operations
- **Docker Support** - Easy deployment with Docker and Docker Compose
- **Input Validation** - Request validation using Jakarta Validation
- **Exception Handling** - Global exception handler for consistent error responses

## 📋 Prerequisites

- Java 21
- Maven 3.9+
- MySQL 8.0+ (or use Docker)
- Docker & Docker Compose (optional)

## 🛠️ Tech Stack

- **Framework**: Spring Boot 4.0.1
- **Database**: MySQL 8.0
- **Security**: Spring Security + JWT (jjwt 0.12.3)
- **ORM**: Spring Data JPA / Hibernate
- **Build Tool**: Maven
- **Language**: Java 21

## 📦 Installation

### Option 1: Local Development

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd E-Commerce
   ```

2. **Configure MySQL**
   - Create a MySQL database named `ecommerce_db`
   - Update `src/main/resources/application.properties` with your MySQL credentials:
     ```properties
     spring.datasource.username=your_username
     spring.datasource.password=your_password
     ```

3. **Build the project**
   ```bash
   mvn clean install
   ```

4. **Run the application**
   ```bash
   mvn spring-boot:run
   ```

### Option 2: Docker (Recommended)

1. **Start all services**
   ```bash
   docker-compose up -d
   ```

2. **View logs**
   ```bash
   docker-compose logs -f app
   ```

For detailed Docker instructions, see [README-Docker.md](README-Docker.md)

## 🔐 API Endpoints

### Authentication (`/api/auth`)

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/auth/register` | Register a new user | Public |
| POST | `/api/auth/login` | Login and get JWT token | Public |

### Products (`/api/products`)

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/products` | Get all products | Public |
| GET | `/api/products/{id}` | Get product by ID | Public |
| POST | `/api/products` | Create a new product | Admin |
| PUT | `/api/products/{id}` | Update a product | Admin |
| DELETE | `/api/products/{id}` | Delete a product | Admin |

### Orders (`/api/orders`)

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/orders` | Create a new order | User, Admin |
| GET | `/api/orders` | Get all orders (own orders for users, all for admins) | User, Admin |
| GET | `/api/orders/{id}` | Get order by ID | User, Admin |

### Admin (`/api/admin`)

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| PUT | `/api/admin/orders/{id}/status` | Update order status | Admin |

## 📝 API Usage Examples

### 1. Register a New User

```bash
POST http://localhost:8080/api/auth/register
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "username": "john_doe",
  "role": "USER",
  "message": "User registered successfully"
}
```

### 2. Login

```bash
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "username": "john_doe",
  "password": "password123"
}
```

### 3. Get All Products (No Authentication Required)

```bash
GET http://localhost:8080/api/products
```

### 4. Create an Order (Requires Authentication)

```bash
POST http://localhost:8080/api/orders
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "orderItems": [
    {
      "productId": 1,
      "quantity": 2
    },
    {
      "productId": 2,
      "quantity": 1
    }
  ]
}
```

### 5. Create a Product (Admin Only)

```bash
POST http://localhost:8080/api/products
Authorization: Bearer <admin-jwt-token>
Content-Type: application/json

{
  "name": "Laptop",
  "description": "High-performance laptop",
  "price": 999.99,
  "stock": 50
}
```

## 🔑 Authentication

All protected endpoints require a JWT token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

**Token Expiration**: 24 hours (86400000 milliseconds)

## 👥 User Roles

### USER Role
- Can view products
- Can create orders
- Can view their own orders

### ADMIN Role
- All USER permissions
- Can create, update, and delete products
- Can view all orders
- Can update order status

## 📁 Project Structure

```
E-Commerce/
├── src/main/java/com/personal/E_Commerce/
│   ├── ECommerceApplication.java       # Main application entry point
│   ├── controller/                     # REST controllers
│   │   ├── AuthController.java
│   │   ├── ProductController.java
│   │   ├── OrderController.java
│   │   └── AdminController.java
│   ├── service/                        # Business logic
│   │   ├── AuthService.java
│   │   ├── ProductService.java
│   │   └── OrderService.java
│   ├── repository/                     # Data access layer
│   │   ├── UserRepository.java
│   │   ├── ProductRepository.java
│   │   ├── OrderRepository.java
│   │   └── OrderItemRepository.java
│   ├── model/                          # Entity models
│   │   ├── User.java
│   │   ├── Product.java
│   │   ├── Order.java
│   │   ├── OrderItem.java
│   │   ├── Role.java
│   │   └── OrderStatus.java
│   ├── dto/                            # Data Transfer Objects
│   │   ├── LoginRequest.java
│   │   ├── RegisterRequest.java
│   │   ├── AuthResponse.java
│   │   ├── ProductRequest.java
│   │   ├── ProductResponse.java
│   │   ├── OrderRequest.java
│   │   └── OrderResponse.java
│   ├── security/                       # Security configuration
│   │   ├── SecurityConfig.java
│   │   ├── JwtUtil.java
│   │   ├── JwtAuthenticationFilter.java
│   │   └── CustomUserDetailsService.java
│   └── exception/                      # Exception handling
│       └── GlobalExceptionHandler.java
├── src/main/resources/
│   └── application.properties          # Application configuration
├── Dockerfile                          # Docker image definition
├── docker-compose.yml                  # Docker Compose configuration
└── pom.xml                             # Maven dependencies
```

For detailed architecture explanation, see [ARCHITECTURE.md](ARCHITECTURE.md)

## ⚙️ Configuration

### Database Configuration

Edit `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce_db
spring.datasource.username=root
spring.datasource.password=your_password
```

### JWT Configuration

```properties
jwt.secret=your-secret-key-min-64-characters
jwt.expiration=86400000  # 24 hours in milliseconds
```

## 🧪 Testing

Run tests with:
```bash
mvn test
```

## 📊 Database Schema

- **users** - User accounts with roles
- **products** - Product catalog
- **orders** - Customer orders
- **order_items** - Order line items

## 🐳 Docker

See [README-Docker.md](README-Docker.md) for detailed Docker setup instructions.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 📄 License

This project is licensed under the MIT License.

## 👨‍💻 Author

Personal E-Commerce Project

## 📞 Support

For issues and questions, please open an issue in the repository.
