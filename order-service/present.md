# Order Service — Screencast Presentation Script

Date: 2026-03-01

Presenter checklist (prep before recording)
- [ ] Project built and running locally: `mvn clean install && mvn spring-boot:run` (server runs on `http://localhost:8081`)
- [ ] H2 console accessible at `http://localhost:8081/h2-console` (JDBC URL: `jdbc:h2:mem:testdb`, user: `sa`, password: empty)
- [ ] Postman collection or terminal ready for curls
- [ ] (Optional) Docker Compose: `docker-compose up -d` if demo using MySQL — note MySQL config is commented in `application.properties` by default
- [ ] Reduce unnecessary desktop notifications; close unrelated apps
- [ ] Microphone and screen recorder configured; set pointer highlight on

Recording tips
- Focus camera on top-left where code editor shows file tree when explaining structure
- Zoom in on the controller/service method when explaining the flow; avoid line-by-line
- Pause 1–2 seconds after each curl command to let viewers read response
- Keep terminal font large and colors high-contrast

---

0:00 – 0:45 | Opening Context (≈45 seconds)

Narration (short, clear):
- "This is the Order Service — a REST API that manages Customers and Orders. It allows creating customers, placing orders with multiple items, updating order status, and fetching paginated lists of customers and orders."
- "Problem domain: e-commerce-like domain with Customers and their Orders. We need consistency (transactions), clear API boundaries, and predictable error handling."
- "Core design goals: separation of concerns via layers; DTOs for API contracts; transactional, safe updates; and pagination for large collections."

On-screen actions & visual cues:
- Show `README.md` briefly (top area) to set context
- Switch to `src/main/resources/application.properties` and point to `server.port=8081` and H2 config

---

0:45 – 2:00 | High-Level Architecture (≈75 seconds)

Narration:
- "Let's look at the project structure and the main layers: controllers, services, repositories, entities, and DTOs. These layers keep responsibilities separate: controllers handle HTTP mapping, services contain business rules and transactions, repositories handle persistence, and DTOs control the shape of API input/output."

On-screen actions & highlights:
- Open project tree in editor and highlight `controller/`, `service/`, `repository/`, `entity/`, `dto/`, `exception/` folders.
- Open `CustomerController.java` and highlight method signatures (no need to read full methods).
- Open `CustomerService.java` and point at `@Transactional` annotations.
- Open `CustomerRepository.java` (show it extends `JpaRepository`).

Talking points (data flow):
- "A request arrives → controller maps HTTP to a method and does validation (`@Valid`)."
- "Controller calls service → service applies business rules, starts transactions with `@Transactional` and calls repository methods."
- "Repository talks to the database (JPA/Hibernate). The service converts entities to DTOs for responses."

Quick visual demo: show one request flow (create customer):
- `POST /api/customers` (highlight controller)
- `CustomerService.createCustomer()` (highlight duplication check and `customerRepository.save()`)
- `CustomerResponse.from(customer)` mapping (highlight DTO static method)

---

2:00 – 3:30 | Data Model & Relationships (≈90 seconds)

Narration:
- "The data model has three main entities: Customer, Order, and OrderItem. A Customer has many Orders (one-to-many). An Order has many OrderItems (one-to-many)."
- "Relationships are enforced via JPA mappings (`@ManyToOne`, `@OneToMany`) and foreign keys in the database."

On-screen actions & visual cues:
- Open `entity/Customer.java` and point at `@OneToMany(mappedBy = "customer")`.
- Open `entity/Order.java` and point at `@ManyToOne` and `@OneToMany` for items.
- Switch to `README.md` diagram or show a quick ERD drawn in a slide or in the editor (if diagram exists). If not, show the code-based diagram from README.

API endpoint examples (curl) to expose relationships:
- Fetch all orders for a customer (paginated):

```bash
curl "http://localhost:8081/api/customers/1/orders?page=0&size=10"
```

Expected (example) response structure (short):

```json
{
  "data": [
    {
      "id": 5,
      "customerName": "John Doe",
      "status": "PENDING",
      "totalPrice": 1059.97,
      "items": [ ... ],
      "createdAt": "2026-02-28T10:30:00"
    }
  ],
  "page": 0,
  "size": 10,
  "totalElements": 1,
  "totalPages": 1
}
```

Explain briefly:
- "Each order includes a `customerName` field assembled from the `Customer` entity, and the items are nested."
- "Foreign keys are created by JPA; when retrieving orders, JPA handles joins lazily or eagerly depending on fetch settings."

---

3:30 – 5:00 | DTOs and Data Flow (≈90 seconds)

Narration:
- "DTOs define the API contract. They prevent internal-only fields from leaking (like internal IDs or audit details you don't want to expose) and enable shaping the response (for example `OrderResponse` contains `customerName` instead of full customer object)."

On-screen actions & highlights:
- Open `dto/OrderResponse.java` and highlight the `from(Order order)` method.
- Open `dto/CustomerResponse.java` and show fields returned.
- Show `CreateOrderRequest.java` and `OrderItemRequest.java` to demonstrate validation annotations.

Example mapping explanation:
- "`OrderResponse.from(order)` composes `customerName` using `order.getCustomer().getFirstName()` + last name, and maps `OrderItem` to `OrderItemResponse`. This is done in service layer before returning to controller."

Show a quick curl fetching a single order that demonstrates DTO output:

```bash
curl http://localhost:8081/api/orders/1
```

Example response (truncated):

