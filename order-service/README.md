# Order Service

A comprehensive microservice for managing customer orders built with **Spring Boot 4.0.3**, **Spring Data JPA**, and **Jakarta Bean Validation**. This service provides a complete REST API for customer and order management with advanced features like pagination, transaction handling, and comprehensive error management.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Package Overview](#package-overview)
- [Entity Relationships](#entity-relationships)
- [API Documentation](#api-documentation)
- [Getting Started](#getting-started)
- [Database Configuration](#database-configuration)
- [Running the Application](#running-the-application)
- [API Examples](#api-examples)
- [Error Handling](#error-handling)

---

## 🎯 Project Overview

The Order Service is a REST API microservice designed to manage:
- **Customers**: Create, read, update, and delete customer records
- **Orders**: Create, read, update, and manage customer orders with status tracking
- **Order Items**: Manage individual items within orders with pricing and quantities

### Key Features
✅ Full CRUD operations for customers and orders
✅ Pagination support with customizable page size
✅ Transaction management with rollback on errors
✅ Comprehensive error handling and validation
✅ Order status lifecycle management (PENDING → CONFIRMED → SHIPPED → DELIVERED/CANCELLED)
✅ Price calculation with BigDecimal precision
✅ Timestamp tracking for audit purposes
✅ H2 in-memory database for development (MySQL ready for production)
✅ Jakarta Bean Validation for request validation
✅ Lombok for boilerplate reduction

---

## 🛠 Technology Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| Java | 21 | Programming language |
| Spring Boot | 4.0.3 | Framework |
| Spring Data JPA | Latest | ORM and database access |
| Jakarta Bean Validation | Latest | Request validation |
| H2 Database | Latest | In-memory database (development) |
| MySQL | 8.0+ | Production database |
| Lombok | Latest | Code generation (getters, setters, builders) |
| Maven | 3.8+ | Build tool |
| Jakarta Persistence API | Latest | JPA implementation |

---

## 🏗 Architecture

The application follows a **layered architecture** pattern:

```
┌─────────────────────────────────────────────┐
│         REST API Controllers                │
│  (CustomerController, OrderController)      │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Service Layer                       │
│  (CustomerService, OrderService)            │
│  - Business logic                           │
│  - Transaction management                   │
│  - Validation                               │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│      Repository Layer (Spring Data JPA)     │
│  (CustomerRepository, OrderRepository)      │
│  - Database access                          │
│  - Query methods                            │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Database Layer                      │
│  (H2 / MySQL)                               │
│  - Data persistence                         │
└─────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
order-service/
├── src/
│   ├── main/
│   │   ├── java/com/tus/orderservice/
│   │   │   ├── OrderServiceApplication.java          # Main application entry point
│   │   │   ├── controller/                           # REST API endpoints
│   │   │   ├── service/                              # Business logic layer
│   │   │   ├── repository/                           # Data access layer
│   │   │   ├── entity/                               # JPA entities
│   │   │   ├── dto/                                  # Data Transfer Objects
│   │   │   └── exception/                            # Custom exceptions & handlers
│   │   └── resources/
│   │       └── application.properties                # Configuration
│   └── test/
│       └── java/com/tus/orderservice/
│           └── OrderServiceApplicationTests.java    # Unit tests
├── pom.xml                                           # Maven configuration
├── Dockerfile                                        # Docker image definition
├── docker-compose.yml                                # Multi-container setup
├── DOCKER_SETUP.md                                   # Docker instructions
└── README.md                                         # This file
```

---

## 📦 Package Overview

### **1. Controller Package** (`controller/`)
**Purpose**: Handles HTTP requests and responses

| Class | Responsibility |
|-------|-----------------|
| **CustomerController** | Manages customer-related HTTP endpoints |
| **OrderController** | Manages order-related HTTP endpoints |

**Key Responsibilities**:
- Map HTTP requests to service methods
- Handle request validation via `@Valid` annotations
- Return appropriate HTTP status codes
- Serialize/deserialize JSON payloads

**Endpoints Summary**:
- `POST /api/customers` - Create customer
- `GET /api/customers` - List all customers (paginated)
- `GET /api/customers/{id}` - Get single customer
- `PUT /api/customers/{id}` - Update customer
- `DELETE /api/customers/{id}` - Delete customer
- `POST /api/orders` - Create order
- `GET /api/orders` - List all orders (paginated)
- `GET /api/orders/{id}` - Get single order
- `PUT /api/orders/{id}` - Update order
- `PATCH /api/orders/{id}/status` - Update order status
- `DELETE /api/orders/{id}` - Delete order

---

### **2. Service Package** (`service/`)
**Purpose**: Contains business logic and transaction management

| Class | Responsibility |
|-------|-----------------|
| **CustomerService** | Customer business operations |
| **OrderService** | Order business operations |

**Key Responsibilities**:
- Implement business logic
- Validate business rules
- Handle transactions with `@Transactional`
- Call repository methods for data access
- Throw domain-specific exceptions
- Convert entities to DTOs

**CustomerService Methods**:
- `createCustomer()` - Create new customer with email uniqueness validation
- `getCustomerById()` - Retrieve customer by ID
- `getAllCustomers()` - Retrieve paginated customer list
- `updateCustomer()` - Update customer details
- `deleteCustomer()` - Delete customer record

**OrderService Methods**:
- `createOrder()` - Create order with items and price calculation
- `getOrderById()` - Retrieve order by ID
- `getAllOrders()` - Retrieve paginated order list
- `getOrdersByCustomer()` - Get customer's orders
- `updateOrder()` - Update full order details
- `updateOrderStatus()` - Update order status with validation
- `deleteOrder()` - Delete order (prevents deletion of confirmed orders)

---

### **3. Repository Package** (`repository/`)
**Purpose**: Data access layer using Spring Data JPA

| Class | Responsibility |
|-------|-----------------|
| **CustomerRepository** | CRUD operations for customers |
| **OrderRepository** | CRUD operations for orders |

**Key Features**:
- Extends `JpaRepository<T, ID>` for automatic CRUD methods
- Custom query methods:
  - `CustomerRepository.existsByEmail()` - Check email uniqueness
  - `OrderRepository.findByCustomerId()` - Get orders by customer

---

### **4. Entity Package** (`entity/`)
**Purpose**: JPA entities representing database tables

| Class | Responsibility |
|-------|-----------------|
| **Customer** | Represents a customer record |
| **Order** | Represents an order record |
| **OrderItem** | Represents an item within an order |
| **OrderStatus** | Enum for order status values |

**Entity Details**:

#### **Customer Entity**
- Fields: `id`, `firstName`, `lastName`, `email`, `createdAt`
- Relationships: One-to-many with Order
- Constraints: Email must be unique, firstName/lastName/email required
- Audit: Auto-generated `createdAt` timestamp

#### **Order Entity**
- Fields: `id`, `customer`, `status`, `totalPrice`, `items`, `createdAt`, `updatedAt`
- Relationships: Many-to-one with Customer, One-to-many with OrderItem
- Constraints: totalPrice uses BigDecimal with 10 digits, 2 decimals
- Audit: Auto-generated `createdAt` and `updatedAt` timestamps
- Default Status: PENDING on creation

#### **OrderItem Entity**
- Fields: `id`, `order`, `productId`, `productName`, `quantity`, `unitPrice`
- Relationships: Many-to-one with Order
- Constraints: All fields required

#### **OrderStatus Enum**
- Values: PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
- Used for order lifecycle management

---

### **5. DTO Package** (`dto/`)
**Purpose**: Data Transfer Objects for request/response mapping

| Class | Purpose |
|-------|---------|
| **CustomerResponse** | Response DTO for customer data |
| **CreateCustomerRequest** | Request DTO for customer creation |
| **OrderResponse** | Response DTO for order data |
| **CreateOrderRequest** | Request DTO for order creation |
| **OrderItemRequest** | Request DTO for order items |
| **OrderItemResponse** | Response DTO for order items |
| **UpdateOrderStatusRequest** | Request DTO for status updates |
| **PagedResponse<T>** | Generic response wrapper for paginated results |
| **ErrorResponse** | Response DTO for error handling |

**Key Features**:
- Separate request/response objects for decoupling
- All DTOs use Lombok annotations for code generation
- Response DTOs include `from()` static methods for entity conversion
- Validation annotations on request DTOs
- Paged response includes metadata (page, size, totalElements, totalPages)

---

### **6. Exception Package** (`exception/`)
**Purpose**: Custom exception handling

| Class | Purpose |
|-------|---------|
| **ResourceNotFoundException** | Thrown when entity not found |
| **DuplicateResourceException** | Thrown when duplicate entry detected |
| **InvalidOrderStateException** | Thrown on invalid order operations |
| **GlobalExceptionHandler** | Spring `@ControllerAdvice` for centralized error handling |

**Error Handling Flow**:
1. Service throws custom exception
2. GlobalExceptionHandler catches it
3. Returns standardized ErrorResponse with HTTP status
4. Client receives error in consistent format

**Example Error Response**:
```json
{
  "timestamp": "2026-02-28T10:30:00",
  "status": 404,
  "error": "NOT_FOUND",
  "message": "Customer with ID 999 not found",
  "path": "/api/customers/999"
}
```

---

## 🔗 Entity Relationships

```
┌─────────────────────┐         ┌──────────────────┐
│     CUSTOMER        │         │      ORDER       │
├─────────────────────┤    1:N  ├──────────────────┤
│ id (PK)             │◄────────│ id (PK)          │
│ firstName           │         │ customer_id (FK) │
│ lastName            │         │ status           │
│ email (UNIQUE)      │         │ totalPrice       │
│ createdAt           │         │ createdAt        │
└─────────────────────┘         │ updatedAt        │
                                └────────┬─────────┘
                                         │
                                         │ 1:N
                                         │
                                ┌────────▼─────────┐
                                │    ORDER_ITEM    │
                                ├──────────────────┤
                                │ id (PK)          │
                                │ order_id (FK)    │
                                │ productId        │
                                │ productName      │
                                │ quantity         │
                                │ unitPrice        │
                                └──────────────────┘
```

---

## 📡 API Documentation

### Customer Endpoints

#### Create Customer
```
POST /api/customers
Content-Type: application/json

Request Body:
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com"
}

Response: 201 Created
{
  "id": 1,
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "createdAt": "2026-02-28T10:30:00"
}
```

#### Get All Customers (Paginated)
```
GET /api/customers?page=0&size=10

Response: 200 OK
{
  "data": [...],
  "page": 0,
  "size": 10,
  "totalElements": 25,
  "totalPages": 3
}
```

#### Get Customer by ID
```
GET /api/customers/{id}

Response: 200 OK
{
  "id": 1,
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "createdAt": "2026-02-28T10:30:00"
}
```

#### Update Customer
```
PUT /api/customers/{id}
Content-Type: application/json

Request Body:
{
  "firstName": "Jane",
  "lastName": "Doe",
  "email": "jane.doe@example.com"
}

Response: 200 OK
{
  "id": 1,
  "firstName": "Jane",
  "lastName": "Doe",
  "email": "jane.doe@example.com",
  "createdAt": "2026-02-28T10:30:00"
}
```

#### Delete Customer
```
DELETE /api/customers/{id}

Response: 204 No Content
```

---

### Order Endpoints

#### Create Order
```
POST /api/orders
Content-Type: application/json

Request Body:
{
  "customerId": 1,
  "items": [
    {
      "productId": 101,
      "productName": "Laptop",
      "quantity": 1,
      "unitPrice": 999.99
    },
    {
      "productId": 102,
      "productName": "Mouse",
      "quantity": 2,
      "unitPrice": 29.99
    }
  ]
}

Response: 201 Created
{
  "id": 5,
  "customerId": 1,
  "status": "PENDING",
  "totalPrice": 1059.97,
  "items": [...],
  "createdAt": "2026-02-28T10:30:00",
  "updatedAt": "2026-02-28T10:30:00"
}
```

#### Get All Orders (Paginated)
```
GET /api/orders?page=0&size=10

Response: 200 OK
{
  "data": [...],
  "page": 0,
  "size": 10,
  "totalElements": 50,
  "totalPages": 5
}
```

#### Get Order by ID
```
GET /api/orders/{id}

Response: 200 OK
{
  "id": 5,
  "customerId": 1,
  "status": "PENDING",
  "totalPrice": 1059.97,
  "items": [...],
  "createdAt": "2026-02-28T10:30:00",
  "updatedAt": "2026-02-28T10:30:00"
}
```

#### Update Order
```
PUT /api/orders/{id}
Content-Type: application/json

Request Body:
{
  "customerId": 1,
  "items": [
    {
      "productId": 101,
      "productName": "Laptop",
      "quantity": 2,
      "unitPrice": 999.99
    }
  ]
}

Response: 200 OK
{
  "id": 5,
  "customerId": 1,
  "status": "PENDING",
  "totalPrice": 1999.98,
  "items": [...],
  "createdAt": "2026-02-28T10:30:00",
  "updatedAt": "2026-02-28T10:30:00"
}
```

#### Update Order Status
```
PATCH /api/orders/{id}/status
Content-Type: application/json

Request Body:
{
  "status": "CONFIRMED"
}

Response: 200 OK
{
  "id": 5,
  "customerId": 1,
  "status": "CONFIRMED",
  "totalPrice": 1999.98,
  "items": [...],
  "createdAt": "2026-02-28T10:30:00",
  "updatedAt": "2026-02-28T10:30:00"
}
```

#### Delete Order
```
DELETE /api/orders/{id}

Response: 204 No Content
```

#### Get Customer's Orders
```
GET /api/customers/{customerId}/orders?page=0&size=10

Response: 200 OK
{
  "data": [...],
  "page": 0,
  "size": 10,
  "totalElements": 5,
  "totalPages": 1
}
```

---

## 🚀 Getting Started

### Prerequisites
- Java 21 or higher
- Maven 3.8+
- Git (optional)

### Installation Steps

1. **Clone or download the project**
```bash
git clone <repository-url>
cd order-service
```

2. **Build the project**
```bash
mvn clean install
```

3. **Run the application**
```bash
mvn spring-boot:run
```

The service will start on **http://localhost:8081**

---

## 🗄️ Database Configuration

### Current Configuration: H2 (In-Memory)

The application is configured to use **H2 in-memory database** by default for quick development and testing.

**Configuration** (in `application.properties`):
```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# H2 Console (accessible at http://localhost:8081/h2-console)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA/Hibernate Configuration
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```

### MySQL Configuration (Production-Ready)

To switch to MySQL, uncomment the MySQL section in `application.properties`:

```properties
# MySQL Database Configuration
spring.datasource.url=jdbc:mysql://mysql:3306/order_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=orderuser
spring.datasource.password=orderpass
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate Configuration
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.ddl-auto=update
```

### Database Schema

The schema is automatically created by Hibernate based on entity annotations:

**Tables Created**:
- `customers` - Customer records
- `orders` - Order records
- `order_items` - Items within orders

---

## 🐳 Running the Application

### Option 1: Local Development (H2 Database)

```bash
# Build
mvn clean install

# Run
mvn spring-boot:run

# Access API
curl http://localhost:8081/api/customers

# Access H2 Console
# Open browser: http://localhost:8081/h2-console
# JDBC URL: jdbc:h2:mem:testdb
# Username: sa
# Password: (leave empty)
```

### Option 2: Docker with MySQL

```bash
# Build and start services
docker-compose up -d

# Verify services
docker-compose ps

# View logs
docker-compose logs order-service
docker-compose logs mysql

# Stop services
docker-compose down

# Clean up (including volumes)
docker-compose down -v
```

**Docker Compose Services**:
- **MySQL**: `localhost:3306` (credentials in docker-compose.yml)
- **Order Service**: `http://localhost:8080/api`

---

## 📝 API Examples

### Using cURL

```bash
# Create a customer
curl -X POST http://localhost:8081/api/customers \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com"
  }'

# Get all customers
curl http://localhost:8081/api/customers?page=0&size=10

# Create an order
curl -X POST http://localhost:8081/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": 1,
    "items": [
      {
        "productId": 1,
        "productName": "Laptop",
        "quantity": 1,
        "unitPrice": 999.99
      }
    ]
  }'

# Update order status
curl -X PATCH http://localhost:8081/api/orders/1/status \
  -H "Content-Type: application/json" \
  -d '{"status": "CONFIRMED"}'

# Delete an order
curl -X DELETE http://localhost:8081/api/orders/1
```

### Using Postman

1. Import the request collection
2. Set `{{baseUrl}}` to `http://localhost:8081`
3. Each endpoint folder contains example requests

---

## ⚠️ Error Handling

The application implements centralized error handling using Spring's `@ControllerAdvice`.

### Exception Types

| Exception | HTTP Status | Scenario |
|-----------|------------|----------|
| **ResourceNotFoundException** | 404 NOT FOUND | Entity not found by ID |
| **DuplicateResourceException** | 409 CONFLICT | Email already exists for customer |
| **InvalidOrderStateException** | 400 BAD REQUEST | Invalid order status transition |
| **MethodArgumentNotValidException** | 400 BAD REQUEST | Validation error on request body |

### Error Response Format

```json
{
  "timestamp": "2026-02-28T10:30:00.123456",
  "status": 404,
  "error": "NOT_FOUND",
  "message": "Customer with ID 999 not found",
  "path": "/api/customers/999"
}
```

### Order Status Validation Rules

- ✅ PENDING → CONFIRMED (allowed)
- ✅ PENDING → CANCELLED (allowed)
- ✅ CONFIRMED → SHIPPED (allowed)
- ✅ CONFIRMED → CANCELLED (allowed)
- ✅ SHIPPED → DELIVERED (allowed)
- ❌ CONFIRMED → PENDING (NOT allowed - no reverting)
- ❌ CANCELLED → * (NOT allowed - cannot update cancelled orders)

---

## 🔧 Configuration Details

### Application Properties

| Property | Default | Description |
|----------|---------|-------------|
| `spring.application.name` | order-service | Application identifier |
| `server.port` | 8081 | Port where service runs |
| `spring.jpa.show-sql` | true | Log SQL queries |
| `spring.jpa.hibernate.ddl-auto` | create-drop | Auto-create schema on startup |

### Maven Configuration

- **Java Version**: 21
- **Parent POM**: Spring Boot 4.0.3
- **Build Tool**: Maven 3.8+
- **Annotation Processing**: Lombok

---

## 📚 Additional Resources

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Jakarta Bean Validation](https://jakarta.ee/specifications/bean-validation/)
- [Lombok Project](https://projectlombok.org/)
- [MySQL Documentation](https://dev.mysql.com/doc/)

---

## 📄 License

This project is part of a microservices architecture for Master's program coursework.

---

## 👤 Author

Developed as part of Microservices Architecture course.

---

**Last Updated**: February 28, 2026

