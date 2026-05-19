# Auth Service

This is a small Spring Boot Authentication Service that provides user registration and login using JWT (JSON Web Tokens). It exposes a minimal API to register users and obtain JWT tokens for authenticated access to other services.

---

## Table of Contents

- Project overview
- Prerequisites
- Build & Run
- Configuration
- API Endpoints
- Authentication flow
- Example requests (curl / PowerShell)
- Notes & security

---

## Project overview

Location: repository root

Key packages and classes:

- `com.example.auth.controller.AuthController` — exposes `/auth/register` and `/auth/login` endpoints
- `com.example.auth.security.JwtUtil` — generates and validates JWT tokens (currently uses a hard-coded secret)
- `com.example.auth.security.CustomUserDetailsService` — loads users from the database for authentication
- `com.example.auth.entity.User` — JPA entity for users
- `com.example.auth.repository.UserRepository` — Spring Data JPA repository
- `com.example.auth.config.SecurityConfig` — security configuration (permits `/auth/**`, Swagger and actuator endpoints)

The app runs on port `8081` by default (see `src/main/resources/application.properties`).

---

## Prerequisites

- Java 17+ (or the version configured in `pom.xml`)
- Maven 3.6+
- A running MySQL instance (or change the datasource to any supported DB)

---

## Build & Run

From the project root (PowerShell):

```powershell
mvn clean package
java -jar target/auth-service-0.0.1-SNAPSHOT.jar
```

The service will start on http://localhost:8081 unless you change `server.port` in `src/main/resources/application.properties`.

You can also run the application from your IDE (run `AuthServiceApplication` main class).

---

## Configuration

Settings are in `src/main/resources/application.properties`.

Important properties (current project defaults):

- `server.port=8081`
- `spring.datasource.url=jdbc:mysql://localhost:3306/auth_db`
- `spring.datasource.username` & `spring.datasource.password`
- JPA: `spring.jpa.hibernate.ddl-auto=update`

Note: JWT secret is currently defined inside `JwtUtil` as a static string. For production you should move it to configuration (environment variable or encrypted property) and avoid committing secrets to source control.

---

## API Endpoints

Base path: `/auth`

- POST /auth/register
  - Request: JSON { "username": "user1", "password": "pass" }
  - Response: plain text message `User registered successfully` on success

- POST /auth/login
  - Request: JSON { "username": "user1", "password": "pass" }
  - Response: JSON { "token": "<JWT_TOKEN>" }

Public endpoints (per `SecurityConfig`):

- `/auth/**` (register & login)
- Swagger UI: `/swagger-ui/**`
- OpenAPI docs: `/v3/api-docs/**`
- Actuator endpoints: `/actuator/**`

All other endpoints (if added) require an Authorization header with a valid JWT bearer token.

---

## Authentication flow

1. Client registers using POST `/auth/register` with username and password.
2. Client authenticates using POST `/auth/login`. On successful authentication, the server returns a JWT.
3. Client includes the token in requests to protected endpoints using the Authorization header:

   Authorization: Bearer <JWT_TOKEN>

The JWT contains the username as its subject and is signed using HMAC-SHA256 (see `JwtUtil`). Token expiry is set to 1 hour in the current implementation.

---

## Example requests

Register a user (PowerShell curl or curl):

```powershell
curl -Method POST -Uri http://localhost:8081/auth/register -ContentType 'application/json' -Body '{"username":"user1","password":"pass123"}'
```

Login and get token:

```powershell
$resp = curl -Method POST -Uri http://localhost:8081/auth/login -ContentType 'application/json' -Body '{"username":"user1","password":"pass123"}'
$resp.Content
```

Or using plain curl (Linux/macOS / Windows with curl):

```bash
curl -X POST http://localhost:8081/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"user1","password":"pass123"}'
```

Use the returned token to access protected endpoints:

```bash
curl -H "Authorization: Bearer <JWT_TOKEN>" http://localhost:8081/some/protected
```

Swagger UI is available at (after the app starts):

http://localhost:8081/swagger-ui/index.html

OpenAPI JSON:

http://localhost:8081/v3/api-docs

Actuator health (permitted in SecurityConfig):

http://localhost:8081/actuator/health

---

## Notes & security considerations

- The JWT secret is hard-coded in `JwtUtil`. Move it to `application.properties` or an environment variable for production, and load it via `@Value` or a configuration properties class.
- Passwords are stored hashed using BCrypt (`BCryptPasswordEncoder`) — this is good.
- Consider adding validation to registration (unique username check already enforced by DB unique constraint, but you may want to return a friendly error).
- Consider implementing token revocation/blacklist if needed.
- Use HTTPS in production to protect credentials in transit.

---

If you'd like, I can:

- move the JWT secret into `application.properties` and change `JwtUtil` to read it from configuration,
- add API examples using `httpie`/Postman collections,
- add more endpoints (user details, refresh token), or
- add tests for the controller and JWT behavior.

Tell me which of the above you'd like next and I'll implement it.

