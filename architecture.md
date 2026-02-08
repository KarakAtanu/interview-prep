# Architecture Overview

This document describes the high-level architecture and solution structure for the project. It complements `copilot-instructions.md`, which focuses on coding style and clean code practices.

We follow an Onion Architecture style for Web API applications targeting modern .NET (e.g., .NET 8, .NET 10).

---

## 1. Architectural Goals

- **Separation of concerns** between domain, application logic, infrastructure, and API.
- **Testability** of core business logic without infrastructure concerns.
- **Maintainability** by isolating changes to specific layers.
- **Flexibility** to swap infrastructure concerns (databases, external services) with minimal impact on the core.

---

## 2. Onion Architecture Overview

Onion Architecture prioritizes **dependency inversion** and ensures the application is loosely coupled. It is structured around concentric layers, where dependencies flow inward. The core application logic is independent of external dependencies like UI, database, or frameworks.

The typical layers in the Onion Architecture are:

1. **Core (Domain Layer)**  
   Contains business entities, domain services, and interfaces.

2. **Application Layer**  
   Contains application-specific logic such as commands, queries, DTOs, and use cases.

3. **Infrastructure Layer**  
   Implements database access, third-party services, logging, etc.

4. **API Layer (Presentation/UI)**  
   Exposes APIs (e.g., ASP.NET Core controllers) to clients.

Dependencies always point **inward**: outer layers depend on inner layers, never the other way around.

---

## 3. Solution Structure

**Example solution structure (adapt to this repo as needed):**
```plaintext
MyApp/
│
├── MyApp.Domain/          (Core Domain Layer)
│   ├── Entities/          - Domain entities or aggregates
│   ├── Interfaces/        - Domain-driven interfaces (e.g., repositories, services)
│   ├── Events/            - Domain events
│   └── ValueObjects/      - Value-based domain objects
│
├── MyApp.Application/     (Application Layer)
│   ├── Interfaces/        - Use case interface definitions
│   ├── Models/            - DTOs, ViewModels
│   ├── Commands/          - Command handler logic
│   ├── Queries/           - Query handler logic
│   └── Services/          - Application-specific services
│
├── MyApp.Infrastructure/  (Infrastructure Layer)
│   ├── Persistence/
│   │   ├── DbContext.cs   - Database context for EF Core
│   │   ├── Configurations/- Model configurations
│   │   └── Repositories/  - Implementations of repository interfaces
│   ├── ExternalServices/  - Third-party service integrations
│   └── Logging/           - Logging implementations
│
├── MyApp.API/             (Presentation/UI Layer)
│   ├── Controllers/       - API controllers
│   └── Middleware/        - Custom middleware (e.g., exception handling)
│
├── MyApp.Tests/           (Test Project)
│   ├── UnitTests/
│   │   ├── Domain/        - Unit tests for core domain logic
│   │   ├── Application/   - Unit tests for application logic
│   └── IntegrationTests/  - Integration testing
```

---

### **3.1 Explanation of the Layers**

#### 1. **Domain Layer**
The **Domain Layer** is the core of the application and includes business logic and rules. It is **independent** of database, UI, and frameworks.

**Key Components**:
- **Entities**: Encapsulates domain objects and aggregates.
- **Interfaces**: Abstract contracts like repository interfaces (e.g., `IRepository`).
- **Domain Services**: Encapsulate domain-specific business rules.
- **Value Objects**: Immutable domain objects (like `Money`, `DateRange`).

**Example (Entity)**:
```csharp
namespace MyApp.Domain.Entities
{
    public class User
    {
        public Guid Id { get; private set; }
        public string Name { get; private set; }

        public User(Guid id, string name)
        {
            Id = id;
            Name = name;
        }
    }
}
```

---

#### 2. **Application Layer**
The **Application Layer** handles **use cases** by coordinating domain logic and external systems. It contains:
- **Commands and Command Handlers**: Command/request handling logic.
- **Queries and Query Handlers**: Query/read-side logic.
- **DTOs and Exceptions**: Data transfer objects and error handling.

**Example (Command)**:
```csharp
namespace MyApp.Application.Commands
{
    public class CreateUserCommand
    {
        public string Name { get; set; }
    }

    public interface ICreateUserHandler
    {
        Task HandleAsync(CreateUserCommand command);
    }
}
```

---

#### 3. **Infrastructure Layer**
The **Infrastructure Layer** implements low-level details such as:
- Database access (through Entity Framework Core, Dapper, etc.).
- External service integrations (e.g., payment gateways, REST APIs).
- Logging and file storage.

---

#### 4. **API Layer (Presentation Tier)**
The **API Layer** interacts directly with the user. In an ASP.NET Core application, this consists of controllers and middleware.

**Example (Controller)**:
```csharp
namespace MyApp.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class UsersController : ControllerBase
    {
        private readonly ICreateUserHandler _createUserHandler;

        public UsersController(ICreateUserHandler createUserHandler)
        {
            _createUserHandler = createUserHandler;
        }

        [HttpPost]
        public async Task<IActionResult> CreateUser(CreateUserCommand command)
        {
            await _createUserHandler.HandleAsync(command);
            return Ok();
        }
    }
}
```

---

## Technologies

- Runtime: .NET 10
- Web/API: ASP.NET Core Web API (Controllers)
- Data Access: Entity Framework Core (databse-first)
- Database: [SQL Server]
- DI & Configuration: Built-in .NET dependency injection and `IOptions<>`
- Logging: `ILogger<T>` with Serilog

---

## Dependency Rules

- `Domain` has **no dependencies** on other solution projects or external frameworks (except basic BCL).
- `Application` depends only on:
  - `Domain`
  - Cross-cutting libraries (logging abstractions, validation abstractions).
- `Infrastructure` depends on:
  - `Application`
  - `Domain`
  - External frameworks (EF Core, HTTP clients, logging implementations, etc.).
- `API` depends on:
  - `Application`
  - `Domain`
  - `Infrastructure`
  - ASP.NET Core framework.

Circular dependencies are not allowed. New code must respect these rules.

---

## Cross-Cutting Concerns

- **Error Handling**: Centralized via API middleware (`ExceptionHandlingMiddleware`) that:
  - Translates domain/application exceptions into standardized HTTP responses.
  - Logs unexpected errors with correlation ID.

- **Validation**:
  - Request validation is performed in the Application layer (e.g., FluentValidation) before executing use cases.
  - Controllers should remain thin and delegate to application services/handlers.

- **Transactions**:
  - Application handlers coordinate transactions using `IUnitOfWork` abstraction.
  - Infrastructure implements `IUnitOfWork` using EF Core `DbContext`.

- **Logging**:
  - Use `ILogger<T>` in each layer via DI.
  - Avoid logging directly in the Domain; instead, emit domain events that outer layers handle.

---

## Testing Strategy

- **Domain Unit Tests**:
  - Test domain entities, value objects, and domain services in isolation.
  - No database or framework dependencies.

- **Application Unit Tests**:
  - Test command/query handlers using mocked repositories and services.
  - Focus on business rules and use-case flow.

- **Integration Tests**:
  - Use the real `DbContext` (with test database) and real HTTP endpoints via `WebApplicationFactory`.
  - Verify happy-path and error scenarios at API boundary
