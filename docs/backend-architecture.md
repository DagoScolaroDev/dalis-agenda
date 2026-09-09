# Dalis Agenda - Backend Architecture

## 1. Overview

The Dalis Agenda backend will be built with ASP.NET Core using a layered architecture.

The goal is to keep business logic, application logic, infrastructure concerns, and HTTP concerns separated.

The backend will be organized into four .NET projects inside the same solution:

```text
DalisAgenda.Api
DalisAgenda.Application
DalisAgenda.Domain
DalisAgenda.Infrastructure
```

These projects will live inside:

```text
apps/api/
```

---

## 2. Solution Structure

```text
apps/api/
├── DalisAgenda.sln
│
├── DalisAgenda.Api/
│   ├── DalisAgenda.Api.csproj
│   ├── Controllers/
│   ├── Middleware/
│   └── Program.cs
│
├── DalisAgenda.Application/
│   ├── DalisAgenda.Application.csproj
│   ├── Appointments/
│   ├── Services/
│   ├── StaffMembers/
│   ├── Customers/
│   ├── Auth/
│   └── Common/
│
├── DalisAgenda.Domain/
│   ├── DalisAgenda.Domain.csproj
│   ├── Entities/
│   ├── Enums/
│   └── Common/
│
└── DalisAgenda.Infrastructure/
    ├── DalisAgenda.Infrastructure.csproj
    ├── Persistence/
    ├── Authentication/
    └── Services/
```

---

## 3. DalisAgenda.Domain

The Domain project contains the core business model.

It should not depend on ASP.NET Core, Entity Framework Core, PostgreSQL, JWT, or other infrastructure technologies.

It represents the main concepts of the system.

### Main Entities

```text
Tenant
User
StaffMember
Service
StaffMemberService
Customer
Appointment
AvailabilityRule
BlockedTime
```

### Example Structure

```text
DalisAgenda.Domain/
├── Entities/
│   ├── Tenant.cs
│   ├── User.cs
│   ├── StaffMember.cs
│   ├── Service.cs
│   ├── Customer.cs
│   ├── Appointment.cs
│   ├── AvailabilityRule.cs
│   └── BlockedTime.cs
│
├── Enums/
│   ├── AppointmentStatus.cs
│   └── UserRole.cs
│
└── Common/
```

### Responsibilities

The Domain layer is responsible for:

- Core entities
- Domain rules
- Domain enums
- Business invariants

Example:

```text
An appointment cannot end before it starts.

A service duration must be greater than zero.

A price cannot be negative.
```

---

## 4. DalisAgenda.Application

The Application project contains the use cases of the system.

It defines what the application can do.

Examples:

```text
CreateService
UpdateService
CreateAppointment
CancelAppointment
GetAvailableSlots
CreateStaffMember
GetAppointments
```

### Example Structure

```text
DalisAgenda.Application/
├── Auth/
├── Appointments/
├── Services/
├── StaffMembers/
├── Customers/
├── Tenants/
├── Availability/
└── Common/
```

A feature may contain:

```text
CreateService/
├── CreateServiceCommand.cs
├── CreateServiceHandler.cs
├── CreateServiceValidator.cs
└── CreateServiceResponse.cs
```

The exact folder structure may be simplified during implementation.

### Responsibilities

The Application layer is responsible for:

- Use cases
- Application logic
- DTOs
- Validation
- Interfaces
- Business workflows
- Coordination between domain entities and infrastructure

Example workflow:

```text
Create Appointment

1. Validate tenant
2. Validate service
3. Validate staff member
4. Verify staff member can perform the service
5. Check availability
6. Check appointment overlap
7. Create appointment
8. Save appointment
```

---

## 5. DalisAgenda.Infrastructure

The Infrastructure project contains implementations that interact with external systems.

### Example Structure

```text
DalisAgenda.Infrastructure/
├── Persistence/
│   ├── AppDbContext.cs
│   ├── Configurations/
│   └── Migrations/
│
├── Authentication/
│   ├── JwtTokenService.cs
│   └── PasswordHasher.cs
│
└── Services/
```

### Responsibilities

Infrastructure will handle:

- PostgreSQL
- Entity Framework Core
- Database migrations
- JWT token generation
- Password hashing
- External services
- Email providers in the future
- WhatsApp providers in the future

The Application layer may define interfaces.

Example:

```text
ITokenService
```

Infrastructure provides the implementation:

```text
JwtTokenService
```

---

## 6. DalisAgenda.Api

The Api project is the HTTP entry point of the backend.

It receives requests from the Angular frontend or other clients.

### Example Structure

```text
DalisAgenda.Api/
├── Controllers/
│   ├── AuthController.cs
│   ├── ServicesController.cs
│   ├── StaffMembersController.cs
│   ├── CustomersController.cs
│   └── AppointmentsController.cs
│
├── Middleware/
│   ├── ExceptionMiddleware.cs
│   └── TenantMiddleware.cs
│
└── Program.cs
```

### Responsibilities

The Api layer is responsible for:

- HTTP endpoints
- Controllers
- Middleware
- Authentication configuration
- Authorization
- Dependency injection
- Swagger
- HTTP request and response handling

Example:

```http
POST /api/services
```

The controller receives the HTTP request and delegates the work to the Application layer.

The controller should not contain complex business logic.

---

## 7. Project Dependencies

The projects should depend on each other in one direction.

Recommended dependency flow:

```text
Api
 ↓
Application
 ↓
Domain
```

Infrastructure can also depend on:

```text
Application
Domain
```

Conceptually:

