# Docker Setup for E-Commerce API

## Prerequisites
- Docker
- Docker Compose

## Quick Start

### Using Docker Compose (Recommended)

1. **Build and start all services:**
   ```bash
   docker-compose up -d
   ```

2. **View logs:**
   ```bash
   docker-compose logs -f app
   ```

3. **Stop all services:**
   ```bash
   docker-compose down
   ```

4. **Stop and remove volumes (clears database):**
   ```bash
   docker-compose down -v
   ```

### Using Dockerfile Only

If you want to build and run the application separately:

1. **Build the image:**
   ```bash
   docker build -t ecommerce-app .
   ```

2. **Run the container (requires MySQL to be running separately):**
   ```bash
   docker run -p 8080:8080 \
     -e SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/ecommerce_db?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=UTC \
     -e SPRING_DATASOURCE_USERNAME=root \
     -e SPRING_DATASOURCE_PASSWORD=yourpassword \
     ecommerce-app
   ```

## Services

- **MySQL**: Running on port 3306
  - Database: `ecommerce_db`
  - Root password: `rootpassword`
  - User: `ecommerce_user`
  - Password: `ecommerce_password`

- **Spring Boot App**: Running on port 8080
  - API available at: `http://localhost:8080`

## Environment Variables

You can customize the configuration by modifying `docker-compose.yml` or setting environment variables:

- `SPRING_DATASOURCE_URL`: MySQL connection URL
- `SPRING_DATASOURCE_USERNAME`: MySQL username
- `SPRING_DATASOURCE_PASSWORD`: MySQL password
- `JWT_SECRET`: JWT secret key
- `JWT_EXPIRATION`: JWT expiration time in milliseconds

## Accessing the API

Once the containers are running, you can access:
- API Base URL: `http://localhost:8080/api`
- Register: `POST http://localhost:8080/api/auth/register`
- Login: `POST http://localhost:8080/api/auth/login`

## Troubleshooting

1. **Check if containers are running:**
   ```bash
   docker-compose ps
   ```

2. **Check MySQL logs:**
   ```bash
   docker-compose logs mysql
   ```

3. **Check application logs:**
   ```bash
   docker-compose logs app
   ```

4. **Restart services:**
   ```bash
   docker-compose restart
   ```
