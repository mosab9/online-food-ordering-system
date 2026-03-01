# Order Service - Docker Setup

This guide explains how to run the Order Service with MySQL using Docker Compose.

## Prerequisites

- Docker Desktop installed and running
- Docker Compose (usually included with Docker Desktop)

## Quick Start

### 1. Start all services (MySQL + Order Service)

```bash
docker-compose up -d
```

This will:
- Build the Spring Boot application image
- Start MySQL 8.0 database
- Start the Order Service application
- Create and configure the `order_db` database

### 2. Verify services are running

```bash
docker-compose ps
```

You should see both `order_service_mysql` and `order_service_app` running.

### 3. Access the application

- **Order Service API**: http://localhost:8080
- **MySQL**: localhost:3306

### 4. Check logs

View application logs:
```bash
docker-compose logs order-service
```

View MySQL logs:
```bash
docker-compose logs mysql
```

## Database Configuration

The docker-compose file automatically creates:
- **Database**: `order_db`
- **User**: `orderuser`
- **Password**: `orderpass`

These credentials are pre-configured in `application.properties`.

## Development Mode (Local)

If you want to run only MySQL for local development:

```bash
docker-compose up mysql -d
```

Then update `application.properties` to use:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/order_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=orderuser
spring.datasource.password=orderpass
```

And run the Spring Boot application from your IDE.

## Stopping Services

```bash
docker-compose down
```

To also remove the MySQL data volume:
```bash
docker-compose down -v
```

## Rebuilding the Application

If you make changes to the code:

```bash
docker-compose up -d --build
```

## Troubleshooting

### Port already in use
If port 3306 or 8080 is already in use, modify the ports in `docker-compose.yml`:
```yaml
ports:
  - "3307:3306"  # MySQL on 3307
  - "8081:8080"  # App on 8081
```

### Database connection issues
- Wait 10-15 seconds after starting for MySQL to fully initialize
- Check logs: `docker-compose logs mysql`
- Ensure MySQL container is healthy: `docker-compose ps`

### Rebuild from scratch
```bash
docker-compose down -v
docker system prune
docker-compose up -d --build
```