```text
                 ┌──────────────┐
                 │     Api      │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │ Application  │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │    Domain    │
                 └──────────────┘

                 Infrastructure
                   │        │
                   └────────┘
                Application / Domain
```

Domain must remain the most independent project.

---

## 8. Request Flow

A typical request should follow this flow:

```text
Angular Frontend
      ↓
HTTP Request
      ↓
Api Controller
      ↓
Application Use Case
      ↓
Domain Rules
      ↓
Infrastructure
      ↓
PostgreSQL
```

Example:

```text
POST /api/appointments
```

Flow:

```text
AppointmentsController
        ↓
CreateAppointment
        ↓
Appointment validation
        ↓
Check availability
        ↓
AppDbContext
        ↓
PostgreSQL
```

---

## 9. Persistence

Entity Framework Core will be used as the ORM.

The main database context will be:

```text
AppDbContext
```

It will live in:

```text
DalisAgenda.Infrastructure/Persistence/
```

Example responsibilities:

```text
DbSet<Tenant>
DbSet<User>
DbSet<StaffMember>
DbSet<Service>
DbSet<Customer>
DbSet<Appointment>
DbSet<AvailabilityRule>
DbSet<BlockedTime>
```

Entity configurations should preferably be separated from the DbContext.

Example:

```text
Persistence/
├── AppDbContext.cs
└── Configurations/
    ├── TenantConfiguration.cs
    ├── UserConfiguration.cs
    ├── ServiceConfiguration.cs
    └── AppointmentConfiguration.cs
```

---

## 10. Multi-Tenancy

Dalis Agenda will initially use:

```text
Shared Database
Shared Schema
Tenant ID Column
```

All tenant-owned entities will contain:

```text
TenantId
```

The authenticated user will belong to one tenant.

The backend must determine the current tenant from the authenticated user.

Every tenant-scoped query must use the current tenant.

Example:

```text
Appointment Id = X
Tenant Id = CurrentTenant
```

A resource must never be loaded using only its ID.

---

## 11. Authentication

Authentication will initially use JWT.

Initial flow:

```text
User logs in
    ↓
Backend validates credentials
    ↓
Backend generates JWT
    ↓
Frontend stores authentication state
    ↓
Frontend sends JWT with protected requests
```

The token should contain essential claims such as:

```text
UserId
TenantId
Role
```

Passwords must never be stored directly.

Passwords will be stored as secure hashes.

---

## 12. Authorization

Initial roles:

```text
Owner
Admin
Staff
```

Example permissions:

### Owner

Full access to tenant management.

### Admin

Can manage:

- Services
- Staff
- Customers
- Appointments
- Schedules

### Staff

Can primarily access:

- Own appointments
- Own schedule
- Own blocked times

Authorization rules will be refined as features are implemented.

---

## 13. Validation

Input validation should happen before executing application logic.

Examples:

```text
Service name cannot be empty.

Service duration must be greater than zero.

Price cannot be negative.

Appointment start time is required.
```

FluentValidation may be used for request validation.

---

## 14. Error Handling

Errors should be handled centrally.

A middleware should convert application exceptions into consistent HTTP responses.

Example:

```json
{
  "error": "appointment_not_available",
  "message": "The selected appointment time is no longer available."
}
```

The API should avoid returning internal stack traces to clients.

---

## 15. API Conventions

Endpoints will use REST-style conventions.

Examples:

```text
GET    /api/services
GET    /api/services/{id}

POST   /api/services

PUT    /api/services/{id}

DELETE /api/services/{id}
```

Appointments:

```text
GET    /api/appointments
GET    /api/appointments/{id}

POST   /api/appointments

PATCH  /api/appointments/{id}/cancel
PATCH  /api/appointments/{id}/complete
```

Public booking endpoints may use:

```text
GET  /api/public/{tenantSlug}/services
GET  /api/public/{tenantSlug}/staff
GET  /api/public/{tenantSlug}/availability
POST /api/public/{tenantSlug}/appointments
```

These endpoints do not require staff authentication.

---

## 16. Dependency Injection

ASP.NET Core dependency injection will be used.

Services should be registered in `Program.cs` or extension methods.

Example concepts:

```text
Application services
Database context
JWT service
Current tenant service
Validators
```

The exact registrations will be implemented progressively.

---

## 17. Logging

The backend should log important events and errors.

Examples:

```text
Application started

User login failed

Appointment created

Unhandled exception

Database connection failure
```

The initial implementation can use ASP.NET Core built-in logging.

More advanced logging can be added later if needed.

---

## 18. Swagger

Swagger / OpenAPI should be enabled in development.

It will allow testing endpoints without requiring the Angular frontend.

Example:

```text
POST /api/services
GET /api/appointments
POST /api/auth/login
```

This will be especially useful during backend development.

---

## 19. Testing

Testing will be introduced progressively.

Initial priorities:

```text
Appointment availability calculation

Appointment overlap prevention

Tenant isolation

Service validation

Authentication
```

xUnit will be used for .NET tests.

A future test project may be added:

```text
DalisAgenda.Tests
```

or split into:

```text
DalisAgenda.UnitTests
DalisAgenda.IntegrationTests
```

This is not required during the initial setup.

---

## 20. Initial Architecture Scope

The first implementation should remain simple.

The architecture should not introduce unnecessary abstractions or patterns.

Initial focus:

- Clear project separation
- Domain entities
- Application use cases
- EF Core persistence
- PostgreSQL
- JWT authentication
- Tenant isolation
- REST API
- Swagger

Advanced patterns should only be introduced when they solve a real problem.

The main objective is to maintain a clean and understandable backend while learning and building the product progressively.