```json
{
  "id": 1,
  "customerName": "John Doe",
  "status": "PENDING",
  "totalPrice": 123.45,
  "items": [
    {
      "productId": 101,
      "productName": "Widget",
      "quantity": 2,
      "unitPrice": 61.725
    }
  ],
  "createdAt": "2026-02-28T10:30:00",
  "updatedAt": "2026-02-28T10:30:00"
}
```

Explain: "Notice we don't expose the full `Customer` entity or internal DB IDs for other relations unless intentionally included."

---

5:00 – 6:30 | API Demonstration: Core Endpoints (≈90 seconds)

Narration:
- "Now I'll demonstrate core endpoints using curl (alternatively, show Postman). We'll create a customer, create an order, fetch the list and a single resource, and delete a resource."

On-screen actions (commands & expected responses):

1) Create a customer (POST)

```bash
curl -i -X POST http://localhost:8081/api/customers \
  -H "Content-Type: application/json" \
  -d '{"firstName":"John","lastName":"Doe","email":"john.doe@example.com"}'
```

Expect: 201 Created and JSON body with `id` and `createdAt`.

2) Create an order (POST)

```bash
curl -i -X POST http://localhost:8081/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": 1,
    "items": [
      {"productId":101,"productName":"Laptop","quantity":1,"unitPrice":999.99},
      {"productId":102,"productName":"Mouse","quantity":2,"unitPrice":29.99}
    ]
  }'
```

Expect: 201 Created with order payload including `totalPrice` and `status: PENDING`.

3) Get all orders (GET collection)

```bash
curl http://localhost:8081/api/orders?page=0&size=10
```

Expect: 200 OK with paged response.

4) Get single order (GET single)

```bash
curl http://localhost:8081/api/orders/1
```

Expect: 200 OK with `OrderResponse` fields.

5) Delete an order (DELETE)

```bash
curl -i -X DELETE http://localhost:8081/api/orders/1
```

Expect: 204 No Content (or 400/403 if deletion is forbidden due to business rules).

Explain status codes briefly as you run each command (201 Created, 200 OK, 204 No Content).

---

6:30 – 7:45 | Pagination & Date Handling (≈75 seconds)

Narration:
- "Pagination prevents overwhelming clients and servers when collections grow. The API supports `page` and `size` query params. Sorting by date is the default for lists. Dates are formatted as ISO-8601 (`LocalDateTime`) by Jackson."

On-screen actions & commands:
- Create multiple orders quickly (or assume some exist). Then show:

```bash
curl "http://localhost:8081/api/orders?page=0&size=2"

curl "http://localhost:8081/api/orders?page=1&size=2"
```

Explain response fields:
- `data`, `page`, `size`, `totalElements`, `totalPages`.

Date formatting:
- Open `OrderResponse.java` to show `LocalDateTime` fields; explain Jackson's default ISO format. Mention timezone handling assumption (server default timezone). If needed, show how to customize via `application.properties` or `Jackson2ObjectMapperBuilder`.

---

7:45 – 9:00 | Error Handling & Validation (≈75 seconds)

Narration:
- "We enforce validation via Jakarta Bean Validation on DTOs and handle errors centrally via `GlobalExceptionHandler`. Let's trigger two errors: a 404 for a non-existent resource and a 400 for validation failure."

On-screen actions & curl commands:

1) Trigger 404 — request a missing customer

```bash
curl -i http://localhost:8081/api/customers/99999
```

Expected response (HTTP 404):

```json
{
  "timestamp": "2026-03-01T12:00:00",
  "status": 404,
  "error": "NOT_FOUND",
  "message": "Customer with ID 99999 not found",
  "path": "/api/customers/99999"
}
```

2) Trigger 400 — invalid create order payload (no items)

```bash
curl -i -X POST http://localhost:8081/api/orders \
  -H "Content-Type: application/json" \
  -d '{"customerId":1, "items": []}'
```

Expected response (HTTP 400):

```json
{
  "timestamp": "2026-03-01T12:00:00",
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "Order must contain at least one item",
  "path": "/api/orders"
}
```

Explain briefly:
- "Validation annotations (`@NotNull`, `@NotEmpty`, `@Min`) enforce input rules; Spring's `MethodArgumentNotValidException` is mapped to a structured `ErrorResponse` by `GlobalExceptionHandler`. This provides consistent errors for API clients."

---

9:00 – 10:00 | Wrap-Up & Reflection (≈60 seconds)

Narration bullets:
- "Today we covered: project overview, architecture, data model, DTOs, core endpoints, pagination, and error handling."
- "Key design decisions: layered architecture for separation of concerns; DTOs for controlled API surface; transaction boundaries in service layer; and H2 for fast development with MySQL setup commented for production."
- "One challenge: ensuring stateful operations (like preventing updates on cancelled orders). We solved this by adding explicit business checks in `OrderService` and throwing domain-specific exceptions handled centrally."
- Closing: "Thanks — the repo contains this README and developer notes to reproduce the demo."

---

Troubleshooting quick tips
- If curl returns `Connection refused`: verify server started on port 8081 and not blocked by a firewall.
- If H2 console empty: remember H2 is in-memory; data is created when you run the app and will be lost on restart.
- If 500 on DB save: check console logs for SQL errors; ensure `spring.jpa.hibernate.ddl-auto` is appropriate for environment.

---

Optional Postman notes
- Import endpoints as requests with `{{baseUrl}}` set to `http://localhost:8081`.
- Save example request bodies from this script for quick replay.

---

End of present.md

